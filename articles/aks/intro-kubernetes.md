---
title: Présentation d’Azure Kubernetes Service
description: Découvrez les fonctionnalités et les avantages d’Azure Kubernetes Service pour déployer et gérer des applications en conteneur dans Azure.
services: container-service
ms.topic: overview
ms.date: 05/06/2019
ms.custom: mvc
ms.openlocfilehash: 75f4602be15ef2487272ed5790d92c4c884c551f
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94681548"
---
# <a name="azure-kubernetes-service-aks"></a>Azure Kubernetes Service (AKS)

Azure Kubernetes Service (AKS) simplifie le déploiement d’un cluster Kubernetes géré dans Azure. AKS permet de réduire la complexité et la surcharge opérationnelle de la gestion d’un cluster Kubernetes en déléguant une grande partie de cette responsabilité à Azure. En tant que service Kubernetes hébergé, Azure gère pour vous des tâches critiques telles que l’analyse de l'intégrité et la maintenance. Les maîtres Kubernetes sont gérés par Azure. Vous gérez uniquement les nœuds de l’agent. En tant que service Kubernetes managé, AKS est gratuit. Vous payez uniquement pour les nœuds de l’agent au sein de vos clusters, pas pour les maîtres.

Vous pouvez créer un cluster AKS dans le portail Azure, avec Azure CLI ou des options de déploiement piloté par modèle, comme les modèles Resource Manager et Terraform. Lors du déploiement d’un cluster AKS, le maître Kubernetes et tous les nœuds sont déployés et configurés pour vous. Des fonctionnalités supplémentaires telles que la mise en réseau avancée, l’intégration d’Azure Active Directory et la surveillance peuvent aussi être configurées lors du processus de déploiement. Les conteneurs Windows Server sont pris en charge dans AKS.

Pour plus d’informations sur les principes fondamentaux de Kubernetes, consultez [Concepts de base de Kubernetes pour AKS][concepts-clusters-workloads].

Pour commencer, effectuez le démarrage rapide d’AKS [dans le Portail Azure][aks-portal] ou [avec Azure CLI][aks-cli].

[!INCLUDE [azure-lighthouse-supported-service](../../includes/azure-lighthouse-supported-service.md)]

## <a name="access-security-and-monitoring"></a>Accès, sécurité et surveillance

Pour améliorer la sécurité et la gestion, vous pouvez intégrer AKS à Azure Active Directory et utiliser les contrôles d’accès en fonction du rôle (RBAC Kubernetes) de Kubernetes. Vous pouvez également superviser l’intégrité de votre cluster et de vos ressources.

### <a name="identity-and-security-management"></a>Gestion de la sécurité et identité

Pour limiter l’accès aux ressources de cluster, AKS prend en charge le [contrôle d’accès en fonction du rôle (RBAC Kubernetes) de Kubernetes][kubernetes-rbac]. Le contrôle RBAC Kubernetes vous permet de contrôler l’accès aux ressources et aux espaces de noms Kubernetes, ainsi que les autorisations sur ces ressources. Un cluster AKS peut aussi être configuré pour s’intégrer à Azure Active Directory (AD). Avec l’intégration Azure AD, l’accès à Kubernetes peut être configuré en fonction de l’identité et de l’appartenance du groupe existantes. Vos utilisateurs et groupes Azure AD existants peuvent disposer de l’accès aux ressources AKS et avec une expérience d’authentification intégrée.

Pour plus d’informations sur l’identité, consultez [Options d’accès et d’identité pour AKS][concepts-identity].

Pour sécuriser vos clusters AKS, consultez [Intégrer Azure Active Directory à AKS][aks-aad].

### <a name="integrated-logging-and-monitoring"></a>Fonctions de journalisation et de surveillance intégrées

Pour comprendre les performances de vos applications déployées et de votre cluster AKS, Azure Monitor pour l’intégrité des conteneurs collecte les métriques relatives au processeur et à la mémoire à partir des conteneurs, des nœuds et des contrôleurs. Les journaux du conteneur sont disponibles, et vous pouvez également [passer en revue les journaux Kubernetes][aks-master-logs]. Ces données de monitoring sont stockées dans un espace de travail Azure Log Analytics, et sont disponibles via le portail Azure, Azure CLI ou un point de terminaison REST.

Pour plus d’informations, consultez [Superviser l’intégrité des conteneurs Azure Kubernetes Service][container-health].

## <a name="clusters-and-nodes"></a>Clusters et nœuds

