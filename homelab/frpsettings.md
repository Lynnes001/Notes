


```
# frps.toml

bindPort = 7000

# [auth]
auth.token = "<your_token>"

# [dashboard]
webServer.addr = "0.0.0.0"
webServer.port = 7500
webServer.user = "<user>"
webServer.password = "<pwd>"

# [log]
log_file = "/home/frp/frps.log"
log_level = "info"
log_max_days = 3
```

```config
# /lib/systemd/system/frps.service

[Unit]
Description=frps
After=network.target
Wants=network.target

[Service]
Restart=on-failure
RestartSec=5
ExecStart=~/frp/frps -c ~/frp/frps.toml

[Install]
WantedBy=multi-user.target
```
