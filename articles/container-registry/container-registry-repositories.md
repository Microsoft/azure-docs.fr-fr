---
title: Affichage des dépôts dans le portail
description: Utilisez le portail Azure pour voir les dépôts Azure Container Registry, qui hébergent des images conteneur Docker et d’autres artefacts pris en charge.
ms.topic: article
ms.date: 01/05/2018
ms.openlocfilehash: 1da72706d2554610a685f71199ab14af5e30ce1a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74456296"
---
# <a name="view-container-registry-repositories-in-the-azure-portal"></a>Affichage des référentiels Container Registry dans le portail Azure

Un référentiel Azure Container Registry vous permet de stocker des images de conteneur Docker. En stockant les images dans des référentiels, vous pouvez stocker des groupes d’images (ou une version de ces images) au sein d’environnements isolés. Vous pouvez spécifier ces référentiels lorsque vous envoyez des images à votre Registre via une transmission de type push, et afficher leur contenu dans le portail Azure.

## <a name="prerequisites"></a>Prérequis

* **Container Registry** (Registre de conteneurs) : créez un Registre de conteneurs dans votre abonnement Azure. Par exemple, utilisez le [portail Azure](container-registry-get-started-portal.md) ou [Azure CLI](container-registry-get-started-azure-cli.md).
* **Docker CLI** (Interface de ligne de commande Docker) : installez [Docker][docker-install] sur votre ordinateur local, ce qui vous fournit l’interface de ligne de commande Docker.
* **Container image (Image du conteneur)**  : envoyez une image vers votre registre de conteneurs. Pour savoir comment envoyer des images via une transmission de type push et en extraire, voir [Envoyer et extraire une image](container-registry-get-started-docker-cli.md).

## <a name="view-repositories-in-azure-portal"></a>Affichage des référentiels dans le portail Azure

Vous pouvez voir une liste des référentiels hébergeant vos images, ainsi que les balises d’image dans le portail Azure.

Si vous avez suivi les étapes de [Envoyer et extraire une image](container-registry-get-started-docker-cli.md) (et n’avez pas supprimé l’image par la suite), vous devez voir apparaître une image Nginx dans le Registre de conteneur. Les instructions de l’article spécifient que vous balisez l’image avec un espace de noms, les « exemples » dans `/samples/nginx`. Pour rappel, la commande [docker push][docker-push] spécifiée dans cet article était :

```Bash
docker push myregistry.azurecr.io/samples/nginx
```

 Azure Container Registry prenant en charge les espaces de noms à plusieurs niveaux, vous pouvez étendre des collections d’images liées à une application spécifique, ou une collection d’applications à des équipes opérationnelles ou de développement différentes. Pour en savoir plus sur les référentiels des Registres de conteneur, voir [Présentation des registres de conteneurs Docker privés](container-registry-intro.md).

Pour afficher un référentiel :

1. Connectez-vous au [portail Azure][portal]
1. Sélectionnez l’instance **Azure Container Registry** sur laquelle vous avez poussé l’image Nginx
1. Sélectionnez **Référentiels** pour afficher la liste des espaces de stockage qui contiennent les images dans le registre
1. Sélectionnez un référentiel pour consulter les balises d’image qui s’y trouvent

Par exemple, si vous avez poussé l’image Nginx, comme indiqué dans [Envoyer et extraire une image](container-registry-get-started-docker-cli.md), vous devriez voir quelque chose de similaire à :

![Affichage de référentiels dans le portail](./media/container-registry-repositories/container-registry-repositories.png)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous connaissez les principes fondamentaux de l’affichage et de l’utilisation des référentiels dans le portail, essayez d’utiliser Azure Container Registry avec un cluster [Azure Kubernetes Service (AKS)](../aks/tutorial-kubernetes-prepare-app.md).

<!-- LINKS - External -->
[docker-install]: https://docs.docker.com/engine/installation/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[portal]: https://portal.azure.com
