---
title: Gérer les espaces de travail Azure Sentinel à grande échelle
description: Apprenez à gérer efficacement Azure Sentinel sur les ressources client déléguées.
ms.date: 09/30/2020
ms.topic: how-to
ms.openlocfilehash: 91e2f875aa2fc067420c0c6eda4e7dd56bd2b088
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92424093"
---
# <a name="manage-azure-sentinel-workspaces-at-scale"></a>Gérer les espaces de travail Azure Sentinel à grande échelle

En tant que fournisseur de services, vous avez peut-être intégré les locataires de plusieurs clients à [Azure Lighthouse](../overview.md). Azure Lighthouse permet aux fournisseurs de services d’effectuer des opérations à grande échelle sur plusieurs locataires Azure Active Directory (Azure AD) à la fois, améliorant ainsi l’efficacité des tâches de gestion.

Azure Sentinel assure une analyse de sécurité intelligente et fournit des informations sur les menaces. Elle constitue une solution unique pour la détection des alertes, la visibilité des menaces, la chasse proactive et la réponse face aux menaces. Avec Azure Lighthouse, vous pouvez gérer plusieurs espaces de travail Azure Sentinel entre les locataires à grande échelle. Cela permet des scénarios tels que l’exécution de requêtes sur plusieurs espaces de travail ou la création de classeurs pour visualiser et surveiller les données de vos sources de données connectées afin de mieux les exploiter. Les protocoles Internet tels que les requêtes et les guides opérationnels restent dans votre locataire de gestion, mais ils peuvent être utilisés pour effectuer la gestion de la sécurité dans les locataires clients.

Cette rubrique fournit une vue d’ensemble de l’utilisation d’[Azure Sentinel](../../sentinel/overview.md) de manière évolutive pour la visibilité entre locataires et les services de sécurité managés.

> [!TIP]
> Bien que nous faisons référence aux fournisseurs de services et aux clients dans cette rubrique, ces instructions s’appliquent également aux [entreprises utilisant Azure Lighthouse pour gérer plusieurs locataires](../concepts/enterprise.md).

## <a name="architectural-considerations"></a>Considérations sur l’architecture

Pour un fournisseur MSSP (Managed Security Service Provider) souhaitant constituer une offre de sécurité en tant que service avec Azure Sentinel, un centre d’opérations de sécurité (SOC) peut être nécessaire pour surveiller, gérer et configurer de manière centralisée plusieurs espaces de travail Azure Sentinel déployés au sein de locataires clients individuels. De même, les entreprises avec plusieurs locataires Azure AD peuvent souhaiter gérer de manière centralisée plusieurs espaces de travail Azure Sentinel déployés chez l’ensemble de leurs locataires.

Ce modèle centralisé de déploiement présente les avantages suivants :

- La propriété des données est conservée par chaque locataire géré.
- Il prend en charge des exigences de stockage des données dans les limites géographiques.
- Il garantit l’isolation des données, car les données de plusieurs clients ne sont pas stockées dans le même espace de travail.
- Il empêche l’exfiltration des données des locataires managés, garantissant ainsi la conformité des données.
- Les coûts associés sont facturés à chaque locataire géré, plutôt qu’au locataire gérant.
- Les données de toutes les sources de données et de tous les connecteurs de données intégrés à Azure Sentinel (comme les journaux d’activité Azure AD, les journaux Office 365 ou les alertes de Protection Microsoft contre les menaces) sont conservées dans chaque locataire client.
- Il réduit le temps de réponse du réseau.
- Il est facile d’ajouter ou de supprimer de nouvelles filiales ou clients.

