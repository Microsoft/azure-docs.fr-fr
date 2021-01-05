---
title: Contrôle de sécurité Azure - Récupération des données
description: Contrôle de sécurité Azure – Récupération de données
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 835e4f681d514bb6b92caa5ee076e3794ed59236
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94698816"
---
# <a name="security-control-data-recovery"></a>Contrôle de sécurité : Récupération des données

Assurez-vous que toutes les données, configurations et secrets du système sont automatiquement sauvegardés de façon régulière.

## <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 9,1 | 10.1 | Customer |

Activez la Sauvegarde Azure et configurez la source de sauvegarde (machines virtuelles Azure, SQL Server ou partages de fichiers), ainsi que la fréquence souhaitée et la période de rétention.

- [Guide pratique pour activer la Sauvegarde Azure](../../backup/index.yml)

## <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 9.2 | 10.2 | Customer |

Activez la Sauvegarde Azure et la ou les machines virtuelles cibles, ainsi que la fréquence souhaitée et les périodes de rétention. Clés de sauvegarde gérées par le client dans Azure Key Vault.

- [Guide pratique pour activer la Sauvegarde Azure](../../backup/index.yml)

- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

## <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 9.3 | 10.3 | Customer |

Assurez-vous que la restauration des données du contenu s’effectue régulièrement dans Sauvegarde Azure. Testez la restauration des clés gérées par le client sauvegardées.

- [Guide pratique pour récupérer des fichiers à partir d’une sauvegarde de machines virtuelles Azure](../../backup/backup-azure-restore-files-from-vm.md)

- [Guide pratique pour restaurer des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

## <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 9,4 | 10,4 | Customer |

Pour la sauvegarde sur site, le chiffrement au repos est assuré à l’aide de la phrase secrète que vous fournissez lorsque vous sauvegardez sur Azure. Pour les machines virtuelles Azure, les données sont chiffrées au repos à l’aide de Storage Service Encryption (SSE). Utilisez un contrôle d’accès en fonction du rôle Azure pour protéger les sauvegardes et les clés gérées par le client.  

Activez la suppression réversible et la protection contre la purge dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.  Si le Stockage Azure est utilisé pour stocker les sauvegardes, la suppression réversible vous permet d’enregistrer et de récupérer vos données en cas de suppression d’objets blob ou d’instantanés d’objets blob. 

- [Présentation d’Azure RBAC](../../role-based-access-control/overview.md)

- [Guide pratique pour activer la suppression réversible et la protection contre la purge dans Key Vault](../../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

- [Suppression réversible pour les objets blob de Stockage Azure](../../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)


## <a name="next-steps"></a>Étapes suivantes

- Consultez le contrôle de sécurité suivant :  [Réponse aux incidents](security-control-incident-response.md)