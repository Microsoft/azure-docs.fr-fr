---
title: Voir l’activité des fournisseurs de services
description: Les clients peuvent consulter l’activité journalisée pour voir les actions effectuées par les fournisseurs de services par le biais de la gestion des ressources déléguées Azure.
ms.date: 12/11/2020
ms.topic: how-to
ms.openlocfilehash: dcf177cc41dac846d096607445ff4c3d433620ca
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97356376"
---
# <a name="view-service-provider-activity"></a>Voir l’activité des fournisseurs de services

Les clients disposant d’abonnements délégués pour [Azure Lighthouse](../overview.md) peuvent [consulter les données du journal d’activité Azure](../../azure-monitor/platform/platform-logs-overview.md) pour voir toutes les actions entreprises. Ils bénéficient ainsi d’une visibilité complète sur les opérations effectuées par les fournisseurs de services par le biais de la [gestion des ressources déléguées Azure](../concepts/azure-delegated-resource-management.md), ainsi que par les utilisateurs dans le locataire Azure Active Directory (Azure AD) du client.

> [!TIP]
> Nous fournissons également des définitions de stratégie intégrée Azure Policy pour [restreindre la délégation à des locataires gestionnaires spécifiques](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Lighthouse/AllowCertainManagingTenantIds_Deny.json) et [auditer la délégation d’étendues sur un locataire gestionnaire](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Lighthouse/Lighthouse_Delegations_Audit.json). Pour plus d’informations, consultez [Auditer des délégations dans votre environnement](view-manage-service-providers.md#audit-delegations-in-your-environment).

## <a name="view-activity-log-data"></a>Voir les données du journal d’activité

Vous pouvez [voir le journal d’activité](../../azure-monitor/platform/activity-log.md#view-the-activity-log) à partir du menu **Superviser** dans le portail Azure. Pour limiter les résultats à un abonnement spécifique, utilisez les filtres afin de sélectionner un abonnement spécifique. Vous pouvez également [voir et récupérer les événements du journal d’activité](../../azure-monitor/platform/activity-log.md#view-the-activity-log) programmatiquement.

> [!NOTE]
> Les utilisateurs du locataire d’un fournisseur de services peuvent voir les résultats du journal d’activité d’un abonnement délégué dans un locataire client s’ils ont reçu le rôle [Lecteur](../../role-based-access-control/built-in-roles.md#reader) (ou un autre rôle intégré comprenant un accès Lecteur) quand l’abonnement a été intégré à Azure Lighthouse.

Dans le journal d’activité, vous verrez le nom de l’opération et son état, ainsi que la date et l’heure de son exécution. La colonne **Événement lancé par** indique l’utilisateur qui a effectué l’opération, qu’il s’agisse d’un utilisateur dans le locataire d’un fournisseur de services agissant par le biais d’Azure Lighthouse ou d’un utilisateur dans le locataire du client. Notez que le nom de l’utilisateur est indiqué, et non le locataire ou le rôle auquel l’utilisateur a été affecté pour cet abonnement.

L’activité journalisée des 90 derniers jours est disponible dans le portail Azure. Pour savoir comment stocker ces données pendant plus de 90 jours, consultez [Collecter et analyser les journaux d’activité Azure dans l’espace de travail Log Analytics](../../azure-monitor/platform/activity-log.md).

> [!NOTE]
> Les utilisateurs du fournisseur de services apparaissent dans le journal d’activité, mais ces utilisateurs et leurs attributions de rôles ne sont pas affichés dans **Contrôle d’accès (IAM)** ni lors de la récupération des informations d’attribution de rôle par le biais des API.

## <a name="set-alerts-for-critical-operations"></a>Définir des alertes pour les opérations critiques

Pour rester informé des opérations critiques que les fournisseurs de services (ou les utilisateurs de votre propre locataire) effectuent, nous vous recommandons de créer des [alertes de journal d’activité](../../azure-monitor/platform/activity-log-alerts.md). Par exemple, vous souhaiterez peut-être effectuer le suivi de toutes les actions d’administration pour un abonnement ou recevoir une notification quand une machine virtuelle d’un groupe de ressources particulier est supprimée. Quand vous créez des alertes, celles-ci incluent les actions effectuées par les utilisateurs dans le propre locataire du client, ainsi que dans tous les locataires gestionnaires.

Pour plus d’informations, consultez [Créer et gérer les alertes de journal d’activité](../../azure-monitor/platform/alerts-activity-log.md).

## <a name="create-log-queries"></a>Créer des requêtes de journal

Vous pouvez créer des requêtes pour analyser votre activité journalisée ou vous concentrer sur des éléments spécifiques. Par exemple, il est possible qu’un audit vous oblige à signaler toutes les actions de niveau administratif effectuées sur un abonnement. Vous pouvez créer une requête pour filtrer uniquement ces actions et trier les résultats selon l’utilisateur, la date ou une autre valeur.

Pour plus d’informations, consultez [Vue d’ensemble des requêtes de journal dans Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).

## <a name="view-user-activity-across-domains"></a>Afficher l’activité des utilisateurs dans les domaines

Vous pouvez afficher l’activité d’utilisateurs individuels dans plusieurs domaines à l’aide de l’exemple de classeur [Journaux d’activité par domaine](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/workbook-activitylogs-by-domain).

Les résultats peuvent être filtrés par nom de domaine. Vous pouvez également appliquer des filtres supplémentaires tels qu’une catégorie, un niveau ou des groupe de ressources.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Azure Monitor](../../azure-monitor/index.yml).
- Découvrez comment [afficher et gérer les offres du fournisseur de services](view-manage-service-providers.md) dans le portail Azure.
