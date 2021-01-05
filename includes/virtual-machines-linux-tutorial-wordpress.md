---
author: cynthn
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: 76a080d40721fa78ad703f77f6dbe7a3363ab77e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68857373"
---
## <a name="install-wordpress"></a>Installer WordPress

Si vous souhaitez essayer votre pile, installez un exemple d’application. Ainsi, les étapes suivantes installent la plateforme open source [WordPress](https://wordpress.org/) pour créer des sites web et des blogs. Les autres charges de travail à essayer incluent [Drupal](http://www.drupal.org) et [Moodle](https://moodle.org/). 

Ce programme d’installation de WordPress est destiné uniquement à la preuve de concept. Pour installer la dernière version de WordPress en production avec les paramètres de sécurité recommandés, consultez la [documentation de WordPress](https://codex.wordpress.org/Main_Page). 



### <a name="install-the-wordpress-package"></a>Installez le package WordPress

Exécutez la commande suivante :

```bash
sudo apt install wordpress
```

### <a name="configure-wordpress"></a>Configurer WordPress

Configurer WordPress pour utiliser PHP et MySQL.

Dans un répertoire de travail, créez un fichier texte `wordpress.sql` pour configurer la base de données MySQL pour WordPress : 

```bash
sudo sensible-editor wordpress.sql
```

Ajoutez les commandes suivantes, en remplaçant *yourPassword* par le mot de passe de base de données de votre choix (laissez les autres valeurs inchangées). Si vous aviez configuré une stratégie de sécurité MySQL pour valider la force du mot de passe, vérifiez que le mot de passe répond à ces exigences. Enregistrez le fichier .

```sql
CREATE DATABASE wordpress;
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER
ON wordpress.*
TO wordpress@localhost
IDENTIFIED BY 'yourPassword';
```

Exécutez la commande suivante pour créer la base de données :

```bash
cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
```

Étant donné que le fichier `wordpress.sql` contient des informations d’identification de base de données, supprimez-le après utilisation :

```bash
sudo rm wordpress.sql
```

Pour configurer PHP, exécutez la commande suivante pour ouvrir un éditeur de texte de votre choix et créer le fichier `/etc/wordpress/config-localhost.php` :

```bash
sudo sensible-editor /etc/wordpress/config-localhost.php
```
Copiez les lignes suivantes dans le fichier, en remplaçant *yourPassword* par votre mot de passe de base de données (laissez les autres valeurs inchangées). Puis enregistrez le fichier.

```php
<?php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpress');
define('DB_PASSWORD', 'yourPassword');
define('DB_HOST', 'localhost');
define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
?>
```


Déplacez l’installation de WordPress à la racine du document du serveur web :

```bash
sudo ln -s /usr/share/wordpress /var/www/html/wordpress

sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
```

Vous pouvez désormais terminer l’installation de WordPress et publier sur la plateforme. Ouvrez un navigateur web et accédez à `http://yourPublicIPAddress/wordpress`. Remplacez l’adresse IP publique de votre machine virtuelle. Elle doit ressembler à cette image.

![Page d’installation de WordPress](./media/virtual-machines-linux-tutorial-wordpress/wordpressstartpage.png)
