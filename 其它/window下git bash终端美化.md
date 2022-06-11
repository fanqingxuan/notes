### 其它
- 修改命令提示符
以下所有命令都需要在 Git Bash 中执行
```shell
vim /etc/profile.d/git-prompt.sh
```
如果不熟悉 vim，并且电脑上装了 VSCode，可以用以下命令打开文件
```shell
code /etc/profile.d/git-prompt.sh
```
将其修改为如下内容

	```shell
	if test -f /etc/profile.d/git-sdk.sh
	then
		TITLEPREFIX=SDK-${MSYSTEM#MINGW}
	else
		TITLEPREFIX=$MSYSTEM
	fi

	if test -f ~/.config/git/git-prompt.sh
	then
		. ~/.config/git/git-prompt.sh
	else
		PS1='\[\033]0;Bash\007\]'      # 窗口标题
		PS1="$PS1"'\n'                 # 换行
		PS1="$PS1"'\[\033[32;1m\]'     # 高亮绿色
		PS1="$PS1"'➜ '               # unicode 字符，右箭头
		PS1="$PS1"'\[\033[33;1m\]'     # 高亮黄色
		PS1="$PS1"'\W'                 # 当前目录
		if test -z "$WINELOADERNOEXEC"
		then
			GIT_EXEC_PATH="$(git --exec-path 2>/dev/null)"
			COMPLETION_PATH="${GIT_EXEC_PATH%/libexec/git-core}"
			COMPLETION_PATH="${COMPLETION_PATH%/lib/git-core}"
			COMPLETION_PATH="$COMPLETION_PATH/share/git/completion"
			if test -f "$COMPLETION_PATH/git-prompt.sh"
			then
				. "$COMPLETION_PATH/git-completion.bash"
				. "$COMPLETION_PATH/git-prompt.sh"
				PS1="$PS1"'\[\033[31m\]'   # 红色
				PS1="$PS1"'`__git_ps1`'    # git 插件
			fi
		fi
		PS1="$PS1"'\[\033[36m\] '      # 青色
	fi

	MSYS2_PS1="$PS1"
	```
效果如下：
![git_1.gif](http://www.fxjson.com/usr/uploads/2021/04/892709726.gif)

- 解决 Unicode 字符显示异常问题
 - [下载](https://raw.githubusercontent.com/powerline/fonts/master/DejaVuSansMono/DejaVu%20Sans%20Mono%20for%20Powerline.ttf)DejaVu Sans Mono 字体
 - 执行以下命令，会打开字体文件夹，将字体托进去会自动安装，然后将修改 Git Bash 的字体就能正常显示 Unicode 字符了。(start 是 cmd 所提供的命令)
 ```shell
  start c://Windows//Fonts
 ```

- 修改主题
	```shell
	vim ~/.minttyrc
	```
	内容如下:
	```shell
Font=DejaVu Sans Mono for Powerline
FontHeight=14
Transparency=low
FontSmoothing=default
Locale=C
Charset=UTF-8
Columns=88
Rows=26
OpaqueWhenFocused=no
Scrollbar=none
Language=zh_CN
ForegroundColour=131,148,150
BackgroundColour=0,43,54
CursorColour=220,130,71
BoldBlack=128,128,128
Red=255,64,40
BoldRed=255,128,64
Green=64,200,64
BoldGreen=64,255,64
Yellow=190,190,0
BoldYellow=255,255,64
Blue=0,128,255
BoldBlue=128,160,255
Magenta=211,54,130
BoldMagenta=255,128,255
Cyan=64,190,190
BoldCyan=128,255,255
White=200,200,200
BoldWhite=255,255,255
BellTaskbar=no
Term=xterm
FontWeight=400
FontIsBold=no
RightClickAction=paste
	```
效果如下:
![git_2.gif](http://www.fxjson.com/usr/uploads/2021/04/3002196454.gif)
- 重启git终端