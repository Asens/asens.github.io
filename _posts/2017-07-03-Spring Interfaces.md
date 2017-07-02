### InstantiationStrategy?

负责创建RootBeanDefinition相应实例的接口

实现类SimpleInstantiationStrategy

实现了下列3个方法,factory是独立实现的

如果beanDefination有MethodOverride的话需要有cglib子类初始化

<pre class="java hljs"><span class="hljs-keyword">public</span>?<span class="hljs-class"><span class="hljs-keyword">interface</span>?<span class="hljs-title">InstantiationStrategy</span>?</span>{
<span class="hljs-comment">//无参数的创建bean</span>
????<span class="hljs-function">Object?<span class="hljs-title">instantiate</span><span class="hljs-params">(RootBeanDefinition?beanDefinition,?String?beanName,?BeanFactory?owner)</span>
??????<span class="hljs-keyword">throws</span>?BeansException</span>;
??????<span class="hljs-comment">//有参数的创建bean</span>
??????<span class="hljs-function">Object?<span class="hljs-title">instantiate</span><span class="hljs-params">(RootBeanDefinition?beanDefinition,?String?beanName,?BeanFactory?owner,
??????Constructor<?>?ctor,?Object[]?args)</span>?<span class="hljs-keyword">throws</span>?BeansException</span>;
??????<span class="hljs-comment">//factoryBean的方式创建bean</span>
??????<span class="hljs-function">Object?<span class="hljs-title">instantiate</span><span class="hljs-params">(RootBeanDefinition?beanDefinition,?String?beanName,?BeanFactory?owner,
??????Object?factoryBean,?Method?factoryMethod,?Object[]?args)</span>?<span class="hljs-keyword">throws</span>?BeansException</span>;
}</pre>

### BeanMetadataElement

搭载元数据配置来源的信息

<pre class="java hljs"><span class="hljs-keyword">public</span>?<span class="hljs-class"><span class="hljs-keyword">interface</span>?<span class="hljs-title">BeanMetadataElement</span>?</span>{

???<span class="hljs-function">Object?<span class="hljs-title">getSource</span><span class="hljs-params">()</span></span>;

}</pre>

### ParameterNameDiscoverer?

获取方法的参数的名字

实现类

<span style="color: rgb(84, 141, 212);">**PrioritizedParameterNameDiscoverer**</span>:里面放一组parameterNameDiscoverers,取第一个

<span style="color: rgb(84, 141, 212);">**DefaultParameterNameDiscoverer**</span>:判断是不是JDK1.8,JDK1.8有个refect.Parameter?

JDK1.8使用StandardReflectionParameterNameDiscoverer,使用Parameter极为简单

然而并不是,使用LocalVariableTableParameterNameDiscoverer需要读取字节码

<pre class="hljs cs"><span class="hljs-keyword">public</span>?<span class="hljs-keyword">interface</span>?<span class="hljs-title">ParameterNameDiscoverer</span>?{

???<span class="hljs-comment">/**
????*?Return?parameter?names?for?this?method
????*/</span>
???<span class="hljs-function">String[]?<span class="hljs-title">getParameterNames</span>(<span class="hljs-params">Method?method</span>)</span>;

???<span class="hljs-comment">/**
????*?Return?parameter?names?for?this?constructor,
????*/</span>
???<span class="hljs-function">String[]?<span class="hljs-title">getParameterNames</span>(<span class="hljs-params">Constructor<?>?ctor</span>)</span>;

}</pre>

### AutowireCandidateResolver?

判断一个指定的beanDefination能无能成为一个指定bean的注入候选

<span style="color: rgb(84, 141, 212);">**SimpleAutowireCandidateResolver**</span>:isAutowireCandidate直接返回beanDefination的属性,其他返回null

<pre class="hljs java"><span class="hljs-keyword">public</span>?<span class="hljs-class"><span class="hljs-keyword">interface</span>?<span class="hljs-title">AutowireCandidateResolver</span>?</span>{

????<span class="hljs-comment">//这个bean能不能注入的这个描述符的field里面</span>
???<span class="hljs-function"><span class="hljs-keyword">boolean</span>?<span class="hljs-title">isAutowireCandidate</span><span class="hljs-params">(BeanDefinitionHolder?bdHolder,?DependencyDescriptor?descriptor)</span></span>;

???<span class="hljs-function">Object?<span class="hljs-title">getSuggestedValue</span><span class="hljs-params">(DependencyDescriptor?descriptor)</span></span>;

???<span class="hljs-function">Object?<span class="hljs-title">getLazyResolutionProxyIfNecessary</span><span class="hljs-params">(DependencyDescriptor?descriptor,?String?beanName)</span></span>;

}</pre>

### BeanNameGenerator?

生成bean的name

