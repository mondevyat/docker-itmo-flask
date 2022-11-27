### Лаборатные работы ПИС.

#### Серверное приложение на платформе node.js

Обрабатываются следующие запросы:
```js
app.get("/", (_, res: Response) => {
  res.end(JSON.stringify(counter))
});

app.get("/stat", (_, res: Response) => {
  res.end(JSON.stringify(counter++))
});

app.get("/about", (_, res: Response) => {
  const name = "Артём Монченко"
  const html = `<h3>Привет, ${name}!</h3>`

  res.writeHead(200, {'Content-Type': 'text/html'})
  res.end(html)
});
```

```dockerfile
# используется базовым образ node
FROM node:latest
# определяется рабочая директория для приложения
WORKDIR /app
# копируются нужные для сборки и запуска проекта файлы в рабочую директорию 
COPY ["index.ts",  "package.json", "tsconfig.json", ".env", "./"]
# скачиваются нужные библиотеки в соответствии с package.json
RUN npm install
# собирается проект
RUN npm run build
# запускается проект
CMD npm run start
# открывается порт
EXPOSE 8080
```

Собирается образ такой командой:
```text
docker build -t mondevyat/itmo-counter:1.0.0 .
```

Либо через docker-compose (внутри теги расставляются):
```text
docker-compose build .
```

Помещается образ в репозиторий Docker Hub командой:
```text
docker push mondevyat/itmo-counter:1.0.0
```

Данный образ можно использовать командой:
```text
docker pull mondevyat/itmo-counter:1.0.0
```

Запускается контейнер командой:
```text
docker run mondevyat/itmo-counter:1.0.0
```

C помощью утилиты docker-compose запускаются два контейнера (первый с базой, второй – с приложением-счетчиком):
```text
docker-compose up
```

GitHub Actions скрипты позволяют автоматическую сборку и развертывание приложения. Разворачивание производится на физический сервер. 

Для репликации приложения используются OpenShift-манифесты - для deployment'ов и service'ов.
Имеется развернутый OpenShift / Kubernetes, либо локально запущенный (minikube).
Команды для обработки шаблона deployment, применения сервисов для каждого приложения:

```text
oc process -f openshift/itmo-counter/deployment-template.yml -p IMAGE_TAG=latest | oc apply -f -
oc apply -f openshift/itmo-counter/service.yml

oc process -f openshift/itmo-postgres/deployment-template.yml -p IMAGE_TAG=latest | oc apply -f -
oc apply -f openshift/itmo-postgres/service.yml
```

Для деплоймента счетчика указываются 4 реплики - будут созданы 4 подов, смотрящие на базу PostgreSQL(которая создается другим деплойментом). Взаимодействие между ними происходит через сервисы.

