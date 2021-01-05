---
title: Présentation de Start/Stop VMs during off-hours d’Azure Automation
description: Cet article décrit la fonctionnalité Start/Stop VMs during off-hours, qui démarre ou arrête des machines virtuelles selon une planification et les surveille de manière proactive à partir des journaux d’activité d’Azure Monitor.
services: automation
ms.subservice: process-automation
ms.date: 09/22/2020
ms.topic: conceptual
ms.openlocfilehash: 3210aa5ae2ff94ba2c7dda673fbb60847c4dfd0b
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92372155"
---
# <a name="startstop-vms-during-off-hours-overview"></a>Vue d’ensemble de Start/Stop VMs during off-hours

La fonctionnalité Start/Stop VMs during off-hours démarre ou arrête les machines virtuelles Azure activées. Elle démarre ou arrête les machines selon une planification définie par l’utilisateur. En outre, elle fournit des informations via Azure les Azure journaux Azure Monitor et peut envoyer des e-mails à l’aide de [groupes d’actions](../azure-monitor/platform/action-groups.md). La fonctionnalité peut être activée sur Azure Resource Manager et les machines virtuelles classiques dans la plupart des scénarios. 

Cette fonctionnalité utilise l’applet de commande [Start-AzVm](/powershell/module/az.compute/start-azvm) pour démarrer des machines virtuelles. Elle utilise [Stop-AzVM](/powershell/module/az.compute/stop-azvm) pour l’arrêt des machines virtuelles.

> [!NOTE]
> Les runbooks ont été mis à jour pour utiliser les nouvelles applets de commande du module Azure Az et utilisent l’alias de préfixe AzureRM.

> [!NOTE]
> Start/Stop VMs during off-hours a été mise à jour pour prendre en charge les versions les plus récentes des modules Azure disponibles. La version mise à jour de cette fonctionnalité, disponible sur Place de marché, ne prend pas en charge les modules AzureRM, car nous avons migré des modules AzureRM vers des modules Az.

La fonctionnalité offre une option décentralisée pour les utilisateurs souhaitant réduire les coûts de leurs machines virtuelles. Vous pouvez utiliser la fonctionnalité pour :

