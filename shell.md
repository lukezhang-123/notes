

### 打印日志，右对齐输出ok

```
printf "%20s\n" "Your Text"

printf "%${COLUMNS}s\n" "Your Text"

MSG="Hello World"
COLS=$(tput cols) # Get terminal width
let COL_POS=COLS-${#MSG} # Calculate the position for the status
echo -n "$MSG"
printf "%${COL_POS}s" "[OK]"
echo "" # Add a final newline
```

### 暂停history记录

```
记忆，设置history off，+o理解为设置为off，-o是取消off
set +o history
set -o history

```

### 记录命令到history

```
执行注释行，记录到history, 需要 ctrl d，或者exit正常退出
# your command
立即追加当前session到history
history -a

新session读取history
history -r
打印出指定序号的history命令，上翻第一个，方便编辑
!NUMBER:p

```

### 检查rsa公私钥匹配，私钥生成公钥，对比现在公钥内容

```
ssh-keygen -y -e -f <private key>
```

### 计算ssh公钥sha256

```
ssh-keygen -lf ~/.ssh/id_rsa.pub
```

### screen

```
screen -S session_name
screen -ls
screen -r session_name/pid from screen ls
screen -dr session_name  # force attach session
screen -D -RR session_name
echo $STY  # check inside screen

ctrl a d  deattach session, not exit screen session
```

### date

```
指定时区的时间->时间戳
echo $(TZ=Asian/Dubai date -d '2025-11-07 06:00:00' +%s)
指定时区的时间戳->时间
echo $(TZ=Asian/Dubai date -d @1762257000 +'%Y-%m-%d %H:%M:%S')
```

### ls

```
ll --block-size=M path
```

### 查看当前bash类型，默认shell

```
echo $SHELL
ps -p $$
echo $0
```

### 查看分区 格式ext4 uuid

```lsblk -f```

