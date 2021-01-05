---
title: Configurer des modules IoT Edge dans Azure SQL Edge
description: Dans la deuxième partie de ce tutoriel Azure SQL Edge en trois parties consacré à la prédiction des impuretés contenues dans le minerai de fer, vous allez configurer les modules et les connexions IoT Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: VasiyaKrishnan
ms.author: vakrishn
ms.reviewer: sourabha, sstein
ms.date: 09/22/2020
ms.openlocfilehash: 75e6ebaea4c5ba883820d2309212b35fed128142
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422125"
---
# <a name="set-up-iot-edge-modules-and-connections"></a>Configurer les modules et les connexions IoT Edge

Dans la deuxième partie de ce tutoriel en trois parties consacré à la prédiction des impuretés contenues dans le minerai de fer dans Azure SQL Edge, vous allez configurer les modules IoT Edge suivants :

- Azure SQL Edge
- Module IoT Edge de génération de données

## <a name="specify-container-registry-credentials"></a>Spécifier les informations d'identification au registre de conteneurs

Les informations d'identification aux registres de conteneurs hébergeant les images de module doivent être spécifiées. Celles-ci se trouvent dans le registre de conteneurs qui a été créé dans votre groupe de ressources. Accédez à la section **Clés d'accès**. Prenez note des champs suivants :

- Nom du registre
- Serveur de connexion
- Nom d’utilisateur
- Mot de passe

À présent, spécifiez les informations d'identification au conteneur dans le module IoT Edge.

1. Accédez au hub IoT qui a été créé dans votre groupe de ressources.

2. Dans la section **IoT Edge**, sous **Gestion automatique des appareils**, cliquez sur **ID d'appareil**. Pour ce tutoriel, l'ID est `IronOrePredictionDevice`.

3. Sélectionnez la section **Définir des modules**.

4. Sous **Informations d'identification au registre de conteneurs**, entrez les valeurs suivantes :

   _Champ_|_Valeur_
   -------|-------
   Nom|Nom du registre
   Adresse|Serveur de connexion
   User Name|Nom d’utilisateur
   Mot de passe|Mot de passe
  
## <a name="build-push-and-deploy-the-data-generator-module"></a>Générer, envoyer (push) et déployer le module Générateur de données

