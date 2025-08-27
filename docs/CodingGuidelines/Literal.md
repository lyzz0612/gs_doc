---
sidebar_label: 'GS中构造数据时的书写格式'
sidebar_position: 4
---

# GS中构造数据时的书写格式

## 简介

这里主要是展示构造几种数据类型时书写格式。

## String

- **构造空字符串**
    ```c++
    string str1 = "";
    ```

- **构造非空字符串**
    ```c++
    string str1 = "hello";
    ```

- **构造非空字符串段落**
    ```c++
    string str1 = """p
        hello world
        hello everyone
    """p;
    ```

## Map

- **构造空 map**
    ```c++
    map dict = {};
    ```

- **构造非空 map**
    ```c++
    map dict = {
        "key1" : value1,
        "key2" : value2
    };
    ```

    或者

    ```c++
    map dict = { "key1" : value1, "key2" : value };
    ```

## Array

- **构造空 array**
    ```c++
    array list = [];
    ```

- **构造非空 array**
    ```c++
    array list = [
        value1,
        value2
    ];
    ```

    或者

    ```c++
    array list = [ value1, value2 ];
    ```

## Function

- **构造函数实例**
    ```c++
    function func = (: write, 1, 2, 3 :);
    ```

- **构造匿名函数**
    ```c++
    function func = () { printf("hello\n"); };
    ```

    或者

    ```c++
    function func = () {
        printf("hello\n");
    };
    ```

- **引用捕获**
    ```c++
    int a = 10;
    int b = 20;
    int c = 30;

    function func = [&a, &b, &c] () { printf("a=%d\n", a); };
    ```

    或者

    ```c++
    int a = 10;
    int b = 20;
    int c = 30;

    function func = [&a, &b, &c] () {
        printf("a=%d\n", a);
    };
    ```
```