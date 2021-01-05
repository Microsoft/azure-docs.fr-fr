---
title: Concepts de base de Kubernetes pour AKS (Azure Kubernetes Service)
description: Découvrez les composants de charge de travail et de cluster de base de Kubernetes et leur lien avec les fonctionnalités d’AKS (Azure Kubernetes Service)
services: container-service
ms.topic: conceptual
ms.date: 06/03/2019
ms.openlocfilehash: 17203123ceb0c196bd8f9011e2962f5022e54698
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92901298"
---
# <a name="kubernetes-core-concepts-for-azure-kubernetes-service-aks"></a>Concepts de base de Kubernetes pour AKS (Azure Kubernetes Service)

Le développement d’applications adoptant progressivement une approche basée sur les conteneurs, il est important et nécessaire d’orchestrer et de gérer les ressources. Kubernetes est la plateforme leader du marché qui offre la possibilité de fournir une planification fiable des charges de travail d’applications à tolérance de panne. AKS (Azure Kubernetes Service) est une offre Kubernetes managés qui simplifie la gestion et le déploiement des applications basées sur des conteneurs.

Cet article présente les principaux composants de l’infrastructure Kubernetes, tels que le *plan de contrôle* , les *nœuds* et les *pools de nœuds* . Les ressources de charge de travail telles que les *pods* , les *déploiements* et les *ensembles* sont également présentées, ainsi que le regroupement de ressources dans des *espaces de noms* .

## <a name="what-is-kubernetes"></a>Présentation de Kubernetes

Kubernetes est une plateforme évoluant rapidement qui gère les applications basées sur les conteneurs, ainsi que leurs composants de mise en réseau et de stockage. L’accent est mis sur les charges de travail d’applications, pas sur les composants de l’infrastructure sous-jacente. Kubernetes fournit une approche déclarative des déploiements, assortie d’un ensemble robuste d’API pour les opérations de gestion.

Vous pouvez générer et exécuter des applications modernes, portables et basées sur des microservices qui bénéficient de l’orchestration et de la gestion par Kubernetes de la disponibilité des composants d’application. Kubernetes prend en charge les applications sans état et avec état à mesure que les équipes adoptent les applications basées sur des microservices.

En tant que plateforme ouverte, Kubernetes vous permet de créer des applications avec vos langages de programmation, système d’exploitation, bibliothèques ou bus de messagerie préférés. Les outils d’intégration et de livraison continues (CI/CD) existants peuvent s’intégrer à Kubernetes dans le cadre de la planification et du déploiement de versions.

AKS (Azure Kubernetes Service) fournit un service Kubernetes managé qui réduit la complexité des tâches de gestion principales et de déploiement, y compris la coordination des mises à niveau. Le plan de contrôle AKS est géré par la plateforme Azure ; vous ne payez que pour les nœuds AKS qui exécutent vos applications. AKS repose sur le moteur open source Azure Kubernetes Service ([aks-engine][aks-engine]).

## <a name="kubernetes-cluster-architecture"></a>Architecture d’un cluster Kubernetes

Un cluster Kubernetes comprend deux composants :

- Les nœuds de *plan de contrôle* fournissent les services Kubernetes de base et l’orchestration des charges de travail d’applications.
- Les *nœuds* exécutent vos charges de travail d’applications.

![Plan de contrôle et composants de nœud Kubernetes](media/concepts-clusters-workloads/control-plane-and-nodes.png)

## <a name="control-plane"></a>Plan de contrôle

Quand vous créez un cluster AKS, un plan de contrôle est automatiquement créé et configuré. Ce plan de contrôle est fourni en tant que ressource Azure managée tirée de l’utilisateur. Il n’existe aucun coût lié au plan de contrôler ; seuls les nœuds qui font partie du cluster AKS occasionnent des frais. Le plan de contrôle et ses ressources se trouvent uniquement sur la région dans laquelle le cluster a été créé.

Le plan de contrôle inclut les composants Kubernetes principaux suivants :

