---
title: Migrer d’Orchestrator vers Azure Automation (bêta)
description: Cet article explique comment effectuer la migration des runbooks et des packs d’intégration entre Orchestrator et Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 03/16/2018
ms.topic: conceptual
ms.openlocfilehash: a47f720344a16d0f77559d6aabfb2b0245e62976
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89426331"
---
# <a name="migrate-from-orchestrator-to-azure-automation-beta"></a>Migrer d’Orchestrator vers Azure Automation (bêta)

Dans [System Center 2012 - Orchestrator](/previous-versions/system-center/system-center-2012-R2/hh237242(v=sc.12)), les runbooks sont basés sur les activités de packs d’intégration spécifiquement écrits pour Orchestrator, tandis que dans Azure Automation, ils sont basés sur Windows PowerShell. Dans Azure Automation, les [runbooks graphiques](automation-runbook-types.md#graphical-runbooks) ont une apparence semblable à celle des runbooks Orchestrator, avec leurs activités représentant les applets de commande PowerShell, les runbooks enfants et les ressources. En plus de convertir les runbooks proprement dits, vous devez convertir les packs d’intégration avec les activités qu’ils utilisent en modules d’intégration à l’aide d’applets de commande Windows PowerShell. 

[Service Management Automation](/previous-versions/system-center/system-center-2012-R2/dn469260(v=sc.12)) (SMA) stocke et exécute des runbooks dans votre centre de données local comme Orchestrator, et utilise les mêmes modules d’intégration qu’Azure Automation. Lorsqu’il est disponible, le convertisseur de runbooks convertit les runbooks Orchestrator en runbooks graphiques, lesquels ne sont pas pris en charge dans SMA. Vous pouvez toujours installer le module d’activités standard et les modules d’intégration de System Center Orchestrator dans SMA, mais vous devez [réécrire vos runbooks](/system-center/sma/authoring-automation-runbooks) manuellement.

## <a name="download-the-orchestrator-migration-toolkit"></a>Télécharger le kit de migration Orchestrator

La première étape de migration consiste à télécharger le [kit de migration System Center Orchestrator](https://www.microsoft.com/download/details.aspx?id=47323). Le kit de migration comprend des outils destinés à convertir des runbooks Orchestrator en runbooks Azure Automation.  

## <a name="import-the-standard-activities-module"></a>Importer le module d’activités Standard

Importez le [module d’activités Standard](/system-center/orchestrator/standard-activities?view=sc-orch-2019) dans Azure Automation. Cela inclut les versions converties des activités Orchestrator standard que les runbooks graphiques convertis peuvent utiliser.

## <a name="import-orchestrator-integration-modules"></a>Importer les modules d’intégration Orchestrator

Microsoft fournit des [packs d'intégration](/previous-versions/system-center/packs/hh295851(v=technet.10)) destinés à la création des Runbooks afin d'automatiser les composants de System Center et d'autres produits. Certains de ces packs d'intégration sont actuellement basés sur OIT, mais ne peuvent actuellement pas être convertis en modules d'intégration en raison de problèmes connus. Importez les [modules d’intégration de System Center Orchestrator](https://www.microsoft.com/download/details.aspx?id=49555) dans Azure Automation pour les packs d’intégration utilisés par vos runbooks ayant accès à System Center. Ce package comprend des versions converties de ces packs d’intégration qui peuvent être importées dans Azure Automation et dans Service Management Automation.  

## <a name="convert-integration-packs"></a>Convertir des packs d’intégration

Utilisez le [convertisseur de packs d’intégration](/system-center/orchestrator/orch-integration-toolkit/integration-pack-wizard?view=sc-orch-2019) pour convertir les packs d’intégration créés avec [Orchestrator Integration Toolkit (OIT)](/previous-versions/system-center/developer/hh855853(v=msdn.10)) en modules d’intégration basés sur PowerShell et pouvant être importés dans Azure Automation ou dans Service Management Automation. Lorsque vous exécutez le convertisseur de packs d’intégration, un Assistant vous permettant de sélectionner un fichier de pack d’intégration (.oip) s’affiche. Cet Assistant liste ensuite les activités incluses dans ce pack d’intégration et vous permet de sélectionner celles qui doivent faire l’objet d’une migration. Une fois l'Assistant terminé, il crée un module qui inclut une applet de commande correspondant pour chacune des activités du pack d'intégration d'origine.

> [!NOTE]
> Vous ne pouvez pas utiliser le convertisseur de packs d’intégration pour convertir les packs d’intégration qui n’ont pas été créés avec OIT. Il existe également des packs d’intégration fournis par Microsoft qui ne peuvent pas être convertis avec cet outil. Les versions converties de ces packs d’intégration sont fournies pour le téléchargement afin de pouvoir être installées dans Azure Automation ou Service Management Automation.

### <a name="parameters"></a>Paramètres

Toutes les propriétés d'une activité du pack d'intégration sont converties en paramètres de l'applet de commande correspondante dans le module d'intégration.  Les applets de commande Windows PowerShell possèdent un ensemble de [paramètres communs](/powershell/module/microsoft.powershell.core/about/about_commonparameters) qui peuvent être utilisés avec toutes les applets de commande. Par exemple, le paramètre -Verbose entraîne l'affichage, par une applet de commande, des informations détaillées relatives à son fonctionnement.  Aucune applet de commande ne peut avoir un paramètre portant le même nom qu'un paramètre commun. Si une activité a une propriété portant le même nom qu’un paramètre commun, l’Assistant vous invite à fournir un autre nom pour le paramètre.

### <a name="monitor-activities"></a>Activités d'analyse

Dans Orchestrator, les Runbooks d’analyse commencent par une [activité d’analyse](/previous-versions/system-center/system-center-2012-R2/hh403827(v=sc.12)) et s’exécutent en permanence, attendant d’être appelés par un événement particulier. Azure Automation ne prend pas en charge les runbooks de supervision. Les activités de supervision se trouvant dans le pack d’intégration ne sont donc pas converties. Au lieu de cela, une applet de commande réservée est créée dans le module d'intégration pour l'activité d'analyse.  Cette applet de commande n'a aucune fonctionnalité, mais elle permet l'installation de tout Runbook converti qui l'utilise. Ce runbook ne peut pas s’exécuter dans Azure Automation, mais il peut être installé afin d’être modifié.

Orchestrator inclut un ensemble d' [activités standard](/previous-versions/system-center/system-center-2012-R2/hh403832(v=sc.12)) qui ne sont pas incluses dans un pack d'intégration, mais qui sont utilisées par de nombreux Runbooks.  Le module d’activités standard est un module d’intégration qui inclut une applet de commande équivalente pour chacune de ces activités. Vous devez installer le module d'intégration dans Azure Automation avant d'importer des Runbooks convertis qui utilisent une activité standard.

En plus de prendre en charge les Runbooks convertis, les applets de commande du module d'activités standard peuvent être utilisées par une personne ayant une bonne connaissance d'Orchestrator pour créer de nouveaux Runbooks dans Azure Automation. Alors que les fonctionnalités de toutes les activités standard peuvent être effectuées à l'aide d'applets de commande, il est possible qu'elles fonctionnent différemment. Les applets de commande du module d’activités standard converties fonctionnent de la même manière que leurs activités correspondantes et utilisent les mêmes paramètres. Cela peut faciliter votre transition vers les runbooks Azure Automation.

## <a name="convert-orchestrator-runbooks"></a>Convertir des runbooks Orchestrator

Le convertisseur de runbooks Orchestrator convertit les runbooks Orchestrator en [runbooks graphiques](automation-runbook-types.md#graphical-runbooks) qui peuvent être importés dans Azure Automation. Le convertisseur de runbooks est implémenté sous forme de module PowerShell à l’aide de l’applet de commande `ConvertFrom-SCORunbook` qui effectue la conversion. Lorsque vous installez le convertisseur, cela crée un raccourci vers une session PowerShell qui charge l’applet de commande.   

Voici les étapes de base pour convertir un runbook et l’importer dans Azure Automation. L’utilisation de l’applet de commande est expliquée en détail plus loin dans cette section.

1. Exporter un ou plusieurs Runbooks depuis Orchestrator.
2. Obtenir des modules d'intégration pour toutes les activités dans le Runbook.
3. Convertir les Runbooks Orchestrator dans le fichier exporté.
4. Passer en revue les informations des journaux d’activité pour valider la conversion et déterminer les tâches manuelles requises.
5. Importer les Runbooks convertis dans Azure Automation.
6. Créer tous les éléments requis dans Azure Automation.
7. Modifier le Runbook dans Azure Automation pour modifier toutes les activités requises.

La syntaxe de `ConvertFrom-SCORunbook` est la suivante :

```powershell
ConvertFrom-SCORunbook -RunbookPath <string> -Module <string[]> -OutputFolder <string>
```

* RunbookPath - chemin d'accès au fichier d'exportation contenant les Runbooks à convertir.
* Module - liste délimitée par des virgules de modules d'intégration contenant des activités dans les Runbooks.
* OutputFolder - chemin d'accès au dossier pour créer des Runbooks graphiques convertis.

L’exemple de commande suivant convertit les Runbooks dans le fichier d’exportation **MyRunbooks.ois_export**.  Ces Runbooks utilisent les packs d'intégration Active Directory et Data Protection Manager.

```powershell
ConvertFrom-SCORunbook -RunbookPath "c:\runbooks\MyRunbooks.ois_export" -Module c:\ip\SystemCenter_IntegrationModule_ActiveDirectory.zip,c:\ip\SystemCenter_IntegrationModule_DPM.zip -OutputFolder "c:\runbooks"
```

### <a name="use-runbook-converter-log-files"></a>Utiliser les fichiers journaux du convertisseur de runbooks

Le convertisseur de runbooks crée les fichiers journaux suivants au même emplacement que le runbook converti.  Si les fichiers existent déjà, ils sont remplacés par les informations de la dernière conversion.

| Fichier | Contents |
|:--- |:--- |
| Convertisseur de Runbooks - Progress.log |Étapes détaillées de la conversion incluant des informations sur chaque activité convertie avec succès et un avertissement pour chaque activité non convertie. |
| Convertisseur de Runbooks - Summary.log |Résumé de la dernière conversion, y compris tous les avertissements et le suivi des tâches que vous devez effectuer, par exemple la création d'une variable requise pour le Runbook converti. |

### <a name="export-runbooks-from-orchestrator"></a>Exporter des runbooks à partir d’Orchestrator

Le convertisseur de Runbooks fonctionne avec un fichier exporté Orchestrator qui contient un ou plusieurs Runbooks.  Il crée un runbook Azure Automation correspondant pour chaque runbook Orchestrator du fichier d’exportation.  

Pour exporter un Runbook à partir d'Orchestrator, cliquez sur le nom du Runbook dans Runbook Designer et sélectionnez **Exporter**.  Pour exporter tous les Runbooks, cliquez avec le bouton droit sur le nom du dossier, puis sélectionnez **Exporter**.

### <a name="convert-runbook-activities"></a>Convertir les activités de runbook

Le convertisseur de Runbooks convertit chaque activité du Runbook Orchestrator en activité correspondante dans Azure Automation.  Pour les activités qui ne peuvent pas être converties, une activité d'espace réservé avec un texte d'avertissement est créée dans le Runbook.  Après avoir importé le Runbook converti dans Azure Automation, vous devez remplacer toutes ces activités par des activités valides qui effectuent la fonctionnalité requise.

Toutes les activités Orchestrator du module d’activités standard sont converties. Certaines activités standard d'Orchestrator ne sont pas dans ce module et ne sont pas converties. Par exemple, `Send Platform Event` n'a pas d'équivalent dans Azure Automation, dans la mesure où cet événement est spécifique à Orchestrator.

[Activités d'analyse](/previous-versions/system-center/system-center-2012-R2/hh403827(v=sc.12)) ne sont pas converties, dans la mesure où elles n'ont pas d'équivalent dans Azure Automation. La seule exception concerne les activités de supervision des packs d’intégration convertis qui sont converties en activités d’espace réservé.

Les activités d’un pack d’intégration converti sont converties si vous fournissez le chemin du module d’intégration avec le paramètre `modules`. Pour les packs d’intégration de System Center, vous pouvez utiliser les modules d’intégration de System Center Orchestrator.

### <a name="manage-orchestrator-resources"></a>Gérer les ressources Orchestrator

Le convertisseur de Runbooks convertit uniquement les Runbooks, pas les autres ressources Orchestrator telles que les compteurs, les variables ou les connexions.  Les compteurs ne sont pas pris en charge dans Azure Automation.  Les variables et les connexions sont pris en charge, mais vous devez les créer manuellement. Les fichiers journaux vous informent lorsque le runbook a besoin de ces ressources et spécifie les ressources correspondantes que vous devez créer dans Azure Automation pour que le runbook converti fonctionne correctement.

Par exemple, un Runbook peut utiliser une variable pour remplir une valeur particulière dans une activité.  Le runbook converti convertit l’activité et spécifie dans Azure Automation une ressource de variable portant le même nom que la variable Orchestrator. Ceci sera consigné dans le fichier **Runbook Converter - Summary.log** créé après la conversion. Vous devez créer manuellement cette ressource de variable dans Azure Automation avant d’utiliser le runbook.

### <a name="work-with-orchestrator-input-parameters"></a>Utiliser des paramètres d’entrée Orchestrator

Les runbooks d'Orchestrator acceptent les paramètres d'entrée avec l'activité `Initialize Data`.  Si le Runbook en cours de conversion inclut cette activité, un [Paramètre d'entrée](automation-graphical-authoring-intro.md#handle-runbook-input) dans le Runbook d'Azure Automation est créé pour chaque paramètre dans l'activité.  Une activité de [Contrôle de script de flux de travail](automation-graphical-authoring-intro.md#use-activities) est créée dans le Runbook converti qui extrait et renvoie chaque paramètre. Toutes les activités dans le Runbook qui utilisent un paramètre d'entrée font référence à la sortie de cette activité.

Cette stratégie est utilisée parce qu'elle reflète le mieux la fonctionnalité dans le Runbook Orchestrator.  Les activités dans de nouveaux Runbooks graphiques doivent faire directement référence aux paramètres d'entrée utilisant une source de données d'entrée Runbook.

### <a name="invoke-runbook-activity"></a>Appeler l'activité Runbook

Les runbooks d'Orchestrator démarrent d'autres runbooks avec l'activité `Invoke Runbook`. Si le runbook en cours de conversion inclut cette activité et que l'option `Wait for completion` est définie, une activité de runbook est créée pour lui dans le runbook converti.  Si l’option `Wait for completion` n'est pas définie, une activité de script de flux de travail utilisant [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook?view=azps-3.7.0) pour démarrer le runbook est créée. Après avoir importé le Runbook converti dans Azure Automation, vous devez modifier cette activité avec les informations spécifiées dans l'activité.

## <a name="create-orchestrator-assets"></a>Créer des ressources Orchestrator

Le convertisseur de runbooks ne convertit pas les ressources Orchestrator. Vous devez créer manuellement les ressources Orchestrator nécessaires dans Azure Automation.

## <a name="configure-hybrid-runbook-worker"></a>Configurer un runbook Worker hybride

Orchestrator stocke les runbooks sur un serveur de base de données et les exécute sur des serveurs de runbooks, les deux se trouvant dans votre centre de données local. Dans Azure Automation, les runbooks sont stockés dans le cloud Azure et peuvent s’exécuter dans votre centre de données local à l’aide d’un [Runbook Worker hybride](automation-hybrid-runbook-worker.md). Vous devez configurer un Worker pour qu’il exécute vos runbooks convertis à partir d’Orchestrator, puisqu’ils sont conçus pour s’exécuter sur des serveurs locaux et accéder aux ressources locales.

## <a name="related-articles"></a>Articles connexes

* Pour plus d’informations sur Orchestrator, consultez [System Center 2012 - Orchestrator](/previous-versions/system-center/system-center-2012-R2/hh237242(v=sc.12)).
* Pour plus d’informations sur l’automatisation de la gestion des services, consultez [Service Management Automation](/previous-versions/system-center/system-center-2012-R2/dn469260(v=sc.12)).
* Pour plus d’informations sur les activités Orchestrator, consultez [Activités standard Orchestrator](/previous-versions/system-center/system-center-2012-R2/hh403832(v=sc.12)).
* Pour obtenir le kit de migration Orchestrator, consultez [Télécharger le kit de migration System Center Orchestrator](https://www.microsoft.com/download/details.aspx?id=47323).
* Pour obtenir une présentation des runbooks Worker hybrides Azure Automation, consultez [Vue d’ensemble des runbooks Worker hybrides](automation-hybrid-runbook-worker.md).
