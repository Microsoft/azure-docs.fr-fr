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
ms.openlocfilehash: dc018b5d09c9b33c10cd2d54ac6572537e05ed25
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010793"
---
Le tableau suivant répertorie la configuration requise pour les passerelles VPN basées sur des stratégies ou des itinéraires. Cette table s’applique aux modèles de déploiement classique et Resource Manager. Pour le modèle classique, les passerelles VPN basées sur des stratégies sont identiques aux passerelles statiques et les passerelles basées sur des itinéraires sont identiques aux passerelles dynamiques.

|  | **Passerelle VPN basée sur des stratégies De base** | **Passerelle VPN basée sur des itinéraires De base** | **Passerelle VPN basée sur des itinéraires Standard** | **Passerelle VPN à hautes performances basée sur des itinéraires** |
| --- | --- | --- | --- | --- |
| **Connectivité de site à site (S2S)** |Configuration de VPN basé sur une stratégie |Configuration de VPN basé sur les itinéraires |Configuration de VPN basé sur les itinéraires |Configuration de VPN basé sur les itinéraires |
| **Connectivité de point à site (P2S)** |Non pris en charge |Prise en charge (peut coexister avec S2S) |Prise en charge (peut coexister avec S2S) |Prise en charge (peut coexister avec S2S) |
| **Méthode d’authentification** |Clé prépartagée |Clé prépartagée pour la connectivité de site à site, certificats pour la connectivité de point à site |Clé prépartagée pour la connectivité de site à site, certificats pour la connectivité de point à site |Clé prépartagée pour la connectivité de site à site, certificats pour la connectivité de point à site |
| **Nombre maximal de connexions de site à site** |1 |10 |10 |30 |
| **Nombre maximal de connexions de point à site** |Non pris en charge |128 |128 |128 |
| **Routage actif (BGP)** (*) |Non pris en charge |Non pris en charge |Prise en charge |Prise en charge |

  (*) Le protocole BGP n’est pas pris en charge avec le modèle de déploiement Classic.
