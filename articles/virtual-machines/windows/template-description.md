---
title: Machines virtuelles dans un modèle Azure Resource Manager | Microsoft Azure
description: En savoir plus sur la manière dont la ressource de machine virtuelle est définie dans un modèle Azure Resource Manager.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure
ms.topic: how-to
ms.date: 01/03/2019
ms.author: cynthn
ms.openlocfilehash: 228814db76ea14fa8b74c0c5d634e5afd6d54ac6
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483043"
---
# <a name="virtual-machines-in-an-azure-resource-manager-template"></a>Machines virtuelles dans un modèle Azure Resource Manager

Cet article décrit les aspects d’un modèle Azure Resource Manager qui s’appliquent à des machines virtuelles. Cet article ne décrit pas un modèle complet pour la création d’une machine virtuelle ; pour cela, vous avez besoin de définitions de ressource pour des comptes de stockage, d'interfaces réseau, d'adresses IP publiques et de réseaux virtuels. Pour plus d’informations sur la façon dont ces ressources peuvent être définies, consultez le [Guide de création d’un modèle Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

Il existe de nombreux [modèles dans la galerie](https://azure.microsoft.com/documentation/templates/?term=VM), qui incluent la ressource de la machine virtuelle. Les éléments qui peuvent être inclus dans un modèle ne sont tous décrits ici.

 

Cet exemple montre une section de ressources standard d’un modèle pour la création d’un nombre spécifié de machines virtuelles :

```json
"resources": [
  { 
    "apiVersion": "2016-04-30-preview", 
    "type": "Microsoft.Compute/virtualMachines", 
    "name": "[concat('myVM', copyindex())]", 
    "location": "[resourceGroup().location]",
    "copy": {
      "name": "virtualMachineLoop", 
      "count": "[parameters('numberOfInstances')]"
    },
    "dependsOn": [
      "[concat('Microsoft.Network/networkInterfaces/myNIC', copyindex())]" 
    ], 
    "properties": { 
      "hardwareProfile": { 
        "vmSize": "Standard_DS1" 
      }, 
      "osProfile": { 
        "computername": "[concat('myVM', copyindex())]", 
        "adminUsername": "[parameters('adminUsername')]", 
        "adminPassword": "[parameters('adminPassword')]" 
      }, 
      "storageProfile": { 
        "imageReference": { 
          "publisher": "MicrosoftWindowsServer", 
          "offer": "WindowsServer", 
          "sku": "2012-R2-Datacenter", 
          "version": "latest" 
        }, 
        "osDisk": { 
          "name": "[concat('myOSDisk', copyindex())]",
          "caching": "ReadWrite", 
          "createOption": "FromImage" 
        },
        "dataDisks": [
          {
            "name": "[concat('myDataDisk', copyindex())]",
            "diskSizeGB": "100",
            "lun": 0,
            "createOption": "Empty"
          }
        ] 
      }, 
      "networkProfile": { 
        "networkInterfaces": [ 
          { 
            "id": "[resourceId('Microsoft.Network/networkInterfaces',
              concat('myNIC', copyindex()))]" 
          } 
        ] 
      },
      "diagnosticsProfile": {
        "bootDiagnostics": {
          "enabled": "true",
          "storageUri": "[concat('https://', variables('storageName'), '.blob.core.windows.net')]"
        }
      } 
    },
    "resources": [ 
      { 
        "name": "Microsoft.Insights.VMDiagnosticsSettings", 
        "type": "extensions", 
        "location": "[resourceGroup().location]", 
        "apiVersion": "2016-03-30", 
        "dependsOn": [ 
          "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]" 
        ], 
        "properties": { 
          "publisher": "Microsoft.Azure.Diagnostics", 
          "type": "IaaSDiagnostics", 
          "typeHandlerVersion": "1.5", 
          "autoUpgradeMinorVersion": true, 
          "settings": { 
            "xmlCfg": "[base64(concat(variables('wadcfgxstart'), 
            variables('wadmetricsresourceid'), 
            concat('myVM', copyindex()),
            variables('wadcfgxend')))]", 
            "storageAccount": "[variables('storageName')]" 
          }, 
          "protectedSettings": { 
            "storageAccountName": "[variables('storageName')]", 
            "storageAccountKey": "[listkeys(variables('accountid'), 
              '2015-06-15').key1]", 
            "storageAccountEndPoint": "https://core.windows.net" 
          } 
        } 
      },
      {
        "name": "MyCustomScriptExtension",
        "type": "extensions",
        "apiVersion": "2016-03-30",
        "location": "[resourceGroup().location]",
        "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]"
        ],
        "properties": {
          "publisher": "Microsoft.Compute",
          "type": "CustomScriptExtension",
          "typeHandlerVersion": "1.7",
          "autoUpgradeMinorVersion": true,
          "settings": {
            "fileUris": [
              "[concat('https://', variables('storageName'),
                '.blob.core.windows.net/customscripts/start.ps1')]" 
            ],
            "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File start.ps1"
          }
        }
      } 
    ]
  } 
]
``` 

> [!NOTE] 
>Cet exemple repose sur un compte de stockage créé précédemment. Vous pouvez créer le compte de stockage en le déployant à partir du modèle. L’exemple s’appuie également sur une interface réseau et ses ressources dépendantes qui seraient définies dans le modèle. Ces ressources ne sont pas affichées dans l’exemple.
>
>

## <a name="api-version"></a>Version de l'API

Lorsque vous déployez des ressources à l’aide d’un modèle, vous devez spécifier une version de l’API à utiliser. L’exemple montre la ressource de la machine virtuelle en utilisant cet élément apiVersion :

```json
"apiVersion": "2016-04-30-preview",
```

La version de l'API que vous spécifiez dans votre modèle affecte les propriétés que vous pouvez définir dans le modèle. En général, vous devez sélectionner la version de l'API la plus récente lorsque vous créez des modèles. Pour les modèles existants, vous pouvez choisir soit de continuer d’utiliser une version de l'API antérieure, soit de mettre à jour votre modèle à la dernière version afin de tirer parti des nouvelles fonctionnalités.

Utilisez ces opportunités pour obtenir les dernières versions de l'API :

- API REST - [Répertorier tous les fournisseurs de ressources](/rest/api/resources/providers)
- PowerShell - [Get-AzResourceProvider](/powershell/module/az.resources/get-azresourceprovider)
- Azure CLI - [az provider show](/cli/azure/provider)


## <a name="parameters-and-variables"></a>Paramètres et variables

Les [paramètres](../../azure-resource-manager/templates/template-syntax.md) vous permettent de spécifier facilement les valeurs du modèle lors de son exécution. Cette section de paramètres est utilisée dans l’exemple :

```json
"parameters": {
  "adminUsername": { "type": "string" },
  "adminPassword": { "type": "securestring" },
  "numberOfInstances": { "type": "int" }
},
```

Lorsque vous déployez l’exemple de modèle, vous entrez les valeurs de nom et de mot de passe du compte d’administrateur sur chaque machine virtuelle ainsi que le nombre de machines virtuelles à créer. Vous pouvez spécifier les valeurs de paramètres dans un fichier distinct qui est géré avec le modèle, ou fournir ces valeurs lorsque vous y êtes invité.

Les [variables](../../azure-resource-manager/templates/template-syntax.md) vous permettent de définir facilement les valeurs utilisées à plusieurs reprises tout au long du modèle ou qui peuvent changer au fil du temps. Cette section de variables est utilisée dans l’exemple :

```json
"variables": { 
  "storageName": "mystore1",
  "accountid": "[concat('/subscriptions/', subscription().subscriptionId, 
    '/resourceGroups/', resourceGroup().name,
  '/providers/','Microsoft.Storage/storageAccounts/', variables('storageName'))]", 
  "wadlogs": "<WadCfg> 
    <DiagnosticMonitorConfiguration overallQuotaInMB=\"4096\" xmlns=\"http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration\"> 
      <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter=\"Error\"/> 
      <WindowsEventLog scheduledTransferPeriod=\"PT1M\" > 
        <DataSource name=\"Application!*[System[(Level = 1 or Level = 2)]]\" /> 
        <DataSource name=\"Security!*[System[(Level = 1 or Level = 2)]]\" /> 
        <DataSource name=\"System!*[System[(Level = 1 or Level = 2)]]\" />
      </WindowsEventLog>", 
  "wadperfcounters": "<PerformanceCounters scheduledTransferPeriod=\"PT1M\">
      <PerformanceCounterConfiguration counterSpecifier=\"\\Process(_Total)\\Thread Count\" sampleRate=\"PT15S\" unit=\"Count\">
        <annotation displayName=\"Threads\" locale=\"en-us\"/>
      </PerformanceCounterConfiguration>
    </PerformanceCounters>", 
  "wadcfgxstart": "[concat(variables('wadlogs'), variables('wadperfcounters'), 
    '<Metrics resourceId=\"')]", 
  "wadmetricsresourceid": "[concat('/subscriptions/', subscription().subscriptionId, 
    '/resourceGroups/', resourceGroup().name , 
    '/providers/', 'Microsoft.Compute/virtualMachines/')]", 
  "wadcfgxend": "\"><MetricAggregation scheduledTransferPeriod=\"PT1H\"/>
    <MetricAggregation scheduledTransferPeriod=\"PT1M\"/>
    </Metrics></DiagnosticMonitorConfiguration>
    </WadCfg>"
}, 
```

Lorsque vous déployez l'exemple de modèle, les valeurs de variables sont utilisées pour le nom et l’identificateur du compte de stockage créé précédemment. Les variables servent également à fournir les paramètres pour l’extension de diagnostic. Utilisez les [bonnes pratiques relatives à la création de modèles Azure Resource Manager](../../azure-resource-manager/templates/template-best-practices.md) pour choisir comment structurer les paramètres et les variables de votre modèle.

## <a name="resource-loops"></a>boucles de ressources

Lorsque vous avez besoin de plusieurs machines virtuelles pour votre application, vous pouvez utiliser un élément copy dans un modèle. Cet élément facultatif effectue une boucle en créant le nombre de machines virtuelles que vous avez spécifié en tant que paramètre :

```json
"copy": {
  "name": "virtualMachineLoop", 
  "count": "[parameters('numberOfInstances')]"
},
```

En outre, notez dans l’exemple que l’index de la boucle est utilisé pour spécifier les valeurs de la ressource. Par exemple, si vous avez entré un nombre d’instances de trois, les noms des disques du système d’exploitation sont myOSDisk1, myOSDisk2 et myOSDisk3 :

```json
"osDisk": { 
  "name": "[concat('myOSDisk', copyindex())]",
  "caching": "ReadWrite", 
  "createOption": "FromImage" 
}
```

> [!NOTE] 
>Cet exemple utilise des disques managés pour les machines virtuelles.
>
>

N’oubliez pas que pour créer une boucle pour une ressource dans le modèle, vous devez utiliser la boucle lors de la création ou de l’accès à d’autres ressources. Par exemple, plusieurs machines virtuelles ne peuvent pas utiliser la même interface réseau. Par conséquent, si votre modèle effectue une boucle en créant trois machines virtuelles, il doit également effectuer une boucle en créant trois interfaces réseau. Lorsque vous affectez une interface réseau à une machine virtuelle, l’index de la boucle est utilisé pour l’identifier :

```json
"networkInterfaces": [ { 
  "id": "[resourceId('Microsoft.Network/networkInterfaces',
    concat('myNIC', copyindex()))]" 
} ]
```

## <a name="dependencies"></a>Les dépendances

La plupart des ressources dépendent d’autres ressources pour fonctionner correctement. Les machines virtuelles doivent être associées à un réseau virtuel et pour cela, une interface réseau est requise. L'élément [dependsOn](../../azure-resource-manager/templates/define-resource-dependency.md) permet de vérifier que l’interface réseau est prête à être utilisée avant la création des machines virtuelles :

```json
"dependsOn": [
  "[concat('Microsoft.Network/networkInterfaces/', 'myNIC', copyindex())]" 
],
```

Resource Manager déploie en parallèle toutes les ressources qui ne dépendent pas d’une autre ressource en cours de déploiement. Soyez prudent lors de la définition des dépendances car vous pouvez accidentellement ralentir votre déploiement en spécifiant des dépendances inutiles. Les dépendances peuvent couvrir plusieurs ressources. Par exemple, l’interface réseau dépend de l’adresse IP publique et des ressources du réseau virtuel.

Comment savoir si une dépendance est nécessaire ? Examinez les valeurs que vous avez définies dans le modèle. Si un élément dans la définition des ressources de la machine virtuelle pointe vers une autre ressource déployée dans le même modèle, vous avez besoin d’une dépendance. Par exemple, votre exemple de machine virtuelle définit un profil de réseau :

```json
"networkProfile": { 
  "networkInterfaces": [ { 
    "id": "[resourceId('Microsoft.Network/networkInterfaces',
      concat('myNIC', copyindex())]" 
  } ] 
},
```

Pour définir cette propriété, l’interface réseau doit exister. Vous avez donc besoin d’une dépendance. Vous devez également définir une dépendance lorsqu’une ressource (un enfant) est définie dans une autre ressource (un parent). Par exemple, les paramètres de diagnostic et les extensions de script personnalisé sont définies en tant que ressources enfants de la machine virtuelle. Ils ne peuvent pas être créés tant que la machine virtuelle n’existe pas. Par conséquent, les deux ressources sont marquées comme dépendantes de la machine virtuelle.

## <a name="profiles"></a>Profils

Plusieurs éléments de profil sont utilisés lors de la définition d’une ressource de machine virtuelle. Certains sont obligatoires, et d’autres facultatifs. Par exemple, les éléments hardwareProfile, osProfile, storageProfile et networkProfile sont requis, mais diagnosticsProfile est facultatif. Ces profils définissent des paramètres tels que :
   
- [size](../sizes.md)
- [nom](/azure/architecture/best-practices/resource-naming) et informations d’identification
- disque et [paramètres du système d’exploitation](cli-ps-findimage.md)
- [interface réseau](/previous-versions/azure/virtual-network/virtual-network-deploy-multinic-classic-ps) 
- diagnostics de démarrage

## <a name="disks-and-images"></a>Disques et images
   
Dans Azure, les fichiers de disque dur virtuel peuvent représenter [des disques ou des images](../managed-disks-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Quand le système d’exploitation d’un fichier vhd est spécialisé pour être une machine virtuelle spécifique, il est appelé disque. Quand le système d’exploitation d’un fichier vhd est généralisé pour permettre la création de plusieurs machines virtuelles, il est appelé image.   
    
### <a name="create-new-virtual-machines-and-new-disks-from-a-platform-image"></a>Créer des machines virtuelles et des disques à partir d’une image de plateforme

Lorsque vous créez une machine virtuelle, vous devez décider le système d’exploitation à utiliser. L’élément imageReference sert à définir le système d’exploitation d’une nouvelle machine virtuelle. L’exemple montre une définition pour un système d’exploitation Windows :

```json
"imageReference": { 
  "publisher": "MicrosoftWindowsServer", 
  "offer": "WindowsServer", 
  "sku": "2012-R2-Datacenter", 
  "version": "latest" 
},
```

Si vous souhaitez créer un système d’exploitation Linux, vous pouvez utiliser cette définition :

```json
"imageReference": {
  "publisher": "Canonical",
  "offer": "UbuntuServer",
  "sku": "14.04.2-LTS",
  "version": "latest"
},
```

Les paramètres de configuration pour le disque du système d’exploitation sont affectés avec l’élément osDisk. L’exemple définit un nouveau disque géré avec le mode cache défini sur **ReadWrite**, et spécifie que les disques sont en cours de création à partir d’une [image de plateforme](cli-ps-findimage.md) :

```json
"osDisk": { 
  "name": "[concat('myOSDisk', copyindex())]",
  "caching": "ReadWrite", 
  "createOption": "FromImage" 
},
```

### <a name="create-new-virtual-machines-from-existing-managed-disks"></a>Créer de nouvelles machines virtuelles à partir de disques managés existants

Si vous souhaitez créer des machines virtuelles à partir de disques existants, supprimez les éléments imageReference et osProfile puis définissez ces paramètres de disque :

```json
"osDisk": { 
  "osType": "Windows",
  "managedDisk": { 
    "id": "[resourceId('Microsoft.Compute/disks', [concat('myOSDisk', copyindex())])]" 
  }, 
  "caching": "ReadWrite",
  "createOption": "Attach" 
},
```

### <a name="create-new-virtual-machines-from-a-managed-image"></a>Créer de nouvelles machines virtuelles à partir d'une image gérée

Si vous souhaitez créer une machine virtuelle à partir d’une image gérée, modifiez l’élément imageReference puis définissez ces paramètres de disque :

```json
"storageProfile": { 
  "imageReference": {
    "id": "[resourceId('Microsoft.Compute/images', 'myImage')]"
  },
  "osDisk": { 
    "name": "[concat('myOSDisk', copyindex())]",
    "osType": "Windows",
    "caching": "ReadWrite", 
    "createOption": "FromImage" 
  }
},
```

### <a name="attach-data-disks"></a>Connecter des disques de données

Vous pouvez aussi ajouter des disques de données aux machines virtuelles. Le [nombre de disques](../sizes.md) dépend de la taille du disque de système d’exploitation que vous utilisez. Si la taille des machines virtuelles est définie sur Standard_DS1_v2, le nombre maximal de disques de données qui peuvent leur être ajoutés est de deux. Dans l’exemple, un disque de données géré est ajouté à chaque machine virtuelle :

```json
"dataDisks": [
  {
    "name": "[concat('myDataDisk', copyindex())]",
    "diskSizeGB": "100",
    "lun": 0, 
    "caching": "ReadWrite",
    "createOption": "Empty"
  }
],
```

## <a name="extensions"></a>Extensions

Bien que les [extensions](../extensions/features-windows.md) représentent une ressource distincte, elles sont étroitement liées aux machines virtuelles. Les extensions peuvent être ajoutées comme une ressource enfant de la machine virtuelle ou comme une ressource distincte. L’exemple montre l'ajout de l'[extension de diagnostics](../extensions/diagnostics-template.md) aux machines virtuelles :

```json
{ 
  "name": "Microsoft.Insights.VMDiagnosticsSettings", 
  "type": "extensions", 
  "location": "[resourceGroup().location]", 
  "apiVersion": "2016-03-30", 
  "dependsOn": [ 
    "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]" 
  ], 
  "properties": { 
    "publisher": "Microsoft.Azure.Diagnostics", 
    "type": "IaaSDiagnostics", 
    "typeHandlerVersion": "1.5", 
    "autoUpgradeMinorVersion": true, 
    "settings": { 
      "xmlCfg": "[base64(concat(variables('wadcfgxstart'), 
      variables('wadmetricsresourceid'), 
      concat('myVM', copyindex()),
      variables('wadcfgxend')))]", 
      "storageAccount": "[variables('storageName')]" 
    }, 
    "protectedSettings": { 
      "storageAccountName": "[variables('storageName')]", 
      "storageAccountKey": "[listkeys(variables('accountid'), 
        '2015-06-15').key1]", 
      "storageAccountEndPoint": "https://core.windows.net" 
    } 
  } 
},
```

Cette ressource d’extension utilise la variable storageName et les variables de diagnostic pour fournir des valeurs. Si vous souhaitez modifier les données collectées par cette extension, vous pouvez ajouter des compteurs de performances supplémentaires à la variable wadperfcounters. Vous pouvez également choisir placer les données de diagnostic dans un autre compte de stockage que celui où sont stockés les disques de machine virtuelle.

Il existe de nombreuses extensions que vous pouvez installer sur une machine virtuelle, mais l'extension la plus utile est probablement l'[extension de script personnalisé](../extensions/custom-script-windows.md). Dans l’exemple, un script PowerShell nommé start.ps1 s’exécute sur chaque machine virtuelle au premier démarrage :

```json
{
  "name": "MyCustomScriptExtension",
  "type": "extensions",
  "apiVersion": "2016-03-30",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]"
  ],
  "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.7",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "[concat('https://', variables('storageName'),
          '.blob.core.windows.net/customscripts/start.ps1')]" 
      ],
      "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File start.ps1"
    }
  }
}
```

Le script start.ps1 permettre d'effectuer de nombreuses tâches de configuration. Par exemple, les disques de données ajoutés aux machines virtuelles dans l’exemple ne sont pas initialisés. Vous pouvez utiliser un script personnalisé pour les initialiser. Si vous avez plusieurs tâches de démarrage à exécuter, vous pouvez utiliser le fichier start.ps1 pour appeler d’autres scripts PowerShell dans le stockage Azure. L’exemple utilise PowerShell, mais vous pouvez utiliser n’importe quelle méthode de script disponible sur le système d’exploitation que vous utilisez.

Vous pouvez consulter l’état des extensions installées à l'aide des paramètres Extensions sur le portail :

![Obtenir l’état de l’extension](./media/template-description/virtual-machines-show-extensions.png)

Vous pouvez aussi obtenir des informations sur l’extension à l’aide de la commande PowerShell **Get-AzRmVMExtension**, de la commande Azure CLI **vm extension get** ou de l’API REST **Obtenir des informations sur l’extension**.

## <a name="deployments"></a>Déploiements

Lorsque vous déployez un modèle, Azure effectue le suivi des ressources que vous déployez en tant que groupe et assigne automatiquement un nom à ce groupe déployé. Le nom du déploiement est identique à celui du modèle.

Si vous souhaitez en savoir plus sur l’état des ressources dans le déploiement, affichez le groupe de ressources dans le Portail Azure :

![Obtenir les informations de déploiement](./media/template-description/virtual-machines-deployment-info.png)
    
Vous pouvez utiliser le même modèle pour créer des ressources ou pour mettre à jour des ressources existantes. Lorsque vous utilisez des commandes pour déployer des modèles, vous avez la possibilité de choisir le [mode](../../azure-resource-manager/templates/deploy-powershell.md) à utiliser. Le mode peut être défini avec la valeur **Complète** ou **Incrémentielle**. La valeur par défaut consiste à effectuer des mises à jour incrémentielles. Soyez prudent lorsque vous utilisez le mode **Complète** car vous risquez de supprimer accidentellement des ressources. Quand vous choisissez le mode **Complète**, Resource Manager supprime toutes les ressources du groupe de ressources qui ne figurent pas dans le modèle.

## <a name="next-steps"></a>Étapes suivantes

- Créez votre propre modèle à l’aide de la rubrique [Création de modèles Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md).
- Déployez le modèle que vous avez créé à l'aide de la rubrique [Créer une machine virtuelle Windows avec un modèle Resource Manager](ps-template.md).
- Découvrez comment gérer les machines virtuelles que vous avez créées en lisant [Créer et gérer des machines virtuelles Windows avec le module Azure PowerShell](tutorial-manage-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
- Pour connaître la syntaxe JSON et les propriétés des types de ressource dans les modèles, consultez la [référence du modèle Azure Resource Manager](/azure/templates/).