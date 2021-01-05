---
title: IBM DB2 pureScale sur Azure
description: Dans cet article, nous présentons une architecture d’exécution d’un environnement IBM DB2 pureScale sur Azure.
author: njray
manager: edprice
editor: edprice
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/09/2018
ms.author: edprice
ms.openlocfilehash: 0297c8674cc47a1d5f59fef196a60175244eaae2
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91978320"
---
# <a name="ibm-db2-purescale-on-azure"></a>IBM DB2 pureScale sur Azure

L’environnement IBM DB2 pureScale fournit un cluster de bases de données pour Azure avec haute disponibilité et scalabilité sur les systèmes d’exploitation Linux. Cet article présente une architecture d’exécution de DB2 pureScale sur Azure.

## <a name="overview"></a>Vue d’ensemble

Les entreprises ont longtemps utilisé des plateformes traditionnelles de système de gestion de base de données relationnelle (SGBDR) pour répondre à leurs besoins en matière de traitement transactionnel en ligne (OLTP). Aujourd’hui, nombreuses sont celles qui migrent leur environnement de base de données reposant sur un ordinateur central vers Azure afin d’étendre la capacité, de réduire les coûts et de maintenir une structure de coûts d’exploitation stable. La migration est souvent la première étape du processus de modernisation d’une plateforme héritée. 

Un client d’entreprise a récemment migré son environnement IBM DB2 s’exécutant sur z/OS vers IBM DB2 pureScale sur Azure. La solution de cluster de bases de données Db2 pureScale fournit haute disponibilité et scalabilité sur les systèmes d’exploitation Linux. Le client a exécuté Db2 avec succès en tant qu’instance autonome, avec montée en puissance sur une machine virtuelle unique dans un système à grande échelle sur Azure avant d’installer Db2 pureScale. 

Quoique différent de l’environnement d’origine, IBM DB2 pureScale sur Linux offre des fonctionnalité de haute disponibilité et d’évolutivité similaires à IBM DB2 pour z/OS exécuté dans une configuration Parallel Sysplex sur un ordinateur central. Dans ce scénario, le cluster est connecté via iSCSI à un cluster de stockage partagé. Nous avons utilisé le système de fichiers GlusterFS, qui est un système de fichiers distribué, scalable, gratuit et open source, optimisé spécialement pour le stockage cloud. Toutefois, IBM ne prend plus en charge cette solution. Pour continuer à bénéficier de la prise en charge d’IBM, vous devez utiliser un système de fichiers compatible iSCSI pris en charge. Microsoft offre l’option d’espaces de stockage direct

