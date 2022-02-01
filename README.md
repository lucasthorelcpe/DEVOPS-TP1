#TP1
## Création de l'image Docker
1. Créer un répertoire pour l'image et s'y rendre
2. Créer un Dockerfile
```
FROM postgres:11.6-alpine
ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd
COPY sqlscript /docker-entrypoint-initdb.d`
```
3. (facultatif) Créer un fichier requirements.txt avec la bonne version de l'application à utiliser
> **Remarque : **
`FROM`  permet de partir d'une image spécifique et doit obligatoirement débuter le fichier 
`ENV`  permet de préciser certaines variables d'environnement
`COPY` permet de copier un fichier ou un répertoire sur notre image Docker

## Création du build de l'image Docker
```
docker build -t Username/ApplicationName .
```
> **Remarque :**
`-t` permet d'attribuer un tag à un build
Dans le cadre du tp  `docker build -t lucas/tp1 .`

## Lancement du container Docker
```
docker run -p 8888:5432 --name tp1 lucas/tp1
```
> **Remarque :**
`-d` permet de lancer le container en arrière plan
`-v /my/own/datadir:/var/lib/postgresql/data` permet de rendre les données persistantes
`-e` permet de préciser certaines variables d'environnement et donc d'éviter d'écrire le mot de passe d'une application en dur dans le code

## Arrêt et suppression du container Docker
```
docker stop test
docker rm test
docker rm -f tesst
```
