---
title: Voir et gérer les clients et les ressources déléguées
description: En tant que fournisseur de services ou entreprise utilisant Azure Lighthouse, vous pouvez afficher l'ensemble de vos ressources et abonnements délégués en accédant à Mes clients sur le portail Azure.
ms.date: 08/12/2020
ms.topic: how-to
ms.openlocfilehash: 165110556282e90bbacf4ce3b3c59e34b203a528
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88167196"
---
# <a name="view-and-manage-customers-and-delegated-resources"></a>Voir et gérer les clients et les ressources déléguées

Les fournisseurs de services qui utilisent [Azure Lighthouse](../overview.md) peuvent consulter la page **Mes clients** du [Portail Azure](https://portal.azure.com) pour connaître les ressources et abonnements délégués des clients. 

> [!TIP]
> Si nous faisons référence ici aux fournisseurs de services et aux clients, des [entreprises gérant plusieurs locataires](../concepts/enterprise.md) peuvent suivre le même processus pour consolider leur expérience de gestion.

Pour accéder à la page **Mes clients** du portail Azure, sélectionnez **Tous les services**, puis recherchez et sélectionnez **Mes clients**. Vous pouvez également la trouver en entrant « Mes clients » dans la zone de recherche dans la partie supérieure du portail Azure.

N’oubliez pas que la section **Clients** supérieure de la page **Mes clients** affiche des informations uniquement sur les clients qui ont délégué des abonnements ou des groupes de ressources. Si vous travaillez avec d’autres clients (par exemple, dans le cadre du programme [Fournisseur de solutions cloud](/partner-center/csp-overview)), vous ne voyez pas d’informations à leur sujet dans la section **Clients**, sauf si vous avez [intégré leurs ressources à Azure Lighthouse](onboard-customer.md).

Plus bas dans la page, une section distincte appelée **Fournisseur de solutions cloud (préversion)** , affiche des informations de facturation et des ressources pour vos clients CSP qui ont [signé le contrat client Microsoft (MCA)](/partner-center/confirm-customer-agreement) et sont [sous le plan Azure](/partner-center/azure-plan-get-started). Pour plus d’informations, voir [Bien démarrer avec votre compte de facturation dans le cadre d’un Contrat Partenaire Microsoft](../../cost-management-billing/understand/mpa-overview.md). Notez que de tels clients CSP apparaissent dans cette section, que vous les ayez ou non également intégrés à Azure Lighthouse. De même, il n’est pas nécessaire qu’un client CSP apparaisse dans la section **Fournisseur de solutions cloud (préversion)** de **Mes clients** pour pouvoir les intégrer à Azure Lighthouse.

> [!NOTE]
> Vos clients peuvent afficher des informations sur les fournisseurs de services en accédant à **Fournisseurs de services** sur le portail Azure. Pour plus d’informations, consultez [Voir et gérer les fournisseurs de services](view-manage-service-providers.md).

## <a name="view-and-manage-customer-details"></a>Voir et gérer les détails du client

Pour afficher les détails du client, sélectionnez **Clients** sur le côté gauche de la page **Mes clients**.

Pour chaque client, vous voyez le nom du client, l’ID client (ID de locataire) et l’offre associée à l’engagement. Dans la colonne **Délégations**, vous voyez le nombre d’abonnements ou de groupes de ressources délégués.

> [!IMPORTANT]
> Pour voir une délégation, les utilisateurs doivent avoir reçu le rôle [Lecteur](../../role-based-access-control/built-in-roles.md#reader) (ou un autre rôle intégré incluant l’accès Lecteur) dans le cadre du processus d’intégration.

Les filtres en haut de la page vous permettent de trier et regrouper vos informations client, ou de filtrer sur des clients, offres ou mots clés spécifiques.

Vous pouvez afficher les informations suivantes de cette page :

- Pour afficher l’ensemble des abonnements, offres et délégations associés à un client, sélectionnez le nom de celui-ci.
- Pour afficher des détails supplémentaires sur une offre et ses délégations, sélectionnez le nom de l’offre.
- Pour voir plus de détails sur les affectations de rôles pour les abonnements ou groupes de ressources délégués, sélectionnez l’entrée dans la colonne **Délégations**.

## <a name="view-and-manage-delegations"></a>Afficher et gérer les délégations

Les délégations affichent l'abonnement ou le groupe de ressources délégué, ainsi que les utilisateurs et autorisations qui y ont accès. Pour afficher ces informations, sélectionnez **Délégations** sur le côté gauche de la page **Mes clients**.

Les filtres en haut de la page vous permettent de trier et de regrouper vos informations d’affectation d’accès ou de filtrer sur des clients, offres ou mots clés spécifiques.

### <a name="view-role-assignments"></a>Voir les attributions de rôles

Les utilisateurs et les autorisations associés à chaque délégation apparaissent dans la colonne **Attributions de rôles**. Vous pouvez sélectionner chaque entrée pour afficher la liste complète des utilisateurs, groupes et principaux de service qui ont reçu l’accès à l’abonnement ou au groupe de ressources. À partir de là, vous pouvez sélectionner un utilisateur, un groupe ou un nom de principal du service particuliers pour obtenir plus de détails.

### <a name="remove-delegations"></a>Supprimer des délégations

Si vous avez inclus des utilisateurs possédant le [rôle Supprimer l’attribution de l’inscription aux services gérés](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) lors de l’intégration d’un client à Azure Lighthouse, ils peuvent supprimer une délégation en sélectionnant l’icône de la corbeille figurant sur la ligne correspondante. Dans ce cas, aucun utilisateur du locataire du fournisseur de services ne pourra accéder aux ressources précédemment déléguées.

## <a name="work-in-the-context-of-a-delegated-subscription"></a>Travailler dans le contexte d’un abonnement délégué

Vous pouvez travailler directement dans le contexte d’un abonnement délégué au sein du portail Azure sans changer le répertoire dans lequel vous travaillez. Pour ce faire :

1. Sélectionnez l’icône **Répertoires délégués + actuels** dans la partie supérieure du portail Azure.
2. Dans le filtre **abonnement global**, vérifiez que seule la case correspondant à cet abonnement délégué est sélectionnée. Vous pouvez utiliser la zone de liste déroulante **Répertoires délégués + actuels** pour afficher uniquement les abonnements figurant dans un répertoire spécifique (n’utilisez pas l’option **Changer de répertoire**, car cela modifie le répertoire auquel vous êtes connecté).

Si vous accédez ensuite à un service qui prend en charge les [expériences de gestion inter-locataire](../concepts/cross-tenant-management-experience.md), le service est par défaut dans le contexte de l’abonnement délégué que vous avez sélectionné. Vous pouvez modifier cela en suivant les étapes ci-dessus et en activant la case à cocher **Sélectionner tout** (ou en choisissant un ou plusieurs abonnements dans lesquels travailler à la place).

> [!NOTE]
> Si vous avez obtenu l’accès à un ou plusieurs groupes de ressources, au lieu d’accéder à un abonnement entier, vous pouvez sélectionner l’abonnement auquel appartient ce groupe de ressources. Vous travaillerez ensuite dans le contexte de cet abonnement, mais ne pourrez accéder qu’aux groupes de ressources désignés.

Vous pouvez également accéder à des fonctionnalités associées à des abonnements ou à des groupes de ressources délégués à partir de services qui prennent en charge les expériences de gestion inter-locataire en sélectionnant l’abonnement ou le groupe de ressources au sein de ce service.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez les [Expériences de gestion inter-locataire](../concepts/cross-tenant-management-experience.md).
- Découvrez comment vos clients peuvent [Voir et gérer les fournisseurs de services](view-manage-service-providers.md) en accédant à **Fournisseurs de service** dans le portail Azure.
