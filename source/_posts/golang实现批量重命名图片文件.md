---
title: Golang实现批量重命名图片文件
tags:
  - golang
id: '233'
categories:
  - - Golang
date: 2019-06-05 12:00:17
---

因为自己喜欢设置win10壁纸自动切换，经常会收集很多好看的图片作为壁纸。 作为一个有轻度洁癖的人，一般都是用工具将图片重命名为数字递增的形式，之前一致用的photoscape，后来又用了Adobe Air下的一个Resize工具。 渐渐的觉得麻烦了，还不如自己写一个。记得Golang可以生成exe程序，而且不依赖运行环境，所以就干脆自己写一个。 注意，我设置的是图片重命名规则是1，2，3，4，5这种累加的形式。而且图片格式只设置了jpg，png两种。 这是我的第一个程序源代码：

```
package main

import (
    "fmt"
    "io/ioutil"
    "os"
    "strings"
    "strconv"
    "reflect"
    "crypto/md5"
    "encoding/hex"
)

func main() {
    dir := "./"
    renLogFileName := ".renlog"
    files, err := ioutil.ReadDir(dir)
    if err != nil {
        fmt.Printf("Failed to read dir %s, the error is %v\n", dir, err)
        return
    }
    renameLog := ""
    num := 1;
    for _, f := range files {
        fileName := f.Name()
        if fileName == renLogFileName {
            continue
        }
        // 代表取文件名第一个字符
        if fileName[:1] == "." {
            fmt.Printf("Skip hidden file %s\n", fileName)
            continue
        }
        var imageExtensions = []string{".jpg",".png"}
        // 只修改当前文件目录下文件
        if !f.IsDir() {
            dotIndex := strings.LastIndex(fileName, ".")
            //判断文件是一个合法的常规文件
            if dotIndex != -1 && dotIndex != 0 {
                extensionName := fileName[dotIndex:]
                ok , _ := in_array(extensionName,imageExtensions)
                if !ok{
                    continue
                }
                fmt.Printf("file: %s\n",fileName)
                // go里面没有do while循环体，只能用这种写法
                // int转string
                newFileName := strconv.Itoa(num)
                // 注意：++ 和 --只能作为语句使用，不能作为表达式
                num++
                newFileName += extensionName
                // 如果新文件名被占用，就把它先修改成随机的，这种逻辑还是最省事了
                fmt.Printf("%s 文件即将被修改成 %s", fileName, newFileName)
                if file_exists(newFileName) {
                    // 如果当前文件和新文件同名，直接跳过
                    if fileName == newFileName{
                        fmt.Printf("，因为同名，直接跳过 \n")
                        continue
                    }
                    fmt.Printf("，%s 文件名已经被占用\n", newFileName)
                    tmpFileName := GetMD5Hash(fileName)
                    tmpFileName += extensionName
                    os.Rename(newFileName, tmpFileName)
                    fmt.Printf("\n把 %s 临时修改成 %s\n", fileName, tmpFileName)
                }
                fmt.Printf("，转化 %s 为 %s\n", fileName, newFileName)
                err = os.Rename(fileName, newFileName)
                if err != nil {
                    fmt.Printf("Failed to rename file %s to %s, the error is %v\n", fileName, newFileName, err)
                    continue
                }
                renameLog += fmt.Sprintf("%s\t%s\n", fileName, newFileName)
            }
        }
    }
    fmt.Printf(renameLog)
    ioutil.WriteFile(renLogFileName, []byte(renameLog), 0666)
}

func in_array(val interface{}, array interface{}) (exists bool, index int) {
    exists = false
    index = -1

    switch reflect.TypeOf(array).Kind() {
    case reflect.Slice:
        s := reflect.ValueOf(array)

        for i := 0; i < s.Len(); i++ {
            if reflect.DeepEqual(val, s.Index(i).Interface()) == true {
                index = i
                exists = true
                return
            }
        }
    }

    return
}
// 判断所给路径文件/文件夹是否存在
func file_exists(path string) bool {
    _, err := os.Stat(path)    //os.Stat获取文件信息
    if err != nil {
        if os.IsExist(err) {
            return true
        }
        return false
    }
    return true
}
func GetMD5Hash(text string) string {
    return GetByteMD5Hash([]byte(text))
}
func GetByteMD5Hash(content []byte) string {
    hasher := md5.New()
    hasher.Write(content)
    return hex.EncodeToString(hasher.Sum(nil))
}
```

