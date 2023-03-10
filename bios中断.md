# BIOS int 10H中断介绍

`int 10H` 是BIOS对屏幕及显示器所提供的服务程序。使用`int 10H`中断服务程序时，先要进行一些简单的配置，比如指定功能号和子功能号。其中寄存器AH表示的就是功能号，而其他寄存器的详细说明，参考表格中对应功能的文字，当一切设定好之后再调用 `int 10H`。下面是可供参考的表格：BIOS int 10H中断介绍

![BIOS int 10H中断介绍(1)](.\picture\BIOS int 10H中断介绍(1).png)

![BIOS int 10H中断介绍(2)](.\picture\BIOS int 10H中断介绍(2).png)

![BIOS int 10H中断介绍(3)](.\picture\BIOS int 10H中断介绍(3).png)

![BIOS int 10H中断介绍(4)](.\picture\BIOS int 10H中断介绍(4).png)

![BIOS int 10H中断介绍(5)](.\picture\BIOS int 10H中断介绍(5).png)



`AH=00H`是用来设定显示模式的，AL 寄存器表示欲设定的模式：

| AL   | 文字/图形 | 分辨率  | 颜色 |
| ---- | --------- | ------- | ---- |
| 00H  | 文字      | 40*25   | 2色  |
| 01H  | 文字      | 40*25   | 16色 |
| 02H  | 文字      | 80*25   | 2色  |
| 03H  | 文字      | 80*25   | 16色 |
| 04H  | 图形      | 320*200 | 2色  |
| 05H  | 图形      | 320*200 | 4色  |
| 06H  | 图形      | 640*200 | 2色  |



`AH=01H`，你可以把光标想成一个小的矩形，平时这个矩形扁平位于某字底部，但藉由此功能可以改变其大小与位置。光标起始处与终止处分别由 CL 与 CH 的 0 到 4 位表示，而 CH 的第 7 位必须是 0，第 5、6 位表示光标属性：

| 位6  | 位5  | 属性     |
| ---- | ---- | -------- |
| 0    | 0    | 正常     |
| 0    | 1    | 隐形     |
| 1    | 0    | erratic  |
| 1    | 1    | 闪烁缓慢 |



`AH=02H`，此功能是设定光标位置，位置用 DH、DL 表示，DH 表示列号，DL 表示行号。由左至右称之为『列』，屏幕最上面一列为第零列，紧靠第零列的下一列称为第一列……；由上而下称之为『行』，屏幕最左边一行称之为第零行，紧靠第零行右边的一行为第一行。故最左边，最上面的位置为 DH=0 且 DL=0；最左边第二列，DH=1，DL=0。如果是文字模式时，BH 为欲改变光标位置的显示页，如果是图形模式，BH 要设为 0。

在文字模式下，字符的位置类似数学直角坐标系的坐标，但是 Y 轴方向相反，Y 轴是以屏幕最上面为零，越下面越大，直到 24 为止，存于 DH 内。X 轴和直角座标系相同，越右边越大，存于 DL 内，其最大值视显示模式而变。

`AH=03H`，用于读取光标位置，在这个中断服务程序返回时，会将光标的行列信息存储在DX里面，CX中存储光标的开始行和结束行。

`AH=04H`，此功能是探测光笔的位置，似乎只有 CGA 卡有接上光笔？？

`AH=05H`，这个功能是把指定的显示页显示于屏幕上，欲显示的显示页由AL寄存器指定。此功能只能在文字模式下才能发生作用。

`AH=06H/07H`，这个服务程序的作用是把某一个设定好的矩形区域内的文字向上或向下移动。先说明向上移动，即AH=06H。当此服务程序工作时，会使矩形区域的文字向上移动，而矩形区域底端移进空格列。向上移动的列数存入 AL 中 ( 如果 AL 为零，表示使矩形区域的所有列均向上移 )，底端移入空格列的属性存于 BH，矩形区域是藉由 CX、DX 来设定左上角与右下角的座标，左上角的行与列分别由 CL、CH 设定，右下角的行与列由 DL、DH 设定。AH=07H和 AH=06H相似，只是卷动方像不同而已。

`AH=08H`，这个服务程序是用来取得光标所在位置的字符及属性，调用前，BH表示想要读取的显示页，返回时，AL为该位置的ASCII字符，AH为其属性，有关属性的说明，请参考下面的注1。

`AH=09H`，这个功能是在光标位置显示字符，所要显示字符的 ASCII 码存于 AL 寄存器，字符重复次数存于 CX 寄存器，显示页存于 BH 寄存器，属性存于 BL 寄存器，其属性使用与AH=08H一样。

`AH=0AH`，这个功能和AH=09H一样，差别在 AH=0AH 只能写入一个字符，而且不能改变字符属性。

