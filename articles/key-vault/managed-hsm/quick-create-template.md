---
title: Démarrage rapide Azure – Créer un HSM managé avec un modèle Azure Resource Manager
description: Démarrage rapide montrant comment créer un HSM managé Azure Key Vault à l’aide d’un modèle Resource Manager
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: quickstart
ms.custom: mvc, devx-track-azurecli
ms.date: 09/15/2020
ms.author: mbaldwin
ms.openlocfilehash: d47935f76347b2d5272b386942a85643a732e643
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94831750"
---
# <a name="quickstart-create-an-key-vault-managed-hsm-using-an-azure-resource-manager-template"></a>Démarrage rapide : Créer un HSM managé Key Vault avec un modèle Azure Resource Manager

Managed HSM est un service cloud complètement managé, à haut niveau de disponibilité et monolocataire qui vous permet de protéger les clés de chiffrement de vos applications cloud à l’aide de HSM certifiés **FIPS 140-2 de niveau 3**.  

Ce guide de démarrage rapide porte essentiellement sur le déploiement d’un modèle Resource Manager en vue de créer un HSM managé.  Un [modèle Resource Manager](../../azure-resource-manager/templates/overview.md) est un fichier JSON (JavaScript Object Notation) qui définit l’infrastructure et la configuration pour votre projet. Le modèle utilise la syntaxe déclarative, qui vous permet d’indiquer ce que vous envisagez de déployer sans avoir à écrire la séquence de commandes de programmation pour le créer. Pour plus d’informations sur le développement de modèles Resource Manager, consultez la [documentation Resource Manager](../../azure-resource-manager/index.yml) et les [informations de référence sur les modèles](/azure/templates/microsoft.keyvault/allversions).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de cet article, vous devez disposer des éléments suivants :

- Un abonnement à Microsoft Azure. Si vous n’en avez pas, vous pouvez vous inscrire pour bénéficier d’un [essai gratuit](https://azure.microsoft.com/pricing/free-trial).
- Azure CLI 2.12.0 ou version ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer l’interface de ligne de commande Azure]( /cli/azure/install-azure-cli)


[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-to-azure"></a>Connexion à Azure

Pour vous connecter à Azure à l’aide de l’interface CLI, vous pouvez taper la commande suivante :

```azurecli
az login
```

Pour plus d’informations sur les options de connexion via l’interface CLI, consultez [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli?view=azure-cli-latest&preserve-view=true)

## <a name="create-a-manage-hsm"></a>Créer un HSM managé

Le modèle utilisé dans ce guide de démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-managed-hsm-create/).

Ressource Azure définie dans le modèle :

* **Microsoft. keyvault/managedHSMs** : créez un HSM managé Azure Key Vault.

Vous pouvez trouver d’autres exemples de modèles Azure Key Vault [ici](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Keyvault).

Le modèle a besoin de l’ID d’objet associé à votre compte. Pour le trouver, utilisez la commande Azure CLI [az ad user show](/cli/azure/ad/user?view=azure-cli-latest&preserve-view=true#az_ad_user_show) en transmettant votre adresse e-mail au paramètre `--id`. Vous pouvez limiter la sortie au seul ID d’objet avec le paramètre `--query`.

```azurecli-interactive
az ad user show --id <your-email-address> --query "objectId"
```

Il se peut que vous ayez aussi besoin de votre ID de locataire. Pour le trouver, utilisez la commande Azure CLI [az ad user show](/cli/azure/account?view=azure-cli-latest&preserve-view=true#az_account_show). Vous pouvez limiter la sortie au seul ID de locataire avec le paramètre `--query`.

 ```azurecli-interactive
 az account show --query "tenantId"
 ```

1. Cliquez sur l’image ci-après pour vous connecter à Azure et ouvrir un modèle. Le modèle crée un coffre de clés et un secret.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-managed-hsm-create%2Fazuredeploy.json"><img src="../media/deploy-to-azure.svg" alt="deploy to azure"/></a>

2. Sélectionnez ou entrez les valeurs suivantes.

    Sauf si elle est spécifiée, utilisez la valeur par défaut pour créer le coffre de clés et un secret.

    - **Abonnement**: Sélectionnez un abonnement Azure.
    - **Groupe de ressources** : Sélectionnez **Créer**, entrez un nom unique pour le groupe de ressources, puis cliquez sur **OK**.
    - **Emplacement** : Sélectionnez un emplacement. Par exemple, **USA Centre Sud**.
    - **managedHSMName** : attribuez un nom à votre HSM managé.
    - **ID du locataire** : la fonction de modèle récupère automatiquement votre ID de locataire ; ne changez pas la valeur par défaut.  En l’absence de valeur, entrez l’ID de locataire que vous avez récupéré dans [Prérequis](#prerequisites).
    * **initialAdminObjectIds** : entrez l’ID d’objet que vous avez récupéré dans [Prérequis](#prerequisites).

3. Sélectionnez **Achat**. Une fois que le coffre de clés a été déployé avec succès, vous recevez une notification :

Le portail Azure est utilisé pour déployer le modèle. Outre le portail Azure, vous pouvez également utiliser Azure PowerShell, l’interface Azure CLI et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../../azure-resource-manager/templates/deploy-powershell.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un HSM managé. Ce HSM managé ne sera pas entièrement opérationnel tant qu’il n’aura pas été activé. Consultez [Activer votre HSM managé](quick-create-cli.md#activate-your-managed-hsm) pour savoir comment activer votre HSM.

- Lire une [présentation de Managed HSM](overview.md)
- Découvrir la [gestion des clés dans un HSM managé](key-management.md)
- Passer en revue les [bonnes pratiques de Managed HSM](best-practices.md)
