# 九、vim程序编辑器

所有的Linux发行版都会内置vi编辑器，所以学会vi编辑器是基础。此外vim是高级版vi，vim不但可以用不同颜色显示文字内容，还能够进行诸如shell脚本、C语言等程序编辑，可以视为一种程序编辑器。

vi共有三种模式，分别是一般命令模式（command mode）、编辑模式（insert mode）和命令行模式（command-line mode）。

- 一般命令模式（command mode）

以vi打开一个文件就直接进入一般命令模式，这是默认的模式。在这个模式中，可以使用上下左右来移动光标，可以使用删除字符、删除整行、复制、黏贴等功能。

- 编辑模式

在一般命令模式下按下i、I、o、O、a、A、r、R等任何一个字母后才会进入该模式，通常按下这些按键时，界面左下方会出现INSERT或REPLACE字样。要退回一般命令模式只需按esc即可。

- 命令行模式

在一般模式中。输入:、/、?三个中任意一个即可将光标移到最下面一行。在这个模式中可以查找数据、读取、保存、批量替换、退出vi、显示行号等。

## vi基础功能

### 第一部分：一般命令模式可用的功能按键说明

|移动光标的方法||
| ---- | ---- |
|h或向左箭头键（←）|光标向左移动一个字符|
|j或向下箭头键（↓）|光标向下移动一个字符|
|k或向左箭头键（↑）|光标向上移动一个字符|
|l或向左箭头键（↓）|光标向右移动一个字符|

若想多次移动可执行次数加方向，例如想向下移动30行，可以使用"30j"或者"30↓"的组合键来实现。



| 移动光标的方法  |                                                              |
| --------------- | ------------------------------------------------------------ |
| [ctrl] + [f]    | 屏幕向下移动一页，相当于pagedown按键                         |
| [ctrl] + [b]    | 屏幕向上移动一页，相当于pageup按键                           |
| [ctrl] + [d]    | 屏幕向下移动半页                                             |
| [ctrl] + [u]    | 屏幕向上移动半页                                             |
| +               | 光标移动到非空格的下一行                                     |
| -               | 光标移动到非空格的上一行                                     |
| n<space>        | 那个n表示数字，例如20，按下数字后再按空格键，光标会向右移动这一行的n各字符，例如20<space>则光标会向后移动20各字符距离 |
| 0或功能键[Home] | 这是数字0，移动到这一行的最前面字符处                        |
| $或功能键[End]  | 移动到这一行的最后面字符处                                   |
| H               | 光标移动到这个屏幕的最上方那一行的第一个字符                 |
| M               | 光标移动到这个屏幕的中央那一行的第一个字符                   |
| L               | 光标移动到这个屏幕的最下方那一行的第一个字符                 |
| G               | 移动到这个文件的最后一行                                     |
| nG              | n为数字，移动到这个文件的第n行                               |
| gg              | 移动到这个文件的第一行，相当于1G                             |
| n<Enter>        | n为数字，光标向下移动n行                                     |



| 查找与替换            |                                                              |
| --------------------- | ------------------------------------------------------------ |
| /word                 | 向光标之后查找字符串word                                     |
| ?word                 | 向光标之前查找字符串word                                     |
| n                     | 重复前一个查找操作，配合/word或者?word可继续查找该字符出     |
| N                     | 反向进行前一个查找操作                                       |
|                       | 使用/word配合n及N是非常有帮助的，可以让你重复的找到一些你查询的关键词 |
| :n1,n2s/word1/word2/g | n1和n2为数字，在第n1行与n2行之间查找字符串word1，并将该字符串替换为word2 |
| :1,$s/word1/word2/g   | 从第一行到最后一行查找字符串word1，并将其替换成word2         |
| :1,$s/word1/word2/gc  | 从第一行到最后一行查找字符串word1，并将其替换成word2，且在替换前显示提示字符给用户确认（confirm）是否需要替换 |



