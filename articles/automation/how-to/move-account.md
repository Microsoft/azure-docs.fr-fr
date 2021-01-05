---
title: Déplacer votre compte Azure Automation vers un autre abonnement
description: Cet article explique comment déplacer votre compte Automation vers un autre abonnement.
services: automation
ms.service: automation
ms.subservice: process-automation
author: mgoedtel
ms.author: magoedte
ms.date: 03/11/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 562ea5e0e9e4851ed59bd3ef917be2f9c48cd2a7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86185549"
---
# <a name="move-your-azure-automation-account-to-another-subscription"></a>Déplacer votre compte Azure Automation vers un autre abonnement

Azure Automation vous permet de déplacer certaines ressources vers un nouveau groupe de ressources ou un nouvel abonnement. Vous pouvez déplacer des ressources via le portail Azure, PowerShell, Azure CLI ou l’API REST. Pour en savoir plus sur le processus, consultez [Déplacer des ressources vers un nouveau groupe de ressources ou un nouvel abonnement](../../azure-resource-manager/management/move-resource-group-and-subscription.md).

Le compte Automation est l’une des ressources que vous pouvez déplacer. Dans cet article, vous allez apprendre à déplacer des comptes Automation vers une autre ressource ou un autre abonnement. Les étapes générales du déplacement de votre compte Automation sont les suivantes :

1. Désactivez vos fonctionnalités.
2. Dissociez votre espace de travail.
3. Déplacez le compte Automation.
4. Supprimez et recréez les comptes d’identification.
5. Réactivez vos fonctionnalités.

## <a name="remove-features"></a>Supprimer des fonctionnalités

Pour dissocier votre espace de travail de votre compte Automation, vous devez supprimer des ressources de fonctionnalités de votre espace de travail :

- Suivi des modifications et inventaire
- Update Management
- Démarrer/arrêter des machines virtuelles pendant les heures creuses

1. Dans le Portail Azure, localisez votre groupe de ressources.
2. Recherchez chaque fonctionnalité, puis sélectionnez **Supprimer** dans la page **Supprimer les ressources**.

    ![Capture d’écran de la suppression de ressources de fonctionnalités du portail Azure](../media/move-account/delete-solutions.png)

Si vous préférez, vous pouvez supprimer les solutions à l’aide de la cmdlet [Remove-AzResource](/powershell/module/Az.Resources/Remove-AzResource?view=azps-3.7.0) :

```azurepowershell-interactive
$workspaceName = <myWorkspaceName>
$resourceGroupName = <myResourceGroup>
Remove-AzResource -ResourceType 'Microsoft.OperationsManagement/solutions' -ResourceName "ChangeTracking($workspaceName)" -ResourceGroupName $resourceGroupName
Remove-AzResource -ResourceType 'Microsoft.OperationsManagement/solutions' -ResourceName "Updates($workspaceName)" -ResourceGroupName $resourceGroupName
Remove-AzResource -ResourceType 'Microsoft.OperationsManagement/solutions' -ResourceName "Start-Stop-VM($workspaceName)" -ResourceGroupName $resourceGroupName
```

### <a name="remove-alert-rules-for-startstop-vms-during-off-hours"></a>Supprimer des règles d’alerte de la solution Start/Stop VMs during off-hours

Pour la solution Start/Stop VMs during off-hours, vous devez également supprimer les règles d’alerte créées par la fonctionnalité.

1. Dans le portail Azure, accédez à votre groupe de ressources, puis sélectionnez **Supervision** > **Alertes** > **Gérer les règles d’alerte**.

   ![Capture d’écran de la page Alertes montrant la sélection effectuée dans Gérer les règles d’alerte](../media/move-account/alert-rules.png)

2. Dans la page Règles, vous devez voir une liste des alertes configurées dans ce groupe de ressources. La fonctionnalité crée les règles suivantes :

    * AutoStop_VM_Child
    * ScheduledStartStop_Parent
    * SequencedStartStop_Parent

3. Sélectionnez les règles l’une après l’autre, puis sélectionnez **Supprimer** pour les supprimer.

    ![Capture d’écran de la page Règles, demandant la confirmation de la suppression de règles sélectionnées](../media/move-account/delete-rules.png)

    > [!NOTE]
    > Si vous ne voyez aucune règle d’alerte dans la page Règles, définissez le champ **État** sur **Désactivé** pour afficher les alertes désactivées. 

4. Lorsque vous supprimez les règles d’alerte supprimées, vous devez supprimer le groupe d’actions créé pour les notifications de la solution Start/Stop VMs during off-hours. Dans le portail Azure, sélectionnez **Superviser** > **Alertes** > **Gérer les groupes d’actions**.

5. Sélectionnez **StartStop_VM_Notification**. 

6. Dans la page du groupe d’actions, sélectionnez **Supprimer**.

    ![Capture d’écran de la page Groupe d’actions](../media/move-account/delete-action-group.png)

Si vous préférez, vous pouvez supprimer votre groupe d’actions à l’aide de la cmdlet [Remove-AzActionGroup](/powershell/module/az.monitor/remove-azactiongroup?view=azps-3.7.0) :

```azurepowershell-interactive
Remove-AzActionGroup -ResourceGroupName <myResourceGroup> -Name StartStop_VM_Notification
```

## <a name="unlink-your-workspace"></a>Dissocier votre espace de travail

Vous pouvez maintenant dissocier votre espace de travail :

