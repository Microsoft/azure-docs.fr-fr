---
title: Voir et gérer les fournisseurs de services
description: Les clients peuvent utiliser la page Fournisseurs de services du portail Azure pour afficher des informations sur les fournisseurs de services, les offres de fournisseurs de services et les ressources déléguées.
ms.date: 10/12/2020
ms.topic: how-to
ms.openlocfilehash: e94a9fd6562a9fa8bc0f2d84cce477e7907a1e48
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92371101"
---
# <a name="view-and-manage-service-providers"></a>Voir et gérer les fournisseurs de services

Les clients peuvent utiliser la page **Fournisseurs de services** du [Portail Azure](https://portal.azure.com) pour visualiser des informations sur les fournisseurs de services et les offres de fournisseurs de services, déléguer des ressources spécifiques à [Azure Lighthouse](../overview.md) et souscrire à de nouvelles offres de fournisseurs de services.

> [!TIP]
> Si nous faisons référence ici aux fournisseurs de services et aux clients, des [entreprises gérant plusieurs locataires](../concepts/enterprise.md) peuvent suivre le même processus pour consolider leur expérience de gestion.

Pour accéder à la page **Fournisseurs de services** sur le portail Azure, le client peut sélectionner **Tous les services** , puis rechercher et sélectionner **Fournisseurs de services** . Il peut également la trouver en tapant « Fournisseurs de services » ou « Azure Lighthouse » dans la zone de recherche dans la partie supérieure du portail Azure.

> [!NOTE]
> Pour afficher la page **Fournisseurs de services** , un utilisateur du locataire du client doit disposer du [rôle intégré Lecteur](../../role-based-access-control/built-in-roles.md#reader) (ou d’un autre rôle intégré qui comprend l’accès en lecture).
>
> Pour ajouter ou mettre à jour des offres, déléguer des ressources et supprimer des offres, l’utilisateur doit disposer du [rôle intégré Propriétaire](../../role-based-access-control/built-in-roles.md#owner) pour l’abonnement.

N’oubliez pas que la page **Fournisseurs de services** affiche des informations uniquement sur les fournisseurs de services qui ont accès aux abonnements ou aux groupes de ressources du client via Azure Lighthouse. Si un client travaille avec des fournisseurs de services supplémentaires qui n’utilisent pas Azure Lighthouse pour accéder aux ressources du client, les informations sur ces fournisseurs de services n’apparaissent pas ici.

> [!TIP]
> Les fournisseurs de services peuvent afficher des informations sur leurs clients en accédant à **Mes clients** sur le Portail Azure. Pour plus d’informations, consultez [Voir et gérer les clients et les ressources déléguées](view-manage-customers.md).

## <a name="view-service-provider-details"></a>Afficher les détails du fournisseur de services

Pour afficher des informations sur les fournisseurs de services, le client peut sélectionner **Offres de fournisseur de services** sur le côté gauche de la page **Fournisseurs de services** .

Pour chaque offre de fournisseur de services, le client voit le nom du fournisseur de services et l’offre qui y est associée, ainsi que le nom que le client a entré pendant le processus d’intégration.

Dans la colonne **Délégations** , le client voit combien d’abonnements ou groupes de ressources ont été délégués au fournisseur de services pour cette offre. Le fournisseur de services est en mesure d’accéder à ces abonnements ou groupes de ressources, ainsi que de les gérer, en fonction des niveaux d’accès spécifiés dans l’offre.

## <a name="add-or-remove-service-provider-offers"></a>Ajouter ou supprimer des offres de fournisseurs de services

Un client peut ajouter une nouvelle offre de fournisseur de services à partir de la page **Offres de fournisseur de services** en sélectionnant **Ajouter une offre** . Le fournisseur de services doit avoir publié une offre pour ce client. Le client peut ensuite sélectionner cette offre dans l’écran **Offres privées** , puis choisir **Créer** .

Si le client souhaite supprimer une offre de fournisseur de services, il peut sélectionner l’icône Corbeille sur la ligne de cette offre. Après confirmation de la suppression, ce fournisseur de services n’a plus accès aux ressources du client qui étaient précédemment déléguées pour cette offre.

## <a name="delegate-resources"></a>Déléguer des ressources

Pour qu’un fournisseur de services puisse accéder aux ressources d’un client et les gérer, celles-ci doivent être déléguées. Si un client a accepté une offre mais n’a pas encore délégué de ressources, il voit une note s’afficher en haut de la section des **Offres de fournisseur de services** . Cela lui permet de savoir qu’il doit intervenir pour que le fournisseur de services ne puisse accéder à ses ressources.

Pour déléguer des abonnements ou des groupes de ressources :

1. Cochez la case correspondant à la ligne contenant le fournisseur de services, l’offre et le nom. Sélectionnez ensuite **Déléguer des ressources** en haut de l’écran.
1. Dans la section **Détails de l’offre** de la page **Déléguer des ressources** , examinez les détails relatifs au fournisseur de services et à l’offre. Pour réviser les attributions de rôles pour l’offre, sélectionnez **Cliquer ici pour voir les détails de l’offre sélectionnée** .
1. Dans la section **Déléguer** , sélectionnez **Déléguer des abonnements** ou **Déléguer des groupes de ressources** .
1. Choisissez les abonnements ou groupes de ressources que aimeriez déléguer pour cette offre, puis sélectionnez **Ajouter** .
1. Activez la case à cocher en bas de la page pour confirmer que vous souhaitez accorder à ce fournisseur de services l’accès aux ressources que vous avez sélectionnées, puis sélectionnez **Déléguer** .

## <a name="update-service-provider-offers"></a>Mettre à jour les offres de fournisseur de services

Une fois qu’un client a ajouté une offre, un fournisseur de services peut publier une version mise à jour de cette même offre sur la place de marché Microsoft Azure. Par exemple, ils souhaiteront peut-être ajouter une nouvelle définition de rôle. Si une nouvelle version de l’offre a été publiée, la page **Offres de fournisseur de services** comprend une icône de « mise à jour » dans la ligne de l’offre en question. Le client peut alors sélectionner cette icône pour voir les différences entre la version actuelle de l'offre et la nouvelle.

 ![Icône Mettre à jour une offre](../media/update-offer.jpg)

Après examen des les modifications, le client peut décider de mettre à jour vers la nouvelle version. Dès qu’il le fait, les autorisations et autres paramètres spécifiés dans la nouvelle version s’appliquent à tous les abonnements et/ou groupes de ressources qui ont été délégués pour cette offre.

## <a name="view-delegations"></a>Afficher les délégations

Les délégations représentent les attributions de rôles qui accordent des autorisations au fournisseur de services pour les ressources qu’un client a déléguées. Pour afficher ces informations, sélectionnez **Délégations** sur le côté gauche de la page **Fournisseurs de services** .

Les filtres du haut de la page vous permettent de trier et regrouper vos informations de délégation. Vous pouvez également filtrer sur des clients, offres ou mots clés spécifiques.

> [!NOTE]
> Les clients ne verront pas ces attributions de rôles, ni les utilisateurs du locataire du fournisseur de services auxquels ces rôles ont été attribués, lors de l’[affichage des informations d’attribution de rôle pour l’étendue déléguée dans le Portail Azure](../../role-based-access-control/role-assignments-list-portal.md#list-role-assignments-at-a-scope) ou par le biais des API.

## <a name="audit-delegations-in-your-environment"></a>Auditer des délégations dans votre environnement

Les clients peuvent souhaiter plus de visibilité sur les abonnements et/ou groupes de ressources délégués à Azure Lighthouse. Cela s’avère particulièrement utile pour les clients disposant d’un grand nombre d’abonnements ou en présence de nombreux utilisateurs effectuant des tâches de gestion.

Nous fournissons une [définition de stratégie intégrée Azure Policy](../../governance/policy/samples/built-in-policies.md#lighthouse) pour [auditer la délégation d’étendues sur un locataire gestionnaire](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Lighthouse/Lighthouse_Delegations_Audit.json). Vous pouvez attribuer cette stratégie à un groupe d’administration comprenant tous les abonnements que vous souhaitez auditer. Lorsque vous utilisez cette stratégie pour vérifier la conformité, tous les abonnements et/ou groupes de ressources délégués (dans le groupe d’administration auquel la stratégie est attribuée) apparaissent dans un état non conforme. Vous pouvez ensuite passer en revue les résultats afin de vous assurer de l'absence de délégations inattendues.

Une autre [définition de stratégie intégrée](../../governance/policy/samples/built-in-policies.md#lighthouse) vous permet de [limiter les délégations à des locataires gestionnaires spécifiques](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Lighthouse/AllowCertainManagingTenantIds_Deny.json). Cette stratégie peut être appliquée de manière similaire à un groupe d’administration qui comprend tous les abonnements pour lesquels vous souhaitez limiter les délégations. Une fois la stratégie déployée, toute tentative de déléguer un abonnement à un locataire en dehors de ceux que vous spécifiez est refusée.

Pour plus d’informations sur l’attribution d’une stratégie et l’affichage des résultats de l’état de conformité, consultez [Démarrage rapide : Créer une attribution de stratégie](../../governance/policy/assign-policy-portal.md).

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur [Azure Lighthouse](../overview.md).
- Découvrez comment les fournisseurs de services peuvent [voir et gérer les clients](view-manage-customers.md) dans la page **Mes clients** sur le portail Azure.
