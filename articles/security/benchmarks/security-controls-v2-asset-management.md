---
title: Benchmark de sécurité Azure v2 - Gestion des ressources
description: 'Benchmark de sécurité Azure v2 : Gestion des ressources'
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/20/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: f0c2fe78c32357798e1f9acb43f5867df9148b38
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368900"
---
# <a name="security-control-v2-asset-management"></a>Contrôle de sécurité V2 : Gestion des ressources

La gestion des ressources couvre les contrôles permettant de garantir la visibilité et la gouvernance de sécurité sur les ressources Azure. Cela comprend des recommandations relatives aux autorisations pour le personnel de sécurité, l’accès sécurisé à l’inventaire des ressources et la gestion des approbations pour les services et les ressources (inventaire, suivi et correction).

## <a name="am-1-ensure-security-team-has-visibility-into-risks-for-assets"></a>AM-1 : S’assurer que l’équipe de sécurité a une visibilité sur les risques pour les ressources

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| AM-1 | 1.1, 1.2 | CM-8, PM-5 |

Assurez-vous que les équipes de sécurité reçoivent des autorisations de lecteur de sécurité dans votre locataire et vos abonnements Azure afin qu’elles puissent surveiller les risques de sécurité à l’aide d’Azure Security Center. 

Selon la façon dont les responsabilités de l’équipe de sécurité sont structurées, la surveillance des risques de sécurité peut tenir de la responsabilité d’une équipe de sécurité centrale ou d’une équipe locale. Cela dit, les insights et les risques liés à la sécurité doivent toujours être agrégés de manière centralisée au sein d’une organisation. 

Les autorisations de lecteur de sécurité peuvent être appliquées globalement à un locataire entier (groupe d’administration racine) ou étendues à des groupes d’administration ou à des abonnements spécifiques. 

Remarque : Des autorisations supplémentaires peuvent être nécessaires pour obtenir une visibilité sur les charges de travail et services. 

- [Vue d’ensemble du rôle lecteur de sécurité](../../role-based-access-control/built-in-roles.md#security-reader)

- [Vue d'ensemble des groupes d'administration Azure](../../governance/management-groups/overview.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="am-2-ensure-security-team-has-access-to-asset-inventory-and-metadata"></a>AM-2 : S’assurer que l’équipe de sécurité a accès à l’inventaire des ressources et aux métadonnées

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| AM-2 | 1.1, 1.2,  1.4, 1.5,  9.1, 12.1 | CM-8, PM-5 |

Assurez-vous que les équipes de sécurité ont accès à un inventaire des ressources continuellement mis à jour sur Azure. Les équipes de sécurité ont souvent besoin de cet inventaire pour évaluer l’exposition potentielle de leur organisation à des risques émergents, et en tant qu’entrée pour des améliorations de sécurité continues. 

La fonctionnalité d’inventaire d’Azure Security Center et Azure Resource Graph peuvent rechercher et découvrir toutes les ressources dans vos abonnements, notamment les services Azure, les applications et les ressources réseau.  

Organisez logiquement les ressources en fonction de la taxonomie de votre organisation à l’aide de balises, ainsi que d’autres métadonnées dans Azure (nom, description et catégorie).  

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../../governance/resource-graph/first-query-portal.md)

- [Gestion de l’inventaire de sécurité dans Azure Security Center](../../security-center/asset-inventory.md)

- [Pour plus d’informations sur l’étiquetage des ressources, consultez le guide de décision concernant le nommage et l’étiquetage des ressources](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=%2fazure%2fazure-resource-manager%2fmanagement%2ftoc.json)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="am-3-use-only-approved-azure-services"></a>AM-3 : Utiliser des services Azure approuvés uniquement

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| AM-3 | 2.3, 2.4 | CM-7, CM-8 |

Utilisez Azure Policy pour auditer et limiter les services que les utilisateurs peuvent approvisionner dans votre environnement. Utilisez Azure Resource Graph pour interroger et découvrir des ressources dans leurs abonnements.  Vous pouvez également utiliser Azure Monitor pour créer des règles afin de déclencher des alertes lorsqu’un service non approuvé est détecté.

- [Configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../../governance/policy/samples/index.md)

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../../governance/resource-graph/first-query-portal.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)  

## <a name="am-4-ensure-security-of-asset-lifecycle-management"></a>AM-4 : Garantir la sécurité de la gestion du cycle de vie des actifs

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| AM-4 | 2.3, 2.4, 2.5 | CM-7, CM-8, CM-10, CM-11 |

Établissez ou mettez à jour des stratégies de sécurité qui traitent les processus de gestion du cycle de vie pour les modifications à fort impact potentiel. Ces modifications incluent les modifications apportées aux fournisseurs d’identité et aux accès, à la sensibilité des données, à la configuration du réseau et à l’attribution des privilèges d’administration.

Supprimez les ressources Azure qui ne sont plus nécessaires.

- [Supprimer un groupe de ressources et des ressources Azure](../../azure-resource-manager/management/delete-resource-group.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)  

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="am-5-limit-users-ability-to-interact-with-azure-resource-manager"></a>AM-5 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| AM-5 | 2,9 | AC-3 |

Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Guide pratique pour configurer l’accès conditionnel de façon à bloquer l’accès à Azure Resource Manager](../../role-based-access-control/conditional-access-azure-management.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)  

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

## <a name="am-6-use-only-approved-applications-in-compute-resources"></a>AM-6 : Utiliser uniquement des applications approuvées dans les ressources de calcul

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| AM-6 | 2.6, 2.7 | AC-3, CM-7, CM-8, CM-10, CM-11 |

S’assurer que seuls les logiciels autorisés s’exécutent et que l’exécution de tous les logiciels non autorisés est bloquée sur les machines virtuelles Azure.

Utilisez des contrôles d’application adaptative Azure Security Center (ASC) pour détecter et générer une liste d’applications autorisées. Vous avez également la possibilité d’utiliser des contrôles d’application adaptatifs d’ASC pour faire en sorte que seuls les logiciels autorisés s’exécutent et que l’exécution de tous les logiciels non autorisés soit bloquée sur les machines virtuelles Azure.

Utilisez la fonctionnalité Suivi des modifications et inventaire d’Azure Automation Change pour automatiser la collecte d’informations d’inventaire à partir de vos machines virtuelles Windows et Linux. Le nom, la version, l’éditeur et l’heure d’actualisation du logiciel sont disponibles sur le portail Azure. Pour obtenir la date d’installation du logiciel, ainsi que d’autres informations, activez les diagnostics au niveau de l’invité, et acheminez les journaux les événements Windows vers un espace de travail Log Analytics.

Selon le type de script, vous pouvez utiliser des configurations de système d’exploitation spécifiques ou des ressources tierces pour limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul Azure. 

Vous pouvez également utiliser une solution tierce pour découvrir et identifier les logiciels non approuvés.

- [Guide pratique pour utiliser les contrôles d’application adaptatifs d’Azure Security Center](../../security-center/security-center-adaptive-application.md)

- [Comprendre la fonctionnalité Suivi des modifications et inventaire d’Azure Automation](../../automation/change-tracking/overview.md)

- [Guide pratique pour contrôler l’exécution des scripts PowerShell dans des environnements Windows](/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)  

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)