1. Clonez les [fichiers projet](https://github.com/microsoft/sqlsourabh/tree/main/SQLEdgeSamples/IoTEdgeSamples/IronOreSilica) sur votre machine.
2. Ouvrez le fichier **IronOre_Silica_Predict.sln** à l’aide de Visual Studio 2019.
3. Mettez à jour les détails du registre de conteneurs dans **deployment.template.json**. 
   ```json
   "registryCredentials":{
        "RegistryName":{
            "username":"",
            "password":""
            "address":""
        }
    }
   ```
4. Mettez à jour le fichier **modules.json** de façon à spécifier le registre de conteneurs cible (ou le référentiel pour le module)
   ```json
   "image":{
        "repository":"samplerepo.azurecr.io/ironoresilicapercent",
        "tag":
    }
   ```
5. Exécutez le projet en mode Debug ou Release pour vérifier qu’il s’exécute sans aucun problème. 
6. Envoyez (push) le projet vers votre registre de conteneurs en cliquant avec le bouton droit sur le nom du projet, puis en sélectionnant **Générer et envoyer (push) les modules IoT Edge**.
7. Déployez le module Générateur de données en tant que module IoT Edge sur votre appareil Edge. 

## <a name="deploy-the-azure-sql-edge-module"></a>Déployer le module Azure SQL Edge

1. Déployez le module Azure SQL Edge en cliquant sur **+ Ajouter** puis sur **Module de la Place de marché**. 

2. Dans le panneau **Place de marché de module IoT Edge**, recherchez *Azure SQL Edge*, puis sélectionnez *Azure SQL Edge - Développeur*. 

3. Cliquez sur le module *Azure SQL Edge* que vous venez d’ajouter sous **Modules IoT Edge** pour configurer le module Azure SQL Edge. Pour plus d’informations sur les options de configuration, consultez [Déployer Azure SQL Edge](./deploy-portal.md).

4. Ajoutez la variable d’environnement `MSSQL_PACKAGE` au déploiement du module *Azure SQL Edge*, puis spécifiez l’URL SAS du fichier dacpac de base de données créé à l’étape 8 de la [première partie](tutorial-deploy-azure-resources.md) de ce tutoriel.

5. Cliquez sur **Update** (Mettre à jour).

6. Dans la page **Définir des modules sur l’appareil**, cliquez sur **Étape suivante : Routes >** .

7. Dans le volet Routes de la page **Définir des modules sur l’appareil**, spécifiez les routes pour la communication entre le module et le hub IoT Edge, comme décrit ci-dessous. Mettez à jour le nom des modules dans les définitions de routes ci-dessous.

   ```
   FROM /messages/modules/<your_data_generator_module>/outputs/IronOreMeasures INTO
   BrokeredEndpoint("/modules/<your_azure_sql_edge_module>/inputs/IronOreMeasures")
   ```

   Par exemple :

   ```
   FROM /messages/modules/ASEDataGenerator/outputs/IronOreMeasures INTO BrokeredEndpoint("/modules/AzureSQLEdge/inputs/IronOreMeasures")
   ```


7. Dans la page **Définir des modules sur l’appareil**, cliquez sur **Étape suivante : Vérifier + créer >**

8. Dans la page **Définir des modules sur l’appareil**, cliquez sur **Créer**.

## <a name="create-and-start-the-t-sql-streaming-job-in-azure-sql-edge"></a>Créez puis démarrez une tâche de streaming T-SQL dans Azure SQL Edge.

1. Ouvrez Azure Data Studio.

2. Sous l'onglet **Bienvenue**, lancez une nouvelle connexion avec les informations suivantes :

   |_Champ_|_Valeur_|
   |-------|-------|
   |Type de connexion| Microsoft SQL Server|
   |Serveur|Adresse IP publique mentionnée dans la machine virtuelle créée pour cette démo|
   |Nom d’utilisateur|SA|
   |Mot de passe|Mot de passe fort utilisé lors de la création de l'instance d'Azure SQL Edge|
   |Base de données|Default|
   |Groupe de serveurs|Default|
   |Name (facultatif)|Entrez un nom facultatif|

3. Cliquez sur **Connexion**

4. Sous l’onglet du menu **Fichier**, ouvrez un nouveau notebook ou utilisez le raccourci clavier Ctrl + N.

5. Dans la nouvelle fenêtre de requête, exécutez le script ci-dessous pour créer la tâche de streaming T-SQL. Avant d’exécuter le script, changez les variables suivantes. 
   - *SQL_SA_Password :* Valeur MSSQL_SA_PASSWORD spécifiée lors du déploiement du module Azure SQL Edge. 
   
   ```sql
   Use IronOreSilicaPrediction
   Go

   Declare @SQL_SA_Password varchar(200) = '<SQL_SA_Password>'
   declare @query varchar(max) 

   /*
   Create Objects Required for Streaming
   */

   CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'MyStr0ng3stP@ssw0rd';

   If NOT Exists (select name from sys.external_file_formats where name = 'JSONFormat')
   Begin
      CREATE EXTERNAL FILE FORMAT [JSONFormat]  
      WITH ( FORMAT_TYPE = JSON)
   End 


   If NOT Exists (select name from sys.external_data_sources where name = 'EdgeHub')
   Begin
      Create EXTERNAL DATA SOURCE [EdgeHub] 
      With(
         LOCATION = N'edgehub://'
      )
   End 

   If NOT Exists (select name from sys.external_streams where name = 'IronOreInput')
   Begin
      CREATE EXTERNAL STREAM IronOreInput WITH 
      (
         DATA_SOURCE = EdgeHub,
         FILE_FORMAT = JSONFormat,
         LOCATION = N'IronOreMeasures'
       )
   End


   If NOT Exists (select name from sys.database_scoped_credentials where name = 'SQLCredential')
   Begin
       set @query = 'CREATE DATABASE SCOPED CREDENTIAL SQLCredential
                 WITH IDENTITY = ''sa'', SECRET = ''' + @SQL_SA_Password + ''''
       Execute(@query)
   End 

   If NOT Exists (select name from sys.external_data_sources where name = 'LocalSQLOutput')
   Begin
      CREATE EXTERNAL DATA SOURCE LocalSQLOutput WITH (
      LOCATION = 'sqlserver://tcp:.,1433',CREDENTIAL = SQLCredential)
   End

   If NOT Exists (select name from sys.external_streams where name = 'IronOreOutput')
   Begin
      CREATE EXTERNAL STREAM IronOreOutput WITH 
      (
         DATA_SOURCE = LocalSQLOutput,
         LOCATION = N'IronOreSilicaPrediction.dbo.IronOreMeasurements'
      )
   End

   EXEC sys.sp_create_streaming_job @name=N'IronOreData',
   @statement= N'Select * INTO IronOreOutput from IronOreInput'

   exec sys.sp_start_streaming_job @name=N'IronOreData'
   ```

6. Utilisez la requête suivante pour vérifier que les données du module de génération de données sont envoyées en streaming à la base de données. 

   ```sql
   Select Top 10 * from dbo.IronOreMeasurements
   order by timestamp desc
   ```


Dans ce tutoriel, nous avons déployé le module de génération de données et le module SQL Edge. Ensuite, nous avons créé une tâche de streaming pour envoyer en streaming les données générées par le module de génération de données vers SQL. 

## <a name="next-steps"></a>Étapes suivantes

- [Déployer un modèle Machine Learning sur Azure SQL Edge à l'aide d'ONNX](tutorial-run-ml-model-on-sql-edge.md)