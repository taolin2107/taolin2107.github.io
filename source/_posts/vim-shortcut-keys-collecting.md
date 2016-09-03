---
title: Vim快键键汇总（持续更新）
date: 2015-08-23 11:37:37
tags: [vim, 编辑器, 命令行]
categories: vim
---


### 1. 移动光标

- h &nbsp;&nbsp;  左移
- j &nbsp;&nbsp;  下移
- k &nbsp;&nbsp;  上移
- l &nbsp;&nbsp;  右移
- $ &nbsp;&nbsp;  移动到当前行行尾
- ^ &nbsp;&nbsp;  移动到当前行的第一个非空白字符上
- 0 &nbsp;&nbsp;  移动到当前行的行首，同Home键
- gg &nbsp;&nbsp;  移动到文档的第一行
- G  &nbsp;&nbsp;  移动到文档的最后一行
- H  &nbsp;&nbsp;  移动到当前屏幕中的第一行
- M  &nbsp;&nbsp;  移动到当前屏幕中的中间行
- L  &nbsp;&nbsp;  移动到当前屏幕中的最后一行
- 行数+G  &nbsp;&nbsp; 将光标移动到文档的指定行，比如移动到23行（23G）
- 数字+%  &nbsp;&nbsp; 将光标移动到文档百分比位置，比如移动到文档12%的位置（12%）
- `.  &nbsp;&nbsp; 跳转至上次编辑位置
- %  &nbsp;&nbsp;  跳到与当前括号匹配的括号处，如当前在'{'，则跳转到与之匹配的'}'处
- Ctrl+g  &nbsp;&nbsp;   获取当前位置信息
- zz, zt, zb  &nbsp;&nbsp;  滚动文档，将光标所在行居中/置顶/置尾
- f[x]  &nbsp;&nbsp;  向右移动至字母x在当前行内下一次出现的位置，f指forward，x可以是任一个字母，你还可以用;来重复执行刚才的fx
- tx  &nbsp;&nbsp;   同fx，区别在于光标会停留到x的左侧
- Fx  &nbsp;&nbsp;   同fx，只是向左移动

### 2. 内容搜索

- /string  &nbsp;&nbsp;   向前搜索指定字符串
- ?string   &nbsp;&nbsp;  向后搜索指定字符串
- `/\<make\>`  &nbsp;&nbsp; 搜索make，前后不包含其他字母
- `*`  &nbsp;&nbsp;   查找光标所在处的单词，向下查找
- `#`  &nbsp;&nbsp;   查找光标所在处的单词，向上查找
**n/N配合以上查找命令执行**
- n   &nbsp;&nbsp;  搜索指定字符串的下一个出现位置
- N   &nbsp;&nbsp;  搜索指定字符串的上一个出现位置

### 3. 查找替换

- :%s/old/new/g   &nbsp;&nbsp;    全文替换指定字符串
- :s/vivian/sky/  &nbsp;&nbsp;    替换当前行第一个vivian 为sky
- :s/vivian/sky/g  &nbsp;&nbsp;   替换当前行所有vivian 为sky
- :n,$s/vivian/sky/  &nbsp;&nbsp; 替换第 n 行开始到最后一行中每一行的第一个vivian为sky, n为数字，若n为.，表示从当前行开始到最后一行
- :2,$s/vivian/sky/g  &nbsp;&nbsp;   替换第 2 行开始到最后一行中每一行所有vivian为sky
- :%s/vivian/sky/   &nbsp;&nbsp;    （等同于 :g/vivian/s//sky/） 替换每一行的第一个vivian 为sky
- :%s/vivian/sky/g   &nbsp;&nbsp;  （等同于 :g/vivian/s//sky/g） 替换每一行中所有vivian 为sky
- `(<w+>)_s*1`   &nbsp;&nbsp;   查找重复的连续单词，如：this this，输入`/\(\<\w\+\>\)\_s*\1`
<!-- more -->

### 4. 内容编辑

