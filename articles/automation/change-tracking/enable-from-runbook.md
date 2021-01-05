---
title: Activer la fonctionnalité Suivi des modifications et inventaire d’Azure Automation à partir d’un runbook
description: Cet article explique comment activer la fonctionnalité Suivi des modifications et inventaire à partir d’un runbook.
services: automation
ms.subservice: change-inventory-management
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: 842b0a92ba4a2cb6b3ceb54675ef95f9c8275311
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92209059"
---
# <a name="enable-change-tracking-and-inventory-from-a-runbook"></a>Activer Change Tracking et Inventory à partir d’un runbook

Cet article explique comment utiliser un runbook pour activer la fonctionnalité [Suivi des modifications et inventaire](overview.md) pour les machines virtuelles de votre environnement. Pour activer des machines virtuelles Azure à grande échelle, vous devez activer une machine virtuelle existante à l’aide de la fonctionnalité Suivi des modifications et inventaire.

> [!NOTE]
> Lors de l’activation de la fonctionnalité Suivi des modifications et inventaire, seules certaines régions sont prises en charge pour la liaison d’un espace de travail Log Analytics et d’un compte Automation. Pour obtenir la liste des paires de mappages prises en charge, consultez [Mappage de régions pour un compte Automation et un espace de travail Log Analytics](../how-to/region-mappings.md).

Cette méthode utilise deux runbooks :

* **Enable-MultipleSolution**  : runbook principal qui invite à entrer les informations de configuration, interroge la machine virtuelle spécifiée et effectue d’autres vérifications de validation, puis qui appelle le runbook **Enable-AutomationSolution** pour configurer la fonctionnalité Suivi des modifications et inventaire pour chaque machine virtuelle au sein du groupe de ressources spécifié.
* **Enable-AutomationSolution**  : active la fonctionnalité Suivi des modifications et inventaire pour une ou plusieurs machines virtuelles spécifiées dans le groupe de ressources cible. Il vérifie que les conditions préalables sont remplies, vérifie que l’extension de machine virtuelle Log Analytics est installée et l’installe le cas échéant et ajoute les machines virtuelles à la configuration d’étendue dans l’espace de travail Log Analytics spécifié lié au compte Automation.

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer [un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Compte Automation](../automation-security-overview.md) pour gérer les machines.
* [Espace de travail Log Analytics](../../azure-monitor/platform/design-logs-deployment.md)
* Une [machine virtuelle](../../virtual-machines/windows/quick-create-portal.md).
* Deux ressources Automation, qui sont utilisées par le runbook **Enable-AutomationSolution** . Ce runbook, s’il n’existe pas déjà dans votre compte Automation, est automatiquement importé par le runbook **Enable-MultipleSolution** lors de sa première exécution.
    * *LASolutionSubscriptionId*  : ID d’abonnement de l’emplacement où se trouve l’espace de travail Log Analytics.
    * *LASolutionWorkspaceId*  : ID de l’espace de travail Log Analytics lié à votre compte Automation.

    Ces variables sont utilisées pour configurer l’espace de travail de la machine virtuelle intégrée. Si elles ne sont pas spécifiées, le script recherche d’abord toute machine virtuelle intégrée à la fonctionnalité Suivi des modifications et inventaire dans son abonnement, puis l’abonnement dans lequel se trouve le compte Automation et enfin tous les autres abonnements auxquels votre compte d’utilisateur a accès. Si l’espace de travail n’est pas correctement configuré, cela peut entraîner l’intégration de vos machines à un espace de travail Log Analytics aléatoire.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="enable-change-tracking-and-inventory"></a>Activer Change Tracking et Inventory

1. Dans le portail Azure, accédez à **Comptes Automation** . Sur la page **Comptes Automation** , sélectionnez votre compte dans la liste.

1. Dans votre compte Automation, sélectionnez **Inventaire** ou **Change Tracking** sous **Gestion de la configuration** .

1. Sélectionnez l’espace de travail Log Analytics, puis cliquez sur **Activer** . Lorsque Inventaire ou Change Tracking est activé, une bannière s’affiche.

    ![Activer Change Tracking et Inventory](media/enable-from-automation-account/enable-feature.png)

## <a name="install-and-update-modules"></a>Installer et mettre à jour les modules

Vous devez effectuer la mise à jour vers les derniers modules Azure et importer le module [Az.OperationalInsights](/powershell/module/az.operationalinsights) afin d’activer correctement Update Management pour vos machines virtuelles à l’aide du runbook.

1. Dans votre compte Automation, sélectionnez **Modules** sous **Ressources partagées** .

2. Sélectionnez **Mettre à jour les modules Azure** pour mettre à jour les modules Azure vers la dernière version.

3. Cliquez sur **Oui** pour mettre à jour tous les modules Azure existants vers la dernière version.

    ![Mettre à jour les modules](media/enable-from-runbook/update-modules.png)

4. Revenez à **Modules** sous **Ressources partagées** .

