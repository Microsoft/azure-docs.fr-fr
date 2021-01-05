---
title: Azure VMware Solution by CloudSimple - Connexion locale à l’aide d’ExpressRoute
description: Décrit comment demander une connexion locale avec ExpressRoute à partir d’un réseau de région CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 0dd5ede110255b6e53bbc397e683e66b3beffc65
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77019619"
---
# <a name="connect-from-on-premises-to-cloudsimple-using-expressroute"></a>Connecter un réseau local à CloudSimple à l’aide d’ExpressRoute

Si vous disposez déjà d’une connexion Azure ExpressRoute à partir d’un emplacement externe (comme un emplacement local) vers Azure, vous pouvez la connecter à votre environnement CloudSimple. Vous pouvez le faire via une fonctionnalité d’Azure qui permet à deux circuits ExpressRoute de se connecter entre eux. Cette méthode établit une connexion sécurisée, privée, à bande passante élevée et à faible latence entre les deux environnements.

[![Connexion ExpressRoute locale - Global Reach](media/cloudsimple-global-reach-connection.png)](media/cloudsimple-global-reach-connection.png)

## <a name="before-you-begin"></a>Avant de commencer

Un bloc d’ adresses réseau **/29** est requis pour établir une connexion Global Reach à partir d’un emplacement local.  L’espace d’adressage /29 est utilisé pour le réseau de transit entre les circuits ExpressRoute.  Le réseau de transit ne doit pas chevaucher vos réseaux virtuels Azure, réseaux locaux ou réseaux de cloud privé CloudSimple.

## <a name="prerequisites"></a>Prérequis

* Un circuit Azure ExpressRoute est requis avant de pouvoir établir la connexion entre le circuit et les réseaux de cloud privé de CloudSimple.
* Un utilisateur est requis, avec les privilèges pour créer des clés d’autorisation sur un circuit ExpressRoute.

## <a name="scenarios"></a>Scénarios

La connexion de votre réseau local à votre réseau de cloud privé vous permet d’utiliser le cloud privé de différentes façons, notamment pour :

* Accéder à votre réseau de cloud privé sans créer de connexion VPN de site à site.
* Utiliser votre annuaire Active Directory local en tant que source d’identité sur votre cloud privé.
* Migrer des machines virtuelles exécutées localement vers votre cloud privé.
* Utiliser votre cloud privé dans le cadre d’une solution de récupération d’urgence.
* Utiliser des ressources locales sur vos machines virtuelles de charge de travail de cloud privé.

## <a name="connecting-expressroute-circuits"></a>Connexion de circuits ExpressRoute

Pour établir la connexion ExpressRoute, vous devez créer une autorisation sur votre circuit ExpressRoute et fournir les informations d’autorisation à CloudSimple.


### <a name="create-expressroute-authorization"></a>Créer l’autorisation ExpressRoute

1. Connectez-vous au portail Azure.

2. Dans la barre de recherche supérieure, recherchez **Circuit ExpressRoute** et cliquez sur **Circuits ExpressRoute** sous **Services**.
    [![Circuits ExpressRoute](media/azure-expressroute-transit-search.png)](media/azure-expressroute-transit-search.png)

3. Sélectionnez le circuit ExpressRoute que vous souhaitez connecter à votre réseau CloudSimple.

4. Dans la page ExpressRoute, cliquez sur **Autorisations**, entrez un nom pour l’autorisation, puis cliquez sur **Enregistrer**.
    [![Autorisation du circuit ExpressRoute](media/azure-expressroute-transit-authorizations.png)](media/azure-expressroute-transit-authorizations.png)

5. Copiez l’ID de ressource et la clé d’autorisation en cliquant sur l’icône de copie. Collez l’ID et la clé dans un fichier texte.
    [![Copier une autorisation du circuit ExpressRoute](media/azure-expressroute-transit-authorization-copy.png)](media/azure-expressroute-transit-authorization-copy.png)

    > [!IMPORTANT]
    > **L’ID de ressource** doit être copié à partir de l’interface utilisateur et doit être au format ```/subscriptions/<subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.Network/expressRouteCircuits/<express-route-circuit-name>``` lorsque vous le fournissez pour qu’il soit pris en charge.

6. Configurez un ticket avec <a href="https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest" target="_blank">Prise en charge</a> pour la connexion à créer.
    * Type de problème : **Technique**
    * Abonnement : **Abonnement dans lequel le service CloudSimple est déployé**
    * Service : **VMware Solution by CloudSimple**
    * Type de problème : **Demande de service**
    * Sous-type de problème : **Créer une connexion ExpressRoute au réseau local**
    * Indiquez l’ID de ressource et la clé d’autorisation que vous avez copiés et enregistrés dans le volet d’informations.
    * Fournissez un espace d’adressage réseau /29 pour le réseau de transit.
    * Envoyez-vous la route par défaut par le biais d’ExpressRoute ?
    * Le trafic de cloud privé doit-il utiliser la route par défaut envoyée par le biais d’ExpressRoute ?

    > [!IMPORTANT]
    > L’envoi d’une route par défaut vous permet d’envoyer tout le trafic Internet à partir d’un cloud privé à l’aide de votre connexion Internet locale.  Pour désactiver la route par défaut configurée sur le cloud privé et utiliser la route par défaut de la connexion locale, fournissez des informations détaillées dans le ticket de support.

## <a name="next-steps"></a>Étapes suivantes

* [Apprenez-en davantage sur les connexions réseau Azure](cloudsimple-azure-network-connection.md)  
