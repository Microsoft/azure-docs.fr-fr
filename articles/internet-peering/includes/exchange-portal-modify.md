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
ms.openlocfilehash: e06b5261ca6923e158c818d236a30cf6ebff189b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81680922"
---
Cette section décrit comment effectuer les opérations de modification suivantes pour le Peering direct.

### <a name="add-exchange-peering-connections"></a>Ajouter des connexions de Peering Exchange

1. Sélectionnez le bouton **+ Ajouter des connexions** et configurez une nouvelle connexion de Peering.
    > [!div class="mx-imgBorder"]
    > ![Affichage des ressources de peering](../media/setup-exchange-modify-addconnection.png)
1. Remplissez le formulaire **Connexion de Peering Exchange**, puis sélectionnez **Enregistrer**. Pour obtenir de l’aide sur la configuration d’une connexion de Peering, consultez les étapes de la section « Créer et approvisionner un Peering direct ».
    > [!div class="mx-imgBorder"]
    > ![Exchange Peering Connection form](../media/setup-exchange-modify-savenewconnection.png)

### <a name="remove-exchange-peering-connections"></a>Supprimer des connexions de peering Exchange

1. Sélectionnez une connexion de Peering que vous souhaitez supprimer, puis sélectionnez **…**  > **Supprimer la connexion**.
    > [!div class="mx-imgBorder"]
    > ![Delete connection button](../media/setup-exchange-modify-deleteconnection.png)
1. Entrez l’ID de ressource dans la case **Confirmer la suppression**, puis sélectionnez **Supprimer**.
    > [!div class="mx-imgBorder"]
    > ![Delete confirmation](../media/setup-exchange-modify-deleteconnectionconfirm.png)

### <a name="add-an-ipv4-or-ipv6-session-on-active-connections"></a>Ajouter une session IPv4 ou IPv6 sur des connexions actives

1. Sélectionnez une connexion de Peering que vous souhaitez modifier, puis sélectionnez **…**  > **Modifier la connexion**.
    > [!div class="mx-imgBorder"]
    > ![Edit connection button](../media/setup-exchange-modify-editconnection.png)
1. Ajoutez des informations sur l’**adresse IPv4** ou l’**adresse IPv6**, puis sélectionnez **Enregistrer**.
    > [!div class="mx-imgBorder"]
    > ![Peering connection modifications](../media/setup-exchange-modify-editconnectionsettings.png)

### <a name="remove-an-ipv4-or-ipv6-session-on-active-connections"></a>Supprimer une session IPv4 ou IPv6 sur des connexions actives

La suppression d’une session IPv4 ou IPv6 sur une connexion existante n’est actuellement pas prise en charge sur le portail. Pour plus d’informations, contactez l’équipe [Peering Microsoft](mailto:peeringexperience@microsoft.com).