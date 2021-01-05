---
title: Sécurité physique des HSM - HSM dédié Azure | Microsoft Docs
description: Informations sur la sécurité physique des appareils du service HSM dédié Azure dans les centres de données
services: dedicated-hsm
author: msmbaldwin
manager: rkarlin
ms.custom: mvc, seodec18
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 12/07/2018
ms.author: mbaldwin
ms.openlocfilehash: ede1af4625d06af4e280eda86d09ae1db3dfdfd1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "70881028"
---
# <a name="azure-dedicated-hsm-physical-security"></a>Sécurité physique du service HSM dédié Azure

Le service HSM dédié Azure vous permet de respecter les exigences de sécurité avancée pour le stockage de clés. Il est géré selon les pratiques de sécurité strictes décrites ci-dessous tout au long de son cycle de vie complet de façon à répondre aux besoins des clients.

## <a name="security-through-procurement"></a>La sécurité tout au long de l’approvisionnement

Microsoft suit un processus d’approvisionnement sécurisé. Nous gérons la chaîne de responsabilité et veillons à ce que l’appareil spécifique qui est commandé et livré correspond bien à l’appareil qui parvient à nos centres de données. Les appareils sont expédiés dans des emballages de protection en plastique. Ils sont maintenus dans une zone de stockage sécurisée jusqu’à ce qu’ils soient mis en service dans la galerie de données du centre de données.  Les racks contenant les appareils HSM sont considérés comme ayant un fort impact commercial (HBI). Les appareils sont verrouillés et font l’objet d’une surveillance vidéo de tous les instants à l’avant et à l’arrière.

## <a name="security-through-deployment"></a>La sécurité tout au long du déploiement

Les modules HSM sont installés dans des racks avec les composants réseau associés. Une fois installés, ils doivent être configurés avant d’être mis à disposition dans le cadre du service HSM dédié Azure. Cette activité de configuration est assurée par des employés Microsoft qui ont été soigneusement sélectionnés. L’accès est limité aux employés idoines, qui pratiquent une administration « juste-à-temps » (ou JIT), et seulement pour la durée nécessaire. Les procédures et les systèmes utilisés prévoient également la journalisation de toute l’activité liée aux appareils HSM.

## <a name="security-in-operations"></a>La sécurité dans les opérations

Les modules HSM étant des appliances matérielles (le module HSM proprement dit est en fait une carte PCI logée à l’intérieur de l’appliance), des problèmes au niveau des composants ne sont pas à écarter. Ces problèmes peuvent notamment toucher le ventilateur ou le bloc d’alimentation. Ce type d’événement nécessite des activités de maintenance ou de réparation afin de remplacer les composants échangeables.

### <a name="component-replacement"></a>Remplacement de composants

Dès lors qu’un appareil est provisionné et qu’il est géré par le client, le bloc d’alimentation échangeable à chaud est le seul composant à pouvoir être remplacé. Ce composant se trouve en dehors de la limite de sécurité et n’occasionne pas un événement de falsification. Un système de création de tickets est utilisé pour autoriser un ingénieur Microsoft à accéder à l’arrière du rack HBI. Dès que le ticket est traité, une clé physique temporaire est remise. Cette clé permet à l’ingénieur d’accéder à l’appareil et d’échanger le composant affecté. Tout autre accès (c’est-à-dire, susceptible d’occasionner un événement de falsification) n’est possible que si l’appareil n’est pas alloué à un client, ce qui réduit les risques de sécurité et de disponibilité.  

### <a name="device-replacement"></a>Remplacement d’appareil

En cas de défaillance totale de l’appareil, le processus suivi est similaire à celui employé à l’occasion d’une défaillance de composant. Si un client ne parvient pas à mettre l’appareil à zéro ou si celui-ci est dans un état inconnu, l’appareil contenant des données est retiré et placé dans un compartiment de destruction à l’intérieur du rack. Les appareils placés dans ce compartiment sont détruits de façon contrôlée et sécurisée. Aucun appareil de rack HBI contenant des données ne quitte un centre de données Microsoft.

### <a name="other-rack-access-activities"></a>Autres activités nécessitant un accès au rack

Si un ingénieur Microsoft doit accéder au rack utilisé par des appareils HSM (par exemple, pour assurer la maintenance des appareils réseau), des procédures de sécurité standard sont employées pour accéder au rack sécurisé HBI. Tous les accès font l’objet d’une vidéosurveillance. Les appareils HSM étant certifiés [FIPS 140-2 de niveau 3](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.140-2.pdf), tout accès non autorisé aux appareils HSM est signalé au client et les données sont mises à zéro.

## <a name="logical-level-security-considerations"></a>Considérations relatives à la sécurité de niveau logique

Les modules HSM sont provisionnés sur un réseau virtuel créé par le client. Il s’agit de l’espace d’adressage IUP privé du client.  Cette configuration offre une isolation appréciable au niveau du réseau logique et garantit un accès limité au client. Cela sous-entend que tous les contrôles de sécurité de niveau logique sont de la responsabilité du client.

## <a name="next-steps"></a>Étapes suivantes

Il est recommandé de bien comprendre tous les concepts clés du service, comme par exemple la haute disponibilité, la sécurité et la prise en charge, avant de procéder au provisionnement des appareils et à la conception ou au déploiement d’applications.

* [Haute disponibilité](high-availability.md)
* [Mise en réseau](networking.md)
* [Prise en charge](supportability.md)
* [Surveillance](monitoring.md)
* [Architecture de déploiement](deployment-architecture.md)