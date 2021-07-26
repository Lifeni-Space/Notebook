---
name: missing-semester
title: The Missing Semester of Your CS Education 练习记录
create-date: 2021-03-28
date: 2021-07-06
description: 记录学习 The Missing Semester of Your CS Education 课程时做的一些练习，只包含部分自己觉得比较有意义的题。
license: CC-BY-SA-4.0
---

# The Missing Semester of Your CS Education 练习记录

## 前排提示

- 英文官网：https://missing.csail.mit.edu/ ，中文翻译：https://missing-semester-cn.github.io/
- 我用的是 WSL2 (Ubuntu-20.04)，下面的代码均在此环境上测试通过
- 文章中练习题采用中文翻译版本，部分练习只截取前面第一句作为标题，其余部分在正文

## [课程概览与 shell](https://missing-semester-cn.github.io/2020/course-shell/)

### 5. 将以下内容一行一行地写入 semester 文件 ...

将以下内容一行一行地写入 semester 文件，第一行可能有点棘手，`#` 在 Bash 中表示注释，而 `!` 即使被双引号（`"`）包裹也具有特殊的含义。 单引号（`'`）则不一样，此处利用这一点解决输入问题。更多信息请参考 [Bash quoting 手册](https://www.gnu.org/software/bash/manual/html_node/Quoting.html)。

```bash
#!/bin/sh
curl --head --silent https://missing.csail.mit.edu
```

#### ✅ 解决方法

```bash
echo '#!/bin/sh' > /tmp/missing/semester
echo "curl --head --silent https://missing.csail.mit.edu" >> /tmp/missing/semester
```

包含 `!` 则需要用单引号而不是双引号。

### 9. 使用 `chmod` 命令改变权限 ...

使用 `chmod` 命令改变权限，使 `./semester` 能够成功执行，不要使用 `sh semester` 来执行该程序。

#### ✅ 解决方法

```bash
chmod 744 semester
```

744 对应 `rwxr-xr-x` 也就是先转换成二进制 `111101101` 再分成三个三位二进制数 `111` `101` `101`，最后转换成十进制也就是 744。

### 10. 使用 `|` 和 `>` ...

使用 `|` 和 `>` ，将 `semester` 文件输出的最后更改日期信息，写入 `/home` 目录下的 `last-modified.txt` 的文件中

#### ✅ 解决方法

```bash
ls -l semester | cut -d" " -f6,7,8 > ~/last-modified.txt
```

`cut` 命令把 `-rwxr--r-- 1 focal focal 61 Mar 28 18:36 semester` 按空格分割，选择第 6、7、8 段输出，也就是 `Mar 28 18:36`。

## [Shell 工具和脚本](https://missing-semester-cn.github.io/2020/shell-tools/)

### 3. 假设您有一个命令，它很少出错 ...

假设您有一个命令，它很少出错。因此为了在出错时能够对其进行调试，需要花费大量的时间重现错误并捕获输出。 编写一段 bash 脚本，运行如下的脚本直到它出错，将它的标准输出和标准错误流记录到文件，并在最后输出所有内容。加分项：报告脚本在失败前共运行了多少次。

```bash
#!/usr/bin/env bash

n=$(( RANDOM % 100 ))

if [[ n -eq 42 ]]; then
    echo "Something went wrong"
    >&2 echo "The error was using magic numbers"
    exit 1
fi

echo "Everything went according to plan"
```

#### ✅ 解决方法

```bash
#!/bin/bash

# test.sh 就是上面的测试文件
# test-out.log 是正常输出
# test-err.log 是错误输出

./test.sh 1>> ./test-out.log 2>> ./test-err.log

count=1

while [[ $? -eq 0 ]]; do
  let count++
  ./test.sh 1>> ./test-out.log 2>> ./test-err.log
done

echo "运行次数 $count"
```

`1>>` 代表标准输出流，`2>>` 代表错误输出流，`$?` 代表前一个命令的返回值，把变量 +1 需要在语句前加上 `let`。

### 4. 本节课我们讲解的 `find` 命令中的 `-exec` 参数非常强大 ...

本节课我们讲解的 `find` 命令中的 `-exec` 参数非常强大，它可以对我们查找的文件进行操作。但是，如果我们要对所有文件进行操作呢？例如创建一个 zip 压缩文件？我们已经知道，命令行可以从参数或标准输入接受输入。在用管道连接命令时，我们将标准输出和标准输入连接起来，但是有些命令，例如 `tar` 则需要从参数接受输入。这里我们可以使用 [`xargs`](https://www.man7.org/linux/man-pages/man1/xargs.1.html) 命令，它可以使用标准输入中的内容作为参数。 例如 `ls | xargs rm` 会删除当前目录中的所有文件。

您的任务是编写一个命令，它可以递归地查找文件夹中所有的 HTML 文件，并将它们压缩成 zip 文件。注意，即使文件名中包含空格，您的命令也应该能够正确执行（提示：查看 `xargs` 的参数 `-d`）

#### ✅ 解决方法（`tar` + `xargs`）

```bash
# 压缩
find . -name "*.html" | xargs -d "\n" tar -czf t.tar.gz

# 解压
mkdir ex
tar -xzf t.tar.gz -C ex
```

`tar` 命令的 `-c` 代表创建压缩包，`-x` 代表解压缩包，`-z` 代表用 GZip 处理，`-f` 后面跟要操作的文件名。

#### ✅ 解决方法（`zip`）

```bash
# 压缩
find . -name "*.html" -print | zip t.zip -@
```

```bash
# 查看压缩包
unzip -l t.zip

# 输出
Archive:  t.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
        7  2021-04-04 19:32   中文.html
        4  2021-04-04 19:31   are you ok.html
        6  2021-04-04 19:31   hi.html
        4  2021-04-04 19:32   folder/num.html
---------                     -------
       21                     4 files
```

### 5. 编写一个命令或脚本递归的查找文件夹中最近使用的文件 ...

编写一个命令或脚本递归的查找文件夹中最近使用的文件。更通用的做法，你可以按照最近的使用时间列出文件吗？

#### ✅ 解决方法

```bash
find . -type f | xargs ls -lt
```

`find` 命令查找所有文件，`xargs` 可以把前面的输出转换成参数传给后面的命令，`ls -t` 负责按最近修改时间显示。

## [编辑器 (Vim)](https://missing-semester-cn.github.io/2020/editors/)

> 这节课的练习都是实践的形式，没有标准答案 🧐

## [数据整理](https://missing-semester-cn.github.io/2020/data-wrangling/)

TODO

## [命令行环境](https://missing-semester-cn.github.io/2020/command-line/)

TODO

## [版本控制(Git)](https://missing-semester-cn.github.io/2020/version-control/)

TODO

## [调试及性能分析](https://missing-semester-cn.github.io/2020/debugging-profiling/)

TODO

## [元编程](https://missing-semester-cn.github.io/2020/metaprogramming/)

TODO

## [安全和密码学](https://missing-semester-cn.github.io/2020/security/)

TODO
