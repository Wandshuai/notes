## https://github.com/openjdk/jdk

git clone 速度太慢,前提是配置好代理

```
 git config --global http.proxy 'socks5://127.0.0.1:1080'

 git config --global https.proxy 'socks5://127.0.0.1:1080'
```

对于使用git@协议的，可以配置socks5代理，在~/.ssh/config 文件后面添加几行，没有可以新建一个

Host github.com

ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p



## https://hg.openjdk.java.net/jdk-updates/jdk11u/

hg  clone 会经常中断，可以使用下面命令先拉一个早的commit，然后慢慢update

```
hg clone https://hg.openjdk.java.net/jdk-updates/jdk11u/ -r 1000
```

写一个脚本慢慢update

```
hg pull -r 40000
hg pull -r 41000
hg pull -r 42000
hg pull -r 43330
hg pull -r 44000
hg pull -r 45000
hg pull -r 46000
hg pull -r 47000
hg pull -r 48000
hg pull -r 49000
```

## Clion 配置opendjk 11 

下载完代码后直接configure ,确保bootjdk配置正确

```
bash configure --disable-warnings-as-errors
cd build/linux-x86_64-normal-server-release/
make images
```

上面是正常编译流程

Clion打开openjdk工程需要配置`Compilation database`，对于jdk11可以之间make生成

```
make compile-commands
```

或者

```
make compile-commands-hotspot
```

## Clion 配置opendjk 8

需要先安装compiledb

```
# pip3 install compiledb
```

然后修改源码路径下的make/Main.gmk,注释掉77行。

```
 71 # Setup a rule for SPEC file that fails if executed. This check makes sure the configuration
 72 # is up to date after changes to configure
 73 $(SPEC): $(wildcard $(SRC_ROOT)/common/autoconf/*)
 74         @$(ECHO) "ERROR: $(SPEC) is not up to date."
 75         @$(ECHO) "Please rerun configure! Easiest way to do this is by running"
 76         @$(ECHO) "'make reconfigure'."
 77         # @if test "x$(IGNORE_OLD_CONFIG)" != "xtrue"; then exit 1; fi
 78 
 79 start-make: $(SPEC)
```

顺便把警告当错误的选项也注释掉，hotspot/make/linux/makefiles/gcc.make:204

```
196 # Keep temporary files (.ii, .s)
197 ifdef NEED_ASM
198   CFLAGS += -save-temps
199 else
200   CFLAGS += -pipe
201 endif
202 
203 # Compiler warnings are treated as errors
204 # WARNINGS_ARE_ERRORS = -Werror
205 
206 ifeq ($(USE_CLANG), true)

```

然后进行编译

```
bash configure --enable-jfr --disable-zip-debug-info
cd build/linux-x86_64-normal-server-release/
compiledb make images
```

等待生成`compile_commands.json` 文件，编译最后会提示几个错误，不过没影响。