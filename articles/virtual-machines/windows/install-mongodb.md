---
title: Installation de MongoDB sur une machine virtuelle Windows dans Azure
description: Apprenez à installer MongoDB sur une machine virtuelle Azure fonctionnant sous Windows Server 2012 R2 créée avec le modèle de déploiement Resource Manager.
documentationcenter: ''
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: how-to
ms.date: 12/15/2017
ms.author: cynthn
ms.openlocfilehash: d02fa8fa23b587db06f3d2d1e08f0a8565471123
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88510417"
---
# <a name="install-and-configure-mongodb-on-a-windows-vm-in-azure"></a>Installation et configuration de MongoDB sur une machine virtuelle Windows dans Azure
[MongoDB](https://www.mongodb.org) est une base de données NoSQL open-source qui offre des performances élevées. Cet article vous guide lors de l’installation et de la configuration de MongoDB sur une machine virtuelle Windows Server 2016 (VM) dans Azure. Vous pouvez également [installer MongoDB sur une machine virtuelle Linux dans Azure](../linux/install-mongodb.md).

## <a name="prerequisites"></a>Prérequis
Avant d’installer et configurer MongoDB, vous devez créer une machine virtuelle et, dans l’idéal, ajouter un disque de données à celle-ci. Consultez les articles suivants pour créer une machine Virtuelle et ajouter un disque de données :

* Création d’une machine virtuelle Windows Server à l’aide du [portail Azure](quick-create-portal.md) ou [d’Azure PowerShell](quick-create-powershell.md).
* Association d’un disque de données à une machine virtuelle Windows Server à l’aide du [portail Azure](attach-managed-disk-portal.md) ou [d’Azure PowerShell](attach-disk-ps.md).

Pour commencer à installer et à configurer MongoDB, [connectez-vous à votre machine virtuelle Windows Server](connect-logon.md) à l’aide du Bureau à distance.

## <a name="install-mongodb"></a>Installation de MongoDB
> [!IMPORTANT]
> Les fonctionnalités de sécurité MongoDB, comme l’authentification et la liaison d’adresse IP, ne sont pas activées par défaut. Elles doivent être activées avant le déploiement de MongoDB dans un environnement de production. Pour en savoir plus, consultez la page [Sécurité et authentification pour MongoDB](https://www.mongodb.org/display/DOCS/Security+and+Authentication).


1. Une fois que vous êtes connecté à la machine virtuelle à l’aide du Bureau à distance, ouvrez Internet Explorer à partir de la barre des tâches.
2. Sélectionnez **Utiliser les paramètres de sécurité, de confidentialité et de compatibilité recommandés** lorsque s’ouvre pour la première fois, puis cliquez sur **OK**.
3. La configuration de la sécurité renforcée d’Internet Explorer est activée par défaut. Ajoutez le site Web MongoDB à la liste des sites autorisés :

   * Sélectionnez l’icône **Outils** dans le coin supérieur droit.
   * Dans **Options Internet**, sélectionnez l’onglet **Sécurité**, puis l’icône **Sites de confiance**.
   * Cliquez sur le bouton **Sites**. Ajoutez le site *https://\*.mongodb.com* à la liste des sites de confiance et fermez la boîte de dialogue.

     ![Configuration des paramètres de sécurité Internet Explorer](./media/install-mongodb/configure-internet-explorer-security.png)
4. Accédez à la page [MongoDB - Downloads](https://www.mongodb.com/downloads) (MongoDB - Téléchargements) (https://www.mongodb.com/downloads).
5. Si nécessaire, sélectionnez l’édition **Community Server** et la dernière version stable mise à jour pour *Windows Server 2008 R2 64 bits et versions ultérieures*. Pour télécharger le programme d’installation, cliquez sur **TÉLÉCHARGER (msi)** .

    ![Téléchargez le programme d’installation de MongoDB](./media/install-mongodb/download-mongodb.png)

    Exécutez le programme d’installation une fois le téléchargement terminé.
6. Lisez et acceptez le contrat de licence. Lorsqu’une invite s’affiche, sélectionnez **Terminer** le programme d’installation.
7. Si vous le souhaitez, vous pouvez choisir d’installer également Compass, une interface graphique dédiée à MongoDB.
8. Sur le dernier écran, cliquez sur **Installer**.

## <a name="configure-the-vm-and-mongodb"></a>Configuration de la machine virtuelle et de MongoDB
1. Les variables de chemin d’accès ne sont pas mises à jour par le programme d’installation de MongoDB. Sans l’emplacement `bin` de MongoDB dans votre variable PATH, vous devez spécifier le chemin d’accès complet à chaque fois que vous utilisez un fichier exécutable MongoDB. Pour ajouter l’emplacement à votre variable PATH :

   * Avec le bouton droit, cliquez sur le menu **Démarrer** et sélectionnez **Système**.
   * Cliquez sur **Paramètres système avancés**, puis cliquez sur **Variables d’environnement**.
   * Sous **Variables système**, sélectionnez **Chemin d’accès**, puis cliquez sur **Modifier**.

     ![Configurez les variables PATH](./media/install-mongodb/configure-path-variables.png)

     Ajoutez le chemin d’accès vers le dossier `bin` de MongoDB. MongoDB est généralement installé sur *C:\Program Files\MongoDB*. Vérifiez le chemin d’installation sur votre machine virtuelle. L’exemple suivant ajoute la valeur par défaut d’emplacement d’installation de MongoDB la variable `PATH` :

     ```
     ;C:\Program Files\MongoDB\Server\3.6\bin
     ```

     > [!NOTE]
     > Veillez à ajouter le point-virgule en premier (`;`) pour indiquer que vous ajoutez un emplacement pour votre variable `PATH`.

2. Créez les répertoires de journaux et de données MongoDB sur votre disque dur. Dans le menu **Démarrer**, sélectionnez **Invite de commande**. L’exemple suivant crée les répertoires sur le lecteur F:

    ```
    mkdir F:\MongoData
    mkdir F:\MongoLogs
    ```
3. Démarrez une instance MongoDB avec la commande suivante, en adaptant le chemin d’accès à vos données et journaux :

    ```
    mongod --dbpath F:\MongoData\ --logpath F:\MongoLogs\mongolog.log
    ```

    Il se peut que plusieurs minutes soient nécessaires pour que MongaDB alloue les fichiers journaux et commence à écouter les connexions. Tous les messages du journal sont dirigés vers le fichier *F:\MongoLogs\mongolog.log* lorsque le serveur `mongod.exe` démarre et alloue les fichiers journaux.

   > [!NOTE]
   > L’invite de commandes reste axée sur cette tâche pendant que votre instance MongoDB est en cours d’exécution. Laissez la fenêtre d’invite de commandes ouverte pour poursuivre l’exécution de MongoDB. Vous pouvez aussi installer MongoDB en tant que service, comme indiqué dans l’étape suivante.

4. Pour une expérience plus robuste avec MongoDB, installez `mongod.exe` en tant que service. La création d’un service signifie que vous n’avez pas besoin de laisser une invite de commande s’exécuter chaque fois que vous souhaitez utiliser MongoDB. Créez le service en procédant comme suit, en adaptant le chemin d’accès à vos répertoires de données et de journaux :

    ```
    mongod --dbpath F:\MongoData\ --logpath F:\MongoLogs\mongolog.log --logappend  --install
    ```

    La commande précédente crée un service nommé « MongoDB » avec « Mongo DB » comme description. Les paramètres suivants sont également spécifiés :

   * L’option `--dbpath` spécifie l’emplacement du répertoire de données.
   * L’option `--logpath` permet de spécifier un fichier journal puisque le service en cours d’exécution n’a pas de fenêtre de commande pour afficher la sortie.
   * L’option `--logappend` spécifie qu’au redémarrage du service, la sortie est ajoutée au fichier journal existant.

   Pour démarrer le service MongoDB, exécutez la commande suivante :

    ```
    net start MongoDB
    ```

    Pour plus d’informations sur la création du service MongoDB, consultez [Configuration d’un service Windows pour MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/#mongodb-as-a-windows-service).

## <a name="test-the-mongodb-instance"></a>Testez l’instance de MongoDB
Avec MongoDB lancé en tant qu’instance unique ou installé en tant que service, vous pouvez maintenant commencer la création et l’utilisation de vos bases de données. Pour lancer l’interpréteur de commandes d’administration de MongoDB, ouvrez une autre fenêtre d’invite de commande à partir du menu **Démarrer** et entrez la commande suivante :

```
mongo
```

Vous pouvez répertorier les bases de données avec la commande `db`. Insérez des données comme suit :

```
db.foo.insert( { a : 1 } )
```

Recherchez des données comme suit :

```
db.foo.find()
```

Le résultat ressemble à l’exemple suivant :

```
{ "_id" : "ObjectId("57f6a86cee873a6232d74842"), "a" : 1 }
```

Quittez la console `mongo` comme suit :

```
exit
```

## <a name="configure-firewall-and-network-security-group-rules"></a>Configuration des règles de pare-feu et de groupe de sécurité réseau
Maintenant que MongoDB est installé et en cours d’exécution, ouvrez un port dans le Pare-feu Windows pour vous connecter à distance à MongoDB. Pour créer une nouvelle règle de trafic entrant pour autoriser le port TCP 27017, ouvrez une invite PowerShell administrative et saisissez la commande suivante :

```powershell
New-NetFirewallRule `
    -DisplayName "Allow MongoDB" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 27017 `
    -Action Allow
```

Vous pouvez également créer la règle à l’aide de l’outil de gestion graphique **Pare-feu Windows avec fonctions de sécurité avancées**. Créez une nouvelle règle de trafic entrant pour autoriser le port TCP 27017.

Si nécessaire, créez une règle de groupe de sécurité réseau pour autoriser l’accès à MongoDB en dehors du sous-réseau du réseau virtuel Azure existant. Vous pouvez créer les règles du groupe de sécurité réseau à l’aide du [portail Azure](nsg-quickstart-portal.md) ou [d’Azure PowerShell](nsg-quickstart-powershell.md). Comme avec les règles de pare-feu Windows, autorisez le port TCP 27017 pour l’interface réseau virtuelle de votre machine virtuelle MongoDB.

> [!NOTE]
> Le port TCP 27017 est le port par défaut utilisé par MongoDB. Vous pouvez modifier ce port à l’aide du paramètre `--port` lors du démarrage de `mongod.exe`, manuellement ou à partir d’un service. Si vous modifiez le port, veillez à mettre à jour les règles de pare-feu Windows et le groupe de sécurité réseau dans les étapes précédentes.


## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à installer et à configurer MongoDB sur votre machine virtuelle Windows. Vous pouvez maintenant accéder à MongoDB sur votre machine virtuelle Windows en suivant les rubriques avancées dans la [Documentation MongoDB](https://docs.mongodb.com/manual/).