1. Dans le portail Azure, sélectionnez **Compte Automation** > **Ressources associées** > **Espace de travail lié**. 

2. Sélectionnez **Dissocier l’espace de travail** pour dissocier l’espace de travail de votre compte Automation.

    ![Capture d’écran de la dissociation d’un espace de travail d’un compte Automation](../media/move-account/unlink-workspace.png)

## <a name="move-your-automation-account"></a>Déplacer votre compte Automation

Vous pouvez maintenant déplacer votre compte Automation et ses runbooks. 

1. Dans le portail Azure, accédez au groupe de ressources de votre compte Automation. Sélectionnez **Déplacer** > **Déplacer vers un autre abonnement**.

    ![Capture d’écran de la page Groupe de ressources, déplacer vers un autre abonnement](../media/move-account/move-resources.png)

2. Sélectionnez les ressources de votre groupe de ressources que vous souhaitez déplacer. Veillez à inclure vos ressources du compte Automation, des runbooks et de l’espace de travail Log Analytics.

## <a name="re-create-run-as-accounts"></a>Recréer les comptes d’identification

Les [comptes d’identification](../manage-runas-account.md) permettent de créer un principal de service dans Azure Active Directory pour l’authentification auprès des ressources Azure. Quand vous changez d’abonnement, le compte Automation n’utilise plus le compte d’identification existant. Pour recréer les comptes d’identification :

1. Accédez à votre compte Automation dans le nouvel abonnement, puis sélectionnez **Comptes d’identification** sous **Paramètres de compte**. Comme vous pouvez le constater, les comptes d’identification apparaissent comme étant incomplets.

    ![Capture d’écran de Comptes d’identification, montrant ceux qui sont incomplets](../media/move-account/run-as-accounts.png)

2. Supprimez les comptes d’identification un par un en sélectionnant **Supprimer** sur la page **Propriétés**. 

    > [!NOTE]
    > Si vous n’avez pas les autorisations nécessaires pour créer ou voir les comptes d’identification, le message suivant s’affiche : `You do not have permissions to create an Azure Run As account (service principal) and grant the Contributor role to the service principal.` Pour plus d’informations, consultez [Autorisations nécessaires à la configuration des comptes d’identification](../manage-runas-account.md#permissions).

3. Une fois que vous avez supprimé les comptes d’identification, sélectionnez **Créer** sous **Compte d’identification Azure**. 

4. Dans la page Ajouter un compte d’identification Azure, sélectionnez **Créer** pour créer le compte d’identification et le principal de service. 

5. Répétez les étapes ci-dessus avec le compte d’identification Azure Classic.

## <a name="enable-features"></a>Activer des fonctionnalités

Après avoir recréé les comptes d’identification, vous devez réactiver les fonctionnalités que vous avez supprimées avant le déplacement :

1. Pour activer la fonctionnalité Suivi des modifications et inventaire, sélectionnez **Suivi des modifications et inventaire** dans votre compte Automation. Choisissez l’espace de travail Log Analytics que vous avez déplacé, puis sélectionnez **Activer**.

2. Répétez l’étape 1 pour Update Management.

    ![Capture d’écran de la réactivation des fonctionnalités dans votre compte Automation déplacé](../media/move-account/reenable-solutions.png)

3. Les machines activées avec vos fonctionnalités sont visibles une fois que vous avez connecté l’espace de travail Log Analytics existant. Pour activer la fonctionnalité Start/Stop VMs during off-hours, vous devez la réactiver. Sous **Ressources liées**, sélectionnez **Start/Stop VMs** > **En savoir plus sur la solution et l’activer** > **Créer** pour démarrer le déploiement.

4. Dans la page Ajouter une solution, choisissez votre espace de travail Log Analytics et votre compte Automation.

    ![Capture d’écran du menu Ajouter une solution](../media/move-account/add-solution-vm.png)

5. Configurez la fonctionnalité en procédant de la manière décrite dans [Vue d’ensemble de la solution Start/stop VMs during off-hours](../automation-solution-vm-management.md).

## <a name="verify-the-move"></a>Vérifier le déplacement

Une fois le déplacement terminé, vérifiez que les capacités répertoriées ci-dessous sont activées. 

|Fonctionnalité|Tests|Dépannage|
|---|---|---|
|Runbooks|Un runbook peut s’exécuter et se connecter correctement aux ressources Azure.|[Résolution des problèmes des runbooks](../troubleshoot/runbooks.md)
|Contrôle de code source|Vous pouvez exécuter une synchronisation manuelle sur votre référentiel de contrôle de code source.|[Intégration du contrôle de code source](../source-control-integration.md)|
|Suivi des modifications et inventaire|Vérifiez que les données d’inventaire actuelles s’affichent sur vos machines.|[Résoudre les problèmes liés au suivi des modifications](../troubleshoot/change-tracking.md)|
|Gestion des mises à jour|Vérifiez que vous voyez vos machines et qu’elles sont saines.</br>Exécutez un test de déploiement de mise à jour de logiciel.|[Résoudre les problèmes liés à Update Management](../troubleshoot/update-management.md)|
|Ressources partagées|Vérifiez que vous voyez toutes vos ressources partagées, telles que les [informations d’identification](../shared-resources/credentials.md) et les [variables](../shared-resources/variables.md).|

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir le déplacement de ressources dans Azure, consultez [Déplacer des ressources dans Azure](../../azure-resource-manager/management/move-support-resources.md).
