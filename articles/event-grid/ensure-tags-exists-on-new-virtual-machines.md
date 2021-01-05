---
title: Intégrer Azure Automation à Event Grid | Microsoft Docs
description: Découvrez comment ajouter automatiquement une balise lors de la création d’une nouvelle machine virtuelle et envoyer une notification à Microsoft Teams.
keywords: automation, runbook, teams, event grid, machine virtuelle, VM
services: automation,event-grid
author: eamonoreilly
ms.service: automation
ms.topic: tutorial
ms.workload: infrastructure-services
ms.date: 07/07/2020
ms.author: eamono
ms.openlocfilehash: 3b9b49a4d38566891f442a3d2d7eac9bf1d36465
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87462001"
---
# <a name="tutorial-integrate-azure-automation-with-event-grid-and-microsoft-teams"></a>Tutoriel : Intégrer Azure Automation à Event Grid et Microsoft Teams

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Importer un exemple de runbook Event Grid.
> * Créer un Webhook Microsoft Teams en option.
> * Créer un Webhook pour le runbook.
> * Créer un abonnement Event Grid.
> * Créer une machine virtuelle qui déclenche le runbook.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [requires-azurerm](../../includes/requires-azurerm.md)]

Un [compte Azure Automation](../automation/index.yml) est requis pour terminer ce didacticiel et mettre le runbook déclenché par l’abonnement Azure Event Grid en attente.

* Le module `AzureRM.Tags` doit être chargé dans votre compte Automation, consultez [How to import modules in Azure Automation](../automation/automation-update-azure-modules.md) (Importation de modules dans Azure Automation) pour apprendre à importer des modules dans Azure Automation.

## <a name="import-an-event-grid-sample-runbook"></a>Importer un exemple de runbook Event Grid

1. Sélectionnez votre compte Automation puis cliquez sur la page **Runbooks**.

   ![Sélectionner des runbooks](./media/ensure-tags-exists-on-new-virtual-machines/select-runbooks.png)

2. Cliquez sur le bouton **Parcourir la galerie**.

3. Recherchez **Event Grid** et sélectionnez **Intégration d’Azure Automation à Event grid**.

    ![Importer le runbook de la galerie](media/ensure-tags-exists-on-new-virtual-machines/gallery-event-grid.png)

4. Sélectionnez **Importer** et nommez-le **Watch-VMWrite**.

5. Une fois importé, sélectionnez **Modifier** pour afficher la source du runbook. 
6. Mettez à jour la ligne 74 du script pour utiliser `Tag` plutôt que `Tags`.

    ```powershell
    Update-AzureRmVM -ResourceGroupName $VMResourceGroup -VM $VM -Tag $Tag | Write-Verbose
    ```
7. Cliquez sur le bouton **Publier**.

## <a name="create-an-optional-microsoft-teams-webhook"></a>Créer un Webhook Microsoft Teams en option

1. Dans Microsoft Teams, sélectionnez **Plus d’options** à côté du nom du canal, puis sélectionnez **Connecteurs**.

    ![Connexions Microsoft Teams](media/ensure-tags-exists-on-new-virtual-machines/teams-webhook.png)

2. Faites défiler la liste des connecteurs jusqu’à **Webhook entrants** et cliquez sur **Ajouter**.

3. Saisissez **AzureAutomationIntegration** pour le nom et sélectionnez **Créer**.

4. Copiez l'URL du webhook dans le presse-papiers et enregistrez-la. L’URL du Webhook permet d’envoyer des informations à Microsoft Teams.

5. Sélectionnez **Terminé** pour enregistrer le Webhook.

## <a name="create-a-webhook-for-the-runbook"></a>Créer un Webhook pour le runbook

1. Ouvrez le runbook Watch-VMWrite.

2. Sélectionnez **Webhooks**puis cliquez sur le bouton **Ajouter Webhook**.

3. Saisissez **WatchVMEventGrid** pour le nom. Copiez l’URL dans le presse-papiers et enregistrez-le.

    ![Configurer le nom du Webhook](media/ensure-tags-exists-on-new-virtual-machines/copy-url.png)

4. Sélectionnez **Configurer les paramètres et les paramètres d’exécution** puis saisissez l’URL du Webhook Microsoft Teams pour **CHANNELURL**. Laissez **WEBHOOKDATA** vide.

    ![Configurer les paramètres du Webhook](media/ensure-tags-exists-on-new-virtual-machines/configure-webhook-parameters.png)

5. Sélectionnez **Créer** pour créer le Webhook du runbook Automation.

## <a name="create-an-event-grid-subscription"></a>Créer un abonnement Event Grid

1. Dans la page d’aperçu **Compte Automation**, sélectionnez **Event grid**.

    ![Sélectionner Event Grid](media/ensure-tags-exists-on-new-virtual-machines/select-event-grid.png)

2. Cliquez sur **+ Abonnement à un événement**.

3. Configurez l’abonnement avec les informations suivantes :
    1. Pour **Type de rubrique**, sélectionnez **Abonnements Azure**.
    2. Décochez la case **S’abonner à tous les types d’événements**.
    3. Saisissez **AzureAutomation** pour le nom.
    4. Dans la liste déroulante **Defined Event Types** (Types d’événements définis), décochez toutes les options sauf **Resource Write Success** (Écriture de ressource réussie).

        > [!NOTE] 
        > Actuellement, Azure Resource Manager ne différencie pas Créer et Mettre à jour et dès lors, l’implémentation de ce tutoriel pour tous les événements Microsoft.Resources.ResourceWriteSuccess dans votre abonnement Azure peut se traduire par un volume d’appels élevé.
    1. Pour **Type de point de terminaison**, sélectionnez **Webhook**.
    2. Cliquez sur **Sélectionner un point de terminaison**. Sur la page **Select Web Hook** (Sélectionner un webhook) qui s’ouvre, collez l’URL de webhook que vous avez créée pour le runbook Watch-VMWrite.
    3. Sous **FILTRES**, saisissez le groupe d’abonnements et de ressources à l’endroit où vous voulez rechercher les nouvelles machines virtuelles créées. Cela peut ressembler à : `/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachines`

4. Sélectionnez **Créer** pour enregistrer l’abonnement Event Grid.

## <a name="create-a-vm-that-triggers-the-runbook"></a>Créer une machine virtuelle qui déclenche le runbook

1. Créez la nouvelle machine virtuelle dans le groupe de ressources spécifié dans le filtre du préfixe de l’abonnement Event Grid.

2. Le runbook Watch-VMWrite doit être appelé et une nouvelle balise ajoutée à la machine virtuelle.

    ![Balise machine virtuelle](media/ensure-tags-exists-on-new-virtual-machines/vm-tag.png)

3. Un nouveau message est envoyé au canal Microsoft Teams.

    ![Notification Microsoft Teams](media/ensure-tags-exists-on-new-virtual-machines/teams-vm-message.png)

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous permet de configurer l’intégration entre Event Grid et Automation. Vous avez appris à :

> [!div class="checklist"]
> * Importer un exemple de runbook Event Grid.
> * Créer un Webhook Microsoft Teams en option.
> * Créer un Webhook pour le runbook.
> * Créer un abonnement Event Grid.
> * Créer une machine virtuelle qui déclenche le runbook.

> [!div class="nextstepaction"]
> [Créer et acheminer des événements personnalisés avec Event Grid](../event-grid/custom-event-quickstart.md)
