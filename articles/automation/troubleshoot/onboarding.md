---
title: Résoudre les problèmes de déploiement des fonctionnalités Azure Automation
description: Cet article explique comment résoudre les problèmes de déploiement des fonctionnalités Azure Automation.
services: automation
ms.date: 06/30/2020
ms.topic: conceptual
ms.service: automation
ms.openlocfilehash: a6b8384193e821e6c41a0d4d979cda51f6c65b3a
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92070484"
---
# <a name="troubleshoot-feature-deployment-issues"></a>Résoudre les problèmes de déploiement de fonctionnalités

Vous pouvez recevoir des messages d’erreur lorsque vous déployez la fonctionnalité Update Management ou la fonctionnalité Change Tracking and Inventory d’Azure Automation sur vos machines virtuelles. Cet article décrit les erreurs qui pourraient se produire et explique comment les résoudre.

## <a name="known-issues"></a>Problèmes connus

### <a name="scenario-renaming-a-registered-node-requires-unregister-or-register-again"></a><a name="node-rename"></a>Scénario : La modification du nom d’un nœud inscrit requiert l’annulation de l’inscription ou une réinscription

#### <a name="issue"></a>Problème

Un nœud est inscrit sur Azure Automation, puis le nom d’ordinateur du système d’exploitation est modifié. Les rapports du nœud continuent à s’afficher avec le nom d’origine.

#### <a name="cause"></a>Cause

La modification du nom d’un nœud inscrit n’a pas pour effet de modifier son nom dans Azure Automation.

#### <a name="resolution"></a>Résolution

Annulez l’inscription du nœud dans le service State Configuration d’Azure Automation, puis réinscrivez-le. Les rapports publiés sur le service avant ce délai ne seront plus disponibles.

### <a name="scenario-re-signing-certificates-via-https-proxy-isnt-supported"></a><a name="resigning-cert"></a>Scénario : La nouvelle signature de certificats via un proxy HTTPS n’est pas prise en charge

#### <a name="issue"></a>Problème

Quand vous vous connectez via un proxy qui met fin au trafic HTTPS, puis chiffre à nouveau le trafic à l’aide d’un nouveau certificat, le service n’autorise pas la connexion.

#### <a name="cause"></a>Cause

Azure Automation ne prend pas en charge la nouvelle signature de certificats utilisés pour chiffrer le trafic.

#### <a name="resolution"></a>Résolution

Il n’existe actuellement aucun moyen de contourner ce problème.

## <a name="general-errors"></a>Erreurs générales.

### <a name="scenario-feature-deployment-fails-with-the-message-the-solution-cannot-be-enabled"></a><a name="missing-write-permissions"></a>Scénario : Le déploiement de la fonctionnalité échoue avec le message : « La solution ne peut pas être activée ».

#### <a name="issue"></a>Problème

L’un des messages suivants s’affiche lorsque vous tentez d’activer une fonctionnalité sur une machine virtuelle :

```error
The solution cannot be enabled due to missing permissions for the virtual machine or deployments
```

```error
The solution cannot be enabled on this VM because the permission to read the workspace is missing
```

#### <a name="cause"></a>Cause

Cette erreur est due à des autorisations incorrectes ou manquantes sur la machine virtuelle ou dans l’espace de travail, ou bien pour l'utilisateur.

#### <a name="resolution"></a>Résolution

