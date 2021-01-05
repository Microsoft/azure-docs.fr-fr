---
title: Contrôle de sécurité Azure - Gestion des stocks et des ressources
description: Contrôle de sécurité Azure – Gestion des stocks et des ressources
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 464439516ee86d2fb6f900c3fcaf179095192acf
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498878"
---
# <a name="security-control-inventory-and-asset-management"></a>Contrôle de sécurité : Gestion des stocks et des ressources

Les recommandations en matière de gestion des stocks et des ressources visent à résoudre les problèmes liés à la gestion active (stocks, suivi et correction) de toutes les ressources Azure, de sorte que seules les ressources autorisées disposent d’un accès et que les ressources non autorisées et non gérées soient identifiées et supprimées.

## <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.1 | 1.1, 1.2, 1.3, 1.4, 9.1, 12.1 | Customer |

Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (telles que calcul, stockage, réseau, ports et protocoles) dans vos abonnements.  Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

- [Présentation d’Azure RBAC](../../role-based-access-control/overview.md)

## <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.2 | 1.5 | Customer |

Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

- [Guide pratique pour créer et utiliser des étiquettes](../../azure-resource-manager/management/tag-resources.md)

## <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.3 | 1.6 | Customer |

Utilisez des étiquettes, des groupes d’administration et diviser des abonnements, le cas échéant, pour organiser et suivre les ressources. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../../azure-resource-manager/management/tag-resources.md)

## <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.4 | 2.1 | Customer |

Créez un inventaire des ressources Azure et logiciels approuvés pour les ressources de calcul en fonction des besoins de votre organisation.

## <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.5 | 2.3, 2.4 | Customer |

Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.

Utilisez Azure Resource Graph pour interroger/découvrir des ressources dans leurs abonnements.  Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../../governance/resource-graph/first-query-portal.md)

## <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.6 | 2.3, 2.4 | Customer |

Utilisez l’inventaire des machines virtuelles Azure pour automatiser la collecte d’informations sur tous les logiciels présents sur les machines virtuelles. Le nom, la version, l’éditeur et l’heure d’actualisation du logiciel sont disponibles sur le portail Azure. Pour avoir accès à la date d’installation et à d’autres informations, activez les diagnostics au niveau de l’invité et placez les journaux des événements Windows dans un espace de travail Log Analytics.

- [Guide pratique pour activer l’inventaire des machines virtuelles Azure](../../automation/automation-tutorial-installed-software.md)

## <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.7 | 2.5 | Customer |

Utilisez le monitoring d’intégrité des fichiers d’Azure Security Center (Change Tracking) et l’inventaire des machines virtuelles pour identifier tous les logiciels installés sur les machines virtuelles. Vous pouvez implémenter votre propre processus de suppression des logiciels non autorisés. Vous pouvez également utiliser une solution tierce pour identifier les logiciels non approuvés.

- [Guide pratique pour utiliser le monitoring d’intégrité des fichiers](../../security-center/security-center-file-integrity-monitoring.md)

- [Présentation d’Azure Change Tracking](../../automation/change-tracking/overview.md)

- [Guide pratique pour activer l’inventaire des machines virtuelles Azure](../../automation/automation-tutorial-installed-software.md)

## <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.8 | 2.6 | Customer |

Utilisez les contrôles d’application adaptatifs d’Azure Security Center pour vous assurer que seuls les logiciels autorisés s’exécutent et que l’exécution de tous les logiciels non autorisés est bloquée sur les machines virtuelles Azure.

- [Guide pratique pour utiliser les contrôles d’application adaptatifs Azure Security Center](../../security-center/security-center-adaptive-application.md)

## <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.9 | 2.6 | Customer |

Utilisez Azure Policy pour limiter les services que vous pouvez approvisionner dans votre environnement.

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../../governance/policy/samples/index.md)

## <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.10 | 2.7 | Customer |

Utilisez les contrôles d’application adaptatifs d’Azure Security Center pour spécifier les types de fichiers auxquels une règle peut ou non s’appliquer.

Implémentez une solution tierce si cela ne répond pas à la configuration requise.

- [Guide pratique pour utiliser les contrôles d’application adaptatifs Azure Security Center](../../security-center/security-center-adaptive-application.md)

## <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.11 | 2,9 | Customer |

Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Guide pratique pour configurer l’accès conditionnel de façon à bloquer l’accès à Azure Resource Manager](../../role-based-access-control/conditional-access-azure-management.md)

## <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.12 | 2,9 | Customer |

Selon le type de script, vous pouvez utiliser des configurations de système d’exploitation spécifiques ou des ressources tierces pour limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul Azure.  Vous avez également la possibilité d’exploiter les contrôles d’application adaptatifs Azure Security Center pour faire en sorte que seuls les logiciels autorisés s’exécutent et que l’exécution de tous les logiciels non autorisés soit bloquée sur les machines virtuelles Azure.

- [Guide pratique pour contrôler l’exécution des scripts PowerShell dans des environnements Windows](/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6)

- [Guide pratique pour utiliser les contrôles d’application adaptatifs Azure Security Center](../../security-center/security-center-adaptive-application.md)

## <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 6.13 | 2,9 | Customer |

Les logiciels nécessaires à des opérations métier, mais qui peuvent poser un risque élevé pour l’organisation, doivent être isolés sur leur propre machine virtuelle et/ou réseau virtuel et être suffisamment sécurisés à l’aide d’un Pare-feu Azure ou d’un groupe de sécurité réseau.

- [Guide pratique pour créer un réseau virtuel](../../virtual-network/quick-create-portal.md)

- [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../../virtual-network/tutorial-filter-network-traffic.md)


## <a name="next-steps"></a>Étapes suivantes

- Consultez le contrôle de sécurité suivant : [Configuration sécurisée](security-control-secure-configuration.md)