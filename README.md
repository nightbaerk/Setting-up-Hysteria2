# 🚀 Полная инструкция по настройке Hysteria2

## 📋 Содержание
1. [Создание домена на DuckDNS](#1-создание-домена-на-duckdns)
2. [Подготовка сервера](#2-подготовка-сервера)
3. [Получение SSL сертификата](#3-получение-ssl-сертификата)
4. [Установка Hysteria2](#4-установка-hysteria2)
5. [Настройка конфигурации](#5-настройка-конфигурации)
6. [Создание пользователей](#6-создание-пользователей)
7. [Настройка прав доступа](#7-настройка-прав-доступа)
8. [Запуск сервера](#8-запуск-сервера)
9. [Генерация ссылок](#9-генерация-ссылок-для-клиентов)

---

## 1. Создание домена на DuckDNS

### Регистрация
1. Перейдите на сайт: https://www.duckdns.org
2. Войдите через Google / GitHub / Discord
3. Создайте свой поддомен, например: `myserver.duckdns.org`

---

## 2. Подготовка сервера

### Подключитесь к серверу и обновите систему:
```bash
apt update && apt upgrade -y
```

### Установите Certbot:
```bash
apt install certbot -y
```

### ВАЖНО: Откройте порт 8443/udp
```bash
ufw allow 8443/udp
```


---

## 3. Получение SSL сертификата

### Запросите сертификат от Let's Encrypt:
```bash
certbot certonly --standalone -d ваш_поддомен.duckdns.org
```

При запросе введите ваш email:
```
mail@gmail.com
```

### ✅ Запишите пути к сертификатам:
```
/etc/letsencrypt/live/ваш_поддомен.duckdns.org/fullchain.pem
/etc/letsencrypt/live/ваш_поддомен.duckdns.org/privkey.pem
```

---

## 4. Установка Hysteria2

### Скачайте и установите Hysteria2:
```bash
bash <(curl -fsSL https://get.hy2.sh/)
```

### Включите автозапуск:
```bash
systemctl enable hysteria-server
```

---

## 5. Настройка конфигурации

### Откройте файл конфигурации:
```bash
nano /etc/hysteria/config.yaml
```

### Замените содержимое на:
```yaml
listen: :8443
protocol: udp

tls:
  cert: /etc/letsencrypt/live/ваш_поддомен.duckdns.org/fullchain.pem
  key: /etc/letsencrypt/live/ваш_поддомен.duckdns.org/privkey.pem

auth:
  type: command
  command: /etc/hysteria/auth.sh

masquerade:
  type: proxy
  proxy:
    url: https://www.microsoft.com
    rewriteHost: true

trafficStats:
  listen: :9999
  secret: "stats_secret"
```

**Сохраните файл:** `Ctrl+O` → `Enter` → `Ctrl+X`

---

## 6. Создание пользователей

### Создайте скрипт авторизации:
```bash
nano /etc/hysteria/auth.sh
```

### Вставьте код со списком пользователей:
```bash
#!/bin/bash
users=(
  "user1 StrongPass123"
  "user2 SecurePass456"
  "user3 MyPassword789"
  "user4 Secret2024"
  "user5 HardPass999"
)

for user in "${users[@]}"; do
  echo "$user"
done
```

**Сохраните файл:** `Ctrl+O` → `Enter` → `Ctrl+X`

### Сделайте скрипт исполняемым:
```bash
chmod +x /etc/hysteria/auth.sh
```

### Проверьте работу скрипта:
```bash
/etc/hysteria/auth.sh
```

---

## 7. Настройка прав доступа

### Дайте права на чтение сертификатов:
```bash
chmod 755 /etc/letsencrypt/live/
chmod 755 /etc/letsencrypt/archive/
chmod 644 /etc/letsencrypt/archive/ваш_поддомен.duckdns.org/*.pem
```

### Настройте systemd service для запуска от root:
```bash
nano /etc/systemd/system/hysteria-server.service
```

Убедитесь, что в секции `[Service]` указано:
```ini
User=root
Group=root
```

---

## 8. Запуск сервера

### Перезагрузите systemd и запустите Hysteria2:
```bash
systemctl daemon-reload
systemctl restart hysteria-server
systemctl status hysteria-server
```

### ✅ Если видите `active (running)` — всё работает!

### Просмотр логов:
```bash
journalctl -u hysteria-server -f
```

### Проверка портов:
```bash
ss -tulpn | grep 8443
```

---

## 9. Генерация ссылок для клиентов

### Формат ссылки:
```
hysteria2://username:password@домен:8443/?insecure=0&sni=домен#Название
```

### Пример готовых ссылок:

**User1:**
```
hysteria2://user1:StrongPass123@myserver.duckdns.org:8443/?insecure=0&sni=myserver.duckdns.org#User1-PC
```

**User2:**
```
hysteria2://user2:SecurePass456@myserver.duckdns.org:8443/?insecure=0&sni=myserver.duckdns.org#User2-Mobile
```

**User3:**
```
hysteria2://user3:MyPassword789@myserver.duckdns.org:8443/?insecure=0&sni=myserver.duckdns.org#User3-Laptop
```

---

## 🔧 Дополнительные настройки

### Автообновление SSL сертификатов

Создайте hook для автоматического перезапуска после обновления:
```bash
mkdir -p /etc/letsencrypt/renewal-hooks/deploy

cat > /etc/letsencrypt/renewal-hooks/deploy/restart-hysteria.sh << 'EOF'
#!/bin/bash
systemctl restart hysteria-server
EOF

chmod +x /etc/letsencrypt/renewal-hooks/deploy/restart-hysteria.sh
```

Проверьте автообновление:
```bash
certbot renew --dry-run
```

### Открытие портов в firewall

Если используете UFW:
```bash
ufw allow 8443/udp
ufw status
```

Если используете панель хостинга (Aeza, Hetzner и др.) — откройте порт 8443/UDP в веб-панели.

---

## 📱 Клиентские приложения

Используйте эти приложения для подключения:

- **Android:** v2rayNG, NekoBox
- **iOS:** Shadowrocket, Stash
- **Windows:** v2rayN, Clash Verge
- **macOS:** ClashX, Surge
- **Linux:** Clash, v2ray-core

Просто импортируйте ссылку в приложение!

---

## ❓ Решение проблем

### Сервер не запускается

Проверьте логи:
```bash
journalctl -u hysteria-server -n 50
```

### Проблемы с сертификатами

Убедитесь что пути правильные:
```bash
ls -la /etc/letsencrypt/live/ваш_поддомен.duckdns.org/
```

### Проблемы с подключением

Проверьте что порт открыт:
```bash
ss -tulpn | grep 8443
```

Проверьте DNS:
```bash
nslookup ваш_поддомен.duckdns.org
```

---

## 🎉 Готово!

Ваш Hysteria2 сервер настроен и готов к использованию. Раздавайте ссылки пользователям и наслаждайтесь быстрым и стабильным подключением!