- [Planifier le démarrage et l’arrêt de machines virtuelles](automation-solution-vm-management-config.md#schedule).
- Planifier le démarrage et l’arrêt de machines virtuelles dans un ordre croissant [à l’aide de balises Azure](automation-solution-vm-management-config.md#tags). Cette activité n’est pas prise en charge pour les machines virtuelles classiques.
- Arrêt automatique de machines virtuelles en fonction de la [faible utilisation de l’UC](automation-solution-vm-management-config.md#cpuutil).

Les limitations de la fonctionnalité actuelle sont les suivantes :

- Elle permet de gérer les machines virtuelles de toutes les régions, mais seulement dans le même abonnement que le compte Azure Automation.
- Elle est disponible dans Azure et Azure Government pour toutes les régions qui prennent en charge un espace de travail Log Analytics, un compte Azure Automation et les alertes. À l’heure actuelle, les régions Azure Government ne gèrent pas les fonctionnalités de messagerie électronique.

## <a name="prerequisites"></a>Prérequis

- Les runbooks de la fonctionnalité Start/Stop VMs during off-hours fonctionnent avec un [compte d’identification Azure](./manage-runas-account.md). Le compte d’identification est la méthode d’authentification recommandée, car elle utilise l’authentification par certificat au lieu d’un mot de passe, susceptible d’expirer ou de changer fréquemment.

- Le compte Automation et l’espace de travail Log Analytics liés doivent se trouver dans le même groupe de ressources.

- Il vous est recommandé d’utiliser un compte Automation distinct pour travailler avec la fonctionnalité Start/Stop VMs during off-hours. Les versions de module Azure sont fréquemment mises à niveau et leurs paramètres peuvent changer. La fonctionnalité n’est pas mise à niveau à la même cadence et il est possible qu’elle ne fonctionne pas avec des versions plus récentes des cmdlets qu’elle utilise. Il vous est recommandé de tester les mises à jour de module dans un compte Automation de test avant de les importer dans vos comptes Automation de production.

## <a name="permissions"></a>Autorisations

Vous devez avoir certaines autorisations afin d’activer des machines virtuelles pour la fonctionnalité Start/Stop VMs during off-hours. Les autorisations sont différentes selon que la fonctionnalité utilise un compte Automation et un espace de travail Log Analytics créés au préalable ou qu’elle les crée.

Vous n’avez pas besoin de configurer des autorisations si vous êtes un contributeur pour l’abonnement et un administrateur général dans votre locataire Azure Active Directory (AD). Si vous n’avez pas ces droits ou si vous devez configurer un rôle personnalisé, assurez-vous que vous disposez des autorisations décrites ci-dessous.

### <a name="permissions-for-pre-existing-automation-account-and-log-analytics-workspace"></a>Autorisations pour un compte Automation et un espace de travail Log Analytics préexistants

Afin d’activer des machines virtuelles pour la fonctionnalité Start/Stop VMs during off-hours à l’aide d’un compte Automation et d’un espace de travail Log Analytics existants, vous devez disposer des autorisations suivantes sur l’étendue du groupe de ressources. Pour en savoir plus sur les rôles, consultez [Rôles personnalisés Azure](../role-based-access-control/custom-roles.md).

| Autorisation | Étendue|
| --- | --- |
| Microsoft.Automation/automationAccounts/read | Groupe de ressources |
| Microsoft.Automation/automationAccounts/variables/write | Groupe de ressources |
| Microsoft.Automation/automationAccounts/schedules/write | Groupe de ressources |
| Microsoft.Automation/automationAccounts/runbooks/write | Groupe de ressources |
| Microsoft.Automation/automationAccounts/connections/write | Groupe de ressources |
| Microsoft.Automation/automationAccounts/certificates/write | Groupe de ressources |
| Microsoft.Automation/automationAccounts/modules/write | Groupe de ressources |
| Microsoft.Automation/automationAccounts/modules/read | Groupe de ressources |
| Microsoft.automation/automationAccounts/jobSchedules/write | Groupe de ressources |
| Microsoft.Automation/automationAccounts/jobs/write | Groupe de ressources |
| Microsoft.Automation/automationAccounts/jobs/read | Groupe de ressources |
| Microsoft.OperationsManagement/solutions/write | Groupe de ressources |
| Microsoft.OperationalInsights/workspaces/* | Groupe de ressources |
| Microsoft.Insights/diagnosticSettings/write | Groupe de ressources |
| Microsoft.Insights/ActionGroups/Write | Groupe de ressources |
| Microsoft.Insights/ActionGroups/read | Groupe de ressources |
| Microsoft.Resources/subscriptions/resourceGroups/read | Groupe de ressources |
| Microsoft.Resources/deployments/* | Groupe de ressources |

### <a name="permissions-for-new-automation-account-and-new-log-analytics-workspace"></a>Autorisations pour un nouveau compte Automation et un nouvel espace de travail Log Analytics

Vous pouvez activer des machines virtuelles pour la fonctionnalité Start/Stop VMs during off-hours à l’aide d’un compte Automation et d’un espace de travail Log Analytics nouvellement créés. Dans ce cas, vous avez non seulement besoin des autorisations définies dans la section précédente, mais encore de celles définies dans cette section. Vous avez également besoin des rôles suivants :

- Coadministrateur pour l’abonnement. Ce rôle est requis pour créer le compte d’identification Classic si vous comptez gérer des machines virtuelles classiques. Les [comptes d’identification Classic](automation-create-standalone-account.md#create-a-classic-run-as-account) ne sont plus créés par défaut.
- Appartenance au rôle Développeur d’applications [Azure AD](../active-directory/roles/permissions-reference.md). Pour plus d’informations sur la configuration de comptes d’identification, voir [Autorisations pour configurer des comptes d’identification](manage-runas-account.md#permissions).
- Contributeur sur l’abonnement ou les autorisations suivantes.

| Autorisation |Étendue|
| --- | --- |
| Microsoft.Authorization/Operations/read | Abonnement|
| Microsoft.Authorization/permissions/read |Abonnement|
| Microsoft.Authorization/roleAssignments/read | Abonnement |
| Microsoft.Authorization/roleAssignments/write | Abonnement |
| Microsoft.Authorization/roleAssignments/delete | Abonnement || Microsoft.Automation/automationAccounts/connections/read | Groupe de ressources |
| Microsoft.Automation/automationAccounts/certificates/read | Groupe de ressources |
| Microsoft.Automation/automationAccounts/write | Groupe de ressources |
| Microsoft.OperationalInsights/workspaces/write | Groupe de ressources |

## <a name="components"></a>Components

La fonctionnalité Start/Stop VMs during off-hours inclut des runbooks, des planifications et une intégration préconfigurés avec les journaux Azure Monitor. Vous pouvez utiliser ces éléments pour adapter le démarrage et l’arrêt de vos machines virtuelles aux besoins de votre entreprise.

### <a name="runbooks"></a>Runbooks

Le tableau suivant répertorie les runbooks que la fonctionnalité déploie sur votre compte Automation. Ne modifiez PAS le code de runbook. À la place, écrivez votre propre runbook pour une nouvelle fonctionnalité.

> [!IMPORTANT]
> N’exécutez pas directement de runbook dont le nom contient le suffixe **child** (enfant).

Tous les runbooks parents incluent le paramètre `WhatIf`. Une fois configuré sur True, le paramètre prend en charge la description du comportement exact du runbook durant l’exécution sans le paramètre et valide les machines virtuelles correctes ciblées. Un runbook effectue uniquement ses actions définies quand le paramètre `WhatIf` est défini sur False.

|Runbook | Paramètres | Description|
| --- | --- | ---|
|AutoStop_CreateAlert_Child | VMObject <br> AlertAction <br> WebHookURI | Appelé par le runbook parent. Ce runbook crée des alertes en fonction des ressources pour le scénario d’arrêt automatique.|
|AutoStop_CreateAlert_Parent | VMList<br> WhatIf : True ou False  | Crée ou met à jour des règles d’alerte Azure sur des machines virtuelles dans l’abonnement ou les groupes de ressource ciblées. <br> `VMList` est une liste de machines virtuelles séparées par des virgules (sans espaces), par exemple `vm1,vm2,vm3`.<br> `WhatIf` active la validation de la logique du runbook sans l’exécuter.|
|AutoStop_Disable | None | Désactive les alertes et la planification par défaut de l’arrêt automatique.|
|AutoStop_VM_Child | WebHookData | Appelé par le runbook parent. Les règles d’alerte appellent ce runbook pour arrêter une machine virtuelle classique.|
|AutoStop_VM_Child_ARM | WebHookData |Appelé par le runbook parent. Les règles d’alerte appellent ce runbook pour arrêter une machine virtuelle.  |
|ScheduledStartStop_Base_Classic | CloudServiceName<br> Action : Démarrer ou arrêter<br> VMList  | Effectue une action de démarrage ou d’arrêt dans le groupe de machines virtuelles classiques par le biais de Cloud Services. |
|ScheduledStartStop_Child | VMName <br> Action : Démarrer ou arrêter <br> ResourceGroupName | Appelé par le runbook parent. Exécute une action de démarrage ou d’arrêt pour l’arrêt planifié.|
|ScheduledStartStop_Child_Classic | VMName<br> Action : Démarrer ou arrêter<br> ResourceGroupName | Appelé par le runbook parent. Exécute une action de démarrage ou d’arrêt pour l’arrêt planifié des machines virtuelles classiques. |
|ScheduledStartStop_Parent | Action : Démarrer ou arrêter <br>VMList <br> WhatIf : True ou False | Fait démarrer ou arrête toutes les machines virtuelles de l’abonnement. Modifiez les variables `External_Start_ResourceGroupNames` et `External_Stop_ResourceGroupNames` pour une exécution uniquement sur ces groupes de ressources ciblés. Vous pouvez également exclure des machines virtuelles spécifiques en mettant à jour la variable `External_ExcludeVMNames`.|
|SequencedStartStop_Parent | Action : Démarrer ou arrêter <br> WhatIf : True ou False<br>VMList| Crée des balises nommées **sequencestart** et **sequencestop** sur chaque machine virtuelle pour laquelle vous souhaitez séquencer l’activité de démarrage et d’arrêt. Ces noms de balises respectent la casse. La valeur de la balise doit être une liste d’entiers positifs, par exemple `1,2,3`, correspondant à l’ordre du démarrage ou d’arrêt. <br>**Remarque**  : Les machines virtuelles doivent se trouver dans des groupes de ressources définis dans les variables `External_Start_ResourceGroupNames`, `External_Stop_ResourceGroupNames` et `External_ExcludeVMNames`. Elles doivent disposer des étiquettes appropriées pour que les actions puissent prendre effet.|

### <a name="variables"></a>Variables

Le tableau suivant répertorie les variables créées dans votre compte Automation. Modifiez uniquement les variables ayant le préfixe `External`. La modification de variables avec le préfixe `Internal` a des effets indésirables.

> [!NOTE]
> Les limitations relatives au nom de la machine virtuelle et au groupe de ressources proviennent en grande partie d’une taille définie dans la variable. Consultez la page [Ressources de variables dans Azure Automation](./shared-resources/variables.md).

|Variable | Description|
|---------|------------|
|External_AutoStop_Condition | Opérateur conditionnel requis pour configurer la condition avant le déclenchement d’une alerte. Les valeurs possibles sont `GreaterThan`, `GreaterThanOrEqual`, `LessThan` et `LessThanOrEqual`.|
|External_AutoStop_Description | Alerte pour arrêter la machine virtuelle si le pourcentage d’utilisation de l’UC dépasse le seuil.|
|External_AutoStop_Frequency | Fréquence d’évaluation de la règle. Ce paramètre accepte une entrée au format d’un intervalle de temps. Les valeurs possibles sont comprises entre 5 minutes et 6 heures. |
|External_AutoStop_MetricName | Nom de la métrique de performances pour laquelle la règle d’alerte Azure doit être configurée.|
|External_AutoStop_Severity | Gravité de l’alerte métrique, qui peut être comprise entre 0 et 4. |
|External_AutoStop_Threshold | Seuil de la règle d’alerte Azure spécifiée dans la variable `External_AutoStop_MetricName`. Les valeurs de pourcentage vont de 1 à 100.|
|External_AutoStop_TimeAggregationOperator | Opérateur d’agrégation de temps appliqué à la taille de la fenêtre sélectionnée pour évaluer la condition. Les valeurs possibles sont `Average`, `Minimum`, `Maximum`, `Total` et `Last`.|
|External_AutoStop_TimeWindow | Taille de la fenêtre durant laquelle Azure analyse la métrique sélectionnée pour déclencher une alerte. Ce paramètre accepte une entrée au format d’un intervalle de temps. Les valeurs possibles sont comprises entre 5 minutes et 6 heures.|
|External_EnableClassicVMs| Valeur spécifiant si les machines virtuelles classiques sont ciblées par la fonctionnalité. La valeur par défaut est True. Définissez cette variable sur False pour les abonnements de fournisseurs de solutions Azure Cloud (CSP). Un [compte d’identification Classic](automation-create-standalone-account.md#create-a-classic-run-as-account) est nécessaire pour les machines virtuelles Classic.|
|External_ExcludeVMNames | Liste séparée par des virgules des noms de machines virtuelles à exclure, limitée à 140 machines virtuelles. Si vous ajoutez plus de 140 machines virtuelles à la liste, les machines virtuelles spécifiées pour l’exclusion pourraient être démarrées ou arrêtées par inadvertance.|
|External_Start_ResourceGroupNames | Liste séparée par des virgules d’un ou de plusieurs groupes de ressources ciblés pour des actions de démarrage.|
|External_Stop_ResourceGroupNames | Liste séparée par des virgules d’un ou de plusieurs groupes de ressources ciblés pour des actions d’arrêt.|
|External_WaitTimeForVMRetrySeconds |Délai d’attente, en secondes, des actions à effectuer sur les machines virtuelles pour le runbook **SequencedStartStop_Parent** . Cette variable permet au runbook d’attendre des opérations enfants pendant un nombre de secondes spécifié avant de passer à l’action suivante. Le délai d’attente maximal est 10800, soit trois heures. La valeur par défaut est 2100 secondes.|
|Internal_AutomationAccountName | Spécifie le nom du compte Automation.|
|Internal_AutoSnooze_ARM_WebhookURI | URI du Webhook appelé pour le scénario AutoStop pour les machines virtuelles.|
|Internal_AutoSnooze_WebhookUri | URI du Webhook appelé pour le scénario AutoStop pour les machines virtuelles classiques.|
|Internal_AzureSubscriptionId | L’ID d’abonnement Azure.|
|Internal_ResourceGroupName | Nom du groupe de ressources du compte Automation.|

>[!NOTE]
>Pour la variable `External_WaitTimeForVMRetryInSeconds`, la valeur par défaut a été mise à jour de 600 à 2100. 

Dans tous les scénarios, les variables `External_Start_ResourceGroupNames`, `External_Stop_ResourceGroupNames`et `External_ExcludeVMNames` sont nécessaires au ciblage des machines virtuelles, à l’exception des listes de machines virtuelles séparées par des virgules pour les runbooks **AutoStop_CreateAlert_Parent** , **SequencedStartStop_Parent** et **ScheduledStartStop_Parent** . Autrement dit, vos machines virtuelles doivent faire partie de groupes de ressources cibles pour que les actions de démarrage et d’arrêt se produisent. La logique fonctionne de manière similaire à Azure Policy, dans la mesure où vous pouvez cibler l’abonnement ou un groupe de ressources et faire en sorte que les machines virtuelles nouvellement créées héritent des actions. Cette approche évite de devoir mettre à jour une planification distincte pour chaque machine virtuelle et de gérer le démarrage et l’arrêt à l’échelle.

### <a name="schedules"></a>Planifications

Le tableau suivant répertorie chacune des planifications par défaut créées dans votre compte Automation.  Vous pouvez les modifier ou créer vos propres planifications personnalisées.  Par défaut, toutes les planifications sont désactivées à l’exception des planifications **Scheduled_StartVM** et **Scheduled_StopVM** .

N’activez pas toutes les planifications, car sinon des actions de planification risquent de se chevaucher. Il est préférable de déterminer les optimisations que vous souhaitez y apporter et de les modifier en conséquence. Consultez les exemples de scénarios dans la section Vue d’ensemble pour obtenir davantage d’explications.

|Nom de la planification | Fréquence | Description|
|--- | --- | ---|
|Schedule_AutoStop_CreateAlert_Parent | Toutes les 8 heures | Exécute le runbook **AutoStop_CreateAlert_Parent** toutes les 8 heures, qui à son tour arrête les valeurs basées sur les machines virtuelles dans les variables `External_Start_ResourceGroupNames`, `External_Stop_ResourceGroupNames`et `External_ExcludeVMNames`. Vous pouvez également spécifier une liste de machines virtuelles séparées par des virgules en utilisant le paramètre `VMList`.|
|Scheduled_StopVM | Défini quotidiennement par l'utilisateur | Exécute le runbook **ScheduledStopStart_Parent** avec un paramètre de `Stop` tous les jours à l’heure spécifiée.  Arrête automatiquement toutes les machines virtuelles répondant aux règles définies par les ressources de variables.  Activez la planification associée, **Scheduled-StartVM** .|
|Scheduled_StartVM | Défini quotidiennement par l'utilisateur | Exécute le runbook **ScheduledStopStart_Parent** avec une valeur de paramètre de `Start` tous les jours à l’heure spécifiée. Démarre automatiquement toutes les machines virtuelles répondant aux règles définies par les ressources de variables.  Activez la planification associée, **Scheduled-StopVM** .|
|Sequenced-StopVM | 01:00 (UTC), tous les vendredis | Exécute le runbook **Sequenced_StopStop_Parent** avec une valeur de paramètre de `Stop` tous les vendredis à l’heure spécifiée.  Arrête séquentiellement (dans l’ordre croissant) toutes les machines virtuelles avec la balise **SequenceStop** définie par les variables appropriées. Pour plus d’informations sur les valeurs de balises et les variables de ressources, consultez [Runbooks](#runbooks).  Activez la planification associée, **Sequenced-StartVM** .|
|Sequenced-StartVM | 13:00 (UTC), tous les lundis | Exécute le runbook **SequencedStopStart_Parent** avec une valeur de paramètre de `Start` tous les lundis à l’heure spécifiée. Démarre séquentiellement (dans l’ordre décroissant) toutes les machines virtuelles avec la balise **SequenceStart** définie par les variables appropriées. Pour plus d’informations sur les valeurs de balises et les ressources de variables, consultez [Runbooks](#runbooks). Activez la planification associée, **Sequenced-StopVM** .

## <a name="use-the-feature-with-classic-vms"></a>Utiliser la fonctionnalité avec des machines virtuelles classiques

Si vous utilisez la fonctionnalité Start/Stop VMs during off-hours pour des machines virtuelles classiques, Automation traite toutes vos machines virtuelles de manière séquentielle par service cloud. Les machines virtuelles sont toujours traitées en parallèle à travers les différents services cloud. 

Pour utiliser la fonctionnalité avec des machines virtuelles classiques, vous avez besoin d’un compte d’identification Classic, lequel n’est pas créé par défaut. Pour savoir comment créer un compte d’identification Classic, consultez [Créer un compte d’identification Classic](automation-create-standalone-account.md#create-a-classic-run-as-account).

Si vous avez plus de 20 machines virtuelles par service cloud, voici quelques recommandations :

* Créez plusieurs planifications avec le runbook parent **ScheduledStartStop_Parent** et en spécifiant 20 machines virtuelles par planification. 
* Dans les propriétés de planification, utilisez le paramètre `VMList` pour spécifier des noms de machines virtuelles sous forme de liste séparée par des virgules (sans espaces). 

Sinon, si le travail Automation pour cette fonctionnalité s’exécute pendant plus de trois heures, il est momentanément déchargé ou arrêté par la limite de [répartition de charge équilibrée](automation-runbook-execution.md#fair-share).

Les abonnements Azure CSP prennent uniquement en charge le modèle Azure Resource Manager. Les services non-Azure Resource Manager ne sont pas disponibles dans le programme. Quand la fonctionnalité Start/Stop VMs during off-hours s’exécute, vous pouvez recevoir des messages d’erreur, car elle contient des cmdlets pour gérer des ressources classiques. Pour en savoir plus sur CSP, consultez [Services disponibles dans les abonnements CSP](/azure/cloud-solution-provider/overview/azure-csp-available-services). Si vous utilisez un abonnement CSP, vous devez définir la variable [External_EnableClassicVMs](#variables) sur False après le déploiement.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="enable-the-feature"></a>Activer la fonctionnalité

Pour commencer à utiliser la fonctionnalité, suivez les étapes décrites dans [Activer Start/Stop VMs during off-hours](automation-solution-vm-management-enable.md).

## <a name="view-the-feature"></a>Afficher la fonctionnalité

Utilisez l’un des mécanismes suivants pour accéder à la fonctionnalité activée :

* Dans votre compte Automation, sélectionnez **Start/Stop VM** sous **Ressources associées** . Sur la page Start/Stop VM, sélectionnez **Gérer la solution** sous **Gérer les solutions Start/Stop VM** .

* Accédez à l’espace de travail Log Analytics lié à votre compte Automation. Après avoir sélectionné l’espace de travail, choisissez **Solutions** dans le volet gauche. Sur la page Solutions, sélectionnez la solution **Start-Stop-VM[espace de travail]** dans la liste.  

La sélection de la fonctionnalité affiche la page Start-Stop-VM[espace de travail]. Vous pouvez y consulter des informations importantes, telles que celles contenues dans la vignette **StartStopVM** . Tout comme dans votre espace de travail Log Analytics, cette vignette affiche un compteur et une représentation graphique des tâches de runbooks de la fonctionnalité qui ont été démarrées et terminées avec succès.

![Page Update Management Automation](media/automation-solution-vm-management/azure-portal-vmupdate-solution-01.png)

Vous pouvez analyser plus en détail les enregistrements de tâche en cliquant sur la vignette en forme d’anneau. Le tableau de bord affiche l’historique des travaux et les requêtes Recherche dans les journaux prédéfinies. Passez au portail avancé Log Analytics pour effectuer des recherches en fonction de vos requêtes de recherche.

## <a name="update-the-feature"></a>Mettre à jour la fonctionnalité

Si vous avez déployé une version précédente de Start/Stop VMs during off-hours, supprimez-la de votre compte avant de déployer une version mise à jour. Suivez les étapes de [suppression de la fonctionnalité](#remove-the-feature), puis celles visant à [l’activer](automation-solution-vm-management-enable.md).

## <a name="remove-the-feature"></a>Supprimer la fonctionnalité

Si vous n’avez plus besoin d’utiliser la fonctionnalité, vous pouvez la supprimer à partir du compte Automation. La suppression de la fonctionnalité supprime uniquement les runbooks associés. Elle ne supprime pas les planifications ni les variables créées quand la fonctionnalité a été ajoutée. 

Pour supprimer Start/Stop VMs during off-hours :

1. Dans votre compte Automation, sélectionnez **Espace de travail lié** sous **Ressources connexes** .

2. Sélectionnez **Accéder à l’espace de travail** .

3. Cliquez sur **Solutions** sous **Général** . 

4. Sur la page Solutions, sélectionnez **Start-Stop-VM[espace de travail]** . 

5. Sur la page VMManagementSolution[espace de travail], sélectionnez l’option **Supprimer** dans le menu.<br><br> ![Delete VM management feature](media/automation-solution-vm-management/vm-management-solution-delete.png)

6. Dans la fenêtre Supprimer la solution, confirmez que vous souhaitez supprimer la fonctionnalité.

7. Pendant que les informations sont vérifiées et que la fonctionnalité est supprimée, vous pouvez suivre la progression sous **Notifications** à partir du menu. Vous êtes redirigé vers la page Solutions une fois le processus de suppression terminé.

8. Le compte Automation et l’espace de travail Log Analytics ne sont pas supprimés au cours de ce processus. Si vous ne souhaitez pas conserver l’espace de travail Log Analytics, vous devez le supprimer manuellement du Portail Azure :

    1. Recherchez et sélectionnez **Espaces de travail Log Analytics** .

    2. Sur la page Espaces de travail Log Analytics, sélectionnez l’espace de travail.

    3. Sélectionnez **Supprimer** dans le menu.

    4. Si vous ne souhaitez pas conserver les [composants relatifs à la fonctionnalité](#components) du compte Azure Automation, vous pouvez les supprimer manuellement.

## <a name="next-steps"></a>Étapes suivantes

Pour activer la fonctionnalité sur des machines virtuelles de votre environnement, consultez [Activer Start/Stop VMs during off-hours](automation-solution-vm-management-enable.md).
