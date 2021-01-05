---
title: Résoudre des problèmes liés à Azure Automation State Configuration
description: Cet article explique comment dépanner et résoudre les problèmes liés à Azure Automation State Configuration.
services: automation
ms.service: automation
ms.subservice: ''
author: mgoedtel
ms.author: magoedte
ms.date: 04/16/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 8043369ebfef23ed84ccff8e7428fbd2048e10b0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86187215"
---
# <a name="troubleshoot-azure-automation-state-configuration-issues"></a>Résoudre des problèmes liés à Azure Automation State Configuration

Cet article fournit des informations sur la détection et la résolution des problèmes qui surviennent lorsque vous compilez ou déployez des configurations dans Azure Automation State Configuration. Pour obtenir des informations générales sur la fonctionnalité State Configuration, consultez [Vue d’ensemble d’Azure Automation State Configuration](../automation-dsc-overview.md).

## <a name="diagnose-an-issue"></a>Diagnostiquer un problème

Quand vous recevez une erreur de compilation ou de déploiement pour la configuration, voici quelques étapes à suivre pour essayer de diagnostiquer le problème.

### <a name="1-ensure-that-your-configuration-compiles-successfully-on-the-local-machine"></a>1. Vérifier que votre configuration est correctement compilée sur l’ordinateur local

Azure Automation State Configuration est basé sur PowerShell Desired State Configuration (DSC). Vous trouverez la documentation relative à la syntaxe et au langage DSC dans les [documents PowerShell DSC](/powershell/scripting/overview).

En compilant une configuration DSC sur votre ordinateur local, vous pouvez détecter et résoudre les erreurs courantes, telles que :

   - Modules manquants.
   - Erreurs de syntaxe.
   - Erreurs de logique.

### <a name="2-view-dsc-logs-on-your-node"></a>2. Afficher les journaux DSC sur votre nœud

