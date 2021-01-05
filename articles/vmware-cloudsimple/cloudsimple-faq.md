---
title: FAQ - Azure VMware Solution by CloudSimple
description: Ces questions fréquemment posées sur Azure VMware Solution par CloudSimple incluent la connectivité, la mise en réseau, la sécurité, le stockage, l’intégration d’Azure, etc.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 0db8c13a7467ed864c0845319b37f958f60fa4d9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88140885"
---
# <a name="frequently-asked-questions-about-vmware-solution-by-cloudsimple"></a>Questions fréquentes sur VMware Solution by CloudSimple

## <a name="cloudsimple-service"></a>Service CloudSimple

**En quoi consiste Azure VMware Solution by CloudSimple ?**

Azure VMware Solution by CloudSimple transforme et étend les charges de travail VMware à des clouds privés et dédiés sur Azure en quelques minutes. CloudSimple s’occupe du provisionnement, de la gestion de l’infrastructure et de l’orchestration des charges de travail entre les sites locaux et Azure. Vos applications s’exécutant exactement de la même façon au niveau local et dans Azure, vous bénéficiez de l’élasticité et des services du cloud sans qu’il soit nécessaire de procéder à une refonte complexe de vos applications. CloudSimple réduit votre coût total de possession avec un modèle de consommation cloud qui fournit des caractéristiques telles que le provisionnement à la demande, le paiement en fonction de l’évolution et l’optimisation de la capacité.  Pour examiner les fonctionnalités, avantages et scénarios, consultez [En quoi consiste la solution VMware sur Azure de CloudSimple ?](cloudsimple-vmware-solutions-overview.md).

**Qu’est-ce qu’un cloud privé CloudSimple ?**

Un cloud privé CloudSimple est un cloud privé et dédié qui se compose d’un environnement hautes performances (calcul, stockage et réseau) déployé sur l’infrastructure Microsoft Azure (espace de centre de données et matériel) dans des emplacements Azure.  Un cloud privé fournit une plateforme VMware native en tant que service. Pour VMware, chaque cloud privé contient exactement une instance du serveur vCenter Server. Ce dernier gère plusieurs nœuds ESXi contenus dans un ou plusieurs clusters vSphere, ainsi que le stockage SAN virtuel (vSAN) correspondant. Un service CloudSimple peut contenir plusieurs clouds privés dans votre abonnement Azure.  Pour plus d’informations, consultez [Vue d’ensemble du cloud privé](cloudsimple-private-cloud.md).

**Où le service CloudSimple est-il disponible ?**

CloudSimple est disponible dans les régions USA Est, USA Ouest et Europe Ouest. D’autres régions seront bientôt disponibles.

**Comment activer mon abonnement à CloudSimple ?**

Contactez le responsable de votre compte Microsoft à l’adresse [azurevmwaresales@microsoft.com](mailto:azurevmwaresales@microsoft.com) pour activer votre abonnement au service CloudSimple. Indiquez dans votre e-mail l’ID d’abonnement pour lequel vous souhaitez activer le service CloudSimple.  

**Comment accéder au portail CloudSimple ?**

L’accès au portail CloudSimple s’effectue à partir du portail Azure.  Pour plus d’informations, consultez [Accès au portail VMware Solution by CloudSimple à partir du portail Azure](access-cloudsimple-portal.md).

**Comment accroître la capacité d’un cloud privé ?**

Pour accroître la capacité, achetez des nœuds supplémentaires à partir du portail Azure, puis utilisez les nœuds pour étendre votre cloud privé à partir du portail CloudSimple.  Vous pouvez ajouter des nœuds supplémentaires à un cluster vSphere existant ou les ajouter à un nouveau cluster vSphere.  Pour plus d’informations, consultez [Développer un cloud privé CloudSimple](expand-private-cloud.md).

**Qu’advient-il de mon cloud privé durant la maintenance ?**

CloudSimple envoie une notification plusieurs jours avant une maintenance planifiée.  La maintenance est effectuée de façon à garantir la disponibilité de votre cloud privé sans interrompre le service.  La maintenance se présente sous plusieurs formes :