| 删除、复制与黏贴 |                                                              |
| ---------------- | ------------------------------------------------------------ |
| x与X             | 在一行当中，x为向后删除一个字符（相当于[del]按键），X为向前删除一个字符（相当于[Backspace]按键） |
| nx               | n为数字，连续向后删除n各字符                                 |
| dd               | 删除（剪切）光标所在的那一行                                 |
| ndd              | n为数字，删除（剪切）光标所在的向下n行，连同光标所在行在内20行 |
| d1G              | 删除（剪切）光标所在行到第一行的所有数据                     |
| dG               | 删除（剪切）光标所在行到最后一行的所有数据                   |
| d$               | 删除（剪切）光标所处位置到该行最后一个字符                   |
| d0               | 删除（剪切）光标所处位置到该行最前面一个字符                 |
| yy               | 复制光标所在的那一行                                         |
| nyy              | n为数字，复制光标所在的向下n行，连同光标所在行在内20行       |
| y1G              | 复制光标所在行到第一行的所有数据                             |
| yG               | 复制光标所在行到最后一行的所有数据                           |
| y0               | 复制光标所处位置到该行最前面一个字符                         |
| y$               | 复制光标所处位置到该行最后一个字符                           |
| p与P             | p为将已复制的数据在光标下一行黏贴，P则为上一行黏贴。         |
| J                | 将光标所在行与下一行数据结合成同一行                         |
| c                | 重复删除多个数据，例如向下删除20行，[10cj]                   |
| u                | 恢复前一个操作，撤销的意思                                   |
| [Ctrl]+r         | 重做上一个操作，与撤销相对                                   |
| .                | 重复前一个操作的意思                                         |



### 第二部分：一般命令切换到编辑模式的可用按键说明

| 进入插入或替换的编辑模式 |                                                              |
| ------------------------ | ------------------------------------------------------------ |
| i与I（insert）           | 进入插入模式。i为从目前光标所在位置插入；I为在目前光标所在行的第一个非空格处开始插入 |
| a与A（append）           | 进入插入模式。a为从目前光标所在位置的下一个字符处开始插入；A为从光标所在行的最后一个字符处开始插入 |
| o与O                     | 进入插入模式。o为在目前光标所在的下一行处插入新的一行；O为在目前光标所在处的上一行插入新的一行 |
| r与R                     | 进入替换模式。r为只会替换光标所在的那一个字符一次；R会一直替换光标所在的文字，知道按下esc为止 |
| [Esc]                    | 退出编辑模式，回到一般命令模式                               |



### 第三部分：一般命令切换到命令行的可用按键说明

| 命令行模式的保存、退出等命令 |                                                              |
| ---------------------------- | ------------------------------------------------------------ |
| :w                           | 将编辑的数据写入硬盘文件中                                   |
| :w!                          | 若文件属性为只读时，强制写入该文件。不过到底能不能写入，还是跟你对该文件的文件权限有关 |
| :q                           | 退出vi                                                       |
| :q!                          | 若曾修改过文件，又不想保存，使用!为强制退出不保存            |
|                              | 注意，那个感叹号常常具有强制的意思                           |
| :wq                          | 保存并退出，若为:wq!则为强制保存并退出                       |
| ZZ                           | 这是大写的ZZ，若文件没有修改，则不保存退出，若文件已经被修改过，则保存退出 |
| :w [filename]                | 将编辑的数据保存成另一个文件，类似另存为                     |
| :r [filename]                | 在编辑的数据中，读入另一个文件的数据，即将filename这个文件内容加到光标所在行的后面 |
| :n1,n2 w [filename]          | 将n1到n2行的内容保存为filename这个文件                       |
| :! command                   | 暂时退出vi到命令模式下执行command的显示结果。例如【:! ls /home】即可在vi当中查看/home下面的文件信息 |
| :set nu                      | 显示行号                                                     |
| :set nonu                    | 取消显示行号                                                 |

## vim额外功能

### 可视区块

前面我们提到简单的vi操作过程中，几乎都是以行为单位的操作，若我想要搞定的是一个区块（也就是以列为操作单位）范围呢？举例来说，向下面这种格式的文件：

```
192.168.1.1    host1.class.net
192.168.1.2    host2.class.net
192.168.1.3    host3.class.net
192.168.1.4    host4.class.net
```

假设我只想要将host1、host2、host3、host4这一列复制出来， 并且追加到最后一列呢。于是vim可以做到，那就是使用科室区块（Visual Block）。当我们按下v、V或者[Ctrl]+v时，这时候光标移动过的地方会出现反白，具体的按键意义如下：

| 可是区块的按键意义 |                                      |
| ------------------ | ------------------------------------ |
| v                  | 字符选择，会将光标经过的地方反白选择 |
| V                  | 行选择，会将光标经过的行反白选择     |
| [Ctrl]+v           | 可视区块，可以用矩形的方式选择数据   |
| y                  | 将反白的地方复制起来                 |
| d                  | 将反白的地方删除                     |
| p                  | 将刚刚复制的区块，在光标所在处黏贴   |



### 多文件编辑

假设我想同时打开两个文件，来操作，并复制文件1的内容到文件2，这是就需要同时打开多个文件。可使用 `vim file1 file2`命令同时打开多个文件。打开后切换文件的命令如下：

