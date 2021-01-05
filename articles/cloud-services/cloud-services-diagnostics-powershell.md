---
title: Activer les diagnostics dans Azure Cloud Services à l’aide de PowerShell | Microsoft Docs
description: Découvrez comment utiliser PowerShell pour activer la collecte des données de diagnostic à partir d’un service Cloud Azure avec l’extension Diagnostics Azure.
services: cloud-services
documentationcenter: .net
author: tgore03
ms.service: cloud-services
ms.devlang: dotnet
ms.topic: article
ms.date: 09/06/2016
ms.author: tagore
ms.openlocfilehash: f0bf7fa56ca511981820f63cd7178493a7d048e3
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92077505"
---
# <a name="enable-diagnostics-in-azure-cloud-services-using-powershell"></a>Activer les diagnostics dans Azure Cloud Services à l’aide de PowerShell
Vous pouvez collecter des données de diagnostic telles que les journaux des applications, les compteurs de performances, etc. à partir d’un service cloud à l’aide de l’extension de diagnostics Azure. Cet article décrit comment activer l’extension Diagnostics Azure pour un service cloud à l’aide de PowerShell.  Voir [Installer et configurer Azure PowerShell](/powershell/azure/) pour connaître les conditions requises pour cet article.

## <a name="enable-diagnostics-extension-as-part-of-deploying-a-cloud-service"></a>Activer l’extension de diagnostics lors du déploiement d’un service cloud
Cette approche peut s’appliquer aux scénarios d’intégration continue où l’extension de diagnostics peut être activée dans le cadre du déploiement d’un service cloud. Lorsque vous créez un déploiement de service cloud, vous pouvez activer l’extension de diagnostics en transmettant le paramètre *ExtensionConfiguration* à la cmdlet [New-AzureDeployment](/powershell/module/servicemanagement/azure.service/new-azuredeployment?view=azuresmps-3.7.0) . Le paramètre *ExtensionConfiguration* prend un tableau de configurations de diagnostics qui peut être créé à l’aide de l’applet de commande [New-AzureServiceDiagnosticsExtensionConfig](/powershell/module/servicemanagement/azure.service/new-azureservicediagnosticsextensionconfig?view=azuresmps-3.7.0) .

L’exemple suivant montre comment vous pouvez activer les diagnostics pour un service cloud avec un rôle web et un rôle de travail possédant chacun sa propre configuration de diagnostics.

```powershell
$service_name = "MyService"
$service_package = "CloudService.cspkg"
$service_config = "ServiceConfiguration.Cloud.cscfg"
$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml"
$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"

$webrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WebRole" -DiagnosticsConfigurationPath $webrole_diagconfigpath
$workerrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WorkerRole" -DiagnosticsConfigurationPath $workerrole_diagconfigpath

New-AzureDeployment -ServiceName $service_name -Slot Production -Package $service_package -Configuration $service_config -ExtensionConfiguration @($webrole_diagconfig,$workerrole_diagconfig)
```

Si le fichier de configuration des diagnostics spécifie un élément `StorageAccount` avec un nom de compte de stockage, l’applet de commande `New-AzureServiceDiagnosticsExtensionConfig` utilise automatiquement ce compte de stockage. Pour que cela fonctionne, le compte de stockage doit appartenir au même abonnement que le service cloud déployé.

À partir d’Azure SDK 2.6, les fichiers de configuration de l’extension générés par la sortie cible de publication MSBuild incluent le nom du compte de stockage en fonction de la chaîne de configuration des diagnostics spécifiée dans le fichier de configuration de service (.cscfg). Le script ci-dessous vous montre comment analyser les fichiers de configuration de l’extension à partir de la sortie cible de publication, et comment configurer l’extension de diagnostics pour chaque rôle lorsque vous déployez le service cloud.

```powershell
$service_name = "MyService"
$service_package = "C:\build\output\CloudService.cspkg"
$service_config = "C:\build\output\ServiceConfiguration.Cloud.cscfg"

#Find the Extensions path based on service configuration file
$extensionsSearchPath = Join-Path -Path (Split-Path -Parent $service_config) -ChildPath "Extensions"

$diagnosticsExtensions = Get-ChildItem -Path $extensionsSearchPath -Filter "PaaSDiagnostics.*.PubConfig.xml"
$diagnosticsConfigurations = @()
foreach ($extPath in $diagnosticsExtensions)
{
    #Find the RoleName based on file naming convention PaaSDiagnostics.<RoleName>.PubConfig.xml
    $roleName = ""
    $roles = $extPath -split ".",0,"simplematch"
    if ($roles -is [system.array] -and $roles.Length -gt 1)
    {
        $roleName = $roles[1]
        $x = 2
        while ($x -le $roles.Length)
            {
               if ($roles[$x] -ne "PubConfig")
                {
                    $roleName = $roleName + "." + $roles[$x]
                }
                else
                {
                    break
                }
                $x++
            }
        $fullExtPath = Join-Path -path $extensionsSearchPath -ChildPath $extPath
        $diagnosticsconfig = New-AzureServiceDiagnosticsExtensionConfig -Role $roleName -DiagnosticsConfigurationPath $fullExtPath
        $diagnosticsConfigurations += $diagnosticsconfig
    }
}
New-AzureDeployment -ServiceName $service_name -Slot Production -Package $service_package -Configuration $service_config -ExtensionConfiguration $diagnosticsConfigurations
```

