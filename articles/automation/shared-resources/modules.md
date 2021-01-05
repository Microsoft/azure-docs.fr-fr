---
title: Gérer les modules dans Azure Automation
description: Cet article explique comment utiliser des modules PowerShell pour activer des cmdlets de runbooks et de ressources DSC dans des configurations DSC.
services: automation
ms.subservice: shared-capabilities
ms.date: 10/22/2020
ms.topic: conceptual
ms.openlocfilehash: c940ede63e2a467a29ae56308893d573925d0039
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92458147"
---
# <a name="manage-modules-in-azure-automation"></a>Gérer les modules dans Azure Automation

Azure Automation utilise un certain nombre de modules PowerShell pour activer des cmdlets de runbooks et de ressources DSC dans des configurations DSC. Les modules pris en charge sont les suivants :

* [Azure PowerShell Az.Automation](/powershell/azure/new-azureps-module-az)
* [Azure PowerShell AzureRM.Automation](/powershell/module/azurerm.automation/)
* Autres modules PowerShell
* Module `Orchestrator.AssetManagement.Cmdlets` interne
* Modules Python 2
* Modules personnalisés que vous créez

Lorsque vous créez un compte Automation, Azure Automation importe certains modules par défaut. Voir [Modules par défaut](#default-modules).

Quand Automation exécute des travaux de runbook et de compilation DSC, le service charge les modules dans des bacs à sable où les runbooks peuvent s’exécuter et les configurations DSC compiler. Automation place automatiquement toutes les ressources DSC des modules sur le serveur Pull DSC. Les machines peuvent extraire les ressources lorsqu’elles appliquent les configurations DSC.

>[!NOTE]
>Veillez à importer uniquement les modules dont vos runbooks et configurations DSC ont réellement besoin. Nous vous déconseillons d’importer le module Az racine. Il comprend de nombreux autres modules dont vous n’avez peut-être pas besoin, ce qui peut entraîner des problèmes de performances. Importez des modules individuels, tels que Az.Compute, à la place.

## <a name="default-modules"></a>Modules par défaut

Le tableau suivant répertorie les modules qu’Azure Automation importe par défaut lorsque vous créez votre compte Automation. Automation peut importer des versions plus récentes de ces modules. Toutefois, vous ne pouvez pas supprimer la version originale de votre compte Automation, même si vous supprimez une version plus récente. Notez que ces modules par défaut incluent plusieurs modules AzureRM. 

Automation n’importe pas automatiquement le module Az racine dans les comptes Automation nouveaux ou existants. Pour plus d’informations sur l’utilisation de ces modules, consultez [Migration vers les modules Az](#migrate-to-az-modules).

> [!NOTE]
> Nous vous déconseillons de modifier les modules et runbooks dans des comptes Automation utilisés pour le déploiement de la fonctionnalité [Start/Stop VMs during off-hours](../automation-solution-vm-management.md).

|Nom du module|Version|
|---|---|
| AuditPolicyDsc | 1.1.0.0 |
| Azure | 1.0.3 |
| Azure.Storage | 1.0.3 |
| AzureRM.Automation | 1.0.3 |
| AzureRM.Compute | 1.2.1 |
| AzureRM.Profile | 1.0.3 |
| AzureRM.Resources | 1.0.3 |
| AzureRM.Sql | 1.0.3 |
| AzureRM.Storage | 1.0.3 |
| ComputerManagementDsc | 5.0.0.0 |
| GPRegistryPolicyParser | 0.2 |
| Microsoft.PowerShell.Core | 0 |
| Microsoft.PowerShell.Diagnostics |  |
| Microsoft.PowerShell.Management |  |
| Microsoft.PowerShell.Security |  |
| Microsoft.PowerShell.Utility |  |
| Microsoft.WSMan.Management |  |
| Orchestrator.AssetManagement.Cmdlets | 1 |
| PSDscResources | 2.9.0.0 |
| SecurityPolicyDsc | 2.1.0.0 |
| StateConfigCompositeResources | 1 |
| xDSCDomainjoin | 1.1 |
| xPowerShellExecutionPolicy | 1.1.0.0 |
| xRemoteDesktopAdmin | 1.1.0.0 |

## <a name="az-modules"></a>Modules Az

Pour Az.Automation, la majorité des cmdlets portent les mêmes noms que les modules AzureRM, à ceci près que le préfixe `AzureRM` a été remplacé par `Az`. Pour obtenir la liste des modules Az qui ne suivent pas cette convention de nommage, consultez la [liste des exceptions](/powershell/azure/migrate-from-azurerm-to-az#update-cmdlets-modules-and-parameters).

## <a name="internal-cmdlets"></a>Applets de commande internes

Azure Automation prend en charge le module `Orchestrator.AssetManagement.Cmdlets` interne pour l’agent Log Analytics pour Windows, installé par défaut. Le tableau suivant définit les cmdlets internes. Ces cmdlets sont conçues pour être utilisées à la place des cmdlets Azure PowerShell pour interagir avec les ressources partagées. Elles peuvent récupérer des secrets à partir de variables chiffrées, d’informations d’identification et de connexions chiffrées.

>[!NOTE]
>Les cmdlets internes ne sont disponibles que lorsque vous exécutez des runbooks dans l’environnement de bac à sable (sandbox) Azure ou sur un Runbook Worker hybride Windows. 

|Nom|Description|
|---|---|
|Get-AutomationCertificate|`Get-AutomationCertificate [-Name] <string> [<CommonParameters>]`|
|Get-AutomationConnection|`Get-AutomationConnection [-Name] <string> [-DoNotDecrypt] [<CommonParameters>]` |
|Get-AutomationPSCredential|`Get-AutomationPSCredential [-Name] <string> [<CommonParameters>]` |
|Get-AutomationVariable|`Get-AutomationVariable [-Name] <string> [-DoNotDecrypt] [<CommonParameters>]`|
|Set-AutomationVariable|`Set-AutomationVariable [-Name] <string> -Value <Object> [<CommonParameters>]` |
|Start-AutomationRunbook|`Start-AutomationRunbook [-Name] <string> [-Parameters <IDictionary>] [-RunOn <string>] [-JobId <guid>] [<CommonParameters>]`|
|Wait-AutomationJob|`Wait-AutomationJob -Id <guid[]> [-TimeoutInMinutes <int>] [-DelayInSeconds <int>] [-OutputJobsTransitionedToRunning] [<CommonParameters>]`|

Notez que les cmdlets internes diffèrent au niveau des noms des cmdlets Az et AzureRM. Les noms de cmdlet internes ne contiennent pas de mots tels que `Azure` ou `Az`, mais utilisent le mot `Automation`. Nous vous recommandons de les utiliser plutôt que les cmdlets Az ou AzureRM lors de l’exécution de runbooks dans un bac à sable (sandbox) Azure ou sur un Runbook Worker hybride. Elles nécessitent moins de paramètres et s’exécutent dans le contexte de votre travail en cours d’exécution.

Utiliser des cmdlets Az ou AzureRM pour manipuler des ressources Azure Automation en dehors du contexte d’un runbook. 

## <a name="python-modules"></a>Modules Python

Vous pouvez créer un runbook Python 2 dans Azure Automation. Pour plus d’informations sur le module Python, consultez [Gérer des packages Python 2 dans Azure Automation](../python-packages.md).

## <a name="custom-modules"></a>Modules personnalisés

Azure Automation prend en charge les modules PowerShell personnalisés que vous créez pour les utiliser avec vos runbooks et vos configurations DSC. Un type de module personnalisé est un module d’intégration qui contient éventuellement un fichier de métadonnées pour définir les fonctionnalités personnalisées des cmdlets de module. Un exemple d’utilisation d’un module d’intégration est fourni dans la section [Ajouter un type de connexion](../automation-connections.md#add-a-connection-type).

Azure Automation peut importer un module personnalisé pour mettre ses cmdlets à disposition. En arrière-plan, le service stocke le module et l’utilise dans les bacs à sable Azure, tout comme il le fait avec d’autres modules.

## <a name="migrate-to-az-modules"></a>Migrer vers AZ modules

Cette section explique comment migrer vers les modules Az dans Automation. Pour plus d’informations, consultez [Migrer Azure PowerShell depuis AzureRM vers Az](/powershell/azure/migrate-from-azurerm-to-az).

Nous déconseillons l’utilisation de modules Az et AzureRM dans le même compte Automation. Lorsque vous êtes sûr de vouloir migrer d’AzureRM vers Az, il est préférable d’effectuer une migration complète. Automation réutilise souvent des bacs à sable (sandbox) dans le compte Automation pour réaliser des économies au moment du démarrage. Si vous n’effectuez pas de migration de module complète, vous pouvez démarrer un travail utilisant uniquement des modules AzureRM, puis en démarrer un autre utilisant uniquement des modules Az. Le bac à sable (sandbox) se bloque rapidement et vous recevez une erreur indiquant que les modules ne sont pas compatibles. Cette situation entraîne des blocages aléatoires pour une configuration ou un runbook particuliers.

>[!NOTE]
>Lorsque vous créez un compte Automation, même après migration vers les modules Az, Automation installe les modules AzureRM par défaut. Vous pouvez toujours mettre à jour les runbooks tutoriels avec les applets de commande AzureRM. Cependant, vous ne devez pas exécuter ces runbooks.

### <a name="test-your-runbooks-and-dsc-configurations-prior-to-module-migration"></a>Tester vos runbooks et configurations DSC avant la migration de modules

Avant de migrer vers les modules Az, veillez à tester consciencieusement l’ensemble des runbooks et configurations DSC d’un compte Automation distinct. 

### <a name="stop-and-unschedule-all-runbooks-that-use-azurerm-modules"></a>Arrêter et annuler la planification de tous les runbooks utilisant des modules AzureRM

Pour être certain de n’exécuter aucun runbook ou aucune configuration DSC qui utilise des modules AzureRM, arrêtez et annulez la planification de tous les runbooks et configurations concernés. Tout d’abord, veillez à examiner chaque runbook et configuration DSC et leurs planifications séparément pour vous assurer que vous pouvez replanifier l’élément ultérieurement, si nécessaire.

Lorsque vous êtes prêt à supprimer vos planifications, vous pouvez utiliser le portail Azure ou la cmdlet [Remove-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/remove-azurermautomationschedule). Voir [Supprimer une planification](schedules.md#remove-a-schedule).

### <a name="remove-azurerm-modules"></a>Supprimer des modules AzureRM

Il est possible de supprimer les modules AzureRM avant d’importer les modules Az. Toutefois, cela peut interrompre la synchronisation du contrôle de code source et provoquer l’échec des scripts qui sont toujours planifiés. Si vous décidez de supprimer les modules, consultez [Désinstaller AzureRM](/powershell/azure/migrate-from-azurerm-to-az#uninstall-azurerm).

### <a name="import-az-modules"></a>Importer des modules Az

Le fait d’importer un module Az dans votre compte Automation n’a pas pour effet d’importer automatiquement le module dans la session PowerShell utilisée par les runbooks. Les modules sont importés dans la session PowerShell, dans les situations suivantes :

* Quand un runbook appelle une cmdlet à partir d’un module.
* Quand un runbook importe le module explicitement avec la cmdlet [Import-Module](/powershell/module/microsoft.powershell.core/import-module).
* Quand un runbook importe un autre module dépendant.

Vous pouvez importer les modules Az dans le portail Azure. N’oubliez pas d’importer uniquement les modules Az dont vous avez besoin, et non l’intégralité du module Az.Automation. [Az.Accounts](https://www.powershellgallery.com/packages/Az.Accounts/1.1.0) étant une dépendance pour les autres modules Az, veillez à importer ce module avant tout autre.

1. À partir de votre compte Automation, sous **Ressources partagées** , sélectionnez **Modules**.
2. Sélectionnez **Parcourir la galerie**.  
3. Dans la barre de recherche, entrez le nom du module, par exemple, `Az.Accounts`.
4. Dans la page Module PowerShell, sélectionnez **Importer** pour importer le module dans votre compte Automation.

    ![Capture d’écran de l’importation de modules dans votre compte Automation](../media/modules/import-module.png)

Vous pouvez également effectuer ce processus d’importation via [PowerShell Gallery](https://www.powershellgallery.com) en recherchant le module à importer. Une fois le module trouvé, sélectionnez-le, puis choisissez l’onglet **Azure Automation**. Sélectionnez **Déployer sur Azure Automation**.

![Capture d’écran de l’importation de modules directement à partir de PowerShell Gallery](../media/modules/import-gallery.png)

### <a name="test-your-runbooks"></a>Tester vos runbooks

Une fois que vous avez importé les modules Az dans le compte Automation, vous pouvez commencer à modifier vos runbooks et configurations DSC pour qu’ils utilisent les nouveaux modules. Un moyen de tester la modification d’un runbook afin qu’il utilise les nouvelles applets de commande consiste à utiliser la commande `Enable-AzureRmAlias -Scope Process` au début du runbook. En ajoutant cette commande à votre runbook, le script peut s’exécuter sans modification.

## <a name="author-modules"></a>Créer des modules

Nous vous recommandons de tenir compte des points mentionnés dans cette section quand vous créez un module PowerShell personnalisé en vue de l’utiliser dans Azure Automation. Pour préparer votre module à l’importation, vous devez créer au moins un fichier .psd1, .psm1, ou un fichier **.dll** de modèle PowerShell portant le même nom que le dossier du module. Ensuite, vous compressez le dossier du module au format zip afin qu’Azure Automation puisse l’importer sous forme de fichier unique. Le package **.zip** doit porter le même nom que le dossier du module qu’il contient.

Pour en savoir plus sur la création d’un module PowerShell, consultez [Guide pratique pour écrire un module de script PowerShell](/powershell/scripting/developer/module/how-to-write-a-powershell-script-module).

### <a name="version-folder"></a>Dossier de version

Le contrôle de version de module PowerShell côte à côte vous permet d’utiliser plusieurs versions d’un module dans PowerShell. Cela peut être utile si vous avez d’anciens scripts qui ont été testés et fonctionnent uniquement avec une certaine version d’un module PowerShell, mais que d’autres scripts requièrent une version plus récente du même module PowerShell.

Pour construire des modules PowerShell afin qu’ils contiennent plusieurs versions, créez le dossier de module, puis créez un dossier dans ce dossier de module pour chaque version du module que vous souhaitez pouvoir utiliser. Dans l’exemple suivant, un module appelé *TestModule* fournit deux versions, 1.0.0 et 2.0.0.

```dos
TestModule
   1.0.0
   2.0.0
```

Dans chacun des dossiers de version, copiez vos fichiers .psm1, .psd1 PowerShell ou les fichiers **.dll** du module PowerShell qui constituent un module dans le dossier de version respectif. Compressez le dossier du module au format .zip afin qu’Azure Automation puisse l’importer sous forme de fichier .zip unique. Alors qu’Automation n’affiche que la version la plus récente du module importé, si le package du module contient des versions côte à côte du module, elles sont toutes disponibles pour une utilisation dans vos runbooks ou configurations DSC.  

Alors qu’Automation prend en charge les modules contenant des versions côte à côte dans le même package, il ne prend pas en charge l’utilisation de plusieurs versions d’un module dans les importations de package de module. Par exemple, vous importez **module A** , qui contient les versions 1 et 2 dans votre compte Automation. Plus tard, vous mettez à jour **module A** de manière à inclure les versions 3 et 4. Lorsque vous importez dans votre compte Automation, seules les versions 3 et 4 sont utilisables dans les runbooks ou les configurations DSC. Si toutes les versions (1, 2, 3 et 4) doivent être disponibles, le fichier. zip que vous importez doit contenir les versions 1, 2, 3 et 4.

Si vous envisagez d’utiliser différentes versions du même module entre les runbooks, vous devez toujours déclarer la version que vous souhaitez utiliser dans votre runbook à l’aide de la cmdlet `Import-Module` et inclure le paramètre `-RequiredVersion <version>`. Même si la version que vous souhaitez utiliser est la version la plus récente. Cela est dû au fait que les tâches de runbook peuvent s’exécuter dans le même bac à sable. Si le bac à sable a déjà chargé explicitement un module d’un certain numéro de version, étant donné qu’une tâche précédente de ce bac à sable a déclaré faire cela, les tâches futures de ce bac à sable ne chargent pas automatiquement la dernière version de ce module. En effet, une certaine version est déjà chargée dans le bac à sable.

Pour une ressource DSC, utilisez la commande suivante pour spécifier une version particulière :

```powershell
Import-DscResource -ModuleName <ModuleName> -ModuleVersion <version>
```

### <a name="help-information"></a>Informations d'aide

Incluez un résumé, une description et un URI d’aide pour chaque applet de commande dans votre module. Dans PowerShell, vous pouvez définir des informations d’aide pour des cmdlets en utilisant la cmdlet `Get-Help`. L’exemple suivant montre comment définir un résumé et un URI d’aide dans un fichier de module **.psm1**.

  ```powershell
  <#
       .SYNOPSIS
        Gets a Contoso User account
  #>
  function Get-ContosoUser {
  [CmdletBinding](DefaultParameterSetName='UseConnectionObject', `
  HelpUri='https://www.contoso.com/docs/information')]
  [OutputType([String])]
  param(
     [Parameter(ParameterSetName='UserAccount', Mandatory=true)]
     [ValidateNotNullOrEmpty()]
     [string]
     $UserName,

     [Parameter(ParameterSetName='UserAccount', Mandatory=true)]
     [ValidateNotNullOrEmpty()]
     [string]
     $Password,

     [Parameter(ParameterSetName='ConnectionObject', Mandatory=true)]
     [ValidateNotNullOrEmpty()]
     [Hashtable]
     $Connection
  )

  switch ($PSCmdlet.ParameterSetName) {
     "UserAccount" {
        $cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $UserName, $Password
        Connect-Contoso -Credential $cred
     }
     "ConnectionObject" {
        Connect-Contoso -Connection $Connection
    }
  }
  }
  ```

  Fournir ces informations permet d’afficher le texte d’aide via l’applet de commande `Get-Help` dans la console PowerShell. Ce texte s’affiche aussi sur le portail Azure.

  ![Capture d’écran de l’aide du module d’intégration](../media/modules/module-activity-description.png)

### <a name="connection-type"></a>Type de connexion

Si le module se connecte à un service externe, définissez un type de connexion à l’aide d’un [module d’intégration personnalisé](#custom-modules). Chaque cmdlet du module doit accepter une instance de ce type de connexion (objet de connexion) en tant que paramètre. Les utilisateurs mappent les paramètres de la ressource de connexion aux paramètres correspondants de l’applet de commande chaque fois qu’ils appellent une applet de commande.

![Utiliser une connexion personnalisée sur le portail Azure](../media/modules/connection-create-new.png)

L’exemple de runbook suivant utilise une ressource de connexion Contoso appelée `ContosoConnection` pour accéder aux ressources Contoso et retourner des données provenant du service externe. Dans cet exemple, les champs sont mappés aux propriétés `UserName` et `Password` d’un objet `PSCredential`, puis transmis à la cmdlet.

  ```powershell
  $contosoConnection = Get-AutomationConnection -Name 'ContosoConnection'

  $cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $contosoConnection.UserName, $contosoConnection.Password
  Connect-Contoso -Credential $cred
  }
  ```

Un moyen plus simple et plus efficace de se rapprocher de ce comportement consiste à transmettre directement l’objet de connexion à la cmdlet :

  ```powershell
  $contosoConnection = Get-AutomationConnection -Name 'ContosoConnection'

  Connect-Contoso -Connection $contosoConnection
  }
  ```

Vous pouvez activer un comportement similaire pour vos applets de commande en les autorisant à accepter un objet de connexion directement en tant que paramètre, au lieu d’accepter uniquement des champs de connexion en tant que paramètres. Généralement, vous voudrez disposer d’un jeu de paramètres pour chaque élément, pour qu’un utilisateur n’utilisant pas Azure Automation puisse appeler vos cmdlets sans créer de table de hachage agissant en tant qu’objet de connexion. Le jeu de paramètres `UserAccount` permet de transmettre les propriétés de champ de connexion. `ConnectionObject` permet de transmettre directement la connexion.

### <a name="output-type"></a>Type de sortie

Définissez le type de sortie pour toutes les applets de commande de votre module. La définition d’un type de sortie pour une cmdlet permet à IntelliSense (au moment de la conception) de déterminer les propriétés de sortie de la cmdlet lors de la création. Cette approche est particulièrement utile lors de la création de runbooks graphiques, pour laquelle les connaissances au moment de la conception sont essentielles pour que l’expérience utilisateur de votre module soit conviviale.

Ajoutez `[OutputType([<MyOutputType>])]` où `MyOutputType` est un type valide. Pour en savoir plus sur `OutputType`, consultez [À propos des fonctions OutputTypeAttribute](/powershell/module/microsoft.powershell.core/about/about_functions_outputtypeattribute). Le code suivant montre comment ajouter `OutputType` à une applet de commande :

  ```powershell
  function Get-ContosoUser {
  [OutputType([String])]
  param(
     [string]
     $Parameter1
  )
  # <script location here>
  }
  ```

  ![Capture d’écran d’un type de sortie de runbook graphique](../media/modules/runbook-graphical-module-output-type.png)

  Ce comportement ressemble à celui de la fonctionnalité « frappe au kilomètre » de la sortie d’une cmdlet dans l’environnement de service d’intégration PowerShell, sans avoir à exécuter celle-ci.

  ![Capture d’écran de POSH IntelliSense](../media/modules/automation-posh-ise-intellisense.png)

### <a name="cmdlet-state"></a>État de l’applet de commande

Rendez toutes les applets de commande de votre module sans état. Plusieurs tâches de Runbook peuvent s’exécuter en même temps dans le même domaine `AppDomain` et les mêmes processus et bac à sable. Si un état est partagé sur ces niveaux, les tâches peuvent s’affecter mutuellement. Ce comportement peut occasionner des problèmes intermittents et difficiles à diagnostiquer. Voici un exemple de ce qu’il ne faut PAS faire :

  ```powershell
  $globalNum = 0
  function Set-GlobalNum {
     param(
         [int] $num
     )

     $globalNum = $num
  }
  function Get-GlobalNumTimesTwo {
     $output = $globalNum * 2

     $output
  }
  ```

### <a name="module-dependency"></a>Dépendance de module

Assurez-vous que le module est entièrement contenu dans un package qui peut être copié à l’aide de xcopy. Les modules Automation sont distribués aux bacs à sable Automation quand des runbooks s’exécutent. Les modules doivent fonctionner indépendamment de l’hôte qui les exécute.

Vous devez être en mesure de compresser un package de module pour le déplacer et le faire fonctionner normalement lors de son importation dans l’environnement PowerShell d’un autre hôte. Pour cela, assurez-vous que le module ne dépend d’aucun fichier situé en dehors du dossier du module compressé lors de l’importation du module dans Azure Automation.

Votre module ne doit dépendre d’aucun paramètre de registre unique sur un hôte. par exemple, des paramètres définis lors de l’installation d’un produit.

### <a name="module-file-paths"></a>Chemins d’accès aux fichiers de module

Assurez-vous que les chemins d'accès de tous les fichiers du module comportent moins de 140 caractères. Tout chemin d’accès d’une longueur supérieure à 140 caractères occasionne des problèmes lors de l’importation de runbooks. Automation ne peut pas importer un fichier dont la taille de chemin dépasse 140 caractères dans la session PowerShell avec `Import-Module`.

## <a name="import-modules"></a>Modules d’importation

Cette section décrit plusieurs façons d’importer un module dans votre compte Automation.

### <a name="import-modules-in-the-azure-portal"></a>Importer des modules dans le portail Azure

Pour importer un module dans le portail Azure :

1. Accédez à votre compte Automation.
2. Sous **Ressources partagées** , sélectionnez **Modules**.
3. Sélectionnez **Ajouter un module**.
4. Sélectionnez le fichier **.zip** qui contient votre module.
5. Sélectionnez **OK** pour démarrer le processus d’importation.

### <a name="import-modules-by-using-powershell"></a>Importer des modules à l’aide de PowerShell

Vous pouvez utiliser la cmdlet [New-AzAutomationModule](/powershell/module/az.automation/new-azautomationmodule) pour importer un module dans votre compte Automation. L’applet de commande prend l’URL d’un package .zip de module.

```azurepowershell-interactive
New-AzAutomationModule -Name <ModuleName> -ContentLinkUri <ModuleUri> -ResourceGroupName <ResourceGroupName> -AutomationAccountName <AutomationAccountName>
```

Vous pouvez également utiliser la même cmdlet pour importer un module directement à partir de PowerShell Gallery. Veillez à récupérer `ModuleName` et `ModuleVersion` dans [PowerShell Gallery](https://www.powershellgallery.com).

```azurepowershell-interactive
$moduleName = <ModuleName>
$moduleVersion = <ModuleVersion>
New-AzAutomationModule -AutomationAccountName <AutomationAccountName> -ResourceGroupName <ResourceGroupName> -Name $moduleName -ContentLinkUri "https://www.powershellgallery.com/api/v2/package/$moduleName/$moduleVersion"
```

### <a name="import-modules-from-the-powershell-gallery"></a>Importer des modules à partir de PowerShell Gallery

Vous pouvez importer des modules [PowerShell Gallery](https://www.powershellgallery.com) directement à partir de la galerie ou à partir de votre compte Automation.

Pour importer un module directement à partir de PowerShell Gallery :

1. Accédez à https://www.powershellgallery.com et recherchez le module à importer.
2. Dans **Options d’installation** , sous l’onglet **Azure Automation** , sélectionnez **Déployer sur Azure Automation**. Cette action ouvre le Portail Azure. 
3. Dans la page Importer, sélectionnez votre compte Automation, puis sélectionnez **OK**.

![Capture d’écran du module d’importation de PowerShell Gallery](../media/modules/powershell-gallery.png)

Pour importer un module PowerShell Gallery directement à partir de votre compte Automation :

1. Sous **Ressources partagées** , sélectionnez **Modules**. 
2. Sélectionnez **parcourir la Galerie** , puis recherchez un module dans la Galerie. 
3. Sélectionnez le module à importer, puis **Importer**. 
4. Sélectionnez **OK** pour démarrer le processus d’importation.

![Capture d’écran de l’importation d’un module PowerShell Gallery à partir du portail Azure](../media/modules/gallery-azure-portal.png)

## <a name="delete-modules"></a>Supprimer des modules

Si vous rencontrez des problèmes avec un module ou si vous avez besoin de revenir à une version précédente d’un module, vous pouvez le supprimer de votre compte Automation. Vous ne pouvez pas supprimer les versions d’origine des [modules par défaut](#default-modules) qui sont importés au moment de créer un compte Automation. Si le module à supprimer est une version plus récente de l’un des [modules par défaut](#default-modules), la version installée avec votre compte Automation est restaurée. Sinon, tout module que vous supprimez de votre compte Automation est bel et bien supprimé.

### <a name="delete-modules-in-the-azure-portal"></a>Supprimer des modules dans le portail Azure

Pour supprimer un module dans le portail Azure :

1. Accédez à votre compte Automation. Sous **Ressources partagées** , sélectionnez **Modules**.
2. Sélectionnez le module que vous souhaitez supprimer.
3. Dans la page Module, sélectionnez **Supprimer**. Si ce module fait partie des [modules par défaut](#default-modules), la version présente au moment où le compte Automation a été créé est restaurée.

### <a name="delete-modules-by-using-powershell"></a>Supprimer des modules à l’aide de PowerShell

Pour supprimer un module via PowerShell, exécutez la commande suivante :

```azurepowershell-interactive
Remove-AzAutomationModule -Name <moduleName> -AutomationAccountName <automationAccountName> -ResourceGroupName <resourceGroupName>
```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’utilisation de modules Azure PowerShell, consultez l’article [Prise en main d’Azure PowerShell](/powershell/azure/get-started-azureps).

* Pour en savoir plus sur la création de modules PowerShell, consultez [Écriture d’un module Windows PowerShell](/powershell/scripting/developer/module/writing-a-windows-powershell-module).
