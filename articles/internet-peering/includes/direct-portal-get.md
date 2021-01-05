---
title: Fichier include
titleSuffix: Azure
description: Fichier include
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: include
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: 3507aacc68de25f7368cbe3cda917077564c56eb
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96356242"
---
1. Accédez à **Groupes de ressources** et sélectionnez le groupe de ressources que vous avez sélectionné lors de la création de la ressource **Peering**. Utilisez la case **Filtre** si la liste des groupes de ressources est trop longue.

    > [!div class="mx-imgBorder"]
    > ![Groupes de ressources](../media/setup-direct-get-resourcegroup.png)

1. Sélectionnez la ressource **Peering** que vous avez créée.

    > [!div class="mx-imgBorder"]
    > ![La page Vue d’ensemble est sélectionnée dans le volet gauche. Elle affiche des informations sur PeeringResourceGroup. Dans la liste Peering, AshburnPeering est encadré.](../media/setup-direct-get-open.png)

1. La page **Vue d’ensemble** affiche des informations générales, comme illustré ici.

    > [!div class="mx-imgBorder"]
    > ![Peering resource Overview pane](../media/setup-direct-get-overview.png)

1. Sur la gauche, sélectionnez **Informations ASN** pour afficher les informations envoyées lorsque vous avez créé PeerAsn.

    > [!div class="mx-imgBorder"]
    > ![Peering resource ASN information](../media/setup-direct-get-asninfo.png)

1. Sur la gauche, sélectionnez **Connexions**. En haut de l’écran, vous voyez un résumé des connexions par Peering entre votre ASN et Microsoft, sur différentes installations au sein de la zone urbaine. Vous pouvez également accéder au résumé des connexions à partir de la page **Vue d’ensemble** en sélectionnant **Connexions** au centre du volet, comme illustré.

    > [!div class="mx-imgBorder"]
    > ![Peering resource Connections](../media/setup-direct-get-connectionssummary.png)

    * **État de la connexion** correspond à l’état de configuration de la connexion de Peering. Les états affichés dans ce champ suivent le diagramme d’état indiqué dans la [procédure pas à pas pour le Peering direct](../walkthrough-direct-all.md).
    * **État de session IPv4** et **État de session IPv6** correspondent respectivement aux états de session BGP IPv4 et IPv6. 
    * Lorsque vous sélectionnez une ligne en haut de l’écran, la section **Connexion** dans la partie inférieure affiche les détails de chaque connexion. Sélectionnez les flèches pour développer **Configuration**, **Adresse IPv4** et **Adresse IPv6**.
