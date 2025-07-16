---
theme: default
background: /cover.jpg
title: Creating a Perfect Docker Image for Java
layout: cover
class: text-center
drawings:
  persist: false
selectable: true
remoteAssets: true
canvasWidth: 800
colorSchema: "dark"
---

## Как создать идеальный Docker-образ для Java

<img src="/bellsoft.png" width="200px" class="absolute right-10px bottom-5px"/>

<style>
h2 {
    font-size: 50px;
    line-height: 1.2;
}
</style>

---
layout: image-right
image: "/cat.jpg"
---

# Катрин Эдельвейс

<img/>

🥑 Developer Advocate в BellSoft

😍Пишу на Java, люблю Spring, балуюсь с JavaFX

👩‍💻Техрайтер

👾Соведущая YouTube-канала CyberJAR

<span><line-md-twitter-x /> cat_edelveis</span>

<span><logos-bluesky /> cat-edelveis.bsky.social</span>

---
layout: image-right
image: "/members.png"
---

# О компании

BellSoft основана в 2017. Тех. ядро - эксперты Java и Linux с опытом работы в Sun/Oracle 15+ лет.

- Компания входит в состав:
    - JCP Executive Committee
    - OpenJDK Vulnerability Group
    - GraalVM Advisory Board
    - Linux Foundation
    - Cloud Native Computing Foundation


---
layout: image-right
image: "/qr.png"
---

# О компании


- Продукты:
    - Liberica JDK
    - Liberica Native Image Kit
    - Alpaquita Linux

<br/>
Liberica JDK официально рекомендована <logos-spring-icon />

---

# Идеальный Docker-образ... это какой?

<v-clicks>

- Крошечный размер
- Молниеносный старт
- Безопасность
- Быстрый push/pull
- Автоматическая сборка без Докерфайла
- Всё сразу?

</v-clicks>

---

## Погоня за идеалом может привести сюда 

<img src="/monster.png" class="center"/>

<style>
.center {
  display: block;
  margin-left: auto;
  margin-right: auto;
}
</style>

---

# Идеал - понятие относительное

- Расставляем приоритеты
- Mix & Match разные подходы
<br><br>
<v-click>А вот про подходы я сейчас расскажу 😎</v-click>


---
class: text-center
layout: cover
background: /cubes.png
---

# Начнем с уменьшения размера образа

---

# Что будем гонять?

Демо приложение NeuroWatch

- Петклиника на стероидах
- MongoDB
- Vaadin
- Spring Security

---

# Отправная точка


- Берем образ с Liberica JDK и Debian (популярная ОС для контейнеров)
- Добавляем проект в базовый образ
- Собираем JAR-файл
- Запускаем


```docker {1|3|4|6}
FROM bellsoft/liberica-openjdk-debian:24 as builder
WORKDIR /app
ADD . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction clean package
EXPOSE 8081
ENTRYPOINT java -jar /app/neurowatch/target/*.jar
```

---

# Результат

