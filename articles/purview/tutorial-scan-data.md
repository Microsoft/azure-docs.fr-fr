---
title: 'Tutoriel : Analyser des données avec Azure Purview (préversion)'
description: Dans ce tutoriel, vous allez exécuter un starter kit pour configurer un patrimoine de données, puis vous analyserez les données des sources de données dans votre catalogue Azure Purview.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: tutorial
ms.date: 12/01/2020
ms.openlocfilehash: feda969a5818402f4a5f2dadc80c70c324ad7492
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97559962"
---
# <a name="tutorial-scan-data-with-azure-purview-preview"></a>Tutoriel : Analyser des données avec Azure Purview (préversion)

> [!IMPORTANT]
> Azure Purview est actuellement disponible en PRÉVERSION. L’[Avenant aux conditions d’utilisation pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) contient des conditions légales supplémentaires qui s’appliquent aux fonctionnalités Azure en version bêta, en préversion ou pas encore en disponibilité générale.

Dans cette *série de cinq tutoriels*, vous allez apprendre les principes fondamentaux de la gestion de la gouvernance des données dans un patrimoine de données à l’aide d’Azure Purview (préversion). Le patrimoine de données que vous créez dans cette partie du tutoriel est utile dans toute la série.

Dans la partie 1 de cette série de didacticiels, vous allez :

