---
title: Révoquer un abonnement de partage dans Azure Data Share
description: Découvrez comment révoquer un abonnement de partage d’un destinataire à l’aide d’Azure Data Share.
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 07/30/2020
ms.openlocfilehash: 102a0099c95073793ba6a1a85f518c7539327bf3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87511851"
---
# <a name="how-to-revoke-a-consumers-share-subscription-in-azure-data-share"></a>Comment révoquer l’abonnement de partage d’un consommateur dans Azure Data Share

Cet article explique comment révoquer un abonnement de partage à partir d’un ou de plusieurs de vos consommateurs à l’aide d’Azure Data Share. Cette révocation empêche un consommateur de déclencher d’autres instantanés. Si le consommateur concerné par la révocation n’a pas encore déclenché d’instantané, il ne recevra jamais les données une fois l’abonnement de partage révoqué. S’il a déjà déclenché un instantané, les données les plus récentes qu’il obtient seront conservées dans son compte.

## <a name="navigate-to-a-sent-data-share"></a>Accéder à un partage de données envoyé

Dans Azure Data Share, accédez à votre partage envoyé et sélectionnez l’onglet **Abonnements de partage**.

![Révoquer un abonnement de partage](./media/how-to/how-to-revoke-share-subscription/revoke-share-subscription.png) 

Cochez les cases en regard des destinataires dont vous souhaitez supprimer les abonnements de partage, puis cliquez sur **Révoquer**. Le consommateur ne recevra plus les mises à jour pour ses données.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [surveiller vos partages de données](how-to-monitor.md).