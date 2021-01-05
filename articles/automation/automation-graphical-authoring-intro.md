---
title: Créer des runbooks graphiques dans Azure Automation
description: Cet article explique comment créer un runbook graphique sans utiliser de code.
services: automation
ms.subservice: process-automation
ms.date: 03/16/2018
ms.topic: conceptual
ms.openlocfilehash: 161272fe35ee9ea1e0880b991273e5d1a79eafb4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90987320"
---
# <a name="author-graphical-runbooks-in-azure-automation"></a>Créer des runbooks graphiques dans Azure Automation

Dans Azure Automation, tous les runbooks sont des workflows Windows PowerShell. Les runbooks graphiques et les runbooks de workflow PowerShell graphique génèrent du code PowerShell qui est exécuté par les Workers Automation, mais que vous ne pouvez pas afficher ou modifier. Vous pouvez convertir un runbook graphique en runbook de workflow PowerShell graphique (et vice versa). Toutefois, vous ne pouvez pas convertir ces runbooks en runbook textuel. Par ailleurs, l’éditeur graphique d’Automation ne peut pas importer un runbook textuel.

La création de graphiques vous permet de créer des runbooks pour Azure Automation sans les complexités du code Windows PowerShell ou du code de workflow PowerShell sous-jacent. Vous pouvez ajouter des activités au canevas à partir d’une bibliothèque de cmdlets et de runbooks, puis les lier et les configurer pour former un workflow. Si vous avez déjà travaillé avec System Center Orchestrator ou Service Management Automation (SMA), la création de graphiques vous semblera certainement familière. Cet article propose une introduction aux concepts dont vous avez besoin pour prendre en main la création d’un runbook graphique.

## <a name="overview-of-graphical-editor"></a>Vue d'ensemble de l'éditeur graphique

Vous pouvez ouvrir l’éditeur graphique dans le portail Azure en créant ou en modifiant un Runbook graphique.

![Espace de travail graphique](media/automation-graphical-authoring-intro/runbook-graphical-editor.png)

Les sections suivantes décrivent les contrôles de l'éditeur graphique.

### <a name="canvas-control"></a>Contrôle Canevas

Le contrôle Canevas vous permet de concevoir votre runbook. Vous pouvez ajouter des activités au runbook à partir des nœuds se trouvant dans le contrôle Bibliothèque et les connecter avec des liens pour définir la logique du runbook. En bas du canevas, des commandes vous permettent d’effectuer un zoom avant ou arrière.

### <a name="library-control"></a>Contrôle Bibliothèque

