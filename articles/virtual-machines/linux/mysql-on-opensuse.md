---
title: Installer MySQL sur une machine virtuelle OpenSUSE dans Azure
description: Découvrez comment installer MySQL sur une machine virtuelle Linux OpenSUSE dans Azure.
services: virtual-machines-linux
author: cynthn
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: how-to
ms.date: 07/11/2018
ms.author: cynthn
ms.openlocfilehash: 7fbdeda4cce737a6220d42642473e5f86ef8e525
ms.sourcegitcommit: 192f9233ba42e3cdda2794f4307e6620adba3ff2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96295959"
---
# <a name="install-mysql-on-a-virtual-machine-running-opensuse-linux-in-azure"></a>Installation de MySQL sur une machine virtuelle exécutant OpenSUSE Linux dans Azure

[MySQL](https://www.mysql.com) est une base de données SQL open source connue. Ce didacticiel vous montre comment créer une machine virtuelle sous OpenSUSE Linux, puis installer MySQL.


Si vous choisissez d’installer et d’utiliser l’interface CLI localement, vous devez utiliser Azure CLI 2.0 ou version ultérieure. Pour connaître la version de l’interface, exécutez `az --version`. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli).

## <a name="create-a-virtual-machine-running-opensuse-linux"></a>Création d'une machine virtuelle exécutant OpenSUSE Linux

Créez d’abord un groupe de ressources. Dans cet exemple, le groupe de ressources est nommé *mySQSUSEResourceGroup* et créé dans la région *USA Est*.

```azurecli-interactive
az group create --name mySQLSUSEResourceGroup --location eastus
```

Créez la machine virtuelle. Dans cet exemple, la machine virtuelle est nommée *myVM* et sa taille est *Standard_D2s_v3*, mais vous devez sélectionner la [taille de machine virtuelle](../sizes.md) la plus adaptée à votre charge de travail.

```azurecli-interactive
az vm create --resource-group mySQLSUSEResourceGroup \
   --name myVM \
   --image openSUSE-Leap \
   --size Standard_D2s_v3 \
   --generate-ssh-keys
```

Vous devez également ajouter une règle au groupe de sécurité réseau pour autoriser le trafic sur le port 3306 pour MySQL.

```azurecli-interactive
az vm open-port --port 3306 --resource-group mySQLSUSEResourceGroup --name myVM
```

## <a name="connect-to-the-vm"></a>Connexion à la machine virtuelle

Vous allez utiliser le protocole SSH pour vous connecter à la machine virtuelle. Dans cet exemple, l’adresse IP publique de la machine virtuelle est *10.111.112.113*. Vous pouvez vérifier l’adresse IP dans la sortie après avoir créé la machine virtuelle.

```azurecli-interactive  
ssh 10.111.112.113
```

 
## <a name="update-the-vm"></a>Mettre à jour la machine virtuelle
 
Une fois connecté à la machine virtuelle, installez les mises à jour système et les correctifs. 
   
```bash
sudo zypper update
```

Suivez les invites pour mettre à jour votre machine virtuelle.

## <a name="install-mysql"></a>Installer MySQL 


Installez MySQL sur la machine virtuelle via SSH. Répondez aux invites comme il convient.

```bash
sudo zypper install mysql
```
 
Définissez MySQL pour qu’il démarre en même temps que le système. 

```bash
sudo systemctl enable mysql
```
Vérifiez que MySQL est activé.

```bash
systemctl is-enabled mysql
```

Le résultat retourné devrait être : enabled.

Redémarrez le serveur.

```bash
sudo reboot
```


## <a name="mysql-password"></a>Mot de passe MySQL

Après l’installation, le mot de passe racine MySQL est vide par défaut. Exécutez le script **mysql\_secure\_installation** pour sécuriser MySQL. Le script vous invite à modifier le mot de passe racine MySQL, à supprimer les comptes d’utilisateurs anonymes, à désactiver les connexions racine à distance, à supprimer les bases de données de test et à recharger la table des privilèges. 

Lorsque le serveur redémarre, utilisez à nouveau une clé SSH sur la machine virtuelle.

```azurecli-interactive  
ssh 10.111.112.113
```



```bash
mysql_secure_installation
```

## <a name="sign-in-to-mysql"></a>Se connecter à MySQL

Vous pouvez maintenant vous connecter et entrer l’invite MySQL.

```bash  
sudo mysql -u root -p
```
Cette action vous renvoie à l’invite MySQL où vous pouvez émettre des instructions SQL pour interagir avec la base de données.

Créez à présent un utilisateur MySQL.

```sql
CREATE USER 'mysqluser'@'localhost' IDENTIFIED BY 'password';
```
   
Le point-virgule (;) à la fin de la ligne est indispensable pour terminer la commande.


## <a name="create-a-database"></a>Création d'une base de données


Créez une base de données et accordez les autorisations d’utilisateur `mysqluser`.

```sql
CREATE DATABASE testdatabase;
GRANT ALL ON testdatabase.* TO 'mysqluser'@'localhost' IDENTIFIED BY 'password';
```
   
Les noms et les mots de passe des utilisateurs de base de données sont uniquement utilisés par les scripts pour se connecter à la base de données.  Les noms des comptes d’utilisateurs de base de données ne représentent pas nécessairement les comptes d’utilisateurs du système.

Autorisez la connexion à partir d’un autre ordinateur. Dans cet exemple, l’adresse IP de l’ordinateur à partir duquel autoriser la connexion est *10.112.113.114*.

```sql
GRANT ALL ON testdatabase.* TO 'mysqluser'@'10.112.113.114' IDENTIFIED BY 'password';
```
   
Pour quitter l'utilitaire d’administration de base de données MySQL, entrez :

```    
quit
```


## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur MySQL, consultez la [Documentation MySQL](https://dev.mysql.com/doc).
