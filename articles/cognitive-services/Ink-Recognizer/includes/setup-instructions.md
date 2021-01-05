---
author: aahill
ms.author: aahi
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: include
ms.date: 06/20/2019
ms.openlocfilehash: c68e5b7ab24e2d7e7f30ddc356ae3c4382137507
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369091"
---
>[!NOTE]
> Les points de terminaison pour les ressources créées après le 1er juillet 2019 utilisent le format de sous-domaine personnalisé indiqué ci-dessous. Pour obtenir plus d’informations et une liste complète des points de terminaison régionaux, consultez [Noms de sous-domaines personnalisés pour Cognitive Services](../../cognitive-services-custom-subdomains.md). 

Les services Azure Cognitive Services sont représentés par des ressources Azure auxquelles vous vous abonnez. Créez une ressource pour Ink Recognizer à l’aide du [portail Azure](../../cognitive-services-apis-create-account.md).

Après avoir créé une ressource, obtenez votre point de terminaison et votre clé en ouvrant votre ressource dans le [portail Azure](https://ms.portal.azure.com#blade/HubsExtension/BrowseResourceGroupBlade) puis en cliquant sur **Démarrage rapide**.

Créez deux [variables d’environnement](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) :

* `INK_RECOGNITION_SUBSCRIPTION_KEY` : la clé d’abonnement pour l’authentification de vos requêtes. 

* `INK_RECOGNITION_ENDPOINT` : le point de terminaison de votre ressource. Il se présente comme suit : <br> `https://<your-custom-subdomain>.api.cognitive.microsoft.com`