---
title: Questions fréquentes (FAQ) - Avere vFXT pour Azure
description: Reportez-vous à ces questions fréquemment posées pour décider si les Avere vFXT pour Azure correspond à vos besoins. Découvrez de quelle manière Avere vFXT pour Azure fonctionne avec d’autres composants Azure.
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 12/19/2019
ms.author: rohogue
ms.custom: references_regions
ms.openlocfilehash: 647a9792384a21dfc191371ef444cbdc098bc819
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92342363"
---
# <a name="avere-vfxt-for-azure-faq"></a>Questions fréquentes (FAQ) sur Avere vFXT pour Azure

Cet article fournit des réponses qui peuvent vous aider à déterminer si le système Avere vFXT pour Azure est adapté à vos besoins. Il contient des informations de base sur Avere vFXT et explique comment le système fonctionne avec d'autres composants Azure et avec des produits de fournisseurs externes.

## <a name="general"></a>Général

### <a name="what-is-avere-vfxt-for-azure"></a>En quoi consiste Avere vFXT pour Azure ?

Avere vFXT pour Azure est un système de fichiers hautes performances qui met en cache les données actives dans Azure Compute pour un traitement efficace des charges de travail critiques.

### <a name="is-avere-vfxt-a-storage-solution"></a>Le système Avere vFXT est-il une solution de stockage ?

Non. Avere vFXT pour Azure est un *cache* de système de fichiers qui s’attache à des environnements de stockage, comme votre console EMC, votre système NAS NetApp ou un conteneur d’objets blob Azure. Le système Avere vFXT facilite les requêtes de données émanant des clients et met en cache les données qu'il traite pour améliorer les performances à grande échelle et au fil du temps. Le système Avere vFXT proprement dit ne stocke pas les données. Il ne dispose d’aucune information sur la quantité de données stockées derrière lui.

### <a name="is-avere-vfxt-a-tiering-solution"></a>Le système Avere vFXT est-il une solution de hiérarchisation ?

Avere vFXT pour Azure ne hiérarchise pas automatiquement les données entre les niveaux chaud et froid.  

### <a name="how-do-i-know-if-an-environment-is-right-for-avere-vfxt"></a>Comment savoir si un environnement convient au système Avere vFXT ?

La meilleure façon de réfléchir à cette question consiste à se demander : « La charge de travail peut-elle être mise en cache ? » Autrement dit, la charge de travail a-t-elle un rapport lecture/écriture élevé ? Par exemple, 80/20 ou 70/30 lectures/écritures.

Envisagez d’utiliser Avere vFXT pour Azure si vous disposez d’un pipeline d’analyse basé sur un fichier qui s’exécute sur un grand nombre de machines virtuelles Azure, et qu’il répond à une ou plusieurs des conditions suivantes :

* Les performances globales sont faibles ou incohérentes en raison de longs temps d’accès au fichier (des dizaines de millisecondes ou des secondes, en fonction des besoins). Cette latence est inacceptable pour le client.

* Les données nécessaires pour le traitement se trouvent à l’extrémité d’un environnement de réseau étendu et il n’est pas pratique de déplacer définitivement ces données. Les données peuvent se trouver dans une autre région Azure ou dans un centre de données de client.

* Un nombre important de clients demandent les données ; par exemple, dans un cluster HPC (High Performance Computing, calcul haute performance). Le grand nombre de demandes simultanées peut augmenter la latence.

* Le client souhaite exécuter son pipeline actuel « en l’état » sur des machines virtuelles Azure, et il a besoin d'une solution de stockage (ou de mise en cache) partagée POSIX à des fins d'extensibilité. Si vous utilisez Avere vFXT pour Azure, il est inutile de remanier l'architecture du pipeline de travail pour passer des appels natifs au Stockage Blob Azure.

* Votre application HPC est basée sur les clients NFSv3. (Dans certains cas, des clients SMB 2.1 peuvent être utilisés, mais les performances sont limitées.)

Le diagramme suivant peut vous aider à répondre à cette question. Plus votre workflow est proche du coin supérieur droit, plus il est probable que la solution de mise en cache Avere vFXT pour Azure est adaptée à votre environnement.

![Diagramme montrant que les charges nécessitant beaucoup de lectures avec des milliers de clients conviennent mieux à Avere vFXT](media/avere-vfxt-fit-assessment.png)

