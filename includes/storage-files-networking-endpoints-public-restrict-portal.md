---
title: Fichier include
description: Fichier include
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 6/2/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 6c594dbab51c46c382c21b4d87595cd7322f6036
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84465007"
---
Accédez au compte de stockage dont le point de terminaison public ne doit être accessible qu’à certains réseaux virtuels. Dans la table des matières du compte de stockage, sélectionnez **Pare-feux et réseaux virtuels**. 

En haut de la page, activez la case d’option **Réseaux sélectionnés**. Cela aura pour effet d’afficher un certain nombre de paramètres permettant de contrôler la restriction du point de terminaison public. Cliquez sur **+ Ajouter un réseau virtuel existant** pour sélectionner le réseau virtuel qui doit être autorisé à accéder au compte de stockage via le point de terminaison public. Pour cela, vous devez sélectionner un réseau virtuel et un sous-réseau de ce réseau virtuel. 

Cochez la case **Autoriser les services Microsoft approuvés à accéder à ce compte de service** pour autoriser les services Microsoft tiers approuvés, comme Azure File Sync, à accéder au compte de stockage.

[![Capture d’écran du panneau Pare-feux et réseaux virtuels, avec un réseau virtuel spécifique autorisé à accéder au compte de stockage via le point de terminaison public](media/storage-files-networking-endpoints-public-restrict-portal/restrict-public-endpoint-0.png)](media/storage-files-networking-endpoints-public-restrict-portal/restrict-public-endpoint-0.png#lightbox)