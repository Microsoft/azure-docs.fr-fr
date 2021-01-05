---
title: Déplacer des régions pour des ressources dans Microsoft.Resources
description: Montre comment déplacer des ressources qui se trouvent dans l’espace de noms Microsoft.Resources vers de nouvelles régions.
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 098e5c36969da12966d793b6e1eddd4f79701ead
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88950809"
---
# <a name="move-microsoftresources-resources-to-new-region"></a>Déplacer des ressources Microsoft.Resources vers une nouvelle région

Il se peut que vous deviez déplacer une ressource existante vers une nouvelle région. Cet article explique comment déplacer deux types de ressources, templateSpecs et deploymentScripts, qui se trouvent dans l’espace de noms Microsoft.Resources.

## <a name="move-template-specs-to-new-region"></a>Déplacer des spécifications de modèle vers une nouvelle région

Si vous avez une [spec de modèle](../templates/template-specs.md) dans une région, et souhaitez la déplacer vers une nouvelle région, vous pouvez exporter la spec de modèle et la redéployer.

1. Utilisez la commande pour exporter une spec de modèle existante. Pour les valeurs de paramètres, fournissez les valeurs correspondant à la spec de modèle que vous souhaitez exporter.

   Pour Azure PowerShell, utilisez :

   ```azurepowershell
   Export-AzTemplateSpec `
     -ResourceGroupName demoRG `
     -Name demoTemplateSpec `
     -Version 1.0 `
     -OutputFolder c:\export
   ```

   Pour l’interface de ligne de commande Azure, consultez :

   ```azurecli
   az template-specs export \
     --resource-group demoRG \
     --name demoTemplateSpec \
     --version 1.0 \
     --output-folder c:\export
   ```

1. Utilisez la spec de modèle exportée pour créer une spec de modèle. Les exemples suivants montrent `westus` pour la nouvelle région, mais vous pouvez spécifier la région de votre choix.

   Pour Azure PowerShell, utilisez :

   ```azurepowershell
   New-AzTemplateSpec `
     -Name movedTemplateSpec `
     -Version 1.0 `
     -ResourceGroupName newRG `
     -Location westus `
     -TemplateJsonFile c:\export\1.0.json
   ```

   Pour l’interface de ligne de commande Azure, consultez :

   ```azurecli
   az template-specs create \
     --name movedTemplateSpec \
     --version "1.0" \
     --resource-group newRG \
     --location "westus" \
     --template-file "c:\export\demoTemplateSpec.json"
   ```

## <a name="move-deployment-scripts-to-new-region"></a>Déplacer des scripts de déploiement vers une nouvelle région

1. Sélectionnez le groupe de ressources contenant le script de déploiement que vous souhaitez déplacer vers une nouvelle région.

1. [Exporter le modèle](../templates/export-template-portal.md). Lors de l’exportation, sélectionnez le script de déploiement et d’autres ressources requises.

1. Dans le modèle exporté, supprimez les propriétés suivantes :

   * tenantId
   * principalId
   * clientId

1. Le modèle exporté a une valeur codée en dur pour la région du script de déploiement.

   ```json
   "location": "westus2",
   ```

   Modifiez le modèle pour autoriser un paramètre pour la définition de l’emplacement. Pour plus d’informations, consultez [Définir l’emplacement des ressources dans un modèle Resource Manager](../templates/resource-location.md).

   ```json
   "location": "[parameters('location')]",
   ```

1. [Déployez le modèle exporté](../templates/deploy-powershell.md), et spécifiez une nouvelle région pour le script de déploiement.

## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir le déplacement de ressources vers un nouveau groupe de ressources ou un nouvel abonnement, consultez [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](move-resource-group-and-subscription.md).
* Pour découvrir le déplacement de ressources vers une nouvelle région, consultez [Déplacement des ressources Azure entre les régions](move-region.md).
