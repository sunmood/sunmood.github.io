---
layout: post
title: Bootstrap模态框的使用
categories:
  - Bootstrap
description: null
keywords: Bootstrap
---

# Bootstrap模态框的使用，实现原页面和弹出框页面分离

## 分离了原页面和弹出框页面。具体如下：

**1\. 页面需要引入的js代码**

```
$(function(){
    $(".myModal").click(function(){
       /*链接url*/
        var href = $(this).attr("data-href");
        /*模态窗口宽度*/
        var modalWidth = $(this).attr("data-width");
        /*模态窗口的HTML*/
        var modalDiv = '<div class="modal fade" id="myModal" tabindex="-1" role="dialog" data-backdrop="static">';
        modalDiv += '    <div class="modal-dialog" role="document" style="width:'+ modalWidth +'">';
        modalDiv += '        <div class="modal-content">';
        modalDiv += '        </div>';
        modalDiv += '    </div>';
        modalDiv += '</div>';
        /*删除之前存在的模态窗口*/
        if ($("#myModal").size() > 0){
            $("#myModal").remove();
        }
        /*插入新模态窗口*/
        $(document.body).append(modalDiv);
        /*初始化模态窗口*/
        $("#myModal").modal({
            show:false,
            remote:href
        });
        /*显示模态窗口*/
        $("#myModal").modal('show');
    });
});
```

**2\. 在页面中调用模态框的格式：**

```
<a class="myModal" data-href="form" data-width="80%">BUTTON</a>
```

- class：引入弹出框插件，对应JS代码中的`$(".myModal")`类选择器。
- data-href：弹出框页面的链接。
- data-width：弹出框的宽度。

**3\. 弹出框需要显示的页面代码格式规范示例：**

```
<!--弹出页面头部-->
<div class="modal-header">
    <button type="button" class="close" data-dismiss="modal" aria-hidden="true">×</button>
    <h4 class="modal-title" id="myModalLabel">$title$</h4>
</div>
<!--页面主体内容-->
<div class="modal-body"></div>
<!--页脚-->
<div class="modal-footer text-center">
    <button type="submit" class="btn btn-primary">保　存</button>
    <button type="button" class="btn btn-default" data-dismiss="modal">取　消</button>
</div>
```
