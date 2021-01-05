---
title: Démarrer des machines avec des runbooks Automation dans Azure DevTest Labs
description: Découvrez comment démarrer des machines virtuelles dans un labo dans Azure DevTest Labs en utilisant des runbooks Azure Automation.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 231e79d594aab7c59fa21f9ee512abaa9ac67043
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87282260"
---
# <a name="start-virtual-machines-in-a-lab-in-order-by-using-azure-automation-runbooks"></a>Démarrer des machines virtuelles dans un labo dans un certain ordre en utilisant des runbooks Azure Automation
La fonctionnalité de [démarrage automatique](devtest-lab-set-lab-policy.md#set-autostart) de DevTest Labs vous permet configurer des machines virtuelles pour qu’elles démarrent automatiquement à une heure spécifiée. Cependant, cette fonctionnalité ne prend pas en charge le démarrage des machines dans un ordre spécifique. Il existe plusieurs scénarios où ce type d’automatisation peut être utile.  Voici un de ces scénarios : une machine virtuelle Jumpbox dans un labo doit être démarrée en premier, avant les autres machines virtuelles, car la machine Jumpbox est utilisée comme point d’accès aux autres machines virtuelles.  Cet article vous montre comment configurer un compte Azure Automation avec un runbook PowerShell qui exécute un script. Le script utilise des étiquettes sur les machines virtuelles du labo pour vous permettre de contrôler l’ordre de démarrage sans devoir modifier le script.

## <a name="setup"></a>Programme d’installation
Dans cet exemple, l’étiquette **StartupOrder** avec la valeur appropriée (0, 1, 2, etc.) doit être ajoutée aux machines virtuelles du labo. Désignez les machines qui n’ont pas besoin d’être démarrée en affectant la valeur -1 à l’étiquette.

## <a name="create-an-azure-automation-account"></a>Créer un compte Azure Automation
Créez un compte Azure Automation en suivant les instructions de [cet article](../automation/automation-create-standalone-account.md). Choisissez l’option **Compte d’identification** lors de la création du compte. Une fois que le compte Automation est créé, ouvrez la page **Modules**, puis sélectionnez **Mettre à jour les modules Azure** sur la barre de menus. Les modules par défaut sont dans des versions plus anciennes ; le script ne peut pas fonctionner sans la mise à jour.

## <a name="add-a-runbook"></a>Ajouter un runbook
Maintenant, pour ajouter un runbook au compte Automation, sélectionnez **Runbooks** sur le menu de gauche. Sélectionnez **Ajouter un runbook** sur le menu, puis suivez les instructions pour [créer un runbook PowerShell](../automation/learn/automation-tutorial-runbook-textual-powershell.md).

## <a name="powershell-script"></a>Script PowerShell
Le script suivant prend comme paramètres le nom de l’abonnement et le nom du labo. Le flux du script est destiné à obtenir toutes les machines virtuelles dans le labo, puis d’analyser les informations des étiquettes pour créer une liste des noms des machines virtuelles et de leur ordre de démarrage. Le script parcourt les machines virtuelles dans l’ordre et les démarre. S’il existe plusieurs machines virtuelles avec un même numéro d’ordre, elles sont démarrées de façon asynchrone via des travaux PowerShell. Pour les machines virtuelles qui n’ont pas d’étiquette, définissez la valeur du démarrage sur la dernière (10) : par défaut, elles sont démarrées en dernier.  Si le labo ne veut pas que la machine virtuelle soit démarrée automatiquement, définissez la valeur de l’étiquette sur 11 : elle est alors ignorée.

```powershell
#Requires -Version 3.0
#Requires -Module AzureRM.Resources

param
(
    [Parameter(Mandatory=$false, HelpMessage="Name of the subscription that has the lab")]
    [string] $SubscriptionName,

    [Parameter(Mandatory=$false, HelpMessage="Lab name")]
    [string] $LabName
)

# Connect and add the appropriate subscription
$Conn = Get-AutomationConnection -Name AzureRunAsConnection

Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationID $Conn.ApplicationId -Subscription $SubscriptionName -CertificateThumbprint $Conn.CertificateThumbprint

# Find the lab
$dtLab = Find-AzResource -ResourceType 'Microsoft.DevTestLab/labs' -ResourceNameEquals $LabName

# Get the VMs
$dtlAllVms = New-Object System.Collections.ArrayList
$AllVMs = Get-AzResource -ResourceId "$($dtLab.ResourceId)/virtualmachines" -ApiVersion 2016-05-15

# Get the StartupOrder tag, if missing set to be run last (10)
ForEach ($vm in $AllVMs) {
    if ($vm.Tags) {
        if ($vm.Tags['StartupOrder']) {
            $startupValue = $vm.Tags['StartupOrder']
        } else {
            $startupValue = 10
        }
        } else {
            $startupValue = 10
        }
        $dtlAllVms.Add(@{$vm.Name = $startupValue}) > $null
}

# Setup for the async multiple vm start

# Save profile
$profilePath = Join-Path $env:Temp "profile.json"
If (Test-Path $profilePath){
    Remove-Item $profilePath
}
Save-AzContext -Path $profilePath

# Job to start VMs asynch
$startVMBlock = {
    Param($devTestLab,$vmToStart,$profilePath)
    Import-AzContext -Path ($profilePath)
    Invoke-AzResourceAction `
        -ResourceId "$($devTestLab.ResourceId)/virtualmachines/$vmToStart" `
        -Action Start `
        -Force
    Write-Output "Started: $vmToStart"
}

