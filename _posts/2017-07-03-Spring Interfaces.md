### InstantiationStrategy 

负责创建RootBeanDefinition相应实例的接口

实现类SimpleInstantiationStrategy

实现了下列3个方法,factory是独立实现的

如果beanDefination有MethodOverride的话需要有cglib子类初始化

<pre class="java">public interface InstantiationStrategy {
//无参数的创建bean
    Object instantiate(RootBeanDefinition beanDefinition, String beanName, BeanFactory owner)
      throws BeansException;
      //有参数的创建bean
      Object instantiate(RootBeanDefinition beanDefinition, String beanName, BeanFactory owner,
      Constructor<?> ctor, Object[] args) throws BeansException;
      //factoryBean的方式创建bean
      Object instantiate(RootBeanDefinition beanDefinition, String beanName, BeanFactory owner,
      Object factoryBean, Method factoryMethod, Object[] args) throws BeansException;
}</pre>

### BeanMetadataElement

搭载元数据配置来源的信息

<pre class="java">public interface BeanMetadataElement {

   Object getSource();

}</pre>

### ParameterNameDiscoverer 

获取方法的参数的名字

实现类

<span style="color: rgb(84, 141, 212);">**PrioritizedParameterNameDiscoverer**</span>:里面放一组parameterNameDiscoverers,取第一个

<span style="color: rgb(84, 141, 212);">**DefaultParameterNameDiscoverer**</span>:判断是不是JDK1.8,JDK1.8有个refect.Parameter 

JDK1.8使用StandardReflectionParameterNameDiscoverer,使用Parameter极为简单

然而并不是,使用LocalVariableTableParameterNameDiscoverer需要读取字节码

<pre>public interface ParameterNameDiscoverer {

   /**
    * Return parameter names for this method
    */
   String[] getParameterNames(Method method);

   /**
    * Return parameter names for this constructor,
    */
   String[] getParameterNames(Constructor<?> ctor);

}</pre>

### AutowireCandidateResolver 

判断一个指定的beanDefination能无能成为一个指定bean的注入候选

<span style="color: rgb(84, 141, 212);">**SimpleAutowireCandidateResolver**</span>:isAutowireCandidate直接返回beanDefination的属性,其他返回null

<pre>public interface AutowireCandidateResolver {

    //这个bean能不能注入的这个描述符的field里面
   boolean isAutowireCandidate(BeanDefinitionHolder bdHolder, DependencyDescriptor descriptor);

   Object getSuggestedValue(DependencyDescriptor descriptor);

   Object getLazyResolutionProxyIfNecessary(DependencyDescriptor descriptor, String beanName);

}</pre>

### BeanNameGenerator 

生成bean的name

<pre>public interface BeanNameGenerator {
    String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry);
}</pre>

### BeanDefinitionReader 

读取BeanDefinition

设计一个读取bean信息的东西需要什么

读取完bean之后需要注册到beanDefination的map里面

需要能加载资源的东西

需要加载资源

<span style="color: rgb(84, 141, 212);">**AbstractBeanDefinitionReader**</span>:实现了基本的接口除了loadBeanDefinitions(resource),基本都指向了这个函数

**<span style="color: rgb(84, 141, 212);">XmlBeanDefinitionReader:</span>**处理xml校验解析相关内容

<pre>public interface BeanDefinitionReader {

   BeanDefinitionRegistry getRegistry();

   ResourceLoader getResourceLoader();

   ClassLoader getBeanClassLoader();
   //名字生成器
   BeanNameGenerator getBeanNameGenerator();
    //加载资源,返回数量
   int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException;

   int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException;

   int loadBeanDefinitions(String location) throws BeanDefinitionStoreException;

   int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException;

}</pre>

### ResourceLoader 

<pre>public interface ResourceLoader {

   //classpath:
   String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;

    /**
    * 根据位置加载资源
    * <li>Must support fully qualified URLs, e.g. "file:C:/test.dat".
    * <li>Must support classpath pseudo-URLs, e.g. "classpath:test.dat".
    * <li>Should support relative file paths, e.g. "WEB-INF/test.dat".
    */
   Resource getResource(String location);

   ClassLoader getClassLoader();

}</pre>

### BeanDefinitionRegistry 

基本就是个map<String beanName,BeanDefination bd>

<span style="color: rgb(84, 141, 212);">**DefaultListableBeanFactory**</span>

<pre>public interface BeanDefinitionRegistry extends AliasRegistry {

   void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
         throws BeanDefinitionStoreException;

   void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

   BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

   boolean containsBeanDefinition(String beanName);

   String[] getBeanDefinitionNames();

   int getBeanDefinitionCount();

   boolean isBeanNameInUse(String beanName);

}</pre>

### PropertyResolver 

解决各种source属性的接口

<pre>public interface PropertyResolver {

   boolean containsProperty(String key);

   String getProperty(String key);

   String getProperty(String key, String defaultValue);

   <T> T getProperty(String key, Class<T> targetType);

   <T> T getProperty(String key, Class<T> targetType, T defaultValue);

   <T> Class<T> getPropertyAsClass(String key, Class<T> targetType);

   String getRequiredProperty(String key) throws IllegalStateException;

   <T> T getRequiredProperty(String key, Class<T> targetType) throws IllegalStateException;

   String resolvePlaceholders(String text);

   String resolveRequiredPlaceholders(String text) throws IllegalArgumentException;

}</pre>

Environment 

表示当前应用程序正在运行的环境的接口

配置文件和属性

无论是在XML中还是通过注释定义的配置文件

可能源自各种来源：属性文件，JVM系统属性，系统,环境变量，JNDI，servlet上下文参数，ad-hoc属性对象，地图等等。

环境对象与属性关系的作用是为用户提供方便的服务接口，用于配置资源并从中解析属性。

<pre>public interface Environment extends PropertyResolver {

   String[] getActiveProfiles();

   String[] getDefaultProfiles();

   boolean acceptsProfiles(String... profiles);

}</pre>

### BeanDefinitionDocumentReader 

注册bean

<span style="color: rgb(84, 141, 212);">**DefaultBeanDefinitionDocumentReade**</span><span style="color: rgb(84, 141, 212);">**r**</span>:整个类都在处理这个接口registerBeanDefinitions

<pre>public interface BeanDefinitionDocumentReader {

   void setEnvironment(Environment environment);

   void registerBeanDefinitions(Document doc, XmlReaderContext readerContext)
         throws BeanDefinitionStoreException;

}</pre>