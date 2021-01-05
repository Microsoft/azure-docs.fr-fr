---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/22/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 48f33514510618abadf329a11a9ab71a020be0bd
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "92479566"
---
1. Accédez à la page de votre passerelle de réseau virtuel. Vous pouvez accéder à la passerelle en accédant au **Nom de votre VNet -> Vue d’ensemble -> Appareils connectés -> Nom de votre passerelle**, bien qu’il existe de nombreuses autres façons de naviguer.
1. Dans la page de votre passerelle, sélectionnez **Connexions**. En haut du panneau Connexions, sélectionnez **+Ajouter** pour ouvrir la page **Ajouter une connexion**.

   :::image type="content" source="./media/vpn-gateway-add-site-to-site-connection-portal-include/connection.png" alt-text="Connexion de site à site":::
1. Sur la page **Ajouter une connexion**, configurez les valeurs de votre connexion.

   * **Nom :** Nommez votre connexion.
   * **Type de connexion :** Sélectionnez **Site à site (IPsec)** .
   * **Passerelle de réseau virtuel :** La valeur est fixe, car vous vous connectez à partir de cette passerelle.
   * **Passerelle de réseau local :** Sélectionnez **Choisir une passerelle de réseau local**, puis sélectionnez la passerelle de réseau local à utiliser.
   * **Clé partagée** : la valeur doit correspondre à celle que vous utilisez pour votre périphérique VPN local. Dans cet exemple, nous avons utilisé « abc123 », mais vous pouvez (et devriez) utiliser une valeur plus complexe. L’important, c’est que la valeur que vous spécifiez ici doit être identique à celle spécifiée lors de la configuration de votre périphérique VPN.
   * Laissez l’option **Utiliser une adresse IP privée Azure** décochée.
   * Laissez l’option **Activer BGP** décochée.
   * Sélectionnez **IKEv2**.
   * Les autres valeurs pour **abonnement**, **groupe de ressources**, et **emplacement** sont fixes.

1. Sélectionnez **OK** pour créer votre connexion. Le message *Création de la connexion* clignote à l'écran.
1. Vous pouvez afficher la connexion dans la page **Connexions** de la passerelle de réseau virtuel. L’état passe de *Inconnu* à *Connexion*, puis à *Réussi*.
