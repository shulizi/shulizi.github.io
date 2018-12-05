---
layout: post
title:  "android开发篇：javaweb实现android注册与登录"
date:   2015-12-28
categories: android
tags: javaweb android load register
---

* content
{:toc}

viewpager和listview的合作据我观察是目前手机app比较流行的一种页面开发方式，这里简单探讨一下。






## 项目：register

### LoginServlet.java
注册 连接数据库 插入信息
``` java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // TODO Auto-generated method stub
        String username = request.getParameter("username");
        String userpass = request.getParameter("password");
        if(username.equals("")||userpass.equals("")){
            response.getWriter().append("EMPTY USERNAME OR PASSWORD");
            return ;
        }
        String driver = "com.mysql.jdbc.Driver";
        String url = "jdbc:mysql://localhost:3306/test";
        String user = "root";
        String password = "root";
       Connection ct=null;
       PreparedStatement ps=null;
        try
        {
            Class.forName(driver);
            ct= DriverManager.getConnection(url, user, password);
            ps=(PreparedStatement) ct.prepareStatement("insert into  users            (username,password) values(?,?);");
            ps.setObject(1, username);
            ps.setObject(2, userpass);
            ps.executeUpdate();
            request.getRequestDispatcher("/MainFrame").forward(request, response);
        }
        catch (Exception ex)
        {
            response.getWriter().append("ERROR! YOUR NAME HAS BEEN REGISTERED!");
            ex.printStackTrace();
        }
        finally {
            if(ps!=null)
                try {
                    ps.close();
                } catch (SQLException e) {
                    // TODO Auto-generated catch block
                   e.printStackTrace();
                }
            if(ct!=null)
                try {
                    ct.close();
                } catch (SQLException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
        }


    }

```
### MainFrame.java
``` java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // TODO Auto-generated method stub
        response.getWriter().append("SUCCESS!YOU HAVE REGISTERED:"+request.getParameter("username"));
    }

```
### web.xml
``` xml
<servlet>
    <servlet-name>LoginServlet</servlet-name>
    <servlet-class>com.login.servlet.LoginServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>LoginServlet</servlet-name>
    <url-pattern>/index.html</url-pattern>
</servlet-mapping>


```
## 项目：Login
### LoginSevlet.java


``` java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // TODO Auto-generated method stub
        String driver = "com.mysql.jdbc.Driver";
        String url = "jdbc:mysql://localhost:3306/test";
        String user = "root";
        String password = "root";
        String username=request.getParameter("username");
        String userpass=request.getParameter("password");
       Connection ct=null;
       PreparedStatement ps=null;
       ResultSet rs=null;
        try
        {
            Class.forName(driver);
            ct= DriverManager.getConnection(url, user, password);
            ps=(PreparedStatement) ct.prepareStatement("select * from users where username=? and password=?");
            ps.setObject(1, username);
            ps.setObject(2, userpass);
            rs=ps.executeQuery();
            if(rs.next())
                    request.getRequestDispatcher("/MainFrame").forward(request, response);
            else
                response.getWriter().append("USERNAME OR PASSWORD WRONG!");
        }
        catch (Exception ex)
        {
            response.getWriter().append("ERROR!");
            ex.printStackTrace();
        }
        finally {
            if(ps!=null)
                try {
                    ps.close();
                } catch (SQLException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            if(ct!=null)
                try {
                    ct.close();
                } catch (SQLException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
        }
    }
```
MainFrame.java和web.xml配置与项目register一致，在这不赘述了。
