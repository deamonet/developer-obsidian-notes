# Maven中dependencyManagement的作用

## 说明

使用`dependencyManagement`可以统一管理项目的版本号，确保应用的各个项目的依赖和版本一致，不用每个模块项目都弄一个版本号，不利于管理，当需要变更版本号的时候只需要在父类[容器](https://cloud.tencent.com/product/tke?from=10680)里更新，不需要任何一个子项目的修改；如果某个子项目需要另外一个特殊的版本号时，只需要在自己的模块`dependencies`中声明一个版本号即可。子类就会使用子类声明的版本号，不继承于父类版本号。

##dependencyManagement与dependencys的区别

1)`Dependencies`相对于`dependencyManagement`，所有生命在`dependencies`里的依赖都会自动引入，并默认被所有的子项目继承。

2)`dependencyManagement`里只是声明依赖，并不自动实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且`version`和`scope`都读取自父`pom`;另外如果子项目中指定了版本号，那么会使用子项目中指定的jar`版本`。