- *kube-apiserver*  : le serveur d’API détermine la façon dont les API Kubernetes sous-jacentes sont exposées. Ce composant fournit l’interaction des outils de gestion, tels que `kubectl` ou le tableau de bord Kubernetes.
- *etcd*  : pour maintenir l’état de la configuration et du cluster Kubernetes, le composant *etcd* hautement disponible est un magasin de valeurs essentiel dans Kubernetes.
- *kube-scheduler*  : quand vous créez ou mettez à l’échelle des applications, le planificateur détermine les nœuds pouvant exécuter la charge de travail et les démarre.
- *kube-controller-manager*  : le gestionnaire de contrôleurs surveille une série de contrôleurs plus petits qui effectuent des actions telles que la réplication des pods et la gestion des opérations sur les nœuds.

AKS fournit un plan de contrôle monolocataire doté de dispositifs dédiés (serveur d’API, Scheduler, etc.). Vous définissez le nombre et la taille des nœuds, puis la plateforme Azure configure la communication sécurisée entre les nœuds et le plan de contrôle. L’interaction avec le plan de contrôle se produit par le biais d’API Kubernetes, telles que `kubectl` ou le tableau de bord Kubernetes.

Ce plan de contrôle managé évite d’avoir à configurer des composants tels qu’un magasin *etcd* hautement disponible, mais n’est pas accessible directement. Les mises à niveau de Kubernetes sont orchestrées par l’intermédiaire de l’interface de ligne de commande Azure ou du Portail Azure, qui met à niveau le plan de contrôle, puis les nœuds. Pour résoudre les problèmes éventuels, vous pouvez consulter les journaux d’activité du plan de contrôle par le biais des journaux d’activité Azure Monitor.

Si vous devez configurer le plan de contrôle d’une façon particulière ou avez besoin d’un accès direct à ce dernier, vous pouvez déployer votre propre cluster Kubernetes à l’aide d’[aks-engine][aks-engine].

Pour connaître les meilleures pratiques associées, consultez [Meilleures pratiques relatives aux mises à jour et à la sécurité du cluster dans AKS][operator-best-practices-cluster-security].

## <a name="nodes-and-node-pools"></a>Nœuds et pools de nœuds

Pour exécuter vos applications et les services de prise en charge, vous avez besoin d’un *nœud* Kubernetes. Un cluster AKS a un ou plusieurs nœuds, qui sont des machines virtuelles exécutant les composants des nœuds Kubernetes et le runtime de conteneur :

- `kubelet` est l’agent Kubernetes qui traite les requêtes d’orchestration du plan de contrôle et la planification de l’exécution des conteneurs demandés.
- La mise en réseau virtuelle est gérée par le *kube-proxy* sur chaque nœud. Le proxy route le trafic réseau et gère l’adressage IP pour les services et les pods.
- Le *runtime de conteneur* est le composant qui permet aux applications en conteneur de s’exécuter et d’interagir avec d’autres ressources telles que le réseau virtuel et le stockage. Dans AKS, Moby est utilisé en tant que runtime de conteneur.

![Ressources des machines virtuelles Azure et de prise en charge pour un nœud Kubernetes](media/concepts-clusters-workloads/aks-node-resource-interactions.png)

La taille des machines virtuelles Azure pour vos nœuds détermine le nombre d’UC, la quantité de mémoire, ainsi que la taille et le type de stockage disponible (par exemple, SSD hautes performances ou HDD classique). Si vous pensez avoir un jour besoin d’applications nécessitant une grande quantité d’UC et de mémoire ou un stockage hautes performances, planifiez la taille des nœuds en conséquence. Vous pouvez également augmenter le nombre de nœuds dans votre cluster AKS afin de répondre à la demande.

