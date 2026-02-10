# Лабораторная работа №1 (часть 1): Настройка окружения и знакомство с базовыми концепциями распределённых систем

### Этап 1: Подготовка базового окружения

Выбор правильной версии Ubuntu Server: 22.04 LTS - стабильная версия с долгосрочной поддержкой

### Этап 2: Установка и настройка Docker

Установка Docker:

```bash
# Обновление пакетов после установки ВМ
sudo apt update
sudo apt upgrade -y

# Установка зависимостей
sudo apt install apt-transport-https ca-certificates curl software-properties-common

# Добавление GPG ключа Docker и репозитория
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установка Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Добавление пользователя в группу docker
sudo usermod -aG docker $USER
newgrp docker
```

Создание Dockerfile для Nginx

```dockerfile
# Используем официальный образ Nginx как основу
FROM nginx:alpine

# Копируем наш HTML файл в контейнер
COPY index.html /usr/share/nginx/html/index.html

# Открываем порт 80 для веб-сервера
EXPOSE 80 
```

Создание простого index.html:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Лабораторная работа №1</title>
    <meta charset="utf-8">
</head>
</body>
    <h1>Добро пожаловать в лабораторную работу по распределенным системам!</h1>
    <p>Этот веб-сервер работает в Docker контейнере</p>
    <p>Студент: Солдаткин Александр</p>
    <p>Группа: Пин-22-1</p>
    <p>Дата: <span id="date"></span></p>
    
    <script>
        document.getElementById('date').textContent = new Date().toLocaleDateString();
    </script>
</body>
</html>
```

Сборка и запуск контейнера:

```bash
# Сборка Docker
docker build -t my-nginx .

# Проверка образов docker
docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
my-nginx      latest    7e79cd18ec0e   20 seconds ago   52.5MB
hello-world   latest    1b44b5a3e06a   6 weeks ago      10.1kB

# Запуск контейнера
docker run -d -p 8080:80 --name nginx-container my-nginx
44d280613a6f186de0873e193e9f7c68c0d664087d46cfe40098439e1302f5a0

# Вывод запущенных контейнеров
docker ps
CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS         PORTS                                     NAMES
48d414677c09   my-nginx   "/docker-entrypoint.…"   10 seconds ago   Up 9 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   nginx-container

### Вывод Dom-дерева по адресу
curl http://localhost:8080
<!DOCTYPE html>
<html>
<head>
    <title>Лабораторная работа №1</title>
    <meta charset="utf-8">
</head>
</body>
    <h1>Добро пожаловать в лабораторную работу по распределенным системам!</h1>
    <p>Этот веб-сервер работает в Docker контейнере</p>
    <p>Студент: Солдаткин Александр</p>
    <p>Группа: Пин-22-1</p>
    <p>Дата: <span id="date"></span></p>
    
    <script>
        document.getElementById('date').textContent = new Date().toLocaleDateString();
    </script>
</body>
</html>
```

### Этап 3: Установка Minikube и kubectl

```bash 
# Скачивание Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Установка kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Запуск Minikube
minikube start --driver=docker

# Проверка
kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   3m33s   v1.34.0

kubectl get pods -A
NAMESPACE     NAME                                READY   STATUS    RESTARTS       AGE
default       nginx-deployment-85f8885f8f-fx79v   1/1     Running   0              2d5h
default       nginx-deployment-85f8885f8f-gxgth   1/1     Running   0              2d5h
default       nginx-deployment-85f8885f8f-lfdlt   1/1     Running   0              2d5h
kube-system   coredns-66bc5c9577-48n2s            1/1     Running   0              2d5h
kube-system   etcd-minikube                       1/1     Running   0              2d5h
kube-system   kube-apiserver-minikube             1/1     Running   0              2d5h
kube-system   kube-controller-manager-minikube    1/1     Running   0              2d5h
kube-system   kube-proxy-zwbch                    1/1     Running   0              2d5h
kube-system   kube-scheduler-minikube             1/1     Running   0              2d5h
kube-system   storage-provisioner                 1/1     Running   1 (2d5h ago)   2d5h
```

### Этап 4: Демонстрация ключевых концепций

Создание манифестов Kubernetes

nginx-deployment.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: my-nginx
        ports:
        - containerPort: 80
```

