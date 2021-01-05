---
title: Créer une base de connaissances dans le portail Azure
titleSuffix: Azure Cognitive Search
description: Utilisez l’Assistant Importation de données pour créer une base de connaissances où conserver le contenu enrichi persistant. Connectez-vous à une base de connaissances pour l’analyse à partir d’autres applications, ou envoyez du contenu enrichi aux processus en aval.
author: HeidiSteen
ms.author: heidist
manager: nitinme
ms.service: cognitive-search
ms.topic: quickstart
ms.date: 11/17/2020
ms.openlocfilehash: 3225013f09abd326c619b67caf77918889a64859
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94741805"
---
# <a name="quickstart-create-an-azure-cognitive-search-knowledge-store-in-the-azure-portal"></a>Démarrage rapide : Créer une base de connaissances Recherche cognitive Azure dans le portail Azure

La base de connaissances est une fonctionnalité de la Recherche cognitive Azure qui conserve la sortie d’un pipeline de traitement de contenu en vue d’analyses ultérieures ou d’un traitement en aval. 

Un pipeline accepte du contenu texte et image non structuré, applique l’intelligence artificielle alimentée par Cognitive Services (par exemple, la reconnaissance optique de caractères et le traitement en langage naturel) et génère de nouvelles structures et des informations qui n’existaient pas auparavant. L’un des artefacts physiques créés par un pipeline est une [base de connaissances](knowledge-store-concept-intro.md), à laquelle vous pouvez accéder par le biais d’outils pour analyser et explorer le contenu.

Dans ce guide de démarrage rapide, vous combinez des services et des données dans le cloud Azure pour créer une base de connaissances. Une fois tout en place, vous allez exécuter l’Assistant **Importation de données** dans le portail pour tout rassembler. Vous affichez ensuite le contenu du texte d’origine et le contenu généré par l’intelligence artificielle dans le portail (avec l’[Explorateur Stockage](knowledge-store-view-storage-explorer.md)).

## <a name="prerequisites"></a>Prérequis

Avant de commencer la lecture cet article, vous devez disposer des éléments suivants :

+ Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/).

+ Service Recherche cognitive Azure. [Créez un service](search-create-service-portal.md) ou [recherchez un service existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) dans votre abonnement actuel. Vous pouvez utiliser un service gratuit pour ce guide de démarrage rapide. 

+ Un compte de stockage Azure avec un [stockage blob](../storage/blobs/index.yml).

> [!NOTE]
> Ce guide de démarrage rapide utilise également [Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/) pour l’intelligence artificielle. Parce que la charge de travail est vraiment petite, Cognitive Services est utilisé en arrière-plan pour traiter gratuitement jusqu’à 20 transactions. Cela signifie que vous pouvez effectuer cet exercice sans avoir à créer une ressource Cognitive Services supplémentaire.

## <a name="set-up-your-data"></a>Configurer vos données

Dans les étapes suivantes, configurez un conteneur d’objets blob dans Stockage Azure pour stocker des fichiers de contenu hétérogènes.

1. [Téléchargez le fichier HotelReviews_Free.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Free.csv?sp=r&st=2019-11-04T01:23:53Z&se=2025-11-04T16:00:00Z&spr=https&sv=2019-02-02&sr=b&sig=siQgWOnI%2FDamhwOgxmj11qwBqqtKMaztQKFNqWx00AY%3D). Ce fichier CSV contient des données d’avis d’hôtel (issues de Kaggle.com). Il rassemble 19 commentaires de clients relatifs à un seul hôtel. 

1. [Créez un compte de stockage Azure](../storage/common/storage-account-create.md?tabs=azure-portal) ou [recherchez un compte existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Storage%2storageAccounts/) dans votre abonnement actuel. Vous utilisez le stockage Azure pour le contenu brut à importer, mais aussi pour la base de connaissances qui est le résultat final.

   + Choisissez le type de compte **StorageV2 (usage général v2)** .

1. Ouvrez les pages des services Blob et créez un conteneur nommé *hotel-reviews*.

