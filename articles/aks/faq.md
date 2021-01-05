---
title: Forum aux questions sur Azure Kubernetes Service (AKS)
description: Recherchez des réponses à certaines des questions les plus fréquemment posées sur Azure Kubernetes Service (AKS).
ms.topic: conceptual
ms.date: 08/06/2020
ms.openlocfilehash: 94cbaf417413b3e11071fb8c7237cbb3ac7b9a37
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96780346"
---
# <a name="frequently-asked-questions-about-azure-kubernetes-service-aks"></a>Forum aux questions sur Azure Kubernetes Service (AKS)

Cet article aborde les questions fréquemment posées sur Azure Kubernetes Service (AKS).

## <a name="which-azure-regions-currently-provide-aks"></a>Quelles régions Azure proposent actuellement AKS ?

Pour obtenir la liste complète des régions disponibles, consultez [Régions AKS et disponibilité][aks-regions].

## <a name="can-i-spread-an-aks-cluster-across-regions"></a>Puis-je répartir un cluster AKS dans plusieurs régions ?

Non. Les clusters AKS sont des ressources régionales et ne peuvent pas s’étendre sur plusieurs régions. Consultez les [meilleures pratiques relatives à la continuité d’activité et reprise d’activité][bcdr-bestpractices] pour obtenir des conseils sur la façon de créer une architecture qui comprend plusieurs régions.

## <a name="can-i-spread-an-aks-cluster-across-availability-zones"></a>Puis-je répartir un cluster AKS dans plusieurs zones de disponibilité ?

Oui. Vous pouvez déployer un cluster AKS sur une ou plusieurs [zones de disponibilité][availability-zones] dans les [régions qui les prennent en charge][az-regions].

## <a name="can-i-limit-who-has-access-to-the-kubernetes-api-server"></a>Puis-je limiter qui a accès au serveur d’API Kubernetes ?

Oui. Il existe deux options pour limiter l’accès au serveur d’API :

- Utiliser des [plages d’adresses IP autorisées pour le serveur d’API][api-server-authorized-ip-ranges] si vous souhaitez conserver un point de terminaison public pour le serveur d’API, mais limiter l’accès à un ensemble de plages d’adresses IP approuvées.
- Utiliser [un cluster privé][private-clusters] si vous souhaitez limiter le serveur d’API de sorte qu’il soit accessible *uniquement* à partir de votre réseau virtuel.

## <a name="can-i-have-different-vm-sizes-in-a-single-cluster"></a>Puis-je avoir des machines virtuelles de différentes tailles dans un même cluster ?

Oui, vous pouvez utiliser différentes tailles de machines virtuelles dans votre cluster AKS en créant [plusieurs pools de nœuds][multi-node-pools].

## <a name="are-security-updates-applied-to-aks-agent-nodes"></a>Des mises à jour sécurisées sont-elles appliquées aux nœuds de l’agent AKS ?

Azure applique automatiquement les correctifs de sécurité pour les nœuds Linux de votre cluster selon une planification nocturne. Toutefois, vous êtes tenu de vous assurer que ces nœuds Linux sont redémarrés comme il se doit. Vous avez plusieurs options pour redémarrer les nœuds :

- Manuellement, via le portail Azure ou l’interface de ligne de commande Azure.
- En mettant à niveau votre cluster AKS. Le cluster met automatiquement à niveau des [nœuds drain et cordon][cordon-drain], avant de mettre en ligne un nouveau nœud avec la dernière image Ubuntu et une nouvelle version du correctif ou une version Kubernetes mineure. Pour plus d’informations, consultez [Mettre à niveau un cluster AKS][aks-upgrade].
- En [mettant à niveau l’image du nœud](node-image-upgrade.md).

### <a name="windows-server-nodes"></a>Nœuds Windows Server

Pour les nœuds Windows Server, Windows Update n’exécute pas et n’applique pas automatiquement les dernières mises à jour. Suivez une planification régulière basée sur le cycle de mise à jour de Windows et votre propre processus de validation pour effectuer une mise à niveau sur le cluster ou le ou les pools de nœuds Windows Server dans votre cluster AKS. Ce processus de mise à niveau crée des nœuds qui exécutent la dernière image et les derniers correctifs de Windows Server, puis supprime les anciens nœuds. Pour plus d’informations sur ce processus, consultez [Mettre à niveau un pool de nœuds dans AKS][nodepool-upgrade].

