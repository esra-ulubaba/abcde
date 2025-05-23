1.bununla çalıştır : git clone https://github.com/esra-ulubaba/ymgdeneme.git
cd ymgdeneme
2.java dosyalarını derle : mkdir out
cd src
for %f in (*.java) do javac -d ../out %f
cd ..
3.	docker-compose.yml dosyasını proje kök dizinine ekle. İçeriği şöyle olsun:
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: userdb
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
4.postgresql'e bağlan: docker exec -it postgres psql -U postgres -d userdb
Sonrasında açılan PostgreSQL ekranında sırasıyla:
sql
KopyalaDüzenle
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (username, email) VALUES('esra','esra@example.com');
SELECT * FROM users;
Çıkmak için: \q
5. dockerfile dosyasına git: cd build/web
6.docker image başlat: docker build -t ymsfinal .
7.docker container oluştur: docker run -d -p 8090:8080 -p 4849:4848 --name ymsfinal-container ymsfinal

docker-compose up -d --build

8. 404 hatası alırsan şunu yaz: docker logs -f ymsfinal-container
9.pipeline kodu: pipeline {
    agent any

    environment {
        IMAGE_NAME = "ymsfinal" 
        CONTAINER_NAME = "ymgdeneme-container"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/esra-ulubaba/ymgdeneme.git'
            }
        }

        stage('Build Project (Optional)') {
            steps {
                bat '''
                    mkdir out
                    cd src
                    for %%f in (*.java) do javac -d ../out %%f
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
                    docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} build/web
                """
            }
        }

        stage('Stop Existing Container') {
            steps {
                bat """
                    docker stop ${CONTAINER_NAME} || exit 0
                    docker rm ${CONTAINER_NAME} || exit 0
                """
            }
        }

        stage('Run Docker Container') {
            steps {
                bat """
                    docker run -d -p 8091:8080 -p 4849:4848 --name ${CONTAINER_NAME} ${IMAGE_NAME}:${env.BUILD_NUMBER}
                """
            }
        }
    }
}

github için: cd C:\Users\WINUSER\Desktop\ymsfinal

git init
git add .
git commit -m "YMSFinal Docker WebApp initial commit"
git branch -M main
git remote add origin https://github.com/KULLANICI_ADIN/ymsfinal.git
git push -u origin main

FROM openjdk:8-jdk

ENV GLASSFISH_VERSION 4.1.1
ENV GLASSFISH_HOME /glassfish

# GlassFish indir
RUN wget http://download.oracle.com/glassfish/4.1.1/release/glassfish-${GLASSFISH_VERSION}.zip \
    && unzip glassfish-${GLASSFISH_VERSION}.zip -d / \
    && mv /glassfish4 ${GLASSFISH_HOME} \
    && rm glassfish-${GLASSFISH_VERSION}.zip

# PostgreSQL driver’ı kopyala
COPY lib/postgresql-42.7.3.jar ${GLASSFISH_HOME}/glassfish/domains/domain1/lib/ext/

# Projeyi kopyala
COPY . /usr/src/app
WORKDIR /usr/src/app

# WAR değil direkt deploy et
RUN cp -r . ${GLASSFISH_HOME}/glassfish/domains/domain1/autodeploy/

EXPOSE 8080 4848
CMD ["sh", "-c", "${GLASSFISH_HOME}/bin/asadmin start-domain -v"]
