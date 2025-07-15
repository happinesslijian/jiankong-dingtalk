### 进项目根目录
```
cd jiankong-dingtalk
```
### 拉依赖
```
go mod tidy
```
### 编译
```
CGO_ENABLED=0 go build -ldflags "-s -w" -o jiankong-dingtalk .
```
### 本地运行
```
export DING_WEBHOOK=https://oapi.dingtalk.com/robot/send?access_token=8dcbbd554a6yyyyyyyyyyyy51d5967exxxxxxxxxxxx7d38e27bea14
export DING_SECRET=SEC9fc1a5ef162dd30bexxxxxxxx759eda5a93d7xxxxxxxxxb4eb81d0bb5
./jiankong-dingtalk
```

### 压测方式
```
sudo apt-get install stress
# 4 核全部跑满 100 秒，Ctrl+C 随时停止
stress --cpu 4 --timeout 100s
# 申请 8 GB 内存 100 秒后自动释放
stress --vm 1 --vm-bytes 8G --timeout 100s
./jiankong-dingtalk
```
### systemd管理
#### 二进制放置
```
sudo cp jiankong-dingtalk /usr/local/bin/
sudo chmod +x /usr/local/bin/jiankong-dingtalk
```

#### systemd 单元文件
```
sudo tee /etc/systemd/system/jiankong-dingtalk.service >/dev/null <<'EOF'
[Unit]
Description=Push server status to DingTalk once
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
# 关键环境变量
Environment=DING_WEBHOOK=https://oapi.dingtalk.com/robot/send?access_token=YOUR_TOKEN
Environment=DING_SECRET=YOUR_SECRET
Environment=CPU_THRESHOLD=85
Environment=MEM_THRESHOLD=85
Environment=DISK_THRESHOLD=85
Environment=REPORT_TIME=09:30
# 执行路径
ExecStart=/usr/local/bin/jiankong-dingtalk 
EOF
```
#### systemd 定时器
```
sudo tee /etc/systemd/system/jiankong-dingtalk.timer  >/dev/null <<'EOF'
[Unit]
Description=Daily jiankong to DingTalk

[Timer]
OnCalendar=*-*-* 09:30:00
Persistent=true

[Install]
WantedBy=timers.target
EOF
```

### 启用并立即测试
```
sudo systemctl daemon-reload
sudo systemctl enable --now jiankong-dingtalk.timer
# 手动验证
sudo systemctl start jiankong-dingtalk.service
journalctl -u jiankong-dingtalk --no-pager -e
```

## 效果图
待更新