Dans AKS, l’image de machine virtuelle pour les nœuds de votre cluster est actuellement basée sur Ubuntu Linux ou Windows Server 2019. Quand vous créez un cluster AKS ou augmentez le nombre de nœuds, la plateforme Azure crée le nombre demandé de machines virtuelles et les configure. Vous n’avez aucune configuration manuelle à effectuer. Les nœuds agent étant facturés en tant que machines virtuelles standard, les remises dont vous bénéficiez sur la taille de machine virtuelle que vous utilisez (y compris les [réservations Azure][reservation-discounts]) sont automatiquement appliquées.

Si vous avez besoin d’utiliser un autre système d’exploitation hôte ou runtime de conteneur, ou bien d’inclure des packages personnalisés, vous pouvez déployer votre propre cluster Kubernetes à l’aide [d’aks-engine][aks-engine]. En amont, `aks-engine` assure la mise en production des fonctionnalités et fournit les options de configuration avant qu'elles ne soient officiellement prises en charge dans les clusters AKS. Par exemple, si vous souhaitez utiliser un runtime de conteneur autre que Moby, vous pouvez utiliser `aks-engine` pour configurer et déployer un cluster Kubernetes qui répond à vos besoins actuels.

### <a name="resource-reservations"></a>Réservations de ressources

Les ressources de nœud sont utilisées par AKS pour faire fonctionner le nœud dans le cadre de votre cluster. Cette utilisation peut créer un écart entre les ressources totales du nœud et les ressources allouables quand elles sont utilisées dans AKS. Il est important de noter cette information au moment de définir les demandes et les limites des pods déployés par l’utilisateur.

Pour rechercher les ressources allouables d’un nœud, exécutez :
```kubectl
kubectl describe node [NODE_NAME]

```

Pour conserver les fonctionnalités et les performances des nœuds, les ressources sont réservées sur chaque nœud par AKS. Dans la mesure où un nœud gagne en taille dans les ressources, la réservation de ressources augmente en raison d’une plus grande quantité de pods déployés par l’utilisateur nécessitant une gestion.

>[!NOTE]
> L’utilisation de modules complémentaires AKS tels que Container Insights (OMS) nécessite des ressources de nœud supplémentaires.

- **Processeur**  : le processeur réservé dépend du type de nœud et de la configuration du cluster, ce qui peut le rendre moins allouable en raison de l’exécution de fonctionnalités supplémentaires.

| Cœurs de processeur sur l’hôte | 1    | 2    | 4    | 8    | 16 | 32|64|
|---|---|---|---|---|---|---|---|
|Réservés par Kube (millicores)|60|100|140|180|260|420|740|

- **Mémoire** – la mémoire utilisée par AKS comprend la somme de deux valeurs.

1. Le démon kubelet est installé sur tous les nœuds de l’agent Kubernetes pour gérer la création et l’arrêt du conteneur. Par défaut sur AKS, ce démon a la règle d’éviction suivante : *memory.available<750Mi* , ce qui signifie qu’un nœud doit toujours avoir au moins 750 Mi allouable à tout moment.  Lorsqu’un hôte se trouve au-dessous de ce seuil de mémoire disponible, kubelet met fin à l’un des pods en cours d’exécution pour libérer de la mémoire sur l’ordinateur hôte et le protéger. Cette action se déclenche lorsque la mémoire disponible descend au-dessous du seuil de 750 Mi.

2. La deuxième valeur est une vitesse régressive des réservations de la mémoire pour que le démon kubelet fonctionne correctement (kube-reserved).
    - 25 % des 4 premiers Go de mémoire
    - 20 % des 4 Go suivants de mémoire (jusqu’à 8 Go)
    - 10 % des 8 Go suivants de mémoire (jusqu’à 16 Go)
    - 6 % des 112 Go suivants de mémoire (jusqu’à 128 Go)
    - 2 % de la mémoire au-dessus de 128 Go

Les règles ci-dessus relatives à l’allocation de mémoire et d’UC sont utilisées pour assurer l’intégrité des nœuds de l’agent, dont certains pods de système d’hébergement critiques pour l’intégrité du cluster. Du fait de ces règles d’allocation, le nœud signale moins de mémoire et de processeurs allouables que s’il ne faisait pas partie d’un cluster Kubernetes. Vous ne pouvez pas changer les réservations de ressources ci-dessus.

