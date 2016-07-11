# Linux

## 常用命令

0.mkdir -p 一次可以建立多个目录

复制、删除、移动：

1.复制：cp -a（带文件特性一起复制）, cp -i（覆盖时询问）, cp -r（目录） 

2.删除：rm -f（忽略警告） rm -r（递归删除）

3.移动: mv 文件1 文件2 目录，mv 目录名 新目录名(目录重命名)

磁盘与目录的容量：

4.磁盘：df, df -i（以inode数量显示）, df -h(MB,GB), df -m(MB)

5.目录: du -sm(列出总量,以MB显示)

连接：

6.ln(硬链接)

7.ln -s（软链接）

压缩/解压缩:

7.压缩: gzip -v, bzip2 -z
8.解压缩: gzip -d, bzip2 -d

tar打包：

8.打包为gzip文件：tar -zcvf 新建的文件名 旧文件
解压缩：tar -zxvf 文件

9.打包为bzip2：tar -jcvf 新建的文件名 旧文件
解压缩：tar -jxvf 文件

备份目录：

10.dump -0j -f /tmp/etc.dump.bz2 /etc

11.恢复：restore -t -f /tmp/etc.dump.bz2

## 管道命令

12.选取：cut -d ' ' -f 1（用于分隔字符） cut -c 12-（字符范围，用于排列整齐的信息）

13.grep 'root'（查找root）grep -v 'root'（反向选择）-n（输出行号）-i（不区分大小写）

14.sort（默认） sort -t ':' -k 3

15.uniq: 仅列出一个显示

16.wc: 统计字数字符数 e.g. wc -l（仅列出行） cat /etc/man.config | wc

17.uniq（仅列出一个显示）

18.tee: 双向重定向 last | tee last.list | cut -d ' ' -f 1

### 字符转换命令

tr, col, join, paste, expand

19.tr: e.g. tr -d（删除指定字符）tr [a-z] [A-Z]小写转大写

20.col -x：将断行符^M（tab）转换成对等的空格

21.join -t ':' /etc/passwd /etc/shadow（连接）

22.paste（直接两行贴在一起）

23.expand -t（tab键转空格键）

24.切割命令split

split -b 300k /etc/termcap termcap(按文件大小切割)
split -l 10 - lsroot（按行切割）

25.xargs：很多命令不支持管道命令，可以通过xargs来提供该命令引用stdin之用。

管道命令：cut、grep、sort、wc、uniq、tee、tr、col、join、paste、expand、split、xargs

26.dmesg：列出内核信息

## 正则表达式

sed管道命令

27.sed -[nefr] 动作 (sed用于一整行的处理)

动作：a新增 c替换（整行替换） d删除 i插入 p打印（与-n安静模式搭配）

sed的替换功能（字符串的替换）：sed 's/被替换的字符串/新字符串/g '

28.格式化打印printf

29.awk（倾向于将一行分成数个“字段”来处理）

e.g. last -n 5 | awk '{print $1 "\t" $3}'

FS（目前的分隔字符，默认空格字符） NR（目前awk处理的是第几行） NF（每行拥有的字段总数） 

cat /etc/passwd | awk '{FS=":"} $3<10 {print $1 "\t" $3}'

例子：
cat pay.txt | awk 'NR==1 {printf "%10s %10s %10s %10s %10s\n",$1,$2,$3,$4,"total" } NR>=2 {total=$2+$3+$4;printf "%10s %10d %10d %10d %10.2f\n", $1, $2, $3, $4, total}'

### 数据流重定向

标准输入：0，<或<<
标准输出：1，>或>>
标准错误输出：2，2>或2>>

正确错误写入同一个文件：find /home -name .bashrc > list 2>&1

文件比较：

30.diff passwd.old passwd.new（按行来比较）
31.cmp passwd.old passwd.new（按字节来比较）

32.patch

diff制作补丁文件

diff -Naur passwd.old passwd.new

更新旧文件：patch -p0 < passwd.patch

### 查看

【less】
  功能：less命令可以对文件或其他输出进行分页显示，与more命令相似。退出按q。
  参数详解：
  -a：在当前屏幕显示最后
  -c：从顶部（从上到下）刷新屏幕，并显示文件内容。而不是通过底部滚动完成刷新;
  -f：强制打开文件，二进制文件显示时，不提示警告；
  -i：搜索时忽略大小写；除非搜索串中包含大写字母;
  -I：搜索时忽略大小写，除非搜索串中包含小写字母;
  -m：显示当前读取文件的百分比
  -M：显示当前读取文件的百分比、行号及总行数；
  -N：在每行前输出行号
  -p pattern:搜索日志文件中含有pattern的所有日志内容；
  -s：把连续多个空白行作为一个空白行显示
  -Q：在终端下不响铃
  扩展：
  U：向上
  J：向下
  g：跳到第一行
  G：跳到最后一行
  /pattern:搜索pattern
  q：退出less

