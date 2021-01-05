---
title: Exécution d'un Runbook dans Azure Automation
description: Cet article fournit une vue d’ensemble du traitement des runbooks dans Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 10/06/2020
ms.topic: conceptual
ms.openlocfilehash: 6ac7d99f4a47711f9974d30d877a3237eec15443
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92078831"
---
# <a name="runbook-execution-in-azure-automation"></a>Exécution d'un Runbook dans Azure Automation

Dans Azure Automation, l'automatisation des processus vous permet de créer et de gérer PowerShell, PowerShell Workflow et des runbooks graphiques. Pour plus d'informations, consultez [Utilisation de runbooks Azure Automation](automation-runbook-types.md). 

Automation exécute vos runbooks selon la logique avec laquelle ils ont été définis. Si un runbook est interrompu, il redémarre au début. Ce comportement vous contraint d’écrire des runbooks qui prennent en charge le redémarrage si des problèmes temporaires se produisent.

Dans Azure Automation, le démarrage d'un runbook crée une tâche, qui correspond à une instance d’exécution unique du runbook. Chaque tâche accède à des ressources Azure en établissant une connexion à votre abonnement Azure. La tâche accède uniquement aux ressources de votre centre de données si ces ressources sont accessibles depuis le cloud public.

Azure Automation charge un Worker d’exécuter chaque tâche pendant l’exécution du runbook. Même si les travaux sont partagés par de nombreux comptes Azure, les tâches des différents comptes Automation sont isolées les unes des autres. Vous ne pouvez pas contrôler le Worker qui traite les requêtes de votre tâche.

Quand vous affichez la liste des runbooks sur le portail Azure, elle indique l’état de chaque tâche qui a été démarrée pour chaque runbook. Azure Automation stocke les journaux de tâches pendant une durée maximale de 30 jours.

