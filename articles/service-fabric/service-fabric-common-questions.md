---
title: Questions courantes sur Microsoft Azure Service Fabric
description: Forum aux questions sur Service Fabric, fonctionnalités, cas d’usage et scénarios courants notamment.
ms.topic: troubleshooting
ms.date: 08/18/2017
ms.author: pepogors
ms.openlocfilehash: 1655a8ed03b1f678cc5dba0a165e0bcca1d2517a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87292851"
---
# <a name="commonly-asked-service-fabric-questions"></a>Questions fréquentes sur Service Fabric

Les utilisateurs posent fréquemment des questions sur l’utilisation et les fonctionnalités de Service Fabric. Ce document regroupe un grand nombre de ces questions fréquentes et leurs réponses.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="cluster-setup-and-management"></a>Création et gestion de clusters

### <a name="how-do-i-roll-back-my-service-fabric-cluster-certificate"></a>Comment restaurer mon certificat de cluster Service Fabric ?

Restaurer une mise à niveau pour votre application nécessite de détecter les échecs d’intégrité avant que le quorum du cluster Service Fabric ne valide la modification. Les modifications validées peuvent uniquement être restaurées par progression. Le recours à un ingénieur en charge de la remontée d’information via le support technique peut être nécessaire pour récupérer votre cluster, si un changement de certificat cassant non surveillé a été introduit.  La [mise à niveau des applications de Service Fabric](./service-fabric-application-upgrade.md?branch=master) applique des [paramètres de mise à niveau d’application](./service-fabric-application-upgrade-parameters.md?branch=master), et garantit une mise à niveau sans temps d’arrêt.  En suivant notre mode de surveillance recommandé pour la mise à niveau des applications, la progression automatique via les domaines de mise à jour est basée sur la réussite des vérifications d’intégrité. Une restauration automatique a lieu en cas d’échec de la mise à jour d’un service par défaut.
 
Si votre cluster utilise toujours la propriété classique d’empreinte du certificat dans votre modèle Resource Manager, il est recommandé de [modifier le cluster de l’utilisation de l’empreinte du certificat à l’utilisation du nom commun](./service-fabric-cluster-change-cert-thumbprint-to-cn.md) pour tirer parti des fonctionnalités modernes de gestion des secrets.

### <a name="can-i-create-a-cluster-that-spans-multiple-azure-regions-or-my-own-datacenters"></a>Puis-je créer un cluster qui englobe plusieurs régions Azure ou mes propres centres de données ?

Oui. 

La technologie de clustering Service Fabric permet de combiner des machines qui s’exécutent partout dans le monde, tant qu’elles sont connectées entre elles par un réseau. Toutefois, la création et l’exécution d’un tel cluster peuvent être compliquées.

