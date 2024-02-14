---
title: "2024 02 23 List Directories Recursively in Golang"
date: 2024-02-14T18:08:44+08:00
draft: true
comment: true
description: ""
---

## 使用 `filepath.Walk` 函数

对于需要递归遍历目录及其子目录的场景，Go 中提供了一个专门的函数简化这个过程，它就是 `filepath.Walk`。

这个函数提供了一个便捷的解决方案。它可以遍历所有子目录，允许在遍历过程中处理每个文件。尽管这提供了极高的灵活性，对于大型或深层次的目录结构，性能可能会是一个考虑因素。

```go
func main() {
    err := filepath.Walk(".", func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        fmt.Println(path)
        return nil
    })
    if err != nil {
        fmt.Printf("error walking the path %v: %v\n", ".", err)
    }
}
```
