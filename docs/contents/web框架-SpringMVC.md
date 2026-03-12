# Spring MVC

## 概述

### Spring MVC 是什么

Spring Web MVC 是一种基于 Java 实现的 Web MVC 设计模式的请求驱动型轻量级 Web 框架。它将 Web 层进行职责解耦，基于请求-响应模型工作。

**MVC 架构：**

- **模型（Model）** - 封装应用程序数据，通常由 POJO 类组成
- **视图（View）** - 负责渲染模型数据，生成 HTML 输出
- **控制器（Controller）** - 负责处理用户请求，构建模型并传递给视图

### Spring MVC 版本演进

| 版本 | 发布年份 | 主要特性 |
|------|----------|----------|
| Spring 2.5 | 2007 | 引入注解支持（@RequestMapping） |
| Spring 3.0 | 2009 | REST 支持、注解驱动、验证器 |
| Spring 3.2 | 2013 | Spring MVC 测试框架、异步处理 |
| Spring 4.0 | 2013 | REST 增强、WebSocket 支持 |
| Spring 4.3 | 2016 | 组合注解（@GetMapping 等）、CORS |
| Spring 5.0 | 2017 | 函数式路由、Reactive 响应式支持 |
| Spring 5.3 | 2020 | 改进的日志、RFC 7807 响应式错误处理 |
| Spring 6.0 | 2022 | Jakarta EE 9+、UTF-8 默认编码 |

---

## 核心概念与组件

### 核心组件

| 组件 | 说明 |
|------|------|
| **DispatcherServlet** | 前端控制器，Spring MVC 的统一入口 |
| **HandlerMapping** | 处理器映射器，根据 URL 找到对应的 Handler |
| **HandlerAdapter** | 处理器适配器，调用具体的处理方法 |
| **ViewResolver** | 视图解析器，解析视图名称到具体视图 |
| **View** | 视图，负责渲染模型数据 |
| **HandlerInterceptor** | 拦截器，在请求前后执行自定义逻辑 |

### WebApplicationContext

WebApplicationContext 是专门为 Web 应用设计的上下文环境，是 ApplicationContext 的扩展。

```
┌─────────────────────────────────────────────────────────────┐
│                  根 ApplicationContext                      │
│              (业务层、数据层 Bean)                           │
└─────────────────────────────────────────────────────────────┘
                          │
         ┌────────────────┴────────────────┐
         ▼                                 ▼
┌─────────────────────┐         ┌─────────────────────┐
│ 父容器可访问子容器  │         │  子 WebApplicationContext │
│ 子容器不可访问父容器│         │   (Controller、ViewResolver)│
└─────────────────────┘         └─────────────────────┘
```

**父子容器关系：**
- 子容器可以访问父容器的 Bean
- 父容器不能访问子容器的 Bean
- 通常：根容器配置 DAO/Service，子容器配置 Controller

---

## 执行流程

### 请求处理流程

```
┌─────────────────────────────────────────────────────────────┐
│                        HTTP 请求                            │
└─────────────────────────┬───────────────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                   DispatcherServlet                         │
│                    (前端控制器)                              │
└─────────┬─────────────────────────────────────┬───────────────┘
          │                                     │
          ▼                                     ▼
┌─────────────────────┐           ┌─────────────────────────┐
│    HandlerMapping   │           │   HandlerInterceptor    │
│   (找到 Handler)    │           │     (预处理/后处理)      │
└─────────┬───────────┘           └────────────┬────────────┘
          │                                    │
          ▼                                    │
┌─────────────────────┐                       │
│   HandlerAdapter    │───────────────────────┘
│  (调用 Handler)     │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│    Controller       │
│   (处理业务逻辑)    │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   ViewResolver      │
│  (解析视图名称)     │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│        View         │
│   (渲染视图)        │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────┐
│                        HTTP 响应                            │
└─────────────────────────────────────────────────────────────┘
```

### DispatcherServlet 初始化

DispatcherServlet 启动时会根据 `DispatcherServlet.properties` 配置文件初始化以下组件：

1. HandlerMapping - 处理器映射器
2. HandlerAdapter - 处理器适配器
3. ViewResolver - 视图解析器
4. LocaleResolver - 本地化解析器
5. ThemeResolver - 主题解析器
6. MultipartResolver - 文件上传解析器
7. ExceptionHandlerExceptionResolver - 异常处理器

