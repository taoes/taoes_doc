双亲委派是 JVM 类加载机制。加载类时，子类加载器先委托父加载器；父加载器找不到，自己才加载。


- **沙箱安全**：防止核心类被篡改。比如自己写 java.lang.String，不会加载我们自定义的，优先加载 JDK 原生 String，避免破坏核心 API。
- **类复用**：同一个类只加载一次，避免重复加载，节省内存。
- **避免类重复冲突**：不同加载器统一委托父加载，保证全应用同一个类。

- 父加载器无法访问子加载器的类，SPI 场景受限；原生不支持热更新；灵活性不足，需要时要打破双亲委派。


> 哪些场景打破双亲委派？

1. JDBC SPI，使用线程上下文类加载器；
2. Tomcat 自定义类加载器（每个 webapp 独立类加载，隔离各个 web 应用）；
3. OSGi、热部署框架。

> 问：类卸载条件？

1. 类加载器实例被回收 + 该类没有任何实例对象 + 没有地方引用这个 Class 对象。满足才能被卸载。



## 演示示例

> 定义网络类加载器
```java
import java.io.ByteArrayOutputStream;  
import java.io.IOException;  
import java.io.InputStream;  
import java.net.MalformedURLException;  
import java.net.URI;  
import java.net.URL;  
  
/**  
 * 网络类加载器：遵循双亲委派机制，从网络（HTTP）加载 class 字节码。  
 *  
 * <p>双亲委派实现要点（关键）： 1. 继承 ClassLoader，但【只重写 findClass，绝不重写 loadClass】。 loadClass 的默认模板方法会先委派  
 * parent.loadClass， 仅当整条父委派链都失败时，才回调 findClass。 2. 本类的职责仅是“父加载不到时去网络找”，从而天然保留双亲委派。 3. 在 findClass  
 * 中下载字节码并调用 defineClass 完成类定义。  
 */  
public class NetworkClassLoader extends ClassLoader {  
  
  /** 网络 class 仓库的基础地址，例如 http://localhost:8000/ */  
  private final String baseUrl;  
  
  /** 默认以系统类加载器为父 */  
  public NetworkClassLoader(String baseUrl) {  
    this.baseUrl = normalize(baseUrl);  
  }  
  
  /** 显式指定父加载器，便于演示委派链 */  
  public NetworkClassLoader(String baseUrl, ClassLoader parent) {  
    super(parent);  
    this.baseUrl = normalize(baseUrl);  
  }  
  
  private static String normalize(String url) {  
    if (url == null || url.isEmpty()) {  
      throw new IllegalArgumentException("baseUrl 不能为空");  
    }  
    return url.endsWith("/") ? url : url + "/";  
  }  
  
  /** 双亲委派下的“兜底”加载逻辑：仅当父加载器全部无法加载时才被调用。 从网络下载对应 class 的字节码并定义类。 */  
  @Override  
  protected Class<?> findClass(String name) throws ClassNotFoundException {  
    // 类名转路径：com.example.RemoteHello -> com/example/RemoteHello.class  
    String path = name.replace('.', '/') + ".class";  
    URL url;  
    try {  
      url = URI.create(baseUrl + path).toURL(); // new URL(String) 在 JDK 20 已废弃  
    } catch (MalformedURLException e) {  
      throw new ClassNotFoundException("非法的网络地址：" + baseUrl + path, e);  
    }  
  
    byte[] classData = download(url, name);  
    if (classData == null || classData.length == 0) {  
      throw new ClassNotFoundException("从网络未获取到类字节码：" + name + " @ " + url);  
    }  
  
    System.out.println("字节码长度：" + classData.length);  
    // defineClass 会执行字节码校验，并触发链接（验证/准备/解析）。  
    // 使用带 ProtectionDomain 的重载（JDK 9+），避免已废弃的无参版本。  
    Class<?> clazz = defineClass(name, classData, 0, classData.length, null);  
    if (clazz == null) {  
      throw new ClassNotFoundException("defineClass 失败：" + name);  
    }  
    return clazz;  
  }  
  
  /** 从指定 URL 下载 class 字节码，异常包装为加载失败 */  
  private byte[] download(URL url, String name) throws ClassNotFoundException {  
    ByteArrayOutputStream buffer = new ByteArrayOutputStream();  
    try (InputStream in = url.openStream()) {  
      byte[] chunk = new byte[4096];  
      int len;  
      while ((len = in.read(chunk)) != -1) {  
        buffer.write(chunk, 0, len);  
      }  
    } catch (IOException e) {  
      throw new ClassNotFoundException("下载类字节码失败：" + url, e);  
    }  
    return buffer.toByteArray();  
  }  
}
```