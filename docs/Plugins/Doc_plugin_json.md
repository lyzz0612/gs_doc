---
sidebar_label: 'json'
sidebar_position: 3
---


# json - json文件读写插件

## 概述
提供:

* json字符串的解析接口
* 将map或array转化成string的save接口

##  传统Json语法：

Json的值可以是:

1. 数字（整形或者浮点数）  
2. 字符串（在双引号中）  
3. 逻辑值（true或者false）  
4. 数组（在中括号中）  
5. 对象（在大括号中）  
6. 空值（null)  

Json对象:

1. Json对象在大括号中，并且由多个key:value对组成；    
2. key必须是字符串，value可以是合法的Json数据类型（数字，字符串，逻辑值，数组，对象，空值）；  
3. key和value使用冒号（:）分隔；  
4. 每个key:value对使用逗号（,）分隔。  

## GS对传统Json的改进

GS实现的Json与标准 Json 相比，有如下改进:

1. Map 类型可以以 int 作为 key；    
2. 增加 @"XXXX" 格式数据，解析到 XXXX 后将交由脚本层自行解析；    
   通过该配置可完成常数配置等需求；    
3. 增加行注释 '//' 和段注释 '/\* \*/' 的支持。      

## 如何使用Json：

### parse
```cpp
/**
 * @param str json格式的字符串
 * @param func 自定义替换函数：将文本中`@"<custom>"`替换为期望的合法值
 */
mixed json.parse(string str, function? func = nil);    // 解析Json格式的文本，一般返回map
```

举个例子：
```cpp
string json_text = """P
{
   "members" : [
        { "name" : "hai0", "age" : 1       },
        { "name" : "hai1", "age" : 0x7F    },
        { "name" : "hai2", "age" : @"DEF"  },
//      { "name" : "hai3", "age" : @"DEF"  },   // 行注释
/*                                              // 段注释
        { "name" : "hai4", "age" : @"DEF"  },
*/
    ],
    "test" : @custom,
    "real" : 5.0,
    "inited" : true,
}
"""P;

private mixed custom_parse(string str)
{
    if (str == "custom")
        return "OK";

    if (str == "DEF")
        return 10000;

    return "FAILED";
}

// Test for efun of math operations
public void test_json()
{
    map m = json.parse(json_text, (: custom_parse :));
    array members = m["members"];       // 取json_text的members
    write(members[0]["name"], "\n");    // 输出 hai0
    write(members.length(), "\n");       // 输出 3

    write(members[0]["age"], "\n");     // 输出 1
    write(members[1]["age"], "\n");     // 输出 127
    write(members[2]["age"], "\n");     // 输出 10000

    write(m["test"], "\n");             // 输出 OK
    write(m["real"], "\n");             // 输出 5.0
    write(m["inited"], "\n");           // 输出 true
}

test_json();
```

### save

```cpp
/**
 * @param val
 * @param func 自定义序列化函数
 */
mixed json.save(mixed val, function? func = nil);    // 将map或array转化为Json格式的文本
```

举个例子：
```cpp
map data = {
    "members": [
        { "name" : "hai0", "age" : 1       },
        { "name" : "hai1", "age" : 0x7F    },
        { "name" : "hai2", "age" : 23      },
        { "name" : "hai3", "age" : 31      },
        { "name" : "hai4", "age" : 56      },
    ],
    "test" : nil,
    "real" : 5.0,
    "inited" : true,
};


// 自定义序列化函数必须返回string或nil，为nil时忽略该函数，走常规转化流程
mixed custom_serializer(mixed val)
{
    string ret = "";
    array members = val["members"];
    for (mixed member: members)
    {
        string name = member["name"];
        int age = member["age"];
        if (age < 100)
        {
            ret += name + " ";
        }
    }
    return nil;
}

string json_save(mixed val)
{
    return json.save(val);
}

string json_custom_save(mixed val, function func)
{
    return json.save(val, func);
}

void test()
{

    string result = json_save(data);
    write("Default json save result:\n");
    write(result, "\n");

    result = json_custom_save(data, (: custom_serializer :));
    write("Custom json save result:\n");
    write(result);
}

test();

```
