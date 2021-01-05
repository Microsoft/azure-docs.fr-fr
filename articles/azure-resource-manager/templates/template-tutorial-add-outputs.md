---
title: Tutoriel - Ajouter des sorties au modèle
description: Ajoutez des sorties à votre modèle Azure Resource Manager (modèle ARM) pour simplifier la syntaxe.
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: 7acfb168462354017168ee333ea0b1ecaea434c1
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107122"
---
# <a name="tutorial-add-outputs-to-your-arm-template"></a>Tutoriel : Ajouter des sorties à votre modèle ARM

Dans ce tutoriel, vous allez apprendre à retourner une valeur à partir de votre modèle Azure Resource Manager (modèle ARM). Vous utilisez des sorties lorsque vous avez besoin d’une valeur provenant d’une ressource déployée. Ce tutoriel dure environ **7 minutes**.

## <a name="prerequisites"></a>Prérequis

Nous vous recommandons de suivre le [tutoriel sur les variables](template-tutorial-add-variables.md), mais ce n’est pas obligatoire.

Vous devez disposer de Visual Studio Code avec l’extension Outils Resource Manager et, au choix, d’Azure PowerShell ou d’Azure CLI. Pour plus d’informations, consultez les [outils de modèle](template-tutorial-create-first-template.md#get-tools).

## <a name="review-template"></a>Vérifier le modèle

À la fin du précédent tutoriel, votre modèle présentait le code JSON suivant :

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-variable/azuredeploy.json":::

Il déploie un compte de stockage, mais il ne retourne aucune information sur le compte de stockage. Vous aurez peut-être besoin de capturer les propriétés d’une nouvelle ressource afin de les avoir sous la main pour référence.

## <a name="add-outputs"></a>Ajouter des sorties

Vous pouvez utiliser des sorties pour retourner des valeurs à partir du modèle. Par exemple, il peut être utile d’obtenir les points de terminaison pour votre nouveau compte de stockage.

L’exemple suivant met en évidence la modification apportée à votre modèle pour ajouter une valeur de sortie. Copiez l’intégralité du fichier et remplacez votre modèle par son contenu.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-outputs/azuredeploy.json" range="1-53" highlight="47-52":::

Il y a quelques éléments importants à noter concernant la valeur de sortie que vous avez ajoutée.

Le type de valeur retournée est défini sur `object`, ce qui signifie qu’il retourne un objet JSON.

Il utilise la fonction [reference](template-functions-resource.md#reference) pour récupérer l’état d’exécution du compte de stockage. Pour obtenir l’état d’exécution d’une ressource, vous transmettez le nom ou l’ID d’une ressource. Dans ce cas, vous utilisez la même variable que celle que vous avez utilisée pour créer le nom du compte de stockage.

Finalement, il retourne la propriété `primaryEndpoints` à partir du compte de stockage.

## <a name="deploy-template"></a>Déployer un modèle

Vous êtes prêt à déployer le modèle et à examiner la valeur retournée.

Si vous n’avez pas créé le groupe de ressources, consultez [Créer un groupe de ressources](template-tutorial-create-first-template.md#create-resource-group). L’exemple suppose que vous avez défini la variable `templateFile` sur le chemin du fichier de modèle, comme indiqué dans le [premier tutoriel](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addoutputs `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile `
  -storagePrefix "store" `
  -storageSKU Standard_LRS
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour exécuter cette commande de déploiement, vous devez disposer de la [dernière version](/cli/azure/install-azure-cli) d’Azure CLI.

```azurecli
az deployment group create \
  --name addoutputs \
  --resource-group myResourceGroup \
  --template-file $templateFile \
  --parameters storagePrefix=store storageSKU=Standard_LRS
```

---

Dans la sortie de la commande de déploiement, un objet semblable à l’exemple suivant s’affiche uniquement si la sortie est au format JSON :

```json
{
    "dfs": "https://storeluktbfkpjjrkm.dfs.core.windows.net/",
    "web": "https://storeluktbfkpjjrkm.z19.web.core.windows.net/",
    "blob": "https://storeluktbfkpjjrkm.blob.core.windows.net/",
    "queue": "https://storeluktbfkpjjrkm.queue.core.windows.net/",
    "table": "https://storeluktbfkpjjrkm.table.core.windows.net/",
    "file": "https://storeluktbfkpjjrkm.file.core.windows.net/"
}
```

> [!NOTE]
> Si le déploiement a échoué, utilisez le commutateur `verbose` pour obtenir des informations sur les ressources en cours de création. Utilisez le commutateur `debug` pour obtenir des informations supplémentaires sur le débogage.

## <a name="review-your-work"></a>Passer en revue votre travail

Vous avez fait beaucoup de choses au cours des six derniers tutoriels. Prenons un moment pour regarder ensemble votre travail. Vous avez créé un modèle avec des paramètres qui sont faciles à fournir. Le modèle est réutilisable dans différents environnements parce qu’il peut être personnalisé et crée de façon dynamique les valeurs nécessaires. Il retourne également des informations sur le compte de stockage que vous pouvez utiliser dans votre script.

À présent, examinons le groupe de ressources et l’historique de déploiement.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le menu de gauche, sélectionnez **Groupes de ressources**.
1. Sélectionnez le groupe de ressources sur lequel vous avez effectué le déploiement.
1. Selon les étapes que vous avez effectuées, vous devez avoir au moins un compte de stockage, si ce n’est plusieurs, dans le groupe de ressources.
1. Vous devez également disposer de plusieurs déploiements réussis dans l’historique. Sélectionnez ce lien.

   ![Sélectionnez les déploiements.](./media/template-tutorial-add-outputs/select-deployments.png)

1. Vous voyez tous vos déploiements dans l’historique. Sélectionnez le déploiement nommé **addoutputs**.

   ![Affichage de l’historique des déploiements.](./media/template-tutorial-add-outputs/show-history.png)

1. Vous pouvez passer en revue les entrées.

   ![Affichage des entrées](./media/template-tutorial-add-outputs/show-inputs.png)

1. Vous pouvez examiner les sorties.

   ![Affichage des sorties](./media/template-tutorial-add-outputs/show-outputs.png)

1. Vous pouvez examiner le modèle.

   ![Afficher le modèle](./media/template-tutorial-add-outputs/show-template.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous passez au tutoriel suivant, vous n’avez pas besoin de supprimer le groupe de ressources.

Si vous arrêtez maintenant, vous pouvez nettoyer les ressources que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** dans le menu de gauche.
2. Entrez le nom du groupe de ressources dans le champ **Filtrer par nom**.
3. Sélectionnez le nom du groupe de ressources.
4. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez ajouté une valeur retournée au modèle. Dans le tutoriel suivant, vous allez apprendre à exporter un modèle et à utiliser des parties de ce modèle exporté dans votre modèle.

> [!div class="nextstepaction"]
> [Utiliser un modèle exporté](template-tutorial-export-template.md)
