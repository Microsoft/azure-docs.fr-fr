---
title: Disponibilité de SAP HANA au sein d’une région Azure | Microsoft Docs
description: Décrivez les opérations SAP HANA sur les machines virtuelles Azure natives dans une région Azure.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/27/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e522e358a1b76cea08dac550b33d7a2dfa7d926d
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94950225"
---
# <a name="sap-hana-availability-within-one-azure-region"></a>Disponibilité de SAP HANA au sein d’une région Azure
Cet article décrit plusieurs scénarios de disponibilité au sein d’une région Azure. Azure dispose de plusieurs régions, réparties à travers le monde. Pour obtenir la liste des régions Azure, voir [Régions Azure](https://azure.microsoft.com/regions/). Pour le déploiement de SAP HANA sur des machines virtuelles au sein d’une région Azure, Microsoft offre le déploiement d’une machine virtuelle unique avec une instance HANA. Pour une disponibilité accrue, vous pouvez déployer deux machines virtuelles avec deux instances HANA au sein d’un [groupe à haute disponibilité Azure](../../windows/tutorial-availability-sets.md) qui utilise la réplication de système HANA pour des raisons de disponibilité. 

Actuellement, Azure offre les [Zones de disponibilité Azure](../../../availability-zones/az-overview.md). Cet article ne décrit pas en détail les zones de disponibilité. Toutefois, nous incluons une présentation générale sur l’utilisation des groupes à haute disponibilité par rapport aux zones de disponibilité.

Les régions Azure proposant des zones de disponibilité disposent de plusieurs centres de données. Les centres de données sont indépendants de la fourniture de la source d’alimentation, du refroidissement et du réseau. Nous offrons différentes zones au sein d’une seule région Azure pour déployer des applications sur deux ou trois des zones de disponibilité offertes. Si les problèmes d’alimentation et de réseau n’affectent qu’une seule infrastructure de zone de disponibilité Azure, le déploiement de votre application au sein d’une région Azure devrait toujours opérationnel. Une capacité réduite peut se produire. Par exemple, des machines virtuelles dans une zone peuvent être perdues, mais les machines virtuelles dans les deux autres zones seront toujours en exécution. 
 
Un groupe à haute disponibilité Azure est une fonctionnalité de regroupement logique qui aide à assurer que les ressources de machine virtuelle que vous y incluez sont isolées les unes des autres lors de leur déploiement dans un centre de données Azure. Azure veille à ce que les machines virtuelles que vous placez dans un groupe à haute disponibilité s’exécutent sur plusieurs serveurs physiques, racks de calcul, unités de stockage et commutateurs réseau. Dans certaines documentations Azure, cette configuration est considérée comme des placements dans différents [Mise à jour et domaines d’erreur](../../manage-availability.md). En règle générale, ces placements se trouvent dans un centre de données Azure. En imaginant que les problèmes d’alimentation et de réseau affectent le centre de données que vous déployez, toutes vos capacités dans une région Azure seraient aussi affectées.

Le placement des centres de données représentant les zones de disponibilité Azure est un compromis entre une latence réseau entre des services déployés dans différentes zones et une distance spécifique entre des centres de données. Idéalement, les catastrophes naturelles n’affectent pas l’alimentation, le réseau et l’infrastructure de toutes vos zones de disponibilité dans cette région. Toutefois, comme nous l’avons déjà vu avec des catastrophes naturelles d’envergure, les zones de disponibilité peuvent ne pas fournir autant de disponibilité que vous le souhaitez au sein d’une région. Pensez à l’ouragan Maria qui a touché l’ile de Porto Rico le 20 septembre 2017. L’ouragan a entrainé une coupure de courant de la quasi-totalité de cette ile de 145 kilomètres.

## <a name="single-vm-scenario"></a>Scénario de machine virtuelle unique

Dans un scénario de machine virtuelle unique, vous créez une machine virtuelle Azure pour l’instance SAP HANA. Vous utilisez le stockage Premium Azure pour héberger le disque du système d’exploitation et tous vos disques de données. Le contrat SLA de disponibilité de 99,9 % d’Azure et les contrats SLA d’autres composants Azure vous suffisent à réaliser vos contrats SLA de disponibilité pour vos clients. Dans ce scénario, vous n’avez pas besoin de tirer profit d’un groupe à haute disponibilité pour des machines virtuelles qui exécutent le gestionnaire de bases de données. Dans ce scénario, vous vous appuyez sur deux fonctionnalités :

- Redémarrage automatique de machine virtuelle Azure (aussi appelé Azure Service Healing)
- Redémarrage automatique SAP HANA

Le redémarrage automatique de machine virtuelle Azure, ou Service Healing, est une fonctionnalité Azure qui fonctionne sur deux niveaux :

- L’hôte serveur Azure vérifie l’intégrité d’une machine virtuelle hébergée sur le serveur hôte.
- Le contrôleur de structure Azure surveille l’intégrité et la disponibilité du serveur hôte.

Une fonctionnalité d’analyse de l’intégrité surveille l’intégrité de chaque machine virtuelle hébergée sur un serveur hôte Azure. Si une machine virtuelle passe en état défaillant, un redémarrage peut être initié par l’agent hôte Azure qui vérifie l’intégrité de la machine virtuelle. Le contrôleur de structure vérifie l’intégrité de l’hôte en vérifiant les nombreux paramètres qui peuvent indiquer des problèmes avec le matériel hôte. Il vérifie également l’accessibilité de l’hôte via le réseau. Une indication de problèmes avec l’hôte peut entraîner les événements suivants :

- Si l’hôte signale un état défaillant, un redémarrage de l’hôte et des machines virtuelles exécutées sur l’hôte est déclenché.
- Si l’hôte n’a pas recouvré son intégrité après un redémarrage réussi, un redéploiement des machines virtuelles qui étaient initialement sur le nœud maintenant non intègre sur un serveur hôte intègre est lancé. Dans ce cas, l’hôte d’origine est marqué comme défaillant. Il ne sera pas utilisé pour des déploiements ultérieurs jusqu'à ce qu’il soit effacé ou remplacé.
- Si l’hôte défaillant rencontre des difficultés lors de son redémarrage, un redémarrage immédiat des machines virtuelles sur un hôte intègre est déclenché. 

Avec la surveillance d’hôte et de machine virtuelle fournie par Azure, les machines virtuelles Azure qui subissent des problèmes d’hôte sont automatiquement redémarrées sur un hôte Azure intègre. 

>[!IMPORTANT]
>La réparation des services Azure ne redémarre pas les machines virtuelles Linux où le système d’exploitation invité est affecté par une alerte sur le noyau. Les paramètres par défaut des versions de Linux couramment utilisées n’autorisent pas redémarrer automatiquement les machines virtuelles ou le serveur contenant un noyau Linux en alerte. Le paramétrage par défaut prévoit au contraire de maintenir le système d’exploitation dans un état d’alerte du noyau pour pouvoir y attacher un débogueur du noyau et procéder à des analyses. Azure suit ce comportement en ne redémarrant pas automatiquement une machine virtuelle dont le système d’exploitation invité est dans un tel état. Nous partons de l’hypothèse que de tels événements sont extrêmement rares. Vous pouvez remplacer le comportement par défaut pour permettre un redémarrage de la machine virtuelle. Pour modifier le comportement par défaut, activez le paramètre « kernel.panic » dans /etc/sysctl.conf. La durée pour ce paramètre est en secondes. Les valeurs recommandées courantes sont d’attendre 20 à 30 secondes avant de déclencher le redémarrage via ce paramètre. Voir aussi <https://gitlab.com/procps-ng/procps/blob/master/sysctl.conf>.

La deuxième fonctionnalité sur laquelle vous pouvez vous appuyer dans un tel scénario est le redémarrage automatique du service HANA qui s’exécute dans une machine virtuelle après qu’elle ait été redémarrée. Vous pouvez configurer le [service de redémarrage automatique HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/cf10efba8bea4e81b1dc1907ecc652d3.html) via les services de surveillance des différents services HANA.

Vous pouvez améliorer ce scénario de machine virtuelle unique en ajoutant un nœud de basculement froid à une configuration SAP HANA. Dans la documentation de SAP HANA, cette configuration est appelée [Héberger le basculement automatique](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/ae60cab98173431c97e8724856641207.html). Cette configuration peut être logique dans une situation de déploiement local, où le matériel de serveur est limité et où vous dédiez un seul nœud de serveur comme nœud d’hôte de basculement automatique pour un ensemble d’hôtes de production. Mais dans Azure, où l’infrastructure sous-jacente fournit un serveur cible intègre pour un redémarrage réussi d’une machine virtuelle, le déploiement du basculement automatique d’hôte de SAP HANA n’est pas logique. La réparation des services Azure fait qu’il n’existe aucune architecture de référence qui prévoit un nœud de veille pour le basculement automatique de l’hôte SAP HANA. 

### <a name="special-case-of-sap-hana-scale-out-configurations-in-azure"></a>Cas spécial des configurations de scale-out SAP HANA dans Azure
La haute disponibilité pour les configurations de scale-out SAP HANA repose sur la réparation des services des machines virtuelles Azure et le redémarrage de l’instance SAP HANA au moment où la machine virtuelle est à nouveau opérationnelle. Les architectures à haute disponibilité basées sur la réplication du système HANA seront présentées par la suite. 


## <a name="availability-scenarios-for-two-different-vms"></a>Scénarios de disponibilité pour deux machines virtuelles différentes

Si vous utilisez deux machines virtuelles Azure au sein d’un groupe à haute disponibilité Azure, vous pouvez augmenter le temps entre ces deux machines virtuelles, si elles sont placées dans un groupe à haute disponibilité Azure au sein d’une région Azure. L’installation de base dans Azure ressemble à ceci :

![Diagramme de deux machines virtuelles avec toutes les couches](./media/sap-hana-availability-one-region/two_vm_all_shell.PNG)

Pour illustrer les différents scénarios de disponibilité, quelques-unes des couches dans le diagramme sont omises. Le diagramme montre uniquement les couches qui décrivent les machines virtuelles, les hôtes, les groupes à haute disponibilité et les régions Azure. Des instances de Azure Virtual Network, des groupes de ressources et des abonnements ne jouent pas un rôle dans les scénarios décrits dans cette section.

### <a name="replicate-backups-to-a-second-virtual-machine"></a>Répliquer des sauvegardes vers une seconde machine virtuelle

L’une des configurations les plus rudimentaires consiste à utiliser des sauvegardes. En particulier, vous pouvez avoir des sauvegardes des journaux des transactions depuis une machine virtuelle vers une autre machine virtuelle Azure. Vous pouvez choisir le type de stockage Azure. Dans cette configuration, vous êtes chargé de scripter la copie des sauvegardes planifiées conduites de la première machine virtuelle vers la seconde. Si vous devez utiliser les instances de la seconde machine virtuelle, vous devez restaurer les sauvegardes complètes différentielles/incrémentielles et de fichier journal à l’endroit requis. 

L’architecture ressemble à ceci :

![Diagramme de l’architecture de deux machines virtuelles avec réplication du stockage](./media/sap-hana-availability-one-region/two_vm_storage_replication.PNG) 

Cette configuration n’est pas bien adaptée à la réalisation de bons temps d’objectif de point de récupération (RPO) et d’objectif de délai de récupération (RTO). Les temps de RTO, plus particulièrement, souffriraient en raison du besoin de restaurer entièrement toute la base de données à l’aide des sauvegardes copiées. Toutefois, cette configuration est utile pour récupérer d’une suppression de données involontaire sur les instances principales. Avec cette configuration, vous êtes en mesure de restaurer à un certain point dans le temps, d’extraire les données et d’importer les données supprimées dans votre instance principale en tout temps. Ainsi, il est peut-être logique d’utiliser une méthode de copie de sauvegarde avec d’autres fonctionnalités de haute disponibilité. 

Pendant la copie des sauvegardes, vous serez peut-être en mesure d’utiliser une machine virtuelle plus petite autre que la machine virtuelle principale qui exécute l’instance SAP HANA. Gardez à l’esprit que vous pouvez attacher un plus petit nombre de disques durs virtuels sur les machines virtuelles plus petites. Pour plus d’information sur les limites des types de machines virtuelles individuelles, consultez [Tailles des machines virtuelles Linux dans Azure](../../sizes.md).

### <a name="sap-hana-system-replication-without-automatic-failover"></a>Réplication de système SAP HANA sans basculement automatique

Les scénarios décrits dans cette section utilisent la réplication du système SAP HANA. Pour obtenir la documentation de SAP, consultez [Réplication du système](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html). Les scénarios sans basculement automatique ne sont pas courants pour les configurations au sein d’une seule région Azure. Une configuration sans basculement automatique, même si elle vous évite de configurer Pacemaker, vous oblige de procéder à une surveillance et un basculement manuels. Cela prend du temps et de l’énergie, c’est pourquoi la plupart des clients utilisent plutôt la réparation des services Azure. Il existe quelques cas extrêmes où cette configuration peut être utile en termes de scénarios d’échec. Ou, dans certains cas, un client peut souhaiter plus d’efficacité.

#### <a name="sap-hana-system-replication-without-auto-failover-and-without-data-preload"></a>Réplication de système SAP HANA sans basculement automatique ni données pré-chargées

Dans ce scénario, vous utilisez la réplication du système SAP HANA pour déplacer des données de façon synchrone pour atteindre un RPO à 0. En revanche, vous disposez d’un RTO suffisamment long pour ne pas avoir besoin du basculement ou du préchargement des données dans le cache d’instance HANA. Dans ce cas, il est possible de réaliser des économies plus importantes dans votre configuration en effectuant les actions suivantes :

- Exécuter une autre instance SAP HANA dans la deuxième machine virtuelle. L’instance SAP HANA dans la deuxième machine virtuelle utilise une grande partie de la mémoire de la machine virtuelle. Dans le cas d’un basculement vers la deuxième machine virtuelle, vous devez arrêter l’instance SAP HANA en cours d’exécution qui charge les données dans la deuxième machine virtuelle, afin que les données répliquées puissent être chargées dans le cache de l’instance HANA ciblée dans la deuxième machine virtuelle.
- Utilisez une taille de machine virtuelle réduite pour la deuxième machine virtuelle. Si un basculement se produit, vous disposez d’une étape supplémentaire avant le basculement manuel. Dans cette étape, vous redimensionnez la machine virtuelle à la taille de la machine virtuelle source. 
 
Le scénario ressemble à ceci :

![Diagramme de deux machines virtuelles avec réplication de stockage](./media/sap-hana-availability-one-region/two_vm_HSR_sync_nopreload.PNG)

> [!NOTE]
> Même si vous n’utilisez pas le préchargement de données dans la cible de réplication du système HANA, vous avez besoin d’au moins 64 Go de mémoire. Vous avez également besoin de suffisamment de mémoire en plus des 64 Go pour conserver les données de stockage de lignes dans la mémoire de l’instance cible.

#### <a name="sap-hana-system-replication-without-auto-failover-and-with-data-preload"></a>Réplication de système SAP HANA sans basculement automatique et avec des données pré-chargées

Dans ce scénario, les données répliquées vers l’instance HANA dans la deuxième machine virtuelle sont préchargées. Cela élimine les deux avantages de l’absence de préchargement des données. Dans ce cas, vous ne pouvez pas exécuter d’autres systèmes SAP HANA sur la deuxième machine virtuelle, Vous ne pouvez pas utiliser une taille de machine virtuelle réduite. Par conséquent, les clients implémentent rarement ce scénario.

### <a name="sap-hana-system-replication-with-automatic-failover"></a>Réplication de système SAP HANA avec basculement automatique

Dans la configuration de disponibilité standard et la plus courante au sein d’une région Azure, deux machines virtuelles Azure exécutant SLES Linux disposent d’un cluster de basculement défini. Le cluster SLES Linux est basé sur l’infrastructure [Pacemaker](./high-availability-guide-suse-pacemaker.md), conjointement avec un appareil [STONITH](./high-availability-guide-suse-pacemaker.md#create-azure-fence-agent-stonith-device). 

Côté SAP HANA, le mode de réplication utilisé est synchronisé et un basculement automatique est configuré. Dans la deuxième machine virtuelle, l’instance SAP HANA agit comme un nœud de serveur de secours. Le nœud de serveur de secours reçoit un flux de données synchrone d’enregistrements de modification depuis l’instance de SAP HANA principale. Tandis que les transactions sont acceptées par l’application au nœud HANA principal, ce dernier attend la confirmation de réception de l’enregistrement de la part du nœud SAP HANA secondaire avant de confirmer la réception à l’application. SAP HANA dispose de deux modes de réplication synchrone différents. Pour en savoir plus et pour connaître les différences entre les deux modes de réplication synchrone, consultez l’article SAP [Modes de réplication pour la réplication de système SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.02/en-US/c039a1a5b8824ecfa754b55e0caffc01.html).

La configuration générale ressemble à ceci :

![Diagramme de deux machines virtuelles avec réplication de stockage et basculement](./media/sap-hana-availability-one-region/two_vm_HSR_sync_auto_pre_preload.PNG)

Vous pouvez choisir cette solution car elle vous permet d’obtenir un objectif de point de récupération égal à 0 et un objectif de délai de récupération faible. Configurez la connectivité du client SAP HANA de façon à ce que les clients SAP HANA utilisent l’adresse IP virtuelle pour se connecter à la configuration de réplication de système HANA. Cette configuration évite de reconfigurer l’application en cas de basculement sur le nœud secondaire. Dans ce scénario, les références SKU de machine virtuelle Azure pour les machines virtuelles principales et secondaires doivent être identiques.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des instructions sur la mise en place de ces configurations dans Azure, consultez :

- [Installer la réplication de système SAP HANA sur des machines virtuelles Azure](sap-hana-high-availability.md)
- [Haute disponibilité de SAP HANA sur des machines virtuelles Azure](https://blogs.sap.com/2018/01/08/your-sap-on-azure-part-4-high-availability-for-sap-hana-using-system-replication/)

Pour plus d’informations sur la disponibilité SAP HANA dans l’ensemble des régions Azure, consultez :

- [Disponibilité de SAP HANA dans l’ensemble des régions Azure](./sap-hana-availability-across-regions.md)