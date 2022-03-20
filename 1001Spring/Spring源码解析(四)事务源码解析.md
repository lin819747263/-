我们从@EnableTransactionManagement作为入口

```
protected String[] selectImports(AdviceMode adviceMode) {
   switch (adviceMode) {
      case PROXY:
         return new String[] {AutoProxyRegistrar.class.getName(),
               ProxyTransactionManagementConfiguration.class.getName()};
      case ASPECTJ:
         return new String[] {determineTransactionAspectClass()};
      default:
         return null;
   }
}
```

从这个方法看这个注解为我们导入两个组件AutoProxyRegistrar和ProxyTransactionManagementConfiguration

AutoProxyRegistrar 为我们导入了InfrastructureAdvisorAutoProxyCreator的组件，我们从继承图可以看出，它也是一个BeanPostProcessor

![image-20220320195457903](https://gitee.com/linmsen/picture/raw/master//img/202203201954966.png)