* **Infrastructure CloudSimple**.  L’infrastructure CloudSimple est conçue pour être hautement disponible.  Pendant ce type d’intervalle de maintenance, les composants redondants sont mis à jour un par un pour éviter toute interruption de service. Vous conservez l’accès au serveur vCenter de votre cloud privé, à toutes les machines virtuelles, à la connexion internet à partir de votre cloud privé et aux connexions au réseau local ou à Azure.
* **Portail CloudSimple**. Pendant ce type d’intervalle de maintenance, certaines fonctionnalités du portail CloudSimple peuvent être désactivées ou inaccessibles.  La notification avant l’intervalle de maintenance inclut des détails sur les limitations des fonctionnalités pendant la maintenance.

## <a name="connectivity"></a>Connectivité

**Quelles sont les options à ma disposition pour me connecter au réseau de la région CloudSimple ?**

CloudSimple offre les options de connectivité suivantes pour vous connecter au réseau de votre région CloudSimple. Plusieurs options peuvent être utilisées en même temps.

* **Connexion ExpressRoute de votre centre de données local au réseau de la région CloudSimple**. Il s’agit d’une connexion privée, sécurisée, haut débit et à faible latence qui relie votre circuit ExpressRoute local à votre circuit ExpressRoute CloudSimple à l’aide de Global Reach. Pour obtenir des instructions de configuration de la connexion, consultez [Connecter un réseau local à CloudSimple à l’aide d’ExpressRoute](on-premises-connection.md).
* **Connexion ExpressRoute de votre réseau virtuel Azure au réseau de votre région CloudSimple**. Il s’agit d’une connexion privée, sécurisée, haut débit et à faible latence qui relie votre réseau virtuel sur Azure à votre circuit ExpressRoute CloudSimple à l’aide de passerelles de réseau virtuel. Pour obtenir des instructions de configuration de la connexion, consultez [Connecter votre environnement de cloud privé de CloudSimple au réseau virtuel Azure à l’aide d’ExpressRoute](azure-expressroute-connection.md).
* **Connexion VPN de site à site de votre centre de données local au réseau de votre région CloudSimple**. Il s’agit d’un réseau privé virtuel sécurisé de votre périphérique VPN local à la région de votre cloud privé CloudSimple.  Pour plus d’informations, consultez [Configurer des passerelles VPN sur le réseau CloudSimple](vpn-gateway.md).

**Comment se connecter à un cloud privé ?**

Vous pouvez afficher les détails de votre cloud privé dans le portail CloudSimple. Pour vous connecter au serveur vCenter qui correspond à votre cloud privé, vérifiez d’abord qu’une connexion réseau est établie (VPN site à site, VPN point à site ou ExpressRoute). Ensuite, lancez le portail CloudSimple à partir du portail Azure et cliquez sur **Lancer le client vSphere** sur la page d’accueil ou sur la page des détails du cloud privé.

**Quel est l’avantage d’un circuit ExpressRoute ?**

Un circuit Azure ExpressRoute est une connexion sécurisée à haut débit et à faible latence.  CloudSimple fournit un circuit ExpressRoute dédié par région et par client.  Ce circuit vous permet d’établir une connexion sécurisée à partir de votre réseau local ou de votre abonnement Azure.

**Quels sont les coûts réseau d’une connexion à destination de CloudSimple ?  Des frais de sortie s’appliquent-ils entre CloudSimple et Azure, ou entre les régions ?**

Il n’existe aucun frais CloudSimple pour la sortie réseau.  Les tarifs Azure standard s’appliquent à tout le trafic de sortie issu de votre réseau virtuel ou d’un circuit ExpressRoute local.

## <a name="networking"></a>Mise en réseau

**Quelles sont les fonctionnalités réseau disponibles pour mon cloud privé ?**

Vous pouvez provisionner des réseaux locaux virtuels (et leurs sous-réseaux) et des tables de pare-feu, et attribuer des adresses IP publiques qui sont mappées à une machine virtuelle en cours d’exécution dans votre cloud privé. Pour plus d’informations sur les fonctionnalités de mise en réseau, consultez [Vue d’ensemble des réseaux locaux virtuels et des sous-réseaux](cloudsimple-vlans-subnets.md), [Vue d’ensemble des tables de pare-feu](cloudsimple-firewall-tables.md) et [Vue d’ensemble des adresses IP publiques](cloudsimple-public-ip-address.md).