- u  &nbsp;&nbsp;   撤销(undo)
- ctrl+r  &nbsp;&nbsp; 重做(redo)
- r[x]   &nbsp;&nbsp;  替换一个字符，将当前字符替换为x
- J  &nbsp;&nbsp;   将下一行和当前行连接为一行
- x  &nbsp;&nbsp;   删除当前字符(此处删除是剪切)
- X  &nbsp;&nbsp;   删除前一个字符
- v  &nbsp;&nbsp;   进入选择模式
- V  &nbsp;&nbsp;   进入行选择模式
- d  &nbsp;&nbsp;   删除选定内容
- dd &nbsp;&nbsp;   删除光标所在行
- dw &nbsp;&nbsp;   删除当前光标之后的一个词(word)
- db &nbsp;&nbsp;   删除当前光标之前的一个词(word)
- D  &nbsp;&nbsp;   删除到行末(包含当前光标位置)
- cc &nbsp;&nbsp;   删除当前行并进入编辑模式
- cw &nbsp;&nbsp;   删除当前词(word)，并进入编辑模式
- c$ &nbsp;&nbsp;   删除从当前位置至行末的内容，并进入编辑模式
- s  &nbsp;&nbsp;   删除当前字符并进入编辑模式
- S  &nbsp;&nbsp;   删除光标所在行并进入编辑模式
- y  &nbsp;&nbsp;   复制当前选中内容
- yy &nbsp;&nbsp;   复制一行，此命令前可跟数字，标识复制多行，如6yy，表示从当前行开始复制6行
- yw &nbsp;&nbsp;   复制一个词(word)
- y$ &nbsp;&nbsp;   复制到行末
- P  &nbsp;&nbsp;   粘贴粘贴板的内容到当前行的上面
- p  &nbsp;&nbsp;   粘贴粘贴板的内容到当前行的下面
- dd+p  &nbsp;&nbsp;  下移一行
- dd+shit+p   &nbsp;&nbsp;  上移一行
- .   &nbsp;&nbsp;  重复上一个编辑命令
- ~   &nbsp;&nbsp;  切换大小写，当前字符
- g~iw   &nbsp;&nbsp;  切换当前字的大小写
- gUiw   &nbsp;&nbsp;  将当前字变成大写
- guiw   &nbsp;&nbsp;  将当前字变成小写
- guu/Vu &nbsp;&nbsp;  把一行全部变成小写
- gUU/VU &nbsp;&nbsp;  把一行全部变成大写
- `>>`   &nbsp;&nbsp;  将当前行右移一个单位
- `<<`   &nbsp;&nbsp;  将当前行左移一个单位(一个tab符)
- ==   &nbsp;&nbsp;  自动缩进当前行
- ctrl+a/x  &nbsp;&nbsp;   若当前光标所在word是数字，可递增/递减该数字
- ctrl+d/t  &nbsp;&nbsp;   insert模式下，缩进/反缩进当前行
- ctrl+w    &nbsp;&nbsp;   insert模式下，向后删除一个word
- ctrl+y    &nbsp;&nbsp;   insert模式下，复制上一行同列字符
- ctrl+n/p  &nbsp;&nbsp;   insert模式下，代码提示下拉框

### 5. 插入模式

- i  &nbsp;&nbsp;   从当前光标处进入插入模式，重复行插入，如3i，ESC后重复3行
- I  &nbsp;&nbsp;   进入插入模式，并置光标于行首
- a  &nbsp;&nbsp;   追加模式，置光标于当前光标之后
- A  &nbsp;&nbsp;   追加模式，置光标于行末
- o  &nbsp;&nbsp;   在当前行之下新加一行，并进入插入模式
- O  &nbsp;&nbsp;   在当前行之上新加一行，并进入插入模式
- Esc  &nbsp;&nbsp;   退出插入模式

### 6. 可视模式

- v   &nbsp;&nbsp;  进入可视模式，单字符模式
- V   &nbsp;&nbsp;  进入可视模式，行模式
- ctrl+v   &nbsp;&nbsp;   进入列编辑模式，编辑完一行，ESC后其他行生效，插入是I和A，替换是r
- o   &nbsp;&nbsp;  跳转光标到选中块的另一个端点
- U   &nbsp;&nbsp;  将选中块中的内容转成大写
- O   &nbsp;&nbsp;  跳转光标到块的另一个端点
- aw  &nbsp;&nbsp;   选中一个字
- ab  &nbsp;&nbsp;   选中括号中的所有内容，包括括号本身
- aB  &nbsp;&nbsp;   选中{}括号中的所有内容
- ib  &nbsp;&nbsp;   选中括号中的内容，不含括号
- iB  &nbsp;&nbsp;   选中{}中的内容，不含{}
- ctrl+r  &nbsp;&nbsp;   在插入模式下按ctrl+r，然后输入=，再输入运算式，按Enter键，就会将结果插入当前位置
- :ab [缩写] [要替换的内容]  &nbsp;&nbsp;   回车后，之后输入的缩写都会实时的替换
- :%!xxd   &nbsp;&nbsp;  转为16进制模式，:%!xxd -r 恢复
- ggVGg?   &nbsp;&nbsp;  进行ROT13编码
- ctrl+o/i  &nbsp;&nbsp;   跳到之前、之后的修改位置

### 7. 标记块操作

- `>`   &nbsp;&nbsp;  块右移
- `<`   &nbsp;&nbsp;  块左移
- y   &nbsp;&nbsp;  复制块
- d   &nbsp;&nbsp;  删除块
- ~   &nbsp;&nbsp;  切换块中内容的大小写

### 8. 保存与退出

- :w     &nbsp;&nbsp;  将缓冲区写入文件，即保存修改
- :wq    &nbsp;&nbsp;  保存修改并退出
- :x/ZZ  &nbsp;&nbsp;  保存修改并退出
- :q     &nbsp;&nbsp;  退出，如果对缓冲区进行过修改，则会提示
- :q!/ZQ  &nbsp;&nbsp;  强制退出，放弃修改
- :w !sudo tee %   &nbsp;&nbsp;  忘记以root身份修改了只有root可写的文件

### 9. 高级移动

