# OTUS ДЗ 14 Docker  (Ubuntu 24.04)

---

### Домашнее задание

1. Установите Docker на хост машину

2. Установите Docker Compose \- как плагин, или как отдельное приложение

3. Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)

   

4. Определите разницу между контейнером и образом. Вывод опишите в домашнем задании.  
     
5. Ответьте на вопрос: Можно ли в контейнере собрать ядро?

#### Установка Docker и Docker Compose

Настройте apt репозиторий Docker.  
Добавляем официальный GPG ключ  
`sudo apt-get update`  
`sudo apt-get install ca-certificates curl`  
`sudo install -m 0755 -d /etc/apt/keyrings`  
`sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc`  
`sudo chmod a+r /etc/apt/keyrings/docker.asc`  
`Добавьте репозиторий в источники Apt`  
`echo \`  
 `"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \`

 `$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \`

 `sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`  
`sudo apt-get update`

Установите пакеты Docker  
`sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`  
убедитесь, что установка Docker Engine прошла успешно, запустив hello-world образ  
`sudo docker run hello-world`  
И проверяем установленные пакеты   
`dpkg --get-selections | grep docker`  
`root@dz14-otus:~/custom-nginx# dpkg --get-selections | grep docker`  
`docker-buildx-plugin                            install`  
`docker-ce                                       install`  
`docker-ce-cli                                   install`  
`docker-ce-rootless-extras                       install`  
`docker-compose-plugin                           install`

Данный алгоритм установки взять с официального сайта Docker.  
[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

#### Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)

Создаем отдельную директорию и кладем туда два файла  
`mkdir custom-nginx`

Создаем файл Dockerfile (во вложении)

Создаем файл index.html с измененным содержимым (этот файл будет импортирован в наш образ при создании) (во вложении)

После чего собираем наш образ командой   
`docker build -t root/custom-nginx:nginx .`

Далее можем запустить наш контейнер командой   
`docker run -d -p 8080:80 root/custom-nginx:nginx`

Видим наш образы и наши контейнеры:

`[root@borg-server docker]# docker images`

`root@dz14-otus:~/custom-nginx# docker images`  
`REPOSITORY                  TAG        IMAGE ID       CREATED         SIZE`  
`root/custom-nginx           nginx      89c89c465b3e   19 hours ago    242MB`

`root@dz14-otus:~/custom-nginx# docker ps`  
`CONTAINER ID   IMAGE                     COMMAND                  CREATED        STATUS        PORTS                                              NAMES`  
`5aee55aee993   root/custom-nginx:nginx   "nginx -g 'daemon of…"   19 hours ago   Up 19 hours   443/tcp, 0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   ecstatic_pasteur`

`Проверяем проброс портов` 

`root@dz14-otus:~/custom-nginx# ss -ntlp`  
`State    Recv-Q   Send-Q     Local Address:Port       Peer Address:Port   Process`                                                       
`LISTEN   0        4096       127.0.0.53%lo:53              0.0.0.0:*       users:(("systemd-resolve",pid=11221,fd=15))`                  
`LISTEN   0        4096          127.0.0.54:53              0.0.0.0:*       users:(("systemd-resolve",pid=11221,fd=17))`                  
`LISTEN   0        1024           127.0.0.1:41207           0.0.0.0:*       users:(("code-38c31bc77e",pid=1932,fd=10))`                   
`LISTEN   0        4096             0.0.0.0:8080            0.0.0.0:*       users:(("docker-proxy",pid=40324,fd=4))`                      
`LISTEN   0        4096                   *:22                    *:*       users:(("sshd",pid=20096,fd=3),("systemd",pid=1,fd=109))`     
`LISTEN   0        4096                [::]:8080               [::]:*       users:(("docker-proxy",pid=40328,fd=4))`                    

Смотрим на стартовую страницу curl [http://localhost:8080](http://localhost:8080)

`root@dz14-otus:~/custom-nginx# curl http://localhost:8080`  
`echo "<!DOCTYPE html>`  
`<html>`  
`<head>`  
    `<title>My custom page</title>`  
`</head>`  
`<body>`  
    `<h1>Hello, Otus!</h1>`  
    `<p>This is my custom Nginx page.</p>`  
`</body>`  
`</html>"`

![Снимок](https://github.com/user-attachments/assets/2b298426-d089-4993-9b17-026b74fc66d0)


#### Собранный образ необходимо запушить в docker hub и дать ссылку на ваш репозиторий

Выполняем команду docker login, и вводим логин и пароль от нашего зарегистрированного аккаунта из docker-hub:

`root@dz14-otus:~/custom-nginx# docker login -u otusalex`  
`Password:`   
`WARNING! Your password will be stored unencrypted in /root/.docker/config.json.`  
`Configure a credential helper to remove this warning. See`  
`https://docs.docker.com/engine/reference/commandline/login/#credential-stores`

Меняем имя контейнера ( так-как имя пользователя и контейнера должны совпадать и hub )  
`root@dz14-otus:~/custom-nginx# docker tag root/custom-nginx:nginx otusalex/otus-dz14-docker`  
`root@dz14-otus:~/custom-nginx#`

Далее делаем push

`root@dz14-otus:~/custom-nginx# docker push otusalex/otus-dz14-docker`  
`Using default tag: latest`  
`The push refers to repository [docker.io/otusalex/otus-dz14-docker]`  
`739d1af6fac8: Pushed`   
`f4cb4a5fbcee: Pushed`   
`13b9809b5a72: Pushed`   
`0fd195ec84ae: Pushed`   
`63ca1fbb43ae: Pushed`   
`latest: digest: sha256:787247562da6f6780994d4d4c816a12c2490b435cd608b50874387ac206a8977 size: 1365`

Ссылка на репозиторий  
[https://hub.docker.com/repository/docker/otusalex/otus-dz14-docker/general](https://hub.docker.com/repository/docker/otusalex/otus-dz14-docker/general)

#### 

#### Определите разницу между контейнером и образом. Вывод опишите в домашнем задании.

Образ — это шаблон, содержащий всё необходимое для запуска контейнера, включая исходный код, библиотеки, зависимости, конфигурационные файлы и инструкции для выполнения приложения.  
Контейнер — это запущенный экземпляр образа. Он представляет собой изолированную среду, в которой работает приложение.  
Образы и контейнеры — это две стороны одной медали, где образы служат основой, а контейнеры обеспечивают выполнение приложений.

#### Ответьте на вопрос: Можно ли в контейнере собрать ядро?

Да, в Docker-контейнере можно собрать ядро Linux, хотя необходимо учитывать несколько аспектов и особенностей этого процесса.  
Контейнеры Docker используют ядро хост-операционной системы. Это означает, что у контейнера нет своего собственного ядра; он использует ядро хост-системы.  
Контейнер будет зависеть от версии ядра, установленного на хосте  
Сборка ядра в Docker-контейнере возможна, но учтите, что это требует правильной настройки и может не обеспечивать полноценное тестирование и управление. Если вы собираетесь разрабатывать настройку ядра и управлять модулями, возможно, более целесообразно использовать виртуальные машины или физическую машину для этих целей.

[image1]: <data:image/png;base64,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>
