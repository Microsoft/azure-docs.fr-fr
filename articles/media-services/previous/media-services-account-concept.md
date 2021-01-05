---
title: Gérer les comptes Azure Media Services v2 | Microsoft Docs
description: Pour commencer à gérer, chiffrer, coder, analyser, gérer et diffuser en continu du contenu multimédia dans Azure, vous devez créer un compte Media Services. Cet article explique comment gérer les comptes Azure Media Services v2.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 07/05/2019
ms.author: juliako
ms.openlocfilehash: 167028cda6962445373b22c5f3d0a9e56c3e608a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89255408"
---
# <a name="manage-azure-media-services-v2-accounts"></a>Gérer les comptes Azure Media Services v2

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Pour commencer à gérer, chiffrer, coder, analyser, gérer et diffuser en continu du contenu multimédia dans Azure, vous devez créer un compte Media Services. Lorsque vous créez un compte Media Services, vous devez indiquer le nom d’une ressource de compte de stockage Azure. Le compte de stockage spécifié est lié à votre compte Media Services. Le compte Media Services et tous les comptes de stockage associés doivent être faire partie du même abonnement Azure.  

## <a name="moving-a-media-services-account-between-subscriptions"></a>Déplacement d’un compte Media Services entre abonnements

Si vous devez déplacer un compte Media Services vers un nouvel abonnement, vous devez d’abord déplacer le groupe de ressources complet qui contient le compte Media Services vers le nouvel abonnement. Vous devez déplacer toutes les ressources jointes : comptes de stockage Azure, profils Azure CDN, etc. Pour plus d’informations, consultez la page [Déplacement de ressources vers un nouveau groupe de ressources ou un abonnement](../../azure-resource-manager/management/move-resource-group-and-subscription.md). Comme avec toutes les ressources dans Azure, les déplacements de groupes de ressources peuvent prendre un certain temps.

Media Services v2 ne prend pas en charge le modèle d’architecture multilocataire. Si vous devez déplacer un compte Media Services vers un abonnement dans un nouveau locataire, créez une application Azure Active Directory (Azure AD) dans le nouveau locataire. Ensuite, déplacez votre compte vers l’abonnement dans le nouveau locataire. Une fois terminé le déplacement dans le locataire, vous pouvez commencer à utiliser une application Azure AD à partir du nouveau locataire pour accéder au compte Media Services à l’aide de l’API v2.

> [!IMPORTANT]
> Vous devez réinitialiser les informations relatives à l’[authentification Azure AD](media-services-portal-get-started-with-aad.md) pour accéder à l’API Media Services v2.
  
### <a name="considerations"></a>Considérations

* Créez des sauvegardes de toutes les données de votre compte avant la migration vers un autre abonnement.
* Vous devez arrêter tous les points de terminaison de streaming et toutes les ressources de streaming en direct. Vos utilisateurs ne seront pas en mesure d’accéder à votre contenu pendant la durée du déplacement du groupe de ressources.

> [!IMPORTANT]
> Ne démarrez pas le point de terminaison de streaming tant que le déplacement ne s’est pas terminé correctement.

### <a name="troubleshoot"></a>Dépanner

Si un compte Media Services ou un compte de stockage Azure associé devient « Déconnecté » suite au déplacement du groupe de ressources, essayez d’utiliser la rotation des clés de compte de stockage. Si la rotation des clés de compte de stockage ne résout pas l’état « Déconnecté » du compte Media Services, envoyez une nouvelle demande de support à partir du menu « Support + dépannage » dans le compte Media Services.  

## <a name="next-steps"></a>Étapes suivantes

[Créer un compte](media-services-portal-create-account.md)
