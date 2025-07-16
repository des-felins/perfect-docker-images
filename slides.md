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


---
layout: image-right
image: "/cubes-v.png"
---

# 780МБ - нужно беспокоиться?


- `push` дольше: больше времени на обновление
- `pull` дольше: замедленное масштабирование
- Занимает много места на диске


---

# 