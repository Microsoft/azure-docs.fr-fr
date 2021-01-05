---
title: Référencer une image personnalisée dans un modèle de groupe identique Azure
description: Découvrez comment ajouter une image personnalisée à un modèle de groupe de machines virtuelles identiques Azure existant
author: cynthn
ms.author: cynthn
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.date: 04/26/2018
ms.reviewer: akjosh
ms.custom: akjosh
ms.openlocfilehash: 5ed9ee79dde73e738417031b928a675ea913179c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83124905"
---
# <a name="add-a-custom-image-to-an-azure-scale-set-template"></a>Ajout d’une image personnalisée à un modèle de groupe identique Azure

Cet article montre comment modifier le [modèle de groupe identique de base](virtual-machine-scale-sets-mvss-start.md) afin de déployer avec une image personnalisée.

## <a name="change-the-template-definition"></a>Modifier la définition du modèle
Dans un [article précédent](virtual-machine-scale-sets-mvss-start.md), nous avions créé un modèle de groupe identique de base. Nous allons maintenant utiliser ce modèle antérieur et le modifier pour créer un modèle qui déploie un groupe identique à partir d’une image personnalisée.  

### <a name="creating-a-managed-disk-image"></a>Création d’une image de disque géré

Si vous disposez déjà d’une image disque gérée personnalisée (une ressource de type `Microsoft.Compute/images`), vous pouvez ignorer cette section.

Tout d’abord, ajoutez un paramètre `sourceImageVhdUri`, qui est l’URI vers l’objet blob généralisé dans le stockage Azure qui contient l’image personnalisée à partir de laquelle le déploiement est effectué.


```diff
     },
     "adminPassword": {
       "type": "securestring"
+    },
+    "sourceImageVhdUri": {
+      "type": "string",
+      "metadata": {
+        "description": "The source of the generalized blob containing the custom image"
+      }
     }
   },
   "variables": {},
```

Ensuite, ajoutez une ressource de type `Microsoft.Compute/images`, qui est l’image de disque managé basée sur le blob généralisé situé à l’URI `sourceImageVhdUri`. Cette image doit se trouver dans la même région que le groupe identique qui l’utilise. Dans les propriétés de l’image, spécifiez le type de système d’exploitation, l’emplacement de l’objet blob (à partir du paramètre `sourceImageVhdUri`) et le type de compte de stockage :

```diff
   "resources": [
     {
+      "type": "Microsoft.Compute/images",
+      "apiVersion": "2019-03-01",
+      "name": "myCustomImage",
+      "location": "[resourceGroup().location]",
+      "properties": {
+        "storageProfile": {
+          "osDisk": {
+            "osType": "Linux",
+            "osState": "Generalized",
+            "blobUri": "[parameters('sourceImageVhdUri')]",
+            "storageAccountType": "Standard_LRS"
+          }
+        }
+      }
+    },
+    {
       "type": "Microsoft.Network/virtualNetworks",
       "name": "myVnet",
       "location": "[resourceGroup().location]",

```

Dans la ressource groupe identique, ajoutez une clause `dependsOn` faisant référence à l’image personnalisée pour nous assurer que l’image est créée avant que le groupe identique tente de se déployer à partir de cette image :

```diff
       "location": "[resourceGroup().location]",
       "apiVersion": "2019-03-01-preview",
       "dependsOn": [
-        "Microsoft.Network/virtualNetworks/myVnet"
+        "Microsoft.Network/virtualNetworks/myVnet",
+        "Microsoft.Compute/images/myCustomImage"
       ],
       "sku": {
         "name": "Standard_A1",

```

### <a name="changing-scale-set-properties-to-use-the-managed-disk-image"></a>Modification des propriétés du groupe identique pour utiliser l’image de disque géré

Dans la `imageReference` du groupe identique `storageProfile`, au lieu de spécifier l’éditeur, l’offre, la référence (SKU) et la version d’une image de plateforme, spécifiez la valeur `id` de la ressource `Microsoft.Compute/images` :

```json
         "virtualMachineProfile": {
           "storageProfile": {
             "imageReference": {
              "id": "[resourceId('Microsoft.Compute/images', 'myCustomImage')]"
             }
           },
           "osProfile": {
```

Dans cet exemple, utilisez la fonction `resourceId` pour obtenir l’ID de ressource de l’image créée dans le même modèle. Si vous avez créé l’image de disque managé au préalable, vous devez fournir l’ID de cette image à la place. Cet ID doit être au format : `/subscriptions/<subscription-id>resourceGroups/<resource-group-name>/providers/Microsoft.Compute/images/<image-name>`.


## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]
