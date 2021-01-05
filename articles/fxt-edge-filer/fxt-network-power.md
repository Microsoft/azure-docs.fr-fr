---
title: 'Tutoriel : Connecter des câbles à Azure FXT Edge Filer'
description: Découvrez comment câbler les ports réseau et alimenter le matériel Azure FXT Edge Filer
author: ekpgh
ms.author: rohogue
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 07/01/2019
ms.openlocfilehash: 80458d99ed6ad19c8a837f0989798d4dcc156b61
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92219528"
---
# <a name="tutorial-make-network-connections-and-supply-power-to-the-azure-fxt-edge-filer-node"></a>Tutoriel : Établir des connexions réseau et fournir une alimentation au nœud de l’Azure FXT Edge Filer

Ce tutoriel vous explique comment câbler les connexions réseau pour un nœud matériel de l’Azure FXT Edge Filer.

Ce tutoriel vous apprendra à effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Choisir le type de câble réseau approprié pour votre environnement
> * Connecter un nœud de l’Azure FXT Edge Filer au réseau de votre centre de données
> * Router les câbles par le passe-câbles (CMA)
> * Connecter une alimentation à l’appareil en rack et mettre l’appareil sous tension

## <a name="prerequisites"></a>Prérequis

Avant de démarrer ce tutoriel, l’Azure FXT Edge Filer doit être déjà installé dans un rack d’équipement standard. Le passe-câbles doit être déjà installé sur le nœud du filer.

## <a name="identify-ports"></a>Identifier les ports

Identifiez les différents ports situés à l’arrière de votre appareil Azure FXT Edge Filer.

![Arrière d’un appareil câblé](media/fxt-back-annotated.png)

## <a name="cable-the-device"></a>Câblage de l’appareil

