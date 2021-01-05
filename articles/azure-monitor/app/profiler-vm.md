---
title: Profilez des applications web sur une machine virtuelle Azure - Application Insights Profiler
description: Profilez des applications web sur une machine virtuelle Azure avec Application Insights Profiler.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 11/08/2019
ms.reviewer: mbullwin
ms.openlocfilehash: f514dd7b54ac091535aeab43a8a7d2a645b50a09
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87315816"
---
# <a name="profile-web-apps-running-on-an-azure-virtual-machine-or-a-virtual-machine-scale-set-by-using-application-insights-profiler"></a>Profiler des applications web s’exécutant sur une machine virtuelle Azure ou un groupe de machines virtuelles identiques avec Application Insights Profiler

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Vous pouvez également déployer Azure Application Insights Profiler sur ces services :
* [Azure App Service](./profiler.md?toc=%2fazure%2fazure-monitor%2ftoc.json)
* [Azure Cloud Services](profiler-cloudservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Service Fabric](?toc=%2fazure%2fazure-monitor%2ftoc.json)

## <a name="deploy-profiler-on-a-virtual-machine-or-a-virtual-machine-scale-set"></a>Déployer Profiler sur une machine virtuelle ou un groupe de machines virtuelles identiques
Cet article vous montre comment exécuter Application Insights Profiler sur votre machine virtuelle Azure ou votre groupe de machines virtuelles identiques Azure. Profiler est installé avec l’extension Diagnostics Azure pour machines virtuelles. Configurez l’extension pour exécuter Profiler et générez le SDK Application Insights dans votre application.

1. Ajoutez le Kit de développement logiciel (SDK) Application Insights SDK à votre [application ASP.NET](./asp-net.md).

   Pour voir les profils de vos demandes, vous devez envoyer les données de télémétrie des demandes à Application Insights.

