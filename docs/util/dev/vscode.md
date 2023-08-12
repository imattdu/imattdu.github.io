













![](http://raw.githubusercontent.com/imattdu/img/main/img/20210518224538.png)

 



新建文件夹



![](https://raw.githubusercontent.com/imattdu/img/main/img/20210518200238.png)





vscode打开该文件夹



![](https://raw.githubusercontent.com/imattdu/img/main/img/20210518200350.png)









![](https://raw.githubusercontent.com/imattdu/img/main/img/20210518200442.png)









**文件->首选项->设置->搜索auto save**

off: 关闭自动保存
afterDelay: XX毫秒后自动保存，这个就是我所讲的解决方法，下面会详细介绍
onFocusChange: 当焦点移出编辑框
onWindowChange: 当焦点移出VSCode窗口
这里说的是焦点而不是鼠标，移到外面去后还要点一下感觉还是不太方便，这也是网上搜到的大多数答案。。。所以我们果断选择afterDelay，然后再如下图设置自动保存时间间隔





![](http://raw.githubusercontent.com/imattdu/img/main/img/202110100305252.png)







![](http://raw.githubusercontent.com/imattdu/img/main/img/202110100306673.png)







![](http://raw.githubusercontent.com/imattdu/img/main/img/202110100308714.png)







![](http://raw.githubusercontent.com/imattdu/img/main/img/202110100309508.png)







![](http://raw.githubusercontent.com/imattdu/img/main/img/202110100310443.png)





![](http://raw.githubusercontent.com/imattdu/img/main/img/202110100311819.png)











### plugin 插件

vscode 右上角出现运行按钮

#### code runner

![](https://raw.githubusercontent.com/imattdu/img/main/img/202307151604897.png)







#### go 



![](https://raw.githubusercontent.com/imattdu/img/main/img/202307151607981.png)



安装完成后  cmd+shift+p 输入 Go:install/update

安装相关工具



#### IntelliJ IDEA Keybindings

idea 快捷键

![](https://raw.githubusercontent.com/imattdu/img/main/img/202307151609197.png)







#### C/C++ Extension Pack

安装这个会自动安装c/c++ ,c/c++ Themes, CMake Tools 作为扩展包安装





![](https://raw.githubusercontent.com/imattdu/img/main/img/202307151626972.png)







Chinese (Simplified) (简体中文)









#### Atom One Dark Theme

![](https://raw.githubusercontent.com/imattdu/img/main/img/202307160204815.png)





### 配置语言环境



#### go



launch.json



```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch Package",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${file}"
        }
    ]
}
```





点击运行按钮接口



![](https://raw.githubusercontent.com/imattdu/img/main/img/202307151630893.png)





会自动创建tasks.json

```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: gcc 生成活动文件",
            "command": "/usr/bin/gcc",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```







