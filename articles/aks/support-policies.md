---
title: Stratégies de support pour Azure Kubernetes Service (AKS)
description: Découvrez les stratégies de support Azure Kubernetes Service (AKS), la responsabilité partagée et les fonctionnalités disponibles en préversion (ou alpha ou bêta).
services: container-service
ms.topic: article
ms.date: 09/18/2020
ms.openlocfilehash: 86b1c0bba30b41a2ee17cfbdf05286c4d2b3fb8a
ms.sourcegitcommit: b437bd3b9c9802ec6430d9f078c372c2a411f11f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91892708"
---
# <a name="support-policies-for-azure-kubernetes-service"></a>Stratégies de support pour Azure Kubernetes Service

Cet article fournit des informations sur les stratégies de support technique et les limitations pour Azure Kubernetes Service (AKS). Il détaille également la gestion des nœuds d’agent, les composants de plan de contrôle géré, les composants open source tiers et la gestion de la sécurité ou des correctifs.

## <a name="service-updates-and-releases"></a>Versions et mises à jour du service

* Pour des informations sur les versions, consultez les [notes de publication relatives à AKS](https://github.com/Azure/AKS/releases).
* Pour des informations sur les fonctionnalités en préversion, consultez [AKS Preview Features and Related Projects](https://awesomeopensource.com/projects/aks?categoryPage=11) (Fonctionnalités d’évaluation AKS et projets associés).

## <a name="managed-features-in-aks"></a>Fonctionnalités gérées dans AKS

Les composants cloud infrastructure as a service (IaaS) de base, tels que les composants réseau ou de calcul, vous permettent d’accéder à des contrôles de niveau inférieur et à des options de personnalisation. En revanche, AKS fournit un déploiement clé en main de Kubernetes qui vous offre l’ensemble courant de configurations et fonctionnalités dont vous avez besoin pour votre cluster. En tant qu’utilisateur AKS, vous disposez d’options de personnalisation et de déploiement limitées. En contrepartie, vous n’avez pas à vous soucier des clusters Kubernetes, ni à les gérer directement.

Avec AKS, vous bénéficiez d’un *plan de contrôle* entièrement géré. Le plan de contrôle contient tous les composants et services dont vous avez besoin pour faire fonctionner et proposer des clusters Kubernetes aux utilisateurs finaux. Tous les composants Kubernetes sont conservés et gérés par Microsoft.

Microsoft gère et surveille les composants suivants via le panneau de contrôle :

* Kubelet ou serveurs d’API Kubernetes
* Etcd ou un stockage clé-valeur compatible assurant la Qualité de service (QoS), l’extensibilité et le runtime
* Services DNS (par exemple, kube-dns ou CoreDNS)
* Proxy Kubernetes ou réseau
* Tout module complémentaire ou composant système en cours d’exécution dans l’espace de noms kube-system

AKS n’est pas une solution PaaS (Platform-as-a-Service). Certains composants, tels que des nœuds d’agent ont *la responsabilité partagée*, où les utilisateurs doivent participer à la gestion du cluster AKS. L’entrée utilisateur est nécessaire, par exemple, pour appliquer un correctif de sécurité du système d’exploitation de nœud d’agent.

Les services sont *gérés* de façon que Microsoft et l’équipe AKS déploient, gèrent et soient responsables de l’opérationnalité et de la disponibilité du service. Les clients ne peuvent pas modifier ces composants gérés. Microsoft limite la personnalisation, afin de garantir une expérience utilisateur cohérente et évolutive. Pour une solution entièrement personnalisable, consultez [AKS Engine](https://github.com/Azure/aks-engine) (Moteur AKS).

## <a name="shared-responsibility"></a>Responsabilité partagée

Quand un cluster est créé, vous définissez les nœuds de l’agent Kubernetes créés par AKS. Vos charges de travail sont exécutées sur ces nœuds.

Étant donné que les nœuds de votre agent exécutent du code privé et stockent des données sensibles, le Support Microsoft peut y accéder de manière très limitée uniquement. Le Support Microsoft ne peut pas se connecter, exécuter des commandes ou afficher les journaux de ces nœuds sans votre autorisation expresse ou assistance.

Toute modification apportée directement aux nœuds de l’agent à l’aide de l’une des API IaaS rend le cluster non pris en charge. Toute modification apportée aux nœuds de l’agent doit être effectuée à l’aide de mécanismes Kubernetes natifs, tels que `Daemon Sets`.

De même, si vous pouvez ajouter des métadonnées au cluster et à des nœuds, comme des balises et des étiquettes, la modification des métadonnées créées par le système rendra le cluster non pris en charge.

## <a name="aks-support-coverage"></a>Couverture du support AKS

Microsoft fournit un support technique pour les exemples suivants :

* Connectivité à tous les composants Kubernetes que le service Kubernetes fournit et prend en charge, tel que le serveur d’API.
* Gestion, durée de fonctionnement, QoS et opérations des services de plan de contrôle Kubernetes (plan de contrôle Kubernetes, serveur d’API, etcd et coreDNS, par exemple).
* Magasin de données etcd. Le support inclut des sauvegardes automatisées et transparentes de l’ensemble des données etcd toutes les 30 minutes pour la planification de la récupération d’urgence et la restauration de l’état du cluster. Ces sauvegardes ne sont pas directement disponibles pour vous ou les utilisateurs. Elles garantissent la cohérence et la fiabilité des données. Etcd. la restauration à la demande ou la restauration ne sont pas prises en charge en tant que fonctionnalité.
* Points d’intégration dans le pilote du fournisseur cloud Azure pour Kubernetes. Ceux-ci incluent des intégrations à d’autres services Azure tels que les équilibreurs de charge, les volumes persistants ou la mise en réseau (Kubernetes et Azure CNI).
* Questions ou problèmes relatifs à la personnalisation des composants du plan de contrôle tels que le serveur d’API Kubernetes, etcd et coreDNS.
* Problèmes relatifs au réseau, tels qu’Azure CNI, que kubenet, ou d’autres problématiques en lien avec des fonctionnalités ou l’accès réseau. Les problèmes peuvent inclure la résolution DNS, la perte de paquets, le routage, etc. Microsoft prend en charge différents scénarios de mise en réseau :
  * Kubenet et Azure CNI avec des réseaux virtuels gérés ou des sous-réseaux personnalisés (« Apporter votre propre »).
  * Connectivité à d’autres services et applications Azure
  * Contrôleurs d’entrée et configurations d’équilibreur de charge ou d’entrée
  * Performances et latence réseau


> [!NOTE]
> Toute action de cluster prise par Microsoft/AKS est effectuée avec le consentement de l’utilisateur sous un rôle Kubernetes intégré `aks-service` et une liaison de rôle intégrée `aks-service-rolebinding`. Ce rôle permet à AKS de dépanner et de diagnostiquer les problèmes de cluster, mais ne peut pas modifier les autorisations, ni créer des rôles ou des liaisons de rôle, ni d’autres actions à privilèges élevés. L’accès aux rôles est activé uniquement sous les tickets de support actifs avec l’accès juste-à-temps (JIT).

Microsoft ne fournit pas de support technique pour les exemples suivants :

* Questions sur l’utilisation de Kubernetes. Par exemple, le Support Microsoft ne fournit pas de conseils sur la façon dont les contrôleurs d’entrée doivent être créés, sur l’utilisation des charges de travail d’application ou sur l’application d’outils ou de packages logiciels open source ou tiers.
  > [!NOTE]
  > Le Support Microsoft peut vous conseiller sur la fonctionnalité de cluster AKS, la personnalisation et le paramétrage (par exemple, les procédures et problèmes relatifs aux opérations Kubernetes).
* Les projets open source tiers qui ne sont pas fournis dans le cadre du plan de contrôle Kubernetes ou déployés avec des clusters AKS. Ces projets peuvent inclure Istio, Helm, Envoy, etc.
  > [!NOTE]
  > Microsoft fera de son mieux pour assurer le support des projets open source tiers tels que Helm. Quand l’outil open source tiers est associé au fournisseur cloud Azure Kubernetes ou à d’autres bogues spécifiques à AKS, Microsoft fournit des exemples et applications issus de sa documentation.
* Logiciels fermés tiers. Ces logiciels peuvent inclure des outils d’analyse de sécurité et des logiciels ou appareils réseau.
* Personnalisations réseau autres que celles répertoriées dans la [documentation AKS](./index.yml).


## <a name="aks-support-coverage-for-agent-nodes"></a>Couverture du support AKS pour les nœuds d’agent

### <a name="microsoft-responsibilities-for-aks-agent-nodes"></a>Responsabilités de Microsoft pour les nœuds d’agent AKS

Microsoft et les clients partagent la responsabilité relative aux nœuds d’agent Kubernetes quand :

* L’image du système d’exploitation de base a requis des ajouts (par exemple, des agents réseau et de surveillance).
* Les nœuds d’agent reçoivent automatiquement des correctifs de système d’exploitation.
* Des problèmes en lien avec les composants du plan de contrôle Kubernetes qui s’exécutent sur les nœuds d’agent sont automatiquement corrigés. Ces composants comprennent les suivants :
  * `Kube-proxy`
  * Tunnels de mise en réseau qui fournissent des chemins d’accès de communication vers les composants master Kubernetes
  * `Kubelet`
  * `Moby` ou `ContainerD`

> [!NOTE]
> Si un nœud d’agent n’est pas opérationnel, AKS peut redémarrer des composants individuels ou l’intégralité du nœud de l’agent. Ces opérations de redémarrage sont automatisées et assurent une résolution automatique des problèmes courants. Si vous souhaitez en savoir plus sur les mécanismes de correction automatique, consultez [Réparation automatique des nœuds](node-auto-repair.md)

### <a name="customer-responsibilities-for-aks-agent-nodes"></a>Responsabilités du client pour les nœuds d’agent AKS

Microsoft fournit des correctifs et de nouvelles images pour vos nœuds d’image chaque semaine, mais ne les applique pas automatiquement par défaut. Pour que les composants d’exécution et de système d’exploitation du nœud de l’agent soient corrigés, vous devez conserver une planification régulière de [mise à niveau d’image de nœud](node-image-upgrade.md) ou l’automatiser.

De même, AKS publie régulièrement de nouveaux correctifs et de nouvelles versions mineures de Kubernetes. Ces mises à jour peuvent contenir des améliorations en matière de sécurité ou de fonctionnalités pour Kubernetes. Vous êtes responsable de la mise à jour de la version de Kubernetes des clusters conformément à la [Stratégie de gestion des versions prises en charge d’AKS Kubernetes](supported-kubernetes-versions.md).

#### <a name="user-customization-of-agent-nodes"></a>Personnalisation par l’utilisateur des nœuds de l’agent
> [!NOTE]
> Les nœuds d’agent AKS sont affichés dans le portail Azure en tant que ressources Azure IaaS standard. Toutefois, ces machines virtuelles sont déployées dans un groupe de ressources Azure personnalisé (portant généralement le préfixe MC_\*). Vous ne pouvez pas modifier l’image de base du système d’exploitation ou effectuer des personnalisations directes sur ces nœuds à l’aide des API ou des ressources IaaS. Les modifications personnalisées qui ne sont pas effectuées via l’API AKS ne sont pas conservées lors d’une mise à niveau, d’une mise à l’échelle, d’une mise à jour ou d’un redémarrage. Évitez d’effectuer des modifications sur les nœuds de l’agent, sauf si le Support Microsoft vous demande d’apporter ces modifications.

AKS gère le cycle de vie et les opérations des nœuds d’agent pour votre compte ; la modification des ressources IaaS associées aux nœuds d’agent n'est **pas prise en charge**. À titre d'exemple, la personnalisation d’un jeu de mise à l’échelle de machines virtuelles de pools de nœuds en modifiant manuellement les configurations sur le groupe de machines virtuelles identiques (VMSS) via le portail VMSS ou l’API VMSS n'est pas prise en charge.
 
Pour les configurations ou packages spécifiques à la charge de travail, AKS recommande l’utilisation de [ressources Kubernetes `daemon sets`](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

L’utilisation de ressources Kubernetes `daemon sets` privilégiées et de conteneurs init permet aux clients de paramétrer/modifier ou d’installer des logiciels tiers sur des nœuds d’agent de cluster. Parmi ces personnalisations, citons l’ajout d’un logiciel d’analyse de sécurité personnalisé ou la mise à jour des paramètres sysctl.

Bien qu’il s’agisse d’un chemin d’accès recommandé si les exigences ci-dessus s’appliquent, l’ingénierie et la prise en charge AKS ne permettent pas de résoudre les problèmes ou de diagnostiquer des modifications qui rendent le nœud indisponible en raison du déploiement d'une ressource `daemon set` personnalisée.

### <a name="security-issues-and-patching"></a>Problèmes de sécurité et application de correctifs

Si une faille de sécurité se trouve dans un ou plusieurs composants gérés d’AKS, l’équipe AKS appliquera des correctifs à tous les clusters affectés pour atténuer le problème. L’équipe peut également donner des conseils en matière de mise à niveau aux utilisateurs.

Pour les nœuds d’agent affectés par une faille de sécurité, Microsoft vous informe des détails sur l’impact et les étapes à suivre pour corriger ou atténuer le problème de sécurité (normalement, une mise à niveau d’image de nœud ou une mise à niveau de correctif de cluster).

### <a name="node-maintenance-and-access"></a>Accès et maintenance des nœuds

Même si vous pouvez vous connecter aux nœuds d’agent et les modifier, ces opérations sont déconseillées, car ces modifications peuvent affecter le support du cluster.

## <a name="network-ports-access-and-nsgs"></a>Ports réseau, accès et groupes de sécurité réseau

Vous pouvez uniquement personnaliser les groupes de sécurité réseau sur des sous-réseaux personnalisés. Vous ne pouvez pas personnaliser les groupes de sécurité réseau sur des sous-réseaux gérés ou au niveau de la carte réseau des nœuds de l’agent. AKS a des exigences de sortie sur des points de terminaison spécifiques. Pour contrôler la sortie et garantir la connectivité nécessaire, consultez [Limiter le trafic sortant](limit-egress-traffic.md).

## <a name="stopped-or-de-allocated-clusters"></a>Clusters arrêtés ou désalloués

Comme indiqué précédemment, la désallocation manuelle de tous les nœuds de cluster via les API IaaS, l’interface CLI ou le portail supprime toute prise en charge du cluster. La seule méthode prise en charge pour arrêter/désallouer tous les nœuds consiste à [arrêter le cluster AKS](start-stop-cluster.md#stop-an-aks-cluster), ce qui préserve l’état du cluster pendant un maximum de 12 mois.

Les clusters arrêtés depuis plus de 12 mois ne conservent plus l’état. 

Les clusters qui sont désalloués en dehors des API AKS n’ont aucune garantie de conservation de l’état. Les plans de contrôle pour les clusters dans cet état seront archivés au bout de 30 jours et supprimés au bout de 12 mois.

AKS se réserve le droit d’archiver les plans de contrôle qui ont été configurés sans respecter les instructions de prise en charge pendant des périodes prolongées supérieures ou égales à 30 jours. AKS tient à jour les sauvegardes des métadonnées etcd du cluster et peut facilement réallouer le cluster. Cette réallocation peut être lancée par toute opération PUT qui permet au cluster de bénéficier à nouveau d’une prise en charge, par exemple une mise à niveau ou une mise à l’échelle vers des nœuds d’agent actifs.

Si votre abonnement est suspendu ou supprimé, le plan de contrôle et l’état de votre cluster seront supprimés au bout de 90 jours.

## <a name="unsupported-alpha-and-beta-kubernetes-features"></a>Fonctionnalités Kubernetes alpha et bêta non supportées

AKS prend uniquement en charge les fonctionnalités stables et bêta au sein du projet Kubernetes en amont. Sauf mention contraire, AKS ne supporte pas les fonctionnalités alpha qui sont disponibles dans le projet Kubernetes en amont.

## <a name="preview-features-or-feature-flags"></a>Fonctionnalités d’évaluation ou indicateurs de fonctionnalités

Pour les fonctionnalités et fonctions qui exigent des tests étendus et des commentaires des utilisateurs, Microsoft publie de nouvelles fonctionnalités d’évaluation ou des fonctionnalités appartenant à un indicateur de fonctionnalité. Considérez ces fonctionnalités comme des fonctionnalités bêta ou en version préliminaire.

Les fonctionnalités d’évaluation ou fonctionnalités avec indicateur de fonctionnalité ne sont pas destinées aux environnements de production. Les modifications continues apportées aux API et aux comportements, corrections de bogues et autres types de modifications peuvent entraîner des temps d’arrêt et un manque de stabilité des clusters.

Les fonctionnalités en préversion publique font l’objet d’un support relatif, puisque ces fonctionnalités ne sont pas destinées à la production et sont supportées par les équipe du support technique AKS pendant les heures de bureau uniquement. Pour plus d'informations, consultez les pages suivantes :

* [FAQ du support Azure](https://azure.microsoft.com/support/faq/)

## <a name="upstream-bugs-and-issues"></a>Problèmes et bogues en amont

Étant donné la vitesse de développement dans le projet Kubernetes en amont, des bogues surviennent invariablement. Certains de ces bogues ne peuvent pas être corrigés ni contournés au sein du système AKS. Au lieu de cela, des corrections de bogues exigent des correctifs plus importants pour les projets en amont (comme Kubernetes, systèmes d’exploitation d’agent ou de nœud et noyau). Pour les composants appartenant à Microsoft (par exemple, le fournisseur cloud Azure), AKS et le personnel Azure s’engagent à corriger les problèmes en amont dans la communauté.

Lorsqu’un problème de support technique est la cause racine d’un ou de plusieurs bogues en amont, les équipes d’ingénierie et de support AKS procèdent comme suit :

* Ils identifient et associent les bogues en amont aux détails de support afin d’expliquer pourquoi ce problème affecte votre cluster ou charge de travail. Les clients reçoivent des liens vers les référentiels nécessaires afin de pouvoir étudier les problèmes et voir quand une nouvelle version fournira des correctifs.
* Ils proposent des atténuations ou solutions alternatives éventuelles. Si le problème peut être atténué, un [problème connu](https://github.com/Azure/AKS/issues?q=is%3Aissue+is%3Aopen+label%3Aknown-issue) est consigné dans le référentiel AKS. La consignation du problème connu décrit :
  * Le problème, avec des liens vers des bogues en amont.
  * La solution alternative et des informations sur une mise à jour ou une autre persistance de la solution.
  * Des chronologies approximatives pour l’inclusion du problème, en fonction de la cadence des versions en amont.
