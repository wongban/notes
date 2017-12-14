# Head First Servlets and JSP
`CGI(Common Gateway Interface)通用网关接口` `URL(Uniform Resource Locator)统一资源定位符`
`TLD(Tag Library Descriptor)标记库描述文件` `JSTL(JSP Standard Tag Library)JSP标准定制标记库`
`WAR(Web ARchive)Web归档` `JAR(Java ARchive)Java归档`
`JNDI(Java Naming and Directory Interface)Java命名和目录接口` `RMI(Remote Method Invocation)远程方法调用`

**HTTP POST请求**

    <!-- 请求行 -->
    POST /advisor/selectBeerTaste.do HTTP/1.1
    <!-- 请求首部 -->
    Host: www.wickedlysmart.com
    User-Agent: Mozilla/5.0 (Macintosh; U; PPC Mac OS X Mach-O; en-US; rv:1.4) Gecko/
    20030624 Netscape/7.1
    Accept: text/xml,application/xml,application/xhtml+xml,text/html;q=0.9,text/
    plain;q=0.8,video/x-mng,image/png,image/jpeg,image/gif;q=0.2,*/*;q=0.1
    Accept-Language: en-us,en;q=0.5
    Accept-Encoding: gzip,deflate
    Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
    Keep-Alive: 300
    Connection: keep-alive
    <!-- 消息体，有时也称为"负载" -->
    color=dark&taste=malty

**HTTP响应**

    <!-- HTTP响应首部 -->
    HTTP/1.1 200 OK <!-- 响应的HTTP状态码、状态码的相应文本 -->
    Set-Cookie: JSESSIONID=0AAB6C8DE415E2E5F307CF334BFCA0C1; Path=/testEL
    Content-Type: text/html
    Content-Length: 397
    Date: Wed, 19 Nov 2003 03:25:40 GMT
    Server: Apache-Coyote/1.1
    Connection: close
    <!-- 体中包含HTML，或其他要显示的内容 -->
    <html>
    ...
    </html>

**servlet生命周期**

+ 容器要加载类、调用servlet的无参数构造函数，并调用servlet的init()方法，从而初始化servlet。
+ init()方法在servlet一生中值调用一次，往往在servlet为客户请求提供服务之前调用。使servlet可以访问ServletConfig和ServletContext对象。
+ 容器通过调用servlet的destroy()方法来结束servlet的生命。
+ servlet的每个请求都在一个单独的线程中运行，任何特定servlet类只有一个实例。

**三种<url-pattern\>元素**

1. 容器会按下面的顺序查找匹配。
2. 如果一个请求与多个目录<url-pattern>匹配，容器会选择最长的匹配。

```xml
<!-- 完全匹配 -->
<url-pattern>/Beer/SelectBeer.do</url-pattern>
<!-- 目录匹配 -->
<url-pattern>/Beer/*</url-pattern>
<!-- 扩展名匹配 -->
<url-pattern>*.do</url-pattern>
```

**HttpServletRequest、HttpServletResponse**

```java
String paramname = request.getParameter("paramname"); // 取得请求参数
String[] paramnames = request.getParameterValues("paramname")
response.setContentType("application/jar");// 设置内容类型（MIME）
OutputSteam os = response.getOutputStream();// 获取字节输出流
PrintWriter writer = response.getWriter();// 获取字符输出流
response.sendRedirect("http://www.oreilly.com");// 重定向，不能在响应已经提交之后才调用
RequestDispatcher view = request.getRequestDispatcher("result.jsp");// 请求分派
view.forward(request, response);
```

>根据HTTP规范GAT请求是幂等的，POST请求不是幂等的。HTML表单默认为GET。

**属性、参数**

||属性|参数
|---------|----|--------------------------------
|类型|应用上下文、请求、会话|应用上下文、servlet初始化参数、请求参数
|设置方法|setAttribute(String name, Object value)|应用和Servlet初始化参数在DD设置（请求参数，可以调整查询串）
|返回类型|Object|String
|获取方法|getAttribute(String name)|getInitParameter(String name)

```xml
<servlet>
    <servlet-name>BeerParamTest</servlet-name>
    <servlet-class>TestInitParams</servlet-class>
    <!-- servlet初始化参数 -->
    <init-param>
        <param-name>adminEmail</param-name>
        <param-value>wangbin1992@outlook.com</param-value>
    </init-param>
    <!-- 初始化JSP -->
    <servlet-name>MyTestInit</servlet-name>
    <jsp-file>/TestInit.jsp</jsp-file>
    <init-param>
        <param-name>email</param-name>
        <param-value>wangbin1992@outlook.com</param-value>
    </init-param>
</servlet>
<!-- 上下文初始化参数 -->
<context-param>
    <param-name>adminEmail</param-name>
    <param-value>wangbin1992@outlook.com</param-value>
</context-param>
```


```java
getServletConfig().getInitParameter("adminEmail");// 获取servlet初始化参数
this.getServletContext().getInitParameter("adminEmail");// 获取上下文初始化参数
getServletConfig().getServletContext().getInitParameter();
// 只有一种情况需要通过ServletConfig得到ServletContext，就是Servlet类没有扩展HttpServlet或GenericServlet。
```

除了标准的**servlet请求**、**会话**和**应用（上下文）**作用域外，JSP还增加了页面作用域
可以使用PageContext引用来得到任意作用域的属性

**九大内置对象**

    API                  隐式对象      servlet               作用域描述

    ServletContext       application  getServletContext()   应用
    HttpServletRequest   request      request               请求
    HttpSession          session      request.getSession()  会话
    PageContext          pageContext                        页面
    JspWriter            out
    HttpServletResponse  response
    ServletConfig        config
    Throwable            exception
    Object               page

**JSP元素**

+ **JSP声明**：`<%! %>`，用于声明所生成servlet类的成员
+ **Scriptlet**：`<% %>`，普通的`Java`代码
+ **表达式**：`<%= %>`，会成为`out.print()`的参数，所以不要加分号
+ **指令**：`<%@ %>`，指令用于向容器提供特殊的指示
+ **EL表达式**：`${applicationScope.mail}`
+ **动作**：**标准动作**`<jsp:include page="wickedFooter.jsp" />`，**其他动作**`<c:set var="rate" value="32" />`

**三个指令**

+ **page指令** `<%@ page import="foo.*" session="false" %>`
    定义页面特定的属性，如字符编码、页面相应的内容类型，是否要有隐式会话对象
+ **taglib指令** `<%@ taglib tagdir="/WEB-INF/tags/cool" prefix="cool" %>`
    定义JSP可以使用的标记库
+ **include指令** `<%@ include file="wickedHeader.html" %>`
    定义在转换时增加到当前页面的文本和代码。从而允许你建立可重用的块

>只有3个`JSP`指令，但是指令可以有属性，`import`是`page`指令的属性。

**禁用脚本元素、EL**

    :::xml
    <web-app>
      <jsp-config>
        <jsp-property-group>
          <url-pattern>*.jsp</url-pattern>
          <!-- 禁用脚本元素 -->
          <scripting-invalid>true</scripting-invalid>
          <!-- 忽略EL -->
          <el-ignored>true</el-ignored>
        </jsp-property-group>
      </jsp-config>
    </web-app>

**关于bean的标准动作：**

    :::jsp
    <!-- 使用了type，但没有class，bean必须已经存在
         使用了class（有或没有type），class不能是抽象类，而且必须有一个无参数的公共构造函数
         scope属性默认为page -->
    <jsp:useBean id="person" type="foo.Person" class="foo.Employee" scope="page" />
    <jsp:setProperty name="person" property="name" value="wangbin" />
    <jsp:getProperty name="person" property="name" />
    <!-- 体中的代码会有条件地运行，只有找不到bean而且创建一个新bean时才会运行 -->
    <jsp:useBean id="person" type="foo.Person" class="foo.Employee">
      <jsp:setProperty name="person" property="name" value="wangbin" />
    </jsp:useBean>

    <!-- 使用请求参数设置bean性质的几种方法
         1：结合使用标准动作和脚本 -->
    <jsp:useBean id="person" type="foo.Person" class="foo.Employee" />
    <% person.setName(request.getParameter("userName")); %>
    <!-- 2：在标准动作内部使用脚本 -->
    <jsp:setProperty name="person" property="name" value="<%=request.getParameter(\"userName\")%>" />
    <!-- 3：param属性 -->
    <jsp:setProperty name="person" property="name" param="userName" />
    <!-- 4：请求参数名与bean中的性质名相同 -->
    <input type="text" name="name">
    <jsp:setProperty name="person" property="name" />
    <!-- 5：如果所有请求参数都与bean性质名匹配 -->
    <input type="text" name="name"> <input type="text" name="empID">
    <jsp:setProperty name="person" property="*" />

**`<jsp:useBean>`生成的servlet代码**

    :::java
    foo.Person person = null;// 根据id的值声明一个变量，从而允许JSP中的其他部分（包括其他bean标记）引用这个变量。
    synchronized (request) {
        // 尝试在标记中定义的作用域中得到属性，并把结果赋给id变量。
        person = (foo.Person)_jspx_page_context.getAttribute("person", PageContext.REQUEST_SCOPE);
        if (person == null) {// 如果这个作用域中没有这样一个属性
            person = new foo.Person();// 建立一个实例，并把它赋给id变量
            _jspx_page_context.setAttribute("person", person, PageContext.REQUEST_SCOPE);
        }
    }

>把所有Java代码放在JSP中是一个不好的实践，为什么学习？已经有大量这样的JSP文件，页面中每一处都充斥着Java代码，这些Java代码嵌在scriptlet、表达式和声明标记中。到处都是脚本，对此过去没有人能做任何改变。所以，这说明必须知道如何阅读和理解这些元素。

**EL：表达式语言（Expression Language)**
>从JSP2.0规范开始，它已经正式成为规范的一部分。原先能用scriptliet和表达式完成的事情，都能用EL完成，而且EL往往更为简单。EL的用途是提供一种更简单的方法来调用Java代码，但是代码本身放在别的地方。

    EL隐式对象   或     属性

    pageScope          页面作用域中的属性
    requestScope       请求作用域中的属性
    sessionScope       会话作用域中的属性
    applicationScope   应用作用域中的属性
    param
    paramValues
    header
    headerValues
    cookie
    initParam
    pageContext

+ 表达式中第一个命名变量可以是一个隐式对象，也可以是一个属性`${firstThing.secondThing}`
+ 表达式中变量后是中括号[]，左边的变量则有更多选择，可以是Map、bean、List或数组`${musicList["something"]}`
+ 中括号里可以使用嵌套表达式`${musicMap[MusicType[0]]}`

```jsp
<!-- 请求参数 -->
<input type="text" name="name">
<input type="text" name="empID">
<input type="text" name="food"> <input type="text" name="food">
${param.name} ${param.empID} ${paramValues.food[0]} ${paramValues.food[1]}
<!-- header首部 -->
${header["host"]} ${header.host}
<!-- cookie -->
${cookie.userName.value}
<!-- 上下文初始化参数 -->
${initParam.mainEmail}
```

**包含(include)**

+ include指令`<%@ include file="Header.jsp"%>`
+ 标准动作`<jsp:include page="Header.jsp" />`

>include指令在转换时插入"header.jsp"的源代码。而标准动作在运行时插入"Header.jsp"的相应。
include指令，容器只是针对第一个请求才需要做很多工作。从第二个请求开始，就再没有额外的运行时开销了。
include标准动作，转换时没有多少工作，对应每个请求倒有更多的事情要做，特别当所包含的文件是一个JSP时，每次请求都有一些运行时开销。

    :::jsp
    <!-- 定制包含的内容 -->
    <jsp:include page="Header.jsp">
        <jsp:param name="subTitle" value="We take the sting out of SOAP." />
    </jsp:include>

    <!-- Header.jsp -->
    <em><strong>${param.subTitle}</strong></em>

**转发`<jsp:forward>`**

+ `<jsp:forward>`标准动作可以把请求转发到同一个Web应用中的另一个资源（类似RequestDispatcher)

```jsp
<!-- 缓冲区会在转发之前清空，所以如果userName为null，这句话不会打印出来 -->
Welcome to our page!
<% if (request.getParameter("userName") == null) { %>
    <jsp:forward page="HandleIt.jsp" />
<% } %>
Hello ${param.userName}
```

**JSP标准定制标记库(JSTL)**
>有时只是EL或标准动作还不够，比如循环处理一个数组中的数据。

    :::jsp
    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    <!-- escapeXml为true表示所有XML都将转换为web浏览器能呈现的形式，包括尖括号。默认为true
    有5个html特殊字符要转换：< > & ' "对应&lt; &gt; &amp; &#039; &#034; -->
    <c:out value='${pageContent.currentTip}' escapeXml='true' />
    <!-- default属性，表达式计算为null时的值 -->
    <c:out value='${user}' default='guest' />
    <c:out value='${user}'>guest</c:out>

    <c:forEach var="movie" items="${movieList}" varStatus="movieLoopCount">
        <tr><td>${movieLoopCount.count}</td></tr><!-- 循环计数器 -->
        <tr><td>${moive}</td></tr>
    </c:forEach>

    <c:if test="${userType eq 'member'}">
        <jsp:include page="inputComment.jsp"/>
    </c:if>

    <c:choose>
        <c:when test="${userPref == 'performance'}">xxx</c:when>
        <c:when test="${userPref == 'safety'}">xxx</c:when>
        <c:when test="${userPref == 'maintenance'}">xxx</c:when>
        <c:otherwise>xxx</c:otherwise>
    </c:choose>

    <!-- 设置属性变量var -->
    <c:set var="userLevel" scope="session" value="Cowboy" />
    <c:set var="Fido" value="${person.dog}" />
    <c:set var="userLevel" scope="session">Sheriff, Bartender, Cowgirl</c:set>
    <!-- 设置目标性质或值，"target"必须计算为一个对象，不能键入"id"名 -->
    <c:set target="${PetMap}" property="dogName" value="Clover" />
    <c:set target="${Person}" property="name">${foo.name}</c:set>
    <!-- 不同于另外两个包含机制，<c:import>url可以来自web容器范围之外 -->
    <c:import url="Header.jsp">
        <c:param name="subTitle" value="We take the sting out of SOAP." />
    </c:import>
    <!-- URL重写，会在最后增加jsessionid -->
    <c:url value="/inputComments.jsp" var="inputURL">
        <c:param name="uesrName" value="wang bin" />
    </c:url>
    <a href="${inputURL}">click me</a>

>标记文件利用另一个页面（使用jsp）实现标记功能。标记处理器利用一个特殊的java类实现标记功能。

**标记文件**

```jsp
<!-- attribute只能由标记文件使用，required说明属性可不可选，rtexprvalue说明可以是一个String直接量，也可以是表达式 -->
<%@ attribute name="fontColor" required="true" %>
<%@ tag body-content="tagdependent" %>

<em><strong><font color="${fontColor}"><jsp:doBody/></font></strong></em>
```

**标记处理器**

```java
/* 标记处理器类 */
public class SimpleTagTest extends SimpleTagSupport {
    public void doTag() throws JspException, IOException {
      // 处理标记的体，并把它打印到响应。
      // null参数是指输出到响应，而不是输出到作为参数传入的某个书写器writer
      getJspBody().invoke(null);
      // getJspContext().getOut().print("This is the lamest use of a custom tag");
    }
}
```

```xml
<!-- 标记的TLD -->
<?xml version="1.0" encoding="UTF-8" ?>
<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd" version="2.0">

