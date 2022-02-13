# TP DevOps

## Docker

### Définition
Docker permet de faciliter la déployabilité d'applications distribuées de plusieurs manières:
-  En générant une image qui doit être la plus légère possible
- En générant un runtime (tout ce qu'il faut pour exécuter l'application)
C'est différent d'une VM car l'image Docker est identique pour tous les environnements.
Cela accélère le lancement, l'arrêt et la création d'une application via un container.
Docker s'appuie sur LXC (`Linux Container`) et ajoute de la gestion réseau.

###  Bonnes pratiques
On ne met pas toute une application dans un container.
Chaque processus doit avoir son propre container.
Les images Docker doivent être versionnées.
Les images à utiliser doivent de préférente être officielles, de la plus petite taille possible, et d'une version précise (il faut éviter latest)

### Autres
Un `Docker daemon` est un service lancé en arrière plan sur un hôte afin de contrôler la phase de construction, de lancement et de distribution d'un container Docker.
Un `Docker client` permet à un client à travers un terminal de communiquer avec un `Docker daemon`.
Le `Docker store` est un espace regroupant les images officielles et de confiance d'entreprises et d'applications.

La scalabilité horizontal admet l'ajout de serveurs.
La scalabilité verticale admet l'ajout de ressources serveurs.

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

## Git

### Définition
Git est un système permettant de gérer de multiples révisions d'un projet. C'est le VSC `Version Control System`.
Git a été créé par Linus Torvalds, le créateur du kerner Linux.
L'algorithme de somme de contrôle interne de Git est le SHA-1, produisant une valeur de hachage vulnérable de 160 bits, publié en 1995.

