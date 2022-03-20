我们从@EnableAspectJAutoProxy 入手

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {
   boolean proxyTargetClass() default false;

   boolean exposeProxy() default false;

}
```

AspectJAutoProxyRegistrar

```
public void registerBeanDefinitions(
      AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

   AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

   AnnotationAttributes enableAspectJAutoProxy =
         AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
   if (enableAspectJAutoProxy != null) {
      if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
         AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
      }
      if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
         AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
      }
   }
}
```

AopConfigUtils#registerAspectJAnnotationAutoProxyCreatorIfNecessary()
```
@Nullable
public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(
      BeanDefinitionRegistry registry, @Nullable Object source) {

   return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source);
}
```

它为我们导入了AnnotationAwareAspectJAutoProxyCreator这样一个组件，

我们看一下这个类的继承结构

![image-20220320154212858](https://gitee.com/linmsen/picture/raw/master//img/202203201542931.png)

由继承树可以看出

该类继承了BeanPostProcessor

我们来看在初始化之前做了什么AbstractAutoProxyCreator#postProcessBeforeInstantiation



AnnotationAwareAspectJAutoProxyCreator#findCandidateAdvisors

```
protected List<Advisor> findCandidateAdvisors() {
		// Add all the Spring advisors found according to superclass rules.
		List<Advisor> advisors = super.findCandidateAdvisors();
		// Build Advisors for all AspectJ aspects in the bean factory.
		if (this.aspectJAdvisorsBuilder != null) {
			advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
		}
		return advisors;
}
```



我们来看是如何找到增强器



