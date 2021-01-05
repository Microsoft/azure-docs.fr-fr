---
title: Fichier include
description: Fichier include
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 07/09/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 5915830e4521399ad322dd4a6f3926428d811455
ms.sourcegitcommit: f6236e0fa28343cf0e478ab630d43e3fd78b9596
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94942272"
---
Dans un navigateur, accédez au [portail Azure](https://portal.azure.com) et connectez-vous avec votre compte Azure.

1. Dans le portail, sélectionnez **+ Créer une ressource**. Tapez **Virtual WAN** dans la zone de recherche et sélectionnez **Entrée**.
1. Sélectionnez **WAN virtuel** dans les résultats. Dans la page Virtual WAN, sélectionnez **Créer** pour ouvrir la page Créer un WAN.
1. Dans la page **Créer un WAN**, sous l’onglet **Fonctions de base**, renseignez les champs suivants :

   :::image type="content" source="./media/virtual-wan-create-vwan-include/basics.png" alt-text="La capture d’écran montre le volet Créer un WAN avec l’onglet Informations de base sélectionné.":::

   * **Abonnement** : sélectionnez l’abonnement que vous voulez utiliser.
   * **Groupe de ressources** : créez-en un ou utilisez-en un qui existe déjà.
   * **Emplacement du groupe de ressources** : choisissez un emplacement de la ressource dans la liste déroulante. Un WAN est une ressource globale et ne réside pas dans une région particulière. Toutefois, vous devez sélectionner une région pour gérer et localiser la ressource WAN que vous créez.
   * **Nom** : tapez le nom que vous souhaitez donner à votre WAN.
   * **Type** : de base ou standard. Sélectionnez **Standard**. Si vous sélectionnez Basic VWAN (WAN virtuel de base), sachez que les WAN virtuels de base peuvent uniquement contenir des hubs de base, ce qui limite votre type de connexion aux connexions site à site.
1. Quand vous avez fini de renseigner les champs, cliquez sur **Vérifier + Créer**.
1. Après la validation, sélectionnez **Créer** pour créer le WAN virtuel.
