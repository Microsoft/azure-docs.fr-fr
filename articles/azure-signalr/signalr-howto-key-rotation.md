---
title: Comment faire pivoter les clés d’accès pour Azure SignalR Service ?
description: Clarification des raisons pour lesquelles le client doit régulièrement changer les clés d’accès et description de la procédure à suivre dans l’interface utilisateur du portail Azure et dans Azure CLI.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.date: 03/01/2019
ms.author: zhshang
ms.openlocfilehash: 133edc64ac2f858a397a4a184c24497dae8af333
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67565727"
---
# <a name="how-to-rotate-access-key-for-azure-signalr-service"></a>Comment faire pivoter les clés d’accès pour Azure SignalR Service ?

Chaque instance Azure SignalR Service a une paire de clés d’accès appelées clé primaire et clé secondaire. Ces clés servent à authentifier les clients SignalR au moment des demandes au service. Les clés sont associées à l’URL du point de terminaison de l’instance. Protégez vos clés et changez-les régulièrement. Vous disposez de deux clés d’accès, ce qui signifie que vous pouvez maintenir des connexions à l’aide d’une clé pendant que vous regénérez l’autre clé.

## <a name="why-rotate-access-keys"></a>Pourquoi régénérer les clés d’accès ?

Pour des raisons de sécurité et de conformité, vous devez changer régulièrement vos clés d’accès.

## <a name="regenerate-access-keys"></a>Régénération de clés d'accès

1. Accédez au [portail Azure ](https://portal.azure.com/) et connectez-vous à l’aide de vos informations d’identification.

1. Recherchez la section **Clés** dans l’instance Azure SignalR Service avec les clés à regénérer.

1. Sélectionnez **Clés** dans le menu de navigation.

1. Sélectionnez **Régénérer la clé primaire** ou **Régénérer la clé secondaire**.

   Une nouvelle clé est créée et s’affiche avec la chaîne de connexion correspondante.

   ![Régénérer les clés](media/signalr-howto-key-rotation/regenerate-keys.png)

Vous pouvez également regénérer les clés à l’aide d’[Azure CLI](/cli/azure/signalr/key?view=azure-cli-latest#az-signalr-key-renew).

## <a name="update-configurations-with-new-connection-strings"></a>Mettre à jour les configurations avec les nouvelles chaînes de connexion

1. Copiez la chaîne de connexion que vous venez de générer.

1. Mettez à jour toutes les configurations pour utiliser la nouvelle chaîne de connexion.

1. Redémarrez l’application si besoin.

## <a name="forced-access-key-regeneration"></a>Régénération forcée des clés d’accès

Azure SignalR Service peut imposer une regénération des clés d’accès dans certaines situations. Le service en informe alors les clients par e-mail et via une notification sur le portail. Si vous recevez ce message ou si vous rencontrez un problème de clé d’accès au niveau du service, regénérez les clés en suivant ce guide.

## <a name="next-steps"></a>Étapes suivantes

Changez régulièrement les clés d’accès pour des raisons de sécurité.

Ce guide vous a expliqué comment regénérer des clés d’accès. Passez aux didacticiels suivants pour en savoir plus sur l’authentification avec OAuth ou Azure Functions.

> [!div class="nextstepaction"]
> [Intégrer avec une identité ASP.NET Core](./signalr-concept-authenticate-oauth.md)

> [!div class="nextstepaction"]
> [Générer une application serverless en temps réel avec l’authentification](./signalr-tutorial-authenticate-azure-functions.md)