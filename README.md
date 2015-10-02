protobuf是google的一个序列化框架，类似XML，JSON，其特点是基于二进制，比XML表示同样一段内容要短小得多，还可以定义一些可选字段，广泛用于服务端与客户端通信。文章将着重介绍在erlang中如何使用protobuf。

首先google没有提供对erlang语言的直接支持，所以这里使用到的第三方的protobuf库（erlang_protobuffs）

定义一个protobuf结构，保存为test.proto，如下：
[plain] view plaincopy在CODE上查看代码片派生到我的代码片
message Person {  
  required int32 age = 1;  
  required string name = 2;  
}  
  
message Family {  
  repeated Person person = 1;  
}  
编译这个protobuf结构，生成相应的erlang代码：
[plain] view plaincopy在CODE上查看代码片派生到我的代码片
% 生成相应的erl和hrl文件  
protobuffs_compile:scan_file_src("test.proto").  
  
% 生成相应的beam和hrl文件  
protobuffs_compile:scan_file("test.proto").  
下面我们以例子简单说明如何使用：
[plain] view plaincopy在CODE上查看代码片派生到我的代码片
-module(test).  
  
-compile([export_all]).  
  
-include("test_pb.hrl").  
  
encode() ->  
    Person = #person{age=25, name="John"},  
    test_pb:encode_person(Person).  
  
decode() ->  
    Data = encode(),  
    test_pb:decode_person(Data).  
  
encode_repeat() ->  
    RepeatData =  
    [  
        #person{age=25, name="John"},  
        #person{age=23, name="Lucy"},  
        #person{age=2, name="Tony"}  
    ],  
    Family = #family{person=RepeatData},  
    test_pb:encode_family(Family).  
      
decode_repeat() ->  
    Data = encode_repeat(),  
    test_pb:decode_family(Data).  
运行代码，如下：
[plain] view plaincopy在CODE上查看代码片派生到我的代码片
6> c(test).  
{ok,test}  
  
7> test:encode().  
<<8,25,18,4,74,111,104,110>>  
  
8> test:decode().  
{person,25,"John"}  
  
9> test:encode_repeat().  
<<10,8,8,25,18,4,74,111,104,110,10,8,8,23,18,4,76,117,99,  
  121,10,8,8,2,18,4,84,111,110,...>>  
  
10> test:decode_repeat().  
{family,[{person,25,"John"},  
         {person,23,"Lucy"},  
         {person,2,"Tony"}]}  
文章完整例子下载：http://download.csdn.net/detail/cwqcwk1/7087293
参考:
https://github.com/ngerakines/erlang_protobuffs/tree/master
其他语言的protobuff
https://github.com/google/protobuf/wiki/Third-Party-Add-ons