### Principes
Git permet :
- La collaboration
- Le versionning de projet avec des commits immutables (historique de commits avec la dateet l'utilisateur qui a fait les changements)
- Résolution de conflits

Le répertoire de travail contient un `.git` qui stock la configuration du repository et les métadonnées (snapshots, commits, etc.).
Un `commit` est une unité de changement.

### Les commandes
Pour afficher le détail des changements du répertoire de travail : `git diff`
Pour afficher les informations sur les zones de transit : `git status`
Pour afficher l'historique des commits : `git log`

Pour s'authentifier : `git config --global user.name NomUtilisateur` puis `git config --global user.email EmailUtilisateur`

Pour initialiser le repository `cd DossierProjet` puis `git init`

Pour prendre en compte un nouvel élément : `git add` 
Pour prendre en compte tous les nouveaux éléments : `git add .`

Pour commit : `git commit -m "Message"`

Il est possible d'ajouter un fichier `.gitignore` afin d'éviter de prendre en compte certains éléments

Pour créer une branche : `git branch NomBranche`
Pour changer de branche : `git checkout NomBranche`

## CI/CD

### Définition
Le CI/CD désigne une méthode de travail qui vise à vérifier constamment, à la moindre modification du code, que les révisions ne provoquent pas de régression ou de dysfonctionnement. Cette approche automatise une partie du développement des apps, en instaurant des dispositifs de surveillance pour s'assurer que tout fonctionne bien. Ainsi, les développeurs n'ont pas à se soucier d'éventuels problèmes d'intégration, et peuvent se concentrer sur l'amélioration constante de leur code. Le sigle "CI" signifie l'intégration continue, et le sigle "CD" le déploiement continu.

## Ansible

### Définition
Ansible est un logiciel libre de gestion des configurations qui automatise le déploiement des applications et la livraison continue des mises à jour. Disponible sous licence GPL v3, il s'adosse au protocole de cryptage réseau SSH (pour Secure Socket sHell) pour déployer les mises en production de code via des fichiers décrivant les configurations applicatives cibles au format Json (pour JavaScript object notation). 

## TP1 - Docker

### Configuration du container pour la base de donées
1. Création du répertoire nommé `database`
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

## TP2 - Git & GitHub

### Setup Github Actions

La commande `--file /path/to/pom.xml` va effacer vos précédentes constructions du cache puis il va recompiler chaque module de l'application et enfin il va exécuter les tests unitaires et les tests d'intégration

Les tests de composants ont pour but de tester les différents composants du logiciel séparément afin de s'assurer que chaque élément fonctionne comme spécifié. Ces tests sont aussi appelés test unitaires et sont généralement écrits et exécutés par le développeur qui a écrit le code du composant.

Les tests d'intégrations sont des tests effectués entre les composants afin de s'assurer du fonctionnement des interactions et de l'interface entre les différents composants. Ces tests sont également gérés, en général, par des développeurs.

`Testcontainers` offre une bibliothèque Java permettant d'instancier dans des containers Docker de nombreux systèmes comme des bases de données, les services AWS, Redis, Elasticsearch…
Chaque test peut ainsi lancer son propre container Docker avec les systèmes dont il dépend et y insérer son propre jeu de données. Il devient alors très simple de tester les interactions entre notre système et celui lancé via `Testcontainers`.

Le fichier `.main.yml` stocké dans `.github/workflows/` contient l'architecture de la pipeline. Chaque job représente une étape de ce que le DevOps veut faire. Chaque job sera exécuté en parallèle, sauf si un lien est spécifié.

Voici le fichier en question : 
```
name: CI DevOps 2022 CPE
on:
  push:
    branches: 
      - main
  pull_request:
jobs:
  test-backend:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2.3.3
      - uses: actions/setup-java@v2
        with:
          distribution: 'temurin'
          java-version: '11'
          cache: 'maven'
    	#finally build your app with the latest command
      - name: Build and test with Maven
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Needed to get PR information, if any
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: mvn clean verify sonar:sonar -Dsonar.projectKey=lucasthorelcpe_DEVOPS-TP1 -Dsonar.organization=lucasthorelcpe -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }} --file apibackend/MultistageBuild/simple-api/pom.xml
  build-and-push-docker-image:
    needs: test-backend
    # run only when code is compiling and tests are passing
    runs-on: ubuntu-latest
    # steps to perform in job
    steps:
    - name: Login to DockerHub
      run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{secrets.DOCKERHUB_TOKEN }}
    - name: Checkout code
      uses: actions/checkout@v2
    - name: Build image and push backend
      uses: docker/build-push-action@v2
      with:
        push: ${{ github.ref == 'refs/heads/main' }}
      # relative path to the place where source code withDockerfile is located
        context: ./apibackend/MultistageBuild
      # Note: tags has to be all lower-case
        tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:backend
    - name: Build image and push db
      uses: docker/build-push-action@v2
      with:
        push: ${{ github.ref == 'refs/heads/main' }}
      # relative path to the place where source code withDockerfile is located
        context: ./database
      # Note: tags has to be all lower-case
        tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:database
    - name: Build image and push reverse proxy
      uses: docker/build-push-action@v2
      with:
        push: ${{ github.ref == 'refs/heads/main' }}
      # relative path to the place where source code withDockerfile is located
        context: ./httpserver/reserveproxy
      # Note: tags has to be all lower-case
        tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:rproxy3
    - name: Build image and push apache
      uses: docker/build-push-action@v2
      with:
        push: ${{ github.ref == 'refs/heads/main' }}
      # relative path to the place where source code withDockerfile is located
        context: ./httpserver/apache
      # Note: tags has to be all lower-case
        tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:web
    - name: Build image and push front
      uses: docker/build-push-action@v2
      with:
        push: ${{ github.ref == 'refs/heads/main' }}
      # relative path to the place where source code withDockerfile is located
        context: ./devops-front-main
      # Note: tags has to be all lower-case
        tags: ${{secrets.DOCKERHUB_USERNAME}}/tp-devops-cpe:front1
```
On spécifie en début de fichier le nom de la pipeline, la branche ou est poussée l'application.
Ensuite, on spécifie les choses à faire.
Chaque `job` est spécifié par un nom, puis décrit toutes les étapes à réaliser afin de valider cette partie du test.

> **Remarque : **
Il est extrêmement important de ne rentrer aucune donnée sensible comme des mots de passe dans nos fichiers.
Il faut donc spécifier les références à Github Actions dans un fichier spécifique inclus dans le `.gitignore`.
Si l'erreur est faite une fois de publier les mots de passe en clair sur Git, c'est déjà trop tard et il faudra tout modifier.

Il faut spécifier `needs: build-and-test-backend` dans le `job` nommé `build-and-push-docker-image` car il ne peut pas être réaliser sans.

Pousser des images docker permet de les stocker et ligne et de les rendre disponible au plus grand nombre.

### SonarCloud
SonarCloud aide les DevOps à rendre le code plus sécurisé. Il offre 2 indicateurs. Le premier c'est la vulnérabilité. SonarCloud est capable d'identifier et de vous dire les parties votre code qui pourraient être exploités par des hackers .
Et le deuxième paramètre ce sont les Security Hotspots. Cet indicateur va relever toutes les parties de code sensible en termes de sécurité et qui nécessite un examen manuel pour que vous puissiez évaluer s'il existe vraiment une vulnérabilité ou non.

Afin de le configurer, il suffit de créer une organisation et d'ajouter un `projectkey` et un `organisationkey` en ligne avec GitHub.

## TP3 - Ansible

La configuration d'Ansible se décompose en plusieurs éléments :
- Un fichier `playbook.yml` ou sont mentionnés toutes les tâches qu'Ansible doit exécuter (configuration de l'environnement de production, étapes de déploiement et de mise à jour, etc.).
- Un dossier `inventories` qui contient un fichier `setup.yml` . Il définit des variables comme la clé ssh et spécfit l'hôte.
- Un dossier `roles` qui permet de bien séparer chaque bloc de notre architecture.

Pour créer un nouveau rôle : `ansible-galaxy init roles/NomRôle`

Il est possible de faire un test de ping en utilisant la commande `ansible all -i inventories/setup.yml -m ping`

Il est aussi possible de connaitre l'OS de la machine hôte en utilisant `ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution"`

Pour exécuter un playbook : `ansible-playbook -i inventories/setup.yml playbook.yml`

Voici le contenu du playbook :
```
- hosts: all
  gather_facts: false
  become: yes
  roles:
    - docker
    - network
    - database
    - app
    - web
    - proxy
    - front
```

On définit ici tous les rôles à exécuter dans le playblook.

Dans le fichier `main.yml` définit dans `roles/docker/tasks/` est contenu :
```
# tasks file for roles/docker
- name: Clean packages
  command:
    cmd: dnf clean -y packages
- name: Install device-mapper-persistent-data
  dnf:
    name: device-mapper-persistent-data
    state: latest
- name: Install lvm2
  dnf:
    name: lvm2
    state: latest
- name: add repo docker
  command:
    cmd: sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
- name: Install Docker
  dnf:
    name: docker-ce
    state: present
- name: install python3
  dnf:
    name: python3
- name: Pip install
  pip:
    name: docker
- name: Make sure Docker is running
  service: name=docker state=started
  tags: docker
```
Ce fichier caractérise les tâches à réaliser par ce rôle.
On définit un nom à chaque, tâche, puis une série de commandes.
