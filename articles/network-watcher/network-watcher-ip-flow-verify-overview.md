---
title: Présentation de la vérification des flux IP dans Azure Network Watcher | Microsoft Docs
description: Cette page fournit une vue d’ensemble de la fonctionnalité de vérification des flux IP de Network Watcher
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/30/2017
ms.author: damendo
ms.openlocfilehash: 69aca5e0901a0da8aa98fe310ac220898bf650b2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76845004"
---
# <a name="introduction-to-ip-flow-verify-in-azure-network-watcher"></a>Présentation de la vérification des flux IP dans Azure Network Watcher

La vérification des flux IP vérifie si un paquet est autorisé ou refusé en direction ou en provenance d’une machine virtuelle. Les informations se composent de la direction, du protocole, de l’adresse IP locale, de l’adresse IP distante, du port local et du port distant. Si le paquet est refusé par un groupe de sécurité, le nom de la règle qui a refusé le paquet est renvoyé. Même si toutes les adresses IP source et de destination peuvent être choisies, la vérification des flux IP permet aux administrateurs de diagnostiquer rapidement les problèmes de connectivité en provenance ou à destination d’Internet et en provenance ou à destination de l’environnement local.

La vérification des flux IP examine les règles pour tous les groupes de sécurité réseau (NSG) appliquées à l’interface réseau, par exemple un sous-réseau ou une carte réseau de machine virtuelle. Le flux de trafic est ensuite vérifié en fonction des paramètres configurés vers ou à partir de cette interface réseau. La vérification des flux IP est utile pour confirmer si une règle d’un groupe de sécurité réseau bloque le trafic entrant ou sortant d’une machine virtuelle.

Une instance de Network Watcher doit être créée dans toutes les régions dans lesquelles vous prévoyez d’exécuter la vérification des flux IP. Network Watcher est un service régional et peut uniquement être exécuté sur des ressources de la même région. L’instance utilisée n’affecte pas les résultats de la vérification des flux IP, puisque toute route associée à la carte réseau ou au sous-réseau est toujours retournée.

![1][1]

## <a name="next-steps"></a>Étapes suivantes

Consultez l’article suivant pour savoir si un paquet est autorisé ou refusé pour une machine virtuelle spécifique via le portail. [Vérifier si le trafic est autorisé ou refusé en direction ou en provenance d’une machine virtuelle avec le composant de vérification des flux IP d’Azure Network Watcher](diagnose-vm-network-traffic-filtering-problem.md)

[1]: ./media/network-watcher-ip-flow-verify-overview/figure1.png

