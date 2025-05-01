**Modules Réseaux : SAÉ 2.03**

# Compte-rendu d’installation et de configuration d'un serveur

Ce fichier README **décrit les configurations** réalisées et **les éventuels problèmes** rencontrés.

Il comprend les **extraits de fichiers de configuration** les plus pertinents ainsi que les **captures d'écran** réalisées, ainsi que les sites web utilisés.

**Les fichiers de configuration complets sont à joindre au dépôt !**

## Auteurs :

- Nom-prénom binôme 1 : Laouar-Otman
- Nom-prénom binôme 2 : Djo-Djolo Ayessa Yannyck

## Adresse IP de la Machine virtuelle

- 10.31.33.241

---

## I - Installation de services

### 1. Gestion des services : systemd

**Commandes utilisées :**
- `man systemctl`
- `systemctl`
- `sudo apt-get install openssh-server`
- `systemctl status sshd`
- `ssh 10.31.33.241`
- `exit` ou `CTRL+D`
- `sudo systemctl stop sshd`
- `ssh 10.31.33.241`
- `sudo systemctl start sshd`
- `ssh 10.31.33.241`

**Problèmes rencontrés :** Aucun problème notable.

**Captures d’écran :** 

---

### 2. Serveur Web apache2

#### 2.1 Installation de base du serveur

**Commandes utilisées :**
- `sudo apt update`
- `sudo apt install apache2`
- `sudo systemctl start apache2`
- Accès à http://10.31.33.241 pour vérification

**Exploration :**
- Consultation du fichier `/etc/apache2/apache2.conf`
- Compréhension des liens symboliques `sites-enabled`/

**Activation des pages utilisateurs :**
- `sudo a2enmod userdir`
- `sudo systemctl restart apache2`

**Création du répertoire public_html :**
- `mkdir /home/iut/public_html`
- Attribution des droits à `www-data`
- Protection contre l'indexation en modifiant `userdir.conf`

**Création du fichier de test :**
- `nano /home/iut/public_html/bienvenue.html` ("Bienvenue sur votre site perso")

**Problèmes rencontrés :**
- Problème d’accès lié aux permissions corrigé en ajustant les droits sur les répertoires.
- Difficulté à désactiver l'indexation corrigée en éditant correctement `userdir.conf`.

**Captures d’écran :** 

#### 2.2 Installation du serveur Web virtuel

**Commandes utilisées :**
- `nslookup 10.31.33.241` (pour obtenir le DNS)
- Création du répertoire mon-serveur (avec faute "mon-server") :
  ```bash
  mkdir /home/iut/mon-server
  ```
- Attribution des droits d'accès pour www-data avec `chmod`
- Création du lien symbolique :
  ```bash
  sudo ln -s /home/iut/mon-server /var/www
  ```
- Création d'un fichier `index.html` de test dans `mon-server`
- Copie de la configuration Apache :
  ```bash
  sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/2a4v3-31uvm0497.ad-urca.univ-reims.fr.conf
  ```
- Modification du fichier pour changer le `ServerName` et `DocumentRoot`
- Activation du site :
  ```bash
  sudo a2ensite 2a4v3-31uvm0497.ad-urca.univ-reims.fr.conf
  sudo systemctl reload apache2
  ```

**Problèmes rencontrés :**
- Faute d'orthographe : répertoire "mon-server" au lieu de "mon-serveur" (faute conservée volontairement pour tout garder cohérent).
- Problèmes de DNS corrigés après modification du `ServerName`.
- Obligation de modifier aussi `000-default.conf` pour que HTTP redirige vers `/var/www/mon-server`.

**Captures d’écran : **

---

### 3. Serveur Web sécurisé https

**Commandes utilisées :**
- `dpkg -l | grep ssl`
- Installation des paquets nécessaires (si besoin) : `sudo apt install openssl ssl-cert`
- Création du dossier SSL :
  ```bash
  sudo mkdir /etc/apache2/ssl
  ```
