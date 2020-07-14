---
layout: post
title: summernote的使用
categories:
  - Bootstrap
description: summernote的使用
keywords: 'summernote, 富文本编辑器'
---

Summernote是一个JavaScript库，可帮助您在线创建所见即所得编辑器。

## 安装使用

- 下载summernote，[summernote官网](http://summernote.org/)，[summernote在GitHub上的地址](https://github.com/summernote/summernote)

- 在页面中引入。因为Summernote基于jQuery和Bootstrap库， 所以我们在使用的使用需要引入jQuery和Bootstrap。

```
<!-- include libraries(jQuery, bootstrap) -->
<script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/jquery/2.1.4/jquery.min.js"></script>
<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.5/css/bootstrap.min.css" />
<script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.5/js/bootstrap.min.js"></script>

<!-- include summernote css/js-->
<link href="summernote.css" rel="stylesheet">
<script src="summernote.js"></script>
```

- 创建一个目标div。

```
<div id="summernote">Hello Summernote</div>
```

- 根据id进行初始化。（当需要多个富文本框时，也可以在div中设置class属性，根据class属性进行初始化）

```
$(document).ready(function() {
  $('#summernote').summernote();
});
```

## 在项目中的使用

- jsp页面中的代码

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
    <%@include file="/static/common/common.jsp"%>
    <link href="/static/plugin/summernote/summernote.css" rel="stylesheet">
    <script src="/static/plugin/summernote/summernote.js"></script>
    <script src="/static/plugin/summernote/lang/summernote-zh-CN.js"></script>
    <script src="/static/js/init_summernote.js"></script>
</head>
<body>
<div class="container">
    <div class="wrapper">
        <div class="row">
            <div class="col-lg-12">
                <textarea id="summernote" name="content" class="form-control">${message.content}</textarea>
            </div>
        </div>
    </div>
</div>
</body>
</html>
```

- `<script src="/static/js/init_summernote.js"></script>`，这一句引入自己定义的初始化summernote的js代码。 `init_summernote.js`页面的代码如下

```
$(document).ready(function(){

    /**
     * 初始化编辑器
     */
    $('#summernote').summernote({
        /*自定义工具栏*/
        toolbar: [
            ['style', ['style']],
            ['fontsize', ['fontsize']],
            ['font', ['bold', 'italic', 'underline', 'clear']],
            ['fontname', ['fontname']],
            ['color', ['color']],
            ['para', ['ul', 'ol', 'paragraph']],
            ['height', ['height']],
            ['insert', ['hr','link','picture', 'video'],['template']],
            ['table', ['table']],
            ['view', ['fullscreen', 'codeview']],
        ],
        /*设置高度*/
        height: 300,
        /*设置语言为简体汉字*/
        lang:'zh-CN',
        /*设置字体*/
        fontNames: ['宋体','仿宋','楷体','黑体','幼圆','华文行楷','华文隶书','华文细黑','新宋体',
            'Microsoft Yahei',
            'Arial', 'Arial Black', 'Comic Sans MS', 'Courier New',
            'Impact','Tahoma','Times New Roman', 'Verdana'],
        /*定义回调函数*/
        callbacks: {
            onImageUpload: function(image) {
                uploadImage(image[0]);
            }
        }
    });

    /**
     * 上传图片
     * @param image
     */
    function uploadImage(image) {
        var data = new FormData();
        data.append("image", image);
        $.ajax({
            url: "/summerNote/uploadImage",
            cache: false,
            contentType: false,
            processData: false,
            data: data,
            type: "post",
            success: function(url) {
                var image = $('<img>').attr('src', url);
                $('#summernote').summernote("insertNode", image[0]);
            },
            error: function(data) {
                console.log(data);
            }
        });
    }

});
```

我们自己定义了图片上传的方法，后台使用SpringMVC上传图片。

页面效果如下

![](http://sunmood.github.io/assets/images/summernote/example.png)
