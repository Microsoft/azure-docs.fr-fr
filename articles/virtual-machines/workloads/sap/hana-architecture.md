---
title: Architecture de SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Architecture de SAP HANA sur Azure (grandes instances).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/25/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a23a177945316bca89aa5cbcc46f840213dbe832
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94964811"
---
# <a name="sap-hana-large-instances-architecture-on-azure"></a>Architecture SAP HANA (grandes instances) sur Azure

À un niveau élevé, SAP HANA sur la solution Azure (grandes instances) dispose de la couche Application SAP résidant dans des machines virtuelles. La couche de bases de données réside sur le matériel configuré TDI SAP situé dans un tampon de grande Instance dans la même région Azure connectée à Azure IaaS.

> [!NOTE]
> Déployez la couche Application SAP dans la même région Azure que la couche SGBD SAP. Cette règle est bien documentée dans les informations publiées sur les charges de travail SAP sur Azure. 

L’architecture globale de SAP HANA sur Azure (grandes instances) fournit une configuration matérielle certifiée TDI SAP, c’est-à-dire un serveur non virtualisé, nu et hautes performances pour la base de données SAP HANA. Elle fournit également la capacité et la flexibilité d’Azure pour mettre à l’échelle les ressources pour que la couche Application SAP réponde à vos besoins.

![Présentation de l’architecture de SAP HANA sur Azure (grandes instances)](./media/hana-overview-architecture/image1-architecture.png)

L’architecture présentée est divisée en trois sections :

- **À droite** : affiche une infrastructure sur site qui exécute différentes applications dans les centres de données afin que les utilisateurs finaux puissent accéder à des applications métier, telles que SAP. Dans l’idéal, cette infrastructure locale est connectée à Azure avec [ExpressRoute](https://azure.microsoft.com/services/expressroute/).

- **Au centre** : montre Azure IaaS et, dans ce cas, l’utilisation des machines virtuelles pour héberger SAP ou d’autres applications qui utilisent SAP HANA en tant que système SGBD. Les instances HANA plus petites qui fonctionnent avec la mémoire que les machines virtuelles fournissent sont déployées dans des machines virtuelles avec leur couche Application. Pour plus d’informations sur les machines virtuelles, consultez[Machines virtuelles](https://azure.microsoft.com/services/virtual-machines/).

   Les services réseau Azure permettent de regrouper des systèmes SAP ainsi que d’autres applications dans des réseaux virtuels. Ces réseaux virtuels se connectent à des systèmes locaux et à SAP HANA sur Azure (grandes instances).

   Pour les bases de données et les applications SAP NetWeaver prises en charge pour être exécutées dans Azure, voir [Remarque sur la prise en charge SAP n° 1928533 – Applications SAP sur Azure : Produits et types de machines virtuelles Azure pris en charge](https://launchpad.support.sap.com/#/notes/1928533). Pour la documentation sur le déploiement des solutions SAP sur Azure, consultez :

  -  [Utiliser SAP sur des machines virtuelles Windows](./get-started.md?toc=/azure/virtual-machines/linux/toc.json)
  -  [Utiliser des solutions SAP sur des machines virtuelles Azure](get-started.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

- **À gauche** : montre le matériel certifié TDI SAP HANA dans le tampon de grande instance Azure. Les unités de grande instance HANA sont connectées aux réseaux virtuels de votre abonnement Azure à l’aide de la même technologie que la connectivité du système local à Azure. Depuis le mois de mai 2019, une optimisation a été introduite, qui permet la communication entre les unités d’instance de grande taille HANA et les machines virtuelles Azure sans implication de la passerelle ExpressRoute. Cette optimisation appelée ExpressRoute Fast Path est présentée dans cette architecture (lignes rouges). 

Le tampon de grande instance Azure combine les composants suivants :

- **Calcul** : serveurs basés sur différentes générations de processeurs Intel Xeon qui offrent la capacité de calcul nécessaire et sont certifiés SAP HANA.
- **Réseau** : structure réseau haut débit unifiée qui relie les composants de calcul, de stockage et LAN.
- **Stockage** : infrastructure de stockage accessible via une infrastructure de réseau unifiée. La capacité de stockage spécifique fournie varie selon la configuration SAP HANA sur Azure (grandes instances) spécifique déployée. Une capacité de stockage supplémentaire est disponible pour un coût mensuel supérieur.

Dans l’infrastructure multilocataire du tampon de grande instance, les clients sont déployés en tant que locataires isolés. Au moment du déploiement de l’abonné, nommez un abonnement Azure au sein de votre inscription Azure. Cet abonnement Azure correspond à la grande instance HANA facturée. Ces locataires ont une relation 1:1 avec l’abonnement Azure. En termes de réseau, il est possible d’accéder à une unité de grande instance HANA déployée sur un abonné dans une région Azure à partir de différents réseaux virtuels qui appartiennent à différents abonnements Azure. Ces abonnements Azure doivent appartenir à la même inscription Azure. 

Comme avec les machines virtuelles, SAP HANA sur Azure (grandes instances) est proposé dans plusieurs régions Azure. Pour offrir des fonctionnalités de récupération d’urgence, vous pouvez choisir d’accepter. Les différents tampons de grande instance d’une même région géopolitique sont connectés entre eux. Par exemple, les tampons de grande instance HANA des USA Ouest et des USA Est sont connectés via une liaison réseau dédiée à des fins de réplication de la récupération d’urgence. 

Tout comme vous avez le choix entre différents types de machines virtuelles avec des machines virtuelles Azure, vous pouvez choisir parmi différentes références SKU de la grande instance HANA adaptées aux divers types de charges de travail de SAP HANA. SAP applique la mémoire aux rapports de socket de processeur pour différentes charges de travail en fonction des générations de processeurs Intel. Le tableau qui suit affiche les types de références SKU proposées.

Vous pouvez trouver le [références SKU disponibles pour HLI](hana-available-skus.md).

**Étapes suivantes**
- Voir [Architecture réseau de SAP HANA (grandes instances)](hana-network-architecture.md)
