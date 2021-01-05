---
author: baanders
description: Fichier include relatif aux prérequis des autorisations dans le programme d’installation d’Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 10/14/2020
ms.author: baanders
ms.openlocfilehash: 919cc934920d922a82ceb9e30630006627af44e9
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92205480"
---
## <a name="prerequisites-permission-requirements"></a>Configuration requise : Spécifications relatives aux autorisations

Pour être en mesure d’effectuer toutes les étapes décrites dans cet article, vous devez disposer d’un [rôle dans votre abonnement](../articles/role-based-access-control/rbac-and-directory-admin-roles.md) qui dispose des autorisations suivantes :
* Créer et gérer des ressources Azure
* Gérer l’accès des utilisateurs aux ressources Azure (y compris l’octroi et la délégation des autorisations)

Les rôles communs qui répondent à cette exigence sont *Propriétaire* , *Administrateur de compte* ou la combinaison des rôles *Administrateur de l’accès utilisateur* et *Contributeur* . Pour obtenir une explication complète des rôles et des autorisations, notamment les autorisations qui sont incluses avec d’autres rôles, consultez l’article [*Rôles d’administrateur d’abonnement classique, rôles Azure et rôles Azure AD*](../articles/role-based-access-control/rbac-and-directory-admin-roles.md) dans la documentation relative à Azure RBAC.

Pour consulter votre rôle dans l’abonnement, accédez à la page [Abonnements](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) du portail Azure (vous pouvez utiliser ce lien ou rechercher *Abonnements* dans la barre de recherche du portail). Recherchez le nom de l’abonnement que vous utilisez et affichez votre rôle dans la colonne *Mon rôle*  :

:::image type="content" source="../articles/digital-twins/media/how-to-set-up-instance/portal/subscriptions-role.png" alt-text="Vue de la page Abonnements dans le portail Azure, affichant l’utilisateur en tant que propriétaire" lightbox="../articles/digital-twins/media/how-to-set-up-instance/portal/subscriptions-role.png":::

Si vous constatez que la valeur est *Contributeur* ou un autre rôle qui ne dispose pas des autorisations requises décrites ci-dessus, vous pouvez contacter l’utilisateur de votre abonnement qui *en dispose* (par exemple, le propriétaire de l’abonnement ou l’administrateur du compte) et procéder de l’une des façons suivantes :
* Demandez-lui d’effectuer les étapes de cet article en votre nom.
* Demandez qu’il élève votre rôle sur l’abonnement pour que vous disposiez des autorisations nécessaires pour continuer la procédure. Le fait que cela soit approprié dépend de votre organisation et de votre rôle au sein de celle-ci.