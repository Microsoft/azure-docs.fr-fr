---
title: Forum aux questions pour Azure Red Hat OpenShift
description: Trouvez des réponses aux questions fréquentes sur Azure Red Hat OpenShift
author: jimzim
ms.author: jzim
ms.service: container-service
ms.topic: conceptual
ms.date: 07/31/2020
ms.openlocfilehash: 3a474228776c689dbbd6f15ddd926f29383400ce
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94964709"
---
# <a name="azure-red-hat-openshift-faq"></a>FAQ sur Azure Red Hat OpenShift

Cet article répond aux questions fréquemment posées (FAQ) sur Microsoft Azure Red Hat OpenShift.

## <a name="installation-and-upgrade"></a>Installation et mise à niveau

### <a name="which-azure-regions-are-supported"></a>Quelles sont les régions Azure prises en charge ?

Pour obtenir la liste des régions prises en charge pour Azure Red Hat OpenShift 4.x, consultez [Régions disponibles](https://azure.microsoft.com/global-infrastructure/services/?products=openshift&regions=all).

Pour obtenir la liste des régions prises en charge pour Azure Red Hat OpenShift 3.11, consultez [Produits disponibles par région](supported-resources.md#azure-regions).

### <a name="what-virtual-machine-sizes-can-i-use"></a>Quelles tailles de machines virtuelles puis-je utiliser ?

Pour obtenir la liste des tailles de machines virtuelles prises en charge pour Azure Red Hat OpenShift 4, consultez [Ressources prises en charge pour Azure Red Hat OpenShift 4](support-policies-v4.md).

Pour obtenir la liste des tailles de machines virtuelles prises en charge pour Azure Red Hat OpenShift 3.11, consultez [Ressources prises en charge pour Azure Red Hat OpenShift 3.11](supported-resources.md).

### <a name="what-is-the-maximum-number-of-pods-in-an-azure-red-hat-openshift-cluster--what-is-the-maximum-number-of-pods-per-node-in-azure-red-hat-openshift"></a>Quel est le nombre maximal de pods que peut contenir un cluster Azure Red Hat OpenShift ?  Quel est le nombre maximal de pods par nœud dans Azure Red Hat OpenShift ?

Le nombre réel de pods pris en charge dépend de la mémoire, de l’UC et des exigences de stockage d’une application.

Azure Red Hat OpenShift 4.x a une limite de 250 pods par nœud et une limite de 100 nœuds de calcul. Cela limite le nombre maximal de pods pris en charge dans un cluster à 250&times;100 = 25 000.

Azure Red Hat OpenShift 3.11 a une limite de 50 pods par nœud et une limite de 20 nœuds de calcul. Cela limite le nombre maximal de pods pris en charge dans un cluster à 50&times;20 = 1 000.

### <a name="can-a-cluster-have-compute-nodes-across-multiple-azure-regions"></a>Un cluster peut-il disposer de nœuds de calcul dans plusieurs régions Azure ?

Non. Tous les nœuds d’un cluster Azure Red Hat OpenShift doivent provenir de la même région Azure.

### <a name="can-a-cluster-be-deployed-across-multiple-availability-zones"></a>Est-il possible de déployer un cluster sur plusieurs zones de disponibilité ?

Oui. Cela se produit automatiquement si votre cluster est déployé dans une région Azure qui prend en charge les zones de disponibilité. Pour plus d’informations, consultez [Zones de disponibilité](../availability-zones/az-overview.md#availability-zones).

### <a name="are-control-plane-nodes-abstracted-away-as-they-are-with-azure-kubernetes-service-aks"></a>Les nœuds de plan de contrôle sont-ils extraits comme ils le sont avec Azure Kubernetes service (AKS) ?

Non. Toutes les ressources, y compris les nœuds du cluster principal, s’exécutent dans votre abonnement client. Ces types de ressources sont placés dans un groupe de ressources en lecture seule.

### <a name="does-the-cluster-reside-in-a-customer-subscription"></a>Le cluster réside-t-il dans un abonnement client ? 

L’application managée Azure réside dans un groupe de ressources verrouillé avec l’abonnement client. Les clients peuvent consulter les objets de ce groupe de ressources, mais pas les modifier.

### <a name="is-there-any-element-in-azure-red-hat-openshift-shared-with-other-customers-or-is-everything-independent"></a>Des éléments d’Azure Red Hat OpenShift sont-ils partagés avec d’autres clients ? Ou est-ce que tout est indépendant ?

Chaque cluster Azure Red Hat OpenShift est dédié à un client donné et se trouve dans l’abonnement du client. 

### <a name="are-infrastructure-nodes-available"></a>Des nœuds d’infrastructure sont-ils disponibles ?

Sur les clusters Azure Red Hat OpenShift 4.x, les nœuds d’infrastructure ne sont pas disponibles actuellement.

Sur les clusters Azure Red Hat OpenShift 3.11, les nœuds d’infrastructure sont inclus par défaut.

## <a name="how-do-i-handle-cluster-upgrades"></a>Comment faire gérer les mises à niveau de cluster ?

Pour plus d’informations sur les mises à niveau, la maintenance et les versions prises en charge, consultez le [Guide du cycle de vie du support ](support-lifecycle.md).

### <a name="how-will-the-host-operating-system-and-openshift-software-be-updated"></a>Comment sont mis à jour le système d’exploitation hôte et le logiciel OpenShift ?

Les systèmes d’exploitation hôtes et le logiciel OpenShift sont mis à jour à mesure qu’Azure Red Hat OpenShift consomme des versions mineures et des correctifs de la plateforme de conteneurs OpenShift en amont.

### <a name="whats-the-process-to-reboot-the-updated-node"></a>Comment se passe le processus de redémarrage du nœud mis à jour ?

Les nœuds sont redémarrés dans le cadre d’une mise à niveau.

## <a name="cluster-operations"></a>Opérations de cluster

### <a name="can-i-use-prometheus-to-monitor-my-applications"></a>Puis-je utiliser Prometheus pour surveiller mes applications ?

Prometheus est préinstallé et configuré pour les clusters Azure Red Hat OpenShift 4.x. En savoir plus sur la [surveillance du cluster](https://docs.openshift.com/container-platform/3.11/install_config/prometheus_cluster_monitoring.html).

Pour les clusters Azure Red Hat OpenShift 3.11, vous pouvez déployer Prometheus dans votre espace de noms et surveiller les applications de ce dernier. Pour plus d’informations, consultez [Déployer une instance Prometheus dans un cluster Azure Red Hat OpenShift](howto-deploy-prometheus.md).

### <a name="can-i-use-prometheus-to-monitor-metrics-related-to-cluster-health-and-capacity"></a>Puis-je utiliser Prometheus pour surveiller les mesures relatives à la capacité et à l’intégrité du cluster ?

Dans Azure Red Hat OpenShift 4.x : Oui.

Dans Azure Red Hat OpenShift 3.11 : Non.

### <a name="can-logs-of-underlying-vms-be-streamed-out-to-a-customer-log-analysis-system"></a>Les journaux des machines virtuelles sous-jacentes peuvent-ils être diffusés en continu vers un système d’analyse des journaux client ?

Les journaux des machines virtuelles sous-jacentes sont traités par le service géré et ne sont pas exposés aux clients.

### <a name="how-can-a-customer-get-access-to-metrics-like-cpumemory-at-the-node-level-to-take-action-to-scale-debug-issues-etc-i-cannot-seem-to-run-kubectl-top-on-an-azure-red-hat-openshift-cluster"></a>Comment un client peut-il accéder aux métriques telles que l’UC ou la mémoire au niveau du nœud, en vue d’effectuer une mise à l’échelle ou un débogage, par exemple ? Je ne peux pas exécuter kubectl top sur un cluster Azure Red Hat OpenShift.

Pour les clusters Azure Red Hat OpenShift 4.x, la console Web OpenShift contient toutes les métriques au niveau du nœud. Pour plus d’informations, consultez la documentation Red Hat sur [l’affichage des informations des clusters](https://docs.openshift.com/aro/4/web_console/using-dashboard-to-get-cluster-information.html).

Pour les clusters Azure Red Hat OpenShift 3.11, les clients peuvent accéder aux mesures de l’UC/la mémoire au niveau du nœud à l’aide de la commande `oc adm top nodes` ou `kubectl top nodes` avec le rôle de cluster client-administrateur. Les clients peuvent également accéder aux mesures de l’UC/la mémoire de `pods` à l’aide de la commande `oc adm top pods` ou `kubectl top pods`.

### <a name="if-we-scale-up-the-deployment-how-do-azure-fault-domains-map-into-pod-placement-to-ensure-all-pods-for-a-service-do-not-get-knocked-out-by-a-failure-in-a-single-fault-domain"></a>Lors du scale-up d’un déploiement, comment les domaines d’erreur Azure sont-ils mappés à l’emplacement pod pour éviter que tous les pods d’un service ne soient rendus hors d’usage par une défaillance dans un domaine d’erreur ?

Par défaut, il existe cinq domaines d’erreur lorsque vous utilisez des groupes de machines virtuelles identiques dans Azure. Chaque instance de machine virtuelle d’un groupe identique sera placée dans l’un de ces domaines d’erreur. Cela garantit que les applications déployées sur les nœuds de calcul d’un cluster seront placées dans des domaines d’erreur distincts.

Pour plus d’informations, consultez [Choisir le bon nombre de domaines d’erreur pour un groupe de machines virtuelles identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-manage-fault-domains.md).

### <a name="is-there-a-way-to-manage-pod-placement"></a>Existe-t-il un moyen de gérer l’emplacement des pods ?

Les clients ont la possibilité d’obtenir des nœuds et d’afficher des étiquettes en tant que client-administrateur. Cela permettra de cibler les machines virtuelles du groupe identique.

Vous devez être prudent avec certaines étiquettes :

- Vous ne devez pas utiliser le nom d’hôte. Le nom d’hôte « tourne » souvent avec les mises à niveau et les mises à jour, et est donc amené à changer.
- Si le client fait une demande concernant certaines étiquettes ou une stratégie de déploiement, il est possible d’y répondre. Toutefois, cela nécessite des efforts d’ingénierie et n’est pas pris en charge pour le moment.

Pour plus d’informations, consultez [Contrôle de la sélection élective des pods](https://docs.openshift.com/aro/4/nodes/scheduling/nodes-scheduler-about.html).

### <a name="is-the-image-registry-available-externally-so-i-can-use-tools-such-as-jenkins"></a>Le registre d’images est-il disponible en externe afin que je puisse utiliser des outils tels que Jenkins ?

Pour les clusters 4.x, vous devez exposer un registre sécurisé et configurer l’authentification. Pour plus d’informations, consultez la documentation Red Hat suivante :

- [Exposition d’un registre](https://docs.openshift.com/aro/4/registry/securing-exposing-registry.html)
- [Accès au registre](https://docs.openshift.com/aro/4/registry/accessing-the-registry.html)

Pour les clusters 3.11, le registre d’images Docker est disponible. Le registre Docker est disponible à l’adresse `https://docker-registry.apps.<clustername>.<region>.azmosa.io/`. Vous pouvez également utiliser Azure Container Registry.

## <a name="networking"></a>Mise en réseau

### <a name="can-i-deploy-a-cluster-into-an-existing-virtual-network"></a>Puis-je déployer un cluster dans un réseau virtuel existant ?

Dans les clusters 4.x, vous pouvez déployer un cluster dans un réseau virtuel existant.

Dans les clusters 3.11, vous ne pouvez pas déployer de cluster dans un réseau virtuel existant. Vous pouvez connecter un cluster Azure Red Hat OpenShift 3.11 à un réseau virtuel existant via le Peering.

### <a name="is-cross-namespace-networking-supported"></a>La mise en réseau sur les espaces de noms est-elle prise en charge ?

Le client et les administrateurs de projet individuel peuvent personnaliser la mise en réseau sur les espaces de noms (y compris la refuser) par projet à l’aide d’objets `NetworkPolicy`.

### <a name="i-am-trying-to-peer-into-a-virtual-network-in-a-different-subscription-but-getting-failed-to-get-vnet-cidr-error"></a>J’essaie de regarder dans le réseau virtuel d’un autre abonnement, mais j’obtiens une erreur Échec du CIDR du réseau virtuel.

Dans l’abonnement comportant le réseau virtuel, veillez à inscrire le fournisseur `Microsoft.ContainerService` avec la commande suivante : `az provider register -n Microsoft.ContainerService --wait`.

### <a name="can-we-specify-ip-ranges-for-deployment-on-the-private-vnet-avoiding-clashes-with-other-corporate-vnets-once-peered"></a>Est-il possible de spécifier des plages d’adresses IP pour un déploiement sur un réseau virtuel privé, en évitant les conflits avec d’autres réseaux virtuels d’entreprise après le Peering ?

Dans les clusters 4.x, vous pouvez spécifier vos propres plages d’adresses IP.

Dans les clusters 3.11, Azure Red Hat OpenShift prend en charge le Peering de réseaux virtuels et permet au client de fournir un réseau virtuel avec lequel effectuer un Peering, ainsi qu’un routage CIDR (Classless InterDomain Routing) de réseau virtuel pour le réseau OpenShift.

Le réseau virtuel créé par Azure Red Hat OpenShift est protégé et sa configuration ne peut pas être modifiée. Le réseau virtuel appairé est contrôlé par le client et se trouve dans son abonnement.

### <a name="is-the-software-defined-network-module-configurable"></a>Le module Réseau défini par logiciel est-il configurable ?

Le réseau défini par logiciel est `openshift-ovs-networkpolicy` et n’est pas configurable.

### <a name="what-azure-load-balancer-is-used-by-azure-red-hat-openshift--is-it-standard-or-basic-and-is-it-configurable"></a>Quel équilibrage de charge Azure est utilisé par Azure Red Hat OpenShift ?  Est-il de niveau Standard ou De base, et peut-il être configuré ?

Azure Red Hat OpenShift utilise l’équilibreur de charge Azure Standard, qui n’est pas configurable.

## <a name="permissions"></a>Autorisations

### <a name="can-an-admin-manage-users-and-quotas"></a>Un administrateur peut-il gérer des quotas et des utilisateurs ?

Oui. Un administrateur Azure Red Hat OpenShift peut gérer des utilisateurs et des quotas et dispose en plus d’un accès à tous les projets créés par les utilisateurs.

### <a name="can-i-restrict-a-cluster-to-only-certain-azure-ad-users"></a>Puis-je limiter un cluster à certains utilisateurs Azure AD seulement ?

Oui. Vous pouvez désigner les utilisateurs Azure AD pouvant se connecter à un cluster en configurant l’application Azure AD. Pour plus d’informations, consultez [Procédure : Limiter votre application à un ensemble d’utilisateurs](../active-directory/develop/howto-restrict-your-app-to-a-set-of-users.md).

### <a name="can-i-restrict-users-from-creating-projects"></a>Puis-je empêcher les utilisateurs de créer des projets ?

Oui. Connectez-vous à votre cluster en tant qu’administrateur, puis exécutez la commande suivante :

```
oc adm policy \
    remove-cluster-role-from-group self-provisioner \
    system:authenticated:oauth
```

Pour plus d’informations, consultez la documentation OpenShift sur la désactivation de l’approvisionnement automatique pour la version de votre cluster :

- [Désactivation de l’approvisionnement automatique dans les clusters 4.3](https://docs.openshift.com/aro/4/applications/projects/configuring-project-creation.html#disabling-project-self-provisioning_configuring-project-creation)
- [Désactivation de l’approvisionnement automatique dans les clusters 3.11](https://docs.openshift.com/container-platform/3.11/admin_guide/managing_projects.html#disabling-self-provisioning)

### <a name="which-unix-rights-in-iaas-are-available-for-mastersinfraapp-nodes"></a>Quels droits UNIX (dans IaaS) sont disponibles pour les nœuds principaux/d’infrastructure/d’application ?

Pour les clusters 4.x, l’accès aux nœuds est disponible via le rôle Administrateur de cluster. Pour plus d’informations, consultez [Vue d’ensemble de RBAC Kubernetes](https://docs.openshift.com/container-platform/4.3/authentication/using-rbac.html).

Pour les clusters 3.11, l’accès aux nœuds est interdit.

### <a name="which-ocp-rights-do-we-have-cluster-admin-project-admin"></a>De quels droits OCP disposons-nous ? Administrateur de cluster ? Administrateur de projet ?

Pour les clusters 4.x, le rôle Administrateur de cluster est disponible. Pour plus d’informations, consultez [Vue d’ensemble de RBAC Kubernetes](https://docs.openshift.com/container-platform/4.3/authentication/using-rbac.html).

Pour plus d’informations sur les clusters 3.11, consultez la [vue d’ensemble de l’administration des clusters](https://docs.openshift.com/aro/admin_guide/index.html).

### <a name="which-identity-providers-are-available"></a>Quels sont les fournisseurs d’identité disponibles ?

Pour les clusters 4.x, vous configurez votre propre fournisseur d’identité. Pour plus d’informations, consultez la documentation Red Hat sur [la configuration des fournisseurs d’identité](https://docs.openshift.com/aro/4/authentication/identity_providers/configuring-ldap-identity-provider.html).

Pour les clusters 3.11, vous pouvez utiliser l’intégration Azure AD. 

## <a name="storage"></a>Stockage

### <a name="is-data-on-my-cluster-encrypted"></a>Les données de mon cluster sont-elles chiffrées ?

Par défaut, les données sont chiffrées au repos. La plateforme de stockage Azure Storage chiffre automatiquement vos données avant de les rendre persistantes et les déchiffre avant la récupération. Pour plus d’informations, consultez [Azure Storage Service Encryption pour les données au repos](../storage/common/storage-service-encryption.md).

### <a name="is-data-stored-in-etcd-encrypted-on-azure-red-hat-openshift"></a>Les données stockées dans etcd sont-elles chiffrées sur Azure Red Hat OpenShift ?

Pour les clusters Azure Red Hat OpenShift 4, les données ne sont pas chiffrées par défaut, mais vous avez la possibilité d’activer le chiffrement. Pour plus d’informations, consultez le guide sur [le chiffrement d’etcd](https://docs.openshift.com/container-platform/4.3/authentication/encrypting-etcd.html).

Pour les clusters 3.11, les données ne sont pas chiffrées au niveau d’etcd. Il n’est pas possible d’activer le chiffrement dans ce cas. OpenShift prend en charge cette fonctionnalité. Toutefois, des efforts d’ingénierie sont nécessaires pour suivre la feuille de route. Les données sont chiffrées au niveau du disque. Pour plus d’informations, consultez [Encrypting Data at Datastore Layer](https://docs.openshift.com/container-platform/3.11/admin_guide/encrypting_data.html).

### <a name="can-we-choose-any-persistent-storage-solution-like-ocs"></a>Est-il possible de choisir une solution de stockage persistant, comme OCS ? 

Pour les clusters 4.x, Azure Data Box Disk (Premium_LRS) est configuré en tant que classe de stockage par défaut. Pour obtenir des fournisseurs de stockage supplémentaires et pour plus d’informations sur la configuration (notamment Azure Files), consultez la documentation Red Hat sur le [stockage persistant](https://docs.openshift.com/aro/4/storage/understanding-persistent-storage.html).

Pour les clusters 3.11, deux classes de stockage sont fournies par défaut : une pour Azure Data Box Disk (Premium_LRS) et l’autre pour Azure Files.

## <a name="does-aro-store-any-customer-data-outside-of-the-clusters-region"></a>ARO stocke-t-il des données client en dehors de la région du cluster ?

Non. Toutes les données créées dans un cluster ARO sont conservées dans la région du cluster.
