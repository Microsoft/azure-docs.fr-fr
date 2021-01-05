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
ms.openlocfilehash: c3212acd5015edbb639b8904b885c718d654e5b4
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96356259"
---
Cette section décrit comment effectuer les opérations de modification suivantes pour le Peering direct.

### <a name="add-direct-peering-connections"></a>Ajouter les connexions de Peering direct
1. Sélectionnez le bouton **+ Ajouter des connexions** et configurez une nouvelle connexion de Peering.
    > [!div class="mx-imgBorder"]
    > ![La page AshburnPeering – Connections donne la liste des connexions. Le bouton + Ajouter des connexions est encadré.](../media/setup-direct-modify-addconnection.png)

1. Remplissez le formulaire **Connexion de Peering direct**, puis sélectionnez **Enregistrer**. Pour obtenir de l’aide sur la configuration d’une connexion de Peering, consultez les étapes de la section « Créer et approvisionner un Peering direct ».
    > [!div class="mx-imgBorder"]
    > ![Direct Peering Connection form](../media/setup-direct-modify-savenewconnection.png)

### <a name="remove-direct-peering-connections"></a>Supprimer les connexions de Peering direct

Pour le moment, le Portail Azure ne prend pas en charge la suppression d’une connexion. Pour plus d’informations, contactez l’équipe [Peering Microsoft](mailto:peeringexperience@microsoft.com).

### <a name="upgrade-or-downgrade-bandwidth-on-active-connections"></a>Mettre à niveau ou rétrograder la bande passante sur les connexions actives
1. Sélectionnez une connexion de Peering que vous souhaitez modifier, puis sélectionnez **…**  > **Modifier la connexion**.
    > [!div class="mx-imgBorder"]
    > ![Modifier la connexion](../media/setup-direct-modify-editconnection.png)

1. Modifiez la bande passante en déplaçant le curseur, puis sélectionnez **Enregistrer**.
    > [!div class="mx-imgBorder"]
    > ![Modify bandwidth](../media/setup-direct-modify-editconnectionsettings.png)

### <a name="add-ipv4-or-ipv6-session-information-on-active-connections"></a>Ajouter des informations de session IPv4 ou IPv6 sur des connexions actives
1. Sélectionnez une connexion de Peering que vous souhaitez modifier, puis sélectionnez **…**  > **Modifier la connexion** comme indiqué à l’étape 1.
1. Entrez les informations **Préfixe IPv4 de session** ou **Préfixe IPv6 de session**, puis sélectionnez **Enregistrer**.

### <a name="remove-ipv4-or-ipv6-session-information-on-active-connections"></a>Supprimer des informations de session IPv4 ou IPv6 sur des connexions actives
La suppression d’informations **Préfixe IPv4 de session** ou **Préfixe IPv6 de session** n’est pas actuellement prise en charge dans le portail. Pour plus d’informations, contactez l’équipe [Peering Microsoft](mailto:peeringexperience@microsoft.com).
