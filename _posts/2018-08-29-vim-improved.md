---
title: vim 入门提高
key: 20180829
tags: Vim
key: "vim-20180829"
---

# 写在开头的话
我将vim的入门分为两部分<br/>
<!--more-->
第一部分是熟悉各种操作，掌握vim[速查卡](http://michael.peopleofhonoronly.com/vim/)的绿色和黄色操作。<br/>
第二部分是vimrc的无插件配置以及映射<br/>

本文将不再讲述，如何入门vim第一部分。直接给出方法。
1. 使用vimtutor一星期，最后达到5分钟可以过完一遍的水平。
2. 看完这篇[文章](https://coolshell.cn/articles/5426.html)
3. 或者可以根据vim速查卡，百度使用，刻意练习。  
<br/>

二者配合使用大概一星期左右时间，代表你进入vim入门一半，剩下来一半在于熟悉vimrc的各种设置以及映射。完成这俩部分才能代表你的vim入门。
本文就是给出学习后半部分的一些建议。<br/> 
对于第二部分，仅挑出设置选项的基本方式和基本映射简单讲解。继续深入请看文章最后。在这篇文章之前，你需要了解基本的vimrc设置的功能，不需要明白太深。比如，你需要明白set number 是设置行号，set wrap是开启显示折叠之类的基本即可。  <br/>
<br/>
在文章的最后，将尽量按照我的学习曲线的方式给出一些资料，和学习方法。

## 设置选项
vim 总共有俩种设置方式，分别为
* 布尔选项
    * set `<name>`  &emsp;  e.g: set number
    * set `no<name>`  &emsp;  e.g: set nonumber
    * set `<name>!`  &emsp;  e.g: set number!
    * set `<name>?`  &emsp;  e.g: set number?  
* 键值选项
    * set `<name>=<value>`  &emsp;  e.g: set numberwidth=5

### 解释
#### 布尔选项  
number就是一个布尔选项，所有布尔选项都可以通过一种  
set `<name>` 的方式开启， set no`<name>`的方式关闭，这个`<name>`就是所谓的布尔值, 
可以用set `<name>?`的方式查看这个布尔值, 可以通过set `<name>!`将这个布尔值取反。
* 使用示例  
`: set number` &emsp; 开启行号  
`: set nonumber` &emsp; 关闭行号  
`: set number?` &emsp; 查看行号是否开启，如果显示number代表开启，显示nonumber代表关闭  
`: set number!` &emsp; 对bool值取反，如果行号开启，则关闭，如果关闭，则开启    
<br/>
#### 键值选项  
* 使用示例  
`: set numberwidth=5` 

#### 小结
* 以上所有选项既可以直接使用，即对当前buff生效，也可以写在vimrc中，这时不用加`:`
* `help 'number'` 请善用帮助 

## 基本映射
很多人不太明白映射是干什么的，其基本含义同数学中的概念一样，比如我将str映射为string。
对写的人来说，就是写str，对看的人来说就是看到的string

* 例子1：  
`: imap str string` 此时在插入模式下输入str就是输入string。目前不要去弄明白imap是什么意思，先明白映射可以做什么，理解映射的含义即可。
* 理解映射模式 http://www.pythonclub.org/linux/vim/map 接下请查阅这篇文章
1. map和nmap, imap的区别? hind-工作模式不同
2. 如何查看已经存在的映射? ma 
3. 如何查看keyname，即键盘符号?
4. 如何取消映射? unmap ...

### 精确映射
目前你应该已经会制作`map`，`nmap`，`imap`的映射了。根据设计模式的单一原则也知道，对于不需要这个模式的映射，尽量不使用。因此，`map`在你学会使用映射后，使用时将会被抛弃

问题：map 和 noremap的区别？  
noremap的意思是no recursive map的意思。就是非递归映射。那么map自然就是递归的了。  
递归和非递归又有什么区别呢？  
`: map x dd`   
`: map \ x`  
当你按下\ 后，你觉得是删除一个字符，还是删除一行呢？如果你的递归学的好的话应该明白你将会删除一行。因为vim的本质其实是c语言写的。  
这显然不是我们想要的，因此使用递归映射将大大增加你写映射的难度。  
建议是：**永远不要写递归映射** 

### 练习肌肉记忆
算是vim trick之一吧。很多tutorial上建议这样一个映射  
`inoremap jk <Esc>`  
`inoremap <Esc> <nop>`  
到现在，看懂第一个映射应该毫无压力。第二个映射就是禁止了原来的`<Esc>`的功能，现在你使用Esc将无法退出。现在只能使用jk来退出。

##### Trick
`set number`  
`nnoremap <F1> :set number!`  
达到使用F1对行号进行开关，请自己尝试设置`set wrap` `set hlsearch` `set incsearch`

## 写在最后的话
我自己学习vim算是东拼西凑的学习，晚上经常喜欢看看帖子研究下别人的vimrc是什么意思，也找了一些资料来看。  
以下列举  
《vim中文文档》  
[《笨方法学vim》](http://learnvimscriptthehardway.onefloweroneworld.com)  
第一本毫无疑问工具书籍，第二本是学习书籍。建议掌握除vim脚本之外的所有内容。  
如果你可以翻墙的话，直接google ：how to learn vim 可以按此方法学习。我目前就是按照这个方法，还在学习中，如果我可以学到一定高度，就会更新《vim中级and提高》。  
本文没有讲有关插件的事情。这个直接参照[这里](https://github.com/yangyangwithgnu/use_vim_as_ide)即可，但是在没有对vim有一定掌握后不建议使用插件。会妨碍vim的学习。  
### 关于vim的学习  
强烈建议看这篇[文章](https://medium.com/actualize-network/how-to-learn-vim-a-four-week-plan-cd8b376a9b85)
需要使用科学上网工具，[如何建立自己的科学上网工具](http://jiyiren.github.io/2016/10/06/fanqiang)

## 下期vim篇预告
我会写一个自己的vimrc。带有中文注释的，不涉及任何插件的，以及我习惯使用的映射，还有些我了解vim trick。