- ctrl-f  &nbsp;&nbsp;   上翻一页
- ctrl-b  &nbsp;&nbsp;   下翻一页
- ctrl-u  &nbsp;&nbsp;   上翻半页
- ctrl-d  &nbsp;&nbsp;   下翻半页
- ctrl-e  &nbsp;&nbsp;   上滚一行，光标行数不变
- ctrl-y  &nbsp;&nbsp;   下滚一行，光标行数不变
- set sub  &nbsp;&nbsp;  两个分屏中的文件同步滚动
- set sub! &nbsp;&nbsp;   取消文件同步滚动。注：set scb 是set scrollbind 的简写。
- w  &nbsp;&nbsp;    跳到下一个字首，按标点，空格或单词分割，比如："."、","或者")"
- W  &nbsp;&nbsp;    跳到下一个字首，长跳，以空白为分界的word成为WORD
- e  &nbsp;&nbsp;    跳到下一个字尾，先跳到当前字尾
- E  &nbsp;&nbsp;    跳到下一个字尾，长跳
- ge &nbsp;&nbsp;    跳到上一个字尾
- b  &nbsp;&nbsp;    跳到上一个字首
- B  &nbsp;&nbsp;    跳到上一个字，长跳
- fx  &nbsp;&nbsp;    在当前行中找x字符，找到了就跳转至
- ;   &nbsp;&nbsp;    重复上一个f命令，而不用重复的输入fx
- tx  &nbsp;&nbsp;    与fx类似，但是只是跳转到x的前一个字符处
- Fx  &nbsp;&nbsp;    跟fx的方向相反
- ),(   &nbsp;&nbsp;   跳转到上/下一个语句
- 书签
- ma   &nbsp;&nbsp;   把当前位置存成标签a
- `a   &nbsp;&nbsp;   跳转到标签a处

### 10. 窗口操作

- :split/sp   &nbsp;&nbsp;   水平分割一个窗口
- :vsplit/svp  &nbsp;&nbsp;  垂直分割一个窗口
- :close   &nbsp;&nbsp;      关闭一个窗口
- ctrl+w   &nbsp;&nbsp;      2次，切换窗口，ctrl+w+h/j/k/l到左/下/上/右边窗口，大写则是移动窗口位置
- ctrl+w s/v &nbsp;&nbsp;    水平/垂直分割当前窗口，打开当前文件
- ctrl+w c   &nbsp;&nbsp;    关闭当前窗口
- ctrl+w q   &nbsp;&nbsp;    关闭当前窗口，如果是最后一个，则退出vim
- ctrl+w </>/+/-/=  &nbsp;&nbsp;  调节窗口的宽度，高度，恢复均等
- :He  He!  &nbsp;&nbsp;   在下面/上面分屏浏览目录
- :Ve  Ve!  &nbsp;&nbsp;   在左边/右边分屏浏览目录
- :Te  &nbsp;&nbsp;    打开一个标签，浏览目录
- gt/gT &nbsp;&nbsp;   切换到下一个/上一个标签
- :tabs  &nbsp;&nbsp;  查看打开的标签
- :q   &nbsp;&nbsp;    退出当前标签
- :ctrl+f  &nbsp;&nbsp;   查看历史命令

### 11. 其他

- :set fileformat=unix  &nbsp;&nbsp;  去掉因编码问题出现的`^M`，或者fileformats，简写为ff和ffs
- :set ff=win  &nbsp;&nbsp; 让linux下的Vim按照Windows格式来解析文件。
- set ffs=mac,unix,dos  &nbsp;&nbsp;  添加到配置文件来告诉Vim按照怎样的顺序来尝试适配编码
- :set fileencoding    &nbsp;&nbsp;  可以显示文件的编码格式，简写形式是:set fenc
- :set fenc=utf-8   &nbsp;&nbsp;  可以转换文件的编码格式为utf-8
- :set encoding  &nbsp;&nbsp;  可以显示编辑器当前使用什么编码方案来展示文档，简写为enc
- :set enc=utf-8  &nbsp;&nbsp;  可以将vim使用的编码方案切换的utf-8
- :set nu/nonu   &nbsp;&nbsp;   显示/取消 行号
- :set mouse=n/   &nbsp;&nbsp;   启用/取消 鼠标操作光标
- :sh 或 ctrl+z/:sh    &nbsp;&nbsp;   进入shell命令行，vim后台运行
- fg 或 exit  &nbsp;&nbsp;   从shell命令行返回到vim
- ga/g8  &nbsp;&nbsp;   查看光标处的ascii码/utf-8码

### 12. 常用插件

- NERDTree    &nbsp;&nbsp;   打开树形目录
- vim-scripts/taglist.vim    &nbsp;&nbsp;    打开函数列表 :TlistToggle，需要安装ctags(apt-get install ctags)
- kien/ctrlp.vim    &nbsp;&nbsp;    款速查找文件  ctrl+p, ctrl+j/k 上下选择
- Shougo/neocomplcache    &nbsp;&nbsp;    自动补全
- matchit.zip    &nbsp;&nbsp;    使%不仅能匹配括号，还能匹配html tags
- fencview.vim    &nbsp;&nbsp;    自动检测文件编码和手动选择文件编码格式  :FencAutoDectect  :FencView
