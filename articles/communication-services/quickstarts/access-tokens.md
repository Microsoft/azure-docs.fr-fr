---
title: 'Démarrage rapide : Créer et gérer des jetons d’accès'
titleSuffix: An Azure Communication Services quickstart
description: Découvrez comment gérer des identités et des jetons d’accès à l’aide de la bibliothèque de client Azure Communication Services Administration.
author: tomaschladek
manager: nmurav
services: azure-communication-services
ms.author: tchladek
ms.date: 08/20/2020
ms.topic: quickstart
ms.service: azure-communication-services
zone_pivot_groups: acs-js-csharp-java-python
ms.openlocfilehash: 246b8041e7e12ea1a1391cb4690d02892bc31b63
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325291"
---
# <a name="quickstart-create-and-manage-access-tokens"></a>Démarrage rapide : Créer et gérer des jetons d’accès

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

Commencez avec Azure Communication Services en utilisant la bibliothèque de client Communication Services Administration. Elle vous permet de créer des identités et de gérer vos jetons d’accès. L’identité représente une entité de votre application dans la solution Azure Communication Service (par exemple, un utilisateur ou un appareil). Les jetons d’accès permettent aux bibliothèques de client Chat et Calling de s’authentifier directement auprès d’Azure Communication Services. Nous vous recommandons de générer des jetons d’accès sur un service côté serveur. Les jetons d’accès sont ensuite utilisés pour initialiser les bibliothèques de client Communication Services sur les appareils clients.

Les tarifs indiqués sur les images tout au long de ce tutoriel sont fournis à des fins de démonstration uniquement.

::: zone pivot="programming-language-csharp"
[!INCLUDE [.NET](./includes/user-access-token-net.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript](./includes/user-access-token-js.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python](./includes/user-access-token-python.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Java](./includes/user-access-token-java.md)]
::: zone-end

La sortie de l’application décrit chaque action terminée :
<!---cSpell:disable --->
```console
Azure Communication Services - Access Tokens Quickstart

Created an identity: 8:acs:4ccc92c8-9815-4422-bddc-ceea181dc774_00000006-19e0-2727-80f5-8b3a0d003502

Issued an access token with 'voip' scope that expires at Fri Nov 27 2020 16:47:05 GMT-0800 (Pacific Standard Time):
<token signature here>

Successfully revoked all access tokens for identity with ID: 8:acs:4ccc92c8-9815-4422-bddc-ceea181dc774_00000006-19e0-2727-80f5-8b3a0d003502

Deleted the identity with ID: 8:acs:4ccc92c8-9815-4422-bddc-ceea181dc774_00000006-19e0-2727-80f5-8b3a0d003502
```
<!---cSpell:enable --->

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez nettoyer et supprimer un abonnement Communication Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées. Apprenez-en davantage sur le [nettoyage des ressources](./create-communication-resource.md#clean-up-resources).


## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez appris comment :

> [!div class="checklist"]
> * Gérer des identités
> * Émettre des jetons d’accès
> * Utiliser la bibliothèque de client Communication Services Administration


> [!div class="nextstepaction"]
> [Ajouter Voice calling à votre application](./voice-video-calling/getting-started-with-calling.md)

Vous voudrez peut-être aussi :

 - [Découvrir l’authentification](../concepts/authentication.md)
 - [Ajouter un système de conversation à votre application](./chat/get-started.md)
 - [Découvrir l’architecture client et serveur](../concepts/client-and-server-architecture.md)
 