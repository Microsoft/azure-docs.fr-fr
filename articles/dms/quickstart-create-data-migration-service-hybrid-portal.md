---
title: 'Démarrage rapide : Créer une instance en mode hybride à l’aide du portail Azure'
titleSuffix: Azure Database Migration Service
description: Utilisez le portail Azure pour créer une instance d’Azure Database Migration Service en mode hybride.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: quickstart
ms.date: 03/13/2020
ms.openlocfilehash: 39f893e9375970ff4bb6e3cfa2c93ceea48ec896
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92331802"
---
# <a name="quickstart-create-a-hybrid-mode-instance-with-azure-portal--azure-database-migration-service"></a>Démarrage rapide : Créer une instance en mode hybride à l’aide du portail Azure et d’Azure Database Migration Service

Le mode hybride d’Azure Database Migration Service gère les migrations de base de données à l’aide d’un Worker de migration hébergé localement avec une instance d’Azure Database Migration Service exécutée dans le cloud. Le mode hybride est particulièrement utile pour les scénarios dans lesquels la connectivité de site à site entre le réseau local et Azure fait défaut ou si la bande passante de la connectivité de site à site est limitée.

>[!NOTE]
>Actuellement, Azure Database Migration Service exécuté en mode hybride prend en charge les migrations SQL Server vers :
>
>- Une instance managée d’Azure SQL avec un temps d’arrêt quasi nul (en ligne).
>- Une base de données unique Azure SQL Database avec un temps d’arrêt (hors connexion).
>- MongoDb vers Azure CosmosDB avec un temps d’arrêt quasi nul (en ligne).
>- MongoDb vers Azure CosmosDB avec un temps d’arrêt (hors connexion).

Dans ce guide de démarrage rapide, vous allez utiliser le portail Azure pour créer une instance d’Azure Database Migration Service en mode hybride. Vous allez ensuite télécharger, installer et configurer le Worker hybride dans votre réseau local. Pendant la préversion, vous pouvez utiliser Azure Database Migration Service en mode hybride pour migrer les données d’une instance SQL Server locale vers Azure SQL Database.

> [!NOTE]
> Le programme d’installation hybride d’Azure Database Migration Service est exécutable sur Microsoft Windows Server 2012 R2, Windows Server 2016, Windows Server 2019 et Windows 10.