Par exemple, si un nœud offre 7 Go, il signalera 34 % de la mémoire non-allouable incluant le seuil d’éviction dur de 750Mi.

`0.75 + (0.25*4) + (0.20*3) = 0.75GB + 1GB + 0.6GB = 2.35GB / 7GB = 33.57% réservé`

En plus des réservations pour Kubernetes lui-même, le système d’exploitation du nœud sous-jacent réserve également une quantité de ressources de processeur et de mémoire pour gérer les fonctions du système d’exploitation.

Pour connaître les meilleures pratiques associées, consultez la section [Meilleures pratiques relatives aux fonctionnalités de base du planificateur dans AKS][operator-best-practices-scheduler].

### <a name="node-pools"></a>Pools de nœuds

Les nœuds d’une même configuration sont regroupés dans des *pools de nœuds* . Un cluster Kubernetes contient un ou plusieurs pools de nœuds. Le nombre et la taille initiaux des nœuds sont définis quand vous créez un cluster AKS, opération qui engendre la création d’un *nœud de pools par défaut* . Ce pool de nœuds par défaut dans AKS contient les machines virtuelles sous-jacentes qui exécutent vos nœuds d’agent.

> [!NOTE]
> Pour garantir un fonctionnement fiable de votre cluster, vous devez exécuter au moins 2 (deux) nœuds dans le pool de nœuds par défaut.

Quand vous mettez à l’échelle ou à niveau un cluster AKS, l’action est effectuée sur le pool de nœuds par défaut. Vous pouvez aussi choisir de mettre à l’échelle ou de mettre à niveau un pool de nœuds spécifique. Pour les opérations de mise à niveau, les conteneurs en cours d’exécution sont planifiés sur d’autres nœuds du pool de nœuds jusqu’à ce que tous les nœuds soient mis à niveau.

Pour en savoir plus sur l’utilisation de plusieurs pools de nœuds dans AKS, voir [Créer et gérer plusieurs pools de nœuds pour un cluster dans AKS][use-multiple-node-pools].

### <a name="node-selectors"></a>Sélecteurs de nœud

Dans un cluster AKS qui contient plusieurs pools de nœuds, vous devrez peut-être indiquer au planificateur Kubernetes Scheduler le pool de nœuds qui devra être utilisé pour une ressource donnée. Par exemple, les contrôleurs d’entrée ne doivent pas s’exécuter sur des nœuds Windows Server. Les sélecteurs de nœud vous permettent de définir différents paramètres, comme le système d’exploitation des nœuds, pour contrôler à quel endroit un pod doit être planifié.

L’exemple de base suivant planifie une instance NGINX sur un nœud Linux en utilisant le sélecteur de nœud *"beta.kubernetes.io/os": linux*  :

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx
spec:
  containers:
    - name: myfrontend
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.12-alpine
  nodeSelector:
    "beta.kubernetes.io/os": linux
