---
title: Créer une fonction dans Azure déclenchée par des messages de file d’attente
description: Utilisez Azure Functions pour créer une fonction serverless appelée par un message soumis à une file d’attente dans Azure.
ms.assetid: 361da2a4-15d1-4903-bdc4-cc4b27fc3ff4
ms.topic: how-to
ms.date: 10/01/2018
ms.custom: mvc, cc996988-fb4f-47
ms.openlocfilehash: d722d420597bb459d3e7b6d2ca33fdc49bfe6f09
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90981572"
---
# <a name="create-a-function-triggered-by-azure-queue-storage"></a>Créer une fonction déclenchée par une file d’attente de stockage Azure

Découvrez comment créer une fonction qui est déclenchée lorsque des messages sont envoyés à une file d’attente de stockage Azure.

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-an-azure-function-app"></a>Création d’une application Azure Function

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

   :::image type="content" source="./media/functions-create-storage-queue-triggered-function/function-app-create-success.png" alt-text="Function App créée avec succès." border="true":::

Créez ensuite une fonction dans la nouvelle Function App.

<a name="create-function"></a>

## <a name="create-a-queue-triggered-function"></a>Créer une fonction déclenchée par une file d’attente

1. Sélectionnez **Fonctions**, puis **+ Ajouter** pour ajouter une nouvelle fonction.

   :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-app-quickstart-choose-template.png" alt-text="Function App créée avec succès." border="true":::

1. Choisissez le **modèle déclencheur de file d’attente de stockage**.

1. Utilisez les paramètres spécifiés dans le tableau sous l’image.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-create-queue-storage-trigger-portal.png" alt-text="Function App créée avec succès." border="true":::


    | Paramètre | Valeur suggérée | Description |
    |---|---|---|
    | **Nom** | Unique dans votre Function App | Nom de cette fonction déclenchée par la file d’attente. |
    | **Nom de la file d’attente**   | éléments myqueue    | Le nom de la file d’attente à connecter à votre compte de stockage. |
    | **Connexion au compte de stockage** | AzureWebJobsStorage | Vous pouvez utiliser la connexion de compte de stockage qui est déjà utilisée par votre application de fonction, ou créez-en une.  |    

1. Sélectionnez **Créer une fonction** pour créer votre fonction.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-create-queue-storage-trigger-portal-3.png" alt-text="Function App créée avec succès." border="true":::

Ensuite, connectez-vous à votre compte de stockage Azure et créez la file d’attente de stockage **myqueue-items**.

## <a name="create-the-queue"></a>Créer la file d’attente

1. Dans votre fonction, sur la page **Vue d’ensemble**, sélectionnez votre groupe de ressources.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-resource-group.png" alt-text="Function App créée avec succès." border="true":::

1. Recherchez et sélectionnez le compte de stockage de votre groupe de ressources.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-account-access.png" alt-text="Function App créée avec succès." border="true":::

1. Choisissez **Files d’attente**, puis **+ File d’attente**. 

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-add-queue.png" alt-text="Function App créée avec succès." border="true":::

1. Dans le champ **Nom**, tapez `myqueue-items`, puis sélectionnez **Créer**.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-name-queue.png" alt-text="Function App créée avec succès." border="true":::

Maintenant que vous disposez d’une file d’attente de stockage, vous pouvez tester la fonction en ajoutant un message à la file d’attente.

## <a name="test-the-function"></a>Tester la fonction

1. Dans le portail Azure, accédez à votre fonction, développez les **Journaux d’activité** en bas de la page et vérifiez que la diffusion de journaux d’activité n’est pas suspendue.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-queue-storage-log-expander.png" alt-text="Function App créée avec succès." border="true":::

1. Dans une fenêtre de navigateur distincte, accédez à votre groupe de ressources dans le Portail Azure, puis sélectionnez le compte de stockage.

1. Sélectionnez **Files d’attente**, puis sélectionnez le conteneur **myQueue-items**.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-queue.png" alt-text="Function App créée avec succès." border="true":::

1. Sélectionnez **Ajouter un message**, puis tapez « Hello World ! » dans **Texte du message**. Sélectionnez **OK**.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/functions-storage-queue-test.png" alt-text="Function App créée avec succès." border="true":::

1. Attendez quelques secondes, puis retournez à vos journaux d’activité de fonction et vérifiez que le nouveau message a été lu à partir de la file d’attente.

    :::image type="content" source="./media/functions-create-storage-queue-triggered-function/function-app-in-portal-editor.png" alt-text="Function App créée avec succès." border="true":::

1. De retour dans votre file d’attente de stockage, sélectionnez **Actualiser** et vérifiez que le message a été traité et qu’il ne se trouve plus dans la file d’attente.

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [Next steps note](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Étapes suivantes

Vous avez créé une fonction qui s’exécute lorsqu’un message est ajouté à une file d’attente de stockage. Pour en savoir plus sur les déclencheurs de stockage en file d’attente, consultez la page [Liaisons de file d’attente de stockage Azure Functions](functions-bindings-storage-queue.md).

Maintenant que vous avez créé votre première fonction, nous allons ajouter une liaison de sortie à la fonction qui écrit un message dans une autre file d’attente.

> [!div class="nextstepaction"]
> [Ajouter des messages à une file d’attente de stockage Azure à l’aide de Functions](functions-integrate-storage-queue-output-binding.md)