> [!IMPORTANT]
> Le programme d’installation hybride Azure Database Migration Service nécessite .NET 4.7.2 ou ultérieur. Pour trouver les versions les plus récentes de .NET, consultez la page [Télécharger .NET Framework](https://dotnet.microsoft.com/download/dotnet-framework).

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Ouvrez votre navigateur web, accédez au [portail Microsoft Azure](https://portal.azure.com/), puis saisissez vos informations de connexion au portail.

Il s’ouvre par défaut sur le tableau de bord des services.

## <a name="register-the-resource-provider"></a>Inscrire le fournisseur de ressources

Avant de créer votre première instance d’Azure Database Migration Service, vous devez inscrire le fournisseur de ressources Microsoft.DataMigration.

1. Dans le portail Azure, sélectionnez **Abonnements** , puis l’abonnement dans lequel vous souhaitez créer l’instance Azure Database Migration Service, puis sélectionnez **Fournisseurs de ressources** .

    ![Rechercher un fournisseur de ressources](media/quickstart-create-data-migration-service-hybrid-portal/dms-portal-search-resource-provider.png)

2. Recherchez migration, puis à droite de **Microsoft.DataMigration** , sélectionnez **Inscrire** .

    ![S’inscrire auprès du fournisseur de ressources](media/quickstart-create-data-migration-service-hybrid-portal/dms-portal-register-resource-provider.png)

## <a name="create-an-instance-of-the-service"></a>Créer une instance du service

1. Sélectionnez + **Créer une ressource** pour créer une instance d’Azure Database Migration Service.

2. Recherchez « migration » sur la Place de marché, sélectionnez **Azure Database Migration Service** et, dans l’écran **Azure Database Migration Service** , sélectionnez **Créer** .

3. Sur l’écran **Créer un service de migration** :

    - Choisissez un **Nom de service** unique et facile à mémoriser pour identifier votre instance d’Azure Database Migration Service.
    - Sélectionnez l’ **abonnement** Azure dans lequel vous souhaitez créer l’instance.
    - Sélectionnez un **Groupe de ressources** existant ou créez-en un.
    - Choisissez **l’emplacement** le plus proche de votre serveur source ou cible.
    - Pour **Mode de service** , sélectionnez **Hybride (préversion)** .

         ![Créer un service de migration - Notions de base](media/quickstart-create-data-migration-service-hybrid-portal/dms-create-service-basics.png)

4. Sélectionnez **Revoir + créer** .

5. Sous l’onglet **Vérifier + créer** , passez en revue les conditions, vérifiez les autres informations fournies, puis sélectionnez **Créer** .

    ![Créer un service de migration - Vérifier + créer](media/quickstart-create-data-migration-service-hybrid-portal/dms-create-service-review-and-create.png)

    Votre instance d’Azure Database Migration Service en mode hybride est créée et prête pour la configuration au bout de quelques instants. L’instance d’Azure Database Migration Service s’affiche alors comme indiqué dans l’image suivante :

    ![Instance Azure Database Migration Service en mode hybride](media/quickstart-create-data-migration-service-hybrid-portal/dms-instance-hybrid-mode.png)

6. Une fois le service créé, sélectionnez **Propriétés** , puis copiez la valeur affichée dans la zone **ID de ressource** , que vous utiliserez pour installer le Worker hybride Azure Database Migration Service.

    ![Propriétés du mode hybride d’Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-copy-resource-id.png)

## <a name="create-azure-app-registration-id"></a>Créer un ID d’inscription Azure App

Vous devez créer un ID d’inscription Azure App qui permettra au Worker hybride local de communiquer avec Azure Database Migration Service dans le cloud.

1. Dans le portail Azure, sélectionnez **Azure Active Directory** , **Inscriptions d’applications** , puis **Nouvelle inscription** .
2. Nommez l’application, puis, sous **Types de comptes pris en charge** , sélectionnez le type de compte à prendre en charge pour indiquer qui peut utiliser l’application.

    ![Inscription d’application dans le mode hybride d’Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-register-application.png)

3. Utilisez les valeurs par défaut pour le champ **URI de redirection (facultatif)** , puis sélectionnez **Inscrire** .

4. Une fois l’inscription de l’ID d’application terminée, notez l’ **ID d’application (client)** , que vous utiliserez lors de l’installation du Worker hybride.

5. Dans le portail Azure, accédez à Azure Database Migration Service, sélectionnez **Contrôle d’accès (IAM)** , puis **Ajouter une attribution de rôle** pour attribuer un accès de contributeur à l’ID d’application.

    ![Affectation du rôle de contributeur dans le mode hybride d’Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-app-assign-contributor.png)

6. Sélectionnez le rôle de **Contributeur** , attribuez l’accès à **utilisateur Azure AD ou principal de service** , puis sélectionnez le nom de l’ID d’application.

    ![Détails de l’attribution du rôle de contributeur dans le mode hybride d’Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-add-role-assignment.png)

7. Sélectionnez **Enregistrer** pour enregistrer l’attribution de rôle de l’ID d’application sur la ressource Azure Database Migration Service.

## <a name="download-and-install-the-hybrid-worker"></a>Télécharger et installer le Worker hybride

1. Dans le portail Azure, accédez à votre instance d’Azure Database Migration Service.

2. Sous **Paramètres** , sélectionnez **Hybride** , puis **Téléchargement du programme d’installation** pour télécharger le Worker hybride.

    ![Téléchargement du Worker hybride Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-installer-download.png)

3. Extrayez le fichier zip sur le serveur qui hébergera le Worker hybride Azure Database Migration Service.

    > [!IMPORTANT]
    > Le programme d’installation hybride Azure Database Migration Service nécessite .NET 4.7.2 ou ultérieur. Pour trouver les versions les plus récentes de .NET, consultez la page [Télécharger .NET Framework](https://dotnet.microsoft.com/download/dotnet-framework).

4. Dans le dossier d’installation, recherchez et ouvrez le fichier **dmsSettings.json** , indiquez les valeurs **ApplicationId** et **resourceId** , puis enregistrez le fichier.

    ![Paramètres du Worker hybride Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-settings.png)

5. Générez un certificat qui permettra à Azure Database Migration Service d’authentifier la communication à partir du Worker hybride à l’aide de la commande suivante.

    ```
    <drive>:\<folder>\Install>DMSWorkerBootstrap.exe -a GenerateCert
    ```

    Un certificat est généré dans le dossier d’installation.

    ![Certificat du Worker hybride Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-certificate.png)

6. Dans le portail Azure, accédez à l’ID d’application. Sous **Gérer** , sélectionnez **Certificats et secrets** , puis **Charger un certificat** pour sélectionner le certificat public que vous avez généré.

    ![Chargement du certificat du Worker hybride Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-app-upload-certificate.png)

7. Installez le Worker hybride Azure Database Migration Service sur votre serveur local en exécutant la commande suivante :

    ```
    <drive>:\<folder>\Install>DMSWorkerBootstrap.exe -a Install -IAcceptDMSLicenseTerms -d
    ```

    > [!NOTE]
    > Vous pouvez également exécuter la commande install en spécifiant les paramètres suivants :
    >
    > - **-TelemetryOptOut**  : empêche le Worker d’envoyer des données de télémétrie, mais continue d’effectuer une journalisation minimale au niveau local.  Le programme d’installation envoie toujours des données de télémétrie.
    > - **-p {InstallLocation}** . Autorise le changement du chemin d’installation, qui est par défaut « C:\Program Files\DatabaseMigrationServiceHybrid ».

8. Si le programme d’installation s’exécute sans erreur, le service présente l’état en ligne dans Azure Database Migration Service et vous pouvez commencer à migrer vos bases de données.

    ![État en ligne d’Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-instance-hybrid-mode-online.png)

## <a name="uninstall-azure-database-migration-service-hybrid-mode"></a>Désinstaller Azure Database Migration Service en mode hybride

Actuellement, la désinstallation d’Azure Database Migration Service en mode hybride est prise en charge uniquement via le programme d’installation du Worker hybride Azure Database Migration Service sur votre serveur local, à l’aide de la commande suivante :

```
<drive>:\<folder>\Install>DMSWorkerBootstrap.exe -a uninstall
```

> [!NOTE]
> Vous pouvez également exécuter la commande uninstall en spécifiant le paramètre « -ReuseCert » pour conserver le certificat AdApp généré par le workflow generateCert.  Cela vous permet de réutiliser le certificat qui a été précédemment généré et chargé.

## <a name="set-up-the-azure-database-migration-service-hybrid-worker-using-powershell"></a>Installer le Worker hybride Azure Database Migration Service à l’aide de PowerShell

Vous pouvez installer le Worker hybride Azure Database Migration Service par le biais du portail Azure, mais aussi l’installer au moyen d’un [script PowerShell](https://techcommunity.microsoft.com/gxcuf89792/attachments/gxcuf89792/MicrosoftDataMigration/119/1/DMS_Hybrid_Script.zip) que nous fournissons. Avec ce script, vous automatisez les étapes d’installation du Worker après avoir créé une instance d’Azure Database Migration Service en mode hybride. Le script :

1. Crée une AdApp.
2. Télécharge le programme d’installation.
3. Exécute le workflow generateCert.
4. Charge le certificat.
5. Ajoute l’AdApp comme contributeur à votre instance d’Azure Database Migration Service.
6. Exécute le workflow d’installation.

Ce script est conçu pour un prototypage rapide lorsque l’utilisateur possède déjà toutes les autorisations nécessaires dans l’environnement. Notez que dans votre environnement de production, l’AdApp et le certificat peuvent avoir des exigences différentes, et faire échouer le script.

> [!IMPORTANT]
> Ce script suppose qu’il existe déjà une instance d’Azure Database Migration Service en mode hybride et que le compte Azure utilisé a les autorisations appropriées pour créer des AdApps dans le locataire et modifier les règles Azure RBAC sur l’abonnement.

Spécifiez les paramètres en haut du script, puis exécutez le script à partir d’une instance de PowerShell en tant qu’administrateur.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Migrer SQL Server vers une instance managée d’Azure SQL en ligne](tutorial-sql-server-managed-instance-online.md)
> [Migrer SQL Server vers Azure SQL Database hors connexion](tutorial-sql-server-to-azure-sql.md)
