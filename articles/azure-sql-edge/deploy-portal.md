---
title: Déployer Azure SQL Edge à l’aide du portail Azure
description: Découvrez comment déployer Azure SQL Edge à l’aide du portail Azure
keywords: déployer SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 5d7d232ada814d5d3c30e7b012c6289f847d641f
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93395085"
---
# <a name="deploy-azure-sql-edge"></a>Déployer Azure SQL Edge 

Azure SQL Edge est un moteur de base de données relationnelle optimisé pour les déploiements IoT et Azure IoT Edge. Il offre des fonctionnalités permettant de créer une couche de traitement et de stockage des données hautes performances pour les solutions et applications IoT. Ce guide de démarrage rapide vous montre comment prendre en main la création d’un module Azure SQL Edge via Azure IoT Edge à l’aide du portail Azure.

## <a name="before-you-begin"></a>Avant de commencer

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/).
* Connectez-vous au [portail Azure](https://portal.azure.com/).
* Créez une ressource [Azure IoT Hub](../iot-hub/iot-hub-create-through-portal.md).
* Créez un [appareil Azure IoT Edge](../iot-edge/how-to-install-iot-edge.md).

> [!NOTE]   
> Pour déployer une machine virtuelle Linux Azure en tant qu’appareil IoT Edge, consultez ce [guide de démarrage rapide](../iot-edge/quickstart-linux.md).

## <a name="deploy-sql-edge-module-from-azure-marketplace"></a>Déployer le module SQL Edge à partir de la Place de marché Azure

La Place de marché Azure est un marché d’applications et de services en ligne qui vous permet de naviguer au milieu d’un large éventail d’applications et de solutions d’entreprise certifiées et optimisées pour s’exécuter sur Azure, notamment les [modules IoT Edge](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules). Azure SQL Edge peut être déployé sur un périphérique via la Place de marché.

1. Recherchez le module Azure SQL Edge sur la Place de marché Azure.<br><br>

   ![SQL Edge sur la Place de marché](media/deploy-portal/find-offer-marketplace.png)

2. Sélectionnez l’abonnement logiciel qui correspond le mieux à vos besoins, puis cliquez sur **Créer**. <br><br>

   ![Sélectionner l’abonnement logiciel approprié](media/deploy-portal/pick-correct-plan.png)

3. Sur la page des appareils cibles pour le module IoT Edge, spécifiez les informations suivantes, puis cliquez sur **Créer**

   |**Champ**  |**Description**  |
   |---------|---------|
   |Abonnement  |  L’abonnement Azure dans lequel l’IoT Hub a été créé |
   |IoT Hub   |  Nom de l’IoT Hub où l’appareil IoT Edge est inscrit. Sélectionnez ensuite l’option « Déployer sur un appareil »|
   |Nom de l’appareil IoT Edge  |  Nom de l’appareil IoT Edge sur lequel SQL Edge sera déployé |

4. Dans la page **Définir des modules sur l’appareil** , cliquez sur le module Azure SQL Edge sous **Modules IoT Edge**. Le nom de module par défaut est défini sur *AzureSQLEdge*. 

5. Dans la section *Paramètres du module* du panneau **Mettre à jour le module IoT Edge** , spécifiez les valeurs souhaitées pour *Nom du module IoT Edge* , *Stratégie de redémarrage* et *État souhaité*. 

   > [!IMPORTANT]    
   > Ne modifiez pas ou ne mettez pas à jour les paramètres d’ **URI d’image** du module.

6. Dans la section *Variables d’environnement* du panneau **Mettre à jour le module IoT Edge** , spécifiez les valeurs souhaitées pour les variables d’environnement. Pour obtenir la liste complète des variables d’environnement d’Azure SQL Edge, consultez [Configurer à l’aide de variables d’environnement](configure.md#configure-by-using-environment-variables). Les variables d’environnement par défaut suivantes sont définies pour le module. 

   |**Paramètre**  |**Description**|
   |---------|---------|
   | MSSQL_SA_PASSWORD  | Modifiez la valeur par défaut pour spécifier un mot de passe fort pour le compte d’administrateur SQL Edge. |
   | MSSQL_LCID   | Modifiez la valeur par défaut en définissant l’ID de langue souhaitée à utiliser pour SQL Edge. Par exemple, 1036 correspond au français. |
   | MSSQL_COLLATION | Modifiez la valeur par défaut en définissant le classement par défaut pour SQL Edge. Ce paramètre remplace le mappage par défaut de l’ID de langue (LCID) par le classement. |

   > [!IMPORTANT]    
   > Ne modifiez pas ou ne mettez pas à jour la variable d’environnement **ACCEPT_EULA** pour le module.

7. Dans la section *Options de création de conteneur* du panneau **Mettre à jour le module IoT Edge** , mettez à jour les options suivantes en fonction de l’exigence. 
   - **Port de l’hôte :** Mappez le port d’hôte spécifié au port 1433 (port SQL par défaut) dans le conteneur.
   - **Liaisons** et **Montages**  : Si vous devez déployer plusieurs modules SQL Edge, veillez à mettre à jour l’option Montages pour créer une paire source-cible pour le volume persistant. Pour plus d’informations sur les montages et le volume, consultez [Utiliser des volumes](https://docs.docker.com/storage/volumes/) sur la documentation Docker. 

   ```json
   {
    "HostConfig": {
        "CapAdd": [
            "SYS_PTRACE"
        ],
        "Binds": [
            "sqlvolume:/sqlvolume"
        ],
        "PortBindings": {
            "1433/tcp": [
                {
                    "HostPort": "1433"
                }
            ]
        },
        "Mounts": [
            {
                "Type": "volume",
                "Source": "sqlvolume",
                "Target": "/var/opt/mssql"
            }
        ]
    },
    "Env": [
        "MSSQL_AGENT_ENABLED=TRUE",
        "ClientTransportType=AMQP_TCP_Only",
        "PlanId=asde-developer-on-iot-edge"
    ]
   }
   ```
   > [!IMPORTANT]    
   > Ne modifiez pas la variable d’environnement `PlanId` définie dans le paramètre Créer une configuration. Si vous modifiez cette valeur, le démarrage du conteneur Azure SQL Edge échouera. 
   
8. Dans le volet **Mettre à jour le module IoT Edge** , cliquez sur **Mettre à jour**.
9. Dans la page **Définir des modules sur l’appareil** , cliquez sur **Suivant : Itinéraires >** si vous devez définir des itinéraires pour votre déploiement. Sinon, cliquez sur **Vérifier + créer**. Pour plus d’informations sur la configuration d’itinéraires, consultez [Déployer des modules et établir des routes dans IoT Edge](../iot-edge/module-composition.md).
11. Dans la page **Définir des modules sur l’appareil** , cliquez sur **Créer**.

## <a name="connect-to-azure-sql-edge"></a>Se connecter à Azure SQL Edge

La procédure suivante utilise l’outil en ligne de commande Azure SQL Edge, **sqlcmd** , dans le conteneur pour se connecter à Azure SQL Edge.

> [!NOTE]      
> Les outils en ligne de commande SQL (sqlcmd) ne sont pas disponibles dans la version ARM64 des conteneurs Azure SQL Edge.

1. Utilisez la commande `docker exec -it` pour démarrer un interpréteur de commandes bash interactif dans votre conteneur en cours d’exécution. Dans l’exemple suivant, `azuresqledge` est le nom spécifié par le paramètre `Name` de votre module IoT Edge.

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

Les sections suivantes vous guident lors de l’utilisation de **sqlcmd** et Transact-SQL pour créer une base de données, ajouter des données et exécuter une requête.

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

Vous pouvez vous connecter et exécuter des requêtes SQL sur votre instance Azure SQL Edge à partir de n’importe quel outil externe Linux, Windows ou macOS qui prend en charge les connexions SQL. Pour plus d’informations sur la connexion à un conteneur SQL Edge depuis l’extérieur, consultez [Connecter et interroger Azure SQL Edge](./connect.md).

Dans ce guide de démarrage rapide, vous avez déployé un module SQL Edge sur un appareil IoT Edge. 

## <a name="next-steps"></a>Étapes suivantes

- [Machine Learning et intelligence artificielle avec ONNX dans SQL Edge](onnx-overview.md)
- [Création d’une solution IoT de bout en bout avec SQL Edge à l’aide d’IoT Edge](tutorial-deploy-azure-resources.md)
- [Streaming de données dans Azure SQL Edge](stream-data.md)
- [Résoudre les erreurs de déploiement](troubleshoot.md)