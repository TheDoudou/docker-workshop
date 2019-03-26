Présentation de Docker


# Utilisation de docker :

### Partie 1 préparation :

Se connecter au serveur avec les infos fournis sur le papier.
`ssh -p 200x root@10.203.0.xxx`
Vous pouvez changer le mot de passe avec la commande passwd
S'il vous plait connectez vous uniquement à votre machine.

Je peux voir le statut de chacun donc pas de glandouille possible ;)

Installer docker
`curl -fsSL https://get.docker.com | sh`
Lancer docker
`service docker start`
Voir si il tourne
`service docker status`
Tester si cela marche
`docker run hello-world`

### Partie 2 utilisation :

Pour start/stop un container.
`docker start/stop ID/name`

Avoir les infos de docker
`docker info`

Effacer le container hello-world.
`docker container ls -a` 
`docker container rm ID/name`

Effacer l'image hello-world.
`docker image ls -a`
`docker image rm ID/name`

Chercher dans le hub (et bien foutu en plus)
`docker search httpd` (cherche httpd)

Pour plus de visuel vous pouvez utilisé ctop :
`apt-get install wget`

`wget https://github.com/bcicen/ctop/releases/download/v0.7.2/ctop-0.7.2-linux-amd64 -O /usr/local/bin/ctop`

`chmod +x /usr/local/bin/ctop`

`ctop`

## Utilisation avec "partage de dossier" :

### Partie 3 serveur web (apache only il existe une version php-apache ;) ) :

Créer un dossier **web** dans /root (`mkdir web`) et s'y rendre (`cd web`).

Créer un fichier index.html avec juste Hello World dedans (ou autre).

`echo "<h1>Hello World</h1>" > index.html`

Lancer un container apache2 avec ce dossier comme racine web.

`docker run -dit --name apache-app -p 80:80 -v /root/web:/usr/local/apache2/htdocs/ httpd:2.4`

En gros les option **d** pour détached (cela évite d'etre dans le container apres le run), it pour avoir un terminal si besoin dans le container.

Le **p** est pour le port, le **v** pour le partage d'un dossier /path/to/host:/path/inside/container.

Connectez vous sur le site web http://10.203.0.xxx:800x (remplacé les x par vos infos).



Garder ce dossier web il sera utile pour la suite.


Vous pouvez effacer le container **apache-app**

`docker container rm apache-app`


### Partie 4 server node/web + mysql :


Créer un dossier **app** dans /root et s'y rendre (`cd ../; mkdir app; cd app` (oui le ; enchaine les commandes sur une ligne ;)).

Charger un super projet node

`curl -O http://10.203.0.82:8080/node.js`

Lancer un container node qui execute ce script.

`docker run -dit --name node-app -p 7xxx:3000 -v /root/app:/usr/src/app -w /usr/src/app node node node.js`

Les options le **w** indique le dossier de départ du container.

Le 1er node est l'image à télécharger, la suite "node node.js" est la commande (CMD) qui sera executé au lancement.

Connectez vous sur le site web http://10.203.0.xxx:7xxx (remplacé les x par vos infos).

Vous pouvez voir les log.

`docker logs node-app`

Un compteur augmente à chaque visite


Pour se rendre sur le process du container.

`docker attach ID/name` (ctrl-c stop le process/container)


Pour créer un nouveau shell dans le container

`docker exec -it ID/name bash` (ou systemd ou autre shell / commande que vous voulez faire)




## Utilisation en standalone :


### Partie 5 WordPress et base de donnée :


Vous allez avoir besoin de l'image de wordpress et MariaDB (clone de mysql).

`docker pull wordpress`

`docker pull mariadb`


Ensuite il faut lancer un container avec l'image de MariaDB avec un mot de passe root.

`docker run -d --name mariadb -e MYSQL_ROOT_PASSWORD=motdepasse mariadb`


L'option **e** est une variable d'environement dans le container (dans notre cas le mot de passe)
Enfin lancer WP avec un lien à la DB.

`docker run -d --name wordpress -p 7xxx:80 --link mariadb:mysql wordpress`

Nouvelle option **link** container_name:nom_ressource permet le lien entre 2 container à une resource (PID) précis, le container WP n'a pas besoin du mot de passe root il l'a aussi grace au link (un container à les **e** d'un autre container tant qu'ils sont link).

Connectez vous sur le site web http://10.203.0.xxx:7xxx (remplacé les x par vos infos).
Faite une configuration de WP et testez.


Pour stop WP

`docker stop wordpress`


Pour le start

`docker start wordpress`


Et pour restart ... pas besoin si ?

(restart si jamais)


## Création d'une image


### Partie 6 création d'une image avec Dockerfile :

Se rendre dans le dossier web de la partie 3 (`cd /root/web`).

Charger le fichier Dockerfile (toujours avec un D)

`wget http://10.203.0.82:8080/Dockerfile`

Contenus du Dockerfile (`nano Dockerfile`):

```
# L'image qui sera créer le sera depuis une image de httpd version 2.4
FROM httpd:2.4

# Copie le dossier dans la nouvelle image
COPY ./ /usr/local/apache2/htdocs/

# Info sur le créateur de l'image

MAINTAINER TonBlazz <email@mail.com>

# Ouvre le port 80 d'apache
EXPOSE 80

###
#
# Optionnel
# Vous pouvez exposer directement sur le port 7xxx (pas le 800x cela ne marchera pas)
# Pour cela il faut changer le port dans le fichier de config
# 2 solutions possible :
# La 1ere il faut le fichier de config de base d'apache (et la bonne version)
# Ensuite il faut la mettre ou il faut dans le container
# COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf
# La 2eme consite à modifier le fichier à la créaction de l'image
# RUN sed -i -e 's/^#\(Listen 7xxx\)/\1/' conf/httpd.conf
# 
# Enfin changer le EXPOSE par le 7xxx
#
# Pour bien faire il faudrait mettre l'index web dans un dossier www
# et mettre COPY ./www /usr/local[...] car la le Dockerfile (ou httpd.conf)
# sont aussi à la racine du site web
#
###

CMD [“httpd-foreground”] # Executera httpd en tache de fond au lancement de l'image
```


Pour créer l'image

`docker build /root/web/ -t my-web-app:v1`

Pour la tester

`docker run -p 80:80 --name appweb my-web-app:v1`

(pourquoi pas de -d pour voir si cela marche comme il faut)


Ensuite direction http://10.203.0.xxx:800x pour voir votre belle page


Pour clean si vous avez envie/besoin (le force efface les containers qui sont lié)

`docker image rm --force my-web-app:v1`


Ceux qui auront bien suivi du coup ce demande,

Mais le Dockerfile de httpd à aussi un FROM (debian),

Et le Dockerfile de debian alors ?

Lui à un FROM scratch, scratch est une image ultra minimaliste,

Ensuite un `ADD rootfs.tar.xz /` pour avoir le root

Scratch permet donc de faire des version ultra minimaliste d'une app

Du coup vous pouvez faire un FROM scratch COPY ./projet/ /projet,

et ensuite le link avec un container node/apache/etc.


Pour allé encore plus loin il existe des outils comme celui ci

https://github.com/wagoodman/dive

Cela permet de voir les couches et plus encore.

https://imagelayers.io/

Pour voir les couches


### Partie 7 encore plus loin


La on passe aux choses sérieuse.


Il faut comprendre la persistence des data.

Pour x ou y raison vous modifier le container, si vous faite un reset

il revient à son état d'origine (sur base de l'image), ou pour une simple

