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
- Быстрый push/pull
- Автоматическая сборка без Докерфайла
- Всё сразу?

</v-clicks>

---

## Погоня за идеалом может привести сюда 

<img src="/monster.png" class="center"  width="400px"/>

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

# Демо приложение NeuroWatch

- Петклиника на стероидах
- MongoDB
- Vaadin
- Spring Security

<img src="/nw1.png" width="350px" class="absolute right-10px bottom-5px"/>
<img src="/nw2.png" width="400px" class="absolute right-360px bottom-5px"/>

---

# Отправная точка

```dockerfile {none|1|3|4|6}
FROM bellsoft/liberica-openjdk-debian:24 as builder
WORKDIR /app
ADD . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction clean package
EXPOSE 8081
ENTRYPOINT java -jar /app/neurowatch/target/*.jar
```

<v-click at="1">

- Берем образ с Liberica JDK и Debian (популярная ОС для контейнеров)

</v-click>

<v-click at="2">

- Добавляем проект в базовый образ

</v-click>

<v-click at="3">

- Собираем JAR-файл (`mvn clean package`)

</v-click>

<v-click at="4">

- Запускаем

</v-click>

---

# Результат: 788MiB (820MB)

```plain {none|19,18|9-17|8|5,4}{maxHeight:'300px'}
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

# 820MB - нужно беспокоиться?


- `push` дольше: больше времени на обновление
- `pull` дольше: замедленное масштабирование
- Десятки коммитов день, каждый по 820MB: быстро закончится место<br> для хранения всех этих образов

---
class: text-center
layout: cover
background: /cubes.png
---

# Оптимизируем
# размер!

---

## Раунд 1: Многоэтапные сборки (multi-stage builds)

- Несколько FROM инструкций в одном Докерфайле
- "Чистый" итоговый образ без лишних компонентов
- Можно использовать разные базовые образы

---

# Меняем Докерфайл

````md magic-move
```dockerfile
FROM bellsoft/liberica-openjdk-debian:24 as builder
WORKDIR /app
ADD . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction clean package
EXPOSE 8081
ENTRYPOINT java -jar /app/neurowatch/target/*.jar
```

```docker
FROM bellsoft/liberica-openjdk-debian:24 as builder

WORKDIR /app
ADD . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction clean package

FROM bellsoft/liberica-openjre-debian:24
WORKDIR /app
COPY --from=builder /app/neurowatch/target/neurowatch-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```
````

---

# Результат:  343MiB (359MB)

```plain {none|7,15,16|4}{maxHeight:'300px'}
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

## Раунд 2: Легковесный Линукс для контейнеров

|                      | Alpaquita<br/>(musl) | Alpaquita<br/>(glibc) | Alpine<br/>(musl) | RHEL<br/>(Distroless UBI 10 Micro) | Ubuntu Jammy | Debian Slim |
|----------------------|----------------------|-----------------------|-------------------|------------------------------------|--------------|-------------|
| Размер на Docker Hub | 3.8MB                | 11.7MB                | 3.45MB            | 7.37MB                             | 28.17MB      | 27.8MB      |


---

# Снова меняем Докерфайл


```docker{none|1,7|2}{maxHeight:'300px'}
FROM bellsoft/liberica-runtime-container:jdk-24-musl as builder
RUN apk add --no-cache nodejs npm
WORKDIR /app
ADD . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction clean package

FROM bellsoft/liberica-runtime-container:jre-24-slim-musl
WORKDIR /app
COPY --from=builder /app/neurowatch/target/neurowatch-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```


---

# Результат: 189MiB (189MB)

```plain{none|6,2}{maxHeight:'300px'}
ID         TAG                          SIZE      COMMAND                                                                         │
│c88af46c34 neurowatch-neurowatch:latest 62.83MiB  [stage-1 3/3] COPY --from=builder /app/neurowatch/target/neurowatch-*.jar app.ja│
│<missing>                               0B        ENTRYPOINT ["java" "-jar" "/app/app.jar"]                                       │
│<missing>                               0B        EXPOSE map[8080/tcp:{}]                                                         │
│<missing>                               0B        COPY /app/neurowatch/target/neurowatch-*.jar app.jar # buildkit                 │
│<missing>                               126.43MiB WORKDIR /app  
```

---

## Раунд 3: jlink

- Инструмент в составе JDK
- Вырезает кастомный рантайм только с модулями, необходимыми приложению


---

# Узнаём, какие модули использует приложение
<br>
Билдим JAR и извлекаем Exploded JAR (libs отдельно, JAR отдельно)

```bash
mvn clean package
java -Djarmode=tools -jar target/app.jar extract
```

Используем jdeps для идентификации используемых модулей

```bash
jdeps --multi-release 24 --class-path 'app/lib/*' \
--ignore-missing-deps --list-deps app/app.jar
```

---

# Снова меняем Докерфайл

```dockerfile {none|1,2,6|8-14|16|19,20|23}{maxHeight:'300px'}
FROM bellsoft/liberica-runtime-container:jdk-all-24-musl as builder
RUN apk add --no-cache nodejs npm

WORKDIR /app
ADD . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction clean package

RUN jlink --compress=2 --strip-debug --no-header-files \
    --no-man-pages --add-modules \
    java.base,java.compiler,java.desktop,java.instrument,java.logging,\
    java.management,java.naming,java.net.http,java.prefs,java.rmi,\
    java.scripting,java.security.jgss,java.security.sasl,java.sql,\
    java.transaction.xa,java.xml,jdk.attach,jdk.jdi,jdk.jfr,jdk.unsupported \
    --output /jlink-runtime

FROM bellsoft/alpaquita-linux-base:stream-musl
WORKDIR /app

COPY --from=builder /app/neurowatch/target/neurowatch-*.jar app.jar
COPY --from=builder /jlink-runtime /jlink-runtime

EXPOSE 8080
ENTRYPOINT ["/jlink-runtime/bin/java", "-jar", "app/app.jar"]
```

---

# Результат: 141MiB (147MB)

```plain {none|6|5|2}{maxHeight:'300px'}
│ID         TAG                          SIZE     COMMAND                                                                                                     │
│8946bef82b neurowatch-neurowatch:latest 62.83MiB [stage-1 3/3] COPY --from=builder /app/neurowatch/target/neurowatch-*.jar app.jar                           │
│<missing>                               0B       ENTRYPOINT ["/jlink-runtime/bin/java" "-jar" "app.jar"]                                                     │
│<missing>                               0B       EXPOSE map[8080/tcp:{}]                                                                                     │
│<missing>                               70.12MiB COPY /jlink-runtime /jlink-runtime # buildkit                                             │
│<missing>                               8.23MiB  WORKDIR /app 
```


---
layout: image
image: size.svg
---

---
class: text-center
layout: cover
background: /cubes.png
---

## Но!
## Слой с приложением
## остаётся такого же размера
## (65MB)

---
layout: image-right
image: "/cubes-v-2.png"
---

# Что такое 65MB?

Малейшее изменение - и нужно делать push/pull 65MB

Можно что-то с этим сделать?

---
class: text-center
layout: cover
background: /cubes.png
---

# Оптимизируем push/pull!

---

## Делим слой с приложением на несколько

- Spring Boot layered JARs
- Отдельно:
  - зависимости
  - SpringBootLoader
  - SNAPSHOT-зависимости
  - приложение

---

# Докерфайл

```dockerfile {none|7,9|10|12,15-18|20}{maxHeight:'180px'}
FROM bellsoft/liberica-runtime-container:jdk-24-musl as builder
RUN apk add --no-cache nodejs npm
WORKDIR /app
ADD . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction clean package

FROM bellsoft/liberica-runtime-container:jre-24-musl as optimizer
WORKDIR /app
COPY --from=builder /app/neurowatch/target/neurowatch-*.jar app.jar
RUN java -Djarmode=tools -jar /app/app.jar extract --layers --destination extracted

FROM bellsoft/liberica-runtime-container:jre-24-slim-musl

WORKDIR /app
COPY --from=optimizer /app/extracted/dependencies/ ./
COPY --from=optimizer /app/extracted/spring-boot-loader/ ./
COPY --from=optimizer /app/extracted/snapshot-dependencies/ ./
COPY --from=optimizer /app/extracted/application/ ./
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```
<v-click at="1">

- Промежуточный этап: optimizer

</v-click>

<v-click at="2">

- Извлекаем "слоёный" JAR

</v-click>

<v-click at="3">

- Копируем слои в итоговый базовый образ

</v-click>

<v-click at="4">

- Запускаем как обычно

</v-click>

---

# Результат: 189MiB (198MB)

```plain{none|5-8|2}{maxHeight:'180px'}
│ID         TAG                          SIZE      COMMAND                                                                        │
│b811866cc5 neurowatch-neurowatch:latest 6.72MiB   [stage-2 6/6] COPY --from=optimizer /app/extracted/application/ ./             │
│<missing>                               0B        ENTRYPOINT ["java" "-jar" "/app/app.jar"]                                      │
│<missing>                               0B        EXPOSE map[8080/tcp:{}]                                                        │
│<missing>                               0B        COPY /app/extracted/application/ ./ # buildkit                                 │
│<missing>                               55.86MiB  COPY /app/extracted/snapshot-dependencies/ ./ # buildkit                       │
│<missing>                               459.4 KB  COPY /app/extracted/spring-boot-loader/ ./ # buildkit                          │
│<missing>                               0B        COPY /app/extracted/dependencies/ ./ # buildkit                                │
│<missing>                               126.43MiB WORKDIR /app
```

---
layout: image-right
image: "/pyramid.png"
---

# В чем заключается оптимизация?

В размере слоя для push/pull!

Размер активно-обновляемого слоя теперь 7MB!

<v-click at="1">А можно всё это не писать в Докерфайле? 😰</v-click>
<br>
<v-click at="2">А можно вообще без Докерфайла? 😳</v-click>

---
layout: image
image: /stream.png
---

> # Мама, смотри, я могу без Докерфайла!

<br/>

Билдпаки

<style>
h1 {
font-size: 40px;
color: #FFFFFF;
}
div {
    font-size: 24px;
    text-align: right;
}
</style>

---
layout: image-right
image: "/stream-v.png"
---

# Автоматизируем сборку образа с билдпаками

- Преобразуют исходный код приложения в контейнерный образ
- Одна команда для запуска сборки
- "Слоёный" образ + SBOM по умолчанию
- Можно конфигурировать

---

# Базовый пример использования со Спрингом

Утилита pack

```bash {1|2|3|4}
pack build neurowatch \
  --path . \
  -e BP_JVM_VERSION=24
```

Maven-плагин

```bash
mvn spring-boot:build-image
```

Gradle-плагин

```bash
gradle bootBuildImage
```

---

# Результат: 357MB

Что происходит под капотом?

```plain{none|1,2|4-10|11|13-16|33}{maxHeight:'180px'}
[INFO]  > Pulling builder image 'docker.io/paketobuildpacks/builder-noble-java-tiny:latest' 100%
[INFO]  > Pulling run image 'docker.io/paketobuildpacks/ubuntu-noble-run-tiny:0.0.20' for platform 'linux/arm64' 100%
[INFO]  > Running creator
[INFO]     [creator]     6 of 26 buildpacks participating
[INFO]     [creator]     paketo-buildpacks/ca-certificates   3.10.3
[INFO]     [creator]     paketo-buildpacks/bellsoft-liberica 11.2.4
[INFO]     [creator]     paketo-buildpacks/syft              2.16.1
[INFO]     [creator]     paketo-buildpacks/executable-jar    6.13.2
[INFO]     [creator]     paketo-buildpacks/dist-zip          5.10.2
[INFO]     [creator]     paketo-buildpacks/spring-boot       5.33.3
[INFO]     [creator]       BellSoft Liberica JRE 24.0.1: Contributing to layer
[INFO]     [creator]       Creating slices from layers index
[INFO]     [creator]         dependencies (77.4 MB)
[INFO]     [creator]         spring-boot-loader (459.4 KB)
[INFO]     [creator]         snapshot-dependencies (0.0 B)
[INFO]     [creator]         application (18.0 MB)
[INFO]     [creator]     ===> EXPORTING
[INFO]     [creator]     Adding layer 'paketo-buildpacks/ca-certificates:helper'
[INFO]     [creator]     Adding layer 'paketo-buildpacks/bellsoft-liberica:helper'
[INFO]     [creator]     Adding layer 'paketo-buildpacks/bellsoft-liberica:java-security-properties'
[INFO]     [creator]     Adding layer 'paketo-buildpacks/bellsoft-liberica:jre'
[INFO]     [creator]     Adding layer 'paketo-buildpacks/executable-jar:classpath'
[INFO]     [creator]     Adding layer 'paketo-buildpacks/spring-boot:helper'
[INFO]     [creator]     Adding layer 'paketo-buildpacks/spring-boot:spring-cloud-bindings'
[INFO]     [creator]     Adding layer 'paketo-buildpacks/spring-boot:web-application-type'
[INFO]     [creator]     Adding layer 'buildpacksio/lifecycle:launch.sbom'
[INFO]     [creator]     Adding layer 'buildpacksio/lifecycle:launcher'
[INFO]     [creator]     Adding layer 'buildpacksio/lifecycle:config'
[INFO]     [creator]     Adding layer 'buildpacksio/lifecycle:process-types'
[INFO]     [creator]     Saving docker.io/library/neurowatch:0.0.1-SNAPSHOT...
[INFO]     [creator]     Adding cache layer 'paketo-buildpacks/syft:syft'
[INFO]     [creator]     Adding cache layer 'paketo-buildpacks/spring-boot:spring-cloud-bindings'
[INFO]     [creator]     Adding cache layer 'buildpacksio/lifecycle:cache.sbom'

```

<v-click at="6">Ubuntu slim: нет shell, но образ больше</v-click>
<br>
<v-click at="7">Можно взять билдпак bellsoft/buildpacks.builder:musl: меньше и с shell</v-click>
<br>
<v-click at="8">А еще билдпаком можно делать нативные образы в контейнерах</v-click>


---
class: text-center
layout: cover
background: /speed.png
---

# А если скорость старта сервиса важнее, чем размер образа?

---

# Быстрее старт - быстрее rollout

- Нужен быстрый rollout для быстрого feedback loop
- Наша маленькая демка стартует за 3.2 секунды
- Enterprise-grade приложения стартуют еще дольше

---
class: text-center
layout: cover
background: /speed.png
---

# Оптимизируем скорость старта!

---

## Раунд 1: AOT Cache

- Логическая цепочка: CDS → AppCDS → AOT Cache
- Часть Project Leyden
- Java 24: Ahead-of-Time Class Loading & Linking (JEP 483)
- Пока AOT Cache содержит метаданные об инициализированных и слинкованных классах приложения


> Improve startup time by making the classes of an application instantly available, in a loaded and linked state, when the HotSpot Java Virtual Machine starts. Achieve this by monitoring the application during one run and storing the loaded and linked forms of all classes in a cache for use in subsequent runs. Lay a foundation for future improvements to both startup and warmup time.

---

# Как использовать со Спрингом?

Не обязательно, но полезно: добавить поддержку Spring AOT

(еще больше данных в архиве!)

```xml
<plugin>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
	<executions>
		<execution>
			<id>process-aot</id>
			<goals>
				<goal>process-aot</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

---

# Как использовать со Спрингом?

```dockerfile {none|12|22-24|25-27|15}{maxHeight:'180px'}
FROM bellsoft/liberica-runtime-container:jdk-24-cds-musl as builder
RUN apk add --no-cache nodejs npm
WORKDIR /app
ADD . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction clean package

FROM bellsoft/liberica-runtime-container:jre-24-cds-musl as optimizer
WORKDIR /app
COPY --from=builder /app/neurowatch/target/neurowatch-*.jar app.jar
RUN java -Djarmode=tools -jar /app/app.jar extract --layers --destination extracted

FROM bellsoft/liberica-runtime-container:jre-24-cds-slim-musl

WORKDIR /app
ENTRYPOINT ["java", "-Dspring.aot.enabled=true", "-XX:AOTCache=app.aot", "-jar", "/app/app.jar"]
COPY --from=optimizer /app/extracted/dependencies/ ./
COPY --from=optimizer /app/extracted/spring-boot-loader/ ./
COPY --from=optimizer /app/extracted/snapshot-dependencies/ ./
COPY --from=optimizer /app/extracted/application/ ./
EXPOSE 8080

RUN java -Dspring.aot.enabled=true -XX:AOTMode=record / \
    -XX:AOTConfiguration=app.aotconf -Dspring.context.exit=onRefresh / \
    -jar /app/app.jar
RUN java -Dspring.aot.enabled=true -XX:AOTMode=create \
    -XX:AOTConfiguration=app.aotconf -XX:AOTCache=app.aot \
    -jar /app/app.jar

```

---

# Цена успеха: 293.9MiB (308MB)

```plain {2}{maxHeight:'180px'}
│ID         TAG                          SIZE      COMMAND                                                                        │
│858ba1d1e4 neurowatch-neurowatch:latest 75.87MiB  mount / from exec /bin/sh -c java -Dspring.aot.enabled=true -XX:AOTMode=create │
│<missing>                               2.00MiB   RUN /bin/sh -c java -Dspring.aot.enabled=true -XX:AOTMode=create -XX:AOTConfigu│
│<missing>                               6.72MiB   RUN /bin/sh -c java -Dspring.aot.enabled=true -XX:AOTMode=record -XX:AOTConfigu│
│<missing>                               0B        EXPOSE map[8080/tcp:{}]                                                        │
│<missing>                               0B        COPY /app/extracted/application/ ./ # buildkit                                 │
│<missing>                               0B        COPY /app/extracted/snapshot-dependencies/ ./ # buildkit                       │
│<missing>                               55.86MiB  COPY /app/extracted/spring-boot-loader/ ./ # buildkit                          │
│<missing>                               0B        COPY /app/extracted/dependencies/ ./ # buildkit                                │
│<missing>                               0B        ENTRYPOINT ["java" "-Dspring.aot.enabled=true" "-XX:AOTCache=app.aot" "-jar" "/│
│<missing>                               153.52MiB WORKDIR /app 
```

Итоговый образ больше, т.к. архив 76MB
<br>
Но! Старт быстрее на 50-60% (1.2s)


---

## Раунд 2: Native Image

- GraalVM Native Image - AOT компиляция Java-приложения
- Платформо-зависимый .exe файл
- Не требует JVM
- Стартует очень быстро и без разогрева

Но
- Долгая сборка
- Особое внимание нужно уделить динамизму в приложении (Reflection, Dynamic Proxies, etc.)

---

# Native Image в Docker

```dockerfile {none|1|5|7|9,10}{maxHeight:'180px'}
FROM bellsoft/liberica-native-image-kit-container:jdk-24-nik-24-musl as builder
RUN apk add --no-cache nodejs npm
WORKDIR /app
ADD . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction -Pnative native:compile

FROM bellsoft/alpaquita-linux-base:musl
WORKDIR /app
ENTRYPOINT ["/app/app"]
COPY --from=builder /app/neurowatch/target/native/neurowatch /app/app
```

---

# Результат: 198MB

```plain {2,5}{maxHeight:'180px'}
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
757636acfada   8 minutes ago    [stage-1 3/3] COPY --from=builder /app/neuro…   185MB     buildkit.exporter.image.v0
<missing>      55 minutes ago   COPY /app/neurowatch/target/native/neurowatc…   0B        buildkit.dockerfile.v0
<missing>      55 minutes ago   ENTRYPOINT ["/app/app"]                         0B        buildkit.dockerfile.v0
<missing>      20 hours ago     WORKDIR /app                                    8.64MB    buildkit.dockerfile.v0
```

- Один файл, который нужно полностью ребилдить

Но зато:
- В основе только Линукс
- Старт за 0,4 с

---

## Раунд 3: CRaC

- Coordinated Restore at Checkpoint - проект OpenJDK
- Снэпшот "разогретого" Java-приложения
- Как в видеоигре: поставили на паузу, начали с той же точки
- Старт за несколько миллисекунд

<br><br>
<v-click> Звучит заманчиво... В чем подвох?</v-click>


---

# CRaC - это сложно

- Снэпшот может содержать конфиденциальные данные
- Возможно, придется писать дополнительный код для успешного чекпойнта и восстановления
- Не всех технологии работают с CRaC "из коробки"
- Одним Докерфайлом не обойтись

---

# Пример изменения кода

```java {none|1-4|18-22|24-34|6-16}{maxHeight:'200px'}
public class ExampleWithCRaCRestore {
    private ScheduledExecutorService executor;
    private long startTime = System.currentTimeMillis();
    private int counter = 0;

    class ExampleWithCRaCRestoreResource implements Resource {
        @Override
        public void beforeCheckpoint(Context<? extends Resource> context) throws Exception {
            executor.shutdown();
        }

        @Override
        public void afterRestore(Context<? extends Resource> context) throws Exception {
            ExampleWithCRaCRestore.this.startTask();
        }
    }

    public static void main(String args[]) throws InterruptedException {
        ExampleWithCRaCRestore exampleWithCRaC = new ExampleWithCRaCRestore();
        Core.getGlobalContext().register(exampleWithCRaC.new ExampleWithCRaCRestoreResource());
        exampleWithCRaC.startTask();
    }

    private void startTask() throws InterruptedException {
        executor = Executors.newScheduledThreadPool(1);
        executor.scheduleAtFixedRate(() -> {
            long currentTimeMillis = System.currentTimeMillis();
            System.out.println("Counter: " + counter + "(passed " + (currentTimeMillis-startTime) + " ms)");
            startTime = currentTimeMillis;
            counter++;
        }, 1, 1, TimeUnit.SECONDS);
        Thread.sleep(1000*30);
        executor.shutdown();
    }
}
```


---

# Процесс в общих чертах

```docker {none|7|11}
FROM bellsoft/liberica-runtime-container:jdk-21-musl as builder
RUN apk add --no-cache nodejs npm
WORKDIR /app
ADD . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction clean package

FROM bellsoft/liberica-runtime-container:jre-21-crac-cds-stream-musl

WORKDIR /app
COPY --from=builder /app/neurowatch/target/neurowatch-*.jar app.jar
ENTRYPOINT ["java", "-XX:CRaCCheckpointTo=/app/checkpoint", "-jar", "/app/app.jar"]
```

<v-click at="2">Но на этом этапе мы еще не создаем снэпшот!</v-click>

---

# Первый этап
<br>
Создаем предварительный образ

```bash
docker build -t neurowatch-for-crac -f Dockerfile-crac .
```

Запускаем

```bash
ID=$(docker run --cap-add CAP_SYS_PTRACE --cap-add CAP_CHECKPOINT_RESTORE \
-p 8080:8080 --network=host -d nw-pre-crac)

```

- CAP_SYS_PTRACE для доступа к информации обо всех процессах
- CAP_CHECKPOINT_RESTORE для успешной реалимзации checkpoint/restore без root-доступа

---

# Второй этап
<br>
Делаем чекпойнт

```bash
docker exec -it $ID jcmd 129 JDK.checkpoint
```

Создаём новый образ с крэкнутым приложением

```bash
docker commit $ID cracked
```

<v-click>

А вот теперь можно запускать!

```bash
docker run --rm \
    --entrypoint java \
    --network host cracked:latest \
    -XX:CRaCRestoreFrom=/app/checkpoint
```

</v-click>

---

# Petclinic запустилась бы...
А вот наше демо - нет

- Spring Data MongoDB не поддерживает CRaC

<br>
Варианты:

- Мигрировать на другую технологию
- Запросить поддержку
- Самостоятельно имплементировать поддержку


---

# Как подружить MongoDB с CRaC

Custom `MongoClient`

```java {1|2-5|6-8|9}
public class MongoClientProxy implements MongoClient {
    volatile MongoClient delegate;

    public MongoClientProxy(MongoClient initialClient) {
        this.delegate = initialClient;
    }

    public void close() {
        delegate.close();
    }
}  
```

---

# Как подружить MongoDB с CRaC

Кастомный CRaC Resource

```java {1,2|7-11|4,8|5,9|10|13-16|18-21|20}{maxHeight:'260px'}
@Component
static public class MongoClientResource implements Resource {

    private final MongoClientProxy mongoClientProxy;
    private final MongoConnectionDetails details;

    public MongoClientResource(MongoClient mongoClientProxy, MongoConnectionDetails details) {
        this.mongoClientProxy = (MongoClientProxy) mongoClientProxy;
        this.details = details;
        Core.getGlobalContext().register(this);
    }

    @Override
    public void beforeCheckpoint(Context<? extends Resource> context) {
        mongoClientProxy.delegate.close();
    }

    @Override
    public void afterRestore(Context<? extends Resource> context) {
        mongoClientProxy.delegate = MongoClients.create(details.getConnectionString());
    }
}
```

---

# Как подружить MongoDB с CRaC

Нужно заменить `MongoClient` в контексте

```java {1,3|4|2,5}
@Bean
@Primary
public MongoClient mongoClient(MongoConnectionDetails details) {
    MongoClient initialClient = MongoClients.create(details.getConnectionString());
    return new MongoClientProxy(initialClient);
}
```

<v-click at="3">
А вот теперь всё сработает!<br>
Билдим образ и наслаждаемся молниеносным стартом.
</v-click>

---
layout: image
image: start.svg
---


---
class: text-center
layout: cover
background: /cubes2.png
---

# Подведём итог?

---

# Размер и скорость обновлений
1. Многоэтапные сборки + маленький базовый образ = оптимальный размер
2. Используем слои для более быстрого push/pull

---

# Время старта
1. AOT Cache - ускоренный старт без "кровопролития"
2. Native Image - быстрый старт, оптимальный размер, но есть нюансы
3. CRaC - самый быстрый старт, но придётся потрудиться

---

# Удобство сборки
1. Билдпаки: не нужно писать и поддерживать Докерфайл
2. Тонкая настройка не всегда возможна


---
layout: image
image: bingo.png
---


---
class: text-center
layout: cover
background: /cubes2.png
---

## Собрали тулкит для создания ВАШЕО идеального образа
## Теперь: Mix & Match!

---
layout: image-right
image: "/qr.png"
---

# Спасибо за внимание!

- <logos-bluesky /> @cat-edelveis.bsky.social
- <logos-twitter /> cat_edelveis
- <logos-linkedin-icon /> cat-edelveis
- <logos-youtube /> @cbrjar