Le diagramme suivant illustre le cycle de vie d’une tâche de runbook pour les [runbooks PowerShell](automation-runbook-types.md#powershell-runbooks), les [runbooks de workflow PowerShell](automation-runbook-types.md#powershell-workflow-runbooks) et les [runbooks graphiques](automation-runbook-types.md#graphical-runbooks).

![États des tâches - Workflow PowerShell](./media/automation-runbook-execution/job-statuses.png)

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="runbook-execution-environment"></a>Environnement d’exécution de runbook

Les runbooks d’Azure Automation peuvent s’exécuter dans un bac à sable Azure ou dans un [runbook Worker hybride](automation-hybrid-runbook-worker.md). 

Les runbooks conçus pour s’authentifier et s’exécuter sur des ressources dans Azure s'exécutent dans un bac à sable Azure, à savoir un environnement partagé utilisé par plusieurs travaux. Les travaux qui utilisent le même bac à sable sont liés par les limitations de ressources du bac à sable. L’environnement de bac à sable Azure ne prend pas en charge les opérations interactives. Il empêche l’accès à tous les serveurs COM hors processus. Il requiert également l’utilisation de fichiers MOF locaux pour les runbooks effectuant des appels Win32.

Vous pouvez également utiliser un [runbook Worker hybride](automation-hybrid-runbook-worker.md) pour exécuter des runbooks directement sur l’ordinateur qui héberge le rôle et avec les ressources disponibles dans l’environnement. Azure Automation stocke et gère les runbooks et les remet à un ou plusieurs ordinateurs assignés.

>[!NOTE]
>Pour s’exécuter sur un runbook Worker hybride Linux, vos scripts doivent être signés et le Worker configuré en conséquence. Sinon, la [validation de la signature doit être désactivée](automation-linux-hrw-install.md#turn-off-signature-validation).

Le tableau suivant liste certaines tâches d’exécution de runbook avec l’environnement d’exécution recommandé indiqué pour chacune d’elles.

|Tâche|Recommandation|Notes|
|---|---|---|
|Intégration à des ressources Azure|Bac à sable Azure|Hébergé dans Azure, l’authentification est plus simple. Si vous utilisez un Runbook Worker hybride sur une machine virtuelle Azure, vous pouvez [utiliser une authentification de runbook avec des identités managées](automation-hrw-run-runbooks.md#runbook-auth-managed-identities).|
|Obtention de performances optimales pour gérer les ressources Azure|Bac à sable Azure|Le script est exécuté dans le même environnement, dont la latence est moindre.|
|Réduction des coûts d'exploitation|Bac à sable Azure|Il n’y a pas de surcharge de calcul ni de besoin d’une machine virtuelle.|
|Exécution d’un script de longue durée|Runbook Worker hybride|Les bacs à sable Azure présentent des [limites de ressources](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits).|
|Interaction avec les services locaux|Runbook Worker hybride|Accédez directement à l’ordinateur hôte, ou aux ressources dans d’autres environnements cloud ou dans l’environnement local. |
|Imposer des logiciels et des fichiers exécutables tiers|Runbook Worker hybride|Vous gérez le système d’exploitation et pouvez installer des logiciels.|
|Surveillance d'un fichier ou d'un dossier avec un runbook|Runbook Worker hybride|Utilisez une [tâche Watcher](automation-watchers-tutorial.md) sur un runbook Worker hybride.|
|Exécution d’un script gourmand en ressources|Runbook Worker hybride| Les bacs à sable Azure présentent des [limites de ressources](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits).|
|Utilisation de modules ayant des exigences spécifiques| Runbook Worker hybride|Quelques exemples :</br> WinSCP – dépend de winscp.exe </br> Administration IIS - Dépendance sur l'activation ou la gestion d’IIS|
|Installation d’un module avec un programme d’installation|Runbook Worker hybride|Les modules pour bac à sable doivent prendre en charge la copie.|
|Utilisation de runbooks ou de modules nécessitant une version de .NET Framework différente de la version 4.7.2|Runbook Worker hybride|Les bacs à sable Azure prennent en charge .NET Framework 4.7.2. La mise à niveau vers une autre version n’est pas prise en charge.|
|Exécution de scripts qui nécessitent une élévation|Runbook Worker hybride|Les bacs à sable ne permettent pas l’élévation. Avec un runbook Worker hybride, vous pouvez désactiver le Contrôle de compte d’utilisateur (UAC) et utiliser [Invoke-Command](/powershell/module/microsoft.powershell.core/invoke-command) au moment d’exécuter la commande qui nécessite une élévation.|
|Exécuter des scripts nécessitant un accès à WMI (Windows Management Instrumentation)|Runbook Worker hybride|Les tâches s’exécutant dans les bacs à sable du cloud ne peuvent pas accéder au fournisseur WMI. |

## <a name="temporary-storage-in-a-sandbox"></a>Stockage temporaire dans un environnement de type « bac à sable » (« sandbox »)

Si vous devez créer des fichiers temporaires dans le cadre de votre logique runbook, vous pouvez utiliser le dossier Temp (autrement dit, `$env:TEMP`) dans l’environnement de type « bac à sable » (« sandbox ») d’Azure pour les runbooks s’exécutant dans Azure. La seule limitation est que vous ne pouvez pas utiliser plus de 1 Go d’espace disque, qui est le quota pour chaque bac à sable. Lorsque vous travaillez avec des workflows PowerShell, ce scénario peut provoquer un problème, car les workflows PowerShell utilisent des points de contrôle et le script peut être retenté dans un bac à sable différent.

L’environnement de bac à sable hybride vous permet d’utiliser `C:\temp` en fonction de la disponibilité du stockage sur un Runbook Worker hybride. Toutefois, selon les suggestions applicables à Azure VM, vous ne devez pas utiliser le [disque temporaire](../virtual-machines/managed-disks-overview.md#temporary-disk) sur Windows ou Linux pour les données qui doivent être conservées.

## <a name="resources"></a>Ressources

Vos runbooks doivent inclure une logique pour gérer des [ressources](/rest/api/resources/resources), par exemple, des machines virtuelles, le réseau et des ressources sur le réseau. Les ressources sont liées à un abonnement Azure et les runbooks requièrent des informations d’identification appropriées pour accéder à une ressource. Pour obtenir un exemple de gestion des ressources dans un runbook, consultez [Gérer les ressources](manage-runbooks.md#handle-resources).

## <a name="security"></a>Sécurité

Azure Automation utilise [Azure Security Center (ASC)](../security-center/security-center-introduction.md) pour garantir la sécurité de vos ressources et détecter les compromissions dans les systèmes Linux. La sécurité est fournie dans vos charges de travail, que les ressources se trouvent dans Azure ou non. Voir [Présentation de l’authentification dans Azure Automation](automation-security-overview.md).

ASC applique des contraintes aux utilisateurs qui peuvent exécuter des scripts, signés ou non, sur une machine virtuelle. Si vous êtes un utilisateur disposant d’un accès racine à une machine virtuelle, vous devez configurer explicitement la machine avec une signature numérique, ou la désactiver. Sinon, vous ne pouvez exécuter un script que pour appliquer des mises à jour du système d’exploitation après avoir créé un compte Automation et activé la fonctionnalité appropriée.

## <a name="subscriptions"></a>Abonnements

Un [abonnement](/office365/enterprise/subscriptions-licenses-accounts-and-tenants-for-microsoft-cloud-offerings) Azure est un accord avec Microsoft qui vous autorise à utiliser un ou plusieurs services cloud pour lesquels vous êtes facturé. Pour Azure Automation, chaque abonnement est lié à un compte Azure Automation et vous pouvez [créer plusieurs abonnements](manage-runbooks.md#work-with-multiple-subscriptions) dans le compte.

## <a name="credentials"></a>Informations d'identification

Un runbook nécessite des [informations d’identification](shared-resources/credentials.md) appropriées pour accéder à toutes les ressources, qu’il s’agisse d’Azure ou de systèmes tiers. Ces informations d’identification sont stockées dans Azure Automation, Key Vault, etc.  

## <a name="azure-monitor"></a>Azure Monitor

Azure Automation utilise [Azure Monitor](../azure-monitor/overview.md) pour superviser ses opérations sur les machines. Les opérations nécessitent un espace de travail Log Analytics et un [agent Log Analytics](../azure-monitor/platform/log-analytics-agent.md).

### <a name="log-analytics-agent-for-windows"></a>Agent Log Analytics pour Windows

L’[agent Log Analytics pour Windows](../azure-monitor/platform/agent-windows.md) fonctionne avec Azure Monitor pour gérer les machines virtuelles et les ordinateurs physiques Windows. Les ordinateurs peuvent être exécutés dans Azure ou dans un environnement non-Azure, par exemple un centre de données local.

>[!NOTE]
>L’agent Log Analytics pour Windows s’appelait auparavant Microsoft Monitoring Agent (MMA).

### <a name="log-analytics-agent-for-linux"></a>Agent Log Analytics pour Linux

L’[agent Log Analytics pour Linux](../azure-monitor/platform/agent-linux.md) fonctionne de la même façon que l’agent pour Windows, mais il connecte les ordinateurs Linux à Azure Monitor. L’agent est installé avec un compte d’utilisateur **nxautomation** qui autorise l’exécution de commandes nécessitant des autorisations racine, par exemple, sur un runbook Worker hybride. Le compte **nxautomation** est un compte système qui ne requiert aucun mot de passe.

Le compte **nxautomation** avec les autorisations sudo correspondantes doit être présent lors de l’[installation d’un runbook Worker hybride Linux](automation-linux-hrw-install.md). Si vous essayez d’installer le Worker et que le compte n’est pas présent ou ne dispose pas des autorisations appropriées, l’installation échoue.

Vous ne devez pas modifier les autorisations du dossier `sudoers.d` ou sa propriété. L’autorisation sudo est requise pour le compte **nxautomation** et les autorisations ne doivent pas être supprimées. Limiter ce principe à certains dossiers ou à certaines commandes peut entraîner une modification critique.

Les journaux disponibles pour l’agent Log Analytics et le compte **nxautomation** sont les suivants :

* /var/opt/microsoft/omsagent/log/omsagent.log - Journal de l’agent Log Analytics
* /var/opt/microsoft/omsagent/run/automationworker/worker.log - Journal de travail Automation

>[!NOTE]
>L’utilisateur **nxautomation** activé en lien avec Update Management exécute uniquement des runbooks signés.

## <a name="runbook-permissions"></a>Autorisations de Runbook

Un runbook a besoin d’autorisations pour l’authentification auprès d’Azure, par le biais des informations d’identification. Consultez [Gérer les comptes d’identification Azure Automation](manage-runas-account.md).

## <a name="modules"></a>Modules

Azure Automation prend en charge un certain nombre de modules par défaut, dont des modules AzureRM (AzureRM.Automation) et un module contenant plusieurs cmdlets internes. Les modules installables sont également pris en charge, dont les modules Az (Az.Automation) actuellement utilisés de préférence aux modules AzureRM. Pour plus d’informations sur les modules disponibles pour vos runbooks et configurations DSC, consultez [Gérer des modules dans Azure Automation](shared-resources/modules.md).

## <a name="certificates"></a>Certificats

Azure Automation utilise des [certificats](shared-resources/certificates.md) pour l’authentification auprès Azure, ou les ajoute à Azure ou à des ressources tierces. Les certificats sont stockés de façon sécurisée pour l’accès par des runbooks et des configurations DSC.

Vos runbooks peuvent utiliser des certificats auto-signés, qui ne sont pas signés par une autorité de certification (CA). Voir [Créer un certificat](shared-resources/certificates.md#create-a-new-certificate).

## <a name="jobs"></a>travaux

Azure Automation prend en charge un environnement pour exécuter des tâches à partir du même compte Automation. Un même runbook peut avoir beaucoup de tâches qui s’exécutent simultanément. Plus vous exécutez de travaux simultanément, plus ils peuvent être répartis vers le même bac à sable. 

Les tâches qui s’exécutent dans le même processus de bac à sable peuvent s’influencer mutuellement. C’est par exemple ce qui arrive quand l’applet de commande [Disconnect-AzAccount](/powershell/module/az.accounts/disconnect-azaccount) est exécutée. Dans ce cas, cette applet de commande déconnecte chaque tâche du runbook dans le processus de bac à sable partagé. Pour obtenir un exemple d’utilisation de ce scénario, consultez [Prévention des travaux simultanés](manage-runbooks.md#prevent-concurrent-jobs).

>[!NOTE]
>Les tâches PowerShell démarrées à partir d’un runbook qui s’exécute dans un bac à sable Azure peut ne pas s’exécuter en [mode langage PowerShell](/powershell/module/microsoft.powershell.core/about/about_language_modes) complet.

### <a name="job-statuses"></a>États des tâches

Le tableau suivant décrit les différents états possibles d’une tâche. Vous pouvez afficher un résumé de l’état de toutes les tâches du runbook ou explorer les détails d’une tâche spécifique du runbook dans le portail Azure. Vous pouvez également configurer une intégration à votre espace de travail Log Analytics pour transférer l’état et les flux de travaux du runbook. Pour plus d’informations sur l’intégration avec les journaux d’activité Azure Monitor, consultez [Transférer l’état d’un travail et des flux de travail d’Automation vers les journaux d’activité Azure Monitor](automation-manage-send-joblogs-log-analytics.md). Pour obtenir un exemple d’utilisation des états dans un runbook, voir aussi [Obtenir les états des tâches](manage-runbooks.md#obtain-job-statuses).

| Statut | Description |
|:--- |:--- |
| Completed |La tâche s'est terminée avec succès. |
| Échec |Un runbook graphique ou un graphique de workflow PowerShell n’a pas pu être compilé. Un runbook PowerShell n’a pas pu démarrer ou la tâche a rencontré une exception. Consultez [Types de runbooks Azure Automation](automation-runbook-types.md).|
| Échec, en attente de ressources |La tâche a échoué, car elle a atteint la limite de [répartition de charge équilibrée](#fair-share) trois fois et a démarré à partir du même point de contrôle ou à partir du début du Runbook à chaque fois. |
| Mis en file d'attente. |La tâche attend que les ressources d’un Worker Automation deviennent disponibles pour pouvoir démarrer. |
| Reprise |Le système reprend la tâche suspendue. |
| Exécution en cours |La tâche est en cours d'exécution. |
| En cours d'exécution, en attente de ressources |La tâche a été déchargée, car elle a atteint la limite de répartition de charge équilibrée. Elle va bientôt reprendre depuis son dernier point de contrôle. |
| Démarrage en cours |La tâche a été attribuée à un travail et le système la démarre. |
| Arrêté |La tâche a été arrêtée par l'utilisateur avant qu'elle n'ait été terminée. |
| En cours d’arrêt |Le système arrête la tâche. |
| Interrompu |S’applique aux [runbooks graphiques et aux runbooks de workflow PowerShell](automation-runbook-types.md) uniquement. La tâche a été suspendue par l'utilisateur, le système ou une commande du Runbook. Si un runbook n’a pas de point de contrôle défini, il démarre à partir du début. S’il a un point de contrôle défini, il peut recommencer et reprendre à partir de son dernier point de contrôle. Le système suspend uniquement le runbook quand une exception se produit. Par défaut, la variable `ErrorActionPreference` est définie sur Continue, ce qui indique que la tâche continue de s’exécuter en cas d’erreur. Si la variable de préférence est définie sur Stop, la tâche est suspendue en cas d’erreur.  |
| Suspension |S’applique aux [runbooks graphiques et aux runbooks de workflow PowerShell](automation-runbook-types.md) uniquement. Le système tente de suspendre la tâche à la demande de l’utilisateur. Le Runbook doit atteindre son prochain point de contrôle avant de pouvoir être suspendu. S’il a déjà passé son dernier point de contrôle, il se termine avant d’être suspendu. |

## <a name="activity-logging"></a>Journalisation de l’activité

L’exécution de runbooks dans Azure Automation consigne les détails dans un journal d’activité pour le compte Automation. Pour plus d’informations sur l’utilisation du journal, consultez [Récupérer les détails à partir du journal d’activité](manage-runbooks.md#retrieve-details-from-activity-log).

## <a name="exceptions"></a>Exceptions

Cette section décrit des méthodes pour gérer les exceptions ou les problèmes intermittents qui se produisent dans vos runbooks. Par exemple, une exception WebSocket. Une gestion correcte des exceptions empêche des pannes de réseau temporaires de provoquer une défaillance de vos runbooks.

### <a name="erroractionpreference"></a>ErrorActionPreference

La variable [ErrorActionPreference](/powershell/module/microsoft.powershell.core/about/about_preference_variables#erroractionpreference) détermine la façon dont PowerShell répond à une erreur qui ne met pas fin à l’exécution. Les erreurs avec fin d’exécution provoquent systématiquement l'arrêt et ne sont pas affectées par `ErrorActionPreference`.

Lorsque le runbook utilise `ErrorActionPreference`, une erreur qui ne met normalement pas fin à l’exécution, comme `PathNotFound`Get-ChildItem[ issue de la cmdlet ](/powershell/module/microsoft.powershell.management/get-childitem) qui empêche le runbook d’aboutir. L’exemple suivant illustre l’utilisation de `ErrorActionPreference`. La commande finale [Write-Output](/powershell/module/microsoft.powershell.utility/write-output) ne s’exécute jamais, car le script s’arrête.

```powershell-interactive
$ErrorActionPreference = 'Stop'
Get-ChildItem -path nofile.txt
Write-Output "This message will not show"
```

### <a name="try-catch-finally"></a>Try Catch

[Try Catch Finally](/powershell/module/microsoft.powershell.core/about/about_try_catch_finally) est utilisé dans les scripts PowerShell pour gérer les erreurs qui mettent fin à l’exécution. Le script peut utiliser ce mécanisme pour intercepter des exceptions spécifiques ou générales. L’instruction `catch` doit être utilisée pour suivre ou essayer de gérer les erreurs. L’exemple suivant essaie de télécharger un fichier qui n’existe pas. Il intercepte l’exception `System.Net.WebException` et retourne la dernière valeur pour toute autre exception.

```powershell-interactive
try
{
   $wc = new-object System.Net.WebClient
   $wc.DownloadFile("http://www.contoso.com/MyDoc.doc")
}
catch [System.Net.WebException]
{
    "Unable to download MyDoc.doc from http://www.contoso.com."
}
catch
{
    "An error occurred that could not be resolved."
}
```

### <a name="throw"></a>Throw

[Throw](/powershell/module/microsoft.powershell.core/about/about_throw) peut être utilisée pour générer une erreur avec fin d’exécution. Ce mécanisme peut vous être utile au moment de définir votre propre logique dans un runbook. Si le script remplit un critère qui doit l’arrêter, il peut utiliser l’instruction `throw` pour s’arrêter. L’exemple suivant utilise cette instruction pour afficher un paramètre de fonction obligatoire.

```powershell-interactive
function Get-ContosoFiles
{
  param ($path = $(throw "The Path parameter is required."))
  Get-ChildItem -Path $path\*.txt -recurse
}
```

## <a name="errors"></a>Erreurs

Vos runbooks doivent gérer les erreurs. Azure Automation prend en charge deux types d’erreurs PowerShell : celles qui mettent fin à l’exécution et celles qui ne mettent pas fin à l’exécution. 

Quand elles se produisent, les erreurs qui mettent fin à l’exécution arrêtent l’exécution d’un runbook. Le runbook s’arrête et l’état de la tâche devient Échec.

Les erreurs qui ne mettent pas fin à l’exécution n’empêchent pas la poursuite d’un script. Par exemple, un runbook qui utilise l’applet de commande `Get-ChildItem` avec un chemin qui n’existe pas génère une erreur qui ne met pas fin à l’exécution. PowerShell détecte que le chemin n’existe pas, il génère une erreur et il passe au dossier suivant. Dans ce cas, l’erreur n’affecte pas l’état Échec à la tâche du runbook, laquelle peut même arriver à son terme. Pour forcer l’arrêt d’un runbook après une erreur sans fin d’exécution, utilisez `ErrorAction Stop` avec l’applet de commande.

## <a name="calling-processes"></a>Processus d’appel

Les runbooks qui s’exécutent dans les bacs à sable Azure ne prennent pas en charge les processus d’appel, comme les exécutables (fichiers **.exe**) ou les sous-processus. Cela s’explique par le fait qu’un bac à sable Azure est un processus partagé qui s’exécute dans un conteneur qui n’a peut-être pas accès à toutes les API sous-jacentes. Pour les scénarios qui exigent un logiciel tiers ou des appels à des sous-processus, exécutez le runbook sur un [runbook Worker hybride](automation-hybrid-runbook-worker.md).

## <a name="device-and-application-characteristics"></a>Caractéristiques des appareils et des applications

Les tâches de runbook dans les bacs à sable Azure ne peuvent avoir accès aux caractéristiques des appareils ou des applications. Pour interroger les métriques de performances sur Windows, notamment celles, courantes, qui portent sur l’utilisation de la mémoire et du processeur, L’API la plus utilisée est WMI. Cependant, quelle que soit l’API utilisée, les tâches qui s’exécutent dans le cloud ne peuvent avoir accès à l’implémentation Microsoft de WBEM (Web-Based Enterprise Management). Cette plateforme est basée sur CIM (Common Information Model), qui fait office de standard sectoriel pour la définition des caractéristiques des appareils et des applications.

## <a name="webhooks"></a>webhooks

Les services externes, par exemple, Azure DevOps Services et GitHub, peuvent démarrer un runbook dans Azure Automation. Pour effectuer ce type de démarrage, le service utilise un [webhook](automation-webhooks.md) via une requête HTTP unique. L’utilisation d’un Webhook permet de démarrer runbooks sans implémenter une fonctionnalité Azure Automation complète.

## <a name="shared-resources"></a><a name="fair-share"></a>Ressources partagées

Pour partager des ressources entre tous les runbooks dans le cloud, Azure utilise un concept appelé « répartition de charge équilibrée ». Grâce à la répartition de charge équilibrée, Azure décharge ou arrête toute tâche exécutée depuis plus de trois heures. Les tâches des [runbooks PowerShell](automation-runbook-types.md#powershell-runbooks) et des [runbooks Python](automation-runbook-types.md#python-runbooks) sont arrêtées et non redémarrées, et leur état devient Arrêté.

Pour les tâches Azure Automation de longue durée, il est recommandé d’utiliser un runbook Worker hybride. Les Runbook Workers hybrides ne sont pas limités par la répartition de charge équilibrée et n'imposent aucune limitation en termes de durée d'exécution des runbooks. Les autres [limites](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits) du travail s’appliquent à la fois aux bacs à sable Azure et aux Runbooks Workers hybrides. Les runbooks Workers hybrides ne sont pas limités par la répartition de charge équilibrée de trois heures, mais vous devez développer des runbooks qui s’exécutent sur les Workers qui prennent en charge les redémarrages après des problèmes inattendus au niveau de l’infrastructure locale.

Une autre option consiste à optimiser un runbook en utilisant des runbooks enfants. Par exemple, il peut arriver que votre runbook exécute la même fonction en boucle sur plusieurs ressources, comme une opération de base de données sur diverses bases de données. Vous pouvez déplacer cette fonction dans un [runbook enfant](automation-child-runbooks.md) et faire en sorte que votre runbook l’appelle à l’aide de [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook). Les runbooks enfants s’exécutent en parallèle dans des processus distincts.

L’utilisation de runbooks enfants diminue le délai d’exécution total du runbook parent. Votre runbook peut utiliser la cmdlet [Get-AzAutomationJob](/powershell/module/az.automation/get-azautomationjob) pour vérifier l’état de la tâche d’un runbook enfant s’il lui reste des opérations après que l’enfant a terminé.

## <a name="next-steps"></a>Étapes suivantes

* Pour commencer à utiliser un runbook PowerShell, voir [Tutoriel : Créer un runbook PowerShell](learn/automation-tutorial-runbook-textual-powershell.md).
* Pour savoir comment utiliser des runbooks, voir [Gérer les runbooks dans Azure Automation](manage-runbooks.md).
* Pour plus d’informations sur PowerShell, consultez la [documentation PowerShell](/powershell/scripting/overview).
* Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation#automation).