---
title: Activer Azure Automation Update Management à partir du portail Azure
description: Cet article explique comment activer Update Management à partir du portail Azure.
services: automation
ms.date: 04/11/2019
ms.topic: article
ms.custom: mvc
ms.openlocfilehash: ac402587c8346c14f34b683ba10832c7779b660b
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182765"
---
# <a name="enable-update-management-from-the-azure-portal"></a>Activer Update Management à partir du portail Azure

Cet article explique comment activer la fonctionnalité [Update Management](overview.md) pour les machines virtuelles en parcourant le portail Azure. Pour activer des machines virtuelles Azure à grande échelle, vous devez activer une machine virtuelle Azure existante en utilisant Update Management.

Le nombre de groupes de ressources que vous pouvez utiliser pour la gestion de vos machines virtuelles est restreint par les [limites de déploiement Resource Manager](../../azure-resource-manager/templates/deploy-to-resource-group.md). Les déploiements Resource Manager (à ne pas confondre avec les déploiements de mises à jour) sont limités à cinq groupes de ressources par déploiement. Deux de ces groupes de ressources sont réservés à la configuration de l’espace de travail Log Analytics, du compte Automation et des ressources associées. Vous disposez ainsi de trois groupes de ressources à sélectionner pour la gestion par Update Management. Cette limite s’applique uniquement à la configuration simultanée, et non au nombre de groupes de ressources qui peuvent être gérés par une fonctionnalité Automation.

> [!NOTE]
> Lors de l’activation d’Update Management, seules certaines régions sont prises en charge pour la liaison d’un espace de travail Log Analytics et d’un compte Automation. Pour obtenir la liste des paires de mappages prises en charge, consultez [Mappage de région pour un compte Automation et l’espace de travail Log Analytics](../how-to/region-mappings.md).

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer [un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Compte Automation](../index.yml) pour gérer les machines.
* Une [machine virtuelle](../../virtual-machines/windows/quick-create-portal.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à Azure sur https://portal.azure.com.

## <a name="enable-update-management"></a>Activer Update Management

1. Dans le portail Azure, accédez à **Machines virtuelles**.

2. Dans la page **Machines virtuelles**, utilisez les cases à cocher pour choisir les machines virtuelles à ajouter à Update Management. Vous pouvez ajouter des machines pour trois différents groupes de ressources à la fois. Les machines virtuelles Azure peuvent exister dans n’importe quelle région, quel que soit l’emplacement de votre compte Automation.

    ![Liste des machines virtuelles](media/enable-from-portal/vmlist.png)

    > [!TIP]
    > Utilisez les contrôles de filtre pour sélectionner des machines virtuelles provenant de différents abonnements, emplacements et groupes de ressources. Vous pouvez cliquer sur la case à cocher supérieure pour sélectionner toutes les machines virtuelles dans une liste.

    [ ![Activer Update Management](./media/enable-from-portal/onboard-feature.png)](./media/enable-from-portal/onboard-feature-expanded.png#lightbox)

3. Sélectionnez **Services**, puis **Update Management** pour la fonctionnalité Update Management.

4. La liste des machines virtuelles est filtrée pour afficher uniquement celles qui se trouvent dans le même abonnement et le même emplacement. Si vos machines virtuelles se trouvent dans plus de trois groupes de ressources, les trois premiers groupes de ressources sont sélectionnés.

5. Un espace de travail Log Analytics et un compte Automation existants sont sélectionnés par défaut. Si vous souhaitez utiliser un espace de travail Log Analytics et un compte Automation différents, cliquez sur **PERSONNALISER** pour les sélectionner dans la page Configuration personnalisée. Lorsque vous choisissez un espace de travail Log Analytics, une vérification est effectuée pour déterminer si ce dernier est lié à un compte Automation. Si un compte Automation lié est trouvé, l’écran suivant s’affiche. Lorsque vous avez terminé, sélectionnez **OK**.

    [ ![Sélectionner un espace de travail et un compte](./media/enable-from-portal/select-workspace-and-account.png)](./media/enable-from-portal/select-workspace-and-account-expanded.png#lightbox)

6. Si l’espace de travail sélectionné n’est pas lié à un compte Automation, l’écran suivant s’affiche. Sélectionnez un compte Automation, puis **OK** lorsque vous avez terminé.

    ![Aucun espace de travail](media/enable-from-portal/no-workspace.png)

7. Désélectionnez les machines virtuelles que vous ne souhaitez pas activer. Les machines virtuelles qui ne peuvent pas être activées sont déjà désélectionnées.

8. Sélectionnez **Activer** pour activer la fonctionnalité. Après l’activation d’Update Management, l’affichage de l’évaluation des mises à jour peut prendre environ 15 minutes.

## <a name="next-steps"></a>Étapes suivantes

* Pour utiliser Update Management pour des machines virtuelles, consultez [Gérer les mises à jour et les correctifs pour vos machines virtuelles](manage-updates-for-vm.md).
* Pour résoudre les erreurs générales d’Update Management, consultez [Résoudre les problèmes liés à Update Management](../troubleshoot/update-management.md).
* Pour résoudre les problèmes liés à l’agent de mise à jour Windows, consultez [Résoudre les problèmes de l’agent de mise à jour Windows](../troubleshoot/update-agent-issues.md).
* Pour résoudre les problèmes liés à l’agent de mise à jour Linux, consultez [Résoudre les problèmes de l’agent de mise à jour Linux](../troubleshoot/update-agent-issues-linux.md).