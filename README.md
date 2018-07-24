# 一种简洁的模型定义语法（.mon）

> 注：本文还只是一种设想，并没有开发实际可用的工具

## 前言

在日常的需求开发过程中，我们经常需要约定接口的返回模型，同时各端需要定义与之对应的本地代码的模型。这里有两个问题：

1. 如何简单且无歧义地表示接口数据？
2. 如何各端（**iOS**，**Android**，**Web**）统一模型定义？

对于问题一，我们一般用一个示例 **JSON** 来表示。如，对于请求用户详情接口 `api/user/:id`，我们有：

```json
{
    "code": 0,
    "result": {
        "id": "123456",
        "nickname": "testName",
        "avatar": "https://domain.com/abc.jpg",
        "lastLogin": 1532341993000,
        "lastIP": "192.168.0.10",
        "status": 2,
        "level": 5,
        ...
    }
}
```

大部分情况下这样协作是没有问题的，但也存在一些缺陷，比如，无法表示部分字段是可选的（用户没有传头像时，`avatar` 字段有可能缺失），等等。

对于问题二，我们只能每个端自己去定义了。

```swift
// Swift
class User : Codable {
    open var id: String = ""
    open var nickname: String = ""
    open var avatar: URL?
    open var lastLogin: Date?
    open var lastIP: String?
    open var status: Int = 0
    open var level: Int = 0
}
```

```objc
// Objective-C
@interface XXUser : NSObject <YYModel, NSCoding>
@property (nonatomic, strong) NSString *id;
@property (nonatomic, strong) NSString *nickname;
@property (nonatomic, strong, nullable) NSURL *avatar;
@property (nonatomic, strong, nullable) NSDate *lastLogin;
@property (nonatomic, strong, nullable) NSString *lastIP;
@property (nonatomic, assign) NSInteger status;
@property (nonatomic, assign) NSInteger level;
@end
```

```java
// Android
public class User implements Serializable {
    public String id;
    public String nickname;
    public @Nullable String avatar;
    public @Nullable Date lastLogin;
    public @Nullable String lastIP;
    public int status;
    public int level;
}
```

如果存在一种简单的语法，能优雅地表示服务端的返回结构，同时还能一键生成三端代码及 **Mock** 数据岂不是可以省去很多时间？

## 语法

为了减少学习成本，语法的设计上很大程度上参考了已有的 **DSL** 语言，如 **CSS**，**JSON** 等。

### 匿名模型

与 **JSON** 一样，我们用 `{}` 表示简单的匿名模型。在没有指明成员的情况下，与[内置模型](#内置模型) **Any** 语义一样。

### 命名模型

命名模型是一个大写字母开头的英文名词（当然也可以带定语），如 **User**，**Order**，**Product**。模型的成员是[内置模型](#内置模型)，也可以是匿名模型，或另一个命名模型

#### 内置模型

- **Any**，未定类型，可以为任何类型
- **Int**，一个有符号整数（不区分 `int` 和 `long`）
- **UInt**，无符号整数（不区分 `unsigned int` 和 `unsigned long`）
- **Float**，一个浮点数（不区分 `float` 和 `double`）
- **Bool**，布尔值
- **String**，一个无规则的字符串（可能为空串）
- **Url**，一个符合 url 定义的有规则字符串
- **ISODate**，一个符合 [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 定义的有规则字符串
- **Timestamp**，**Unix** 时间戳，一种特殊的 **UInt**

#### 数组

与 **JSON** 一样，我们用 `[]` 表示数组，`[Model]` 表示成员类型为 **Model** 的数组。

#### 注释

与大部分语言一样，使用双斜杠注释 `//`：

```
Model {		// This is a comment
    
}
```

#### 可选

我们用 `?` 问题表示一个字段是可选的，在 **Objective-C** 中表现为 `nullable`，在 **Swift** 等语言中与它本身的语法一致。例如：

```objc
@interface Book : NSObject
@property (nonatomic, strong, nullable) NSString *vendorName;
@end
```

```swift
class Book {
	open var vendorName : String?
}
```

#### 自定义模型

自定义一个模型，语法类似 **CSS**，一个模型名，加大括号定义体，定义体中由字段名、字段类型键值对构成。

* 普通模型

```
GeoLocation {
    latitude: Float,
    longitude: Float
}
```

```
User {
	id: String,
    nickname: String,
    avatar: Url?,
    location: GeoLocation?,
    birthday: ISODate?
}
```

​		字符串类型可以省略，为缺省类型：

```
User {
    firstName: String?,
    middleName: String?,
    lastName: String?
}
```

​		可以缩写为：

```
User {
    firstName?,
    middleName?,
    lastName?
}
```

* 嵌套模型

```
Article {
    coverImages: [{url: Url, caption: String?}]?,
    authors: [User],
    publishDate: ISODate?,
    publisher: User?
}
```

​		嵌套匿名模型

```
Post {
    id: String,
    title: String,
    author: {
        id: String,
        nickname: String,
        avatar: Url?
    },
    createdDate: ISODate
}
```

### 继承

```
SimpleUser {
    id: String,
    nickname: String,
    avatar: Url?
}

FullUser : SimpleUser {
    birthday: ISODate?,
    introduction: String?,
    tags: [Tag]?
}
```



### 自定义规则字符串

```
Cellphone String/1\d{10}/   // 中国手机号码是 1 开头的 11 位数

User {
    nickname: String,
    avatar: Url?,
    phone: Cellphone?
}
```

### 枚举（命名整型）

```
UserType Int(UserType1=1, UserType2=2)

User {
    nickname: String,
    avatar: Url?,
    type: UserType
}
```



