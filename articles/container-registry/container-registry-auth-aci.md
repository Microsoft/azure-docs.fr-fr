---
title: Accès à partir de Container Instances
description: Découvrez comment fournir un accès aux images de votre registre de conteneurs privé à partir d’Azure Container Instances à l’aide d’un principal de service Azure Active Directory.
ms.topic: article
ms.date: 04/23/2018
ms.openlocfilehash: b1bc8119c495dea99c6bdc4923db198d041a1e9e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74456504"
---
# <a name="authenticate-with-azure-container-registry-from-azure-container-instances"></a>S’authentifier avec Azure Container Registry à partir d’Azure Container Instances

Vous pouvez utiliser un principal de service Azure Active Directory (Azure AD) pour fournir l’accès à vos registres de conteneurs privés dans Azure Container Registry.

Dans cet article, vous apprendrez à créer et à configurer un principal de service Azure AD avec l’autorisation d’accès en *extraction* à votre registre. Dans Azure Container Instances (ACI), vous démarrerez ensuite un conteneur qui extrait son image de votre registre privé, en utilisant le principal de service pour l’authentification.

## <a name="when-to-use-a-service-principal"></a>Quand utiliser un principal de service

Vous devez utiliser un principal de service pour l’authentification à partir d’ACI dans les **scénarios sans affichage**, par exemple dans les applications ou services qui créent des instances de conteneur de façon automatisée ou sans assistance.

Par exemple, si vous disposez d’un script automatisé qui s’exécute la nuit et crée une [instance de conteneur basée sur les tâches](../container-instances/container-instances-restart-policy.md) pour traiter des données, il peut utiliser un principal de service avec une autorisation d’extraction uniquement pour s’authentifier auprès du registre. Vous pouvez ensuite faire tourner les informations d’identification du principal de service ou révoquer complètement son accès sans affecter les autres applications et services.

Les principaux de service doivent également être utilisés lorsque le registre [admin user](container-registry-authentication.md#admin-account) est désactivé.

[!INCLUDE [container-registry-service-principal](../../includes/container-registry-service-principal.md)]

## <a name="authenticate-using-the-service-principal"></a>S’authentifier à l’aide du principal de service

Pour lancer un conteneur dans Azure Container Instances à l’aide d’un principal de service, spécifiez l’ID pour `--registry-username` et son mot de passe pour `--registry-password`.

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mycontainerregistry.azurecr.io/myimage:v1 \
    --registry-login-server mycontainerregistry.azurecr.io \
    --registry-username <service-principal-ID> \
    --registry-password <service-principal-password>
```

## <a name="sample-scripts"></a>Exemples de scripts

Vous trouverez les exemples de scripts précédents pour Azure CLI sur GitHub, ainsi que les versions pour Azure PowerShell :

* [Azure CLI][acr-scripts-cli]
* [Azure PowerShell][acr-scripts-psh]

## <a name="next-steps"></a>Étapes suivantes

Les articles suivants incluent des détails supplémentaires sur l’utilisation des principaux de service et ACR :

* [Authentification Azure Container Registry avec des principaux de service](container-registry-auth-service-principal.md)
* [S’authentifier auprès d’Azure Container Registry à partir d’Azure Kubernetes Service (AKS)](../aks/cluster-container-registry-integration.md)

<!-- IMAGES -->

<!-- LINKS - External -->
[acr-scripts-cli]: https://github.com/Azure/azure-docs-cli-python-samples/tree/master/container-registry
[acr-scripts-psh]: https://github.com/Azure/azure-docs-powershell-samples/tree/master/container-registry

<!-- LINKS - Internal -->
