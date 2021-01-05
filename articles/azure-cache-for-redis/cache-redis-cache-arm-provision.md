---
title: Déployez Azure Cache pour Redis en utilisant un modèle Resource Manager
description: Découvrez comment utiliser un modèle Azure Resource Manager (modèle ARM) pour déployer une ressource Azure Cache pour Redis. Des modèles sont fournis pour les scénarios les plus courants.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.custom: subject-armqs
ms.date: 08/18/2020
ms.openlocfilehash: 8bd9a45ec7c43d9338dec184afd784d2b163c410
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92735967"
---
# <a name="quickstart-create-an-azure-cache-for-redis-using-an-arm-template"></a>Démarrage rapide : Créer un répartiteur Azure Cache pour Redis à l’aide d’un modèle ARM

Apprenez à créer un modèle Azure Resource Manager (modèle ARM) qui déploie un répartiteur Azure Cache pour Redis. Le cache peut être utilisé avec un compte de stockage existant pour conserver les données de diagnostic. Vous allez également apprendre comment définir les ressources à déployer et configurer les paramètres qui sont spécifiés lors de l’exécution du déploiement. Vous pouvez utiliser ce modèle pour vos propres déploiements, ou le personnaliser afin qu’il réponde à vos besoins. Actuellement, les paramètres de diagnostic sont partagés entre tous les caches de la même région d’un abonnement. La mise à jour d’un cache dans la région affecte tous les autres caches dans la région.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-redis-cache%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

* **Abonnement Azure** : Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* **Compte de stockage**  : pour en créer un, consultez [Créer un compte de stockage Azure](../storage/common/storage-account-create.md?tabs=azure-portal). Le compte de stockage est utilisé pour les données de diagnostic.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-redis-cache/).

:::code language="json" source="~/quickstart-templates/101-redis-cache/azuredeploy.json":::

Les ressources suivantes sont définies dans le modèle :

* [Microsoft.Cache/Redis](/azure/templates/microsoft.cache/redis)
* [Microsoft.Insights/diagnosticsettings](/azure/templates/microsoft.insights/diagnosticsettings)

Des modèles Resource Manager pour le nouveau [niveau Premium](cache-overview.md#service-tiers) sont également disponibles.

* [Créer un cache Azure Premium pour Redis avec clustering](https://azure.microsoft.com/resources/templates/201-redis-premium-cluster-diagnostics/)
* [Créer un cache Azure Premium pour Redis avec persistance des données](https://azure.microsoft.com/resources/templates/201-redis-premium-persistence/)
* [Créer un Cache Redis Premium déployé dans un réseau virtuel](https://azure.microsoft.com/resources/templates/201-redis-premium-vnet/)

Pour connaître les derniers modèles, consultez [Modèles de démarrage rapide Azure](https://azure.microsoft.com/documentation/templates/) et recherchez _Azure Cache pour Redis_.

## <a name="deploy-the-template"></a>Déployer le modèle

1. Sélectionnez l’image ci-après pour vous connecter à Azure et ouvrir le modèle.

    [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-redis-cache%2Fazuredeploy.json)
1. Sélectionnez ou entrez les valeurs suivantes :

    * **Abonnement**  : sélectionnez un abonnement Azure utilisé pour créer le partage de données et les autres ressources.
    * **Groupe de ressources**  : sélectionnez **Créer** pour créer un groupe de ressources ou sélectionnez un groupe de ressources existant.
    * **Emplacement** : sélectionnez un emplacement pour le groupe de ressources. Le compte de stockage et le cache Redis doivent se trouver dans la même région. Par défaut, le cache Redis utilise le même emplacement que le groupe de ressources. Spécifiez donc le même emplacement que le compte de stockage.
    * **Nom du cache Redis**  : entrez un nom pour le cache Redis.
    * **Compte de stockage de diagnostics existant**  : entrez l’ID de ressource d’un compte de stockage. La syntaxe est `/subscriptions/&lt;SUBSCRIPTION ID>/resourceGroups/&lt;RESOURCE GROUP NAME>/providers/Microsoft.Storage/storageAccounts/&lt;STORAGE ACCOUNT NAME>`.

    Utilisez la valeur par défaut pour les autres paramètres.
1. Sélectionnez **J'accepte les conditions générales mentionnées ci-dessus** , puis **Acheter**.

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Ouvrez le cache Redis que vous avez créé.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources, ce qui supprime également les ressources qu’il contient.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à créer un modèle Resource Manager qui déploie une instance Azure Cache pour Redis. Pour apprendre à créer un modèle Resource Manager qui déploie une application web Azure avec Azure Cache pour Redis, consultez [Créer une application web avec Azure Cache pour Redis à l’aide d’un modèle](./cache-web-app-arm-with-redis-cache-provision.md).
