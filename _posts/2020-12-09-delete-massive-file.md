---
layout: post
title: "删除大量小文件"
subtitle: ''
tags:
  - Golang 
  - 删除文件
  - 大量小文件
---

机器报警，排查发现是iNode满了，原来是一php服务创建了大量的文件用于存放session。对于这种有大量文件的场景，直接运行`rm -rf *`基本上会直接把CPU,内存打满，还不能直接删除

```
package main

import (
	"log"
	"os"
	"path/filepath"
)

func MyWalkFunc(path string, info os.FileInfo, err error) error {
	log.Println(path)

	if info.IsDir(){
		return nil
	}

	os.Remove(path)

	return nil
}

func main(){
	log.Println("start")

	p := "/var/lib/php/sessions"
	filepath.Walk(p,MyWalkFunc)

	log.Println("end")
}
```