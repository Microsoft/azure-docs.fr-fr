---
title: Fichier include
description: Fichier include
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 10/06/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 134f1dc7cb6e53c181b2f518055e5cb758fccf31
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91812721"
---
1. Sous votre WAN virtuel, sélectionnez Hubs, puis **+ Nouveau hub**.

   :::image type="content" source="media/virtual-wan-p2s-hub/new-hub.jpg" alt-text="nouveau hub":::

1. Dans la page Créer un hub virtuel, renseignez les champs suivants.

   * **Région** : sélectionnez la région dans laquelle vous souhaitez déployer le hub virtuel.
   * **Nom** : entrez le nom que vous souhaitez donner à votre hub virtuel.
   * **Espace d’adressage privé du hub** : plage d’adresses du hub en notation CIDR.

   :::image type="content" source="media/virtual-wan-p2s-hub/create-hub.jpg" alt-text="créer un hub virtuel":::

1. Sous l’onglet Point à site, complétez les champs suivants :

   * **Unités d’échelle de la passerelle**, qui représentent la capacité d’agrégation de la passerelle VPN utilisateur.
   * **Configuration point à site**, que vous avez créée à l’étape précédente.
   * **Pool d’adresses des clients**, pour les utilisateurs distants.
   * **Adresse IP de serveur DNS personnalisé**.

   :::image type="content" source="media/virtual-wan-p2s-hub/hub-with-p2s.png" alt-text="hub avec point à site":::

1. Sélectionnez **Revoir + créer**.
1. Dans la page **Validation réussie**, sélectionnez **Créer**.