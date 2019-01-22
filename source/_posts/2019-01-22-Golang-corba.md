---
title: Golang-corba
date: 2019-01-22 11:18:51
categories:
- Golang
tags:
- Golang
- cli
comments: true
---

使用corba库可以方便的编写cli程序，其中Docker和Kubernetes的客户端程序便是基于corba库进行开发的。

corba库的特点:

- 支持子命令，如`app fetch`, `app push`等
- 兼容Posix标准的命令行模式
- 支持子命令的嵌套
- 支持全局、局部和串联的`Flag`
- 使用`corba create appname`和`corba add cmdname`可以方便的创建应用程序和增加命令
- 命令输入错误时的智能提示
- 自动生成命令和Flag的帮助信息
- 默认支持`-h`和`--help`，输出帮助信息
- Bash下命令的自动补全
- 自动生成应用的man手册信息
- 支持命令别名
- 支持自定义`Help`、`Usage`等信息

## 基本概念

- `Command`: 命令，代表执行的动作
- `Arg`: 命令对象，代表动作对应的对象
- `Flag`: 命令参数

因此，cli程序的使用方式为`AppName Command Arg --Flag`，其中`Command`还有可能包含子命令

## 项目初始化

- 安装:`go get -u github.com/spf13/cobra/cobra`
- 导入: `import "github.com/spf13/cobra"`

- 项目结构:

```Go
appName/
    cmd/
        cmd1.go
        cmd2.go
    main.go
```

- main.go: 初始化corba

```Go
package main

import (
  "appName/cmd"
)
func main(){
    cmd.Execute()
}
```

### 使用corba命令自动生成代码

- 执行`corba init toolchain`: 在{GOPATH}/src/toolchain下生成应用代码

- 在{GOPATH}/src/toolchain下执行`corba add scenario`: 为应用程序增加命令scenario

- 在{GOPATH}/src/toolchain下执行`corba add create -p scenarioCmd`: 为scenario增加子命令create

### 手动编写代码

- 在{GOPATH}/src/toolchain下手动创建main.go文件，进行corba的初始化

- 在{GOPATH}/src/toolchain/cmd手动创建各个命令文件，对命令进行定义

## Command对象

命令对象是cli应用的核心，命令可能还会包括子命令

```Go
type Command struct {
    Use string // 命令的名称

    Aliases []string // 命令的别名列表

    SuggestFor []string // 该命令可以被提议使用的名称列表

    Short string // 命令的简短描述，在help信息中展示

    Long string // 命令的详细描述，在'help <this-command>'中展示

    Example string // 命令的使用例子

    ValidArgs []string // 在Bash自动补全中的所有合法的Arg列表

    Args PositionalArgs // Arg参数校验规则函数

    ArgAliases []string // Arg参数别名列表

    BashCompletionFunction string // Bash自动补全时的字符串

    Deprecated string // 使用已被弃用的命令时输出的字符串

    Hidden bool // 命令是否隐藏，如果隐藏则不会输出给用户

    Annotations map[string]string // 命令关联的KV对，用于标识命令或对命令进行分组

    Version string // 命令的版本

    // *Run系列函数的执行顺序如下:
    //   * PersistentPreRun() 当前命令的子命令可以实现和运行
    //   * PreRun() 当前命令的子命令无法实现和运行
    //   * Run()
    //   * PostRun() 当前命令的子命令无法实现和运行
    //   * PersistentPostRun() 当前命令的子命令可以实现和运行
    // 所有函数的输入参数相同的Arg参数
    PersistentPreRun func(cmd *Command, args []string)
    PersistentPreRunE func(cmd *Command, args []string) error
    PreRun func(cmd *Command, args []string)
    PreRunE func(cmd *Command, args []string) error
    Run func(cmd *Command, args []string) // 命令的具体实现逻辑，大部分的命令只需要实现该参数
    RunE func(cmd *Command, args []string) error
    PostRun func(cmd *Command, args []string)
    PostRunE func(cmd *Command, args []string) error
    PersistentPostRun func(cmd *Command, args []string)
    PersistentPostRunE func(cmd *Command, args []string) error
}
```

## Flag的使用

- Persistent Flag: 全局Flag，对当前命令及其子命令皆可用

```Go
rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")
```

- Local Flag: 局部Flag，只对当前命令可用

```Go
rootCmd.Flags().StringVarP(&Source, "source", "s", "", "Source directory to read from")
```

- Flag与Config绑定:

```Go
rootCmd.PersistentFlags().StringVar(&author, "author", "YOUR NAME", "Author name for copyright attribution")
viper.BindPFlag("author", RootCmd.PersistentFlags().Lookup("author"))
```

- 必填的Flag

```Go
rootCmd.Flags().StringVarP(&Region, "region", "r", "", "AWS region (required)")
rootCmd.MarkFlagRequired("region")
```

## Args的校验

- 内置校验规则

NoArgs: 如果包含任何Arg，命令报错
ArbitraryArgs: 命令接受任何Arg
OnlyValidArgs: 如果有Arg不在ValidArgs中，命令报错
MinimumArgs(init): 如果Arg数目少于N个后，命令报错
MaximumArgs(init): 如果Arg数目多余N个后，命令报错
ExactArgs(init): 如果Arg数目不是N个话，命令报错
RangeArgs(min, max): 如果Arg数目不在范围(min, max)中，命令报错

- 自定义校验规则

```Go
var cmd = &cobra.Command{
    Short: "hello",
    Args: func(cmd *cobra.Command, args []string) error {
        if len(args) < 1 {
        return errors.New("requires at least one arg")
        }
        if myapp.IsValidColor(args[0]) {
        return nil
        }
        return fmt.Errorf("invalid color specified: %s", args[0])
    },
    Run: func(cmd *cobra.Command, args []string) {
        fmt.Println("Hello, World!")
    },
}
```

## 自定义Help和Usage

- Help

```Go
cmd.SetHelpCommand(cmd *Command)
cmd.SetHelpFunc(f func(*Command, []string))
cmd.SetHelpTemplate(s string)
```

- Usage

```Go
cmd.SetUsageFunc(f func(*Command) error)
cmd.SetUsageTemplate(s string)
```

## 生成帮助文档

```Go
package main

import (
    "log"

    "github.com/spf13/cobra"
    "github.com/spf13/cobra/doc"
)

func main() {
    cmd := &cobra.Command{
        Use:   "test",
        Short: "my test program",
    }
    err := doc.GenMarkdownTree(cmd, "/tmp")
    if err != nil {
        log.Fatal(err)
    }
}
```

[corba](https://github.com/spf13/cobra)
