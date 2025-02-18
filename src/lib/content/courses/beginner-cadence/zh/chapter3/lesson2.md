---
title: 在 Dictionary 和数组 Array 中的 Resource
lesson: 2
language: zh
excerpt: 在 Dictionary 和数组 Array 中的 Resource
---

# 第三章 第二天 - 在 Dictionary 和数组 Array 中的 Resource

在第二章中，我们学过 Dictionary 和数组 Array，今天我们来学习如果把 resource 应用到 Dictionary 和 Array 中区。我们单独使用他们的时候，也许还不难，但我们把他们弄到一起时，就会有点复杂了。

## 视频

你可以从 08：00 一直看到最后（我们昨天讨论了前半部分）: https://www.youtube.com/watch?v=SGa2mnDFafc

## 为什么是 Dictionaries & Arrays?

首先，我们为什么要讨论 Dictionary 中的 resource，而不是 structs 中的 resource 呢？因为，首先你要知道，_你不能在 struct 中存储 resource_。虽然 struct 是一个数据的容器，但我们不能把 resource 放在里面。

那我们能在哪里存储 resource 呢？

1. 在 dictionary 或者 array 中
2. 在另外一个 resource 中
3. 一个合约的状态变量
4. 在账户的存储空间中 (我们稍后会讨论)

就是这样，今天我们会讨论第一点。

## Arrays 数组中的 resource

通过示例还学习总是很好的，我们来打开一个 Flow playground，并在 Chapter 3 Lesson 1 中部署这个合约。

```cadence
pub contract Test {

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

}
```

到此为止，我们有了 1 个 `@Greeting`类型的 resource。现在我们试着创建一个状态变量，把一列 Greetings 存储在一个数组 array 里。

```cadence
pub contract Test {

    pub var arrayOfGreetings: @[Greeting]

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    init() {
        self.arrayOfGreetings <- []
    }

}
```

请注意这个 `arrayOfGreetings`: `@[Greeting]` 类型。我们昨天学到 resource 在开头必须有 `@` 符号。这在包含 resource 的数组中也同样适用，你需要在开头加入 `@` 来告知 Cadence 这是一个包含 resources 的数组。并且，这个 `@` 必须是在括号外的，不是里面。

`[@Greeting]` - 错误

`@[Greeting]` - 正确

同时请注意，在 `init` 函数中，我们是用 `<-` 运算符来初始化它的，而不是 `=`。再一次的，当我们处理 resource 的时候（不管是单纯的一个 resource，还是在 Dictionary 中，或者 array 中），我们必须使用 `<-`。

### 加入到数组中

好的，现在我们创建了我们的自己的 resource 数组。现在我们来看一下我们怎么才能把 resource 加到数组中。

_请注意：今天我们将会以变量的形式把 resource 传入到我们的函数中。也就是说我们不关心 resource 是如何被创建的，我们只是用一些例子来展示如何把 resource 加入到 array 和 Dictionary 中_

```cadence
pub contract Test {

    pub var arrayOfGreetings: @[Greeting]

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun addGreeting(greeting: @Greeting) {
        self.arrayOfGreetings.append(<- greeting)
    }

    init() {
        self.arrayOfGreetings <- []
    }

}
```

在这个例子中，我们添加了一个我 `addGreeting` 函数，把传入的 `@Greeting` 通过 `append` 函数添加到 array 中。很简单，对吧？ 这和我们平时如何加入 array 是一样的，只是我们是用的 `<-` 运算符把 resource “移动” 到 array 中。

### 从 Array 中删除

好的，我们知道了如何加入到数组中。我们来看一下如何从数组中删除 resource

```cadence
pub contract Test {

    pub var arrayOfGreetings: @[Greeting]

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun addGreeting(greeting: @Greeting) {
        self.arrayOfGreetings.append(<- greeting)
    }

    pub fun removeGreeting(index: Int): @Greeting {
        return <- self.arrayOfGreetings.remove(at: index)
    }

    init() {
        self.arrayOfGreetings <- []
    }

}
```

还是那么简单直接。在一个通常的数组中，我们会用 `remove` 函数来把某一个元素取出来。在处理 resource 时也同样适用，唯一的区别就是我们要用 `<-` 运算符来把 resource 从数组中取出。

## Dictionaries 中的 resource

Dictionaries 中的 resource 相对有一点复杂。原因之一是，如果你还记得在前面第二章第三天时我们提到，当时访问 Dictionary 中的值的时候，返回的一定是 optional 的。这使得存储和提取 resource 更复杂点。无论怎样，resource _通常也都是存储在 Dictionary 中_，因此学习如何处理这些是非常重要的。

我们用一个类似的合约作为例子：

