---
title: Gérer les comptes Azure Media Services v3 | Microsoft Docs
description: Pour commencer à gérer, chiffrer, coder, analyser, gérer et diffuser en continu du contenu multimédia dans Azure, vous devez créer un compte Media Services. Cet article explique comment gérer les comptes Azure Media Services v3.
services: media-services
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 11/05/2020
ms.author: inhenkel
ms.openlocfilehash: 77453aa2d8989a0542beffb1c88ebe7ee53d1506
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "96008325"
---
# <a name="manage-azure-media-services-v3-accounts"></a>Gérer les comptes Azure Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Pour commencer à gérer, chiffrer, coder, analyser, gérer et diffuser en continu du contenu multimédia dans Azure, vous devez créer un compte Media Services. Lorsque vous créez un compte Media Services, vous devez indiquer le nom d’une ressource de compte de stockage Azure. Le compte de stockage spécifié est lié à votre compte Media Services. Le compte Media Services et tous les comptes de stockage associés doivent être faire partie du même abonnement Azure. Pour plus d’informations, consultez [Comptes de stockage](storage-account-concept.md).

[!INCLUDE [account creation note](./includes/note-2020-05-01-account-creation.md)]

## <a name="moving-a-media-services-account-between-subscriptions"></a>Déplacement d’un compte Media Services entre abonnements 

Si vous devez déplacer un compte Media Services vers un nouvel abonnement, vous devez d’abord déplacer le groupe de ressources complet qui contient le compte Media Services vers le nouvel abonnement. Vous devez déplacer toutes les ressources jointes : comptes de stockage Azure, profils Azure CDN, etc. Pour plus d’informations, consultez la page [Déplacement de ressources vers un nouveau groupe de ressources ou un abonnement](../../azure-resource-manager/management/move-resource-group-and-subscription.md). Comme avec toutes les ressources dans Azure, les déplacements de groupes de ressources peuvent prendre un certain temps.

> [!NOTE]
> Media Services v3 prend en charge le modèle mutualisé.

### <a name="considerations"></a>Considérations

* Créez des sauvegardes de toutes les données de votre compte avant la migration vers un autre abonnement.
* Vous devez arrêter tous les points de terminaison de streaming et toutes les ressources de streaming en direct. Vos utilisateurs ne seront pas en mesure d’accéder à votre contenu pendant la durée du déplacement du groupe de ressources. 

> [!IMPORTANT]
> Ne démarrez pas le point de terminaison de streaming tant que le déplacement ne s’est pas terminé correctement.

### <a name="troubleshoot"></a>Dépanner

Si un compte Media Services ou un compte de stockage Azure associé devient « Déconnecté » suite au déplacement du groupe de ressources, essayez d’utiliser la rotation des clés de compte de stockage. Si la rotation des clés de compte de stockage ne résout pas l’état « Déconnecté » du compte Media Services, envoyez une nouvelle demande de support à partir du menu « Support + dépannage » dans le compte Media Services.  

## <a name="next-steps"></a>Étapes suivantes

[Créer un compte](./create-account-howto.md)