nginx-service.yaml:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

Развертывание приложения:

```bash
# Применяем манифесты
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml

# Проверяем развертывание
kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           5m35s

kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85f8885f8f-g9lfn   1/1     Running   0          3m13s
nginx-deployment-85f8885f8f-gxgth   1/1     Running   0          20s
nginx-deployment-85f8885f8f-lfdlt   1/1     Running   0          20s

kubectl get services
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        11m
nginx-service   NodePort    10.101.237.87   <none>        80:31838/TCP   5m31s
```

Демонстрация масштабируемости:

```bash
# Увеличиваем количество реплик до 3
kubectl scale deployment nginx-deployment --replicas=3

# Наблюдаем за созданием подов
kubectl get pods -w
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85f8885f8f-9kp6x   1/1     Running   0          110s
nginx-deployment-85f8885f8f-nrpfg   1/1     Running   0          4s
nginx-deployment-85f8885f8f-vr88c   1/1     Running   0          4s
```

Демонстрация отказоустойчивости:

```bash
# Удаляем один под
kubectl delete pod $(kubectl get pods -l app=nginx -o jsonpath='{.items[0].metadata.name}')

# Наблюдаем за автоматическим восстановлением
kubectl get pods -w
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85f8885f8f-g9lfn   1/1     Running   0          8s
nginx-deployment-85f8885f8f-nrpfg   1/1     Running   0          44s
nginx-deployment-85f8885f8f-vr88c   1/1     Running   0          44s
```

Проверка доступности:

```bash
# Узнаем NodePort
kubectl get service nginx-service
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.101.237.87   <none>        80:31838/TCP   2m38s

# Используем Minikube сервис
minikube service nginx-service
┌───────────┬───────────────┬─────────────┬───────────────────────────┐
│ NAMESPACE │     NAME      │ TARGET PORT │            URL            │
├───────────┼───────────────┼─────────────┼───────────────────────────┤
│ default   │ nginx-service │ 80          │ http://192.168.49.2:31838 │
└───────────┴───────────────┴─────────────┴───────────────────────────┘
🎉  Opening service default/nginx-service in default browser...
```

# Лабораторная работа №1 (часть 1): Знакомство с HDFS и Spark Cluster

### Этап 1: Установка и настройка Hadoop HDFS и Spark

Устанавливаем Java

``` bash
# Устанавливаем OpenJDK 11
sudo apt install openjdk-11-jdk -y
```

Создание пользователя и настройка окружения

```bash
# Создаем пользователя hadoop
sudo adduser hadoop

sudo usermod -aG sudo hadoop
\Adding user `hadoop' ...
Adding new group `hadoop' (1001) ...
Adding new user `hadoop' (1001) with group `hadoop' ...
Creating home directory `/home/hadoop' ...
Copying files from `/etc/skel' ...
New password: 
BAD PASSWORD: The password is a palindrome
Retype new password: 
passwd: password updated successfully
Changing the user information for hadoop
Enter the new value, or press ENTER for the default
	Full Name []: vboxuser
	Room Number []: 5
	Work Phone []: 5
	Home Phone []: 5
	Other []: 5
Is the information correct? [Y/n] y 

su - hadoop

# Устанавливаем JAVA_HOME в текущей сессии
update-alternatives --list java
/usr/lib/jvm/java-11-openjdk-amd64/bin/java

export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin

echo $JAVA_HOME
/usr/lib/jvm/java-11-openjdk-amd64
```

Установка Hadoop

```bash
# Скачиваем Hadoop
cd /tmp
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz

# Распаковываем в /opt
sudo tar -xzf hadoop-3.3.6.tar.gz -C /opt/
sudo mv /opt/hadoop-3.3.6 /opt/hadoop
sudo chown -R $USER:$USER /opt/hadoop

# Проверяем
/opt/hadoop/bin/hadoop version
```

Настройка Hadoop (псевдо-распределенный режим)

Создаем конфигурационные файлы:

hadoop-env.sh:
```bash
nano /opt/hadoop/etc/hadoop/hadoop-env.sh
bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_HOME=/opt/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
```

core-site.xml:
```bash
nano /opt/hadoop/etc/hadoop/core-site.xml
xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