| 多文件编辑的按键 |                               |
| ---------------- | ----------------------------- |
| :n               | 编辑下一个文件                |
| :N               | 编辑上一个文件                |
| :files           | 列出目前这个vim开启的所有文件 |



### 多窗口功能

类似上下分屏的功能，可分屏显示同一个文件，也可显示不同的文件。具体命令如下：

| 多窗口情况下的按键功能     |                                                              |
| -------------------------- | ------------------------------------------------------------ |
| :sp [filename]             | 打开一个新窗口，如哦有filename，表示新窗口打开或创建一个文件，否则表示两个窗口显示同一个文件的内容 |
| [Ctrl]+w+j或者[Ctrl]+w+↓   | 按键的按法：按住[Ctrl]，在按下w，松开[Ctrl]和w，再按下j或↓，光标可移动到下方的窗口 |
| [Ctrl]+w+k或者[Ctrl]+w+上  | 光标可移动到上方的窗口                                       |
| [Ctrl]+w+q或者:q或者:close | 退出当前窗口                                                 |



### vim的关键词补全功能

vim补齐功能大致有下面几个：

| 组合键               | 补齐的内容                                                   |
| -------------------- | ------------------------------------------------------------ |
| [Ctrl]+x -> [Ctrl]+n | 通过目前正在编辑的这个【文件的内容文字】作为关键词，给予补齐 |
| [Ctrl]+x -> [Ctrl]+f | 以当前目录内的【文件名】作为关键词，给予补齐                 |
| [Ctrl]+x -> [Ctrl]+o | 以扩展名作为语法补充，以vim内置的关键词，给予补齐            |



### vim环境设置与记录：~/.vimrc、~/.viminfo

我们每次vim进入到一个文件时，光变会在上一次退出来的位置。这是因为我们的vim会主动将你曾经做过的操作记录下来，好让你下次可以轻松地作业，这个记录操作的文件就是`~/.viminfo`

此外，每个Linux发行版对vim的默认环境都不太一样。整体的环境参数配置文件在`/etc/vimrc`，不建议修改。但可以修改自己home目录下的配置文件`~/.vimrc`默认不存在，可以自行创建。具体的参数配置如下：

| vim的环境设置参数                  |                                                              |
| ---------------------------------- | ------------------------------------------------------------ |
| :set no与:set nonu                 | 设置与取消行号                                               |
| :set hlsearch与:set nohlsearch     | hlsearch就是highlight search，就是设置是否将查找的字符串反白的设置，默认是hlsearch |
| :set autoindent与:set noautoindent | 是否自动缩进                                                 |
| :set backup                        | 是否自动保存备份文件？一般是nobackup，如果设置backup的话，那么当你修改任何一个文件时，源文件会被另存成一个文件名为filename~的文件。 |
| :set ruler                         | 右下角的一些状态栏说明                                       |
| :set showmode                      | 是否显示--INSERT--之类的的字眼在左下角的状态栏               |
| :set backspace=(012)               | 一般来说，如果我们按下i进入编辑模式后，可以利用退格键（Backspace）来删除任意字符。但是，某些Linux发行版则不允许。此时，如果backspace为2就是可以删除任意值；为0或1，仅可以删除刚刚输入的字符，无法删除原本就已经存在的文字 |
| :set all                           | 显示目前所有的环境参数设置值                                 |
| :set                               | 显示与系统默认值不同的设置参数，一般来说就是你有自行变动过的设置参数 |
| :syntax on与:syntax off            | 是否一句程序相关语法显示不同颜色？                           |
| :set bg=dark与:set bg=light        | 可用以显示不同的颜色色调，默认时light                        |



## 其他vim使用注意事项

### 中文编码的问题

若打开一个文件乱码，则需要考虑的东西有很多。比如：

1. Linux系统默认支持的语系数据：这与/etc/locale.conf有关
2. 终端（bash）的语系，这与LANG、LC_ALL这几个环境变量有关
3. 文件本身的编码
4. 打开终端的软件，例如在GNOME下面的窗口界面

事实上最重要的时上面的第三点和第四点，只要这两点的编码一致，一般就不会乱码。

### DOS与Linux的换行符

DOS的换行符为^M$，我们称为CR与LF；Linux的换行符为$(LF)。可以通过`dos2unix`和`unix2dos`两个命令进行换行符的转换。

```shell
[root@localhost ~]# dos2unix [-kn] file [newfile]
[root@localhost ~]# unix2dos [-kn] file [newfile]
选项与参数：
-k：保留该文件原本的mtime时间格式，
-n：保留原本的旧文件，将转换后的内容输出到新文件，例如：dos2unix -n old new
```
