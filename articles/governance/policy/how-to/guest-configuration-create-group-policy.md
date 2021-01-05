---
title: Comment créer des définitions de stratégie de configuration d’invité à partir de la stratégie de groupe de référence pour Windows
description: Découvrez comment convertir la stratégie de groupe issue de la sécurité de référence du Windows Server 2019 en une définition de politique.
ms.date: 08/17/2020
ms.topic: how-to
ms.openlocfilehash: 7f7e2af70efa6771d94d7ceaa14d1408175b1d12
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93348642"
---
# <a name="how-to-create-guest-configuration-policy-definitions-from-group-policy-baseline-for-windows"></a>Comment créer des définitions de stratégie de configuration d’invité à partir de la stratégie de groupe de référence pour Windows

Avant de créer des définitions de stratégie personnalisées, il est judicieux de lire les informations conceptuelles de la section [Azure Policy Guest Configuration](../concepts/guest-configuration.md). Pour en savoir plus sur la création de politiques Guest Configuration personnalisées pour Linux, consultez la page [Créer des stratégies Guest Configuration pour Linux](./guest-configuration-create-linux.md). Pour en savoir plus sur la création de politiques Guest Configuration personnalisées pour Windows, consultez la page [Créer des stratégies Guest Configuration pour Windows](./guest-configuration-create.md).

Lors de l’audit Windows, Guest Configuration utilise un module de ressources [Desired State Configuration](/powershell/scripting/dsc/overview/overview) (DSC) pour créer le fichier de configuration. La configuration DSC définit la condition dans laquelle la machine doit se trouver. Si l’évaluation de la configuration est **non conforme** , l’effet de stratégie *auditIfNotExists* est déclenché.
L’[Azure Policy Guest Configuration](../concepts/guest-configuration.md) effectue uniquement un audit des paramètres à l’intérieur de la machine.

> [!IMPORTANT]
> L’extension Guest Configuration (Configuration d’invité) est requise pour effectuer des audits sur des machines virtuelles Azure. Pour déployer l’extension à grande échelle sur tous les ordinateurs Windows, attribuez les définitions de stratégie suivantes :
> - [Déployer les prérequis pour activer la stratégie de configuration d’invité sur les machines virtuelles Windows.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0ecd903d-91e7-4726-83d3-a229d7f2e293)
> 
> N’utilisez pas de secrets ni d’informations confidentielles dans les packages de contenu personnalisés.

