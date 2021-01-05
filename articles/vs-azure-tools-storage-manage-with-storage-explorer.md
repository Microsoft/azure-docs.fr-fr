---
title: Prise en main Prise en main de l’Explorateur de stockage | Microsoft Docs
description: Commencez à gérer les ressources de stockage Azure avec Explorateur Stockage. Téléchargez et installez Explorateur Stockage Azure, connectez-vous à un compte de stockage ou à un service, et bien plus encore.
services: storage
author: cawaMS
ms.service: storage
ms.devlang: multiple
ms.topic: article
ms.date: 11/08/2019
ms.author: cawa
ms.openlocfilehash: be9b2d9a31d4affc9615f5d2f4b2585b7533a0f6
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95545905"
---
# <a name="get-started-with-storage-explorer"></a>Prise en main de l’Explorateur Stockage

## <a name="overview"></a>Vue d’ensemble

L’Explorateur Stockage Microsoft Azure est une application autonome qui vous permet d’utiliser facilement les données Stockage Azure sur Windows, macOS et Linux. Dans cet article, vous allez découvrir différentes façons de vous connecter à vos comptes de stockage Azure et de les gérer.

![Explorateur Stockage Microsoft Azure][0]

## <a name="prerequisites"></a>Prérequis

# <a name="windows"></a>[Windows](#tab/windows)

L’Explorateur Stockage est pris en charge sur les versions suivantes de Windows :

* Windows 10 (recommandé)
* Windows 8
* Windows 7

Pour toutes les versions de Windows, l’Explorateur Stockage nécessite au minimum .NET Framework 4.7.2.

# <a name="macos"></a>[macOS](#tab/macos)

L’Explorateur Stockage est pris en charge sur les versions suivantes de macOS :

* macOS 10.12 Sierra et versions ultérieures

# <a name="linux"></a>[Linux](#tab/linux)

