---
title: Créer un cluster Kubernetes avec Azure Dev Spaces activé – Azure Cloud Shell
services: azure-dev-spaces
ms.date: 10/04/2018
ms.topic: conceptual
description: Découvrez comment créer rapidement un cluster Kubernetes activé pour Azure Dev Spaces directement depuis votre navigateur, sans rien installer.
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs, Helm, service Mesh, routage du service Mesh, kubectl, k8s
ms.openlocfilehash: ce73f46a2451dfa87751d90f9cd6b31d5c80683f
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91963548"
---
# <a name="create-a-kubernetes-cluster-with-azure-dev-spaces-enabled-with-azure-cloud-shell"></a>Créer un cluster Kubernetes avec Azure Dev Spaces activé à l’aide d’Azure Cloud Shell

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

Vous pouvez utiliser [Azure Cloud Shell](/azure/cloud-shell) pour créer un cluster Azure Kubernetes Service via le bouton **Essayer** de cette page. Si vous n’êtes pas connecté, suivez les invites afin de vous connecter en utilisant un compte Azure, puis tapez les commandes à l’invite Azure Cloud Shell lorsqu’elle s’affiche.

## <a name="create-the-cluster"></a>Créer le cluster

Créez d’abord le groupe de ressources dans une [région qui prend en charge Azure Dev Spaces][supported-regions].

```azurecli-interactive
az group create --name MyResourceGroup --location <region>
```

Créez un cluster Kubernetes avec la commande suivante :

```azurecli-interactive
az aks create -g MyResourceGroup -n MyAKS --location <region> --generate-ssh-keys
```

La création du cluster ne prend que quelques minutes.  Une fois terminée, la sortie s’affiche au format JSON. Recherchez `provisioningState` et vérifiez qu’il s’agit bien de `Succeeded`.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des liens vers les didacticiels complets, consultez la section [Azure Dev Spaces](../index.yml).

> [!IMPORTANT]
> La plupart des didacticiels et guides de démarrage rapide pour Azure Dev Spaces utilisent l’interface CLI d’Azure Dev Spaces pour effectuer les opérations. Vous ne pouvez pas installer l’interface CLI d’Azure Dev Spaces dans Azure Cloud Shell.


[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
