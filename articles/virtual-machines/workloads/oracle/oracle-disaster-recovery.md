---
title: Vue d’ensemble du scénario de récupération d’urgence Oracle dans votre environnement Azure | Documents Microsoft
description: Scénario de récupération d’urgence pour une base de données Oracle Database 12c dans votre environnement Azure
author: dbakevlar
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 08/02/2018
ms.author: kegorman
ms.reviewer: cynthn
ms.openlocfilehash: b8da0b5c55b291af42d9a30db23d6f55f7c0bf2d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022783"
---
# <a name="disaster-recovery-for-an-oracle-database-12c-database-in-an-azure-environment"></a>Récupération d’urgence pour une base de données Oracle Database 12c dans votre environnement Azure

## <a name="assumptions"></a>Hypothèses

- Vous disposez d’une connaissance de la conception d’Oracle Data Guard et des environnements Azure.


## <a name="goals"></a>Objectifs
- Concevoir la topologie et la configuration adaptées à vos besoins de récupération d’urgence (DR).

## <a name="scenario-1-primary-and-dr-sites-on-azure"></a>Scénario 1 : site principal et site de récupération d’urgence sur Azure

Un client dispose d’une configuration de base de données Oracle sur le site principal. Un site de récupération d’urgence se trouve dans une autre région. Le client utilise Oracle Data Guard pour une récupération rapide entre ces sites. Le site principal possède également une base de données secondaire pour la création de rapports et d’autres utilisations. 

### <a name="topology"></a>Topologie

Voici un résumé de la configuration Azure :

- Deux sites (un site principal et un site de récupération d’urgence)
- Deux réseaux virtuels
- Deux bases de données Oracle avec Data Guard (primaire et secondaire)
- Deux bases de données Oracle avec Golden Gate ou Data Guard (site principal uniquement)
- Deux services d’application sur le site principal et un sur le site de récupération d’urgence
- Un *groupe à haute disponibilité,* utilisé pour la base de données et le service d’application sur le site principal
- Une jumpbox sur chaque site, qui limite l’accès au réseau privé et autorise uniquement l’administrateur à se connecter
- Jumpbox, service d’application, base de données et passerelle VPN sur des sous-réseaux distincts
- Le groupe de sécurité réseau est appliqué sur les sous-réseaux d’application et de base de données

![Diagramme illustrant les sites principaux et de récupération d’urgence sur Azure.](./media/oracle-disaster-recovery/oracle_topology_01.png)

## <a name="scenario-2-primary-site-on-premises-and-dr-site-on-azure"></a>Scénario 2 : site principal local et site de récupération d’urgence sur Azure

Un client dispose d’une configuration de base de données Oracle locale (site principal). Le site de récupération d’urgence est sur Azure. Oracle Data Guard permet une récupération rapide entre ces sites. Le site principal possède également une base de données secondaire pour la création de rapports et d’autres utilisations. 

Deux approches sont possibles pour cette configuration.

### <a name="approach-1-direct-connections-between-on-premises-and-azure-requiring-open-tcp-ports-on-the-firewall"></a>Approche 1 : connexions directes entre un site local et un site sur Azure, nécessitant des ports TCP ouverts sur le pare-feu 

Nous ne recommandons pas des connexions directes, car ils exposent les ports TCP au monde extérieur.

#### <a name="topology"></a>Topologie

Voici un résumé de la configuration Azure :

- Un site de récupération d’urgence 
- Un seul réseau virtuel
- Une base de données Oracle avec Data Guard (active)
- Un service d’application sur le site de récupération d’urgence
- Une jumpbox, qui limite l’accès au réseau privé et autorise uniquement l’administrateur à se connecter
- Jumpbox, service d’application, base de données et passerelle VPN sur des sous-réseaux distincts
- Le groupe de sécurité réseau est appliqué sur les sous-réseaux d’application et de base de données
- Une règle/stratégie de groupe de sécurité réseau pour autoriser le port TCP entrant 1521 (ou un port défini par l’utilisateur)
- Une stratégie/règle de groupe de sécurité réseau pour limiter l’accès du réseau virtuel à l’adresse IP/aux adresses locales (base de données ou application) uniquement

![Diagramme montrant les connexions directes entre un site local et un site sur Azure, nécessitant des ports TCP ouverts sur le pare-feu.](./media/oracle-disaster-recovery/oracle_topology_02.png)

### <a name="approach-2-site-to-site-vpn"></a>Approche 2 : VPN de site à site
La meilleure approche consiste à utiliser des VPN de site à site. Pour plus d’informations sur la configuration d’un VPN, consultez [Créer un réseau virtuel avec une connexion VPN de Site à Site à l’aide de CLI](../../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md).

#### <a name="topology"></a>Topologie

Voici un résumé de la configuration Azure :

- Un site de récupération d’urgence 
- Un seul réseau virtuel 
- Une base de données Oracle avec Data Guard (active)
- Un service d’application sur le site de récupération d’urgence
- Une jumpbox, qui limite l’accès au réseau privé et autorise uniquement l’administrateur à se connecter
- Une jumpbox, un service d’application, une base de données et une passerelle VPN se trouvent sur des sous-réseaux distincts
- Le groupe de sécurité réseau est appliqué sur les sous-réseaux d’application et de base de données
- Connexion VPN de site à site entre un site local et un site sur Azure

![Capture d’écran de la page de topologie de récupération d’urgence](./media/oracle-disaster-recovery/oracle_topology_03.png)

## <a name="additional-reading"></a>Documentation supplémentaire

- [Concevoir et mettre en œuvre une base de données Oracle sur Azure](oracle-design.md)
- [Implémenter Oracle Data Guard sur une machine virtuelle Linux Azure](configure-oracle-dataguard.md)
- [Implémenter Oracle Golden Gate sur une machine virtuelle Linux Azure](configure-oracle-golden-gate.md)
- [Sauvegarde et récupération Oracle](./oracle-overview.md)


## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Créer des machines virtuelles hautement disponibles](../../linux/create-cli-complete.md)
- [Explorer des exemples Azure CLI de déploiement de machines virtuelles](../../linux/cli-samples.md)