```cadence
pub contract Test {

    pub var dictionaryOfGreetings: @{String: Greeting}

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    init() {
        self.dictionaryOfGreetings <- {}
    }

}
```

我们有一个 Dictionary 包含 `message` 与包含 message 的`@Greeting` resource 的映射. 请注意这个 dictionary 类型: `@{String: Greeting}`. `@` 是在花括号外.

### 添加到 Dictionary

把 resource 添加到 dictionary 有两种方式。我们来分别看一下

##### 1 - 最简单的, 但有些限制

将一个 resource 加到 dictionary 里最简单的办法就是用 “强制移动” 运算符 `<-!`，比如这样：

```cadence
pub contract Test {

    pub var dictionaryOfGreetings: @{String: Greeting}

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun addGreeting(greeting: @Greeting) {
        let key = greeting.message
        self.dictionaryOfGreetings[key] <-! greeting
    }

    init() {
        self.dictionaryOfGreetings <- {}
    }

}
```

在 `addGreeting` 函数中, 我们首先通过 `greeting` 里的 `message`来获得 `key` . 之后我们用“强制移动”把 `greeting` 加到 `dictionaryOfGreetings` dictionary 的 `key` 的位置.

强制移动运算符 `<-!` 的意思是：“如果要存放的位置有值，就报错并终止程序；否则的话，就把值放进去”

##### 2 - 复杂点，但能处理重复值的情况

第二种方式是两次移动语法，比如这样：

```cadence
pub contract Test {

    pub var dictionaryOfGreetings: @{String: Greeting}

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun addGreeting(greeting: @Greeting) {
        let key = greeting.message

        let oldGreeting <- self.dictionaryOfGreetings[key] <- greeting
        destroy oldGreeting
    }

    init() {
        self.dictionaryOfGreetings <- {}
    }

}
```

在这个例子中，你能看到这个比较奇怪的两次移动运算符。这个是干什么用的？我们来一步步分解来看：

1. 把 `key` 中的任何值取出来，移动到 `oldGreeting`
2. 现在我们知道 `key` 没有映射到任何东西, 移动 `greeting` 到这个位置
3. 销毁 `oldGreeting`

本质上来说，这个确实有点麻烦且看上去怪怪的，但 **这能让你处理如果原来已经有值的情况**。在上面的例子中，我们是简单的销毁了 resource，但如果你想的话，你可以做一些其他的操作。

### 从 Dictionary 中删除

下面是，我们如何从 dictionary 中删除一个 resource：

```cadence
pub contract Test {

    pub var dictionaryOfGreetings: @{String: Greeting}

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun addGreeting(greeting: @Greeting) {
        let key = greeting.message

        let oldGreeting <- self.dictionaryOfGreetings[key] <- greeting
        destroy oldGreeting
    }

    pub fun removeGreeting(key: String): @Greeting {
        let greeting <- self.dictionaryOfGreetings.remove(key: key) ?? panic("Could not find the greeting!")
        return <- greeting
    }

    init() {
        self.dictionaryOfGreetings <- {}
    }

}
```

还记得我们在“从 array 中删除值的”那一部分讲到的，需要我们做的只是调用那个 `remove` 函数。在 dictionary 中，访问一个元素的返回值是 optional 的，所以我们需要怎么着“打开”它。如果我们这样写。。。

```cadence
pub fun removeGreeting(key: String): @Greeting {
    let greeting <- self.dictionaryOfGreetings.remove(key: key)
    return <- greeting
}
```

我们会发现一个报错："Mismatched types. Expected `Test.Greeting`, got `Test.Greeting?`"。修正这个问题，我们可以要么使用 `panic` ，或者我们使用强制打开运算符 `！`，比如这样：

```cadence
pub fun removeGreeting(key: String): @Greeting {
    let greeting <- self.dictionaryOfGreetings.remove(key: key) ?? panic("Could not find the greeting!")
    // 或者...
    // let greeting <- self.dictionaryOfGreetings.remove(key: key)!
    return <- greeting
}
```

## 结语

这就是我们今天的内容！也许你想问： “如果我想*获取*一个数组或 dictionary 中的一个 resource，然后再对其进行一些操作，该怎么做？你当然可以做，但你第一步要先把 resource 从 arry/dictionary 中移动出来，做某些操作，然后再移动回去。明天，我们会讨论 reference，通过 reference 我们可以对 resource 做一些操作，但不需要移动 resource”

## 任务

今天的任务，是一个大一点的任务。

1. 编写你自己的合约，包含两个状态变量：一个 resource array，一个 resource dictionary。编写对应的函数来问添加/移除对应的元素。必须与上面用到的例子不同。