```

Pour plus d’informations sur la façon de contrôler l’endroit où sont planifiés les pods, consultez la section [Meilleures pratiques relatives aux fonctionnalités avancées du planificateur dans AKS][operator-best-practices-advanced-scheduler].

## <a name="pods"></a>Pods

Kubernetes Utilise des *pods* pour exécuter une instance de votre application. Un pod représente une instance unique de votre application. Les pods ont généralement un mappage 1 à 1 avec un conteneur, bien qu’il existe des scénarios avancés où un pod peut contenir plusieurs conteneurs. Ces pods multiconteneurs sont planifiés ensemble sur le même nœud et permettent aux conteneurs de partager des ressources connexes.

Quand vous créez un pod, vous pouvez définir des *demandes de ressources* afin de demander une certaine quantité de ressources de processeur ou de mémoire. Le planificateur de Kubernetes essaie de planifier les pods afin qu’ils s’exécutent sur un nœud dont les ressources permettent de répondre à la demande. Vous pouvez également spécifier des limites de ressources maximales qui empêchent un pod donné de consommer trop de ressources de calcul à partir du nœud sous-jacent. Une bonne pratique consiste à inclure des limites de ressources pour tous les pods afin d'aider le Scheduler Kubernetes à identifier les ressources nécessaires et autorisées.

Pour plus d’informations, consultez [Kubernetes pods][kubernetes-pods] (Pods Kubernetes) et [Kubernetes pod lifecycle][kubernetes-pod-lifecycle] (cycle de vie des pods Kubernetes).

Un pod est une ressource logique, tandis que les conteneurs sont l’endroit où s’exécutent les charges de travail d’applications. Les pods sont en général des ressources éphémères jetables, et ceux planifiés individuellement ne bénéficient pas de toutes les fonctionnalités de haute disponibilité et de redondance fournies par Kubernetes. En effet, les pods sont déployés et gérés par des *contrôleurs* Kubernetes, comme le contrôleur de déploiement.

## <a name="deployments-and-yaml-manifests"></a>Déploiements et manifestes YAML

Un *déploiement* représente un ou plusieurs pods identiques, gérés par le contrôleur de déploiement Kubernetes. Un déploiement définit le nombre de *réplicas* (pods) à créer, tandis que le planificateur de Kubernetes veille à ce que des pods supplémentaires soient planifiés sur des nœuds sains si les pods ou les nœuds rencontrent des problèmes.

Vous pouvez mettre à jour les déploiements pour changer la configuration des pods, l’image de conteneur utilisée ou le stockage attaché. Le contrôleur de déploiement draine et met fin à un certain nombre de réplicas, crée des réplicas à partir de la nouvelle définition de déploiement et poursuit le processus jusqu’à ce que tous les réplicas dans le déploiement soient mis à jour.

La plupart des applications sans état dans AKS doivent utiliser le modèle de déploiement plutôt que la planification de pods individuels. Kubernetes peut superviser l’intégrité et l’état des déploiements pour s’assurer que le nombre requis de réplicas s’exécutent dans le cluster. Quand vous planifiez uniquement des pods individuels, ces derniers ne sont pas redémarrés s’ils rencontrent un problème et ne sont pas replanifiés sur des nœuds sains si leur nœud actuel rencontre un problème.

Si une application requiert qu’un quorum d’instances soit toujours disponible pour les prises de décisions de gestion, il convient qu’aucun processus de mise à jour ne rompe ce dispositif. Vous pouvez utiliser des *budgets d’interruption de pods* pour définir le nombre de réplicas dans un déploiement pouvant être retirés pendant une mise à niveau d’un nœud ou une mise à jour. Par exemple, si votre déploiement comprend *cinq (5)*  réplicas, vous pouvez définir une interruption de pods de *4* pour limiter la suppression ou la replanification autorisée à un seul réplica à la fois. Comme dans le cas des limites de ressources des pods, une bonne pratique consiste à définir des budgets d’interruption de pods sur les applications qui nécessitent la présence systématique d’un nombre minimal de réplicas.

Les déploiements sont généralement créés et gérés avec `kubectl create` ou `kubectl apply`. Pour créer un déploiement, vous définissez un fichier manifeste dans le format YAML (YAML Ain't Markup Language). L’exemple suivant crée un déploiement de base du serveur web NGINX. Le déploiement spécifie la création de *trois (3)*  réplicas ; le port  *80* doit être ouvert sur le conteneur. Des demandes et limites de ressources sont également définies pour l’UC et la mémoire.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: mcr.microsoft.com/oss/nginx/nginx:1.15.2-alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 250m
            memory: 64Mi
          limits:
            cpu: 500m
            memory: 256Mi
```

Vous pouvez également créer des applications plus complexes en incluant des services tels que des équilibreurs de charge dans le manifeste YAML.

Pour plus d’informations, consultez la section [Déploiements Kubernetes][kubernetes-deployments].

### <a name="package-management-with-helm"></a>Gestion des packages avec Helm

