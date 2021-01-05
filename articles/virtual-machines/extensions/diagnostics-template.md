---
title: Ajouter le monitorage et les diagnostics à une machine virtuelle Azure
description: Utilisez un modèle Azure Resource Manager pour créer une machine virtuelle Windows avec l’extension Diagnostics Azure.
services: virtual-machines-windows
documentationcenter: ''
author: mimckitt
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: 8cde8fe7-977b-43d2-be74-ad46dc946058
ms.service: virtual-machines-windows
ms.subservice: extensions
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: article
ms.date: 05/31/2017
ms.author: mimckitt
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 181f226a4d7aa37ffd8c667db4736a96450e2be5
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94955954"
---
# <a name="use-monitoring-and-diagnostics-with-a-windows-vm-and-azure-resource-manager-templates"></a>Utiliser la surveillance et les diagnostics avec une machine virtuelle Windows et des modèles Azure Resource Manager
L’extension Diagnostics Azure fournit des fonctionnalités d’analyse et de diagnostics sur une machine virtuelle Azure basée sur Windows. Vous pouvez activer ces fonctionnalités sur la machine virtuelle en incluant l’extension dans le modèle Azure Resource Manager. Pour plus d’informations sur l’ajout d’une extension dans un modèle de machine virtuelle, consultez [Création de modèles Azure Resource Manager avec des extensions de machine virtuelle](../windows/template-description.md#extensions) . Cet article décrit comment ajouter l’extension Diagnostics Azure à un modèle de machine virtuelle Windows.  

## <a name="add-the-azure-diagnostics-extension-to-the-vm-resource-definition"></a>Ajouter l’extension Diagnostics Azure à la définition de ressource de machine virtuelle
Pour activer l’extension Diagnostics sur une machine virtuelle Windows, vous devez ajouter l’extension comme ressource de machine virtuelle dans le modèle Resource Manager.

Pour une simple machine virtuelle basée sur Resource Manager, ajoutez la configuration de l’extension au tableau *Ressources* de la machine virtuelle : 

```json
"resources": [
    {
        "name": "Microsoft.Insights.VMDiagnosticsSettings",
        "type": "extensions",
        "location": "[resourceGroup().location]",
        "apiVersion": "2015-06-15",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
        ],
        "tags": {
            "displayName": "AzureDiagnostics"
        },
        "properties": {
            "publisher": "Microsoft.Azure.Diagnostics",
            "type": "IaaSDiagnostics",
            "typeHandlerVersion": "1.5",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "xmlCfg": "[base64(concat(variables('wadcfgxstart'), variables('wadmetricsresourceid'), variables('vmName'), variables('wadcfgxend')))]",
                "storageAccount": "[parameters('existingdiagnosticsStorageAccountName')]"
            },
            "protectedSettings": {
                "storageAccountName": "[parameters('existingdiagnosticsStorageAccountName')]",
                "storageAccountKey": "[listkeys(variables('accountid'), '2015-05-01-preview').key1]",
                "storageAccountEndPoint": "https://core.windows.net"
            }
        }
    }
]
```

Une autre convention commune est d’ajouter la configuration de l’extension au nœud de ressources racine du modèle au lieu de la définir sous le nœud de ressources de la machine virtuelle. Avec cette approche, vous devez spécifier explicitement une relation hiérarchique entre l’extension et la machine virtuelle avec les valeurs de *nom* et de *type*. Par exemple : 

```json
"name": "[concat(variables('vmName'),'Microsoft.Insights.VMDiagnosticsSettings')]",
"type": "Microsoft.Compute/virtualMachines/extensions",
```

L’extension est toujours associée à la machine virtuelle. Vous pouvez la définir soit directement sous le nœud de ressource de la machine virtuelle, soit au niveau de base et utiliser la convention d’affectation de noms hiérarchique pour l’associer à la machine virtuelle.

Pour les jeux de mise à l’échelle de machine virtuelle, la configuration des extensions est spécifiée dans la propriété *extensionProfile* de l’élément *VirtualMachineProfile*.

La propriété *publisher* avec la valeur **Microsoft.Azure.Diagnostics** et la propriété *type* avec la valeur **IaaSDiagnostics** identifient de façon unique l’extension Diagnostics Azure.

La valeur de la propriété *name* peut être utilisée pour faire référence à l’extension dans le groupe de ressources. Le fait de la définir sur **Microsoft.Insights.VMDiagnosticsSettings** lui permet d’être facilement identifiée par le portail Azure et garantit que les graphiques de surveillance s’affichent correctement dans le portail Azure.

L’élément *typeHandlerVersion* spécifie la version de l’extension que vous souhaitez utiliser. Le fait de définir la version mineure *autoUpgradeMinorVersion* sur **true** garantit que vous obtenez la dernière version mineure de l’extension qui est disponible. Il est fortement recommandé de toujours définir *autoUpgradeMinorVersion* sur **true** afin de toujours utiliser l’extension Diagnostics la plus récente avec l’ensemble des nouvelles fonctionnalités et des correctifs de bogues. 

L’élément *settings* contient des propriétés de configuration pour l’extension pouvant être définies et lues à partir de l’extension (on parle alors parfois de configuration publique). La propriété *xmlcfg* comporte la configuration XML des journaux de diagnostic, compteurs de performances, etc. qui sont collectés par l’agent de diagnostics. Pour plus d’informations sur le schéma XML, consultez la page [Schéma de configuration des diagnostics](../../azure-monitor/platform/diagnostics-extension-schema-windows.md). Une pratique courante consiste à stocker la configuration XML réelle en tant que variable dans le modèle Azure Resource Manager, puis à la concaténer et la coder en base64 pour définir la valeur de *xmlcfg*. Consultez la section [Variables de configuration des diagnostics](#diagnostics-configuration-variables) pour en savoir plus sur la façon de stocker le code XML dans des variables. La propriété *storageAccount* spécifie le nom du compte de stockage vers lequel les données de diagnostics sont transférées. 

Les propriétés dans *protectedSettings* (parfois désignées par le terme « configuration privée ») peuvent être définies, mais ne peuvent pas être lues ensuite. La nature en écriture seule de *protectedSettings* est utile pour stocker des secrets tels que la clé de compte de stockage où les données de diagnostics sont écrites.    

## <a name="specifying-diagnostics-storage-account-as-parameters"></a>Spécification du compte de stockage de diagnostics en tant que paramètres
L’extrait de code JSON de l’extension Diagnostics ci-dessus suppose deux paramètres *existingdiagnosticsStorageAccountName* et *existingdiagnosticsStorageResourceGroup* pour spécifier le compte de stockage des données de diagnostics. La spécification du compte de stockage des diagnostics comme paramètre permet de changer facilement le compte de stockage des diagnostics dans différents environnements. Par exemple, vous pouvez utiliser un compte de stockage des diagnostics pour les tests et un autre pour le déploiement en production.  

```json
"existingdiagnosticsStorageAccountName": {
    "type": "string",
    "metadata": {
"description": "The name of an existing storage account to which diagnostics data is transfered."
    }
},
"existingdiagnosticsStorageResourceGroup": {
    "type": "string",
    "metadata": {
"description": "The resource group for the storage account specified in existingdiagnosticsStorageAccountName"
    }
}
```

Il est recommandé de spécifier un compte de stockage des diagnostics dans un groupe de ressources différent du groupe de ressources de la machine virtuelle. Un groupe de ressources peut être considéré comme une unité de déploiement avec sa propre durée de vie. Une machine virtuelle peut être déployée et redéployée à mesure que de nouvelles mises à jour de configurations sont appliquées, mais vous pouvez continuer à stocker les données de diagnostic dans le même compte de stockage lors des différents déploiements de machines virtuelles. Le fait que le compte de stockage soit dans une autre ressource lui permet d’accepter les données à partir de différents déploiements de machines virtuelles, facilitant ainsi la résolution des problèmes sur les différentes versions.

> [!NOTE]
> Si vous créez un modèle de machine virtuelle Windows à partir de Visual Studio, le compte de stockage par défaut peut être défini de manière à utiliser le même compte de stockage que celui sur lequel le disque dur virtuel de machine virtuelle est chargé. Cela permet de simplifier la configuration initiale de la machine virtuelle. Refactorisez le modèle pour qu’il utilise un autre compte de stockage qui peut être transmis en tant que paramètre. 
> 
> 

## <a name="diagnostics-configuration-variables"></a>Variables de configuration des diagnostics
L’extrait de code JSON de l’extension Diagnostics qui précède définit une variable *accountid* pour simplifier l’obtention de la clé de compte de stockage pour le stockage des diagnostics :   

```json
"accountid": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/',parameters('existingdiagnosticsStorageResourceGroup'), '/providers/','Microsoft.Storage/storageAccounts/', parameters('existingdiagnosticsStorageAccountName'))]"
```

La propriété *xmlcfg* de l’extension Diagnostics est définie à l’aide de plusieurs variables concaténées. Les valeurs de ces variables étant au format XML, elles doivent être correctement placées dans une séquence d’échappement lorsque vous définissez les variables JSON.

L’exemple suivant décrit le XML de configuration des diagnostics qui collecte les compteurs de performances au niveau du système standard, ainsi que certains journaux des événements Windows et journaux d’activité d’infrastructure de diagnostics. Il a été correctement placé dans une séquence d’échappement et mis en forme afin que la configuration puisse être collée directement dans la section des variables de votre modèle. Consultez le [schéma de configuration des diagnostics](../../azure-monitor/platform/diagnostics-extension-schema-windows.md) pour obtenir un exemple plus lisible du XML de configuration.

```json
"wadlogs": "<WadCfg> <DiagnosticMonitorConfiguration overallQuotaInMB=\"4096\" xmlns=\"http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration\"> <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter=\"Error\"/> <WindowsEventLog scheduledTransferPeriod=\"PT1M\" > <DataSource name=\"Application!*[System[(Level = 1 or Level = 2)]]\" /> <DataSource name=\"Security!*[System[(Level = 1 or Level = 2)]]\" /> <DataSource name=\"System!*[System[(Level = 1 or Level = 2)]]\" /></WindowsEventLog>",
"wadperfcounters1": "<PerformanceCounters scheduledTransferPeriod=\"PT1M\"><PerformanceCounterConfiguration counterSpecifier=\"\\Processor(_Total)\\% Processor Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"CPU utilization\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Processor(_Total)\\% Privileged Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"CPU privileged time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Processor(_Total)\\% User Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"CPU user time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Processor Information(_Total)\\Processor Frequency\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"CPU frequency\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\System\\Processes\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"Processes\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Process(_Total)\\Thread Count\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"Threads\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Process(_Total)\\Handle Count\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"Handles\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\% Committed Bytes In Use\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Memory usage\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\Available Bytes\" sampleRate=\"PT15S\" unit=\"Bytes\"><annotation displayName=\"Memory available\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\Committed Bytes\" sampleRate=\"PT15S\" unit=\"Bytes\"><annotation displayName=\"Memory committed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\Commit Limit\" sampleRate=\"PT15S\" unit=\"Bytes\"><annotation displayName=\"Memory commit limit\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\% Disk Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk active time\" locale=\"en-us\"/></PerformanceCounterConfiguration>",
"wadperfcounters2": "<PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\% Disk Read Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk active read time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\% Disk Write Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk active write time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Transfers/sec\" sampleRate=\"PT15S\" unit=\"CountPerSecond\"><annotation displayName=\"Disk operations\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Reads/sec\" sampleRate=\"PT15S\" unit=\"CountPerSecond\"><annotation displayName=\"Disk read operations\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Writes/sec\" sampleRate=\"PT15S\" unit=\"CountPerSecond\"><annotation displayName=\"Disk write operations\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Bytes/sec\" sampleRate=\"PT15S\" unit=\"BytesPerSecond\"><annotation displayName=\"Disk speed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Read Bytes/sec\" sampleRate=\"PT15S\" unit=\"BytesPerSecond\"><annotation displayName=\"Disk read speed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Write Bytes/sec\" sampleRate=\"PT15S\" unit=\"BytesPerSecond\"><annotation displayName=\"Disk write speed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\LogicalDisk(_Total)\\% Free Space\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk free space (percentage)\" locale=\"en-us\"/></PerformanceCounterConfiguration></PerformanceCounters>",
"wadcfgxstart": "[concat(variables('wadlogs'), variables('wadperfcounters1'), variables('wadperfcounters2'), '<Metrics resourceId=\"')]",
"wadmetricsresourceid": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name , '/providers/', 'Microsoft.Compute/virtualMachines/')]",
"wadcfgxend": "\"><MetricAggregation scheduledTransferPeriod=\"PT1H\"/><MetricAggregation scheduledTransferPeriod=\"PT1M\"/></Metrics></DiagnosticMonitorConfiguration></WadCfg>"
```

Le nœud XML de définition de métriques de la configuration ci-dessus est un élément de configuration important, car il indique la manière dont les compteurs de performances définis précédemment dans le fichier XML du nœud *PerformanceCounter* sont regroupés et stockés. 

> [!IMPORTANT]
> Ces mesures déterminent les graphiques et alertes de surveillance dans le portail Azure.  Le nœud **Mesures** avec les paramètres *resourceID* et **MetricAggregation** doit être inclus dans la configuration de diagnostic pour votre machine virtuelle si vous voulez que les données de surveillance de la machine virtuelle apparaissent dans le portail Azure. 
> 
> 

Voici un exemple de code XML présentant les définitions de mesures : 

```xml
<Metrics resourceId="/subscriptions/subscription().subscriptionId/resourceGroups/resourceGroup().name/providers/Microsoft.Compute/virtualMachines/vmName">
    <MetricAggregation scheduledTransferPeriod="PT1H"/>
    <MetricAggregation scheduledTransferPeriod="PT1M"/>
</Metrics>
```

L’attribut *resourceID* identifie de façon unique la machine virtuelle dans votre abonnement. Veillez à utiliser les fonctions subscription() et resourceGroup() afin que le modèle mette automatiquement à jour ces valeurs en fonction de l’abonnement et du groupe de ressources concernés par le déploiement.

Si vous créez plusieurs machines virtuelles dans une boucle, vous devez remplir la valeur *resourceID* avec une fonction copyIndex() pour différencier correctement chaque machine virtuelle. La valeur *xmlCfg* peut être mise à jour pour prendre en charge ce paramètre, comme indiqué ici :  

```json
"xmlCfg": "[base64(concat(variables('wadcfgxstart'), variables('wadmetricsresourceid'), concat(parameters('vmNamePrefix'), copyindex()), variables('wadcfgxend')))]", 
```

La valeur MetricAggregation de *PT1M* et *PT1H* fait référence à une agrégation sur une minute et à une agrégation sur une heure, respectivement.

## <a name="wadmetrics-tables-in-storage"></a>Tables WADMetrics dans le stockage
La configuration des métriques ci-dessus génère les tables de votre compte de stockage de diagnostics avec les conventions de nommage suivantes :

* **WADMetrics** : Préfixe standard pour toutes les tables WADMetrics
* **PT1H** ou **PT1M** : Indique que la table contient des données agrégées sur 1 heure ou 1 minute
* **P10D** : Indique que la table contient les données d’une période de 10 jours à partir du moment où la table a commencé à collecter les données
* **V2S** : Constante de chaîne
* **aaaammjj** : Date à laquelle la table a démarré la collecte de données

Exemple : *WADMetricsPT1HP10DV2S20151108* contient les données de métriques agrégées sur une heure pendant 10 jours à partir du 11 novembre 2015    

Chaque table WADMetrics contient les colonnes suivantes :

* **PartitionKey** : La clé de partition se base sur la valeur *resourceID* pour identifier de manière unique la ressource de machine virtuelle. Par exemple : `002Fsubscriptions:<subscriptionID>:002FresourceGroups:002F<ResourceGroupName>:002Fproviders:002FMicrosoft:002ECompute:002FvirtualMachines:002F<vmName>`  
* **RowKey** : Suit le format `<Descending time tick>:<Performance Counter Name>`. Le calcul du cycle horaire décroissant correspond aux cycles horaires maximaux moins l’heure de début de la période d’agrégation. Par exemple, si la période d’échantillonnage a démarré le 10 novembre 2015 à 00h00 UTC, le calcul est le suivant : `DateTime.MaxValue.Ticks - (new DateTime(2015,11,10,0,0,0,DateTimeKind.Utc).Ticks)`. Pour le compteur de performances d’octets disponibles en mémoire, la clé de ligne a l’aspect suivant : `2519551871999999999__:005CMemory:005CAvailable:0020Bytes`
* **CounterName** : Nom du compteur de performances. Cela correspond à l’élément *counterSpecifier* défini dans la configuration XML.
* **Maximum** : Valeur maximale du compteur de performances sur la période d’agrégation.
* **Minimale** : Valeur minimale du compteur de performances sur la période d’agrégation.
* **Total** : Somme de toutes les valeurs du compteur de performances signalées sur la période d’agrégation.
* **Nombre** : Nombre total de valeurs signalées pour le compteur de performances.
* **Average** : Valeur moyenne (total/count) du compteur de performances sur la période d’agrégation.

## <a name="next-steps"></a>Étapes suivantes
* Pour obtenir un exemple de modèle complet d’une machine virtuelle Windows avec l’extension Diagnostics, consultez [201-vm-monitoring-diagnostics-extension](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-monitoring-diagnostics-extension)   
* Déployer le modèle Azure Resource Manager à l’aide [d’Azure PowerShell](../windows/ps-template.md) ou de la [ligne de commande Azure](../linux/create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* En savoir plus sur la [création de modèles Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md)