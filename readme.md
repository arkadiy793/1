### Введение в Jenkins
Здесь я выполняю упражнения по теме курса
https://www.youtube.com/playlist?list=PLmxB7JSpraiew9igtD89o33AaniUrmUzm
https://github.com/ksemaev/project_template


#### Запуск виртуалок
Для запуска выполнить 
```
vagrant up
vagrant provision
```
Что бы получить ssh нужно выполнить `vagrant ssh jenkins`. Ожидается, что в папке с Vagrantfile 
существует папка с именем sync, она будет замаплена на каталог /vagrant на виртуалках.

После установки UI доступен через localhost:18080.
При первом входе будет запрошен пароль, который можно посмотреть командой
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Затем вероятно возникнет сообщение "Offline. This Jenkins instance appears to be offline."
причиной которого являются просроченные сертификаты и как следствие неработающий https.
Лечится заменой https на http в теге url в файле /var/lib/jenkins/hudson.model.UpdateCenter.xml.
После это правки нужно рестартовать дженкинс `sudo service jenkins restart` и снова открыть интерфейс.

Появятся 2 кнопки выбор плагинов по умолчанию и избирательно, можно установить плагины по умолчанию. 
После нажатия нужно дождаться завершения загрузки.

Затем будет предложено создать пользователя - можно отказаться и продолжить под учеткой админа (паролем будет код подтвержения см.выше).
Что бы установить пароль, нужно перейти в раздел "Пользователи", выбрать в списке "admin", в меню "Настроить", 
ввести новый пароль и подтверждение и нажать сохранить - это потребует перелогиниться.


#### Настройка ssh на слейв-машине
Дженскинс будет подключаться к целевой системе по ssh и выполнять в ней определенные действия.
Для этого ему нужно предоставить ssh-доступ по ключу.
Сначала нужно сгенерировать ключ, войти на машину с дженкинсом `vagrant ssh jenkins` и выполнить последовательность команд:
```
sudo cat /etc/passwd
```
Проверить имя пользователя под учеткой которого работает jenkins. Зайти под этим пользователем, и сгенерировать ssh-ключ:
```
sudo su jenkins
ssh-keygen
```
принять имя файла и пустую контрольную фразу <ENTER>, <ENTER>.
Затем можно выйти из под учетки jenkins, "забрать" сгененрированный открытый ключ, и выйти из шелла:
```
exit
cp /var/lib/jenkins/.ssh/id_rsa.pub /vagrant
logout
```

Теперь нужно подключиться к слейву `vagrant ssh slave`.\
Предполагается, что на слейве дженкинс будет работать под root.
Нужно перейти в root `sudo su root`, в домашнем каталоге создать папку `cd ~ && mkdir .ssh`, если её нет.
И скопировать в неё файл с открытым ключем из расшареной папки `cp /vagrant/id_rsa.pub ~/.ssh/authorized_keys`,
копия должна называться `authorized_keys`.\
Можно выйти из учетки и из шелла:
```
exit
exit
```

Для проверки подключаемся к другой виртуалке `vagrant ssh jenkins`. Перейти под учетку jenkins 
и попытаться подключиться по ssh:
```
sudo su jenkins
ssh root@192.168.33.20
```
При подключении возникнет запрос на подтверждение fingerprint, нужно ответить yes.
После этого должно произойти подключение с правами root. Это показатель успешной настройки.


#### Тестовый джоб
Сейчас всё готово, что бы можно было создать "hello world" джоб через интерфейс дженкинса.\
Нужно зайти под админом, на основной вкладке будет отображаться предложение создать новый джоб.
После клика по кнопке отобразится экран на котором нужно ввести имя Item'a (джоба) например test-job,
и из вариантов выбрать "Создать задачу со свободной конфигурацией" и нажать "Ок".\
Отобразится экран со множеством панелей, на панели "Сборка" нужно добавить шаг "Выполнить команду shell".\
В поле ввести `ssh 192.168.33.20 'hostname'` - войти в систему slave под root и выполнить команду hostname.
Нажать "Сохранить", что переключит экран в раздел данного джоба. Здесть нужно выбрать "Собрать сейчас".\
Джоб начнет выполняться, на что указывает прогрессбар в левой части экрана. Через некоторое время "шарик" станет красным -
джоб завершился с ошибкой. Это произошло т.к. дженкинс пытался подключиться к удаленной машине под учеткой jenkins.\
Нужно исправить команду: `ssh root@192.168.33.20 'hostname'`, для чего перейти в меню "Настройки" и повторить 
шаги аналогиные созданию джоба.\
Повторить "Соборку" - теперь шарик станет синим, это говорит о том, что задача выполнена без ошибок.

