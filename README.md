# homelab-devops-platform

---
Стек технологий:

- k3s 
- MetalLB
- Docker-registry
- Gieta
- Drone CI
- Prometheus
- Grafana
- Elasticsearch

---

## k3s кластер

Чтобы k3s функционировала нам необходимо включить на сервере cgroup контроллеры 

1. cgroup_enable=cpuset (возможность привязывать контейнеры к CPU-ядрам)
2. cgroup_enable=memory (отвечает за лимиты и учёт потребления RAM)
3. cgroup_memory=1 (использование v1 или смешанного режима памяти)
```
sudo nano /etc/default/grub
``` 
Затем ребутаем 
```
sudo reboot
```

На всякий случай напоминаю что `curl` не является встроенным инструментом
```
sudo apt install curl
```

Скачиваем с зеркала gitHub потому что огранечения в Российском регионе
```
curl -L https://github.com/k3s-io/k3s/releases/download/v1.30.4+k3s1/k3s-arm64 -o /usr/local/bin/
sudo chmod +x /usr/local/bin/k3s
```

Ручками запускаем сервис
```
sudo tee /etc/systemd/system/k3s.service >/dev/null <<'EOF'
[Unit]
Description=k3s
After=network.target
[Service]
Type=notify
ExecStart=/usr/local/bin/k3s server
TimeoutStartSec=0
Restart=always
RestartSec=10
KillMode=mixed
LimitNOFILE=65536
LimitNPROC=infinity
TasksMax=infinity
OOMScoreAdjust=-1000
[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable --now k3s
```

Проверим что всё стоит
```
sudo systemctl status k3s
sudo k3s kubectl get nodes
```

Копируем токен для подключения к k3s кластеру
```
sudo cat /var/lib/rancher/k3s/server/node-token
```

На 2 сервере ставим 
```
curl -L https://github.com/k3s-io/k3s/releases/latest/download/k3s-arm64 -o k3s
chmod +x k3s
sudo mv k3s /usr/local/bin/
```

```
sudo k3s agent --server https://<ip_or_hostname_of_existing_node>:6443 --token <server_token>
```

---

