# Spring MVC Post请求返回403错误，Get请求却正常？

困惑：很奇怪，明明在方法上面配置了RequestMethod.POST，POST表单提交却返回403状态码，可是使用GET方式却没问题啊！！！

@RequestMapping(value="***", method = { RequestMethod.POST })
public ModelAndView edit() {
   ModelAndView model = new ModelAndView("**");

   return model;
}

难道这配置会有问题？！！！！

原因分析：如果是使用的是 Java 代码配置 Spring Security，那么 CSRF 保护默认是开启的，那么在 POST 方式提交表单的时候就必须验证 Token，如果没有，那么自然也就是 403 没权限了。

解决方式：这里主要有两种解决方式

1) 关闭 CSRF 保护：

@Override
protected void configure (HttpSecurity http) throws Exception {
    http.csrf().disable();
}

2) 添加 CSRF
如果是表单提交那么，可以添加一个隐藏输入框：

<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
1
如果是 Ajax 提交，那么可以在提交时将 Token 一起提交：

$.ajax({
    type: "POST", 
    url: "***", 
    data: {content:content, "${_csrf.parameterName}":"${_csrf.token}"}, 
        error: function() { 
    },
           success:function(data) {
    }
});

如果不想在每个 Ajax提交时都带上这个，那么也可以这样，首先在 Header添加 meta：

<html>
  <head>
    <meta name="_csrf" content="${_csrf.token}"/>
    <!-- default header name is X-CSRF-TOKEN -->
    <meta name="_csrf_header" content="${_csrf.headerName}"/>
    <!-- ... -->
  </head>
  <!-- ... -->

然后再每次 Ajax 提交前，插入 Token：

$(function () {
  var token = $("meta[name='_csrf']").attr("content");
  var header = $("meta[name='_csrf_header']").attr("content");
  $(document).ajaxSend(function(e, xhr, options) {
    xhr.setRequestHeader(header, token);
  });
});

这样在 Ajax 提交时会自动将 token一并提交（在 header 里，而不是表单数据里了），那么也就可以通过验证了。
