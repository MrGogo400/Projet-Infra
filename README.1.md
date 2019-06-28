
## Le serveur Web : apache

  
  
On installe le serveur apache et son module permettant de gérer PHP plus tard :  
  
Code :

    apt-get install apache2 libapache2-mod-php7.0

  
  
A partir de là, si on accède au serveur via son adresse IP, on a la page d'Apache, le serveur web fonctionne.  
  

![](https://www.linuxtricks.fr/upload/debian-apache.png)

  
  
On peut s'assurer que le service démarrera automatiquement au démarrage :  
  
Code :

    systemctl enable apache2


La configuration d'Apache se fait via le fichier **/etc/apache2/apache2.conf**. C'est la configuration générale :  
  
Code :

    vi /etc/apache2/apache2.conf

## PHP

  
  
D'abord, on installe PHP. Sous Debian, PHP par défaut est PHP5. Il est obsolète, on va lui préférer PHP7 :  
  
Code :

    apt-get install php7.0 php7.0-cli

  
  
A ce stade, PHP est installé, mais on n'a pas grand chose. Il faut installer des modules en fonction des besoins. Avec PHP 7, les modules sont nommés de la manière suivante : **php7.0-xxx**. On peut les lister avec :  
  
Code BASH :

apt-cache search php7.0-

  
  
Pour notre utilisation, on va installer cli :  
  
Code :

    apt-get install php7.0-cli

  
  
Pour interragir avec SQL :  
Code :

    apt-get install php7.0-mysql

  
Ce paquet générique fournit **php7.0-mysqli** et **php7.0-pdo-mysql**.  
  
Une fois tout ça installé, on recharge apache :  
  
Code :

    systemctl reload apache2

  
  
On va tester le bon fonctionnement de PHP.  
On se rend dans le répertoire par défaut de la racine d'apache :  
  
Code :

    cd /var/www/html

  
  
Et on créé un fichier dans lequel on demande d'afficher les infos PHP :  
  
Code :

    echo "<?php phpinfo(); ?>" > test.php

  
  
Ensuite, on affiche notre page via le navigateur :  
  

![](https://www.linuxtricks.fr/upload/debian-php-avec-apache.png)

  
  
On peut éditer les options de PHP via le fichier **/etc/php/7.0/apache2/php.ini** ou créer un fichier personnalisé dans **/etc/php/7.0/apache2/conf.d**.  
  
  

## MariaDB : La base de données

  
  
Maintenant, il ne reste plus que le moteur de base de données à installer.  
  
On installe donc MariaDB, le serveur :  
  
Code :

    apt-get install mariadb-server

  
  
Une fois fait, on démarre l'installation :  
  
Code :

    mysql_secure_installation

  
  
Une série de questions vont s'afficher.  
  
On définit le mot de passe root :  
Code :

    Change the root password? [Y/n] Y
    New password:
    Re-enter new password:
    Password updated successfully!

  
  
On supprime les utilisateurs anonymes, les connexions distantes de root, etc...  
Code :

    Remove anonymous users? [Y/n] Y
    Disallow root login remotely? [Y/n] Y
    Remove test database and access to it? [Y/n] Y
    Reload privilege tables now? [Y/n] Y

  
  
On teste la connexion à la base de données :  
  
Code :

    mysql -u root -p

  
  
On créé ensuite un utilisateur avec tous les droits pour ne pas utiliser root :  
  
Code SQL :

    CREATE USER 'hugo'@'localhost' IDENTIFIED BY 'mdp';
    GRANT ALL PRIVILEGES ON *.* TO 'adrien'@'localhost' WITH GRANT OPTION;
    FLUSH PRIVILEGES;

  
  
Et on se déconnecte :  
Code SQL :

    exit

  
  
On peut s'assurer que le service se lance à chaque démarrage du serveur :  
Code :

    systemctl enable mariadb

  
  
Notre serveur LAMP est installé.

Installe docker : 

[https://docs.docker.com/install/linux/docker-ce/debian/](https://docs.docker.com/install/linux/docker-ce/debian/)

## Suite à ça on installe installe les applications web que l'on souhaite.

### NetData :

    bash <(curl -Ss https://my-netdata.io/kickstart.sh)

L'installation ce fait en une seul ligne de commande.

### Cloud-Torrent :

    docker run -d -p 3000:3000 -v /path/to/my/downloads:/downloads jpillora/cloud-torrent
Vue que c'est un docker "l'installation" ce fait aussi en une seul ligne de commande.

Dans nos DNS on ajoute des sous-domaines : 
https://i.imgur.com/3bwtS50.png


On modifie les virtual-host pour les faire fonctionner avec les sous-domaines : 


    <VirtualHost *:80>
      ServerAdmin hugomarques.hugomarques@hugomarques.fr
      DocumentRoot /var/www/html
      <Directory /var/www/html/>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      # Possible values include: debug, info, notice, warn, error, crit,
      # alert, emerg.
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    
    <VirtualHost *:443>
        ServerName hugomarques.fr  
        ServerAlias www.hugomarques.fr
        ServerAdmin hugomarques.hugomarques@hugomarques.fr
        DocumentRoot /var/www/html/          
     
           # directives obligatoires pour TLS
            SSLEngine on
            SSLCertificateFile    /etc/letsencrypt/live/hugomarques.fr-0001/fullchain.pem
            SSLCertificateKeyFile   /etc/letsencrypt/live/hugomarques.fr-0001/privkey.pem
     
            ErrorLog /var/log/apache2/error.hugomarques.fr.log
            CustomLog /var/log/apache2/access.hugomarques.fr.log combined
    </VirtualHost>
    
    <VirtualHost *:80>
    	ServerName wp.hugomarques.fr
      ServerAlias www.wp.hugomarques.fr
    	ServerAdmin contact@hugomarques.fr
    	DocumentRoot /var/www/wp/
    </VirtualHost>
    
    <VirtualHost *:80>
    	ServerName torrent.hugomarques.fr
    	ServerAlias www.torrent.hugomarques.fr
    	ProxyPreserveHost On
    	ProxyPass / http://hugomarques.fr:3000/
    	ProxyPassReverse / http://hugomarques.fr:3000/
    </VirtualHost>
    
    <VirtualHost *:80>
    	ServerName monitor.hugomarques.fr
      ServerAlias www.monitor.hugomarques.fr
    	ProxyPreserveHost On
    	ProxyPass / http://localhost:19999/
    	ProxyPassReverse / http://localhost:19999/
    
      <Location />
        AuthType Basic
        AuthName "Protected site"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
        Order deny,allow
        Allow from all
      </Location>
    
    </VirtualHost>






<!--stackedit_data:
eyJoaXN0b3J5IjpbOTY1NjY5NjYsLTk2NDY2Mzg5OCwxMjIwNT
k1NTAwXX0=
-->