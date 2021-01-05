---
title: Configurer un déploiement continu d’Azure Automation avec Chocolatey
description: Cet article explique comment configurer un déploiement continu avec State Configuration et le gestionnaire de package Chocolatey.
services: automation
ms.subservice: dsc
ms.date: 08/08/2018
ms.topic: conceptual
ms.custom: references_regions
ms.openlocfilehash: 87504625c298c4fb858ff90430d707081e87cd5a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86186569"
---
# <a name="set-up-continuous-deployment-with-chocolatey"></a>Configurer un déploiement continu avec Chocolatey

L’univers des opérations de développement offre de nombreux outils conçus pour aider les développeurs à franchir plus facilement différents stades dans le pipeline de l’intégration continue. Le nouveau service Azure Automation [State Configuration](automation-dsc-overview.md) vient aujourd’hui enrichir la liste des options disponibles pour les équipes DevOps. 

Azure Automation est un service géré dans Microsoft Azure qui vous permet d’automatiser différentes tâches à l’aide de Runbooks, de nœuds et de ressources partagées, notamment des informations d'identification, des planifications et des variables globales. Azure Automation State Configuration étend cette fonctionnalité d’automatisation pour intégrer les outils PowerShell Desired State Configuration (DSC). En voici une excellente [présentation](automation-dsc-overview.md).

Cet article explique comment configurer le déploiement continu (CD) pour un ordinateur Windows. Vous pouvez facilement étendre la technique afin d’inclure autant d’ordinateurs Windows que vous le souhaitez dans le rôle, par exemple un site Web, puis dans des rôles supplémentaires.

![Déploiement continu pour machines virtuelles IaaS](./media/automation-dsc-cd-chocolatey/cdforiaasvm.png)

## <a name="at-a-high-level"></a>À un niveau élevé

Il y aurait beaucoup à dire sur ce sujet, mais il est heureusement possible de décomposer toutes ces informations en deux grands processus :

- Écrire du code et le tester, puis créer et publication des packages d’installation pour les versions majeures et mineures du système.
- Créer et gérer des machines virtuelles qui installent et exécutent le code dans les packages.  

Une fois ces deux processus en place, vous pouvez facilement mettre automatiquement à jour le package sur vos machines virtuelles à mesure que de nouvelles versions sont créées et déployées.

## <a name="component-overview"></a>Vue d’ensemble des composants

