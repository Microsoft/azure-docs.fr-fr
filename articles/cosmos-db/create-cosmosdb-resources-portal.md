---
title: 'Démarrage rapide : Créer des ressources Azure Cosmos DB à partir du portail Azure'
description: Ce guide de démarrage rapide montre comment créer une base de données, un conteneur et des éléments Azure Cosmos à l’aide du Portail Azure.
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 11/10/2020
ms.openlocfilehash: baeb5fbadfaf128c2c491a1fdb7e880b413878d6
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94491067"
---
# <a name="quickstart-create-an-azure-cosmos-account-database-container-and-items-from-the-azure-portal"></a>Démarrage rapide : Créer un compte, une base de données, un conteneur et des éléments Azure Cosmos à partir du portail Azure
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Azure portal](create-cosmosdb-resources-portal.md)
> * [.NET](create-sql-api-dotnet.md)
> * [Java](create-sql-api-java.md)
> * [Node.JS](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)
>  

Azure Cosmos DB est le service de base de données multi-modèle de Microsoft distribué à l’échelle mondiale. Vous pouvez utiliser Azure Cosmos DB pour créer et interroger rapidement des bases de données de paires clé/valeur, des bases de données de documents et des bases de données de graphes, qui bénéficient toutes des fonctionnalités de distribution mondiale et de mise à l’échelle horizontale au cœur même d’Azure Cosmos DB. 

Ce guide de démarrage rapide montre comment utiliser le portail Azure pour créer un compte d’[API SQL](./introduction.md) Azure Cosmos DB, comment créer une base de données de documents et un conteneur, et comment ajouter des données au conteneur. 

## <a name="prerequisites"></a>Prérequis

Un abonnement Azure ou un compte d’essai gratuit Azure Cosmos DB
- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] 

- [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]  

<a id="create-account"></a>
## <a name="create-an-azure-cosmos-db-account"></a>Création d’un compte Azure Cosmos DB

