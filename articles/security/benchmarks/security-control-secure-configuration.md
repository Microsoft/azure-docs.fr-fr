---
title: Contrôle de sécurité Azure - Configuration sécurisée
description: Configuration sécurisée du contrôle de sécurité Azure
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 349456b0a4de65520dd633efe0573d24ea1c89c4
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94409023"
---
# <a name="security-control-secure-configuration"></a>Contrôle de sécurité : Configuration sécurisée

Établissez, implémentez et gérez activement (suivi, création de rapports, correction) de la configuration de sécurité des ressources Azure afin d’empêcher les attaquants de tirer parti des services et des paramètres vulnérables.

## <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.1 | 5,1 | Customer |

Utilisez des alias Azure Policy pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration de vos ressources Azure. Vous pouvez aussi utiliser des définitions Azure Policy intégrées.

Par ailleurs, Azure Resource Manager a la possibilité d’exporter le modèle au format JSON (JavaScript Object Notation), qui doit être examiné pour vérifier que les configurations répondent/dépassent les exigences de sécurité de votre organisation.

Vous pouvez aussi utiliser les recommandations d’Azure Security Center comme base de référence de configuration sécurisée pour vos ressources Azure.

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../../governance/policy/tutorials/create-and-manage.md)

- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../../azure-resource-manager/templates/export-template-portal.md)

- [Recommandations de sécurité - Guide de référence](../../security-center/recommendations-reference.md)

## <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.2 | 5,1 | Customer |

Utilisez les recommandations d'Azure Security Center pour préserver les configurations de sécurité sur toutes les ressources de calcul.  En outre, vous pouvez utiliser des images de système d'exploitation personnalisées ou Azure Automation State Configuration pour établir la configuration de sécurité du système d'exploitation requise par votre organisation.

