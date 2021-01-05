---
title: Simulation de déploiement de modèle
description: Déterminez les modifications qui seront apportées à vos ressources avant de déployer un modèle Azure Resource Manager.
author: tfitzmac
ms.topic: conceptual
ms.date: 12/15/2020
ms.author: tomfitz
ms.openlocfilehash: a1ce7f8f718b364dc4b47593cf9ea37e8baf1e72
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97563090"
---
# <a name="arm-template-deployment-what-if-operation"></a>Opération what-if de déploiement de modèle ARM

Avant de déployer un modèle Azure Resource Manager (modèle ARM), vous pouvez prévisualiser les changements qui se produiront. Azure Resource Manager met à votre disposition l’opération de simulation, qui vous permet de voir comment les ressources changent si vous déployez le modèle. L’opération de simulation n’apporte aucune modification aux ressources existantes. Au lieu de cela, elle prédit les modifications si le modèle spécifié est déployé.

Vous pouvez utiliser l’opération de simulation avec des opérations d’Azure PowerShell, d’Azure CLI ou d’API REST. La simulation est prise en charge pour les déploiements de groupe de ressources, d’abonnement et de niveau locataire.

## <a name="install-azure-powershell-module"></a>Installer le module Azure PowerShell

Pour utiliser une simulation dans PowerShell, vous devez disposer de la version **4.2 ou d’une version ultérieure du module Az**.

Toutefois, avant d’installer le module requis, vérifiez que vous disposez de PowerShell Core (6.x ou 7.x). Si vous disposez de PowerShell 5.x ou version antérieure, [mettez à jour votre version de PowerShell](/powershell/scripting/install/installing-powershell). Vous ne pouvez pas installer le module requis sur PowerShell version 5.x ou antérieure.

### <a name="install-latest-version"></a>Installer la dernière version

Pour installer le module, utilisez :

```powershell
Install-Module -Name Az -Force
```

Pour plus d’informations sur l’installation des modules, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="install-azure-cli-module"></a>Installer le module Azure CLI

Pour utiliser une simulation dans Azure CLI, vous devez disposer d’Azure CLI 2.5.0 ou une version ultérieure. Si nécessaire, [installez la dernière version d’Azure CLI](/cli/azure/install-azure-cli).

## <a name="see-results"></a>Afficher les résultats

Lorsque vous utilisez une simulation dans PowerShell ou Azure CLI, la sortie comprend des résultats à code de couleurs qui vous permettent de distinguer les différents types de modifications.

![Opération de simulation de déploiement de modèle Resource Manager : charge utile de ressource complète et types de modification](./media/template-deploy-what-if/resource-manager-deployment-whatif-change-types.png)

La sortie de texte est :

```powershell
Resource and property changes are indicated with these symbols:
  - Delete
  + Create
  ~ Modify

The deployment will update the following scope:

Scope: /subscriptions/./resourceGroups/ExampleGroup

  ~ Microsoft.Network/virtualNetworks/vnet-001 [2018-10-01]
    - tags.Owner: "Team A"
    ~ properties.addressSpace.addressPrefixes: [
      - 0: "10.0.0.0/16"
      + 0: "10.0.0.0/15"
      ]
    ~ properties.subnets: [
      - 0:

          name:                     "subnet001"
          properties.addressPrefix: "10.0.0.0/24"

      ]

Resource changes: 1 to modify.
```

