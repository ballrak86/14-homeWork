## Описание файлов в директории
logFileFull.log - полный лог выполнения  
used_commands.txt - команды которые использовал  

docker_folder - все что понадобится для создания docker image  
Dockerfile - docker файл
```
FROM alpine:latest
ENV NGINX_VERSION nginx-1.21.3
RUN apk --update --no-cache add build-base \
        openssl-dev \
        pcre-dev \
        zlib-dev \
        wget
RUN mkdir -p /tmp/src && \
    cd /tmp/src && \
    wget http://nginx.org/download/${NGINX_VERSION}.tar.gz && \
    tar zxf ${NGINX_VERSION}.tar.gz && \
    cd ${NGINX_VERSION} && \
    ./configure --sbin-path=/usr/bin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --pid-path=/var/run/nginx.pid \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --with-pcre \
        --with-http_ssl_module && \
    make && \
    make install && \
    rm -rf /tmp/src
RUN ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log
COPY index.html /usr/local/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
index.html - кастомная страница для nginx
```
Hello Otus! This is from Alexey!
```

## Теоретические вопросы
1. Определите разницу между контейнером и образом?  
Образ Docker упаковывает приложение и среду, необходимые приложению для запуска. Docker контейнер - это экземпляр запущенного образа.  
2. Можно ли в контейнере собрать ядро?  
Можно собрать ядро, но загрузиться с него нельзя, так как docker использует ядро системы.  
 
## Практическая часть. Создать свой кастомный образ nginx на базе alpine
Установим docker
```
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io
systemctl enable docker; systemctl start docker
```
создадим свой Dockerfile, содержимое файла описанно выше
```
mkdir /docker
cd /docker
nano Dockerfile
cat Dockerfile
```
Создадим образ и запустим docker контейнер (укажем порт системы 8080 и порт контейнера 80 чтобы пробросить порт). Проверим что контейнер запущен
```
docker build . --tag alenchik/otus:nginx_v1
docker run -d -p 8080:80 alenchik/otus:nginx_v1
[root@vm-gkuOA1801016 docker]# docker ps
  CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                                   NAMES
  127dbc6e8112   alenchik/otus:nginx_v1   "nginx -g 'daemon of…"   33 seconds ago   Up 32 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp   heuristic_mendel
```
Выполним запрос на интересующий нас порт
```
curl http://127.0.0.1:8080
  Hello Otus! This is from Alexey!
```
Осталось только разместить docker контейнер на docker hub. Для этого залогинимся и выполним push.
```
[root@vm-gkuOA1801016 docker]# docker login
  Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
  Username: alenchik
  Password:
  WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
  Configure a credential helper to remove this warning. See
  https://docs.docker.com/engine/reference/commandline/login/#credentials-store
  
  Login Succeeded
  
[root@vm-gkuOA1801016 docker]# docker push alenchik/otus:nginx_v1
  The push refers to repository [docker.io/alenchik/otus]
  e7b746f1f6dd: Pushed
  26ce46e9a9c6: Pushed
  9ae0270a7611: Pushed
  447185ab6580: Pushed
  e2eb06d8af82: Mounted from library/nginx
  nginx_v1: digest: sha256:f1572e32b03626a0d5d1aaf9b7dfadb80d082ba0906ad608210c311b45c40471 size: 1365
```

### Ссылка на образ
https://hub.docker.com/r/alenchik/otus/tags

📚Домашнее задание/проектная работа разработано(-на) для курса ["Administrator Linux. Professional"](https://otus.ru/lessons/linux-professional/)