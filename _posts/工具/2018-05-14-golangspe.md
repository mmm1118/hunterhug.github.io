---
layout: post
title: "Golang语言编程规范"
date: 2018-05-14
author: silly
categories: ["工具"]
desc: "写代码需要规范，编程规范很重要。"
tags: ["golang"]
permalink: "/tool/golangspe.html"
---

# 一.说明

1. 编程规范好，可避免语言陷阱，可有利团队协作，有利项目维护。
2. 正常的Go编程规范有两种：编译器强制的（必须的），gofmt格式化非强制的（非必须）。
3. Go宣告支持驼峰命名法，排斥下划线法。
4. 自定义原则：
 
 a.统一工作区间，避免目录及文件名随意
 
 b.规范变量/结构体/方法及接口名

 c.规范注释

 d.单元测试/程序效率等建议

两个等级： （S）建议，（M）必须。以下是细节。



# 二.代码组织结构

1. （M）一个目录只包含一个包，模块复杂拆分子模块/子目录
2. （S）内部项目GOPATH如果指向多个工作目录。公开项目为第一个工作区间（即go get默认下载到第一个目录）
3. （M）非测试文件(*_test.go)禁止使用，简化包
4. （M）禁止相对路径导入包
5. （S）建议goimports或者IDE管理import
6. （S）项目rep需要包含所有的代码，依赖库放在vendor下
7. （S）建议使用Golide,Godep管理第三方包

总而言之：组织结构需要简化，一目了然，允许多个工作区间，但是环境变量的第一个工作区间必须是公开的项目，最好使用一个工作区间，不要建立相对路径的包，最好用godep等依赖库控制工具来管理依赖库，每个项目所有的依赖库最好放在本目录vendor下。



# 三.代码风格

1. （M）提交代码时gofmt格式化代码，golint检查代码(使用IDE时默认这两个工具会自动用到)
2. （S）json字符串建议使用反单引号(`)
3. （M）文件名必须小写，允许下划线'_’，但头尾不能。避免与_test.go或者系统相关_386.go等冲突
4. （S）文件名以功能为指引，不需要再出现模块名
5. （M）目录名必须小写，允许中划线'-'，但头尾不能
6. （S）不建议目录名出现下划线'_'
7. （M）包名必须全部小写，无下划线，越短越好，尽量不要与标准库重名，禁止通过中划线连接多个单词
8. （S）包名尽量与目录名一致
9. （M） 函数名和结构体名必须为大小写驼峰模式，最好不带特殊字符如划线等
10. （S）函数名建议动词或者动宾结构单词，结构体建议名词或者动名词
11. （S）常量和枚举名，大小写驼峰法，不允许下划线，第三方包例外。
12. （M）函数参数首字母小写，不能有下划线，按大小驼峰法
13. （S）函数参数按紧密程度安排位置，同类型参数应该相邻
14. （S）参数不大于5个
15. （M）变量名不允许下划线，大小写驼峰法，局部变量首字母小写，全局变量首字母大写
16. （S）避免全局变量多使用，for循环可用单字母
17. （M）接口名大小写驼峰法，首字母大写，不能下划线，名词
18. （S）接口名'er'结尾
19. （M）复杂功能请多写注释备注，注释表达需清晰，不要啰嗦。注释标准暂时不强制，最好参考godoc，如包注释使用/**/,首字母大写，注释后空一行，函数注释写在函数上方等。

总而言之，文件名和目录名，包名都必须小写。数据类型变量和参数等定义最好使用驼峰大小写法，不要使用下划线或者中划线


# 四. 单元测试/程序效率

1. （S）建议少使用main方法测试，而是使用_test.go做测试
2. （M）与其他语言类似，避免多级if或者for嵌用，代码层次需简单，绕脑层次少
3. （M）避免傻逼命名，如IsTrue变量，if(!IsTrue).
4. （M）请熟悉Go语言各特征，避免低效用法。请至少阅览一遍：


```

yum -y install docker-io
# 拉镜像
docker pull hunterhug/gotourzh

# 前台运行
docker run -it -p 9999:9999 hunterhug/gotourzh

# 后台运行(类似nohup)
docker run -d -p 9999:9999 hunterhug/gotourzh
打开http://127.0.0.1:9999即可!

```

# 五. 特殊说明

```
package main

import "fmt"

func main() {
  buffedChan := make(chan int, 2)
  buffedChan <- 2
  buffedChan <- 3
  close(buffedChan) // 关闭后才能for打印出，否则死锁
  //close(buffedChan) // 不能重复关闭
  //buffedChan <- 4  // 关闭后就不能再送数据了，但是之前的数据还在
  for i := range buffedChan { // 必须关闭，否则死锁
    fmt.Println(i)
  }

  buffedChan1 := make(chan int, 2)
  buffedChan1 <- 2
  buffedChan1 <- 3
  j, ok := <-buffedChan1
  fmt.Println(j, ok)
  j, ok = <-buffedChan1
  fmt.Println(j, ok)

  close(buffedChan1)    // 关闭后才能，否则死锁
  j, ok = <-buffedChan1 // 如果未关闭，否则堵塞后死锁
  fmt.Println(j, ok)

  select {
  case j, ok := <-buffedChan1:
    fmt.Println("jjj", j, ok)
  default:
    fmt.Println("will not out")
  }
}

```




