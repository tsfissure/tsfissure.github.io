---
title: Vim tysformatter.vim manual
date: 2016-09-21 15:29:37
tags: Vim
---

## Vim format for c/cpp 1.0 manual

### 可用功能
- 去掉多余空格
- 在花括号两边加空格
- 在`= ^ |`两边加空格
- 在形如`+= *=`两边加空格

<-- more -->
### 使用
① 点击下载[tysformatter.vim](https://github.com/tsfissure/vim/tree/master/indent)
② 在你的`$VIMRUNTIME/indent/`目录下的c和cpp.vim中添加`source path`(`path`为刚才下载的tysextend.vim的存放路径)
③ 打开C或者C++文件按`F2`即可完成格式化,如果不喜欢`F2`或者被其他占用，可以`tysformatter.vim`的最后那个映射修改成你想要的
④ 格式化完成是会提示`Format Over`，你可以在`tysformatter.vim`中找到这字符串进行修改或者把那行删掉(不提示)
⑤ 建议用这格式化后再加一个`gg=G`命令缩进，因为我没加缩进，必要性不大

### 我的粗暴实现
需要在旁边添加空格的符号被我分成了四类:
> 
  1, 需要在左边加空格如`{`
  2, 需要在右边加空格如`; }`
  3, 需要在两边加空格的单个符号如`^ | =`
  4, 需要在两边加空格的两个符号如`>> << == +=`
> 

遍历文件每一行，对一行单独处理,取出那行为`text`，修改过程中也是对`text`修改，先是去掉多余空格，然后需要右边加的加上，左边的两边的依次加上。如果这行有修改，就把修改过后的`text`,`set`回去

### TODO
部分符号如`+,-,>,<`等没有处理，比较复杂，在后续心(灵)血(感)来(涌)潮(现)的时候一个个补上
把`{，else`等按设置处理是否新行
...