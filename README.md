---


---

<h2 id="le-serveur-web--apache">Le serveur Web : apache</h2>
<p>On installe le serveur apache et son module permettant de gérer PHP plus tard :</p>
<p>Code :</p>
<pre><code>apt-get install apache2 libapache2-mod-php7.0
</code></pre>
<p
A partir de là, si on accède au serveur via son adresse IP, on a la page d’Apache, le serveur web fonctionne.</p>
<p><img src="https://www.linuxtricks.fr/upload/debian-apache.png" alt=""></p>
<p>On peut s’assurer que le service démarrera automatiquement au démarrage :</p>
<p>Code :</p>
<pre><code>systemctl enable apache2
</code></pre>
<p>La configuration d’Apache se fait via le fichier <strong>/etc/apache2/apache2.conf</strong>. C'est la configuration générale :</p>
<p>Code :</p>
<pre><code>vi /etc/apache2/apache2.conf
</code></pre>
<h2 id="php">PHP</h2>
<p>D’abord, on installe PHP. Sous Debian, PHP par défaut est PHP5. Il est obsolète, on va lui préférer PHP7 :</p>
<p>Code :</p>
<pre><code>apt-get install php7.0 php7.0-cli
</code></pre>
A ce stade, PHP est installé, mais on n’a pas grand chose. Il faut installer des modules en fonction des besoins. Avec PHP 7, les modules sont nommés de la manière suivante : <strong>php7.0-xxx</strong>. On peut les lister avec :</p>
<p>Code BASH :<p>

apt-cache search php7.0-</p>
<p>Pour notre utilisation, on va installer cli :</p>
<p>Code :</p>
<pre><code>apt-get install php7.0-cli
</code></pre>
Pour interragir avec SQL :<br>  
Code :</p>
<pre><code>

    apt-get install php7.0-mysql
</code></pre>
<p>
  
Ce paquet générique fournit <strong>php7.0-mysqli</strong> et <strong>php7.0-pdo-mysql</strong>.</p>
<p>Une fois tout ça installé, on recharge apache :</p>
<p>Code :</p>
<pre><code>systemctl reload apache2
</code></pre>
<p
On va tester le bon fonctionnement de PHP.<br>
On se rend dans le répertoire par défaut de la racine d’'apache :</p>
<p>Code :</p>
<pre><code>cd /var/www/html
</code></pre>
<p> 
Code  on créé un fichier dans lequel on demande d’afficher les infos PHP :</p>
<p>Code :</p>
<pre><code>echo "&lt;<?php phpinfo(); ?&gt;" &gt; test.php
</code></pre>
<p>Ensuite, on affiche notre page via le navigateur :</p>
<p><img src="https://www.linuxtricks.fr/upload/debian-php-avec-apache.png" alt=""></p>
<p>On peut éditer les options de PHP via le fichier <strong>/etc/php/7.0/apache2/php.ini</strong> ou créer un fichier personnalisé dans <strong>/etc/php/7.0/apache2/conf.d</strong>.</p>
<h2 id="mariadb--la-base-de-données">MariaDB : La base de données</h2>
<p>Maintenant, il ne reste plus que le moteur de base de données à installer.</p>
<p>On installe donc MariaDB, le serveur :</p>
<p>Code :</p>
<pre><code>apt-get install mariadb-server
</code></pre>
Une fois fait, on démarre l’installation :</p>
<p>Code :</p>
<pre><code    mysql_secure_installation
</code></pre>
<p>
Une série de questions vont s’afficher.</p>
<p>On définit le mot de passe root :Code :</p>
<pre><code>

    Change the root password? [Y/n] Y
New password:
Re-enter new password:
Password updated successfully!
</code></pre>
<p>
On supprime les utilisateurs anonymes, les connexions distantes de root, etc…<br>...  
Code :</p>
<pre><code>

    Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] YRemove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
</code></pre>
<p>
  
  
On teste la connexion à la base de données :</p>
<p>Code :</p>
<pre><code>mysql -u root -p
</code></pre>
<p>mysql On créé ensuite un utilisateur avec tous les droits pour ne pas utiliser root :</p>
<p>  
  
Code SQL :</p>
<pre><code>CREATE USER 'hugo'@'localhost' IDENTIFIED BY 'mdp';
GRANT ALL PRIVILEGES ON *.* TO 'adrien'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
</code></pre>
<p>
Et on se déconnecte :<br>
Code SQL :</p>
<pre><code>exit
</code></pre>
<p>On peut s’assurer que le service se lance à chaque démarrage du serveur :<br>  
Code :</p>
<pre><code  systemctl enable mariadb
</code></pre>
Notre serveur LAMP est installé.</p>
<p>

Installe docker :</p>
<p><a href="https://docs.docker.com/install/linux/docker-ce/debian/">https://docs.docker.com/install/linux/docker-ce/debian/</a></p>
<h2 id="suite-à-ça-on-installe-installe-les-applications-web-que-lon-souhaite.">Suite à ça on installe installe les applications web que l’on souhaite.</h2>
<h3 id="netdata-">NetData :</h3>
<pre><code>bash &lt;(curl -Ss https://my-netdata.io/kickstart.sh)
</code></pre>
<p>L’
L'installation ce fait en une seul ligne de commande.</p>
<h3 id="cloud-torrent-">Cloud-Torrent :</h3>
<pre><code>docker run -d -p 3000:3000 -v /path/to/my/downloads:/downloads jpillora/cloud-torrent
</code></pre>
<p>Vue que c’est un docker “l’installation”" ce fait aussi en une seul ligne de commande.</p>
Dans nos DNS on ajoute des sous-domaines :<br>
<img src=" 
![enter image description here](https://i.imgur.com/3bwtS50.png" alt=")
![enter image description here"><br>
<img src="](https://i.imgur.com/JPef8bm.png" alt="enter image description here"></p>
<p>)

On modifie les virtual-host pour les faire fonctionner avec les sous-domaines :</p>
<pre><code>&lt;VirtualHost *:80ServerAdmin hugomarques.hugomarques@hugomarques.fr
  DocumentRoot /var/www/html
  &lt;Directory /var/www/html/&gt;
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Require all granted
  &lt;/Directory&gt;
  ErrorLog ${APACHE_LOG_DIR}/error.log
  # Possible values include: debug, info, notice, warn, error, crit,
  # alert, emerg.
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
&lt;/VirtualHostVirtualHost *:443&gt;
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
&lt;/VirtualHost&gt;

&lt; <VirtualHost *:80&gt;
	ServerName wp.hugomarques.fr
  ServerAlias www.wp.hugomarques.fr
	ServerAdmin contact@hugomarques.fr
    	DocumentRoot /var/www/wp/
&lt;/VirtualHost&gt;

&lt;>
    
    <VirtualHost *:80&gt;
	ServerName torrent.hugomarques.fr
    	ServerAlias www.torrent.hugomarques.fr
	ProxyPreserveHost On
	ProxyPass / http://hugomarques.fr:3000/
	ProxyPassReverse / http://hugomarques.fr:3000/
&lt;/VirtualHost&gt;

&lt;VirtualHost *:80&gt;
>
    	ServerName monitor.hugomarques.fr
  ServerAlias www.monitor.hugomarques.fr
	ProxyPreserveHost On
	ProxyPass / http://localhost:19999/
	ProxyPassReverse / http://localhost:19999/

  &lt;Location /&gt;
    AuthType Basic
    AuthName "Protected site"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
    Order deny,allow
    Allow from all
  &lt;/Location&gt;

&lt;/VirtualHost&gt;
</code></pre>

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNDczNTg3OTldfQ==
-->