# **Task 1: Git State Reconciliation**

## 1. **Initialize repository**:

Репозиторий инициализирован стандартным способом git

```bash
$ mkdir gitops-lab && cd gitops-lab
$ git init
```

## 2. **Create desired state**

Некоторая информация, записанная в файл

```bash
$ echo "version: 1.0" > desired-state.txt

$ ls
desired-state.txt

$ cat desired-state.txt
version: 1.0

# Это вызвано двойной загрузкой и использованием одного каталога git из обеих систем
$ git add . && git commit -m "Initial state"
fatal: detected dubious ownership in repository at '/Data/File_Storage/University/Inno/DevOps_Sum25/gitops-lab'
To add an exception for this directory, call:

        git config --global --add safe.directory /Data/File_Storage/University/Inno/DevOps_Sum25/gitops-lab

# Так что я должен сделать то, что предложил
$ git config --global --add safe.directory /Data/File_Storage/University/Inno/DevOps_Sum25/gitops-lab

```

## 3. **Simulate live cluster**
Здесь мы сохранили наше состояние как состояние для мониторинга и поддержки (возможно, было удобнее сначала создать текущее состояние и только потом настроить его по своему усмотрению. Но я подумал об этом только после того, как все сделал).

```bash
$ cp desired-state.txt current-state.txt

$ cat desired-state.txt 
version: 1.0
```

## 4. **Create reconciliation script**

Здесь я поместил скрипт в файл `reconcile.sh`:

```sh
#!/bin/bash
# reconcile.sh
DESIRED=$(cat desired-state.txt) # Содержимое файла в желаемом состоянии
CURRENT=$(cat current-state.txt) # Содержимое файла в желаемом состоянии

# Если в текущем состоянии внесены какие-то изменения, то мы производим сверку (здесь мы просто заменяем измененный файл файлом, содержащим желаемое состояние).
if [ "$DESIRED" != "$CURRENT" ]; then
echo "$(date) - DRIFT DETECTED! Reconciling..."
cp desired-state.txt current-state.txt
fi
```

## 5. **Trigger manual drift**
Затем мы запускаем ручной дрейф (например, просто меняем содержимое файла).

```bash
echo "version: 2.0" > current-state.txt # Имитировать ручное изменение кластера
```

## 6. **Run reconciliation**
Затем мы запускаем проверку с помощью вновь созданного скрипта

```bash
$ chmod +x reconcile.sh
peter@peter-pc:/Data/File_Storage/University/Inno/DevOps_Sum25/gitops-lab$ ./reconcile.sh # Следует обнаруживать и фиксировать дрейф
Пн 28 июл 2025 20:08:17 MSK - DRIFT DETECTED! Reconciling...
```

И... Это работает! Состояние, похожее на измененное обратно на желаемое

```bash
$ cat desired-state.txt 
version: 1.0

$ cat current-state.txt 
version: 1.0
```

## 7. **Automate reconciliation**

Мы можем автоматизировать этот процесс очень простым способом.

В первом терминале, который я запустил:

```bash
$ watch -n 5 ./reconcile.sh

Every 5,0s: ./reconcile.sh
```

А во втором я изменил `current-state.txt` file:

```bash
$ echo "some change" > current-state.txt && cat current-state.txt
some change

$ cat current-state.txt 
some change

$ cat current-state.txt 
some change

$ cat current-state.txt 
some change

$ cat current-state.txt 
version: 1.0
```

Как мы видим, сверка выполнена успешно

Вывод в первом терминале:

```bash
Every 5,0s: ./reconcile.sh

Пн 27 июл 2025 20:05:44 MSK - DRIFT DETECTED! Reconciling...
```

# **Task 2: GitOps Health Monitoring**

## 1. **Create health check script**:

Созданный Sctript

```bash
#!/bin/bash
# healthcheck.sh
DESIRED_MD5=$(md5sum desired-state.txt | awk '{print $1}')
CURRENT_MD5=$(md5sum current-state.txt | awk '{print $1}')

if [ "$DESIRED_MD5" != "$CURRENT_MD5" ]; then
echo "$(date) - CRITICAL: State mismatch!" >> health.log
else
echo "$(date) - OK: States synchronized" >> health.log
fi
```

## 2. **Make executable**:

Затем я сделал его исполняемым

```bash
$ chmod +x healthcheck.sh

$ ls -la healthcheck.sh 
-rwxrwxrwx 1 root root 314 июл 12 15:03 healthcheck.sh
```

## 3. **Simulate healthy state**:

```bash
$ ./healthcheck.sh

$ cat health.log # Должно отображаться "ОК".
Пн 28 июл 2025 15:05:21 MSK - OK: States synchronized

```
В отслеживаемом файле нет ссылок, поэтому выводите данные так, как ожидалось

## 4. **Create drift**:

Затем я записываю некоторые "неутвержденные изменения" (lol) в файл состояния

```bash
$ echo "unapproved change" >> current-state.txt
```

## 5. **Run health check**:

```bash
$ ./healthcheck.sh
$ cat health.log # Теперь отображается "CRITICAL"

Сб 28 июл 2025 15:05:21 MSK - OK: States synchronized
Сб 28 июл 2025 15:08:18 MSK - CRITICAL: State mismatch!
```

Таким образом, мы можем отслеживать несанкционированные изменения в файлах monitord (но лучше использовать более удобные инструменты).

