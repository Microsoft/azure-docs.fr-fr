---
title: Résolution des problèmes Azure Automation Update Management
description: Cet article explique comment dépanner et résoudre les problèmes liés à Azure Automation Update Management.
services: automation
ms.date: 12/04/2020
ms.topic: conceptual
ms.service: automation
ms.openlocfilehash: c6d0f38eaa25f2fe033a5e2cf48ee6daa51fcbe6
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929274"
---
# <a name="troubleshoot-update-management-issues"></a>Résoudre les problèmes liés à Update Management

Cet article décrit les problèmes que vous pouvez rencontrer lors du déploiement de la fonctionnalité Update Management sur vos ordinateurs. Il existe un utilitaire de résolution des problèmes qui permet à l’agent Runbook Worker hybride de déterminer le problème sous-jacent. Pour en savoir plus sur l’utilitaire de résolution des problèmes, consultez [Résoudre les problèmes de l’agent de mise à jour Windows](update-agent-issues.md) et [Résoudre les problèmes de l’agent de mise à jour Linux](update-agent-issues-linux.md). Pour d’autres problèmes de déploiement de fonctionnalités, voir [Résoudre les problèmes de déploiement de fonctionnalités](onboarding.md).

>[!NOTE]
>Si vous rencontrez des problèmes lors du déploiement d’Update Management sur un ordinateur Windows, ouvrez l’observateur d’événements Windows et examinez le journal **Operations Manager** sous **Journaux des applications et des services** sur l’ordinateur local. Recherchez les événements présentant l’ID d’événement 4502 et les détails d’événement qui contiennent `Microsoft.EnterpriseManagement.HealthService.AzureAutomation.HybridAgent`.

## <a name="scenario-linux-updates-shown-as-pending-and-those-installed-vary"></a><a name="updates-linux-installed-different"></a>Scénario : Les mises à jour Linux indiquées comme étant en attente et celles qui sont installées varient

### <a name="issue"></a>Problème

Pour votre machine Linux, Update Management affiche des mises à jour spécifiques disponibles sous les classification **Sécurité** et **Autres**. Toutefois, lorsqu’une planification de mise à jour est exécutée sur l’ordinateur, par exemple pour installer uniquement les mises à jour correspondant à la classification **Sécurité**, les mises à jour installées sont différentes ou constituent un sous-ensemble des mises à jour indiquées précédemment correspondant à cette classification.

### <a name="cause"></a>Cause