Vérifiez que vous disposez des [autorisations de déploiement de fonctionnalité](../automation-role-based-access-control.md#feature-setup-permissions) nécessaires, puis réessayez de déployer la fonctionnalité. Si vous recevez le message d’erreur `The solution cannot be enabled on this VM because the permission to read the workspace is missing`, consultez les [informations de dépannage](update-management.md#failed-to-enable-error) suivantes.

### <a name="scenario-feature-deployment-fails-with-the-message-failed-to-configure-automation-account-for-diagnostic-logging"></a><a name="diagnostic-logging"></a>Scénario : Le déploiement de la fonctionnalité échoue avec le message : « Échec de la configuration du compte Automation pour la journalisation des diagnostics »

#### <a name="issue"></a>Problème

Le message suivant s’affiche lorsque vous tentez d’activer une fonctionnalité sur une machine virtuelle :

```error
Failed to configure automation account for diagnostic logging
```

#### <a name="cause"></a>Cause

Cette erreur peut survenir si le niveau tarifaire ne correspond pas au modèle de facturation de l’abonnement. Pour plus d’informations, consultez [Surveiller l’utilisation et l’estimation des coûts dans Azure Monitor](../../azure-monitor/platform/usage-estimated-costs.md).

#### <a name="resolution"></a>Résolution

Créez votre espace de travail Log Analytics manuellement, puis répétez la procédure de déploiement de fonctionnalité pour sélectionner l’espace de travail créé.

### <a name="scenario-computergroupqueryformaterror"></a><a name="computer-group-query-format-error"></a>Scénario : ComputerGroupQueryFormatError

#### <a name="issue"></a>Problème

Ce code d’erreur signifie que la requête de recherche de groupe d’ordinateurs enregistrée utilisée pour cibler la fonctionnalité n’est pas au bon format. 

#### <a name="cause"></a>Cause

Vous avez peut-être modifié la requête, ou le système l’a peut-être modifiée.

#### <a name="resolution"></a>Résolution

Vous pouvez supprimer la requête pour cette fonctionnalité, puis réactiver la fonctionnalité, ce qui aura pour effet de recréer la requête. La requête se trouve dans votre espace de travail, sous **Recherches enregistrées**. Le nom de la requête est **MicrosoftDefaultComputerGroup**, et la catégorie de la requête est le nom de la fonctionnalité associée. Si plusieurs fonctionnalités sont activées, la requête **MicrosoftDefaultComputerGroup** s’affiche plusieurs fois sous **Recherches enregistrées**.

### <a name="scenario-policyviolation"></a><a name="policy-violation"></a>Scénario : PolicyViolation

#### <a name="issue"></a>Problème

Ce code d’erreur indique que le déploiement a échoué en raison d’une violation d’une ou de plusieurs stratégies.

#### <a name="cause"></a>Cause 

Une stratégie empêche l’opération de se terminer.

#### <a name="resolution"></a>Résolution

Pour déployer correctement la fonctionnalité, vous devez modifier la stratégie indiquée. Étant donné qu’il existe plusieurs types de stratégies peuvent être définis, les modifications requises dépendent de la stratégie enfreinte. Par exemple, si une stratégie est définie sur un groupe de ressources qui refuse l’autorisation de modifier le contenu de certaines ressources contenues, vous pouvez choisir l’un de ces correctifs :

* Retirer la stratégie.
* Essayez d’activer la fonctionnalité pour un autre groupe de ressources.
* Ciblez de nouveau la stratégie sur une ressource spécifique, par exemple un compte Automation.
* Révisez l’ensemble de ressources pour le refus desquelles cette stratégie a été configurée.

Consultez les notifications dans l’angle supérieur droit du portail Azure, ou accédez au groupe de ressources comportant votre compte Automation, puis sélectionnez **Déploiements** sous **Paramètres** pour afficher le déploiement en échec. Pour en savoir plus sur Azure Policy, consultez [Présentation d’Azure Policy](../../governance/policy/overview.md?toc=%2fazure%2fautomation%2ftoc.json).

### <a name="scenario-errors-trying-to-unlink-a-workspace"></a><a name="unlink"></a>Scénario : Erreurs lors de la tentative de dissociation d’un espace de travail

#### <a name="issue"></a>Problème

Vous recevez le message d’erreur suivant lorsque vous tentez de dissocier un espace de travail :

```error
The link cannot be updated or deleted because it is linked to Update Management and/or ChangeTracking Solutions.
```

#### <a name="cause"></a>Cause

Cette erreur se produit lorsque des fonctionnalités de votre espace de travail Log Analytics qui dépendent de votre compte Automation et de l’espace de travail Log Analytics associé sont toujours actives.

### <a name="resolution"></a>Résolution

Dans votre espace de travail, supprimez les ressources des fonctionnalités suivantes, si vous les utilisez :

* Update Management
* Suivi des modifications et inventaire
* Démarrer/arrêter des machines virtuelles pendant les heures creuses

Une fois les ressources des fonctionnalités supprimées, vous pouvez dissocier votre espace de travail. Il est important de supprimer tous les artefacts existants de ces fonctionnalités dans votre espace de travail et votre compte Automation.

* Pour Update Management, supprimez les **déploiements de mises à jour (planifications)** de votre compte Automation.
* Pour « Start/Stop VMs during off-hours », supprimez tous les verrous des composants de la fonctionnalité dans votre compte Automation sous **Paramètres** > **Verrous**. Pour plus d’informations, consultez [Supprimer la fonctionnalité](../automation-solution-vm-management.md#remove-the-feature).

## <a name="log-analytics-for-windows-extension-failures"></a><a name="mma-extension-failures"></a>Log Analytics pour les échecs de l’extension Windows

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)] 