Dans Kubernetes, la gestion des applications fait souvent appel à [Helm][helm]. Vous pouvez créer et utiliser des *graphiques* Helm (charts) publics existants qui contiennent une version empaquetée d’un code d’application et de manifestes YAML Kubernetes pour le déploiement de ressources. Ces graphiques Helm peuvent être stockés localement, ou souvent dans un référentiel distant, tel qu’un [référentiel de graphiques Helm Azure Container Registry][acr-helm].

Pour utiliser Helm, installez le client Helm sur votre ordinateur, ou utilisez le client Helm dans [Azure Cloud Shell][azure-cloud-shell]. Vous pouvez rechercher ou créer des graphiques Helm avec le client, puis les installer sur votre cluster Kubernetes. Pour plus d’informations, consultez la section [Installer des applications existantes avec Helm dans AKS][aks-helm].

## <a name="statefulsets-and-daemonsets"></a>Ressources StatefulSet et ressources DaemonSet

Le contrôleur de déploiement utilise le planificateur de Kubernetes pour exécuter un certain nombre de réplicas sur n’importe quel nœud disponible ayant des ressources. Cette approche de l’utilisation des déploiements peut être suffisante pour les applications sans état, mais pas pour les applications qui nécessitent un stockage ou une convention de nommage persistant. Pour les applications qui nécessitent qu’un réplica existe sur chaque nœud, ou certains nœuds, au sein d’un cluster, le contrôleur de déploiement ne considère pas la façon dont les réplicas sont répartis entre les nœuds.

Il existe deux ressources Kubernetes qui vous permettent de gérer ces types d’applications :

- *Ressources StatefulSet*  : maintiennent l’état des applications au-delà du cycle de vie d’un pod individuel, tel que le stockage.
- *Ressources DaemonSet*  : garantissent une instance en cours d’exécution sur chaque nœud, tôt dans le processus de démarrage de Kubernetes.

### <a name="statefulsets"></a>Ressources StatefulSet

Le développement d’applications modernes s’adresse souvent aux applications sans état, mais vous pouvez utiliser des *ressources StatefulSet* pour les applications avec état, telles que les applications qui incluent des composants de base de données. Une ressource StatefulSet est similaire à un déploiement, car un ou plusieurs pods identiques sont créés et gérés. Les réplicas d’une ressource StatefulSet suivent une approche séquentielle et sans perte de données du déploiement, de la mise à l’échelle, des mises à niveau et des arrêts. Avec une ressource StatefulSet (quand les réplicas sont replanifiés), la convention d’affectation de noms, les noms de réseau et le stockage persistent.

Vous définissez l’application au format YAML en utilisant `kind: StatefulSet`, puis le contrôleur StatefulSet gère le déploiement et la gestion des réplicas requis. Les données sont écrites dans un stockage persistant, fourni par Azure Disques managés ou Azure Files. Avec une ressource StatefulSet, le stockage permanent sous-jacent demeure, même quand la ressource est supprimée.

Pour plus d’informations, consultez la section [Kubernetes StatefulSets][kubernetes-statefulsets].

Les réplicas dans une ressource StatefulSet sont planifiés et exécutés sur n’importe quel nœud disponible dans un cluster AKS. Si vous devez vous assurer qu’au moins un pod dans votre jeu s’exécute sur un nœud, vous pouvez utiliser à la place une ressource DaemonSet.

### <a name="daemonsets"></a>Ressources DaemonSet

Dans le cadre d’une supervision ou d’une collecte de journaux spécifique, vous pouvez être amené à exécuter un pod donné sur la totalité ou une partie des nœuds. Une ressource *DaemonSet* permet de déployer un ou plusieurs pods identiques, mais le contrôleur DaemonSet garantit que chaque nœud spécifié exécute une instance du pod.

Le contrôleur DaemonSet peut planifier des pods sur des nœuds tôt dans le processus de démarrage du cluster, avant que ne démarre le planificateur Kubernetes par défaut. Ainsi, les pods dans une ressource DaemonSet sont démarrés avant que ne soient planifiés les pods traditionnels dans un déploiement ou une ressource StatefulSet.