Visual Studio Online utilise une approche similaire pour les déploiements automatisés de services cloud avec l’extension de diagnostics. Consultez [Publish-AzureCloudDeployment.ps1](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureCloudPowerShellDeploymentV1/Publish-AzureCloudDeployment.ps1) pour obtenir un exemple complet.

Si aucun `StorageAccount` n’a été spécifié dans la configuration des diagnostics, vous devez transmettre le paramètre *StorageAccountName* à l’applet de commande. Si le paramètre *StorageAccountName* est spécifié, l’applet de commande utilise toujours le compte de stockage spécifié dans le paramètre et non celui spécifié dans le fichier de configuration des diagnostics.

Si le compte de stockage de diagnostics appartient à un autre abonnement que celui du service cloud, vous devez transmettre explicitement les paramètres *StorageAccountName* et *StorageAccountKey* à l’applet de commande. Le paramètre *StorageAccountKey* n’est pas nécessaire lorsque le compte de stockage de diagnostics appartient au même abonnement si l’applet de commande peut interroger et définir automatiquement la valeur clé lors de l’activation de l’extension de diagnostics. Toutefois, si le compte de stockage de diagnostics appartient à un autre abonnement, l’applet de commande n’est peut-être pas en mesure d’obtenir automatiquement la clé, et vous devez explicitement spécifier celle-ci par le biais du paramètre *StorageAccountKey* .

```powershell
$webrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WebRole" -DiagnosticsConfigurationPath $webrole_diagconfigpath -StorageAccountName $diagnosticsstorage_name -StorageAccountKey $diagnosticsstorage_key
$workerrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WorkerRole" -DiagnosticsConfigurationPath $workerrole_diagconfigpath -StorageAccountName $diagnosticsstorage_name -StorageAccountKey $diagnosticsstorage_key
```

## <a name="enable-diagnostics-extension-on-an-existing-cloud-service"></a>Activer l’extension de diagnostics sur un service cloud existant
Vous pouvez utiliser l’applet de commande [Set-AzureServiceDiagnosticsExtension](/powershell/module/servicemanagement/azure.service/set-azureservicediagnosticsextension?view=azuresmps-3.7.0) pour activer ou mettre à jour la configuration de diagnostics sur un service cloud qui est déjà en cours d’exécution.

[!INCLUDE [cloud-services-wad-warning](../../includes/cloud-services-wad-warning.md)]

```powershell
$service_name = "MyService"
$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml"
$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"

$webrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WebRole" -DiagnosticsConfigurationPath $webrole_diagconfigpath
$workerrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WorkerRole" -DiagnosticsConfigurationPath $workerrole_diagconfigpath

Set-AzureServiceDiagnosticsExtension -DiagnosticsConfiguration @($webrole_diagconfig,$workerrole_diagconfig) -ServiceName $service_name
```

## <a name="get-current-diagnostics-extension-configuration"></a>Obtenir la configuration actuelle de l’extension de diagnostics
Pour obtenir la configuration de diagnostics actuelle pour un service cloud, utilisez l’applet de commande [Get-AzureServiceDiagnosticsExtension](/powershell/module/servicemanagement/azure.service/get-azureservicediagnosticsextension?view=azuresmps-3.7.0) :

```powershell
Get-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```

## <a name="remove-diagnostics-extension"></a>Supprimer l’extension de diagnostics
Pour désactiver les diagnostics sur un service cloud, vous pouvez utiliser la cmdlet [Remove-AzureServiceDiagnosticsExtension](/powershell/module/servicemanagement/azure.service/remove-azureservicediagnosticsextension?view=azuresmps-3.7.0).

```powershell
Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```

Si vous avez activé l’extension de diagnostics à l’aide de *Set-AzureServiceDiagnosticsExtension* ou *New-AzureServiceDiagnosticsExtensionConfig* sans le paramètre *Role*, vous pouvez supprimer l’extension à l’aide de *Remove-AzureServiceDiagnosticsExtension* sans le paramètre *Role*. Si le paramètre *Role* a été utilisé lors de l’activation de l’extension, il doit également être utilisé lors de la suppression de l’extension.

Pour supprimer l’extension de diagnostics de chaque rôle individuel :

```powershell
Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService" -Role "WebRole"
```

## <a name="next-steps"></a>Étapes suivantes
* Pour obtenir une aide supplémentaire sur l’utilisation des diagnostics Azure et d’autres techniques pour résoudre les problèmes, consultez la page [Activation de Diagnostics dans Azure Cloud Services et Azure Virtual Machines](cloud-services-dotnet-diagnostics.md).
* Le [schéma de configuration des diagnostics](../azure-monitor/platform/diagnostics-extension-schema-windows.md) explique les différentes options de configuration xml pour l’extension de diagnostics.
* Pour savoir comment activer l’extension de diagnostics pour les machines virtuelles, consultez [Créer une machine virtuelle Windows avec la surveillance et les diagnostics à l’aide d’un modèle Azure Resource Manager](../virtual-machines/extensions/diagnostics-template.md)