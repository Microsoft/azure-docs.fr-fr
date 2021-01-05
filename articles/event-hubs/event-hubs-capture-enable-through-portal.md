---
title: Event Hubs – Capturer des événements de streaming avec le portail Azure
description: Cet article décrit comment activer la capture d’événements diffusés en continu par le biais d’Azure Event Hubs à l’aide du portail Azure.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 2381bfa627d00a78ed91af0ba81579588ee016ce
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91613575"
---
# <a name="enable-capturing-of-events-streaming-through-azure-event-hubs"></a>Activer la capture d’événements diffusés en continu par le biais d’Azure Event Hubs

Azure [Event Hubs Capture][capture-overview] vous permet de fournir automatiquement les données de streaming d’Event Hubs à un compte [Stockage Blob Azure](https://azure.microsoft.com/services/storage/blobs/) ou [Azure Data Lake Storage Gen1 ou Gen2](https://azure.microsoft.com/services/data-lake-store/) de votre choix.

Vous pouvez configurer la fonctionnalité Capture lors de la création du concentrateur d’événements, à l’aide du [portail Azure](https://portal.azure.com). Vous pouvez capturer les données dans un conteneur [Stockage Blob](https://azure.microsoft.com/services/storage/blobs/) Azure ou dans un compte [Azure Data Lake Storage Gen1 ou Gen2](https://azure.microsoft.com/services/data-lake-store/).

Pour plus d’informations, consultez la section [Vue d’ensemble d’Event Hubs Capture][capture-overview].

> [!IMPORTANT]
> Le compte de stockage de destination (Stockage Azure ou Azure Data Lake Storage) doit faire partie du même abonnement que le hub d’événements.

## <a name="capture-data-to-azure-storage"></a>Capturer des données dans Stockage Azure

Lors de la création d’un concentrateur d’événement, vous pouvez activer la fonctionnalité Capture en cliquant sur le bouton **Activé** sur la page **Créer un concentrateur d’événement** du portail. Précisez ensuite le compte et le conteneur de stockage à utiliser en cliquant sur **Stockage Azure** dans la zone **Fournisseur de capture**. Comme Event Hubs Capture utilise l’authentification de service à service avec le stockage, il est inutile de spécifier une chaîne de connexion de stockage. Le sélecteur de ressources sélectionne automatiquement l’URI de ressource pour votre compte de stockage. Si vous utilisez Azure Resource Manager, vous devez fournir explicitement cet URI en tant que chaîne.

La fenêtre temporelle par défaut est de cinq minutes. La valeur minimum est 1, la valeur maximum 15. La fenêtre **Taille** présente une plage de 10 à 500 Mo.

![Fenêtre de temps pour la capture][1]

> [!NOTE]
> Vous pouvez activer ou désactiver l’envoi de fichiers vides lorsqu’aucun événement ne se produit pendant la fenêtre de capture. 

## <a name="capture-data-to-azure-data-lake-storage-gen-2"></a>Capturer des données dans Azure Data Lake Storage Gen2 

1. Pour créer un compte Stockage Azure, consultez [Créer un compte de stockage](../storage/common/storage-account-create.md?tabs=azure-portal#create-a-storage-account). Affectez la valeur **Activé** à **Espace de noms hiérarchique** sous l’onglet **Avancé** pour en faire un compte Azure Data Lake Storage Gen2.
2. Lors de la création d’un hub d’événements, effectuez les étapes suivantes : 

    1. Sélectionnez **Activé** pour **Capture**. 
    2. Sélectionnez **Stockage Azure** comme fournisseur de capture. L’option **Azure Data Lake Store** que vous voyez pour le **Fournisseur de capture** concerne Azure Data Lake Storage Gen1. Pour utiliser Azure Data Lake Storage Gen2, sélectionnez **Stockage Azure**.
    2. Sélectionnez le bouton **Sélectionner un conteneur**. 

        ![Activer la capture pour Data Lake Storage Gen2](./media/event-hubs-capture-enable-through-portal/data-lake-storage-gen2.png)
3. Sélectionnez le compte **Azure Data Lake Storage Gen2** dans la liste. 

    ![Sélectionnez Data Lake Storage Gen2](./media/event-hubs-capture-enable-through-portal/select-data-lake-storage-gen2.png)
4. Sélectionnez le **conteneur** (système de fichiers dans Data Lake Storage Gen2).

    ![Sélectionner le système de fichiers dans le stockage](./media/event-hubs-capture-enable-through-portal/select-file-system-data-lake-storage.png)
5. Dans la page **Créer un Event Hub**, sélectionnez **Créer**. 

    ![Sélectionnez le bouton Créer](./media/event-hubs-capture-enable-through-portal/create-event-hub-data-lake-storage.png)

    > [!NOTE]
    > Le conteneur que vous créez dans Azure Data Lake Storage Gen2 à l’aide de cette interface utilisateur est affiché sous **Systèmes de fichiers** dans l’**Explorateur Stockage**. De même, le système de fichiers que vous créez dans un compte Data Lake Storage Gen2 s’affiche sous la forme d’un conteneur dans cette interface utilisateur. 


## <a name="capture-data-to-azure-data-lake-storage-gen-1"></a>Capturer des données dans Azure Data Lake Storage Gen1 

Pour capturer des données dans Azure Data Lake Storage Gen1, vous devez créer un compte Data Lake Storage Gen1 et un hub d’événements :

### <a name="create-an-azure-data-lake-storage-gen-1-account-and-folders"></a>Créer un compte et des dossiers Azure Data Lake Storage Gen1

1. Pour créer un compte Data Lake Storage, suivez les instructions dans [Bien démarrer avec Azure Data Lake Storage Gen1 à l’aide du Portail Azure](../data-lake-store/data-lake-store-get-started-portal.md).
2. Suivez les instructions de la section [Affecter des autorisations à Event Hubs](../data-lake-store/data-lake-store-archive-eventhub-capture.md#assign-permissions-to-event-hubs) pour créer un dossier au sein du compte Data Lake Storage Gen1 dans lequel vous souhaitez capturer les données Event Hubs, puis affecter des autorisations à Event Hubs de sorte qu’il puisse écrire des données dans votre compte Data Lake Storage Gen1.  


### <a name="create-an-event-hub"></a>Créer un hub d’événements

1. Le hub d’événements doit se trouver dans le même abonnement Azure que le compte Azure Data Lake Storage Gen1 que vous venez de créer. Pour créer le concentrateur d’événements, cliquez sur le bouton **Activé** dans **Capture** sur la page **Créer un concentrateur d’événements** du portail. 
2. Sur la page **Créer un concentrateur d’événements** du portail, sélectionnez **Azure Data Lake Store** dans la zone **Fournisseur de capture**.
3. Dans **Sélectionner le Store** en regard de **Data Lake Store**, spécifiez le compte Data Lake Storage Gen1 que vous avez créé puis, dans le champ **Chemin de Data Lake**, entrez le chemin du dossier de données que vous avez créé.

    ![Sélectionner un compte Data Lake Storage][3]


## <a name="add-or-configure-capture-on-an-existing-event-hub"></a>Ajouter ou configurer la fonctionnalité Capture sur un concentrateur d’événements existant

Vous pouvez configurer la fonctionnalité Capture sur des concentrateurs d’événements se trouvant dans des espaces de noms Event Hubs. Pour activer Capture sur un hub d’événements existant, ou pour modifier les paramètres de cette fonctionnalité, cliquez sur l’espace de noms afin de charger l’écran de vue d’ensemble, puis cliquez sur le hub d’événements pour lequel vous souhaitez activer ou modifier la configuration de Capture. Pour finir, cliquez sur l’option **Capture** à gauche du panneau ouvert, puis modifiez les paramètres de Capture, comme indiqué dans la figure suivante :

### <a name="azure-blob-storage"></a>Stockage Blob Azure

![Configurer Stockage Blob Azure][2]

### <a name="azure-data-lake-storage-gen-2"></a>Azure Data Lake Storage Gen 2

![Configurer Azure Data Lake Storage Gen2](./media/event-hubs-capture-enable-through-portal/configure-data-lake-storage-gen2.png)

### <a name="azure-data-lake-storage-gen-1"></a>Azure Data Lake Storage Gen 1 

![Configurer Azure Data Lake Storage][4]

[1]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture1.png
[2]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture2.png
[3]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture3.png
[4]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture4.png

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur Event Hubs Capture, consultez [Vue d’ensemble d’Event Hubs Capture][capture-overview].
- Vous pouvez également configurer Event Hubs Capture via des modèles Azure Resource Manager. Pour en savoir plus, consultez la section relative à [l’activation de Capture à l’aide d’un modèle Azure Resource Manager](event-hubs-resource-manager-namespace-event-hub-enable-capture.md).
- [Découvrez comment créer un abonnement Azure Event Grid en utilisant un espace de noms Event Hubs comme source](store-captured-data-data-warehouse.md)
- [Prise en main d’Azure Data Lake Store à l’aide du portail Azure](../data-lake-store/data-lake-store-get-started-portal.md)

[capture-overview]: event-hubs-capture-overview.md
