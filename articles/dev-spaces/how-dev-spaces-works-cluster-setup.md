---
title: Comment configurer un cluster pour Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Décrit comment configurer un cluster Azure Kubernetes Service pour Azure Dev Spaces
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs
ms.openlocfilehash: 841e67b96e95aa251fa5bf1ef469b68de30f54d9
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91972677"
---
# <a name="how-setting-up-a-cluster-for-azure-dev-spaces-works"></a>Comment configurer un cluster pour Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Azure Dev Spaces vous offre plusieurs façons de parcourir et de déboguer rapidement des applications Kubernetes et de collaborer avec votre équipe sur un cluster Azure Kubernetes Service (AKS). Une méthode consiste à activer Azure Dev Spaces sur votre cluster AKS afin de pouvoir [exécuter des services directement sur votre cluster][how-it-works-up] et à utiliser des [fonctionnalités de routage et de mise en réseau supplémentaires][how-it-works-routing]. Cet article décrit ce qui se passe lorsque vous préparez votre cluster et activez Azure Dev Spaces.

## <a name="prepare-your-aks-cluster"></a>Préparer votre cluster AKS

Pour préparer votre cluster AKS pour Azure Dev Spaces, vérifiez que votre cluster AKS se trouve dans une région [prise en charge par Azure Dev Spaces][supported-regions] et que vous exécutez Kubernetes 1.10.3 ou une version ultérieure. Vous pouvez également activer Azure Dev Spaces à partir d’Azure CLI en exécutant `az aks use-dev-spaces`.

Quand Azure Dev Spaces est activé sur votre cluster AKS, il installe le contrôleur pour votre cluster. Le contrôleur réside à l’extérieur de votre cluster AKS. Il gère le comportement et la communication entre les outils côté client et le cluster AKS. Une fois le contrôleur activé, vous pouvez interagir avec lui à l’aide des outils côté client.

Le contrôleur effectue les actions suivantes :

* Gère la création et la sélection de l'espace de développement.
* Installe le graphique Helm de votre application et crée des objets Kubernetes.
* Génère l'image du conteneur de votre application.
* Déploie votre application sur AKS.
* Crée des builds incrémentales et redémarre lorsque votre code source change.
* Gère les journaux et les traces HTTP.
* Transfère stdout et stderr vers les outils côté client.
* Configure le routage pour les applications à l'intérieur d'un espace et entre les espaces parents et enfants.

Le contrôleur est une ressource Azure distincte à l'extérieur de votre cluster et qui effectue les actions suivantes sur les ressources de votre cluster :

* Crée ou désigne un espace de noms Kubernetes à utiliser comme espace de développement.
* Supprime tout espace de noms Kubernetes nommé *azds*, s'il existe, et en crée un nouveau.
* Déploie une configuration webhook Web Kubernetes.
* Déploie un serveur d’admission webhook.

Il utilise le même principal de service que votre cluster AKS pour passer des appels de service vers d’autres composants Azure Dev Spaces.

![Azure Dev Spaces prépare le cluster](media/how-dev-spaces-works/prepare-cluster.svg)

Pour utiliser Azure Dev Spaces, au moins un espace de développement est nécessaire. Azure Dev Spaces utilise les espaces de noms Kubernetes de votre cluster AKS comme espaces de développement. Lorsqu'un contrôleur est installé, il vous invite à créer un espace de noms Kubernetes ou à choisir un espace de noms existant à utiliser comme espace de développement initial. Le contrôleur propose de mettre à niveau l’espace de noms Kubernetes existant *par défaut* vers votre premier espace de développement.

Lorsqu'un espace de noms est désigné comme espace de développement, le contrôleur ajoute l’étiquette *azds.io/space=true* à cet espace de noms pour l'identifier comme espace de développement. Une fois que vous avez préparé votre cluster, l'espace de développement initial que vous créez ou désignez est sélectionné par défaut. Lorsqu'un espace est sélectionné, il est utilisé par Azure Dev Spaces pour créer des charges de travail.

Vous pouvez utiliser les outils côté client pour créer des espaces de développement et supprimer les espaces de développement existants. En raison d'une limitation dans Kubernetes, l'espace de développement *default* (par défaut) ne peut pas être supprimé. Le contrôleur supprime également tous les espaces de noms Kubernetes existants nommés *azds* pour éviter les conflits avec la commande `azds` utilisée par les outils côté client.