Une installation de l’agent Log Analytics pour l’extension Windows peut échouer pour diverses raisons. La section suivante décrit les problèmes de déploiement de fonctionnalité qui peuvent provoquer des échecs lors du déploiement de l’agent Log Analytics pour l’extension Windows.

>[!NOTE]
>L’agent Log Analytics pour Windows est le nom utilisé actuellement dans Azure Automation pour l’agent Microsoft Monitoring Agent (MMA).

### <a name="scenario-an-exception-occurred-during-a-webclient-request"></a><a name="webclient-exception"></a>Scénario : Une exception s’est produite lors d’une requête WebClient

Log Analytics pour l’extension Windows de la machine virtuelle ne peut pas communiquer avec les ressources externes, et le déploiement échoue.

#### <a name="issue"></a>Problème

Voici quelques exemples des messages d’erreur qui peuvent s’afficher :

```error
Please verify the VM has a running VM agent, and can establish outbound connections to Azure storage.
```

```error
'Manifest download error from https://<endpoint>/<endpointId>/Microsoft.EnterpriseCloud.Monitoring_MicrosoftMonitoringAgent_australiaeast_manifest.xml. Error: UnknownError. An exception occurred during a WebClient request.
```

#### <a name="cause"></a>Cause

Voici des causes possibles de cette erreur :

* Un proxy configuré dans la machine virtuelle autorise uniquement certains ports.
* Un paramètre du pare-feu a bloqué l’accès aux adresses et aux ports nécessaires.

#### <a name="resolution"></a>Résolution

Vérifiez que les bons ports et adresses sont ouverts à la communication. Pour obtenir la liste des ports et adresses, consultez [Planification de votre réseau](../automation-hybrid-runbook-worker.md#network-planning).

### <a name="scenario-install-failed-because-of-transient-environment-issues"></a><a name="transient-environment-issue"></a>Scénario : L’installation a échoué en raison de problèmes temporaires liés à l’environnement

L’installation de l’extension Log Analytics pour Windows a échoué pendant le déploiement, car une autre installation ou action bloque l’installation.

#### <a name="issue"></a>Problème

Voici quelques exemples des messages d’erreur qui peuvent s’afficher :

```error
The Microsoft Monitoring Agent failed to install on this machine. Please try to uninstall and reinstall the extension. If the issue persists, please contact support.
```

```error
'Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.4) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.4\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 1618'
```

```error
'Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.2) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.2\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 1601'
```

#### <a name="cause"></a>Cause

Voici des causes possibles de cette erreur :

* Une autre installation est en cours d’exécution.
* Le redémarrage du système est déclenché pendant le déploiement du modèle.

#### <a name="resolution"></a>Résolution

Cette erreur est temporaire par nature. Recommencez le déploiement pour installer l’extension.

### <a name="scenario-installation-timeout"></a><a name="installation-timeout"></a>Scénario : Expiration du délai d’installation

L’installation de l’extension Log Analytics pour Windows n’a pas pu se terminer, car le délai d’attente a expiré.

#### <a name="issue"></a>Problème

Voici un exemple de message d’erreur qui pourrait s’afficher :

```error
Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.4) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.4\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 15614
```

#### <a name="cause"></a>Cause

Ce type d’erreur se produit car la machine virtuelle est surchargée pendant l’installation.

### <a name="resolution"></a>Résolution

Essayez d’installer l’agent Log Analytics pour l’extension Windows quand la machine virtuelle est moins sollicitée.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas ici ou que vous ne pouvez pas le résoudre, utilisez un des canaux suivants pour obtenir une aide supplémentaire :

* Obtenez des réponses de la part d’experts Azure via les [Forums Azure](https://azure.microsoft.com/support/forums/).
* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Le Support Azure fournit à la communauté Azure des réponses, un support technique et des experts.
* Signaler un incident au support Azure Accédez au [site du support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez **Obtenir de l’aide**.