## <a name="why-are-two-resource-groups-created-with-aks"></a>Pourquoi deux groupes de ressources sont-ils créés avec AKS ?

AKS s’appuie sur différentes ressources de l'infrastructure Azure, notamment les groupes de machines virtuelles identiques, réseaux virtuels et disques managés. Cela vous permet de tirer parti des nombreuses fonctionnalités essentielles de la plateforme Azure dans l’environnement Kubernetes managé fourni par AKS. Par exemple, la plupart des machines virtuelles Azure peuvent être directement utilisées avec AKS. En outre, les Réservations Azure permettent de bénéficier automatiquement de remises sur ces ressources.

Pour permettre cette architecture, chaque déploiement AKS s’étend sur deux groupes de ressources :

1. Vous créez le premier groupe de ressources. Ce groupe contient uniquement la ressource de service Kubernetes. Le fournisseur de ressources AKS crée automatiquement le second groupe de ressources au cours du déploiement. Un exemple du second groupe de ressources est *MC_myResourceGroup_myAKSCluster_eastus*. Pour obtenir des informations sur la façon de spécifier le nom de ce second groupe de ressources, consultez la section suivante.
1. Le second groupe de ressources, nommé *groupe de ressources de nœud*, contient toutes les ressources d’infrastructure associées au cluster. Ces ressources incluent les machines virtuelles de nœud Kubernetes, la mise en réseau et le stockage. Par défaut, le nom du groupe de ressources de nœud ressemble à ceci : *MC_myResourceGroup_myAKSCluster_eastus*. AKS supprime automatiquement la ressource de nœud à chaque fois que le cluster est supprimé. Cette ressource doit donc être utilisée uniquement pour les ressources qui partagent le cycle de vie du cluster.

## <a name="can-i-provide-my-own-name-for-the-aks-node-resource-group"></a>Puis-je nommer mon groupe de ressources d’infrastructure AKS comme je le veux ?

Oui. Par défaut, AKS nomme le groupe de ressources de nœud *MC_resourcegroupname_clustername_location*, mais vous pouvez également entrer votre propre nom.

Pour spécifier votre propre nom de groupe de ressources, installez la version *0.3.2* ou une version ultérieure de l’extension Azure CLI [aks-preview][aks-preview-cli]. Lorsque vous créez un cluster AKS à l’aide de la commande [az aks create][az-aks-create], utilisez le paramètre `--node-resource-group` et spécifiez un nom pour le groupe de ressources. Si vous [utilisez un modèle Azure Resource Manager][aks-rm-template] pour déployer un cluster AKS, vous pouvez définir le nom du groupe de ressources à l’aide de la propriété *nodeResourceGroup*.

* Le groupe de ressources secondaire est automatiquement créé par le fournisseur de ressources Azure dans votre propre abonnement.
* Vous pouvez spécifier un nom de groupe de ressources personnalisé uniquement lorsque vous créez le cluster.

Lorsque vous travaillez avec le groupe de ressources de nœud, n’oubliez pas que vous ne pouvez pas :

* Spécifier un groupe de ressources existant pour le groupe de ressources de nœud.
* Spécifier un autre abonnement pour le groupe de ressources de nœud.
* Modifier le nom du groupe de ressources de nœud une fois que le cluster a été créé.
* Spécifier des noms pour les ressources managées au sein du groupe de ressources de nœud.
* Modifier ni supprimer les étiquettes des ressources managées créées par Azure au sein du groupe de ressources de nœud. (Consultez des informations supplémentaires dans la section suivante.)

## <a name="can-i-modify-tags-and-other-properties-of-the-aks-resources-in-the-node-resource-group"></a>Puis-je modifier les balises et d’autres propriétés des ressources AKS dans le groupe de ressources de nœud ?

