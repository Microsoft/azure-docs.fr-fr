---
title: Explorer vos ressources Azure
description: Apprenez à utiliser le langage de requête Resource Graph pour explorer vos ressources et découvrir comment elles sont connectées.
ms.date: 10/14/2020
ms.topic: conceptual
ms.openlocfilehash: 2b0ef3935d865618a9d4dda2825f7d4383baf772
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92056240"
---
# <a name="explore-your-azure-resources-with-resource-graph"></a>Explorer vos ressources Azure avec Resource Graph

Azure Resource Graph vous permet d’explorer et de détecter vos ressources Azure rapidement et à l’échelle. Conçu pour fournir des réponses rapides, il s’avère très efficace pour en apprendre davantage sur votre environnement, mais aussi sur les propriétés présentes sur vos ressources Azure.

## <a name="explore-virtual-machines"></a>Explorer des machines virtuelles

Dans Azure, les machines virtuelles sont des ressources courantes. En tant que type de ressource, les machines virtuelles comptent de nombreuses propriétés qui peuvent faire l’objet de requêtes. Chaque propriété propose une option pour filtrer ou trouver exactement la ressource que vous recherchez.

### <a name="virtual-machine-discovery"></a>Détection des machines virtuelles

Commençons avec une requête simple qui vise à obtenir une seule machine virtuelle dans notre environnement et examinons les propriétés retournées.

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| limit 1
```

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | limit 1"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | limit 1" | ConvertTo-Json -Depth 100
```

> [!NOTE]
> La cmdlet Azure PowerShell `Search-AzGraph` ​​renvoie un objet **PSCustomObject** par défaut. Pour que la sortie soit identique à celle renvoyée par Azure CLI, la cmdlet `ConvertTo-Json` est utilisée. La valeur par défaut du paramètre **Profondeur** est _2_. Définissez-le sur _100_ pour convertir tous les niveaux renvoyés.

Les résultats JSON sont structurés de façon similaire à l’exemple suivant :

```json
[
  {
    "id": "/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/ContosoVM1",
    "kind": "",
    "location": "westus2",
    "managedBy": "",
    "name": "ContosoVM1",
    "plan": {},
    "properties": {
      "hardwareProfile": {
        "vmSize": "Standard_B2s"
      },
      "networkProfile": {
        "networkInterfaces": [
          {
            "id": "/subscriptions/<subscriptionId>/MyResourceGroup/providers/Microsoft.Network/networkInterfaces/contosovm1535",
            "resourceGroup": "MyResourceGroup"
          }
        ]
      },
      "osProfile": {
        "adminUsername": "localAdmin",
        "computerName": "ContosoVM1",
        "secrets": [],
        "windowsConfiguration": {
          "enableAutomaticUpdates": true,
          "provisionVMAgent": true
        }
      },
      "provisioningState": "Succeeded",
      "storageProfile": {
        "dataDisks": [],
        "imageReference": {
          "offer": "WindowsServer",
          "publisher": "MicrosoftWindowsServer",
          "sku": "2016-Datacenter",
          "version": "latest"
        },
        "osDisk": {
          "caching": "ReadWrite",
          "createOption": "FromImage",
          "diskSizeGB": 127,
          "managedDisk": {
            "id": "/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166",
            "resourceGroup": "MyResourceGroup",
            "storageAccountType": "Premium_LRS"
          },
          "name": "ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166",
          "osType": "Windows"
        }
      },
      "vmId": "bbb9b451-6dc7-4117-bec5-c971eb1118c6"
    },
    "resourceGroup": "MyResourceGroup",
    "sku": {},
    "subscriptionId": "<subscriptionId>",
    "tags": {},
    "type": "microsoft.compute/virtualmachines"
  }
]
```

Les propriétés donnent des informations supplémentaires sur la ressource de la machine virtuelle proprement dite : système d’exploitation, disques, balises, groupe de ressources et abonnement dont elle est membre, etc.

### <a name="virtual-machines-by-location"></a>Machines virtuelles par emplacement

Sur la base de ce que nous avons appris sur la ressource de machine virtuelle, nous allons utiliser la propriété **location** pour compter toutes les machines virtuelles par emplacement. Pour mettre à jour la requête, nous allons supprimer la limite et résumer le nombre de valeurs d’emplacement.

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| summarize count() by location
```

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | summarize count() by location"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | summarize count() by location"
```

Les résultats JSON sont structurés de façon similaire à l’exemple suivant :