**PS:** Шарик синий т.к. в японии вместо зеленого используется синий цвет светофора (что можно исправить плагином "Green balls").
Для установки плагина надо открыть меню "Настроить Jenkins" - "Управление плагинами" - "Доступные" и в списке отметить галочкой "Green Balls" (можно использовать фильтр),
нажать "Установить без перезагрузки", на следующем экране отметив "Перезапустить Jenkins по окончанию установки и отсутствии активных задач".
Во время установки интерфейс может зафризиться, поэтом через некоторое время можно попробовать нажать F5. Шарики должны стать зелеными.

#### Pipeline
Подготовительные шаги: установить git на машине jenkins: `sudo apt-get -y install git`, задать "DNS" для 192.168.33.20: `echo '192.168.33.20 slave' >> /etc/hosts` (возможно это можно сделать более "продакшн" способом).
После добавления DNS имени сервера нужно зайти под jenkins: `sudo su jenkins` и выполнить `ssh root@slave`, снова утвердительно ответить на вопрос про fingerprint, выйти из под jenkins.

Создать папку jenkinsfiles и в ней файл first_steps.jenkins - это описание пайплайна на groovy. Файлы этого проекта будут размещены на GitHub в репозитории https://github.com/Nikolayill/jenkins_tutorial

Теперь создадим пайплайн в Jenkins. На основном экране с джобами нужно выбрать "Создать Item", вести "first_steps", выбрать Pipline, нажать Ok. Перейти на вкладку Pipeline, Definition установить в "Pipeline script from SCM",
SCM - "Git", Repository URL "https://github.com/Nikolayill/jenkins_tutorial.git", Script Path "jenkinsfiles/first_steps.jenkins", "Сохранить".
Теперь можно "Собрать сейчас", джоб должен выполниться без ошибок.
В меню джоба "Status" будут отображаться два выполненных стейджа, имена и команды для которых определены в first_steps.jenkins (см. комментарии в файле). Для каждого стейджа можно посмотреть отдельный лог, если кликнуть на соответсвующем прямоугольнике.

