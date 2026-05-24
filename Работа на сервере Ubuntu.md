### Если подключаемся на прямую по публичному IP
```
ssh ip-server@root
```
###Если подключаемся к приватному серверу через сервер с публичным IP
```
ssh -J root@PUBLIC_NGINX_IP root@192.168.0.3
```
### Обновите список пакетов

```
sudo apt update
```
## Установите Git

```

```
### Установите необходимые пакеты

```
sudo apt install -y git
sudo apt install -y ca-certificates 
curl gnupg lsb-release
```
### Добавьте GPG-ключ Docker

```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
### Подключите официальный репозиторий Docker

```
rm -f /etc/apt/sources.list.d/docker.list
. /etc/os-release
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $VERSION_CODENAME stable" > /etc/apt/sources.list.d/docker.list
apt update
```
### Обновите список пакетов

```

```
### Установите Docker Engine и Docker Compose Plugin

```
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### Проверьте установку

```
docker --version
docker compose version
```
## Ожидаемый результат

```
Docker version 28.x.x
Docker Compose version v2.x.x
```

Запустите службу docker:

```
sudo systemctl start docker
```
Потом проверьте статус:

```
sudo systemctl status docker
```
Чтобы Docker запускался автоматически после перезагрузки сервера:

```
sudo systemctl enable docker
```
## Быстрый запуск тестового контейнера

```
sudo docker run hello-world
```
## Установите UFW

```
sudo apt update
sudo apt install -y ufw
```
## Откройте порт 8001

```
sudo ufw allow 8001/tcp
```

## Переименовываем .env.example 
```
cp .env.example .env
```