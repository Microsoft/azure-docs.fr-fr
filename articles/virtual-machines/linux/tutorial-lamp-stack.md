---
title: 'Tutoriel : Déployer LAMP sur une machine virtuelle Linux dans Azure'
description: Avec ce didacticiel, vous allez apprendre à installer la pile LAMP sur une machine virtuelle Linux dans Azure
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: 6c12603a-e391-4d3e-acce-442dd7ebb2fe
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 01/30/2019
ms.author: cynthn
ms.openlocfilehash: d0d86e1a9c40eb6860508cf136ab9d466cc28ecd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88225901"
---
# <a name="tutorial-install-a-lamp-web-server-on-a-linux-virtual-machine-in-azure"></a>Tutoriel : Installer un serveur web LAMP sur une machine virtuelle Linux dans Azure

Cet article vous guide à travers le déploiement d’un serveur web Apache, de celui de MySQL et de PHP (la pile LAMP) sur une machine virtuelle Ubuntu dans Azure. Pour voir le serveur LAMP fonctionner, vous pouvez éventuellement installer et configurer un site WordPress. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une machine virtuelle Ubuntu (la lettre « L » dans la pile LAMP)
> * Ouvrez le port 80 pour le trafic web
> * Installer Apache, MySQL et PHP
> * Vérifier l’installation et la configuration
> * Installer WordPress sur le serveur LAMP

Ce programme d’installation est destiné aux tests rapides ou à la preuve de concept. Pour plus d’informations sur la pile LAMP, notamment des recommandations relatives à un environnement de production, consultez la [Documentation Ubuntu](https://help.ubuntu.com/community/ApacheMySQLPHP).

Ce tutoriel utilise l’interface CLI disponible dans [Azure Cloud Shell](../../cloud-shell/overview.md), qui est constamment mise à jour vers la dernière version. Pour ouvrir Cloud Shell, sélectionnez **Essayer** en haut d’un bloc de code.

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, ce didacticiel nécessite que vous exécutiez Azure CLI version 2.0.30 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli).

[!INCLUDE [virtual-machines-linux-tutorial-stack-intro.md](../../../includes/virtual-machines-linux-tutorial-stack-intro.md)]

## <a name="install-apache-mysql-and-php"></a>Installer Apache, MySQL et PHP

Exécutez la commande suivante pour mettre à jour les sources de package Ubuntu et installer Apache, MySQL et PHP. Notez le signe insertion (^) à la fin de la commande, qui fait partie du nom du package `lamp-server^`. 


```bash
sudo apt update && sudo apt install lamp-server^
```

Vous êtes invité à installer les packages et les autres dépendances. Ce processus installe les extensions PHP minimales requises pour utiliser PHP avec MySQL.  

## <a name="verify-installation-and-configuration"></a>Vérifier l’installation et la configuration


### <a name="verify-apache"></a>Vérifier Apache

Vérifiez la version d’Apache à l’aide de la commande suivante :
```bash
apache2 -v
```

Avec Apache installé et le port 80 ouvert sur votre machine virtuelle, le serveur web est maintenant accessible à partir d’Internet. Pour afficher la page par défaut d’Apache2 Ubuntu, ouvrez un navigateur web et entrez l’adresse IP publique de la machine virtuelle. Utilisez l’adresse IP publique dont vous vous êtes servi pour vous connecter par SSH à la machine virtuelle :

![Page par défaut d’Apache][3]


### <a name="verify-and-secure-mysql"></a>Vérifier et sécuriser MySQL

Vérifiez la version de MySQL avec la commande suivante (remarquez le paramètre `V` avec une majuscule) :

```bash
mysql -V
```

Pour mieux sécuriser l’installation de MySQL, notamment en définissant un mot de passe racine, exécutez le script `mysql_secure_installation`. 

```bash
sudo mysql_secure_installation
```

Vous pouvez éventuellement configurer le plug-in de validation de mot de passe (recommandé). Ensuite, définissez un mot de passe pour l’utilisateur racine MySQL et configurez les autres paramètres de sécurité pour votre environnement. Nous vous recommandons de répondre par l’affirmative à toutes les questions.

Si vous souhaitez essayer des fonctionnalités de MySQL (créer une base de données MySQL, ajouter des utilisateurs ou changer des paramètres de configuration), connectez-vous à MySQL. Cette étape n’est pas nécessaire pour suivre ce didacticiel.

```bash
sudo mysql -u root -p
```

Lorsque vous avez terminé, quittez l’invite de mysql en tapant `\q`.

### <a name="verify-php"></a>Vérifier PHP

Vérifiez la version de PHP avec la commande suivante :

```bash
php -v
```

Si vous voulez poursuivre le test, créez une page d’informations PHP rapide pour l’afficher dans un navigateur. La commande suivante crée la page d’informations PHP :

```bash
sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
```

À présent, vous pouvez vérifier la page d’informations PHP que vous avez créée. Ouvrez un navigateur et accédez à `http://yourPublicIPAddress/info.php`. Remplacez l’adresse IP publique de votre machine virtuelle. Elle doit ressembler à cette image.

![Page d’informations PHP][2]

[!INCLUDE [virtual-machines-linux-tutorial-wordpress.md](../../../includes/virtual-machines-linux-tutorial-wordpress.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez déployé un serveur LAMP dans Azure. Vous avez appris à :

> [!div class="checklist"]
> * Créer une machine virtuelle Ubuntu
> * Ouvrez le port 80 pour le trafic web
> * Installer Apache, MySQL et PHP
> * Vérifier l’installation et la configuration
> * Installer WordPress sur le serveur LAMP

Passez au tutoriel suivant pour savoir comment mieux protéger les serveurs web à l’aide des certificats TLS/SSL.

> [!div class="nextstepaction"]
> [Sécuriser un serveur web avec TLS](tutorial-secure-web-server.md)

[2]: ./media/tutorial-lamp-stack/phpsuccesspage.png
[3]: ./media/tutorial-lamp-stack/apachesuccesspage.png
