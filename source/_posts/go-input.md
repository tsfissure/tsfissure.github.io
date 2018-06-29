---
title: go_input
date: 2018-06-29 15:03:01
tags: go
---

## go入门之像cpp一样输入效果

在hackerrank上面练习的时候，被读到eof难住了，故总结一下;
学习一下fmt和bufio.

<!-- more-->

### 读一整行
cpp:
```cpp
//读一行
gets(s);
读到eof
for(; gets(s);){
    // do something
}

```
go:
```go
//利用scanner读一行,返回的是字符串
func main() {
    scanner := bufio.NewScanner(os.Stdin)
    //读一行
    scanner.Scan()
    fmt.Println(scanner.Text())
    //读到eof
    for scanner.Scan() {
        fmt.Println(scanner.Text())
    }
}

///////////////

//利用Reader返回的是ascii码集合,需要转和判断error,比较麻烦
func main() {
    reader := bufio.NewReader(os.Stdin)
    //读一行
    a, b, err := reader.ReadLine()
    if err == nil {
        fmt.Println(string(a), b)
    }
    //读到eof
    for {
        a, b, err := reader.ReadLine()
        if err != nil {
            break
        }
        fmt.Println(string(a), b)
    }
}


```

### 自定义读到EOF

cpp:
```cpp
for(; ~scanf("xxx");){
    // do something
}
```
go:
```go
func main(){
    var a string
    var b int
    for {
        n, err := fmt.Scan(&a, &b)
        if err != nil || 2 != n {
            break
        }
        fmt.Println(a, b)
    }

```
