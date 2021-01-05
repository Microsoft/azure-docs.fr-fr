---
title: Offres de services managés dans la Place de marché Azure
description: Les offres de services gérés vous permettent de vendre des offres de gestion des ressources à des clients sur la Place de marché Azure.
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: 6c3047cd95128f689e75d9c1f5fba5a39f86291c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88163320"
---
# <a name="managed-service-offers-in-azure-marketplace"></a>Offres de services managés dans la Place de marché Azure

Cet article décrit le type d’offre **Services gérés** sur la [Place de marché Azure](https://azuremarketplace.microsoft.com). Les offres de services gérés vous permettent d’offrir des services de gestion des ressources à des clients par le biais [d’Azure Lighthouse](../overview.md). Vous pouvez mettre ces offres à la disposition de tous les clients potentiels ou uniquement d’un ou plusieurs clients spécifiques. Dans la mesure où vous facturez directement les clients pour les coûts liés à ces services managés, Microsoft ne facture aucuns frais.

## <a name="understand-managed-service-offers"></a>Comprendre les offres de services gérés

Les offres de services gérés simplifient le processus d’intégration des clients pour Azure Lighthouse. Quand un client achète une offre dans la Place de marché Azure, il peut spécifier les abonnements et/ou les groupes de ressources à intégrer.

Après cela, les utilisateurs de votre organisation seront en mesure de travailler sur ces ressources à partir de votre locataire de gestion avec la [gestion des ressources déléguées Azure](azure-delegated-resource-management.md), en fonction de l’accès que vous avez défini lors de la création de l’offre. Cette opération s’effectue par le biais d’un manifeste spécifiant les utilisateurs, groupes et principaux de service Azure Active Directory (Azure AD) qui auront accès aux ressources du client, ainsi que les rôles déterminant leur niveau d’accès. En affectant des autorisations à un groupe Azure AD plutôt qu’à une série de comptes d’utilisateur ou d’application individuels, vous pouvez ajouter ou supprimer des utilisateurs lorsque vos besoins d’accès changent.

## <a name="public-and-private-offers"></a>Offres publiques et privées

Chaque offre de services gérés comprend un ou plusieurs plans. Les plans peuvent être privés ou publics.

Si vous souhaitez limiter votre offre à des clients spécifiques, vous pouvez publier un plan privé. Dans ce cas, le plan ne peut être acheté que pour les ID d’abonnement que vous spécifiez. Pour plus d’informations, voir [Offres privées](../../marketplace/private-offers.md).

> [!NOTE]
> Les offres privées ne sont pas prises en charge avec les abonnements souscrits via un revendeur participant au programme des fournisseurs de solutions cloud (CSP).

Les plans publics vous permettent de promouvoir vos services auprès de nouveaux clients. Ils sont généralement plus appropriés lorsque vous n’avez besoin que d’un accès limité au locataire du client. Une fois que vous avez établi une relation avec un client, si celui-ci décide d’accorder un accès supplémentaire à votre organisation, vous pouvez soit publier un nouveau plan privé exclusivement pour ce client, soit [intégrer celui-ci pour un accès supplémentaire à l’aide de modèles Resource Manager](../how-to/onboard-customer.md).

Le cas échéant, vous pouvez inclure des plans publics et privés dans la même offre.

> [!IMPORTANT]
> Une fois que vous avez publié un plan public, vous ne pouvez plus le changer en plan privé. Pour contrôler les clients qui peuvent accepter votre offre et déléguer des ressources, utilisez un plan privé. Avec un plan public, vous ne pouvez pas limiter la disponibilité à des clients spécifiques ou à un certain nombre de clients (en revanche, vous pouvez arrêter complètement la vente du plan si vous le souhaitez). Vous pouvez [supprimer l’accès à une délégation](../how-to/remove-delegation.md) après qu’un client a accepté une offre uniquement si vous avez inclus une **Autorisation** avec la **Définition de rôle** définie sur [Inscription des services managés, attribution Supprimer le rôle](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) lors de la publication de l’offre. Vous pouvez également contacter le client et lui demander de [supprimer votre accès](../how-to/view-manage-service-providers.md#add-or-remove-service-provider-offers).

## <a name="publish-managed-service-offers"></a>Publier des offres de services gérés

Pour savoir comment publier une offre de services gérés, consultez [Publier une offre de services gérés sur la Place de marché Azure](../how-to/publish-managed-services-offers.md).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez la [Gestion des ressources déléguées Azure](azure-delegated-resource-management.md) et les [Expériences de gestion inter-locataire](cross-tenant-management-experience.md).
- [Publier des offres de services gérés](../how-to/publish-managed-services-offers.md) sur la Place de marché Azure.
