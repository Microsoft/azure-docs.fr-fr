---
title: Vue d’ensemble de SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Vue d’ensemble du déploiement de SAP HANA sur Azure (grandes instances).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/12/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 0144c57549e374404d3dde0fe997ba41adc8606b
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967582"
---
#  <a name="what-is-sap-hana-on-azure-large-instances"></a>Qu’est-ce que SAP HANA sur Azure (grandes instances) ?

SAP HANA sur Azure (grandes instances) est une solution Azure unique. En plus de fournir des machines virtuelles à des fins de déploiement et d’exécution de SAP HANA, Azure vous offre la possibilité d’exécuter et de déployer SAP HANA sur des serveurs nus qui vous sont dédiés. La solution SAP HANA sur Azure (grandes instances) s’appuie sur un matériel nu de type serveur/hôte non partagé, qui vous est assigné. Le matériel de serveur est incorporé dans des tampons plus volumineux qui contiennent l’infrastructure de calcul/serveur, de réseau et de stockage. En tant que combinaison, il s’agit d’une intégration de centre de données adaptée (TDI) à HANA certifiée. SAP HANA sur Azure (grandes instances) propose différentes références SKU ou tailles de serveurs. Les unités peuvent disposer de 36 cœurs de processeur Intel et de 768 Go de mémoire, et aller jusqu’à 480 cœurs de processeur Intel avec 24 To de mémoire.

L’isolation du client dans le tampon d’infrastructure s’effectue dans le cadre des abonnés, de la manière suivante :

- **Réseau** : isolation des clients au sein de la pile d’infrastructures au travers de réseaux virtuels par abonné assigné à un client. Un locataire est assigné à un seul client. Un client peut avoir plusieurs locataires. L’isolation réseau des abonnés interdit la communication réseau entre les abonnés au niveau du tampon d’infrastructure, même si les abonnés appartiennent au même client.
- **Composants de stockage** : isolation par le biais de machines virtuelles de stockage auxquelles des volumes de stockage sont assignés. Les volumes de stockage ne peuvent être assignés qu’à une seule machine virtuelle de stockage. Une machine virtuelle de stockage est assignée exclusivement à un seul locataire dans la pile d’infrastructure certifiée SAP HANA TDI. Par conséquent, les volumes de stockage assignés à une machine virtuelle de stockage sont accessibles dans un seul abonné associé. Et ils ne sont pas visibles entre les différents abonnés déployés.
- **Serveur ou hôte** : une unité de serveur ou hôte n’est pas partagée entre les clients ou les abonnés. Un serveur ou un hôte déployé pour un client constitue une unité de calcul nue atomique qui est assignée à un seul locataire. *Aucun* partitionnement matériel ou logiciel utilisé ne peut vous conduire à partager un hôte ou un serveur avec un autre client. Les volumes de stockage qui sont assignés à la machine virtuelle de stockage du locataire spécifique sont montés sur ce type de serveur. Une ou plusieurs unités de serveur de différentes références SKU peuvent être exclusivement assignées à un seul locataire.
- Au sein d’une solution SAP HANA sur le tampon d’infrastructure Azure (grandes instances), plusieurs abonnés différents sont déployés et isolés les uns par rapport aux autres, au travers des concepts d’abonné sur les niveaux de mise en réseau, de stockage et de calcul. 


Ces unités de serveur nues exécutent uniquement SAP HANA. La couche Application SAP ou la couche intermédiaire de la charge de travail est en cours d’exécution dans les machines virtuelles. Les tampons d’infrastructure qui exécutent SAP HANA sur des unités Azure (grandes instances) sont connectés aux principaux fournisseurs de services réseau Azure. De cette façon, la connectivité à faible latence entre des unités SAP HANA sur Azure (grandes instances) et des machines virtuelles est fournie.

Depuis juillet 2019, nous faisons la distinction entre deux révisions différentes des tampons Grande instance HANA et les emplacements des déploiements :

- « Révision 3 » (Rév. 3) : il s’agit des tampons mis à disposition du client pour le déploiement avant juillet 2019
- « Révision 4 » (Rév. 4) : nouvelle conception de tampon déployée à proximité des hôtes de machine virtuelle Azure et qui jusqu’à présent sont disponibles dans les régions Azure suivantes :
    -  USA Ouest 2 
    -  USA Est 
    -  Europe Ouest
    -  Europe Nord


Ce document fait partie des nombreux documents qui traitent de SAP HANA sur Azure (grandes instances). Ce document présente l’architecture de base, les responsabilités et les services fournis par la solution. Il présente également les capacités de haut niveau de la solution. Pour la plupart des autres domaines, tels que la mise en réseau et la connectivité, quatre autres documents couvrent les détails et les informations déroulantes. La documentation de SAP HANA sur Azure (grandes instances) n’aborde pas les aspects de l’installation de SAP NetWeaver ni les déploiements de SAP NetWeaver dans des machines virtuelles. SAP NetWeaver sur Azure est couvert dans des documents distincts disponibles dans le même conteneur de documentation Azure. 


Les différents documents de grande instance HANA couvrent les domaines suivants :

- [Vue d’ensemble et architecture de SAP HANA (grandes instances) sur Azure](hana-overview-architecture.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Infrastructure et connectivité à SAP HANA (grandes instances) sur Azure](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Installer et configurer SAP HANA (grandes instances) sur Azure](hana-installation.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Haute disponibilité et récupération d’urgence de SAP HANA (grandes instances) sur Azure](hana-overview-high-availability-disaster-recovery.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Résolution des problèmes et analyse de SAP HANA (grandes instances) sur Azure](troubleshooting-monitoring.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
- [Configuration de la haute disponibilité dans SUSE à l’aide de STONITH](./ha-setup-with-stonith.md)
- [Sauvegarde et restauration du système d’exploitation pour les SKU Type II des tampons Révision 3](./os-backup-type-ii-skus.md)
- [Enregistrer sur de grandes instances SAP HANA avec une réservation Azure](../../../cost-management-billing/reservations/prepay-hana-large-instances-reserved-capacity.md)

**Étapes suivantes**
- Voir [Maîtriser la terminologie](hana-know-terms.md)
