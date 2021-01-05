---
title: Fichier include
description: Fichier Include
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 10/06/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 317d75dae83e8529bda25897b77824d8cd36e72e
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95553444"
---
1. Dans la page de votre WAN virtuel, sélectionnez **Configurations de VPN utilisateur**.
1. En haut de la page, sélectionnez **Télécharger une configuration de VPN utilisateur**. Quand vous téléchargez la configuration de niveau WAN, vous obtenez un profil VPN utilisateur basé sur Traffic Manager intégré. Pour plus d’informations sur les profils globaux ou un profil basé sur un hub, consultez [Profils hub](../articles/virtual-wan/global-hub-profile.md). Les scénarios de basculement sont simplifiés avec le profil global.

   Si un hub n’est pas disponible pour une raison ou une autre, la gestion du trafic intégré fourni par le service garantit la connectivité (via un hub différent) aux ressources Azure pour les utilisateurs point à site. Vous pouvez toujours télécharger une configuration VPN propre à un hub en accédant au hub. Sous **VPN utilisateur (point à site)** , téléchargez le profil **VPN utilisateur** du hub virtuel.
1. Une fois le fichier créé, vous pouvez le télécharger en sélectionnant le lien correspondant.
1. Utilisez le fichier de profil pour configurer les clients VPN.