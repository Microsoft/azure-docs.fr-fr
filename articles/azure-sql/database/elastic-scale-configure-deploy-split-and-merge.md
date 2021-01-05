---
title: Déployer un service de fractionnement et de fusion
description: Utilisez l’outil de fractionnement et de fusion pour déplacer les données entre les différentes bases de données partitionnées.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 12/04/2018
ms.openlocfilehash: 71aad7699c5af6ce2a1b9d82a340138200cfb5e1
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792070"
---
# <a name="deploy-a-split-merge-service-to-move-data-between-sharded-databases"></a>Déployer un service de fractionnement et de fusion pour déplacer des données entre bases de données partitionnées
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

L’outil de fractionnement et de fusion vous permet de déplacer les données entre les différentes bases de données partitionnées. Consultez [Déplacement de données entre des bases de données cloud montées en charge](elastic-scale-overview-split-and-merge.md)

## <a name="download-the-split-merge-packages"></a>Téléchargement des packages du service de fractionnement/fusion

1. Téléchargez la dernière version de NuGet à partir de [NuGet](https://docs.nuget.org/docs/start-here/installing-nuget).

1. Ouvrez une invite de commandes et accédez au répertoire où vous avez téléchargé nuget.exe. Le téléchargement inclut les commandes PowerShell.

1. Téléchargez le dernier package de fractionnement/fusion dans le répertoire actif à l’aide de la commande ci-dessous :

   ```cmd
   nuget install Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge
   ```  

Les fichiers sont placés dans un répertoire nommé **Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge.x.x.xxx.x** où *x.x.xxx.x* correspond au numéro de version. Recherchez les fichiers du service de fractionnement et de fusion dans le sous-répertoire **content\splitmerge\service** et les scripts PowerShell de fractionnement et de fusion (ainsi que les dll clientes nécessaires) dans le sous-répertoire **content\splitmerge\powershell** .

## <a name="prerequisites"></a>Prérequis

1. Créez une base de données Azure SQL Database qui servira de base de données d’état de l’état de fractionnement/fusion. Accédez au [portail Azure](https://portal.azure.com). Créez une **base de données SQL** . Nommez la base de données et créez un administrateur ainsi qu’un mot de passe. Veillez à enregistrer le nom et le mot de passe pour une utilisation ultérieure.

1. Vérifiez que votre serveur autorise la connexion des services Azure. Dans le portail, dans **Paramètres du pare-feu** , vérifiez que le paramètre **Autoriser l’accès aux services Azure** a la valeur **Activé** . Cliquez sur l’icône « Enregistrer ».

1. Créez un compte de Stockage Azure pour les sorties de diagnostics.

1. Créez un service cloud Azure pour votre service de Fractionnement-Fusion.

## <a name="configure-your-split-merge-service"></a>Configurer votre service de fractionnement et de fusion

### <a name="split-merge-service-configuration"></a>Configuration du service de fractionnement/fusion

1. Dans le dossier où vous avez téléchargé les assemblys de fractionnement et de fusion, créez une copie du fichier *ServiceConfiguration.Template.cscfg* fourni avec *SplitMergeService.cspkg* , puis renommez-la *ServiceConfiguration.cscfg* .

1. Ouvrez *ServiceConfiguration.cscfg* dans un éditeur de texte de type Visual Studio qui valide les entrées telles que le format des empreintes numériques de certificat.

1. Créez une base de données ou sélectionnez-en une qui fera office de base de données d’état pour les opérations de fractionnement-fusion et qui permettra de récupérer la chaîne de connexion de la base de données concernée.

   > [!IMPORTANT]
   > À ce stade, la base de données de l’état doit utiliser le classement Latin (SQL\_Latin1\_General\_CP1\_CI\_AS). Pour plus d’informations, consultez [Nom de classement Windows (Transact-SQL)](/sql/t-sql/statements/windows-collation-name-transact-sql).

   Dans Azure SQL Database, la chaîne de connexion se présente généralement sous la forme suivante :

      `Server=<serverName>.database.windows.net; Database=<databaseName>;User ID=<userId>; Password=<password>; Encrypt=True; Connection Timeout=30`

1. Entrez cette chaîne de connexion dans le fichier *.cscfg* dans les sections de rôle **SplitMergeWeb** et **SplitMergeWorker** du paramètre ElasticScaleMetadata.

1. Pour le rôle **SplitMergeWorker** , entrez une chaîne de connexion valide pour le stockage Azure pour le paramètre **WorkerRoleSynchronizationStorageAccountConnectionString** .

### <a name="configure-security"></a>Configurer la sécurité

Pour obtenir des instructions détaillées permettant de configurer la sécurité du service, reportez-vous à la [configuration de la sécurité de la fusion et du fractionnement](elastic-scale-split-merge-security-configuration.md).

Dans le cadre d’un simple déploiement de test pour ce didacticiel, le bon fonctionnement du service nécessite d’effectuer un nombre minimum d’étapes de configuration. Ces dernières permettent uniquement à l’ordinateur/au compte qui les exécute de communiquer avec le service.

### <a name="create-a-self-signed-certificate"></a>Créer un certificat auto-signé

Créez un répertoire à partir duquel vous allez exécuter la commande suivante à l’aide d’une fenêtre [d’invite de commandes développeur pour Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs) :

   ```cmd
   makecert ^
    -n "CN=*.cloudapp.net" ^
    -r -cy end -sky exchange -eku "1.3.6.1.5.5.7.3.1,1.3.6.1.5.5.7.3.2" ^
    -a sha256 -len 2048 ^
    -sr currentuser -ss root ^
    -sv MyCert.pvk MyCert.cer
   ```

Un mot de passe vous est demandé pour protéger la clé privée. Entrez un mot de passe fort et confirmez-le. Vous êtes ensuite de nouveau invité à entrer le mot de passe. Cliquez sur **Oui** à la fin pour l’importer dans le magasin racine des autorités de certification approuvées.

### <a name="create-a-pfx-file"></a>Création d’un fichier PFX

Exécutez la commande suivante à partir de la même fenêtre que celle où makecert a été exécuté. Utilisez le même mot de passe que celui utilisé pour créer le certificat :

   ```cmd
   pvk2pfx -pvk MyCert.pvk -spc MyCert.cer -pfx MyCert.pfx -pi <password>
   ```

### <a name="import-the-client-certificate-into-the-personal-store"></a>Importation du certificat client dans le magasin personnel

1. Dans l’Explorateur Windows, double-cliquez sur *MyCert.pfx* .
2. Dans **l’Assistant Importation de certificat** , sélectionnez **Utilisateur actuel** et cliquez sur **Suivant** .
3. Vérifiez le chemin d’accès au fichier et cliquez sur **Suivant** .
4. Tapez le mot de passe, laissez l’option **Inclure toutes les propriétés étendues** activée, puis cliquez sur **Suivant** .
5. Laissez l’option **Sélectionner automatiquement le magasin de certificats…** activée, puis cliquez sur **Suivant** .
6. Cliquez sur **Terminer** et sur **OK** .

### <a name="upload-the-pfx-file-to-the-cloud-service"></a>Téléchargement du fichier PFX dans le service cloud

1. Accédez au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Services Cloud** .
3. Sélectionnez le service cloud créé ci-dessus pour le service de fractionnement/fusion.
4. Dans le menu supérieur, cliquez sur **Certificats** .
5. Cliquez sur **Télécharger** dans la barre inférieure.
6. Sélectionnez le fichier PFX et entrez le même mot de passe que ci-dessus.
7. Lorsque vous avez terminé, copiez l’empreinte de certificat à partir de la nouvelle entrée dans la liste.

### <a name="update-the-service-configuration-file"></a>Mise à jour du fichier de configuration de service

Collez l’empreinte de certificat copiée précédemment dans l’attribut d’empreinte/de valeur des paramètres suivants.
Pour le rôle de travail :

   ```xml
    <Setting name="DataEncryptionPrimaryCertificateThumbprint" value="" />
    <Certificate name="DataEncryptionPrimary" thumbprint="" thumbprintAlgorithm="sha1" />
   ```

Pour le rôle web :

   ```xml
    <Setting name="AdditionalTrustedRootCertificationAuthorities" value="" />
    <Setting name="AllowedClientCertificateThumbprints" value="" />
    <Setting name="DataEncryptionPrimaryCertificateThumbprint" value="" />
    <Certificate name="SSL" thumbprint="" thumbprintAlgorithm="sha1" />
    <Certificate name="CA" thumbprint="" thumbprintAlgorithm="sha1" />
    <Certificate name="DataEncryptionPrimary" thumbprint="" thumbprintAlgorithm="sha1" />
   ```

Veuillez noter que pour les déploiements de production, des certificats distincts doivent être utilisés pour l’autorité de certification, le chiffrement, le certificat de serveur et les certificats clients. Pour plus d’informations, consultez la rubrique [Configuration de la sécurité](elastic-scale-split-merge-security-configuration.md).

## <a name="deploy-your-service"></a>Déployer votre service

1. Accédez au [Portail Azure](https://portal.azure.com).
2. Sélectionnez le service cloud que vous avez créé précédemment.
3. Cliquez sur **Overview** .
4. Choisissez l’environnement de préproduction, puis cliquez sur **Charger** .
5. Dans la boîte de dialogue, entrez une étiquette de déploiement. Pour Package et Configuration, cliquez sur À partir de local, puis choisissez le fichier *SplitMergeService.cspkg* et le fichier cscfg que vous avez configuré précédemment.
6. Assurez-vous que la case **Déployer même si un ou plusieurs rôles contiennent une seule instance** est cochée.
7. Appuyez sur le bouton en forme de coche dans le coin inférieur droit pour commencer le déploiement. Cette opération peut prendre plusieurs minutes.

## <a name="troubleshoot-the-deployment"></a>Résoudre les problèmes de déploiement

Si votre rôle web ne parvient pas à être en ligne, il s’agit probablement d’un problème avec la configuration de sécurité. Vérifiez que le protocole TLS/SSL est configuré comme décrit ci-dessus.

Si votre rôle de travail ne parvient pas à être en ligne, mais que votre rôle web réussit, il s’agit probablement d’un problème de connexion à la base de données d’états que vous avez créée précédemment.

- Vérifiez que la chaîne de connexion du fichier cscfg ne contient aucune erreur.
- Vérifiez que le serveur et la base de données existent et que l’identifiant utilisateur et le mot de passe sont corrects.
- Dans Azure SQL Database, la chaîne de connexion doit se présenter sous la forme suivante :

   `Server=<serverName>.database.windows.net; Database=<databaseName>;User ID=<user>; Password=<password>; Encrypt=True; Connection Timeout=30`

- Assurez-vous que le nom du serveur ne commence pas par **https://** .
- Vérifiez que votre serveur autorise la connexion des services Azure. Pour ce faire, ouvrez votre base de données dans le portail et vérifiez que le paramètre **Autoriser l’accès aux services Azure** est défini sur **Activé** **.

## <a name="test-the-service-deployment"></a>Tester le déploiement du service

### <a name="connect-with-a-web-browser"></a>Se connecter avec un navigateur Web

Déterminez le point de terminaison web de votre service de fractionnement/fusion. Vous pouvez le trouver dans le portail en accédant à la **Vue d’ensemble** de votre service cloud et en effectuant une recherche dans la zone **URL du site** située sur la droite. Remplacez **http://** par **https://** , car les paramètres de sécurité par défaut désactivent le point de terminaison HTTP. Chargez la page correspondant à cette URL dans votre navigateur.

### <a name="test-with-powershell-scripts"></a>Effectuer des tests avec des scripts PowerShell

Le déploiement et votre environnement peuvent être testés en exécutant les exemples de scripts PowerShell fournis.

> [!IMPORTANT]
> Les exemples de scripts s’exécutent sur PowerShell 5.1. Ils ne s’exécutent actuellement pas sur PowerShell 6 ou version ultérieure.

Les fichiers de script inclus sont les suivants :

1. *SetupSampleSplitMergeEnvironment.ps1* : configure une couche de données de test pour la fusion et le fractionnement (voir le tableau ci-dessous pour obtenir une description détaillée)
2. *ExecuteSampleSplitMerge.ps1* : exécute les opérations de test sur la couche de données de test (voir le tableau ci-dessous pour obtenir une description détaillée)
3. *GetMappings.ps1*  : exemple de script de niveau supérieur qui imprime l’état actuel des mappages de partitions.
4. *ShardManagement.psm1*  : script d’assistance qui encapsule l’API ShardManagement
5. *SqlDatabaseHelpers.psm1* : script d’assistance pour la création et la gestion des bases de données dans SQL Database

   <table style="width:100%">
     <tr>
       <th>Fichier PowerShell</th>
       <th>Étapes</th>
     </tr>
     <tr>
       <th rowspan="5">SetupSampleSplitMergeEnvironment.ps1</th>
       <td>1. Crée une base de données pour le Gestionnaire de cartes de partitions</td>
     </tr>
     <tr>
       <td>2. Crée 2&#160;bases de données de partition.
     </tr>
     <tr>
       <td>3. Crée une carte de partitions pour les bases de données concernées (supprime les cartes de partitions existantes sur ces dernières). </td>
     </tr>
     <tr>
       <td>4. Crée un petit exemple de table dans les deux partitions et remplit la table dans l’une des partitions.</td>
     </tr>
     <tr>
       <td>5. Déclare le SchemaInfo pour la table partitionnée.</td>
     </tr>
   </table>
   <table style="width:100%">
     <tr>
       <th>Fichier PowerShell</th>
       <th>Étapes</th>
     </tr>
   <tr>
       <th rowspan="4">ExecuteSampleSplitMerge.ps1 </th>
       <td>1. Envoie une demande de fractionnement vers le serveur web frontal du service de fractionnement/fusion, qui fractionne la moitié des données de la première partition et les envoie vers la deuxième partition.</td>
     </tr>
     <tr>
       <td>2. Interroge le serveur Web frontal sur l’état de la demande de fractionnement et attend jusqu’à ce que la demande soit terminée.</td>
     </tr>
     <tr>
       <td>3. Envoie une demande de fusion vers le serveur Web frontal du service de fractionnement/fusion, qui déplace les données de la deuxième partition vers la première partition.</td>
     </tr>
     <tr>
       <td>4. Interroge le serveur web frontal sur l’état de la demande de fusion et attend jusqu’à ce que la demande soit terminée.</td>
     </tr>
   </table>

## <a name="use-powershell-to-verify-your-deployment"></a>Utiliser PowerShell pour vérifier le déploiement

1. Ouvrez une nouvelle fenêtre PowerShell et accédez au répertoire dans lequel vous avez téléchargé le package de fractionnement/fusion, puis accédez au répertoire « powershell ».

2. Créez un serveur (ou choisissez un serveur existant) dans lequel le manager des cartes de partitions et les partitions seront créés.

   > [!NOTE]
   > Par défaut, le script *SetupSampleSplitMergeEnvironment.ps1* crée toutes ces bases de données sur le même serveur pour que le script reste simple. Il ne s’agit pas d’une restriction du service de fractionnement/fusion.

   Une connexion d’authentification SQL avec un accès en lecture/écriture aux bases de données est requis pour le service de fractionnement/fusion afin de déplacer les données et de mettre à jour la carte de partitions. Le service de fractionnement/fusion s’exécutant dans le cloud, il ne prend pas actuellement en charge l’authentification intégrée.

   Assurez-vous que le serveur est configuré pour autoriser l’accès à partir de l’adresse IP de l’ordinateur exécutant les scripts. Ce paramètre se trouve sous SQL Server/pare-feux et réseaux virtuels/adresses IP du client.

3. Exécutez le script *SetupSampleSplitMergeEnvironment.ps1* pour créer l’exemple d’environnement.

   L’exécution de ce script efface toutes les structures de données de gestion des cartes de partitions existantes dans la base de données du Gestionnaire des cartes de partitions et dans les partitions. Il peut être utile de réexécuter le script si vous souhaitez réinitialiser la carte de partitions ou les partitions.

   Exemple de ligne de commande :

   ```cmd
   .\SetupSampleSplitMergeEnvironment.ps1
    -UserName 'mysqluser' -Password 'MySqlPassw0rd' -ShardMapManagerServerName 'abcdefghij.database.windows.net'
   ```

4. Exécutez le script Getmappings.ps1 pour afficher les mappages qui existent actuellement dans l’exemple d’environnement.

   ```cmd
   .\GetMappings.ps1
    -UserName 'mysqluser' -Password 'MySqlPassw0rd' -ShardMapManagerServerName 'abcdefghij.database.windows.net'
   ```

5. Exécutez le script *ExecuteSampleSplitMerge.ps1* pour exécuter une opération de fractionnement (la moitié des données sont déplacées de la première partition vers la deuxième), puis une opération de fusion (les données sont redéplacées vers la première partition). Si vous avez configuré TLS et laissé le point de terminaison http désactivé, veillez à utiliser le point de terminaison https:// à la place.

    Exemple de ligne de commande :

    ```cmd
    .\ExecuteSampleSplitMerge.ps1
    -UserName 'mysqluser' -Password 'MySqlPassw0rd'
    -ShardMapManagerServerName 'abcdefghij.database.windows.net'
    -SplitMergeServiceEndpoint 'https://mysplitmergeservice.cloudapp.net'
    -CertificateThumbprint '0123456789abcdef0123456789abcdef01234567'
    ```

    Si vous recevez l’erreur ci-dessous, il s’agit probablement d’un problème avec le certificat de votre point de terminaison web. Essayez de vous connecter au point de terminaison web avec votre navigateur web préféré et vérifiez s’il existe une erreur de certificat.

    `Invoke-WebRequest : The underlying connection was closed: Could not establish trust relationship for the SSL/TLSsecure channel.`

    Si l’opération s’est déroulée sans erreur, le résultat doit ressembler à ce qui suit :

    ```output
    > .\ExecuteSampleSplitMerge.ps1 -UserName 'mysqluser' -Password 'MySqlPassw0rd' -ShardMapManagerServerName 'abcdefghij.database.windows.net' -SplitMergeServiceEndpoint 'http://mysplitmergeservice.cloudapp.net' -CertificateThumbprint 0123456789abcdef0123456789abcdef01234567
    > Sending split request
    > Began split operation with id dc68dfa0-e22b-4823-886a-9bdc903c80f3
    > Polling split-merge request status. Press Ctrl-C to end
    > Progress: 0% | Status: Queued | Details: [Informational] Queued request
    > Progress: 5% | Status: Starting | Details: [Informational] Starting split-merge state machine for request.
    > Progress: 5% | Status: Starting | Details: [Informational] Performing data consistency checks on target     shards.
    > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
    > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Waiting for reference tables copy     completion.
    > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
    > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Moving key range [100:110) of     Sharded tables
    > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Successfully copied key range     [100:110) for table [dbo].[MyShardedTable]
    > ...
    > ...
    > Progress: 90% | Status: Completing | Details: [Informational] Successfully deleted shardlets in table     [dbo].[MyShardedTable].
    > Progress: 90% | Status: Completing | Details: [Informational] Deleting any temp tables that were created     while processing the request.
    > Progress: 100% | Status: Succeeded | Details: [Informational] Successfully processed request.
    > Sending merge request
    > Began merge operation with id 6ffc308f-d006-466b-b24e-857242ec5f66
    > Polling request status. Press Ctrl-C to end
    > Progress: 0% | Status: Queued | Details: [Informational] Queued request
    > Progress: 5% | Status: Starting | Details: [Informational] Starting split-merge state machine for request.
    > Progress: 5% | Status: Starting | Details: [Informational] Performing data consistency checks on target     shards.
    > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
    > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Moving key range [100:110) of     Sharded tables
    > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Successfully copied key range     [100:110) for table [dbo].[MyShardedTable]
    > ...
    > ...
    > Progress: 90% | Status: Completing | Details: [Informational] Successfully deleted shardlets in table     [dbo].[MyShardedTable].
    > Progress: 90% | Status: Completing | Details: [Informational] Deleting any temp tables that were created     while processing the request.
    > Progress: 100% | Status: Succeeded | Details: [Informational] Successfully processed request.
    >
    ```

6. Faites des essais avec d’autres types de données. Tous ces scripts nécessitent un paramètre -ShardKeyType facultatif qui vous permet de spécifier le type de clé. La valeur par défaut est Int32, mais vous pouvez également spécifier Int64, Guid ou Binary.

## <a name="create-requests"></a>Créer des requêtes

Le service peut être utilisé à l’aide de l’interface utilisateur Web ou par l’importation et l’utilisation du module PowerShell SplitMerge.psm1 qui envoie vos demandes via le rôle Web.

Ce service peut déplacer les données dans les tables partitionnées et les tables de référence. Une table partitionnée possède une colonne de clés de partitionnement et comporte des données de ligne différentes sur chaque partition. Une table de référence n’est pas partitionnée. De ce fait, elle comporte les mêmes données de ligne sur chaque partition. Les tables de référence sont utiles pour les données qui ne changent pas souvent et qui sont utilisées pour S’ASSOCIER à des tables partitionnées dans les requêtes.

Pour effectuer une opération de fractionnement/fusion, vous devez déclarer les tables partitionnées et les tables de référence que vous souhaitez déplacer. Cette opération est effectuée avec l’API **SchemaInfo** . Cette API se trouve dans l’espace de noms **Microsoft.Azure.SqlDatabase.ElasticScale.ShardManagement.Schema** .

1. Pour chaque table partitionnée, créez un objet **ShardedTableInfo** décrivant le nom du schéma parent de la table (facultatif, la valeur par défaut est « dbo »), le nom de la table et le nom de la colonne de cette table qui comporte la clé de partitionnement.
2. Pour chaque table de référence, créez un objet **ReferenceTableInfo** décrivant le nom du schéma parent de la table (facultatif, la valeur par défaut est « dbo ») et le nom de la table.
3. Ajoutez les objets TableInfo ci-dessus à un nouvel objet **SchemaInfo** .
4. Obtenez une référence à un objet **ShardMapManager** et appelez **GetSchemaInfoCollection** .
5. Ajoutez **SchemaInfo** à **SchemaInfoCollection** , en fournissant le nom du mappage de partitions.

Le script SetupSampleSplitMergeEnvironment.ps1 contient un exemple de cette opération.

Le service de fractionnement/fusion ne crée pas la base de données cible (ou le schéma pour les tables de la base de données) à votre place. Ils doivent être créés au préalable avant d’envoyer une demande au service.

## <a name="troubleshooting"></a>Dépannage

Le message ci-dessous peut apparaître lors de l’exécution des exemples de scripts PowerShell :

   `Invoke-WebRequest : The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel.`

Cette erreur signifie que votre certificat TLS/SSL n’est pas correctement configuré. Suivez les instructions de la section « Connexion avec un navigateur web ».

Si vous ne pouvez pas soumettre les demandes, vous pouvez voir ceci :

   `[Exception] System.Data.SqlClient.SqlException (0x80131904): Could not find stored procedure 'dbo.InsertRequest'.`

Dans ce cas, vérifiez votre fichier de configuration, notamment le paramètre pour **WorkerRoleSynchronizationStorageAccountConnectionString** . Cette erreur indique généralement que le rôle de travail n’a pas pu initialiser avec succès la base de données de métadonnées à la première utilisation.

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/allowed-services.png
[2]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/manage.png
[3]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/staging.png
[4]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/upload.png
[5]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/storage.png