La communauté DSC a publié le [module BaselineManagement](https://github.com/microsoft/BaselineManagement) pour convertir les modèles de stratégie de groupe exportés au format DSC. Avec la cmdlet GuestConfiguration, le module BaselineManagement crée un package Azure Policy Guest Configuration pour Windows à partir du contenu de la stratégie de groupe. Pour plus d’informations sur l’utilisation du module BaselineManagement, consultez l’article [Démarrage rapide : Convertir une stratégie de groupe en DSC](/powershell/scripting/dsc/quickstarts/gpo-quickstart).

Dans ce guide, nous allons passer en revue le processus de création d’un package Azure Policy Guest Configuration à partir d’un objet de stratégie de groupe (GPO). Tandis que la procédure guidée décrit la conversion de la sécurité de référence Windows Server 2019, le même processus peut être appliqué à d’autres objets de stratégie de groupe.  

## <a name="download-windows-server-2019-security-baseline-and-install-related-powershell-modules"></a>Télécharger la sécurité de base Windows Server 2019 et installer les modules PowerShell associés

Pour installer les modules **DSC** , **GuestConfiguration** , **BaselineManagement** et les modules Azure associés dans PowerShell :

1. À partir d’une invite de commandes PowerShell, exécutez la commande suivante :

   ```azurepowershell-interactive
   # Install the BaselineManagement module, Guest Configuration DSC resource module, and relevant Azure modules from PowerShell Gallery
   Install-Module az.resources, az.policyinsights, az.storage, guestconfiguration, gpregistrypolicyparser, securitypolicydsc, auditpolicydsc, baselinemanagement -scope currentuser -Repository psgallery -AllowClobber
   ```

1. Créez un répertoire dédié et téléchargez la sécurité de base de Windows Server 2019 à partir de la boîte à outils pour la conformité de la sécurité Windows.

   ```azurepowershell-interactive
   # Download the 2019 Baseline files from https://docs.microsoft.com/windows/security/threat-protection/security-compliance-toolkit-10
   New-Item -Path 'C:\git\policyfiles\downloads' -Type Directory
   Invoke-WebRequest -Uri 'https://download.microsoft.com/download/8/5/C/85C25433-A1B0-4FFA-9429-7E023E7DA8D8/Windows%2010%20Version%201909%20and%20Windows%20Server%20Version%201909%20Security%20Baseline.zip' -Out C:\git\policyfiles\downloads\Server2019Baseline.zip
   ```

1. Débloquez et développez la référence Server 2019 téléchargée.

   ```azurepowershell-interactive
   Unblock-File C:\git\policyfiles\downloads\Server2019Baseline.zip
   Expand-Archive -Path C:\git\policyfiles\downloads\Server2019Baseline.zip -DestinationPath C:\git\policyfiles\downloads\
   ```

1. Vérifiez le contenu de la référence Server 2019 à l’aide de **MapGuidsToGpoNames.ps1**.

   ```azurepowershell-interactive
   # Show content details of downloaded GPOs
   C:\git\policyfiles\downloads\Scripts\Tools\MapGuidsToGpoNames.ps1 -rootdir C:\git\policyfiles\downloads\GPOs\ -Verbose
   ```

## <a name="convert-from-group-policy-to-azure-policy-guest-configuration"></a>Convertir le contenu de stratégie de groupe Windows en Azure Policy Guest Configuration

Ensuite, nous allons convertir la référence Server 2019 téléchargée en un package Guest Configuration à l’aide des modules Guest Configuration et Baseline Management.

1. Convertissez le stratégie de groupe en Desired State Configuration à l’aide du module Baseline Management.

   ```azurepowershell-interactive
   ConvertFrom-GPO -Path 'C:\git\policyfiles\downloads\GPOs\{3657C7A2-3FF3-4C21-9439-8FDF549F1D68}\' -OutputPath 'C:\git\policyfiles\' -OutputConfigurationScript -Verbose
   ```

1. Renommez, reformatez et exécutez les scripts convertis avant de créer un package de contenu de stratégie.

   ```azurepowershell-interactive
   Rename-Item -Path C:\git\policyfiles\DSCFromGPO.ps1 -NewName C:\git\policyfiles\Server2019Baseline.ps1
   (Get-Content -Path C:\git\policyfiles\Server2019Baseline.ps1).Replace('DSCFromGPO', 'Server2019Baseline') | Set-Content -Path C:\git\policyfiles\Server2019Baseline.ps1
   (Get-Content -Path C:\git\policyfiles\Server2019Baseline.ps1).Replace('PSDesiredStateConfiguration', 'PSDscResources') | Set-Content -Path C:\git\policyfiles\Server2019Baseline.ps1
   C:\git\policyfiles\Server2019Baseline.ps1
   ```

1. Créez un package de contenu Azure Policy Guest Configuration.

   ```azurepowershell-interactive
   New-GuestConfigurationPackage -Name Server2019Baseline -Configuration c:\git\policyfiles\localhost.mof -Verbose
   ```

## <a name="create-azure-policy-guest-configuration"></a>Créez une Azure Policy Guest Configuration

1. L’étape suivante consiste à publier le fichier dans Stockage Blob Azure. La commande `Publish-GuestConfigurationPackage` requiert le module `Az.Storage`.

   ```azurepowershell-interactive
   Publish-GuestConfigurationPackage -Path ./AuditBitlocker.zip -ResourceGroupName  myResourceGroupName -StorageAccountName myStorageAccountName
   ```

1. Une fois qu’un package de stratégie personnalisée Guest Configuration a été créé et chargé, créez la définition de la stratégie Guest Configuration. Utilisez la cmdlet `New-GuestConfigurationPolicy` pour créer la Guest Configuration.

   ```azurepowershell-interactive
    $NewGuestConfigurationPolicySplat = @{
        ContentUri = $Uri 
        DisplayName = 'Server 2019 Configuration Baseline' 
        Description 'Validation of using a completely custom baseline configuration for Windows VMs' 
        Path = 'C:\git\policyfiles\policy'  
        Platform = Windows 
        }
   New-GuestConfigurationPolicy @NewGuestConfigurationPolicySplat
   ```
    
1. Publiez les définitions de stratégie à l’aide de la cmdlet `Publish-GuestConfigurationPolicy`. La cmdlet ne dispose que du paramètre **Path** qui pointe vers l’emplacement des trois fichiers JSON créés par `New-GuestConfigurationPolicy`. Pour exécuter la commande Publish, vous devez avoir accès à la création de définitions de stratégies dans Azure. Les exigences spécifiques en matière d’autorisations sont documentées dans la page [vue d’ensemble d’Azure Policy](../overview.md#getting-started). Le meilleur rôle intégré est le rôle **Contributeur de la stratégie de ressource**.

   ```azurepowershell-interactive
   Publish-GuestConfigurationPolicy -Path C:\git\policyfiles\policy\ -Verbose
   ```

## <a name="assign-guest-configuration-policy-definition"></a>Assigner une définition à la stratégie Guest Configuration

Avec la stratégie créée dans Azure, la dernière étape consiste à assigner l’initiative. Découvrez comment assigner l’initiative avec le [portail](../assign-policy-portal.md), [Azure CLI](../assign-policy-azurecli.md) et [Azure PowerShell](../assign-policy-powershell.md).

> [!IMPORTANT]
> Les définitions de stratégies Guest Configuration doivent **toujours** être assignées via l’initiative qui combine les stratégies _AuditIfNotExists_ et _DeployIfNotExists_. Si seule la stratégie _AuditIfNotExists_ est assignée, les prérequis ne sont pas déployés et la stratégie montre toujours que « 0 » serveur est conforme.

L’affectation d’une définition de stratégie avec l’effet _DeployIfNotExists_ requiert un niveau d’accès supplémentaire. Pour accorder le privilège le plus bas, vous pouvez créer une définition de rôle personnalisée qui étend le rôle **Contributeur de stratégie de ressource**. L’exemple ci-dessous crée un rôle appelé **Contributeur de stratégie de ressource DINE** avec l’autorisation supplémentaire _Microsoft.Authorization/roleAssignments/write_.

   ```azurepowershell-interactive
   $subscriptionid = '00000000-0000-0000-0000-000000000000'
   $role = Get-AzRoleDefinition "Resource Policy Contributor"
   $role.Id = $null
   $role.Name = "Resource Policy Contributor DINE"
   $role.Description = "Can assign Policies that require remediation."
   $role.Actions.Clear()
   $role.Actions.Add("Microsoft.Authorization/roleAssignments/write")
   $role.AssignableScopes.Clear()
   $role.AssignableScopes.Add("/subscriptions/$subscriptionid")
   New-AzRoleDefinition -Role $role
   ```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur l’audit des machines virtuelles avec [Guest Configuration](../concepts/guest-configuration.md).
- Découvrez comment [créer des stratégies par programmation](./programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](./get-compliance-data.md).
