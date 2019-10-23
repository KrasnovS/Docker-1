# Docker-1

##### 1. Создаем виртуальную машину с именем Char
`docker-machine create --driver virtualbox Char`
##### 2. Получаем  IP-адрес виртуальной машины Char.
`docker-machine ip Char`
##### 3. Определить переменные, чтобы `docker ps` работал без ошибок.
`eval $(docker-machine env Char)`
- docker ps -  показывает список запущенных контейнеров.
- Функция eval позволяет выполнить код, переданный ей в виде строки.
##### 4. Получите контейнер hello-world из Docker Hub
`docker pull hello-world`
##### 5. Запустите контейнер hello-world и убедитесь что рабоатет.
 `docker run hello-world`
##### 6. Запустить в фоне nginx (c Docker Hub'a), назвать его overlord, порты 80-5000, он должен иметь возможность перезапуска.
`docker run -d -p 5000:80 --name overlord --restart always nginx`
##### 7. Получить внутренний IP-адрес контейнера overlord без запуска его оболочки.
`docker inspect -f '{{.NetworkSettings.IPAddress}}' overlord`
##### 8. Запустите оболочку из контейнера alpine и убедитесь, что вы можете напрямую взаимодействовать с контейнером через терминал и что контейнер удаляется после выполнения оболочки.
`docker pull alpine && docker run --rm -it alpine`
- `-it` флаг для интерактивных процессов, таких как оболочка.
- `--rm` удалит контейнер после выхода.
##### 9. Из оболочки контейнера debian, установить все что нужно для компиляции Си и вставить это в репозиторий git.
#check (run -it --rm --name test debian). - для проверки.
`apt update -y && apt upgrade -y && apt install -y gcc git`
##### 10. Создайте том с именем `hatchery`.
`docker volume create --name hatchery`
##### 11. Вывести список Docker volumes созданных на машине.
`docker volume ls`
##### 12. Запустить контейнер MySQL в фоне, в случае ошиибки он перезапускается, пароль `Kerrigan` бд хранится в томе `hetchery`, создает имя бд `zerglings`, сам контейнер называется `spawning-pool`.
`docker run -d --restart on-failure --name=spawning-pool -e MYSQL_ROOT_PASSWORD=Kerrigan -e MYSQL_DATABASE=zerglings -v hatchery:/var/lib/mysql mysql:5.7`
##### 13. Вывести переменные среды контейнера spawning-pool одной командой.
`docker inspect -f '{{.Config.Env}}' spawning-pool`
##### 14. Запустить контейнер WordPress в фоне, он должен называться `lair` его порт 80 на порту 8080 ВМ. Он должен иметь возможность использовать контейнер spawning-pool.
`docker run -d --name lair -p 8080:80 --link spawning-pool:mysql wordpress`
##### 15. Запустить контейнер phpmyadmin в фоне, он должен называться `roach-warden` с портом 80 на порту 8081 ВМ. Он должен иметь возможность обозревать контейнер spawning-pool.
`docker run --name roach-warden -d --link spawning-pool:db -p 8081:80 phpmyadmin/phpmyadmin`
##### 16. Посмотреть логи контейнера `spawning-pool`, без запуска его оболочки.
`docker logs -f spawning-pool`
##### 17. Отобразить всех активных в данный момент контейнеров на виртуальной машине Char.
`docker ps`
##### 18. Перезапустить контейнер `overlord`.
`docker restart overlor`
##### 19. Запустиь контейнер с именем `Abathur`. Это будет контейнер Python, версия 2-slim, его папка /root будет связана с папкой HOME на вашем хосте, а ее порт 3000 будет связан с портом 3000 вашей виртуальной машины. Создать html-страницу которая выведет Hello World с помощью Flask.
```
docker run --name Abathur -dti -v /Users/prawney:/root -p 3000:3000 python:2-slim
docker exec -it Abathur /bin/bash
pip install flask
echo "from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

	if __name__ == '__main__':
		    app.run()" > app.py
FLASK_APP=app.py
flask run --host=0.0.0.0 --port 3000
```
##### 20. Создайте локальный swarm, виртуальная машина Char должна быть его менеджером.
`docker swarm init --advertise-addr $(docker-machine ip Char)`
проверять командой: docker node ls
##### 21. Создайте еще одну виртуальную машину с докер-машиной, используя драйвер virtualbox, и назовите ее Aiur.
`docker-machine create --driver virtualbox Aiur`
##### 22. Превратите Aiur в раба местного swarm, в котором Char является лидером (команда для захвата контроля над Aiur не запрашивается)
`docker-machine ssh Aiur "docker swarm join --token $(docker swarm join-token worker -q) $(docker-machine ip Char):2377"`
##### 23. Создайте внутреннюю сеть типа owerlay, которую вы назовете overmind.
`docker network create -d overlay overmind`
##### 24. Запустите сервис rabbitmq, который будет называться orbital-command. Вы должны определить конкретного пользователя и пароль для службы RabbitMQ, они могут быть любыми. Эта услуга будет в сети overmind.
`docker service create --name orbital-command --network overmind -e RABBITMQ_DEFAULT_USER=prawney -e RABBITMQ_DEFAULT_PASS=pass rabbitmq`
проверяем командой: docker inspect -f "{{.Spec.TaskTemplate.ContainerSpec.Env}}" orbital-command
##### 25. Вывести все сервисы swarm.
`docker service ls`
##### 26. Запустить сервис `42school/engineering-bay` в двух репликах
`docker service create -d --name engineering-bay --network overmind --replicas=2 -e OC_USERNAME=orbital_command -e OC_PASSWD=orbital-command 42school/engineering-bay`
проверяем командой: docker service ps engineering-bay
##### 27. Получить в режиме реального времени журналы одной из задач службы технической поддержки.
`docker service logs -f engineering-bay 2>&1 | grep "engineering-bay.1"`
##### 28. Запустите `42school/marine-squad` в двух репликах. Эта служба будет называться `marines` и будет находиться в сети `overmind`.
`docker service create -d --name marines --network overmind --replicas=2 -e OC_USERNAME=orbital-command -e OC_PASSWD=orbital-command 42school/marine-squad`
##### 29. Показать все задачи службы `marines`.
`docker service ps marines`
##### 30. Увеличьте количество копий службы `marines` до двадцати.
`docker service scale -d marines=20`
##### 31. Принудительно выйдите и удалите все сервисы на локальном swarm в одну команду.
`docker service rm $(docker service ls -q)`
проверяем: docker service ls
##### 32. Принудительно завершить работу и удалить все контейнеры (независимо от их статуса) одной командой.
`docker container rm -f $(docker container ls -q)`
проверяем: docker ps -a
##### 33. Удалите все образы контейнеров, хранящиеся на виртуальной машине Char, также одной командой.
`docker rmi $(docker images -a -q)`
проверяем: docker images ls
##### 34. Удалите виртуальную машину Aiur без использования rm -rf.
`docker-machine rm -y Aiur`
проверяем: docker-machine ls