1. Cliquez sur **Télécharger**.

    ![Charger les données](media/knowledge-store-create-portal/upload-command-bar.png "Charger les avis sur les hôtels")

1. Sélectionnez le fichier **HotelReviews-Free.csv** que vous avez téléchargé à la première étape.

    ![Créer le conteneur d’objets Blob Azure](media/knowledge-store-create-portal/hotel-reviews-blob-container.png "Créer le conteneur d’objets Blob Azure")

1. Avant de quitter les pages du Stockage Blob, utilisez un lien dans le volet de navigation gauche pour ouvrir la page **Clés d’accès**. Obtenez une chaîne de connexion pour récupérer les données du Stockage Blob. Une chaîne de connexion ressemble à l’exemple suivant : `DefaultEndpointsProtocol=https;AccountName=<YOUR-ACCOUNT-NAME>;AccountKey=<YOUR-ACCOUNT-KEY>;EndpointSuffix=core.windows.net`

Vous êtes maintenant prêt à passer à l’**Assistant Importation de données**.

## <a name="run-the-import-data-wizard"></a>Exécuter l’Assistant Importation de données

1. Connectez-vous au [portail Azure](https://portal.azure.com/) avec votre compte Azure.

1. [Trouvez votre service de recherche](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Storage%2storageAccounts/). Ensuite, dans la page Vue d’ensemble, cliquez sur **Importer des données** dans la barre de commandes pour créer une base de connaissances en quatre étapes.

   ![Commande Importer des données](media/cognitive-search-quickstart-blob/import-data-cmd2.png)

### <a name="step-1-create-a-data-source"></a>Étape 1 : Création d'une source de données

1. Dans **Se connecter à vos données**, choisissez **Stockage Blob Azure**, puis sélectionnez le compte et le conteneur que vous avez créés. 

1. Pour **Nom**, entrez `hotel-reviews-ds`.

1. Pour **Mode d’analyse**, sélectionnez **Texte délimité**, puis cochez la case **La première ligne contient l’en-tête**. Vérifiez que le **Caractère délimiteur** est une virgule (,).

1. Dans **Chaîne de connexion**, collez la chaîne de connexion que vous avez copiée à partir de la page **Clés d’accès** dans Stockage Azure.

1. Dans **Conteneurs**, entrez le nom du conteneur d’objets blob contenant les données.

    Votre page doit ressembler à la capture d’écran suivante.

    ![Créer un objet de source de données](media/knowledge-store-create-portal/hotel-reviews-ds.png "Créer un objet de source de données")

1. Passez à la page suivante.

### <a name="step-2-add-cognitive-skills"></a>Étape 2 : Ajouter des compétences cognitives

Dans cette étape de l’Assistant, vous allez créer un ensemble de compétences par enrichissement des compétences cognitives. Les données sources sont constituées des évaluations des clients dans plusieurs langues. Les compétences pertinentes pour ce jeu de données incluent l’extraction d’expressions clés, la détection de sentiments et la traduction de texte. Dans une étape ultérieure, ces enrichissements seront « projetés » dans une base de connaissances en tant que tables Azure.

1. Développez **Attacher Cognitive Services**. **Gratuit (enrichissements limités)** est sélectionné par défaut. Vous pouvez utiliser cette ressource, car le nombre d’enregistrements dans HotelReviews-Free.csv est de 19, et cette ressource gratuite autorise jusqu’à 20 transactions par jour.

1. Développez **Ajouter des enrichissements**.

1. Pour **Nom de l’ensemble de compétences**, entrez `hotel-reviews-ss`.

1. Pour **Champ de données source**, sélectionnez **reviews_text**.

1. Pour **Niveau de précision d’enrichissement**, sélectionnez **Pages (segments de 5 000 caractères)**

1. Sélectionnez les compétences cognitives suivantes :
    + **Extraire des expressions clés**
    + **Traduire le texte**
    + **Détecter le sentiment**

      ![Créer un ensemble de compétences](media/knowledge-store-create-portal/hotel-reviews-ss.png "Créer un ensemble de compétences")

1. Développez **Enregistrer les enrichissements dans une base de connaissances**.

1. Sélectionnez les **projections de table Azure** suivantes :
    + **Documents**
    + **Pages**
    + **Phrases clés**

1. Entrez la **Chaîne de connexion du compte de stockage**, que vous avez enregistrée à une étape précédente.

    ![Configurer la base de connaissances](media/knowledge-store-create-portal/hotel-reviews-ks.png "Configurer la base de connaissances")

1. Éventuellement, téléchargez un modèle Power BI. Quand vous accédez au modèle à partir de l’Assistant, le fichier .pbit local est adapté pour refléter la forme de vos données.

1. Passez à la page suivante.

### <a name="step-3-configure-the-index"></a>Étape 3 : Configurer l’index

Dans cette étape de l’Assistant, vous allez configurer un index pour d’éventuelles requêtes de recherche en texte intégral. L’Assistant va échantillonner votre source de données pour en déduire des champs et des types de données. Il vous suffit de sélectionner les attributs correspondant au comportement souhaité. Par exemple, l’attribut **Récupérable** permet au service de recherche de retourner une valeur de champ, alors que l’attribut **Possibilité de recherche** active la recherche en texte intégral sur le champ.

1. Pour **Nom de l’index**, entrez `hotel-reviews-idx`.

1. Pour les attributs, acceptez les sélections par défaut : **Récupérable** et **Possibilité de recherche** pour les champs que le pipeline crée.

    Votre index doit ressembler à l’image suivante. Dans la mesure où la liste est longue, tous les champs ne sont pas visibles dans l’image.

    ![Configurer un index](media/knowledge-store-create-portal/hotel-reviews-idx.png "Configurer un index")

1. Passez à la page suivante.

### <a name="step-4-configure-the-indexer"></a>Étape 4 : Configurer l’indexeur

Dans cette étape de l’Assistant, vous allez configurer un indexeur qui doit rassembler la source de données, l’ensemble de compétences et l’index que vous avez définis dans les étapes précédentes de l’Assistant.

1. Pour **Nom**, entrez `hotel-reviews-idxr`.

1. Pour **Planification**, conservez la valeur par défaut **Une fois**.

1. Cliquez sur **Envoyer** pour exécuter l’indexeur. Les opérations d’extraction de données, d’indexation et d’application des compétences cognitives se produisent toutes à cette étape.

## <a name="monitor-status"></a>Superviser l’état

L’indexation des compétences cognitives prend plus de temps que l’indexation standard basée sur du texte. L’Assistant doit ouvrir la liste de l’indexeur sur la page Vue d’ensemble afin que vous puissiez suivre la progression. Pour une navigation automatique, accédez à la page Vue d’ensemble et cliquez sur **Indexeurs**.

Dans le portail Azure, vous pouvez également superviser le journal d’activité Notifications, et rechercher le lien d’état cliquable **Notification Recherche cognitive Azure**. L’exécution peut prendre plusieurs minutes.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez enrichi vos données à l’aide de Cognitive Services et que vous avez projeté les résultats dans une base de connaissances, vous pouvez utiliser l’Explorateur Stockage ou Power BI pour explorer votre jeu de données enrichi.

Vous pouvez afficher le contenu dans l’Explorateur Stockage, ou aller un peu plus loin avec Power BI pour obtenir des insights par le biais de la visualisation.

> [!div class="nextstepaction"]
> [Afficher avec l’Explorateur Stockage](knowledge-store-view-storage-explorer.md)
> [Se connecter à Power BI](knowledge-store-connect-power-bi.md)

> [!Tip]
> Si vous souhaitez répéter cet exercice ou essayer une autre procédure pas à pas d’enrichissement par IA, supprimez l’indexeur *hotel-reviews-idxr*. La suppression de l’indexeur réinitialise le compteur de transactions quotidiennes gratuites à zéro pour le traitement Cognitive Services.