```json
[
  {
    "count_": 386,
    "location": "eastus"
  },
  {
    "count_": 215,
    "location": "southcentralus"
  },
  {
    "count_": 59,
    "location": "westus"
  }
]
```

Nous pouvons maintenant voir combien de machines virtuelles nous avons dans chaque région Azure.

### <a name="virtual-machines-by-sku"></a>Machines virtuelles par référence (SKU)

Après avoir rétabli les propriétés de machine virtuelle d’origine, essayons de trouver toutes les machines virtuelles dont la taille SKU est **Standard_B2s**. À la lecture du JSON retourné, nous voyons que cette information est stockée dans **properties.hardwareprofile.vmsize**. Nous allons mettre à jour la requête de façon à trouver toutes les machines virtuelles qui ont cette taille et pour que seuls le nom et la région des machines virtuelles soient retournés.

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualMachines' and properties.hardwareProfile.vmSize == 'Standard_B2s'
| project name, resourceGroup
```

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' and properties.hardwareProfile.vmSize == 'Standard_B2s' | project name, resourceGroup"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' and properties.hardwareProfile.vmSize == 'Standard_B2s' | project name, resourceGroup"
```

### <a name="virtual-machines-connected-to-premium-managed-disks"></a>Machines virtuelles connectées à des disques managés Premium

Pour obtenir les détails des disques managés Premium attachés à ces machines virtuelles **Standard_B2s**, nous étendons la requête de façon à retourner l’ID de la ressource de ces disques managés.

```kusto
Resources
| where type =~ 'Microsoft.Compute/virtualmachines' and properties.hardwareProfile.vmSize == 'Standard_B2s'
| extend disk = properties.storageProfile.osDisk.managedDisk
| where disk.storageAccountType == 'Premium_LRS'
| project disk.id
```

