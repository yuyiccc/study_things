### 将程序挂起到后台运行：
- nohup 和 &
   - nohup 可以在终端关闭时进程不被kill掉。
   - & 可以使得在终端使用Ctrl+C时进程不被打断。
   - nohup和&一起用可以使得程序在后台运行而不被这两种情况打断。例子1.nohup python a.py & 2.nohup sh a.sh &
### 输出重定向：
- 这里包括了标准输出（stdout）和错误输出（stderr）的输出重定向，标准输出代号为1，错误输出的代号为2。

- commond 1> out.log 等价于commond > out.log 都只会将标准输出写入到out.log而错误输出会打印到屏幕上。
- commond 2> out.log 则只会将错误输出写入到out.log而标准输出打印到屏幕上。
- commond > out.log 2>&1 等价于 commond 1> out.log 2> out.log。也就是说&1表示前一个文件的名字
- 例子: python a.py > out.log 2>&1 这条语句就可以将a.py的标准输出和错误输出都写入到out.log

### 结合挂起程序到后台运行和输出重定向
- 例子：nohup python a.py > out.log 2>&1 &