---

## Spring Boot 集成

### 快速入门

```java
// 1. 添加依赖
// pom.xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```java
// 2. 创建 Controller
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        // 查询用户
        return new User(id, "张三", 25);
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        // 创建用户
        return user;
    }
}
```

```java
// 3. 启动类
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 配置文件

```properties
# 服务端口
server.port=8080

# 应用名称
spring.application.name=myapp

# 上下文路径
server.servlet.context-path=/api

# Jackson 配置
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
spring.jackson.default-property-inclusion=non_null

# 文件上传
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=20MB

# 静态资源
spring.web.resources.static-locations=classpath:/static/,classpath:/public/
spring.mvc.static-path-pattern=/static/**
```

**常用配置项：**

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| server.port | 服务端口 | 8080 |
| server.servlet.context-path | 上下文路径 | / |
| spring.mvc.view.prefix | 视图前缀 | - |
| spring.mvc.view.suffix | 视图后缀 | - |
| spring.web.resources.static-locations | 静态资源路径 | classpath:/static |
| spring.mvc.static-path-pattern | 静态资源路径匹配 | /static/** |

---

## 控制器注解

### @RequestMapping

```java
// 类级别：指定基础路径
@RequestMapping("/api")
public class ApiController {

    // 方法级别：指定具体路径
    @RequestMapping("/users")
    public String users() {
        return "users";
    }
}
```

### 组合注解

```java
@GetMapping       // 查询
@PostMapping      // 创建
@PutMapping       // 更新
@DeleteMapping    // 删除
@PatchMapping     // 部分更新
```

```java
@GetMapping("/users")
public List<User> getUsers() {
    return userService.findAll();
}

@PostMapping("/users")
public User createUser(@RequestBody User user) {
    return userService.save(user);
}

@PutMapping("/users/{id}")
public User updateUser(@PathVariable Long id, @RequestBody User user) {
    user.setId(id);
    return userService.update(user);
}

@DeleteMapping("/users/{id}")
public void deleteUser(@PathVariable Long id) {
    userService.delete(id);
}
```

### 注解属性

```java
@RequestMapping(
    value = "/users",
    method = RequestMethod.GET,      // 请求方法
    params = "action=view",          // 请求参数条件
    headers = "Content-Type=application/json",  // 请求头条件
    consumes = "application/json",   // 消费的数据类型
    produces = "application/json"     // 生产的数据类型
)
public User getUser() {
    return new User();
}
```

---

## 参数绑定

### 基本类型参数

```java
// URL 参数绑定
// GET /users?id=1&name=张三
@GetMapping("/users")
public User getUser(Long id, String name) {
    // 参数名与请求参数一致
    return userService.findById(id);
}

// 使用 @RequestParam 指定参数名
@GetMapping("/users")
public User getUser(@RequestParam("id") Long userId,
                    @RequestParam(value = "name", required = false) String userName) {
    return userService.findById(userId);
}
```

**@RequestParam 属性：**

| 属性 | 说明 |
|------|------|
| value/name | 请求参数名 |
| required | 是否必需，默认 true |
| defaultValue | 默认值 |

### PathVariable 路径变量

```java
// RESTful 风格 URL
// GET /users/1
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);
}

// 多个路径变量
// GET /users/1/orders/10
@GetMapping("/users/{userId}/orders/{orderId}")
public Order getOrder(@PathVariable Long userId, @PathVariable Long orderId) {
    return orderService.findById(userId, orderId);
}

// 路径变量名与方法参数名不同
@GetMapping("/users/{id}")
public User getUser(@PathVariable("id") Long userId) {
    return userService.findById(userId);
}
```

### POJO 对象绑定

```java
// 表单提交或 URL 参数
// GET /users?id=1&name=张三&age=25
@GetMapping("/users")
public User getUser(User user) {
    // 自动将参数映射到 User 对象的属性
    return user;
}

// POST 请求体绑定
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    return userService.save(user);
}
```

### @RequestBody 与 @ResponseBody

```java
// @RequestBody - 将请求体反序列化为对象
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    return userService.save(user);
}

// @ResponseBody - 将返回值序列化为响应体
@ResponseBody
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    return userService.save(user);
}

// @RestController = @Controller + @ResponseBody
@RestController
public class UserController {
    // 所有方法都返回 JSON
}
```

### @ModelAttribute

```java
// 绑定请求参数到对象
@PostMapping("/users/save")
public String saveUser(@ModelAttribute User user) {
    userService.save(user);
    return "redirect:/users";
}

// 在方法参数中使用
@GetMapping("/users/edit")
public String editUser(@ModelAttribute User user) {
    // 会自动从 session 或请求参数中绑定
    return "user-edit";
}

// 标注在方法上 - 在所有方法执行前执行
@ModelAttribute
public void init(Model model) {
    model.addAttribute("message", "Hello");
}
```

---

## 响应处理

### 返回视图

```java
// 返回视图名称
@Controller
public class UserController {

    @GetMapping("/users")
    public String listUsers(Model model) {
        model.addAttribute("users", userService.findAll());
        return "user-list";  // 视图名称
    }
}
```

### 返回 JSON

```java
// 返回 JSON 数据
@RestController
public class UserController {

    @GetMapping("/api/users")
    public List<User> getUsers() {
        return userService.findAll();
    }

    @GetMapping("/api/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}
```

### 返回文本

```java
// 返回纯文本
@GetMapping("/text")
@ResponseBody
public String getText() {
    return "Hello World";
}
```

### Model 和 ModelMap

```java
@GetMapping("/users")
public String listUsers(Model model) {
    // 添加属性
    model.addAttribute("users", userService.findAll());
    model.addAttribute("count", 10);

    // 指定属性在视图中的名称
    model.addAttribute("userList", userService.findAll());

    // 返回视图
    return "user-list";
}

@GetMapping("/users")
public String listUsers(ModelMap modelMap) {
    // ModelMap 的用法与 Model 类似
    modelMap.addAttribute("users", userService.findAll());
    return "user-list";
}
```

### Session Attribute

```java
// 操作 Session 属性
@GetMapping("/cart")
public String cart(HttpSession session, Model model) {
    List<Item> items = (List<Item>) session.getAttribute("cart");
    model.addAttribute("items", items);
    return "cart";
}

@PostMapping("/cart/add")
public String addToCart(@RequestParam Long itemId, HttpSession session) {
    List<Item> cart = (List<Item>) session.getAttribute("cart");
    if (cart == null) {
        cart = new ArrayList<>();
    }
    cart.add(itemService.findById(itemId));
    session.setAttribute("cart", cart);
    return "redirect:/cart";
}
```

---

## 拦截器

### HandlerInterceptor 接口

```java
public class AuthInterceptor implements HandlerInterceptor {

    // 预处理 - 在 Controller 执行前调用
    @Override
    public boolean preHandle(HttpServletRequest request,
                            HttpServletResponse response,
                            Object handler) throws Exception {
        String token = request.getHeader("Authorization");
        if (token == null) {
            response.setStatus(401);
            return false;  // 不继续执行
        }
        return true;  // 继续执行
    }

    // 后处理 - 在 Controller 执行后、视图渲染前调用
    @Override
    public void postHandle(HttpServletRequest request,
                          HttpServletResponse response,
                          Object handler,
                          ModelAndView modelAndView) throws Exception {
        // 可以修改 ModelAndView
        if (modelAndView != null) {
            modelAndView.addObject("timestamp", System.currentTimeMillis());
        }
    }

    // 完成处理 - 在视图渲染后调用
    @Override
    public void afterCompletion(HttpServletRequest request,
                               HttpServletResponse response,
                               Object handler,
                               Exception ex) throws Exception {
        // 清理资源、记录日志
        if (ex != null) {
            log.error("Request processing error", ex);
        }
    }
}
```

### 注册拦截器

```java
// 方式1：实现 WebMvcConfigurer
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuthInterceptor())
                .addPathPatterns("/api/**")      // 拦截路径
                .excludePathPatterns("/api/login", "/api/public/**");  // 排除路径
    }
}

// 方式2：@Bean 方式
@Configuration
public class WebConfig {

    @Bean
    public AuthInterceptor authInterceptor() {
        return new AuthInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authInterceptor())
                .addPathPatterns("/api/**");
    }
}
```

### 拦截器执行顺序

```
请求 → Interceptor1.preHandle → Interceptor2.preHandle → Controller
                                                  ↓
                                          Interceptor2.postHandle
                                          Interceptor1.postHandle
                                                  ↓
                                              View 渲染
                                                  ↓
                                          Interceptor2.afterCompletion
                                          Interceptor1.afterCompletion → 响应
```

---

## 异常处理

### @ExceptionHandler

```java
@Controller
public class UserController {

    @ExceptionHandler(NullPointerException.class)
    public String handleNullPointer(NullPointerException e, Model model) {
        model.addAttribute("error", "空指针异常: " + e.getMessage());
        return "error";
    }

    @ExceptionHandler(BusinessException.class)
    @ResponseBody
    public Result handleBusiness(BusinessException e) {
        return Result.error(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    @ResponseBody
    public Result handleException(Exception e) {
        return Result.error(500, "系统错误");
    }
}
```

### @ControllerAdvice 全局异常处理

```java
// 全局异常处理
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NullPointerException.class)
    public Result handleNullPointer(NullPointerException e) {
        return Result.error(500, "空指针异常");
    }

    @ExceptionHandler(BusinessException.class)
    public Result handleBusiness(BusinessException e) {
        return Result.error(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result handleValidation(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldError().getDefaultMessage();
        return Result.error(400, message);
    }
}
```

### 自定义异常

```java
// 自定义业务异常
public class BusinessException extends RuntimeException {
    private Integer code;

    public BusinessException(Integer code, String message) {
        super(message);
        this.code = code;
    }

    public Integer getCode() {
        return code;
    }
}

// 使用
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    if (user.getName() == null) {
        throw new BusinessException(400, "用户名不能为空");
    }
    return userService.save(user);
}
```

---

## 文件上传

### 单文件上传

```java
@RestController
public class FileController {

    @PostMapping("/upload")
    public Result upload(@RequestParam("file") MultipartFile file) {
        if (file.isEmpty()) {
            return Result.error(400, "文件为空");
        }

        String filename = file.getOriginalFilename();
        String path = "uploads/" + filename;

        try {
            file.transferTo(new File(path));
            return Result.ok(path);
        } catch (IOException e) {
            return Result.error(500, "文件上传失败");
        }
    }
}
```

### 多文件上传

```java
@PostMapping("/upload/multiple")
public Result uploadMultiple(@RequestParam("files") MultipartFile[] files) {
    List<String> paths = new ArrayList<>();

    for (MultipartFile file : files) {
        if (!file.isEmpty()) {
            String filename = file.getOriginalFilename();
            String path = "uploads/" + filename;
            try {
                file.transferTo(new File(path));
                paths.add(path);
            } catch (IOException e) {
                return Result.error(500, "文件上传失败: " + filename);
            }
        }
    }

    return Result.ok(paths);
}
```

### Spring Boot 配置

```properties
# 文件上传配置
spring.servlet.multipart.enabled=true
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=20MB
spring.servlet.multipart.file-size-threshold=2KB
spring.servlet.multipart.location=${java.io.tmpdir}
```

---

## RESTful 风格

### RESTful API 设计

```
GET    /api/users        # 获取用户列表
GET    /api/users/{id}   # 获取单个用户
POST   /api/users        # 创建用户
PUT    /api/users/{id}   # 更新用户
DELETE /api/users/{id}   # 删除用户
```

### 完整示例

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    // 获取用户列表
    @GetMapping
    public Result<List<User>> list(
            @RequestParam(defaultValue = "1") Integer pageNum,
            @RequestParam(defaultValue = "10") Integer pageSize) {
        Page<User> page = new Page<>(pageNum, pageSize);
        IPage<User> result = userService.page(page);
        return Result.ok(result.getRecords());
    }

    // 获取单个用户
    @GetMapping("/{id}")
    public Result<User> getById(@PathVariable Long id) {
        User user = userService.getById(id);
        if (user == null) {
            return Result.error(404, "用户不存在");
        }
        return Result.ok(user);
    }

    // 创建用户
    @PostMapping
    public Result<User> create(@RequestBody @Valid User user) {
        userService.save(user);
        return Result.ok(user);
    }

    // 更新用户
    @PutMapping("/{id}")
    public Result<User> update(@PathVariable Long id, @RequestBody @Valid User user) {
        user.setId(id);
        userService.updateById(user);
        return Result.ok(user);
    }

    // 删除用户
    @DeleteMapping("/{id}")
    public Result<Void> delete(@PathVariable Long id) {
        userService.removeById(id);
        return Result.ok();
    }
}
```

### 统一返回结果

```java
@Data
public class Result<T> {
    private Integer code;
    private String message;
    private T data;

    public static <T> Result<T> ok(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage("success");
        result.setData(data);
        return result;
    }

    public static <T> Result<T> error(Integer code, String message) {
        Result<T> result = new Result<>();
        result.setCode(code);
        result.setMessage(message);
        return result;
    }
}
```

---

## 常用配置

### 静态资源处理

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    // 方式1：配置文件
    // spring.web.resources.static-locations=classpath:/static/
    // spring.mvc.static-path-pattern=/static/**

    // 方式2：代码配置
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/static/");

        // 外部文件目录
        registry.addResourceHandler("/uploads/**")
                .addResourceLocations("file:uploads/");
    }
}
```

### 视图解析器配置

```properties
# Thymeleaf 配置
spring.thymeleaf.cache=false
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
spring.thymeleaf.mode=HTML
spring.thymeleaf.encoding=UTF-8
```

```java
// JSP 视图解析器配置
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/views/", ".jsp");
    }
}
```

### CORS 跨域配置

```java
// 方式1：注解方式
@RestController
@CrossOrigin(origins = "*")
public class UserController {
    // 所有方法支持跨域
}

// 方法级别
@CrossOrigin(origins = "http://localhost:3000")
@GetMapping("/users")
public List<User> getUsers() {
    return users;
}

// 方式2：全局配置
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:3000")
                .allowedMethods("GET", "POST", "PUT", "DELETE")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }
}
```

### 拦截器配置

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 日志拦截器
        registry.addInterceptor(new LogInterceptor())
                .addPathPatterns("/api/**")
                .order(1);

        // 认证拦截器
        registry.addInterceptor(new AuthInterceptor())
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/login", "/api/public/**")
                .order(2);
    }
}
```

---

## 常见问题与 Tips

### Tips

**1. 获取 HttpServletRequest**

```java
@GetMapping("/request")
public void handleRequest(HttpServletRequest request) {
    String uri = request.getRequestURI();
    String method = request.getMethod();
    String ip = request.getRemoteAddr();
}
```

**2. 重定向**

```java
// 重定向到视图
return "redirect:/users";

// 重定向到 URL
return "redirect:https://example.com";

// 带参数重定向
return "redirect:/users?id=" + id;
```

**3. 转发**

```java
// 转发到视图
return "forward:/users";

// 转发到其他 Controller
return "forward:/api/users";
```

**4. 避免空指针**

```java
// 使用 Optional
@GetMapping("/user")
public Optional<User> getUser(@RequestParam Long id) {
    return userService.findById(id);
}
```

### FAQ

**Q: 404 但 Controller 存在？**

A: 检查：
1. Controller 是否在包扫描范围内
2. @RequestMapping 路径是否正确
3. Spring Boot 自动配置是否生效

**Q: 参数绑定失败？**

A: 检查：
1. 参数名与请求参数名是否一致
2. 数据类型是否匹配
3. 是否需要 @RequestParam 注解

**Q: 中文乱码？**

A: 解决方案：
```properties
server.servlet.encoding.charset=UTF-8
server.servlet.encoding.enabled=true
server.servlet.encoding.force=true
```

**Q: 文件上传大小限制？**

A: 配置：
```properties
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=50MB
```

**Q: @ResponseBody 返回字符串带双引号？**

A: 使用 @RestController 或在方法上添加 @ResponseBody

**Q: Session 共享问题？**

A: 分布式环境下使用 Redis Session：
```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

**Q: Spring MVC 执行流程？**

A:
1. 请求到达 DispatcherServlet
2. HandlerMapping 找到 Handler
3. HandlerInterceptor.preHandle
4. HandlerAdapter 调用 Handler
5. Handler 返回 ModelAndView
6. ViewResolver 解析视图
7. View 渲染页面
8. HandlerInterceptor.postHandle
9. HandlerInterceptor.afterCompletion