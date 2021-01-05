---
title: Collecter les mesures de groupes identiques Windows dans Azure Monitor avec un modèle
description: Envoyer des métriques de système d’exploitation invité vers le magasin de métriques d’Azure Monitor à l’aide d’un modèle Resource Manager pour un groupe de machines virtuelles identiques Windows
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 09/09/2019
ms.author: ancav
ms.subservice: metrics
ms.openlocfilehash: 76f73df01b34cb20be48aefa3b5b3a6392a35b8b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87045193"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metric-store-by-using-an-azure-resource-manager-template-for-a-windows-virtual-machine-scale-set"></a>Envoyer des métriques de système d’exploitation invité vers le magasin de métriques d’Azure Monitor à l’aide d’un modèle Azure Resource Manager pour un groupe de machines virtuelles identiques Windows

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

En utilisant l’[extension Diagnostics Azure pour Windows (WAD)](diagnostics-extension-overview.md) d’Azure Monitor, vous pouvez collecter des métriques et des journaux à partir du système d’exploitation invité qui est exécuté dans le cadre d’une machine virtuelle, d’un service cloud ou d’un cluster Azure Service Fabric. L’extension peut envoyer des données de télémétrie vers de nombreux emplacements différents répertoriés dans l’article précédemment lié.  

Cet article décrit le processus pour envoyer les métriques de performance du SE invité d’un groupe de machines virtuelles identiques Windows vers le magasin de données d’Azure Monitor. À partir de la version 1.11 de Diagnostics Azure pour Windows, vous pouvez écrire des métriques directement dans le magasin de métriques d’Azure Monitor, où les métriques standard de la plateforme sont déjà collectées. En les stockant dans cet emplacement, vous avez accès aux mêmes actions que pour les métriques de la plateforme. Ces actions sont les suivantes : génération d’alertes en temps quasi réel, création de graphiques, routage, accès à partir de l’API REST et bien plus encore. Auparavant, l’extension Diagnostics Azure pour Windows écrivait les données dans le Stockage Azure, et non dans le magasin de données d’Azure Monitor.  

Si vous découvrez les modèles Resource Manager, obtenez plus d’informations sur les [déploiements de modèle](../../azure-resource-manager/management/overview.md), leur structure et leur syntaxe.  

## <a name="prerequisites"></a>Conditions préalables requises

- Votre abonnement doit être inscrit auprès de [Microsoft.Insights](../../azure-resource-manager/management/resource-providers-and-types.md). 

- Vous devez avoir installé [Azure PowerShell](/powershell/azure), ou vous pouvez utiliser [Azure Cloud Shell](../../cloud-shell/overview.md). 

