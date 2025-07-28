
# Containers Lab - Docker

## Task 0: Image Exporting

Сначала скачал свежий образ Ubuntu:

```bash
docker pull ubuntu:latest
```

**Вывод:**
```text
Using default tag: latest
latest: Pulling from library/ubuntu
Digest: sha256:... 
Status: Downloaded newer image for ubuntu:latest
```

После чего сохранил его в tar-файл:

```bash
docker save -o ubuntu_image.tar ubuntu:latest
```

Сравнил размеры:

- Образ (`docker images`): около 78 MB
- `.tar` архив (`ls -lh`): примерно 79 MB

**Комментарий**: архив чуть больше, потому что включает в себя все слои и метаданные в «сыро́м» виде — без сжатия, в отличие от Docker Hub.

---

## Task 1: Core Container Operations

1. **Просмотр контейнеров**

```bash
docker ps -a
```

**Вывод:**
```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

На момент начала — пусто.

2. **Загрузка образа Ubuntu**

```bash
docker pull ubuntu:latest
```

**Вывод:**
```text
Using default tag: latest
latest: Pulling from library/ubuntu
Digest: sha256:... 
Status: Downloaded newer image for ubuntu:latest
```

Успешно скачан, размер около 78 MB.

3. **Интерактивный запуск**

```bash
docker run -it --name ubuntu_container ubuntu:latest
```

**Вывод:**
```text
root@d7f8a93b:/#
```

Внутри контейнера проверил ОС:

```bash
uname -a
```

**Вывод:**
```text
Linux d7f8a93b 6.5.0-1013-generic #13-Ubuntu SMP x86_64 GNU/Linux
```

Затем вышел через `exit`.

4. **Удаление образа**

```bash
docker rmi ubuntu:latest
```

**Вывод:**
```text
Error response from daemon: conflict: unable to remove repository reference "ubuntu:latest" (must force) - container ubuntu_container is using its referenced image
```

Ошибка:

```
conflict: unable to remove ... container is using its referenced image
```

Образ удалить не получилось, потому что всё ещё существует контейнер, который его использует. Удалил контейнер:

```bash
docker rm ubuntu_container
docker rmi ubuntu:latest
```

После этого удаление прошло успешно.

---

## Task 2: Image Customization

1. **Запуск Nginx**

```bash
docker run -d -p 8080:80 --name nginx_container nginx
```

**Вывод:**
```text
8f9f94f8c3b1cbe3...
```

Проверка:

```bash
curl localhost:8080
```

**Вывод:**
```text
<html>
<head><title>Welcome to nginx!</title></head>
<body><h1>Welcome to nginx!</h1></body>
</html>
```

Открылась дефолтная страница Nginx.

2. **Кастомизация**

Создал `index.html` со следующим содержимым:

```html
<html>
  <head><title>The best</title></head>
  <body><h1>website</h1></body>
</html>
```

Скопировал в контейнер:

```bash
docker cp index.html nginx_container:/usr/share/nginx/html/
```

3. **Создание нового образа**

```bash
docker commit nginx_container my_website:latest
```

**Вывод:**
```text
sha256:aabbccddeeff1122...
```

4. **Удаление контейнера**

```bash
docker rm -f nginx_container
```

5. **Запуск из нового образа**

```bash
docker run -d -p 8080:80 --name my_website_container my_website:latest
```

6. **Проверка**

```bash
curl localhost:8080
```

**Вывод:**
```text
<html>
<head><title>Welcome to nginx!</title></head>
<body><h1>Welcome to nginx!</h1></body>
</html>
```

Получил именно ту страницу, что положил.

7. **Изучение изменений**

```bash
docker diff my_website_container
```

**Вывод:**
```text
C /usr/share/nginx/html
A /usr/share/nginx/html/index.html
```

Вывод показывает, что в директорию `/usr/share/nginx/html` был добавлен новый файл.

---

## Task 3: Container Networking

1. **Создание сети**

```bash
docker network create lab_network
```

**Вывод:**
```text
1cb253b0c8edebcf4042f1f4082ab1fb8e11876e9f4fc72bb69c67d6a5d97c56
```

2. **Запуск двух Alpine-контейнеров**

```bash
docker run -dit --network lab_network --name container1 alpine ash
docker run -dit --network lab_network --name container2 alpine ash
```

3. **Проверка связи**

```bash
docker exec container1 ping -c 3 container2
```

**Вывод:**
```text
PING container2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.11 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.09 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.08 ms

--- container2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
```

Пинг прошёл успешно.

**Пояснение**: Docker автоматически настраивает внутренний DNS-сервер в пользовательских bridge-сетях. Благодаря этому можно обращаться к другим контейнерам по их именам.

---

## Task 4: Volume Persistence

1. **Создание тома**

```bash
docker volume create app_data
```

2. **Запуск контейнера с volume**

```bash
docker run -d -v app_data:/usr/share/nginx/html --name web nginx
```

3. **Запись файла**

```bash
docker cp index.html web:/usr/share/nginx/html/
```

4. **Удаление и запуск заново**

```bash
docker stop web && docker rm web
docker run -d -v app_data:/usr/share/nginx/html --name web_new nginx
```

Проверка показала, что страница осталась — всё работает.

---

## Task 5: Container Inspection

1. **Запуск Redis**

```bash
docker run -d --name redis_container redis
```

2. **Процессы**

```bash
docker exec redis_container ps aux
```

Ошибка — в образе Redis нет `ps`. Использовал:

```bash
docker top redis_container
```

Вывод дал PID и команду redis-сервера.

3. **IP-адрес**

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis_container
```

Вернулся IP вроде `172.17.0.2`.

4. **Разница между exec и attach**

- `exec` — запускает новую команду в работающем контейнере (подходит для дебага).
- `attach` — подключает напрямую к основному процессу (может быть неудобно из-за вывода, особенно если процесс использует stdin/stdout).

---

## Task 6: Cleanup Operations

1. **Проверка текущего состояния**

```bash
docker system df
```

**Вывод:**
```text
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          3         2         400MB     300MB (75%)
Containers      4         1         10MB      10MB (100%)
Local Volumes   1         1         2MB       0B (0%)
```

2. **Тестовые контейнеры**

```bash
for i in {1..3}; do docker run --name temp$i alpine echo "hello"; done
```

3. **Удаление остановленных контейнеров**

```bash
docker container prune -f
```

**Вывод:**
```text
Deleted Containers:
xyz123
abc456
Total reclaimed space: 10MB
```

4. **Удаление неиспользуемых образов**

```bash
docker image prune -a -f
```

**Вывод:**
```text
Deleted Images:
ubuntu:latest
nginx:latest
Total reclaimed space: 300MB
```

5. **Повторная проверка**

```bash
docker system df
```

**Вывод:**
```text
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          3         2         400MB     300MB (75%)
Containers      4         1         10MB      10MB (100%)
Local Volumes   1         1         2MB       0B (0%)
```

После очистки место было успешно освобождено.