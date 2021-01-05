---
title: Bonnes pratiques relatives à l’implémentation de l’infrastructure Azure Service Fabric sous forme de code
description: Bonnes pratiques et considérations relatives à la conception pour la gestion d’Azure Service Fabric en tant qu’infrastructure sous forme de code.
author: peterpogorski
ms.topic: conceptual
ms.date: 01/23/2019
ms.author: pepogors
ms.openlocfilehash: a0f0324d1f7308eb1392c4f7a98a6a5d226026be
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97705504"
---
# <a name="infrastructure-as-code"></a>Infrastructure as code

Dans un scénario de production, créez des clusters Azure Service Fabric à l’aide de modèles Resource Manager. Les modèles Resource Manager permettent un meilleur contrôle des propriétés de ressource et garantissent la cohérence de votre modèle de ressource.

Des exemples de modèles Resource Manager pour Windows et Linux sont disponibles dans les [exemples Azure sur GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates). Ces modèles peuvent être utilisés comme point de départ pour votre modèle de cluster. Téléchargez `azuredeploy.json` et `azuredeploy.parameters.json`, puis modifiez-les selon vos besoins.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour déployer les modèles `azuredeploy.json` et `azuredeploy.parameters.json` que vous avez téléchargés précédemment, utilisez les commandes Azure CLI suivantes :

```azurecli
ResourceGroupName="sfclustergroup"
Location="westus"

az group create --name $ResourceGroupName --location $Location 
az deployment group create --name $ResourceGroupName  --template-file azuredeploy.json --parameters @azuredeploy.parameters.json
```

Création d’une ressource à l’aide de PowerShell

```powershell
$ResourceGroupName="sfclustergroup"
$Location="westus"
$Template="azuredeploy.json"
$Parameters="azuredeploy.parameters.json"

New-AzResourceGroup -Name $ResourceGroupName -Location $Location
New-AzResourceGroupDeployment -Name $ResourceGroupName -TemplateFile $Template -TemplateParameterFile $Parameters
```

## <a name="azure-service-fabric-resources"></a>Ressources Azure Service Fabric

Vous pouvez déployer des applications et services sur votre cluster Service Fabric par le biais d’Azure Resource Manager. Pour plus d’informations, consultez [Gérer des applications et services en tant que ressources Azure Resource Manager](./service-fabric-application-arm-resource.md). Voici les bonnes pratiques relatives aux applications Service Fabric, qui concernent plus spécifiquement les ressources à inclure dans vos ressources de modèle Resource Manager.

```json
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applicationTypes",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applicationTypes/versions",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'), '/', parameters('applicationTypeVersion'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applications",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applications/services",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName'))]",
    "location": "[variables('clusterLocation')]"
}
```

Pour déployer votre application à l’aide d’Azure Resource Manager, vous devez d’abord [créer un package d’application sfpkg](./service-fabric-package-apps.md#create-an-sfpkg) Service Fabric. Le script Python suivant montre comment créer un package sfpkg :

```python
# Create SFPKG that needs to be uploaded to Azure Storage Blob Container
microservices_sfpkg = zipfile.ZipFile(
    self.microservices_app_package_name, 'w', zipfile.ZIP_DEFLATED)
package_length = len(self.microservices_app_package_path)

for root, dirs, files in os.walk(self.microservices_app_package_path):
    root_folder = root[package_length:]
    for file in files:
        microservices_sfpkg.write(os.path.join(
            root, file), os.path.join(root_folder, file))

microservices_sfpkg.close()
```

## <a name="azure-virtual-machine-operating-system-automatic-upgrade-configuration"></a>Configuration de mise à niveau automatique du système d’exploitation Machine virtuelle Azure 
La mise à niveau de vos machines virtuelles est une opération initiée par l’utilisateur. Il est recommandé d’utiliser la [mise à niveau automatique du système d’exploitation dans des groupes de machines virtuelles identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) pour la gestion des correctifs hôtes des clusters Azure Service Fabric. L’application d’orchestration des correctifs est une solution alternative utilisée en cas d’hébergement en dehors d’Azure, bien qu’elle puisse être utilisée dans Azure, sachant que la surcharge d’hébergement de l’application d’orchestration des correctifs dans Azure est une raison courante de préférer la mise à niveau automatique du système d’exploitation dans des groupes de machines virtuelles identiques. Les propriétés du modèle Groupe de machines virtuelles identiques de calcul Resource Manager pour activer la mise à niveau automatique du système d’exploitation sont les suivantes :

```json
"upgradePolicy": {
   "mode": "Automatic",
   "automaticOSUpgradePolicy": {
        "enableAutomaticOSUpgrade": true,
        "disableAutomaticRollback": false
    }
},
```
Lors de l’utilisation de mises à niveau automatiques du système d’exploitation avec Service Fabric, la nouvelle image du système d’exploitation est déployée, un domaine de mise à jour après l’autre, pour maintenir la haute disponibilité des services en cours d’exécution dans Service Fabric. Pour utiliser les mises à niveau automatiques du système d’exploitation dans Service Fabric, votre cluster doit être configuré pour utiliser le niveau de durabilité Silver ou une version supérieure.

Vérifiez que la clé de Registre suivante est définie sur false pour empêcher le lancement de mises à jour non coordonnées sur vos ordinateurs hôtes Windows : HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU.

Les propriétés du modèle Groupe de machines virtuelles identiques de calcul Resource Manager pour définir la clé de Registre WindowsUpdate sur false sont les suivantes :
```json
"osProfile": {
        "computerNamePrefix": "{vmss-name}",
        "adminUsername": "{your-username}",
        "secrets": [],
        "windowsConfiguration": {
          "provisionVMAgent": true,
          "enableAutomaticUpdates": false
        }
      },
```

## <a name="azure-service-fabric-cluster-upgrade-configuration"></a>Configuration de la mise à niveau d’un cluster Azure Service Fabric
Voici la propriété de modèle Resource Manager de cluster Service Fabric pour activer la mise à niveau automatique :
```json
"upgradeMode": "Automatic",
```
Pour mettre à niveau manuellement votre cluster, téléchargez la distribution cab/deb sur une machine virtuelle de cluster, puis appelez la commande PowerShell suivante :
```powershell
Copy-ServiceFabricClusterPackage -Code -CodePackagePath <"local_VM_path_to_msi"> -CodePackagePathInImageStore ServiceFabric.msi -ImageStoreConnectionString "fabric:ImageStore"
Register-ServiceFabricClusterPackage -Code -CodePackagePath "ServiceFabric.msi"
Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion <"msi_code_version">
```

## <a name="next-steps"></a>Étapes suivantes

* Créez un cluster sur des machines virtuelles ou des ordinateurs exécutant Windows Server : [Création de clusters Service Fabric pour Windows Server](service-fabric-tutorial-create-vnet-and-windows-cluster.md)
* Créez un cluster sur des machines virtuelles ou des ordinateurs exécutant Linux : [Créer un cluster Linux](service-fabric-tutorial-create-vnet-and-linux-cluster.md)
* En savoir plus sur les [options de prise en charge de Service Fabric](service-fabric-support.md)
