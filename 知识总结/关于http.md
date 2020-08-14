## content-type类型中application/json和application/x-www-form-urlencoded区别

参考文章：https://www.jianshu.com/p/53b5bd0f1d44

参考文章：https://blog.csdn.net/jackjia2015/article/details/94381171



首先get请求是不存在content-type这个字段的，而post请求是需要这个字段的



application/x-www-form-urlencoded是浏览器默认的content-type，是最常见的

application/json告诉服务器消息主体是序列化的json字符串，JSON 格式支持比键值对复杂得多的结构化数据，

**注意：当使用这种content-type的时候需要让后台对json进行解析，如果他们不去解析，那么可以将自己的content-type改成application/x-www-form-urlencoded**

multipart/form-data是用来提交文件上传，我们使用表单上传文件时，必须让 form 的 enctyped 等于这个值。
消息主体里按照字段个数又分为多个结构类似的部分，每部分都是以 –boundary 开始，紧接着内容描述信息，然后是回车，最后是字段具体内容（文本或二进制）。如果传输的是文件，还要包含文件名和文件类型信息。消息主体最后以 –boundary– 标示结束。