hdfs-site.xml:
```bash
nano /opt/hadoop/etc/hadoop/hdfs-site.xml
```

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/opt/hadoop/data/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/opt/hadoop/data/datanode</value>
    </property>
</configuration>
```

Настройка SSH для Hadoop

```bash
# Генерируем SSH ключ
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
Generating public/private rsa key pair.
Your identification has been saved in /home/vboxuser/.ssh/id_rsa
Your public key has been saved in /home/vboxuser/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:uUjFCuxvYZeagrjje+G1valj/XCRdtTkeAWN/GFVAAM vboxuser@linux
The key's randomart image is:
+---[RSA 3072]----+
|          E.++=o=|
|   .   .    =+.+ |
|    o   o  o +o .|
|   . . o oo .  . |
|    . = S+ .     |
| . o = *..o      |
|. o + X...       |
|.. o * o+        |
|o+o ..ooo.       |
+----[SHA256]-----+

cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys

hadoop@ubuntu1:~$ ssh localhost
ssh: connect to host localhost port 22: Connection refused
```

Форматирование HDFS и запуск

```bash
# Создаем директории для данных
mkdir -p /opt/hadoop/data/namenode
mkdir -p /opt/hadoop/data/datanode

# Форматируем HDFS
hdfs namenode -format

# Запускаем HDFS сервисы
start-dfs.sh
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [ubuntu1]

# Проверяем запущенные процессы
jps
136818 DataNode
136672 NameNode
137515 Jps
137052 SecondaryNameNode
```

Установка Apache Spark

```bash
# Скачиваем Spark
cd /tmp
wget https://downloads.apache.org/spark/spark-3.5.1/spark-3.5.1-bin-hadoop3.tgz

# Распаковываем в /opt
sudo tar -xzf spark-3.5.1-bin-hadoop3.tgz -C /opt/
sudo mv /opt/spark-3.5.1-bin-hadoop3 /opt/spark
sudo chown -R $USER:$USER /opt/spark
```

Запуск Spark кластера

```bash
# Запускаем Spark Master
/opt/spark/sbin/start-master.sh

# Запускаем Spark Worker
/opt/spark/sbin/start-worker.sh spark://$(hostname):7077

# Проверяем процессы
jps
136818 DataNode
141824 Master
136672 NameNode
142049 Jps
137052 SecondaryNameNode
141933 Worker
```

### Этап 2: Основы работы с HDFS

Проверка статуса HDFS

```bash
# Через команду
hdfs dfsadmin -report
Name: 127.0.0.1:9866 (localhost)
Hostname: ubuntu1.myguest.virtualbox.org
```

Работа с файлами в HDFS

```bash
# Создаем пользовательскую директорию
hdfs dfs -mkdir -p /user/student

# Создаем тестовый файл локально
echo "Hello HDFS World" > ~/test_data.txt
echo "This is line 2" >> ~/test_data.txt
echo "Line 3 for distributed storage" >> ~/test_data.txt

# Загружаем файл в HDFS
hdfs dfs -put ~/test_data.txt /user/student/

# Просматриваем содержимое директории
hdfs dfs -ls /user/student
Found 1 items
-rw-r--r--   1 hadoop supergroup         63 2025-09-26 21:34 /user/student/test_data.txt

# Просматриваем содержимое файла
hdfs dfs -cat /user/student/test_data.txt
Hello HDFS World
This is line 2
Line 3 for distributed storage

# Копируем файл обратно на локальную ФС
hdfs dfs -get /user/student/test_data.txt ~/test_data_copy.txt

# Проверяем локальную копию
cat ~/test_data_copy.txt
Hello HDFS World
This is line 2
Line 3 for distributed storage
```

### Запуск Spark приложения

Веб-интерфейс Spark: http://localhost:8080

Запуск тестового приложения Pi

```bash
# Определяем URL мастера
SPARK_MASTER="spark://$(hostname):7077"

# Запускаем вычисление Pi
spark-submit --master $SPARK_MASTER --class org.apache.spark.examples.SparkPi  $SPARK_HOME/examples/jars/spark-examples_2.12-3.5.1.jar 100
Pi is roughly 3.141592653589793
```