Cet article décrit l’architecture utilisée pour cette migration Azure. Le client a utilisé Red Hat Linux 7.4 pour tester la configuration. Cette version est disponible sur Azure Marketplace. Avant de choisir une distribution Linux, veillez à vérifier les versions actuellement prises en charge. Pour plus d’informations, consultez la documentation relative à [IBM DB2 pureScale](https://www.ibm.com/support/knowledgecenter/SSEPGG) et [GlusterFS](https://docs.gluster.org/en/latest/).

Cet article est un point de départ pour votre plan d’implémentation de DB2. Vos besoins seront différents, mais le même modèle de base s’appliquera. Ce modèle d’architecture peut également être utilisé pour les applications de traitement analytique en ligne (OLAP) sur Azure.

Cet article n’aborde pas les différences et les tâches de migration possibles dans le cadre de la migration d’une base de données IBM DB2 pour z/OS vers IBM DB2 pureScale s’exécutant sur Linux. Il ne propose pas non plus d’estimations de dimensionnement et d’analyses de charge de travail pour la migration de DB2 z/OS vers DB2 pureScale. 

Pour vous aider à choisir la meilleure architecture DB2 pureScale pour votre environnement, nous vous recommandons d’estimer globalement le dimensionnement et d’élaborer une hypothèse. Sur le système source, veillez à prendre en compte DB2 z/OS Parallel Sysplex avec l’architecture de partage de données, la configuration Coupling Facility et les statistiques d’utilisation DDF (Distributed Data Facility).

> [!NOTE]
> Cet article décrit une approche de migration de DB2, mais il en existe d’autres. Par exemple, DB2 pureScale peut également s’exécuter dans des environnements locaux virtualisés. IBM prend en charge DB2 sur Microsoft Hyper-V dans différentes configurations. Pour plus d’informations, consultez [Architecture de visualisation DB2 pureScale](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.qb.server.doc/doc/r0061462.html) dans IBM Knowledge Center.

## <a name="architecture"></a>Architecture

Pour prendre en charge la haute disponibilité et l’évolutivité sur Azure, vous pouvez utiliser une architecture de données partagées scale-out pour DB2 pureScale. La migration du client a utilisé l’exemple d’architecture suivant.

![DB2 PureScale sur des machines virtuelles Azure montrant le stockage et la mise en réseau](media/db2-purescale-on-azure/pureScaleArchitecture.png "Db2 pureScale sur des machines virtuelles Azure montrant le stockage et la mise en réseau")


Le diagramme illustre les couches logiques nécessaires à un cluster DB2 pureScale. Ces couches incluent des machines virtuelles pour un client, pour la gestion, pour la mise en cache, pour le moteur de base de données et pour le stockage partagé. 

En plus des nœuds du moteur de base de données, le diagramme comporte deux nœuds utilisés pour les installations de mise en cache de cluster. Un minimum de deux nœuds sont utilisés pour le moteur de base de données lui-même. Un serveur DB2 qui appartient à un cluster pureScale est appelé membre. 

Le cluster est connecté via iSCSI à un cluster de stockage partagé comportant trois nœuds afin de fournir un stockage scale-out et une haute disponibilité. DB2 pureScale est installé sur les machines virtuelles Azure exécutant Linux.

Cette approche est un modèle que vous pouvez modifier selon la taille et l’échelle de votre organisation. Elle se base sur les éléments suivants :

-   Deux membres de la base de données ou plus sont combinés avec au moins deux nœuds d’installations de mise en cache. Les nœuds gèrent un pool de mémoires tampons global pour les services de mémoire partagée et de gestionnaire de verrou global afin de contrôler l’accès partagé et la contention de verrou des membres actifs. Un nœud CF joue le rôle de nœud principal et l’autre, en temps que nœud secondaire, joue le rôle de nœud CF de basculement. Pour éviter un point de défaillance unique dans l’environnement, un cluster DB2 pureScale nécessite au moins quatre nœuds.

-   Stockage partagé hautement performant (illustré dans la taille P30 dans le diagramme). Chaque nœud utilise ce stockage.

-   Réseau hautes performances pour les membres de données et le stockage partagé.

### <a name="compute-considerations"></a>Considérations relatives à la capacité de calcul

Cette architecture exécute les couches application, stockage et données sur des machines virtuelles Azure. Les [scripts de configuration de déploiement](https://aka.ms/db2onazure) créent les éléments suivants :

-   Un cluster DB2 pureScale. Le type de ressources de calcul dont vous avez besoin sur Azure dépend de votre installation. En règle générale, vous pouvez utiliser deux approches :

    -   Utiliser un réseau de type calcul haute performance (HPC) à plusieurs nœuds, où des instances de petite à moyenne taille accèdent au stockage partagé. Pour ce type de configuration HPC, les [machines virtuelles](../sizes.md) de série E à mémoire optimisée Azure ou de série L à stockage optimisé fournissent les ressources de calcul nécessaires.

    -   Utiliser moins de grandes instances de machines virtuelles pour les moteurs de données. Pour les grandes instances, les plus grandes machines virtuelles de [série M](https://azure.microsoft.com/pricing/details/virtual-machines/series/) à mémoire optimisée sont idéales pour les lourdes charges de travail en mémoire. Vous aurez peut-être besoin d’une instance dédiée, selon la taille de la partition logique qui est utilisée pour exécuter DB2.

-   L’installation de mise en cache DB2 utilise des machines virtuelles à mémoire optimisée, comme la série E ou la série L.

-   Un cluster de stockage partagé utilise des machines virtuelles \_DS4\_v2 Standard exécutant Linux.

-   Le « jumpbox » de gestion est une machine virtuelle \_DS2\_v2 standard exécutant Linux.  Une alternative est Azure Bastion, service qui offre une expérience RDP/SSH sécurisée pour toutes les machines virtuelles de votre réseau virtuel.

-   Le client est une machine virtuelle \_DS3\_v2 standard exécutant Windows (pour le test).

-   *Facultatif*. Un serveur témoin. Ceci est nécessaire uniquement avec certaines versions antérieures de Db2 pureScale. Cet exemple utilise une machine virtuelle \_DS3\_v2 standard exécutant Linux (utilisé pour Db2 pureScale).

> [!NOTE]
> Un cluster DB2 pureScale nécessite au moins deux instances de DB2. Il nécessite aussi une instance de cache et une instance de gestionnaire de verrou.

### <a name="storage-considerations"></a>Considérations relatives au stockage

Comme Oracle RAC, DB2 pureScale est une base de données scale-out d’E/S de bloc hautes performances. Nous vous recommandons d’utiliser l’option [Stockage SSD Azure](../disks-types.md) la plus élevée répondant à vos besoins. Les options de stockage plus petites peuvent convenir aux environnements de développement et de test, tandis que les environnements de production nécessitent souvent de plus grandes capacités de stockage. L’exemple d’architecture utilise [P30](https://azure.microsoft.com/pricing/details/managed-disks/) en raison de son rapport entre IOPS et taille/prix. Quelle que soit la taille, utilisez le Stockage Premium pour de meilleures performances.

DB2 pureScale utilise une architecture de partage intégral, où toutes les données sont accessibles à partir de tous les nœuds de cluster. Stockage Premium doit être partagé entre plusieurs instances, que ce soit à la demande ou sur des instances dédiées.

Un grand cluster DB2 pureScale peut nécessiter 200 téraoctets (To) ou plus de stockage partagé Premium, avec un IOPS de 100 000. DB2 pureScale prend en charge une interface de bloc iSCSI que vous pouvez utiliser sur Azure. L’interface iSCSI nécessite un cluster de stockage partagé que vous pouvez implémenter avec S2D ou un autre outil. Ce type de solution crée un appareil de réseau de zone de stockage virtuel (vSAN) dans Azure. DB2 pureScale utilise le réseau vSAN pour installer le système de fichiers en cluster qui est utilisé pour partager des données entre plusieurs machines virtuelles.

### <a name="networking-considerations"></a>Mise en réseau - Éléments à prendre en compte

IBM recommande une mise en réseau InfiniBand pour tous les membres d’un cluster DB2 pureScale. DB2 pureScale utilise également l’accès direct à la mémoire à distance (RDMA), le cas échéant, pour les installations de mise en cache.

Pendant l’installation, vous créez un [groupe de ressources](../../azure-resource-manager/management/overview.md) Azure pour contenir toutes les machines virtuelles. En règle générale, vous regroupez les ressources en fonction de leur durée de vie et de qui va les gérer. Les machines virtuelles dans cette architecture nécessitent une [mise en réseau accélérée](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/). Il s’agit d’une fonctionnalité Azure qui fournit une latence du réseau cohérente et très faible par le biais d’une virtualisation des E/S sur une racine unique (SR-IOV) à une machine virtuelle.

Chaque machine virtuelle est déployée dans un réseau virtuel qui comporte des sous-réseaux : principal, frontal Gluster FS (gfsfe), back-end Gluster FS (bfsbe), DB2 pureScale (db2be) et frontal DB2 pureScale (db2fe). Le script d’installation crée également les [cartes réseau](./multiple-nics.md) principales sur les machines virtuelles dans le sous-réseau principal.

Utilisez des [groupes de sécurité réseau](../../virtual-network/virtual-network-vnet-plan-design-arm.md) pour limiter le trafic réseau au sein du réseau virtuel et pour isoler les sous-réseaux.

Sur Azure, DB2 pureScale doit utiliser la connexion réseau TCP/IP pour le stockage.

## <a name="next-steps"></a>Étapes suivantes

-   [Déployer cette architecture sur Azure](deploy-ibm-db2-purescale-azure.md)