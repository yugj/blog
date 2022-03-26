# Feign httpclient文件上传问题记录

## 问题说明

原先项目http请求通过feign + ribbon + urlconnection 完成，考虑urlconnection频繁连接释放带来网络及cpu开销问题采用http client作为连接池，升级完后发现发现出现调用下游服务乱码、传文件 文件内容被篡改问题



## 处理

### 乱码问题

原先系统feign-core、feign-httpclient使用9.5.0 ，按[GitHub issue](https://github.com/OpenFeign/feign/issues/984)看，这个版本并不是使用utf-8作为编码，导致中文乱码情况，鉴于项目用的版本相对较老，这边只升级到9.6.0解决了乱码问题，之前也有通过feign拦截器处理，但是这样需要判断特定的content-type进行处理，避免麻烦，open-feign强调他们的编码是utf-8，部分版本不是，可以定义为bug，既然是bug，能通过升级解决问题就直接升级了

###  传文件内容被篡改

有问题找GitHub，还是由于版本问题，[老版本的一个bug](https://github.com/OpenFeign/feign-form/issues/18)，通过httpclient导致文件上传内容篡改，3.0.3版本修复了，早期项目使用的版本是2.0.1，有点年头，参考了下官方建议要求

## Requirements

The `feign-form` extension depend on `OpenFeign` and its *concrete* versions:

- all `feign-form` releases before **3.5.0** works with `OpenFeign` **9.\*** versions;
- starting from `feign-form`'s version **3.5.0**, the module works with `OpenFeign` **10.1.0** versions and greater.

> **IMPORTANT:** there is no backward compatibility and no any gurantee that the `feign-form`'s versions after **3.5.0**work with `OpenFeign` before **10.\***. `OpenFeign` was refactored in 10th release, so the best approach - use the freshest `OpenFeign` and `feign-form` versions.

Notes:

- [spring-cloud-openfeign](https://github.com/spring-cloud/spring-cloud-openfeign) uses `OpenFeign` **9.\*** till **v2.0.3.RELEASE** and uses **10.\*** after. Anyway, the dependency already has suitable `feign-form` version, see [dependency pom](https://github.com/spring-cloud/spring-cloud-openfeign/blob/master/spring-cloud-openfeign-dependencies/pom.xml#L19), so you don't need to specify it separately;
- `spring-cloud-starter-feign` is a **deprecated** dependency and it always uses the `OpenFeign`'s **9.\*** versions.



使用了如下版本pom，接口httpclient下文件上传内容被篡改问题

```xml
feign-version = 9.6.0
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form-spring</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
			<groupId>io.github.openfeign</groupId>
			<artifactId>feign-httpclient</artifactId>
			<version>${feign.version}</version>
		</dependency>
		<dependency>
			<groupId>io.github.openfeign</groupId>
			<artifactId>feign-core</artifactId>
			<version>${feign.version}</version>
		</dependency>
		<dependency>
			<groupId>io.github.openfeign</groupId>
			<artifactId>feign-hystrix</artifactId>
			<version>${feign.version}</version>
		</dependency>
		<dependency>
			<groupId>io.github.openfeign</groupId>
			<artifactId>feign-slf4j</artifactId>
			<version>${feign.version}</version>
		</dependency>
```

### 多附件文件上传问题

默认SpringFormEncoder只处理单文件，一个强转就完事了

```java
@Override
public void encode (Object object, Type bodyType, RequestTemplate template) throws EncodeException {
  if (!bodyType.equals(MultipartFile.class)) {
    super.encode(object, bodyType, template);
    return;
  }

  val file = (MultipartFile) object;
  val data = singletonMap(file.getName(), object);
  super.encode(data, MAP_STRING_WILDCARD, template);
}


```

我们系统文件上传支持多附件，这个需要重写下

```java
/**
 * @author yugj
 * @date 2019/8/28 下午4:27.
 */
public class FeignSpringFormEncoder extends FormEncoder {


    /**
     * Constructor with the default Feign's encoder as a delegate.
     */
    public FeignSpringFormEncoder() {
        this(new Default());
    }


    /**
     * Constructor with specified delegate encoder.
     *
     * @param delegate delegate encoder, if this encoder couldn't encode object.
     */
    public FeignSpringFormEncoder(Encoder delegate) {
        super(delegate);

        MultipartFormContentProcessor processor = (MultipartFormContentProcessor) getContentProcessor(ContentType.MULTIPART);
        processor.addWriter(new SpringSingleMultipartFileWriter());
        processor.addWriter(new SpringManyMultipartFilesWriter());
    }


    @Override
    public void encode(Object object, Type bodyType, RequestTemplate template) throws EncodeException {
        if (bodyType.equals(MultipartFile.class)) {
            MultipartFile file = (MultipartFile) object;
            Map data = Collections.singletonMap(file.getName(), object);
            super.encode(data, MAP_STRING_WILDCARD, template);
            return;
        } else if (bodyType.equals(MultipartFile[].class)) {
            MultipartFile[] file = (MultipartFile[]) object;
            if(file != null) {
                Map data = Collections.singletonMap(file.length == 0 ? "" : "file", object);
                super.encode(data, MAP_STRING_WILDCARD, template);
                return;
            }
        }
        super.encode(object, bodyType, template);
    }
}
```

对应feignclient使用这个配置即可

```java
@Service
@FeignClient(name = "xx-file-sv",
        fallbackFactory = FileGeneralHystrixFallbackAdapter.class,
        configuration = FeignSpringFormEncoder.class)
public interface IFileClient extends IFile {

}
```