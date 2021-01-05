---
title: Modèle de facturation MAU pour les identités externes Azure AD
description: Apprenez-en davantage sur le modèle de facturation des utilisateurs actifs mensuels (Monthly active Users, MAU) des identités externes Azure AD pour la collaboration d’utilisateur invité (B2B) dans Azure AD. Découvrez comment lier votre locataire Azure AD à un abonnement Azure.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 09/21/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.workload: identity
ms.collection: M365-identity-device-management
ms.openlocfilehash: 983a803245467145a0b1161a4495e8045759e7ab
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92442063"
---
# <a name="billing-model-for-azure-ad-external-identities"></a>Modèle de facturation pour les identités externes Azure AD

La tarification des identités externes Azure Active Directory (Azure AD) est basée sur les utilisateurs actifs mensuels (MAU), c’est-à-dire le nombre d’utilisateurs uniques ayant une activité d’authentification au cours d’un mois civil. Ce modèle de facturation s’applique à la fois à la collaboration d’utilisateur invité Azure AD (B2B) et aux [locataires B2C Azure AD](../../active-directory-b2c/billing.md). La facturation MAU contribue à réduire les coûts en offrant un niveau gratuit et une tarification flexible et prévisible. Cet article décrit la facturation MAU et la liaison de vos locataires Azure AD à un abonnement.

> [!IMPORTANT]
> Cet article ne contient pas de détails sur la tarification. Pour obtenir les informations les plus récentes sur les prix et la facturation, consultez [Tarification Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="what-do-i-need-to-do"></a>Que dois-je faire ?

Pour tirer parti de la facturation MAU, votre locataire Azure AD doit être lié à un abonnement Azure.

|Si votre locataire est :  |Vous devez :  |
|---------|---------|
| Un locataire Azure AD lié à un abonnement     | Ne rien faire. Lorsque vous utilisez des fonctionnalités d’identités externes pour collaborer avec des utilisateurs invités, vous êtes automatiquement facturé à l’aide du modèle MAU.        |
| Un locataire Azure AD non encore lié à un abonnement     | [Lier votre locataire Azure AD à un abonnement](#link-your-azure-ad-tenant-to-a-subscription) pour activer la facturation MAU.        |
|  |  |

## <a name="about-monthly-active-users-mau-billing"></a>À propos de la facturation pour des utilisateurs actifs mensuels (MAU)

Dans votre locataire Azure AD, le recours à la collaboration d’utilisateur invité est facturé en fonction du nombre d’utilisateurs invités uniques associé à une activité d’authentification au cours d’un mois civil. Ce modèle remplace le modèle de facturation selon le rapport 1:5, qui autorise jusqu’à cinq utilisateurs invités pour chaque licence de Azure AD Premium disponible dans votre locataire. Lorsque votre locataire est lié à un abonnement et que vous utilisez des fonctionnalités d’identités externes pour collaborer avec des utilisateurs invités, vous êtes automatiquement facturé à l’aide du modèle basé MAU.
  
Le niveau tarifaire qui s’applique à vos utilisateurs invités est basé sur le niveau tarifaire le plus élevé attribué à votre locataire Azure AD. Par exemple, si le niveau tarifaire le plus élevé dans votre locataire est Azure AD Premium P1, le niveau tarifaire Premium P1 s’applique également à vos utilisateurs invités. Si la tarification la plus élevée est Azure AD Free, vous êtes invité à effectuer une mise à niveau vers un niveau tarifaire Premium lorsque vous essayez d’utiliser des fonctionnalités Premium pour des utilisateurs invités.

## <a name="link-your-azure-ad-tenant-to-a-subscription"></a>Lier votre locataire Azure AD à un abonnement

Pour obtenir une facturation et un accès corrects aux fonctionnalités, Un locataire Azure AD doit être lié à un abonnement Azure. Si le répertoire ne dispose pas déjà d’un abonnement auquel vous pouvez le lier, vous avez la possibilité d’en ajouter un au cours de ce processus.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un compte Azure auquel a été attribué au moins le rôle [Contributeur](../../role-based-access-control/built-in-roles.md) au sein de l’abonnement ou d’un groupe de ressources.

2. Sélectionnez le répertoire que vous souhaitez lier : Dans la barre d’outils du portail Azure, sélectionnez l’icône **Annuaire et abonnement** , puis sélectionnez le répertoire.

    ![Sélectionner l’icône Annuaire et abonnement](media/external-identities-pricing/portal-mau-pick-directory.png)

3. Sous **Services Azure** , sélectionnez **Azure Active Directory** .

4. Dans le menu de gauche, sélectionnez **Identités externes** .

5. Sous **Abonnements** , sélectionnez **Abonnements liés** .

6. Dans la liste des locataires, activez la case à cocher en regard du locataire, puis sélectionnez **Lier l’abonnement** .

    ![Sélectionner le locataire et lier un abonnement](media/external-identities-pricing/linked-subscriptions.png)

7. Dans le volet Lier un abonnement, sélectionnez un **Abonnement** , puis un **Groupe de ressources** . Ensuite, sélectionnez **Appliquer** .

   > [!NOTE]
   > Si aucun abonnement n’est répertorié, vous pouvez [associer un abonnement à votre locataire](../fundamentals/active-directory-how-subscriptions-associated-directory.md). Vous pouvez également ajouter un nouvel abonnement en sélectionnant le lien **Si vous n’avez pas encore d’abonnement, vous pouvez en créer un ici** .

    ![Sélectionner un abonnement et un groupe de ressources](media/external-identities-pricing/link-subscription-resource.png)

Une fois ces étapes accomplies, votre abonnement Azure est facturé sur la base des détails de votre contrat Azure Direct ou Enterprise, le cas échéant.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir les informations les plus récentes sur la tarification, consultez les [Tarification Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).