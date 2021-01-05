---
title: Résoudre les problèmes courants liés à Azure Kubernetes Service
description: Découvrir comment résoudre les problèmes courants liés à l’utilisation d’AKS (Azure Kubernetes Service)
services: container-service
ms.topic: troubleshooting
ms.date: 06/20/2020
ms.openlocfilehash: 02ff1581bafe62a092be87d16df51c7ca8c020b4
ms.sourcegitcommit: beacda0b2b4b3a415b16ac2f58ddfb03dd1a04cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97830936"
---
# <a name="aks-troubleshooting"></a>Résolution des problèmes liés à AKS

Lorsque vous créez ou gérez des clusters Azure Kubernetes Service (AKS), vous pouvez occasionnellement rencontrer des problèmes. Cet article décrit en détail certains problèmes courants, et indique comment les résoudre.

## <a name="in-general-where-do-i-find-information-about-debugging-kubernetes-problems"></a>En règle générale, où trouver des informations sur le débogage de problèmes liés à Kubernetes ?

Essayez de vous référer au [guide officiel de résolution des problèmes de clusters Kubernetes](https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/).
Il existe également un [guide de résolution des problèmes](https://github.com/feiskyer/kubernetes-handbook/blob/master/en/troubleshooting/index.md) publié par un ingénieur Microsoft, portant sur le dépannage de pods, nœuds, clusters et autres fonctionnalités.

## <a name="im-getting-a-quota-exceeded-error-during-creation-or-upgrade-what-should-i-do"></a>J’obtiens une erreur de dépassement de quota pendant une opération de création ou de mise à niveau. Que dois-je faire ? 

 [Demandez plus de cœurs](../azure-portal/supportability/resource-manager-core-quotas-request.md).

## <a name="im-getting-an-insufficientsubnetsize-error-while-deploying-an-aks-cluster-with-advanced-networking-what-should-i-do"></a>J’obtiens un erreur insufficientSubnetSize quand je déploie un cluster AKS avec des fonctionnalités réseau avancées. Que dois-je faire ?

Cette erreur indique qu’un sous-réseau utilisé pour un cluster n’a plus d’adresses IP disponibles dans son routage CIDR (Classless InterDomain Routing) pour que l’attribution des ressources réussisse. Pour les clusters Kubenet, un espace d’adressage IP suffisant pour chaque nœud du cluster est exigé. Pour les clusters Azure CNI, un espace d’adressage IP suffisant pour chaque nœud et pod du cluster est exigé.
En savoir plus sur la [conception d’Azure CNI pour attribuer des adresses IP aux pods](configure-azure-cni.md#plan-ip-addressing-for-your-cluster).

Ces erreurs sont également signalées dans [AKS Diagnostics](concepts-diagnostics.md), qui prend l’initiative de signaler des problèmes tels qu’une taille de sous-réseau insuffisante.

Les trois (3) cas suivants entraînent une erreur de taille de sous-réseau insuffisante :

1. AKS Scale ou AKS Node pool scale
   1. Si vous utilisez Kubenet, lorsque le `number of free IPs in the subnet` est **inférieur** à `number of new nodes requested`.
   1. Si vous utilisez Azure CNI, lorsque le `number of free IPs in the subnet` est **inférieur** à `number of nodes requested times (*) the node pool's --max-pod value`.

1. AKS Upgrade ou AKS Node pool upgrade
   1. Si vous utilisez Kubenet, lorsque le `number of free IPs in the subnet` est **inférieur** au `number of buffer nodes needed to upgrade`.
   1. Si vous utilisez Azure CNI, lorsque le `number of free IPs in the subnet` est **inférieur** à `number of buffer nodes needed to upgrade times (*) the node pool's --max-pod value`.
   
   Par défaut, les clusters AKS définissent une valeur max-surge (tampon de mise à niveau) d’un (1), mais ce comportement de mise à niveau peut être personnalisé en définissant la valeur max-surge d’un pool de nœuds, ce qui augmentera le nombre d’adresses IP disponibles nécessaires à la mise à niveau.

1. AKS create ou AKS Node pool add
   1. Si vous utilisez Kubenet, lorsque le `number of free IPs in the subnet` est **inférieur** à `number of nodes requested for the node pool`.
   1. Si vous utilisez Azure CNI, lorsque le `number of free IPs in the subnet` est **inférieur** à `number of nodes requested times (*) the node pool's --max-pod value`.

L’atténuation suivante peut être effectuée en créant de nouveaux sous-réseaux. L’autorisation de créer un sous-réseau est requise pour l’atténuation en raison de l’impossibilité de mettre à jour la plage CIDR d’un sous-réseau existant.

1. Pour reconstruire un nouveau sous-réseau avec une plus grande plage CIDR suffisante pour les objectifs de l’opération :
   1. Créez un nouveau sous-réseau avec la nouvelle plage sans chevauchement souhaitée.
   1. Créez un nouveau pool de nœuds sur le nouveau sous-réseau.
   1. Drainez les pods de l’ancien pool de nœuds résidant dans l’ancien sous-réseau à remplacer.
   1. Supprimez l’ancien sous-réseau et l’ancien pool de nœuds.

## <a name="my-pod-is-stuck-in-crashloopbackoff-mode-what-should-i-do"></a>Mon pod est bloqué en mode CrashLoopBackOff. Que dois-je faire ?

Il peut y avoir diverses raisons pour que le pod soit bloqué dans ce mode. Vous pourriez examiner :

* Le pod lui-même, en utilisant `kubectl describe pod <pod-name>`.
* Les journaux d’activité en utilisant `kubectl logs <pod-name>`.

Pour plus d’informations sur la façon de résoudre les problèmes de pod, voir [Déboguer des applications](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/#debugging-pods).

## <a name="im-receiving-tcp-timeouts-when-using-kubectl-or-other-third-party-tools-connecting-to-the-api-server"></a>Je reçois `TCP timeouts` lors de l’utilisation de `kubectl` ou d’autres outils tiers qui se connectent au serveur d’API.
AKS a des plans de contrôle de haute disponibilité qui sont mis à l’échelle verticalement en fonction du nombre de cœurs pour garantir ses objectifs de niveau de service (SLO) et ses contrats de niveau de service (SLA). Si le délai de connexion est dépassé, vérifiez les éléments suivants :

- **Toutes vos commandes API expirent-elles régulièrement ou seulement quelques-unes ?** S’il ne s’agit que de quelques-unes, votre pod `tunnelfront` ou `aks-link`, responsable de la communication entre le nœud et le plan de contrôle, n’est peut-être pas en état de marche. Assurez-vous que les nœuds qui hébergent ce pod ne sont pas surutilisés ou soumis à un stress. Envisagez de les déplacer vers leur propre [pool de nœuds `system`](use-system-pools.md).
- **Avez-vous ouvert tous les ports, noms de domaine complets et adresses IP requis indiqués dans les [documents de restriction du trafic de sortie d’AKS](limit-egress-traffic.md) ?** Sinon, plusieurs appels de commandes peuvent échouer.
- **Votre adresse IP actuelle est-elle couverte par les [plages autorisées d’adresses IP d’API](api-server-authorized-ip-ranges.md) ?** Si vous utilisez cette fonctionnalité et que votre adresse IP n’est pas incluse dans les plages, vos appels seront bloqués. 
- **Un client ou une application laissent-ils filtrer des appels au serveur d’API ?** Veillez à utiliser des espions plutôt que des appels GET fréquents et que vos applications tierces ne laissent pas filtrer de tels appels. Par exemple, un bogue dans le mélangeur Istio entraîne la création d’une nouvelle connexion de l’espion au serveur d’API à chaque fois qu’un secret est lu en interne. Étant donné que ce comportement se produit à intervalles réguliers, les connexions de l’espion s’accumulent rapidement et finissent par surcharger le serveur d’API, quel que soit le modèle de mise à l’échelle. https://github.com/istio/istio/issues/19481
- **Avez-vous de nombreuses versions dans vos déploiements Helm ?** Ce scénario peut amener les deux tillers à utiliser trop de mémoire sur les nœuds, ainsi qu’une grande quantité de `configmaps`, ce qui peut entraîner des pics inutiles sur le serveur d’API. Envisagez de configurer `--history-max` sur `helm init` et tirez parti du nouveau Helm 3. Plus d’informations sur les problèmes suivants : 
    - https://github.com/helm/helm/issues/4821
    - https://github.com/helm/helm/issues/3500
    - https://github.com/helm/helm/issues/4543
- **[Le trafic interne entre les nœuds est-il bloqué ?](#im-receiving-tcp-timeouts-such-as-dial-tcp-node_ip10250-io-timeout)**

## <a name="im-receiving-tcp-timeouts-such-as-dial-tcp-node_ip10250-io-timeout"></a>Je reçois un message `TCP timeouts`, par exemple `dial tcp <Node_IP>:10250: i/o timeout`.

Ces délais peuvent être liés au blocage du trafic interne entre les nœuds. Vérifiez que ce trafic n’est pas bloqué, par exemple par [des groupes de sécurité réseau](concepts-security.md#azure-network-security-groups) sur le sous-réseau pour les nœuds de votre cluster.

## <a name="im-trying-to-enable-kubernetes-role-based-access-control-kubernetes-rbac-on-an-existing-cluster-how-can-i-do-that"></a>J’essaie d’activer le contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes) sur un cluster existant. Comment procéder ?

L’activation du contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes) sur des clusters existants n’est pas prise en charge actuellement. Elle doit être définie au moment de la création de clusters. Le RBAC Kubernetes est activé par défaut lors de l’utilisation de l’interface CLI, du portail ou d’une version d’API postérieure à `2020-03-01`.

## <a name="i-cant-get-logs-by-using-kubectl-logs-or-i-cant-connect-to-the-api-server-im-getting-error-from-server-error-dialing-backend-dial-tcp-what-should-i-do"></a>Je ne parviens pas à obtenir les journaux d’activité à l’aide des journaux d’activité de kubectl, ou à me connecter au serveur API. J’obtiens le message « Error from server: error dialing backend: dial tcp… ». Que dois-je faire ?

Vérifiez que les ports 22, 9000 et 1194 sont ouverts pour la connexion au serveur d’API. Vérifiez que le pod `tunnelfront` ou `aks-link` s’exécute dans l’espace de noms *kube-system* à l’aide de la commande `kubectl get pods --namespace kube-system`. Si ce n’est pas le cas, forcez la suppression du pod pour qu’il redémarre.

## <a name="im-getting-tls-client-offered-only-unsupported-versions-from-my-client-when-connecting-to-aks-api-what-should-i-do"></a>Je reçois un message `"tls: client offered only unsupported versions"` de la part de mon client lors de la connexion à l’API AKS. Que dois-je faire ?

La version TLS minimale prise en charge dans AKS est TLS 1.2.

## <a name="im-trying-to-upgrade-or-scale-and-am-getting-a-changing-property-imagereference-is-not-allowed-error-how-do-i-fix-this-problem"></a>J’essaie d’effectuer une mise à niveau ou une mise à l’échelle et j’obtiens l’erreur `"Changing property 'imageReference' is not allowed"`. Comment résoudre ce problème ?

Cette erreur s’affiche peut-être parce que vous avez modifié les balises dans les nœuds d’agent à l’intérieur du cluster AKS. La modification ou la suppression de balises et d’autres propriétés de ressources dans le groupe de ressources MC_* peuvent entraîner des résultats inattendus. La modification des ressources du groupe MC_* dans le cluster AKS empêche d’atteindre l’objectif de niveau de service (SLO).

## <a name="im-receiving-errors-that-my-cluster-is-in-failed-state-and-upgrading-or-scaling-will-not-work-until-it-is-fixed"></a>Je reçois des erreurs qui indiquent que mon cluster est en état d’échec et que la mise à niveau ou la mise à l’échelle n’aboutira pas tant que ce problème n’aura pas été résolu

*Cette assistance de dépannage est redirigée depuis https://aka.ms/aks-cluster-failed*

Cette erreur se produit quand des clusters basculent dans un état d’échec pour plusieurs raisons. Suivez les étapes ci-dessous pour résoudre l’état d’échec de votre cluster avant de tenter à nouveau l’opération qui a échoué précédemment :

1. Tant que le cluster n’aura pas quitté l’état `failed`, les opérations `upgrade` et `scale` n’aboutiront pas. Voici quelques-unes des causes courantes à l’origine du problème ainsi que des solutions :
    * Mise à l’échelle avec un **quota de calcul (CRP) insuffisant**. Pour résoudre ce problème, remettez votre cluster dans un état stable dans les limites du quota. Suivez ensuite ces [étapes pour demander une augmentation du quota de calcul](../azure-portal/supportability/resource-manager-core-quotas-request.md) avant de tenter à nouveau un scale-up au-delà des limites du quota initial.
    * Mise à jour d’un cluster avec des fonctionnalités réseau avancées et des **ressources (réseau) de sous-réseau insuffisantes**. Pour résoudre ce problème, remettez votre cluster dans un état stable dans les limites du quota. Suivez ensuite ces [étapes pour demander une augmentation du quota de ressources](../azure-resource-manager/templates/error-resource-quota.md#solution) avant de tenter à nouveau un scale-up au-delà des limites du quota initial.
2. Une fois que la cause sous-jacente de l’échec de mise à niveau est résolue, votre cluster doit être dans un état de réussite. Une fois que l’état de réussite a été vérifié, tentez à nouveau l’opération d’origine.

## <a name="im-receiving-errors-when-trying-to-upgrade-or-scale-that-state-my-cluster-is-being-upgraded-or-has-failed-upgrade"></a>Je reçois des erreurs lors de la mise à niveau ou de la mise à l’échelle qui indiquent que mon cluster fait actuellement l’objet d’une mise à niveau ou que l’opération a échoué.

*Cette assistance de dépannage est redirigée depuis https://aka.ms/aks-pending-upgrade*

 Il ne peut pas y avoir de mise à niveau et de mise à l’échelle simultanées d’un cluster ou d’un pool de nœuds. En effet, chaque opération doit être terminée sur la ressource cible avant l’exécution de la demande suivante sur cette même ressource. De ce fait, les opérations sont limitées quand des opérations actives de mise à niveau ou de mise à l’échelle se produisent ou sont entreprises. 

Pour faciliter le diagnostic du problème, exécutez `az aks show -g myResourceGroup -n myAKSCluster -o table` pour récupérer l’état détaillé de votre cluster. Selon le résultat :

* Si le cluster est en cours de mise à niveau, patientez le temps que l’opération se termine. Si elle a abouti, tentez à nouveau l’opération qui a échoué auparavant.
* Si la mise la mise à niveau du cluster a échoué, suivez les étapes décrites dans la section précédente.

## <a name="can-i-move-my-cluster-to-a-different-subscription-or-my-subscription-with-my-cluster-to-a-new-tenant"></a>Puis-je déplacer mon cluster vers un autre abonnement ou mon abonnement et mon cluster vers un nouveau locataire ?

Si vous avez déplacé votre cluster AKS vers un autre abonnement ou l’abonnement du cluster vers un nouveau locataire, le cluster ne fonctionnera pas en raison de l’absence d’autorisations d’identité de cluster. **AKS ne prend pas en charge le déplacement de clusters entre abonnements ou locataires** à cause de cette contrainte.

## <a name="im-receiving-errors-trying-to-use-features-that-require-virtual-machine-scale-sets"></a>Je reçois des erreurs quand j’essaie d’utiliser des fonctionnalités qui nécessitent des groupes de machines virtuelles identiques

*Cette assistance de dépannage est redirigée depuis aka.ms/aks-vmss-enablement*

Vous pouvez recevoir des erreurs qui indiquent que votre cluster AKS n’appartient pas à un groupe de machines virtuelles identiques, comme dans l’exemple suivant :

**La mise à l’échelle automatique de l’AgentPool `<agentpoolname>` est activée, mais elle ne l’est pas sur Virtual Machine Scale Sets**

Les fonctionnalités telles que le programme de mise à l’échelle automatique de cluster ou plusieurs pools de nœuds nécessitent des groupes de machines virtuelles identiques en tant que `vm-set-type`.

Suivez les étapes *Avant de commencer* dans le document approprié pour créer correctement un cluster AKS :

* [Utiliser le composant Cluster Autoscaler](cluster-autoscaler.md)
* [Créer et utiliser plusieurs pools de nœuds](use-multiple-node-pools.md)
 
## <a name="what-naming-restrictions-are-enforced-for-aks-resources-and-parameters"></a>Quelles restrictions d’affectation de noms sont appliquées pour les paramètres et les ressources AKS ?

*Cette assistance de dépannage est redirigée depuis aka.ms/aks-naming-rules*

Les restrictions d’affectation de noms sont implémentées par la plateforme Azure et AKS. Si un nom ou paramètre de ressource enfreint une de ces restrictions, une erreur est retournée qui vous invite à fournir une entrée différente. Voici quelques-unes des recommandations qui s’appliquent en matière d’affectation de noms :

* Les noms de cluster doivent comporter entre 1 et 63 caractères. Les seuls caractères autorisés sont les lettres, les chiffres, les tirets et le trait de soulignement. Le premier et le dernier caractères doivent être une lettre ou un chiffre.
* Le nom de groupe de ressources Nœud AKS/*MC_* combine le nom du groupe de ressources et le nom de la ressource. La syntaxe générée automatiquement de `MC_resourceGroupName_resourceName_AzureRegion` ne doit pas dépasser 80 caractères. Si nécessaire, réduisez la longueur du nom de groupe de ressources ou du nom de cluster AKS. Vous pouvez également [personnaliser le nom de votre groupe de ressources de nœud](cluster-configuration.md#custom-resource-group-name).
* L'élément *dnsPrefix* doit commencer et se terminer par des valeurs alphanumériques et doit comporter 1 à 54 caractères. Parmi les caractères autorisés figurent les valeurs alphanumériques et les traits d’union (-). L’élément *dnsPrefix* ne peut pas inclure de caractères spéciaux comme un point (.).
* Les noms de pools de nœuds AKS doivent être en minuscules et comprendre 1 à 11 caractères pour les pools de nœuds Linux et 1 à 6 caractères pour les pools de nœuds Windows. Le nom doit commencer par une lettre et les seuls caractères autorisés sont les lettres et les chiffres.
* *admin-username*, qui définit le nom d’utilisateur de l’administrateur pour les nœuds Linux, doit commencer par une lettre, ne peut contenir que des lettres, des chiffres, des traits d’union et des traits de soulignement, et sa longueur maximale est de 64 caractères.

## <a name="im-receiving-errors-when-trying-to-create-update-scale-delete-or-upgrade-cluster-that-operation-is-not-allowed-as-another-operation-is-in-progress"></a>Je reçois une erreur quand j’essaie de créer, mettre à jour, mettre à l’échelle, supprimer ou mettre à niveau un cluster, m’informant que cette opération n’est pas autorisée quand une autre opération est en cours.

*Cette assistance de dépannage est redirigée depuis aka.ms/aks-pending-operation*

Les opérations de cluster sont limitées quand une opération précédente est toujours en cours. Pour récupérer un état détaillé de votre cluster, utilisez la commande `az aks show -g myResourceGroup -n myAKSCluster -o table`. Utilisez le nom de votre propre groupe de ressources et de votre propre cluster AKS, selon les besoins.

Selon la sortie de l’état du cluster :

* Si le cluster est dans un état d’approvisionnement autre que *Opération réussie* ou *En échec*, attendez que l’opération (*mise à niveau/mise à jour/création/mise à l’échelle/suppression/migration*) se termine. Une fois que l’opération précédente a abouti, tentez à nouveau votre dernière opération de cluster.

* Si la mise à niveau du cluster a échoué, suivez les étapes décrites dans [Je reçois des erreurs qui indiquent que mon cluster est en état d’échec et que la mise à niveau ou la mise à l’échelle n’aboutira pas tant que ce problème n’aura pas été résolu](#im-receiving-errors-that-my-cluster-is-in-failed-state-and-upgrading-or-scaling-will-not-work-until-it-is-fixed).

## <a name="received-an-error-saying-my-service-principal-wasnt-found-or-is-invalid-when-i-try-to-create-a-new-cluster"></a>Je reçois une erreur indiquant que mon principal de service est introuvable ou n’est pas valide lorsque j’essaie de créer un nouveau cluster.

Lors de la création d’un cluster AKS, un principal de service ou une identité managée doit créer des ressources en votre nom. AKS peut créer automatiquement un principal de service au moment de la création du cluster ou en recevoir un existant. Lors de l’utilisation d’un principal de service créé automatiquement, Azure Active Directory doit le propager dans chaque région pour que la création aboutisse. Si le processus de propagation prend trop de temps, la validation de la création du cluster échoue, car il ne trouve pas de principal de service disponible. 

Essayez les solutions de contournement suivantes pour résoudre le problème :
* Utilisez un principal de service existant qui a déjà été propagé dans les régions et qui peut être passé dans AKS au moment de la création du cluster.
* Si vous utilisez des scripts d’automatisation, augmentez les délais entre la création du principal de service et la création du cluster AKS.
* Si vous utilisez le portail Azure, revenez aux paramètres du cluster au moment de la création, puis recommencez la page de validation après quelques minutes.

## <a name="im-getting-aadsts7000215-invalid-client-secret-is-provided-when-using-aks-api-what-should-i-do"></a>Je reçois un message `"AADSTS7000215: Invalid client secret is provided."` lors de l’utilisation de l’API AKS. Que dois-je faire ?

Ce problème est dû à l’expiration des informations d’identification du principal de service. [Mettez à jour les informations d’identification d’un cluster AKS.](update-credentials.md)

## <a name="i-cant-access-my-cluster-api-from-my-automationdev-machinetooling-when-using-api-server-authorized-ip-ranges-how-do-i-fix-this-problem"></a>Je ne peux pas accéder à mon API de cluster depuis mon système d’automatisation/de développement/d’outils lorsque j’utilise des plages d’adresses IP autorisées par le serveur d’API. Comment résoudre ce problème ?

Pour résoudre ce problème, assurez-vous que `--api-server-authorized-ip-ranges` inclue les adresses IP ou les plages d’adresses IP des systèmes d’automatisation/de développement/d’outils utilisés. Reportez-vous à la section « Comment trouver mon adresse IP » dans [Sécuriser l’accès au serveur d’API à l’aide de plages d’adresses IP autorisées](api-server-authorized-ip-ranges.md).

## <a name="im-unable-to-view-resources-in-kubernetes-resource-viewer-in-azure-portal-for-my-cluster-configured-with-api-server-authorized-ip-ranges-how-do-i-fix-this-problem"></a>Je ne parviens pas à consulter les ressources dans la visionneuse de ressources Kubernetes du portail Azure pour mon cluster configuré avec des plages d’adresses IP autorisées par le serveur d’API. Comment résoudre ce problème ?

La [visionneuse de ressources Kubernetes](kubernetes-portal.md) requiert que `--api-server-authorized-ip-ranges` inclue l’accès pour l’ordinateur client local ou la plage d’adresses IP (à partir desquels le portail est parcouru). Reportez-vous à la section « Comment trouver mon adresse IP » dans [Sécuriser l’accès au serveur d’API à l’aide de plages d’adresses IP autorisées](api-server-authorized-ip-ranges.md).

## <a name="im-receiving-errors-after-restricting-egress-traffic"></a>Je reçois des erreurs après avoir restreint mon trafic de sortie.

Lors de la restriction du trafic sortant d’un cluster AKS, il existe des règles de ports de sortie/réseau ainsi que des règles de nom de domaine complet (FQDN)/application [obligatoires et facultatives recommandées](limit-egress-traffic.md) pour AKS. Si vos paramètres sont en conflit avec l’une de ces règles, certaines commandes `kubectl` ne fonctionneront pas correctement. Vous pouvez également voir des erreurs lors de la création d’un cluster AKS.

Vérifiez que vos paramètres ne sont pas en conflit avec l’une règles de ports de sortie/réseau ainsi que des règles de nom de domaine complet (FQDN)/application obligatoires ou facultatives recommandées.

## <a name="im-receiving-429---too-many-requests-errors"></a>Je reçois des erreurs « 429 Trop de requêtes »

Lorsqu’un cluster Kubernetes sur Azure (AKS ou non) effectue fréquemment un scale up ou scale-down ou utilise la mise à l’échelle automatique de cluster, ces opérations peuvent entraîner un grand nombre d’appels HTTP qui, à leur tour, dépassent le quota d’abonnement attribué, ce qui conduit à l’échec. Les erreurs se présentent comme suit

```
Service returned an error. Status=429 Code=\"OperationNotAllowed\" Message=\"The server rejected the request because too many requests have been received for this subscription.\" Details=[{\"code\":\"TooManyRequests\",\"message\":\"{\\\"operationGroup\\\":\\\"HighCostGetVMScaleSet30Min\\\",\\\"startTime\\\":\\\"2020-09-20T07:13:55.2177346+00:00\\\",\\\"endTime\\\":\\\"2020-09-20T07:28:55.2177346+00:00\\\",\\\"allowedRequestCount\\\":1800,\\\"measuredRequestCount\\\":2208}\",\"target\":\"HighCostGetVMScaleSet30Min\"}] InnerError={\"internalErrorCode\":\"TooManyRequestsReceived\"}"}
```

Ces erreurs de limitation sont décrites en détail [ici](../azure-resource-manager/management/request-limits-and-throttling.md) et [ici](../virtual-machines/troubleshooting/troubleshooting-throttling-errors.md).

La recommandation de l’équipe d’ingénieurs d’AKS est de vous assurer que vous utilisez au moins la version 1.18.x, qui contient de nombreuses améliorations. Vous trouverez plus d’informations sur ces améliorations [ici](https://github.com/Azure/AKS/issues/1413) et [ici](https://github.com/kubernetes-sigs/cloud-provider-azure/issues/247).

Étant donné que ces erreurs de limitation sont mesurées au niveau de l’abonnement, elles peuvent encore se produire dans les cas suivants :
- Il existe des applications tierces qui effectuent des requêtes GET (par exemple, des applications de surveillance, etc.). La recommandation est de réduire la fréquence de ces appels.
- Il existe de nombreux clusters AKS/pools de nœuds utilisant des groupes de machines virtuelles identiques. Essayez de fractionner votre nombre de clusters et des les répartir dans différents abonnements, en particulier si vous vous attendez à ce qu’ils soient très actifs (par exemple, un outil actif de mise à l’échelle automatique de cluster) ou qu’ils aient plusieurs clients (par exemple, Rancher, Terraform, etc.).

## <a name="my-clusters-provisioning-status-changed-from-ready-to-failed-with-or-without-me-performing-an-operation-what-should-i-do"></a>L’état de l’approvisionnement de mon cluster est passé de Prêt à Échec, que j’effectue une opération ou non. Que dois-je faire ?

Si l’état d’approvisionnement de votre cluster passe de *Prêt* à *Échec*, que vous effectuiez des opérations ou non, mais que les applications sur votre cluster continuent à s’exécuter, ce problème peut être résolu automatiquement par le service et vos applications ne doivent pas être affectées.

Si l’état d’approvisionnement de votre cluster reste en *Échec* ou si les applications sur votre cluster cessent de fonctionner, [envoyez une demande de support](https://azure.microsoft.com/support/options/#submit).

## <a name="my-watch-is-stale-or-azure-ad-pod-identity-nmi-is-returning-status-500"></a>Mon espion est obsolète ou Azure AD Pod Identity NMI retourne l’état 500

Si vous utilisez Pare-feu Azure comme dans cet [exemple](limit-egress-traffic.md#restrict-egress-traffic-using-azure-firewall), vous pouvez rencontrer ce problème, car les connexions TCP de longue durée via le pare-feu utilisant des règles d’application ont actuellement un bogue (à résoudre dans Q1CY21) qui provoque l’arrêt des `keepalives` Go sur le pare-feu. Tant que ce problème n’est pas résolu, vous pouvez l’atténuer en ajoutant une règle de réseau (au lieu de la règle d’application) à l’adresse IP du serveur de l’API AKS.

## <a name="azure-storage-and-aks-troubleshooting"></a>Résolution des problèmes de stockage Azure et AKS

### <a name="failure-when-setting-uid-and-gid-in-mountoptions-for-azure-disk"></a>Échec lors de la définition de l’UID et du `GID` dans mountOptions pour le disque Azure

Le disque Azure utilise par défaut le filesytem ext4,xfs, et des mountOptions telles que uid=x,gid=x ne peuvent pas être définies au moment du montage. Par exemple, si vous avez essayé de définir mountOptions uid=999,gid=999, une erreur semblable à la suivante s’affiche :

```console
Warning  FailedMount             63s                  kubelet, aks-nodepool1-29460110-0  MountVolume.MountDevice failed for volume "pvc-d783d0e4-85a1-11e9-8a90-369885447933" : azureDisk - mountDevice:FormatAndMount failed with mount failed: exit status 32
Mounting command: systemd-run
Mounting arguments: --description=Kubernetes transient mount for /var/lib/kubelet/plugins/kubernetes.io/azure-disk/mounts/m436970985 --scope -- mount -t xfs -o dir_mode=0777,file_mode=0777,uid=1000,gid=1000,defaults /dev/disk/azure/scsi1/lun2 /var/lib/kubelet/plugins/kubernetes.io/azure-disk/mounts/m436970985
Output: Running scope as unit run-rb21966413ab449b3a242ae9b0fbc9398.scope.
mount: wrong fs type, bad option, bad superblock on /dev/sde,
       missing codepage or helper program, or other error
```

Vous pouvez atténuer le problème en procédant de l’une des manières suivantes :

* [Configurez le contexte de sécurité pour un pod](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) en définissant uid dans runAsUser et gid dans fsGroup. Par exemple, le paramètre suivant définit l’exécution de pod en tant que root et le rend accessible à n’importe quel fichier :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 0
    fsGroup: 0
```

  >[!NOTE]
  > Comme gid et uid sont montés en tant que root ou 0 par défaut. Si gid ou uid sont définis comme non root, par exemple 1000, Kubernetes utilise `chown` pour modifier tous les répertoires et fichiers sous ce disque. Cette opération peut prendre du temps et peut ralentir considérablement le montage du disque.

* Utilisez `chown` dans initContainers pour définir `GID` et `UID`. Par exemple :

```yaml
initContainers:
- name: volume-mount
  image: busybox
  command: ["sh", "-c", "chown -R 100:100 /data"]
  volumeMounts:
  - name: <your data volume>
    mountPath: /data
```

### <a name="azure-disk-detach-failure-leading-to-potential-race-condition-issue-and-invalid-data-disk-list"></a>Échec du détachement de disque Azure menant à un problème potentiel de condition de concurrence et à une liste de disques de données non valides

En cas d’échec de détachement d’un disque Azure, une nouvelle tentative de détachement de disque est effectuée jusqu’à six fois à l’aide de l’interruption exponentielle. Un verrou est également maintenu au niveau du nœud sur la liste des disques de données pendant environ 3 minutes. Si la liste de disques est mise à jour manuellement pendant cette période, la liste de disques détenue par le verrou au niveau du nœud devient obsolète et entraîne une instabilité sur le nœud.

Ce problème a été résolu dans les versions suivantes de Kubernetes :

| Version de Kubernetes | Version corrigée |
|--|:--:|
| 1.12 | 1.12.9 ou version ultérieure |
| 1.13 | 1.13.6 ou version ultérieure |
| 1.14 | 1.14.2 ou version ultérieure |
| 1.15 et versions ultérieures | N/A |

Si vous utilisez une version de Kubernetes qui ne présente pas le correctif pour ce problème et que votre nœud présente une liste de disques obsolète, vous pouvez atténuer le problème en détachant tous les disques non existants de la machine virtuelle à l’aide d’une opération en bloc. **Le détachement individuel de disques non existants peut échouer.**

### <a name="large-number-of-azure-disks-causes-slow-attachdetach"></a>Un nombre élevé de disques Azure ralentit la procédure d’attachement/détachement

Lorsque le nombre d’opérations d’attachement ou de détachement de disques Azure ciblant une machine virtuelle à nœud unique est supérieur à 10, ou supérieur à 3 lorsqu’elles ciblent un pool de groupes de machines virtuelles identiques, elles peuvent être plus lentes que prévu, car elles sont effectuées de façon séquentielle. Ce problème est une limitation connue et il n’existe aucune solution de contournement pour l’instant. [Sondage UserVoice pour la prise en charge l’attachement ou le détachement parallèle au-delà du nombre.](https://feedback.azure.com/forums/216843-virtual-machines/suggestions/40444528-vmss-support-for-parallel-disk-attach-detach-for).

### <a name="azure-disk-detach-failure-leading-to-potential-node-vm-in-failed-state"></a>Échec du détachement de disque Azure menant à un état d’échec potentiel pour la machine virtuelle de nœud

Dans certains cas, le détachement d’un disque Azure peut échouer partiellement et laisser la machine virtuelle du nœud dans un état d’échec.

Ce problème a été résolu dans les versions suivantes de Kubernetes :

| Version de Kubernetes | Version corrigée |
|--|:--:|
| 1.12 | 1.12.10 ou version ultérieure |
| 1.13 | 1.13.8 ou version ultérieure |
| 1.14 | 1.14.4 ou version ultérieure |
| 1.15 et versions ultérieures | N/A |

Si vous utilisez une version de Kubernetes qui ne présente pas le correctif pour ce problème et que votre nœud est en échec, vous pouvez l’atténuer en mettant à jour manuellement l’état de la machine virtuelle à l’aide de l’une des opérations ci-dessous :

* Pour un cluster à base de groupes à haute disponibilité :
    ```azurecli
    az vm update -n <VM_NAME> -g <RESOURCE_GROUP_NAME>
    ```

* Pour un cluster basé sur VMSS :
    ```azurecli
    az vmss update-instances -g <RESOURCE_GROUP_NAME> --name <VMSS_NAME> --instance-id <ID>
    ```

## <a name="azure-files-and-aks-troubleshooting"></a>Résolution des problèmes Azure Files et AKS

### <a name="what-are-the-recommended-stable-versions-of-kubernetes-for-azure-files"></a>Quelles sont les versions stables recommandées de Kubernetes pour les fichiers Azure ?
 
| Version de Kubernetes | Version recommandée |
|--|:--:|
| 1.12 | 1.12.6 ou version ultérieure |
| 1.13 | 1.13.4 ou version ultérieure |
| 1.14 | 1.14.0 ou version ultérieure |

### <a name="what-are-the-default-mountoptions-when-using-azure-files"></a>Quelles sont les mountOptions par défaut lors de l’utilisation d’Azure Files ?

Paramètres recommandés :

| Version de Kubernetes | Valeur de fileMode et dirMode|
|--|:--:|
| 1.12.0 - 1.12.1 | 0755 |
| 1.12.2 et versions ultérieures | 0777 |

Les options de montage peuvent être spécifiées sur l’objet de classe de stockage. L’exemple suivant définit *0777* :

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azurefile
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=1000
  - gid=1000
  - mfsymlinks
  - nobrl
  - cache=none
parameters:
  skuName: Standard_LRS
```

Certains paramètres *mountOptions* supplémentaires utiles :

* `mfsymlinks` fait en sorte que le montage Azure Files (cifs) prenne en charge les liens symboliques
* `nobrl`  empêche l’envoi des demandes de verrous de plage d’octets au serveur. Ce paramètre est nécessaire pour certaines applications qui s’arrêtent avec des verrous de plage d’octets obligatoires de type cifs. La plupart des serveurs cifs ne prennent pas encore en charge la demande de verrous de plage d’octets. Si vous n’utilisez pas *nobrl*, les applications qui s’arrêtent avec des verrous de plage d’octets obligatoires de type cifs peuvent entraîner des messages d’erreur comme suit :
    ```console
    Error: SQLITE_BUSY: database is locked
    ```

### <a name="error-could-not-change-permissions-when-using-azure-files"></a>Erreur « Impossible de modifier les autorisations » lors de l’utilisation d’Azure Files

Quand vous exécutez PostgreSQL sur le plug-in Azure Files, vous pouvez obtenir une erreur semblable à la suivante :

```console
initdb: could not change permissions of directory "/var/lib/postgresql/data": Operation not permitted
fixing permissions on existing directory /var/lib/postgresql/data
```

Cette erreur est provoquée par le plug-in Azure Files utilisant le protocole cifs/SMB. Lors de l’utilisation du protocole cifs/SMB, les autorisations de fichiers et de répertoires n’ont pas pu être modifiées après le montage.

Pour résoudre ce problème, utilisez `subPath`  avec le plug-in de disque Azure. 

> [!NOTE] 
> Pour le type de disque ext3/4, il y a un répertoire des « objets trouvés » une fois le disque formaté.

### <a name="azure-files-has-high-latency-compared-to-azure-disk-when-handling-many-small-files"></a>Azure Files présente une latence élevée par rapport au disque Azure lors du traitement de nombreux petits fichiers

Dans certains cas, tels que la gestion de nombreux petits fichiers, vous pouvez rencontrer une latence élevée lors de l’utilisation d’Azure Files par rapport au disque Azure.

### <a name="error-when-enabling-allow-access-allow-access-from-selected-network-setting-on-storage-account"></a>Erreur lors de l’activation du paramètre « Autoriser l’accès à partir du réseau sélectionné » sur le compte de stockage

Si vous activez *Autoriser l’accès à partir du réseau sélectionné* sur un compte de stockage utilisé pour l’approvisionnement dynamique dans AKS, vous obtiendrez une erreur lorsqu’AKS crée un partage de fichiers :

```console
persistentvolume-controller (combined from similar events): Failed to provision volume with StorageClass "azurefile": failed to create share kubernetes-dynamic-pvc-xxx in account xxx: failed to create file share, err: storage: service returned error: StatusCode=403, ErrorCode=AuthorizationFailure, ErrorMessage=This request is not authorized to perform this operation.
```

Cette erreur est due au fait que le *persistentvolume-controller*  Kubernetes ne figure pas sur le réseau choisi lors de la configuration de l’option *Autoriser l’accès à partir du réseau sélectionné*.

Vous pouvez atténuer le problème à l’aide de [l’approvisionnement statique avec Azure Files](azure-files-volume.md).

### <a name="azure-files-fails-to-remount-in-windows-pod"></a>Le remontage d’Azure Files échoue dans le pod Windows

Si un pod Windows avec un montage Azure Files est supprimé, puis planifié pour être recréé sur le même nœud, le montage échoue. Cet échec est dû à l’échec de la commande `New-SmbGlobalMapping`, car le montage d’Azure Files est déjà effectué sur le nœud.

Par exemple, vous pouvez recevoir une erreur comme suit :

```console
E0118 08:15:52.041014    2112 nestedpendingoperations.go:267] Operation for "\"kubernetes.io/azure-file/42c0ea39-1af9-11e9-8941-000d3af95268-pvc-d7e1b5f9-1af3-11e9-8941-000d3af95268\" (\"42c0ea39-1af9-11e9-8941-000d3af95268\")" failed. No retries permitted until 2019-01-18 08:15:53.0410149 +0000 GMT m=+732.446642701 (durationBeforeRetry 1s). Error: "MountVolume.SetUp failed for volume \"pvc-d7e1b5f9-1af3-11e9-8941-000d3af95268\" (UniqueName: \"kubernetes.io/azure-file/42c0ea39-1af9-11e9-8941-000d3af95268-pvc-d7e1b5f9-1af3-11e9-8941-000d3af95268\") pod \"deployment-azurefile-697f98d559-6zrlf\" (UID: \"42c0ea39-1af9-11e9-8941-000d3af95268\") : azureMount: SmbGlobalMapping failed: exit status 1, only SMB mount is supported now, output: \"New-SmbGlobalMapping : Generic failure \\r\\nAt line:1 char:190\\r\\n+ ... ser, $PWord;New-SmbGlobalMapping -RemotePath $Env:smbremotepath -Cred ...\\r\\n+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\r\\n    + CategoryInfo          : NotSpecified: (MSFT_SmbGlobalMapping:ROOT/Microsoft/...mbGlobalMapping) [New-SmbGlobalMa \\r\\n   pping], CimException\\r\\n    + FullyQualifiedErrorId : HRESULT 0x80041001,New-SmbGlobalMapping\\r\\n \\r\\n\""
```

Ce problème a été résolu dans les versions suivantes de Kubernetes :

| Version de Kubernetes | Version corrigée |
|--|:--:|
| 1.12 | 1.12.6 ou version ultérieure |
| 1.13 | 1.13.4 ou version ultérieure |
| 1.14 et versions ultérieures | N/A |

### <a name="azure-files-mount-fails-because-of-storage-account-key-changed"></a>Échec du montage d’Azure Files en raison de la modification de la clé du compte de stockage

Si votre clé de compte de stockage a changé, vous pouvez voir des échecs de montage d’Azure Files.

Vous pouvez atténuer le problème en mettant manuellement à jour le champ `azurestorageaccountkey` dans un secret du fichier Azure avec votre clé de compte de stockage codée en base64.

Pour encoder votre clé de compte de stockage en base64, vous pouvez utiliser `base64`. Par exemple :

```console
echo X+ALAAUgMhWHL7QmQ87E1kSfIqLKfgC03Guy7/xk9MyIg2w4Jzqeu60CVw2r/dm6v6E0DWHTnJUEJGVQAoPaBc== | base64
```

Pour mettre à jour votre fichier de secret Azure, utilisez `kubectl edit secret`. Par exemple :

```console
kubectl edit secret azure-storage-account-{storage-account-name}-secret
```

Après quelques minutes, le nœud de l’agent réessaie le montage d’Azure Files avec la clé de stockage mise à jour.


### <a name="cluster-autoscaler-fails-to-scale-with-error-failed-to-fix-node-group-sizes"></a>Échec de la mise à l’échelle automatique du cluster en cas d’échec de la résolution des tailles des groupes de nœuds

Si votre programme de mise à l’échelle automatique de cluster n’effectue pas de scale-up/scale-down et que vous voyez une erreur comme celle ci-dessous sur les [journaux du programme de mise à l’échelle automatique de cluster][view-master-logs].

```console
E1114 09:58:55.367731 1 static_autoscaler.go:239] Failed to fix node group sizes: failed to decrease aks-default-35246781-vmss: attempt to delete existing nodes
```

Cette erreur est due à une condition de concurrence du programme de mise à l’échelle automatique de cluster en amont. Dans ce cas, le programme de mise à l’échelle automatique de cluster se termine par une valeur différente de celle qui est réellement dans le cluster. Pour sortir de cet état, désactivez et réactivez le [programme de mise à l’échelle automatique de cluster][cluster-autoscaler].

### <a name="slow-disk-attachment-getazuredisklun-takes-10-to-15-minutes-and-you-receive-an-error"></a>Attachement du disque lent : `GetAzureDiskLun` prend 10 à 15 minutes et une erreur s’affiche

Dans les versions de Kubernetes **antérieures à 1.15.0**, vous pouvez recevoir une erreur telle que **Erreur WaitForAttach : le numéro d’unité logique du disque est introuvable**.  Pour contourner ce problème, attendez environ 15 minutes, puis réessayez.


### <a name="why-do-upgrades-to-kubernetes-116-fail-when-using-node-labels-with-a-kubernetesio-prefix"></a>Pourquoi les mises à niveau vers Kubernetes 1.16 échouent lors de l’utilisation d’étiquettes de nœud avec un préfixe kubernetes.io ?

À compter de Kubernetes [1.16](https://v1-16.docs.kubernetes.io/docs/setup/release/notes/), [seul un sous-ensemble défini d’étiquettes avec le préfixe kubernetes.io](https://v1-18.docs.kubernetes.io/docs/concepts/overview/working-with-objects/labels/) peut être appliqué par le kubelet à des nœuds. AKS ne peut pas supprimer d’étiquettes actives en votre nom sans votre consentement, car cela peut entraîner un temps d’arrêt des charges de travail affectées.

Par conséquent, pour atténuer ce risque, vous pouvez :

1. Mettre à niveau votre plan de contrôle de cluster vers la version 1.16 ou ultérieure.
2. Ajouter un nouveau pool de nœuds sur la version 1.16 ou ultérieure sans les étiquettes kubernetes.io non prises en charge.
3. Supprimer l’ancien pool de nœuds.

AKS étudie actuellement la capacité de faire muter des étiquettes actives sur un pool de nœuds afin d’améliorer cette atténuation.



<!-- LINKS - internal -->
[view-master-logs]: view-master-logs.md
[cluster-autoscaler]: cluster-autoscaler.md
