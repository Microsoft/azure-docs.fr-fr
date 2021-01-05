---
title: Utilisation de groupes dynamiques avec Azure Automation Update Management
description: Cet article décrit le fonctionnement des groupes dynamiques avec Azure Automation Update Management.
services: automation
ms.subservice: update-management
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: a2df2489e4b216b4b33524e35691bf93468edda0
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221539"
---
# <a name="use-dynamic-groups-with-update-management"></a>Utiliser des groupes dynamiques avec Update Management

Update Management permet de cibler un groupe dynamique de machines virtuelles Azure ou autres dans le cadre d’un déploiement de mises à jour. L’utilisation d’un groupe dynamique vous évite d’avoir à modifier votre déploiement pour mettre à jour les machines.

> [!NOTE]
> Les groupes dynamiques ne fonctionnent pas avec des machines virtuelles classiques.

Vous pouvez définir des groupes dynamiques pour des machines Azure ou non-Azure à partir de **Gestion des mises à jour** dans le portail Azure. Consultez [Gérer les mises à jour pour les machines virtuelles](manage-updates-for-vm.md).

Un groupe dynamique est défini par une requête qu’Azure Automation évalue au moment du déploiement. Même si une requête de groupe dynamique récupère un grand nombre de machines, Azure Automation ne peut traiter qu’un maximum de 1 000 machines à la fois. Consultez [Abonnement Azure et limites, quotas et contraintes de service](../../azure-resource-manager/management/azure-subscription-service-limits.md#update-management).

> [!NOTE]
> Si vous envisagez de mettre à jour plus de 1 000 machines, nous vous recommandons de répartir les mises à jour entre plusieurs planifications de mise à jour. 

## <a name="define-dynamic-groups-for-azure-machines"></a>Définir des groupes dynamiques pour les machines Azure

Lorsque vous définissez une requête de groupe dynamique pour des machines Azure, vous pouvez utiliser les éléments suivants qui permettent de renseigner le groupe dynamique :

* Abonnement
* Groupes de ressources
* Emplacements
* Balises

![Sélection de groupes](./media/configure-groups/select-groups.png)

Pour afficher un aperçu des résultats de votre requête de groupe dynamique, cliquez sur **Aperçu**. L’aperçu montre l’appartenance aux groupes à l’instant t. Dans l’exemple, nous recherchons les machines comportant la balise `Role` pour le groupe **BackendServer**. Si plusieurs machines présentent cette balise, elles sont ajoutées à tous les déploiements ultérieurs effectués pour ce groupe.

![aperçu des groupes](./media/configure-groups/preview-groups.png)

## <a name="define-dynamic-groups-for-non-azure-machines"></a>Définir des groupes dynamiques pour les machines non-Azure

Un groupe dynamique pour les machines non-Azure utilise des recherches enregistrées, également appelées groupes d’ordinateurs. Pour apprendre à créer une recherche enregistrée, consultez [Création d’un groupe d’ordinateurs](../../azure-monitor/platform/computer-groups.md#creating-a-computer-group). Une fois votre recherche enregistrée créée, vous pouvez la sélectionner dans la liste des recherches enregistrées au niveau de **Gestion des mises à jour** dans le portail Azure. Cliquez sur **Aperçu** pour afficher un aperçu des ordinateurs figurant dans la recherche enregistrée.

![Capture d’écran montrant la page Sélectionner des groupes pour les machines non Azure (préversion) et le volet de visualisation sur le côté droit.](./media/configure-groups/select-groups-2.png)

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez [interroger des journaux d’Azure Monitor](query-logs.md) pour analyser les évaluations de mise à jour, les déploiements et autres tâches de gestion associées. Vous trouverez des requêtes prédéfinies pour vous aider à commencer.