**Comment configurer différents sous-réseaux pour les applications de mon cloud privé ?**

Vous créez des réseaux locaux virtuels sur votre cloud privé à partir du portail CloudSimple.  Après avoir créé un réseau local virtuel, vous pouvez créer un groupe de ports distribué sur le serveur vCenter de votre cloud privé en utilisant le réseau local virtuel et créer des machines virtuelles qui sont connectées au groupe de ports distribué.  Vous pouvez activer des tables de pare-feu pour le réseau local virtuel ou le sous-réseau et définir des règles de pare-feu pour sécuriser le trafic réseau.

**Quels sont les paramètres de pare-feu disponibles pour mes clouds privés ?**

Vous pouvez configurer des règles pour le trafic Nord-Sud et Est-Ouest.  Les règles sont définies dans une table de pare-feu.  La table de pare-feu peut être associée à des réseaux locaux virtuels sur votre cloud privé.  Pour plus d’informations, consultez [Configurer des tables et des règles de pare-feu pour les clouds privés](firewall.md).

**Puis-je attribuer des adresses IP publiques à des machines virtuelles dans mon environnement de cloud privé ?**

Dans le portail CloudSimple, vous pouvez attribuer une nouvelle adresse IP publique et l’associer à l’adresse IP privée d’une machine virtuelle ou d’une appliance.  Vous pouvez également créer des règles de pare-feu ou appliquer des règles de pare-feu existantes pour autoriser le trafic issu de ports et d’adresses IP spécifiques dans le portail. Pour plus d’informations, consultez [Allouer des adresses IP publiques pour un environnement de cloud privé](public-ips.md).

## <a name="security"></a>Sécurité

**Quelles sont mes options de sécurité sur CloudSimple ?**

CloudSimple fournit les fonctionnalités de sécurité suivantes pour sécuriser votre environnement de cloud privé :

* **Chiffrement des données au repos**. Vous pouvez chiffrer les données au repos qui résident sur le stockage vSAN dans votre cloud privé. vSAN prend en charge des serveurs gestionnaires de clés externe qui peuvent être déployés dans votre réseau virtuel Azure ou votre environnement local.  Pour plus d’informations, consultez [Configurer le chiffrement vSAN pour votre cloud privé CloudSimple](vsan-encryption.md).
* **Sécurité réseau**. Contrôlez le flux de trafic réseau avec des règles de pare-feu qui s’appliquent entre votre cloud privé et Internet, votre cloud privé et votre environnement local, ou dans des sous-réseaux de votre cloud privé.
* **Connexion privée et sécurisée**. Une connexion privée et sécurisée est établie entre votre réseau local et votre abonnement Azure.

## <a name="compute"></a>Calcul

**Quels sont les types d’hôtes disponibles ?**

CloudSimple propose ces types d’hôtes :

* **Nœud CS28** : Processeur : 2 x 2,2 GHz, 28 cœurs en tout, 48 HT.  RAM : 256 Go.  Stockage : 1 600 Go de cache NVMe, 5 760 Go de données (100 % Flash). Réseau : Carte réseau 4x25Gbe
* **Nœud CS36** : Processeur : 2 x 2,3 GHz, 36 cœurs en tout, 72 HT.  RAM : 512 Go.  Stockage : 3 200 Go de cache NVMe, 11 520 Go de données (100 % Flash).  Réseau : Carte réseau 4x25Gbe
* **Nœud CS36m :** Processeur : 2 x 2,3 GHz, 36 cœurs en tout, 72 HT.  RAM : 576 Go.  Stockage : 3200 Go de cache NVMe, 13360 Go de données (100 % Flash).  Réseau : Carte réseau 4x25Gbe

**Comment sont gérées les pannes matérielles ?**

Toute l’infrastructure de CloudSimple est supervisée en permanence par la plateforme CloudSimple et nos équipes en charge des opérations de service.  Si une panne matérielle est détectée, un nouveau nœud est ajouté à votre cloud privé et le nœud ayant échoué est supprimé.

## <a name="storage"></a>Stockage

