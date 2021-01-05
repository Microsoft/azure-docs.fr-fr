---
title: Tutoriel - Ajouter une ressource au modèle
description: Décrit les étapes de la création de votre premier modèle ARM (Azure Resource Manager). Vous découvrez la syntaxe du fichier de modèle et comment déployer un compte de stockage.
author: mumian
ms.date: 03/27/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: 49cee5c98c4099e214a732371269e935db353152
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97106969"
---
# <a name="tutorial-add-a-resource-to-your-arm-template"></a>Tutoriel : Ajouter une ressource à votre modèle ARM

Dans le [précédent tutoriel](template-tutorial-create-first-template.md), vous avez appris à créer un modèle ARM (Azure Resource Manager) vide et à le déployer. Vous voici prêt à déployer une ressource réelle. Dans ce tutoriel, vous ajoutez un compte de stockage. Comptez environ **9 minutes** pour suivre ce tutoriel.

## <a name="prerequisites"></a>Prérequis

Nous vous recommandons de suivre le [tutoriel de présentation sur les modèles](template-tutorial-create-first-template.md), mais ce n’est pas obligatoire.

Vous devez disposer de Visual Studio Code avec l’extension Outils Resource Manager et, au choix, d’Azure PowerShell ou d’Azure CLI. Pour plus d’informations, consultez les [outils de modèle](template-tutorial-create-first-template.md#get-tools).

## <a name="add-resource"></a>Ajouter une ressource

Pour ajouter une définition de compte de stockage au modèle existant, examinez le code JSON mis en évidence dans l’exemple suivant. Plutôt que d’essayer de copier des sections du modèle, copiez l’intégralité du fichier et remplacez votre modèle par son contenu.

Remplacez `{provide-unique-name}` et les accolades `{}` par un nom de compte de stockage unique.

> [!IMPORTANT]
> Le nom du compte de stockage doit être unique dans Azure. Le nom ne doit contenir que des lettres minuscules ou des chiffres. Il ne doit pas compter plus de 24 caractères. Vous pouvez essayer un modèle de nommage, par exemple en utilisant **store1** comme préfixe et le faisant suivre de vos initiales et de la date du jour. Ainsi, le nom que vous utilisez peut ressembler à **store1abc09092019**.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-storage/azuredeploy.json" range="1-19" highlight="5-17":::

Trouver un nom unique pour un compte de stockage n’est pas facile, et ne fonctionne pas bien dans l’automatisation des déploiements à grande échelle. Plus loin dans cette série de tutoriels, vous utiliserez des fonctionnalités de modèle qui aident à la création d’un nom unique.

## <a name="resource-properties"></a>Propriétés de la ressource

Vous vous demandez peut-être comment trouver les propriétés à utiliser pour chaque type de ressource. Vous pouvez utiliser la [référence de modèle ARM](/azure/templates/) pour rechercher les types de ressources que vous souhaitez déployer.

Chaque ressource que vous déployez présente au moins les trois propriétés suivantes :

- `type` : Type de la ressource. Cette valeur est une combinaison de l’espace de noms du fournisseur de ressources et du type de ressource (comme `Microsoft.Storage/storageAccounts`).
- `apiVersion` : La version de l'API REST à utiliser pour la création de la ressource. Chaque fournisseur de ressources publiant ses propres versions d’API, cette valeur est donc propre au type.
- `name` : Nom de la ressource.

La plupart des ressources possèdent également une propriété `location`, qui définit la région dans laquelle la ressource est déployée.

Les autres propriétés varient selon le type de la ressource et la version de l’API. Il est important de comprendre le lien existant entre la version de l’API et les propriétés disponibles. Voyons donc cela d’un peu plus près.

Dans ce tutoriel, vous avez ajouté un compte de stockage au modèle. Vous pouvez voir cette version d’API à la page [storageAccounts 2019-04-01](/azure/templates/microsoft.storage/2019-04-01/storageaccounts). Notez que vous n’avez pas ajouté toutes les propriétés à votre modèle. Beaucoup de ces propriétés sont facultatives. Le fournisseur de ressources `Microsoft.Storage` peut publier une nouvelle version de l’API, pour autant la version que vous déployez ne doit pas nécessairement être changée. Vous pouvez continuer à utiliser cette version tout en sachant que les résultats de votre déploiement seront cohérents.

Si vous affichez une ancienne version de l’API, par exemple [storageAccounts 2016-05-01](/azure/templates/microsoft.storage/2016-05-01/storageaccounts), vous remarquerez qu’un ensemble de propriétés plus restreint est disponible.

Si vous décidez de modifier la version de l’API pour une ressource, veillez à évaluer les propriétés de cette version et à ajuster votre modèle en conséquence.

## <a name="deploy-template"></a>Déployer un modèle

Vous pouvez déployer le modèle pour créer le compte de stockage. Donnez un nom différent à votre déploiement, afin de pouvoir le retrouver facilement dans l’historique.

Si vous n’avez pas créé le groupe de ressources, consultez [Créer un groupe de ressources](template-tutorial-create-first-template.md#create-resource-group). L’exemple suppose que vous avez défini la variable `templateFile` sur le chemin du fichier de modèle, comme indiqué dans le [premier tutoriel](template-tutorial-create-first-template.md#deploy-template).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name addstorage `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour exécuter cette commande de déploiement, vous devez disposer de la [dernière version](/cli/azure/install-azure-cli) d’Azure CLI.

```azurecli
az deployment group create \
  --name addstorage \
  --resource-group myResourceGroup \
  --template-file $templateFile
```

---

> [!NOTE]
> Si le déploiement a échoué, utilisez le commutateur `verbose` pour obtenir des informations sur les ressources en cours de création. Utilisez le commutateur `debug` pour obtenir des informations supplémentaires sur le débogage.

Deux échecs de déploiement peuvent se produire :

- `Error: Code=AccountNameInvalid; Message={provide-unique-name}` n’est pas un nom de compte de stockage valide. Un nom de compte de stockage doit être compris entre 3 et 24 caractères composés uniquement de chiffres et de lettres en minuscules.

    Dans le modèle, remplacez `{provide-unique-name}` par un nom de compte de stockage unique. Consultez [Ajouter une ressource](#add-resource).

- `Error: Code=StorageAccountAlreadyTaken; Message=The storage account named store1abc09092019` est déjà pris.

    Dans le modèle, essayez un autre nom de compte de stockage.

Ce déploiement prend plus de temps que celui de votre modèle vide, car le compte de stockage est créé. Il faut compter environ une minute, mais l’opération est généralement plus rapide.

## <a name="verify-deployment"></a>Vérifier le déploiement

Vous pouvez vérifier le déploiement en explorant le groupe de ressources à partir du portail Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le menu de gauche, sélectionnez **Groupes de ressources**.
1. Sélectionnez le groupe de ressources sur lequel vous avez effectué le déploiement.
1. Vous voyez qu’un compte de stockage a été déployé.
1. Remarquez que l’étiquette de déploiement indique maintenant : **Déploiements : 2 Réussites**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous passez au tutoriel suivant, vous n’avez pas besoin de supprimer le groupe de ressources.

Si vous arrêtez maintenant, vous pouvez nettoyer les ressources que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** dans le menu de gauche.
2. Entrez le nom du groupe de ressources dans le champ **Filtrer par nom**.
3. Sélectionnez le nom du groupe de ressources.
4. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

## <a name="next-steps"></a>Étapes suivantes

Vous avez créé un modèle simple pour déployer un compte de stockage Azure. Dans les tutoriels suivants, vous allez découvrir comment ajouter des paramètres, des variables, des ressources et des sorties à un modèle. Ces fonctionnalités constituent les composants élémentaires de modèles bien plus complexes.

> [!div class="nextstepaction"]
> [Ajouter des paramètres](template-tutorial-add-parameters.md)
