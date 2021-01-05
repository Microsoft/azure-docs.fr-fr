---
title: Activer Azure Automation Update Management à partir d’une machine virtuelle Azure
description: Cet article explique comment activer Update Management à partir d’une machine virtuelle Azure.
services: automation
ms.date: 11/04/2020
ms.topic: conceptual
ms.custom: mvc
ms.openlocfilehash: 35feb136ca1fd7ea0be16a906f7d5193cf8570c7
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183547"
---
# <a name="enable-update-management-from-an-azure-vm"></a>Activer Update Management à partir d’une machine virtuelle Azure

Cet article explique comment activer la fonctionnalité [Update Management](overview.md) sur une ou plusieurs machines virtuelles Azure. Pour activer des machines virtuelles Azure à grande échelle, vous devez activer une machine virtuelle Azure existante en utilisant Update Management.

> [!NOTE]
> Lors de l’activation d’Update Management, seules certaines régions sont prises en charge pour la liaison d’un espace de travail Log Analytics et d’un compte Automation. Pour obtenir la liste des paires de mappages prises en charge, consultez [Mappage de régions pour un compte Automation et un espace de travail Log Analytics](../how-to/region-mappings.md).

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer [un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Compte Automation](../index.yml) pour gérer les machines.
* Une [machine virtuelle](../../virtual-machines/windows/quick-create-portal.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="enable-the-feature-for-deployment"></a>Activer la fonctionnalité pour le déploiement

1. Dans le [Portail Azure](https://portal.azure.com), sélectionnez **Machines virtuelles**, ou recherchez et sélectionnez **Machines virtuelles** sur la page d’accueil.

2. Sélectionnez la machine virtuelle pour laquelle vous souhaitez activer Update Management. Les machines virtuelles peuvent exister dans n’importe quelle région, quel que soit l’emplacement de votre compte Automation. Vous

3. Sur la page de la machine virtuelle, sous **Opérations**, sélectionnez **Mises à jour de l’hôte et de l’invité**.

    ![Sélectionner Mises à jour de l’hôte et de l’invité dans le volet de gauche](media/enable-from-vm/select-guest-and-os-updates.png)

4. Vous devez disposer de l’autorisation `Microsoft.OperationalInsights/workspaces/read` pour déterminer si la machine virtuelle est activée pour un espace de travail. Pour en savoir plus sur les autorisations supplémentaires nécessaires, consultez les [autorisations nécessaires pour activer des machines](../automation-role-based-access-control.md#feature-setup-permissions). Pour savoir comment activer plusieurs machines en même temps, consultez [Activer Update Management à partir d’un compte Automation](./enable-from-automation-account.md).

5. Sur la page Activer Update Management, choisissez l’espace de travail Log Analytics et un compte Automation, puis cliquez sur **Activer** pour activer Update Management. Après l’activation d’Update Management, l’affichage de l’évaluation des mises à jour à partir de la machine virtuelle peut prendre environ 15 minutes.

    ![Activer Update Management](media/enable-from-vm/enable-update-management.png)

## <a name="next-steps"></a>Étapes suivantes

* Pour utiliser Update Management pour des machines virtuelles, consultez [Gérer les mises à jour et les correctifs pour vos machines virtuelles Azure](manage-updates-for-vm.md).

* Pour résoudre les erreurs générales d’Update Management, consultez [Résoudre les problèmes liés à Update Management](../troubleshoot/update-management.md).