Le contrôle Bibliothèque vous permet de sélectionner les [activités](#use-activities) à ajouter à votre runbook. Vous les ajoutez au canevas, où vous pouvez les relier à d’autres activités. Le contrôle Bibliothèque comprend les sections définies dans le tableau suivant.

| Section | Description |
|:--- |:--- |
| Applets de commande |Toutes les cmdlets qui peuvent être utilisées dans votre runbook. Les applets de commande sont organisées par module. Tous les modules que vous avez installés dans votre compte Automation sont disponibles. |
| Runbooks |Runbooks de votre compte Automation. Vous pouvez ajouter ces runbooks au canevas pour qu’ils soient utilisés en tant que runbooks enfants. Seuls les runbooks dotés du même type de noyau que le runbook en cours de modification sont affichés. Pour les runbooks graphiques, seuls les runbooks basés sur PowerShell sont affichés. Pour les runbooks de workflow PowerShell graphiques, seuls les runbooks basés sur un workflow PowerShell sont affichés. |
| Éléments multimédias |[Ressources Automation](/previous-versions/azure/dn939988(v=azure.100)) de votre compte Automation qui peuvent être utilisées dans votre runbook. L’ajout d’une ressource à un runbook ajoute une activité de workflow qui obtient la ressource sélectionnée. Pour les ressources de type Variable, vous pouvez choisir d'ajouter une activité pour obtenir la variable ou de définir la variable. |
| Contrôle de Runbook |Activités de contrôle qui peuvent être utilisées dans votre runbook actuel. Une activité Jonction prend plusieurs entrées et attend que tout soit terminé pour poursuivre le workflow. Une activité Code exécute une ou plusieurs lignes de code PowerShell ou de workflow PowerShell en fonction du type de runbook graphique. Vous pouvez utiliser cette activité pour du code personnalisé ou pour une fonctionnalité difficile à obtenir avec d’autres activités. |

### <a name="configuration-control"></a>Contrôle Configuration

Le contrôle Configuration vous permet de renseigner les détails concernant un objet sélectionné dans le canevas. Les propriétés disponibles dans ce contrôle dépendent du type d’objet sélectionné. Quand vous choisissez une option dans le contrôle Configuration, d’autres panneaux s’ouvrent pour fournir davantage d’informations.

### <a name="test-control"></a>Contrôle Test

Le contrôle Test n'est pas affiché lors du premier démarrage de l'éditeur graphique. Il s’ouvre lorsque vous testez un runbook graphique de manière interactive.

## <a name="use-activities"></a>Utiliser des activités

Les activités sont les éléments essentiels d'un Runbook. Une activité peut être une cmdlet PowerShell, un runbook enfant ou un workflow. Vous pouvez ajouter une activité au runbook en cliquant avec le bouton droit dans le contrôle Bibliothèque, puis en sélectionnant **Ajouter au canevas**. Vous pouvez ensuite cliquer sur l'activité et la faire glisser pour la placer à l'emplacement de votre choix sur le canevas. L’emplacement de l’activité sur le canevas n’a aucun effet sur le fonctionnement du runbook. Vous pouvez disposer votre runbook de la façon qui vous convient le mieux pour visualiser son fonctionnement.

![Ajouter au canevas](media/automation-graphical-authoring-intro/add-to-canvas-cmdlet.png)

Sélectionnez une activité sur le canevas pour configurer ses propriétés et ses paramètres dans le panneau Configuration. Vous pouvez remplacer l’étiquette de l’activité par un nom descriptif. Le runbook exécute toujours la cmdlet d’origine. Vous modifiez simplement le nom d’affichage utilisé par l’éditeur graphique. Notez que l’étiquette doit être unique dans le runbook.

### <a name="parameter-sets"></a>Jeux de paramètres

Un jeu de paramètres définit les paramètres obligatoires et facultatifs qui acceptent des valeurs pour une applet de commande particulière. Toutes les cmdlets ont au moins un jeu de paramètres ; certaines en ont plusieurs. Si une applet de commande a plusieurs jeux de paramètres, vous devez d’abord sélectionner celui qui doit être utilisé pour pouvoir configurer les paramètres. Vous pouvez changer le jeu de paramètres utilisé par une activité en sélectionnant **Jeu de paramètres** et en choisissant un autre jeu. Dans ce cas, toutes les valeurs de paramètres que vous avez déjà configurées sont perdues.

Dans l’exemple suivant, la cmdlet [Get-AzVM](/powershell/module/az.compute/get-azvm?view=azps-3.5.0&preserve-view=true) contient trois jeux de paramètres. L’exemple utilise un seul jeu de paramètres appelé **ListVirtualMachineInResourceGroupParamSet**, avec un seul paramètre facultatif, pour retourner toutes les machines virtuelles d’un groupe de ressources. L’exemple utilise aussi le jeu de paramètres **GetVirtualMachineInResourceGroupParamSet** pour spécifier la machine virtuelle à retourner. Ce jeu comporte deux paramètres obligatoires et un paramètre facultatif.

![Jeu de paramètres](media/automation-graphical-authoring-intro/get-azvm-parameter-sets.png)

#### <a name="parameter-values"></a>Valeurs de paramètres

Lorsque vous spécifiez une valeur pour un paramètre, vous sélectionnez une source de données pour déterminer la façon dont la valeur est spécifiée. Les sources de données disponibles pour un paramètre particulier dépendent des valeurs valides pour ce paramètre. Par exemple, Null n’est pas une option disponible pour un paramètre qui n'autorise pas les valeurs null.

| source de données | Description |
|:--- |:--- |
| Valeur constante |Entrez une valeur pour le paramètre. Cette source de données est uniquement disponible pour les types de données suivants : Int32, Int64, String, Boolean, DateTime, Switch. |
| Sortie d'activité |Utilisez la sortie d’une activité qui précède l’activité actuelle dans le workflow. Toutes les activités valides sont répertoriées. Pour la valeur de paramètre, utilisez uniquement l’activité qui produit la sortie. Si l’activité génère un objet ayant plusieurs propriétés, vous pouvez taper le nom d’une propriété spécifique après avoir sélectionné l’activité. |
| Entrée de Runbook |Sélectionnez une entrée de runbook comme entrée pour le paramètre de l’activité. |
| Ressource de variable |Sélectionnez une variable Automation comme entrée. |
| Ressource d’informations d’identification |Sélectionnez les informations d’identification Automation comme entrée. |
| Ressource de certificat |Sélectionnez un certificat Automation comme entrée. |
| Ressource de connexion |Sélectionnez une connexion Automation comme entrée. |
| Expression PowerShell |Spécifiez une [expression PowerShell](#work-with-powershell-expressions) simple. L’expression est évaluée avant l’activité, et le résultat est utilisé pour la valeur du paramètre. Vous pouvez utiliser des variables pour faire référence à la sortie d'une activité ou à un paramètre d'entrée de Runbook. |
| Non configuré |Effacez toute valeur qui a été précédemment configurée. |

#### <a name="optional-additional-parameters"></a>Autres paramètres facultatifs

Toutes les applets de commande peuvent fournir des paramètres supplémentaires. Il s’agit de paramètres communs PowerShell ou d’autres paramètres personnalisés. L’éditeur graphique présente une zone de texte dans laquelle vous pouvez fournir des paramètres en utilisant la syntaxe PowerShell s’affiche. Par exemple, pour utiliser le paramètre commun `Verbose`, vous devez spécifier `-Verbose:$True`.

### <a name="retry-activity"></a>Nouvelles tentatives d’activité

La fonctionnalité Nouvelles tentatives permet à une activité de s’exécuter plusieurs fois jusqu’à ce qu’une condition particulière soit remplie, à l’instar d’une boucle. Vous pouvez utiliser cette fonctionnalité pour les activités qui doivent s’exécuter plusieurs fois, sont sujettes à des erreurs, peuvent nécessiter plusieurs tentatives avant d’aboutir ou testent les informations de sortie de l’activité pour vérifier la validité des données.

Lorsque vous activez les nouvelles tentatives pour une activité, vous pouvez définir un délai et une condition. Le délai correspond au temps (exprimé en secondes ou minutes) d’attente du Runbook avant la nouvelle exécution de l’activité. Si vous ne spécifiez pas de délai, l’activité s’exécute à nouveau immédiatement après son achèvement.

:::image type="content" source="media/automation-graphical-authoring-intro/retry-delay.png" alt-text="Capture d’écran des paramètres de la fonctionnalité Activer les nouvelles tentatives.":::

La condition de nouvelle tentative est une expression PowerShell qui est évaluée après chaque exécution de l’activité. Si l’expression correspond à True, l’activité s’exécute à nouveau. Si l’expression correspond à False, l’activité n’est pas réexécutée et le runbook passe à l’activité suivante.

:::image type="content" source="media/automation-graphical-authoring-intro/retry-condition.png" alt-text="Capture d’écran des paramètres de la fonctionnalité Activer les nouvelles tentatives.":::

La condition de nouvelle tentative peut utiliser une variable nommée `RetryData` qui donne accès aux informations sur les nouvelles tentatives d’activité. Cette variable possède les propriétés indiquées dans le tableau suivant :

| Propriété | Description |
|:--- |:--- |
| `NumberOfAttempts` |Nombre d’exécutions de l’activité. |
| `Output` |Sortie de la dernière exécution de l’activité. |
| `TotalDuration` |Délai écoulé depuis la première exécution de l’activité. |
| `StartedAt` |Heure (au format UTC) à laquelle l’activité a été démarrée pour la première fois. |

Voici des exemples de conditions de nouvelles tentatives d’activité.

```powershell-interactive
# Run the activity exactly 10 times.
$RetryData.NumberOfAttempts -ge 10
```

```powershell-interactive
# Run the activity repeatedly until it produces any output.
$RetryData.Output.Count -ge 1
```

```powershell-interactive
# Run the activity repeatedly until 2 minutes has elapsed.
$RetryData.TotalDuration.TotalMinutes -ge 2
```

Après avoir configuré une condition de nouvelle tentative pour une activité, l’activité émet un rappel à l’aide de deux signaux visuels. Le premier est présenté dans l’activité et le second apparaît lorsque vous passez en revue la configuration de l’activité.

![Indicateurs visuels de nouvelle tentative d’activité](media/automation-graphical-authoring-intro/runbook-activity-retry-visual-cue.png)

### <a name="workflow-script-control"></a>Contrôle Script de workflow

Un contrôle de script de workflow est une activité spéciale qui accepte un script PowerShell ou un script de workflow PowerShell, selon le type de runbook graphique qui est créé. Ce contrôle fournit des fonctionnalités qui, sans cela, ne seraient peut-être pas disponibles. Il n'accepte pas de paramètres, mais il peut utiliser des variables pour les paramètres de sortie d'activité et d'entrée de Runbook. Toute sortie de l’activité est ajoutée au bus de données. Une exception est générée sans lien sortant, auquel cas la sortie est ajoutée à la sortie du runbook.

Par exemple, le code suivant effectue des calculs de date en utilisant une variable d’entrée de runbook appelée `NumberOfDays`. Il envoie ensuite la valeur DateTime calculée en tant que sortie qui doit être utilisée par les activités suivantes dans le runbook.

```powershell-interactive
$DateTimeNow = (Get-Date).ToUniversalTime()
$DateTimeStart = ($DateTimeNow).AddDays(-$NumberOfDays)}
$DateTimeStart
```

## <a name="use-links-for-workflow"></a>Utiliser des liens pour flux de travail

Dans un runbook graphique, un lien connecte deux activités. Il est affiché sur le canevas sous la forme d'une flèche pointant de l'activité source vers l'activité de destination. Les activités s'exécutent dans le sens de la flèche, l'activité de destination commençant une fois l'activité source terminée.

### <a name="create-a-link"></a>Créer un lien

Vous pouvez créer un lien entre deux activités en sélectionnant l’activité source, puis en cliquant sur le cercle situé en bas de la forme. Faites glisser la flèche vers l'activité de destination, puis relâchez-la.

![Créer un lien](media/automation-graphical-authoring-intro/create-link-options.png)

Sélectionnez le lien pour configurer ses propriétés dans le panneau Configuration. Les propriétés incluent le type du lien, qui est décrit dans le tableau suivant.

| Type de lien | Description |
|:--- |:--- |
| Pipeline |L’activité de destination s’exécute une fois pour chaque sortie d’objet de l’activité source. L'activité de destination n'est pas exécutée si l'activité source ne génère aucune sortie. La sortie de l'activité source est disponible sous la forme d'un objet. |
| Séquence |L’activité de destination s’exécute une seule fois lorsqu’elle reçoit la sortie de l’activité source. La sortie de l'activité source est disponible sous la forme d'un tableau d'objets. |

### <a name="start-runbook-activity"></a>Appeler l’activité runbook

Un Runbook graphique commence par toute activité qui n'a pas un lien entrant. Il n’y a souvent qu’une seule activité qui agit comme activité de démarrage pour le runbook. Si plusieurs activités n’ont pas de lien entrant, le runbook démarre en les exécutant en parallèle. Lorsque chaque activité est terminée, il suit les liens pour exécuter les autres activités.

### <a name="specify-link-conditions"></a>Spécifier des conditions de lien

Lorsque vous spécifiez une condition sur un lien, l’activité de destination s’exécute uniquement si la condition a pour résultat la valeur True. Vous utilisez généralement une variable `ActivityOutput` dans une condition pour récupérer la sortie de l’activité source.

Pour un lien de pipeline, vous devez spécifier une condition pour un seul objet. Le runbook évalue la condition pour chaque sortie d’objet générée par l’activité source. Il exécute ensuite l’activité de destination pour chaque objet qui remplit la condition. Par exemple, avec une activité source `Get-AzVM`, vous pouvez utiliser la syntaxe suivante pour qu’un lien de pipeline conditionnel récupère uniquement les machines virtuelles se trouvant dans le groupe de ressources nommé Group1.

```powershell-interactive
$ActivityOutput['Get Azure VMs'].Name -match "Group1"
```

Pour un lien de séquence, le runbook évalue la condition une seule fois, car un tableau unique contenant tous les objets de l’activité source est retourné. Pour cette raison, le runbook ne peut pas utiliser un lien de séquence pour le filtrage, ce qui est possible avec un lien de pipeline. Le lien de séquence peut simplement déterminer si l’activité suivante est exécutée.

Prenez par exemple l’ensemble d’activités suivant dans le runbook de **démarrage de machine virtuelle** :

![Lien conditionnel avec séquences](media/automation-graphical-authoring-intro/runbook-conditional-links-sequence.png)

Le runbook utilise trois liens de séquence différents qui vérifient les valeurs des paramètres d’entrée `VMName` et `ResourceGroupName` pour déterminer l’action appropriée à entreprendre. Les actions possibles sont démarrer une seule machine virtuelle, démarrer toutes les machines virtuelles dans le groupe de ressources ou démarrer toutes les machines virtuelles d’un abonnement. Pour le lien de séquence entre `Connect to Azure` et `Get single VM`, voici la logique de la condition :

```powershell-interactive
<#
Both VMName and ResourceGroupName runbook input parameters have values
#>
(
(($VMName -ne $null) -and ($VMName.Length -gt 0))
) -and (
(($ResourceGroupName -ne $null) -and ($ResourceGroupName.Length -gt 0))
)
```

Lorsque vous utilisez un lien conditionnel, les données disponibles de l'activité source vers d'autres activités dans cette branche sont filtrées par la condition. Si une activité est la source vers plusieurs liens, les données disponibles pour les activités de chaque branche dépendent de la condition spécifiée dans le lien de connexion à cette branche.

Par exemple, l’activité `Start-AzVM` du runbook ci-dessous démarre toutes les machines virtuelles. Elle comporte deux liens conditionnels. Le premier lien conditionnel utilise l’expression `$ActivityOutput['Start-AzVM'].IsSuccessStatusCode -eq $true` pour filtrer si l’activité `Start-AzVM` se termine avec succès. Le deuxième lien conditionnel utilise l’expression `$ActivityOutput['Start-AzVM'].IsSuccessStatusCode -ne $true` pour filtrer si l’activité `Start-AzVm` ne parvient pas à démarrer la machine virtuelle.

![Exemple de lien conditionnel](media/automation-graphical-authoring-intro/runbook-conditional-links.png)

Toute activité qui suit le premier lien et utilise la sortie d’activité de `Get-AzureVM` obtient uniquement les machines virtuelles qui étaient démarrées au moment de l’exécution de `Get-AzureVM`. Toute activité qui suit le deuxième lien obtient uniquement les machines virtuelles qui étaient arrêtées au moment de l’exécution de `Get-AzureVM`. Toute activité suivant le troisième lien obtient toutes les machines virtuelles, quel que soit leur état d'exécution.

### <a name="use-junctions"></a>Utiliser des jonctions

Une jonction est une activité spéciale qui attend que toutes les branches entrantes aient terminé. Cela permet au runbook d’exécuter plusieurs activités en parallèle et de s’assurer qu’elles ont toutes terminé avant de continuer.

Alors qu’une jonction peut avoir un nombre illimité de liens entrants, un seul de ces liens peut être un pipeline. Le nombre de liens de type séquence entrants n'est pas limité. Vous pouvez créer la jonction avec plusieurs liens de pipeline entrants et enregistrer le runbook, mais celui-ci échouera lorsqu’il sera exécuté.

L'exemple ci-dessous représente une partie d'un Runbook qui démarre un ensemble de machines virtuelles tout en téléchargeant simultanément des correctifs à appliquer à ces machines. Il utilise une jonction pour s’assurer que les deux processus sont terminés avant de poursuivre.

![jonction](media/automation-graphical-authoring-intro/runbook-junction.png)

### <a name="work-with-cycles"></a>Utiliser des cycles

Un cycle est formé lorsqu’une activité de destination est reliée à son activité source ou à une autre activité qui, au final, est reliée à sa source. La création de graphiques ne prend pas actuellement en charge les cycles. Si votre Runbook a un cycle, il s'enregistre correctement, mais il reçoit une erreur lorsqu'il s'exécute.

![Cycle](media/automation-graphical-authoring-intro/runbook-cycle.png)

### <a name="share-data-between-activities"></a>Partage de données entre activités

Toutes les données générées par une activité avec un lien sortant sont écrites dans le bus de données du runbook. Toute activité du Runbook peut utiliser les données présentes dans le bus de données pour renseigner les valeurs de paramètres ou les inclure dans le code de script. Une activité peut accéder à la sortie de toute activité précédente du workflow.

La façon dont les données sont écrites dans le bus de données varie en fonction du type de lien figurant sur l'activité. Pour un lien de pipeline, les données sont générées sous la forme de plusieurs objets. Pour un lien de séquence, les données sont générées sous la forme d’un tableau. S’il n’y a qu’une seule valeur, elles sont générées sous la forme d’un tableau à un seul élément.

Votre runbook dispose de deux méthodes pour accéder aux données du bus de données : 
* Utiliser une sortie d’activité comme source de données.
* Utiliser une expression PowerShell comme source de données.

Le premier mécanisme utilise une source de données de sortie d’activité pour renseigner le paramètre d’une autre activité. Si la sortie est un objet, le runbook peut spécifier une seule propriété.

![sortie d’activité](media/automation-graphical-authoring-intro/activity-output-datasource-revised20165.png)

Le deuxième mécanisme d’accès aux données récupère la sortie d’une activité dans une source de données d’expression PowerShell ou à partir d’une activité de script de workflow avec une variable `ActivityOutput`, en utilisant la syntaxe présentée ci-dessous. Si la sortie est un objet, votre runbook peut spécifier une seule propriété.

```powershell-interactive
$ActivityOutput['Activity Label']
$ActivityOutput['Activity Label'].PropertyName
```

### <a name="use-checkpoints"></a>Utilisez des points de contrôle

Vous pouvez définir des [points de contrôle](automation-powershell-workflow.md#use-checkpoints-in-a-workflow) dans un runbook de workflow PowerShell graphique en sélectionnant **Runbook de point de contrôle** sur n’importe quelle activité. Cette option permet de définir un point de contrôle après l’exécution de l’activité.

![Point de contrôle](media/automation-graphical-authoring-intro/set-checkpoint.png)

Les points de contrôle sont activés uniquement dans les runbooks de workflow PowerShell graphique ; ils ne sont pas disponibles dans les runbooks graphiques. Si le runbook utilise des cmdlets Azure, il doit faire suivre une activité de point de contrôle par une activité `Connect-AzAccount`. L’opération de connexion est utilisée au cas où le runbook est interrompu et doit redémarrer à partir de ce point de contrôle sur un autre Worker.

## <a name="handle-runbook-input"></a>Gérer l’entrée de runbook

Un runbook nécessite une entrée soit d’un utilisateur qui démarre le runbook via le Portail Azure, soit d’un autre runbook si le runbook actuel est utilisé en tant qu’enfant. Par exemple, pour un runbook qui crée une machine virtuelle, l’utilisateur peut devoir fournir des informations telles que le nom de la machine virtuelle et d’autres propriétés chaque fois que le runbook démarre.

Le runbook accepte une entrée en définissant un ou plusieurs paramètres d’entrée. L’utilisateur fournit des valeurs pour ces paramètres chaque fois que le runbook démarre. Lorsque l’utilisateur démarre le runbook à l’aide du Portail Azure, l’utilisateur est invité à fournir des valeurs pour chaque paramètre d’entrée pris en charge par le runbook.

Lorsque vous créez votre runbook, vous pouvez accéder à ses paramètres d’entrée en cliquant sur **Entrée et sortie** dans la barre d’outils du runbook. Cela ouvre le contrôle Entrée et sortie où vous pouvez modifier un paramètre d’entrée existant ou en créer un nouveau en cliquant sur **Ajouter une entrée**.

![Ajouter une entrée](media/automation-graphical-authoring-intro/runbook-edit-add-input.png)

Chaque paramètre d'entrée est défini par les propriétés figurant dans le tableau suivant :

| Propriété | Description |
|:--- |:--- |
| Nom | Obligatoire. Le nom du paramètre. Ce nom doit être unique dans le runbook. Il doit commencer par une lettre et ne peut contenir que des lettres, des chiffres et des traits de soulignement. Le nom ne doit pas contenir un espace. |
| Description |facultatif. Description de l’objet du paramètre d’entrée. |
| Type | facultatif. Type de données attendu pour la valeur de paramètre. Le portail Azure fournit un contrôle approprié pour le type de données de chaque paramètre lorsque vous êtes invité à indiquer une entrée. Les types de paramètres pris en charge sont String, Int32, Int64, Decimal, Boolean, DateTime et Object. Si un type de données n’est pas sélectionné, le type par défaut est String.|
| Obligatoire | facultatif. Paramètre qui spécifie si une valeur doit être fournie pour le paramètre. Si vous choisissez `yes`, une valeur doit être fournie au démarrage du runbook. Si vous choisissez `no`, aucune valeur n’est exigée au démarrage du runbook, et une valeur par défaut peut être utilisée. Le runbook ne peut pas démarrer si vous ne fournissez pas une valeur pour chaque paramètre obligatoire pour lequel aucune valeur par défaut n’est définie. |
| Valeur par défaut | facultatif. Valeur utilisée pour un paramètre si aucune valeur n’est passée au démarrage du runbook. Pour définir une valeur par défaut, choisissez `Custom`. Sélectionnez `None` si vous ne souhaitez pas fournir de valeur par défaut. |

## <a name="handle-runbook-output"></a>Gérer la sortie de runbook

La création de graphiques enregistre les données créées par toute activité qui ne dispose pas d’un lien sortant dans la [sortie du runbook](./automation-runbook-output-and-messages.md). La sortie est enregistrée avec la tâche du Runbook et est disponible pour un Runbook parent lorsque le Runbook est utilisé en tant qu'enfant.

## <a name="work-with-powershell-expressions"></a>Utiliser des expressions PowerShell

Un des avantages de la création de graphiques est qu’elle vous permet de créer un runbook avec une connaissance minimale de PowerShell. À l’heure actuelle, vous devez néanmoins avoir quelques connaissances de base sur PowerShell pour remplir certaines [valeurs de paramètres](#use-activities) et définir des [conditions de lien](#use-links-for-workflow). Cette section fournit une brève introduction aux expressions PowerShell. La totalité des informations sur PowerShell est disponible dans [Écriture de scripts avec Windows PowerShell](/powershell/scripting/overview).

### <a name="use-a-powershell-expression-as-a-data-source"></a>Utiliser une expression PowerShell comme source de données

Vous pouvez utiliser une expression PowerShell comme source de données pour remplir la valeur d’un [paramètre d’activité](#use-activities) avec les résultats d’un code PowerShell. L’expression peut être une seule ligne de code qui exécute une fonction simple ou plusieurs lignes qui suivent une logique complexe. Toute sortie de commande non affectée à une variable correspond à la sortie de la valeur du paramètre.

Par exemple, la commande suivante affiche la date actuelle.

```powershell-interactive
Get-Date
```

L’extrait de code suivant crée une chaîne à partir de la date actuelle et l’affecte à une variable. Le code envoie le contenu de la variable à la sortie.

```powershell-interactive
$string = "The current date is " + (Get-Date)
$string
```

Les commandes suivantes évaluent la date actuelle et renvoient une chaîne qui indique si le jour actuel est un jour de la semaine ou du week-end.

```powershell-interactive
$date = Get-Date
if (($date.DayOfWeek = "Saturday") -or ($date.DayOfWeek = "Sunday")) { "Weekend" }
else { "Weekday" }
```

### <a name="use-activity-output"></a>Utiliser une sortie d’activité

Pour utiliser la sortie d’une activité précédente dans votre runbook, utilisez la variable `ActivityOutput` avec la syntaxe suivante.

```powershell-interactive
$ActivityOutput['Activity Label'].PropertyName
```

Par exemple, vous pouvez avoir une activité dont la propriété requiert le nom d’une machine virtuelle. Dans ce cas, votre runbook peut utiliser l’expression suivante.

```powershell-interactive
$ActivityOutput['Get-AzureVM'].Name
```

Si la propriété requiert l’objet de machine virtuelle au lieu d’un simple nom, le runbook retourne l’objet entier à l’aide de la syntaxe suivante.

```powershell-interactive
$ActivityOutput['Get-AzureVM']
```

Le runbook peut utiliser la sortie d’une activité dans une expression plus complexe, telle que la suivante. Cette expression concatène le texte à l’identité de machine virtuelle.

```powershell-interactive
"The computer name is " + $ActivityOutput['Get-AzureVM'].Name
```

### <a name="compare-values"></a>Comparer des valeurs

Utilisez les [opérateurs de comparaison](/powershell/module/microsoft.powershell.core/about/about_comparison_operators) pour comparer des valeurs ou déterminer si une valeur correspond à un modèle spécifié. Une comparaison renvoie la valeur True ou False.

Par exemple, la condition suivante détermine si la machine virtuelle d’une activité nommée `Get-AzureVM` est actuellement arrêtée.

```powershell-interactive
$ActivityOutput["Get-AzureVM"].PowerState –eq "Stopped"
```

La condition suivante détermine si la même machine virtuelle est dans un état autre qu’arrêté.

```powershell-interactive
$ActivityOutput["Get-AzureVM"].PowerState –ne "Stopped"
```

Vous pouvez joindre plusieurs conditions dans votre runbook en utilisant un [opérateur logique](/powershell/module/microsoft.powershell.core/about/about_logical_operators), comme `-and` ou `-or`. Par exemple, la condition suivante détermine si la machine virtuelle de l’exemple précédent est à l’état Arrêté ou En cours d’arrêt.

```powershell-interactive
($ActivityOutput["Get-AzureVM"].PowerState –eq "Stopped") -or ($ActivityOutput["Get-AzureVM"].PowerState –eq "Stopping")
```

### <a name="use-hashtables"></a>Utiliser des tables de hachage

Les [tables de hachage](/powershell/module/microsoft.powershell.core/about/about_hash_tables) sont des paires nom-valeur servant à renvoyer un ensemble de valeurs. Vous pouvez également voir une table de hachage appelée dictionnaire. Les propriétés de certaines activités attendent une table de hachage plutôt qu’une valeur simple.

Créez une table de hachage à l’aide de la syntaxe suivante. Elle peut contenir un nombre quelconque d’entrées, mais chacune d’entre elles est définie par un nom et une valeur.

```powershell-interactive
@{ <name> = <value>; [<name> = <value> ] ...}
```

Par exemple, l’expression suivante crée une table de hachage à utiliser comme source de données pour un paramètre d’activité qui attend une table de hachage contenant des valeurs pour une recherche sur Internet.

```powershell-interactive
$query = "Azure Automation"
$count = 10
$h = @{'q'=$query; 'lr'='lang_ja';  'count'=$Count}
$h
```

L'exemple suivant utilise la sortie d'une activité nommée `Get Twitter Connection` pour remplir une table de hachage.

```powershell-interactive
@{'ApiKey'=$ActivityOutput['Get Twitter Connection'].ConsumerAPIKey;
    'ApiSecret'=$ActivityOutput['Get Twitter Connection'].ConsumerAPISecret;
    'AccessToken'=$ActivityOutput['Get Twitter Connection'].AccessToken;
    'AccessTokenSecret'=$ActivityOutput['Get Twitter Connection'].AccessTokenSecret}
```

## <a name="authenticate-to-azure-resources"></a>Authentifier auprès de ressources Azure

Dans Azure Automation, les Runbooks qui gèrent des ressources Azure doivent s’authentifier auprès d’Azure. La fonctionnalité [Compte d’identification](./manage-runas-account.md), également appelée principal de service, est le mécanisme utilisé par défaut par un runbook Automation pour accéder aux ressources Azure Resource Manager dans votre abonnement. Vous pouvez ajouter cette fonctionnalité à un runbook graphique en ajoutant la ressource de connexion `AzureRunAsConnection`, qui utilise la cmdlet PowerShell [Get-AutomationConnection](/system-center/sma/manage-global-assets), au canevas. Vous pouvez aussi utilisez l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Ce scénario est illustré dans l’exemple suivant.

![Activités d’authentification de l’identification](media/automation-graphical-authoring-intro/authenticate-run-as-account.png)

L’activité `Get Run As Connection`, ou `Get-AutomationConnection`, est configurée avec une source de données de valeur constante nommée `AzureRunAsConnection`.

![Configuration de la connexion d’identification](media/automation-graphical-authoring-intro/authenticate-runas-parameterset.png)

L’activité suivante, `Connect-AzAccount`, ajoute le compte d’identification authentifié pour l’utiliser dans le runbook.

![Jeu de paramètres Connect-AzAccount](media/automation-graphical-authoring-intro/authenticate-conn-to-azure-parameter-set.png)

>[!NOTE]
>Pour les runbooks PowerShell, `Add-AzAccount` et `Add-AzureRMAccount` sont des alias de `Connect-AzAccount`. Notez que ces alias ne sont pas disponibles pour vos runbooks graphiques. Un runbook graphique peut uniquement utiliser `Connect-AzAccount`.

Pour les champs de paramètres **APPLICATIONID**, **CERTIFICATETHUMBPRINT** et **TENANTID**, spécifiez le nom de la propriété pour le chemin du champ, car l’activité génère un objet comportant plusieurs propriétés. À défaut, la tentative d’authentification échoue lors de l’exécution du runbook. Vous devez au minimum renseigner ces informations pour authentifier votre runbook auprès du compte d’identification.

Certains abonnés créent un compte Automation en utilisant un [compte d’utilisateur Azure AD](./shared-resources/credentials.md) pour gérer le déploiement Azure Classic ou pour bénéficier des ressources Azure Resource Manager. Pour assurer une compatibilité descendante pour ces abonnés, le mécanisme d’authentification à utiliser dans votre runbook est la cmdlet `Add-AzureAccount` avec une [ressource d’informations d’identification](./shared-resources/credentials.md). La ressource représente un utilisateur Active Directory ayant accès au compte Azure.

Vous pouvez activer cette fonctionnalité pour votre runbook graphique en ajoutant une ressource d’informations d’identification au canevas, suivie d’une activité `Add-AzureAccount` qui utilise la ressource d’informations d’identification pour son entrée. Consultez l’exemple qui suit.

![Activités d'authentification](media/automation-graphical-authoring-intro/authentication-activities.png)

Le runbook doit s’authentifier au moment de son démarrage et après chaque point de contrôle. Vous devez donc utiliser une activité `Add-AzureAccount` après toute activité `Checkpoint-Workflow`. Vous n’avez pas besoin d’utiliser une activité d’informations d’identification supplémentaire.

![Sortie d'activité](media/automation-graphical-authoring-intro/authentication-activity-output.png)

## <a name="export-a-graphical-runbook"></a>Exporter un runbook graphique

Vous pouvez exporter uniquement la version publiée d'un Runbook graphique. Si le runbook n’a pas encore été publié, le bouton **Exporter** est désactivé. Lorsque vous cliquez sur le bouton **Exporter**, le runbook est téléchargé sur votre ordinateur local. Le nom du fichier correspond au nom du runbook avec une extension **.graphrunbook**.

## <a name="import-a-graphical-runbook"></a>Importer un runbook graphique

Vous pouvez importer un fichier de runbook graphique ou de workflow PowerShell graphique en sélectionnant l’option **Importer** lors de l’ajout d’un runbook. Lorsque vous sélectionnez le fichier à importer, vous pouvez conserver le même nom ou en fournir un nouveau. Le champ **Type de runbook** affiche le type de runbook après avoir évalué le fichier sélectionné. Si vous tentez de sélectionner un autre type qui n’est pas correct, l’éditeur graphique affiche un message indiquant qu’il existe des conflits potentiels et qu’il peut y avoir des erreurs de syntaxe lors de la conversion.

![Importer un Runbook](media/automation-graphical-authoring-intro/runbook-import.png)

## <a name="test-a-graphical-runbook"></a>Tester un runbook graphique

Dans Azure Automation, chaque runbook graphique a une version brouillon et une version publiée. Vous ne pouvez exécuter que la version publiée et modifier la version brouillon. La version publiée n'est pas affectée par les modifications apportées à la version Bouillon. Lorsque la version brouillon est prête à être utilisée, vous la publiez, ce qui remplace la version publiée actuelle par votre version brouillon.

Vous pouvez tester la version brouillon d’un runbook dans le Portail Azure tout en gardant la version publiée inchangée. Vous pouvez également tester un nouveau runbook avant sa publication afin de vérifier qu’il fonctionne correctement avant tout remplacement de version. Le test d’un runbook exécute la version brouillon et garantit que toutes les actions qu’il effectue sont terminées. Aucun historique des travaux n’est créé, mais la sortie est affichée dans le volet Sortie de test.

Ouvrez le contrôle Test pour votre runbook graphique en ouvrant le runbook en vue de sa modification, puis en cliquant sur **Volet de test**. Le contrôle Test demande les paramètres d’entrée, et vous pouvez démarrer le runbook en cliquant sur **Démarrer**.

## <a name="publish-a-graphical-runbook"></a>Publier un runbook graphique

Publiez un runbook graphique en ouvrant le runbook en vue de sa modification, puis en cliquant sur **Publier**. Les états possibles pour le runbook sont les suivants :

* Nouveau : le runbook n’a pas encore été publié. 
* Publié : le runbook a été publié.
* En mode Édition : le runbook a été modifié après sa publication, et les versions brouillon et publiée sont différentes.

![États du Runbook](media/automation-graphical-authoring-intro/runbook-statuses-revised20165.png)

Vous avez la possibilité de rétablir la version publiée d’un runbook. Cette opération supprime toutes les modifications apportées depuis la dernière publication de runbook. Cela remplace la version brouillon du runbook par la version publiée.

## <a name="next-steps"></a>Étapes suivantes

* Pour commencer à utiliser des runbooks graphiques, consultez [Tutoriel : Créer un runbook graphique](learn/automation-tutorial-runbook-graphical.md).
* Pour en savoir plus sur les types de runbook, leurs avantages et leurs limites, consultez [Types de runbooks Azure Automation](automation-runbook-types.md).
* Pour comprendre les mécanismes d’authentification à l’aide du compte d’identification Automation, consultez [Compte d’identification](automation-security-overview.md#run-as-account).
* Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0&preserve-view=true#automation).