$current = 0
# Start in order from 0 to 10

While ($current -le 10) {
# Get the VMs in the current stage
    $tobeStarted = $null
    $tobeStarted = $dtlAllVms | Where-Object { $_.Values -eq $current}
    if ($tobeStarted.Count -eq 1) {
        # Run sync – jobs not necessary for a single VM
        $returnStatus = Invoke-AzResourceAction `
                -ResourceId "$($dtLab.ResourceId)/virtualmachines/$($tobeStarted.Keys)" `
                -Action Start `
                -Force
        Write-Output "$($tobeStarted.Keys) status: $($returnStatus.status)"
    } elseif ($tobeStarted.Count -gt 1) {
        # Start multiple VMs async
        $jobs = @()
        Write-Output "Start Jobs start: $(Get-Date)"
        
        # Jobs
        $jobs += Start-Job -ScriptBlock $startVMBlock -ArgumentList $dtLab, $($singlevm.Keys), $profilePath
        Write-Output "Start Jobs end: $(Get-Date)"
    }

    # Get results from all jobs
    if($jobs.Count -ne 0) {
        Write-Output "Receive Jobs start: $(Get-Date)"
        foreach ($job in $jobs){
            $jobResult = Receive-Job -Job $job -Wait | Write-Output
        }
        Remove-Job -Job $jobs -Force
    }
    else
    {
        Write-Output "Information: No jobs available"
    }
}
```

## <a name="create-a-schedule"></a>Créer une planification
Pour que ce script s’exécute tous les jours, [créez une planification](../automation/shared-resources/schedules.md#create-a-schedule) dans le compte Automation. Une fois que la planification est créée, [liez-la au runbook](../automation/shared-resources/schedules.md#link-a-schedule-to-a-runbook). 

Dans une situation à grande échelle où il existe plusieurs abonnements avec plusieurs labos, stockez les informations des paramètres dans un fichier pour les différents labos et passez le fichier au script à la place des paramètres individuels. Le script devrait alors être modifié, mais l’exécution resterait fondamentalement la même. Bien que cet exemple utilise Azure Automation pour exécuter le script PowerShell, il existe d’autres options, comme l’utilisation d’une tâche dans un pipeline de build/mise en production.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur Azure Automation, consultez l’article suivant : [Présentation d’Azure Automation](../automation/automation-intro.md).
