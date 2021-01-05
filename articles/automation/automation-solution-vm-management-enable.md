---
title: Activer Start/Stop VMs during off-hours d’Azure Automation
description: Cet article explique comment activer la fonctionnalité Start/Stop VMs during off-hours pour vos machines virtuelles Azure.
services: automation
ms.subservice: process-automation
ms.date: 04/01/2020
ms.topic: conceptual
ms.openlocfilehash: dde2c3e4cf496bb15ca91c72d9a41936af7051c5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83743752"
---
# <a name="enable-startstop-vms-during-off-hours"></a>Activer Start/Stop VMs during off-hours

Effectuez les étapes de cette rubrique dans l’ordre pour activer la fonctionnalité Start/Stop VMs during off-hours pour les machines virtuelles à l’aide d’un compte Automation nouveau ou existant et d’un espace de travail Log Analytics lié. Au terme du processus de configuration, configurez les variables pour personnaliser la fonctionnalité.

>[!NOTE]
>Pour utiliser cette fonctionnalité avec des machines virtuelles classiques, vous avez besoin d’un compte d’identification Classic, lequel n’est pas créé par défaut. Voir [Créer un compte d’identification Classic](automation-create-standalone-account.md#create-a-classic-run-as-account).
>

## <a name="create-resources-for-the-feature"></a>Créer des ressources pour la fonctionnalité

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Recherchez et sélectionnez **Comptes Automation**.
3. Dans la page Comptes Automation, sélectionnez votre compte Automation dans la liste.
4. À partir du compte Automation, sélectionnez **Start/Stop VM** sous **Ressources associées**. Vous pouvez alors cliquer sur **En savoir plus sur la solution et l’activer**. Si vous avez déjà déployé la fonctionnalité, vous pouvez cliquer sur **Gérer la solution** et la trouver dans la liste.

   ![Activer à partir d’un compte Automation](./media/automation-solution-vm-management/enable-from-automation-account.png)

   > [!NOTE]
   > Vous pouvez également créer la ressource n’importe où sur le portail Azure en cliquant sur **Créer une ressource**. Dans la page Place de marché, saisissez un mot clé, tel que **Démarrer** ou **Arrêter/Démarrer**. Au fur et à mesure de la saisie, la liste est filtrée. Vous pouvez également saisir un ou plusieurs mots clés du nom complet de la fonctionnalité, puis appuyer sur la touche **Entrée**. Sélectionnez **Start/Stop VMs during off-hours** dans les résultats de la recherche.

5. Sur la page Start/Stop VMs during off-hours du déploiement sélectionné, vérifiez les informations résumées, puis cliquez sur **Créer**.

   ![Portail Azure](media/automation-solution-vm-management/azure-portal-01.png)

## <a name="configure-the-feature"></a>Configurer la fonctionnalité

Une fois la ressource créée, la page Ajouter une solution s’affiche. Vous êtes invité à configurer la fonctionnalité pour pouvoir l’importer dans votre abonnement Automation. Voir [Configurer Start/Stop VMs during off-hours](automation-solution-vm-management-config.md).

   ![Page Ajouter une solution de Gestion de machines virtuelles](media/automation-solution-vm-management/azure-portal-add-solution-01.png)

## <a name="select-a-log-analytics-workspace"></a>Sélectionner un espace de travail Log Analytics

1. Sur la page Ajouter une solution, sélectionnez **Espace de travail**. Sélectionnez un espace de travail Log Analytics lié à l’abonnement Azure utilisé par le compte Automation. 

2. Si vous ne disposez pas d’espace de travail, sélectionnez **Créer un espace de travail**. Sur la page Espace de travail Log Analytics, suivez les étapes suivantes :

   - Spécifiez un nom pour le nouvel espace de travail Log Analytics, comme**ContosoLAWorkspace**.
   - Dans la liste déroulante **Abonnement**, sélectionnez un abonnement à lier si la valeur par défaut sélectionnée n’est pas appropriée.
   - Sous **Groupe de ressources**, vous pouvez créer un groupe de ressources ou en sélectionner un qui existe déjà.
   - Sélectionnez un **emplacement**.
   - Sélectionner un **niveau de tarification**. Choisissez l’option **Par Go (autonome)** . Les journaux Azure Monitor ont mis à jour les [tarifs](https://azure.microsoft.com/pricing/details/log-analytics/), et le niveau Par Go est la seule option disponible.

   > [!NOTE]
   > Lors de l’activation de fonctionnalités, seules certaines régions sont prises en charge pour la liaison d’un espace de travail Log Analytics et d’un compte Automation. Pour obtenir la liste des paires de mappages prises en charge, consultez [Mappage de régions pour un compte Automation et un espace de travail Log Analytics](how-to/region-mappings.md).

3. Après avoir entré les informations requises sur la page Espace de travail Log Analytics, cliquez sur **Créer**. Vous pouvez suivre sa progression sous **Notifications** dans le menu, qui vous renvoie à la page Ajouter une solution une fois terminé.

## <a name="add-automation-account"></a>Ajouter un compte Automation

Accédez à nouveau à la page Ajouter une solution et sélectionnez **Compte Automation**. Vous pouvez sélectionner un compte Automation existant qui n’est pas encore lié à un espace de travail Log Analytics. Si vous créez un espace de travail Log Analytics, vous pouvez créer un compte Automation pour l’y associer. Sélectionnez un compte Automation existant ou cliquez sur **Créer un compte Automation**, puis, sur la page Ajouter un compte Automation, indiquez le nom du compte Automation dans le champ **Nom**.

Toutes les autres options sont renseignées automatiquement en fonction de l’espace de travail Log Analytics sélectionné. Vous ne pouvez pas modifier ces options. Un compte d’identification Azure est la méthode d’authentification par défaut pour les runbooks inclus avec la fonctionnalité. 

Après avoir cliqué sur **OK**, les options de configuration sont validées et le compte Automation est créé. Vous pouvez suivre la progression sous **Notifications** dans le menu.

## <a name="define-feature-parameters"></a>Définir les paramètres de la fonctionnalité

1. Sur la page Ajouter une solution, sélectionnez **Configuration**. La page Paramètres s’affiche.

    ![Page Paramètres pour la solution](media/automation-solution-vm-management/azure-portal-add-solution-02.png)

2. Spécifiez une valeur pour le champ **Noms des groupes de ressources cibles**. Le champ définit les noms de groupes qui contiennent des machines virtuelles que la fonctionnalité doit gérer. Vous pouvez entrer plusieurs noms et les séparer par des virgules (les valeurs ne respectent pas la casse). Si vous souhaitez cibler les machines virtuelles de tous les groupes de ressources de l’abonnement, l’utilisation d’un caractère générique est prise en charge. Les valeurs sont stockées dans les variables `External_Start_ResourceGroupNames` et `External_Stop_ResourceGroupNames`.

    > [!IMPORTANT]
    > La valeur par défaut pour les **noms des groupes de ressources cibles** est un **&ast;** . Ce paramètre cible toutes les machines virtuelles d’un abonnement. Si vous ne souhaitez pas que la fonctionnalité cible toutes les machines virtuelles de votre abonnement, vous devez fournir une liste de noms de groupes de ressources avant de sélectionner une planification.
  
3. Spécifiez une valeur pour le champ **Liste d’exclusion de machines virtuelles (chaîne)** . Cette valeur est le nom d’une ou de plusieurs machines virtuelles appartenant au groupe de ressources cible. Vous pouvez entrer plusieurs noms et les séparer par des virgules (les valeurs ne respectent pas la casse). Les caractères génériques sont pris en charge. La valeur est stockée dans la variable `External_ExcludeVMNames`.
  
4. Utilisez le champ **Planifier** pour sélectionner une planification de la gestion des machines virtuelles par la fonctionnalité. Sélectionnez une date et une heure de début pour votre planification, afin de créer une planification quotidienne récurrente qui commence à l’heure choisie. La sélection d’une autre région n’est pas possible. Pour configurer la planification sur votre propre fuseau horaire après la configuration de la fonctionnalité, consultez [Modifier les planifications de démarrage et d’arrêt](automation-solution-vm-management-config.md#modify-the-startup-and-shutdown-schedules).

5. Pour recevoir des notifications par e-mail de la part d’un [groupe d’actions](../azure-monitor/platform/action-groups.md), acceptez la valeur par défaut **Oui** dans le champ **Notifications par e-mail**, puis fournissez une adresse e-mail valide. Si vous sélectionnez **Non**, mais décidez ultérieurement de recevoir les notifications par e-mail, vous pouvez mettre à jour le groupe d’actions créé en y ajoutant des adresses e-mail valides, séparées par des virgules. 

6. Activez les règles d’alerte suivantes :

   - `AutoStop_VM_Child`
   - `Scheduled_StartStop_Parent`
   - `Sequenced_StartStop_Parent`

## <a name="create-alerts"></a>Créez des alertes

Start/Stop VMs during off-hours n’inclut aucun ensemble prédéfini d’alertes. Consultez [Créer des alertes de journal avec Azure Monitor](../azure-monitor/platform/alerts-log.md) pour apprendre à créer des alertes d’échec de tâche afin d’assister vos procédures et processus opérationnels ou DevOps.

## <a name="deploy-the-feature"></a>Déployer la fonctionnalité

1. Après avoir configuré les paramètres initiaux requis pour la fonctionnalité, cliquez sur **OK** pour fermer la page Paramètres.

2. Cliquez sur **Créer**. Quand tous les paramètres sont validés, la fonctionnalité est déployée dans votre abonnement. Ce processus peut prendre plusieurs secondes. Vous pouvez suivre la progression sous **Notifications** dans le menu.

    > [!NOTE]
    > Si vous avez un abonnement Azure Cloud Solution Provider (Azure CSP), une fois le déploiement terminé, dans votre compte Automation, accédez à **Variables** sous **Ressources partagées** et définissez la variable [External_EnableClassicVMs](automation-solution-vm-management.md#variables) sur **False**. La solution arrête de rechercher des ressources de machine virtuelle classiques.

## <a name="next-steps"></a>Étapes suivantes

* Pour configurer la fonctionnalité, consultez [Configurer Start/Stop VMs during off-hours](automation-solution-vm-management-config.md).
* Pour résoudre les erreurs liées à la fonctionnalité, consultez [Résoudre les problèmes liés à Start/Stop VMs during off-hours](troubleshoot/start-stop-vm.md).