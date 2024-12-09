# Настройка Jenkins и Nexus для CI/CD в доккере

Этот проект описывает шаги, которые мы предприняли для настройки Jenkins и Nexus 
для автоматического деплоя артефактов с использованием Maven. 
В нем мы объясним, как интегрировать Jenkins с Nexus для автоматической сборки и деплоя.

## Содержание

1. [Введение](#введение)
2. [Настройка Nexus](#настройка-nexus)
3. [Настройка Jenkins](#настройка-jenkins)

## Введение

Для автоматизации процесса сборки и деплоя артефактов был настроен CI/CD пайплайн с использованием Jenkins и Nexus. 
В процессе работы мы настроили Maven для работы с Nexus, 
интегрировали Jenkins с Nexus и настроили автоматическую сборку и деплой артефактов.

## Настройка Nexus

1. **Установка Nexus**:
Мы установили Nexus Repository Manager, который будет использоваться для хранения артефактов (JAR, WAR и т.д.).
- не забудь создать <b>volumes в проекте
```dockerfile
version: '3.8'

services:
  nexus:
    image: sonatype/nexus3:latest
    container_name: nexus
    ports:
      - "8081:8081"
    volumes:
      - E:\javaProject\jkin-api\nexusdata:/nexus-data
    environment:
      - INSTALL4J_ADD_VM_PARAMS=-Xms512m -Xmx512m -XX:MaxDirectMemorySize=2g
    networks:
      - jenkins_network

networks:
  jenkins_network:
    driver: bridge
```

2. **Создание репозитория**: 
В Nexus был создан новый репозиторий для хранения артефактов:
    - Вид и тип репозитория: Maven Hosted репозиторий
    - Название: `test-repo`
    - Политика версий: `RELEASE` <b> Если 400 ошибка, то проверь нет ли в верии сборки постфикса SNAPSHOT
    - Политика деплоя: Allow redeploy

3. **Настройка аутентификации**:
   - В Nexus настроена аутентификация с использованием логина и пароля. 
   Создан пользователь с правами на запись в репозиторий `test-repo` (выдал админа) - mcdodik/1234.

## Настройка Jenkins

1. **Установка Jenkins**:
   Мы использовали Docker для развертывания Jenkins.
```dockerfile
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: always
    user: root
    privileged: true
    ports:
      - "8080:8080" # Порт для доступа к веб-интерфейсу Jenkins
      - "50000:50000" # Порт для связи агентов Jenkins
    volumes:
      - E:\javaProject\jkin-api\jenkinsdata:/var/jenkins_home
#    environment: # пропустить установку (лучше не надо)
#      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false
    networks:
      - jenkins_network

networks:
  jenkins_network:
    driver: bridge
```

2. **Установка плагинов**:
   Для работы с Maven и Nexus были установлены следующие плагины:
    - **Pipeline Plugin**: Для создания пайплайнов в Jenkins.
    - **Maven Integration Plugin**: Для работы с Maven в Jenkins.
    - **Nexus Artifact Uploader**: Для деплоя артефактов в Nexus.

3. **Настройка инструмента Maven**:
   - В Jenkins мы настроили Maven, указав путь к установке Maven в настройках Jenkins 
   (Tools → Установка мавен, автоматическая установка). Имя maaven.
   - В разделе managed files создадим глобавльные и локальные настройки settings.xml. 
   - <b>У меня настройки не применились и мне пришлось изменять файл настроек мавена по пути </b>
     `.\jenkinsdata\tools\hudson.tasks.Maven_MavenInstallation\maaven\conf\settings.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <pluginGroups>
  </pluginGroups>
  <proxies>
  </proxies>
  <servers>
    <server>
      <id>nexus</id>
      <username>mcdodik</username>
      <password>1234</password>
    </server>
  </servers>
  <mirrors>
    <mirror>
      <id>nexus-mirror</id>
      <mirrorOf>*</mirrorOf>
      <url>http://nexus:8081/repository/test-repo/</url>
    </mirror>
    <mirror>
      <id>central</id>
      <mirrorOf>central</mirrorOf>
      <url>https://repo.maven.apache.org/maven2/</url>
    </mirror>
    <mirror>
      <id>maven-default-http-blocker</id>
      <mirrorOf>external:http:*</mirrorOf>
      <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
      <url>http://0.0.0.0/</url>
    </mirror>
  </mirrors>
  <profiles>
  </profiles>
</settings>
```

4. **Создание пайплайна**:
   Мы создали Jenkins pipeline, который включает в себя следующие этапы:
    - Сборка проекта с помощью Maven.
    - Деплой артефактов в Nexus.
```
pipeline {
    agent any

    tools {
        maven 'maaven'
    }

    stages {
        stage('Check Version') {
            steps {
                sh 'mvn -v'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Nexus') {
            steps {
                sh 'mvn deploy'
            }
        }
    }
}
```

5. **Альтернативный пайплан .yaml**
   - Необходимо установить плагин Pipeline as YAML
```yaml
version: '1'
stages:
  - name: Build
    steps:
      - script: |
          echo "Building the project"
          mvn clean install

  - name: Test
    steps:
      - script: |
          echo "Running tests"
          mvn test

  - name: Deploy
    steps:
      - script: |
          echo "Deploying to Nexus"
          mvn deploy -s /path/to/settings.xml
```


## Настройка пайплайна - изголяемся как можем