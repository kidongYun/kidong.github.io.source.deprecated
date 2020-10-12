---
layout: post
title:  "Spring controller data types"
date:   2019-11-04 08:47:54 +0900
categories: spring
---

### HttpServletRequest

```js

    function httpServletRequestSample() {
        let data = {
            "name" : "kidongyun",
            "age"  : 26
        }

        $.ajax({
            url:'/httpServletRequestSample',
            type:'get',
            data: data,
            success:function(){

            }
        })
    }

```

```java

    @RequestMapping(value = "/httpServletRequestSample")
    public void httpServletRequestSample(HttpServletRequest request){

        String name = request.getParameter("name");
        String age = request.getParameter("age");

        System.out.println("HttpServletRequest(Object -> Primitive) : " + name + ", " + age);
    }

```

### @RequestParam

```js

    function requestParamSample() {
        let data = {
            "name" : "kidongyun",
            "age"  : 26
        }

        $.ajax({
            url:'/requestParamSample',
            type:'get',
            data: data,
            success:function(){

            }
        })
    }

```

```java

    @RequestMapping(value = "/requestParamSample")
    public void requestParamSample(@RequestParam String name, @RequestParam int age) {
        System.out.println("RequestParam(Object -> Primitive) : " + name + ", " + age);
    }

```

### @RequestBody Map

```js

    function requestBodyMapTypeSample() {
        let data = {
            "name" : "kidongyun",
            "age"  : 26
        }

        $.ajax({
            url:'/requestBodyMapTypeSample',
            type:'post',
            contentType: 'application/json',
            data: JSON.stringify(data),
            success:function(){

            }
        })
    }

```

```java

    @RequestMapping(value = "/requestBodyMapTypeSample")
    public void requestBodyMapTypeSample(@RequestBody Map<String, Object> param) {
        System.out.println("RequestBody(Json -> Map) : " + param.get("name") + ", " + param.get("age"));
    }

```

### @RequestBody VO

```js

    function requestBodyVOTypeSample() {
        let data = {
            "name" : "kidongyun",
            "age"  : 26
        }

        $.ajax({
            url:'/requestBodyVOTypeSample',
            type:'post',
            contentType: 'application/json',
            data: JSON.stringify(data),
            success:function(){

            }
        })
    }

```

```java

    @RequestMapping(value = "/requestBodyVOTypeSample")
    public void requestBodyVOTypeSample(@RequestBody Human human) {
        System.out.println("RequestBody(Json -> VO) : " + human.getName() + ", " + human.getAge());
    }

```

### ModelAndView

```jsp

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>ModelAndViewSamplePage</title>
</head>
<body>
    ${name}
    ${age}
</body>
</html>

```

```java

    @RequestMapping(value = "/modelAndViewSample")
    public ModelAndView modelAndViewSample() {
        ModelAndView view = new ModelAndView();

        view.addObject("name", "kidongyun");
        view.addObject("age", 26);
        view.setViewName("/modelAndViewSamplePage");

        return view;
    }

```

### ViewResolver

```jsp

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>ViewResolverSample</title>
</head>
<body>
    ViewResolverSample
</body>
</html>

```

```java

    @RequestMapping(value = "/viewResolverSample")
    public String viewResolverSample() {
        return "viewResolverSample";
    }

```

### HttpServletResponse
```java

    @RequestMapping(value = "/httpServletResponseSample")
    public void httpServletResponseSample(HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();

        out.println("<html>");
        out.println("<head><title>httpServletResponseSample</title></head>");
        out.println("<body>");
        out.println("name : kidongyun");
        out.println("age : 26");
        out.println("</body>");
        out.println("</html>");
    }

```

### @ResponseBody

```js

    function responseBodyStringTypeSample() {
        $.ajax({
            url:'/responseBodyStringTypeSample',
            type:'get',
            success:function(data){
                console.log(data);
            }
        })
    }

```

```java

    @ResponseBody
    @RequestMapping(value = "/responseBodyStringTypeSample")
    public String responseBodyStringTypeSample() {
        return "It's plain String Type";
    }

```
