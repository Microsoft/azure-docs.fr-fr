---
title: Fichier include
description: Fichier include
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 11/04/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 64470b42efeea49b7c778d6dffd88465b8445e36
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90606850"
---
1. Localisez l’instance Virtual WAN que vous avez créée. Dans la page Virtual WAN, sous la section **Connectivité**, sélectionnez **Hubs**.
2. Dans la page Hubs, cliquez sur **+Nouveau hub** pour ouvrir la page **Créer un hub virtuel**.

    ![Capture d’écran montrant le volet Créer un hub virtuel avec l’onglet Informations de base sélectionné.](./media/virtual-wan-tutorial-hub-include/basics.png "Concepts de base")
3. Dans la page **Créer un hub virtuel**, sous l’onglet **Fonctions de base**, renseignez les champs suivants :

    **Détails du projet**

   * Région (précédemment appelée Emplacement)
   * Nom
   * Espace d’adressage privé du hub. L’espace d’adressage minimal pour créer un hub est /24. Cela implique que toute plage comprise entre /25 et /32 produit une erreur lors de la création. Le service Azure Virtual WAN, qui est géré par Microsoft, crée les sous-réseaux appropriés dans le hub virtuel pour les divers passerelles/services (par exemple, les passerelles VPN, les passerelles ExpressRoute, les passerelles VPN utilisateur/point à site, le pare-feu, le routage, etc.). L’utilisateur n’a pas besoin de planifier explicitement l’espace d’adressage de sous-réseau pour les services dans le hub virtuel, car Microsoft s’en charge dans le cadre du service.
4. Sélectionnez **Suivant : Site à site**.

    ![Capture d’écran montrant le volet Créer un hub virtuel avec l’option Site à site sélectionnée.](./media/virtual-wan-tutorial-hub-include/site-to-site.png "De site à site")

5. Sous l’onglet **Site à site**, complétez les champs suivants :

   * Sélectionnez **Oui** pour créer un VPN de site à site.
   * Le champ Numéro de système autonome n’est pas modifiable dans le hub virtuel pour l’instant.
   * Dans la liste déroulante, sélectionnez la valeur **Unités d’échelle de la passerelle**. L’unité d’échelle vous permet de choisir le débit d’agrégats de la passerelle VPN en cours de création dans le hub virtuel auquel connecter les sites. Si vous choisissez 1 unité d’échelle de 500 Mbits/s, cela implique que deux instances sont créées pour la redondance, chacune d’un débit maximal de 500 Mbits/s. Par exemple, si vous avez cinq branches offrant chacune un débit de 10 Mbits/s, vous avez besoin d’un agrégat de 50 Mbits/s à l’extrémité de la tête. La planification de la capacité agrégée de la passerelle VPN Azure doit être effectuée après évaluation de la capacité nécessaire pour prendre en charge le nombre de branches conduisant au hub.
6. Sélectionnez **Vérifier + créer** pour valider.
7. Sélectionnez **Créer** pour créer le hub. Après 30 minutes, **actualisez** pour afficher le hub dans la page **Hubs**. Sélectionnez **Accéder à la ressource** pour accéder à ressource.
