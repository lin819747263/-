我们上一篇讲了Beandefine的加载过程，但BeanDefine是一个中间产物，并不是最总的Bean

我们的Bean初始化的世纪是什么呢，由于BeanFactory是懒加载，我们只有在第一次用的时候才会进行加载

所以我们从BeanFactory#getBean说起

真正的实现从AbstractBeanFactory#doGetBean开始

```
protected <T> T doGetBean(
			String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)
			throws BeansException {

		// 1转换Bean的名称
		String beanName = transformedBeanName(name);
		Object beanInstance;

		// 2检查是否有缓冲的Bean
		Object sharedInstance = getSingleton(beanName);
		if (sharedInstance != null && args == null) {
			if (logger.isTraceEnabled()) {
				if (isSingletonCurrentlyInCreation(beanName)) {
					logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
							"' that is not fully initialized yet - a consequence of a circular reference");
				}
				else {
					logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
				}
			}
			beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, null);
		}

		else {
			// Fail if we're already creating this bean instance:
			// We're assumably within a circular reference.
			if (isPrototypeCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(beanName);
			}

			// Check if bean definition exists in this factory.
			BeanFactory parentBeanFactory = getParentBeanFactory();
			if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
				// Not found -> check parent.
				String nameToLookup = originalBeanName(name);
				if (parentBeanFactory instanceof AbstractBeanFactory) {
					return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
							nameToLookup, requiredType, args, typeCheckOnly);
				}
				else if (args != null) {
					// Delegation to parent with explicit args.
					return (T) parentBeanFactory.getBean(nameToLookup, args);
				}
				else if (requiredType != null) {
					// No args -> delegate to standard getBean method.
					return parentBeanFactory.getBean(nameToLookup, requiredType);
				}
				else {
					return (T) parentBeanFactory.getBean(nameToLookup);
				}
			}

			if (!typeCheckOnly) {
				markBeanAsCreated(beanName);
			}

			StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate")
					.tag("beanName", name);
			try {
				if (requiredType != null) {
					beanCreation.tag("beanType", requiredType::toString);
				}
				RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
				checkMergedBeanDefinition(mbd, beanName, args);

				// Guarantee initialization of beans that the current bean depends on.
				String[] dependsOn = mbd.getDependsOn();
				if (dependsOn != null) {
					for (String dep : dependsOn) {
						if (isDependent(beanName, dep)) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
						}
						registerDependentBean(dep, beanName);
						try {
							getBean(dep);
						}
						catch (NoSuchBeanDefinitionException ex) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"'" + beanName + "' depends on missing bean '" + dep + "'", ex);
						}
					}
				}

				// Create bean instance.
				if (mbd.isSingleton()) {
					sharedInstance = getSingleton(beanName, () -> {
						try {
						// 3.创建Bean
							return createBean(beanName, mbd, args);
						}
						catch (BeansException ex) {
							// Explicitly remove instance from singleton cache: It might have been put there
							// eagerly by the creation process, to allow for circular reference resolution.
							// Also remove any beans that received a temporary reference to the bean.
							destroySingleton(beanName);
							throw ex;
						}
					});
					beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}

				else if (mbd.isPrototype()) {
					// It's a prototype -> create a new instance.
					Object prototypeInstance = null;
					try {
						beforePrototypeCreation(beanName);
						prototypeInstance = createBean(beanName, mbd, args);
					}
					finally {
						afterPrototypeCreation(beanName);
					}
					beanInstance = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
				}

				else {
					String scopeName = mbd.getScope();
					if (!StringUtils.hasLength(scopeName)) {
						throw new IllegalStateException("No scope name defined for bean '" + beanName + "'");
					}
					Scope scope = this.scopes.get(scopeName);
					if (scope == null) {
						throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
					}
					try {
						Object scopedInstance = scope.get(beanName, () -> {
							beforePrototypeCreation(beanName);
							try {
								return createBean(beanName, mbd, args);
							}
							finally {
								afterPrototypeCreation(beanName);
							}
						});
						beanInstance = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
					}
					catch (IllegalStateException ex) {
						throw new ScopeNotActiveException(beanName, scopeName, ex);
					}
				}
			}
			catch (BeansException ex) {
				beanCreation.tag("exception", ex.getClass().toString());
				beanCreation.tag("message", String.valueOf(ex.getMessage()));
				cleanupAfterBeanCreationFailure(beanName);
				throw ex;
			}
			finally {
				beanCreation.end();
			}
		}

		return adaptBeanInstance(name, beanInstance, requiredType);
	}
```

