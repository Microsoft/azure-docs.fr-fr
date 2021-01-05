---
title: Déplacer votre ressource de service d'une région vers une autre
titleSuffix: Azure Cognitive Search
description: Cet article explique comment déplacer vos ressources Recherche cognitive Azure d'une région vers une autre dans le cloud Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 09/10/2020
ms.openlocfilehash: 774b605859df41e0b71ee82c38a6b08bdf5b9c49
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91629809"
---
# <a name="move-your-azure-cognitive-search-service-to-another-azure-region"></a>Déplacer votre service Recherche cognitive Azure vers une autre région Azure

Parfois, les clients demandent à déplacer un service de recherche vers une autre région. Actuellement, il n’existe aucun mécanisme ou outil intégré pour faciliter cette tâche, mais cet article peut vous aider à comprendre les étapes manuelles permettant d’obtenir le même résultat.

> [!NOTE]
> Dans le portail Azure, tous les services ont une commande **Exporter le modèle**. Dans le cas de la Recherche cognitive Azure, cette commande produit une définition de base d’un service (nom, localisation, niveau, réplica et nombre de partitions), mais ne reconnaît pas le contenu de votre service, pas plus que les clés, les rôles ou les journaux. Bien que la commande existe, nous vous déconseillons de l’utiliser pour déplacer un service de recherche.

## <a name="prerequisites"></a>Prérequis

+ Vérifiez que les services et fonctionnalités utilisés par votre compte sont pris en charge dans la région cible.

+ Pour les fonctionnalités en préversion, vérifiez que votre abonnement est approuvé pour la région cible.

## <a name="prepare-and-move"></a>Préparer et déplacer

1. Identifiez les dépendances et les services associés pour comprendre l’impact complet du déplacement d’un service, au cas où vous devriez déplacer plus que simplement la Recherche cognitive Azure.

   Le stockage Azure est utilisé pour la journalisation, la création d’une base de connaissances et est une source de données externe couramment utilisée pour l’enrichissement et l’indexation par IA. Cognitive Services est une dépendance dans l’enrichissement par IA. Cognitive Services et votre service de recherche doivent se trouver dans la même région si vous utilisez l’enrichissement par IA.

1. Créez un inventaire de tous les objets sur le service afin de savoir ce qu’il faut déplacer : les index, les cartes de synonymes, les indexeurs, les sources de données, les ensembles de compétences. Si vous avez activé la journalisation, créez et archivez les rapports dont vous pouvez avoir besoin pour un enregistrement historique.

1. Vérifiez les tarifs et la disponibilité dans la nouvelle région pour garantir la disponibilité de la Recherche cognitive Azure ainsi que tous les services associés dans la même région. La majorité des fonctionnalités sont disponibles dans toutes les régions, mais certaines fonctionnalités d’évaluation ont une disponibilité limitée.

1. Créez un service dans la nouvelle région et republiez à partir du code source les index, les cartes de synonymes, les indexeurs, les sources de données et les ensembles de compétences. N’oubliez pas que les noms de service doivent être uniques afin que vous ne puissiez pas réutiliser le nom existant. Vérifiez chaque ensemble de compétences pour voir si les connexions à Cognitive Services sont toujours valides dans le cadre de la spécification de la même région. En outre, si vous créez des bases de connaissances, vérifiez les chaînes de connexion pour le stockage Azure si vous utilisez un autre service.

1. Rechargez les index et les bases de connaissances, le cas échéant. Vous allez utiliser du code d’application pour envoyer (push) des données JSON à un index ou réexécuter des indexeurs pour tirer (pull) des documents de sources externes. 

1. Activez la journalisation et, si vous les utilisez, recréez les rôles de sécurité.

1. Mettez à jour les applications clientes et les suites de tests pour utiliser le nouveau nom de service et les clés API, puis testez toutes les applications.

## <a name="discard-or-clean-up"></a>Ignorer ou nettoyer

Supprimez l’ancien service une fois que le nouveau service est entièrement testé et opérationnel. La suppression du service supprime automatiquement tout le contenu qui lui est associé.

## <a name="next-steps"></a>Étapes suivantes

Les liens suivants peuvent vous aider à trouver plus d’informations lorsque vous effectuez les étapes décrites ci-dessus.