mise à jour du container car il y'a une nouvelle image disponible.


Pour cela il existe les volumes.


Par exemple vous lancer une image php link à une image mysql.

Vous pouvez faire `docker volume create mon_app_web`

Au moment de la création du container -v mon_app_web:/path/in/container


Cela permet de mettre à jour le container sans devoir toucher/recopier vos fichier (ou perdre le contenus d'une db ou autre).


Ensuite pour des étapes comme wordpress (ou multi-container), il existe
docker-compose 

En gros cela permet de lancer et manager plusieurs container comme une app


### Partie 8 exemple avec WP


Créer un dossier my_wordpress dans le dossier root puis s'y rendre.

`mkdir /root/wordpress/; cd /root/wordpress/`

Créer un fichier `docker-compose.yml` avec ceci dedans (`nano docker-compose.yml`) :

```
version: '1'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "80:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: {}
```

Donc cela vas créer 2 containers l'un avec la DB un autre avec WP (il a deja son serveur web).

Les volumes permette de save ce qu'il y'a dans /var/lib/mysql dans le volume db_data


l'option `restart` (existe aussi avec le docker run et peut etre modifier par la suite ce n'est pas le cas de toute les options au moment de la création du container) en cas de crash dans le container de reboot de la machine ou autre le container se relancera toujours (il existe d'autre paramètre en car d'erreur uniquement etc)


Enfin il faut lancer avec docker-compose sauf que ... il faut l'installer aussi et sans script cette fois :D

Let's go il faut telecharger et mettre l'app dans le dossier bin

`curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

Puis mettre les droits d'execution sur le programme.

`chmod +x /usr/local/bin/docker-compose`

Apres il faut créer un lien symbolique vers le dossier bin du system.

`ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`

Enfin

`docker-compose -v`


Si tout est ok

`docker-compose up -d` (-d ou pas pareil c'est le mode détaché)


Pour stoper

`docker-compose stop`


Pour effacer juste les contaiers (pas la db sql du coup)

`docker-compose down`

Pour tout effacer

`docker-compose down --volumes`


### Partie 9 la fin il faut bien


Voici des commandes docker utile pour le managment.


Affiche les container actif

`docker ps `

Affacer un contrainer (plus court que l'autre)

`docker rm ID/name`

Et une image

`docker rmi ID/name`

Stop tous les containers

`docker kill $(docker ps -q)` (kill force l'arret alors que stop coupe "proprement")

Efface les containers qui ne tourne pas

`docker rm $(docker ps -a -q)`

Efface les images (que celles qui ne sont pas utilisé)

`docker rmi $(docker images -q)`



![](http://3.bp.blogspot.com/-qJrED1Dk890/Ul5rrklcKvI/AAAAAAAAtsI/w6LU6kgXMAw/s1600/funny-cats-gifs-073-007.gif)
