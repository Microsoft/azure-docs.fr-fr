---
title: Créer un certificat Azure Attestation à l’aide d’un modèle Azure Resource Manager
description: Découvrez comment créer un certificat Azure Attestation à l’aide d’un modèle Azure Resource Manager.
services: azure-resource-manager
author: msmbaldwin
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: mbaldwin
ms.date: 10/16/2020
ms.openlocfilehash: 8c56a37ebcc799b0170785666212eb4e3b00a5f0
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92144974"
---
# <a name="quickstart-create-an-azure-attestation-provider-with-an-arm-template"></a>Démarrage rapide : Créer un fournisseur Azure Attestation avec un modèle Resource Manager

[Microsoft Azure Attestation](overview.md) est une solution pour l’attestation des environnements d’exécution approuvés (TEE, Trusted Execution Environment). Ce guide de démarrage rapide porte essentiellement sur le processus de déploiement d’un modèle ARM (Azure Resource Manager) en vue de créer une stratégie Microsoft Azure Attestation.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure** . Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-attestation-provider-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-attestation-provider-create).

:::code language="json" source="~/quickstart-templates/101-attestation-provider-create/azuredeploy.json":::

Ressources Azure définies dans le modèle :

- Microsoft.Attestation/attestationProviders

## <a name="deploy-the-template"></a>Déployer le modèle

1. Sélectionnez l’image ci-après pour vous connecter à Azure et ouvrir le modèle.

    [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-attestation-provider-create%2Fazuredeploy.json)

1. Sélectionnez ou entrez les valeurs suivantes.

    Sauf si elle est spécifiée, utilisez la valeur par défaut pour créer le fournisseur d’attestations.

    - **Nom du fournisseur d’attestation**  : Sélectionnez un nom pour votre fournisseur Azure Attestation.
    - **Emplacement** : Sélectionnez un emplacement. Par exemple, **USA Centre** .
    - **Étiquettes**  : Sélectionnez un emplacement. Par exemple, **USA Centre** .

1. Sélectionnez **Achat** . Une fois que la ressource d’attestation a été déployée, vous recevez une notification.

Le portail Azure est utilisé pour déployer le modèle. Outre le portail Azure, vous pouvez également utiliser Azure PowerShell, l’interface Azure CLI et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Vous pouvez utiliser le portail Azure pour vérifier la ressource d’attestation.

## <a name="clean-up-resources"></a>Nettoyer les ressources

D’autres aspects d’Azure Attestation reposent sur ce démarrage rapide. Si vous prévoyez d’utiliser d’autres démarrages rapides et didacticiels, il peut être utile de conserver ces ressources.

Si vous n’en avez plus besoin, supprimez le groupe de ressources. La ressource Attestation sera également supprimée. Pour supprimer le groupe de ressources avec Azure CLI ou Azure PowerShell :

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une ressource d’attestation à l’aide d’un modèle Resource Manager et avez validé le déploiement. Pour en savoir plus sur Azure Attestation, consultez [Vue d’ensemble d’Azure Attestation](overview.md).