1. Installez l’extension Diagnostics Azure sur votre machine virtuelle. Pour obtenir des exemples complets de modèles Resource Manager, consultez :  
   * [Machine virtuelle](https://github.com/Azure/azure-docs-json-samples/blob/master/application-insights/WindowsVirtualMachine.json)
   * [Groupe de machines virtuelles identiques](https://github.com/Azure/azure-docs-json-samples/blob/master/application-insights/WindowsVirtualMachineScaleSet.json)
    
     La partie clé est ApplicationInsightsProfilerSink dans WadCfg. Pour que Diagnostics Azure autorise Profiler à envoyer des données à votre iKey, ajoutez un autre récepteur à cette section.
    
     ```json
     "SinksConfig": {
       "Sink": [
         {
           "name": "ApplicationInsightsSink",
           "ApplicationInsights": "85f73556-b1ba-46de-9534-606e08c6120f"
         },
         {
           "name": "MyApplicationInsightsProfilerSink",
           "ApplicationInsightsProfiler": "85f73556-b1ba-46de-9534-606e08c6120f"
         }
       ]
     },
     ```

1. Déployez la définition du déploiement d’environnement modifiée.  

   L’application des modifications implique généralement un déploiement de modèle complet ou une publication basée sur les services cloud par le biais d’applets de commande PowerShell ou Visual Studio.  

   Les commandes PowerShell suivantes offrent une autre approche pour les machines virtuelles existantes, qui touchent uniquement l’extension Diagnostics Azure. Ajoutez le ProfilerSink mentionné précédemment à la configuration retournée par la commande Get-AzVMDiagnosticsExtension. Passez ensuite la configuration mise à jour à la commande Set-AzVMDiagnosticsExtension.

    ```powershell
    $ConfigFilePath = [IO.Path]::GetTempFileName()
    # After you export the currently deployed Diagnostics config to a file, edit it to include the ApplicationInsightsProfiler sink.
    (Get-AzVMDiagnosticsExtension -ResourceGroupName "MyRG" -VMName "MyVM").PublicSettings | Out-File -Verbose $ConfigFilePath
    # Set-AzVMDiagnosticsExtension might require the -StorageAccountName argument
    # If your original diagnostics configuration had the storageAccountName property in the protectedSettings section (which is not downloadable), be sure to pass the same original value you had in this cmdlet call.
    Set-AzVMDiagnosticsExtension -ResourceGroupName "MyRG" -VMName "MyVM" -DiagnosticsConfigurationPath $ConfigFilePath
    ```

1. Si l’application souhaitée s’exécute avec [IIS](https://www.microsoft.com/web/downloads/platform.aspx), activez la fonctionnalité Windows `IIS Http Tracing`.

   a. Établissez un accès à distance à l’environnement, puis utilisez la fenêtre [Ajouter des fonctionnalités Windows](/iis/configuration/system.webserver/tracing/). Sinon, exécutez la commande suivante dans PowerShell (comme administrateur) :  

    ```powershell
    Enable-WindowsOptionalFeature -FeatureName IIS-HttpTracing -Online -All
    ```  
   b. Si l’établissement de l’accès à distance est un problème, vous pouvez utiliser [Azure CLI](/cli/azure/get-started-with-azure-cli) pour exécuter la commande suivante :  

    ```powershell
    az vm run-command invoke -g MyResourceGroupName -n MyVirtualMachineName --command-id RunPowerShellScript --scripts "Enable-WindowsOptionalFeature -FeatureName IIS-HttpTracing -Online -All"
    ```

1. Déployez votre application.

## <a name="set-profiler-sink-using-azure-resource-explorer"></a>Définir le récepteur du profileur en utilisant Azure Resource Explorer
Nous n’avons pas encore trouvé le moyen de définir l’application Insights Profiler à partir du portail. Au lieu d’utiliser Powerhell comme décrit ci-dessus, vous pouvez utiliser Azure Resource Explorer pour définir le récepteur. Mais notez que, si vous redéployez la machine virtuelle, le récepteur sera perdu. Vous devez mettre à jour la configuration que vous utilisez lorsque vous déployez la machine virtuelle pour conserver ce paramètre.

1. Vérifiez que l’extension Diagnostics Azure pour Windows est installée en consultant les extensions installées dans votre machine virtuelle.  

    ![Vérification de l’installation de l’extension Diagnostics Azure pour Windows][wadextension]

2. Recherchez l’extension VM Diagnostics pour votre machine virtuelle. Accédez à [https://resources.azure.com](https://resources.azure.com). Développez votre groupe de ressources, Microsoft.Compute virtualMachines, le nom de votre machine virtuelle et ses extensions.  

    ![Accédez à la configuration de Diagnostics Azure pour Windows dans Azure Resource Explorer][azureresourceexplorer]

3. Ajoutez le récepteur Application Insights Profiler sur le nœud SinksConfig sous WadCfg. Si vous n’avez pas déjà une section SinksConfig, vous devrez peut-être en ajouter une. Assurez-vous de spécifier l’iKey Application Insights appropriée dans vos paramètres. Vous devrez basculer du mode Explorateur au mode Lecture/Écriture situé dans le coin supérieur droit, puis appuyer sur le bouton bleu « Modifier ».

    ![Ajouter le récepteur d’Application Insights Profiler][resourceexplorersinksconfig]

4. Lorsque vous avez terminé la modification de la configuration, appuyez sur « Put » (« Placer »). Si cette opération ne réussit pas, une coche verte s’affiche au milieu de l’écran.

    ![Envoyer une requête PUT pour appliquer les modifications][resourceexplorerput]






## <a name="can-profiler-run-on-on-premises-servers"></a>Profiler peut-il être exécuté sur des serveurs locaux ?
Nous n’envisageons pas de prendre en charge Application Insights Profiler pour les serveurs locaux.

## <a name="next-steps"></a>Étapes suivantes

- Générez le trafic vers votre application (par exemple, lancez un [test de disponibilité](monitor-web-app-availability.md)). Attendez ensuite 10 à 15 minutes le temps que les traces soient envoyées à l’instance Application Insights.
- Consultez [Traces du profileur](profiler-overview.md?toc=/azure/azure-monitor/toc.json) dans le portail Azure.
- Pour obtenir de l'aide sur la résolution des problèmes liés au profileur, consultez [Résolution des problèmes liés au profileur](profiler-troubleshooting.md?toc=/azure/azure-monitor/toc.json).

[azureresourceexplorer]: ./media/profiler-vm/azure-resource-explorer.png
[resourceexplorerput]: ./media/profiler-vm/resource-explorer-put.png
[resourceexplorersinksconfig]: ./media/profiler-vm/resource-explorer-sinks-config.png
[wadextension]: ./media/profiler-vm/wad-extension.png

