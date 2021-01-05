---
title: Fichier Include
description: Fichier Include
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 11/04/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 6f910dbe91ed8e1cb65eefa6dfc48c72a689bf25
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "73491585"
---
1. Localisez l’instance Virtual WAN que vous avez créée. Dans la page Virtual WAN, sous la section **Connectivité**, sélectionnez **Hubs**.
2. Dans la page Hubs, cliquez sur **+Nouveau hub** pour ouvrir la page **Créer un hub virtuel**.
3. Dans la page **Créer un hub virtuel**, sous l’onglet **Fonctions de base**, renseignez les champs suivants :

   ![Concepts de base](./media/virtual-wan-tutorial-er-hub-include/hub1.png "Concepts de base")

    **Détails du projet**

   * Région (précédemment appelée Emplacement)
   * Name
   * Espace d’adressage privé du hub. L’espace d’adressage minimal pour créer un hub est /24. Cela implique que toute plage comprise entre /25 et /32 produit une erreur lors de la création.
4. Sélectionnez l’**onglet ExpressRoute**.

5. Sous l’onglet **ExpressRoute**, complétez les champs suivants :

   ![ExpressRoute](./media/virtual-wan-tutorial-er-hub-include/hub2.png "ExpressRoute")

   * Sélectionnez **Oui** pour créer une passerelle **ExpressRoute**.
   * Dans la liste déroulante, sélectionnez la valeur **Unités d’échelle de la passerelle**.
6. Sélectionnez **Vérifier + créer** pour valider.
7. Sélectionnez **Créer** pour créer le hub. Après 30 minutes, **actualisez** pour afficher le hub dans la page **Hubs**. Sélectionnez **Accéder à la ressource** pour accéder à ressource.