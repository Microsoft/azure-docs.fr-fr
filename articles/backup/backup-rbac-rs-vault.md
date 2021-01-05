---
title: Gérer des sauvegardes avec le contrôle d’accès en fonction du rôle Azure
description: Le contrôle d’accès en fonction du rôle Azure permet de gérer l’accès aux opérations de gestion des sauvegardes dans le coffre Recovery Services.
ms.reviewer: utraghuv
ms.topic: conceptual
ms.date: 06/24/2019
ms.openlocfilehash: 0dd8d08c4ee79082f47929cf7d453f3f4bbd60ee
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92090877"
---
# <a name="use-azure-role-based-access-control-to-manage-azure-backup-recovery-points"></a>Utilisation du contrôle d’accès en fonction du rôle Azure pour gérer les points de récupération Sauvegarde Azure

Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) permet une gestion des accès affinée pour Azure. Avec Azure RBAC, vous pouvez séparer les tâches au sein de votre équipe et accorder aux utilisateurs uniquement les accès nécessaires pour accomplir leur travail.

> [!IMPORTANT]
> Les rôles fournis par Azure Backup sont limités à des actions qui peuvent être effectuées dans le portail Azure ou via l’API REST, le coffre Recovery Services Powershell, ou les cmdlets d’interface de ligne de commande. Les actions effectuées dans l’IU cliente de l’agent Sauvegarde Azure, l’IU System Center Data Protection Manager ou l’IU de serveur de sauvegarde Azure ne sont pas contrôlées par ces rôles.

Azure Backup fournit 3 rôles intégrés pour contrôler les opérations de gestion des sauvegardes. Apprenez-en davantage sur les [rôles intégrés Azure](../role-based-access-control/built-in-roles.md)

