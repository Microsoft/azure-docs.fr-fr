---
title: Activer Azure Dev Spaces sur AKS et installer les outils côté client
services: azure-dev-spaces
ms.date: 07/24/2019
ms.topic: conceptual
description: Découvrez comment activer Azure Dev Spaces sur un cluster AKS et installer les outils côté client.
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs, Helm, service Mesh, routage du service Mesh, kubectl, k8s
ms.openlocfilehash: 8ebfd8de59ae096e6cee87509bc19e0d161fcaa7
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91972983"
---
# <a name="enable-azure-dev-spaces-on-an-aks-cluster-and-install-the-client-side-tools"></a>Activer Azure Dev Spaces sur un cluster AKS et installer les outils côté client

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

Cet article montre plusieurs manières d’activer Azure Dev Spaces sur un cluster AKS et comment installer les outils côté client.

## <a name="enable-azure-dev-spaces-using-the-cli"></a>Activer Azure Dev Spaces à l’aide de l’interface CLI

Pour activer Dev Spaces à l’aide de l’interface CLI, vous avez besoin des éléments suivants :
* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte gratuit][az-portal-create-account].
* [Azure CLI installé][install-cli]
* [Un cluster AKS][create-aks-cli] dans une [région prise en charge][supported-regions]

Utilisez la commande `use-dev-spaces` pour activer Dev Spaces sur votre cluster AKS et suivez les invites.

```azurecli
az aks use-dev-spaces -g myResourceGroup -n myAKSCluster
```

La commande ci-dessus active Dev Spaces sur le cluster *myAKSCluster* dans le groupe *myResourceGroup* et crée un espace de développement *par défaut*.

```console
'An Azure Dev Spaces Controller' will be created that targets resource 'myAKSCluster' in resource group 'myResourceGroup'. Continue? (y/N): y

Creating and selecting Azure Dev Spaces Controller 'myAKSCluster' in resource group 'myResourceGroup' that targets resource 'myAKSCluster' in resource group 'myResourceGroup'...2m 24s

Select a dev space or Kubernetes namespace to use as a dev space.
 [1] default
Type a number or a new name: 1

Kubernetes namespace 'default' will be configured as a dev space. This will enable Azure Dev Spaces instrumentation for new workloads in the namespace. Continue? (Y/n): Y

Configuring and selecting dev space 'default'...3s

Managed Kubernetes cluster 'myAKSCluster' in resource group 'myResourceGroup' is ready for development in dev space 'default'. Type `azds prep` to prepare a source directory for use with Azure Dev Spaces and `azds up` to run.
```

La commande `use-dev-spaces` installe également l’interface CLI Azure Dev Spaces.

## <a name="install-the-client-side-tools"></a>Installer les outils côté client

Vous pouvez utiliser les outils côté client Azure Dev Spaces pour interagir avec des espaces de développement sur un cluster AKS à partir de votre ordinateur local. Il existe plusieurs façons d’installer les outils côté client :

* Dans [Visual Studio Code][vscode], installez l’[extension Azure Dev Spaces][vscode-extension].
* Dans [Visual Studio 2019][visual-studio], installez la charge de travail Développement Azure.
* Téléchargez et installez l’interface CLI [Windows][cli-win], [Mac][cli-mac] ou [Linux][cli-linux]

## <a name="remove-azure-dev-spaces-using-the-cli"></a>Supprimer Azure Dev Spaces à l’aide de l’interface CLI

Pour supprimer Azure Dev Spaces de votre cluster AKS, utilisez la commande `azds remove`.

```azurecli
azds remove -g MyResourceGroup -n MyAKS
```

L’exemple de sortie ci-dessous montre la suppression d’Azure Dev Spaces du cluster *MyAKS*.

```azurecli
$ azds remove -g MyResourceGroup -n MyAKS
Azure Dev Spaces Controller 'MyAKS' in resource group 'MyResourceGroup' that targets resource 'MyAKS' in resource group 'MyResourceGroup' will be deleted. This will remove Azure Dev Spaces instrumentation from the target resource for new workloads. Continue? (y/N): y

Deleting Azure Dev Spaces Controller 'MyAKS' in resource group 'MyResourceGroup' that targets resource 'MyAks' in resource group 'MyResourceGroup' (takes a few minutes)...
```

Les espaces de noms que vous avez créés avec Azure Dev Spaces demeurent, de même que leurs charges de travail, mais les nouvelles charges de travail de ces espaces de noms ne sont pas instrumentées avec Azure Dev Spaces. En outre, si vous redémarrez des pods existants instrumentés avec Azure Dev Spaces, vous risquez de voir des erreurs. Ces pods doivent être redéployés sans les outils Azure Dev Spaces. Pour supprimer complètement Azure Dev Spaces de votre cluster, supprimez tous les pods de tous les espaces de noms où Azure Dev Spaces a été activé.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage plus sur le fonctionnement d’Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Fonctionnement d’Azure Dev Spaces](../how-dev-spaces-works.md)

[create-aks-cli]: ../../aks/kubernetes-walkthrough.md#create-a-resource-group
[install-cli]: /cli/azure/install-azure-cli?view=azure-cli-latest
[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[az-portal]: https://portal.azure.com
[az-portal-create-account]: https://azure.microsoft.com/free
[cli-linux]: https://aka.ms/get-azds-linux
[cli-mac]: https://aka.ms/get-azds-mac
[cli-win]: https://aka.ms/get-azds-windows
[visual-studio]: https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs
[visual-studio-k8s-tools]: https://aka.ms/get-vsk8stools
[vscode]: https://code.visualstudio.com/download
[vscode-extension]: https://marketplace.visualstudio.com/items?itemName=azuredevspaces.azds