S’ils sont communément employés dans l’univers Linux, les gestionnaires de packages tels que [apt-get](https://en.wikipedia.org/wiki/Advanced_Packaging_Tool) demeurent assez méconnus dans le monde de Windows.
[Chocolatey](https://chocolatey.org/) fait partie de ces packages. Pour mieux le découvrir, je vous invite à consulter le [blog](https://www.hanselman.com/blog/IsTheWindowsUserReadyForAptget.aspx) de Scott Hanselman qui lui est dédié. En résumé, Chocolatey vous permet, à l’aide de la ligne de commande, d’installer des packages à partir d’un référentiel central dans un système d’exploitation Windows. Vous pouvez créer et gérer votre propre référentiel et Chocolatey peut installer des packages à partir de tous les référentiels que vous désignez, quel qu’en soit le nombre.

[PowerShell DSC](/powershell/scripting/dsc/overview/overview) est un outil PowerShell qui vous permet de déclarer la configuration que vous souhaitez affecter à une machine. Par exemple, si vous souhaitez installer Chocolatey, IIS, ouvrir le port 80 et installer la version 1.0.0 de votre site Web, le gestionnaire de configuration locale DSC Local Configuration Manager (LCM) implémente cette configuration. Un serveur Pull DSC contient un référentiel des configurations de vos machines. Le LCM résidant sur chaque ordinateur vérifie régulièrement si sa configuration correspond à la configuration enregistrée. Il peut signaler l’état ou tenter de réaligner la configuration de la machine sur la configuration enregistrée. Vous pouvez modifier la configuration enregistrée sur le serveur Pull de manière à aligner la configuration d’une machine ou d’un ensemble de machines sur la configuration modifiée.

Une ressource DSC est un module de code qui présente des fonctionnalités spécifiques, telles que la gestion de mise en réseau, Active Directory ou SQL Server. La ressource DSC Chocolatey sait comment accéder à un serveur NuGet (entre autres), télécharger les packages, installer les packages et ainsi de suite. Il existe de nombreuses autres ressources DSC dans la [PowerShell Gallery](https://www.powershellgallery.com/packages?q=dsc+resources&prerelease=&sortOrder=package-title). Vous installez ces modules sur le serveur Pull Azure Automation State Configuration afin qu’ils puissent être utilisés par vos configurations.

Les modèles Resource Manager offrent un moyen de générer votre infrastructure par des déclarations (réseaux, sous-réseaux, sécurité du réseau et routage, équilibreurs de charge, NIC, machines virtuelles, etc.). Voici un [article](../azure-resource-manager/management/deployment-models.md) qui compare le modèle de déploiement Resource Manager (déclaratif) au modèle de déploiement Azure Service Management (ASM ou classique) (impératif). Cet article inclut une discussion sur les fournisseurs de ressources de base : calcul, stockage et réseau.

Un modèle Resource Manager se distingue par sa capacité à installer une extension de machine virtuelle dans la machine virtuelle à mesure de son approvisionnement. Une extension de machine virtuelle possède des fonctionnalités spécifiques, telles que l’exécution d’un script personnalisé, l’installation d’un logiciel antivirus et l’exécution d’un script de configuration DSC. Il existe de nombreux autres types d’extensions de machine virtuelle.

## <a name="quick-trip-around-the-diagram"></a>Tour d’horizon rapide du diagramme

Commençons par le point de départ : vous écrivez votre code, vous le générez, vous le testez, puis vous créez un package d’installation. Chocolatey peut gérer différents types de packages d’installation, tels que MSI, MSU ou ZIP. Et vous disposez de toute la puissance de PowerShell pour effectuer l’installation si les fonctionnalités natives de Chocolatey ne sont pas adaptées. Placez le package dans un endroit facilement accessible, tel qu’un référentiel de packages. Cet exemple utilise un dossier public dans un compte de stockage d’objets blob Azure, mais vous pouvez utiliser n’importe quel emplacement. Chocolatey fonctionne en mode natif avec des serveurs NuGet et quelques autres serveurs pour la gestion des métadonnées de packages. [Cet article](https://github.com/chocolatey/choco/wiki/How-To-Host-Feed) en décrit les options. Cet exemple utilise NuGet. Un Nuspec désigne les métadonnées concernant vos packages. Les informations Nuspec sont compilées dans un NuPkg et stockées sur un serveur NuGet. Lorsque votre configuration demande un package par son nom et fait référence à un serveur NuGet, la ressource DSC Chocolatey sur la machine virtuelle extrait le package et l’installe. Vous pouvez également demander une version spécifique d’un package.

Dans la partie inférieure gauche de l’image, vous verrez un modèle Azure Resource Manager. Dans cet exemple, l’extension de machine virtuelle enregistre la machine virtuelle auprès du serveur d’extension Automation State Configuration en tant que nœud. La configuration est stockée dans le serveur d’extraction à deux reprises : une fois en texte brut et une fois compilée sous forme de fichier MOF. Dans le portail Azure, la structure MOF est une configuration de nœud, et non une simple configuration. Il s’agit de l’artefact qui est associé à un nœud afin que le nœud connaisse sa configuration. Les détails ci-dessous montrent comment affecter la configuration de nœud au nœud.

Créer un Nuspec, le compiler et le stocker dans un serveur NuGet n’a rien de compliqué. Et vous gérez déjà des machines virtuelles. 

L’étape suivante du déploiement continu suppose de configurer le serveur d’extraction une fois, d’y enregistrer vos nœuds une fois et d’y créer et stocker la configuration initiale. À mesure que les packages sont mis à niveau et déployés dans le référentiel, il vous suffit d’actualiser la configuration et la configuration des nœuds dans le serveur d’extraction, selon vos besoins.

Si vous ne commencez pas par un modèle Resource Manager, il n’y a aucun problème. Des commandes PowerShell vous aident à enregistrer vos machines virtuelles auprès du serveur d’extraction. Pour plus d’informations, consultez [Intégration des machines pour la gestion avec Azure Automation State Configuration](automation-dsc-onboarding.md).

## <a name="about-the-usage-example"></a>À propos de l’exemple d’utilisation

Dans l’exemple d’utilisation de cet article, nous partons d’une machine virtuelle provenant d’une image Windows Server 2012 R2 générique de la galerie Azure. Vous pouvez démarrer à partir de n’importe quelle image stockée, puis la modifier avec la configuration DSC.
Toutefois, la modification d’une configuration intégrée à une image est beaucoup plus difficile que la mise à jour dynamique de la configuration à l’aide de DSC.

Vous n’êtes pas obligé d’utiliser un modèle Resource Manager ou l’extension de machine virtuelle pour utiliser cette technique avec vos machines virtuelles. De même, il n’est pas nécessaire que vos machines virtuelles se trouvent sur Azure au niveau de la gestion des CD. Vous devez simplement faire en sorte d’installer Chocolatey et de configurer LCM sur la machine virtuelle afin qu’elle parvienne à localiser le serveur d’extraction.

Lorsque vous mettez à jour un package sur une machine virtuelle en production, vous devez sortir cette machine virtuelle pendant l’installation de la mise à jour. La procédure varie considérablement. Par exemple, avec une machine virtuelle placée derrière un équilibreur de charge Azure, vous pouvez ajouter une sonde personnalisée. Lors de la mise à jour de la machine virtuelle, faites en sorte que le point de terminaison de la sonde retourne un code 400. La solution nécessaire pour forcer cette modification peut se trouver à l’intérieur même de votre configuration, tout comme la solution qui lui permettra de retourner un code 200 une fois la mise à jour terminée.

La source complète de cet exemple se trouve dans ce [projet Visual Studio](https://github.com/sebastus/ARM/tree/master/CDIaaSVM) sur GitHub.

## <a name="step-1-set-up-the-pull-server-and-automation-account"></a>Étape 1 : Configurer le serveur d’extraction et le compte Automation

Ouvrez une ligne de commande PowerShell (`Connect-AzAccount`) authentifiée : (le processus peut prendre quelques minutes en attendant que le serveur d’extraction soit configuré)

```azurepowershell-interactive
New-AzResourceGroup –Name MY-AUTOMATION-RG –Location MY-RG-LOCATION-IN-QUOTES
New-AzAutomationAccount –ResourceGroupName MY-AUTOMATION-RG –Location MY-RG-LOCATION-IN-QUOTES –Name MY-AUTOMATION-ACCOUNT
```

Vous pouvez installer votre compte Automation dans l’une des régions suivantes (également appelées « emplacements ») : USA Est 2, USA Centre Sud, US Gov Virginie, Europe Ouest, Asie Sud-Est, Japon Est, Inde Centre et Australie Sud-Est, Canada Centre, Europe Nord.

## <a name="step-2-make-vm-extension-tweaks-to-the-resource-manager-template"></a>Étape 2 : Ajuster l’extension de machine virtuelle au modèle Resource Manager

Voir les détails de l’enregistrement d’une machine virtuelle (à l’aide de l’extension PowerShell DSC VM) fournis dans ce [modèle de démarrage rapide d’Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/dsc-extension-azure-automation-pullserver).
Cette étape consiste à inscrire votre nouvelle machine virtuelle auprès du serveur d’extraction dans la liste des nœuds State Configuration. Une partie de cette inscription consiste à spécifier la configuration du nœud à appliquer au nœud. Comme cette configuration de nœud n’a pas encore besoin d’exister déjà sur le serveur d’extraction, cette opération peut être effectuée pour la première fois à l’étape 4. Mais ici, à l'étape 2, vous devez avoir choisi le nom du nœud et celui de la configuration. Dans cet exemple d'utilisation, le nœud est ’isvbox’ et la configuration est ’ISVBoxConfig’. Le nom de la configuration de nœud (à préciser dans DeploymentTemplate.json) est donc ’ISVBoxConfig.isvbox’.

## <a name="step-3-add-required-dsc-resources-to-the-pull-server"></a>Étape 3 : Ajouter des ressources DSC nécessaires au serveur Pull

La PowerShell Gallery est conçue pour installer les ressources DSC dans votre compte Azure Automation.
Accédez à la ressource que vous souhaitez ajouter et cliquez sur le bouton « Deploy to Azure Automation » (Déployer vers Azure Automation).

![Exemple de la PowerShell Gallery](./media/automation-dsc-cd-chocolatey/xNetworking.PNG)

Une autre technique récemment ajoutée au portail Azure vous permet d’extraire de nouveaux modules ou de mettre à jour des modules existants. Cliquez sur la ressource Compte Automation, sur la vignette Ressources et enfin sur la vignette Modules. L’icône Parcourir la galerie vous permet de consulter la liste des modules dans la galerie, d’étudier le tout plus en détail et enfin d’importer dans votre compte Automation. Il s’agit là d’un excellent moyen pour mettre à jour vos modules de temps à autre. De plus, la fonctionnalité d’importation vérifie les dépendances avec d’autres modules pour garantir que rien n’est désynchronisé.

Il existe également une approche manuelle, utilisée une seule fois par ressource, à moins que vous ne souhaitiez la mise à niveau ultérieurement. Pour plus d’informations sur la création de modules d’intégration PowerShell, consultez [Création de modules d’intégration pour Azure Automation](https://azure.microsoft.com/blog/authoring-integration-modules-for-azure-automation/).

>[!NOTE]
>La structure de dossier d’un module d’intégration PowerShell pour un ordinateur Windows est un peu différente de celle à laquelle s’attend Azure Automation. 

1. Installez [Windows Management Framework v5](https://aka.ms/wmf5latest) (inutile pour Windows 10).

2. Installer le module d’intégration.

    ```azurepowershell-interactive
    Install-Module –Name MODULE-NAME`    <—grabs the module from the PowerShell Gallery
    ```

3. Copiez le dossier du module de **c:\Program Files\WindowsPowerShell\Modules\MODULE-NAME** dans un dossier temporaire.

4. Supprimez les modèles et la documentation dans le dossier principal.

5. Compressez le dossier principal en attribuant au fichier ZIP le même nom que celui du dossier.

6. Placez le fichier ZIP dans un emplacement HTTP accessible, par exemple un stockage d’objets blob dans un compte de stockage Azure.

7. Exécutez la commande suivante :

    ```azurepowershell-interactive
    New-AzAutomationModule `
      -ResourceGroupName MY-AUTOMATION-RG -AutomationAccountName MY-AUTOMATION-ACCOUNT `
      -Name MODULE-NAME –ContentLinkUri 'https://STORAGE-URI/CONTAINERNAME/MODULE-NAME.zip'
    ```

L’exemple fourni implémente ces étapes pour cChoco et xNetworking. 

## <a name="step-4-add-the-node-configuration-to-the-pull-server"></a>Étape 4 : Ajouter la configuration de nœud au serveur Pull

L’importation et la compilation initiales de votre configuration dans le serveur Pull ne présentent aucune difficulté particulière. Toutes les importations ou compilations ultérieures de la même configuration auront exactement le même aspect. Chaque fois que vous mettez à jour votre package et que vous devez la mettre en production, vous devez effectuer cette étape après avoir vérifié que le fichier de configuration est correct et qu’il comporte la nouvelle version de votre package. Voici le fichier de configuration **ISVBoxConfig.ps1** :

```powershell
Configuration ISVBoxConfig
{
    Import-DscResource -ModuleName cChoco
    Import-DscResource -ModuleName xNetworking

    Node 'isvbox' {

        cChocoInstaller installChoco
        {
            InstallDir = 'C:\choco'
        }

        WindowsFeature installIIS
        {
            Ensure = 'Present'
            Name   = 'Web-Server'
        }

        xFirewall WebFirewallRule
        {
            Direction    = 'Inbound'
            Name         = 'Web-Server-TCP-In'
            DisplayName  = 'Web Server (TCP-In)'
            Description  = 'IIS allow incoming web site traffic.'
            Enabled       = 'True'
            Action       = 'Allow'
            Protocol     = 'TCP'
            LocalPort    = '80'
            Ensure       = 'Present'
        }

        cChocoPackageInstaller trivialWeb
        {
            Name      = 'trivialweb'
            Version   = '1.0.0'
            Source    = 'MY-NUGET-V2-SERVER-ADDRESS'
            DependsOn = '[cChocoInstaller]installChoco','[WindowsFeature]installIIS'
        }
    }
}
```

Voici le script **New-ConfigurationScript.ps1** (modifié pour utiliser le module Az) :

```powershell
Import-AzAutomationDscConfiguration `
    -ResourceGroupName MY-AUTOMATION-RG –AutomationAccountName MY-AUTOMATION-ACCOUNT `
    -SourcePath C:\temp\AzureAutomationDsc\ISVBoxConfig.ps1 `
    -Published –Force

$jobData = Start-AzAutomationDscCompilationJob `
    -ResourceGroupName MY-AUTOMATION-RG –AutomationAccountName MY-AUTOMATION-ACCOUNT `
    -ConfigurationName ISVBoxConfig

$compilationJobId = $jobData.Id

Get-AzAutomationDscCompilationJob `
    -ResourceGroupName MY-AUTOMATION-RG –AutomationAccountName MY-AUTOMATION-ACCOUNT `
    -Id $compilationJobId
```

Ces étapes génèrent une nouvelle configuration de nœud nommée **ISVBoxConfig.isvbox** placée sur le serveur d’extraction. Le nom de la configuration du nœud est créé en tant que `configurationName.nodeName`.

## <a name="step-5-create-and-maintain-package-metadata"></a>Étape 5 : Créer et maintenir les métadonnées de package

Pour chaque package que vous placez dans le référentiel de packages, vous avez besoin d’un Nuspec descriptif. Il doit être compilé et stocké sur votre serveur NuGet. Cette opération est décrite [ici](https://docs.nuget.org/create/creating-and-publishing-a-package). 

Vous pouvez utiliser **MyGet.org** comme serveur NuGet. Vous pouvez acheter ce service, mais il existe un SKU gratuit destiné aux débutants. Rendez-vous sur [NuGet](https://www.nuget.org/) pour obtenir des instructions sur l’installation de votre propre serveur NuGet pour vos packages privés.

## <a name="step-6-tie-it-all-together"></a>Étape 6 : Tout assembler

Chaque fois qu'une version passe l'assurance qualité et est approuvée pour le déploiement, le package est créé, nuspec et nupkg sont mis à jour et déployés sur le serveur NuGet. La configuration (étape 4) doit également être mise à jour pour correspondre au nouveau numéro de version. Elle doit être alors envoyée au serveur d’extraction puis compilée.

À ce stade, les machines virtuelles qui dépendent de cette configuration doivent extraire la mise à jour et l'installer. Chacune de ces mises à jour est simple et ne nécessite qu’une ou deux lignes PowerShell. Pour Azure DevOps, certaines d’entre elles sont encapsulées dans les tâches de génération et peuvent être chaînées dans une build. Cet [article](https://www.visualstudio.com/docs/alm-devops-feature-index#continuous-delivery) fournit plus de détails. Ce [référentiel GitHub](https://github.com/Microsoft/vso-agent-tasks) détaille les tâches de génération disponibles.

## <a name="related-articles"></a>Articles connexes
* [Vue d’ensemble d’Azure Automation DSC](automation-dsc-overview.md)
* [Gestion de machines avec Azure Automation DSC](automation-dsc-onboarding.md)

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble, consultez [Vue d’ensemble d’Azure Automation State Configuration](automation-dsc-overview.md).
- Pour commencer à utiliser cette fonctionnalité, consultez [Bien démarrer avec Azure Automation State Configuration](automation-dsc-getting-started.md).
- Pour découvrir comment compiler des configurations DSC pour pouvoir les affecter à des nœuds cibles, voir [Compiler des configurations DSC dans Azure Automation State Configuration](automation-dsc-compile.md).
- Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).
- Pour obtenir des informations sur les prix, consultez [Tarification d’Azure Automation State Configuration](https://azure.microsoft.com/pricing/details/automation/).
