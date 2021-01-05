---
title: Gérer le contrôle d’accès en fonction du rôle Azure dans Azure Site Recovery
description: Cet article décrit comment appliquer le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour gérer l’accès à Azure Site Recovery.
ms.service: site-recovery
ms.date: 04/08/2019
author: mayurigupta13
ms.topic: conceptual
ms.author: mayg
ms.openlocfilehash: d3e1334f513e8ac587d639758d83ce080c5b4ab9
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92516900"
---
# <a name="manage-site-recovery-access-with-azure-role-based-access-control-azure-rbac"></a>Gérez à la récupération de site avec le contrôle d’accès en fonction du rôle Azure (Azure RBAC)

Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) permet une gestion des accès affinée pour Azure. Avec Azure RBAC, vous pouvez séparer les responsabilités au sein de votre équipe et accorder uniquement des autorisations d’accès spécifiques aux utilisateurs en fonction des besoins pour effectuer des tâches spécifiques.

Azure Site Recovery fournit 3 rôles intégrés pour contrôler les opérations de gestion de Site Recovery. Apprenez-en davantage sur les [rôles intégrés Azure](../role-based-access-control/built-in-roles.md)

* [Site Recovery Contributor](../role-based-access-control/built-in-roles.md#site-recovery-contributor) - Ce rôle dispose de toutes les autorisations requises pour gérer les opérations d’Azure Site Recovery dans un coffre Recovery Services. Toutefois, un utilisateur disposant de ce rôle ne peut pas créer ou supprimer un coffre Recovery Services, ni affecter des droits d’accès à d’autres utilisateurs. Ce rôle est tout indiqué pour les administrateurs de récupération d’urgence, qui peuvent activer et gérer la récupération d’urgence pour des applications ou organisations, selon les cas.
* [Site Recovery Operator](../role-based-access-control/built-in-roles.md#site-recovery-operator) - Ce rôle dispose des autorisations d’exécution et de gestion des opérations de basculement et de restauration automatique. Un utilisateur disposant de ce rôle ne peut pas activer ou désactiver la réplication, créer ou supprimer des coffres, enregistrer une nouvelle infrastructure ou affecter des droits d’accès à d’autres utilisateurs. Ce rôle est tout indiqué pour un opérateur de récupération d’urgence, qui peut basculer des machines virtuelles ou des applications lorsqu’il en reçoit la demande des propriétaires et administrateurs informatiques, dans une situation d’urgence réelle ou simulée telle qu’un test de récupération d’urgence. Une fois l’incident résolu, l’opérateur de récupération d’urgence peut à nouveau protéger et restaurer les machines virtuelles.
* [Site Recovery Reader](../role-based-access-control/built-in-roles.md#site-recovery-reader) : Ce rôle dispose des autorisations pour afficher toutes les opérations de gestion de Site Recovery. Ce rôle est tout indiqué pour un responsable de suivi informatique, qui peut surveiller l’état de protection actuel et envoyer des billets d’assistance, si nécessaire.

Si vous avez besoin de définir vos propres rôles pour un meilleur contrôle, découvrez comment créer [des rôles personnalisés dans Azure](../role-based-access-control/custom-roles.md).

## <a name="permissions-required-to-enable-replication-for-new-virtual-machines"></a>Autorisations requises pour activer la réplication pour les nouvelles machines virtuelles
Lorsqu’une nouvelle machine virtuelle est répliquée vers Azure à l’aide d’Azure Site Recovery, les niveaux d’accès de l’utilisateur associé sont validés pour garantir que l’utilisateur dispose des autorisations requises pour utiliser les ressources Azure fournies à Site Recovery.

Pour activer la réplication pour une nouvelle machine virtuelle, un utilisateur doit avoir :
* L’autorisation de créer une machine virtuelle dans le groupe de ressources sélectionné
* L’autorisation de créer une machine virtuelle dans le réseau virtuel sélectionné
* L’autorisation d’écrire sur le compte de stockage sélectionné

Un utilisateur a besoin des autorisations suivantes pour effectuer la réplication d’une nouvelle machine virtuelle.

> [!IMPORTANT]
>Assurez-vous que les autorisations pertinentes sont ajoutées pour chaque modèle de déploiement (Resource Manager/classique) utilisé pour le déploiement des ressources.

> [!NOTE]
> Si vous activez la réplication pour une machine virtuelle Azure et souhaitez autoriser Site Recovery à gérer les mises à jour, tout en activant la réplication, vous pouvez également créer un nouveau compte Automation, auquel cas vous devrez obtenir l'autorisation de créer un compte Automation dans le même abonnement que le coffre.

| **Type de ressource** | **Modèle de déploiement** | **Permission** |
| --- | --- | --- |
| Calcul | Gestionnaire de ressources | Microsoft.Compute/availabilitySets/read |
|  |  | Microsoft.Compute/virtualMachines/read |
|  |  | Microsoft.Compute/virtualMachines/write |
|  |  | Microsoft.Compute/virtualMachines/delete |
|  | Classique | Microsoft.ClassicCompute/domainNames/read |
|  |  | Microsoft.ClassicCompute/domainNames/write |
|  |  | Microsoft.ClassicCompute/domainNames/delete |
|  |  | Microsoft.ClassicCompute/virtualMachines/read |
|  |  | Microsoft.ClassicCompute/virtualMachines/write |
|  |  | Microsoft.ClassicCompute/virtualMachines/delete |
| Réseau | Gestionnaire de ressources | Microsoft.Network/networkInterfaces/read |
|  |  | Microsoft.Network/networkInterfaces/write |
|  |  | Microsoft.Network/networkInterfaces/delete |
|  |  | Microsoft.Network/networkInterfaces/join/action |
|  |  | Microsoft.Network/virtualNetworks/read |
|  |  | Microsoft.Network/virtualNetworks/subnets/read |
|  |  | Microsoft.Network/virtualNetworks/subnets/join/action |
|  | Classique | Microsoft.ClassicNetwork/virtualNetworks/read |
|  |  | Microsoft.ClassicNetwork/virtualNetworks/join/action |
| Stockage | Gestionnaire de ressources | Microsoft.Storage/storageAccounts/read |
|  |  | Microsoft.Storage/storageAccounts/listkeys/action |
|  | Classique | Microsoft.ClassicStorage/storageAccounts/read |
|  |  | Microsoft.ClassicStorage/storageAccounts/listKeys/action |
| Groupe de ressources | Gestionnaire de ressources | Microsoft.Resources/deployments/* |
|  |  | Microsoft.Resources/subscriptions/resourceGroups/read |

Envisagez d’utiliser les [rôles intégrés](../role-based-access-control/built-in-roles.md) « Collaborateur de machine virtuelle » et « Collaborateur de machine virtuelle classique » pour les modèles de déploiement Resource Manager et classiques, respectivement.

## <a name="next-steps"></a>Étapes suivantes
* [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md) : Découvrez comment bien démarrer avec Azure RBAC dans le portail Azure.
* Découvrez comment gérer l’accès avec :
  * [PowerShell](../role-based-access-control/role-assignments-powershell.md)
  * [Azure CLI](../role-based-access-control/role-assignments-cli.md)
  * [REST API](../role-based-access-control/role-assignments-rest.md)
* [Résolution des problèmes liés à RBAC Azure](../role-based-access-control/troubleshooting.md) : obtenez des suggestions pour résoudre les problèmes courants.
