


# 10_JavaWeb

## Servlet使用

- servlet是运行在服务器上的一个小程序，用来处理服务器请求。
- 重要步骤：
  - 自定义Servlet类，继承HttpServlet类；重写doGet，doPost方法。
  - 修改web.xml，或者用注解：
    来指定servlet与请求的映射关系，指定哪个Servlet拦截哪个请求。
- 几个重要对象：
  - HttpServletRequest：
    - doGet，doPost等方法的参数，将当前http请求的信息封装为一个HttpServletRequest对象，提供方法获取请求信息。（请求头，获取cookie）
  - HttpServletResponse：
    - doGet，doPost等方法的参数，将当前http返回的信息封装为一个HttpServletResponse对象，提供方法返回响应信息。（添加cookie，重定向到指定请求地址）
  - ServletConfig：
    - 获取Servlet信息
  - ServletContext：
    - 一个web应用在启动时会创建一个ServletContext对象，表示web应用的上下文，可以用来配置读取当前应用的全局配置，servlet之间通过servletContext对象来进行通信。
    - 获取servlet全局属性、添加过滤器、添加监听器

## Servlet的生命周期

- 调用init方法进行初始化操作，如初始化连接池，连接数据库等。
- 调用service方法处理请求，根据请求方法调用对应的doGet、doPost、doPut等方法
- 调用destroy方法销毁

## Servlet线程不安全

- 当有新的客户端请求该Servlet时，一般不会再实例化该Servlet类，也就是有多个线程在使用这个实例。

- 尽量的不要在servlet中定义成员变量。如果不得不定义成员变量，那么不要去：

  ①不要去修改成员变量的值

  ②不要去根据成员变量的值做一些逻辑判断

## Http协议

- Http 称之为超文本传输协议
- Http包含两个部分：请求和响应
  - 请求：
    - 请求包含三个部分：请求行、请求消息头、请求主体
      - 请求行包含是三个信息：请求的方式、请求的URL、请求的协议
      - 请求消息头中包含了很多客户端需要告诉服务器的信息，比如：我的浏览器型号、版本、我能接收的内容的类型、我给你发的内容的类型、内容的长度等等
      - 请求体，三种情况
        - get方式，没有请求体
        - post方式，有请求体
        - json格式，有请求体
  - 响应：
    - 响应三个部分：响应行、响应头、响应体
      - 响应行包含三个信息：协议、响应状态码(200)、响应状态(ok)
      - 响应头：包含了服务器的信息；服务器发送给浏览器的信息（内容的媒体类型、编码、内容长度等）
      - 响应体：响应的实际内容（比如请求add.html页面时，响应的内容就是<html><head><body><form....）

## 会话
- 需要原因：

  Http无状态：服务器无法判断这两次请求是同一个客户端发过来的，还是不同的客户端发过来的。
  解决：通过会话跟踪技术。

   - 会话跟踪技术
     - 客户端第一次发请求给服务器，服务器获取session，获取不到，则创建新的，然后响应给客户端。
     - 下次客户端给服务器发请求时，会把sessionID带给服务器，那么服务器就能获取到了，那么服务器就判断这一次请求和上次某次请求是同一个客户端，从而能够区分开客户端。
         - 常用的API：
         request.getSession() -> 获取当前的会话，没有则创建一个新的会话
     - session保存作用域和具体的某一个session对应
         - 常用的API：
           - session.setAttribute(k,v)
           - session.getAttribute(k)
           - removeAttribute(k)

## 服务器内部转发以及客户端重定向

- 服务器内部转发 : 
  - request.getRequestDispatcher("...").forward(request,response);
  - 一次请求响应的过程，对于客户端而言，内部经过了多少次转发，客户端是不知道的，地址栏没有变化
- 客户端重定向： response.sendRedirect("....");
  - 两次请求响应的过程。客户端肯定知道请求URL有变化，地址栏有变化