<tlib-version>1.2</tlib-version>
<uri>simpleTags</uri><!-- taglib指令中使用的唯一名 -->
<tag>
    <description>worst use of a custom tag</description>
    <name>simple</name><!-- 标记中要用的名，例：<myTags:simple> -->
    <tag-class>foo.SimpleTagTest</tag-class>
    <body-content>scriptless</body-content><!-- 说明标记可能有体，但是体中不能有脚本(scriptlet、脚本表达式或声明) -->

    <attribute><!-- 每个标记属性都需要一个attribute元素 -->
        <name>user</name>
        <required>true</required><!-- user属性必选 -->
        <rtexprvalue>true</rtexprvalue><!-- user属性可以是一个表达式值，默认为false -->
    </attribute>
</tag>
</taglib>
```

```jsp
<!-- 使用标记的JSP -->
<%@ taglib prefix="myTags1" tagdir="/WEB-INF/tags" %>
<%@ taglib prefix="myTags2" uri="simpleTags" %>
<html>
<body>
<myTags1:Header fontColor="#660099">
    We take the sting out of SOAP.
</myTags1:Header><br>
<myTags2:simple>
    This is the body
</myTags2:simple>
</body>
</html>
```

**WEB应用部署**

+ **META-INF**目录必须是**jar**中的顶级目录，**tld**文件要放在**META-INF**目录下的某个位置（可以在任何子目录中，不必是**tlds**）。**标记文件**（扩展名为.tag或.tagx）必须放在**META-INF/tags**下的某个位置）。
+ 不在**jar**中的**tld**文件必须放在**WEB-INF**下的某个位置。**标记文件**必须放在**WEB-INF/tags**下的某个位置。
+ 在Tomcat中，war文件的文件名会成为Web应用的名字

**JNDI核心手册**
这是一个访问命名和目录服务的API。基于JNDI，可以在网络中的一个集中位置上完成查找。如果有一些对象，而且希望网络上的其他程序找到并访问这些对象，就要向JNDI注册这些对象。其他程序想使用你的对象时，则可以使用JNDI来查找。