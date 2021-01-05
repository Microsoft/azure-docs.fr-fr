---
title: Benchmark de sécurité Azure V2 – Sauvegarde et récupération
description: Benchmark de sécurité Azure V2 pour la sauvegarde et la récupération
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/20/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 089cf521a7c5428833be340001c88b870c568a8f
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368883"
---
# <a name="security-control-v2-backup-and-recovery"></a>Contrôle de sécurité V2 : Sauvegarde et récupération

La sauvegarde et la récupération recouvrent les contrôles destinés à s’assurer que les sauvegardes de données et de configurations aux différents niveaux de service sont effectuées, validées et protégées.

## <a name="br-1-ensure-regular-automated-backups"></a>BR-1 : Garantir des sauvegardes automatiques régulières

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| BR-1 | 10.1 | CP-2, CP4, CP-6, CP-9 |

Veillez à sauvegarder les systèmes et les données pour assurer la continuité de l’activité après un événement inattendu. Ceci doit être défini par des objectifs de point de récupération (RPO) et de délai de récupération (RTO).

Activez le service Sauvegarde Azure et configurez la source de la sauvegarde (par exemple, machines virtuelles Azure, SQL Server ou partages de fichiers), ainsi que la fréquence et la période de rétention souhaitées.  

Pour un niveau plus élevé de protection, vous pouvez activer l’option de stockage géoredondant afin de répliquer les données de sauvegarde dans une région secondaire et de les récupérer à l’aide d’une restauration inter-région.

- [Continuité d’activité et reprise d’activité à l’échelle de l’entreprise](/azure/cloud-adoption-framework/ready/enterprise-scale/business-continuity-and-disaster-recovery)

- [Guide pratique pour activer la Sauvegarde Azure](../../backup/index.yml)

- [Comment activer la restauration inter-région ?](../../backup/backup-azure-arm-restore-vms.md#cross-region-restore)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Stratégie et normes](/azure/cloud-adoption-framework/organize/cloud-security-policy-standards)

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Préparation aux incidents](/azure/cloud-adoption-framework/organize/cloud-security-incident-preparation)

## <a name="br-2-encrypt-backup-data"></a>BR-2 : Chiffrer les données de sauvegarde

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| BR-2 | 10,2 | CP-9 |

Assurez-vous que vos sauvegardes sont protégées contre les attaques. Cela doit inclure le chiffrement des sauvegardes afin de vous protéger contre la perte de confidentialité.   

Pour une sauvegarde locale à l’aide du service Sauvegarde Azure, le chiffrement au repos est assuré à l’aide de la phrase secrète que vous fournissez. Pour une sauvegarde de service Azure normale, les données de sauvegarde sont automatiquement chiffrées à l’aide de clés gérées par la plateforme Azure. Vous pouvez choisir de chiffrer la sauvegarde à l’aide d’une clé gérée par le client. Dans ce cas, assurez-vous que cette clé gérée par le client dans le coffre de clés est également dans l’étendue de la sauvegarde. 

Utilisez un contrôle d’accès en fonction du rôle Azure dans le service Sauvegarde Azure, la solution Azure Key Vault ou d’autres ressources pour protéger les sauvegardes et les clés gérées par le client. En outre, vous pouvez activer des fonctionnalités de sécurité avancées pour exiger une authentification multifacteur avant que les sauvegardes soient modifiées ou supprimées.

- [Vue d’ensemble des fonctionnalités de sécurité de Sauvegarde Azure](../../backup/security-overview.md)

- [Chiffrement des données de sauvegarde à l’aide de clés gérées par le client](../../backup/encryption-at-rest-with-cmk.md) 

- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

- [Fonctionnalités de sécurité pour la protection de sauvegardes hybrides contre des attaques](../../backup/backup-azure-security-feature.md#prevent-attacks)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Préparation aux incidents](/azure/cloud-adoption-framework/organize/cloud-security-incident-preparation)

## <a name="br-3-validate-all-backups-including-customer-managed-keys"></a>BR-3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| BR-3 | 10.3 | CP-4, CP-9 |

Effectuez régulièrement une restauration des données de votre sauvegarde. Assurez-vous que vous pouvez restaurer les clés gérées par le client sauvegardées.

- [Guide pratique pour récupérer des fichiers à partir d’une sauvegarde de machines virtuelles Azure](../../backup/backup-azure-restore-files-from-vm.md)

- [Guide pratique pour restaurer des clés du coffre de clés dans Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Préparation aux incidents](/azure/cloud-adoption-framework/organize/cloud-security-incident-preparation)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="br-4-mitigate-risk-of-lost-keys"></a>BR-4 : Atténuer les risques liés aux clés perdues

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| BR-4 | 10,4 | CP-9 |

Assurez-vous que vous avez pris les mesures nécessaires pour empêcher la perte de clés et récupérer d’une telle perte. Activez la suppression réversible et la protection contre la purge dans Azure Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.  

- [Guide pratique pour activer la suppression réversible et la protection contre la purge dans Key Vault](../../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Préparation aux incidents](/azure/cloud-adoption-framework/organize/cloud-security-incident-preparation)

- [Sécurité des données](/azure/cloud-adoption-framework/organize/cloud-security-data-security)