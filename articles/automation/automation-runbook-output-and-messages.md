---
title: Configurer les flux de sortie et de messages d’un runbook
description: Cet article explique comment implémenter une logique de gestion des erreurs et décrit les flux de sortie et de messages dans les runbooks Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 11/03/2020
ms.topic: conceptual
ms.openlocfilehash: beed3ec50d0c7990168ee75976c732796cdbe246
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93324426"
---
# <a name="configure-runbook-output-and-message-streams"></a>Configurer les flux de sortie et de messages d’un runbook

La plupart des Runbooks Azure Automation ont une certaine forme de sortie. Cette sortie peut être un message d’erreur destiné à l’utilisateur ou un objet complexe que vous souhaitez utiliser avec un autre Runbook. Windows PowerShell fournit [plusieurs flux](/powershell/module/microsoft.powershell.core/about/about_redirection) pour l’envoi d’une sortie à partir d’un script ou d’un flux de travail. Azure Automation fonctionne différemment avec chaque flux. Vous devez suivre les meilleures pratiques d’utilisation des flux lorsque vous créez un Runbook.

Le tableau suivant décrit brièvement chaque flux avec son comportement dans le portail Azure pour les Runbooks publiés et pendant le [test d’un Runbook](./manage-runbooks.md). Le flux de sortie est le flux principal utilisé pour la communication entre les Runbooks. Les autres flux sont classés comme des flux de messages destinés à la communication des informations à l’utilisateur.

| STREAM | Description | Publié | Test |
|:--- |:--- |:--- |:--- |
| Error |Message d’erreur destiné à l’utilisateur. Contrairement à une exception, le Runbook se poursuit après un message d’erreur par défaut. |Consignation dans l’historique des tâches |Affichage dans le volet de sortie du test |
| Débogage |Messages destinés à un utilisateur interactif. Utilisation proscrite dans les Runbooks. |Pas de consignation dans l’historique des tâches |Pas d’affichage dans le volet de sortie du test |
| Output |Objets destinés à être consommés par d’autres Runbooks. |Consignation dans l’historique des tâches |Affichage dans le volet de sortie du test |
| Progress |Informations générées automatiquement avant et après chaque activité dans le Runbook. Le Runbook ne doit pas tenter de créer ses propres informations de progression dans la mesure où elles sont destinées à un utilisateur interactif. |Consignation dans l’historique des tâches uniquement si l’enregistrement de la progression est activé pour le Runbook |Pas d’affichage dans le volet de sortie du test |
| Commentaires |Messages fournissant des informations générales ou de débogage. |Consignation dans l’historique des tâches uniquement si la journalisation documentée est activée pour le Runbook |Affichage dans le volet de sortie du test uniquement si la variable `VerbosePreference` est définie sur Continue dans le Runbook |
| Avertissement |Message d’avertissement destiné à l’utilisateur. |Consignation dans l’historique des tâches |Affichage dans le volet de sortie du test |

## <a name="use-the-output-stream"></a>Utiliser le flux de sortie

