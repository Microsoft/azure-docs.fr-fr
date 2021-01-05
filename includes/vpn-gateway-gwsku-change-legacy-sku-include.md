---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/15/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 4c232e1ce183c6935d625b5bc9987a4981865ae4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010780"
---
Si vous utilisez le modèle de déploiement du Gestionnaire des ressources, vous pouvez basculer vers les nouvelles références SKU de passerelle. Lorsque vous passez d’une ancienne référence SKU de passerelle vers une nouvelle référence SKU, vous supprimez la passerelle VPN existante et créez une nouvelle passerelle VPN.

Flux de travail :

1. Supprimez toutes les connexions à la passerelle de réseau virtuel.
2. Supprimez l’ancienne passerelle VPN.
3. Créez la nouvelle passerelle VPN.
4. Mettez à jour vos périphériques VPN locaux avec la nouvelle adresse IP de passerelle VPN (pour les connexions site à site, S2S).
5. Mettez à jour la valeur de l’adresse IP de passerelle de toutes les passerelles de réseau local d’une connexion entre deux réseaux virtuels qui s’y connecteront.
6. Téléchargez les nouveaux packages de configuration VPN pour les clients point à site (P2S) se connectant au réseau virtuel via cette passerelle VPN.
7. Recréez les connexions à la passerelle de réseau virtuel.

Considérations :

* Pour passer aux nouvelles références SKU, votre passerelle VPN doit être dans le modèle de déploiement du Gestionnaire des ressources.
* Si vous avez une passerelle VPN classique, vous devez continuer à utiliser les anciennes références SKU pour cette passerelle, toutefois, vous pouvez redimensionner entre les anciennes références SKU. Vous ne pouvez pas basculer vers les nouvelles références SKU.
* Un temps d’arrêt de connectivité se produit lorsque vous basculez d’une ancienne référence SKU vers une nouvelle référence SKU.
* Lorsque vous basculez vers une nouvelle référence SKU de passerelle, l’adresse IP publique de votre passerelle VPN change. Cela se produit même si vous spécifiez le même objet d’adresse IP publique que vous utilisiez précédemment.