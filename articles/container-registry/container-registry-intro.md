---
title: Registres de conteneurs managés
description: Présentation du service Azure Container Registry, proposant des registres Docker privés, gérés et basés sur le cloud.
author: stevelas
ms.topic: overview
ms.date: 02/10/2020
ms.author: stevelas
ms.custom: seodec18, mvc
ms.openlocfilehash: b5c81d8b2e6d7eac2dcd9070bf1f448340ea1a18
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96341263"
---
# <a name="introduction-to-private-docker-container-registries-in-azure"></a>Présentation des registres de conteneurs Docker privés dans Azure

Azure Container Registry est un service de registre Docker managé privé, qui est basé sur le registre open source Docker 2.0. Créez et tenez à jour des registres de conteneurs Azure pour stocker et gérer vos images conteneur Docker privées et les artefacts associés.

Utilisez des registres de conteneurs Azure avec vos pipelines de développement et de déploiement existants, ou utilisez Azure Container Registry Tasks pour créer des images conteneur dans Azure. Créez des builds à la demande ou des builds entièrement automatisées avec des déclencheurs tels que des validations du code source ou des mises à jour d’images de base.

Pour plus d’informations sur les concepts relatifs aux registres et à Docker, consultez [Présentation de Docker](https://docs.docker.com/engine/docker-overview/) et [À propos des registres, des référentiels et des images](container-registry-concepts.md).

## <a name="use-cases"></a>Cas d'utilisation

Extrayez des images à partir d’un registre de conteneur Azure pour différents objectifs de déploiement :

* Des **systèmes d’orchestration évolutifs** qui gèrent des applications en conteneur sur des clusters d’hôtes, y compris de [Kubernetes](https://kubernetes.io/docs/), [du contrôleur de domaine/système d’exploitation](https://docs.mesosphere.com/) et de [Docker Swarm](https://docs.docker.com/get-started/swarm-deploy/).
* Des **services Azure** qui prennent en charge la création et l’exécution des applications à grande échelle, y compris [Azure Kubernetes Service (AKS)](../aks/index.yml), [App Service](../app-service/index.yml), [Batch](../batch/index.yml), [Service Fabric](../service-fabric/index.yml) et d’autres services.

Les développeurs peuvent également effectuer un push vers un registre de conteneur dans le cadre d’un flux de travail de développement de conteneur. Par exemple, vous pouvez cibler un registre de conteneurs à partir d’un outil de livraison et d’intégration continue comme [Azure Pipelines](/azure/devops/pipelines/ecosystems/containers/acr-template) ou [Jenkins](https://jenkins.io/).

Configurez ACR Tasks pour recréer automatiquement des images d’application lorsque leurs images de base sont mises à jour, ou automatisez la génération d’images lorsque votre équipe valide le code dans un dépôt Git. Créez des tâches à plusieurs étapes pour automatiser la création, le test et la correction de plusieurs images conteneur en parallèle dans le cloud.

Azure fournit des outils, notamment l’interface de ligne de commande Azure, le portail Azure et la prise en charge des API, qui permettent de gérer vos registres de conteneurs Azure. Si vous le souhaitez, vous pouvez installer l’[extension Docker pour Visual Studio Code](https://code.visualstudio.com/docs/azure/docker) et l’extension [Compte Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account) pour utiliser vos registres de conteneurs Azure. Dans Visual Studio Code, vous pouvez tirer (pull) et envoyer (push) des images vers un registre de conteneurs Azure, et exécuter ACR Tasks.

## <a name="key-features"></a>Fonctionnalités clés

* **Niveaux de service des registres** : créez au moins un registre de conteneurs dans votre abonnement Azure. Les registres sont disponibles en trois niveaux : [De base, Standard et Premium](container-registry-skus.md), chacune prenant en charge l’intégration webhook, l’authentification de registres auprès d’Azure Active Directory et la fonctionnalité de suppression. Tirez parti du stockage local proche du réseau de vos images de conteneur en créant un registre dans le même emplacement Azure que vos déploiements. Utilisez la fonctionnalité [géoréplication](container-registry-geo-replication.md) des registres Premium pour les scénarios avancés de réplication et de distribution d’image conteneur. 

* **Sécurité et accès** : vous vous connectez à un registre à l’aide d’Azure CLI ou de la commande `docker login` standard. Azure Container Registry transfère les images conteneur via HTTPS et prend en charge TLS pour sécuriser les connexions clientes. 

  > [!IMPORTANT]
  > À partir du 13 janvier 2020, Azure Container Registry exigera l’utilisation de TLS 1.2 pour toutes les connexions sécurisées établies à partir des serveurs et des applications. Activez TLS 1.2 à l’aide de n’importe quel client docker récent (version 18.03.0 ou ultérieure). TLS 1.0 et 1.1 ne seront plus pris en charge. 

  Vous [contrôlez l’accès](container-registry-authentication.md) à un registre de conteneurs à l’aide d’une identité Azure, d’un [principal de service](../active-directory/develop/app-objects-and-service-principals.md) pris en charge par Azure Active Directory ou d’un compte d’administration fourni. Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour affecter des autorisations affinées au niveau utilisateurs ou systèmes à un registre.

  Les fonctionnalités de sécurité du niveau de service Premium incluent l’[approbation de contenu](container-registry-content-trust.md) pour la signature des étiquettes d’images, et les [pare-feu et réseaux virtuels (préversion)](container-registry-vnet.md) pour restreindre l’accès au registre. Azure Security Center s’intègre éventuellement avec Azure Container Registry pour [analyser les images](../security-center/defender-for-container-registries-introduction.md?bc=%2fazure%2fcontainer-registry%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fcontainer-registry%2ftoc.json) chaque fois qu’une image est envoyée (push) vers un registre.

* **Images et artefacts pris en charge** : placée dans un dépôt, chaque image est une capture instantanée en lecture seule d’un conteneur compatible avec Docker. Les registres de conteneur Azure peuvent inclure des images de Windows et Linux. Vous contrôlez les noms d’images pour tous les déploiements de votre conteneur. Utilisez des [commandes Docker](https://docs.docker.com/engine/reference/commandline/) standard pour envoyer les images dans un référentiel ou extraire une image d’un référentiel. En plus des images conteneurs Docker, Azure Container Registry stocke des [formats de contenu associés](container-registry-image-formats.md) comme les [graphiques Helm](container-registry-helm-repos.md) et les images générées selon la [spécification du format d’image Open Container Initiative (OCI)](https://github.com/opencontainers/image-spec/blob/master/spec.md).

* **Génération d’images automatisée** : utilisez [Azure Container Registry Tasks](container-registry-tasks-overview.md) (ACR Tasks) pour créer, tester, envoyer (push) et déployer plus simplement des images dans Azure. Par exemple, vous pouvez utiliser ACR Tasks pour étendre votre développement interne dans le cloud en déchargeant les opérations `docker build` dans Azure. Configurez des tâches de build pour automatiser le pipeline des mises à jour correctives (infrastructure et système d’exploitation du conteneur) et ainsi créer automatiquement des images lorsque votre équipe valide le code pour contrôler la source.

  Les [tâches multiétapes](container-registry-tasks-overview.md#multi-step-tasks) permettent la définition et l’exécution basées sur une tâche pour la génération, le test et la mise à jour corrective d’images conteneur dans le cloud. Les étapes de la tâche définissent les opérations build et push d’une image de conteneur individuelle. Elles permettent également de définir l’exécution d’un ou plusieurs conteneurs, en utilisant à chaque étape le conteneur comme son environnement d’exécution.

## <a name="next-steps"></a>Étapes suivantes

* [Créer un registre de conteneur à l’aide du portail Azure](container-registry-get-started-portal.md)
* [Créer un registre de conteneur à l’aide de l’interface de ligne de commande Azure](container-registry-get-started-azure-cli.md)
* [Automatiser la création et la maintenance des conteneurs avec ACR Tasks](container-registry-tasks-overview.md)