À l’image des ressources StatefulSet, une ressource DaemonSet est définie dans le cadre d’une définition YAML à l’aide de `kind: DaemonSet`.

Pour plus d’informations, consultez la section [Kubernetes DaemonSets][kubernetes-daemonset].

> [!NOTE]
> Si vous utilisez le [complément add-on Virtual Nodes](virtual-nodes-cli.md#enable-virtual-nodes-addon), les DaemonSets ne créeront pas de pods sur le nœud virtuel.

## <a name="namespaces"></a>Espaces de noms

Les ressources Kubernetes, telles que les pods et les déploiements, sont regroupées logiquement dans un *espace de noms* . Ces regroupements permettent de scinder un cluster AKS logiquement et de restreindre l’accès pour la création, l’affichage ou la gestion des ressources. Vous pouvez créer des espaces de noms pour séparer les groupes métier, par exemple. Les utilisateurs ne peuvent interagir qu’avec les ressources appartenant aux espaces de noms qui leur sont attribués.

![Espaces de noms Kubernetes pour séparer logiquement les ressources et les applications](media/concepts-clusters-workloads/namespaces.png)

Quand vous créez un cluster AKS, les espaces de noms suivants sont disponibles :

- *default*  : espace de noms dans lequel sont créés par défaut les pods et les déploiements quand aucun espace de noms n’est fourni. Dans les environnements plus petits, vous pouvez déployer les applications directement dans l’espace de noms par défaut sans provoquer la création de séparations logiques supplémentaires. Quand vous interagissez avec l’API Kubernetes, comme avec `kubectl get pods`, l’espace de noms par défaut est utilisé si aucun n’est spécifié.
- *kube-system*  : espace de noms où se trouvent les principales ressources, telles que les fonctionnalités réseau, comme le DNS et le proxy, ou bien le tableau de bord Kubernetes. En règle générale, vous ne déployez pas vos propres applications dans cet espace de noms.
- *kube-public*  : cet espace de noms n'est généralement pas utilisé, mais vous pouvez y recourir pour rendre les ressources visibles dans l'ensemble du cluster et consultables par tous les utilisateurs.

Pour plus d’informations, consultez la section [Espace de noms Kubernetes][kubernetes-namespaces].

## <a name="next-steps"></a>Étapes suivantes

Cet article décrit certains des principaux composants Kubernetes et leur application aux clusters AKS. Pour plus d’informations sur les concepts fondamentaux de Kubernetes et d’AKS, consultez les articles suivants :

- [Accès et identité Kubernetes/AKS][aks-concepts-identity]
- [Sécurité Kubernetes/AKS][aks-concepts-security]
- [Réseaux virtuels Kubernetes/AKS][aks-concepts-network]
- [Stockage Kubernetes/AKS][aks-concepts-storage]
- [Mise à l’échelle Kubernetes/AKS][aks-concepts-scale]

<!-- EXTERNAL LINKS -->
[aks-engine]: https://github.com/Azure/aks-engine
[kubernetes-pods]: https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/
[kubernetes-pod-lifecycle]: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
[kubernetes-deployments]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
[kubernetes-statefulsets]: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
[kubernetes-daemonset]: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
[kubernetes-namespaces]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
[helm]: https://helm.sh/
[azure-cloud-shell]: https://shell.azure.com

<!-- INTERNAL LINKS -->
[aks-concepts-identity]: concepts-identity.md
[aks-concepts-security]: concepts-security.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-storage]: concepts-storage.md
[aks-concepts-network]: concepts-network.md
[acr-helm]: ../container-registry/container-registry-helm-repos.md
[aks-helm]: kubernetes-helm.md
[operator-best-practices-cluster-security]: operator-best-practices-cluster-security.md
[operator-best-practices-scheduler]: operator-best-practices-scheduler.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[operator-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[reservation-discounts]:../cost-management-billing/reservations/save-compute-costs-reservations.md
