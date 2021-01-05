---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 5b9e036816aa532d32b1b4305ef6ae646ae05bae
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67176952"
---
Le tableau suivant répertorie la configuration requise pour les passerelles VPN basées sur des stratégies ou des itinéraires. Cette table s’applique aux modèles de déploiement classique et Resource Manager. Pour le modèle classique, les passerelles VPN basées sur des stratégies sont identiques aux passerelles statiques et les passerelles basées sur des itinéraires sont identiques aux passerelles dynamiques.

|  | **Passerelle VPN basée sur des stratégies De base** | **Passerelle VPN basée sur des itinéraires De base** | **Passerelle VPN basée sur des itinéraires Standard** | **Passerelle VPN à hautes performances basée sur des itinéraires** |
| --- | --- | --- | --- | --- |
| **Connectivité de site à site (S2S)** |Configuration de VPN basé sur une stratégie |Configuration de VPN basé sur les itinéraires |Configuration de VPN basé sur les itinéraires |Configuration de VPN basé sur les itinéraires |
| **Connectivité de point à site (P2S)** |Non pris en charge |Prise en charge (peut coexister avec S2S) |Prise en charge (peut coexister avec S2S) |Prise en charge (peut coexister avec S2S) |
| **Méthode d’authentification** |Clé prépartagée |Clé prépartagée pour la connectivité de site à site, certificats pour la connectivité de point à site |Clé prépartagée pour la connectivité de site à site, certificats pour la connectivité de point à site |Clé prépartagée pour la connectivité de site à site, certificats pour la connectivité de point à site |
| **Nombre maximal de connexions de site à site** |1 |10 |10 |30 |
| **Nombre maximal de connexions de point à site** |Non pris en charge |128 |128 |128 |
| **Prise en charge de routage actif (BGP)** |Non pris en charge |Non pris en charge |Prise en charge |Prise en charge |