L’Explorateur Stockage est disponible dans le [Snap Store](https://snapcraft.io/storage-explorer) pour les distributions de Linux les plus courantes. Il s’agit de la méthode d’installation recommandée. Le snap Explorateur Stockage installe toutes ses dépendances et mises à jour quand de nouvelles versions sont publiées sur le Snap Store.

Pour obtenir la liste des distributions prises en charge, consultez la [page d’installation de snapd](https://snapcraft.io/docs/installing-snapd).

Vous devez utiliser un gestionnaire de mots de passe avec l’Explorateur Stockage. Vous devrez peut-être vous connecter manuellement à un gestionnaire de mots de passe. Vous pouvez connecter l’Explorateur Stockage au gestionnaire de mots de passe de votre système en exécutant la commande suivante :

```bash
snap connect storage-explorer:password-manager-service :password-manager-service
```

Vous pouvez aussi obtenir l’Explorateur Stockage en téléchargeant un fichier au format *.tar.gz*. Vous devez installer les dépendances manuellement. L’installation à l’aide du fichier *.tar.gz* est prise en charge sur les distributions suivantes de Linux :

* Ubuntu 20.04 x64
* Ubuntu 18.04 x64
* Ubuntu 16.04 x64

L’installation à l’aide du fichier *.tar.gz* peut fonctionner sur d’autres distributions, mais seules celles listées sont officiellement prises en charge.

Pour plus d’informations sur l’installation de l’Explorateur Stockage sur Linux, consultez la section [Dépendances Linux](./storage/common/storage-explorer-troubleshooting.md#linux-dependencies) dans le guide de résolution des problèmes de l’Explorateur Stockage Azure.

---

## <a name="download-and-install"></a>Télécharger et installer

Pour télécharger et installer l’Explorateur Stockage, consultez [Explorateur Stockage Azure](https://www.storageexplorer.com).

## <a name="connect-to-a-storage-account-or-service"></a>Connexion à un service ou un compte de stockage

L’Explorateur Stockage offre de nombreuses façons de se connecter à des comptes de stockage. En règle générale, vous pouvez :

* [Vous connecter à Azure pour accéder à vos abonnements et à leurs ressources](#sign-in-to-azure)
* [Lier une ressource particulière CosmosDB ou de stockage](#attach-a-specific-resource)

### <a name="sign-in-to-azure"></a>Connexion à Azure

> [!NOTE]
> Pour accéder pleinement aux ressources après la connexion, l’Explorateur Stockage nécessite des autorisations de gestion (Azure Resource Manager) et de couche de données. Cela signifie que vous devez disposer d’autorisations Azure Active Directory (Azure AD) qui vous donnent accès à votre compte de stockage, aux conteneurs du compte et aux données dans les conteneurs. Si vous disposez uniquement d’autorisations au niveau de la couche de données, envisagez d’[ajouter une ressource par le biais d’Azure AD](#add-a-resource-via-azure-ad). Pour plus d’informations sur les autorisations spécifiques dont l’Explorateur Stockage a besoin, consultez le [guide de résolution des problèmes de l’Explorateur Stockage Azure](./storage/common/storage-explorer-troubleshooting.md#azure-rbac-permissions-issues).

1. Dans l’Explorateur Stockage, sélectionnez **Afficher** > **Gestion des comptes** ou sélectionnez le bouton **Gérer les comptes**.

    ![Gérer les comptes][1]

1. Le volet **GESTION DES COMPTES** affiche maintenant tous les comptes Azure auxquels vous vous êtes connecté. Pour vous connecter à un autre compte, sélectionnez **Ajouter un compte**.

1. Dans **Se connecter au stockage Azure**, sélectionnez un Cloud Azure dans l’**environnement Azure** pour vous connecter à un Cloud national ou à Azure Stack. Après avoir choisi votre environnement, sélectionnez **Suivant**.

    ![Option de connexion][2]

    L’Explorateur Stockage ouvre une page de connexion. Pour plus d’informations, consultez [Connecter l’Explorateur Stockage à un compte de stockage ou abonnement Azure Stack](/azure-stack/user/azure-stack-storage-connect-se).

1. Une fois connecté avec un compte Azure, le compte et les abonnements Azure associés à ce compte s’affichent sous **GESTION DES COMPTES**. La case à cocher **Tous les abonnements** permet de sélectionner ou de désélectionner l’ensemble des abonnements Azure listés. Sélectionnez les abonnements Azure que vous souhaitez utiliser, puis sélectionnez **Appliquer**.

    ![Sélectionner les abonnements Azure][3]

    Le volet **EXPLORATEUR** affiche les comptes de stockage associés aux abonnements Azure sélectionnés.

    ![Abonnements Azure sélectionnés][4]

### <a name="attach-a-specific-resource"></a>Lier une ressource spécifique

Vous pouvez effectuer l’attachement à une ressource dans l’Explorateur Stockage de plusieurs façons :

* [Ajouter une ressource via Azure AD](#add-a-resource-via-azure-ad). Si vous disposez uniquement d’autorisations au niveau de la couche de données, utilisez cette option pour ajouter un conteneur d’objets blob ou un conteneur de stockage d’objets blob Azure Data Lake Storage Gen2.
* [Utiliser une chaîne de connexion](#use-a-connection-string). Utilisez cette option si vous disposez d’une chaîne de connexion à un compte de stockage. L’Explorateur Stockage prend en charge les chaînes de connexion de clé et de [signature d’accès partagé](./storage/common/storage-sas-overview.md).
* [Utiliser un URI de signature d’accès partagé](#use-a-shared-access-signature-uri). Si vous disposez d’un [URI de signature d’accès partagé](./storage/common/storage-sas-overview.md) à un conteneur d’objets blob, un partage de fichiers, une file d’attente ou une table, utilisez-le pour effectuer l’attachement à la ressource. Pour obtenir un URI de signature d’accès partagé, vous pouvez utiliser l’[Explorateur Stockage](#generate-a-sas-in-storage-explorer) ou le [portail Azure](https://portal.azure.com).
* [Utiliser un nom et une clé](#use-a-name-and-key). Si vous connaissez l’une ou l’autre des clés de compte de votre compte de stockage, vous pouvez utiliser cette option pour vous connecter rapidement. Recherchez vos clés dans la page du compte de stockage en sélectionnant **Paramètres** > **Clés d’accès** dans le [portail Azure](https://portal.azure.com).
* [Attacher à un émulateur local](#attach-to-a-local-emulator) Si vous utilisez un des émulateurs de stockage Azure disponibles, servez-vous de cette option pour vous connecter facilement à votre émulateur.
* [Se connecter à un compte Azure Cosmos DB à l’aide d’une chaîne de connexion](#connect-to-an-azure-cosmos-db-account-by-using-a-connection-string). Utilisez cette option si vous disposez d’une chaîne de connexion à une instance CosmosDB.
* [Se connecter à Azure Data Lake Store par URI](#connect-to-azure-data-lake-store-by-uri). Utilisez cette option si vous disposez d’un URI à Azure Data Lake Store.

#### <a name="add-a-resource-via-azure-ad"></a>Ajouter une ressource via Azure AD

1. Sélectionnez le symbole de **connexion** pour ouvrir **Se connecter au stockage Azure**.

    ![Option Se connecter à Azure Storage][9]

1. Si ce n’est pas déjà fait, utilisez l’option **Ajouter un compte Azure** pour vous connecter au compte Azure ayant accès à la ressource. Une fois connecté, revenez à **Se connecter au stockage Azure**.

1. Sélectionnez **Ajouter une ressource via Azure Active Directory (Azure AD)** , puis **Suivant**.

1. Sélectionnez un locataire et un compte Azure. Ces valeurs doivent avoir accès à la ressource de stockage que vous souhaitez lier. Sélectionnez **Suivant**.

1. Choisissez le type de ressource que vous voulez attacher. Entrez les informations nécessaires pour se connecter. 

   Les informations que vous entrez dans cette page dépendent du type de ressource que vous ajoutez. Veillez à choisir le type de ressource correct. Une fois que vous avez entré toutes les informations obligatoires, sélectionnez **Suivant**.

1. Passez en revue le **récapitulatif des informations de connexion** et vérifiez que tout est correct. Si c’est le cas, sélectionnez **Se connecter**. Sinon, utilisez le bouton **Précédent** pour revenir aux pages précédentes et corriger les informations incorrectes.

Une fois la connexion ajoutée, l’arborescence des ressources accède au nœud représentant la connexion. La ressource apparaît sous **Local et attaché** > **Comptes de stockage** >  **(Conteneurs attachés)**  > **Conteneurs d’objets blob**. Si l’Explorateur Stockage n’a pas pu ajouter votre connexion ou si vous ne pouvez pas accéder à vos données après avoir correctement ajouté la connexion, consultez le [guide de résolution des problèmes de l’Explorateur Stockage Azure](./storage/common/storage-explorer-troubleshooting.md).

#### <a name="use-a-connection-string"></a>Utiliser une chaîne de connexion

1. Sélectionnez le symbole de **connexion** pour ouvrir **Se connecter au stockage Azure**.

    ![Option Se connecter à Azure Storage][9]

1. Sélectionnez **Utiliser une chaîne de connexion**, puis **Suivant**.

1. Choisissez un nom d’affichage pour votre connexion et entrez votre chaîne de connexion. Ensuite, sélectionnez **Suivant**.

1. Passez en revue le **récapitulatif des informations de connexion** et vérifiez que tout est correct. Si c’est le cas, sélectionnez **Se connecter**. Sinon, utilisez le bouton **Précédent** pour revenir aux pages précédentes et corriger les informations incorrectes.

Une fois la connexion ajoutée, l’arborescence des ressources accède au nœud représentant la connexion. La ressource apparaît sous **Local et attaché** > **Comptes de stockage**. Si l’Explorateur Stockage n’a pas pu ajouter votre connexion ou si vous ne pouvez pas accéder à vos données après avoir correctement ajouté la connexion, consultez le [guide de résolution des problèmes de l’Explorateur Stockage Azure](./storage/common/storage-explorer-troubleshooting.md).

#### <a name="use-a-shared-access-signature-uri"></a>Utiliser un URI de signature d’accès partagé

1. Sélectionnez le symbole de **connexion** pour ouvrir **Se connecter au stockage Azure**.

    ![Option Se connecter à Azure Storage][9]

1. Sélectionnez **Utiliser un URI SAP (signature d’accès partagé)** , puis sélectionnez **Suivant**.

1. Choisissez un nom d’affichage pour votre connexion et entrez votre URI de signature d’accès partagé. Le point de terminaison de service pour le type de ressource que vous attachez doit se renseigner automatiquement. Si vous utilisez un point de terminaison personnalisé, il est possible que ce ne soit pas le cas. Sélectionnez **Suivant**.

1. Passez en revue le **récapitulatif des informations de connexion** et vérifiez que tout est correct. Si c’est le cas, sélectionnez **Se connecter**. Sinon, utilisez le bouton **Précédent** pour revenir aux pages précédentes et corriger les informations incorrectes.

Une fois la connexion ajoutée, l’arborescence des ressources accède au nœud représentant la connexion. La ressource apparaît sous **Local et attaché** > **Comptes de stockage** >  **(Conteneurs attachés)**  > *nœud de service pour le type de conteneur que vous avez attaché*. Si l’Explorateur Stockage n’a pas pu ajouter votre connexion, consultez le [guide de résolution des problèmes de l’Explorateur Stockage Azure](./storage/common/storage-explorer-troubleshooting.md). Consultez le guide de résolution des problèmes si vous ne pouvez pas accéder à vos données après avoir correctement ajouté la connexion.

#### <a name="use-a-name-and-key"></a>Utiliser un nom et une clé

1. Sélectionnez le symbole de **connexion** pour ouvrir **Se connecter au stockage Azure**.

    ![Option Se connecter à Azure Storage][9]

1. Sélectionnez **Utiliser le nom et la clé d’un compte de stockage**, puis **Suivant**.

1. Choisissez un nom d’affichage pour votre connexion.

1. Entrez le nom du compte de stockage et une de ses clés d’accès.

1. Choisissez le **Domaine de stockage** à utiliser, puis sélectionnez **Suivant**.

1. Passez en revue le **récapitulatif des informations de connexion** et vérifiez que tout est correct. Si c’est le cas, sélectionnez **Se connecter**. Sinon, utilisez le bouton **Précédent** pour revenir aux pages précédentes et corriger les informations incorrectes.

Une fois la connexion ajoutée, l’arborescence des ressources accède au nœud représentant la connexion. La ressource apparaît sous **Local et attaché** > **Comptes de stockage**. Si l’Explorateur Stockage n’a pas pu ajouter votre connexion ou si vous ne pouvez pas accéder à vos données après avoir correctement ajouté la connexion, consultez le [guide de résolution des problèmes de l’Explorateur Stockage Azure](./storage/common/storage-explorer-troubleshooting.md).

#### <a name="attach-to-a-local-emulator"></a>Lier à un émulateur local

L’Explorateur Stockage prend actuellement en charge deux émulateurs de stockage officiels :

* [Émulateur de stockage Azure](storage/common/storage-use-emulator.md) (Windows uniquement)
* [Azurite](https://github.com/azure/azurite) (Windows, macOS ou Linux)

Si votre émulateur est à l’écoute sur les ports par défaut, vous pouvez utiliser le nœud **Émulateur - Ports par défaut** pour accéder rapidement à votre émulateur. Recherchez **Émulateur - Ports par défaut** sous **Local et attaché** > **Comptes de stockage**.

Si vous souhaitez utiliser un autre nom pour votre connexion, ou si votre émulateur ne s’exécute pas sur les ports par défaut, effectuez les étapes suivantes :

1. Démarrez votre émulateur. Entrez la commande `AzureStorageEmulator.exe status` pour afficher les ports de chaque type de service.

   > [!IMPORTANT]
   > L’Explorateur Stockage ne démarre pas automatiquement votre émulateur. Vous devez le démarrer manuellement.

1. Sélectionnez le symbole de **connexion** pour ouvrir **Se connecter au stockage Azure**.

    ![Option Se connecter à Azure Storage][9]

1. Sélectionnez **Attacher à un émulateur local**, puis sélectionnez **Suivant**.

1. Choisissez un nom d’affichage pour votre connexion et indiquez les ports sur lesquels votre émulateur est à l’écoute pour chaque type de service. **Attacher à un émulateur local** suggère les valeurs de port par défaut pour la plupart des émulateurs. **Port des fichiers** est laissé vide, car aucun des émulateurs officiels ne prend actuellement en charge le service Files. Si l’émulateur que vous utilisez prend en charge Files, vous pouvez entrer le port à utiliser. Ensuite, sélectionnez **Suivant**.

1. Passez en revue le **récapitulatif des informations de connexion** et vérifiez que tout est correct. Si c’est le cas, sélectionnez **Se connecter**. Sinon, utilisez le bouton **Précédent** pour revenir aux pages précédentes et corriger les informations incorrectes.

Une fois la connexion ajoutée, l’arborescence des ressources accède au nœud représentant la connexion. Le nœud devrait apparaître sous **Local et attaché** > **Comptes de stockage**. Si l’Explorateur Stockage n’a pas pu ajouter votre connexion ou si vous ne pouvez pas accéder à vos données après avoir correctement ajouté la connexion, consultez le [guide de résolution des problèmes de l’Explorateur Stockage Azure](./storage/common/storage-explorer-troubleshooting.md).

#### <a name="connect-to-an-azure-cosmos-db-account-by-using-a-connection-string"></a>Connexion à un compte Azure Cosmos DB à l’aide d’une chaîne de connexion

Au lieu de gérer des comptes Azure Cosmos DB par le biais d’un abonnement Azure, vous pouvez vous connecter à Azure Cosmos DB en utilisant une chaîne de connexion. Pour vous connecter, procédez comme suit :

1. Sous **EXPLORATEUR**, développez **Local et attaché**, cliquez avec le bouton droit sur **Comptes Azure Cosmos DB** et sélectionnez **Se connecter à Azure Cosmos DB**.

    ![Se connecter à Azure Cosmos DB avec une chaîne de connexion][21]

1. Sélectionnez l’API Azure Cosmos DB, entrez vos données **Chaîne de connexion**, puis sélectionnez **OK** pour vous connecter au compte Azure Cosmos DB. Pour plus d’informations sur la récupération de la chaîne de connexion, consultez [Gérer un compte Azure Cosmos](./cosmos-db/how-to-manage-database-account.md).

    ![Chaîne de connexion][22]

#### <a name="connect-to-azure-data-lake-store-by-uri"></a>Se connecter à Azure Data Lake Store par URI

Vous pouvez accéder à une ressource qui n’est pas située dans votre abonnement. Pour cela, une personne qui a accès à cette ressource doit vous fournir l’URI associée. Une fois connecté, connectez-vous à Data Lake Store à l’aide de l’URI. Pour vous connecter, procédez comme suit :

1. Sous **EXPLORATEUR**, développez **Local et attaché**.

1. Cliquez avec le bouton droit sur **Data Lake Storage Gen1**, puis sélectionnez **Se connecter à Data Lake Storage Gen1**.

    ![Menu contextuel Se connecter à Data Lake Store](./media/vs-azure-tools-storage-manage-with-storage-explorer/storage-explorer-connect-data-lake-storage.png)

1. Entrez l’URI, puis sélectionnez **OK**. Votre Data Lake Store s’affiche sous **Data Lake Storage**.

    ![Résultat de la connexion à Data Lake Store](./media/vs-azure-tools-storage-manage-with-storage-explorer/storage-explorer-attach-data-lake-finished.png)

Cet exemple utilise Data Lake Storage Gen1. Azure Data Lake Storage Gen2 est désormais disponible. Pour plus d’informations, consultez [Qu’est-ce qu’Azure Data Lake Storage Gen1](./data-lake-store/data-lake-store-overview.md).

## <a name="generate-a-shared-access-signature-in-storage-explorer"></a>Générer une signature d’accès partagé dans l’Explorateur Stockage<a name="generate-a-sas-in-storage-explorer"></a>

### <a name="account-level-shared-access-signature"></a>Signature d’accès partagé au niveau du compte

1. Cliquez avec le bouton droit sur le compte de stockage que vous souhaitez partager, puis sélectionnez **Obtenir une signature d’accès partagé**.

    ![Option de menu contextuel Obtenir une signature d’accès partagé][14]

1. Dans **Signature d’accès partagé**, spécifiez la période et les autorisations souhaitées pour le compte, puis sélectionnez **Créer**.

    ![Obtenir une signature d’accès partagé][15]

1. Copiez la **Chaîne de connexion** ou la **Chaîne de requête** brute dans votre presse-papiers.

### <a name="service-level-shared-access-signature"></a>Signature d’accès partagé au niveau du service

Vous pouvez obtenir une signature d’accès partagé au niveau du service. Pour plus d’informations, consultez [Obtenir la signature d’accès partagé pour un conteneur d’objets blob](vs-azure-tools-storage-explorer-blobs.md#get-the-sas-for-a-blob-container).

## <a name="search-for-storage-accounts"></a>Recherche de comptes de stockage

Vous pouvez utiliser le volet **EXPLORATEUR** pour rechercher une ressource de stockage.

À mesure que vous entrez du texte dans la zone de recherche, l’Explorateur Stockage affiche toutes les ressources correspondant à la valeur en cours de saisie. Cet exemple montre une recherche de **points de terminaison** :

![Recherche de compte de stockage][23]

> [!NOTE]
> Pour accélérer votre recherche, utilisez **Gestion des comptes** afin de désélectionner tous les abonnements qui ne contiennent pas l’élément que vous recherchez. Vous pouvez également cliquer avec le bouton droit sur un nœud et sélectionner **Rechercher à partir d’ici** pour démarrer la recherche à partir d’un nœud spécifique.
>
>

## <a name="next-steps"></a>Étapes suivantes

* [Gérer les ressources de Stockage Blob Azure avec l’Explorateur Stockage](vs-azure-tools-storage-explorer-blobs.md)
* [Utiliser des données à l’aide de l’Explorateur Stockage Azure](./cosmos-db/storage-explorer.md)
* [Gérer les ressources Azure Data Lake Store avec l’Explorateur Stockage](./data-lake-store/data-lake-store-in-storage-explorer.md)

[0]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/Overview.png
[1]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ManageAccounts.png
[2]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/connect-to-azure-storage-azure-environment.png
[3]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/account-panel-subscriptions-apply.png
[4]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/SubscriptionNode.png
[5]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog.png
[7]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/PortalAccessKeys.png
[8]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/AccessKeys.png
[9]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog.png
[10]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog-AddWithKeySelected.png
[11]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog-NameAndKeyPage.png
[12]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/AttachedWithKeyAccount.png
[13]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/AttachedWithKeyAccount-Detach.png
[14]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/get-shared-access-signature-for-storage-explorer.png
[15]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/create-shared-access-signature-for-storage-explorer.png
[16]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog-WithConnStringOrSASSelected.png
[17]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog-ConnStringOrSASPage-1.png
[18]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/AttachedWithSASAccount.png
[19]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ConnectDialog-ConnStringOrSASPage-2.png
[20]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/ServiceAttachedWithSAS.png
[21]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/connect-to-cosmos-db-by-connection-string.png
[22]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/connection-string-for-cosmos-db.png
[23]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/storage-explorer-search-for-resource.png