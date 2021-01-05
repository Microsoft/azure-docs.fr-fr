---
title: Fichier Include
description: Fichier Include
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 02/20/2018
ms.author: tomfitz
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: 53d4aa70b55577cdf2f6a1b898b496eb368157f5
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92755347"
---
Pour ajouter deux balises à un groupe de ressources, utilisez la commande [az group update](/cli/azure/group) :

```azurecli-interactive
az group update -n myResourceGroup --set tags.Environment=Test tags.Dept=IT
```

Supposons que vous souhaitez ajouter une troisième balise. Exécutez de nouveau la commande avec la nouvelle balise. Elle est ajoutée aux balises existantes.

```azurecli-interactive
az group update -n myResourceGroup --set tags.Project=Documentation
```

Les ressources n’héritent pas des balises du groupe de ressources. Votre groupe de ressources possède actuellement trois balises, mais les ressources n’ont pas de balises. Pour appliquer toutes les balises d’un groupe de ressources à ses ressources en conservant les balises existantes, utilisez le script suivant :

```azurecli-interactive
# Get the tags for the resource group
jsontag=$(az group show -n myResourceGroup --query tags)

# Reformat from JSON to space-delimited and equals sign
t=$(echo $jsontag | tr -d '"{},' | sed 's/: /=/g')

# Get the resource IDs for all resources in the resource group
r=$(az resource list -g myResourceGroup --query [].id --output tsv)

# Loop through each resource ID
for resid in $r
do
  # Get the tags for this resource
  jsonrtag=$(az resource show --id $resid --query tags)
  
  # Reformat from JSON to space-delimited and equals sign
  rt=$(echo $jsonrtag | tr -d '"{},' | sed 's/: /=/g')
  
  # Reapply the updated tags to this resource
  az resource tag --tags $t$rt --id $resid
done
```

Autrement, vous pouvez appliquer des balises du groupe de ressources aux ressources sans conserver les balises existantes :

```azurecli-interactive
# Get the tags for the resource group
jsontag=$(az group show -n myResourceGroup --query tags)

# Reformat from JSON to space-delimited and equals sign
t=$(echo $jsontag | tr -d '"{},' | sed 's/: /=/g')

# Get the resource IDs for all resources in the resource group
r=$(az resource list -g myResourceGroup --query [].id --output tsv)

# Loop through each resource ID
for resid in $r
do
  # Apply tags from resource group to this resource
  az resource tag --tags $t --id $resid
done
```

Pour combiner plusieurs valeurs dans une balise unique, utilisez une chaîne JSON.

```azurecli-interactive
az group update -n myResourceGroup --set tags.CostCenter='{"Dept":"IT","Environment":"Test"}'
```

Pour supprimer toutes les balises sur un groupe de ressources, utilisez :

```azurecli-interactive
az group update -n myResourceGroup --remove tags
```