> [!NOTE]
> Vous pouvez gérer des ressources déléguées situées dans différentes [régions](../../availability-zones/az-overview.md#regions). Toutefois, la délégation d’abonnements sur un [cloud national](../../active-directory/develop/authentication-national-cloud.md) et le cloud public Azure, ou sur deux clouds nationaux distincts, n’est pas prise en charge.

## <a name="granular-azure-role-based-access-control-azure-rbac"></a>Contrôle d’accès en fonction du rôle Azure (Azure RBAC) granulaire

Chaque abonnement client géré par un MSSP doit être [intégré à Azure Lighthouse](onboard-customer.md). Cela permet aux utilisateurs désignés du locataire gérant d’accéder aux opérations de gestion et de les effectuer sur les espaces de travail Azure Sentinel déployés chez les locataires clients.

Lorsque vous créez vos autorisations, vous pouvez affecter les rôles intégrés Azure Sentinel aux utilisateurs, groupes ou principaux du service dans votre locataire gérant :

- [Lecteur Azure Sentinel](../../role-based-access-control/built-in-roles.md#azure-sentinel-reader)
- [Répondeur Azure Sentinel](../../role-based-access-control/built-in-roles.md#azure-sentinel-responder)
- [Contributeur Azure Sentinel](../../role-based-access-control/built-in-roles.md#azure-sentinel-contributor)

Vous pouvez également affecter des rôles intégrés supplémentaires pour exécuter des fonctions additionnelles. Pour plus d’informations sur les rôles spécifiques qui peuvent être utilisés avec Azure Sentinel, consultez les [Autorisations dans Azure Sentinel](../../sentinel/roles.md).

Une fois que vous avez intégré vos clients, les utilisateurs désignés peuvent se connecter à votre locataire gérant et [accéder directement à l’espace de travail Azure Sentinel du client](../../sentinel/multiple-tenants-service-providers.md) avec les rôles qui ont été affectés.

## <a name="view-and-manage-incidents-across-workspaces"></a>Afficher et gérer les incidents des espaces de travail

Si vous gérez des ressources Azure Sentinel pour plusieurs clients, vous pouvez afficher et gérer les incidents dans plusieurs espaces de travail et plusieurs locataires à la fois. Pour plus d’informations, consultez [Travailler avec des incidents dans plusieurs espaces de travail à la fois](../../sentinel/multiple-workspace-view.md) et [Étendre Azure Sentinel sur les espaces de travail et les locataires](../../sentinel/extend-sentinel-across-workspaces-tenants.md).

> [!NOTE]
> Assurez-vous que les utilisateurs de votre locataire gérant disposent d’autorisations en lecture et en écriture sur tous les espaces de travail qui sont managés. Si un utilisateur dispose uniquement d’autorisations de lecture sur certains espaces de travail, des messages d’avertissement peuvent s’afficher lorsqu’il sélectionne des incidents dans ces espaces de travail et il ne peut pas modifier ces incidents, ni aucun autre sélectionné (même si vous disposez des autorisations pour les autres).

## <a name="configure-playbooks-for-mitigation"></a>Configurer des playbooks d’atténuation

[Les playbooks](../../sentinel/tutorial-respond-threats-playbook.md) peuvent être utilisés pour atténuer automatiquement lorsqu’une alerte est déclenchée. Ces playbooks peuvent être exécutés manuellement ou s’exécuter automatiquement lorsque des alertes spécifiques sont déclenchées. Les playbooks peuvent être déployés dans le locataire gérant ou client, avec les procédures de réponse configurées en fonction des utilisateurs du locataire qui agir pour répondre à une menace de sécurité.

## <a name="create-cross-tenant-workbooks"></a>Créer des classeurs inter-locataires

[Les classeurs Azure Monitor dans Azure Sentinel](../../sentinel/overview.md#workbooks) vous aident à visualiser et à surveiller les données de vos sources de données connectées pour obtenir des Insights. Vous pouvez utiliser les modèles de classeurs intégrés dans Azure Sentinel ou créer des classeurs personnalisés pour vos scénarios.

Vous pouvez déployer des classeurs dans votre client gérant et créer des tableaux de bord à l’échelle pour surveiller et interroger les données des locataires clients. Pour plus d’informations, consultez l’article [Surveiller plusieurs espaces de travail](../../sentinel/extend-sentinel-across-workspaces-tenants.md#using-cross-workspace-workbooks). 

Vous pouvez également déployer des classeurs directement dans un locataire individuel que vous gérez pour des scénarios qui lui sont spécifiques.

## <a name="run-log-analytics-and-hunting-queries-across-azure-sentinel-workspaces"></a>Exécuter des requêtes d’analytique des journaux d’activité et de chasse dans des espaces de travail Azure Sentinel

Créez et enregistrez des requêtes d’analytique des journaux d’activité pour la détection des menaces de manière centralisée dans le locataire gérant, y compris des [requêtes de chasse](../../sentinel/extend-sentinel-across-workspaces-tenants.md#cross-workspace-hunting). Ces requêtes peuvent ensuite être exécutées sur l’ensemble des espaces de travail Azure Sentinel de vos clients à l’aide de l’opérateur Union et de l’expression de langage d’espace de travail (). Pour plus d’informations, consultez l’article [Interroger plusieurs espaces de travail](../../sentinel/extend-sentinel-across-workspaces-tenants.md#cross-workspace-querying).

## <a name="use-automation-for-cross-workspace-management"></a>Utiliser l’automatisation pour la gestion inter-espaces de travail

Vous pouvez utiliser l’automatisation pour gérer plusieurs espaces de travail Azure Sentinel et configurer des [requêtes de chasse](../../sentinel/hunting.md), des playbooks et des classeurs. Pour plus d’informations, consultez l’article [Gestion inter-espaces de travail à l’aide de l’automatisation](../../sentinel/extend-sentinel-across-workspaces-tenants.md#cross-workspace-management-using-automation).

## <a name="manage-security-of-office-365-environments"></a>Gérer la sécurité des environnements Office 365

Utilisez Azure Lighthouse conjointement avec Azure Sentinel pour gérer la sécurité des environnements Office 365 entre les locataires. Tout d’abord, [les connecteurs de données Office 365 doivent être activés dans le client géré](../../sentinel/connect-office-365.md) afin que les informations sur les activités des utilisateurs et des administrateurs dans Exchange et SharePoint (y compris OneDrive) puissent être ingérées dans un espace de travail Azure Sentinel au sein du locataire géré. Cela comprend des détails sur des actions telles que le téléchargement de fichiers, les demandes d’accès envoyées, les changements apportés aux événements de groupe et les opérations de boîtes aux lettres, ainsi que des informations sur les utilisateurs qui ont effectué les actions. Les [alertes DLP Office 365](https://techcommunity.microsoft.com/t5/azure-sentinel/ingest-office-365-dlp-events-into-azure-sentinel/ba-p/1031820) sont également prises en charge dans le cadre du connecteur Office 365 intégré.

Vous pouvez activer le [connecteur Microsoft Cloud App Security (MCAS)](../../sentinel/connect-cloud-app-security.md) pour diffuser en continu des alertes et des journaux Cloud Discovery dans Azure Sentinel. Cela vous permet de bénéficier d’une visibilité accrue sur vos applications et de fonctionnalités d’analytique sophistiquées pour identifier et combattre les cybermenaces et de contrôler le déplacement des données. Les journaux d’activité pour MCAS peuvent être [utilisés à l’aide du format CEF (Common Event format)](https://techcommunity.microsoft.com/t5/azure-sentinel/ingest-box-com-activity-events-via-microsoft-cloud-app-security/ba-p/1072849).

Après avoir configuré les connecteurs de données Office 365, vous pouvez utiliser des capacités d’Azure Sentinel entre locataires, telles que l’affichage et l’analyse des données dans les classeurs, l’utilisation de requêtes pour créer des alertes personnalisées et la configuration de playbooks pour répondre aux menaces.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Azure Sentinel](../../sentinel/overview.md).
- Accédez à la page de tarification [Azure Sentinel](https://azure.microsoft.com/pricing/details/azure-sentinel/).
- Découvrez les [Expériences de gestion inter-locataire](../concepts/cross-tenant-management-experience.md).