Si vous modifiez ou supprimez des balises créées par Azure et d’autres propriétés de ressources dans le groupe de ressources de nœud, vous pouvez obtenir des résultats inattendus tels que des erreurs de mise à l’échelle et de mise à niveau. AKS vous permet de créer et de modifier des balises personnalisées créées par les utilisateurs finaux. Vous pouvez ajouter ces balises lors de la [création d’un pool de nœuds](use-multiple-node-pools.md#specify-a-taint-label-or-tag-for-a-node-pool). Vous souhaiterez peut-être créer ou modifier des balises personnalisées, par exemple, pour affecter une unité commerciale ou un centre de coûts. Vous pouvez également des stratégies Azure avec une étendue sur le groupe de ressources managé.

Toutefois, la modification de **balises créées par Azure** sur des ressources dépendant du groupe de ressources du nœud dans le cluster AKS est une action non prise en charge, qui rompt l’objectif de niveau de service (SLO). Pour plus d'informations, consultez [AKS offre-t-il un contrat de niveau de service ?](#does-aks-offer-a-service-level-agreement)

## <a name="what-kubernetes-admission-controllers-does-aks-support-can-admission-controllers-be-added-or-removed"></a>Quels sont les contrôleurs d’admission Kubernetes qui sont pris en charge par AKS ? Les contrôleurs d’admission peuvent-ils être ajoutés ou supprimés ?

AKS prend en charge les [contrôleurs d’admission][admission-controllers] suivants :

- *NamespaceLifecycle*
- *LimitRanger*
- *ServiceAccount*
- *DefaultStorageClass*
- *DefaultTolerationSeconds*
- *MutatingAdmissionWebhook*
- *ValidatingAdmissionWebhook*
- *ResourceQuota*
- *PodNodeSelector*
- *PodTolerationRestriction*
- *ExtendedResourceToleration*

Actuellement, vous ne pouvez pas modifier la liste des contrôleurs d’admission dans AKS.

## <a name="can-i-use-admission-controller-webhooks-on-aks"></a>Puis-je utiliser des webhooks de contrôleur d’admission dans AKS ?

Oui, vous pouvez utiliser des webhooks de contrôleur d’admission dans AKS. Il est recommandé d’exclure les espaces de noms AKS internes, qui sont signalés par l’**étiquette control-plane**. Par exemple, en ajoutant ce qui suit à la configuration du webhook :

```
namespaceSelector:
    matchExpressions:
    - key: control-plane
      operator: DoesNotExist
```

AKS protège par un pare-feu la sortie du serveur d’API. Vous devez donc accéder aux webhooks du contrôleur d’admission à partir du cluster.

## <a name="can-admission-controller-webhooks-impact-kube-system-and-internal-aks-namespaces"></a>Les webhooks de contrôleur d’admission peuvent-ils avoir un impact sur les espaces de noms kube-system et AKS internes ?

Pour maintenir la stabilité du système et empêcher les contrôleurs d’admission personnalisés d’avoir un impact sur les services internes de kube-system, l’espace de noms AKS comprend un **exécuteur d’admission**, qui exclut automatiquement les espaces de noms kube-system et AKS internes. Ce service garantit que les contrôleurs d’admission personnalisés n’affecteront pas les services exécutés dans kube-system.

Si vous avez un cas d’usage critique dans lequel un élément déployé sur kube-system (non recommandé) doit être couvert par votre webhook d’admission personnalisé, vous pouvez ajouter l’étiquette ou l’annotation ci-dessous pour que l’exécuteur d’admission l’ignore.

Étiquette : ```"admissions.enforcer/disabled": "true"``` ou annotation : ```"admissions.enforcer/disabled": true```

## <a name="is-azure-key-vault-integrated-with-aks"></a>Azure Key Vault est-il intégré à AKS ?

Pour l’instant, AKS n’est pas intégré nativement à Azure Key Vault. Toutefois, le [fournisseur Azure Key Vault pour le magasin de secrets CSI][csi-driver] permet l’intégration directe à partir de pods Kubernetes à des Secrets Key Vault.

## <a name="can-i-run-windows-server-containers-on-aks"></a>Puis-je exécuter des conteneurs Windows Server sur AKS ?

Oui, les conteneurs Windows Server sont disponibles dans AKS. Pour exécuter des conteneurs Windows Server dans AKS, vous créez un pool de nœuds qui exécute Windows Server en tant que système d’exploitation invité. Les conteneurs Windows Server peuvent utiliser uniquement Windows Server 2019. Pour commencer, consultez [Créer un cluster AKS avec un pool de nœuds Windows Server][aks-windows-cli].

La prise en charge des pools de nœuds dans Windows Server comprend certaines limitations qui font partie du projet amont Windows Server dans Kubernetes. Pour plus d’informations sur ces limitations, consultez [Limitations des conteneurs Windows Server dans AKS][aks-windows-limitations].

## <a name="does-aks-offer-a-service-level-agreement"></a>AKS offre-t-il un contrat de niveau de service ?

AKS fournit des garanties adossées à un contrat SLA à titre complémentaire facultatif avec le [Contrat SLA de durée de fonctionnement][uptime-sla].

## <a name="can-i-apply-azure-reservation-discounts-to-my-aks-agent-nodes"></a>Puis-je appliquer des remises de réservation Azure sur mes nœuds d’agent AKS ?

Les nœuds d’agent AKS sont facturés en tant que machines virtuelles Azure standard. Par conséquent, si vous avez acheté des [réservations Azure][reservation-discounts] pour la taille de machine virtuelle que vous utilisez dans AKS, ces remises sont automatiquement appliquées.

## <a name="can-i-movemigrate-my-cluster-between-azure-tenants"></a>Puis-je déplacer/migrer mon cluster entre des locataires Azure ?

Le déplacement de votre cluster AKS entre locataires n’est actuellement pas pris en charge.

## <a name="can-i-movemigrate-my-cluster-between-subscriptions"></a>Puis-je déplacer/migrer mon cluster entre des abonnements ?

Désolé... Le déplacement des clusters entre des abonnements n’est pas pris en charge pour le moment.

## <a name="can-i-move-my-aks-clusters-from-the-current-azure-subscription-to-another"></a>Puis-je déplacer mes clusters AKS de l’abonnement Azure actuel vers un autre abonnement ? 

Le déplacement de votre cluster AKS et des ressources associées entre des abonnements Azure n’est pas pris en charge.

## <a name="can-i-move-my-aks-cluster-or-aks-infrastructure-resources-to-other-resource-groups-or-rename-them"></a>Puis-je déplacer mon cluster AKS ou mes ressources d'infrastructure AKS vers d'autres groupes de ressources ou les renommer ?

Le déplacement ou le changement de nom de votre cluster AKS et des ressources associées ne sont pas pris en charge.

## <a name="why-is-my-cluster-delete-taking-so-long"></a>Pourquoi la suppression de mon cluster est-elle si longue ? 

La plupart des clusters sont supprimés à la demande de l'utilisateur ; dans certains cas, notamment lorsque des clients apportent leur propre groupe de ressources ou effectuent des suppressions de tâches entre des groupes de ressources, l’opération peut prendre plus de temps ou échouer. Si vous rencontrez un problème avec les suppressions, vérifiez que le groupe de ressources ne contient aucun verrou, que toutes les ressources en dehors du groupe sont dissociées, et ainsi de suite.

## <a name="if-i-have-pod--deployments-in-state-nodelost-or-unknown-can-i-still-upgrade-my-cluster"></a>Si j'ai des pods/déploiements à l'état 'NodeLost' ou 'Unknown', puis-je quand même mettre à niveau mon cluster ?

Vous pouvez, mais AKS ne le recommande pas. Les mises à niveau devraient être effectuées lorsque l’état du cluster est connu et sain.

## <a name="if-i-have-a-cluster-with-one-or-more-nodes-in-an-unhealthy-state-or-shut-down-can-i-perform-an-upgrade"></a>Si j'ai un cluster avec un ou plusieurs nœuds à l’état Unhealthy (non sain) ou shut down (arrêté), puis-je effectuer une mise à niveau ?

Non, supprimez tout nœud en état d’échec ou ayant été supprimé du cluster avant la mise à niveau.

## <a name="i-ran-a-cluster-delete-but-see-the-error-errno-11001-getaddrinfo-failed"></a>J'ai lancé une suppression de cluster, mais l'erreur `[Errno 11001] getaddrinfo failed` s’affiche 

Le plus souvent, cela est dû au fait qu'un ou plusieurs groupes de sécurité réseau (NSG) sont encore utilisés et associés au cluster.  Supprimez-les, puis réessayez la suppression.

## <a name="i-ran-an-upgrade-but-now-my-pods-are-in-crash-loops-and-readiness-probes-fail"></a>J'ai effectué une mise à niveau, mais maintenant mes pods se retrouvent dans des boucles de plantage, et les readiness probes échouent.

Vérifiez que votre principal de service n’a pas expiré.  Consultez l'article : [Principal du service AKS](./kubernetes-service-principal.md) et [Informations d'identification pour la mise à jour d’AKS](./update-credentials.md).

