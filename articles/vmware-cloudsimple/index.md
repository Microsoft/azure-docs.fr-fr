---
title: Azure VMware Solution by CloudSimple
description: Découvrez les solutions Azure VMware Solution by CloudSimple, notamment une vue d’ensemble, des démarrages rapides, des concepts, des tutoriels et des guides pratiques.
author: sharaths-cs
ms.author: b-mashar
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.custom: seo-azure-migrate
keywords: prise en charge de vms, azure vmware solution by cloudsimple, cloudsimple azure, outils vms, documentation vmware
ms.openlocfilehash: fffe0255bb905c0314e2ed59f5beab726e681e00
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89230634"
---
# <a name="azure-vmware-solution-by-cloudsimple"></a>Azure VMware Solution by CloudSimple

Bienvenue sur le portail tout-en-un conçu pour vous aider avec Azure VMware Solution by CloudSimple
Le site de documentation vous permet d’en savoir plus sur les sujets suivants :

## <a name="overview"></a>Vue d’ensemble

Apprenez-en davantage sur Azure VMware Solution by CloudSimple

* Découvrez les fonctionnalités, avantages et scénarios d’usage en consultant [En quoi consiste  Azure VMware Solution by CloudSimple ?](cloudsimple-vmware-solutions-overview.md).
* Passez en revue les [concepts clés pour l’administration](key-concepts.md).

## <a name="quickstart"></a>Démarrage rapide

Découvrez comment commencer à utiliser la solution.

* Apprenez à [initialiser le service et acheter de la capacité](quickstart-create-cloudsimple-service.md).
* Découvrez comment créer un environnement VMware en consultant [Configurer un environnement de cloud privé](quickstart-create-private-cloud.md).
* Découvrez comment unifier la gestion de VMware et Azure en consultant l’article [Utiliser des machines virtuelles VMware sur Azure](quickstart-create-vmware-virtual-machine.md).

## <a name="concepts"></a>Concepts

Apprenez-en davantage sur les concepts suivants.

* Un [service CloudSimple](cloudsimple-service.md) (également appelé «  Azure VMware Solution by CloudSimple - Service »). Cette ressource doit être créée une fois par région.
* Achetez de la capacité pour votre environnement en créant une ou plusieurs ressources de [nœud CloudSimple](cloudsimple-node.md). Ces ressources sont également appelées «  Azure VMware Solution by CloudSimple - Nœud ».
* Initialisez et configurez votre environnement VMware à l’aide des [clouds privés](cloudsimple-private-cloud.md).
* Unifiez la gestion à l’aide de [machines virtuelles CloudSimple](cloudsimple-virtual-machines.md) (également appelées «  Azure VMware Solution by CloudSimple - Machine virtuelle »).
* Concevez le réseau sous-jacent à l’aide de [réseaux VLAN/sous-réseaux](cloudsimple-vlans-subnets.md).
* Segmentez et sécurisez le réseau sous-jacent à l’aide de la ressource de [table de pare-feu](cloudsimple-firewall-tables.md).
* Bénéficiez d’un accès sécurisé à vos environnements VMware sur le réseau WAN à l’aide de [passerelles VPN](cloudsimple-vpn-gateways.md).
* Autorisez l’accès public aux charges de travail à l’aide d’une [adresse IP publique](cloudsimple-public-ip-address.md).
* Établissez une connectivité avec les réseaux virtuels Azure et les réseaux locaux à l’aide de la [connexion réseau Azure](cloudsimple-azure-network-connection.md).
* Configurez les cibles des e-mails d’alerte à l’aide de la [gestion de compte](cloudsimple-account.md).
* Visualisez les journaux d’activité des utilisateurs et des systèmes à l’aide des écrans de [gestion des activités](cloudsimple-activity.md).
* Appréhendez les différents [composants de VMware](vmware-components.md).

## <a name="tutorials"></a>Tutoriels

Découvrez comment effectuer des tâches courantes, notamment :

* [Créer un service CloudSimple](create-cloudsimple-service.md) pour chaque région dans laquelle vous souhaitez déployer des environnements VMware.
* Gérer les fonctionnalités de service principales dans le [portail CloudSimple](access-cloudsimple-portal.md).
* Activer la capacité et optimiser la facturation pour votre infrastructure en [achetant des nœuds CloudSimple](create-nodes.md).
* Gérer les configurations d’environnement VMware à l’aide de clouds privés. Vous pouvez [créer](create-private-cloud.md), [gérer](manage-private-cloud.md), [développer](expand-private-cloud.md) ou [réduire](shrink-private-cloud.md) des clouds privés.
* Mettre en œuvre une gestion unifiée en [mappant les abonnements Azure](azure-subscription-mapping.md).
* Superviser l’activité des utilisateurs et des systèmes à l’aide des [pages d’activité](monitor-activity.md).
* Configurer des réseaux pour vos environnements [en créant et en gérant des sous-réseaux](create-vlan-subnet.md).
* Segmenter et sécuriser votre environnement à l’aide des [tables et règles de pare-feu](firewall.md).
* Autoriser l’accès Internet entrant aux charges de travail en [allouant des adresses IP publiques](public-ips.md).
* Activer la connectivité à partir de vos stations de travail clientes ou réseaux internes en [configurant un VPN](vpn-gateway.md).
* Autoriser les communications vers les [réseaux virtuels Azure](virtual-network-connection.md) également à partir de vos [environnements locaux](on-premises-connection.md).
* Configurer les cibles des alertes et visualiser la capacité totale achetée dans le [résumé du compte](account.md).
* Visualiser les [utilisateurs](users.md) qui ont accédé au portail CloudSimple.
* Gérer des machines virtuelles VMware dans le portail Azure :
    * [Créer des machines virtuelles](azure-create-vm.md) dans le portail Azure
    * [Gérer les machines virtuelles](azure-manage-vm.md) que vous avez créées

## <a name="how-to-guides"></a>Guides pratiques

Ces guides décrivent les solutions à appliquer pour atteindre différents objectifs, par exemple :

* [Sécuriser votre environnement](private-cloud-secure.md).
* Installer des outils tiers, activer des utilisateurs supplémentaires et une source d’authentification externe dans vSphere à l’aide de la [réaffectation de privilèges](escalate-privileges.md).
* Configurer l’accès à différents services VMware en [configurant un DNS local](on-premises-dns-setup.md).
* Activer la répartition des noms et adresses pour les charges de travail en [configurant le DNS et le DHCP des charges de travail](dns-dhcp-setup.md).
* Comprendre comment le service garantit la sécurité et le bon fonctionnement de votre plateforme par le biais de [mises à jour et de mises à niveau](vmware-components.md#updates-and-upgrades) de service.
* Réduire le coût total de possession en créant un exemple d’architecture de sauvegarde avec un [logiciel de sauvegarde tiers tel que Veeam](backup-workloads-veeam.md).
* Créer un environnement sécurisé en activant le chiffrement au repos avec un [logiciel de chiffrement KMS tiers](vsan-encryption.md).
* Étendre la gestion Azure Active Directory (Azure AD) à VMware en configurant la [source d’identité Azure AD](azure-ad.md).