Les nœuds AKS s’exécutent sur des machines virtuelles Azure. Vous pouvez connecter le stockage à des nœuds et des pods, mettre à niveau les composants du cluster et utiliser des GPU. AKS prend en charge les clusters Kubernetes qui exécutent plusieurs pools de nœuds pour prendre en charge des systèmes d’exploitation mixtes et des conteneurs Windows Server. Les nœuds Linux exécutent une image de système d’exploitation Ubuntu personnalisée et les nœuds Windows Server exécutent une image de système d’exploitation Windows Server 2019 personnalisée.

### <a name="cluster-node-and-pod-scaling"></a>Nœud de cluster et la mise à l’échelle de pod

Lorsque la demande des ressources change, le nombre de pods ou de nœuds du cluster qui exécutent vos services peut automatiquement monter ou descendre en puissance. Vous pouvez utiliser l’autoscaler de pod horizontal ou l’autoscaler de cluster. Cette approche de la mise à l’échelle permet au cluster AKS de s’ajuster automatiquement aux demandes et d’exécuter uniquement les ressources nécessaires.

Pour plus d’informations, consultez [Mettre à l’échelle un cluster Azure Kubernetes Service (AKS)][aks-scale].

### <a name="cluster-node-upgrades"></a>Mises à niveau du nœud de cluster

Azure Kubernetes Service offre plusieurs versions de Kubernetes. Au fur et à mesure que de nouvelles versions sont disponibles dans AKS, votre cluster peut être mis à niveau via le portail Azure ou Azure CLI. Pendant le processus de mise à niveau, les nœuds sont soigneusement coordonnés et purgés afin de limiter les perturbations pour les applications en cours d’exécution.

Pour plus d’informations sur les versions du cycle de vie, consultez [Versions de Kubernetes prises en charge dans AKS][aks-supported versions]. Pour connaître les étapes de la mise à niveau, consultez [Mettre à niveau un cluster Azure Kubernetes Service (AKS)][aks-upgrade].

### <a name="gpu-enabled-nodes"></a>Nœuds avec processeur graphique (GPU)

AKS prend en charge la création de pools de nœuds avec processeur graphique (GPU). Azure fournit actuellement une ou plusieurs machines virtuelles avec processeur graphique (GPU). Les machines virtuelles avec processeur graphique (GPU) sont conçues pour des charges de travail de visualisation, mais également de calcul et d’affichage graphique intensifs.

Pour plus d’informations, consultez [Utilisation des GPU sur AKS][aks-gpu].

### <a name="confidential-computing-nodes-public-preview"></a>Nœuds d’informatique confidentielle (préversion publique)

AKS prend en charge la création de pools de nœuds d’informatique confidentielle basés sur Intel SGX (machines virtuelles DCSv2). Les nœuds d’informatique confidentielle permettent aux conteneurs de s’exécuter dans un environnement d’exécution isolé et approuvé basé sur le matériel. L’isolement entre des conteneurs combinés avec l’intégrité du code par le biais d’une attestation peut être utile pour votre stratégie de sécurité de conteneur de défense en profondeur. Les nœuds d’informatique confidentielle prennent en charge à la fois les conteneurs confidentiels (applications Docker existantes) et les conteneurs reconnaissant les enclaves.

Pour plus d’informations, consultez [Nœuds d’informatique confidentielle sur AKS][conf-com-node].

### <a name="storage-volume-support"></a>Prise en charge du volume de stockage

Pour supporter les charges de travail d’application, vous pouvez monter des volumes de stockage pour les données persistantes. Les volumes statiques et dynamiques peuvent être utilisés. En fonction du nombre de pods connectés qui doivent partager le stockage, vous pouvez utiliser un stockage soutenu par Azure Disks pour l’accès d’un pod unique ou Azure Files pour l’accès simultané de plusieurs pods.

Pour plus d’informations, consultez [Options de stockage pour les applications dans AKS][concepts-storage].

Démarrez avec les volumes persistants dynamiques en utilisant [Disques Azure][azure-disk] ou [Azure Files][azure-files].

## <a name="virtual-networks-and-ingress"></a>Réseaux virtuels et entrée

Un cluster AKS peut être déployé dans un réseau virtuel existant. Dans cette configuration, chaque pod du cluster se voit affecter une adresse IP dans le réseau virtuel et peut communiquer directement avec les autres pods du cluster et les autres nœuds du réseau virtuel. Les pods peuvent se connecter à d’autres services dans un réseau virtuel appairé, et à d’autres réseaux locaux via des connexions ExpressRoute ou VPN site à site (S2S).