> [!NOTE]
> L’opération de simulation ne peut pas résoudre la [fonction de référence](template-functions-resource.md#reference). Chaque fois que vous définissez une propriété sur une expression de modèle qui inclut la fonction de référence, les rapports de simulation de la propriété changent. Ce comportement se produit parce que la simulation compare la valeur actuelle de la propriété (telle que `true` ou `false` pour une valeur booléenne) à l’expression de modèle non résolue. Évidemment, ces valeurs ne correspondent pas. Lorsque vous déployez le modèle, la propriété change uniquement lorsque la résolution de l’expression de modèle produit une valeur différente.

## <a name="what-if-commands"></a>Commandes de simulation

### <a name="azure-powershell"></a>Azure PowerShell

Pour afficher un aperçu des modifications avant de déployer un modèle, utilisez la commande [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) ou [New-AzSubscriptionDeployment](/powershell/module/az.resources/new-azdeployment). Ajoutez le paramètre de commutateur `-Whatif` à la commande de déploiement.

* `New-AzResourceGroupDeployment -Whatif` pour des déploiements de groupes de ressources
* `New-AzSubscriptionDeployment -Whatif` et `New-AzDeployment -Whatif` pour des déploiements au niveau de l’abonnement

Vous pouvez utiliser le paramètre de commutateur `-Confirm` pour afficher un aperçu des modifications et être invité à poursuivre le déploiement.

* `New-AzResourceGroupDeployment -Confirm` pour des déploiements de groupes de ressources
* `New-AzSubscriptionDeployment -Confirm` et `New-AzDeployment -Confirm` pour des déploiements au niveau de l’abonnement

Les commandes précédentes retournent un résumé sous forme de texte que vous pouvez inspecter manuellement. Pour obtenir un objet dont vous pouvez inspecter les modifications par programmation, utilisez la commande [AzResourceGroupDeploymentWhatIfResult](/powershell/module/az.resources/get-azresourcegroupdeploymentwhatifresult) ou [AzSubscriptionDeploymentWhatIfResult](/powershell/module/az.resources/get-azdeploymentwhatifresult).

* `$results = Get-AzResourceGroupDeploymentWhatIfResult` pour des déploiements de groupes de ressources
* `$results = Get-AzSubscriptionDeploymentWhatIfResult` ou `$results = Get-AzDeploymentWhatIfResult` pour des déploiements au niveau de l’abonnement

### <a name="azure-cli"></a>Azure CLI

Pour afficher un aperçu des modifications avant de déployer un modèle, utilisez :

* [az deployment group what-if](/cli/azure/deployment/group#az-deployment-group-what-if) pour les déploiements de groupes de ressources
* [az deployment sub what-if](/cli/azure/deployment/sub#az-deployment-sub-what-if) pour les déploiements au niveau de l’abonnement
* [az deployment mg what-if](/cli/azure/deployment/mg#az-deployment-mg-what-if) pour les déploiements de groupes de gestion
* [az deployment tenant what-if](/cli/azure/deployment/tenant#az-deployment-tenant-what-if) pour les déploiements de locataires

Vous pouvez utiliser le commutateur `--confirm-with-what-if` (ou sa forme courte `-c`) pour afficher un aperçu des modifications et être invité à poursuivre le déploiement. Ajoutez ce commutateur à :

* [az deployment group create](/cli/azure/deployment/group#az-deployment-group-create)
* [az deployment sub create](/cli/azure/deployment/sub#az-deployment-sub-create).
* [az deployment mg create](/cli/azure/deployment/mg#az-deployment-mg-create)
* [az deployment tenant create](/cli/azure/deployment/tenant#az-deployment-tenant-create)

Par exemple, utilisez `az deployment group create --confirm-with-what-if` ou `-c` pour des déploiements de groupe de ressources.

Les commandes précédentes retournent un résumé sous forme de texte que vous pouvez inspecter manuellement. Pour obtenir un objet JSON que vous pouvez inspecter par programme pour y détecter des modifications, utilisez le commutateur `--no-pretty-print`. Par exemple, utilisez `az deployment group what-if --no-pretty-print` pour des déploiements de groupe de ressources.

Si vous souhaitez retourner les résultats sans couleurs, ouvrez votre fichier de [configuration d’Azure CLI](/cli/azure/azure-cli-configuration). Définissez **no_color** sur **Oui**.

### <a name="azure-rest-api"></a>API REST Azure

Pour l’API REST, utilisez :

* [Déploiements – Simulation](/rest/api/resources/deployments/whatif) pour les déploiements de groupes de ressources
* [Déploiements – Simulation au niveau de l’étendue d’abonnement](/rest/api/resources/deployments/whatifatsubscriptionscope) pour les déploiements d’abonnements
* [Déploiements What If au niveau de l’étendue du groupe d’administration](/rest/api/resources/deployments/whatifatmanagementgroupscope) pour les déploiements de groupes d’administration
* [Déploiements What If au niveau de l’étendue du locataire](/rest/api/resources/deployments/whatifattenantscope) pour les déploiements de locataires.

## <a name="change-types"></a>Types de modification

L’opération de simulation liste six types différents de modifications :

- **Créer** : la ressource n’existe pas, mais elle est définie dans le modèle. La ressource est créée.

- **Supprimer** : ce type de modification s’applique uniquement lors de l’utilisation du [mode Complete](deployment-modes.md) pour le déploiement. la ressource existe, mais elle n’est pas définie dans le modèle. En mode Complete, la ressource est supprimée. Seules les ressources qui [prennent en charge la suppression en mode Complete](complete-mode-deletion.md) sont incluses dans ce type de modification.

- **Ignore** : la ressource existe, mais elle n’est pas définie dans le modèle. La ressource n’est pas déployée ou modifiée.

- **NoChange** : la ressource existe et est définie dans le modèle. La ressource est redéployée, mais ses propriétés ne changent pas. Ce type de modification est retourné quand [ResultFormat](#result-format) est défini sur `FullResourcePayloads`, qui est la valeur par défaut.

- **Modify** : la ressource existe et est définie dans le modèle. La ressource est redéployée et ses propriétés changent. Ce type de modification est retourné quand [ResultFormat](#result-format) est défini sur `FullResourcePayloads`, qui est la valeur par défaut.

- **Deploy** : la ressource existe et est définie dans le modèle. La ressource est redéployée. Il est possible que les propriétés de la ressource changent. L’opération retourne ce type de modification quand elle ne dispose pas de suffisamment d’informations pour déterminer si des propriétés changeront. Vous n’observez cette condition que si [ResultFormat](#result-format) est défini sur `ResourceIdOnly`.

## <a name="result-format"></a>Format de résultat

Vous pouvez contrôler le niveau de détail retourné sur les modifications prédites. Deux options s'offrent à vous :

* **FullResourcePayloads** retourne la liste des ressources qui changeront et des détails sur les propriétés qui changeront.
* **ResourceIdOnly** retourne la liste des ressources qui seront modifiées.

La valeur par défaut est **FullResourcePayloads**.

Pour les commandes de déploiement PowerShell, utilisez le paramètre `-WhatIfResultFormat`. Dans les commandes d’objet programmatique, utilisez le paramètre `ResultFormat`.

Pour Azure CLI, utilisez le paramètre `--result-format`.
 
Les résultats suivants illustrent les deux formats de sortie :

- Charges utiles de ressources complètes

  ```powershell
  Resource and property changes are indicated with these symbols:
    - Delete
    + Create
    ~ Modify

  The deployment will update the following scope:

  Scope: /subscriptions/./resourceGroups/ExampleGroup

    ~ Microsoft.Network/virtualNetworks/vnet-001 [2018-10-01]
      - tags.Owner: "Team A"
      ~ properties.addressSpace.addressPrefixes: [
        - 0: "10.0.0.0/16"
        + 0: "10.0.0.0/15"
        ]
      ~ properties.subnets: [
        - 0:

          name:                     "subnet001"
          properties.addressPrefix: "10.0.0.0/24"

        ]

  Resource changes: 1 to modify.
  ```

- ID de ressource uniquement

  ```powershell
  Resource and property changes are indicated with this symbol:
    ! Deploy

  The deployment will update the following scope:

  Scope: /subscriptions/./resourceGroups/ExampleGroup

    ! Microsoft.Network/virtualNetworks/vnet-001

  Resource changes: 1 to deploy.
  ```

## <a name="run-what-if-operation"></a>Exécuter l’opération de simulation

### <a name="set-up-environment"></a>Configurer l’environnement

Pour voir comment fonctionne la simulation, nous allons exécuter des tests. Tout d’abord, déployez un [modèle de qui crée un réseau virtuel](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/what-if/what-if-before.json). Vous utiliserez ce réseau virtuel pour tester dans quelle mesure les modifications sont signalées par l’opération de simulation.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroup `
  -Name ExampleGroup `
  -Location centralus
New-AzResourceGroupDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-before.json"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az group create \
  --name ExampleGroup \
  --location "Central US"
az deployment group create \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-before.json"
```

---

### <a name="test-modification"></a>Modification de test

Une fois le déploiement terminé, vous êtes prêt à tester l’opération de simulation. Cette fois, vous déployez un [modèle qui modifie le réseau virtuel](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/what-if/what-if-after.json). Il manque une des balises d’origine, un sous-réseau a été supprimé et le préfixe de l’adresse a changé.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Whatif `
  -ResourceGroupName ExampleGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-after.json"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group what-if \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-after.json"
```

---

La sortie de simulation ressemble à ceci :

![Sortie de l’opération de simulation de déploiement de modèle Resource Manager](./media/template-deploy-what-if/resource-manager-deployment-whatif-change-types.png)

La sortie de texte est :

```powershell
Resource and property changes are indicated with these symbols:
  - Delete
  + Create
  ~ Modify

The deployment will update the following scope:

Scope: /subscriptions/./resourceGroups/ExampleGroup

  ~ Microsoft.Network/virtualNetworks/vnet-001 [2018-10-01]
    - tags.Owner: "Team A"
    ~ properties.addressSpace.addressPrefixes: [
      - 0: "10.0.0.0/16"
      + 0: "10.0.0.0/15"
      ]
    ~ properties.subnets: [
      - 0:

        name:                     "subnet001"
        properties.addressPrefix: "10.0.0.0/24"

      ]

Resource changes: 1 to modify.
```

Notez que, en haut de la sortie, les couleurs sont définies pour indiquer le type de modifications.

Au bas de la sortie apparaît la balise indiquant que le propriétaire a été supprimé. Le préfixe d’adresse est passé de 10.0.0.0/16 à 10.0.0.0/15. Le sous-réseau nommé subnet001 a été supprimé. N’oubliez pas que ces modifications n’ont pas été déployées. Vous voyez un aperçu des modifications qui se produiront si vous déployez le modèle.

Certaines des propriétés répertoriées comme supprimées ne seront pas modifiées. Les propriétés peuvent être incorrectement signalées comme supprimées lorsqu’elles ne sont pas dans le modèle, mais elles sont automatiquement définies comme valeurs par défaut lors du déploiement. Ce résultat est considéré comme du « bruit » dans la réponse de simulation. La ressource déployée finale aura les valeurs définies pour les propriétés. À mesure que l’opération de simulation évolue, ces propriétés sont exclues du résultat.

## <a name="programmatically-evaluate-what-if-results"></a>Évaluer par programmation les résultats de l’opération de simulation

À présent, nous allons évaluer par programmation les résultats de l’opération de simulation en définissant la commande sur une variable.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$results = Get-AzResourceGroupDeploymentWhatIfResult `
  -ResourceGroupName ExampleGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-after.json"
```

Vous pouvez voir un résumé de chaque modification.

```azurepowershell
foreach ($change in $results.Changes)
{
  $change.Delta
}
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
results=$(az deployment group what-if --resource-group ExampleGroup --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-after.json" --no-pretty-print)
```

---

## <a name="confirm-deletion"></a>Confirmer la suppression

L’opération de simulation prend en charge l’utilisation du [mode de déploiement](deployment-modes.md). En mode Complete, les ressources qui ne sont pas dans le modèle sont supprimées. L’exemple suivant déploie un [modèle qui n’a aucune ressource définie](https://github.com/Azure/azure-docs-json-samples/blob/master/empty-template/azuredeploy.json) en mode Complete.

Pour afficher un aperçu des modifications avant de déployer un modèle, utilisez le paramètre de commutateur de confirmation avec la commande de déploiement. Si les modifications sont bien celles que vous attendiez, confirmez que vous souhaitez accomplir le déploiement.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -ResourceGroupName ExampleGroup `
  -Mode Complete `
  -Confirm `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/empty-template/azuredeploy.json"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --resource-group ExampleGroup \
  --mode Complete \
  --confirm-with-what-if \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/empty-template/azuredeploy.json"
```

---

Étant donné qu’aucune ressource n’est définie dans le modèle et que le mode de déploiement est défini sur Complet, le réseau virtuel sera supprimé.

![Sortie de l’opération de simulation de déploiement de modèle Resource Manager : mode de déploiement Complete](./media/template-deploy-what-if/resource-manager-deployment-whatif-output-mode-complete.png)

La sortie de texte est :

```powershell
Resource and property changes are indicated with this symbol:
  - Delete

The deployment will update the following scope:

Scope: /subscriptions/./resourceGroups/ExampleGroup

  - Microsoft.Network/virtualNetworks/vnet-001

      id:
"/subscriptions/./resourceGroups/ExampleGroup/providers/Microsoft.Network/virtualNet
works/vnet-001"
      location:        "centralus"
      name:            "vnet-001"
      tags.CostCenter: "12345"
      tags.Owner:      "Team A"
      type:            "Microsoft.Network/virtualNetworks"

Resource changes: 1 to delete.

Are you sure you want to execute the deployment?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"):
```

Vous voyez les modifications attendues et pouvez confirmer que vous souhaitez que le déploiement s’exécute.

## <a name="sdks"></a>Kits SDK

Vous pouvez utiliser l’opération de simulation par le biais des Kits de développement logiciel (SDK) Azure.

* Pour Python, utilisez [what-if (simulation)](/python/api/azure-mgmt-resource/azure.mgmt.resource.resources.v2019_10_01.operations.deploymentsoperations#what-if-resource-group-name--deployment-name--properties--location-none--custom-headers-none--raw-false--polling-true----operation-config-).

* Pour Java, utilisez [DeploymentWhatIf Class](/java/api/com.microsoft.azure.management.resources.deploymentwhatif).

* Pour .NET, utilisez [DeploymentWhatIf Class](/dotnet/api/microsoft.azure.management.resourcemanager.models.deploymentwhatif).

## <a name="next-steps"></a>Étapes suivantes

- Si vous constatez que l’opération de simulation génère des résultats incorrects, signalez les problèmes via [https://aka.ms/whatifissues](https://aka.ms/whatifissues).
- Pour déployer des modèles avec Azure PowerShell, consultez [Déployer des ressources avec des modèles ARM et Azure PowerShell](deploy-powershell.md).
- Pour déployer des modèles avec Azure CLI, consultez [Déployer des ressources avec des modèles ARM et Azure CLI](deploy-cli.md).
- Pour déployer des modèles avec REST, consultez [Déployer des ressources avec des modèles ARM et l’API REST Resource Manager](deploy-rest.md).
