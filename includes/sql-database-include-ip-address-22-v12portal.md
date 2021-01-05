---
title: Règles de pare-feu au niveau du serveur
description: Règles de pare-feu au niveau du serveur
keywords: connexion sql,chaîne de connexion
services: sql-database
author: dalechen
manager: craigg
ms.service: sql-database
ms.custom: develop apps
ms.topic: include
ms.date: 07/13/2018
ms.author: ninarn
ms.openlocfilehash: 8d0f9899dbb7599340b8d15ca010a0157011fb9e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177142"
---
1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Dans la liste de gauche, sélectionnez **Tous les services**.

3. Faites défiler l’écran, puis sélectionnez **Serveurs SQL**.

    ![Rechercher votre serveur Azure SQL Database sur le portail][b21-FindServerInPortal]
5. Dans la zone de texte de filtre, tapez les premières lettres du nom de votre serveur. La ligne correspondante s’affiche.

6. Sélectionnez la ligne de votre serveur. Un panneau dédié à votre serveur s’affiche.

7. Dans ce panneau, sélectionnez **Paramètres**.

8. Sélectionnez **Pare-feu**.

    ![Sélectionnez Paramètres, puis Pare-feu][b31-SettingsFirewallNavig]
9. Sélectionnez **Ajouter une adresse IP cliente**. Dans la première zone de texte, tapez un nom pour votre nouvelle règle.

10. Tapez les valeurs d’adresse IP basse et haute de la plage que vous souhaitez autoriser.

    * Pour des raisons pratiques, vous pouvez terminer les valeurs basses par **.0** et les valeurs hautes par **.255**.

11. Sélectionnez **Enregistrer**.

<!-- Image references. -->

[b21-FindServerInPortal]: ./media/sql-database-include-ip-address-22-v12portal/firewall-ip-b21-v12portal-findsvr.png

[b31-SettingsFirewallNavig]: ./media/sql-database-include-ip-address-22-v12portal/firewall-ip-b31-v12portal-settingsfirewall.png

[b41-AddRange]: ./media/sql-database-include-ip-address-22-v12portal/firewall-ip-b41-v12portal-addrange.png



<!--
These includes/ files are a sequenced set, but you can pick and choose:

includes/sql-database-include-ip-address-22-v12portal.md
? includes/sql-database-include-ip-address-*.md
-->