Pour plus d’informations, consultez les [Concepts réseau pour les applications dans AKS][aks-networking].

Pour vous familiariser avec le trafic d’entrée, consultez [Routage d’applications HTTP][aks-http-routing].

### <a name="ingress-with-http-application-routing"></a>Entrée avec un routage d’applications HTTP

Le module complémentaire de routage des applications HTTP permet d’accéder facilement aux applications déployées sur votre cluster AKS. Lorsqu’elle est activée, la solution de routage d’application HTTP configure un contrôleur d’entrée de votre cluster AKS. Lorsque les applications sont déployées, les noms de DNS accessibles publiquement sont configurés automatiquement. Le routage d’applications HTTP configure une zone DNS et l’intègre au cluster AKS. Vous pouvez ensuite déployer des ressources d’entrée Kubernetes comme d’habitude.

Pour vous familiariser avec le trafic d’entrée, consultez [Routage d’applications HTTP][aks-http-routing].

## <a name="development-tooling-integration"></a>Intégration d’outils de développement

Kubernetes dispose d’un écosystème riche en outils de gestion et de développement, dont Helm et l’extension Kubernetes pour Visual Studio Code. Ces outils fonctionnent parfaitement avec AKS.

De plus, Azure Dev Spaces offre aux équipes une expérience de développement Kubernetes rapide et itérative. Avec une configuration minimale, vous pouvez exécuter et déboguer des conteneurs directement dans AKS. Pour commencer, consultez [Azure Dev Spaces][azure-dev-spaces].

Le projet Azure DevOps fournit une solution simple pour apporter le code et le référentiel Git existant dans Azure. Le projet DevOps crée automatiquement des ressources Azure telles qu’AKS, un pipeline de mise en production dans Azure DevOps Services comprenant un pipeline de build pour CI (intégration continue), configure un pipeline de mise en production pour CD (déploiement continu), puis crée une ressource Azure Application Insights pour la surveillance.

Pour plus d’informations, consultez [Projet Azure DevOps][azure-devops].

## <a name="docker-image-support-and-private-container-registry"></a>Prise en charge des images Docker et Registre de conteneurs privé

AKS prend en charge le format d’image Docker. Vous pouvez intégrer AKS à Azure Container Registry (ACR) pour stocker vos images Docker en privé.

Pour créer un magasin d’images privé, consultez [Azure Container Registry][acr-docs].

## <a name="kubernetes-certification"></a>Certification Kubernetes

Azure Kubernetes Service (AKS) a été certifié conforme à Kubernetes par CNCF.

## <a name="regulatory-compliance"></a>Conformité aux normes

Azure Kubernetes Service (AKS) est conforme aux normes SOC, ISO, PCI DSS et HIPAA. Pour plus d’informations, consultez [Présentation de la conformité Microsoft Azure][compliance-doc].

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur le déploiement et la gestion d’AKS avec le démarrage rapide Azure CLI.

> [!div class="nextstepaction"]
> [Guide de démarrage rapide AKS][aks-cli]

<!-- LINKS - external -->
[aks-engine]: https://github.com/Azure/aks-engine
[kubectl-overview]: https://kubernetes.io/docs/user-guide/kubectl-overview/
[compliance-doc]: https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942

<!-- LINKS - internal -->
[acr-docs]: ../container-registry/container-registry-intro.md
[aks-aad]: ./azure-ad-integration-cli.md
[aks-cli]: ./kubernetes-walkthrough.md
[aks-gpu]: ./gpu-cluster.md
[aks-http-routing]: ./http-application-routing.md
[aks-networking]: ./concepts-network.md
[aks-portal]: ./kubernetes-walkthrough-portal.md
[aks-scale]: ./tutorial-kubernetes-scale.md
[aks-upgrade]: ./upgrade-cluster.md
[azure-dev-spaces]: ../dev-spaces/index.yml
[azure-devops]: ../devops-project/overview.md
[azure-disk]: ./azure-disks-dynamic-pv.md
[azure-files]: ./azure-files-dynamic-pv.md
[container-health]: ../azure-monitor/insights/container-insights-overview.md
[aks-master-logs]: view-master-logs.md
[aks-supported versions]: supported-kubernetes-versions.md
[concepts-clusters-workloads]: concepts-clusters-workloads.md
[kubernetes-rbac]: concepts-identity.md#kubernetes-role-based-access-control-kubernetes-rbac
[concepts-identity]: concepts-identity.md
[concepts-storage]: concepts-storage.md
[conf-com-node]: ../confidential-computing/confidential-nodes-aks-overview.md