+ [Tarifs et régions Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/)
+ [Choisir un niveau](search-sku-tier.md)
+ [Créer un service de recherche](search-create-service-portal.md)
+ [Charger les documents de recherche](search-what-is-data-import.md)
+ [Activation de la journalisation](search-monitor-logs.md)


<!-- To move your Azure Cognitive Service account from one region to another, you will create an export template to move your subscription(s). After moving your subscription, you will need to move your data and recreate your service.

In this article, you'll learn how to:

> [!div class="checklist"]
> * Export a template.
> * Modify the template: adding the target region, search and storage account names.
> * Deploy the template to create the new search and storage accounts.
> * Verify your service status in the new region
> * Clean up resources in the source region.

## Prerequisites

- Ensure that the services and features that your account uses are supported in the target region.

- For preview features, ensure that your subscription is whitelisted for the target region. For more information about preview features, see [knowledge stores](./knowledge-store-concept-intro.md), [incremental enrichment](./cognitive-search-incremental-indexing-conceptual.md), and [private endpoint](./service-create-private-endpoint.md).

## Assessment and planning

When you move your search service to the new region, you will need to [move your data to the new storage service](../storage/common/storage-account-move.md?tabs=azure-portal#configure-the-new-storage-account) and then rebuild your indexes, skillsets and knowledge stores. You should record current settings and copy json files to make the rebuilding of your service easier and faster.

## Moving your search service's resources

To start you will export and then modify a Resource Manager template.

### Export a template

1. Sign in to the [Azure portal](https://portal.azure.com).

2. Go to your Resource Group page.

> [!div class="mx-imgBorder"]
> ![Resource Group page example](./media/search-move-resource/export-template-sample.png)

3. Select **All resources**.

3. In the left hand navigation menu select **Export template**.

4. Choose **Download** in the **Export template** page.

5. Locate the .zip file that you downloaded from the portal, and unzip that file to a folder of your choice.

The zip file contains the .json files that comprise the template and scripts to deploy the template.

### Modify the template

You will modify the template by changing the search and storage account names and regions. The names must follow the rules for each service and region naming conventions. 

To obtain region location codes, see [Azure Locations](https://azure.microsoft.com/global-infrastructure/locations/).  The code for a region is the region name with no spaces, **Central US** = **centralus**.

1. In the Azure portal, select **Create a resource**.

2. In **Search the Marketplace**, type **template deployment**, and then press **ENTER**.

3. Select **Template deployment**.

4. Select **Create**.

5. Select **Build your own template in the editor**.

6. Select **Load file**, and then follow the instructions to load the **template.json** file that you downloaded and unzipped in the previous section.

7. In the **template.json** file, name the target search and storage accounts by setting the default value of the search and storage account names. 

8. Edit the **location** property in the **template.json** file to the target region for both your search and storage services. This example sets the target region to `centralus`.

```json
},
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Search/searchServices",
            "apiVersion": "2020-03-13",
            "name": "[parameters('searchServices_target_region_search_name')]",
            "location": "centralus",
            "sku": {
                "name": "standard"
            },
            "properties": {
                "replicaCount": 1,
                "partitionCount": 1,
                "hostingMode": "Default"
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "[parameters('storageAccounts_tagetstorageregion_name')]",
            "location": "centralus",
            "sku": {
                "name": "Standard_RAGRS",
                "tier": "Standard"
            },
```

### Deploy the template

1. Save the **template.json** file.

2. Enter or select the property values:

- **Subscription**: Select an Azure subscription.

- **Resource group**: Select **Create new** and give the resource group a name.

- **Location**: Select an Azure location.

3. Click the **I agree to the terms and conditions stated above** checkbox, and then click the **Select Purchase** button.

## Verifying your services' status in new region

To verify the move, open the new resource group and your services will be listed with the new region.

To move your data from your source region to the target region, please see this article's guidelines for [moving your data to the new storage account](../storage/common/storage-account-move.md?tabs=azure-portal#move-data-to-the-new-storage-account).

## Clean up resources in your original region

To commit the changes and complete the move of your service account, delete the source service account.

## Next steps

[Create an index](./search-get-started-portal.md)

[Create a skillset](./cognitive-search-quickstart-blob.md)

[Create a knowledge store](./knowledge-store-create-portal.md) -->