5. Sélectionnez **Parcourir la galerie** pour ouvrir la galerie des modules.

6. Recherchez `Az.OperationalInsights` et importez ce module dans votre compte Automation.

    ![Importer le module OperationalInsights](media/enable-from-runbook/import-operational-insights-module.png)

## <a name="select-azure-vm-to-manage"></a>Sélectionner la machine virtuelle Azure à gérer

Avec la fonctionnalité Suivi des modifications et inventaire, vous pouvez ajouter une machine virtuelle Azure pour la gérer via cette fonction.

1. À partir de votre compte Automation, sélectionnez **Suivi des modifications** ou **Inventaire** sous **Gestion de la configuration** .

2. Cliquez sur **Ajouter des machines virtuelles Azure** pour ajouter votre machine virtuelle.

3. Sélectionnez votre machine virtuelle dans la liste, puis cliquez sur **Activer** . Cette action active la fonction Suivi des modifications et inventaire pour la machine virtuelle.

   ![Activer la fonction Suivi des modifications et inventaire pour une machine virtuelle](media/enable-from-runbook/enable-azure-vm.png)

    > [!NOTE]
    > Si vous essayez d’activer une autre fonctionnalité avant la fin de la configuration de Suivi des modifications et inventaire, ce message s’affiche : `Installation of another solution is in progress on this or a different virtual machine. When that installation completes the Enable button is enabled, and you can request installation of the solution on this virtual machine.`

## <a name="import-a-runbook-to-enable-change-tracking-and-inventory"></a>Importer un runbook pour activer la fonctionnalité Suivi des modifications et inventaire

1. Dans votre compte Automation, sélectionnez **Runbooks** sous **Automatisation des processus** .

2. Sélectionnez **Parcourir la galerie** .

3. Recherchez **les mises à jour et le suivi des modifications** .

4. Sélectionnez le runbook, puis cliquez sur **Importer** dans la page **Afficher la source** .

5. Cliquez sur **OK** pour importer le runbook dans le compte Automation.

   ![Importer un runbook pour la configuration](media/enable-from-runbook/import-from-gallery.png)

6. Sur la page **Runbook** , sélectionnez le runbook **Enable-MultipleSolution** , puis cliquez sur **Modifier** . Dans l’éditeur de texte, sélectionnez **Publier** .

7. Lorsque vous êtes invité à confirmer, cliquez sur **Oui** pour publier le runbook.

## <a name="start-the-runbook"></a>Démarrer le runbook

Pour démarrer ce runbook, vous devez avoir activé Suivi des modifications et inventaire pour une machine virtuelle. La fonctionnalité doit être activée sur une machine virtuelle et un groupe de ressources existants pour pouvoir configurer une ou plusieurs machines virtuelles dans le groupe de ressources cible.

1. Ouvrez le runbook **Enable-MultipleSolution** .

   ![Runbook avec plusieurs solutions](media/enable-from-runbook/runbook-overview.png)

2. Cliquez sur le bouton Démarrer et entrez les valeurs des paramètres dans les champs suivants :

   * **VMNAME** : nom d’une machine virtuelle existante à ajouter à la fonctionnalité Suivi des modifications et inventaire. Laissez ce champ vide pour ajouter toutes les machines virtuelles du groupe de ressources.
   * **VMRESOURCEGROUP** : nom du groupe de ressources des machines virtuelles à activer.
   * **SUBSCRIPTIONID** : ID d’abonnement de la nouvelle machine virtuelle à activer. Laissez ce champ vide pour utiliser l’abonnement de l’espace de travail. Quand vous utilisez un ID d’abonnement différent, ajoutez le compte d’identification de votre compte Automation en tant que contributeur pour l’abonnement.
   * **ALREADYONBOARDEDVM**  : nom de la machine virtuelle qui est déjà activée manuellement pour les mises à jour.
   * **ALREADYONBOARDEDVMRESOURCEGROUP**  : Nom du groupe de ressources auquel appartient la machine virtuelle.
   * **SOLUTIONTYPE** : entrez **ChangeTracking** .

   ![Paramètres du runbook Enable-MultipleSolution](media/enable-from-runbook/runbook-parameters.png)

3. Sélectionnez **OK** pour démarrer le travail du runbook.

4. Surveillez la progression du travail de runbook et les erreurs éventuelles dans la page **Travaux** .

## <a name="next-steps"></a>Étapes suivantes

* Pour planifier un runbook, consultez [Gérer les planifications dans Azure Automation](../shared-resources/schedules.md).

* Pour plus d’informations sur l’utilisation de la fonctionnalité, consultez [Gérer Change Tracking](manage-change-tracking.md) et [Gérer Inventaire](manage-inventory-vms.md).

* Pour résoudre des problèmes généraux liés à la fonctionnalité, consultez [Résoudre les problèmes rencontrés avec la fonctionnalité Suivi des modifications et inventaire](../troubleshoot/change-tracking.md).


