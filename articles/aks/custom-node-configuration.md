---
title: Personnaliser la configuration des nœuds pour les pools de nœuds Azure Kubernetes service (AKS) [préversion]
description: Découvrez comment personnaliser la configuration sur les nœuds de cluster Azure Kubernetes service (AKS) et les pools de nœuds.
ms.service: container-service
ms.topic: article
ms.date: 12/03/2020
ms.author: jpalma
author: palma21
ms.openlocfilehash: f1e9d65baacb9c712b92ef6f00abda169031b47e
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580956"
---
# <a name="customize-node-configuration-for-azure-kubernetes-service-aks-node-pools-preview"></a>Personnaliser la configuration des nœuds pour les pools de nœuds Azure Kubernetes service (AKS) [préversion]

La personnalisation de la configuration de votre nœud vous permet de configurer ou de régler les paramètres de votre système d’exploitation ou les paramètres kubelet pour répondre aux besoins des charges de travail. Lorsque vous créez un cluster AKS ou ajoutez un pool de nœuds à votre cluster, vous pouvez personnaliser un sous-ensemble de paramètres de système d’exploitation et kubelet couramment utilisés. Pour configurer des paramètres au-delà de ce sous-ensemble, [utilisez un ensemble de démons pour personnaliser vos configurations nécessaires sans perdre la prise en charge AKS pour vos nœuds](support-policies.md#shared-responsibility).

## <a name="register-the-customnodeconfigpreview-preview-feature"></a>Inscrire la fonctionnalité d’évaluation `CustomNodeConfigPreview`

Pour créer un cluster AKS ou un pool de nœuds qui peut personnaliser les paramètres kubelet ou les paramètres de système d’exploitation, vous devez activer l’indicateur de fonctionnalité `CustomNodeConfigPreview` sur votre abonnement.

Inscrivez l’indicateur de fonctionnalité `CustomNodeConfigPreview` à l’aide de la commande [az feature register][az-feature-register], comme indiqué dans l’exemple suivant :

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "CustomNodeConfigPreview"
```

Quelques minutes sont nécessaires pour que l’état s’affiche *Registered* (Inscrit). Vérifiez l’état de l’inscription à l’aide de la commande [az feature list][az-feature-list] :

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/CustomNodeConfigPreview')].{Name:name,State:properties.state}"
```

Lorsque vous êtes prêt, actualisez l’inscription du fournisseur de ressources *Microsoft.ContainerService* à l’aide de la commande [az provider register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="install-aks-preview-cli-extension"></a>Installer l’extension CLI de préversion d’aks

Pour créer un cluster AKS ou un pool de nœuds qui peut personnaliser les paramètres kubelet ou les paramètres du système d’exploitation, vous avez besoin de l’extension Azure CLI *aks-preview* la plus récente. Installez l’extension d’Azure CLI *aks-preview* à l’aide de la commande [az extension add][az-extension-add]. Ou installez toutes les mises à jour disponibles à l’aide de la commande [az extension update][az-extension-update].

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
``` 

## <a name="use-custom-node-configuration"></a>Utiliser la configuration de nœuds personnalisée

### <a name="kubelet-custom-configuration"></a>Configuration personnalisée kubelet

Les paramètres kubelet pris en charge et les valeurs acceptées sont répertoriés ci-dessous.

| Paramètre | Valeurs autorisées/intervalle | Default | Description |
| --------- | ----------------------- | ------- | ----------- |
| `cpuManagerPolicy` | none, static | aucun | La stratégie statique permet aux conteneurs se trouvant dans des [pods garantis](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/) avec des requêtes d’UC entières d’accéder aux UC exclusives sur le nœud. |
| `cpuCfsQuota` | true, false | true |  Active/désactive la mise en application du quota CFS de l’UC pour les conteneurs qui spécifient des limites d’UC. | 
| `cpuCfsQuotaPeriod` | intervalle en millisecondes (ms) | `100ms` | Définit la valeur de la période du quota CFS de l’UC. | 
| `imageGcHighThreshold` | 0-100 | 85 % | Pourcentage d’utilisation du disque après lequel le nettoyage de la mémoire d’images est toujours exécuté. Utilisation minimale du disque qui **déclenchera** le nettoyage de la mémoire. Pour désactiver le nettoyage de la mémoire d’images, définissez sur 100. | 
| `imageGcLowThreshold` | 0 à 100, pas plus que `imageGcHighThreshold` | 80 | Pourcentage d’utilisation du disque avant lequel le nettoyage de la mémoire d’images n’est jamais exécuté. Utilisation minimale du disque qui **peut** déclencher le nettoyage de la mémoire. |
| `topologyManagerPolicy` | none, best-effort, restricted, single-numa-node | aucun | Optimisez l’alignement des nœuds NUMA. En savoir plus [ici](https://kubernetes.io/docs/tasks/administer-cluster/topology-manager/). Uniquement kubernetes v1.18 et versions ultérieures. |
| `allowedUnsafeSysctls` | `kernel.shm*`, `kernel.msg*`, `kernel.sem`, `fs.mqueue.*`, `net.*` | Aucun | Liste autorisée de modèles sysctls ou sysctl non sécurisés. | 

### <a name="linux-os-custom-configuration"></a>Configuration personnalisée du système d’exploitation Linux

Les paramètres de système d’exploitation pris en charge et les valeurs acceptées sont répertoriés ci-dessous.

#### <a name="file-handle-limits"></a>Limites de traitement des fichiers

Lorsque le trafic est important, il est fréquent que le trafic que vous desservez provienne d’un grand nombre de fichiers locaux. Vous pouvez modifier les paramètres du noyau et les limites intégrées ci-dessous pour vous permettre d’en gérer davantage, au détriment d’une partie de la mémoire système.

| Paramètre | Valeurs autorisées/intervalle | Default | Description |
| ------- | ----------------------- | ------- | ----------- |
| `fs.file-max` | 8192 à 12000500 | 709620 | Nombre maximal de traitements de fichiers que le noyau Linux allouera. En augmentant cette valeur, vous pouvez augmenter le nombre maximal de fichiers ouverts autorisés. |
| `fs.inotify.max_user_watches` | 781250 à 2097152 | 1 048 576 | Nombre maximal d’espions de fichiers autorisés par le système. Chaque *espion* fait approximativement 90 octets sur un noyau de 32 bits, et environ 160 octets sur un noyau de 64 bits. | 
| `fs.aio-max-nr` | 65536 à 6553500 | 65536 | aio-nr indique le nombre actuel de requêtes io asynchrones à l’échelle du système. aio-max-nr vous permet de modifier la valeur maximale qu’aio-nr peut atteindre. |
| `fs.nr_open` | 8192 à 20000500 | 1 048 576 | Nombre maximal de traitements de fichier pouvant être alloués par un processus. |


#### <a name="socket-and-network-tuning"></a>Réglage du socket et du réseau

Pour les nœuds d’agent, qui sont censés gérer un très grand nombre de sessions simultanées, vous pouvez utiliser le sous-ensemble d’options TCP et réseau ci-dessous, que vous pouvez ajuster par pool de nœuds. 

| Paramètre | Valeurs autorisées/intervalle | Default | Description |
| ------- | ----------------------- | ------- | ----------- |
| `net.core.somaxconn` | 4096 à 3240000 | 16384 | Nombre maximal de demandes de connexion qui peuvent être mises en file d’attente pour un socket d’écoute donné. Limite supérieure pour la valeur du paramètre backlog passé à la fonction [listen(2)](http://man7.org/linux/man-pages/man2/listen.2.html). Si l’argument du backlog est supérieur à `somaxconn`, il est tronqué en mode silencieux à cette limite.
| `net.core.netdev_max_backlog` | 1000 à 3240000 | 1 000 | Nombre maximal de paquets, mis en file d’attente côté ENTRÉE, lorsque l’interface reçoit des paquets plus rapidement que le noyau ne peut les traiter. |
| `net.core.rmem_max` | 212992 à 134217728 | 212992 | Taille maximale de la mémoire tampon du socket de réception, en octets. |
| `net.core.wmem_max` | 212992 à 134217728 | 212992 | Taille maximale de la mémoire tampon du socket d’envoi, en octets. | 
| `net.core.optmem_max` | 20480 à 4194304 | 20480 | Taille maximale de la mémoire tampon auxiliaire (mémoire tampon facultative) autorisée par socket. La mémoire facultative du socket est utilisée dans certains cas pour stocker des structures supplémentaires relatives à l’utilisation du socket. | 
| `net.ipv4.tcp_max_syn_backlog` | 128 à 3240000 | 16384 | Nombre maximal de demandes de connexion mises en file d’attente qui n’ont toujours pas reçu d’accusé de réception de la part du client qui se connecte. Si ce nombre est dépassé, le noyau commence à annuler les demandes. |
| `net.ipv4.tcp_max_tw_buckets` | 8000 à 1440000 | 32 768 | Nombre maximal de sockets `timewait` détenus simultanément par le système. Si ce nombre est dépassé, le socket time-wait est immédiatement détruit et l’avertissement est imprimé. |
| `net.ipv4.tcp_fin_timeout` | 5 à 120 | 60 | Durée pendant laquelle une connexion orpheline (qui n’est plus référencée par une application) restera à l’état FIN_WAIT_2 avant d’être abandonnée à l’extrémité locale. |
| `net.ipv4.tcp_keepalive_time` | 30 à 432000 | 7200 | Fréquence à laquelle TCP envoie des messages `keepalive` lorsque `keepalive` est activée. |
| `net.ipv4.tcp_keepalive_probes` | 1 à 15 | 9 | Nombre de sondes `keepalive` que TCP envoie, jusqu’à ce qu’il décide que la connexion est rompue. |
| `net.ipv4.tcp_keepalive_intvl` | 1 à 75 | 75 | Fréquence à laquelle les sondes sont envoyées. Multipliée par `tcp_keepalive_probes`, elle constitue le temps nécessaire pour tuer une connexion qui ne répond pas, après le démarrage des sondes. | 
| `net.ipv4.tcp_tw_reuse` | 0 ou 1 | 0 | Permet de réutiliser les sockets `TIME-WAIT` pour les nouvelles connexions lorsque cela est sûr du point de vue du protocole. | 
| `net.ipv4.ip_local_port_range` | Premier : 1024 à 60999 et Dernier : 32768 à 65000 | Premier : 32768 et Dernier : 60999 | Plage de ports locaux utilisée par le trafic TCP et UDP pour choisir le port local. Composée de deux nombres : Le premier numéro est le premier port local autorisé pour le trafic TCP et UDP sur le nœud de l’agent, le deuxième est le dernier numéro de port local. | 
| `net.ipv4.neigh.default.gc_thresh1`|  128 à 80000 | 4096 | Nombre minimal d’entrées pouvant figurer dans le cache ARP. Le nettoyage de la mémoire ne sera pas déclenché si le nombre d’entrées est inférieur à ce paramètre. | 
| `net.ipv4.neigh.default.gc_thresh2`|  512 à 90000 | 8 192 | Nombre maximum relatif d’entrées pouvant figurer dans le cache ARP. Ce paramètre est sans doute le plus important, car le nettoyage de la mémoire ARP sera déclenché environ cinq secondes après avoir atteint ce maximum relatif. |
| `net.ipv4.neigh.default.gc_thresh3`|  1024 à 100000 | 16384 | Nombre maximum absolu d’entrées pouvant figurer dans le cache ARP. |
| `net.netfilter.nf_conntrack_max` | 131072 à 589824 | 131 072 | `nf_conntrack` est un module qui effectue le suivi des entrées de connexion pour NAT dans Linux. Le module `nf_conntrack` utilise une table de hachage pour enregistrer la ligne de la *connexion établie* du protocole TCP. `nf_conntrack_max` est le nombre maximal de nœuds dans la table de hachage, autrement dit, le nombre maximal de connexions prises en charge par le module `nf_conntrack` ou la taille de la table de suivi des connexions. | 
| `net.netfilter.nf_conntrack_buckets` | 65536 à 147456 | 65536 | `nf_conntrack` est un module qui effectue le suivi des entrées de connexion pour NAT dans Linux. Le module `nf_conntrack` utilise une table de hachage pour enregistrer la ligne de la *connexion établie* du protocole TCP. `nf_conntrack_buckets` est la taille de la table de hachage. | 

#### <a name="worker-limits"></a>Limites du rôle de travail

Tout comme les limites de descripteurs de fichiers, le nombre de rôles de travail ou de threads qu’un processus peut créer est limité à la fois par un paramètre de noyau et par des limites utilisateur. La limite utilisateur sur AKS est illimitée. 

| Paramètre | Valeurs autorisées/intervalle | Default | Description |
| ------- | ----------------------- | ------- | ----------- |
| `kernel.threads-max` | 20 à 513785 | 55601 | Les processus peuvent faire tourner des threads de travail. Le nombre maximal de tous les threads pouvant être créés est défini avec le paramètre de noyau `kernel.threads-max`. | 

#### <a name="virtual-memory"></a>Mémoire virtuelle

Les paramètres ci-dessous peuvent être utilisés pour régler le fonctionnement du sous-système de mémoire virtuelle du noyau Linux et le `writeout` de données incorrectes sur le disque.

| Paramètre | Valeurs autorisées/intervalle | Default | Description |
| ------- | ----------------------- | ------- | ----------- |
| `vm.max_map_count` |  65530 à 262144 | 65530 | Ce fichier contient le nombre maximal de zones de carte mémoire qu’un processus peut avoir. Les zones de carte mémoire sont utilisées comme un effet secondaire de l’appel de `malloc`, directement par `mmap`, `mprotect` et `madvise`, ainsi que lors du chargement de bibliothèques partagées. | 
| `vm.vfs_cache_pressure` | 1 à 500 | 100 | Cette valeur de pourcentage contrôle la tendance du noyau à récupérer la mémoire, qui est utilisée pour la mise en cache des objets inodes et du répertoire. |
| `vm.swappiness` | 0 - 100 | 60 | Ce contrôle permet de définir l’agressivité du noyau pour l’échange des pages de mémoire. Des valeurs plus élevées augmentent l’agressivité, des valeurs plus faibles diminuent la quantité d’échange. La valeur 0 indique au noyau de ne pas lancer l’échange tant que le nombre de pages libres et sauvegardées est inférieur à la limite supérieure d’une zone. | 
| `swapFileSizeMB` | 1 Mo : taille du [disque temporaire](../virtual-machines/managed-disks-overview.md#temporary-disk) (/dev/sdb) | Aucun | SwapFileSizeMB spécifie la taille en Mo d’un fichier d’échange qui sera créé sur les nœuds d’agent à partir de ce pool de nœuds. | 
| `transparentHugePageEnabled` | `always`, `madvise`, `never` | `always` | [Transparent Hugepages](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html#admin-guide-transhuge) est une fonctionnalité du noyau Linux destinée à améliorer les performances en optimisant l’utilisation du matériel de cartographie de la mémoire de votre processeur. Lorsque cette option est activée, le noyau tente d’allouer des `hugepages` chaque fois que cela est possible, et tout processus Linux reçoit des pages de 2 Mo si la région `mmap` est naturellement alignée sur 2 Mo. Dans certains cas, lorsque les `hugepages` sont activées pour l’ensemble du système, les applications peuvent finir par allouer plus de ressources mémoire. Une application peut `mmap` une grande région, mais ne toucher qu’un octet de celle-ci. Dans ce cas, une page de 2 Mo peut être allouée à la place d’une page de 4 ko sans raison valable. C’est pourquoi il est possible de désactiver les `hugepages` à l’échelle du système ou de ne les avoir qu’à l’intérieur des régions `MADV_HUGEPAGE madvise`. | 
| `transparentHugePageDefrag` | `always`, `defer`, `defer+madvise`, `madvise`, `never` | `madvise` | Cette valeur détermine si le noyau doit utiliser de manière intensive la compression de la mémoire pour rendre plus de `hugepages` disponibles. | 

> [!IMPORTANT]
> Pour faciliter la recherche et la lisibilité, les paramètres du système d’exploitation s’affichent dans ce document par leur nom, mais doivent être ajoutés au fichier config JSON ou à l’API AKS en utilisant la [convention de mise en majuscules camelCase](https://docs.microsoft.com/dotnet/standard/design-guidelines/capitalization-conventions).

Créez un fichier `kubeletconfig.json` avec le contenu suivant :

```json
{
 "cpuManagerPolicy": "static",
 "cpuCfsQuota": true,
 "cpuCfsQuotaPeriod": "200ms",
 "imageGcHighThreshold": 90,
 "imageGcLowThreshold": 70,
 "topologyManagerPolicy": "best-effort",
 "allowedUnsafeSysctls": [
  "kernel.msg*",
  "net.*"
],
 "failSwapOn": false
}
```
Créez un fichier `linuxosconfig.json` avec le contenu suivant :

```json
{
 "transparentHugePageEnabled": "madvise",
 "transparentHugePageDefrag": "defer+madvise",
 "swapFileSizeMB": 1500,
 "sysctls": {
  "netCoreSomaxconn": 163849,
  "netIpv4TcpTwReuse": true,
  "netIpv4IpLocalPortRange": "32000 60000"
 }
}
```

Créez un cluster en spécifiant les configurations kublet et du système d’exploitation à l’aide des fichiers JSON créés à l’étape précédente. 

> [!NOTE]
> Lorsque vous créez un cluster, vous pouvez spécifier la configuration kubelet, la configuration du système d’exploitation ou les deux. Si vous spécifiez une configuration lors de la création d’un cluster, cette configuration sera appliquée uniquement aux nœuds du pool de nœuds initial. Les paramètres qui ne sont pas configurés dans le fichier JSON conservent la valeur par défaut.

```azurecli
az aks create --name myAKSCluster --resource-group myResourceGroup --kubelet-config ./kubeletconfig.json --linux-os-config ./linuxosconfig.json
```

Ajoutez un nouveau pool de nœuds en spécifiant les paramètres kubelet à l’aide du fichier JSON que vous avez créé.

> [!NOTE]
> Lorsque vous ajoutez un pool de nœuds à un cluster existant, vous pouvez spécifier la configuration kubelet, la configuration du système d’exploitation ou les deux. Si vous spécifiez une configuration lors de l’ajout d’un pool de nœuds, cette configuration sera appliquée uniquement aux nœuds du nouveau pool de nœuds. Les paramètres qui ne sont pas configurés dans le fichier JSON conservent la valeur par défaut.

```azurecli
az aks nodepool add --name mynodepool1 --cluster-name myAKSCluster --resource-group myResourceGroup --kubelet-config ./kubeletconfig.json
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez [comment configurer votre cluster AKS](cluster-configuration.md).
- Découvrez comment [mettre à niveau les images de nœud](node-image-upgrade.md) dans votre cluster.
- Pour plus d’informations sur la mise à niveau de votre cluster vers la dernière version de Kubernetes, consultez [Mettre à niveau un cluster Azure Kubernetes Service (AKS)](upgrade-cluster.md).
- Consultez la liste des [questions fréquentes sur AKS](faq.md) pour trouver des réponses à certaines questions courantes concernant AKS.

<!-- LINKS - internal -->
[aks-faq]: faq.md
[aks-faq-node-resource-group]: faq.md#can-i-modify-tags-and-other-properties-of-the-aks-resources-in-the-node-resource-group
[aks-multiple-node-pools]: use-multiple-node-pools.md
[aks-scale-apps]: tutorial-kubernetes-scale.md
[aks-support-policies]: support-policies.md
[aks-upgrade]: upgrade-cluster.md
[aks-view-master-logs]: ./view-master-logs.md#enable-resource-logs
[autoscaler-profile-properties]: #using-the-autoscaler-profile
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-update]: /cli/azure/aks#az-aks-update
[az-aks-scale]: /cli/azure/aks#az-aks-scale
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[upgrade-cluster]: upgrade-cluster.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[max-surge]: upgrade-cluster.md#customize-node-surge-upgrade


<!-- LINKS - external -->
[az-aks-update-preview]: https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview
[az-aks-nodepool-update]: https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview#enable-cluster-auto-scaler-for-a-node-pool
[autoscaler-scaledown]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-types-of-pods-can-prevent-ca-from-removing-a-node
[autoscaler-parameters]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-the-parameters-to-ca
[kubernetes-faq]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#ca-doesnt-work-but-it-used-to-work-yesterday-why