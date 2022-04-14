# http-logger 和grpc-transcode  同时使用BUG

#### 介绍
apisix-2.13.0 http-logger 开启 include_resp_body = true 记录返回body 
同时
使用 grpc-transcode 
接口返回数据错错误 表现为返回 proto 定义的默认值

#### 测试代码
- 测试代码为dubbogo3(golang)需求环境 apisix 2.13.0+nacos
- 配置有docker 可直接启动 熟悉go 也可直接打包成执行文件




#### 重现过程

1.  开启插件 http-logger 开启 include_resp_body = true 记录返回body
2.  编写一个grpc 服务通过 grpc-transcode解析
3.  请求后返回默认值
4.  插件 http-logger 设置 include_resp_body = false  请求返回正常
### 重现过程
#### proto
put /apisix/admin/proto/4

    {
    "content" : "syntax = \"proto3\";
    package helloworld;

    option go_package = \"./;helloworld\";

    // The greeting service definition.
    service Greeter {
      // Sends a greeting
      rpc SayHello (HelloRequest) returns (User) {}
      // Sends a greeting via stream
      rpc SayHelloStream (stream HelloRequest) returns (stream User) {}
    }

    // The request message containing the user's name.
    message HelloRequest {
      string name = 1;
    }

    // The response message containing the greetings
    message User {
      string name = 1;
      string id = 2;
      int32 age = 3;
     }"
    }

#### routes

    {
      "uri": "/aixichen/helloworld/sayhello",
      "name": "aixichen_helloworld_sayhello",
      "methods": [
       "GET",
       "POST",
    "PUT",
    "DELETE",
    "PATCH",
    "HEAD",
    "OPTIONS",
    "CONNECT",
    "TRACE"
      ],
    "plugins": {
    "grpc-transcode": {
      "method": "SayHello",
      "proto_id": "4",
      "service": "helloworld.Greeter"
    }
    },
     "upstream": {
    "type": "roundrobin",
    "scheme": "grpc",
    "discovery_type": "nacos",
    "pass_host": "pass",
    "service_name": "providers:helloworld.Greeter::"
     },
      "status":1
    }


#### 请求 http://XXXX/aixichen/helloworld/sayhello?name=小明
#### 结果
    {
        "age":0,
        "name":"",
        "id":""
    }
