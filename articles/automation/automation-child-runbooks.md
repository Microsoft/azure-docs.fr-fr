---
title: Créer des runbooks modulaires dans Azure Automation
description: Cet article explique comment créer un runbook appelé par un autre runbook.
services: automation
ms.subservice: process-automation
ms.date: 01/17/2019
ms.topic: conceptual
ms.openlocfilehash: 95e156c17b723c679772293401c730cbdff2220b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86169882"
---
# <a name="create-modular-runbooks"></a>Créer des runbooks modulaires

Dans Azure Automation, il est recommandé d’écrire des runbooks réutilisables et modulaires avec une fonction discrète qui est appelée par d’autres runbooks. Un runbook parent appelle souvent un ou plusieurs runbooks enfants pour exécuter la fonctionnalité requise. 

Il existe deux méthodes pour appeler un runbook enfant. Vous devez comprendre leurs spécificités afin de déterminer celle qui convient le mieux à vos scénarios. Le tableau suivant résume les différences entre les deux façons d’appeler un runbook à partir d’un autre runbook.

|  | En ligne | Applet de commande |
|:--- |:--- |:--- |
| **Travail** |Les Runbooks enfants s’exécutent dans la même tâche que le parent. |Une tâche distincte est créée pour le Runbook enfant. |
| **Exécution** |Le Runbook parent attend la fin de l’exécution du Runbook enfant avant de se poursuivre. |Le runbook parent continue immédiatement après le démarrage du runbook enfant *ou* le runbook parent attend que la tâche enfant se termine. |
| **Sortie** |Le Runbook parent peut obtenir directement la sortie du Runbook enfant. |Le runbook parent doit récupérer la sortie à partir de la tâche du runbook enfant *ou* le runbook parent peut obtenir directement la sortie du runbook enfant. |
| **Paramètres** |Les valeurs des paramètres du Runbook enfant sont spécifiées séparément et peuvent utiliser n’importe quel type de données. |Les valeurs des paramètres du runbook enfant doivent être combinées dans une table de hachage unique. Cette table de hachage peut uniquement inclure des types de données simples, tableau et objet qui utilisent la sérialisation JSON. |
| **Compte Automation** |Le runbook parent peut utiliser uniquement un runbook enfant du même compte Automation. |Les runbooks parents peuvent utiliser un runbook enfant de n’importe quel compte Automation, du même abonnement Azure ou d’un autre abonnement auquel vous avez accès. |
| **Publication** |Le Runbook enfant doit être publié avant la publication du Runbook parent. |Le runbook enfant est publié à tout moment avant le démarrage du runbook parent. |

## <a name="invoke-a-child-runbook-using-inline-execution"></a>Appeler un Runbook enfant à l’aide de l’exécution incluse

Pour appeler un runbook inlined à partir d’un autre runbook, utilisez le nom du runbook et définissez des valeurs pour ses paramètres de la même façon qu’avec une activité ou une applet de commande. Tous les Runbooks d’un même compte Automation peuvent être utilisés ainsi par les autres Runbooks. Le runbook parent attend la fin de l’exécution du runbook enfant avant de passer à la ligne suivante, et toute sortie est retournée directement au parent.

Lorsque vous appelez un Runbook en ligne, il est exécuté dans la même tâche que le Runbook parent. L’historique des travaux du runbook enfant ne contient aucune indication. Toutes les exceptions et sorties de flux issues du runbook enfant sont associées au parent. Ce comportement réduit le nombre de tâches et facilite leur suivi et la résolution des problèmes.

Lorsqu’un Runbook est publié, les Runbooks enfants qu’il appelle doivent déjà être publiés. En effet, Azure Automation crée une association avec tous les runbooks enfants lorsqu’il compile un runbook. Si les runbooks enfants n’ont pas encore été publiés, le runbook parent semble être publié correctement, mais génère une exception lorsqu’il est démarré. Dans ce cas, vous pouvez republier le runbook parent pour référencer correctement les runbooks enfants. Il est inutile de republier le runbook parent si des runbooks enfants sont modifiés, car l’association a déjà été créée.