<pre class="hljs cs"><span class="hljs-keyword">public</span>?<span class="hljs-keyword">interface</span>?<span class="hljs-title">BeanNameGenerator</span>?{
????<span class="hljs-function">String?<span class="hljs-title">generateBeanName</span>(<span class="hljs-params">BeanDefinition?definition,?BeanDefinitionRegistry?registry</span>)</span>;
}</pre>

### BeanDefinitionReader?

读取BeanDefinition

设计一个读取bean信息的东西需要什么

读取完bean之后需要注册到beanDefination的map里面

需要能加载资源的东西

需要加载资源

<span style="color: rgb(84, 141, 212);">**AbstractBeanDefinitionReader**</span>:实现了基本的接口除了loadBeanDefinitions(resource),基本都指向了这个函数

**<span style="color: rgb(84, 141, 212);">XmlBeanDefinitionReader:</span>**处理xml校验解析相关内容

<pre class="hljs java"><span class="hljs-keyword">public</span>?<span class="hljs-class"><span class="hljs-keyword">interface</span>?<span class="hljs-title">BeanDefinitionReader</span>?</span>{

???<span class="hljs-function">BeanDefinitionRegistry?<span class="hljs-title">getRegistry</span><span class="hljs-params">()</span></span>;

???<span class="hljs-function">ResourceLoader?<span class="hljs-title">getResourceLoader</span><span class="hljs-params">()</span></span>;

???<span class="hljs-function">ClassLoader?<span class="hljs-title">getBeanClassLoader</span><span class="hljs-params">()</span></span>;
???<span class="hljs-comment">//名字生成器</span>
???<span class="hljs-function">BeanNameGenerator?<span class="hljs-title">getBeanNameGenerator</span><span class="hljs-params">()</span></span>;
????<span class="hljs-comment">//加载资源,返回数量</span>
???<span class="hljs-function"><span class="hljs-keyword">int</span>?<span class="hljs-title">loadBeanDefinitions</span><span class="hljs-params">(Resource?resource)</span>?<span class="hljs-keyword">throws</span>?BeanDefinitionStoreException</span>;

???<span class="hljs-function"><span class="hljs-keyword">int</span>?<span class="hljs-title">loadBeanDefinitions</span><span class="hljs-params">(Resource...?resources)</span>?<span class="hljs-keyword">throws</span>?BeanDefinitionStoreException</span>;

???<span class="hljs-function"><span class="hljs-keyword">int</span>?<span class="hljs-title">loadBeanDefinitions</span><span class="hljs-params">(String?location)</span>?<span class="hljs-keyword">throws</span>?BeanDefinitionStoreException</span>;

???<span class="hljs-function"><span class="hljs-keyword">int</span>?<span class="hljs-title">loadBeanDefinitions</span><span class="hljs-params">(String...?locations)</span>?<span class="hljs-keyword">throws</span>?BeanDefinitionStoreException</span>;

}</pre>

### ResourceLoader?

<pre class="hljs cs"><span class="hljs-keyword">public</span>?<span class="hljs-keyword">interface</span>?<span class="hljs-title">ResourceLoader</span>?{

???<span class="hljs-comment">//classpath:</span>
???String?CLASSPATH_URL_PREFIX?=?ResourceUtils.CLASSPATH_URL_PREFIX;

????<span class="hljs-comment">/**
????*?根据位置加载资源
????*?<li>Must?support?fully?qualified?URLs,?e.g.?"file:C:/test.dat".
????*?<li>Must?support?classpath?pseudo-URLs,?e.g.?"classpath:test.dat".
????*?<li>Should?support?relative?file?paths,?e.g.?"WEB-INF/test.dat".
????*/</span>
???<span class="hljs-function">Resource?<span class="hljs-title">getResource</span>(<span class="hljs-params">String?location</span>)</span>;

???<span class="hljs-function">ClassLoader?<span class="hljs-title">getClassLoader</span>(<span class="hljs-params"></span>)</span>;

}</pre>

### BeanDefinitionRegistry?

基本就是个map<String beanName,BeanDefination bd>

<span style="color: rgb(84, 141, 212);">**DefaultListableBeanFactory**</span>

<pre class="hljs java"><span class="hljs-keyword">public</span>?<span class="hljs-class"><span class="hljs-keyword">interface</span>?<span class="hljs-title">BeanDefinitionRegistry</span>?<span class="hljs-keyword">extends</span>?<span class="hljs-title">AliasRegistry</span>?</span>{

???<span class="hljs-function"><span class="hljs-keyword">void</span>?<span class="hljs-title">registerBeanDefinition</span><span class="hljs-params">(String?beanName,?BeanDefinition?beanDefinition)</span>
?????????<span class="hljs-keyword">throws</span>?BeanDefinitionStoreException</span>;

???<span class="hljs-function"><span class="hljs-keyword">void</span>?<span class="hljs-title">removeBeanDefinition</span><span class="hljs-params">(String?beanName)</span>?<span class="hljs-keyword">throws</span>?NoSuchBeanDefinitionException</span>;

???<span class="hljs-function">BeanDefinition?<span class="hljs-title">getBeanDefinition</span><span class="hljs-params">(String?beanName)</span>?<span class="hljs-keyword">throws</span>?NoSuchBeanDefinitionException</span>;

???<span class="hljs-function"><span class="hljs-keyword">boolean</span>?<span class="hljs-title">containsBeanDefinition</span><span class="hljs-params">(String?beanName)</span></span>;

???String[]?getBeanDefinitionNames();

???<span class="hljs-function"><span class="hljs-keyword">int</span>?<span class="hljs-title">getBeanDefinitionCount</span><span class="hljs-params">()</span></span>;

???<span class="hljs-function"><span class="hljs-keyword">boolean</span>?<span class="hljs-title">isBeanNameInUse</span><span class="hljs-params">(String?beanName)</span></span>;

}</pre>

