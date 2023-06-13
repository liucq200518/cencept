# 概述

使用该方式搭建项目结合规范的代码可以对模块进行任意的组合

这里模块化的设想其实就是可插拔的功能模块，如果需要这个功能，就把这个模块用`Gradle`或`Maven`的方式编译进来

如果不需要，去掉对应的依赖就行了，避免改动代码

假设有`A`，`B`，`C`三个模块

- 我们可以实现`A`，`B`，`C`分别作为一个服务的微服务
- 我们可以实现`AB`合并为一个服务，`C`单独为一个服务的微服务
- 我们可以实现`AC`合并为一个服务，`B`单独为一个服务的微服务
- 我们可以实现`BC`合并为一个服务，`A`单独为一个服务的微服务
- 我们可以实现`ABC`合并为一个单体服务

在开发新项目时，可以先使用单体应用

等业务的体量变大之后，将大体量模块单独作为一个服务，切换为微服务模式

而单体应用和微服务切换不需要修改业务代码

# 使用

需要访问`Github`下载`Gradle`，注意先配置好网络

1. 搜索`Concept Cloud`

2. 新建项目

File -> New -> Project -> Concept Cloud

![plugin-project-create1](https://github.com/Linyuzai/concept/assets/18523183/9b7540ae-7773-4f8e-9811-7675f141dab9)

![plugin-project-create2](https://github.com/Linyuzai/concept/assets/18523183/2f46d4cb-802e-4374-bfda-562990162509)

3. 生成`domain`模块代码

先在`domain`模块下新建包

![plugin-code-domain-select](https://github.com/Linyuzai/concept/assets/18523183/223a173a-6325-4dec-9cde-99e69bbf0516)

选中新建的包，右键 -> Concept Cloud -> Generate Domain Code...

![plugin-code-domain-config](https://github.com/Linyuzai/concept/assets/18523183/fb9c422f-3d3d-44af-b51b-488d18df004a)

在包下会生成对应的文件

![plugin-code-domain-files](https://github.com/Linyuzai/concept/assets/18523183/4db39e27-8ded-4287-9e8c-6efdcd58b60c)

4. 生成`module`模块代码

先在`domain`模块下新建包

![plugin-code-module-select](https://github.com/Linyuzai/concept/assets/18523183/04d6a287-c67b-4f05-98ad-8b9c2dc5ca8f)

选中新建的包，右键 -> Concept Cloud -> Generate Module Code...

![plugin-code-module-config](https://github.com/Linyuzai/concept/assets/18523183/afa4d546-6a48-4c3e-a76d-afd0c847cf3e)

在包下会生成对应的文件

![plugin-code-module-files](https://github.com/Linyuzai/concept/assets/18523183/f0199e70-62aa-46fa-9922-3d2d998a8ef1)

# 掘金专栏

专栏中对于一些功能思路有讲解，还有一些示例

https://juejin.cn/column/7140131104270319629

# 