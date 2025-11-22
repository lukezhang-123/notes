
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