### PropertyResolver?

解决各种source属性的接口

<pre class="hljs java"><span class="hljs-keyword">public</span>?<span class="hljs-class"><span class="hljs-keyword">interface</span>?<span class="hljs-title">PropertyResolver</span>?</span>{

???<span class="hljs-function"><span class="hljs-keyword">boolean</span>?<span class="hljs-title">containsProperty</span><span class="hljs-params">(String?key)</span></span>;

???<span class="hljs-function">String?<span class="hljs-title">getProperty</span><span class="hljs-params">(String?key)</span></span>;

???<span class="hljs-function">String?<span class="hljs-title">getProperty</span><span class="hljs-params">(String?key,?String?defaultValue)</span></span>;

???<T>?<span class="hljs-function">T?<span class="hljs-title">getProperty</span><span class="hljs-params">(String?key,?Class<T>?targetType)</span></span>;

???<T>?<span class="hljs-function">T?<span class="hljs-title">getProperty</span><span class="hljs-params">(String?key,?Class<T>?targetType,?T?defaultValue)</span></span>;

???<T>?<span class="hljs-function">Class<T>?<span class="hljs-title">getPropertyAsClass</span><span class="hljs-params">(String?key,?Class<T>?targetType)</span></span>;

???<span class="hljs-function">String?<span class="hljs-title">getRequiredProperty</span><span class="hljs-params">(String?key)</span>?<span class="hljs-keyword">throws</span>?IllegalStateException</span>;

???<T>?<span class="hljs-function">T?<span class="hljs-title">getRequiredProperty</span><span class="hljs-params">(String?key,?Class<T>?targetType)</span>?<span class="hljs-keyword">throws</span>?IllegalStateException</span>;

???<span class="hljs-function">String?<span class="hljs-title">resolvePlaceholders</span><span class="hljs-params">(String?text)</span></span>;

???<span class="hljs-function">String?<span class="hljs-title">resolveRequiredPlaceholders</span><span class="hljs-params">(String?text)</span>?<span class="hljs-keyword">throws</span>?IllegalArgumentException</span>;

}</pre>

Environment?

表示当前应用程序正在运行的环境的接口

配置文件和属性

无论是在XML中还是通过注释定义的配置文件

可能源自各种来源：属性文件，JVM系统属性，系统,环境变量，JNDI，servlet上下文参数，ad-hoc属性对象，地图等等。

环境对象与属性关系的作用是为用户提供方便的服务接口，用于配置资源并从中解析属性。

<pre class="hljs java"><span class="hljs-keyword">public</span>?<span class="hljs-class"><span class="hljs-keyword">interface</span>?<span class="hljs-title">Environment</span>?<span class="hljs-keyword">extends</span>?<span class="hljs-title">PropertyResolver</span>?</span>{

???String[]?getActiveProfiles();

???String[]?getDefaultProfiles();

???<span class="hljs-function"><span class="hljs-keyword">boolean</span>?<span class="hljs-title">acceptsProfiles</span><span class="hljs-params">(String...?profiles)</span></span>;

}</pre>

### BeanDefinitionDocumentReader?

注册bean

<span style="color: rgb(84, 141, 212);">**DefaultBeanDefinitionDocumentReade**</span><span style="color: rgb(84, 141, 212);">**r**</span>:整个类都在处理这个接口registerBeanDefinitions

<pre class="hljs java"><span class="hljs-keyword">public</span>?<span class="hljs-class"><span class="hljs-keyword">interface</span>?<span class="hljs-title">BeanDefinitionDocumentReader</span>?</span>{

???<span class="hljs-function"><span class="hljs-keyword">void</span>?<span class="hljs-title">setEnvironment</span><span class="hljs-params">(Environment?environment)</span></span>;

???<span class="hljs-function"><span class="hljs-keyword">void</span>?<span class="hljs-title">registerBeanDefinitions</span><span class="hljs-params">(Document?doc,?XmlReaderContext?readerContext)</span>
?????????<span class="hljs-keyword">throws</span>?BeanDefinitionStoreException</span>;

}</pre>
