---
title: Activer la fonctionnalité Change Tracking et Inventory d’Azure Automation à partir d’une machine virtuelle Azure
description: Cet article explique comment activer la fonctionnalité Change Tracking et Inventory à partir d’une machine virtuelle Azure.
services: automation
ms.subservice: change-inventory-management
ms.date: 10/14/2020
ms.topic: conceptual
ms.openlocfilehash: 9b14dcb9ce2f2426d8d1496541022602a114cb6e
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92209041"
---
# <a name="enable-change-tracking-and-inventory-from-an-azure-vm"></a>Activer Change Tracking et Inventory à partir d’une machine virtuelle Azure

Cet article explique comment utiliser une machine virtuelle Azure pour activer la fonctionnalité [Suivi des modifications et inventaire](overview.md) sur d’autres machines. Pour activer des machines virtuelles Azure à grande échelle, vous devez activer une machine virtuelle existante à l’aide de la fonctionnalité Suivi des modifications et inventaire.

> [!NOTE]
> Lors de l’activation de la fonctionnalité Suivi des modifications et inventaire, seules certaines régions sont prises en charge pour la liaison d’un espace de travail Log Analytics et d’un compte Automation. Pour obtenir la liste des paires de mappages prises en charge, consultez [Mappage de régions pour un compte Automation et un espace de travail Log Analytics](../how-to/region-mappings.md).

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer [un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Compte Automation](../index.yml) pour gérer les machines.
* Une [machine virtuelle](../../virtual-machines/windows/quick-create-portal.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="enable-change-tracking-and-inventory"></a>Activer Change Tracking et Inventory

1. Dans le [Portail Azure](https://portal.azure.com), sélectionnez **Machines virtuelles** , ou recherchez et sélectionnez **Machines virtuelles** sur la page d’accueil.

2. Sélectionnez la machine virtuelle pour laquelle vous souhaitez activer Change Tracking et Inventory. Les machines virtuelles peuvent exister dans n’importe quelle région, quel que soit l’emplacement de votre compte Automation.

3. Dans la page de la machine virtuelle, sélectionnez **Inventory** ou **Change Tracking** sous **Gestion de la configuration** .

4. Vous devez disposer de l’autorisation `Microsoft.OperationalInsights/workspaces/read` pour déterminer si la machine virtuelle est activée pour un espace de travail. Pour en savoir plus sur les autorisations supplémentaires nécessaires, consultez [Autorisations de configuration de fonctionnalité](../automation-role-based-access-control.md#feature-setup-permissions). Pour savoir comment activer plusieurs machines en même temps, consultez [Activer Change Tracking et Inventory à partir d’un compte Automation](enable-from-automation-account.md).

5. Choisissez l’espace de travail Log Analytics et un compte Automation, puis cliquez sur **Activer** pour activer Change Tracking et Inventory pour la machine virtuelle. La configuration prend jusqu’à 15 minutes.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’utilisation de la fonctionnalité, consultez [Gérer le suivi des modifications](manage-change-tracking.md) et [Gérer l’inventaire](manage-inventory-vms.md).

* Pour résoudre des problèmes généraux liés à la fonctionnalité, consultez [Résoudre les problèmes rencontrés avec la fonctionnalité Suivi des modifications et inventaire](../troubleshoot/change-tracking.md).