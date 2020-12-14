### ssm 用户注册登录  
注册
所谓注册即是向系统中新增数据，新增一条登陆用户名，密码等。
登陆  
登陆是将用户的输入数据与数据库中的原有数据进行对比，输入正确即对比成功则登陆成功进行相关操作，否则检测错误存在。  

注册页`index.jsp`
```html
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Index</title>
    <script src="/js/jquery-3.2.1.min.js" type="text/javascript"></script>
</head>
<body>
<br>
<br>
<form action="/registe" method="post">
    <table>
        <tr>
            <td>
                用户名：
            </td>
            <td>
                <input type="text" name="name" placeholder="name">
            </td>
        </tr>
        <tr>
            <td>
                密码：
            </td>
            <td>
                <input type="password" name="password">
            </td>
        </tr>
        <tr>
            <td>
                &nbsp;
            </td>
            <td>
                <input type="submit" value="提交">
            </td>
        </tr>
    </table>
</form>
</body>
<script type="text/javascript">
    $(function () {
        if ((${result})) {
            if ((${result}) == 1) {
                alert("注册成功！");
            } else {
                alert("未知错误，重新注册");
            }
        } else {
            alert("");
        }
    })
</script>
</html>
```
对应的controller映射函数：
创建对象 从请求参数set属性，调用dao把这个对象insert数据库；
```java
package com.springmvc.controller;
//import ***
import java.util.UUID;

@Controller
public class IndexController {

    @Autowired
    private UserServices userServices;

    @RequestMapping(value = "/registe",method = RequestMethod.POST)
    public String toregiste(@RequestParam String name, @RequestParam  String password, Model model) {
        User user = new User();
        String uuid = UUID.randomUUID().toString().replace("-", "").toLowerCase();
        user.setUuid(uuid);
        user.setName(name.trim());
        user.setPassword(password.trim());
        try {
            userServices.insert(user);
            model.addAttribute("result", "1");
        } catch (Exception e) {
            model.addAttribute("result", "0");
        }
        return "index";

    }
}
```
登录页:
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Login</title>
    <script type="text/javascript" src="/js/jquery-3.2.1.min.js"></script>
</head>
<body>
<br>
<br>
<form action="/login" method="post">
    <table>
        <tr>
            <td>
                用户名：
            </td>
            <td>
                <input type="text" name="name" placeholder="name">
            </td>
        </tr>
        <tr>
            <td>
                密码：
            </td>
            <td>
                <input type="password" name="password">
            </td>
        </tr>
        <tr>
            <td>
                &nbsp;
            </td>
            <td>
                <input type="submit" value="登陆">
            </td>
        </tr>
    </table>
</form>
</body>
<script type="text/javascript">
    $(function () {
        if ((${result}) != null) {
            if ((${result}) == "1") {
            } else if ((${result}) == "0") {
                alert("账号或密码不正确");
            } else if ((${result}) == "-1") {
                alert("请填写完整");
            } else {
                alert("c");
            }
        }
    })
</script>
</html>
```
控制器：
```java
    @RequestMapping(value = "/login",method = RequestMethod.POST)
    public String toLogin(@RequestParam String name, @RequestParam  String password, Model model, HttpSession session) {
        if (name.equals("") || password.equals("")) {
            model.addAttribute("result", "-1");

        } else {
            User user = userServices.selectByName(name.trim());
            if (user !=null) {
                if (password.equals(user.getPassword())) {
                    session.setAttribute("user", user);
                    model.addAttribute("user",user);
                    model.addAttribute("result", "1");
                    return "user";
                }
            } else {
                model.addAttribute("result", "0");
            }
        }
        return "login";
    }
```
登录success后`user.jsp`
```html
<%--欢迎页--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>User</title>
    <script type="text/javascript" src="/js/jquery-3.2.1.min.js"></script>
</head>
<body>
你好，欢迎你${user.name},<a href="/logout">退出登陆</a>
</body>
</html>
```
退出
```java
@RequestMapping(value = "/logout")
    public String toLogout(HttpSession session) {
        session.removeAttribute("user");
        return "login";
    }
```






