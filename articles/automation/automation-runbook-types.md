---
title: Types de Runbooks Azure Automation
description: Cet article décrit les différents types de runbooks que vous pouvez utiliser dans Azure Automation, et énonce des considérations pour déterminer le type à utiliser.
services: automation
ms.subservice: process-automation
ms.date: 03/05/2019
ms.topic: conceptual
ms.openlocfilehash: 24d0123eecc56b56573e94d831283d8d360cd16e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86185923"
---
# <a name="azure-automation-runbook-types"></a>Types de Runbooks Azure Automation

La fonctionnalité d’automatisation des processus Azure Automation prend en charge plusieurs types de runbooks, conformément à la définition du tableau suivant. Pour en savoir plus sur l’environnement d’automatisation des processus, consultez [Exécution d’un runbook dans Azure Automation](automation-runbook-execution.md).

| Type | Description |
|:--- |:--- |
| [Graphique](#graphical-runbooks)|Runbook graphique basé sur Windows PowerShell et créé et modifié entièrement dans l'éditeur graphique du portail Azure. |
| [Graphique workflow PowerShell](#graphical-runbooks)|Runbook graphique basé sur le workflow Windows PowerShell et créé et modifié entièrement dans l'éditeur graphique du portail Azure. |
| [PowerShell](#powershell-runbooks) |Runbook texte basé sur les scripts Windows PowerShell. |
| [Workflow PowerShell](#powershell-workflow-runbooks)|Runbook texte basé sur le workflow de scripts Windows PowerShell. |
| [Python](#python-runbooks) |Runbook de texte basé sur les scripts Python. |

Prenez en compte les considérations suivantes pour déterminer le type à utiliser pour un runbook donné.

* Vous ne pouvez pas convertir de runbooks du type graphique au type texte ou vice versa.
* Il existe des limitations lorsque des runbooks de différents types sont utilisés comme runbooks enfants. Pour plus d’informations, consultez la page [Runbooks enfants dans Azure Automation](automation-child-runbooks.md).

## <a name="graphical-runbooks"></a>Runbooks graphiques

Vous pouvez créer et modifier des runbooks graphiques de workflow PowerShell avec l’éditeur graphique du portail Azure. Vous ne pouvez cependant ni créer ni modifier ce type de runbook avec un autre outil. Principales fonctionnalités des runbooks graphiques :

* Ils peuvent être exportés vers des fichiers de votre compte Automation, puis importés dans un autre compte Automation. 
* Générer du code PowerShell. 
* Les runbooks graphiques peuvent être convertis en runbooks de workflow PowerShell pendant l’importation et vice versa. 

### <a name="advantages"></a>Avantages

* Utilisation d’un modèle de création visuel insert-link-configure.
* Concentration sur la circulation des flux de données dans tout le processus.
* Représentez visuellement les processus de gestion.
* Inclusion d’autres runbooks en tant que runbooks enfants pour créer des workflows de niveau élevé.
* Programmation modulaire favorisée.

### <a name="limitations"></a>Limites

* Impossible de créer ou de modifier en dehors du portail Azure.
* Peut nécessiter une activité de code contenant du code PowerShell pour exécuter une logique complexe.
* Impossible d’effectuer une conversion vers l’un des [formats de texte](automation-runbook-types.md) ou de convertir un runbook de texte au format graphique. 
* Impossible d'afficher ou de modifier directement du code PowerShell créé par le workflow graphique. Vous pouvez afficher le code créé dans toute activité de code.
* Impossible d’exécuter des runbooks sur un Runbook Worker hybride. Consultez [Automatiser les ressources de votre centre de données ou de votre cloud à l’aide d’un Runbook Worker hybride](automation-hybrid-runbook-worker.md).

## <a name="powershell-runbooks"></a>Runbooks PowerShell

Les Runbooks PowerShell sont basés sur Windows PowerShell. Vous modifiez directement le code du Runbook à l'aide de l'éditeur de texte du portail Azure.  Vous pouvez également utiliser n'importe quel éditeur de texte hors ligne et [importer le Runbook](manage-runbooks.md) dans Azure Automation.

### <a name="advantages"></a>Avantages

* Implémentez tout type de la logique complexe avec le code PowerShell sans la complexité supplémentaire liée au workflow PowerShell.
* Démarrez plus rapidement que les runbooks de workflow PowerShell dans la mesure où ils n’ont pas besoin d'être compilés avant l'exécution.
* Exécutez dans Azure et sur des Runbook Workers hybrides pour Windows et Linux.

### <a name="limitations"></a>Limites

* Vous devez être familiarisé avec les scripts PowerShell.
* Les runbooks ne peuvent pas utiliser un [traitement en parallèle](automation-powershell-workflow.md#use-parallel-processing) pour effectuer plusieurs actions en parallèle.
* Les runbooks ne peuvent pas utiliser de [points de contrôle](automation-powershell-workflow.md#use-checkpoints-in-a-workflow) pour reprendre le runbook en cas d'erreur.
* Vous pouvez inclure uniquement les runbooks de workflow PowerShell et les runbooks graphiques comme runbooks enfants à l'aide de l'applet de commande [Start-AzureAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook?view=azps-3.7.0), ce qui crée un travail.

### <a name="known-issues"></a>Problèmes connus

Voici les problèmes connus actuels rencontrés avec les runbooks PowerShell :

* Les runbooks PowerShell ne peuvent pas récupérer une [ressource variable](./shared-resources/variables.md) non chiffrée avec une valeur null.
* Les runbooks PowerShell ne peuvent pas récupérer une ressource variable dont le nom contient le symbole `*~*`.
* Une opération [Get-Process](/powershell/module/microsoft.powershell.management/get-process?view=powershell-7) dans une boucle d’un runbook PowerShell peut se bloquer après environ 80 itérations.
* Un runbook PowerShell peut échouer s’il essaie d'écrire une grande quantité de données à la fois dans le flux de sortie. Vous pouvez généralement contourner ce problème. Pour ce faire, le runbook doit sortir uniquement les informations nécessaires pour travailler avec des objets volumineux. Par exemple, au lieu d’utiliser `Get-Process` sans aucune limitation, vous pouvez faire en sorte que l’applet de commande génère uniquement les paramètres requis, comme dans `Get-Process | Select ProcessName, CPU`.

## <a name="powershell-workflow-runbooks"></a>Runbooks de workflow PowerShell

Les Runbooks de workflow PowerShell sont des Runbooks texte basés sur un [workflow Windows PowerShell](automation-powershell-workflow.md). Vous modifiez directement le code du Runbook à l'aide de l'éditeur de texte du portail Azure. Vous pouvez également utiliser n'importe quel éditeur de texte hors ligne et [importer le Runbook](manage-runbooks.md) dans Azure Automation.

### <a name="advantages"></a>Avantages

* Implémentez tout type de logique complexe avec le code de workflow PowerShell.
* Utilisez des [points de contrôle](automation-powershell-workflow.md#use-checkpoints-in-a-workflow) pour reprendre l’opération en cas d'erreur.
* Utilisez un [traitement en parallèle](automation-powershell-workflow.md#use-parallel-processing) pour mener plusieurs actions en parallèle.
* Peut inclure d'autres runbooks graphiques et des runbooks de workflow PowerShell en tant que runbooks enfants afin de créer des workflows de haut niveau.

### <a name="limitations"></a>Limites

* Vous devez être familiarisé avec les workflows PowerShell.
* Les runbooks doivent pouvoir gérer la complexité supplémentaire liée au workflow PowerShell, notamment les [objets désérialisés](automation-powershell-workflow.md#deserialized-objects).
* Les runbooks prennent plus de temps à démarrer que les runbooks PowerShell, car il doivent être compilés avant l'exécution.
* Vous pouvez inclure uniquement des runbooks PowerShell en tant que runbooks enfants à l’aide de l’applet de commande `Start-AzAutomationRunbook`.
* Les runbooks ne peuvent pas être exécutés sur un Runbook Worker hybride.

## <a name="python-runbooks"></a>Runbooks Python

Compilation de runbooks Python sous Python 2. Vous pouvez modifierz directement le code du runbook à l'aide de l'éditeur de texte du portail Azure. Vous pouvez également utiliser un éditeur de texte hors ligne et [importer le runbook](manage-runbooks.md) dans Azure Automation.

### <a name="advantages"></a>Avantages

* Utilisation des bibliothèques Python robustes.
* Possibilité d’exécution dans Azure ou sur Runbook Worker hybride. Les runbooks Worker hybrides Windows sont pris en charge si [Python 2.7](https://www.python.org/downloads/release/latest/python2) est installé.

### <a name="limitations"></a>Limites

* Vous devez être familiarisé avec les scripts Python.
* Seul Python 2 est actuellement pris en charge. Toutes les fonctions spécifiques à Python 3 échouent.
* Pour utiliser des bibliothèques tierces, vous devez [importer les packages](python-packages.md) dans le compte Automation.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les runbooks PowerShell, consultez [Tutorial : Créer un runbook PowerShell](learn/automation-tutorial-runbook-textual-powershell.md).
* Pour découvrir les runbooks de flux de travail, consultez [Tutoriel : Créer un runbook de flux de travail PowerShell](learn/automation-tutorial-runbook-textual.md).
* Pour découvrir les runbooks graphiques, consultez [Futoriel : Créer un runbook graphique](learn/automation-tutorial-runbook-graphical.md).
* Pour en savoir plus sur les runbooks Python, consultez [Tutorial : Créer un runbook Python](learn/automation-tutorial-runbook-textual-python2.md).