### <a name="at-what-scale-of-clients-does-the-avere-vfxt-solution-make-the-most-sense"></a>À quel volume de clients la solution Avere vFXT est-elle la plus pertinente ?

La solution de cache Avere vFXT est conçue pour gérer des centaines, des milliers ou des dizaines de milliers de cœurs de calcul. Si vous avez plusieurs machines exécutant du travail léger, Avere vFXT n’est pas la solution appropriée.

Les clients Avere vFXT classiques exécutent des charges de travail exigeantes commençant à environ 1 000 cœurs de processeur. Ces environnements peuvent comporter jusqu’à 50 000 cœurs, voire plus. Étant donné que le système Avere vFXT est évolutif, vous pouvez ajouter des nœuds pour prendre en charge ces charges de travail à mesure qu'elles augmentent pour exiger davantage de débit ou d'opérations d'E/S par seconde (IOPS).

### <a name="how-much-data-can-an-avere-vfxt-environment-store"></a>Quelle quantité de données un environnement Avere vFXT peut-il stocker ?

Avere vFXT pour Azure est un cache. Il ne stocke pas spécifiquement de données. Il utilise une combinaison de RAM et de disques SSD pour stocker les données mises en cache. Les données sont définitivement stockées sur un système de stockage back-end (par exemple, un système NAS NetApp ou un conteneur d'objets blob). Le système Avere vFXT ne dispose d'aucune information sur le volume de données stocké derrière lui. Avere vFXT met uniquement en cache le sous-ensemble des données demandées par les clients.  

### <a name="what-regions-are-supported"></a>Quelles sont les régions prises en charge ?

Avere vFXT pour Azure est pris en charge dans toutes les régions à l’exception des régions souveraines (Chine et Allemagne). Vérifiez que la région que vous souhaitez utiliser peut prendre en charge le grand nombre de cœurs de calcul et les instances de machines virtuelles nécessaires à la création du cluster Avere vFXT.

### <a name="how-do-i-get-help-with-avere-vfxt"></a>Comment obtenir de l'aide sur le système Avere vFXT ?

Un groupe de support spécialisé propose de l’aide sur le système Avere vFXT pour Azure. Suivez les instructions mentionnées dans [Obtenir de l’aide avec votre système](avere-vfxt-open-ticket.md#open-a-support-ticket-for-your-avere-vfxt) pour ouvrir un ticket de support à partir du portail Azure.

### <a name="is-avere-vfxt-highly-available"></a>Le système Avere vFXT est-il hautement disponible ?

Oui, le système Avere vFXT s'exécute exclusivement en tant que solution à haute disponibilité.

### <a name="does-avere-vfxt-for-azure-also-support-other-cloud-services"></a>Avere vFXT pour Azure prend-il également en charge d’autres services cloud ?

Oui, les clients peuvent utiliser plusieurs fournisseurs de cloud avec le cluster Avere vFXT. Il prend en charge les compartiments standard AWS S3, les compartiments standard des services Google Cloud et les conteneurs d'objets blob Azure.

> [!NOTE]
> Des frais de logiciel s’appliquent si vous utilisez Avere vFXT avec un stockage AWS ou Google Cloud. Il n’y a pas de frais de logiciel supplémentaires pour l’utilisation du stockage Blob Azure.

## <a name="technical-compute"></a>Technique : Calcul

### <a name="can-you-describe-what-an-avere-vfxt-environment-looks-like"></a>Pouvez-vous décrire ce à quoi « ressemble » un environnement Avere vFXT ?

Le système Avere vFXT est une appliance en cluster constituée de plusieurs machines virtuelles Azure. Une bibliothèque Python gère la création, la suppression et la modification du cluster. Pour en savoir plus, lisez [En quoi consiste Avere vFXT pour Azure ?](avere-vfxt-overview.md).

### <a name="what-kind-of-azure-virtual-machines-does-avere-vfxt-run-on"></a>Sur quels types de machines virtuelles le système Azure Avere vFXT s'exécute-t-il ?  

Un cluster Avere vFXT pour Azure utilise des machines virtuelles Microsoft Azure E32s_v3.

<!-- ### Can I mix and match virtual machine types for my cluster?

No, you must choose one virtual machine type or the other.
    
### Can I move between virtual machine types?

Yes, there is a migration path to move from one VM type to the other. [Open a support ticket](avere-vfxt-open-ticket.md#open-a-support-ticket-for-your-avere-vfxt) to learn how.
-->

### <a name="does-the-avere-vfxt-environment-scale"></a>L’environnement Avere vFXT se met-il à l’échelle ?

La taille du cluster Avere vFXT peut aller de trois à 24 nœuds de machine virtuelle. Contactez le support technique Azure pour obtenir de l'aide sur la planification si vous pensez que vous avez besoin d'un cluster de plus de neuf nœuds. Plus les nœuds sont nombreux, plus l'architecture de déploiement doit être vaste.

### <a name="does-the-avere-vfxt-environment-autoscale"></a>L’environnement Avere vFXT se met-il automatiquement à l’échelle ?

Non. Vous pouvez augmenter ou réduire la taille du cluster, mais l’ajout ou la suppression de nœuds de cluster est une étape manuelle.

### <a name="can-i-run-the-avere-vfxt-cluster-as-a-virtual-machine-scale-set"></a>Puis-je exécuter le cluster Avere vFXT comme un groupe de machines virtuelles identiques ?

Avere vFXT ne prend pas en charge le déploiement d'un groupe de machines virtuelles identiques. Plusieurs mécanismes de prise en charge de la disponibilité intégrés sont conçus uniquement pour les machines virtuelles atomiques participant à un cluster.  

### <a name="can-i-run-the-avere-vfxt-cluster-on-low-priority-vms"></a>Puis-je exécuter le cluster Avere vFXT sur des machines virtuelles basse priorité ?

Non, le système nécessite un ensemble stable sous-jacent de machines virtuelles.

### <a name="can-i-run-the-avere-vfxt-cluster-in-containers"></a>Puis-je exécuter le cluster Avere vFXT dans des conteneurs ?

Non, le système Avere vFXT doit être déployé en tant qu'application indépendante.

### <a name="do-the-avere-vfxt-vms-count-against-my-compute-quota"></a>Les machines virtuelles Avere vFXT comptent-elles dans mon quota de calcul ?

Oui. Vérifiez que le quota disponible dans la région est suffisant pour prendre en charge le cluster.  

### <a name="can-i-run-the-avere-vfxt-cluster-machines-in-different-availability-zones"></a>Puis-je exécuter les machines du cluster Avere vFXT dans différentes zones de disponibilité ?

Non. Le modèle haute disponibilité d'Avere vFXT ne prend actuellement pas en charge les membres individuels d'un cluster Avere vFXT situés dans des zones de disponibilité différentes.

### <a name="can-i-clone-avere-vfxt-virtual-machines"></a>Puis-je cloner des machines virtuelles Avere vFXT ?

Non, vous devez utiliser le script Python pris en charge pour ajouter ou supprimer des nœuds dans le cluster Avere vFXT. Pour plus d’informations, lisez [Gérer le cluster Avere vFXT](avere-vfxt-manage-cluster.md).

### <a name="is-there-a-vm-version-of-the-software-i-can-run-in-my-own-local-environment"></a>Existe-t-il une version « machine virtuelle » du logiciel que je peux exécuter dans mon propre environnement local ?

Non, le système est proposé sous la forme d’une appliance en cluster et est testé sur des types de machines virtuelles spécifiques. Cette restriction empêche les clients de créer un système qui ne prend pas en charge les exigences de hautes performances d'un flux de travail Avere vFXT classique.

## <a name="technical-disks"></a>Technique : Disques

### <a name="what-types-of-disks-are-supported-for-the-azure-vms"></a>Quels sont les types de disques pris en charge pour les machines virtuelles Azure ?

Avere vFXT pour Azure peut utiliser des configurations de disque SSD Premium de 1 To ou de 4 To. La configuration de disque SSD Premium peut être déployée sous la forme de plusieurs disques managés.

### <a name="does-the-cluster-support-unmanaged-disks"></a>Le cluster prend-il en charge les disques non managés ?

Non, le cluster nécessite des disques managés.

### <a name="does-the-system-support-local-attached-ssds"></a>Le système prend-il en charge les disques SSD locaux (attachés) ?

Avere vFXT pour Azure ne prend actuellement pas en charge les disques SSD locaux. Les disques utilisés pour le système Avere vFXT doivent pouvoir s'arrêter et redémarrer, mais dans cette configuration, les disques SSD locaux joints peuvent uniquement être arrêtés.

### <a name="does-the-system-support-ultra-ssds"></a>Le système prend-il en charge les disques SSD Ultra ?

Non, le système prend uniquement en charge les configurations de disque SSD Premium.

### <a name="can-i-detach-my-premium-ssds-and-reattach-them-later-to-preserve-cache-contents-between-use"></a>Puis-je détacher mes disques SSD Premium et les rattacher ultérieurement pour conserver le contenu du cache entre deux utilisations ?

Le détachement et le rattachement de disques SSD ne sont pas pris en charge. Les métadonnées ou le contenu des fichiers présents sur la source peuvent avoir changé entre deux utilisations, ce qui peut entraîner des problèmes d'intégrité des données.

### <a name="does-the-system-encrypt-the-cache"></a>Le système chiffre-t-il le cache ?

Les données sont réparties entre les disques, mais elles ne sont pas chiffrées. Toutefois, les disques proprement dit peuvent être chiffrés. Pour plus d'informations, consultez [Sécuriser et utiliser des stratégies sur des machines virtuelles dans Azure](../virtual-machines/security-policy.md#encryption).

## <a name="technical-networking"></a>Technique : Réseau

### <a name="what-network-is-recommended"></a>Quel réseau est recommandé ?

Si vous utilisez un stockage local avec Avere vFXT, vous devez disposer d’une connexion réseau de 1 Gbit/s ou plus entre votre stockage et le cluster. Si vous avez une petite quantité de données et que vous voulez copier des données dans le cloud avant d’exécuter des travaux, la connectivité VPN peut être suffisante.

> [!TIP]
> La lenteur des lectures « à froid » initiales est proportionnelle à la lenteur de la liaison réseau. Les lectures lentes augmentent la latence du pipeline de travail.

### <a name="can-i-run-avere-vfxt-in-a-different-virtual-network-than-my-compute-cluster"></a>Puis-je exécuter le système Avere vFXT sur un réseau virtuel différent de celui de mon cluster de calcul ?

Oui, vous pouvez créer votre système Avere vFXT sur un autre réseau virtuel. Pour plus d’informations, lisez [Planifier votre système Avere vFXT](avere-vfxt-deploy-plan.md).

### <a name="does-avere-vfxt-require-its-own-subnet"></a>Le système Avere vFXT nécessite-t-il son propre sous-réseau ?

Oui. Avere vFXT s’exécute exactement comme un cluster haute disponibilité (HA) et nécessite plusieurs adresses IP pour fonctionner. Si le cluster est dans son propre sous-réseau, vous évitez tout risque de conflits d’adresses IP, lesquels peuvent entraîner des problèmes pour l’installation et le fonctionnement normal. Le sous-réseau du cluster peut se trouver dans réseau virtuel utilisé par d’autres ressources, tant que les adresses IP ne se chevauchent pas.

### <a name="can-i-run-avere-vfxt-on-infiniband"></a>Puis-je exécuter le système Avere vFXT sur InfiniBand ?

Non, le système Avere vFXT utilise uniquement Ethernet/IP.

### <a name="how-do-i-access-my-on-premises-nas-environment-from-avere-vfxt"></a>Comment accéder à mon environnement NAS local à partir du système Avere vFXT ?

L'environnement Avere vFXT est semblable à n'importe quelle autre machine virtuelle Azure en ce sens qu'il nécessite un accès routé via une passerelle réseau ou un VPN en direction du centre de données du client (et en provenance de celui-ci). N'hésitez pas à utiliser la connectivité Azure ExpressRoute si elle est disponible dans votre environnement.

### <a name="what-are-the-bandwidth-requirements-for-avere-vfxt"></a>Quelle est la bande passante requise pour le système Avere vFXT ?

L’ensemble des besoins de bande passante dépendent de deux facteurs :

* La quantité de données demandées à la source
* La tolérance à la latence du système client pendant le chargement initial des données  

Pour les environnements sensibles à la latence, vous devez utiliser une solution de fibre optique avec une vitesse de liaison minimale de 1 Gbit/s. Utilisez ExpressRoute s’il est disponible.  

### <a name="can-i-run-avere-vfxt-with-public-ip-addresses"></a>Puis-je exécuter le système Avere vFXT avec des adresses IP publiques ?

Non, le système Avere vFXT est destiné à être utilisé dans un environnement réseau sécurisé en respectant des bonnes pratiques.

### <a name="can-i-restrict-internet-access-from-my-clusters-virtual-network"></a>Puis-je restreindre l’accès à Internet à partir du réseau virtuel du cluster ?

En règle générale, vous pouvez configurer une sécurité supplémentaire sur votre réseau virtuel en fonction des besoins, mais certaines restrictions peuvent interférer avec le fonctionnement du cluster.

Par exemple, limiter l’accès Internet sortant à partir de votre réseau virtuel provoque des problèmes pour le cluster, sauf si vous ajoutez également une règle qui autorise explicitement l’accès à AzureCloud. Cette situation est décrite dans la [documentation supplémentaire sur GitHub](https://github.com/Azure/Avere/tree/master/src/vfxt/internet_access.md).

Pour obtenir de l’aide liée à la sécurité personnalisée, contactez le support comme décrit dans [Get help with your system](avere-vfxt-open-ticket.md#open-a-support-ticket-for-your-avere-vfxt) (Obtenir de l’aide avec votre système).

## <a name="technical-back-end-storage-core-filers"></a>Technique : Stockage back-end (système de stockage principal)

### <a name="how-many-core-filers-does-a-single-avere-vfxt-environment-support"></a>Combien de systèmes de stockage principaux (core filers) un seul environnement Avere vFXT prend en charge ?

Un cluster Avere vFXT prend en charge jusqu’à 20 systèmes de stockage principaux.

### <a name="how-does-the-avere-vfxt-environment-store-data"></a>Comment l'environnement Avere vFXT stocke-t-il les données ?

Avere vFXT n'est pas un système de stockage. Il s'agit d'un cache qui lit et écrit des données à partir de plusieurs cibles de stockage appelées systèmes de stockage principaux (ou core filers). Les données stockées sur les disques SSD Premium du système Avere vFXT sont temporaires et finissent par être vidées sur le stockage du système de stockage principal du back-end.

### <a name="which-core-filers-does-avere-vfxt-support"></a>Quels systèmes de stockage principaux Avere vFXT prend en charge ?

En général, Avere vFXT pour Azure prend en charge les systèmes suivants comme systèmes de stockage principaux :

* Dell EMC Isilon (OneFS 7.1, 7.2, 8.0 et 8.1) 
* NetApp ONTAP (Cluster Mode 9.4, 9.3, 9.2, 9.1P1, 8.0-8.3) et (7-Mode 7.*, 8.0 - 8.3)

* Conteneurs d'objets blob Azure (stockage localement redondant uniquement)
* Compartiments AWS S3
* Compartiments Google Cloud

### <a name="why-doesnt-avere-vfxt-support-all-nfs-filers"></a>Pourquoi le système Avere vFXT ne prend-il pas en charge tous les systèmes de stockage NFS ?

Même si toutes les plateformes NFS répondent aux mêmes normes IETF, dans la pratique, chaque implémentation a ses propres particularités. Ces détails déterminent la façon dont le système Avere vFXT interagit avec le système de stockage. Les systèmes pris en charge sont les plateformes les plus couramment utilisées sur le marché.

### <a name="does-avere-vfxt-support-private-object-storage-such-as-swiftstack"></a>Le système Avere vFXT prend-il en charge le stockage d'objets privés (comme SwiftStack) ?

AVERE vFXT ne prend pas en charge le stockage d’objets privés.

### <a name="how-can-i-get-a-specific-storage-product-under-support"></a>Comment obtenir un produit de stockage spécifique avec support ?

Le support est basé sur la quantité de demandes dans le domaine. Si le nombre de demandes basées sur les revenus est suffisant pour assurer le support d'une solution NAS, nous y réfléchirons. Formulez vos demandes par le biais du support Azure.

### <a name="can-i-use-azure-blob-storage-as-a-core-filer"></a>Puis-je utiliser le Stockage Blob Azure comme système de stockage principal ?

Oui, le système Avere vFXT pour Azure peut utiliser un conteneur d'objets blob de blocs comme système de stockage principal cloud.

### <a name="what-are-the-storage-account-requirements-for-a-blob-core-filer"></a>Quelles sont les exigences en matière de compte de stockage pour un système de stockage principal d'objets blob ?

Votre compte de stockage doit être un compte v2 (GPv2) universel et être configuré pour le stockage localement redondant uniquement. Le stockage géoredondant et le stockage redondant interzone ne sont pas pris en charge.

Pour plus d’informations sur les conditions des comptes de stockage, consultez [Système de stockage back-end cloud pour Stockage Blob Azure](avere-vfxt-add-storage.md#azure-blob-storage-cloud-core-filer).

### <a name="can-i-use-archive-blob-storage"></a>Puis-je utiliser le stockage d'objets blob d'archive ?

Non. Le contrat de niveau de service (SLA) du stockage d'archive n'est pas compatible avec les besoins d'accès en temps réel aux fichiers et aux répertoires du système Avere vFXT.

### <a name="can-i-use-cool-blob-storage"></a>Puis-je utiliser le stockage d'objets blob froid ?

Le stockage d’objets blob de niveau de stockage froid n’est généralement pas recommandé pour un serveur de fichiers Avere vFXT pour Azure. Le niveau de stockage froid offre des coûts de stockage plus bas mais des coûts d'exploitation plus élevés. (Pour plus d’informations, consultez la [tarification des objets blob de blocs](<https://azure.microsoft.com/pricing/details/storage/blobs/>).) Si les données sont accessibles, modifiées ou supprimées fréquemment, envisagez d’utiliser le niveau de stockage chaud.

Les [niveaux d’accès](../storage/blobs/storage-blob-storage-tiers.md#cool-access-tier) fournissent plus d’informations sur le moment où il peut être judicieux d’utiliser le stockage de niveau froid comme un système de stockage back-end vFXT.

### <a name="how-do-i-encrypt-the-blob-container"></a>Comment chiffrer le conteneur d'objets blob ?

Vous pouvez configurer le chiffrement d'objets blob dans Azure (recommandé) ou au niveau du système de stockage principal Avere vFXT.  

### <a name="can-i-use-my-own-encryption-key-for-a-blob-core-filer"></a>Puis-je utiliser ma propre clé de chiffrement pour un système de stockage principal d'objets blob ?

Par défaut, les données sont chiffrées à l'aide de clés gérées par Microsoft pour le stockage Blob, Table et File d'attente Azure ainsi que pour Azure Files. Vous pouvez apporter votre propre clé de chiffrement pour le stockage Blob et Azure Files. Si vous choisissez d'utiliser le chiffrement Avere vFXT, vous devez utiliser la clé générée par Avere et la stocker localement.

## <a name="purchasing"></a>Achat

### <a name="how-do-i-get-avere-vfxt-for-azure-licensing"></a>Comment obtenir une licence Avere vFXT pour Azure ?

Il est facile d’obtenir une licence Avere vFXT pour Azure dans la Place de marché Azure. Inscrivez-vous pour créer un compte Azure, puis suivez les instructions mentionnées dans [Déployer le cluster Avere vFXT](avere-vfxt-deploy.md) afin de créer un cluster Avere vFXT.

### <a name="how-much-does-avere-vfxt-cost"></a>Combien coûte le système Avere vFXT ?

Dans Azure, aucuns frais de licence supplémentaires ne sont facturés pour l’utilisation de clusters Avere vFXT. Les frais de stockage et autres frais de consommation Azure incombent aux clients.

### <a name="can-avere-vfxt-vms-be-run-as-low-priority"></a>Les machines virtuelles Avere vFXT peuvent-elles être exécutées en basse priorité ?

Non, les clusters Avere vFXT exigent un service « toujours activé ». Les clusters peuvent être désactivés quand ils ne sont pas nécessaires.

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer avec Avere vFXT pour Azure, consultez les articles suivants afin d'apprendre à planifier et à déployer votre propre système :

* [Planifier votre système Avere vFXT](avere-vfxt-deploy-plan.md)
* [Vue d’ensemble du déploiement](avere-vfxt-deploy-overview.md)
* [Se préparer à la création d'un cluster Avere vFXT](avere-vfxt-prereqs.md)
* [Déployer le cluster Avere vFXT](avere-vfxt-deploy.md)

Pour en savoir plus sur les fonctionnalités et les cas d'usage du système Avere vFXT, consultez [Avere vFXT pour Azure](https://azure.microsoft.com/services/storage/avere-vfxt/).