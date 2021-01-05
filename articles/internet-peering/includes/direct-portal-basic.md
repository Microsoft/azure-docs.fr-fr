---
title: Fichier include
titleSuffix: Azure
description: Fichier include
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: include
ms.date: 5/22/2020
ms.author: derekol
ms.openlocfilehash: e2804c8f9b1af89ac0ea86ec14136df66d900060
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83846128"
---
1. Sélectionnez **Créer une ressource** > **Afficher tout**.

    > [!div class="mx-imgBorder"]
    > ![Recherchez le peering](../media/setup-seeall.png)

1. Recherchez **Peering** dans la barre de recherche et appuyez sur la touche **Entrée**. À partir des résultats, sélectionnez la ressource **Peering**.

    > [!div class="mx-imgBorder"]
    > ![Lancer le peering](../media/setup-launch.png)

1. Une fois que le **Peering** commence, consultez la page pour en savoir plus. Quand vous êtes prêt, sélectionnez **Créer**.

    > [!div class="mx-imgBorder"]
    > ![Créer un peering](../media/setup-create.png)

1. Dans la page **Créer un Peering**, sur l’onglet **Informations de base**, remplissez les cases comme illustré ici.

    > [!div class="mx-imgBorder"]
    > ![Peering Basics tab](../media/setup-basics-tab.png)

    * Sélectionnez votre **abonnement** Azure.
    * Pour le **groupe de ressources**, sélectionnez un groupe de ressources existant dans la liste déroulante ou créez un nouveau groupe en sélectionnant **Créer nouveau**. Dans cet exemple, ,nous allons créer un nouveau groupe de ressources.
    * **Nom** correspond au nom de la ressource, et vous pouvez le choisir librement.
    * **Région** est sélectionné automatiquement si vous avez choisi un groupe de ressources existant. Si vous avez choisi de créer un nouveau groupe de ressources, vous devez également choisir la région Azure dans laquelle vous souhaitez que la ressource réside.

        > [!NOTE]
        > La région dans laquelle un groupe de ressources réside est indépendante de l’emplacement où vous souhaitez créer le Peering avec Microsoft. Toutefois, il est recommandé d’organiser vos ressources de Peering dans des groupes de ressources qui résident dans les régions Azure les plus proches. Par exemple : pour les Peerings à Ashburn, vous pouvez créer un groupe de ressources dans les régions USA Est ou USA Est2.

    * Sélectionnez votre ASN dans la case **Numéro ASN de l’homologue**.

        > [!IMPORTANT]
        > * Vous pouvez uniquement choisir un ASN dont le paramètre ValidationState est « approuvé » avant de soumettre une demande de Peering. Si vous venez d’envoyer votre demande PeerAsn, patientez environ 12 heures pour que l’association ASN soit « approuvée ». Si l’ASN que vous sélectionnez est en attente de validation, un message d’erreur s’affiche. 
        > * Si vous ne voyez pas l’ASN que vous devez choisir, vérifiez que vous avez sélectionné le bon abonnement. Si c’est le cas, vérifiez si vous avez déjà créé PeerAsn à l’aide de [l’association du numéro ASN de l’homologue à un abonnement Azure](../howto-subscription-association-portal.md).

        > [!div class="mx-imgBorder"]
        > ![Peering Basics filled in](../media/setup-direct-basics-filled-tab.png)

    * Sélectionnez **Suivant : Configuration >** pour continuer.
