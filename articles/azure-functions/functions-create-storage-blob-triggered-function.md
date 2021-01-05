---
title: Créer une fonction dans Azure déclenchée par le stockage Blob
description: Utilisez Azure Functions pour créer une fonction serverless appelée par les éléments ajoutés au conteneur de stockage blob.
ms.assetid: d6bff41c-a624-40c1-bbc7-80590df29ded
ms.topic: how-to
ms.date: 10/01/2018
ms.custom: mvc, cc996988-fb4f-47
ms.openlocfilehash: bf6865d2756579f457dded90b247326d2eec137c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83122988"
---
# <a name="create-a-function-in-azure-thats-triggered-by-blob-storage"></a>Créer une fonction dans Azure qui est déclenchée par le stockage Blob

Apprenez à créer une fonction déclenchée lorsque des fichiers sont chargés dans ou mis à jour dans un conteneur de stockage blob.

## <a name="prerequisites"></a>Prérequis

+ Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-an-azure-function-app"></a>Création d’une application Azure Function

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

Vous avez correctement créé votre application de fonction.

:::image type="content" source="./media/functions-create-storage-blob-triggered-function/function-app-create-success.png" alt-text="Function App créée avec succès." border="true":::

Créez ensuite une fonction dans la nouvelle Function App.

<a name="create-function"></a>

## <a name="create-an-azure-blob-storage-triggered-function"></a>Créer une fonction déclenchée par le stockage Azure Blob

1. Sélectionnez **Fonctions**, puis **+ Ajouter** pour ajouter une nouvelle fonction.

   :::image type="content" source="./media/functions-create-storage-blob-triggered-function/function-app-quickstart-choose-template.png" alt-text="Function App créée avec succès." border="true":::

1. Choisissez le modèle **déclencheur de Stockage d’objets blob Azure**.

1. Utilisez les paramètres spécifiés dans le tableau sous l’image.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-create-blob-storage-trigger-portal-2.png" alt-text="Function App créée avec succès." border="true":::

    | Paramètre | Valeur suggérée | Description |
    |---|---|---|
    | **Nouvelle fonction** | Unique dans votre Function App | Nom de cette fonction déclenchée par l’objet Blob. |
    | **Chemin d’accès**   | samples-workitems/{name}    | Emplacement du stockage Blob analysé. Le nom de fichier de l’objet Blob est transmis dans la liaison en tant que paramètre _name_.  |
    | **Connexion au compte de stockage** | AzureWebJobsStorage | Vous pouvez utiliser la connexion de compte de stockage qui est déjà utilisée par votre application de fonction, ou créez-en une.  |

1. Sélectionnez **Créer une fonction** pour créer votre fonction.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-create-blob-storage-trigger-portal-3.png" alt-text="Function App créée avec succès." border="true":::

Créez ensuite le conteneur **samples-workItems**.

## <a name="create-the-container"></a>Créer le conteneur

1. Dans votre fonction, sur la page **Vue d’ensemble**, sélectionnez votre groupe de ressources.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-resource-group.png" alt-text="Function App créée avec succès." border="true":::

1. Recherchez et sélectionnez le compte de stockage de votre groupe de ressources.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-account-access.png" alt-text="Function App créée avec succès." border="true":::

1. Choisissez **Conteneurs**, puis **+ Conteneur**. 

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-add-container.png" alt-text="Function App créée avec succès." border="true":::

1. Dans le champ **Nom**, tapez `samples-workitems`, puis sélectionnez **Créer**.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-name-blob-container.png" alt-text="Function App créée avec succès." border="true":::

Une fois que vous avez un conteneur d’objets blob, vous pouvez tester la fonction en chargeant un fichier dans le conteneur.

## <a name="test-the-function"></a>Tester la fonction

1. Dans le portail Azure, accédez à votre fonction, développez les **Journaux d’activité** en bas de la page et vérifiez que la diffusion de journaux d’activité n’est pas suspendue.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-log-expander.png" alt-text="Function App créée avec succès." border="true":::

1. Dans une fenêtre de navigateur distincte, accédez à votre groupe de ressources dans le Portail Azure, puis sélectionnez le compte de stockage.

1. Sélectionnez **Conteneurs**, puis sélectionnez le conteneur **samples-workitems**.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-container.png" alt-text="Function App créée avec succès." border="true":::

1. Sélectionnez **Charger**, puis sélectionnez l’icône de dossier pour choisir un fichier à charger.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/functions-storage-manager-upload-file-blob.png" alt-text="Function App créée avec succès." border="true":::

1. Accédez à un fichier sur votre ordinateur local, tel qu’un fichier image, puis choisissez le fichier. Sélectionnez **Ouvrir**, puis **Charger**.

1. Revenez à vos journaux d’activité de fonction et vérifiez que l’objet blob a été lu.

    :::image type="content" source="./media/functions-create-storage-blob-triggered-function/function-app-in-portal-editor.png" alt-text="Function App créée avec succès." border="true":::

    >[!NOTE]
    > Lorsque votre Function App s’exécute dans le plan de consommation par défaut, il peut s’écouler un délai de plusieurs minutes entre l’ajout ou la mise à jour de l’objet blob et le déclenchement de la fonction. Si vous exigez une faible latence pour vos fonctions déclenchées par des objets Blob, exécutez plutôt votre Function App dans un plan App Service.

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [Next steps note](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Étapes suivantes

Vous avez créé une fonction qui s’exécute lorsqu’un objet blob est ajouté ou mis à jour dans le stockage Blob. Pour plus d’informations sur les déclencheurs de stockage Blob, consultez [Liaisons de stockage Blob Azure Functions](functions-bindings-storage-blob.md).

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps.md)]
