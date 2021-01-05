---
title: Qu’est-ce que la délégation de sous-réseau dans un réseau virtuel Azure ?
description: En savoir plus sur la délégation de sous-réseau dans un réseau virtuel Azure
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/20/2019
ms.author: kumud
ms.openlocfilehash: 1fbb683754aed5b2a2e6e9c022713b7e87ad9ba9
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91329205"
---
# <a name="what-is-subnet-delegation"></a>Qu’est-ce que la délégation de sous-réseau ?

La délégation de sous-réseau vous permet de désigner un sous-réseau spécifique pour un service PaaS Azure de votre choix qui doit être injecté dans votre réseau virtuel. La délégation de sous-réseau offre un contrôle total au client sur la gestion de l’intégration des services Azure à ses réseaux virtuels.

Lorsque vous déléguez un sous-réseau à un service Azure, vous permettez à ce service d’établir des règles de configuration de réseau de base pour ce sous-réseau, ce qui permet au service Azure de faire fonctionner ses instances de manière stable. Par conséquent, le service Azure peut établir des conditions préalables ou après le déploiement, comme les suivantes :
- Déployer le service dans un sous-réseau partagé et dédié.
- Ajouter au service, après le déploiement, un ensemble de stratégies d’intention de réseau requis pour que le service fonctionne correctement.

##  <a name="advantages-of-subnet-delegation"></a>Avantages de la délégation de sous-réseau

La délégation d’un sous-réseau à des services spécifiques offre les avantages suivants :

- Permet de désigner un sous-réseau pour un ou plusieurs services Azure et de gérer les instances dans le sous-réseau conformément à la configuration requise. Par exemple, le propriétaire du réseau virtuel peut définir les éléments suivants pour un sous-réseau délégué afin de mieux gérer les ressources et l’accès :
    - stratégies de filtrage du trafic réseau avec les groupes de sécurité réseau.
    - stratégies de routage avec des itinéraires définis par l’utilisateur.
    - intégration des services aux configurations des points de terminaison de service.
- Aide les services injectés à mieux intégrer le réseau virtuel en définissant les conditions préalables aux déploiements sous la forme de stratégies d’intention de réseau. Cela garantit que toutes les actions susceptibles d’affecter le fonctionnement du service injecté peuvent être bloquées au moment de l’opération PUT.


## <a name="who-can-delegate"></a>Qui peut déléguer ?
La délégation de sous-réseau est un exercice que les propriétaires de réseau virtuel doivent effectuer pour désigner un des sous-réseaux pour un service Azure spécifique. Le service Azure déploie ensuite les instances dans ce sous-réseau à des fins de consommation par les charges de travail des clients.

## <a name="impact-of-subnet-delegation-on-your-subnet"></a>Impact de la délégation de sous-réseau sur votre sous-réseau
Chaque service Azure définit son propre modèle de déploiement, où il peut définir les propriétés qu’il prend ou ne prend pas en charge dans un sous-réseau délégué à des fins d’injection, comme suit :
- prend en charge un sous-réseau partagé avec d’autres services ou machines virtuelles/groupes de machines virtuelles identiques Azure dans le même sous-réseau, ou seulement un sous-réseau dédié avec uniquement des instances de ce service.
- prend en charge l’association de groupe de sécurité réseau avec le sous-réseau délégué.
- prend en charge le fait que le groupe de sécurité réseau associé au sous-réseau délégué peut également être associé à n’importe quel autre sous-réseau.
- autorise l’association de table de routage avec le sous-réseau délégué.
- permet à la table de routage associée au sous-réseau délégué d’être associée à n’importe quel autre sous-réseau.
- détermine le nombre minimal d’adresses IP dans le sous-réseau délégué.
- indique que l’espace d’adressage IP dans le sous-réseau délégué doit provenir de l’espace d’adressage IP privé (10.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12).
- exige que la configuration DNS personnalisée ait une entrée Azure DNS.
- exige que la délégation soit supprimée avant que le sous-réseau ou le réseau virtuel puisse être supprimé.

Les services injectés peuvent également ajouter leurs propres stratégies comme suit :
- **Stratégies de sécurité** : Collection de règles de sécurité requises pour le fonctionnement d’un service donné.
- **Stratégies de routage** : Collection d’itinéraires requis pour le fonctionnement d’un service donné.

## <a name="what-subnet-delegation-does-not-do"></a>Ce que la délégation de sous-réseau ne fait pas

Les services Azure qui sont injectés dans un sous-réseau délégué disposent toujours de l’ensemble de propriétés de base qui sont disponibles pour les sous-réseaux non délégués, par exemple :
-  Les services Azure peuvent injecter des instances dans les sous-réseaux des clients, mais ils ne peuvent pas avoir d’impact sur les charges de travail existantes.
-  Les stratégies ou itinéraires que ces services appliquent sont flexibles et peuvent être remplacés par le client.

## <a name="next-steps"></a>Étapes suivantes

- [Déléguer un sous-réseau](manage-subnet-delegation.md)
