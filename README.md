# TP DevOps

## Docker

### Définition
Docker permet de faciliter la déployabilité d'applications distribuées de plusieurs manières:
-  En générant une image qui doit être la plus légère possible
- En générant un runtime (tout ce qu'il faut pour exécuter l'application)
C'est différent d'une VM car l'image Docker est identique pour tous les environnements.
Cela accélère le lancement, l'arrêt et la création d'une application via un container.

###  Bonnes pratiques
On ne met pas toute une application dans un container.
Chaque processus doit avoir son propre container.
Les images Docker doivent être versionnées.
Les images à utiliser doivent de préférente être officielles, de la plus petite taille possible, et d'une version précise (il faut éviter latest)

### Autres
Un `Docker daemon` est un service lancé en arrière plan sur un hôte afin de contrôler la phase de construction, de lancement et de distribution d'un container Docker.
Un `Docker client` permet à un client à travers un terminal de communiquer avec un `Docker daemon`.
Le `Docker store` est un espace regroupant les images officielles et de confiance d'entreprises et d'applications.
### Créer une image Docker
1. Créer un répertoire pour l'image et s'y rendre
2. Créer un Dockerfile
```
FROM postgres:11.6-alpine
ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd
COPY sqlscript /docker-entrypoint-initdb.d`
```
> **Remarque : **
Un `Dockerfile` est un fichier texte qui contient une liste de commandes que les démons Docker appellent à la création d'une image Docker. Il contient aussi toutes les informations dont Docker a besoin pour lancer une application.
3. (facultatif) Créer un fichier requirements.txt avec la bonne version de l'application à utiliser
> **Remarque : **
`FROM`  permet de partir d'une image spécifique et doit obligatoirement débuter le fichier 
`ENV`  permet de préciser certaines variables d'environnement
`COPY` permet de copier un fichier ou un répertoire sur notre image Docker

### Créer un build de l'image Docker
```
docker build -t Username/ImageName .
```
> **Remarque :**
`-t` permet d'attribuer un tag à un build
Dans le cadre du tp  `docker build -t lucas/tp1 .`

## Lancer un container Docker
```
docker run -p PortExposé:PortHost --name Name Username/ImageName
```
> **Remarque :**
`-d` permet de lancer le container en arrière plan
`-it` permet d'obtenir un tty intéractif dans le container et est donc très utile pour le débug
`-name` permet de spécifier un nom au container Docker
`-v /my/own/datadir:/var/lib/postgresql/data` permet de rendre les données persistantes
`-e` permet de préciser certaines variables d'environnement et donc d'éviter d'écrire le mot de passe d'une application en dur dans le code
Dans le cadre du tp, `8888` correspond au port exposé et `5432` correspond au port host

### Arrêter/supprimer un container Docker
Pour arrêter un container Docker lancé : `docker stop NomduContainer`
Pour supprimer un container Docker : `docker rm NomduContainer`
Pour arréter et supprimer un container Docker : `docker rm -f NomduContainer`

### Autres commandes
Pour lister les containers Docker lancés : `docker container ps`.
> **Remarque :**
L'argument `-a` permet d'ajouter les processus morts

Pour en savoir plus sur une image Docker : `docker inspect NomImage`
Pour télécharger une image Docker : `docker pull NomImage`

Pour trouver l'`hostname` sur une machine Windows ou Mac : `docker-machine ip default`.

## TP1 - Docker

### Configuration du container pour la base de donées
1. Création du répertoire nommé `databse`
2. Création d'un Dockerfile
```
FROM postgres:11.6-alpine
ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd
COPY sqlscript /docker-entrypoint-initdb.d
```
3. Création de l'image Docker avec la commande `docker build -t lucas/database .`
4. Lancer le container de la base de données avec la commande `docker run -p 8888:5432 --name database lucas/database`
5. Placer les scripts de génération des tables postgreSQL dans un dossier nommé `sqlscript`
6. Rebuilder l'image et relancer le container en rendant les données persistantes

### Le multistage build
One approach to keeping Docker images small is using multistage builds. A multistage build allows you to use multiple images to build a final product. In a multistage build, you have a single Dockerfile, but can define multiple images inside it to help build the final image.
Le multistage build permet de conserver de petites images Docker. Cela autorise l'utilisation de plusieurs images afin de construire un produit. Nous avons donc un seul Dockerfile qui défini plusieurs images l'aidant à construire une image finale.

Dans le cadre du TP, voici le Dockerfile créé :
```
# Build
FROM maven:3.6.3-jdk-11 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM openjdk:11-jre
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
ENTRYPOINT java -jar myapp.jar
```
Il est divisé en deux parties :
- La première s'occupe de la création de l'image en important maven, qui permet la compilation et le déploiement d'un projet java. `ENV` initialise une variable d'environnement, ici un chemin d'accès. `WORKDIR` définit le répertoire de travail. `COPY` permet d'importer le fichier `pom.xml` ainsi que le répertoire `src`.
- La seconde s'occupe du lancement du container Docker en important une implémentation de référence pour java `openjdk`. De la même manière on définit un repertoire de travail. Puis la copie de l'application java. Enfin, `ENTRYPOINT` définit l'exécutable à lancer au démarrage du container.

### Reverse proxy
Un reverse proxy permet à un utilisateur d'internet d'accéder à un serveur interne.

### Le Docker compose
Docker compose est un outil qui permet de décrire dans un fichier `yaml` et gérer en ligne de commande plusieurs containers comme un ensemble de services interconnectés.

Il faut créer un repertoire nommé `linkapplication` puis un autre nommé `dockercompose`.
Enfin, il faut générer un fichier nommé `docker-compose.yml`.

Voici la composition du fichier :
```
version: '3.7'
services:
  api:
    build: "../../apibackend/MultistageBuild/"
    networks:
      - "my-network"
    depends_on:
      - "db"
  db:
    build: "../../database/"
    networks:
      - "my-network"
  rproxy:
    build: "../../httpserver/reserveproxy/"
    ports:
      - "80:80"
    networks:
      - "my-network"
    depends_on:
      - "web"
      - "api"
  web:
    build: "../../httpserver/apache/"
    networks:
      - "my-network"
networks:
  my-network:
    name: networklucas
```
Pour générer et lancer un Docker compose, il suffit de faire suivre ces deux commandes :
```
docker compose build
docker compose run
```

### Publier sur Docker
La première commande permet de se connecter à son compte Docker `docker login`
Ensuite, il faut tagguer l'image avec un numéro de version `docker tag Username/ApplicationName:X.X
Enfin, il faut publier sur Docker avec la commande `docker push Username/ApplicationName:

> **Remarque :**
Dans le cadre du tp, `Username/ApplicationName` correspond à `lucasthorelcpe/tp-devops-cpe` par exemple.

Publier sur un repository distant permet de le rendre accessible à tous.