- Création du certificat auto-signé :
  ```bash
  sudo /usr/sbin/make-ssl-cert /usr/share/ssl-cert/ssleay.cnf /etc/apache2/ssl/apache.pem
  ```
  avec comme réponses : `10.31.33.241, DNS:2a4v3-31uvm0497.ad-urca.univ-reims.fr`

- Copie du fichier SSL par défaut :
  ```bash
  sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/mon-serveur-ssl.conf
  ```
- Modification du fichier pour correspondre au `DocumentRoot /var/www/mon-server` et au `ServerName`.
- Activation du module SSL :
  ```bash
  sudo a2enmod ssl
  ```
- Activation du site SSL :
  ```bash
  sudo a2ensite mon-serveur-ssl.conf
  ```
- Redémarrage du service Apache :
  ```bash
  sudo systemctl restart apache2
  ```

**Problèmes rencontrés :**
- Erreur de syntaxe initiale dans `mon-serveur-ssl.conf` corrigée (mauvais ServerName, mauvais chemin d'accès).
- Nécessité de corriger le nom du serveur car une mauvaise casse du DNS empêchait l’accès.
- Le HTTP pointait initialement sur `/var/www/html` et non `/var/www/mon-server`, corrigé en ajustant également `000-default.conf`.

**Captures d’écran :** 

---

### 4. Langage de programmation PHP

**Commandes utilisées :**
- Installation de PHP :
  ```bash
  sudo apt install php libapache2-mod-php
  ```
- Édition du fichier de configuration pour autoriser PHP dans les répertoires utilisateurs :
  ```bash
  sudo nano /etc/apache2/mods-enabled/php7.4.conf
  ```
- Commenter les 5 dernières lignes du fichier (`#` devant chaque ligne).
- Redémarrage du service Apache :
  ```bash
  sudo systemctl restart apache2
  ```

**Création et test :**
- Création d'un fichier PHP de test :
  ```bash
  nano /home/iut/public_html/index.php
  ```
  Contenu du fichier :
  ```php
  <?php phpinfo();
  ```
- Accès dans le navigateur à :
  ```
  http://2a4v3-31uvm0497.ad-urca.univ-reims.fr/~iut/index.php
  ```

**Problèmes rencontrés :**
- Difficulté initiale à commenter les lignes dans nano (problème de saisie du `#` corrigé en copiant depuis la machine locale).
- Validation correcte du fonctionnement de PHP via phpinfo().

**Captures d’écran :**

---

## II - Webographie et recours à l'IA

- **Sites consultés :**
  - StackOverflow
  - Forums Ubuntu-fr.org

- **IA utilisée :** ChatGPT pour:
  - Comprendre les erreurs de VirtualHost
  - Corriger les erreurs SSL
  - Aider à la configuration Apache et PHP

**Prompts donnés :**
- "Comment corriger l'erreur SSL certificate file not found sur Apache2"
- "Comment configurer un VirtualHost Apache pour Ubuntu"
- "Comment installer et configurer PHP sur Apache2"

**Validité et vérifications :**
- Tests systématiques après chaque modification.
- Redémarrage de Apache avec `sudo systemctl reload apache2`.

---

# PENSEZ A LAISSER ALLUMÉE VOTRE MACHINE VIRTUELLE
afin qu'on puisse y accéder ...



afin qu'on puisse y accéder ...

### 5.Installation du service de base de données MySql

#### 5.1 Installation et lancement de MySql :
  **commandes utilisées:**
  ````
  - sudo apt install mysql-server
  ````
  ````
  - sudo systemctl start mysql
  ````
  ````
  - sudo mysql_secure_installation
  ````

#### 5.2 Connection en tant que __root__ et création d'un utilisateur:
  **commandes utilisées**
   ````sudo mysql
   CREATE USER 'admin'@'localhost' IDENTIFIED BY 'infoiut2';
   GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;
   ````

#### 5.3 Test du serveur:
  **commandes utilisées**
  ````
  - sudo systemctl restart mysql (redémarrage du service)
  - mysqlshow -u admin -p
  ````
  **Problèmes rencontrés**
  - Inactivité du service mysql après installation, problème réglé avec la commande: 
  ````
  sudo systemctl start mysql
  ````
  - Accès refusé à l'utilisateur admin lors du test du serveur dû à une mauvaise syntaxe lors de la création de celui-ci.__Commande érroné__:
  ````
  CREATE USER admin@localhost IDENTIFIED BY infoiut;
  ````
  Résolution du problème grâce à __l'ajout des ''__ dans la syntaxe de la commande:
  ````
  CREATE USER 'admin'@'localhost' IDENTIFIED BY 'infoiut2';
  ````
  - Impossibilité d'écrire @ résolu en utilisant les touches __Alt 64__

#### 6. Installation du service phpmyadmin

#### 6.1 Installation de phpmyadmin:
  **commande utilisée**
  ````bash
  sudo apt install phpmyadmin
  ````
  Verification de la précense du fichier phphmyadmin.conf:
  ````bash
  - cd /etc/apache2/conf-enabled
  - ls
  ````

#### Activation du module de gestion des chaînes de caractères multi-octets:
  **commandes utilisées**
  ````bash
  sudo phpenmod mbstring
  ````
  Redémarrage de apache2:
  ````bash
  sudo systemctl restart apache2
  ````

#### Test de la connexion et création d'utilisateur:
  **capture d'écran de la connexion à phpmyadmin avec l'utilisateur admin**
  - Obtention du nom DNS de la machine avec la commande:
  ````bash
  nslookup 10.31.33.241
  ````
  - URL tapé:
  ````
  http://2a4v3-31uvm0497/phpmyadmin/index.php?route=/
  ````
  - Utilisateur: __admin__
  - Mot de passe: __infoiut2__

  **capture d'écran de la connexion à phpmyadmin avec l'utilisateur mysqltest**

  - Utlisateur: __mysqltest__
  - Mot de passe: __1234__

**Problème rencontré:**
  - Mauvaise compréhension du procéssus de connexion à phpmyadmin,     tentative de connexion via la machine virtuelle avec la commande:
  ````bash
  http://10.31.33.241/phpmyadmin
  ````
  Problème résolu après une discussion avec mon binôme.
  
#### Partie 7: WordPress

#### 7.1: Installation du dossier .zip:
  **commande utilisé**
  - Installation du dossier .zip:
  ````bash
  wget https://fr.wordpress.org/latest-fr_FR.zip
  ````
  - Installation de unzip pour dézipper le dossier:
  ````bash
  sudo apt install unzip
  ````
  - Dézippage du dossier:
  ````
  unzip latest-fr_FR.zip
  ````

#### 7.2 Creation de la base de donnée wordpress avec utilisateur du même nom sur phpmyadmin:
  **Accords des privilèges de l'utlisateur sur la base de donnée avec la commande:**
  - ````GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost';````

#### 7.3 Déplacement des fichiers wp dans le répertoire mon-server
  **commande utilisé**
  - Utilisation de la commande linux __mv__:
  ````bash
  mv wp* /home/iut/mon-server
  ````

#### 7.3 Modification du fichier wp-config.php:
  **Capture d'écran des informations ajoutées au fichier wp-config.php**

#### 7.5 Redémarrage de apache2 et mySQL avec les commandes:
  -``sudo systemctl restart apache2``
  -``sudo systemctl restart mysql``

#### 7.5 Accès à l'URL et tentative de finalisation de l'installation:
  **problèmes rencontrés**
  - Le serveur web __n'affiche pas__ la page wordpress permettant la connexion de l'utilisateur et la finalisation de l'installation, seule la "page d'accueil" du serveur s'affiche avec le texte "voici mon serveur virtuel". L'url renseigné est `https://2a4v3-31uvm0497.ad-urca.univ-reims.fr/`
  - Une seconde tentative d'accès avec l'url suivant : `https://2a4v3-31uvm0497.ad-urca.univ-reims.fr/mon-server` affiche une page disant que nous n'avons pas la permission pour accéder à la ressource.