> [!div class="checklist"]
>
> * Créer un patrimoine de données avec diverses ressources de données Azure.
> * Analyser les données dans un catalogue.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.
* Un [compte Azure Purview](create-catalog-portal.md).
* [Le starter kit](https://download.microsoft.com/download/9/7/9/979db3b1-0916-4997-a7fb-24e3d8f83174/PurviewStarterKitV4.zip) qui déploiera votre patrimoine de données.

> [!NOTE]
> Le starter kit est disponible uniquement pour Windows.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="create-a-data-estate"></a>Créer un patrimoine de données

Dans cette section, vous exécutez les scripts du starter kit pour créer une simulation de patrimoine de données. Un patrimoine de données est un portefeuille de toutes les données qu’une entreprise possède. Le script du starter kit effectue les actions suivantes :

* Il crée un compte de stockage Blob Azure et il y stocke des données.
* Il crée un compte Azure Data Lake Storage Gen2.
* Il crée une instance Azure Data Factory et associe cette instance à Azure Purview.
* Il configure et déclenche un pipeline d’activité de copie entre les comptes de stockage Blob Azure et Azure Data Lake Storage Gen2.
* Il transfère (push) la traçabilité associée d’Azure Data Factory à Azure Purview.

### <a name="prepare-to-run-the-starter-kit"></a>Préparer l’exécution du starter kit

Procédez comme suit pour configurer le logiciel client du starter kit sur votre ordinateur Windows :

1. [Téléchargez le starter kit](https://download.microsoft.com/download/9/7/9/979db3b1-0916-4997-a7fb-24e3d8f83174/PurviewStarterKitV4.zip) et extrayez son contenu à l’emplacement de votre choix.


1. Sur votre ordinateur, entrez **PowerShell** dans la zone de recherche de la barre des tâches Windows. Dans la liste des résultats de recherche, cliquez avec le bouton droit sur **Windows PowerShell**, puis sélectionnez **Exécuter en tant qu’administrateur**.

1. Dans la fenêtre PowerShell, entrez la commande suivante, en remplaçant `<path-to-starter-kit>` par le chemin d’accès au dossier d’extraction des fichiers du starter kit.

   ```powershell
   dir -Path <path-to-starter-kit> -Recurse | Unblock-File
   ```

1. Entrez la commande suivante pour installer les applets de commande Azure.

   ```powershell
   Install-Module -Name Az -AllowClobber -Scope CurrentUser
   ```

1. Si vous voyez l’invite d’avertissement *Le fournisseur NuGet est requis pour continuer*, entrez **Y** et appuyez sur Entrée.

1. Si vous voyez l’invite d’avertissement *Référentiel non approuvé*, entrez **A** et appuyez sur Entrée.

L’installation par PowerShell des modules requis peut prendre jusqu’à une minute.

### <a name="collect-data-needed-to-run-the-scripts"></a>Collecter les données requises pour exécuter les scripts

Avant d’exécuter les scripts PowerShell pour amorcer le catalogue, obtenez les valeurs des arguments suivants à utiliser dans ces scripts :

* ID de locataire
   1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.
   1. Dans la section **Gérer** du volet de navigation gauche, sélectionnez **Propriétés**. Sélectionnez ensuite l’icône de copie pour l’**ID de locataire** afin d’enregistrer la valeur dans le presse-papiers. Collez cette valeur dans un éditeur de texte dans le but de l’utiliser ultérieurement.

* SubscriptionID
   1. Dans le portail Azure, recherchez et sélectionnez le nom de l’instance Azure Purview que vous avez créée comme prérequis.
   1. Sélectionnez la section **Vue d’ensemble** et enregistrez le GUID de l’**ID d’abonnement**.

   > [!NOTE]
   > Assurez-vous que vous utilisez le même abonnement que celui dans lequel vous avez créé le compte Azure Purview. Il s’agit du même abonnement que celui qui a été placé dans la liste verte.

* CatalogName : nom du compte Azure Purview que vous avez créé dans [Créer un compte Azure Purview](create-catalog-portal.md).

* CatalogResourceGroupName : nom du groupe de ressources dans lequel vous avez créé votre compte Azure Purview.

### <a name="verify-permissions"></a>Vérification des autorisations

Procédez comme suit pour ajouter l’utilisateur qui exécute le script au compte Azure Purview qui a été créé comme prérequis. Les utilisateurs ont besoin des deux rôles *Conservateur de données Purview* et *Administrateur de sources de données Purview*. 

Si vous avez créé vous-même le compte Azure Purview, vous obtenez automatiquement l’accès et vous pouvez ignorer cette section.

1. Accédez à la page des comptes Purview dans le portail Azure et sélectionnez le compte Azure Purview que vous souhaitez modifier.

1. Dans la page du compte, sélectionnez l’onglet **Contrôle d’accès (IAM)** et **+ Ajouter**.

1. Sélectionnez **Ajouter une attribution de rôle**.

1. Entrez **Rôle Conservateur de données Purview** pour *Rôle*.
 
1. Utilisez la valeur par défaut pour *Attribuer l’accès à*. La valeur par défaut doit être **Utilisateur, groupe ou principal de service**.

1. Entrez le nom de l’utilisateur qui exécute le script dans **Sélectionner**.

1. Sélectionnez **Enregistrer**.

1. Répétez les étapes précédentes avec le *rôle* défini sur **Rôle Administrateur de sources de données Purview**.

Pour plus d’informations, consultez [Autorisations du catalogue](catalog-permissions.md).

### <a name="run-the-client-side-setup-scripts"></a>Exécuter les scripts de configuration côté client

Une fois la configuration du catalogue terminée, exécutez les scripts suivants dans la fenêtre PowerShell pour créer les ressources, en remplaçant les espaces réservés par les valeurs que vous avez collectées précédemment.

1. Utilisez la commande suivante pour accéder au répertoire du starter kit. Remplacez `path-to-starter-kit` par le chemin d’accès au dossier d’extraction du fichier.

   ```powershell
   cd <path-to-starter-kit>
   ```

1. La commande suivante définit la stratégie d’exécution pour l’ordinateur local. Entrez **A** pour *Oui pour tout* lorsque vous êtes invité à modifier la stratégie d’exécution.

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Unrestricted
   ```

1. Connectez-vous à Azure en utilisant la commande suivante. Remplacez les valeurs des espaces réservés `TenantID` et `SubscriptionID` .

   ```powershell
   .\RunStarterKit.ps1 -ConnectToAzure -TenantId <TenantID> `
   -SubscriptionId <SubscriptionID>
   ```

   Quand vous exécutez cette commande, une fenêtre contextuelle peut s’afficher pour que vous vous connectiez avec vos informations d’identification Azure Active Directory.


1. Utilisez la commande suivante pour exécuter le starter kit. Remplacez les valeurs des espaces réservés `CatalogName`, `TenantID`, `SubscriptionID`, `NewResourceGroupName` et `CatalogResourceGroupName`. Pour `NewResourceGroupName`, utilisez un nom unique (avec uniquement des caractères alphanumériques minuscules) pour le groupe de ressources qui contiendra le patrimoine de données.

   > [!IMPORTANT]
   > Le nom **newresourcegroupname** doit être composé uniquement de chiffres et de lettres minuscules, et doit compter moins de 17 caractères. **Aucune lettre majuscule ni aucun caractère spécial ne sont autorisés.** Cette contrainte provient des règles de nommage des comptes de stockage.

   ```powershell
   .\RunStarterKit.ps1 -CatalogName <CatalogName> -TenantId <TenantID>`
   -ResourceGroup <newresourcegroupname> `
   -SubscriptionId <SubscriptionID> `
   -CatalogResourceGroup <CatalogResourceGroupName>
   ```

La configuration de l’environnement peut prendre jusqu’à 10 minutes. Pendant ce temps, plusieurs fenêtres contextuelles peuvent s’afficher, que vous pouvez ignorer. Ne fermez pas la fenêtre **BlobDataCreator.exe**. Elle se ferme automatiquement, une fois l’opération terminée.

Quand vous voyez le message `Executing Copy pipeline xxxxxxxxxx-487e-4fc4-9628-92dd8c2c732b`, attendez qu’une autre instance de **BlobDataCreator.exe** démarre et finisse de s’exécuter.

> [!IMPORTANT]
> Si le « nombre de tâches actives » cesse de diminuer, vous pouvez quitter la fenêtre du créateur de blob et appuyer sur Entrée dans la fenêtre PowerShell.

Une fois le processus terminé, un groupe de ressources portant le nom que vous avez spécifié est créé. Les comptes Azure Data Factory, Stockage Blob Azure et Azure Data Lake Storage Gen2 sont tous contenus dans ce groupe de ressources. Le groupe de ressources est contenu dans l’abonnement que vous avez spécifié.

## <a name="scan-data-into-the-catalog"></a>Analyser les données dans le catalogue

L’analyse est un processus par lequel le catalogue se connecte directement à une source de données selon une planification spécifiée par l’utilisateur. Le catalogue reflète le patrimoine de données d’une entreprise par le biais de l’analyse, de la traçabilité, du portail et de l’API. Les objectifs incluent l’examen du contenu, l’extraction de schémas et la compréhension de la sémantique. Dans cette section, vous allez configurer une analyse des sources de données que vous avez générées avec le starter kit.

Le script du starter kit que vous avez exécuté a créé deux sources de données, le stockage Blob Azure et Azure Data Lake Storage Gen2. Vous pouvez analyser ces sources de données dans le catalogue, l’une après l’autre.

### <a name="authenticate-to-your-storage-with-managed-identity"></a>S’authentifier auprès du stockage avec une identité managée

Une identité managée portant le même nom que votre compte Azure Purview est automatiquement créée lors de la création de votre compte. Pour pouvoir analyser vos données, vous devez au préalable accorder au rôle Azure Purview des autorisations sur vos comptes de stockage.

1. Accédez au groupe de ressources créé par le starter kit et sélectionnez votre compte de stockage Blob.

1. Dans le menu de navigation de gauche, sélectionnez **Contrôle d’accès (IAM)** . Sélectionnez ensuite **+ Ajouter**.

1. Définissez le rôle sur **Lecteur des données Blob du stockage** et entrez le nom de votre compte Azure Purview pour **Sélectionner**. Ensuite, sélectionnez **Enregistrer** pour fournir cette attribution de rôle à votre compte Purview.

   :::image type="content" source="media/tutorial-scan-data/add-role-assignment.png" alt-text="Ajouter une attribution de rôle":::

1. Répétez les étapes précédentes pour Azure Data Lake Storage Gen2.

### <a name="scan-your-data-sources"></a>Analyser vos sources de données

1. Sélectionnez **Sources** sur la page web de votre catalogue et sélectionnez **Inscrire**. Ensuite, sélectionnez **Stockage Blob Azure** et **Continuer**.

   :::image type="content" source="media/tutorial-scan-data/add-blob-storage.png" alt-text="Inscrire une ressource de stockage Blob":::

1. Dans la page **Inscrire des sources**, entrez un **nom**. Choisissez le **nom du compte de stockage** du compte de stockage Blob Azure que vous avez créé précédemment avec le starter kit. Le nom du compte est `<YourResourceGroupName>adcblob`. Sélectionnez **Terminer**.

   :::image type="content" source="./media/tutorial-scan-data/register-azure-blob-storage.png" alt-text="Capture d’écran montrant les paramètres d’inscription d’une source de données de stockage Blob Azure" border="true":::

1. Dans la vue cartographique **Sources de données**, sélectionnez **Nouvelle analyse** sur la vignette Stockage Blob Azure.

   :::image type="content" source="./media/tutorial-scan-data/select-setup-scan.png" alt-text="Capture d’écran montrant comment sélectionner une configuration d’analyse à partir d’une source de données" border="true":::

1. Dans la page **Analyser**, entrez un nom d’analyse, sélectionnez votre identité managée dans la liste déroulante **Informations d’identification**, puis sélectionnez **Continuer**.

   :::image type="content" source="media/tutorial-scan-data/scan-blob.png" alt-text="Analyser le stockage Blob dans Azure Purview":::

1. Vous pouvez étendre votre analyse à des objets blob individuels. Dans ce tutoriel, maintenez toutes les ressources cochées pour tout analyser et sélectionnez **Continuer**.

   :::image type="content" source="media/tutorial-scan-data/scope-your-scan.png" alt-text="Étendre votre analyse de stockage":::

1. Sélectionnez l’ensemble de règles d’analyse par défaut et sélectionnez **Continuer**.

1. Vous pouvez configurer un déclencheur d’analyse pour les analyses récurrentes. Pour ce tutoriel, sélectionnez **Une fois** et **Continuer**.

1. Sélectionnez **Enregistrer et exécuter** pour effectuer l’analyse.

1. Répétez les étapes précédentes pour analyser votre compte Azure Data Lake Storage Gen2.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :
> [!div class="checklist"]
>
> * Exécuter le script du starter kit pour configurer un environnement de patrimoine de données.
> * Analyser les données dans un catalogue Azure Purview.

Passez au tutoriel suivant pour apprendre à parcourir la page d’accueil et à rechercher une ressource.

> [!div class="nextstepaction"]
> [Parcourir la page d’accueil et rechercher une ressource](tutorial-asset-search.md)