2 DefaultSingletonBeanRegistry#getSingleton(java.lang.String, boolean)

```
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		// Quick check for existing instance without full singleton lock
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			singletonObject = this.earlySingletonObjects.get(beanName);
			if (singletonObject == null && allowEarlyReference) {
				synchronized (this.singletonObjects) {
					// Consistent creation of early reference within full singleton lock
					singletonObject = this.singletonObjects.get(beanName);
					if (singletonObject == null) {
						singletonObject = this.earlySingletonObjects.get(beanName);
						if (singletonObject == null) {
							ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
							if (singletonFactory != null) {
								singletonObject = singletonFactory.getObject();
								this.earlySingletonObjects.put(beanName, singletonObject);
								this.singletonFactories.remove(beanName);
							}
						}
					}
				}
			}
		}
		return singletonObject;
	}
```

这里有几个Map非常重要，他们就是存储我们bean的地方。



3. AbstractAutowireCapableBeanFactory#doCreateBean

   我们真正创建Bean实在这里创建的

   ```
   protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
   			throws BeanCreationException {
   
   		// Instantiate the bean.
   		BeanWrapper instanceWrapper = null;
   		if (mbd.isSingleton()) {
   			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
   		}
   		if (instanceWrapper == null) {
   			instanceWrapper = createBeanInstance(beanName, mbd, args);
   		}
   		Object bean = instanceWrapper.getWrappedInstance();
   		Class<?> beanType = instanceWrapper.getWrappedClass();
   		if (beanType != NullBean.class) {
   			mbd.resolvedTargetType = beanType;
   		}
   
   		// Allow post-processors to modify the merged bean definition.
   		synchronized (mbd.postProcessingLock) {
   			if (!mbd.postProcessed) {
   				try {
   					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
   				}
   				catch (Throwable ex) {
   					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
   							"Post-processing of merged bean definition failed", ex);
   				}
   				mbd.postProcessed = true;
   			}
   		}
   
   		// Eagerly cache singletons to be able to resolve circular references
   		// even when triggered by lifecycle interfaces like BeanFactoryAware.
   		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
   				isSingletonCurrentlyInCreation(beanName));
   		if (earlySingletonExposure) {
   			if (logger.isTraceEnabled()) {
   				logger.trace("Eagerly caching bean '" + beanName +
   						"' to allow for resolving potential circular references");
   			}
   			//3.1
   			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
   		}
   
   		// Initialize the bean instance.
   		Object exposedObject = bean;
   		try {
   		//3.2 初始化Bean
   			populateBean(beanName, mbd, instanceWrapper);
   			exposedObject = initializeBean(beanName, exposedObject, mbd);
   		}
   		catch (Throwable ex) {
   			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
   				throw (BeanCreationException) ex;
   			}
   			else {
   				throw new BeanCreationException(
   						mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
   			}
   		}
   
   		if (earlySingletonExposure) {
   			Object earlySingletonReference = getSingleton(beanName, false);
   			if (earlySingletonReference != null) {
   				if (exposedObject == bean) {
   					exposedObject = earlySingletonReference;
   				}
   				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
   					String[] dependentBeans = getDependentBeans(beanName);
   					Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
   					for (String dependentBean : dependentBeans) {
   						if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
   							actualDependentBeans.add(dependentBean);
   						}
   					}
   					if (!actualDependentBeans.isEmpty()) {
   						throw new BeanCurrentlyInCreationException(beanName,
   								"Bean with name '" + beanName + "' has been injected into other beans [" +
   								StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
   								"] in its raw version as part of a circular reference, but has eventually been " +
   								"wrapped. This means that said other beans do not use the final version of the " +
   								"bean. This is often the result of over-eager type matching - consider using " +
   								"'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.");
   					}
   				}
   			}
   		}
   
   		// Register bean as disposable.
   		try {
   			registerDisposableBeanIfNecessary(beanName, bean, mbd);
   		}
   		catch (BeanDefinitionValidationException ex) {
   			throw new BeanCreationException(
   					mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
   		}
   
   		return exposedObject;
   	}
   ```

   