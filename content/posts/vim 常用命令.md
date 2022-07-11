---
title: "vim 常用命令"
date: 2022-07-11
categories:
- 其它
thumbnailImagePosition: right
thumbnailImage: img/main/th-7.jpeg
---

<!--more-->

{{< toc >}}

# 命令模式

- 删除当前行： (dd)
- 移动到行头： (0)
- 光标定位到第一行的行首： (gg)
- 光标定位到最后一行的行首:  (G)
- 删除光标所在行到文本的结束: (dG)
- 删除光标到本行结尾: (d$) 
- 复制当前行： (yy)
- 在光标后开始复制: (p)
- 撤销刚才的操作: (u)
- 恢复撤销操作: (ctrl + r)


# 输入模式


# 底线命令模式

- 清空文件内容：（%d）
- 设置文档的编码格式： (set enc=utf8)
- 显示行号： (set nu)    
- 取消行号： (set nonu)
- 定位到文档第n行： (n)
- 删除多行(从n1行到n2行)文本： (n1 ,n2d)
- 在当前行进行文本替换,将str1替换成str2： (s/str1/str2/g)
- 在n1行到n2行之间区域进行替换：(n1,n2s/str1/str2/g)