* [Contributeur de sauvegarde](../role-based-access-control/built-in-roles.md#backup-contributor) : ce rôle dispose de toutes les autorisations pour créer et gérer des sauvegardes, à l’exception de la suppression du coffre Recovery Services et de l’octroi d’un accès à d’autres personnes. Imaginez ce rôle comme un administrateur de gestion des sauvegardes qui peut exécuter chaque opération de gestion des sauvegardes.
* [Opérateur de sauvegarde](../role-based-access-control/built-in-roles.md#backup-operator) : ce rôle dispose des autorisations généralement accordées à un contributeur à l’exception de la suppression de sauvegardes et de la gestion des stratégies de sauvegarde. Ce rôle est équivalent au contributeur, mais il ne peut pas effectuer d’opérations destructrices telles que l’arrêt de la sauvegarde avec suppression des données ou la suppression de l’enregistrement de ressources locales.
* [Lecteur de sauvegarde](../role-based-access-control/built-in-roles.md#backup-reader) : ce rôle dispose des autorisations pour afficher toutes les opérations de gestion des sauvegardes. Imaginez ce rôle comme un responsable de surveillance.

Si vous avez besoin de définir vos propres rôles pour un meilleur contrôle, découvrez comment créer [des rôles personnalisés dans RBAC Azure](../role-based-access-control/custom-roles.md).

## <a name="mapping-backup-built-in-roles-to-backup-management-actions"></a>Mappage des rôles de sauvegarde intégrés pour les actions de gestion des sauvegardes

Le tableau suivant répertorie les actions de gestion des sauvegardes et le rôle Azure minimum nécessaire correspondant pour effectuer cette opération.

| Opération de gestion | Rôle Azure minimum nécessaire | Étendue requise |
| --- | --- | --- |
| Créer un coffre Recovery Services | Contributeur de sauvegarde | Groupe de ressources contenant le coffre |
| Activer la sauvegarde des machines virtuelles Azure | Opérateur de sauvegarde | Groupe de ressources contenant le coffre |
| | Contributeur de machine virtuelle | Ressource de machine virtuelle |
| Sauvegarde de machine virtuelle à la demande | Opérateur de sauvegarde | Coffre Recovery Services |
| Restaurer une machine virtuelle | Opérateur de sauvegarde | Coffre Recovery Services |
| | Contributeur | Groupe de ressources dans lequel la machine virtuelle sera déployée |
| | Contributeur de machine virtuelle | Machine virtuelle source qui a été sauvegardée |
| Restaurer la sauvegarde de machine virtuelle de disques non managés | Opérateur de sauvegarde | Coffre Recovery Services |
| | Contributeur de machine virtuelle | Machine virtuelle source qui a été sauvegardée |
| | Contributeur de compte de stockage | Ressource de compte de stockage où les disques vont être restaurés |
| Restaurer des disques managés de la sauvegarde de machine virtuelle | Opérateur de sauvegarde | Coffre Recovery Services |
| | Contributeur de machine virtuelle | Machine virtuelle source qui a été sauvegardée |
| | Contributeur de compte de stockage | Compte de stockage temporaire sélectionné dans le cadre de la restauration pour conserver les données de l’espace de stockage avant de les convertir en disques managés. |
| | Contributeur | Groupe de ressources vers lequel le(s) disque(s) managé(s) sera (seront) restauré(s) |
| Restaurer des fichiers individuels à partir d’une sauvegarde de machine virtuelle | Opérateur de sauvegarde | Coffre Recovery Services |
| | Contributeur de machine virtuelle | Machine virtuelle source qui a été sauvegardée |
| Créer une stratégie de sauvegarde pour la sauvegarde de machine virtuelle Azure | Contributeur de sauvegarde | Coffre Recovery Services |
| Modifier une stratégie de sauvegarde pour la sauvegarde de machine virtuelle Azure | Contributeur de sauvegarde | Coffre Recovery Services |
| Supprimer une stratégie de sauvegarde pour la sauvegarde de machine virtuelle Azure | Contributeur de sauvegarde | Coffre Recovery Services |
| Arrêt de la sauvegarde (avec conservation ou suppression des données) lors la sauvegarde d’une machine virtuelle | Contributeur de sauvegarde | Coffre Recovery Services |
| Inscrire un Windows Server/client/SCDPM ou un serveur de sauvegarde Azure | Opérateur de sauvegarde | Coffre Recovery Services |
| Supprimer un Windows Server/client/SCDPM inscrit ou un serveur de sauvegarde Azure | Contributeur de sauvegarde | Coffre Recovery Services |

> [!IMPORTANT]
> Si vous spécifiez Contributeur de machine virtuelle au niveau de l’étendue d’une ressource de machine virtuelle et sélectionnez **Sauvegarde** dans les paramètres de la machine virtuelle, l’écran **Activer la sauvegarde** s’ouvre même si la machine virtuelle est déjà sauvegardée. Cela est dû au fait que l’appel permettant de vérifier l’état de la sauvegarde fonctionne uniquement au niveau de l’abonnement. Pour éviter cela, accédez au coffre et ouvrez l’affichage des éléments de sauvegarde de la machine virtuelle ou spécifiez le rôle Contributeur de machine virtuelle au niveau de l’abonnement.

## <a name="minimum-role-requirements-for-the-azure-file-share-backup"></a>Exigences de rôle minimum pour la sauvegarde de partage de fichiers Azure

Le tableau suivant répertorie les actions de gestion des sauvegardes et le rôle nécessaire correspondant pour effectuer cette opération de partage Azure File.

| Opération de gestion | Rôle requis | Ressources |
| --- | --- | --- |
| Activer la sauvegarde d’un partage de fichiers Azure | Contributeur de sauvegarde |Coffre Recovery Services |
| |Compte de stockage | Ressource de compte de stockage de contributeur |
| Sauvegarde de machine virtuelle à la demande | Opérateur de sauvegarde | Coffre Recovery Services |
| Restaurer le partage de fichiers | Opérateur de sauvegarde | Coffre Recovery Services |
| | Contributeur de compte de stockage | Ressources du compte de stockage où les partages de fichiers source et cible de restauration sont présents |
| Restaurer des fichiers individuels | Opérateur de sauvegarde | Coffre Recovery Services |
| |Contributeur de compte de stockage|Ressources du compte de stockage où les partages de fichiers source et cible de restauration sont présents |
| Arrêter la protection |Contributeur de sauvegarde | Coffre Recovery Services |
| Annuler l’inscription d’un compte de stockage au coffre |Contributeur de sauvegarde | Coffre Recovery Services |
| |Contributeur de compte de stockage | Ressource du compte de stockage|

## <a name="next-steps"></a>Étapes suivantes

* [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md) : Découvrez comment bien démarrer avec Azure RBAC dans le portail Azure.
* Découvrez comment gérer l’accès avec :
  * [PowerShell](../role-based-access-control/role-assignments-powershell.md)
  * [Azure CLI](../role-based-access-control/role-assignments-cli.md)
  * [REST API](../role-based-access-control/role-assignments-rest.md)
* [Résolution des problèmes de contrôle d’accès en fonction du rôle Azure](../role-based-access-control/troubleshooting.md) : obtenez des suggestions pour résoudre les problèmes courants.
