---
title: Activer Azure Monitor pour une machine virtuelle ou un groupe de machines virtuelles identiques dans le portail Azure
description: Découvrez comment activer Azure Monitor pour machines virtuelles sur une machine virtuelle ou un groupe de machines virtuelles identiques à l’aide du portail Azure.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: 55e5ff2af62c903efeab5c4932eae0c9dc9b535c
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94842309"
---
# <a name="enable-azure-monitor-for-single-virtual-machine-or-virtual-machine-scale-set-in-the-azure-portal"></a>Activer Azure Monitor pour une machine virtuelle ou un groupe de machines virtuelles identiques dans le portail Azure
Cet article explique comment activer Azure Monitor pour machines virtuelles pour une machine virtuelle ou un groupe de machines virtuelles identiques à l’aide du portail Azure. Cette procédure peut être utilisée pour les éléments suivants :

- Machine virtuelle Azure
- Groupe de machines virtuelles identiques Azure
- Machine virtuelle hybride connectée avec Azure Arc

## <a name="prerequisites"></a>Prérequis

- [Créer et configurer un espace de travail Log Analytics](vminsights-configure-workspace.md) Au cours de ce processus, vous pouvez également créer un espace de travail.
- Pour vous assurer que le système d’exploitation de la machine virtuelle ou du groupe de machines virtuelles identiques que vous activez est pris en charge, consulter [Systèmes d’exploitation pris en charge](vminsights-enable-overview.md#supported-operating-systems). 

## <a name="enable-azure-monitor-for-vms"></a>Activer Azure Monitor pour machines virtuelles

Sur le portail Azure, sélectionnez **Machines virtuelles**, **Groupes de machines virtuelles identiques** ou **Serveurs - Azure Arc** et sélectionnez une ressource dans la liste. Dans la section **Supervision** du menu, sélectionnez **Insights**, puis **Activer**. L’exemple suivant montre une machine virtuelle Azure, mais le menu est similaire pour un groupe de machines virtuelles identiques ou Azure Arc.

![Activer Azure Monitor pour machines virtuelles pour une machine virtuelle](media/vminsights-enable-single-vm/enable-vminsights-vm-portal.png)

Si la machine virtuelle n’est pas encore connectée à un espace de travail Log Analytics, vous êtes invité à en sélectionner un. Si vous n’avez pas encore [créé d’espace de travail](../../azure-monitor/learn/quick-create-workspace.md), vous pouvez sélectionner une valeur par défaut pour l’emplacement où la machine virtuelle ou le groupe de machines virtuelles identiques sont déployés dans l’abonnement. Cet espace de travail sera créé et configuré s’il n’existe pas encore. Si vous sélectionnez un espace de travail existant, il est configuré pour Azure Monitor pour machines virtuelles si ce n’était pas déjà le cas.

> [!NOTE]
> Si vous sélectionnez un espace de travail qui n’a pas été précédemment configuré pour Azure Monitor pour machines virtuelles, le pack d’administration *VMInsights* sera ajouté à cet espace de travail. Cela s’applique à tout agent déjà connectés à l’espace de travail, qu’il soit ou non activé pour Azure Monitor pour machines virtuelles. Les données de performances seront collectées à partir de ces machines virtuelles et stockées dans la table *InsightsMetrics*.

![Sélectionner un espace de travail](media/vminsights-configure-workspace/select-workspace.png)

Vous recevez des messages d’état à mesure que la configuration est effectuée.

>[!NOTE]
>Si vous utilisez un modèle de mise à niveau manuel pour votre groupe de machines virtuelles identiques, mettez à niveau les instances afin de terminer la configuration. Vous pouvez démarrer les mises à niveau depuis la page **Instances** de la section **Paramètres**.

![Activer Azure Monitor pour machines virtuelles surveillant le traitement du déploiement](media/vminsights-enable-single-vm/onboard-vminsights-vm-portal-status.png)



## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir les dépendances d’applications détectées, consultez [Utilisation de la fonctionnalité Map d’Azure Monitor pour machines virtuelles](vminsights-maps.md). 
* Pour identifier les goulots d’étranglement, l’utilisation globale et les performances de votre machine virtuelle, consultez [Afficher les performances des machines virtuelles Azure](vminsights-performance.md).
