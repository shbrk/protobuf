# Go support for Protocol Buffers - Google's data interchange format

[![Build Status](https://travis-ci.org/golang/protobuf.svg?branch=master)](https://travis-ci.org/golang/protobuf)
[![GoDoc](https://godoc.org/github.com/golang/protobuf?status.svg)](https://godoc.org/github.com/golang/protobuf)

Google's data interchange format.
Copyright 2010 The Go Authors.
https://github.com/golang/protobuf

This package and the code it generates requires at least Go 1.6.

This software implements Go bindings for protocol buffers.  For
information about protocol buffers themselves, see
	https://developers.google.com/protocol-buffers/

## Example ##

```c++
syntax = "proto3";

package test;

message Foo {
    string a = 1;
    int32 b = 2;
}


// [1,2,3] 代表1,2,3三个字段支持固定部分序列化
message TestMsg {
    string id = 1;
    string name = 2;
    string addr = 3;
    Foo    foo = 4;
    string test = 1000;
}

```

## 测试代码 ##

```go
package main

import (
	"github.com/shbrk/protobuf/proto"
	"HelloWorld/test"
	"fmt"
		)

func main() {
	var msg = &test.TestMsg{
		Id: "111",
		Name:"China",
		Addr:"Asia",
		Test:"abc",
		Foo:&test.Foo{
			A:"abc",
			B:1,
		},
	}

	var retAll, err = proto.Marshal(msg)
	if err == nil {
		fmt.Println("all fields:")
		fmt.Printf("%b\n",retAll)
	}

	retPartial,err := proto.MarshalPartial(msg)
	if err == nil {
		fmt.Println("partial fields 1,2,3:")
		fmt.Printf("%b\n",retPartial)
	}

	// Id被标记为dirty
	msg.SetId("222")
	retDirty,err :=  proto.MarshalDirty(msg)
	if err == nil {
		fmt.Println("dirty fields id:")
		fmt.Printf("%b\n",retDirty)
	}


	// 注意Id字段不会被标记dirty，下面消息为空
	msg.Id = "xxx"
	retDirtyError,err := proto.MarshalDirty(msg)
	if err == nil {
		fmt.Println("dirty fields empty:")
		fmt.Printf("%b\n",retDirtyError)
	}


	// 注意Foo字段不会被标记dirty
	msg.GetFoo().A = "xxx"
	retDirtyError,err = proto.MarshalDirty(msg)
	if err == nil {
		fmt.Println("dirty fields empty:")
		fmt.Printf("%b\n",retDirtyError)
	}

	// 注意Foo字段不会被标记dirty
	msg.GetFoo().SetA( "xxx")
	retDirtyError,err = proto.MarshalDirty(msg)
	if err == nil {
		fmt.Println("dirty fields empty:")
		fmt.Printf("%b\n",retDirtyError)
	}

	// Foo字段会被标记dirty
	var foo = msg.GetFoo()
	foo.SetA( "xxx") // foo.A = xxx 也可以
	msg.SetFoo(foo) // 关键操作

	retDirtyRight,err := proto.MarshalDirty(msg)
	if err == nil {
		fmt.Println("dirty fields foo:")
		fmt.Printf("%b\n",retDirtyRight)
	}

	//

	var msgback = &test.TestMsg{
	}
	proto.Unmarshal(retAll,msgback)//正常的反序列化
	proto.UnmarshalMerge(retDirtyRight,msgback)//合并方式反序列化

}

```

## 测试结果 ##
```txt
all fields:
[1010 11 110001 110001 110001 10010 101 1000011 1101000 1101001 1101110 1100001 11010 100 1000001 1110011 1101001 1100001 100010 111 1010 11 1100001 1100010 1100011 10000 1 11000010 111110 11 1100001 1100010 1100011]
partial fields 1,2,3:
[1010 11 110001 110001 110001 10010 101 1000011 1101000 1101001 1101110 1100001 11010 100 1000001 1110011 1101001 1100001]
dirty fields id:
[1010 11 110010 110010 110010]
dirty fields empty:
[]
dirty fields empty:
[]
dirty fields empty:
[]
dirty fields foo:
[100010 111 1010 11 1111000 1111000 1111000 10000 1]
```

## 注意 ##
注意测试代码中 关于字段是结构体的操作，只有使用set方法才会记录dirty

