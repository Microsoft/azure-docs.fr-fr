---
title: Activer la fonctionnalité Suivi des modifications et inventaire d’Azure Automation à partir du portail Azure
description: Cet article explique comment activer la fonctionnalité Suivi des modifications et inventaire à partir du portail Azure.
services: automation
ms.subservice: change-inventory-management
ms.date: 10/14/2020
ms.topic: conceptual
ms.openlocfilehash: 6ca5b1df05c04937598278378e001f41f08ef23c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183632"
---
# <a name="enable-change-tracking-and-inventory-from-azure-portal"></a>Activer le Suivi des modifications et inventaire à partir du portail Azure

Cet article explique comment activer la fonctionnalité [Suivi des modifications et inventaire](overview.md) pour une ou plusieurs machines virtuelles Azure dans le portail Azure. Pour activer des machines virtuelles Azure à grande échelle, vous devez activer une machine virtuelle existante à l’aide de la fonctionnalité Suivi des modifications et inventaire.

Le nombre de groupes de ressources que vous pouvez utiliser pour la gestion de vos machines virtuelles est restreint par les [limites de déploiement Resource Manager](../../azure-resource-manager/templates/deploy-to-resource-group.md). Les déploiements Resource Manager sont limités à cinq groupes de ressources par déploiement. Deux de ces groupes de ressources sont réservés à la configuration de l’espace de travail Log Analytics, du compte Automation et des ressources associées. Vous disposez ainsi de trois groupes de ressources à sélectionner pour la gestion par le Suivi des modifications et inventaire. Cette limite s’applique uniquement à la configuration simultanée, et non au nombre de groupes de ressources qui peuvent être gérés par une fonctionnalité Automation.

> [!NOTE]
> Lors de l’activation de la fonctionnalité Suivi des modifications et inventaire, seules certaines régions sont prises en charge pour la liaison d’un espace de travail Log Analytics et d’un compte Automation. Pour obtenir la liste des paires de mappages prises en charge, consultez [Mappage de région pour un compte Automation et l’espace de travail Log Analytics](../how-to/region-mappings.md).

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer [un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Compte Automation](../index.yml) pour gérer les machines.
* Une [machine virtuelle](../../virtual-machines/windows/quick-create-portal.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à Azure sur https://portal.azure.com.

## <a name="enable-change-tracking-and-inventory"></a>Activer Change Tracking et Inventory

1. Dans le portail Azure, accédez à **Machines virtuelles**.

2. Utilisez les cases à cocher pour choisir les machines virtuelles à ajouter au Suivi des modifications et inventaire. Vous pouvez ajouter des machines pour trois différents groupes de ressources à la fois. Les machines virtuelles Azure peuvent exister dans n’importe quelle région, quel que soit l’emplacement de votre compte Automation.

    ![Liste des machines virtuelles](media/enable-from-portal/vmlist.png)

    > [!TIP]
    > Utilisez les contrôles de filtre pour sélectionner des machines virtuelles provenant de différents abonnements, emplacements et groupes de ressources. Vous pouvez cliquer sur la case à cocher supérieure pour sélectionner toutes les machines virtuelles dans une liste.

3. Sélectionnez **Suivi des modifications** ou **Inventaire** sous **Gestion de la configuration**.

4. La liste des machines virtuelles est filtrée pour afficher uniquement celles qui se trouvent dans le même abonnement et le même emplacement. Si vos machines virtuelles se trouvent dans plus de trois groupes de ressources, les trois premiers groupes de ressources sont sélectionnés.

5. Un espace de travail Log Analytics et un compte Automation existants sont sélectionnés par défaut. Si vous souhaitez utiliser un espace de travail Log Analytics et un compte Automation différents, cliquez sur **PERSONNALISER** pour les sélectionner sur la page Configuration personnalisée. Lorsque vous choisissez un espace de travail Log Analytics, une vérification est effectuée pour déterminer si ce dernier est lié à un compte Automation. Si un compte Automation lié est trouvé, l’écran suivant s’affiche. Une fois que vous avez terminé, cliquez sur **OK**.

    ![Sélectionner un espace de travail et un compte](media/enable-from-portal/select-workspace-and-account.png)

6. Si l’espace de travail sélectionné n’est pas lié à un compte Automation, l’écran suivant s’affiche. Sélectionnez un compte Automation, puis cliquez sur **OK** lorsque vous avez terminé.

    ![Aucun espace de travail](media/enable-from-portal/no-workspace.png)

7. Désactivez la case à cocher en regard des machines virtuelles que vous ne souhaitez pas activer. Les machines virtuelles qui ne peuvent pas être activées sont déjà désélectionnées.

8. Cliquez sur **Activer** pour activer la fonctionnalité que vous avez sélectionnée. Le configuration prend jusqu’à 15 minutes.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’utilisation de la fonctionnalité, consultez [Gérer le suivi des modifications](manage-change-tracking.md) et [Gérer l’inventaire](manage-inventory-vms.md).
* Pour résoudre des problèmes généraux liés à la fonctionnalité, consultez [Résoudre les problèmes rencontrés avec la fonctionnalité Suivi des modifications et inventaire](../troubleshoot/change-tracking.md).