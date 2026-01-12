# LAM WebRTC + HTTPS 快速修复

适用场景：页面能打开，但 WebRTC 失败（ICE failed、输入框消失、对话不可用）。

## 1) 配好 coturn 公网映射
编辑 `/etc/coturn/turnserver.conf`：

```
listening-port=3478
listening-ip=0.0.0.0

relay-ip=172.16.0.69
external-ip=139.224.57.224/172.16.0.69

fingerprint
lt-cred-mech
user=admin:admin
realm=turn.open-avatar-chat.turnserver

min-port=49152
max-port=65535
```

重启 coturn：

```
sudo systemctl restart coturn || sudo service coturn restart
```

## 2) 给 LAM 下发 TURN 配置
在 `config/chat_with_lam.yaml` 的 `chat_engine` 下添加：

```
turn_config:
  turn_provider: "turn_server"
  urls:
    - "turn:139.224.57.224:3478?transport=udp"
    - "turn:139.224.57.224:3478?transport=tcp"
  username: "admin"
  credential: "admin"
```

重启服务：

```
bash run_docker_cuda128.sh --config config/chat_with_lam.yaml
```

## 3) 必须用 HTTPS 访问
只要配置了 `cert_file/cert_key`，服务就会以 HTTPS 启动，HTTP 会返回空响应。

访问：

```
https://139.224.57.224:8282/ui/index.html
```

浏览器提示证书不可信时，选择“高级”并继续访问。

## 4) 验证 ICE 配置是否下发

```
curl -vk https://127.0.0.1:8282/openavatarchat/initconfig
```

JSON 里应包含 `rtc_configuration.iceServers`。
