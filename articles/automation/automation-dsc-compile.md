---
title: Compiler des configurations DSC dans Azure Automation State Configuration
description: Cet article explique comment compiler des configurations d’état souhaité (DSC) pour Azure Automation.
services: automation
ms.subservice: dsc
ms.date: 04/06/2020
ms.topic: conceptual
ms.openlocfilehash: 3bb42886c653afbdf8975b532bd2e1e1c3c63ce9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86186535"
---
# <a name="compile-dsc-configurations-in-azure-automation-state-configuration"></a>Compiler des configurations DSC dans Azure Automation State Configuration

Dans Azure Automation State Configuration, vous pouvez compiler des configurations DSC de plusieurs manières :

- Service de compilation Azure State Configuration
  - Méthode pour débutant avec interface utilisateur interactive
   - Suivi aisé de l’état des tâches

- Windows PowerShell
  - Appel à partir de Windows PowerShell sur la station de travail locale ou le service de build
  - Intégration avec le pipeline de test de développement
  - Fourniture de valeurs de paramètres complexes
  - Utilisation de données de nœud et non-nœud à l’échelle
  - Amélioration significative des performances

Vous pouvez également utiliser des modèles Azure Resource Manager avec l’extension Desired State Configuration (DSC) Azure pour envoyer (push) des configurations à vos machines virtuelles Azure. L’extension DSC d’Azure utilise l’infrastructure de l’agent Azure VM pour fournir, mettre en œuvre et créer des rapports sur les configurations DSC sur des machines virtuelles Azure. Pour plus d’informations sur la compilation à l’aide de modèles Azure Resource Manager, consultez [Extension Desired State Configuration avec des modèles Azure Resource Manager](../virtual-machines/extensions/dsc-template.md#details). 

## <a name="compile-a-dsc-configuration-in-azure-state-configuration"></a>Compiler une configuration DSC dans Azure State Configuration

### <a name="portal"></a>Portail

1. Dans votre compte Automation, cliquez sur **Configuration d’état (DSC)** .
1. Cliquez sur l’onglet **Configurations**, puis cliquez sur le nom de la configuration à compiler.
1. Cliquez sur **Compiler**.
1. Si la configuration n’a aucun paramètre, vous devez confirmer que vous souhaitez la compiler. Si la configuration a des paramètres, le panneau **Compiler la configuration** s’ouvre et vous permet de fournir la valeur des paramètres.
1. La page Travail de compilation pour vous permettre de suivre l’état du travail de compilation. Vous pouvez également utiliser cette page pour suivre les configurations de nœuds (documents de configuration MOF) qui sont placées sur le serveur Pull d’Azure Automation State Configuration.

### <a name="azure-powershell"></a>Azure PowerShell

Vous pouvez utiliser la commande [Start-AzAutomationDscCompilationJob](/powershell/module/az.automation/start-azautomationdsccompilationjob) pour commencer la compilation avec Windows PowerShell. L’exemple de code suivant commence la compilation d’une configuration DSC appelée **SampleConfig**.

```powershell
Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'SampleConfig'
```

`Start-AzAutomationDscCompilationJob` renvoie un objet de tâche de compilation que vous pouvez utiliser pour suivre son état. Vous pouvez ensuite utiliser cet objet de travail de compilation avec [Get-AzAutomationDscCompilationJob](/powershell/module/az.automation/get-azautomationdsccompilationjob) pour déterminer l’état du travail de compilation, et avec [Get-AzAutomationDscCompilationJobOutput](/powershell/module/az.automation/get-azautomationdscconfiguration) pour afficher ses flux (sortie). L’exemple suivant démarre la compilation de la configuration SampleConfig, attend qu’elle soit terminée, puis affiche ses flux.

```powershell
$CompilationJob = Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'SampleConfig'

while($null -eq $CompilationJob.EndTime -and $null -eq $CompilationJob.Exception)
{
    $CompilationJob = $CompilationJob | Get-AzAutomationDscCompilationJob
    Start-Sleep -Seconds 3
}

$CompilationJob | Get-AzAutomationDscCompilationJobOutput –Stream Any
```

### <a name="declare-basic-parameters"></a>Déclarer les paramètres de base

La déclaration de paramètres dans les configurations DSC, y compris les types de paramètres et les propriétés, fonctionne de la même manière que pour les runbooks Azure Automation. Consultez [Démarrage d’un runbook dans Azure Automation](./start-runbooks.md) pour en savoir plus sur les paramètres de runbooks.

L’exemple suivant utilise les paramètres `FeatureName` et `IsPresent` pour déterminer les valeurs des propriétés dans la configuration du nœud **ParametersExample.sample**, générée pendant la compilation.

```powershell
Configuration ParametersExample
{
    param(
        [Parameter(Mandatory=$true)]
        [string] $FeatureName,

        [Parameter(Mandatory=$true)]
        [boolean] $IsPresent
    )

    $EnsureString = 'Present'
    if($IsPresent -eq $false)
    {
        $EnsureString = 'Absent'
    }

    Node 'sample'
    {
        WindowsFeature ($FeatureName + 'Feature')
        {
            Ensure = $EnsureString
            Name   = $FeatureName
        }
    }
}
```

Vous pouvez compiler les configurations DSC qui utilisent des paramètres de base dans le portail Azure Automation State Configuration ou à l’aide d’Azure PowerShell.

#### <a name="portal"></a>Portail

Dans le portail, vous pouvez entrer des valeurs de paramètre après avoir cliqué sur **Compiler**.

![Paramètres de compilation de configuration](./media/automation-dsc-compile/DSC_compiling_1.png)

#### <a name="azure-powershell"></a>Azure PowerShell

PowerShell a besoin des paramètres d’une [table de hachage](/powershell/module/microsoft.powershell.core/about/about_hash_tables) où la clé correspond au nom de paramètre et où la valeur est égale à la valeur du paramètre.

```powershell
$Parameters = @{
    'FeatureName' = 'Web-Server'
    'IsPresent' = $False
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'ParametersExample' -Parameters $Parameters
```

Pour plus d’informations sur la transmission d’objets `PSCredential` en tant que paramètres, consultez [Ressources d’informations d’identification](#credential-assets).

### <a name="compile-configurations-containing-composite-resources-in-azure-automation"></a>Compilez les configurations contenant des ressources composites dans Azure Automation

La fonctionnalité **Ressources composites** vous permet d’utiliser des configurations DSC en tant que ressources imbriquées à l’intérieur d’une configuration. Cette fonctionnalité permet d’appliquer plusieurs configurations à une même ressource. Consultez [Ressources composites : utilisation d’une configuration DSC en tant que ressource](/powershell/scripting/dsc/resources/authoringresourcecomposite) pour en savoir plus sur les ressources composites.

> [!NOTE]
> Pour que les configurations contenant des ressources composites se compilent correctement, vous devez d’abord importer dans Azure Automation toutes les ressources DSC sur lesquelles les ressources composites s’appuient. L’ajout d’une ressource composite DSC n’est pas différent de l’ajout d’un module PowerShell à Azure Automation. Le processus est documenté dans [Gérer les modules dans Azure Automation](./shared-resources/modules.md).

### <a name="manage-configurationdata-when-compiling-configurations-in-azure-automation"></a>Gérer ConfigurationData lors de la compilation de configurations dans Azure Automation

`ConfigurationData` est un paramètre DSC intégré qui vous permet de séparer la configuration structurelle de toute configuration spécifique de l’environnement lors de l’utilisation de PowerShell DSC. Pour plus d’informations, consultez [Separating "What" from "Where" in PowerShell DSC](https://devblogs.microsoft.com/powershell/separating-what-from-where-in-powershell-dsc/).

> [!NOTE]
> Lors de la compilation dans Azure Automation State Configuration, vous pouvez utiliser `ConfigurationData` dans Azure PowerShell, mais pas dans le portail Azure.

L’exemple de configuration DSC suivant utilise `ConfigurationData` via les mots clés `$ConfigurationData` et `$AllNodes`. Vous avez également besoin du [module xWebAdministration](https://www.powershellgallery.com/packages/xWebAdministration/) pour cet exemple.

```powershell
Configuration ConfigurationDataSample
{
    Import-DscResource -ModuleName xWebAdministration -Name MSFT_xWebsite

    Write-Verbose $ConfigurationData.NonNodeData.SomeMessage

    Node $AllNodes.Where{$_.Role -eq 'WebServer'}.NodeName
    {
        xWebsite Site
        {
            Name         = $Node.SiteName
            PhysicalPath = $Node.SiteContents
            Ensure       = 'Present'
        }
    }
}
```

Vous pouvez compiler la configuration DSC précédente avec Windows PowerShell. Le script suivant ajoute deux configurations de nœud au serveur Pull Azure Automation State Configuration : **ConfigurationDataSample.MyVM1** et **ConfigurationDataSample.MyVM3**.

```powershell
$ConfigData = @{
    AllNodes = @(
        @{
            NodeName = 'MyVM1'
            Role = 'WebServer'
        },
        @{
            NodeName = 'MyVM2'
            Role = 'SQLServer'
        },
        @{
            NodeName = 'MyVM3'
            Role = 'WebServer'
        }
    )

    NonNodeData = @{
        SomeMessage = 'I love Azure Automation State Configuration and DSC!'
    }
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'ConfigurationDataSample' -ConfigurationData $ConfigData
```

### <a name="work-with-assets-in-azure-automation-during-compilation"></a>Utiliser des ressources dans Azure Automation pendant la compilation

Les références de ressources sont les mêmes dans Azure Automation State Configuration et les runbooks. Pour plus d’informations, consultez les rubriques suivantes :

- [Certificates](./shared-resources/certificates.md)
- [Connexions](automation-connections.md)
- [Informations d'identification](./shared-resources/credentials.md)
- [Variables](./shared-resources/variables.md)

#### <a name="credential-assets"></a>Ressources d’informations d’identification

Les configurations DSC dans Azure Automation peuvent référencer des ressources d’informations d’identification Automation avec la cmdlet `Get-AutomationPSCredential`. Si une configuration a un paramètre qui spécifie un objet `PSCredential`, utilisez `Get-AutomationPSCredential` en passant le nom de la chaîne d’une ressource d’informations d’identification Azure Automation à l’applet de commande afin de récupérer les informations d’identification. Ensuite, utilisez cet objet pour le paramètre nécessitant l’objet `PSCredential`. En arrière-plan, la ressource d’informations d’identification Azure Automation portant le même nom est récupérée et transmise à la configuration. L’exemple ci-dessous illustre ce scénario.

Conserver les informations d’identification en sûreté dans les configurations de nœud (documents de configuration MOF) nécessite le chiffrement des informations d’identification dans le fichier MOF de configuration de nœud. Vous devez autoriser PowerShell DSC à générer des informations d’identification en texte brut lors de la génération du fichier MOF de configuration de nœud. PowerShell DSC ne sait pas qu’Azure Automation chiffre l’intégralité du fichier MOF après sa génération par le biais d’un travail de compilation.

Vous pouvez indiquer à la configuration DSC PowerShell que vous êtes d’accord pour que les informations d’identification soient extraites en texte brut dans les fichiers MOF de configuration de nœud générés en utilisant ConfigurationData. Vous devez passer `PSDscAllowPlainTextPassword = $true` par le biais de `ConfigurationData` pour chaque nom de bloc de nœuds qui s’affiche dans la configuration DSC et qui utilise les informations d’identification.

L’exemple suivant montre une configuration de l’état souhaité qui utilise une ressource d’informations d’identification Automation.

```powershell
Configuration CredentialSample
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration
    $Cred = Get-AutomationPSCredential 'SomeCredentialAsset'

    Node $AllNodes.NodeName
    {
        File ExampleFile
        {
            SourcePath      = '\\Server\share\path\file.ext'
            DestinationPath = 'C:\destinationPath'
            Credential      = $Cred
        }
    }
}
```

Vous pouvez compiler la configuration DSC précédente avec PowerShell. Le code PowerShell suivant ajoute deux configurations de nœud au serveur Pull Azure Automation State Configuration : **CredentialSample.MyVM1** et **CredentialSample.MyVM2**.

```powershell
$ConfigData = @{
    AllNodes = @(
        @{
            NodeName = '*'
            PSDscAllowPlainTextPassword = $True
        },
        @{
            NodeName = 'MyVM1'
        },
        @{
            NodeName = 'MyVM2'
        }
    )
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'CredentialSample' -ConfigurationData $ConfigData
```

> [!NOTE]
> Lorsque la compilation est terminée, vous pouvez recevoir le message d'erreur `The 'Microsoft.PowerShell.Management' module was not imported because the 'Microsoft.PowerShell.Management' snap-in was already imported.` Vous pouvez sans risque ignorer ce message.

## <a name="compile-your-dsc-configuration-in-windows-powershell"></a>Compiler une configuration DSC dans Windows PowerShell

Le processus de compilation des configurations DSC dans Windows PowerShell est inclus dans la documentation DSC PowerShell [Écrire, compiler et appliquer une configuration](/powershell/scripting/dsc/configurations/write-compile-apply-configuration#compile-the-configuration).
Vous pouvez exécuter ce processus à partir d’une station de travail de développeur ou dans un service de build tel qu’[Azure DevOps](https://dev.azure.com). Vous pouvez ensuite importer les fichiers MOF générés par la compilation de la configuration dans le service Azure State Configuration.

La compilation dans Windows PowerShell offre également la possibilité de signer le contenu de configuration. L’agent DSC vérifie localement la configuration signée d’un nœud géré. Cette vérification garantit que la configuration appliquée au nœud provient d’une source autorisée.

Vous pouvez également importer les configurations de nœud (fichiers MOF) qui ont été compilées hors d’Azure. Cette importation comprend la compilation à partir d’une station de travail de développeur ou dans un service comme [Azure DevOps](https://dev.azure.com). Cette approche présente plusieurs avantages, comme les performances et la fiabilité.

> [!NOTE]
> Un fichier de configuration de nœuds ne doit pas être supérieur à 1 Mo pour permettre à Azure Automation de l’importer.

Pour plus d’informations sur la signature des configurations de nœud, consultez [Améliorations apportées à WMF 5.1 – Comment signer la configuration et le module](/powershell/scripting/wmf/whats-new/dsc-improvements#dsc-module-and-configuration-signing-validations).

### <a name="import-a-node-configuration-in-the-azure-portal"></a>Importer une configuration de nœuds dans le Portail Azure

1. Dans votre compte Automation, cliquez sur **State configuration (DSC)** sous **Gestion de la configuration**.
1. Dans la page State Configuration (DSC), cliquez sur l’onglet **Configurations**, puis sur **Ajouter**.
1. Dans la page Importer, cliquez sur l’icône de dossier à côté du champ **Fichier de configuration de nœuds** pour rechercher un fichier de configuration de nœuds (MOF) sur votre ordinateur local.

   ![Rechercher un fichier local](./media/automation-dsc-compile/import-browse.png)

1. Entrez un nom dans le champ **Nom de la configuration**. Ce nom doit correspondre au nom de la configuration à partir de laquelle la configuration de nœuds a été compilée.
1. Cliquez sur **OK**.

### <a name="import-a-node-configuration-with-azure-powershell"></a>Importer une configuration de nœuds avec Azure PowerShell

Vous pouvez utiliser la cmdlet [Import-AzAutomationDscNodeConfiguration](/powershell/module/az.automation/import-azautomationdscnodeconfiguration) pour importer une configuration de nœuds dans votre compte Automation.

```powershell
Import-AzAutomationDscNodeConfiguration -AutomationAccountName 'MyAutomationAccount' -ResourceGroupName 'MyResourceGroup' -ConfigurationName 'MyNodeConfiguration' -Path 'C:\MyConfigurations\TestVM1.mof'
```

## <a name="next-steps"></a>Étapes suivantes

- Pour commencer, consultez [Bien démarrer avec Azure Automation State Configuration](automation-dsc-getting-started.md).
- Pour découvrir comment compiler des configurations DSC pour pouvoir les affecter à des nœuds cibles, consultez [Compiler des configurations DSC dans Azure Automation State Configuration](automation-dsc-compile.md).
- Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).
- Pour obtenir des informations sur les prix, consultez [Tarification d’Azure Automation State Configuration](https://azure.microsoft.com/pricing/details/automation/).
- Pour obtenir un exemple d’utilisation de State Configuration dans un pipeline de déploiement continu, consultez [Configurer un déploiement continu avec Chocolatey](automation-dsc-cd-chocolatey.md).