Les paramètres d’un runbook enfant appelé inlined peuvent être de n’importe quel type de données, y compris des objets complexes. Il n’y a pas de [sérialisation JSON](start-runbooks.md#work-with-runbook-parameters) comme c’est le cas quand vous démarrez le runbook à partir du Portail Azure ou à l’aide de la cmdlet [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook).

### <a name="runbook-types"></a>Types de runbook

Quels types de runbook peuvent s’appeler mutuellement ?

* Un [runbook PowerShell](automation-runbook-types.md#powershell-runbooks) et un [runbook graphique](automation-runbook-types.md#graphical-runbooks) peuvent s’appeler mutuellement inlined, car ils se basent tous les deux sur PowerShell.
* Un [runbook PowerShell Workflow](automation-runbook-types.md#powershell-workflow-runbooks) et un runbook PowerShell Workflow graphique peuvent s’appeler mutuellement inlined, car ils se basent tous les deux sur PowerShell Workflow.
* Les types PowerShell et Workflow PowerShell ne peuvent pas s'appeler mutuellement inlined et doivent utiliser `Start-AzAutomationRunbook`.

Quand l’ordre de publication est-il important ?

L’ordre de publication des runbooks importe uniquement pour les runbooks PowerShell Workflow et les runbooks PowerShell Workflow graphiques.

Lorsque votre runbook appelle un runbook enfant graphique ou PowerShell Workflow avec une exécution inlined, il utilise le nom du runbook. Ce nom doit commencer par `.\\` pour spécifier que le script se trouve dans le répertoire local.

### <a name="example"></a>Exemple

L’exemple suivant démarre un runbook enfant de test qui accepte un objet complexe, une valeur entière et une valeur booléenne. La sortie du Runbook enfant est affectée à une variable. Dans ce cas, le runbook enfant est un runbook de workflow PowerShell.

```azurepowershell-interactive
$vm = Get-AzVM –ResourceGroupName "LabRG" –Name "MyVM"
$output = PSWF-ChildRunbook –VM $vm –RepeatCount 2 –Restart $true
```

Voici le même exemple utilisant un runbook PowerShell en tant qu’enfant.

```azurepowershell-interactive
$vm = Get-AzVM –ResourceGroupName "LabRG" –Name "MyVM"
$output = .\PS-ChildRunbook.ps1 –VM $vm –RepeatCount 2 –Restart $true
```

## <a name="start-a-child-runbook-using-a-cmdlet"></a>Démarrer un runbook enfant à l’aide d’une cmdlet

> [!IMPORTANT]
> Si votre runbook appelle un runbook enfant à l'aide de la cmdlet `Start-AzAutomationRunbook` avec le paramètre `Wait` et que le runbook enfant produit un objet comme résultat, l'opération peut rencontrer une erreur. Pour contourner l’erreur, consultez [Runbooks enfants avec une sortie objet](troubleshoot/runbooks.md#child-runbook-object) afin de savoir comment implémenter la logique pour interroger les résultats à l’aide de la cmdlet [Get-AzAutomationJobOutputRecord](/powershell/module/az.automation/get-azautomationjoboutputrecord).

Vous pouvez utiliser `Start-AzAutomationRunbook` pour démarrer un runbook, comme décrit dans [Démarrage d'un runbook avec Windows PowerShell](start-runbooks.md#start-a-runbook-with-powershell). Il existe deux modes d’utilisation pour cette applet de commande. Dans un mode, l’applet de commande retourne l’ID de la tâche quand la tâche est créée pour le runbook enfant. Dans l’autre mode, que votre script active en spécifiant le paramètre *Wait*, la cmdlet attend que la tâche enfant se termine et retourne la sortie du runbook enfant.

Le tâche d’un runbook enfant démarrée avec une applet de commande s’exécute séparément de celle du runbook parent. Ce comportement génère davantage de tâches que le démarrage de runbook inlined et rend leur suivi plus complexe. Le parent peut démarrer plusieurs runbooks enfants de façon asynchrone, sans attendre la fin de leur exécution. Pour cette exécution parallèle qui appelle des runbooks enfants inlined, le runbook parent doit utiliser le [mot clé parallèle](automation-powershell-workflow.md#use-parallel-processing).

La sortie des runbooks enfants n’est pas retournée au runbook parent de manière fiable en raison du timing. En outre, les variables comme `$VerbosePreference` et `$WarningPreference`, entre autres, peuvent ne pas se propager aux runbooks enfants. Pour éviter ces problèmes, vous pouvez démarrer les runbooks enfants en tant que travaux Automation distincts à l'aide de `Start-AzAutomationRunbook` avec le paramètre `Wait`. Cette technique bloque le runbook parent jusqu’à ce que l’exécution du runbook enfant soit terminée.

Si vous ne voulez pas que le runbook parent se bloque durant l'attente, vous pouvez démarrer le runbook enfant à l'aide de `Start-AzAutomationRunbook` sans le paramètre `Wait`. Dans ce cas, votre runbook doit utiliser [Get-AzAutomationJob](/powershell/module/az.automation/get-azautomationjob) pour attendre la fin de la tâche. Il doit également utiliser [Get-AzAutomationJobOutput](/powershell/module/az.automation/get-azautomationjoboutput) et [Get-AzAutomationJobOutputRecord](/powershell/module/az.automation/get-azautomationjoboutputrecord) pour récupérer les résultats.

Les paramètres d’un runbook enfant démarré avec une applet de commande sont fournis sous forme de table de hachage, comme décrit dans [Paramètres de runbook](start-runbooks.md#work-with-runbook-parameters). Seuls les types de données simples peuvent être utilisés. Si le Runbook possède un paramètre avec un type de données complexe, il doit être appelé en ligne.

Le contexte d’abonnement peut être perdu quand vous démarrez les runbooks enfants comme des travaux distincts. Pour que le runbook enfant exécute des applets de commande de modules Az sur un abonnement Azure spécifique, il doit s’authentifier dans cet abonnement indépendamment du runbook parent.

Si des tâches au sein du même compte Automation utilisent plusieurs abonnements, la sélection d’un abonnement dans une tâche peut changer le contexte d’abonnement actuellement sélectionné pour d’autres tâches. Pour éviter cette situation, utilisez `Disable-AzContextAutosave –Scope Process` au début de chaque runbook. Cette action enregistre le contexte uniquement pour l’exécution de ce runbook.

### <a name="example"></a>Exemple

Dans l'exemple suivant, un runbook enfant avec paramètres est démarré et exécuté à l'aide de la cmdlet `Start-AzAutomationRunbook` avec le paramètre `Wait`. Une fois l’exécution terminée, l’exemple collecte la sortie de la cmdlet issue du runbook enfant. Pour utiliser `Start-AzAutomationRunbook`, le script doit s'authentifier auprès de votre abonnement Azure.

```azurepowershell-interactive
# Ensure that the runbook does not inherit an AzContext
Disable-AzContextAutosave –Scope Process

# Connect to Azure with Run As account
$ServicePrincipalConnection = Get-AutomationConnection -Name 'AzureRunAsConnection'

Connect-AzAccount `
    -ServicePrincipal `
    -Tenant $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint

$AzureContext = Get-AzSubscription -SubscriptionId $ServicePrincipalConnection.SubscriptionID

$params = @{"VMName"="MyVM";"RepeatCount"=2;"Restart"=$true}

Start-AzAutomationRunbook `
    –AutomationAccountName 'MyAutomationAccount' `
    –Name 'Test-ChildRunbook' `
    -ResourceGroupName 'LabRG' `
    -AzContext $AzureContext `
    –Parameters $params –Wait
```

## <a name="next-steps"></a>Étapes suivantes

* Pour exécuter votre runbook, consultez [Démarrer un runbook dans Azure Automation](start-runbooks.md).
* Pour surveiller l’opération runbook, consultez [Sortie et messages de Runbook dans Azure Automation](automation-runbook-output-and-messages.md).