Lorsqu’une évaluation des mises à jour du système d’exploitation en attente pour votre ordinateur Linux est terminée, les fichiers [OVAL](https://oval.mitre.org/) (Open Vulnerability and Assessment Language) fournis par le fournisseur de distribution Linux sont utilisés par Update Management pour la classification. La catégorisation est effectuée pour les mises à jour Linux en tant que **Sécurité** ou **Autres**, en fonction des fichiers OVAL qui signalent les mises à jour traitant des problèmes de sécurité ou des vulnérabilités. Toutefois, lorsque la planification des mises à jour est exécutée, elle s’exécute sur l’ordinateur Linux à l’aide du gestionnaire de package approprié pour les installer (p. ex., YUM, APT ou ZYPPER). Le gestionnaire de package pour la distribution Linux peut avoir un mécanisme différent pour classifier les mises à jour, où les résultats peuvent différer de ceux obtenus à partir des fichiers OVAL par Update Management.

### <a name="resolution"></a>Résolution

Vous pouvez vérifier manuellement l’ordinateur Linux, les mises à jour applicables et leur classification selon le gestionnaire de package de la distribution. Pour comprendre quelles mises à jour sont classées sous **Sécurité** par votre gestionnaire de package, exécutez les commandes suivantes.

Pour YUM, la commande suivante renvoie une liste non nulle des mises à jour classées sous **Sécurité** par Red Hat. Notez que, dans le cas de CentOS, elle renvoie toujours une liste vide et aucune classification de sécurité ne se produit.

```bash
sudo yum -q --security check-update
```

Pour ZYPPER, la commande suivante renvoie une liste non nulle des mises à jour classées sous **Sécurité** par SUSE.

```bash
sudo LANG=en_US.UTF8 zypper --non-interactive patch --category security --dry-run
```

Pour APT, la commande suivante renvoie une liste non nulle des mises à jour classées sous **Sécurité** par les distributions Canonical pour Ubuntu Linux.

```bash
sudo grep security /etc/apt/sources.list > /tmp/oms-update-security.list LANG=en_US.UTF8 sudo apt-get -s dist-upgrade -oDir::Etc::Sourcelist=/tmp/oms-update-security.list
```

À partir de cette liste, vous exécutez ensuite la commande `grep ^Inst` pour récupérer tous les correctifs de sécurité en attente.

## <a name="scenario-you-receive-the-error-failed-to-enable-the-update-solution"></a><a name="failed-to-enable-error"></a>Scénario : Vous recevez l’erreur « Échec de l’activation de la solution Update »

### <a name="issue"></a>Problème

Quand vous tentez d’activer Update Management dans votre compte Automation, vous obtenez l’erreur suivante :

```error
Error details: Failed to enable the Update solution
```

### <a name="cause"></a>Cause

Cette erreur peut se produire pour les raisons suivantes :

* Le pare-feu réseau ne respecte peut-être pas la configuration requise pour l’agent Log Analytics. Cela a pour effet d’empêcher l’agent de résoudre les URL DNS.

* Le ciblage Update Management est mal configuré et l’ordinateur ne reçoit pas les mises à jour comme prévu.

* Vous pouvez également remarquer que l’ordinateur affiche l’état `Non-compliant` sous **Conformité**. Dans le même temps, **Agent Desktop Analytics** affiche l’état `Disconnected` pour l’agent.

### <a name="resolution"></a>Résolution

* Exécutez l’utilitaire de résolution des problèmes pour [Windows](update-agent-issues.md#troubleshoot-offline) ou [Linux](update-agent-issues-linux.md#troubleshoot-offline), selon le système d’exploitation utilisé.

* Accédez à [Configuration réseau](../automation-hybrid-runbook-worker.md#network-planning) pour connaître les adresses et ports à autoriser pour le fonctionnement d’Update Management.  

* Recherchez les problèmes de configuration d’étendue. La [configuration de l’étendue](../update-management/scope-configuration.md) permet de déterminer les ordinateurs qui sont configurés pour Update Management. Si votre ordinateur figure dans votre espace de travail, mais pas dans Update Management, vous devez définir la configuration de l’étendue de sorte qu’elle cible l’ordinateur. Pour en savoir plus sur la configuration de l’étendue, consultez [Activer des machines dans l’espace de travail](../update-management/enable-from-automation-account.md#enable-machines-in-the-workspace).

* Pour supprimer la configuration du Worker, suivez les étapes décrites dans [Supprimer la fonctionnalité Runbook Worker hybride d’un ordinateur Windows local](../automation-windows-hrw-install.md#remove-windows-hybrid-runbook-worker) ou [Supprimer le Runbook Worker hybride à partir d’un ordinateur Linux local](../automation-linux-hrw-install.md#remove-linux-hybrid-runbook-worker).

## <a name="scenario-superseded-update-indicated-as-missing-in-update-management"></a>Scénario : Mise à jour remplacée indiquée comme manquante dans Update Management

### <a name="issue"></a>Problème

Des mises à jour anciennes apparaissent pour un compte Automation comme étant manquantes, alors qu’elles ont été remplacées. Une mise à jour remplacée n’a pas besoin d’être installée, car une mise à jour plus récente corrigeant la même vulnérabilité est disponible. Update Management ignore la mise à jour remplacée et la rend non applicable au profit de la mise à jour de remplacement. Pour plus d’informations sur un problème connexe, consultez [Mise à jour remplacée](/windows/deployment/update/windows-update-troubleshooting#the-update-is-not-applicable-to-your-computer).

### <a name="cause"></a>Cause

Les mises à jour remplacées ne sont pas refusées dans Windows Server Update Services (WSUS) de façon à être considérées comme non applicables.

### <a name="resolution"></a>Résolution

Lorsqu’une mise à jour remplacée devient 100 % non applicable, vous devez modifier l’état d’approbation de cette mise à jour sur `Declined` dans WSUS. Pour modifier l’état d’approbation de toutes vos mises à jour :

1. Dans le compte Automation, sélectionnez **Update Management** pour afficher l’état de vos machines. Consultez [Voir les évaluations des mises à jour](../update-management/view-update-assessments.md).

2. Vérifiez la mise à jour remplacée pour vous assurer qu’elle est 100 % non applicable.

3. Sur le serveur WSUS sur lequel les ordinateurs sont rapportés, [refusez la mise à jour](/windows-server/administration/windows-server-update-services/manage/updates-operations#declining-updates).

4. Sélectionnez **Ordinateurs** et, dans la colonne **Conformité**, forcez une nouvelle analyse de la conformité. Consultez [Gérer les mises à jour pour les machines virtuelles](../update-management/manage-updates-for-vm.md).

5. Répétez les étapes ci-dessus pour les autres mises à jour remplacées.

6. Pour WSUS (Windows Server Update Services), nettoyez toutes les mises à jour remplacées de façon à actualiser l’infrastructure à l’aide de l’[Assistant de nettoyage du serveur](/windows-server/administration/windows-server-update-services/manage/the-server-cleanup-wizard) WSUS.

7. Répétez cette procédure régulièrement pour corriger le problème d’affichage et limiter l’espace disque utilisé à des fins de gestion des mises à jour.

## <a name="scenario-machines-dont-show-up-in-the-portal-under-update-management"></a><a name="nologs"></a>Scénario : Les machines ne s’affichent pas dans le portail sous Update Management

### <a name="issue"></a>Problème

Vos ordinateurs présentent les symptômes suivants :

* Votre ordinateur affiche l’état `Not configured` dans la vue Update Management d’une machine virtuelle.

* Vos ordinateurs ne s’affichent pas dans la vue Update Management de votre compte Azure Automation.

* Vous avez des ordinateurs qui affichent l’état `Not assessed` sous **Conformité**. Toutefois, les données de pulsation s’affichent dans les journaux d’Azure Monitor pour le Runbook Worker hybride, mais pas pour Update Management.

### <a name="cause"></a>Cause

Cela peut provenir de problèmes de configuration locaux ou d’une configuration d’étendue incorrecte. Les causes possibles sont les suivantes :

* Vous devrez peut-être réinscrire et réinstaller le Runbook Worker hybride.

* Vous avez peut-être défini dans votre espace de travail un quota qui a été atteint et empêche le stockages d’autres données.

### <a name="resolution"></a>Résolution

1. Exécutez l’utilitaire de résolution des problèmes pour [Windows](update-agent-issues.md#troubleshoot-offline) ou [Linux](update-agent-issues-linux.md#troubleshoot-offline), selon le système d’exploitation utilisé.

2. Assurez-vous que votre ordinateur est associé à l’espace de travail approprié. Pour obtenir des conseils sur la façon de vérifier cet aspect, consultez l’article [Vérifier la connectivité de l’agent à Azure Monitor](../../azure-monitor/platform/agent-windows.md#verify-agent-connectivity-to-azure-monitor). Assurez-vous également que cet espace de travail est associé à votre compte Azure Automation. Pour le confirmer, accédez à votre compte Automation et sélectionnez **Espace de travail lié** sous **Ressources associées**.

3. Vérifiez que les ordinateurs apparaissent dans l’espace de travail Log Analytics comme étant liés à votre compte Automation. Exécutez la requête suivante dans l’espace de travail Log Analytics.

   ```kusto
   Heartbeat
   | summarize by Computer, Solutions
   ```

4. Si votre ordinateur ne figure pas dans les résultats de la requête, c’est qu’il n’a pas été enregistré récemment. Il existe probablement un problème de configuration locale et vous devez [réinstaller l’agent](../../azure-monitor/learn/quick-collect-windows-computer.md#install-the-agent-for-windows).

5. Si votre ordinateur figure dans les résultats de la requête, recherchez d’éventuels problèmes de configuration de l’étendue. La [configuration de l’étendue](../update-management/scope-configuration.md) permet de déterminer les ordinateurs qui sont configurés pour Update Management.

6. Si votre ordinateur figure dans votre espace de travail, mais pas dans Update Management, vous devez définir la configuration de l’étendue de sorte qu’elle cible l’ordinateur. Pour savoir comment procéder, consultez [Activer des machines dans l’espace de travail](../update-management/enable-from-automation-account.md#enable-machines-in-the-workspace).

7. Dans votre espace de travail, exécutez cette requête.

   ```kusto
   Operation
   | where OperationCategory == 'Data Collection Status'
   | sort by TimeGenerated desc
   ```

8. Si vous obtenez un résultat `Data collection stopped due to daily limit of free data reached. Ingestion status = OverQuota`, le quota défini dans votre espace de travail a été atteint, ce qui a fait cesser l’enregistrement des données. Dans votre espace de travail, accédez à **Gestion du volume de données** sous **Utilisation et estimation des coûts**, puis changez ou supprimez le quota.

9. Si votre problème n’est toujours pas résolu, suivez les étapes dans [Déployer un Runbook Worker hybride Windows](../automation-windows-hrw-install.md) pour réinstaller le Worker hybride pour Windows. Pour Linux, suivez les étapes décrites dans [Déployer un Runbook Worker hybride Linux](../automation-linux-hrw-install.md).

## <a name="scenario-unable-to-register-automation-resource-provider-for-subscriptions"></a><a name="rp-register"></a>Scénario : Impossible d’inscrire le fournisseur de ressources Automation pour les abonnements

### <a name="issue"></a>Problème

Quand vous utilisez des déploiements de fonctionnalités dans votre compte Automation, l’erreur suivante se produit :

```error
Error details: Unable to register Automation Resource Provider for subscriptions
```

### <a name="cause"></a>Cause

Le fournisseur de ressources Automation n’est pas inscrit dans l’abonnement.

### <a name="resolution"></a>Résolution

Pour inscrire le fournisseur de ressources Automation, suivez ces étapes sur le portail Azure.

1. Dans la liste de services Azure en bas du portail, sélectionnez **Tous les services**, puis sélectionnez **Abonnements** dans le groupe de services Général.

2. Sélectionnez votre abonnement.

3. Sous **Paramètres**, sélectionnez **Fournisseurs de ressources**.

4. Dans la liste des fournisseurs de ressources, vérifiez que le fournisseur de ressources Microsoft.Automation est inscrit.

5. S’il ne l’est pas, inscrivez le fournisseur Microsoft.Automation en suivant les étapes dans [Résoudre les erreurs d’inscription de fournisseurs de ressources](../../azure-resource-manager/templates/error-register-resource-provider.md).

## <a name="scenario-scheduled-update-with-a-dynamic-schedule-missed-some-machines"></a><a name="scheduled-update-missed-machines"></a>Scénario : Certains ordinateurs ont été privés d’une mise à jour planifiée avec une planification dynamique

### <a name="issue"></a>Problème

Les ordinateurs inclus dans la préversion d’une mise à jour ne figurent pas tous dans la liste des ordinateurs corrigés au cours d’une exécution planifiée.

### <a name="cause"></a>Cause

Ce problème peut avoir l’une des causes suivantes :

* Les abonnements définis dans l’étendue d’une requête dynamique ne sont pas configurés pour le fournisseur de ressources Automation inscrit.

* Les ordinateurs n’étaient pas disponibles ou n’avaient pas les étiquettes appropriées au moment où la planification s’est exécutée.

### <a name="resolution"></a>Résolution

#### <a name="subscriptions-not-configured-for-registered-automation-resource-provider"></a>Abonnements non configurés pour le fournisseur de ressources Automation inscrit

Si votre abonnement n’est pas configuré pour le fournisseur de ressources Automation, vous ne pouvez pas demander ou extraire d’informations sur les ordinateurs de cet abonnement. Suivez les étapes ci-dessous pour vérifier l’inscription pour l’abonnement.

1. Sur le [portail Azure](../../azure-resource-manager/management/resource-providers-and-types.md#azure-portal), accédez à la liste des services Azure.

2. Sélectionnez **Tous les services**, puis **Abonnements** dans le groupe de services généraux.

3. Recherchez l’abonnement défini dans l’étendue de votre déploiement.

4. Sous **Paramètres**, choisissez **Fournisseurs de ressources**.

5. Vérifiez que le fournisseur de ressources Microsoft.Automation est inscrit.

6. S’il ne l’est pas, inscrivez le fournisseur Microsoft.Automation en suivant les étapes dans [Résoudre les erreurs d’inscription de fournisseurs de ressources](../../azure-resource-manager/templates/error-register-resource-provider.md).

#### <a name="machines-not-available-or-not-tagged-correctly-when-schedule-executed"></a>Ordinateurs non disponibles ou non étiquetés correctement pendant l’exécution de la planification

Utilisez la procédure suivante si votre abonnement est configuré pour le fournisseur de ressources Automation, mais que certains ordinateurs passent au travers de l’exécution de la planification de mise à jour avec les [groupes dynamiques](../update-management/configure-groups.md) spécifiés.

1. Sur le portail Azure, ouvrez le compte Automation, puis sélectionnez **Gestion des mises à jour**.

2. Examinez l’[historique Update Management](../update-management/deploy-updates.md#view-results-of-a-completed-update-deployment) pour déterminer l’heure d’exécution exacte du déploiement des mises à jour.

3. Pour les ordinateurs qui, selon vous, sont passés au travers d’Update Management, utilisez Azure Resource Graph (ARG) pour [localiser les modifications apportées aux ordinateurs](../../governance/resource-graph/how-to/get-resource-changes.md#find-detected-change-events-and-view-change-details).

4. Recherchez les modifications sur une période significative, par exemple une journée, antérieure au déploiement des mises à jour.

5. Repérez dans les résultats de la recherche d’éventuelles modifications systémiques, comme des suppressions ou des mises à jour, sur les ordinateurs au cours de cette période. Ces modifications peuvent changer l’état des ordinateurs ou les étiquettes, si bien que certains ordinateurs ne sont pas sélectionnés dans la liste des ordinateurs au moment où les mises à jour sont déployées.

6. Ajustez les paramètres des ordinateurs et des ressources si nécessaire pour corriger l’état des ordinateurs ou les problèmes d’étiquettes.

7. Réexécutez la planification de mise à jour pour faire en sorte que le déploiement avec les groupes dynamiques spécifiés englobe tous les ordinateurs.

## <a name="scenario-expected-machines-dont-appear-in-preview-for-dynamic-group"></a><a name="machines-not-in-preview"></a>Scénario : Les machines attendues n’apparaissent pas dans la préversion pour le groupe dynamique

### <a name="issue"></a>Problème

Les machines virtuelles correspondant aux étendues sélectionnées d’un groupe dynamique ne figurent pas dans la liste de préversion du Portail Azure. Cette liste comprend toutes les machines récupérées par une requête ARG pour les étendues sélectionnées. Les étendues sont filtrées pour les machines sur lesquelles des Runbooks Worker hybrides sont installés et pour lesquelles vous disposez d’autorisations d’accès.

### <a name="cause"></a>Cause

Voici les causes possibles de ce problème :

* Vous ne disposez pas de l’accès approprié sur les étendues sélectionnées.
* La requête ARG ne récupère pas les machines attendues.
* Runbook Worker hybride n’est pas installé sur les machines.

### <a name="resolution"></a>Résolution 

#### <a name="incorrect-access-on-selected-scopes"></a>Accès incorrect sur les étendues sélectionnées

Le portail Azure présente uniquement les machines pour lesquelles vous disposez d’un accès en écriture dans une étendue donnée. Si vous ne disposez pas de l’accès approprié pour une étendue, consultez [Tutoriel : Accorder un accès utilisateur aux ressources Azure à l’aide du portail Azure](../../role-based-access-control/quickstart-assign-role-user-portal.md).

#### <a name="arg-query-doesnt-return-expected-machines"></a>La requête ARG ne retourne pas les machines attendues

Suivez les étapes ci-dessous pour déterminer si vos requêtes fonctionnent correctement.

1. Exécutez une requête ARG dans le format indiqué ci-dessous dans le panneau Explorateur Resource Graph du portail Azure. Cette requête imite les filtres que vous avez sélectionnés au moment de créer le groupe dynamique dans Update Management. Consultez [Utiliser des groupes dynamiques avec Update Management](../update-management/configure-groups.md).

    ```kusto
    where (subscriptionId in~ ("<subscriptionId1>", "<subscriptionId2>") and type =~ "microsoft.compute/virtualmachines" and properties.storageProfile.osDisk.osType == "<Windows/Linux>" and resourceGroup in~ ("<resourceGroupName1>","<resourceGroupName2>") and location in~ ("<location1>","<location2>") )
    | project id, location, name, tags = todynamic(tolower(tostring(tags)))
    | where  (tags[tolower("<tagKey1>")] =~ "<tagValue1>" and tags[tolower("<tagKey2>")] =~ "<tagValue2>") // use this if "All" option selected for tags
    | where  (tags[tolower("<tagKey1>")] =~ "<tagValue1>" or tags[tolower("<tagKey2>")] =~ "<tagValue2>") // use this if "Any" option selected for tags
    | project id, location, name, tags
    ```

   Voici un exemple :

    ```kusto
    where (subscriptionId in~ ("20780d0a-b422-4213-979b-6c919c91ace1", "af52d412-a347-4bc6-8cb7-4780fbb00490") and type =~ "microsoft.compute/virtualmachines" and properties.storageProfile.osDisk.osType == "Windows" and resourceGroup in~ ("testRG","withinvnet-2020-01-06-10-global-resources-southindia") and location in~ ("australiacentral","australiacentral2","brazilsouth") )
    | project id, location, name, tags = todynamic(tolower(tostring(tags)))
    | where  (tags[tolower("ms-resource-usage")] =~ "azure-cloud-shell" and tags[tolower("temp")] =~ "temp")
    | project id, location, name, tags
    ```

2. Vérifiez si les machines que vous recherchez sont listées dans les résultats de la requête.

3. Si ce n’est pas le cas, il est probable que le filtre sélectionné dans le groupe dynamique présente un problème. Ajustez la configuration du groupe selon les besoins.

#### <a name="hybrid-runbook-worker-not-installed-on-machines"></a>Runbook Worker hybride non installé sur les machines

Bien que les machines apparaissent dans les résultats de la requête ARG, elles ne figurent toujours pas dans l’aperçu du groupe dynamique. Dans ce cas, il est possible que les machines ne soient pas désignées comme étant des Workers hybrides, ce qui explique qu’elles ne peuvent pas exécuter de tâches Azure Automation et Update Management. Pour faire en sorte que les machines attendues soient définies en tant que Runbooks Worker hybrides :

1. Sur le portail Azure, accédez au compte Automation pour une machine qui n’apparaît pas correctement.

2. Sélectionnez **Groupes Worker hybride** sous **Automatisation des processus**.

3. Sélectionnez l’onglet **Groupes Worker hybride du système**.

4. Vérifiez que le Worker hybride est présent pour la machine.

5. Si la machine n’est pas définie en tant que Worker hybride, apportez les modifications indiquées dans [Automatiser les ressources de votre centre de données ou de votre cloud à l’aide d’un Runbook Worker hybride](../automation-hybrid-runbook-worker.md).

6. Joignez la machine au groupe Runbook Worker hybride.

7. Répétez les étapes ci-dessus pour toutes les machines qui ne figurent pas dans l’aperçu.

## <a name="scenario-update-management-components-enabled-while-vm-continues-to-show-as-being-configured"></a><a name="components-enabled-not-working"></a>Scénario : Les composants d’Update Management sont activés, alors que la machine virtuelle continue à s’afficher comme étant configurée

### <a name="issue"></a>Problème

Vous continuez de voir le message suivant sur une machine virtuelle 15 minutes après le début du déploiement :

```error
The components for the 'Update Management' solution have been enabled, and now this virtual machine is being configured. Please be patient, as this can sometimes take up to 15 minutes.
```

### <a name="cause"></a>Cause

Cette erreur peut se produire pour les raisons suivantes :

* La communication avec le compte Automation est bloquée.

* Il existe un nom d’ordinateur en double avec des ID d’ordinateur source différents. Ce scénario se produit lorsqu’une machine virtuelle avec un nom d’ordinateur particulier est créée dans différents groupes de ressources et qu’elle dépend du même espace de travail Log Analytics dans l’abonnement.

* L’image de machine virtuelle déployée peut provenir d’un ordinateur cloné qui n’a pas été préparé avec System Preparation (sysprep) à l’aide de l’agent Log Analytics pour Windows installé.

### <a name="resolution"></a>Résolution

Pour vous aider à déterminer la nature exacte du problème que vous rencontrez avec la machine virtuelle, exécutez la requête suivante dans l’espace de travail Log Analytics lié à votre compte Automation.

```
Update
| where Computer contains "fillInMachineName"
| project TimeGenerated, Computer, SourceComputerId, Title, UpdateState 
```

#### <a name="communication-with-automation-account-blocked"></a>Communication bloquée avec le compte Automation

Accédez à [Planification réseau](../update-management/overview.md#ports) pour savoir quelles adresses et quels ports doivent être autorisés pour le fonctionnement d’Update Management.

#### <a name="duplicate-computer-name"></a>Nom d’ordinateur en double

Renommez vos machines virtuelles pour garantir l’unicité des noms dans leur environnement.

#### <a name="deployed-image-from-cloned-machine"></a>Image déployée provenant de l’ordinateur cloné

Si vous utilisez une image clonée, des noms d’ordinateur différents ont le même ID d’ordinateur source. Dans ce cas :

1. Dans votre espace de travail Log Analytics, supprimez la machine virtuelle de la recherche enregistrée pour la configuration d’étendue `MicrosoftDefaultScopeConfig-Updates` si celle-ci apparaît. Les recherches enregistrées se trouvent sous la section **Général** de votre espace de travail.

2. Exécutez l’applet de commande suivante.

    ```azurepowershell-interactive
    Remove-Item -Path "HKLM:\software\microsoft\hybridrunbookworker" -Recurse -Force
    ```

3. Exécutez `Restart-Service HealthService` pour redémarrer le service de contrôle d’intégrité. Cette opération recrée la clé et génère un nouvel UUID.

4. Si cette approche ne fonctionne pas, commencez par exécuter Sysprep sur l’image, puis installez l’agent Log Analytics pour Windows.

## <a name="scenario-you-receive-a-linked-subscription-error-when-you-create-an-update-deployment-for-machines-in-another-azure-tenant"></a><a name="multi-tenant"></a>Scénario : Vous recevez une erreur d’abonnement lié lorsque vous créez un déploiement de mise à jour pour les ordinateurs d’un autre locataire Azure.

### <a name="issue"></a>Problème

Vous rencontrez l’erreur suivante lorsque vous tentez de créer un déploiement de mise à jour pour les ordinateurs d’un autre locataire Azure :

```error
The client has permission to perform action 'Microsoft.Compute/virtualMachines/write' on scope '/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resourceGroupName/providers/Microsoft.Automation/automationAccounts/automationAccountName/softwareUpdateConfigurations/updateDeploymentName', however the current tenant '00000000-0000-0000-0000-000000000000' is not authorized to access linked subscription '00000000-0000-0000-0000-000000000000'.
```

### <a name="cause"></a>Cause

Cette erreur se produit lorsque vous créez un déploiement de mise à jour dans lequel des machines virtuelles Azure d’un autre locataire sont incluses.

### <a name="resolution"></a>Résolution

Utilisez la solution de contournement suivante pour planifier ces éléments. Vous pouvez utiliser la cmdlet [New-AzAutomationSchedule](/powershell/module/az.automation/new-azautomationschedule) avec le paramètre `ForUpdateConfiguration` pour créer une planification. Ensuite, utilisez l’applet de commande [New-AzAutomationSoftwareUpdateConfiguration](/powershell/module/Az.Automation/New-AzAutomationSoftwareUpdateConfiguration) et transférez les ordinateurs de l’autre locataire vers le paramètre `NonAzureComputer`. L’exemple suivant vous montre comment procéder :

```azurepowershell-interactive
$nonAzurecomputers = @("server-01", "server-02")

$startTime = ([DateTime]::Now).AddMinutes(10)

$s = New-AzAutomationSchedule -ResourceGroupName mygroup -AutomationAccountName myaccount -Name myupdateconfig -Description test-OneTime -OneTime -StartTime $startTime -ForUpdateConfiguration

New-AzAutomationSoftwareUpdateConfiguration  -ResourceGroupName $rg -AutomationAccountName $aa -Schedule $s -Windows -AzureVMResourceId $azureVMIdsW -NonAzureComputer $nonAzurecomputers -Duration (New-TimeSpan -Hours 2) -IncludedUpdateClassification Security,UpdateRollup -ExcludedKbNumber KB01,KB02 -IncludedKbNumber KB100
```

## <a name="scenario-unexplained-reboots"></a><a name="node-reboots"></a>Scénario : Redémarrages inexpliqués

### <a name="issue"></a>Problème

Même si vous avez défini l’option **Contrôle de redémarrage** sur **Ne jamais redémarrer**, les ordinateurs sont toujours redémarrés après l’installation de mises à jour.

### <a name="cause"></a>Cause

Windows Update peut être modifié par plusieurs clés de Registre dont chacune peut modifier les comportements de redémarrage.

### <a name="resolution"></a>Résolution

Passez en revue les clés de Registre répertoriées sous [Configuration automatique des mises à jour par modification du Registre](/windows/deployment/update/waas-wu-settings#configuring-automatic-updates-by-editing-the-registry) et [Clés de Registre utilisées pour gérer le redémarrage](/windows/deployment/update/waas-restart#registry-keys-used-to-manage-restart) pour vous assurer que vos ordinateurs sont configurés correctement.

## <a name="scenario-machine-shows-failed-to-start-in-an-update-deployment"></a><a name="failed-to-start"></a>Scénario : Un ordinateur indique « Échec du démarrage » dans un déploiement de mise à jour

### <a name="issue"></a>Problème

Un ordinateur affiche l’état `Failed to start`. Lorsque vous affichez les détails spécifiques de l’ordinateur, vous voyez l’erreur suivante :

```error
Failed to start the runbook. Check the parameters passed. RunbookName Patch-MicrosoftOMSComputer. Exception You have requested to create a runbook job on a hybrid worker group that does not exist.
```

### <a name="cause"></a>Cause

Ce problème peut se produire pour l’une des raisons suivantes :

* La machine n’existe plus.
* La machine est éteinte et injoignable.
* Le Worker hybride sur l’ordinateur n’est pas joignable, car l’ordinateur a un problème de connectivité réseau.
* L’agent Log Analytics a été mis à jour, ce qui a modifié l’ID d’ordinateur source.
* L’exécution de votre mise à jour a été limitée si vous avez atteint la limite de 200 tâches simultanées dans un compte Automation. Chaque déploiement est considéré comme une tâche, et chaque ordinateur dans un déploiement de mise à jour est également considéré comme une tâche. Toutes les autres tâches d’automatisation ou de déploiement de mise à jour en cours d’exécution dans votre compte Automation comptent dans la limite des tâches qu’il est possible d’effectuer simultanément.

### <a name="resolution"></a>Résolution

Lorsque c’est possible, utilisez les [groupes dynamiques](../update-management/configure-groups.md) pour vos déploiements de mise à jour. Vous pouvez en outre effectuer les étapes suivantes.

1. Vérifiez que votre ordinateur ou serveur est conforme à la [configuration requise](../update-management/overview.md#client-requirements).
2. Vérifiez la connectivité au Runbook Worker hybride à l’aide de l’utilitaire de résolution des problèmes de l’agent Runbook Worker hybride. Pour en savoir plus sur l’utilitaire de résolution des problèmes, consultez [Résoudre les problèmes de l’agent de mise à jour](update-agent-issues.md).

## <a name="scenario-updates-are-installed-without-a-deployment"></a><a name="updates-nodeployment"></a>Scénario : Les mises à jour sont installées sans déploiement

### <a name="issue"></a>Problème

Lorsque vous inscrivez un ordinateur Windows à Update Management, vous voyez les mises à jour installées sans déploiement.

### <a name="cause"></a>Cause

Sur Windows, les mises à jour sont installées automatiquement dès qu’elles sont disponibles. Ce comportement peut être à l’origine d’une certaine confusion si vous n’avez pas planifié de déploiement de mise à jour sur l’ordinateur.

### <a name="resolution"></a>Résolution

La valeur par défaut de 4 est affectée à la clé de Registre `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU` : `auto download and install`.

Nous recommandons aux clients Update Management d’affecter la valeur 3 à cette clé : `auto download but do not auto install`.

Pour plus d’informations, consultez [Configuration des mises à jour automatiques](/windows/deployment/update/waas-wu-settings#configure-automatic-updates).

## <a name="scenario-machine-is-already-registered-to-a-different-account"></a><a name="machine-already-registered"></a>Scénario : La machine est déjà inscrite sur un autre compte

### <a name="issue"></a>Problème

Vous recevez le message d’erreur suivant :

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception System.InvalidOperationException: {"Message":"Machine is already registered to a different account."}
```

### <a name="cause"></a>Cause

L’ordinateur est déjà déployé sur un autre espace de travail pour Update Management.

### <a name="resolution"></a>Résolution

1. Suivez les étapes de [Machines n’apparaissent pas dans le portail sous Update Management](#nologs) pour vous assurer que l’ordinateur est associé au bon espace de travail.
2. Nettoyez les artefacts sur l’ordinateur en [supprimant le groupe de runbooks hybrides](../automation-windows-hrw-install.md#remove-a-hybrid-worker-group), puis réessayez.

## <a name="scenario-machine-cant-communicate-with-the-service"></a><a name="machine-unable-to-communicate"></a>Scénario : L’ordinateur ne peut pas communiquer avec le service

### <a name="issue"></a>Problème

Vous recevez un des messages d’erreur suivants :

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception System.Net.Http.HttpRequestException: An error occurred while sending the request. ---> System.Net.WebException: The underlying connection was closed: An unexpected error occurred on a receive. ---> System.ComponentModel.Win32Exception: The client and server can't communicate, because they do not possess a common algorithm
```

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception Newtonsoft.Json.JsonReaderException: Error parsing positive infinity value.
```

```error
The certificate presented by the service <wsid>.oms.opinsights.azure.com was not issued by a certificate authority used for Microsoft services. Contact your network administrator to see if they are running a proxy that intercepts TLS/SSL communication.
```

```error
Access is denied. (Exception form HRESULT: 0x80070005(E_ACCESSDENIED))
```

### <a name="cause"></a>Cause

Un proxy, une passerelle ou un pare-feu bloquent peut-être la communication réseau.

### <a name="resolution"></a>Résolution

Passez en revue votre réseau, et vérifiez que les ports et les adresses appropriés sont autorisés. Consultez [Configuration réseau requise](../automation-hybrid-runbook-worker.md#network-planning) pour obtenir la liste des ports et des adresses nécessaires pour Update Management et pour les Runbooks Worker hybrides.

## <a name="scenario-unable-to-create-self-signed-certificate"></a><a name="unable-to-create-selfsigned-cert"></a>Scénario : Impossible de créer un certificat auto-signé

### <a name="issue"></a>Problème

Vous recevez un des messages d’erreur suivants :

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception AgentService.HybridRegistration. PowerShell.Certificates.CertificateCreationException: Failed to create a self-signed certificate. ---> System.UnauthorizedAccessException: Access is denied.
```

### <a name="cause"></a>Cause

Le Runbook Worker hybride n’a pas pu générer de certificat autosigné.

### <a name="resolution"></a>Résolution

Vérifiez que le compte système a accès en lecture au dossier **C:\ProgramData\Microsoft\Crypto\RSA**, puis réessayez.

## <a name="scenario-the-scheduled-update-failed-with-a-maintenancewindowexceeded-error"></a><a name="mw-exceeded"></a>Scénario : Échec de la mise à jour planifiée avec une erreur MaintenanceWindowExceeded

### <a name="issue"></a>Problème

La fenêtre de maintenance par défaut pour les mises à jour est de 120 minutes. Vous pouvez augmenter la taille de la fenêtre de maintenance à un maximum de 6 heures, soit 360 minutes.

### <a name="resolution"></a>Résolution

Pour comprendre pourquoi cela s’est produit pendant l’exécution d’une mise à jour après qu’elle a démarré avec succès, [vérifiez la sortie du travail](../update-management/deploy-updates.md#view-results-of-a-completed-update-deployment) de la machine affectée dans l’exécution. Vous trouverez peut-être des messages d’erreur spécifiques provenant de votre machine, effectuer des recherches sur ces erreurs et entreprendre des actions pour les résoudre.  

Modifiez tous les déploiements de mise à jour planifiés ayant échoué et augmentez la taille de la fenêtre de maintenance.

Pour plus d’informations sur les fenêtres de maintenance, consultez [Installer des mises à jour](../update-management/deploy-updates.md#schedule-an-update-deployment).

## <a name="scenario-machine-shows-as-not-assessed-and-shows-an-hresult-exception"></a><a name="hresult"></a>Scénario : L’ordinateur s’affiche comme « Non évalué » et présente une exception HRESULT

### <a name="issue"></a>Problème

* Certaines de vos machines portent la mention `Not assessed` sous **Conformité**, et vous voyez un message d’exception sous celles-ci.
* Vous voyez un code d’erreur HRESULT dans le portail.

### <a name="cause"></a>Cause

L’agent de mise à jour (agent Windows Update sur Windows ; le gestionnaire de package pour une distribution Linux) n’est pas configuré correctement. Update Management s’appuie sur l’agent de mise à jour de l’ordinateur pour fournir les mises à jour nécessaires, l’état du correctif et les résultats des correctifs déployés. Sans ces informations, Update Management ne peut pas générer de rapport exact sur les correctifs nécessaires ou installés.

### <a name="resolution"></a>Résolution

Essayez d’exécuter des mises à jour localement sur l’ordinateur. Si cette opération échoue, cela signifie généralement qu’il existe une erreur de configuration pour l’agent de mise à jour.

C’est souvent dû à des problèmes de configuration de réseau et de pare-feu. Effectuez les vérifications suivantes pour corriger le problème.

* Pour Linux, consultez la documentation appropriée pour vous assurer que vous pouvez atteindre le point de terminaison réseau de votre référentiel de packages.

* Pour Windows, vérifiez la configuration de votre agent, comme indiqué dans [Les mises à jour ne se téléchargent pas à partir du point de terminaison intranet (WSUS/SCCM)](/windows/deployment/update/windows-update-troubleshooting#updates-arent-downloading-from-the-intranet-endpoint-wsussccm).

  * Si les ordinateurs sont configurés pour Windows Update, vérifiez que vous pouvez atteindre les points de terminaison décrits dans [Problèmes liés à HTTP/au proxy](/windows/deployment/update/windows-update-troubleshooting#issues-related-to-httpproxy).
  * Si les ordinateurs sont configurés pour WSUS (Windows Server Update Services), vérifiez que vous pouvez atteindre le serveur WSUS configuré par la [clé de Registre WUServer](/windows/deployment/update/waas-wu-settings).

Si vous voyez un HRESULT, double-cliquez sur l’exception affichée en rouge pour voir le message d’exception. Dans le tableau suivant, recherchez les résolutions possibles ou les actions recommandées.

|Exception  |Résolution ou action  |
|---------|---------|
|`Exception from HRESULT: 0x……C`     | Recherchez le code d’erreur pertinent dans la [liste des codes d’erreur de Windows Update](https://support.microsoft.com/help/938205/windows-update-error-code-list) pour obtenir des détails supplémentaires sur la cause de l’exception.        |
|`0x8024402C`</br>`0x8024401C`</br>`0x8024402F`      | Ceci indique des problèmes de connectivité réseau. Assurez-vous que votre ordinateur dispose de la connectivité réseau pour Update Management. Consultez la section [Planification réseau](../update-management/overview.md#ports) pour obtenir la liste des ports et des adresses.        |
|`0x8024001E`| L’opération de mise à jour a échoué, car le service ou le système était en cours d’arrêt.|
|`0x8024002E`| Le service Windows Update est désactivé.|
|`0x8024402C`     | Si vous utilisez un serveur WSUS, assurez-vous que les valeurs de Registre de `WUServer` et `WUStatusServer` sous la clé de Registre `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate` spécifient le bon serveur WSUS.        |
|`0x80072EE2`|Il y a un problème de connectivité réseau ou problème de communication avec un serveur WSUS configuré. Vérifiez les paramètres WSUS et assurez-vous que le service est accessible à partir du client.|
|`The service cannot be started, either because it is disabled or because it has no enabled devices associated with it. (Exception from HRESULT: 0x80070422)`     | Assurez-vous que le service Windows Update (wuauserv) est en cours d’exécution et qu’il n’est pas désactivé.        |
|`0x80070005`| Une erreur d'accès refusé peut être due à l’une des raisons suivantes :<br> Ordinateur infecté<br> Les paramètres Windows Update ne sont pas correctement configurés<br> Erreur d’autorisation de fichier avec le dossier %WinDir%\SoftwareDistribution<br> Espace disque insuffisant sur le lecteur système (C:)
|Toute autre exception générique     | Effectuez une recherche sur Internet pour découvrir des résolutions possibles et contactez votre support technique local.         |

L’examen du fichier **%Windir%\Windowsupdate.log** peut aussi vous aider à déterminer les causes possibles. Pour savoir comment lire le journal, consultez l’article [Comment lire le fichier Windowsupdate.log](https://support.microsoft.com/help/902093/how-to-read-the-windowsupdate-log-file).

Vous pouvez également télécharger et exécuter l’[utilitaire de résolution des problèmes de Windows Update](https://support.microsoft.com/help/4027322/windows-update-troubleshooter) pour vérifier si l’ordinateur présente des problèmes liés à Windows Update.

> [!NOTE]
> La documentation de l’[outil de résolution des problèmes de Windows Update](https://support.microsoft.com/help/4027322/windows-update-troubleshooter) indique qu’il est destiné à être utilisé sur des clients Windows, mais il fonctionne également sur Windows Server.

## <a name="scenario-update-run-returns-failed-status-linux"></a>Scénario : L’exécution de la mise à jour retourne l’état Échec (Linux)

### <a name="issue"></a>Problème

Une mise à jour démarre mais rencontre des erreurs pendant l’exécution.

### <a name="cause"></a>Cause

Causes possibles :

* Le Gestionnaire de package n’est pas sain.
* L’agent de mise à jour (WUA pour Windows, gestionnaire de package spécifique à la distribution pour Linux) est mal configuré.
* Des packages spécifiques interfèrent avec la mise à jour corrective informatique.
* L’ordinateur est inaccessible.
* Les mises à jour ont des dépendances qui n’ont pas été résolues.

### <a name="resolution"></a>Résolution

Si des échecs se produisent pendant l’exécution d’une mise à jour après qu’elle a démarré avec succès, [vérifiez la sortie du travail](../update-management/deploy-updates.md#view-results-of-a-completed-update-deployment) de la machine affectée dans l’exécution. Vous trouverez peut-être des messages d’erreur spécifiques provenant de votre machine, effectuer des recherches sur ces erreurs et entreprendre des actions pour les résoudre. Update Management nécessite que le Gestionnaire de package soit sain pour que les déploiements des mises à jour réussisse.

Si des correctifs, des packages ou des mises à jour spécifiques s’affichent immédiatement avant l’échec de la tâche, vous pouvez essayer [d’exclure](../update-management/deploy-updates.md#schedule-an-update-deployment) ces éléments du déploiement de mise à jour suivant. Pour collecter des informations de journal à partir de Windows Update, consultez [Fichiers journaux de Windows Update](/windows/deployment/update/windows-update-logs).

Si vous ne pouvez pas résoudre un problème de mise à jour corrective, créez une copie du fichier **/var/opt/microsoft/omsagent/run/automationworker/omsupdatemgmt.log** et conservez-le à des fins de dépannage avant le démarrage du déploiement de mise à jour suivant.

## <a name="patches-arent-installed"></a>Les correctifs ne sont pas installés

### <a name="machines-dont-install-updates"></a>Les ordinateurs n’installent pas les mises à jour

Essayez d’exécuter les mises à jour directement sur la machine. Si l’ordinateur ne parvient pas à se mettre à jour, consultez la [liste des erreurs potentielles dans le guide de résolution des problèmes](#hresult).

Si les mises à jour s’exécutent localement, essayez de supprimer et de réinstaller l’agent sur l’ordinateur en suivant les instructions données dans [Supprimer une machine virtuelle d’Update Management](../update-management/remove-vms.md).

### <a name="i-know-updates-are-available-but-they-dont-show-as-available-on-my-machines"></a>Je sais que des mises à jour sont disponibles, mais elles n’apparaissent pas comme étant disponibles sur mes ordinateurs

Cela se produit souvent lorsque les machines sont configurées pour recevoir des mises à jour de WSUS ou de Microsoft Endpoint Configuration Manager, mais que WSUS et Configuration Manager n’ont pas approuvé les mises à jour.

Vous pouvez vérifier si les machines sont configurées pour WSUS et SCCM en faisant une référence croisée de la clé de Registre `UseWUServer` avec les clés de Registre de la section [Configuration de mises à jour automatiques par modification du Registre](https://support.microsoft.com/help/328010/how-to-configure-automatic-updates-by-using-group-policy-or-registry-s) de cet article.

Si les mises à jour ne sont pas approuvées dans WSUS, elles ne sont pas installées. Vous pouvez rechercher les mises à jour non approuvées dans Log Analytics en exécutant la requête suivante.

  ```kusto
  Update | where UpdateState == "Needed" and ApprovalSource == "WSUS" and Approved == "False" | summarize max(TimeGenerated) by Computer, KBID, Title
  ```

### <a name="updates-show-as-installed-but-i-cant-find-them-on-my-machine"></a>Les mises à jour s’affichent comme étant installées, mais je ne les trouve pas sur ma machine

Les mises à jour sont souvent remplacées par d’autres mises à jour. Pour plus d’informations, consultez [La mise à jour est remplacée](/windows/deployment/update/windows-update-troubleshooting#the-update-is-not-applicable-to-your-computer) dans le guide de résolution des problèmes liés à Windows Update.

### <a name="installing-updates-by-classification-on-linux"></a>Installation de mises à jour par classification sur Linux

Le déploiement de mises à jour sur Linux par classification (« Mises à jour critiques et de sécurité ») a des limitations importantes, en particulier pour CentOS. Ces limitations sont documentées à la [page Vue d’ensemble d’Update Management](../update-management/overview.md#linux).

### <a name="kb2267602-is-consistently-missing"></a>L’article KB2267602 est systématiquement manquant

L’article KB2267602 a trait à la [mise à jour de définitions Windows Defender](https://www.microsoft.com/wdsi/definitions). Il est mis à jour quotidiennement.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou que vous ne pouvez pas le résoudre, utilisez l’un des canaux suivants pour obtenir une aide supplémentaire.

* Obtenez des réponses de la part d’experts Azure via les [Forums Azure](https://azure.microsoft.com/support/forums/).
* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client.
* Signaler un incident au support Azure Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.