**Quel type de stockage est pris en charge sur un cloud privé ?**

CloudSimple offre un stockage VMware vSAN entièrement Flash avec chaque cloud privé.  Chaque vSphere est créé avec a propre banque de données vSAN.  Pour plus d’informations, consultez [Composants VMware de cloud privé - Stockage vSAN](vmware-components.md#vsan-storage).

**Le chiffrement des données est-il pris en charge ?**
Oui.  Vous pouvez configurer le stockage vSAN sur votre cloud privé pour utiliser un serveur de gestionnaire de clés (KMS), qui est déployé localement ou sur Azure pour chiffrer les données stockées sur vSAN.

**Comment sont gérés les disques défaillants ?**

CloudSimple supervise en permanence tous les composants matériels du cloud privé.  Si une panne de disque est détectée ou qu’un disque est identifié comme défaillant (en fonction de paramètres heuristiques), un nouveau nœud est automatiquement ajouté au cloud privé.  Le nœud avec un disque défaillant ou en cours d défaillance est supprimé du cloud privé.

## <a name="vmware"></a>VMware

**Comment faire pour effectuer le chargement ou la migration à grande échelle d’applications et de données à partir d’un réseau local ?**

CloudSimple fournit une solution VMware vSphere native.  Vous pouvez vous servir de tous les outils VMware pour la migration de données en bloc avec votre cloud privé.  Options disponibles :

* VMware HCX pour la migration en bloc de données.
* Migration à froid des données à l’aide de Storage vMotion à partir d’un réseau local vers CloudSimple.

**Puis-je installer des outils VMware ?**

CloudSimple fournit une solution VMware vSphere native.  Vous pouvez utiliser tous les outils VMware pour la gestion de votre environnement vSphere local sur CloudSimple.  CloudSimple prend en charge un modèle BYOL (apportez votre propre licence) pour installer les outils VMware.

**Comment sont gérées les mises à jour et les mises à niveau ?**

CloudSimple gère et met à jour tous les composants d’infrastructure de votre cloud privé de manière transparente et sans interruption de service.  L’application des mises à jour et des correctifs de sécurité publiés par VMware ou les fournisseurs d’infrastructure est planifiée dès que CloudSimple les valide.

CloudSimple n’effectue ni la mise à niveau ni la mise à jour des applications installées sur le cloud privé.

## <a name="azure-integration"></a>Intégration à Azure

**Quels sont les services Azure pris en charge ?**

CloudSimple fournit une connexion Azure ExpressRoute à votre abonnement sur Azure.  Tous les services exécutés dans votre abonnement peuvent se connecter à votre cloud privé.  Voici quelques exemples :

* **Azure Active Directory** comme source d’identité pour votre vCenter CloudSimple.
* **Stockage Azure** pour stocker les sauvegardes, images et autres données de votre cloud privé.
* **Applications hybrides** avec une architecture d’application qui s’étend aux clouds publics et privés.  Par exemple, vous pouvez créer des serveurs web dans Azure qui accèdent aux serveurs d’application et de base de données dans votre cloud privé.
* **Azure Monitor** et **Azure Security Center** pour les charges de travail qui s’exécutent sur VMware prennent en charge la journalisation, les métriques de performances et la gestion de la sécurité.

**Comment mapper mes clients VMware à Azure ?**

CloudSimple est la seule solution qui vous permet de gérer vos machines virtuelles VMware sur un cloud privé à partir du portail Azure.  Un pool de ressources vCenter configuré avec les contraintes de ressources souhaitées peut être mappé à votre abonnement par votre administrateur général.  

**Quels sont les avantages qu’offrent les licences Azure ?**

Avec CloudSimple, vous pouvez tirer parti de l’avantage Azure Hybrid Usage Benefit et économiser jusqu’à 90 % sur les licences. Cet avantage préserve votre investissement dans les licences Microsoft et réduit votre coût total de possession par rapport à d’autres solutions cloud. Vous bénéficiez également de mises à jour de sécurité étendues pour Windows Server 2008 et Microsoft SQL Server 2008.  Le modèle BYOL (apportez votre propre licence) vous permet de réduire les coûts pour les applications courantes comme Veeam et Zerto.  