Le serveur d'admission webhook de Kubernetes sert à injecter des pods avec trois conteneurs pendant le déploiement pour l'instrumentation : un conteneur devspaces-proxy, un conteneur devspaces-proxy-init et un conteneur devspaces-build. **Ces trois conteneurs s'exécutent avec un accès root sur votre cluster AKS.** Ils utilisent également le même principal de service que votre cluster AKS pour passer des appels de service vers d'autres composants Azure Dev Spaces.

![Serveur d'admission webhook Azure Dev Spaces](media/how-dev-spaces-works/kubernetes-webhook-admission-server.svg)

Le conteneur devspaces-proxy est un conteneur sidecar qui gère tout le trafic TCP entrant et sortant du conteneur de l'application et en facilite le routage. Le conteneur devspaces-proxy redirige les messages HTTP si certains espaces sont utilisés. Par exemple, il peut faciliter le routage des messages HTTP entre les applications dans les espaces parents et enfants. Tout le trafic non-HTTP transite par devspaces-proxy, sans être modifié. Le conteneur devspaces-proxy enregistre également tous les messages HTTP entrants et sortants et les envoie aux outils côté client sous forme de traces. Ces traces peuvent ensuite être visualisées par le développeur pour étudier le comportement de l'application.

Le conteneur devspaces-proxy-init est un conteneur [init](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) qui ajoute des règles de routage supplémentaires basées sur la hiérarchie spatiale au conteneur de votre application. Il ajoute des règles de routage en mettant à jour le fichier */etc/resolv.conf* du conteneur de l’application et la configuration des tables d’adresses IP avant le démarrage. Les mises à jour appliquées à */etc/resolv.conf* permettent la résolution DNS des services dans les espaces parents. Les mises à jour de la configuration des tables d’adresses IP garantissent que tout le trafic TCP entrant et sortant du conteneur de l'application est acheminé via devspaces-proxy. Toutes les mises à jour provenant de devspaces-proxy-init viennent compléter les règles que Kubernetes ajoute.

Le conteneur devspaces-build est un conteneur init et contient le code source du projet ainsi que le socket Docker monté. Le code source du projet et l'accès à Docker permettent au conteneur de l’application d'être généré directement par le pod.

> [!NOTE]
> Azure Dev Spaces utilise le même nœud pour générer le conteneur de votre application et l'exécuter. Par conséquent, Azure Dev Spaces n'a pas besoin d'un registre de conteneurs externe pour générer et exécuter votre application.

Le serveur d'admission webhook Kubernetes écoute tout nouveau pod créé dans le cluster AKS. Si ce pod est déployé dans un espace de noms étiqueté *azds.io/space=true*, il injecte ce pod avec les conteneurs supplémentaires. Le conteneur devspaces-build n'est injecté que si le conteneur de l'application est exécuté avec les outils côté client.

Une fois que vous avez préparé votre cluster AKS, vous pouvez utiliser les outils côté client pour préparer et exécuter votre code dans votre espace de développement.

## <a name="client-side-tooling"></a>Outils côté client

Les outils côté client permettent à l’utilisateur d’effectuer les tâches suivantes :
* Générer un Dockerfile, un graphique Helm et un fichier de configuration Azure Dev Spaces pour l'application.
* Créer des espaces de développement parents et enfants.
* Demander au contrôleur de générer et de démarrer votre application.

Pendant que votre application est en cours d'exécution, les outils côté client effectuent aussi les tâches suivantes :
* Reçoivent et affichent stdout et stderr depuis votre application en cours d’exécution dans AKS.
* Utilisent [port-forward](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) pour autoriser l'accès web à votre application via http:\//localhost.
* Associe un débogueur à votre application en cours d'exécution dans AKS.
* Synchronisent le code source sur votre espace de développement lorsqu'un changement est détecté pour les builds incrémentielles, permettant ainsi une itération rapide.
* Vous permet de connecter votre ordinateur de développeur directement à votre cluster AKS.

Vous pouvez utiliser les outils côté client depuis la ligne de commande dans le cadre de la commande `azds`. Vous pouvez également utiliser les outils côté client avec :

* Visual Studio Code avec l'[extension Azure Dev Spaces](https://marketplace.visualstudio.com/items?itemName=azuredevspaces.azds).
* Visual Studio avec la charge de travail de développement Azure.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation des outils côté client pour préparer et exécuter votre code dans votre espace de développement, consultez [Préparation d’un projet pour Azure Dev Spaces][how-it-works-prep].


[how-it-works-prep]: how-dev-spaces-works-prep.md
[how-it-works-routing]: how-dev-spaces-works-routing.md
[how-it-works-up]: how-dev-spaces-works-up.md
[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service