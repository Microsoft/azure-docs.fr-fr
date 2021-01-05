---
title: Fichier include
titleSuffix: Azure
description: Fichier include
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: include
ms.date: 3/18/2020
ms.author: derekol
ms.openlocfilehash: e5804aa1b005e670d8b430b1c0a3bd62efd0bb06
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81687053"
---
1. Sélectionnez la connexion de Peering que vous souhaitez activer pour Microsoft Azure Peering Service. Sélectionnez ensuite **…**  > **Modifier la connexion**.
    > [!div class="mx-imgBorder"]
    > ![Modification de la connexion de peering](../media/setup-direct-modify-editconnection.png)
1. Sous **Utiliser pour Peering Service**, sélectionnez **Activée**, puis **Enregistrer**.
    > [!div class="mx-imgBorder"]
    > ![Peering connection Enable Peering Service](../media/setup-direct-modify-editconnectionsettings-peering-service.png)
1. Sur l’écran **Vue d’ensemble**, les détails du déploiement s’affichent. Une fois le déploiement terminé, sélectionnez **Accéder à la ressource**.
    > [!div class="mx-imgBorder"]
    > ![Your deployment is complete](../media/setup-direct-modify-overview-deployment-complete.png)

1. Dans le volet **Préfixes inscrits**, sélectionnez **Ajouter un préfixe inscrit**.
    > [!div class="mx-imgBorder"]
    > ![Add registered prefix](../media/setup-direct-modify-add-registered-prefix.png)
1. Pour inscrire un préfixe, sélectionnez un **nom** et un **préfixe**, puis sélectionnez **Enregistrer**.
    > [!div class="mx-imgBorder"]
    >  ![Register a prefix](../media/setup-direct-modify-register-a-prefix.png) 

1. Une fois qu’un préfixe est créé, il apparaît dans la liste des **préfixes inscrits**. Sélectionnez le **nom** du préfixe pour afficher plus de détails.
    > [!div class="mx-imgBorder"]
    > ![Registered prefixes and connections](../media/setup-direct-modify-registered-prefixes.png)
1. Dans la page des préfixes inscrits, vous voyez les détails complets, qui incluent la **clé de préfixe** pour chaque préfixe. Cette clé doit être fournie au client à qui ce préfixe aura été attribué par son fournisseur d’accès Internet. Le client peut ensuite inscrire son préfixe dans son abonnement à l’aide de cette clé.
    > [!div class="mx-imgBorder"]
    > ![Préfixe avec clé de préfixe](../media/setup-direct-modify-registered-prefix-detail.png)