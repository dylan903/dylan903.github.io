---
title: Sublime Text3使用指南
date: 2019-08-31 10:20:02
author: dylan
top: false
cover: false
password: 
categories: summary
tags: 
    - Sublime Text3
    - 透明美化
    - python
    - 插件
    - vim
---
# 0x01 前言

sublime是一款轻便、快捷的编辑器，好处多多，
只不过配置起来麻烦了点，所有在此记录一下自用配置。
（本文默认环境为window 10）

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190831220729.png)

# 0x02 下载

[官网](https://www.sublimetext.com/)直达,下载安装即可

# 0x03 使用技巧

## 启用VIM模式

在`Preferences -> Setting - User`中将 
`ignored_packages` 的值注释掉

因为`Ctrl+f` `Ctrl+b`等vim快捷键和Sublime Text3冲突了，
所以在这个配置文件里添加一句

`"vintage_ctrl_keys": true`

附上[vim入门教程](https://www.cnblogs.com/hezhiyao/p/7624831.html)一份

代码如下：

```
    "vintage_ctrl_keys": true,
    "ignored_packages":
	[
		//注释掉开启vim模式
		// "Vintage"
	],
```

## 批量修改

单个文件批量修改：纯相同的内容：选中需要修改的内容Alt+F3(Mac下默认的是Ctrl+Command+G) ， 
或者连续 Ctrl+D（ 连续 Command+D(Mac) ) 之后重新写即可，使用Ctrl + U进行回退，使用Esc退出多重编辑。

## 分屏操作

Windows下：
Alt + Shift + 2进行左右分屏，
Alt + Shift + 8进行上下分屏，
Alt + Shift + 5进行上下左右分屏（即分为四屏）。

# 0x04 常用插件

新版的sublime已经默认安装了package control，直接一手`ctrl+shift+p`
输入框中输入`install`，点击列表中的`Package control:install package`
稍等片刻，弹出列表，即可搜索插件安装。
（加载和安装的失败，多试几次，基本都是网络原因）

## 主题插件 — Material Theme

搜索安装之后在
`Preference -> Settings -> User` 里面复制如下配置：
也可以去[Material Theme](https://github.com/equinusocio/material-theme)的GitHub仓库查看主题配置说明，自己配置

```
{
    "always_show_minimap_viewport": true,
    "bold_folder_labels": true,
    "color_scheme": "Packages/Material Theme/schemes/Material-Theme.tmTheme",
    "fade_fold_buttons": false,
    "font_options":
    [
        "gray_antialias"
    ],
    "font_size": 15,
    "ignored_packages":
    [
        "Vintage"
    ],
    "indent_guide_options":
    [
        "draw_normal",
        "draw_active"
    ],
    "line_padding_bottom": 3,
    "line_padding_top": 3,
    "material_theme_accent_scrollbars": true,
    "material_theme_arrow_folders": false,
    "material_theme_big_fileicons": true,
    "material_theme_bold_tab": true,
    "material_theme_bright_scrollbars": true,
    "material_theme_bullet_tree_indicator": true,
    "material_theme_compact_panel": true,
    "material_theme_compact_sidebar": true,
    "material_theme_contrast_mode": true,
    "material_theme_disable_folder_animation": false,
    "material_theme_disable_tree_indicator": true,
    "material_theme_panel_separator": true,
    "material_theme_small_statusbar": true,
    "material_theme_small_tab": true,
    "material_theme_tabs_autowidth": false,
    "material_theme_tabs_separator": false,
    "material_theme_tree_headings": true,
    "overlay_scroll_bars": "enabled",
    "show_encoding": true,
    "show_line_endings": true,
    "theme": "Material-Theme.sublime-theme"
}
```

## 透明插件 — Transparency

仅支持windows系统，直接搜索`Transparency`在线安装即可实现透明
在`View -> Windows's Transparency` 即可调整透明等级

## 代码补全 — Anaconda

python代码补全

`=>Preferences=>Package setting=> Anaconda =>Setting -User`

```
{
    //Python路径
    "python_interpreter": "C:/Users/AppData/Local/Programs/Python/Python36-32/python.exe",
    //忽略各种空格不对, 超过79字, import的函数没有使用的提醒,
    "pep8_ignore": ["E501", "W292", "E303", "W391", "E225", "E302", "W293", "E402"],
    "pyflakes_explicit_ignore":
    [
        "UnusedImport"
    ],
    //保存文件后自动pep8格式化
    "auto_formatting": true,
    "auto_formatting_timeout": 5,
    //库函数的提示
    "enable_signatures_tooltip": true,
    "merge_signatures_and_doc":true,

    //ST3也有自动补全提示，但只提示文件中输入过的单词，这个功能可用提示变量可用的函数等。
    "suppress_word_completions": true,
    "suppress_explicit_completions": true,
    "complete_parameters": true,
    //代码排版时，行的默认长度太短，根据喜好设置
    "pep8_max_line_length": 120,


}
```

## 调试插件 — SublimeREPL

安装好后，点击`Preferences -> Browse Packages`
找到`SublimeREPL\config\Python\Main.sublime-menu`文件，
然后用Sublime Text 3 打开，找到如图所示行，
修改为`"cmd": ["python", "-i", "-u","$file_basename"]`，保存。
这样相当于将SublimeREPL的python交互环境的命令改为运行当前文件的交互环境。

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190901005107.png)

然后设置快捷键 `Preferences -> Key Bindings `
如下`shift+f10`运行  `shift+f9`调试，可以修改为自己喜欢的快捷键
配合分屏操作，体验极佳

```
[
    {
    "keys": ["shift+f10"],
    "caption": "SublimeREPL: Python - RUN current file",
    "command": "run_existing_window_command",
    "args": {
        "id": "repl_python_run",
        "file": "config/Python/Main.sublime-menu"}
    },
    {
    "keys": ["shift+f9"],
    "caption": "SublimeREPL: Python - PDB current file",
    "command": "run_existing_window_command",
    "args": {
        "id": "repl_python_pdb",
        "file": "config/Python/Main.sublime-menu"}
    },
]
```

> 如果使用快捷键运行Python文件如果出现如下报错：
> `['$file_basename': [Errno 2] No such file or directory]`
> 原因是当你使用`shift+f10`执行了一次之后，焦点已经不再当前执行的`.py`文件上了
> 点击要执行的`.py`文件，再执行就ok

![](https://raw.githubusercontent.com/dylan903/ImgUrl/master/Img/20190901011958.png)


**pdb 常用命令**

|     命令      |           解释           |
| :-----------: | :----------------------: |
|  break 或 b   |         设置断点         |
| continue 或 c |       继续执行程序        |
|   list 或 l   |    查看当前行的代码段     |
|   step 或 s   |         进入函数         |
|  return 或 r  | 执行代码直到从当前函数返回 |
|   exit 或 q   |        中止并退出         |
|   next 或 n   |        执行下一行         |
|      pp       |      	打印变量的值       |
|     help      |           帮助           |

## 注释插件 — DocBlockr 

这个插件可以自动生成函数类型、参数个数及类型、函数返回值等
在函数上方输入`/**`，回车即可

python可以安装`DocBlockr python`
在函数里输入`'''`回车即可

## 终端插件 — Terminal

快捷键 `ctrl+shift+t` `ctrl+alt+shift+t`
安装好后，修改如下配置文件
`=>Preferences=>Package setting=>Terminal=>Setting -User`

```
{
    "terminal": "C:\\windows\\system32\\cmd.exe",
     "parameters": ["/START","%CWD%"]
}
```

## 右键菜单增强 — SideBarEnhancements

增强右键菜单功能：
在资源管理器中打开、新建文件、新建文件夹、以…打开、在浏览器中打开等等
可以配置在浏览器中打开快捷键
这里设置按Ctrl+Shift+C复制文件路径，按F2即可在Chrome浏览器预览效果
(如果需要的话，也可以根据自己的需要为Firefox，Safari，IE，Opera等加上)，
当然你也可以自己定义喜欢的快捷键，最后注意代码中的浏览器路径要以自己电脑里的文件路径为准。

`preferences->package setting->side bar->Key Building-User`

```
[
    { "keys": ["ctrl+shift+c"], "command": "copy_path" },
    //chrome
    { "keys": ["f2"], "command": "side_bar_files_open_with",
            "args": {
                "paths": [],
                "application": "C:\\Users\\jeffj\\AppData\\Local\\Google\\Chrome\\Application\\chrome.exe",
                "extensions":".*"
            }
     }
]
```

## 新建文件模板 — FileHeader

功能强大，自动的监测创建新文件动作，自动根据类型  添加模板。
几乎支持所有的编程语言，并且支持用户自定义语言。
能够自动的更新文件最后修改时间。
能够自动的更新文件最后的修改者。
不仅支持创建已经使用模板初始化好的文件，而且支持将header添加到已经存在的文件头部，并且支持批量添加。

设置默认文件模板:

`=>Preferences=>Package setting=>FileHeader=>Setting -User`

```
{
	"Default":{
		"email":"xxxx@qq.com",
		"last_modified_by":"小红",
		"author":"小明"
	}
}

```

设置对应文件模板:

`Preferences -> Browse Packages`

进入`FileHeader\template\header`目录，
找到你想要添加头文件的语言对应的`.tmpl`文件修改即可

## 快速创建文件 — advancedNewFile

快捷键`ctrl+alt+n`
在弹出的输入框里输入我们需要新建的文件名回车即可,
默认路径为当前文件夹下，如果当前没有目录则会存到用户家目录
也可以带路径输入`test/test.py`
（如果不确定路径，可以在输入框下方小字查看完整路径）

***
# 参考文章

[如何优雅地使用Sublime Text3 - 简书](https://www.jianshu.com/p/3cb5c6f2421c/)
[（干货）自定义使用Sublime Text 3 - 简书](https://www.jianshu.com/p/73304373539f)
[SubLime Text 3 配置SublimeREPL来交互式调试程序](https://www.cnblogs.com/JackyXu2018/p/8821482.html)
[让你用sublime写出最完美的python代码--windows环境](https://www.cnblogs.com/zhaof/p/8126306.html)
[sublime text 3 打造python3环境（代码自动补全，运行程序，高亮显示）](https://blog.csdn.net/zxy987872674/article/details/81707241)