`AH=0BH`，这个服务程序是选择调色盘。显示模式 5 是 320*200 的图形模式，最多可以显示 4 种颜色，这四种颜色的意思是最多可以『同时』显示一种背景色及三种前景色，而这三种前景色有两种方式可供选择，因此事实上，在显示模式 5 有两种调色盘可供选择。就好像您去买 12 种颜色的水彩，但可在调色盘上以任意比例搭配出许多种颜色。

调色盘 0 的三色是红、黄、绿；调色盘 1 的三色是青、紫红、白。背景色有 16 六种可供选择，这 16 种就是下面注1的 16 色。调用此中断时，先决定要设定背景色抑或调色盘：

要设定背景色时，则使BH为 0，再使BL之数值为0到0fh之间表示注1的16色之一。

要设定调色盘时，则使BH为1。再设定BL为零或一表示选择那一种调色盘。

背景色只有在前景色为 0 时才会显现出来。

`AH=0CH`，是在绘图模式中显示一点 ( 也就是写入点像，write graphics pixel )，而 AH=0DH则是读取点像 ( read graphics pixel )。
写入时，要写入位置X坐标存于CX寄存器，Y坐标存于DX寄存器，颜色存于AL寄存器。和文字模式相同，萤光幕上的Y坐标是最上面一列为零，越下面越大，X坐标则和数学的定义相同。CX、DX、AL 值之范围与显示模式有关：

| 显示模式 | X坐标   | Y坐标   | 颜色  |
| -------- | ------- | ------- | ----- |
| 4        | 0 ~ 319 | 0 ~ 199 | 0 、1 |
| 5        | 0 ~ 319 | 0 ~ 199 | 0 ~ 3 |
| 6        | 0 ~ 639 | 0 ~ 199 | 0 、1 |

`AH=0DH`则是读取某一位置的点像，您必须指定CX、DX，而INT 10H会传回该位置点像的颜色。

`AH=0EH`，这个子程序是使显示器像打字机一样的显示字符来，在前面用AH=09H和 AH=0AH都可以在萤光幕上显示字符，但是这两种方式显示字符之后，光标位置并不移动，而AH=0EH则会使光标位置移动，每显示一个字符，光标会往右移一格，假如已经到最右边了，则光标会移到最左边并移到下一列，假如已经移到最下面一列的最右边，则屏幕会向上卷动。

AL寄存器存储要显示的字符，BH为目前的显示页，如果是在图形模式，则BH须设为 0，假如是在图形模式下，也可以设定 BL 来表示文字的颜色，文字模式下的 BL 则无功能。

`AH=0FH`，这个服务程序是得到目前的显示模式，调用前只需使AH设为0fh，当由INT 10H返回时，显示模式存储在AL寄存器(参考`AH=00H`的显示模式表)，目前的显示页存于BH寄存器，总字符行数存存储在AH寄存器。

**注:**

​     所谓属性是指字符的颜色、背景颜色、是否闪烁、有没有底线等性质。在彩色显示卡 (CGA/EGA/VGA等)的文字模式中，颜色是用4个位表示，故可以表现出16种颜色，如下表：

| 二进制数 | 颜色               | 二进制数 | 颜色                      |
| -------- | ------------------ | -------- | ------------------------- |
| 0000     | 黑色（black）      | 1000     | 灰色（gray）              |
| 0001     | 蓝色（blue）       | 1001     | 淡蓝色（light blue）      |
| 0010     | 绿色（green）      | 1010     | 淡绿色（light green）     |
| 0011     | 青色（cyan）       | 1011     | 淡青色（light cyan）      |
| 0100     | 红色（red）        | 1100     | 淡红色（light red）       |
| 0101     | 紫红色（magenta）  | 1101     | 淡紫红色（light magenta） |
| 0110     | 棕色（brown）      | 1110     | 黄色（yellow）            |
| 0111     | 银色（light gray） | 1111     | 白色（white）             |



在彩色显示器里，如CGA、EGA、VGA等，常用一个字节(8个位)来表示文字颜色和背景颜色，通常以第0～3位表示文字本身颜色；第4～6位表示背景颜色，背景颜色只有上表左栏的8种而已；第7个位，表示是否闪烁，0表示不闪烁，1表示闪烁。

但是在单色显示器里，如MDA和Hercules卡中，这些颜色表并无意义，所以属性解释方式不同，请看下表：

| 数值 | 属性                 |
| ---- | -------------------- |
| 00H  | 空格，不显示任何数据 |
| 77H  | 显示白色方块         |
| 07H  | 正常的黑底白字       |
| 70H  | 反白的白底黑子       |
| 01H  | 加底线               |