- Votre ressource de machine virtuelle doit se trouver dans une région [ prenant en charge les métriques personnalisées](metrics-custom-overview.md#supported-regions).

## <a name="set-up-azure-monitor-as-a-data-sink"></a>Configurer Azure Monitor en tant que récepteur de données 
L’extension Diagnostics Azure utilise une fonctionnalité appelée **récepteurs de données** pour acheminer les métriques et les journaux vers différents emplacements. Les étapes suivantes montrent comment utiliser un modèle Resource Manager et PowerShell pour déployer une machine virtuelle à l’aide du nouveau récepteur de données Azure Monitor. 

## <a name="author-a-resource-manager-template"></a>Créer un modèle Resource Manager 
Pour cet exemple, vous pouvez utiliser un [exemple de modèle](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-autoscale) public :  

- **Azuredeploy.json** est un modèle Resource Manager préconfiguré pour le déploiement d’un groupe de machines virtuelles identiques.

- **Azuredeploy.parameters.json** est un fichier de paramètres qui stocke des informations telles que le nom d’utilisateur et le mot de passe que vous souhaitez définir pour votre machine virtuelle. Pendant le déploiement, le modèle Resource Manager utilise les paramètres définis dans ce fichier. 

Téléchargez et enregistrez les deux fichiers en local. 

###  <a name="modify-azuredeployparametersjson"></a>Modifier le fichier azuredeploy.parameters.json
Ouvrez le fichier **azuredeploy.parameters.json** :  
 
- Indiquez le **vmSKU** que vous souhaitez déployer. Nous vous recommandons de choisir « Standard_D2_v3 ». 
- Spécifiez la version **windowsOSVersion** que vous souhaitez pour votre groupe de machines virtuelles identiques. Nous vous recommandons de choisir « 2016-Datacenter ». 
- Attribuez un nom à la ressource du groupe de machines virtuelles identiques à déployer à l’aide d’une propriété **vmssName**. Par exemple, **VMSS-WAD-TEST**.    
- Spécifiez le nombre de machines virtuelles que vous souhaitez exécuter sur le groupe de machines virtuelles identiques à l’aide de la propriété **instanceCount**.
- Entrez les valeurs **adminUsername** et **adminPassword** pour le groupe de machines virtuelles identiques. Ces paramètres sont utilisés pour l’accès à distance aux machines virtuelles du groupe identique. Pour éviter tout accès non autorisé à votre machine virtuelle, **n’utilisez pas** les paramètres de ce modèle. Les bots recherchent sur Internet des noms d’utilisateur et mots de passe dans les référentiels GitHub publics. Ils sont susceptibles de tester les machines virtuelles avec ces valeurs par défaut. 


###  <a name="modify-azuredeployjson"></a>Modifier le fichier azuredeploy.json
Ouvrez le fichier **azuredeploy.json**. 

Ajoutez une variable pour contenir les informations de compte de stockage dans le modèle Resource Manager. Les journaux d’activité ou les compteurs de performances spécifiés dans le fichier de configuration des diagnostics sont écrits dans le magasin de métriques d’Azure Monitor et dans le compte de stockage que vous indiquez ici : 

```json
"variables": { 
//add this line       
"storageAccountName": "[concat('storage', uniqueString(resourceGroup().id))]", 
```
 
Recherchez la définition du groupe de machines virtuelles identiques dans la section des ressources et ajoutez la section **identité** à la configuration. Vous garantissez ainsi qu’Azure lui attribue une identité système. Cette étape garantit que les machines virtuelles du groupe identique peuvent transmettre leurs propres métriques d’invité à Azure Monitor.  

```json
    { 
      "type": "Microsoft.Compute/virtualMachineScaleSets", 
      "name": "[variables('namingInfix')]", 
      "location": "[resourceGroup().location]", 
      "apiVersion": "2017-03-30", 
      //add these lines below
      "identity": { 
           "type": "systemAssigned" 
       }, 
       //end of lines to add
```

Dans la ressource du groupe de machines virtuelles identiques, recherchez la section **virtualMachineProfile**. Ajoutez un nouveau profil nommé **extensionsProfile** pour gérer les extensions.  


Dans **extensionProfile**, ajoutez une nouvelle extension au modèle, comme illustré dans la **section VMSS-WAD-extension**.  Cette section correspond à l’extension des identités managées pour ressources Azure qui garantit que les métriques émises sont acceptées par Azure Monitor. Le champ du **nom** peut contenir n’importe quel nom. 

Le code suivant émis par l’extension MSI ajoute également l’extension de diagnostics et la configuration en tant que ressource d’extension à la ressource du groupe de machines virtuelles identiques. N’hésitez pas à ajouter ou supprimer des compteurs de performances si nécessaire : 

```json
          "extensionProfile": { 
            "extensions": [ 
            // BEGINNING of added code  
            // Managed identities for Azure resources   
                { 
                 "name": "VMSS-WAD-extension", 
                 "properties": { 
                       "publisher": "Microsoft.ManagedIdentity", 
                       "type": "ManagedIdentityExtensionForWindows", 
                       "typeHandlerVersion": "1.0", 
                       "autoUpgradeMinorVersion": true, 
                       "settings": { 
                             "port": 50342 
                           }, 
                       "protectedSettings": {} 
                     } 
                                
            }, 
            // add diagnostic extension. (Remove this comment after pasting.)
            { 
              "name": "[concat('VMDiagnosticsVmExt','_vmNodeType0Name')]", 
              "properties": { 
                   "type": "IaaSDiagnostics", 
                   "autoUpgradeMinorVersion": true, 
                   "protectedSettings": { 
                        "storageAccountName": "[variables('storageAccountName')]", 
                        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')),'2015-05-01-preview').key1]", 
                        "storageAccountEndPoint": "https://core.windows.net/" 
                   }, 
                   "publisher": "Microsoft.Azure.Diagnostics", 
                   "settings": { 
                        "WadCfg": { 
                              "DiagnosticMonitorConfiguration": { 
                                   "overallQuotaInMB": "50000", 
                                   "PerformanceCounters": { 
                                       "scheduledTransferPeriod": "PT1M", 
                                       "sinks": "AzMonSink", 
                                       "PerformanceCounterConfiguration": [ 
                                          { 
              "counterSpecifier": "\\Memory\\% Committed Bytes In Use", 
                                              "sampleRate": "PT15S" 
           }, 
           { 
              "counterSpecifier": "\\Memory\\Available Bytes", 
              "sampleRate": "PT15S" 
           }, 
           { 
              "counterSpecifier": "\\Memory\\Committed Bytes", 
              "sampleRate": "PT15S" 
           } 
                                       ] 
                                 }, 
                                 "EtwProviders": { 
                                       "EtwEventSourceProviderConfiguration": [ 
                                           { 
                                              "provider": "Microsoft-ServiceFabric-Actors", 
                                              "scheduledTransferKeywordFilter": "1", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                              "eventDestination": "ServiceFabricReliableActorEventTable" 
                                           } 
                                           }, 
                                           { 
                                              "provider": "Microsoft-ServiceFabric-Services", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                                   "eventDestination": "ServiceFabricReliableServiceEventTable" 
                                              } 
                                           } 
                                     ], 
                                     "EtwManifestProviderConfiguration": [ 
                                           { 
                                              "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8", 
                                              "scheduledTransferLogLevelFilter": "Information", 
                                              "scheduledTransferKeywordFilter": "4611686018427387904", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                                   "eventDestination": "ServiceFabricSystemEventTable" 
                                              } 
                                          } 
                                     ] 
                               } 
                               }, 
                               "SinksConfig": { 
                                     "Sink": [ 
                                          { 
                                              "name": "AzMonSink", 
                                              "AzureMonitor": {} 
                                          } 
                                      ] 
                               } 
                         }, 
                         "StorageAccount": "[variables('storageAccountName')]" 
                         }, 
                        "typeHandlerVersion": "1.11" 
                  } 
           } 
            ] 
          }
          }
      }
    },
    //end of added code plus a few brackets. Be sure that the number and type of brackets match properly when done. 
    {
      "type": "Microsoft.Insights/autoscaleSettings",
...
```


Ajoutez une propriété **dependsOn** pour le compte de stockage, pour vous assurer qu’il est créé dans le bon ordre : 

```json
"dependsOn": [ 
"[concat('Microsoft.Network/loadBalancers/', variables('loadBalancerName'))]", 
"[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]" 
//add this line below
"[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]" 
```

Créez un compte de stockage s’il n’en existe pas déjà un dans le modèle : 

```json
"resources": [
// add this code    
{
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2015-05-01-preview",
    "location": "[resourceGroup().location]",
    "properties": {
    "accountType": "Standard_LRS"
    }
},
// end added code
{ 
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[variables('virtualNetworkName')]",
```

Enregistrez et fermez les deux fichiers. 

## <a name="deploy-the-resource-manager-template"></a>Déployer le modèle Resource Manager 

> [!NOTE]  
> Vous devez exécuter l’extension Diagnostics Azure version 1.5 ou ultérieure **et** la valeur de la propriété **autoUpgradeMinorVersion** doit être définie sur **true** dans votre modèle Resource Manager. Azure charge alors l’extension appropriée lorsqu’il démarre la machine virtuelle. Si ces paramètres ne figurent pas dans votre modèle, modifiez-les et redéployez le modèle. 


Pour déployer le modèle Resource Manager, utilisez Azure PowerShell :  

1. Lancez PowerShell. 
1. Connectez-vous au portail Azure à l’aide de `Login-AzAccount`.
1. Obtenez la liste de vos abonnements à l’aide de `Get-AzSubscription`.
1. Définissez l’abonnement que vous allez créer ou mettez à jour la machine virtuelle : 

   ```powershell
   Select-AzSubscription -SubscriptionName "<Name of the subscription>" 
   ```
1. Créez un nouveau groupe de ressources pour la machine virtuelle déployée. Exécutez la commande suivante : 

   ```powershell
    New-AzResourceGroup -Name "VMSSWADtestGrp" -Location "<Azure Region>" 
   ```

   > [!NOTE]  
   > N’oubliez pas d’utiliser une région Azure dans laquelle les mesures personnalisées sont activées. N’oubliez pas d’utiliser une [région Azure dans laquelle les mesures personnalisées sont activées](./metrics-custom-overview.md#supported-regions).
 
1. Exécutez les commandes suivantes pour déployer la machine virtuelle :  

   > [!NOTE]  
   > Si vous souhaitez mettre à jour un groupe identique existant, ajoutez **-Mode Incremental** à la fin de la commande. 
 
   ```powershell
   New-AzResourceGroupDeployment -Name "VMSSWADTest" -ResourceGroupName "VMSSWADtestGrp" -TemplateFile "<File path of your azuredeploy.JSON file>" -TemplateParameterFile "<File path of your azuredeploy.parameters.JSON file>"  
   ```

1. Lorsque votre déploiement a réussi, le groupe de machines virtuelles identiques doit figurer dans le portail Azure. De plus, il doit émettre des métriques vers Azure Monitor. 

   > [!NOTE]  
   > Des erreurs liées à la **vmSkuSize** sélectionnée sont susceptibles de se produire. Dans ce cas, revenez au fichier **azuredeploy.json** et mettez à jour la valeur par défaut pour le paramètre **vmSkuSize**. Nous vous recommandons d’essayer **Standard_DS1_v2**. 


## <a name="chart-your-metrics"></a>Représenter vos métriques graphiquement 

1. Connectez-vous au portail Azure. 

1. Dans le menu de gauche, sélectionnez **Surveiller**. 

1. Sur la page **Surveiller**, sélectionnez **Métriques**. 

   ![Surveiller – Page des métriques](media/collect-custom-metrics-guestos-resource-manager-vmss/metrics.png) 

1. Modifiez la période d’agrégation afin d’indiquer **30 dernières minutes**.  

1. Dans le menu déroulant des ressources, sélectionnez le groupe de machines virtuelles identiques que vous venez de créer.  

1. Dans le menu déroulant des espaces de noms, sélectionnez **azure.vm.windows.guest**. 

1. Dans le menu déroulant des métriques, sélectionnez **Mémoire\%Octets validés utilisés**.  

Vous pouvez ensuite également choisir d’utiliser les dimensions de cette métrique afin de la représenter graphiquement pour une machine virtuelle spécifique, ou de tracer chaque machine virtuelle du groupe identique. 



## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [métriques personnalisées](metrics-custom-overview.md).