#### Установка Docker
У меня используется старый дистрибутив Ubuntu поэтому установка докера будет несколько отличаться от туториала (см. https://docs.docker.com/install/linux/docker-ce/ubuntu/)
##### 1. Добавление репозиториев
Добавление ключа (шаг 1-3 в ориг. документе):
```
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
	
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
Добавление репозитория (шаг.4):
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   trusty \
   stable"
```   
- здесь жестко прописан дистрибутив "trusty"
##### 2. Установка пакетов
```
sudo apt-get update

sudo apt-get install docker-ce=17.03.0~ce-0~ubuntu-trusty
```
в отличие от более новых версий, в этой всё ставится одним пакетом - docker-ce-cli containerd.io ставить не требуется.
##### 3. Предоставление прав для Jenkins
Что бы дженкинс мог работать стартовать докер, нужно включить его в группу docker: `sudo usermod -a -G docker jenkins`.
Можно переключиться на учетку jenkins и проверить работоспособность докера:
```
sudo su jenkins
```
```
docker run hello-world
```
Должно отобразиться примерно такое сообщение:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

```
Выйти из учетки jenkins. И перезапустить jenkins что бы он "подхватил" изменения в системе после установки докера.
```
sudo service jenkins restart
```
#### Pipeline сборки docker образа
Протестируем работу докера на примере простейшей сборки образа.

Создадим новый пайплайн docker_build. Далее повторим шаги аналогично first_steps. 
Выберем Script Path "jenkinsfiles/docker_build.jenkins". Теперь можно сохраниться и выполнить сборку.\
**PS:** semaev/toolbox - название образа от автора оригинального курса было оставлено без изменений.

#### Запуск сборки по коммиту
Jenkins можно настроить на запуск пайплайна по факту изменений в Git (либо другой SCM). Настройка подразумевает два возможных режима работы - push и pull.\
В режиме push оповещение об изменениях берет на себя SCM-система, а Jenkins предоставляет точку входа (hook - некий уникальный URL).\
В режиме pull (poll?) Jenkins по расписанию опрашивает SCM и запускает пайплайн при изменениях.\

В данной конфигурации у нас нет возможности смоделировать работу pull механизма т.к. Jenkins находится в виртуалке за NAT-ом т.о. мы не можем предоставить точку входа для SCM.

Pull механизм настраивается в jenkins-файле добавлением секции `triggers{ pollSCM('* * * * *') }`, где через pollSCM задается cron-расписание опроса см. [triggers](https://jenkins.io/doc/book/pipeline/syntax/#triggers).
В данном случае опрос производится с максимальной частотой (раз в минуту).

#### Docker registry
Для дальнейшей работы нам потребуется Docker-registry. Можно воспользоваться DockerHub, либо поднять registry локально (см.[Быстрый запуск и использование своего открытого docker-registry](https://habr.com/ru/post/279659/))

##### Настройка локального регистра (репозитория)
Сначала нужно установить докер, действуя аналогично, как при установки докера на jenkins.\
Что бы запустить регистр локально есть готовый образ https://hub.docker.com/_/registry/?tab=description:
```
sudo docker run -d -p 5000:5000 --restart always --name registry registry:2
```
Что бы докер на этом сервере мог пушить в регистр, нужно добавить опции для докер-демона. 
В Ubuntu используется systemd поэтому опции задаются в `*.conf` файлах в `/etc/systemd/system/docker.service.d` см. https://docs.docker.com/config/daemon/systemd/
```
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo su root
echo [Service] > /etc/systemd/system/docker.service.d/local_insecure.conf
echo Environment="DOCKER_OPTS=--insecure-registry localhost:5000" >> /etc/systemd/system/docker.service.d/local_insecure.conf
exit
service docker restart
```
здесь `DOCKER_OPTS=--insecure-registry localhost:5000` указывает, что для доступа к этому регистру не нужен пароль.
теперь можно проверить регистр:
```
sudo docker pull ubuntu
sudo docker tag ubuntu localhost:5000/ubuntu
sudo docker push localhost:5000/ubuntu
```
и посмотреть на список образов `docker images`:
```
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
localhost:5000/ubuntu   latest              549b9b86cb8d        3 weeks ago         64.2 MB
```
сразу видно ссылку на репозиторий.

##### Доступ к репозиторию по логину/паролю
Для доступа к регистру извне, по логину и паролю, потребуется настройка TLS, без которой 
(см. [Restricting access](https://docs.docker.com/registry/deploying/#restricting-access) 
и [Use self-signed certificates](https://docs.docker.com/registry/insecure/#use-self-signed-certificates))

###### Генерация ключей и сертификата
Создадим каталог и перейдём в него
```
mkdir registry && cd "$_"
```
здесь создадим каталог для сертификатов и другой для конфигов _htpasswd_ `mkdir certs auth`.\
Сгенерируем ключ
```
cd certs/
openssl genrsa 1024 > domain.key
chmod 400 domain.key
```
и используем его для генерации самоподписанного сертификата
```
openssl req -new -x509 -nodes -sha1 -days 365 -key domain.key -out domain.crt
```
ответим на вопросы, затем проверим (`ls`) созданы ли файлы domain.crt и domain.key.

###### Настройка TLS
Перейдём в каталог auth и сгенерируем файл htpasswd используя докер контейнер registry
```
cd ../auth/
docker run --rm --entrypoint htpasswd registry:2 -Bbn username password > htpasswd
```
последней командой, в контейнере registry запускается утилита htpasswd, ключ `-Bbn`
указывает, что запуск идет в пакетном режиме (_b_) т.е. логин/пароль (`username`/`password`)
передаются аргументами, для шифромания используется bcrypt (_B_) и результат выводится в stdout (_n_).
Вывод перенаправляется в файл htpasswd в текущем каталоге. См. [htpasswd - Manage user files for basic authentication](https://httpd.apache.org/docs/current/programs/htpasswd.html)\
Убедимся, что всё получилось `cat htpasswd` должен отобразить `username:зашифрованный_пароль`
Конфигурация завершена.

###### Тестовый запуск
Перед запуском нужно удалить/переименовать контейнер registry который был запущен при старте (что было настроено в предидущих шагах).
```
docker stop registry
docker rm registry
```
Теперь можно вернуться в каталог `registry/` и из него запустить контейнер
```
docker run -d \
  --restart=always \
  --name registry \
  -v `pwd`/auth:/auth \
  -v `pwd`/certs:/certs \
  -v `pwd`/certs:/certs \
  -e REGISTRY_AUTH=htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -p 443:443 \
  registry:2
```
И наконец можно попытаться запушить в него, что нибудь:
```
docker pull busybox
docker tag busybox localhost:443/busybox
docker push localhost:443/busybox
```
появится сообщение вида:
```
The push refers to repository [localhost:443/busybox]
0314be9edf00: Preparing
no basic auth credentials
```
- нужно выполнить логин
```
docker login -u username https://localhost:443
Password:
Login Succeeded
```
и повторить пуш
```
docker push localhost:443/busybox
```
появится сообщение `0314be9edf00: Pushed` что говорит об успешном пуше.

###### Доступ извне
До сего момента доступ к регистру выполнялся только с той же машины, на которой запущен докер.
Первое, что нужно для доступа извне - открытый 443 порт на хост-машине (через группу или фаерволл).
Второе - нужно сконфигурировать докер-демон на клиентской машине, что бы он использовал созданный сертификат.
Для этого скопируем сертификат в расшаренный каталог Vagrant
```
cp certs/domain.crt /vagrant/
```
###### Настройка клиента (машина с jenkins)
Завершим ssh сессию с машиной registry и зайдем на jenkins.
Первое, что нужно - добавить registry в hosts
```
sudo su root
echo '192.168.33.30 registry' >> /etc/hosts
ping registry
```
пинг должен пройти.

Теперь нужно и установить сертификат. Для этого нужно создать каталог 
```
mkdir -p /etc/docker/certs.d/registry:443/
```
- здесь "registry" - домен регистра.
Далее в этот каталог нужно поместить сертификат, файл нужно переименовать в ca.crt
```
cp /vagrant/domain.crt  /etc/docker/certs.d/registry\:443/ca.crt
exit
```
Выйти из под root и перейти в учетку jenkins `sudo su jenkins`
Что бы удостовериться, что настройка выполнена, нужно попытаться залогиниться в регистр
```
docker login -u username https://registry:443
Password:
```
- ввести пароль "password", который настроили в htpasswd. Должно быть выдано `Login Succeeded`,
говорящее об успешном подключении к регистру. Можно повторить тест с контейнером busybox
```
docker pull busybox
docker tag busybox registry:443/busybox
docker push registry:443/busybox
```
появится сообщение о том, что пуш ссылается на репозиторий 
```
The push refers to a repository [registry:443/busybox]
195be5f8be1d: Pushed
```
###### Links
[Deploy a Docker Registry using self-signed certificates and htpasswd](https://medium.com/@lvthillo/deploy-a-docker-registry-using-tls-and-htpasswd-56dd57a1215a)
###### (TODO) Настройка докер-демона для registry
Что бы при рестарте машины регистр становился доступен извне по логину/паролю, нужно передать в контейнер 
переменные среды и ключи так, как он запускался в разделе "Тестовый запуск".\
Если на шаге "Тестовый запуск" контейнер был удален и создан под тем же именем, то сохранит заданную конфигурацию. Но если он будет удален, придется выполнить `docker run ... registry:2 ` заново.

#### Скрытие паролей в пайпах
##### Настройка Credentials
По многим причинам пароли не следует храить в jenkins-файле в открытом виде. Это не безопасно, и не удобно в поддержке.\
Для управления используемыми аккаунтами в интерфейсе Jenkins существует отдельное конфигурационное меню.
Откроем http://localhost:18080/ и главной страницы пройдем в "Credentials" -> "System" -> "Global credentials (unrestricted)", в меню слева выберем "Add Credentials".
Заполним форму, Kind = "Username with Password", Scope = "Global". В поля Username и Password введем логин/пароль для доступа к registry (см. выше "Настройка TLS").
В ID введем "registry_user" - это идентификатор который мы будем использовать в пайплайне и Description = "registry_user" (описание произвольного содержания). Сохранимся - "Ок".

##### Использование Credentials в пайплайне
Теперь нужно модифицировать пайплайн "jenkinsfiles\docker_build.jenkins", что бы он подключался к регистру используя учетные данные и пушил в него собранный образ.\

В начало пайплайна добавим шаг на котором будем выполнен `docker login`. В нём используем плагин _withCredentials_ который связывает идентификаторы $USERNAME и $PASSWORD c логином и паролем указанными для registry_user.\
Второй шаг (build) останется почти без изменений - в начало имени образа добавим имя репозитория.\
Добавим третий шаг - push в наш репозиторий, команду: `docker push registry:443/semaev/toolbox:latest`.
После коммита изменений в git, должна произойти успешная сборка и пуш в удаленный репозиторий.\

Убедиться, что всё сработало можно например так
```
docker rmi registry:443/semaev/toolbox:latest
docker images
```
удаляем образ оставшийся после выполнения build, проверяем по списку.
```
docker push registry:443/semaev/toolbox:latest
docker images
```
в списке снова появится registry:443/semaev/toolbox:latest.