Si votre configuration est correctement compilée, mais échoue lorsqu’elle est appliquée à un nœud, vous pouvez trouver les informations détaillées dans les journaux DSC. Pour plus d’informations sur l’emplacement de ces journaux, consultez [Où se trouvent les journaux des événements DSC ?](/powershell/scripting/dsc/troubleshooting/troubleshooting#where-are-dsc-event-logs)

Le module [xDscDiagnostics](https://github.com/PowerShell/xDscDiagnostics) peut vous aider à analyser des informations détaillées dans les journaux DSC. Si vous contactez le support, ces journaux sont nécessaires pour diagnostiquer votre problème.

Vous pouvez installer le module `xDscDiagnostics` sur votre ordinateur local en suivant les instructions figurant dans la section [Install the stable version module](https://github.com/PowerShell/xDscDiagnostics#install-the-stable-version-module) (Installer le module de version stable).

Pour installer le module `xDscDiagnostics` sur votre ordinateur Azure, utilisez la commande [Invoke-AzVMRunCommand](/powershell/module/az.compute/invoke-azvmruncommand?view=azps-3.7.0). Vous pouvez aussi utiliser l’option **Run Command** dans le portail Azure, en suivant les étapes mentionnées dans [Exécuter des scripts PowerShell dans votre machine virtuelle Windows avec Run Command](../../virtual-machines/windows/run-command.md).

Pour plus d’informations sur l’utilisation de **xDscDiagnostics**, consultez [Utilisation de xDscDiagnostics pour analyser les journaux DSC](/powershell/scripting/dsc/troubleshooting/troubleshooting#using-xdscdiagnostics-to-analyze-dsc-logs). Consultez également les [cmdlets xDscDiagnostics](https://github.com/PowerShell/xDscDiagnostics#cmdlets).

### <a name="3-ensure-that-nodes-and-the-automation-workspace-have-required-modules"></a>3. S’assurer que les nœuds et l’espace de travail Automation disposent des modules nécessaires

DSC dépend des modules installés sur le nœud. Lorsque vous utilisez Azure Automation State Configuration, importez tous les modules requis dans votre compte Automation à l’aide des étapes indiquées dans [Importer des modules](../shared-resources/modules.md#import-modules). Les configurations peuvent également avoir une dépendance sur des versions de modules spécifiques. Pour plus d’informations, consultez [Résoudre les problèmes liés aux modules](shared-resources.md#modules).

## <a name="scenario-a-configuration-with-special-characters-cant-be-deleted-from-the-portal"></a><a name="unsupported-characters"></a>Scénario : Une configuration contenant des caractères spéciaux ne peut pas être supprimée à partir du portail

### <a name="issue"></a>Problème

Lorsque vous tentez de supprimer une configuration DSC à partir du portail, vous rencontrez l’erreur suivante :

```error
An error occurred while deleting the DSC configuration '<name>'.  Error-details: The argument configurationName with the value <name> is not valid.  Valid configuration names can contain only letters,  numbers, and underscores.  The name must start with a letter.  The length of the name must be between 1 and 64 characters.
```

### <a name="cause"></a>Cause

Cette erreur est due à un problème temporaire dont la résolution est prévue.

### <a name="resolution"></a>Résolution

Utilisez l’applet de commande [Remove-AzAutomationDscConfiguration](/powershell/module/Az.Automation/Remove-AzAutomationDscConfiguration?view=azps-3.7.0) pour supprimer la configuration.

## <a name="scenario-failed-to-register-the-dsc-agent"></a><a name="failed-to-register-agent"></a>Scénario : Impossible d’inscrire l’agent DSC

### <a name="issue"></a>Problème

Lorsque [Set-DscLocalConfigurationManager](/powershell/module/psdesiredstateconfiguration/set-dsclocalconfigurationmanager?view=powershell-5.1) ou une autre applet de commande DSC, vous recevez l’erreur :

```error
Registration of the Dsc Agent with the server
https://<location>-agentservice-prod-1.azure-automation.net/accounts/00000000-0000-0000-0000-000000000000 failed. The
underlying error is: Failed to register Dsc Agent with AgentId 00000000-0000-0000-0000-000000000000 with the server htt
ps://<location>-agentservice-prod-1.azure-automation.net/accounts/00000000-0000-0000-0000-000000000000/Nodes(AgentId='00000000-0000-0000-0000-000000000000'). .
    + CategoryInfo          : InvalidResult: (root/Microsoft/...gurationManager:String) [], CimException
    + FullyQualifiedErrorId : RegisterDscAgentCommandFailed,Microsoft.PowerShell.DesiredStateConfiguration.Commands.Re
   gisterDscAgentCommand
    + PSComputerName        : <computerName>
```

### <a name="cause"></a>Cause

Cette erreur est habituellement due à un pare-feu, la machine se trouvant derrière un serveur proxy, ou à d’autres erreurs de réseau.

### <a name="resolution"></a>Résolution

Vérifiez que votre machine a accès aux points de terminaison appropriés pour DSC et réessayez. Pour obtenir la liste des ports et adresses nécessaires, consultez [Planification réseau](../automation-dsc-overview.md#network-planning).

## <a name="a-nameunauthorizedscenario-status-reports-return-the-response-code-unauthorized"></a><a name="unauthorized"><a/>Scénario : Les rapports d’état retournent le code de réponse « Non autorisé »

### <a name="issue"></a>Problème

Lors de l’inscription d’un nœud à l’aide d’Azure Automation State Configuration, vous recevez l’un des messages d’erreur suivants :

```error
The attempt to send status report to the server https://{your Automation account URL}/accounts/xxxxxxxxxxxxxxxxxxxxxx/Nodes(AgentId='xxxxxxxxxxxxxxxxxxxxxxxxx')/SendReport returned unexpected response code Unauthorized.
```

```error
VM has reported a failure when processing extension 'Microsoft.Powershell.DSC / Registration of the Dsc Agent with the server failed.
```

### <a name="cause"></a>Cause

Ce problème est dû à un certificat incorrect ou expiré. Consultez [Réinscrire un nœud](../automation-dsc-onboarding.md#re-register-a-node).

Ce problème pourrait aussi être dû à une configuration de proxy n'autorisant pas l’accès à * **.azure-automation.net**. Pour plus d’informations, consultez [Configuration de réseaux privés](../automation-dsc-overview.md#network-planning). 

### <a name="resolution"></a>Résolution

Utilisez les étapes suivantes pour réinscrire le nœud DSC défaillant.

#### <a name="step-1-unregister-the-node"></a>Étape 1 : Annuler l’inscription du nœud

1. Dans le portail Azure, accédez à **Accueil** > **Comptes Automation** > (votre compte Automation) > **State Configuration (DSC)** .
1. Sélectionnez **Nœuds**, puis sélectionnez le nœud qui pose problème.
1. Sélectionnez **Désinscrire** pour annuler l’inscription du nœud.

#### <a name="step-2-uninstall-the-dsc-extension-from-the-node"></a>Étape 2 : Désinstaller l’extension DSC du nœud

1. Sur le portail Azure, accédez à **Accueil** > **Machine virtuelle** > (nœud défaillant) > **Extensions**.
1. Sélectionnez **Microsoft.PowerShell.DSC**, l’extension PowerShell DSC.
1. Sélectionnez **Désinstaller** pour désinstaller l’extension.

#### <a name="step-3-remove-all-bad-or-expired-certificates-from-the-node"></a>Étape 3 : Supprimer du nœud tous les certificats incorrects ou ayant expiré

Sur le nœud défaillant et à partir d’une invite PowerShell avec élévation de privilèges, exécutez ces commandes :

```powershell
$certs = @()
$certs += dir cert:\localmachine\my | ?{$_.FriendlyName -like "DSC"}
$certs += dir cert:\localmachine\my | ?{$_.FriendlyName -like "DSC-OaaS Client Authentication"}
$certs += dir cert:\localmachine\CA | ?{$_.subject -like "CN=AzureDSCExtension*"}
"";"== DSC Certificates found: " + $certs.Count
$certs | FL ThumbPrint,FriendlyName,Subject
If (($certs.Count) -gt 0)
{ 
    ForEach ($Cert in $certs) 
    {
        RD -LiteralPath ($Cert.Pspath) 
    }
}
```

#### <a name="step-4-reregister-the-failing-node"></a>Étape 4 : Réinscrire le nœud défaillant

1. Dans le portail Azure, accédez à **Accueil** > **Comptes Automation** > (votre compte Automation) > **State Configuration (DSC)** .
1. Sélectionner **Nœuds**.
1. Sélectionnez **Ajouter**.
1. Sélectionnez le nœud défaillant.
1. Sélectionnez **Connecter** et sélectionnez les options souhaitées.

## <a name="scenario-node-is-in-failed-status-with-a-not-found-error"></a><a name="failed-not-found"></a>Scénario : Le nœud est en état d’échec avec une erreur « Introuvable »

### <a name="issue"></a>Problème

Un rapport pour le nœud indique un état Échec et contient l’erreur :

```error
The attempt to get the action from server https://<url>//accounts/<account-id>/Nodes(AgentId=<agent-id>)/GetDscAction failed because a valid configuration <guid> cannot be found.
```

### <a name="cause"></a>Cause

Cette erreur se produit généralement quand le nœud est affecté à un nom de configuration, par exemple **ABC** au lieu d’un nom de configuration de nœud, par exemple **ABC.WebServer**.

### <a name="resolution"></a>Résolution

* Veillez à bien d’affecter le nœud au nom de configuration du nœud et non pas au nom de configuration.
* Vous pouvez affecter une configuration de nœud à un nœud à l'aide du portail Azure ou d’une cmdlet PowerShell.

  * Dans le portail Azure, accédez à **Accueil** > **Comptes Automation** > (votre compte Automation) > **State Configuration (DSC)** . Sélectionnez ensuite un nœud et **Affecter la configuration du nœud**.
  * Utilisez l’applet de commande [Set-AzAutomationDscNode](/powershell/module/Az.Automation/Set-AzAutomationDscNode?view=azps-3.7.0).

## <a name="scenario-no-node-configurations-mof-files-were-produced-when-a-configuration-was-compiled"></a><a name="no-mof-files"></a>Scénario : Aucune configuration de nœud (fichiers MOF) n’a été produite lors de la compilation de configuration

### <a name="issue"></a>Problème

Votre tâche de compilation DSC s’interrompt avec l’erreur :

```error
Compilation completed successfully, but no node configuration **.mof** files were generated.
```

### <a name="cause"></a>Cause

Quand l’expression qui suit le mot-clé `Node` dans la configuration DSC s’évalue à `$null`, aucune configuration de nœud n’est générée.

### <a name="resolution"></a>Résolution

Utilisez l’une des solutions suivantes pour résoudre ce problème :

* Vérifiez que l’expression en regard du mot clé `Node` dans la définition de la configuration n’est pas Null.
* Si vous effectuez une transmission de [ConfigurationData](../automation-dsc-compile.md) pendant la compilation de la configuration, vérifiez que vous transmettez les valeurs que la configuration attend des données de configuration.

## <a name="scenario-the-dsc-node-report-becomes-stuck-in-the-in-progress-state"></a><a name="dsc-in-progress"></a>Scénario : Le rapport du nœud DSC se bloque à l’état En cours

### <a name="issue"></a>Problème

La sortie de l’agent DSC est la suivante :

```error
No instance found with given property values
```

### <a name="cause"></a>Cause

Vous avez mis à niveau votre version de Windows Management Framework (WMF) et vous avez endommagé Windows Management Instrumentation (WMI).

### <a name="resolution"></a>Résolution

Suivez les instructions dans [Limitations et problèmes connus dans DSC](/powershell/scripting/wmf/known-issues/known-issues-dsc).

## <a name="scenario-unable-to-use-a-credential-in-a-dsc-configuration"></a><a name="issue-using-credential"></a>Scénario : Il est impossible d’utiliser des informations d’identification dans une configuration DSC

### <a name="issue"></a>Problème

Votre tâche de compilation DSC est interrompue avec l’erreur :

```error
System.InvalidOperationException error processing property 'Credential' of type <some resource name>: Converting and storing an encrypted password as plaintext is allowed only if PSDscAllowPlainTextPassword is set to true.
```

### <a name="cause"></a>Cause

Vous avez utilisé des informations d’identification dans une configuration, mais n’avez pas fourni la bonne valeur `ConfigurationData` pour définir `PSDscAllowPlainTextPassword` sur true pour chaque configuration de nœud.

### <a name="resolution"></a>Résolution

Assurez-vous de transmettre la bonne valeur `ConfigurationData` pour définir `PSDscAllowPlainTextPassword` sur true pour chaque configuration de nœud mentionnée dans la configuration. Consultez [Compilation de configurations DSC dans Azure Automation State Configuration](../automation-dsc-compile.md).

## <a name="scenario-failure-processing-extension-error-when-enabling-a-machine-from-a-dsc-extension"></a><a name="failure-processing-extension"></a>Scénario : Erreur « Échec lors du traitement de l’extension » au moment de l’activation d’une machine à partir d’une extension DSC

### <a name="issue"></a>Problème

Lors de l’activation d’une machine à l’aide d’une extension DSC, un échec se produit avec l’erreur :

```error
VM has reported a failure when processing extension 'Microsoft.Powershell.DSC'. Error message: \"DSC COnfiguration 'RegistrationMetaConfigV2' completed with error(s). Following are the first few: Registration of the Dsc Agent with the server <url> failed. The underlying error is: The attempt to register Dsc Agent with Agent Id <ID> with the server <url> return unexpected response code BadRequest. .\".
```

### <a name="cause"></a>Cause

Cette erreur se produit généralement quand le nœud est affecté à un nom de configuration de nœud qui n’existe pas dans le service.

### <a name="resolution"></a>Résolution

* Assurez-vous que vous affectez le nœud avec un nom qui correspond exactement au nom dans le service.
* Vous pouvez choisir de ne pas inclure le nom de configuration de nœud, ce qui entraîne l’activation du nœud, sans affectation de configuration de nœud.

## <a name="scenario-one-or-more-errors-occurred-error-when-registering-a-node-by-using-powershell"></a><a name="cross-subscription"></a>Scénario : Erreur « Une ou plusieurs erreurs se sont produites » lors de l’inscription d’un nœud avec PowerShell

### <a name="issue"></a>Problème

Lors de l’inscription d’un nœud à l’aide de [Register-AzAutomationDSCNode](/powershell/module/az.automation/register-azautomationdscnode?view=azps-3.7.0) ou de [Register-AzureRMAutomationDSCNode](/powershell/module/azurerm.automation/register-azurermautomationdscnode?view=azurermps-6.13.0), vous recevez l’erreur suivante :

```error
One or more errors occurred.
```

### <a name="cause"></a>Cause

Cette erreur se produit lorsque vous essayez d’inscrire un nœud dans un autre abonnement que celui utilisé par le compte Automation.

### <a name="resolution"></a>Résolution

Traitez ce nœud d’un autre abonnement comme s’il était défini dans un cloud distinct ou localement. Inscrivez le nœud à l’aide de l’une des options suivantes pour l’activation des machines :

* Windows : [Machines physiques/virtuelles Windows locales ou dans un cloud autre qu’Azure/AWS](../automation-dsc-onboarding.md#enable-physicalvirtual-windows-machines).
* Linux : [Machines physiques/virtuelles Linux locales, dans Azure, ou dans un cloud autre qu’Azure](../automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines).

## <a name="scenario-provisioning-has-failed-error-message"></a><a name="agent-has-a-problem"></a>Scénario : Message d’erreur : « Échec du provisionnement »

### <a name="issue"></a>Problème

Lors de l’inscription d’un nœud, l’erreur suivante s’affiche :

```error
Provisioning has failed
```

### <a name="cause"></a>Cause

Ce message s’affiche lorsqu’il y a un problème de connectivité entre le nœud et Azure.

### <a name="resolution"></a>Résolution

Déterminez si votre nœud se trouve dans un réseau virtuel privé (VPN) ou s’il a d’autres problèmes pour se connecter à Azure. Consultez [Résoudre les problèmes de déploiement de fonctionnalités](onboarding.md).

## <a name="scenario-failure-with-a-general-error-when-applying-a-configuration-in-linux"></a><a name="failure-linux-temp-noexec"></a>Scénario : Échec avec une erreur générale lors de l’application d’une configuration dans Linux

### <a name="issue"></a>Problème

Lorsque vous appliquez une configuration dans Linux, un échec se produit avec l’erreur :

```error
This event indicates that failure happens when LCM is processing the configuration. ErrorId is 1. ErrorDetail is The SendConfigurationApply function did not succeed.. ResourceId is [resource]name and SourceInfo is ::nnn::n::resource. ErrorMessage is A general error occurred, not covered by a more specific error code..
```

### <a name="cause"></a>Cause

Si l’emplacement de **/tmp** est défini sur `noexec`, la version actuelle de DSC ne parvient pas à appliquer les configurations.

### <a name="resolution"></a>Résolution

Supprimez l’option `noexec` de l’emplacement **/tmp**.

## <a name="scenario-node-configuration-names-that-overlap-can-result-in-a-bad-release"></a><a name="compilation-node-name-overlap"></a>Scénario : Les noms des configurations de nœuds qui se chevauchent peuvent entraîner une version incorrecte

### <a name="issue"></a>Problème

Lorsque vous utilisez un script de configuration unique pour générer plusieurs configurations de nœuds et que les noms de certaines d’entre-elles sont des sous-ensembles d’autres noms, le service de compilation peut aboutir à l’attribution d’une configuration incorrecte. Ce problème survient uniquement lorsque vous utilisez un seul script pour générer des configurations avec des données de configuration par nœud, et uniquement quand le chevauchement de nom se produit au début de la chaîne. C’est par exemple le cas si un script de configuration unique est utilisé pour générer des configurations en fonction de données de nœuds transmises sous forme de table de hachage à l’aide d’applets de commande, et que les données de nœuds incluent des serveurs nommés **server** et **1server**.

### <a name="cause"></a>Cause

Ce problème connu avec le service de compilation.

### <a name="resolution"></a>Résolution

La meilleure solution de contournement consiste à compiler localement ou dans un pipeline CI/CD, et à charger les fichiers MOF de la configuration de nœud directement dans le service. Si la compilation dans le service est une exigence, la meilleure solution suivante consiste à fractionner les travaux de compilation de manière qu’il n’y ait pas de chevauchement des noms.

## <a name="scenario-gateway-timeout-error-on-dsc-configuration-upload"></a><a name="gateway-timeout"></a>Scénario : Erreur de délai de dépassement de la passerelle lors du chargement de la configuration DSC

#### <a name="issue"></a>Problème

Une erreur `GatewayTimeout` s’affiche lorsque vous chargez une configuration DSC. 

### <a name="cause"></a>Cause

Les configurations DSC dont la compilation prennent beaucoup de temps peuvent provoquer cette erreur.

### <a name="resolution"></a>Résolution

Vous pouvez accélérer l’analyse des configurations DSC en incluant explicitement le paramètre `ModuleName` pour tous les appels [Import-DSCResource](/powershell/scripting/dsc/configurations/import-dscresource?view=powershell-5.1).

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas ici ou que vous ne pouvez pas le résoudre, utilisez un des canaux suivants pour obtenir une aide supplémentaire :

* Obtenez des réponses de la part d’experts Azure via les [Forums Azure](https://azure.microsoft.com/support/forums/).
* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Le Support Azure fournit à la communauté Azure des réponses, un support technique et des experts.
* Signaler un incident au support Azure Accédez au [site du support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez **Obtenir de l’aide**.
