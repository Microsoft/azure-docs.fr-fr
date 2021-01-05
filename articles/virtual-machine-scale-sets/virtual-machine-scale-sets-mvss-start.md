---
title: En savoir plus sur les modèles de groupes de machines virtuelles identiques
description: Découvrez comment créer un Modèle de groupe identique de base pour les groupes de machines virtuelles identiques Azure en quelques étapes simples.
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: template
ms.date: 04/26/2019
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: 55c826b4baf38732684aaa0465aeaab6a45564db
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87831496"
---
# <a name="learn-about-virtual-machine-scale-set-templates"></a>En savoir plus sur les modèles de groupes de machines virtuelles identiques
Les [modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md#template-deployment-process) constituent un excellent moyen de déployer des groupes de ressources liées. Cette série de didacticiels montre comment créer un modèle de groupe identique de base et comment modifier ce modèle pour l’adapter à différents scénarios. Tous les exemples proviennent de ce [référentiel GitHub](https://github.com/gatneil/mvss).

Ce modèle est conçu pour être simple. Pour des exemples plus complets de modèles de groupes identiques, consultez le [référentiel Github de modèles de démarrage rapide Azure](https://github.com/Azure/azure-quickstart-templates) et recherchez les dossiers qui contiennent la chaîne `vmss`.

Si vous êtes déjà familiarisé avec la création de modèles, vous pouvez passer à la section « Étapes suivantes » pour découvrir comment modifier ce modèle.

## <a name="define-schema-and-contentversion"></a>Définir $schema et contentVersion
Tout d’abord, définissez `$schema` et `contentVersion` dans le modèle. L’élément `$schema` définit la version de langue du modèle et est utilisé pour le surlignage de syntaxe et d’autres fonctionnalités de validation similaires dans Visual Studio. L’élément `contentVersion` n’est pas utilisé par Azure. Au lieu de cela, il vous aide à effectuer le suivi de la version du modèle.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
```

## <a name="define-parameters"></a>Définir les paramètres
Ensuite, définissez deux paramètres, `adminUsername` et `adminPassword`. Les paramètres sont des valeurs que vous spécifiez au moment du déploiement. Le paramètre `adminUsername` est simplement de type `string`, mais étant donné que `adminPassword` est un secret, donnez-lui un type `securestring`. Ensuite, ces paramètres sont passés dans la configuration du groupe identique.

```json
  "parameters": {
    "adminUsername": {
      "type": "string"
    },
    "adminPassword": {
      "type": "securestring"
    }
  },
```
## <a name="define-variables"></a>Définir des variables
Les modèles Resource Manager vous permettent également de définir les variables à utiliser par la suite dans le modèle. Comme l’exemple n’utilise aucune variable, l’objet JSON est vide.

```json
  "variables": {},
```

## <a name="define-resources"></a>Définir les ressources
Passons ensuite à la section des ressources du modèle. Vous définissez ici ce que vous souhaitez déployer. Contrairement à `parameters` et `variables` (qui sont des objets JSON), `resources` est une liste JSON d’objets JSON.

```json
   "resources": [
```

Toutes les ressources nécessitent les propriétés `type`, `name`, `apiVersion` et `location`. La première ressource de cet exemple est de type [Microsoft.Network/virtualNetwork](/azure/templates/microsoft.network/virtualnetworks), avec le nom `myVnet` et apiVersion `2018-11-01`. (Pour trouver la dernière version de l’API pour un type de ressource, reportez-vous à la [documentation de référence des modèles Azure Resource Manager](/azure/templates/).)

```json
     {
       "type": "Microsoft.Network/virtualNetworks",
       "name": "myVnet",
       "apiVersion": "2018-11-01",
```

## <a name="specify-location"></a>Spécifier l’emplacement
Pour spécifier l’emplacement du réseau virtuel, utilisez une [fonction de modèle Resource Manager](../azure-resource-manager/templates/template-functions.md). Cette fonction doit être placée entre guillemets et crochets, comme suit : `"[<template-function>]"`. Dans ce cas, utilisez la fonction `resourceGroup`. Elle ne prend aucun argument et renvoie un objet JSON avec des métadonnées concernant le groupe de ressources cible de ce déploiement. Le groupe de ressources est défini par l’utilisateur au moment du déploiement. Cette valeur est ensuite indexée dans cet objet JSON avec `.location` pour obtenir l’emplacement de l’objet JSON.

```json
       "location": "[resourceGroup().location]",
```

## <a name="specify-virtual-network-properties"></a>Spécifier les propriétés du réseau virtuel
Chaque ressource de Resource Manager a sa propre section `properties` pour les configurations spécifiques à la ressource. Dans ce cas, spécifiez que le réseau virtuel doit avoir un sous-réseau qui utilise la plage d’adresses IP privées `10.0.0.0/16`. Un jeu de mise à l’échelle est toujours contenu dans un sous-réseau. Il ne peut pas s’étendre sur plusieurs sous-réseaux.

```json
       "properties": {
         "addressSpace": {
           "addressPrefixes": [
             "10.0.0.0/16"
           ]
         },
         "subnets": [
           {
             "name": "mySubnet",
             "properties": {
               "addressPrefix": "10.0.0.0/16"
             }
           }
         ]
       }
     },
```

## <a name="add-dependson-list"></a>Ajouter la liste dependsOn
Outre les propriétés `type`, `name`, `apiVersion` et `location` requises, chaque ressource peut avoir une liste `dependsOn` de chaînes facultative. Cette liste spécifie les autres ressources de ce déploiement qui doivent se terminer avant le déploiement de cette ressource.

Dans ce cas, la liste ne contient qu’un seul élément, le réseau virtuel issu de l’exemple précédent. Vous spécifiez cette dépendance, car le groupe identique a besoin que le réseau existe avant la création de machines virtuelles. De cette façon, le groupe identique peut donner des adresses IP privées à ces machines virtuelles à partir de la plage d’adresses IP spécifiée précédemment dans les propriétés du réseau. Le format de chaque chaîne dans la liste dependsOn est `<type>/<name>`. Utilisez les mêmes valeurs `type` et `name` utilisées précédemment dans la définition des ressources du réseau virtuel.

```json
     {
       "type": "Microsoft.Compute/virtualMachineScaleSets",
       "name": "myScaleSet",
       "apiVersion": "2019-03-01",
       "location": "[resourceGroup().location]",
       "dependsOn": [
         "Microsoft.Network/virtualNetworks/myVnet"
       ],
```
## <a name="specify-scale-set-properties"></a>Spécifier les propriétés du groupe identique
Les groupes identiques disposent de nombreuses propriétés pour personnaliser les machines virtuelles dans le groupe identique. Pour obtenir une liste complète de ces propriétés, consultez la [documentation de référence des modèles](/azure/templates/microsoft.compute/virtualmachinescalesets). Pour ce didacticiel, seules quelques propriétés couramment utilisées sont définies.
### <a name="supply-vm-size-and-capacity"></a>Fournir la capacité et la taille de machine virtuelle
Le groupe identique doit connaître la taille de machine virtuelle à créer (le « nom de la référence (SKU) ») et le nombre de ces machines virtuelles à créer (la « capacité de la référence (SKU) »). Pour voir les tailles de machines virtuelles disponibles, consultez la [documentation sur les tailles de machines virtuelles](../virtual-machines/sizes.md).

```json
       "sku": {
         "name": "Standard_A1",
         "capacity": 2
       },
```

### <a name="choose-type-of-updates"></a>Choisir le type de mises à jour
Le jeu de mise à l’échelle doit également savoir comment gérer les mises à jour sur l’ensemble du jeu de mise à l’échelle. Il existe actuellement trois options : `Manual`, `Rolling` et `Automatic`. Pour plus d’informations sur les différences entre les deux, consultez la documentation sur la [Mise à niveau d’un jeu de mise à l’échelle](./virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model).

```json
       "properties": {
         "upgradePolicy": {
           "mode": "Manual"
         },
```

### <a name="choose-vm-operating-system"></a>Choisir le système d’exploitation de la machine virtuelle
Le groupe identique doit savoir quel système d’exploitation mettre sur les machines virtuelles. Créez ici les machines virtuelles avec une image Ubuntu 16.04-LTS entièrement corrigée.

```json
         "virtualMachineProfile": {
           "storageProfile": {
             "imageReference": {
               "publisher": "Canonical",
               "offer": "UbuntuServer",
               "sku": "16.04-LTS",
               "version": "latest"
             }
           },
```

### <a name="specify-computernameprefix"></a>Spécifier computerNamePrefix
Le groupe identique déploie plusieurs machines virtuelles. Au lieu de spécifier le nom de chaque machine virtuelle, spécifiez `computerNamePrefix`. Le groupe identique ajoute un index au préfixe de chaque machine virtuelle, donc les noms des machines virtuelles se présentent sous la forme `<computerNamePrefix>_<auto-generated-index>`.

Dans l’extrait de code suivant, utilisez les paramètres précédents pour définir le nom d’utilisateur de l’administrateur et son mot de passe pour toutes les machines virtuelles dans le groupe identique. Ce processus utilise la fonction de modèle `parameters`. Cette fonction prend une chaîne qui spécifie le paramètre de référence et renvoie la valeur pour ce paramètre.

```json
           "osProfile": {
             "computerNamePrefix": "vm",
             "adminUsername": "[parameters('adminUsername')]",
             "adminPassword": "[parameters('adminPassword')]"
           },
```

### <a name="specify-vm-network-configuration"></a>Spécifier la configuration du réseau de machines virtuelles
Enfin, spécifiez la configuration du réseau pour les machines virtuelles dans le groupe identique. Dans ce cas, il vous suffit de spécifier l’ID du sous-réseau créé précédemment. Cela indique au groupe identique de placer les interfaces réseau dans ce sous-réseau.

Vous pouvez obtenir l’ID du réseau virtuel qui contient le sous-réseau à l’aide de la fonction de modèle `resourceId`. Cette fonction prend le type et le nom d’une ressource et renvoie l’identificateur complet de la ressource. Cet ID se présente sous la forme : `/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/<resourceProviderNamespace>/<resourceType>/<resourceName>`

Toutefois, l’identificateur du réseau virtuel n’est pas suffisant. Indiquez le sous-réseau spécifique où doivent se situer les machines virtuelles du groupe identique. Pour ce faire, vous devez concaténer `/subnets/mySubnet` à l’ID du réseau virtuel. Le résultat est l’ID complet du sous-réseau. Effectuez cette concaténation avec la fonction `concat`, qui prend une série de chaînes et renvoie leur concaténation.

```json
           "networkProfile": {
             "networkInterfaceConfigurations": [
               {
                 "name": "myNic",
                 "properties": {
                   "primary": "true",
                   "ipConfigurations": [
                     {
                       "name": "myIpConfig",
                       "properties": {
                         "subnet": {
                           "id": "[concat(resourceId('Microsoft.Network/virtualNetworks', 'myVnet'), '/subnets/mySubnet')]"
                         }
                       }
                     }
                   ]
                 }
               }
             ]
           }
         }
       }
     }
   ]
}

```

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]