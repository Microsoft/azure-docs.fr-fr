---
title: Déployer Azure SQL Edge avec Docker - Azure SQL Edge
description: En savoir plus sur le déploiement d’Azure SQL Edge avec Docker
keywords: SQL Edge, conteneur, docker
services: sql-edge
ms.service: sql-edge
ms.topic: quickstart
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: fce098767fffd36376399bbd9396699e3d9fbfd3
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93392076"
---
# <a name="deploy-azure-sql-edge-with-docker"></a>Déployer Azure SQL Edge avec Docker

Dans ce guide de démarrage rapide, vous utilisez Docker pour tirer (pull) et exécuter l’image conteneur Azure SQL Edge. Ensuite, vous vous connectez avec **sqlcmd** pour créer votre première base de données et exécuter des requêtes.

Cette image se compose d’Azure SQL Edge basé sur Ubuntu 18.04. Elle peut être utilisée avec Docker Engine 1.8+ sur Linux ou sur Docker pour Mac/Windows.

## <a name="prerequisites"></a>Conditions préalables requises

- Docker Engine 1.8+ sur n’importe quelle distribution Linux prise en charge ou Docker pour Mac/Windows. Pour plus d’informations, consultez [Installer Docker](https://docs.docker.com/engine/installation/). Les images Azure SQL Edge étant basées sur Ubuntu 18.04, il est recommandé d’utiliser un hôte Docker Ubuntu 18.04.
- Pilote de stockage **overlay2** de Docker. Il s’agit de la valeur par défaut pour la plupart des utilisateurs. Si vous n’utilisez pas ce fournisseur de stockage et que vous devez le modifier, consultez les instructions et les avertissements dans la [documentation de Docker pour la configuration d’overlay2](https://docs.docker.com/storage/storagedriver/overlayfs-driver/#configure-docker-with-the-overlay-or-overlay2-storage-driver).
- Au moins 10 Go d’espace disque.
- Au moins 1 Go de RAM.
- [Configuration matérielle requise pour Azure SQL Edge](./features.md#hardware-support).


## <a name="pull-and-run-the-container-image"></a>Extraire et exécuter l’image conteneur

Avant de commencer les étapes suivantes, assurez-vous que vous avez sélectionné votre interpréteur de commandes préféré (bash, PowerShell ou CMD) en haut de cet article.

1. Extrayez l’image conteneurAzure SQL Edge à partir de Microsoft Azure Container Registry.

    - Tirer (pull) l’image conteneur Azure SQL Edge
        ```bash
        sudo docker pull mcr.microsoft.com/azure-sql-edge:latest 
        ```

> [!NOTE]
> Pour les commandes bash de cet article, `sudo` est utilisé. Sur macOS et Windows, sudo n’est peut-être pas nécessaire. Sur Linux, si vous ne souhaitez pas utiliser sudo pour exécuter Docker, vous pouvez configurer un groupe Docker et ajouter des utilisateurs à ce groupe. Pour plus d’informations, consultez [Étapes consécutives à l’installation pour Linux](https://docs.docker.com/engine/install/linux-postinstall/).

La commande précédente tire (pull) les images conteneur Azure SQL Edge les plus récentes. Pour voir toutes les images disponibles, consultez [la page du hub Docker azure-sql-egde](https://hub.docker.com/_/microsoft-azure-sql-edge).

2. Pour exécuter l’image conteneur avec Docker, vous pouvez utiliser la commande suivante à partir d’un interpréteur de commandes bash (Linux/macOS) ou d’une invite de commandes PowerShell avec élévation de privilèges.
    
    - Démarrer une instance Azure SQL Edge s’exécutant en tant qu’édition Developer
        ```bash
        sudo docker run --cap-add SYS_PTRACE -e 'ACCEPT_EULA=1' -e 'MSSQL_SA_PASSWORD=yourStrong(!)Password' -p 1433:1433 --name azuresqledge -d mcr.microsoft.com/azure-sql-edge
        ```

    - Démarrer une instance Azure SQL Edge s’exécutant en tant qu’édition Premium
        ```bash
        sudo docker run --cap-add SYS_PTRACE -e 'ACCEPT_EULA=1' -e 'MSSQL_SA_PASSWORD=yourStrong(!)Password' -e 'MSSQL_PID=Premium' -p 1433:1433 --name azuresqledge -d mcr.microsoft.com/azure-sql-edge
        ```
    > [!NOTE]
    > Si vous utilisez PowerShell sur Windows pour exécuter ces commandes, utilisez des guillemets doubles plutôt que simples.


    > [!NOTE]
    > Le mot de passe doit suivre la stratégie de mot de passe du moteur de base de données Microsoft SQL par défaut, sinon le conteneur ne pourra pas configurer le moteur SQL et cessera de fonctionner. Par défaut, le mot de passe doit comporter au moins 8 caractères et doit contenir des caractères appartenant à trois des quatre catégories suivantes : Lettres majuscules, lettres minuscules, chiffres en base 10 et symboles. Vous pouvez examiner le journal des erreurs en exécutant la commande [docker logs](https://docs.docker.com/engine/reference/commandline/logs/).
    
    Le tableau suivant décrit les paramètres de l’exemple `docker run` précédent :

    | Paramètre | Description |
    |-----|-----|
    | **-e "ACCEPT_EULA=Y"** |  Définissez la variable **ACCEPT_EULA** sur n’importe quelle valeur pour confirmer que vous acceptez le [Contrat de licence utilisateur final](https://go.microsoft.com/fwlink/?linkid=2139274). Paramètre requis pour l’image Azure SQL Edge. |
    | **-e "MSSQL_SA_PASSWORD=yourStrong(!)Password"** | Spécifiez votre propre mot de passe fort, qui doit avoir au moins 8 caractères et respecter les [exigences de mot de passe Azure SQL Edge](/sql/relational-databases/security/password-policy). Paramètre requis pour l’image Azure SQL Edge. |
    | **-p 1433:1433** | Mappez un port TCP sur l’environnement hôte (première valeur) à un port TCP dans le conteneur (deuxième valeur). Dans cet exemple, Azure SQL Edge écoute sur TCP 1433 dans le conteneur, mappé au port 1433 sur l’hôte. |
    | **--name azuresqledge** | Spécifiez un nom personnalisé pour le conteneur plutôt qu’un nom généré de manière aléatoire. Si vous exécutez plusieurs conteneurs, vous ne pouvez pas réutiliser le même nom. |
    | **-d** | Exécutez le conteneur en arrière-plan (démon) |

    Pour obtenir la liste complète de toutes les variables d’environnement Azure SQL Edge, consultez [Configurer Azure SQL Edge avec des variables d’environnement](configure.md#configure-by-using-environment-variables). Vous pouvez également utiliser un [fichier mssql.conf](configure.md#configure-by-using-an-mssqlconf-file) pour configurer des conteneurs Azure SQL Edge.

3. Pour afficher vos conteneurs Docker, utilisez la commande `docker ps`.
   
   ```bash
    sudo docker ps -a
   ```

4. Si la colonne **ÉTAT** affiche **En cours d’exécution** , Azure SQL Edge est en cours d’exécution dans le conteneur et écoute sur le port spécifié dans la colonne **PORTS**. Si la colonne **ÉTAT** pour votre conteneur Azure SQL Edge affiche **Quitté** , consultez la section Résolution des problèmes de la documentation Azure SQL Edge.

    Le paramètre `-h` (nom d’hôte) est également utile, mais il n’est pas utilisé dans ce didacticiel pour plus de simplicité. Il permet de changer le nom interne du conteneur en une valeur personnalisée. Il s’agit du nom retourné dans la requête Transact-SQL suivante :

    ```sql
    SELECT @@SERVERNAME,
        SERVERPROPERTY('ComputerNamePhysicalNetBIOS'),
        SERVERPROPERTY('MachineName'),
        SERVERPROPERTY('ServerName')
    ```

    Pour identifier facilement le conteneur cible, définissez `-h` et `--name` sur la même valeur.

5. Pour finir, modifiez votre mot de passe AS car le `SA_PASSWORD` est visible dans la sortie`ps -eax` et stocké dans la variable d’environnement du même nom. Consultez les étapes ci-dessous.

## <a name="change-the-sa-password"></a>Changer le mot de passe AS

Le compte **SA** est un administrateur système sur l’instance Azure SQL Edge créée lors de l’installation. Une fois le conteneur Azure SQL Edge créé, la variable d’environnement `MSSQL_SA_PASSWORD` que vous avez spécifiée peut être découverte en exécutant `echo $SA_PASSWORD` dans le conteneur. Pour des raisons de sécurité, changez le mot de passe pour SA.

1. Choisissez un mot de passe fort à utiliser pour l’utilisateur SA.

2. Utilisez `docker exec` pour exécuter **sqlcmd** pour changer le mot de passe avec Transact-SQL. Dans l’exemple suivant, remplacez l’ancien mot de passe, `<YourStrong!Passw0rd>`, et le nouveau mot de passe, `<YourNewStrong!Passw0rd>`, par vos propres valeurs de mot de passe.

   ```bash
   sudo docker exec -it azuresqledge /opt/mssql-tools/bin/sqlcmd \
      -S localhost -U SA -P "<YourStrong@Passw0rd>" \
      -Q 'ALTER LOGIN SA WITH PASSWORD="<YourNewStrong@Passw0rd>"'
   ```

## <a name="connect-to-azure-sql-edge"></a>Se connecter à Azure SQL Edge

La procédure suivante utilise l’outil en ligne de commande Azure SQL Edge, **sqlcmd** , dans le conteneur pour se connecter à Azure SQL Edge.

> [!NOTE]
> L’outil sqlcmd n’est pas disponible dans la version ARM64 des conteneurs SQL Edge.

1. Utilisez la commande `docker exec -it` pour démarrer un interpréteur de commandes bash interactif dans votre conteneur en cours d’exécution. Dans l’exemple suivant, `azuresqledge` est le nom spécifié par le paramètre `--name` quand vous avez créé le conteneur.

   ```bash
   sudo docker exec -it azuresqledge "bash"
   ```

2. Une fois dans le conteneur, connectez-vous localement avec sqlcmd. Sqlcmd n’est pas dans le chemin par défaut, vous devez spécifier le chemin complet.

   ```bash
   /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "<YourNewStrong@Passw0rd>"
   ```

   > [!TIP]
   > Vous pouvez omettre le mot de passe sur la ligne de commande pour être invité à l’entrer.

3. Si l’opération réussit, vous devez accéder à une invite de commandes **sqlcmd**  : `1>`.

## <a name="create-and-query-data"></a>Créer et interroger des données

Les sections suivantes vous guident lors de l’utilisation de **sqlcmd** et Transact-SQL pour créer une base de données, ajouter des données et exécuter une requête simple.

### <a name="create-a-new-database"></a>Créer une base de données

La procédure suivante crée une base de données nommée `TestDB`.

1. À partir de l’invite de commandes **sqlcmd** , collez la commande Transact-SQL suivante pour créer une base de données de test :

   ```sql
   CREATE DATABASE TestDB
   Go
   ```

2. Sur la ligne suivante, écrivez une requête pour retourner le nom de toutes les bases de données sur votre serveur :

   ```sql
   SELECT Name from sys.Databases
   Go
   ```

### <a name="insert-data"></a>Insertion des données

Créez ensuite une table, `Inventory`, et insérez deux nouvelles lignes.

1. À partir de l’invite de commandes **sqlcmd** , basculez le contexte vers la nouvelle base de données `TestDB` :

   ```sql
   USE TestDB
   ```

2. Créez une table nommée `Inventory` :

   ```sql
   CREATE TABLE Inventory (id INT, name NVARCHAR(50), quantity INT)
   ```

3. Insérez des données dans la nouvelle table :

   ```sql
   INSERT INTO Inventory VALUES (1, 'banana', 150); INSERT INTO Inventory VALUES (2, 'orange', 154);
   ```

4. Tapez `GO` pour exécuter les commandes précédentes :

   ```sql
   GO
   ```

### <a name="select-data"></a>Sélectionner les données

Exécutez maintenant une requête pour retourner des données de la table `Inventory`.

1. Dans l’invite de commandes **sqlcmd** , entrez une requête qui retourne les lignes de la table `Inventory` dont la quantité est supérieure à 152 :

   ```sql
   SELECT * FROM Inventory WHERE quantity > 152;
   ```

2. Exécutez la commande :

   ```sql
   GO
   ```

### <a name="exit-the-sqlcmd-command-prompt"></a>Quitter l’invite de commandes sqlcmd

1. Pour mettre fin à votre session **sqlcmd** , tapez `QUIT` :

   ```sql
   QUIT
   ```

2. Pour quitter l’invite de commandes interactive dans votre conteneur, tapez `exit`. Le conteneur continue de s’exécuter une fois que vous avez quitté l’interpréteur de commandes bash interactif.

## <a name="connect-from-outside-the-container"></a> Se connecter en dehors du conteneur

Vous pouvez aussi vous connecter à l’instance Azure SQL Edge sur votre machine Docker à partir de n’importe quel outil externe Linux, Windows ou macOS qui prend en charge les connexions SQL. Pour plus d’informations sur la connexion à un conteneur SQL Edge depuis l’extérieur, consultez [Connecter et interroger Azure SQL Edge](connect.md).

## <a name="remove-your-container"></a>Supprimer votre conteneur

Si vous voulez supprimer le conteneur Azure SQL Edge utilisé dans ce didacticiel, exécutez les commandes suivantes :

```bash
sudo docker stop azuresqledge
sudo docker rm azuresqledge
```

> [!WARNING]
> L’arrêt et la suppression d’un conteneur supprime définitivement toutes les données Azure SQL Edge dans le conteneur. Si vous avez besoin de conserver vos données, [créez et copiez un fichier de sauvegarde en dehors du conteneur](backup-restore.md) ou utilisez une [technique de persistance de données de conteneur](configure.md#persist-your-data).

## <a name="next-steps"></a>Étapes suivantes

- [Machine Learning et intelligence artificielle avec ONNX dans SQL Edge](onnx-overview.md).
- [Création d’une solution IoT de bout en bout avec SQL Edge à l’aide d’IoT Edge](tutorial-deploy-azure-resources.md).
- [Streaming de données dans Azure SQL Edge](stream-data.md)