这个源代码，后来我发现会存在一些bug。比如2.jpg被重命名为1.jpg的时候，发现1.jpg已经被占用了，这时候将1.jpg重命名为xxxx.jpg，再将2.jpg重命名为1.jpg。本来我以为xxxx.jpg在之后文件遍历中会被get到，但是实际上并没有。反而1.jpg会在后续的文件遍历中读取到，又被重命名为了某个数字的jpg，造成的最终结果就是，没有1.jpg，但是多了一个xxxx.jpg。这显然不是我想要的结果。 我终于明白为什么PhotoScape重命名图片，是生成在一个output子文件夹里了，就是为了避开一些文件命名冲突问题。 但是我就喜欢做点有挑战性的事情，不希望每次执行完程序还要复制粘贴一遍，直接在图片文件里执行exe就能达到我满意的结果。 有句话说的好，发明家都是懒人。 准备重新修改逻辑，命名冲突用临时文件名避开的方法不再适用了。 新的逻辑：2.jpg被重命名1.jpg的时候，判断1.jpg是否已经被占用，如果占用，数字累加，判断2.jpg，如果继续被占用，继续判断3.jpg。直到拿到一个没有被占用的文件名。同时，已经被占用的文件名，需要做标记，否则1.jpg在后面的循环会被重命名成一个非常大的数字，这样1.jpg就不存在了，取而代之的是一个101.jpg都有可能。 新的源代码如下，测试没有问题，大家有需要的，可以编译拿去用，欢迎留言，一起学习。

```
package main

import (
    "fmt"
    "io/ioutil"
    "os"
    "strings"
    "strconv"
    "reflect"
    "crypto/md5"
    "encoding/hex"
)

func main() {
    dir := "./"
    renLogFileName := ".renlog"
    files, err := ioutil.ReadDir(dir)
    if err != nil {
        fmt.Printf("Failed to read dir %s, the error is %v\n", dir, err)
        return
    }
    renameLog := ""
    ignoreFile := make(map[string]int)
    num := 0;
    L1:
    for _, f := range files {
        fileName := f.Name()
        if fileName == renLogFileName {
            continue
        }
        // 代表取文件名第一个字符
        if fileName[:1] == "." {
            fmt.Printf("Skip hidden file %s\n", fileName)
            continue
        }
        var imageExtensions = []string{".jpg",".png"}
        // 只修改当前文件目录下文件
        if !f.IsDir() {
            dotIndex := strings.LastIndex(fileName, ".")
            //判断文件是一个合法的常规文件
            if dotIndex != -1 && dotIndex != 0 {
                extensionName := fileName[dotIndex:]
                ok , _ := in_array(extensionName,imageExtensions)
                if !ok{
                    continue
                }
                fmt.Printf("开始处理文件: %s\n",fileName)
                if _, ok := ignoreFile[fileName]; ok {
                    fmt.Printf("%s 文件不做改变，直接跳过\n",fileName)
                    continue L1
                }
                // go里面没有do while循环体，只能用这种写法
                var newFileName string
                L2:
                for{
                    // 注意：++ 和 --只能作为语句使用，不能作为表达式
                    num++
                    // int转string
                    newFileName = strconv.Itoa(num)
                    newFileName += extensionName
                    if newFileName==fileName{
                        ignoreFile[fileName]=1
                        continue L1
                    }
                    if !file_exists(newFileName) {
                        break L2
                    }else{
                        ignoreFile[newFileName]=1
                        fmt.Printf("%s 已经被占用\n", newFileName)
                    }
                }
                fmt.Printf("%s 文件即将被修改成 %s", fileName, newFileName)
                
                err = os.Rename(fileName, newFileName)
                fmt.Printf("，转化 %s 为 %s\n", fileName, newFileName)
                if err != nil {
                    fmt.Printf("Failed to rename file %s to %s, the error is %v\n", fileName, newFileName, err)
                    continue
                }
                renameLog += fmt.Sprintf("%s\t%s\n", fileName, newFileName)
            }
        }
    }
    fmt.Printf(renameLog)
    ioutil.WriteFile(renLogFileName, []byte(renameLog), 0666)
}

func in_array(val interface{}, array interface{}) (exists bool, index int) {
    exists = false
    index = -1

    switch reflect.TypeOf(array).Kind() {
    case reflect.Slice:
        s := reflect.ValueOf(array)

        for i := 0; i < s.Len(); i++ {
            if reflect.DeepEqual(val, s.Index(i).Interface()) == true {
                index = i
                exists = true
                return
            }
        }
    }

    return
}
// 判断所给路径文件/文件夹是否存在
func file_exists(path string) bool {
    _, err := os.Stat(path)    //os.Stat获取文件信息
    if err != nil {
        if os.IsExist(err) {
            return true
        }
        return false
    }
    return true
}
func GetMD5Hash(text string) string {
    return GetByteMD5Hash([]byte(text))
}
func GetByteMD5Hash(content []byte) string {
    hasher := md5.New()
    hasher.Write(content)
    return hex.EncodeToString(hasher.Sum(nil))
}
```