- [Suivi des recommandations d'Azure Security Center](../../security-center/security-center-recommendations.md)

- [Recommandations de sécurité - Guide de référence](../../security-center/recommendations-reference.md)

- [Présentation d'Azure Automation State Configuration](../../automation/automation-dsc-overview.md)

- [Charger un disque dur virtuel et l'utiliser pour créer des machines virtuelles Windows dans Azure](../../virtual-machines/windows/upload-generalized-managed.md)

- [Créer une machine virtuelle Linux à partir d'un disque personnalisé avec Azure CLI](../../virtual-machines/linux/upload-vhd.md)

## <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.3 | 5.2 | Customer |

Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.  En outre, vous pouvez utiliser des modèles Azure Resource Manager pour appliquer la configuration de sécurité des ressources Azure requise par votre organisation. 

- [Comprendre les effets d'Azure Policy](../../governance/policy/concepts/effects.md)

- [Créer et gérer des stratégies pour assurer la conformité](../../governance/policy/tutorials/create-and-manage.md)

- [Présentation des modèles Azure Resource Manager](../../azure-resource-manager/templates/overview.md)

## <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.4 | 5.2 | Partagé |

Suivez les recommandations d’Azure Security Center en matière d'évaluation des vulnérabilités sur vos ressources de calcul Azure.  En outre, vous pouvez utiliser des modèles Azure Resource Manager, des images de système d'exploitation personnalisées ou Azure Automation State Configuration pour appliquer la configuration de sécurité du système d'exploitation requise par votre organisation.   Combinés à Azure Automation Desired State Configuration, les modèles de machine virtuelle Microsoft peuvent permettre de satisfaire les exigences de sécurité. 

Notez également que les images de machines virtuelles de la Place de marché Azure publiées par Microsoft sont gérées par Microsoft. 

- [Implémenter les recommandations d'évaluation des vulnérabilités d'Azure Security Center](../../security-center/deploy-vulnerability-assessment-vm.md)

- [Créer une machine virtuelle Azure à partir d'un modèle Azure Resource Manager](../../virtual-machines/windows/ps-template.md)

- [Présentation d'Azure Automation State Configuration](../../automation/automation-dsc-overview.md)

- [Créer une machine virtuelle Windows sur le portail Azure](../../virtual-machines/windows/quick-create-portal.md)

- [Informations sur le téléchargement du modèle de machine virtuelle](../../virtual-machines/windows/download-template.md)

- [Exemple de script permettant de charger un disque dur virtuel sur Azure et de créer une machine virtuelle](../../virtual-machines/scripts/virtual-machines-windows-powershell-upload-generalized-script.md)

## <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.5 | 5.3 | Customer |

Utilisez Azure DevOps pour stocker et gérer en toute sécurité votre code, comme les stratégies Azure personnalisées, les modèles Azure Resource Manager et les scripts Desired State Configuration. Pour accéder aux ressources que vous gérez dans Azure DevOps, vous pouvez accorder ou refuser des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps, ou à Active Directory s’il est intégré à TFS.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

## <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.6 | 5.3 | Customer |

Si vous avez recours à des images personnalisées, utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour garantir que seuls les utilisateurs autorisés ont accès à ces images. Celle-ci vous permet de partager vos images avec différents utilisateurs, principaux de service ou groupes Active Directory au sein de votre organisation.  Pour les images conteneur, stockez-les dans Azure Container Registry et utilisez Azure RBAC pour vous assurer que seuls les utilisateurs autorisés peuvent accéder aux images.  

- [Présentation d’Azure RBAC](../../role-based-access-control/rbac-and-directory-admin-roles.md)

- [Comprendre Azure RBAC pour Container Registry](../../container-registry/container-registry-roles.md)

- [Comment configurer Azure RBAC](../../role-based-access-control/quickstart-assign-role-user-portal.md)

- [Présentation de Shared Image Gallery](../../virtual-machines/windows/shared-image-galleries.md)

## <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7,7 | 5.4 | Customer |

Définissez et implémentez des configurations de sécurité standard pour les ressources Azure à l’aide d’Azure Policy. Utilisez des alias Azure Policy pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration réseau de vos ressources Azure. Vous pouvez également utiliser des définitions de stratégie intégrées en lien avec vos ressources spécifiques.  Par ailleurs, vous pouvez utiliser Azure Automation pour déployer les changements de configuration.

- [Configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Utiliser des alias](../../governance/policy/concepts/definition-structure.md#aliases)

## <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer des outils de gestion de la configuration pour les systèmes d'exploitation

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.8 | 5.4 | Customer |

Azure Automation State Configuration est un service de gestion de la configuration pour les nœuds DSC dans n’importe quel cloud ou centre de données local. Vous pouvez facilement intégrer des machines, leur affecter des configurations déclaratives et afficher des rapports montrant la conformité de chaque machine à l’état souhaité que vous avez spécifié. 

- [Intégration des machines pour la gestion avec Azure Automation State Configuration](../../automation/automation-dsc-onboarding.md)

## <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.9 | 5.5 | Customer |

Utiliser Azure Security Center pour effectuer des analyses de ligne de base pour vos ressources Azure.  En outre, utilisez Azure Policy pour alerter et auditer les configurations des ressources Azure.

- [Corriger les recommandations dans Azure Security Center](../../security-center/security-center-remediate-recommendations.md)

## <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.10 | 5.5 | Customer |

Utilisez Azure Security Center pour effectuer des analyses de base pour les paramètres de système d’exploitation et d’ancrage des conteneurs.

- [Comprendre les recommandations concernant les conteneurs dans Azure Security Center](../../security-center/container-security.md)

## <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.11 | 13.1 | Customer |

Utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets pour vos applications Cloud.

- [Intégration aux identités managées Azure](../../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

- [Créer un coffre de clés](../../key-vault/secrets/quick-create-portal.md)

- [Comment s’authentifier auprès de Key Vault](../../key-vault/general/authentication.md)

- [Comment attribuer une stratégie d’accès Key Vault](../../key-vault/general/assign-access-policy-portal.md)

## <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.12 | 4,1 | Customer |

Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure AD. Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Key Vault, sans informations d’identification dans votre code.

- [Configurer des identités managées](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

## <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 7.13 | 18.1, 18.7 | Customer |

Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault. 

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)


## <a name="next-steps"></a>Étapes suivantes

- Reportez-vous au contrôle de sécurité suivant :  [Défense contre les programmes malveillants](security-control-malware-defense.md)