Si vous êtes intéressé par ce scénario, nous vous encourageons à nous contacter via la [Liste de problèmes Service Fabric sur GitHub](https://github.com/azure/service-fabric-issues) ou via votre technicien de support pour obtenir des indications supplémentaires. L’équipe Service Fabric travaille à fournir des instructions et des recommandations claires pour ce scénario. 

Voici quelques points importants à prendre en compte : 

1. À l’heure actuelle, la ressource de cluster Service Fabric dans Azure est régionale, comme les groupes de machines virtuelles identiques sur lesquels le cluster est basé. De fait, en cas de défaillance régionale, vous risquez de perdre la possibilité de gérer le cluster via Azure Resource Manager ou le portail Azure. Cela peut se produire même si le cluster continue de s’exécuter et que vous pouvez interagir directement avec lui. En outre, Azure ne permet pas à un même réseau virtuel d’être utilisé par plusieurs régions. Ainsi, un cluster Azure à plusieurs régions nécessite soit des [adresses IP publiques pour chaque machine virtuelle dans les groupes de machines virtuelles identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-networking.md#public-ipv4-per-virtual-machine), soit des [passerelles VPN Azure](../vpn-gateway/vpn-gateway-about-vpngateways.md). Toutes ces options de réseau ont un impact différent sur les coûts, les performances, et à un certain degré, sur la conception de l’application. Il est donc nécessaire d’analyser et de planifier avec soin avant de configurer un tel environnement.
2. La maintenance, la gestion et la surveillance de ces machines peuvent devenir complexes, en particulier en cas de répartition sur plusieurs _types_ d’environnements, par exemple, entre différents fournisseurs cloud, ou entre des ressources locales et des ressources Azure. Vérifiez bien que les mises à niveau, la surveillance, la gestion et les diagnostics ont été compris à la fois pour le cluster et les applications avant d’exécuter les charges de travail de production dans un tel environnement. Si vous avez l’habitude de résoudre ces problèmes dans Azure ou dans vos propres centres de données, il est probable que vous puissiez appliquer ces mêmes solutions lors de la création ou de l’exécution de votre cluster Service Fabric. 

### <a name="do-service-fabric-nodes-automatically-receive-os-updates"></a>Les nœuds Service Fabric reçoivent-ils automatiquement les mises à jour du système d’exploitation ?

Vous pouvez utiliser la fonctionnalité [Mise à jour automatique de l’image de système d’exploitation d’un groupe de machines virtuelles identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md), actuellement en disponibilité générale.

Pour les clusters qui ne s’exécutent PAS sur Azure, nous [proposons une application](service-fabric-patch-orchestration-application.md) permettant de corriger les systèmes d’exploitation qui se trouvent sous les nœuds Service Fabric.

### <a name="can-i-use-large-virtual-machine-scale-sets-in-my-sf-cluster"></a>Puis-je utiliser de grands groupes de machines virtuelles identiques dans mon cluster Service Fabric ? 

**Réponse courte** : Non. 

**Réponse longue** : les grands groupes de machines virtuelles identiques peuvent accueillir jusqu'à 1 000 instances de machines virtuelles, mais pour cela ils utilisent des groupes de placement. Les domaines d’erreur (FD) et les domaines de mise à niveau (UD) ne sont cohérents qu’au sein d’un groupe de placement. Service Fabric utilise des domaines d’erreur et des domaines de mise à niveau pour prendre des décisions de placement de vos instances/réplicas de service. Étant donné que les domaines d’erreur et de mise à niveau ne sont comparables qu’au sein d’un groupe de placement, Service Fabric ne peut pas les utiliser. Par exemple, si VM1 dans PG1 a une topologie de FD = 0 et VM9 dans PG2 a une topologie de FD = 4, cela ne signifie pas que VM1 et VM2 se trouvent sur deux racks matériels différents. Par conséquent, Service Fabric ne peut pas dans ce cas utiliser les valeurs FD pour prendre des décisions de placement.

Il y a actuellement d’autres problèmes avec les groupes de machines virtuelles identiques volumineux, notamment l’absence de prise en charge de l’équilibrage de charge de niveau 4. Pour plus d’informations, consultez la page [Grands groupes identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-placement-groups.md)



### <a name="what-is-the-minimum-size-of-a-service-fabric-cluster-why-cant-it-be-smaller"></a>Quelle est la taille minimale d’un cluster Service Fabric ? Pourquoi ne peut-il pas être plus petit ?

La taille minimale prise en charge pour un cluster Service Fabric exécutant des charges de travail de production est de cinq nœuds. Pour les scénarios de développement, nous prenons en charge un seul nœud (optimisé pour une expérience de développement rapide dans Visual Studio) et les clusters à cinq nœuds.

Nous demandons qu’un cluster de production comporte au moins cinq nœuds pour les trois raisons suivantes :
1. Même quand aucun service utilisateur ne s’exécute, un cluster Service Fabric exécute un ensemble de services système avec état, notamment le service de nommage et le service Failover Manager. Ces services système sont essentiels pour que le cluster reste opérationnel.
2. Nous plaçons toujours un réplica d’un service par nœud : ainsi, la taille du cluster est la limite supérieure pour le nombre de réplicas que peut avoir un service (en fait une partition).
3. Dans la mesure où une mise à niveau d’un cluster entraîne l’arrêt d’au moins un nœud, nous voulons avoir une réserve d’au moins un nœud : nous voulons donc qu’un cluster de production ait au moins deux nœuds *en plus* du strict minimum. Le strict minimum est la taille du quorum d’un service système, comme expliqué ci-dessous.  

Nous voulons le cluster soit disponible en cas de défaillance simultanée de deux nœuds. Pour qu’un cluster Service Fabric soit disponible, les services système doivent être disponibles. Les services système avec état, comme le service de nommage et le service Failover Manager, qui font le suivi des services déployés sur le cluster ainsi que leur emplacement d’hébergement actuel, nécessitent une cohérence forte. Cette cohérence forte, quant à elle, dépend de la capacité d’atteindre un *quorum* pour une mise à jour donnée de l’état de ces services, ce quorum représentant une majorité stricte des réplicas (N/2 + 1) pour un service donné. Ainsi, si nous voulons assurer la résilience en cas de perte simultanée de deux nœuds (donc de la perte simultanée de deux réplicas d’un service système), nous devons avoir ClusterSize - QuorumSize >= 2, ce qui fait passer la taille minimale à cinq. Pour voir cela, considérez que le cluster a N nœuds et qu’il existe N réplicas d’un service système, un sur chaque nœud. La taille de quorum pour un service système est (N/2 + 1). L’inégalité ci-dessus se présente comme N - (N/2 + 1) >= 2. Il existe deux cas à prendre en compte : quand N est pair et quand il est impair. Si N est pair, disons N = 2\*m, où m >= 1, l’inégalité se présente comme 2\*m - (2\*m/2 + 1) >= 2 ou m >= 3. La valeur minimale pour N est 6, qui est obtenue quand m = 3. En revanche, si N est impair, disons N = 2\*m + 1, où m >= 1, l’inégalité se présente comme 2\*m + 1 - ( (2\*m+1)/2 + 1 ) >= 2 ou 2\*m+1 - (m+1) >= 2 or m >= 2. La valeur minimale pour N est 5, qui est obtenue quand m = 2. Ainsi, parmi toutes les valeurs de N qui satisfont l’inégalité ClusterSize - QuorumSize >= 2, la valeur minimale est 5.

Notez que dans l’argument ci-dessus, nous avons supposé que chaque nœud a un réplica d’un service système : sa taille est donc calculée en fonction du nombre de nœuds du cluster. Cependant, en modifiant *TargetReplicaSetSize*, nous pourrions rendre la taille du quorum inférieure à (N/2+1), ce qui peut donner l’impression que nous pourrions avoir un cluster inférieur à 5 nœuds et avoir néanmoins toujours 2 nœuds au-dessus de la taille du quorum. Par exemple, dans un cluster de quatre nœuds, si nous définissons TargetReplicaSetSize sur 3, la taille du quorum basé sur TargetReplicaSetSize est (3/2 + 1) ou 2 : nous avons donc ClusterSize - QuorumSize = 4-2 >= 2. Cependant, nous ne pouvons pas garantir que le service système est au niveau du quorum ou au-delà si nous perdons simultanément une paire de nœuds : il est possible que les deux nœuds que nous avons perdus hébergeaient deux réplicas, et que le service système perde donc le quorum (n’ayant plus qu’un seul réplica), devenant ainsi indisponible.

Dans ce cadre, examinons certaines configurations de cluster possibles :

**Un nœud** : cette option n’assure pas une haute disponibilité, car la perte du nœud pour une raison quelconque se traduit par la perte du cluster.

**Deux nœuds** : le quorum d’un service déployé sur deux nœuds (N = 2) est 2 (2/2 + 1 = 2). En cas de perte d’un réplica, il est impossible d’obtenir un quorum. Comme la mise à niveau d’un serveur requiert la mise hors ligne d’un réplica, cette configuration n’a aucune utilité.

**Trois nœuds** : avec trois nœuds (N = 3), la condition requise pour créer un quorum reste de deux nœuds (3/2 + 1 = 2). Cela signifie que vous pouvez perdre un nœud individuel et néanmoins conserver le quorum, mais que la défaillance simultanée de deux nœuds fait que les services système perdent le quorum, entraînant l’indisponibilité du cluster.

**Quatre nœuds** : avec quatre nœuds (N=4), la condition nécessaire pour créer un quorum est trois nœuds (4/2 + 1 = 3). Cela signifie que vous pouvez perdre un nœud individuel et néanmoins conserver le quorum, mais que la défaillance simultanée de deux nœuds fait que les services système perdent le quorum, entraînant l’indisponibilité du cluster.

**Cinq nœuds** : avec cinq nœuds (N=5), la condition nécessaire pour créer un quorum reste trois nœuds (5/2 + 1 = 3). Cela signifie que vous pouvez perdre deux nœuds en même temps et néanmoins conserver le quorum pour les services système.

Pour les charges de travail de production, vous devez être résilient face à une défaillance simultanée d’au moins deux nœuds (par exemple une en raison d’une mise à niveau du cluster et une pour d’autres raisons) : ainsi, cinq nœuds sont nécessaires.

### <a name="can-i-turn-off-my-cluster-at-nightweekends-to-save-costs"></a>Puis-je désactiver mon cluster la nuit et le week-end pour réduire les coûts ?

En général, non. Service Fabric stocke l’état sur des disques éphémères locaux. En d’autres termes, si la machine virtuelle est déplacée vers un hôte différent, les données n’effectuent pas le déplacement. Normalement, ce n’est pas un problème car le nouveau nœud est mis à jour par d’autres nœuds. Toutefois, si vous arrêtez tous les nœuds et que vous les redémarrez ultérieurement, il est fort probable que la plupart des nœuds démarrent sur de nouveaux hôtes et empêchent la restauration du système.

Si vous souhaitez créer des clusters pour tester votre application avant de la déployer, nous vous recommandons de les créer dynamiquement dans le cadre de votre [pipeline d’intégration continue/de déploiement continu](service-fabric-tutorial-deploy-app-with-cicd-vsts.md).


### <a name="how-do-i-upgrade-my-operating-system-for-example-from-windows-server-2012-to-windows-server-2016"></a>Comment mettre à niveau mon système d’exploitation (par exemple de Windows Server 2012 vers Windows Server 2016) ?

Alors que nous travaillons sur une expérience améliorée, vous êtes aujourd’hui chargé de la mise à niveau. Vous devez mettre à niveau l’image du système d’exploitation sur les machines virtuelles du cluster, une machine virtuelle à la fois. 

### <a name="can-i-encrypt-attached-data-disks-in-a-cluster-node-type-virtual-machine-scale-set"></a>Puis-je chiffrer des disques de données associés dans un type de nœud de cluster (groupe de machines virtuelles identiques) ?
Oui.  Pour plus d’informations, consultez [Créer un cluster avec des disques de données joints](../virtual-machine-scale-sets/virtual-machine-scale-sets-attached-disks.md#create-a-service-fabric-cluster-with-attached-data-disks) et [Azure Disk Encryption pour Virtual Machine Scale Sets](../virtual-machine-scale-sets/disk-encryption-overview.md).

### <a name="can-i-use-low-priority-vms-in-a-cluster-node-type-virtual-machine-scale-set"></a>Puis-je utiliser des machines virtuelles de faible priorité dans un type de nœud de cluster (groupe de machines virtuelles identiques) ?
Non. Les machines virtuelles de faible priorité ne sont pas prises en charge. 

### <a name="what-are-the-directories-and-processes-that-i-need-to-exclude-when-running-an-anti-virus-program-in-my-cluster"></a>Quels sont les répertoires et processus à exclure lors de l’exécution d’un programme antivirus dans le cluster ?

| **Répertoires exclus de l’antivirus** |
| --- |
| Program Files\Microsoft Service Fabric |
| FabricDataRoot (dans la configuration du cluster) |
| FabricLogRoot (dans la configuration du cluster) |

| **Processus exclus de l’antivirus** |
| --- |
| Fabric.exe |
| FabricHost.exe |
| FabricInstallerService.exe |
| FabricSetup.exe |
| FabricDeployer.exe |
| ImageBuilder.exe |
| FabricGateway.exe |
| FabricDCA.exe |
| FabricFAS.exe |
| FabricUOS.exe |
| FabricRM.exe |
| FileStoreService.exe |
 
### <a name="how-can-my-application-authenticate-to-key-vault-to-get-secrets"></a>Comment mon application peut s’authentifier auprès de Key Vault pour obtenir des secrets ?
Les procédés suivants permettent à votre application d’obtenir des informations d’identification pour s’authentifier auprès de Key Vault :

R. Au cours de votre travail de génération/compression d’applications, vous pouvez extraire un certificat et l’utiliser dans le package de données de votre application SF en vue de l’authentification auprès de Key Vault.
B. Pour les hôtes MSI d’un groupe de machines virtuelles identiques, vous pouvez développer un élément PowerShell SetupEntryPoint simple pour votre application SF afin d’obtenir [un jeton d’accès à partir du point de terminaison MSI](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md), puis [récupérer vos secrets à partir de Key Vault](/powershell/module/azurerm.keyvault/get-azurekeyvaultsecret).

## <a name="application-design"></a>Conception des applications

### <a name="whats-the-best-way-to-query-data-across-partitions-of-a-reliable-collection"></a>Quel est le meilleur moyen d’interroger des données sur plusieurs partitions d’une collection fiable ?

Les collections fiables sont généralement [partitionnées](service-fabric-concepts-partitioning.md) pour permettre d’effectuer un scale-out et d’augmenter les performances ainsi que le débit. Cela signifie que l’état d’un service peut être réparti entre des dizaines voire des centaines d’ordinateurs. Pour effectuer des opérations sur ce jeu de données complet, plusieurs possibilités s’offrent à vous :

- Créer un service qui interroge toutes les partitions d’un autre service pour extraire les données requises.
- Créer un service capable de recevoir des données provenant de toutes les partitions d’un autre service.
- Envoyer régulièrement des données de chaque service à un magasin externe. Cette approche convient uniquement si les requêtes que vous exécutez ne font pas partie de votre logique d’entreprise, car les données du magasin externe seront périmées.
- Vous pouvez également stocker les données qui doivent prendre en charge l’interrogation de tous les enregistrements directement dans un magasin de données plutôt que dans une collection fiable. Cela élimine le problème avec les données périmées, mais ne permet pas d’exploiter les avantages des collections fiables.


### <a name="whats-the-best-way-to-query-data-across-my-actors"></a>Quel est le meilleur moyen d’interroger les données entre mes différents acteurs ?

Les acteurs sont conçus comme des unités indépendantes d’état et de calcul. Il n’est pas recommandé d’effectuer des requêtes volumineuses sur l’état des acteurs pendant l’exécution. Si vous devez exécuter une requête sur l’ensemble des états d’acteur, vous devez envisager de :

- Remplacer vos services d’acteur par des services fiables avec état, comme le nombre de requêtes réseau, pour collecter toutes les données, depuis le nombre d’acteurs jusqu’au nombre de partitions de votre service.
- Concevoir vos acteurs pour qu’ils envoient régulièrement leur état à un magasin externe afin de faciliter les requêtes. Comme auparavant, cette approche n’est viable que si les requêtes que vous exécutez ne sont pas obligatoires pour votre comportement d’exécution.

### <a name="how-much-data-can-i-store-in-a-reliable-collection"></a>Quelle quantité de données puis-je stocker dans une collection fiable ?

Les services fiables sont généralement partitionnés. La quantité que vous pouvez stocker n’est donc limitée que par le nombre d’ordinateurs dans votre cluster et par la quantité de mémoire disponible sur ces ordinateurs.

Par exemple, supposons que vous ayez une collection fiable dans un service de 100 partitions et 3 réplicas, la taille moyenne des objets stockés est de 1 Ko. Maintenant, supposons que vous disposiez d’un cluster de 10 ordinateurs avec 16 Go de mémoire par ordinateur. Par souci de simplicité et par prudence, supposons que le système d’exploitation, les services système, le runtime Service Fabric et vos services consomment 6 Go, ce qui laisse 10 Go disponibles par ordinateur soit 100 Go pour le cluster.

Sachant que chaque objet doit être stocké trois fois (une copie principale et deux réplicas), vous aurez suffisamment de mémoire pour environ 35 millions d’objets dans votre collection lorsqu’elle fonctionnera à pleine capacité. Toutefois, nous vous recommandons d’assurer une résilience à la perte simultanée d’un domaine de défaillance et d’un domaine de mise à niveau, ce qui représente environ 1/3 de la capacité et réduit le nombre à environ 23 millions.

Ce calcul suppose également :

- Que la répartition des données entre les partitions est à peu près uniforme ou que vous envoyez des métriques de charge à Cluster Resource Manager. Par défaut, Service Fabric équilibre la charge en fonction du nombre de réplicas. Dans l’exemple précédent, 10 réplicas principaux et 20 réplicas secondaires seraient placés sur chaque nœud du cluster. Cela fonctionne bien pour la charge répartie équitablement entre les partitions. Si la charge n’est pas équilibrée, vous devez la signaler afin que le Resource Manager puisse regrouper des réplicas plus petits et autoriser des réplicas plus volumineux à consommer davantage de mémoire sur un nœud individuel.

- Que le service fiable en question est le seul à stocker l’état dans le cluster. Comme vous pouvez déployer plusieurs services dans un cluster, vous devez être conscient que les ressources de chacun doivent exécuter et gérer son état.

- Que le cluster lui-même pas n’augmente pas ou ne diminue pas en taille. Si vous ajoutez d’autres ordinateurs, Service Fabric rééquilibre vos réplicas en fonction des capacités supplémentaires jusqu’à ce que le nombre d’ordinateurs dépasse le nombre de partitions dans votre service, car un réplica ne peut pas englober plusieurs ordinateurs. En revanche, si vous réduisez la taille du cluster en supprimant des ordinateurs, vos réplicas sont davantage compressés et ont une capacité moindre.

### <a name="how-much-data-can-i-store-in-an-actor"></a>Quelle quantité de données puis-je stocker dans un acteur ?

Comme avec les services fiables, la quantité de données que vous pouvez stocker dans un service d’acteur n’est limitée que par l’espace disque total et la mémoire disponible sur les nœuds de votre cluster. Toutefois, les acteurs sont plus efficaces lorsqu’ils sont utilisés pour encapsuler une petite quantité d’état et de logique métier associée. En règle générale, un acteur doit avoir l’état qui est mesuré en kilo-octets.


### <a name="where-does-azure-service-fabric-resource-provider-store-customer-data"></a>Où le fournisseur de ressources Azure Service Fabric stocke les données client ?

Le fournisseur de ressources Azure Service Fabric ne déplace pas ou ne stocke pas les données client hors de la région dans laquelle elles sont déployées.


## <a name="other-questions"></a>Autres questions

### <a name="how-does-service-fabric-relate-to-containers"></a>Quel est le rapport entre Service Fabric et les conteneurs ?

Les conteneurs constituent un moyen simple d’encapsuler les services et leurs dépendances pour qu’ils s’exécutent de manière cohérente dans tous les environnements et puissent fonctionner de manière isolée sur un seul ordinateur. Service Fabric vous permet de déployer et gérer des services, notamment des [services encapsulés dans un conteneur](service-fabric-containers-overview.md).

### <a name="are-you-planning-to-open-source-service-fabric"></a>Prévoyez-vous de diffuser le code de Service Fabric ?

Nous avons des composants de Service Fabric en Open Source ([infrastructure des services fiable](https://github.com/Azure/service-fabric-services-and-actors-dotnet), [infrastructure des acteurs fiable](https://github.com/Azure/service-fabric-services-and-actors-dotnet), [bibliothèques d’intégration ASP.NET Core](https://github.com/Azure/service-fabric-aspnetcore), [Service Fabric Explorer](https://github.com/Azure/service-fabric-explorer) et [l’interface de ligne de commande Service Fabric](https://github.com/Azure/service-fabric-cli)) sur GitHub et acceptons des contributions de communautés à ces projets. 

Nous avons [récemment annoncé](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) que nous prévoyons d’ouvrir la source au runtime Service Fabric. À ce stade, nous avons le [dépôt de Service Fabric](https://github.com/Microsoft/service-fabric/) sur GitHub avec les outils build et de test Linux, ce qui signifie que vous pouvez cloner le dépôt, générer Service Fabric pour Linux, exécuter des tests de base, ouvrir des problèmes et soumettre des requêtes d’extraction. Nous nous efforçons de faire en sorte que l’environnement build Windows migré soit également présent avec un environnement CI complet.

Consultez le [blog Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) pour en savoir plus.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [principaux concepts](service-fabric-technical-overview.md) et les [meilleures pratiques de Service Fabric](service-fabric-best-practices-overview.md)