* Connectez les ports RJ-45 à la source réseau de votre centre de données, comme cela est décrit dans la section [Ports réseau](#network-ports).  
* Connectez en toute sécurité le [port iDRAC](#idrac-port) à un réseau distinct sur lequel se trouve un serveur DHCP sécurisé.
* Utilisez les ports USB et le port VGA pour connecter un clavier et un écran au nœud en vue de la configuration initiale. Vous devez démarrer le nœud et [définir un mot de passe initial](fxt-node-password.md) pour activer les autres ports du nœud. Pour plus d’informations, consultez [Définir des mots de passe initiaux](fxt-node-password.md).

Cet article décrit également comment [connecter l’alimentation secteur](#connect-power-cables) au nœud.

Cet article explique aussi comment établir une connexion au [port série](#serial-port-only-when-necessary) du nœud, pour les besoins éventuels de dépannage avancé.

### <a name="network-ports"></a>Ports réseau

Chaque nœud Azure FXT Edge Filer inclut les ports réseau suivants :

* Six ports de données à haut débit, double débit 25GbE/10GbE :

  * Quatre ports fournis par deux cartes réseau avec plug-in double port
  * Deux ports fournis par la carte réseau mezzanine sur carte mère

* Deux ports 1GbE fournis par la carte réseau mezzanine sur carte mère

Les ports de données 25GbE/10GbE à haut débit sont pourvus de cages SFP28 compatibles standard. Pour utiliser des câbles optiques, vous devez installer les modules émetteur-récepteur optiques SFP28 (non fournis).

Les ports 1GbE sont dotés de connecteurs RJ-45 standard.

Pour obtenir la liste complète des câbles, commutateurs et émetteurs-récepteurs pris en charge, consultez le document [Cavium FastlinQ 41000 Series Interoperability Matrix](https://www.marvell.com/documents/xalflardzafh32cfvi0z/).

Le type de connexions à utiliser pour votre système dépend de l’environnement de votre centre de données.

* Si vous vous connectez à un réseau 25GbE, raccordez chacun des ports de données à haut débit avec l’un des types de câble suivants :

  * Câble optique et émetteur-récepteur optique SFP28 avec une capacité de débit de 25GbE ou double 25GbE/10GbE
  * Câble biaxial à connexion directe de type SFP28 avec une capacité de débit de 25GbE

* Si vous vous connectez à un réseau 10GbE, raccordez chacun des ports de données à haut débit avec l’un des types de câble suivants :

  * Câble optique et émetteur-récepteur optique SFP28 avec une capacité de débit de 10GbE ou double 25GbE/10GbE
  * Câble biaxial à connexion directe de type SFP28 avec une capacité de débit de 25GbE
  * Câble biaxial à connexion directe de type SFP28 avec une capacité de débit de 10GbE

* Les ports réseau 1GbE sont utilisés pour le trafic lié à la gestion du cluster. Sélectionnez l’option **Use 1Gb mgmt network** (Utiliser le réseau de gestion 1Gb) quand vous créez le cluster si vous souhaitez créer un réseau séparé physiquement pour la configuration du cluster (pour plus de détails, consultez [Configurer le réseau de gestion](fxt-cluster-create.md#configure-the-management-network)). Raccordez les ports avec un câble standard de catégorie 5 ou de catégorie supérieure comme cela est décrit dans la liste des câbles pris en charge.

  Vous pouvez ne pas raccorder les ports 1GbE si vous envisagez d’utiliser les ports à haut débit pour l’ensemble du trafic. Par défaut, les ports réseau 1GbE ne sont pas utilisés si un port de données de débit supérieur est disponible.  

### <a name="idrac-port"></a>Port iDRAC  

Le port étiqueté iDRAC est une connexion 1Gb qui permet la communication avec un contrôleur d’accès à distance utilisé pour la gestion et la supervision du matériel. Le logiciel FXT utilise l’interface IPMI (Intelligent Platform Management Interface) avec ce contrôleur pour le dépannage et la reprise d’activité. Vous pouvez utiliser l’[interface iDRAC](https://www.dell.com/support/manuals/idrac9-lifecycle-controller-v3.30.30.30/idrac_3.30.30.30_ug/) intégrée pour superviser le matériel sur ce port. L’accès iDRAC et l’accès IPMI sont activés par défaut.

> [!Note]
> Le port iDRAC peut contourner le système d’exploitation et interagir directement avec le matériel sur le nœud.

Appliquez les stratégies de sécurité ci-dessous quand vous configurez et connectez le port iDRAC :

* Connectez les ports iDRAC uniquement à un réseau qui est physiquement séparé du réseau de données utilisé pour accéder au cluster.
* Définissez un mot de passe d’administrateur iDRAC sécurisé sur chaque nœud. Vous devez définir ce mot de passe pour activer le matériel. Suivez les instructions fournies dans [Définir des mots de passe pour le matériel](fxt-node-password.md).
* La configuration par défaut du port iDRAC utilise DHCP et IPv4 pour l’affectation d’adresses IP. Assurez-vous que votre environnement DHCP est correctement protégé et que les connexions entre les clients DHCP et le serveur DHCP sont limitées. (Le Panneau de configuration du cluster inclut des paramètres qui vous permettent de changer la méthode de configuration des adresses des nœuds après avoir créé le cluster.)
* Laissez le port iDRAC configuré en « mode dédié » (mode par défaut), qui limite le trafic réseau iDRAC/IPMI au port RJ-45 dédié.

Le port iDRAC ne nécessite pas une connexion réseau à haut débit.
  
### <a name="serial-port-only-when-necessary"></a>Port série (uniquement s’il est nécessaire)

Dans certaines situations, le Support Microsoft est amené à vous demander de connecter un terminal au port série d’un nœud pour effectuer le diagnostic d’un problème.

Pour connecter la console :

1. Localisez le port série (COM1) à l’arrière du nœud du FXT Edge Filer.
1. Utilisez un câble modem nul pour connecter le port série à un terminal configuré pour ANSI-115200-8N1.
1. Connectez-vous à la console et effectuez les autres actions indiquées par l’équipe du Support.

## <a name="route-cables-in-the-cable-management-arm-cma"></a>Router les câbles par le passe-câbles (CMA)

Chaque nœud de l’Azure FXT Edge Filer est fourni avec un passe-câbles à usage facultatif. Le passe-câbles simplifie le routage des câbles et permet d’accéder plus facilement à l’arrière du châssis sans avoir à débrancher les câbles.

Suivez ces instructions pour router les câbles par le passe-câbles :

1. Au moyen des colliers de serrage fournis, rassemblez les câbles à l’entrée et à la sortie des paniers afin qu’ils n’interfèrent pas avec les systèmes adjacents (1).
1. Avec le passe-câbles en position de service, passez le faisceau de câbles à l’intérieur des paniers d’entrée et de sortie (2).
1. Utilisez les sangles à boucle et crochet préinstallées aux deux extrémités des paniers pour bien fixer les câbles (3).
1. Remettez le passe-câbles en place sur le plateau (4).
1. Installez le câble d’indicateur d’état à l’arrière du système et fixez-le en le routant par le passe-câbles. Attachez l’autre extrémité du câble au coin du panier de sortie du passe-câbles (5).

   > [!CAUTION]
   > Pour éviter des dommages potentiels occasionnés par des câbles qui dépassent, veillez à ce que rien ne pende au niveau du câble d’indicateur d’état après avoir routé ce câble par le passe-câbles.

![Illustration du passe-câbles avec les câbles installés](media/fxt-install/cma-cabling-400.png)

> [!NOTE]
> Si vous n’avez pas installé le passe-câbles, utilisez les deux sangles à boucle et crochet fournies dans le kit de rails pour router les câbles à l’arrière du système.
>
>  1. Localisez les supports externes du passe-câbles à l’intérieur des deux collerettes du rack.
>  2. Regroupez les câbles avec précaution en veillant à les dégager des connecteurs système situés de chaque côté.
>  3. Faites passer les sangles à boucle et crochet à travers les fentes situées sur les supports externes du passe-câbles de chaque côté du système afin de sécuriser les groupes de câbles.
>
>     ![Câbles routés sans passe-câbles](media/fxt-install/fxt-route-cables-no-cma-400.png)

## <a name="about-ip-address-requirements"></a>Informations sur les exigences relatives aux adresses IP

Pour les nœuds matériel dans un cache de stockage hybride Azure FXT Edge Filer, les adresses IP sont gérées par le logiciel du cluster.

Chaque nœud doit avoir au moins une adresse IP. Les adresses des nœuds sont affectées quand les nœuds sont ajoutés au cluster ou supprimés du cluster.

Le nombre total d’adresses IP requises dépend du nombre de nœuds dans votre cache.

Configurez la plage d’adresses IP à l’aide du logiciel du Panneau de configuration, après que les nœuds ont été installés. Pour en savoir plus, consultez [Collecter des informations pour le cluster](fxt-cluster-create.md#gather-information-for-the-cluster).  

## <a name="connect-power-cables"></a>Brancher les câbles d’alimentation

Chaque nœud de l’Azure FXT Edge Filer utilise deux unités d’alimentation (PSU).

> [!TIP]
> Pour tirer parti des deux unités d’alimentation redondantes, branchez chaque câble d’alimentation à une unité de distribution d’alimentation (PDU) sur un circuit de dérivation indépendant.
>
> Vous pouvez utiliser une unité d’alimentation pour alimenter les unités de distribution afin d’assurer une protection supplémentaire.

1. Branchez les câbles d’alimentation fournis aux unités d’alimentation dans le châssis. Assurez-vous que les câbles et les unités d’alimentation sont bien en place.
1. Branchez les câbles d’alimentation aux unités de distribution d’alimentation sur le rack d’équipement. Si possible, utilisez deux sources d’alimentation distinctes pour les deux câbles.

### <a name="power-on-an-azure-fxt-edge-filer-node"></a>Mettre sous tension un nœud de l’Azure FXT Edge Filer

Pour mettre un nœud sous tension, appuyez sur le bouton d’alimentation à l’avant du système. Le bouton se trouve dans le Panneau de configuration à droite.

### <a name="power-off-an-azure-fxt-edge-filer-node"></a>Mettre hors tension un nœud de l’Azure FXT Edge Filer

Vous pouvez utiliser le bouton d’alimentation pour arrêter le système pendant les tests et avant de l’ajouter à un cluster. Toutefois, dès lors qu’un nœud de l’Azure FXT Edge Filer est activé sur un cluster, vous devez utiliser le logiciel du Panneau de configuration du cluster pour arrêter le matériel. Pour en savoir plus, consultez [Comment mettre hors tension en toute sécurité le matériel Azure FXT Edge Filer](fxt-power-off.md).

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez terminé le câblage du matériel, mettez sous tension chacun des nœuds et activez-les en définissant leurs mots de passe initiaux.
> [!div class="nextstepaction"]
> [Définir des mots de passe initiaux](fxt-node-password.md)
