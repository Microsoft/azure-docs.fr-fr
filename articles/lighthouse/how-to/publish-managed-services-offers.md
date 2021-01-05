---
title: Publier une offre de services gérés sur la place de marché Azure
description: Découvrez comment publier une offre de service géré qui intègre les clients à Azure Lighthouse.
ms.date: 08/18/2020
ms.topic: how-to
ms.openlocfilehash: 033003d7e782ca0e99b1fc908c5261b6e31bf613
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023923"
---
# <a name="publish-a-managed-service-offer-to-azure-marketplace"></a>Publier une offre de services gérés sur la place de marché Azure

Dans cet article, vous allez découvrir comment publier une offre de service managé publique ou privée sur la [Place de marché Azure](https://azuremarketplace.microsoft.com) à l’aide du programme [Place de marché commerciale](../../marketplace/overview.md) dans l’Espace partenaires. Les clients qui achètent l’offre délèguent ensuite des abonnements ou des groupes de ressources, ce qui vous permet de les gérer avec [Azure Lighthouse](../overview.md).

## <a name="publishing-requirements"></a>Exigences de publication

Pour créer et publier des offres, vous devez disposer d’un [compte valide dans l’Espace partenaires](../../marketplace/partner-center-portal/create-account.md). Si vous n’avez pas encore de compte, le [processus d’inscription](https://aka.ms/joinmarketplace) vous guide lors des étapes de création de compte dans l’Espace partenaires et d’inscription au programme de la Place de marché commerciale.

Conformément aux [exigences de certification des offres de services managés](/legal/marketplace/certification-policies#7004-business-requirements), vous devez disposer d’un [niveau de compétence de plateforme cloud Silver ou Gold](/partner-center/learn-about-competencies) ou être [Fournisseur de services managés Azure Expert](https://partner.microsoft.com/membership/azure-expert-msp) pour publier une offre de services managés.

Votre ID Microsoft Partner Network (MPN) est [automatiquement associé](../../cost-management-billing/manage/link-partner-id.md) aux offres que vous publiez pour suivre votre impact sur les engagements client.

Si vous ne souhaitez pas publier d’offre sur la Place de marché Azure, ou si vous ne répondez pas à toutes les exigences, vous pouvez intégrer des clients manuellement à l’aide de modèles Azure Resource Manager. Pour plus d’informations, consultez [Intégration d’un client à Azure Lighthouse](onboard-customer.md).

Le tableau suivant peut vous aider à déterminer si des clients doivent être intégrés en publiant une offre de service managé ou en utilisant des modèles Azure Resource Manager.

|**Considération**  |**Offre de service managé**  |**Modèles ARM**  |
|---------|---------|---------|
|Nécessite un [compte Espace partenaires](../../marketplace/partner-center-portal/create-account.md)   |Oui         |Non        |
|Nécessite le [niveau de compétence de plateforme cloud Silver ou Gold](/partner-center/learn-about-competencies) ou [Azure expert MSP](https://partner.microsoft.com/membership/azure-expert-msp)      |Oui         |Non         |
|Disponible pour les nouveaux clients via la Place de marché Azure     |Oui     |Non       |
|Peut limiter l’offre à des clients spécifiques     |Oui (uniquement avec des offres privées qui ne peuvent pas être utilisées avec des abonnements souscrits via un revendeur participant au programme des fournisseurs de solutions cloud (CSP)).         |Oui         |
|Nécessite l’acceptation du client dans le portail Azure     |Oui     |Non   |
|Peut utiliser une automatisation pour intégrer plusieurs abonnements, groupes de ressources ou clients |Non     |Oui    |
|Accès immédiat aux nouveaux rôles intégrés et aux fonctionnalités d’Azure Lighthouse     |Pas toujours (mis à la disposition générale après un certain délai)         |Oui         |

## <a name="create-your-offer"></a>Créer votre offre

Pour obtenir des instructions détaillées sur la création de votre offre, y compris l’ensemble des informations et ressources que vous devrez fournir, consultez [Créer une offre de service managé](../../marketplace/partner-center-portal/create-new-managed-service-offer.md).

Pour en savoir plus sur le processus général de publication, consultez [Guide de l’éditeur Place de marché Microsoft Azure et AppSource](../../marketplace/overview.md). Vous devez également examiner les [stratégies de certification de la Place de marché commerciale](/legal/marketplace/certification-policies), en particulier la section [Managed Services](/legal/marketplace/certification-policies#700-managed-services).

Après avoir ajouté votre offre, un client peut déléguer un ou plusieurs abonnements ou groupes de ressources, qui seront ensuite [intégrés à Azure Lighthouse](#the-customer-onboarding-process).

> [!IMPORTANT]
> Chaque plan d’une offre de service managé comprend une section **Détails du manifeste** dans laquelle vous définissez les entités Azure Active Directory (Azure AD) de votre locataire qui ont accès aux groupes de ressources ou aux abonnements délégués pour les clients qui achètent ce plan. Il est important de savoir que tout groupe (ou utilisateur ou principal de service) que vous incluez aura les mêmes autorisations pour chaque client achetant le plan. Pour affecter différents groupes à chaque client, vous pouvez publier un [plan privé](../../marketplace/private-offers.md) distinct exclusif pour chaque client. N’oubliez pas que les plans privés ne sont pas pris en charge avec les abonnements souscrits via un revendeur participant au programme des fournisseurs de solutions cloud (CSP).

## <a name="publish-your-offer"></a>Publier votre offre

Lorsque vous avez terminé chacune des sections, l’étape suivante consiste à publier l’offre sur la Place de marché Azure. Sélectionnez le bouton **Publier** pour démarrer le processus de mise en ligne de votre offre. Pour plus d’informations sur ce processus, consultez cette [page](../../marketplace/partner-center-portal/create-new-managed-service-offer.md#publish). 

Vous pouvez [publier une version mise à jour de votre offre](../..//marketplace/partner-center-portal/update-existing-offer.md) à tout moment. Par exemple, vous pouvez souhaiter ajouter une nouvelle définition de rôle à une offre publiée précédemment. Dans ce cas, les clients qui ont déjà ajouté l’offre voient s’afficher une icône sur la page [**Fournisseurs de services**](view-manage-service-providers.md) du Portail Azure, ce qui leur permet de savoir qu’une mise à jour est disponible. Chaque client pourra [passer en revue les modifications](view-manage-service-providers.md#update-service-provider-offers) et décider s’il souhaite effectuer la mise à jour vers la nouvelle version. 

## <a name="the-customer-onboarding-process"></a>Processus d’intégration des clients

Après avoir ajouté votre offre, un client peut déléguer un ou plusieurs abonnements ou groupes de ressources, qui seront ensuite [intégrés à Azure Lighthouse](view-manage-service-providers.md#delegate-resources). Si un client a accepté une offre mais n’a pas encore délégué de ressources, il voit une note s’afficher en haut de la section **Offres de fournisseur** de la page [**Fournisseurs de services**](view-manage-service-providers.md) du portail Azure.

> [!IMPORTANT]
> La délégation doit être effectuée par un compte non invité dans le locataire client qui a le [rôle intégré Propriétaire](../../role-based-access-control/built-in-roles.md#owner) pour l’abonnement en cours d’intégration (ou qui contient les groupes de ressources en cours d’intégration). Pour voir tous les utilisateurs qui peuvent déléguer l’abonnement, un utilisateur du locataire du client peut sélectionner l’abonnement dans le portail Azure, ouvrir **Contrôle d’accès (IAM)** et [afficher tous les utilisateurs ayant le rôle Propriétaire](../../role-based-access-control/role-assignments-list-portal.md#list-owners-of-a-subscription).

Une fois que le client aura délégué un abonnement (ou un ou plusieurs groupes de ressources d’un abonnement), le fournisseur de ressources **Microsoft.ManagedServices** sera inscrit pour cet abonnement, et les utilisateurs de votre locataire pourront accéder aux ressources déléguées conformément aux autorisations de votre offre.

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur la [Place de marché commerciale](../../marketplace/overview.md).
- Découvrez les [Expériences de gestion inter-locataire](../concepts/cross-tenant-management-experience.md).
- [Affichez et gérez les clients](view-manage-customers.md) en accédant à **Mes clients** sur le portail Azure.