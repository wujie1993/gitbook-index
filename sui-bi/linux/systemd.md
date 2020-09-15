# Systemd

### Q&A

如何在systemd中source /etc/profile？

```text
[Unit]
Description=Run command with source /etc/profile

[Service]
Type=oneshot
RemainAfterExit=true
ExecStart=/bin/bash -c 'source /etc/profile && /bin/sh /root/test.sh'

[Install]
WantedBy=multi-user.target
```



