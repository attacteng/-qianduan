# [Springboot整合Spring Cloud Kubernetes读取ConfigMap，支持自动刷新配置](https://www.cnblogs.com/larrydpk/p/13611431.html)



# 整合Spring Cloud Kubenetes

[Spring Cloud Kubernetes](https://docs.spring.io/spring-cloud-kubernetes/docs/1.1.5.RELEASE/reference/html/)提供了`Spring Cloud`应用与`Kubernetes`服务关联，我们也可以自己写`Java`程序来获取`Kubernetes`的特性，但`Spring`又为我们做了。

## 2.1 项目代码

引入依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-kubernetes-config</artifactId>
</dependency>
```

只需要`Springboot Web`和`Spring Cloud Kubernetes Config`即可，很简单。

`Springboot`启动类：

```java
@SpringBootApplication
public class ConfigMapMain {
    public static void main(String[] args) {
        SpringApplication.run(ConfigMapMain.class, args);
    }
}
```

准备一个`EndPoint`来展示所读到的配置信息：

```java
@RestController
public class PkslowController {
    @Value("${pkslow.age:0}")
    private Integer age;

    @Value("${pkslow.email:null}")
    private String email;

    @Value("${pkslow.webSite:null}")
    private String webSite;

    @Value("${pkslow.password:null}")
    private String password;

    @GetMapping("/pkslow")
    public Map<String, String> getConfig() {
        Map<String, String> map = new HashMap<>();
        map.put("age", age.toString());
        map.put("email", email);
        map.put("webSite", webSite);
        map.put("password", password);
        return map;
    }
}
```

默认是为空的，`password`是从`Secret`读取，其它从`ConfigMap`读取。

应用的配置文件如下：

```yaml
server:
  port: 8080
spring:
  application:
    name: spring-cloud-kubernetes-configmap
  cloud:
    kubernetes:
      config:
        name: spring-cloud-kubernetes-configmap
```

这里的`spring.cloud.kubernetes.config.name`是重点，后续要通过它来找`ConfigMap`。

加密密码：

```bash
$ echo -n "pkslow-pass" | base64 
cGtzbG93LXBhc3M=
```

创建`Kubernetes Secret`：

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: spring-cloud-kubernetes-secret
  namespace: default
data:
  pkslow.password: cGtzbG93LXBhc3M=
type: Opaque
```

`ConfigMap`的内容如下：

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: spring-cloud-kubernetes-configmap
  namespace: default
  labels:
    app: scdf-server
data:
  application.yaml: |-
    pkslow:
      age: 19
      email: admin@pkslow.com
      webSite: www.pkslow.com
```

要注意的是，这里的名字与前面配置的是一致的，都是`spring-cloud-kubernetes-configmap`。

接着完成`Dockerfile`和`K8s`部署文件就可以了。注意要将`Secret`的值映射到环境变量：

```yaml
env:
	- name: PKSLOW_PASSWORD
		valueFrom:
			secretKeyRef:
				name: spring-cloud-kubernetes-secret
				key: pkslow.password
```

## 2.2 启动与测试

应用会在启动时就去`Kubernetes`找相应的`ConfigMap`和`Secret`：

```bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.5.RELEASE)

2020-08-25 00:13:17.374  INFO 7 --- [           main] b.c.PropertySourceBootstrapConfiguration : Located property source: CompositePropertySource {name='composite-configmap', propertySources=[ConfigMapPropertySource {name='configmap.spring-cloud-kubernetes-configmap.default'}]}
2020-08-25 00:13:17.376  INFO 7 --- [           main] b.c.PropertySourceBootstrapConfiguration : Located property source: CompositePropertySource {name='composite-secrets', propertySources=[]}
```

访问`spring-cloud-kubernetes-configmap.localhost/pkslow`，可以正确读取配置，`ConfigMap`和`Secret`的内容都获取到了：

![img](https://img2020.cnblogs.com/other/946674/202009/946674-20200904004551787-366847145.png)

# 自动刷新配置

## 3.1 原理介绍与代码变更

我们需要在`Web`运行过程中修改配置并使配置生效，有多种模式。修改配置文件如下：

```yaml
server:
  port: 8080
spring:
  application:
    name: spring-cloud-kubernetes-configmap
  cloud:
    kubernetes:
      config:
        name: spring-cloud-kubernetes-configmap
        namespace: default
      secrets:
        name: spring-cloud-kubernetes-secret
        namespace: default
        enabled: true
      reload:
        enabled: true
        monitoring-config-maps: true
        monitoring-secrets: true
        strategy: restart_context
        mode: event
management:
  endpoint:
    restart:
      enabled: true
  endpoints:
    web:
      exposure:
        include: restart
```

（1） `spring.cloud.kubernetes.reload.enabled=true`需要打开刷新功能；

（2） 加载策略`strategy`：

- `refresh`：只对特定的配置生效，有注解`@ConfigurationProperties` 或 `@RefreshScope`。
- `restart_context`：整个`Spring Context`会优雅重启，里面的所有配置都会重新加载。

需要打开`actuator endpoint`，所以要配置`management.endpoint`。还要增加依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-actuator</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-actuator-autoconfigure</artifactId>
</dependency>
```

- `shutdown`：重启容器。

（3）模式`mode`

- 事件`Event`：会通过`k8s API`监控`ConfigMap`的变更，读取配置并生效。
- `Polling`：定期查看是否有变化，有变化则触发，默认为15秒。

## 3.2 测试

我们修改一下`ConfigMap`的配置，并更新到`K8s`。

```bash
$ kubectl apply -f src/main/k8s/config.yaml 
configmap/spring-cloud-kubernetes-configmap configured
```

查看发现`age`和`email`都修改了：

![img](https://img2020.cnblogs.com/other/946674/202009/946674-20200904004552140-248950922.png)

我们查看一下`Pod`的日志如下：

![img](https://img2020.cnblogs.com/other/946674/202009/946674-20200904004554685-681057683.png)

`Springboot`先是检测到了`ConfigMap`有了变更，然后触发`Context`重启。

# 总结

`Spring Cloud Kubernetes`为我们提供了不少`Spring Cloud`整合`Kubernetes`的特性，可以引入使用。