Accédez au [portail Azure](https://portal.azure.com/) pour créer un compte Azure Cosmos DB. Recherchez et sélectionnez **Azure Cosmos DB**.

   :::image type="content" source="./media/create-cosmosdb-resources-portal/find-nosql-cosmosdb-marketplace.png" alt-text="Volet Bases de données du portail Azure":::

1. Sélectionnez **Ajouter**.
1. Dans la page **Créer un compte Azure Cosmos DB** , entrez les paramètres de base du nouveau compte Azure Cosmos. 

    |Paramètre|Valeur|Description |
    |---|---|---|
    |Abonnement|Nom d’abonnement|Sélectionnez l’abonnement Azure à utiliser pour ce compte Azure Cosmos. |
    |Groupe de ressources|Nom de groupe ressources|Sélectionnez un groupe de ressources ou sélectionnez **Créer** , puis entrez un nom unique pour le nouveau groupe de ressources. |
    |Nom du compte|Un nom unique|Entrez un nom pour identifier votre compte Azure Cosmos. Étant donné que *documents.azure.com* est ajouté au nom que vous fournissez pour créer votre URI, utilisez un nom unique.<br><br>Le nom peut uniquement contenir des lettres minuscules, des chiffres et le caractère de trait d’union (-). Sa longueur doit être comprise entre 3 et 31 caractères.|
    |API|Type de compte à créer|Sélectionnez **Core (SQL)** pour créer une base de données orientée document et effectuez des requêtes à l’aide de la syntaxe SQL. <br><br>L’API détermine le type de compte à créer. Azure Cosmos DB fournit cinq API : Core (SQL) et MongoDB pour les données de document, Gremlin pour les données de graphe, Table Azure et Cassandra. Actuellement, vous devez créer un compte distinct pour chaque API. <br><br>[En savoir plus sur l’API SQL](introduction.md).|
    |Mode de capacité|Débit approvisionné ou serverless|Sélectionnez **Débit approvisionné** pour créer un compte dans mode de [débit approvisionné](set-throughput.md). Sélectionnez **serverless** pour créer un compte en mode [serverless](serverless.md).|
    |Appliquer la remise de niveau gratuit|Appliquer ou ne pas appliquer|Avec Azure Cosmos DB niveau gratuit, vous recevrez 400 RU/s et 5 Go de stockage dans un compte, sans frais. Découvrez-en plus sur le [niveau gratuit](https://azure.microsoft.com/pricing/details/cosmos-db/).|
    |Emplacement|La région la plus proche de vos utilisateurs|Sélectionnez la zone géographique dans laquelle héberger votre compte Azure Cosmos DB. Utilisez l’emplacement le plus proche de vos utilisateurs pour leur donner l’accès le plus rapide possible aux données.|
    |Type de compte|Production ou non-production|Sélectionnez **Production** si le compte sera utilisé pour une charge de travail de production. Sélectionnez **Hors production** si le compte sera utilisé hors production, par exemple pour le développement, les tests, l’assurance qualité ou la préproduction. Il s’agit d’un paramètre d’étiquette de ressource Azure qui permet d’ajuster l’expérience du portail sans affecter le compte Azure Cosmos DB sous-jacent. Vous pouvez modifier cette valeur à tout moment.|
    |Géoredondance|Activer ou désactiver|Activez ou désactivez la diffusion mondiale sur votre compte en appairant votre région avec une région correspondante. Vous pourrez ajouter d’autres régions à votre compte ultérieurement.|
    |Écritures multirégions|Activer ou désactiver|La fonctionnalité d’écritures multirégions vous permet de tirer parti du débit provisionné pour vos bases de données et conteneurs à travers le monde.|
    |Zones de disponibilité|Activer ou désactiver|Les zones de disponibilité vous permettent d’accroître la disponibilité et d’améliorer la résilience de votre application.|

> [!NOTE]
> Vous pouvez avoir un seul compte Azure Cosmos DB de niveau gratuit par abonnement Azure et vous devez vous inscrire lors de la création du compte. Si vous ne voyez pas l’option permettant d’appliquer la remise de niveau gratuit, cela signifie qu’un autre compte dans l’abonnement a déjà été activé avec le niveau gratuit.

> [!NOTE]
> Les options suivantes ne sont pas disponibles si vous sélectionnez **Serverless** comme **Mode de capacité**  :
> - Appliquer la remise de niveau gratuit
> - Géo-redondance
> - Écritures multirégions
   
   :::image type="content" source="./media/create-cosmosdb-resources-portal/azure-cosmos-db-create-new-account-detail.png" alt-text="Page de nouveau compte pour Azure Cosmos DB":::

1. Sélectionnez **Revoir + créer**. Vous pouvez ignorer les sections **Réseau** et **Balises**.

1. Passez en revue les paramètres du compte, puis sélectionnez **Créer**. La création du compte prend quelques minutes. Attendez que la page du portail affiche **Votre déploiement est terminé**. 

    :::image type="content" source="./media/create-cosmosdb-resources-portal/azure-cosmos-db-account-deployment-successful.png" alt-text="Volet Notifications du portail Azure":::

1. Sélectionnez **Accéder à la ressource** pour accéder à la page du compte Azure Cosmos DB. 

    :::image type="content" source="./media/create-cosmosdb-resources-portal/azure-cosmos-db-account-quickstart-pane.png" alt-text="Page du compte Azure Cosmos DB":::

<a id="create-container-database"></a>
## <a name="add-a-database-and-a-container"></a>Ajouter une base de données et un conteneur 

Vous pouvez utiliser l’Explorateur de données du portail Azure pour créer une base de données et un conteneur. 

1.  Dans la page de votre compte Azure Cosmos DB, sur la gauche, sélectionnez **Explorateur de données** , puis **Nouveau conteneur**. 
    
    Vous devrez peut-être faire défiler vers la droite pour voir la fenêtre **Ajouter un conteneur**.
    
    :::image type="content" source="./media/create-sql-api-dotnet/azure-cosmosdb-data-explorer-dotnet.png" alt-text="Explorateur de données du portail Azure, volet Ajouter un conteneur":::
    
1.  Dans le volet **Ajouter un conteneur** , entrez les paramètres du nouveau conteneur.
    
    |Paramètre|Valeur suggérée|Description
    |---|---|---|
    |**ID de base de données**|ToDoList|Entrez *ToDoList* pour le nom de la nouvelle base de données. Les noms de base de données doivent comporter entre 1 et 255 caractères, et ne peuvent pas contenir les caractères `/, \\, #, ?`, ni un espace de fin. Cochez l’option **Provisionner le débit de base de données** qui vous permet de partager le débit provisionné pour la base de données entre tous les conteneurs au sein de la base de données. Cette option permet également de réduire les coûts. |
    |**Débit**|400|Laissez le débit sur 400 unités de requête par seconde (RU/s). Si vous souhaitez réduire la latence, vous pourrez augmenter le débit par la suite.<br><br>**Remarque**  : Ce paramètre n’est pas disponible lors de la création d’un nouveau conteneur dans un compte serverless.| 
    |**ID de conteneur**|Éléments|Entrez *Éléments* comme nom de votre nouveau conteneur. Les ID de conteneur sont soumis aux mêmes exigences en termes de caractères que les noms de base de données.|
    |**Clé de partition**| /category| L’exemple décrit dans cet article utilise */category* comme clé de partition.|

    
    Pour cet exemple, n’ajoutez pas de **Clés uniques**. Les clés uniques vous permettent d’ajouter une couche d’intégrité des données à la base de données en garantissant que chaque clé de partition contient des valeurs uniques. Pour plus d’informations, consultez [Clés uniques dans Azure Cosmos DB](unique-keys.md).
    
1.  Sélectionnez **OK**. L’Explorateur de données présente la nouvelle base de données et le conteneur que vous avez créé.

## <a name="add-data-to-your-database"></a>Ajouter des données à votre base de données

Ajoutez des données à votre nouvelle base de données à l’aide de l’Explorateur de données.

1. Dans l’ **Explorateur de données** , développez la base de données **ToDoList** , puis développez le conteneur **Éléments**. Ensuite, sélectionnez **Éléments** , puis sélectionnez **Nouvel élément**. 
   
   :::image type="content" source="./media/create-sql-api-dotnet/azure-cosmosdb-new-document.png" alt-text="Créer des documents dans l’Explorateur de données du portail Azure":::
   
1. Ajoutez la structure suivante au document, sur la droite du volet **Documents**  :

     ```json
     {
         "id": "1",
         "category": "personal",
         "name": "groceries",
         "description": "Pick up apples and strawberries.",
         "isComplete": false
     }
     ```

1. Sélectionnez **Enregistrer**.
   
   :::image type="content" source="./media/create-sql-api-dotnet/azure-cosmosdb-save-document.png" alt-text="Copie des données json et sélection de l’option Enregistrer dans l’Explorateur de données du portail Azure":::
   
1. Sélectionnez à nouveau **Nouveau document** puis créez et enregistrez un autre document avec un `id` unique, ainsi que les propriétés et valeurs souhaitées. Vos documents peuvent avoir la structure de votre choix, car Azure Cosmos DB n’impose aucun schéma à vos données.

## <a name="query-your-data"></a>Interroger vos données

[!INCLUDE [cosmos-db-create-sql-api-query-data](../../includes/cosmos-db-create-sql-api-query-data.md)] 

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

Si vous souhaitez supprimer uniquement la base de données et utiliser le compte Azure Cosmos à l’avenir, vous pouvez supprimer la base de données en effectuant les étapes suivantes :

* Accédez à votre compte Azure Cosmos.
* Ouvrez l’ **Explorateur de données** , cliquez avec le bouton droit sur la base de données que vous souhaitez supprimer et sélectionnez **Supprimer la de base de données**.
* Entrez l’ID et le nom de la base de données pour confirmer l’opération de suppression. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez appris à créer un compte Azure Cosmos DB, ainsi qu’à créer une base de données et un conteneur à l’aide de l’Explorateur de données. Vous pouvez maintenant importer des données supplémentaires dans votre compte Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importer des données dans Azure Cosmos DB](import-data.md)