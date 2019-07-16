---
title: SpringBoot实现RESTAPI
date: 2017-10-25 17:28:09
tags: 
	- Java	
comments: true
---
>如题，本文讲述如何使用Springboot实现restapi,这里感谢开源社区的作者@简单的土豆，和他的源码[https://github.com/j4fan/spring-boot-api-project-seed.git](https://github.com/j4fan/spring-boot-api-project-seed.git)

## 1基本配置
![项目结构.png](http://upload-images.jianshu.io/upload_images/5834071-d8ba22876750345e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
基本配置包括分环境的profile配置， log4j2配置，mybatis的配置，maven依赖的添加，统一结果封装，结果生成器等等。
这里作者使用了mybatis通用mapper
```
public interface Mapper<T>
        extends
        BaseMapper<T>,
        ConditionMapper<T>,
        IdsMapper<T>,
        InsertListMapper<T> {
}
```
通用service接口
```
public interface Service<T> {
    void save(T model);//持久化
    void save(List<T> models);//批量持久化
    void deleteById(Long id);//通过主鍵刪除
    void deleteByIds(String ids);//批量刪除 eg：ids -> “1,2,3,4”
    void update(T model);//更新
    T findById(Long id);//通过ID查找
    T findBy(String fieldName, Object value) throws TooManyResultsException; //通过Model中某个成员变量名称（非数据表中column的名称）查找,value需符合unique约束
    List<T> findByIds(String ids);//通过多个ID查找//eg：ids -> “1,2,3,4”
    List<T> findByCondition(Condition condition);//根据条件查找
    List<T> findAll();//获取所有
}
```
然后实现abstractService，用通用mapper实现了service,后面的service，只需继承abstractService即可
```
public abstract class AbstractService<T> implements Service<T> {

    @Autowired
    protected Mapper<T> mapper;

    private Class<T> modelClass;    // 当前泛型真实类型的Class

    public AbstractService() {
        ParameterizedType pt = (ParameterizedType) this.getClass().getGenericSuperclass();
        modelClass = (Class<T>) pt.getActualTypeArguments()[0];
    }

    public void save(T model) {
        mapper.insertSelective(model);
    }

    public void save(List<T> models) {
        mapper.insertList(models);
    }

    public void deleteById(Long id) {
        mapper.deleteByPrimaryKey(id);
    }

    public void deleteByIds(String ids) {
        mapper.deleteByIds(ids);
    }

    public void update(T model) {
        mapper.updateByPrimaryKeySelective(model);
    }

    public T findById(Long id) {
        return mapper.selectByPrimaryKey(id);
    }

    @Override
    public T findBy(String fieldName, Object value) throws TooManyResultsException {
        try {
            T model = modelClass.newInstance();
            Field field = modelClass.getDeclaredField(fieldName);
            field.setAccessible(true);
            field.set(model, value);
            return mapper.selectOne(model);
        } catch (ReflectiveOperationException e) {
            throw new ServiceException(e.getMessage(), e);
        }
    }

    public List<T> findByIds(String ids) {
        return mapper.selectByIds(ids);
    }

    public List<T> findByCondition(Condition condition) {
        return mapper.selectByCondition(condition);
    }

    public List<T> findAll() {
        return mapper.selectAll();
    }
}

```
## 2.下面看看Controller
先上代码
```
@RestController
public class FetchConfigController {

    @Autowired
    UserInfoService userInfoService;

    @PostMapping(value = "/add", consumes = "application/json", produces = "application/json")
    public Result add(@RequestBody UserInfo userInfo) {
        userInfoService.save(userInfo);
        return ResultCodeGenerator.genSuccessResult();
    }

    @PostMapping("/delete")
    public Result delete(@RequestParam Long id) {
        userInfoService.deleteById(id);
        return ResultCodeGenerator.genSuccessResult();
    }

    @PostMapping("/update")
    public Result update(@RequestBody UserInfo userInfo) {
        userInfoService.update(userInfo);
        return ResultCodeGenerator.genSuccessResult();
    }

    @PostMapping("/detail")
    public Result detail(@RequestParam Long id) {
        UserInfo userInfo = userInfoService.findById(id);
        return ResultCodeGenerator.genSuccessResult(userInfo);
    }

    @PostMapping("/list")
    public Result list(@RequestParam(defaultValue = "0") Integer page, @RequestParam(defaultValue = "0") Integer size) {
        PageHelper.startPage(page, size);
        List<UserInfo> list = userInfoService.findAll();
        PageInfo pageInfo = new PageInfo(list);
        return ResultCodeGenerator.genSuccessResult(pageInfo);
    }

}
```
1@RestController，可以看到它是@Controller/@ResponseBody的结合体，@ResponseBody这个注解在RESTAPI 中很有意义，它不是将返回资源定位到resouces下面的html/css/js生成视图，而是直接以写入输出流返回给客户端。
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {
    String value() default "";
}
```
这里不得不提到HttpMessageConverter，这个转换器顾名思义，是将httprequest输入流读成对象，或者字符串，再返回的时候将对象转换成HttpResponse输出流
```
public interface HttpMessageConverter<T> {
    boolean canRead(Class<?> var1, MediaType var2);

    boolean canWrite(Class<?> var1, MediaType var2);

    List<MediaType> getSupportedMediaTypes();

    T read(Class<? extends T> var1, HttpInputMessage var2) throws IOException, HttpMessageNotReadableException;

    void write(T var1, MediaType var2, HttpOutputMessage var3) throws IOException, HttpMessageNotWritableException;
}
```
可以看到它是个泛型的接口，主要的两个方法是read/write canread/write,Spring已经实现了abstractHttpMessageConverter，自己可以继承这个abstract的class，拿来使用。converter用canread/write检查http流的类型，然后返回是true,就用read/write方法去读写。一般都是用别人写好的converter,例如阿里的fastjson的converter，可以用于处理application/json,例如MappingJackson2XmlHttpMessageConverter用于处理application/xml。这些内容可以配在configure文件中
```
@Configuration
public class WebMvcConfigurer extends WebMvcConfigurerAdapter {

    private static Logger logger = LogManager.getLogger();

    //当前激活的配置文件
    @Value("${spring.profiles.active}")
    private String env;

    //使用阿里 FastJson 作为JSON MessageConverter
    //使用MappingJackson2XmlHttpMessageConverter进行xml的转换
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        FastJsonHttpMessageConverter4 converter = new FastJsonHttpMessageConverter4();
        FastJsonConfig config = new FastJsonConfig();
        config.setSerializerFeatures(SerializerFeature.WriteMapNullValue,//保留空的字段
                SerializerFeature.WriteNullStringAsEmpty,//String null -> ""
                SerializerFeature.WriteNullNumberAsZero);//Number null -> 0
        converter.setFastJsonConfig(config);
        converter.setDefaultCharset(Charset.forName("UTF-8"));
        MappingJackson2XmlHttpMessageConverter converter1 = new MappingJackson2XmlHttpMessageConverter();
        converters.add(converter1);
        converters.add(converter);
    }
```

我这里添加了两个converter,分别用来处理json和xml.
2.@PostMapping其实是@RequestMapping(method = {RequestMethod.POST})的缩写
3.@RequestParam其实还可以加入default和require的设置
4.@RequestBody直接把inputstream中的json通过converter读成对象进行处理，和@responsebody对应将对象写成json放入outputstream

## 3.研究下Spring自带的RestTemplate
通常我们进行http请求是用apache的httpclient，而且功能比较强大，这里spring也集成了自己的rest请求方法
```
private static ClientHttpRequestFactory getSimpleClientHttpRequestFactory() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setReadTimeout(60000);//ms
        factory.setConnectTimeout(30000);//ms
        return factory;
 }
```
RestTemplate直接new就可以使用了，构造方法里可以注入一个factory用于统一设置一些参数
```
private static RestTemplate getRestTemplate(ClientHttpRequestFactory factory) {
        RestTemplate restTemplate = new RestTemplate(factory);
        List<HttpMessageConverter<?>> messageConverters = new ArrayList<>();
        StringHttpMessageConverter stringHttpMessageConverter
                = new StringHttpMessageConverter();
        stringHttpMessageConverter.setWriteAcceptCharset(false);
        stringHttpMessageConverter.setDefaultCharset(Charset.forName("UTF-8"));
        messageConverters.add(stringHttpMessageConverter);
        messageConverters.add(new FormHttpMessageConverter());
        messageConverters.add(new FastJsonHttpMessageConverter4());
        restTemplate.setMessageConverters(messageConverters);
        return restTemplate;
    }
```
template里面统一可以注入converter,这里我加入了3个converter，这里注意StringmessageConverter里面默认的字符是ISO-8859-1，所以new一个新的converter并且设置编码防止出错，剩下的就是简单粗暴的直接使用咯
```
        RestTemplate restTemplate = getRestTemplate(getSimpleClientHttpRequestFactory());
        String url = "http://localhost:7777/dcs/list";
//        HttpHeaders headers = new HttpHeaders();
//        headers.setContentType(MediaType.ALL.APPLICATION_JSON);
//        UserInfo userInfo = new UserInfo().setUserId((long) 1231).
//                setUserName("test").setAge(13).setEmail("fafa@jancy.com");
//        HttpEntity request = new HttpEntity(userInfo, headers);
//        Result result = restTemplate.postForObject(url, request, Result.class);
//        System.out.println(JSON.toJSONString(result));
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        MultiValueMap<String, Integer> map = new LinkedMultiValueMap<>();
        map.add("page", 1);
        map.add("size", 5);
        HttpEntity<MultiValueMap<String, Integer>> request = new HttpEntity<>(map, headers);
        Result result = restTemplate.postForObject(url, request, Result.class);
        System.out.println(JSON.toJSONString(result));
```
根据api的不同，既可以直接传入只要把HttpEntity包装好即可。
除了写在main函数中，也可以用Spring的注解加载factory/restemplate
```
@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate(ClientHttpRequestFactory factory){
        RestTemplate restTemplate = new RestTemplate(factory);
        List<HttpMessageConverter<?>> messageConverters = new ArrayList<>();
        StringHttpMessageConverter stringHttpMessageConverter
            = new StringHttpMessageConverter();
        stringHttpMessageConverter.setWriteAcceptCharset(false);
        stringHttpMessageConverter.setDefaultCharset(Charsets.UTF_8);
        messageConverters.add(stringHttpMessageConverter);
        messageConverters.add(new FormHttpMessageConverter());
        messageConverters.add(new FastJsonHttpMessageConverter());
        restTemplate.setMessageConverters(messageConverters);
        return restTemplate;
    }

    @Bean
    public ClientHttpRequestFactory simpleClientHttpRequestFactory(){
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setReadTimeout(60000);//ms
        factory.setConnectTimeout(30000);//ms
        return factory;
    }
}
```
在其他代码里只用@autowired即可使用

## 4.总结
以上就是如何用Springboot构建restApi，其实还可以和数据库做很多交互，还有签名认证，session管理等东西，都到后面完善吧。源码地址在最开始就贴了，是个开源的项目，HttpMessageConverter，如何得到json、xml的返回，都需要操作操作。继续努力吧
[个人博客](https://j4fan.github.io/) 欢迎访问~