> [!NOTE]
> Une autre façon d’obtenir la référence (SKU) consiste à utiliser la propriété **aliases****Microsoft.Compute/virtualMachines/sku.name**. Voir les exemples [Afficher des alias](../samples/starter.md#show-aliases) et [Afficher des valeurs distinctes pour un alias](../samples/starter.md#distinct-alias-values).

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualmachines' and properties.hardwareProfile.vmSize == 'Standard_B2s' | extend disk = properties.storageProfile.osDisk.managedDisk | where disk.storageAccountType == 'Premium_LRS' | project disk.id"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualmachines' and properties.hardwareProfile.vmSize == 'Standard_B2s' | extend disk = properties.storageProfile.osDisk.managedDisk | where disk.storageAccountType == 'Premium_LRS' | project disk.id"
```

Le résultat obtenu est une liste d’ID de disques.

### <a name="managed-disk-discovery"></a>Détection des disques managés

Avec le premier enregistrement de la requête précédente, nous allons explorer les propriétés qui existent sur le disque managé qui a été attaché à la première machine virtuelle. La requête mise à jour utilise l’ID de disque et modifie le type.

Exemple de sortie de la requête précédente, par exemple :

```json
[
  {
    "disk_id": "/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166"
  }
]
```

```kusto
Resources
| where type =~ 'Microsoft.Compute/disks' and id == '/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166'
```

Avant d’exécuter la requête, comment aurions-nous pu savoir que le **type** allait maintenant être **Microsoft.Compute/disks** ? Si vous examinez l’ID complet, vous constaterez que **/providers/Microsoft.Compute/disks/** fait partie intégrante de la chaîne.
Ce fragment de chaîne vous donne une indication du type à rechercher. Une autre méthode consiste à retirer la limite par type et à effectuer à la place une recherche sur le champ ID uniquement. Comme l’ID est unique, un seul enregistrement est retourné et la propriété **type** de ce dernier fournit ce détail.

> [!NOTE]
> Pour que cet exemple fonctionne, vous devez remplacer la valeur du champ ID par un résultat tiré de votre propre environnement.

```azurecli-interactive
az graph query -q "Resources | where type =~ 'Microsoft.Compute/disks' and id == '/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166'"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/disks' and id == '/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166'"
```

Les résultats JSON sont structurés de façon similaire à l’exemple suivant :

```json
[
  {
    "id": "/subscriptions/<subscriptionId>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/disks/ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166",
    "kind": "",
    "location": "westus2",
    "managedBy": "",
    "name": "ContosoVM1_OsDisk_1_9676b7e1b3c44e2cb672338ebe6f5166",
    "plan": {},
    "properties": {
      "creationData": {
        "createOption": "Empty"
      },
      "diskSizeGB": 127,
      "diskState": "ActiveSAS",
      "provisioningState": "Succeeded",
      "timeCreated": "2018-09-14T12:17:32.2570000Z"
    },
    "resourceGroup": "MyResourceGroup",
    "sku": {
      "name": "Premium_LRS",
      "tier": "Premium"
    },
    "subscriptionId": "<subscriptionId>",
    "tags": {
      "environment": "prod"
    },
    "type": "microsoft.compute/disks"
  }
]
```

## <a name="explore-virtual-machines-to-find-public-ip-addresses"></a>Explorer des machines virtuelles pour trouver des adresses IP publiques

Cet ensemble de requêtes recherche et stocke d’abord toutes les ressources de carte réseau (NIC) connectées à des machines virtuelles. Les requêtes utilisent ensuite la liste des cartes réseau pour trouver chaque ressource d’adresse IP qui correspond à une adresse IP publique et stocke ces valeurs. Puis, les requêtes fournissent la liste des adresses IP publiques.

```azurecli-interactive
# Use Resource Graph to get all NICs and store in the 'nics.txt' file
az graph query -q "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | project nic = tostring(properties['networkProfile']['networkInterfaces'][0]['id']) | where isnotempty(nic) | distinct nic | limit 20" --output table | tail -n +3 > nics.txt

# Review the output of the query stored in 'nics.txt'
cat nics.txt
```

```azurepowershell-interactive
# Use Resource Graph to get all NICs and store in the $nics variable
$nics = Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Compute/virtualMachines' | project nic = tostring(properties['networkProfile']['networkInterfaces'][0]['id']) | where isnotempty(nic) | distinct nic | limit 20"

# Review the output of the query stored in the variable
$nics.nic
```

Utilisez le fichier (Azure CLI) ou la variable (Azure PowerShell) dans la requête suivante pour obtenir les détails relatifs aux ressources d’interface réseau associées si une adresse IP publique est attachée à la carte réseau.

```azurecli-interactive
# Use Resource Graph with the 'nics.txt' file to get all related public IP addresses and store in 'publicIp.txt' file
az graph query -q="Resources | where type =~ 'Microsoft.Network/networkInterfaces' | where id in ('$(awk -vORS="','" '{print $0}' nics.txt | sed 's/,$//')') | project publicIp = tostring(properties['ipConfigurations'][0]['properties']['publicIPAddress']['id']) | where isnotempty(publicIp) | distinct publicIp" --output table | tail -n +3 > ips.txt

# Review the output of the query stored in 'ips.txt'
cat ips.txt
```

```azurepowershell-interactive
# Use Resource Graph  with the $nics variable to get all related public IP addresses and store in $ips variable
$ips = Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Network/networkInterfaces' | where id in ('$($nics.nic -join "','")') | project publicIp = tostring(properties['ipConfigurations'][0]['properties']['publicIPAddress']['id']) | where isnotempty(publicIp) | distinct publicIp"

# Review the output of the query stored in the variable
$ips.publicIp
```

Enfin, nous allons utiliser la liste de ressources d’adresse IP publique stockée dans le fichier (Azure CLI) ou la variable (Azure PowerShell) pour obtenir et afficher leur adresse IP publique réelle à partir de l'objet associé.

```azurecli-interactive
# Use Resource Graph with the 'ips.txt' file to get the IP address of the public IP address resources
az graph query -q="Resources | where type =~ 'Microsoft.Network/publicIPAddresses' | where id in ('$(awk -vORS="','" '{print $0}' ips.txt | sed 's/,$//')') | project ip = tostring(properties['ipAddress']) | where isnotempty(ip) | distinct ip" --output table
```

```azurepowershell-interactive
# Use Resource Graph with the $ips variable to get the IP address of the public IP address resources
Search-AzGraph -Query "Resources | where type =~ 'Microsoft.Network/publicIPAddresses' | where id in ('$($ips.publicIp -join "','")') | project ip = tostring(properties['ipAddress']) | where isnotempty(ip) | distinct ip"
```

Pour savoir comment effectuer ces étapes au sein d’une même requête avec l’opérateur `join`, consultez l’exemple [Lister les machines virtuelles avec leur interface réseau et leur adresse IP publique](../samples/advanced.md#join-vmpip).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez plus en détails le [langage de requête](query-language.md).
- Examinez le langage utilisé dans les [requêtes de démarrage](../samples/starter.md).
- Examinez les utilisations avancées dans les [Requêtes avancées](../samples/advanced.md).
