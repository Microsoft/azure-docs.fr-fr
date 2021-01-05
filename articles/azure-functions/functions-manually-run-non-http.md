---
title: Exécuter manuellement une fonction Azure non déclenchée via HTTP
description: Utiliser une requête HTTP pour exécuter une fonction Azure non déclenchée via HTTP
author: craigshoemaker
ms.topic: article
ms.date: 04/23/2020
ms.author: cshoe
ms.openlocfilehash: 79aebf7ed80fea370ff7a5d5cc40911da4144414
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91537699"
---
# <a name="manually-run-a-non-http-triggered-function"></a>Exécuter manuellement une fonction non déclenchée via HTTP

Cet article explique comment exécuter manuellement une fonction non déclenchée via une requête HTTP spécialement formatée.

Vous pourrez être amené à exécuter « à la demande » une fonction Azure déclenchée indirectement.  Parmi les exemples de déclencheurs indirects figurent les [fonctions selon une planification](./functions-create-scheduled-function.md) ou les fonctions dont l'exécution résulte de l'[action d’une autre ressource](./functions-create-storage-blob-triggered-function.md). 

[Postman](https://www.getpostman.com/) est utilisé dans l’exemple suivant, mais vous pouvez utiliser [cURL](https://curl.haxx.se/), [Fiddler](https://www.telerik.com/fiddler) ou tout autre outil pour envoyer des requêtes HTTP.

## <a name="define-the-request-location"></a>Définir l’emplacement de la requête

Pour exécuter une fonction non déclenchée via HTTP, vous devez trouver un moyen d’envoyer une requête vers Azure. L’URL utilisée pour effectuer cette requête présente une forme spécifique.

![Définir l’emplacement de la requête : nom d'hôte + chemin d’accès du dossier + nom de fonction](./media/functions-manually-run-non-http/azure-functions-admin-url-anatomy.png)

- **Nom d’hôte :** emplacement public de l’application de fonction composé du nom de l’application plus *azurewebsites.net* ou votre domaine personnalisé.
- **Chemin d’accès du dossier :** pour accéder aux fonctions non déclenchées via HTTP via une requête HTTP, vous devez envoyer cette requête via les dossiers *admin/fonctions*.
- **Nom de fonction :** Nom de la fonction que vous souhaitez exécuter.

Vous utilisez cet emplacement de requête dans Postman, de même que la clé principale de la fonction dans la requête vers Azure pour exécuter la fonction.

> [!NOTE]
> Quand vous exécutez localement, la clé principale de la fonction n’est pas nécessaire. Vous pouvez directement [appeler la fonction](#call-the-function) en omettant l’en-tête `x-functions-key`.

## <a name="get-the-functions-master-key"></a>Obtenir la clé principale de la fonction

1. Accédez à votre application de fonction dans le [portail Azure](https://portal.azure.com), sélectionnez **Clés d’application**, puis la clé `_master`. 

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-functions-master-key.png" alt-text="Localisez la clé principale à copier." border="true":::

1. Dans la section **Modifier la clé**, copiez la valeur de clé dans le Presse-papiers, puis sélectionnez **OK**.

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-functions-master-key-copy.png" alt-text="Localisez la clé principale à copier." border="true":::

1. Après avoir copié la *_master*, sélectionnez **Code + Test**, puis sélectionnez **Journaux**. Vous noterez ici la présence de messages émanant de la fonction consignée lors de l'exécution de la fonction à partir de Postman.

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-function-log.png" alt-text="Localisez la clé principale à copier." border="true":::

> [!CAUTION]  
> En raison des autorisations élevées dans votre application de fonction accordées par la clé principale, vous ne devez pas partager celle-ci avec des tiers, ou la distribuer dans une application. La clé ne doit être envoyée qu’à un point de terminaison HTTPS.

## <a name="call-the-function"></a>Appeler la fonction

Ouvrez Postman et procédez comme suit :

1. Entrez l'**emplacement de la requête dans la zone de texte de l'URL**.
1. Vérifiez que la méthode HTTP est définie sur **POST**.
1. Sélectionnez l’onglet **En-têtes**.
1. Tapez **x-fonctions-key** en tant que première clé et collez la clé principale (à partir du Presse-papiers) en tant que valeur.
1. Tapez **Content-Type** comme deuxième clé et tapez **application/json** comme valeur.

    :::image type="content" source="./media/functions-manually-run-non-http/functions-manually-run-non-http-headers.png" alt-text="Localisez la clé principale à copier." border="true":::

1. Sélectionnez l’onglet **Corps** .
1. Tapez **{ "input": "test" }** en tant que corps de la requête.

    :::image type="content" source="./media/functions-manually-run-non-http/functions-manually-run-non-http-body.png" alt-text="Localisez la clé principale à copier." border="true":::

1. Sélectionnez **Envoyer**.
        
    :::image type="content" source="./media/functions-manually-run-non-http/functions-manually-run-non-http-send.png" alt-text="Localisez la clé principale à copier." border="true":::

    Postman indique alors un état **202 Accepté**.

1. Revenez ensuite à votre fonction sur le Portail Azure. Consultez les journaux pour voir les messages émanant de l’appel manuel de la fonction.

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-functions-master-key-logs.png" alt-text="Localisez la clé principale à copier." border="true":::

## <a name="next-steps"></a>Étapes suivantes

- [Stratégies permettant de tester votre code dans Azure Functions](./functions-test-a-function.md)
- [Débogage local lors du déclenchement de fonctions Azure Event Grid](./functions-debug-event-grid-trigger-local.md)