## <a name="my-cluster-was-working-but-suddenly-cant-provision-loadbalancers-mount-pvcs-etc"></a>Mon cluster fonctionnait, mais, tout à coup, il ne peut plus approvisionner LoadBalancers, monter des revendications de volume persistant (PVC), etc. 

Vérifiez que votre principal de service n’a pas expiré.  Consultez l'article : [Principal du service AKS](./kubernetes-service-principal.md) et [Informations d'identification pour la mise à jour d’AKS](./update-credentials.md).

## <a name="can-i-scale-my-aks-cluster-to-zero"></a>Puis-je mettre à l’échelle mon cluster AKS jusqu’à zéro ?
Vous pouvez complètement [arrêter un cluster AKS en cours d’exécution](start-stop-cluster.md), ce qui vous permet d’économiser les coûts de calcul respectifs. Vous pouvez également choisir de [mettre à l’échelle manuellement ou automatiquement la totalité ou une partie des pools de nœuds `User`](scale-cluster.md#scale-user-node-pools-to-0) sur 0, en conservant uniquement la configuration de cluster nécessaire.
Vous ne pouvez pas mettre directement à l’échelle des [pools de nœuds système](use-system-pools.md) sur 0.

## <a name="can-i-use-the-virtual-machine-scale-set-apis-to-scale-manually"></a>Puis-je utiliser les API du groupe de machines virtuelles identiques pour effectuer une mise à l'échelle manuelle ?

Non, les opérations de mise à l'échelle à l'aide des API du groupe de machines virtuelles identiques ne sont pas prises en charge. Utilisez les API AKS (`az aks scale`).

## <a name="can-i-use-virtual-machine-scale-sets-to-manually-scale-to-zero-nodes"></a>Puis-je utiliser des groupes de machines virtuelles identiques pour mettre à l’échelle manuellement des nœuds sur 0 ?

Non, les opérations de mise à l'échelle à l'aide des API du groupe de machines virtuelles identiques ne sont pas prises en charge. Vous pouvez utiliser l’API AKS pour mettre à l’échelle les pools de nœuds non système sur 0 ou [arrêter votre cluster](start-stop-cluster.md) à la place.

## <a name="can-i-stop-or-de-allocate-all-my-vms"></a>Puis-je arrêter ou désallouer toutes mes machines virtuelles ?

Bien qu’AKS dispose de mécanismes de résilience capable de gérer une telle configuration et de récupérer à partir de celle-ci, cette configuration n’est pas prise en charge. [Arrêtez votre cluster](start-stop-cluster.md) à la place.

## <a name="can-i-use-custom-vm-extensions"></a>Puis-je utiliser des extensions de machine virtuelle personnalisées ?

L’agent Log Analytics est pris en charge, car il s’agit d’une extension gérée par Microsoft. Sinon, non, AKS est un service géré et la manipulation des ressources IaaS n’est pas prise en charge. Pour installer des composants personnalisés, utilisez les mécanismes et les API Kubernetes. Par exemple, utilisez des DaemonSets pour installer les composants nécessaires.

## <a name="does-aks-store-any-customer-data-outside-of-the-clusters-region"></a>AKS stocke-t-il des données client en dehors de la région du cluster ?

La fonctionnalité permettant le stockage de données client dans une seule région n’est actuellement disponible que dans la région Asie Sud-Est (Singapour) de la zone géographique Asie-Pacifique. Pour toutes les autres régions, les données client sont stockées dans Zone géographique.

## <a name="are-aks-images-required-to-run-as-root"></a>Les images AKS sont-elles nécessaires pour fonctionner en tant que racine ?

À l’exception des deux images suivantes, les images AKS ne sont pas requises pour fonctionner en tant que racine :

- *mcr.microsoft.com/oss/kubernetes/coredns*
- *mcr.microsoft.com/azuremonitor/containerinsights/ciprod*

## <a name="what-is-azure-cni-transparent-mode-vs-bridge-mode"></a>Qu’est-ce que le mode transparent d’Azure CNI par rapport au mode pont ?

À partir de la version 1.2.0, Azure CNI utilise le mode transparent comme valeur par défaut pour les déploiements de CNI sous Linux à location unique. Le mode transparent remplace le mode pont. Dans cette section, nous aborderons plus en détail les différences entre les deux modes et les avantages/limitations liés à l’utilisation du mode transparent dans Azure CNI.

### <a name="bridge-mode"></a>Mode pont

Comme son nom l’indique, le mode pont d’Azure CNI, en mode « juste-à-temps », crée un pont L2 nommé « azure0 ». Toutes les interfaces de paire `veth` des pods côté hôte seront connectées à ce pont. Par conséquent, la communication pod à pod interne à une machine virtuelle et le reste du trafic transitent par ce pont. Le pont en question est un appareil virtuel de couche 2 qui, à lui seul, ne peut rien recevoir ni transmettre, sauf si vous lui associez un ou plusieurs appareils réels. Pour cette raison, eth0 de la machine virtuelle Linux doit être converti en un pont subordonné à « azure0 ». Cela crée une topologie de réseau complexe au sein de la machine virtuelle Linux et, en tant que symptôme, CNI doit prendre en charge d’autres fonctions de mise en réseau, telles que la mise à jour du serveur DNS, etc.

:::image type="content" source="media/faq/bridge-mode.png" alt-text="Topologie en mode pont":::

Vous trouverez ci-dessous un exemple de la configuration de l’itinéraire IP en mode pont. Quel que soit le nombre de pods dont dispose le nœud, il n’y aura jamais que deux itinéraires. Le premier indique que tout le trafic, à l’exception du trafic local sur azure0, accède à la passerelle par défaut du sous-réseau par le biais de l’interface ayant l’adresse IP « src 10.240.0.4 » (qui est l’adresse IP principale du nœud), tandis que le second indique un trafic de l’espace de pod « 10.20.x.x » au noyau pour que le noyau décide.

```bash
default via 10.240.0.1 dev azure0 proto dhcp src 10.240.0.4 metric 100
10.240.0.0/12 dev azure0 proto kernel scope link src 10.240.0.4
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
root@k8s-agentpool1-20465682-1:/#
```

### <a name="transparent-mode"></a>Mode transparent
Le mode transparent adopte une approche directe pour configurer la mise en réseau de Linux. Dans ce mode, Azure CNI ne modifie pas les propriétés de l’interface eth0 dans la machine virtuelle Linux. Cette approche minimale de la modification des propriétés de mise en réseau de Linux permet de réduire les problèmes complexes de « corner case » (cas pathologiques) que les clusters pourraient rencontrer avec le mode pont. En mode transparent, Azure CNI crée et ajoute des interfaces de paire `veth` des pods côté hôte qui seront ajoutées au réseau hôte. La communication pod à pod interne à une machine virtuelle se fait via des itinéraires IP que CNI ajoutera. Essentiellement, la communication pod à pod se fait sur la couche 3, et le trafic des pods est acheminé par les règles d’acheminement L3.

:::image type="content" source="media/faq/transparent-mode.png" alt-text="Topologie en mode transparent":::

Vous trouverez ci-dessous un exemple de configuration d’itinéraire IP du mode transparent, l’interface de chaque pod obtenant un itinéraire statique afin que le trafic ayant la même adresse IP de destination que le pod soit envoyé directement à l’interface de la paire `veth` coté hôte du pod.

```bash
10.240.0.216 dev azv79d05038592 proto static
10.240.0.218 dev azv8184320e2bf proto static
10.240.0.219 dev azvc0339d223b9 proto static
10.240.0.222 dev azv722a6b28449 proto static
10.240.0.223 dev azve7f326f1507 proto static
10.240.0.224 dev azvb3bfccdd75a proto static
168.63.129.16 via 10.240.0.1 dev eth0 proto dhcp src 10.240.0.4 metric 100
169.254.169.254 via 10.240.0.1 dev eth0 proto dhcp src 10.240.0.4 metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
```

### <a name="benefits-of-transparent-mode"></a>Avantages du mode transparent

- Permet d’atténuer la condition de concurrence parallèle au DNS `conntrack` et d’éviter les problèmes de latence de cinq secondes du DNS sans qu’il soit nécessaire de configurer un DNS local de nœud (vous pouvez toujours utiliser un DNS local de nœud pour des raisons de performance).
- Élimine le mode pont de CNI à latence DNS initiale de cinq secondes présenté ici en raison de la configuration du pont « juste-à-temps ».
- L’un des corner case en mode pont est qu’Azure CNI ne peut pas continuer à mettre à jour les listes de serveurs DNS personnalisés que les utilisateurs ajoutent à un réseau virtuel ou à une carte réseau. Cela a pour conséquence que CNI ne prend en compte que la première instance de la liste de serveurs DNS. Résolu en mode transparent, car CNI ne modifie pas les propriétés eth0. [En savoir plus](https://github.com/Azure/azure-container-networking/issues/713).
- Permet de mieux gérer le trafic UDP et d’atténuer les tempêtes par saturation UDP lorsque le délai d’attente du protocole ARP expire. En mode pont, lorsque le pont ne connaît pas l’adresse MAC du pod de destination dans la communication pod à pod interne à la machine virtuelle, il en résulte, de par sa conception, une tempête de paquets envoyés vers tous les ports. Résolu en mode transparent, car il n’y a pas d’appareils L2 sous le chemin d’accès. [En savoir plus](https://github.com/Azure/azure-container-networking/issues/704).
- Le mode transparent est plus performant dans la communication pod à pod interne à une machine virtuelle en matière de débit et de latence par rapport au mode pont.


<!-- LINKS - internal -->

[aks-upgrade]: ./upgrade-cluster.md
[aks-cluster-autoscale]: ./cluster-autoscaler.md
[aks-advanced-networking]: ./configure-azure-cni.md
[aks-rbac-aad]: ./azure-ad-integration-cli.md
[node-updates-kured]: node-updates-kured.md
[aks-preview-cli]: /cli/azure/ext/aks-preview/aks
[az-aks-create]: /cli/azure/aks#az-aks-create
[aks-rm-template]: /azure/templates/microsoft.containerservice/2019-06-01/managedclusters
[aks-cluster-autoscaler]: cluster-autoscaler.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[aks-windows-cli]: windows-container-cli.md
[aks-windows-limitations]: ./windows-faq.md
[reservation-discounts]:../cost-management-billing/reservations/save-compute-costs-reservations.md
[api-server-authorized-ip-ranges]: ./api-server-authorized-ip-ranges.md
[multi-node-pools]: ./use-multiple-node-pools.md
[availability-zones]: ./availability-zones.md
[private-clusters]: ./private-clusters.md
[bcdr-bestpractices]: ./operator-best-practices-multi-region.md#plan-for-multiregion-deployment
[availability-zones]: ./availability-zones.md
[az-regions]: ../availability-zones/az-region.md
[uptime-sla]: ./uptime-sla.md

<!-- LINKS - external -->
[aks-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[auto-scaler]: https://github.com/kubernetes/autoscaler
[cordon-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
[admission-controllers]: https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/
[private-clusters-github-issue]: https://github.com/Azure/AKS/issues/948
[csi-driver]: https://github.com/Azure/secrets-store-csi-driver-provider-azure
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