Le flux de sortie est destiné à la sortie des objets créés par un script ou un workflow lorsqu’il s’exécute correctement. Azure Automation utilise principalement ce flux pour les objets destinés à être consommés par les Runbooks parents qui appellent le [Runbook actuel](automation-child-runbooks.md). Lorsqu’un parent [appelle un Runbook en ligne](automation-child-runbooks.md#invoke-a-child-runbook-using-inline-execution), l’enfant retourne les données au parent à partir du flux de sortie.

Votre Runbook utilise le flux de sortie pour communiquer des informations générales à l’utilisateur uniquement si le Runbook n’est jamais appelé par un autre Runbook. Cependant, vos Runbooks devraient utiliser le [flux des commentaires](#write-output-to-verbose-stream) pour communiquer des informations générales à l’utilisateur.

Configurez vos Runbooks pour qu’ils consignent les données dans le flux de sortie à l’aide de [Write-Output](/powershell/module/microsoft.powershell.utility/write-output). Vous pouvez également placer l’objet sur sa propre ligne dans le script.

```powershell
#The following lines both write an object to the output stream.
Write-Output –InputObject $object
$object
```

### <a name="handle-output-from-a-function"></a>Gérer la sortie à partir d’une fonction

Quand une fonction Runbook écrit dans le flux de sortie, la sortie est retournée au Runbook. Si le Runbook affecte cette sortie à une variable, la sortie n’est pas écrite dans le flux de sortie. Toute écriture dans d’autres flux à partir de la fonction se fait dans le flux correspondant du Runbook. Considérez l’exemple de Runbook PowerShell Workflow suivant.

```powershell
Workflow Test-Runbook
{
  Write-Verbose "Verbose outside of function" -Verbose
  Write-Output "Output outside of function"
  $functionOutput = Test-Function
  $functionOutput

  Function Test-Function
  {
    Write-Verbose "Verbose inside of function" -Verbose
    Write-Output "Output inside of function"
  }
}
```

Le flux de sortie pour la tâche du Runbook est :

```output
Output inside of function
Output outside of function
```

Le flux des commentaires pour la tâche du Runbook est :

```output
Verbose outside of function
Verbose inside of function
```

Une fois que vous avez publié le Runbook et avant de démarrer, vous devez activer la journalisation détaillée dans les paramètres du Runbook pour obtenir la sortie de flux détaillé.

### <a name="declare-output-data-type"></a>Déclarer le type de données de sortie

Voici des exemples de types de données de sortie :

* `System.String`
* `System.Int32`
* `System.Collections.Hashtable`
* `Microsoft.Azure.Commands.Compute.Models.PSVirtualMachine`

#### <a name="declare-output-data-type-in-a-workflow"></a>Déclarer le type de données de sortie dans un workflow

Un flux de travail spécifie le type de données de sa sortie à l’aide de l’ [attribut OutputType](/powershell/module/microsoft.powershell.core/about/about_functions_outputtypeattribute). Cet attribut n’a aucun effet lors de l’exécution, mais il vous fournit une indication sur la sortie attendue du Runbook au moment de la conception. Alors que l’ensemble d’outils dédiés aux Runbooks continue d’évoluer, la déclaration des types de données de sortie au moment de la conception augmente. Par conséquent, il est recommandé d’inclure cette déclaration dans tous les Runbooks que vous créez.

L’exemple de Runbook suivant génère un objet String et inclut une déclaration de son type de sortie. Si votre Runbook génère un tableau d’un certain type, vous devez toujours spécifier le type, par opposition à un tableau du type.

```powershell
Workflow Test-Runbook
{
  [OutputType([string])]

  $output = "This is some string output."
  Write-Output $output
}
 ```

#### <a name="declare-output-data-type-in-a-graphical-runbook"></a>Déclarer le type de données de sortie dans un Runbook graphique

Pour déclarer un type de sortie dans un Runbook graphique ou PowerShell Workflow graphique, vous pouvez sélectionner l’option de menu **Entrée et sortie** puis entrer le type de sortie. Il est recommandé d’utiliser le nom de classe .NET complet pour le rendre facilement identifiable lorsqu’un Runbook parent y fait référence. L’utilisation d’un nom complet expose toutes les propriétés de la classe au bus de données du Runbook et offre plus de flexibilité lorsque ces propriétés sont utilisées pour la logique conditionnelle, la journalisation et le référencement en tant que valeurs pour d’autres activités du Runbook.<br> ![Option Entrée et sortie de Runbook](media/automation-runbook-output-and-messages/runbook-menu-input-and-output-option.png)

>[!NOTE]
>Une fois que vous avez entré une valeur dans le champ **Type de sortie** dans le volet des propriétés d’entrée et de sortie, veillez à cliquer à l’extérieur du contrôle afin que ce dernier puisse reconnaître votre entrée.

L’exemple suivant montre deux Runbooks graphiques illustrant la fonctionnalité Entrée et sortie. Si le modèle de conception Runbook modulaire est appliqué, l’un des Runbooks sert de modèle Runbook d’authentification et gère l’authentification auprès d’Azure à l’aide du compte d’identification. Le deuxième Runbook, qui exécute normalement la logique principale pour automatiser un scénario donné, exécute dans ce cas le modèle Runbook d’authentification. Il affiche les résultats dans le volet de sortie du test. Dans des circonstances normales, ce Runbook effectue ses activités par rapport à une ressource utilisant la sortie du Runbook enfant.

Voici la logique de base du Runbook **AuthenticateTo-Azure**.<br> ![Exemple de modèle Runbook d’authentification](media/automation-runbook-output-and-messages/runbook-authentication-template.png).

Le Runbook inclut le type de sortie `Microsoft.Azure.Commands.Profile.Models.PSAzureContext`, qui retourne les propriétés du profil d’authentification.<br> ![Exemple de type de sortie Runbook](media/automation-runbook-output-and-messages/runbook-input-and-output-add-blade.png)

Ce Runbook est simple, mais l’un des éléments de configuration nécessite une attention particulière. La dernière activité exécute l’applet de commande `Write-Output` pour écrire des données de profil dans une variable à l’aide d’une expression PowerShell pour le paramètre `Inputobject`. Ce paramètre est obligatoire pour `Write-Output`.

Le deuxième Runbook de cet exemple, nommé **Test-ChildOutputType** , définit simplement deux activités.<br> ![Exemple de type de sortie enfant Runbook](media/automation-runbook-output-and-messages/runbook-display-authentication-results-example.png)

La première activité appelle le Runbook **AuthenticateTo-Azure**. La deuxième activité exécute l’applet de commande `Write-Verbose` avec l’option **Source de données** définie sur **Sortie d'activité**. De plus, l’option **Chemin de champ** est définie sur **Context.Subscription.SubscriptionName** , la sortie du contexte du Runbook **AuthenticateTo-Azure**.<br> ![Source de données du paramètre d’applet de commande Write-Verbose](media/automation-runbook-output-and-messages/runbook-write-verbose-parameters-config.png)

Le résultat obtenu est le nom de l’abonnement.<br> ![Résultats du Runbook Test-ChildOutputType](media/automation-runbook-output-and-messages/runbook-test-childoutputtype-results.png)

## <a name="working-with-message-streams"></a>Utilisation de flux de messages

Contrairement au flux de sortie, les flux de messages communiquent des informations à l’utilisateur. Il existe plusieurs flux de messages pour les différents types d’informations, et Azure Automation traite chaque flux différemment.

### <a name="write-output-to-warning-and-error-streams"></a>Écrire la sortie dans des flux d’avertissements et d’erreurs

Les flux d’erreurs et d’avertissements consignent les problèmes qui se produisent dans un Runbook. Azure Automation écrit ces flux dans l’historique des travaux lors de l’exécution d’un Runbook. Automation intègre les flux dans le volet Sortie de test du portail Azure lors du test d’un Runbook.

Par défaut, un Runbook continue de s’exécuter après un avertissement ou une erreur. Vous pouvez spécifier l’interruption de votre Runbook en cas d’avertissement ou d’erreur en définissant une [variable de préférence](#work-with-preference-variables) avant de créer le message. Par exemple, pour déclencher l’interruption du Runbook en cas d’erreur, comme dans le cas d’une exception, affectez la valeur Stop à la variable `ErrorActionPreference`.

Créez un message d’avertissement ou un message d’erreur à l’aide de l’applet de commande [Write-Warning](/powershell/module/microsoft.powershell.utility/write-warning) ou [Write-Error](/powershell/module/microsoft.powershell.utility/write-error). Les activités peuvent également écrire dans les flux d’avertissement et d’erreur.

```powershell
#The following lines create a warning message and then an error message that will suspend the runbook.

$ErrorActionPreference = "Stop"
Write-Warning –Message "This is a warning message."
Write-Error –Message "This is an error message that will stop the runbook because of the preference variable."
```

### <a name="write-output-to-debug-stream"></a>Écrire la sortie dans un flux de débogage

Azure Automation utilise le flux de message de débogage pour les utilisateurs interactifs. Par défaut, Azure Automation ne capture pas les données de flux de débogage. Seules les données de sortie, d’erreur et d’avertissement sont capturées, ainsi que les données détaillées si le runbook est configuré pour les capturer.

Pour capturer les données de flux de débogage, vous devez effectuer deux actions dans vos runbooks :

1. Définissez la variable `$GLOBAL:DebugPreference="Continue"`, qui indique à PowerShell de continuer chaque fois qu’un message de débogage est rencontré.  La partie **$GLOBAL:** indique à PowerShell qu’il doit effectuer cette opération dans l’étendue globale plutôt que dans l’étendue locale où se trouve le script au moment de l’exécution de l’instruction.

1. Redirigez le flux de débogage que nous ne capturons pas vers un flux que nous capturons, par exemple un flux de *sortie*. Pour ce faire, définissez la redirection PowerShell sur l’instruction à exécuter. Pour plus d’informations sur la redirection PowerShell, consultez [À propos de la redirection](/powershell/module/microsoft.powershell.core/about/about_redirection).

#### <a name="examples"></a>Exemples

Dans cet exemple, le runbook est configuré à l’aide des applets de commande `Write-Output` et `Write-Debug` dans le but de générer deux flux différents.

```powershell
Write-Output "This is an output message." 
Write-Debug "This is a debug message."
```

Si ce runbook devait être exécuté tel quel, le volet de sortie de la tâche de runbook diffuserait la sortie suivante :

```output
This is an output message.
```

Dans cet exemple, le runbook est configuré de façon similaire à l’exemple précédent, sauf que l’instruction `$GLOBAL:DebugPreference="Continue"` est incluse avec l’ajout de `5>&1` à la fin de l’instruction `Write-Debug`.

```powershell
Write-Output "This is an output message." 
$GLOBAL:DebugPreference="Continue" 
Write-Debug "This is a debug message." 5>&1
```

Si ce runbook devait être exécuté, le volet de sortie de la tâche de runbook diffuserait la sortie suivante :

```output
This is an output message.
This is a debug message.
```

Cela est dû au fait que l’instruction `$GLOBAL:DebugPreference="Continue"` indique à PowerShell d’afficher les messages de débogage, et l’ajout de `5>&1` à la fin de l’instruction `Write-Debug` indique à PowerShell de rediriger le flux 5 (débogage) vers le flux 1 (sortie).

### <a name="write-output-to-verbose-stream"></a>Écrire la sortie dans le flux des commentaires

Le flux des commentaires prend en charge les informations générales sur le fonctionnement du Runbook. Dans la mesure où le flux de débogage n’est pas disponible dans un Runbook, votre Runbook devrait utiliser des messages documentés pour les informations de débogage.

Par défaut, l’historique des travaux ne consigne pas les messages documentés provenant de Runbooks publiés, pour des questions de performances. Pour stocker les messages documentés, utilisez l’onglet **Configurer** du portail Azure en définissant le paramètre **Journaliser les enregistrements détaillés** pour configurer vos Runbooks publiés afin de consigner les messages documentés. Activez cette option uniquement pour résoudre les problèmes liés à un Runbook ou déboguer un Runbook. Dans la plupart des cas, vous devriez conserver le paramètre par défaut (les informations de commentaires ne sont pas consignées).

Lors du [test d’un Runbook](./manage-runbooks.md), les messages documentés ne s’affichent pas, même si le Runbook est configuré pour enregistrer les informations de commentaires. Pour afficher les messages documentés lors du [test d’un Runbook](./manage-runbooks.md), vous devez affecter la valeur Continue à la variable `VerbosePreference`. Dans ce cas, les messages documentés sont affichés dans le volet de sortie du test dans le portail Azure.

Le code suivant crée un message documenté à l’aide de l’applet de commande [Write-Verbose](/powershell/module/microsoft.powershell.utility/write-verbose).

```powershell
#The following line creates a verbose message.

Write-Verbose –Message "This is a verbose message."
```

## <a name="handle-progress-records"></a>Gérer les informations de progression

Vous pouvez utiliser l’onglet **Configurer** du portail Azure pour configurer un Runbook afin de journaliser les informations de progression. Avec le paramètre par défaut, les enregistrements ne sont pas journalisés, afin d’optimiser les performances. Dans la plupart des cas, vous devriez conserver le paramètre par défaut. Activez cette option uniquement pour résoudre les problèmes liés à un Runbook ou déboguer un Runbook.

Si vous activez la journalisation des enregistrements de progression, votre Runbook consigne un enregistrement dans l’historique des travaux avant et après l’exécution de chaque activité. Le test d’un Runbook n’affiche pas les messages de progression, même si le Runbook est configuré pour consigner les informations de progression.

>[!NOTE]
>La cmdlet [Write-Progress](/powershell/module/microsoft.powershell.utility/write-progress) n’est pas valide dans un Runbook, dans la mesure où cette cmdlet est destinée à un utilisateur interactif.

## <a name="work-with-preference-variables"></a>Utiliser les variables de préférence

Vous pouvez définir certaines [variables de préférence](/powershell/module/microsoft.powershell.core/about/about_preference_variables) Windows PowerShell dans vos Runbooks pour contrôler la réponse aux données envoyées aux différents flux de sortie. Le tableau suivant répertorie les variables de préférence qui peuvent être utilisées dans des Runbooks, ainsi que leurs valeurs par défaut et leurs valeurs valides. D’autres valeurs sont disponibles pour les variables de préférence lorsqu’elles sont utilisées dans Windows PowerShell en dehors d’Azure Automation.

| Variable | Valeur par défaut | Valeurs valides |
|:--- |:--- |:--- |
| `WarningPreference` |Continue |Arrêter<br>Continue<br>SilentlyContinue |
| `ErrorActionPreference` |Continue |Arrêter<br>Continue<br>SilentlyContinue |
| `VerbosePreference` |SilentlyContinue |Arrêter<br>Continue<br>SilentlyContinue |

Le tableau suivant indique le comportement associé aux valeurs des variables de préférence valides dans les Runbooks.

| Valeur | Comportement |
|:--- |:--- |
| Continue |Enregistre le message et poursuit l’exécution du Runbook. |
| SilentlyContinue |Poursuit l’exécution du Runbook sans enregistrer le message. Le message est alors ignoré. |
| Arrêter |Enregistre le message et interrompt le Runbook. |

## <a name="retrieve-runbook-output-and-messages"></a><a name="runbook-output"></a>Récupérer la sortie et les messages d’un runbook

### <a name="retrieve-runbook-output-and-messages-in-azure-portal"></a>Récupérer la sortie et les messages d’un Runbook dans Azure Automation

Vous pouvez afficher les détails d’une tâche de Runbook dans le portail Azure à partir de l’onglet **Tâches** du Runbook. Le résumé de la tâche affiche les paramètres d’entrée et le [flux de sortie](#use-the-output-stream), en plus des informations générales sur le travail et les éventuelles exceptions. L’historique des travaux inclut les messages du flux de sortie ainsi que des [flux d’avertissements et des flux d’erreurs](#write-output-to-warning-and-error-streams). Il inclut également les messages du [flux de commentaires](#write-output-to-verbose-stream) et les [informations de progression](#handle-progress-records) si le Runbook est configuré pour journaliser les enregistrements détaillés et les informations de progression.

### <a name="retrieve-runbook-output-and-messages-in-windows-powershell"></a>Récupérer la sortie et les messages d’un Runbook dans Windows PowerShell

Dans Windows PowerShell, vous pouvez récupérer la sortie et les messages d’un Runbook à l’aide de l’applet de commande [Get-AzAutomationJobOutput](/powershell/module/Az.Automation/Get-AzAutomationJobOutput). Cette applet de commande requiert l’ID de la tâche et inclut un paramètre appelé `Stream` qui vous permet de spécifier le flux à récupérer. Vous pouvez spécifier une valeur Any (Quelconque) pour ce paramètre afin de récupérer tous les flux du travail.

Dans l’exemple suivant, un exemple de Runbook est démarré et exécuté. Une fois l’exécution du Runbook terminée, le script collecte le flux de sortie du Runbook à partir de la tâche.

```powershell
$job = Start-AzAutomationRunbook -ResourceGroupName "ResourceGroup01" `
  –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook"

$doLoop = $true
While ($doLoop) {
  $job = Get-AzAutomationJob -ResourceGroupName "ResourceGroup01" `
    –AutomationAccountName "MyAutomationAccount" -Id $job.JobId
  $status = $job.Status
  $doLoop = (($status -ne "Completed") -and ($status -ne "Failed") -and ($status -ne "Suspended") -and ($status -ne "Stopped"))
}

Get-AzAutomationJobOutput -ResourceGroupName "ResourceGroup01" `
  –AutomationAccountName "MyAutomationAccount" -Id $job.JobId –Stream Output

# For more detailed job output, pipe the output of Get-AzAutomationJobOutput to Get-AzAutomationJobOutputRecord
Get-AzAutomationJobOutput -ResourceGroupName "ResourceGroup01" `
  –AutomationAccountName "MyAutomationAccount" -Id $job.JobId –Stream Any | Get-AzAutomationJobOutputRecord
```

### <a name="retrieve-runbook-output-and-messages-in-graphical-runbooks"></a>Récupérer la sortie et les messages du Runbook dans des Runbooks graphiques

Pour les Runbooks graphiques, une journalisation supplémentaire de la sortie et des messages est disponible sous la forme de suivi au niveau de l’activité. Il existe deux niveaux de suivi : de base et détaillé. Le suivi de base affiche l’heure de début et de fin de chaque activité dans le Runbook, ainsi que des informations relatives à toute nouvelle tentative de l’activité. Par exemple, le nombre de tentatives et l’heure de début de l’activité. Le suivi détaillé inclut les fonctionnalités de suivi de base ainsi que des données d’entrée et de sortie pour chaque activité. 

Actuellement, le suivi au niveau de l’activité consigne des enregistrements à l’aide du flux des commentaires. Vous devez donc activer la journalisation détaillée lorsque vous activez le suivi. Pour les runbooks graphiques avec le suivi activé, il est inutile de journaliser les informations de progression. Le suivi de base a le même objectif et fournit des informations plus détaillées.

![Vue du flux de travail de création graphique](media/automation-runbook-output-and-messages/job-streams-view-blade.png)

Cette image illustre le fait que lorsque vous activez la journalisation détaillée et le suivi pour des Runbooks graphiques, vous obtenez beaucoup plus d’informations dans la vue **Flux de travail** de production. Ces informations supplémentaires peuvent être essentielles pour résoudre les problèmes de production affectant un Runbook. 

Cependant, à moins que vous n’ayez besoin de ces informations pour suivre la progression d’un Runbook à des fins de dépannage, vous pouvez choisir de désactiver le suivi. Les enregistrements de suivi peuvent être particulièrement nombreux. Avec le suivi de Runbook graphique, vous pouvez obtenir entre deux et quatre enregistrements par activité, selon votre configuration du suivi de base ou du suivi détaillé.

**Pour activer le suivi au niveau de l’activité :**

1. Dans le portail Azure, ouvrez votre compte Automation.
2. Sélectionnez **Runbooks** sous **Automatisation de processus** pour ouvrir la liste des runbooks.
3. Dans la page Runbooks, sélectionnez un Runbook graphique dans votre liste de Runbooks.
4. Sous **paramètres** , cliquez sur **journalisation et le suivi**.
5. Sur la page Journalisation et suivi, sous **Journaliser les enregistrements détaillés** , cliquez sur **On** (Activé) pour activer la journalisation détaillée.
6. Sous **Suivi au niveau de l’activité** , définissez le niveau de suivi sur **De base** ou **Détaillé** , selon le niveau de suivi dont vous avez besoin.<br>

   ![Page Journalisation et suivi de la création graphique](media/automation-runbook-output-and-messages/logging-and-tracing-settings-blade.png)

### <a name="retrieve-runbook-output-and-messages-in-microsoft-azure-monitor-logs"></a>Récupérer la sortie et les messages de Runbook dans les journaux Microsoft Azure Monitor

Azure Automation peut envoyer un état de tâche du Runbook et des flux de tâches à votre espace de travail Log Analytics. Azure Monitor prend en charge les journaux qui vous permettent d’effectuer les opérations suivantes :

* Obtenir des informations sur vos travaux Automation.
* Déclencher un e-mail ou une alerte en fonction du statut de votre travail de runbook, par exemple, échec ou état suspendu.
* Écrire des requêtes avancées sur plusieurs flux de travaux.
* Mettre en corrélation des travaux sur différents comptes Automation.
* Afficher l'historique des travaux.

Pour plus d’informations sur la configuration de l’intégration aux journaux Azure Monitor dans le but de collecter, de mettre en corrélation et de modifier des données de travail, consultez [Transférer l’état d’un travail et de flux de travail de journaux Automation vers des journaux Azure Monitor](automation-manage-send-joblogs-log-analytics.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour savoir comment utiliser des runbooks, voir [Gérer les runbooks dans Azure Automation](manage-runbooks.md).
* Si vous n’êtes pas familiarisé avec les scripts PowerShell, consultez la documentation [PowerShell](/powershell/scripting/overview).
* Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation).
