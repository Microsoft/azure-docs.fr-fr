---
title: Utilisation de PowerShell pour configurer Application Insights pour une application web Azure | Microsoft Docs
description: Automatisation de la configuration de Azure Diagnostics pour canaliser les données vers Application Insights.
ms.topic: conceptual
ms.date: 08/06/2019
ms.openlocfilehash: 0fd69b90ce6329041f96b8e3173f1f17270f68ee
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94699727"
---
# <a name="using-powershell-to-set-up-application-insights-for-azure-cloud-services"></a>Configuration de Application Insights pour Azure Cloud Services à l’aide de PowerShell

[Microsoft Azure](https://azure.com) peut être [configuré pour envoyer des Diagnostics Azure](../platform/diagnostics-extension-to-application-insights.md) vers [Azure Application Insights](./app-insights-overview.md). Les tests de diagnostic concernent Azure Cloud Services et les machines virtuelles Azure. Ils permettent de compléter les données de télémétrie que vous envoyez depuis l’application à l’aide du kit de développement logiciel d’Application Insights. Dans le cadre de l’automatisation du processus de création de nouvelles ressources dans Azure, vous pouvez configurer des diagnostics avec PowerShell.

## <a name="azure-template"></a>Modèle Azure
Si l’application web est dans Azure et que vous créez vos ressources à l’aide d’un modèle Azure Resource Manager, vous pouvez configurer Application Insights en ajoutant cela au nœud de ressources :

```json
{
  resources: [
    /* Create Application Insights resource */
    {
      "apiVersion": "2015-05-01",
      "type": "microsoft.insights/components",
      "name": "nameOfAIAppResource",
      "location": "centralus",
      "kind": "web",
      "properties": { "ApplicationId": "nameOfAIAppResource" },
      "dependsOn": [
        "[concat('Microsoft.Web/sites/', myWebAppName)]"
      ]
    }
  ]
}
``` 

* `nameOfAIAppResource` : nom de la ressource Application Insights
* `myWebAppName` : ID de l’application web

## <a name="enable-diagnostics-extension-as-part-of-deploying-a-cloud-service"></a>Activer l’extension de diagnostics lors du déploiement d’un service cloud
L’applet de commande `New-AzureDeployment` comporte un paramètre `ExtensionConfiguration` qui utilise un tableau de configurations de diagnostics. Ces derniers peuvent être créés à l’aide de l’applet de commande `New-AzureServiceDiagnosticsExtensionConfig` . Par exemple :

```azurepowershell
$service_package = "CloudService.cspkg"
$service_config = "ServiceConfiguration.Cloud.cscfg"
$diagnostics_storagename = "myservicediagnostics"
$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml" 
$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"

$primary_storagekey = (Get-AzStorageKey `
  -StorageAccountName "$diagnostics_storagename").Primary
$storage_context = New-AzStorageContext `
  -StorageAccountName $diagnostics_storagename `
  -StorageAccountKey $primary_storagekey

$webrole_diagconfig = `
  New-AzureServiceDiagnosticsExtensionConfig `
    -Role "WebRole" -Storage_context $storageContext `
    -DiagnosticsConfigurationPath $webrole_diagconfigpath
$workerrole_diagconfig = `
  New-AzureServiceDiagnosticsExtensionConfig `
    -Role "WorkerRole" `
    -StorageContext $storage_context `
    -DiagnosticsConfigurationPath $workerrole_diagconfigpath

  New-AzureDeployment `
    -ServiceName $service_name `
    -Slot Production `
    -Package $service_package `
    -Configuration $service_config `
    -ExtensionConfiguration @($webrole_diagconfig,$workerrole_diagconfig)
``` 

## <a name="enable-diagnostics-extension-on-an-existing-cloud-service"></a>Activer l’extension de diagnostics sur un service cloud existant
Sur un service existant, utilisez `Set-AzureServiceDiagnosticsExtension`.

```azurepowershell
$service_name = "MyService"
$diagnostics_storagename = "myservicediagnostics"
$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml" 
$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"
$primary_storagekey = (Get-AzStorageKey `
  -StorageAccountName "$diagnostics_storagename").Primary
$storage_context = New-AzStorageContext `
  -StorageAccountName $diagnostics_storagename `
  -StorageAccountKey $primary_storagekey

Set-AzureServiceDiagnosticsExtension `
  -StorageContext $storage_context `
  -DiagnosticsConfigurationPath $webrole_diagconfigpath `
  -ServiceName $service_name `
  -Slot Production `
  -Role "WebRole" 
Set-AzureServiceDiagnosticsExtension `
  -StorageContext $storage_context `
  -DiagnosticsConfigurationPath $workerrole_diagconfigpath `
  -ServiceName $service_name `
  -Slot Production `
  -Role "WorkerRole"
```

## <a name="get-current-diagnostics-extension-configuration"></a>Obtenir la configuration actuelle de l’extension de diagnostics

```azurepowershell
Get-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```


## <a name="remove-diagnostics-extension"></a>Supprimer l’extension de diagnostics

```azurepowershell
Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService"
```

Si vous avez activé l’extension des diagnostics à l’aide de `Set-AzureServiceDiagnosticsExtension` ou de `New-AzureServiceDiagnosticsExtensionConfig` sans paramètre Rôle. Vous pouvez ensuite supprimer l’extension à l’aide de `Remove-AzureServiceDiagnosticsExtension` sans paramètre Rôle. Si le paramètre Rôle a été utilisé lors de l’activation de l’extension, il doit également être utilisé au moment de sa suppression.

Pour supprimer l’extension de diagnostics de chaque rôle individuel :

```azurepowershell
Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService" -Role "WebRole"
```


## <a name="see-also"></a>Voir aussi
* [Surveiller les applications Azure Cloud Services avec Application Insights](./cloudservices.md)
* [Envoyer des diagnostics Azure vers Application Insights.](../platform/diagnostics-extension-to-application-insights.md)