```declarative
ID         TAG                          SIZE      COMMAND                                                                         │
│946a796e83 neurowatch-neurowatch:latest 0B        ENTRYPOINT ["/bin/sh" "-c" "java -jar /app/neurowatch/target/*.jar"]            │
│<missing>                               0B        EXPOSE map[8081/tcp:{}]                                                         │
│<missing>                               501.22MiB RUN /bin/sh -c cd neurowatch && ./mvnw -Pproduction clean package # buildkit    │
│<missing>                               8.65MiB   COPY . /app/neurowatch # buildkit                                               │
│<missing>                               0B        WORKDIR /app                                                                    │
│<missing>                               0B        ENV JAVA_HOME=/usr/lib/jvm/jdk-24.0.1-bellsoft-x86_64 PATH=/usr/lib/jvm/jdk-24.0│
│<missing>                               123.40MiB RUN |8 LIBERICA_IMAGE_VARIANT=lite LIBERICA_VM=server LIBERICA_GENERATE_CDS=fals│
│<missing>                               0B        ARG BASE_URL=https://download.bell-sw.com/java/                                 │
│<missing>                               0B        ARG LIBERICA_ROOT=/usr/lib/jvm/jdk-24.0.1-bellsoft                              │
│<missing>                               0B        ARG LIBERICA_BUILD=11                                                           │
│<missing>                               0B        ARG LIBERICA_VERSION=24.0.1                                                     │
│<missing>                               0B        ARG LIBERICA_JVM_DIR=/usr/lib/jvm                                               │
│<missing>                               0B        ARG LIBERICA_GENERATE_CDS=false                                                 │
│<missing>                               0B        ARG LIBERICA_VM=server                                                          │
│<missing>                               0B        ARG LIBERICA_IMAGE_VARIANT=lite                                                 │
│<missing>                               0B        ENV LANG=en_US.UTF-8 LANGUAGE=en_US:en                                          │
│<missing>                               44.94MiB  RUN /bin/sh -c apt-get update                                &&    apt-get insta│
│<missing>                               111.17MiB # debian.sh --arch 'amd64' out/ 'bookworm' '@1743984000'
```


---
layout: image-right
image: "/cubes-v.png"
---

# 780МБ - нужно беспокоиться?


- `push` дольше: больше времени на обновление
- `pull` дольше: замедленное масштабирование
- Занимает много места на диске


---

# Multi-stage

---

# Dockerfile

---

# Result

```declarative
ID         TAG                          SIZE      COMMAND                                                                         │
│97855e4950 neurowatch-neurowatch:latest 0B        ENTRYPOINT ["java" "-jar" "/app/app.jar"]                                       │
│<missing>                               0B        EXPOSE map[8080/tcp:{}]                                                         │
│<missing>                               62.83MiB  COPY /app/neurowatch/target/neurowatch-*.jar app.jar # buildkit                 │
│<missing>                               0B        WORKDIR /app                                                                    │
│<missing>                               0B        ENV JAVA_HOME=/usr/lib/jvm/jre-24.0.1-bellsoft-x86_64 PATH=/usr/lib/jvm/jre-24.0│
│<missing>                               124.82MiB RUN |6 LIBERICA_VERSION=24.0.1 LIBERICA_BUILD=11 LIBERICA_VARIANT=jre LIBERICA_R│
│<missing>                               0B        ARG LIBERICA_GENERATE_CDS=false                                                 │
│<missing>                               0B        ARG LIBERICA_USE_LITE=1                                                         │
│<missing>                               0B        ARG LIBERICA_ROOT=/usr/lib/jvm/jre-24.0.1-bellsoft                              │
│<missing>                               0B        ARG LIBERICA_VARIANT=jre                                                        │
│<missing>                               0B        ARG LIBERICA_BUILD=11                                                           │
│<missing>                               0B        ARG LIBERICA_VERSION=24.0.1                                                     │
│<missing>                               0B        ENV LANG=en_US.UTF-8 LANGUAGE=en_US:en                                          │
│<missing>                               44.94MiB  RUN /bin/sh -c apt-get update                                &&    apt-get insta│
│<missing>                               111.17MiB # debian.sh --arch 'amd64' out/ 'bookworm' '@1743984000'
```


---

# Result

```declarative
ID         TAG                          SIZE      COMMAND                                                                         │
│c88af46c34 neurowatch-neurowatch:latest 62.83MiB  [stage-1 3/3] COPY --from=builder /app/neurowatch/target/neurowatch-*.jar app.ja│
│<missing>                               0B        ENTRYPOINT ["java" "-jar" "/app/app.jar"]                                       │
│<missing>                               0B        EXPOSE map[8080/tcp:{}]                                                         │
│<missing>                               0B        COPY /app/neurowatch/target/neurowatch-*.jar app.jar # buildkit                 │
│<missing>                               126.43MiB WORKDIR /app  
```