---
title: SAP Business One sur les machines virtuelles Azure | Microsoft Docs
description: SAP Business One sur Azure.
author: msjuergent
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 07/15/2018
ms.author: juergent
ms.reviewer: cynthn
ms.openlocfilehash: 1201ff91634a1f63c8fe0e545ca6d6dee33abe35
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94957382"
---
# <a name="sap-business-one-on-azure-virtual-machines"></a>SAP Business One sur les machines virtuelles Azure
Ce document fournit des conseils pour déployer SAP Business One sur des machines virtuelles Azure. Ce document n’est pas un substitue au document d’installation pour SAP Business One. Le document doit couvrir la planification de base et les instructions de déploiement pour que l’infrastructure Azure exécute des applications Business One.

Business One prend en charge deux bases de données différentes :
- SQL Server - consultez [SAP Note #928839 - Planification de la version de Microsoft SQL Server](https://launchpad.support.sap.com/#/notes/928839)
- SAP HANA - pour la matrice de prise en charge SAP Business One, consultez [Matrice de disponibilité de produit SAP](https://support.sap.com/pam)

Concernant SQL Server, les points à prendre en considérations pour le déploiement de base documentés dans [SQL Server Azure Virtual Machines DBMS deployment for SAP NetWeaver](./dbms_guide_general.md) (Déploiement SGBD des machines virtuelles Azure SQL Server pour SAP NetWeaver) s’appliquent. Pour SAP HANA, les points à prendre en considération sont mentionnés dans ce document.

## <a name="prerequisites"></a>Prérequis
Pour utiliser ce guide, vous devez disposer des connaissances de base quant aux différents composants Azure suivants :

- [Machines virtuelles Azure sur Windows](../../windows/tutorial-manage-vm.md)
- [Machines virtuelles Azure sur Linux](../../linux/tutorial-manage-vm.md)
- [Mise en réseau Azure et gestion des réseaux virtuels avec PowerShell](../../windows/tutorial-virtual-network.md)
- [Mise en réseau et réseaux virtuels Azure avec CLI](../../linux/tutorial-virtual-network.md)
- [Gérer des disques Azure avec Azure CLI](../../linux/tutorial-manage-disks.md)

Même si vous êtes intéressé uniquement par Business One, le document [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver](./planning-guide.md) peut constituer une bonne source d’informations.

Nous partons du principe que, pendant le déploiement de SAP Business One, vous :

- Savez comment installer SAP HANA sur une infrastructure donnée comme une machine virtuelle
- Savez comment installer l’application SAP Business One sur une infrastructure telle que des machines virtuelles Azure
- Savez comment exploiter les systèmes SAP Business One et DBMS choisis
- Savez comment déployer une infrastructure sur Azure

Tous ces sujets ne seront pas traités dans ce document.

Outre la documentation Azure, vous devez aussi prendre connaissance des notes SAP, qui font référence à Business One ou les notes centrales de SAP pour Business One :

- [528296 : Notes de présentation générales sur les versions SAP Business One et produits connexes](https://launchpad.support.sap.com/#/notes/528296)
- [2216195 : Notes des mises à jour de version de SAP Business One 9.2 pour SAP HANA](https://launchpad.support.sap.com/#/notes/2216195)
- [2483583 : Notes centrales pour SAP Business One 9.3](https://launchpad.support.sap.com/#/notes/2483583)
- [2483615 : Notes des mises à jour de version pour SAP Business One 9.3](https://launchpad.support.sap.com/#/notes/2483615)
- [2483595 : Notes générales sur les problèmes globaux de SAP Business One 9.3](https://launchpad.support.sap.com/#/notes/2483595)
- [2027458 : Notes de conseil générales sur les rubriques connexes SAP HANA de SAP Business One, version pour SAP HANA](https://launchpad.support.sap.com/#/notes/2027458)


## <a name="business-one-architecture"></a>Architecture Business One
Business One est une application à deux niveaux :

- Un niveau client avec un client « fat »
- Un niveau base de données qui contient le schéma de la base de données pour un abonné

Une meilleure vue d’ensemble des composants exécutés dans la partie client et ceux exécutés dans la partie serveur est documentée dans le [Guide administrateur SAP Business One](https://help.sap.com/doc/601fbd9113be4240b81d74626439cfa9/10.0/en-US/AdministratorGuide_SQL.pdf) 

Étant donné l’interaction critique en termes de latence importante entre le niveau client et le niveau base de données, les deux niveaux doivent être situés dans Azure lors du déploiement dans Azure. Il est courant que les utilisateurs se connectent à distance à une ou plusieurs machines virtuelles qui exécutent un service Bureau à distance pour les composants client Business One.

### <a name="sizing-vms-for-sap-business-one"></a>Dimensionnement des machines virtuelles pour SAP Business One

Concernant le dimensionnement des machines virtuelles client, les exigences en ressources sont documentées par SAP dans le document [Guide sur les conditions matérielles requises SAP Business One](https://help.sap.com/doc/bfa9770d12284cce8509956dcd4c5fcb/9.3/en-US/B1_Hardware_Requirements_Guide.pdf). Pour Azure, vous devez vous concentrer sur le calcul des conditions requises stipulées dans le chapitre 2.4 du document.

Comme les machines virtuelles Azure qui hébergent les composants client Business One et l’hôte de base de données, seules les machines virtuelles prises en charge par SAP NetWeaver sont autorisées. Pour obtenir la liste des machines virtuelles Azure prises en charge par SAP NetWeaver, consultez les [notes SAP #1928533](https://launchpad.support.sap.com/#/notes/1928533).

Lors de l’exécution de SAP HANA en tant que serveur principal de base de données pour Business One, seules les machines virtuelles répertoriées pour Business sur HANA dans la [liste de plateforme IaaS HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure%23SAP%20Business%20One) sont prises en charge pour HANA. Les composants client Business One ne sont pas affectés par cette importante restriction SAP HANA en tant que système de gestion de base de données.

### <a name="operating-system-releases-to-use-for-sap-business-one"></a>Versions des systèmes d’exploitation à utiliser pour SAP Business One

En principe, il vaut toujours mieux utiliser les versions les plus récentes d’un système d’exploitation. En particulier dans l’espace Linux, où des nouvelles fonctionnalités Azure ont été introduite par le biais de mises à jour de version mineures récentes de Suse et Red Hat. Du côté de Windows, l’utilisation de Windows Server 2016 est fortement conseillée.


## <a name="deploying-infrastructure-in-azure-for-sap-business-one"></a>Déploiement d’une infrastructure dans Azure pour SAP Business One
Dans les prochains chapitres, vous découvrirez les parties de l’infrastructure qui comptent dans le déploiement SAP.

### <a name="azure-network-infrastructure"></a>Infrastructure réseau Azure
L’infrastructure réseau que vous devez déployer est différente selon si vous déployez un système Business One unique pour vous-même. ou si vous hébergez une douzaine de systèmes Business One pour des clients. La conception peut aussi être légèrement différente en fonction de votre méthode de connexion à Azure. En analysant les différentes possibilités, une conception où vous disposez d’une connectivité VPN dans Azure et où vous étendez Active Directory via [VPN](../../../vpn-gateway/vpn-gateway-about-vpngateways.md) ou [ExpressRoute](../../../expressroute/expressroute-introduction.md) dans Azure.

![Configuration réseau simple avec Business One](./media/business-one-azure/simple-network-with-VPN.PNG)

La configuration simplifiée présentée introduit plusieurs instances de sécurité qui permettent de contrôler et de limiter le routage. Elle commence par 

- Le pare-feu/routeur du côté du client local.
- L’instance suivante est le [Groupe de sécurité réseau Azure](../../../virtual-network/network-security-groups-overview.md) que vous pouvez utiliser pour introduire les règles de sécurité et de routage pour le réseau virtuel Azure dans lequel vous exécutez la configuration SAP Business One.
- Afin d’éviter que les utilisateurs du client Business One ne puissent aussi voir le serveur qui exécute le serveur Business One, qui exécute la base de données, vous devez séparer la machine virtuelle qui héberge le client Business One et le serveur Business One et les placer dans deux sous-réseaux différents au sein du réseau virtuel.
- Vous utiliseriez à nouveau le groupe de sécurité réseau Azure assigné aux deux sous-réseaux différents afin de limiter l’accès au serveur Business One.

Une version plus sophistiquée d’une configuration réseau Azure est basée sur les [meilleures pratiques Azure documentée d’architecture hub et spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke). Le modèle d’architecture hub et spoke changera la première configuration simplifiée en une configuration similaire à :


![Configuration Hub et spoke avec Business One](./media/business-one-azure/hub-spoke-network-with-VPN.PNG)

Dans les cas où les utilisateurs se connectent via Internet sans connectivité privé à Azure, la conception du réseau dans Azure doit correspondre aux principes documentés dans l’architecture de référence Azure pour la [zone DMZ entre Azure et Internet](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz).

### <a name="business-one-database-server"></a>Serveur de base de données Business One
SQL Server et SAP HANA sont disponibles comme type de base de données. Indépendamment de SGBD, vous devez lire le document [Considerations for Azure Virtual Machines DBMS deployment for SAP workload](./dbms_guide_general.md) (Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP) pour obtenir une compréhension générale des déploiements SGBD de machines virtuelles Azure et les rubriques connexes à la mise en réseau et au stockage.

Bien que cela soit déjà mentionné dans les documents spécifiques et génériques aux bases de données, nous insistons sur le fait que vous devez savoir :

- [Gérer la disponibilité des machines virtuelles Windows dans Azure](../../manage-availability.md) et [Gérer la disponibilité des machines virtuelles Linux dans Azure](../../manage-availability.md)
- [Contrat SLA pour les machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/)

Ces documents vous aideront à décider de la sélection des types de stockage et de la configuration de haute disponibilité.

En principe, vous devez :

- Préférer les disques SSD Premium aux disques durs standard. Pour en savoir plus sur les types de disques disponibles, consultez notre article [Sélectionner un type de disque](../../disks-types.md)
- Utiliser des disques managés Azure plutôt que des disques non managés
- Vous assurer d’avoir un débit d’E/S par seconde suffisant configuré avec votre configuration de disque
- Combiner /hana/data et /hana/log volume afin d’avoir une configuration de stockage économique


#### <a name="sql-server-as-dbms"></a>SQL Server en tant que système de gestion de base de données (SGBD)
Pour le déploiement de SQL Server en tant que SGBD pour Business One, suivez le document [SQL Server Azure Virtual Machines DBMS deployment for SAP NetWeaver](./dbms_guide_sqlserver.md) (Déploiement SGBD des machines virtuelles Azure SQL Server pour SAP NetWeaver). 

Tailles estimées du côté du système de gestion de base de données pour SQL Server :

| Nombre d’utilisateurs | Processeurs virtuels | Mémoire | Exemples de types de machines virtuelles |
| --- | --- | --- | --- |
| Jusqu’à 20 | 4 | 16 Go | D4s_v3, E4s_v3 |
| Jusqu’à 40 | 8 | 32 Go | D8s_v3, E8s_v3 |
| Jusqu’à 80 | 16 | 64 Go | D16s_v3, E16s_v3 |
| Jusqu’à 150 | 32 | 128 Go | D32s_v3, E32s_v3 |

Les tailles ci-dessus doivent vous donner une idée pour commencer. Vous pourriez avoir besoin de moins ou de plus de ressources, auquel cas l’adaptation sur Azure est simple. Un changement du type de la machine virtuelle est possible en un simple redémarrage de la machine virtuelle.

#### <a name="sap-hana-as-dbms"></a>SAP HANA en tant que système de gestion de base de données
Avec SAP HANA en tant que système de gestion de base de données, vous devez suivre les facteurs à prendre en compte du document [Guide des opérations SAP HANA sur Azure](./hana-vm-operations.md).

Pour les configurations de haute disponibilité et de récupération d’urgence pour SAP HANA en tant que base de données pour Business One dans Azure, vous devez lire la documentation [Haute disponibilité de SAP HANA pour les machines virtuelles Azure](./sap-hana-availability-overview.md) et celle à laquelle elle fait référence.

Pour les stratégies de restauration et de sauvegarde SAP HANA, vous devez lire le document [Guide de sauvegarde pour SAP HANA sur des machines virtuelles Azure](./sap-hana-backup-guide.md), ainsi que la documentation à laquelle il fait référence.

 
### <a name="business-one-client-server"></a>Serveur client Business One
Pour ces composants, les facteurs de stockage ne sont pas primordiaux. Néanmoins, vous devez disposer d’une plateforme fiable. Par conséquent, vous devez utiliser le stockage Premium Azure pour cette machine virtuelle, même pour le disque dur virtuel de base. Dimensionnement de la machine virtuelle, avec les données fournies dans le [Guide des exigences matérielles SAP Business One](https://help.sap.com/doc/bfa9770d12284cce8509956dcd4c5fcb/9.3/en-US/B1_Hardware_Requirements_Guide.pdf). Pour Azure, vous devez vous concentrer sur le calcul des conditions requises stipulées dans le chapitre 2.4 du document. Lors du calcul des exigences, vous devez les comparer aux documents suivants pour trouver la machine virtuelle idéale pour vous :

- [Tailles des machines virtuelles Windows dans Azure](../../sizes.md)
- [Note SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533)

Comparez le nombre de processeurs et la mémoire nécessaires à ce qui est documenté par Microsoft. Gardez aussi à l’esprit le débit réseau lorsque vous choisissez les machines virtuelles.
