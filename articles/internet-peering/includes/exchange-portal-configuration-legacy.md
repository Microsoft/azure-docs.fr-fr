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
ms.openlocfilehash: b8869e93a7156b24d61ac555c95b9ca7f850ae34
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81678568"
---
1. Dans la page **Créer un Peering**, sur l’onglet **Configuration**, remplissez les cases comme illustré ici.

    > [!div class="mx-imgBorder"]
    > ![Create a Peering page Exchange peering type](../media/setup-exchange-conf-tab.png)

    * Dans **Type de Peering**, sélectionnez **Exchange**.
    * Sélectionnez **Gratuit de base** pour la **SKU**.
    * Sélectionnez l’emplacement **Zone urbaine** dans lequel vous souhaitez convertir le Peering en ressource Azure. Si vous avez des connexions de Peering avec Microsoft dans l’emplacement **Zone urbaine** sélectionné qui ne sont pas converties en ressource Azure, elles seront répertoriées dans la section **Connexions de Peering** comme illustré. Vous pouvez maintenant convertir ces connexions de Peering en ressource Azure.

        > [!div class="mx-imgBorder"]
        > ![Peering connections list](../media/setup-exchange-legacy-conf-tab.png)

        > [!NOTE]
        > Vous ne pouvez pas modifier les paramètres pour les connexions de Peering héritées. Si vous souhaitez ajouter des connexions de Peering supplémentaires avec Microsoft dans l’emplacement **Zone urbaine** sélectionné, sélectionnez **Créer nouveau**. Pour plus d’informations, consultez [Créer ou modifier un Peering Exchange à l’aide du portail](../howto-exchange-portal.md).
        >

1. Sélectionnez **Revoir + créer**. Notez que le portail effectue une validation de base des informations que vous avez entrées. Un ruban en haut de l’écran affiche le message *Exécution de la validation finale…* .

    > [!div class="mx-imgBorder"]
    > ![Peering Validation tab](../media/setup-direct-review-tab-validation.png)

1. Lorsque le message devient *Validation réussie*, vérifiez vos informations. Envoyez la demande en sélectionnant **Créer**. Si vous avez besoin de modifier votre demande, sélectionnez **Précédent** et répétez les étapes.

    > [!div class="mx-imgBorder"]
    > ![Peering submission](../media/setup-exchange-review-tab-submit.png)

1. Une fois que vous avez envoyé la demande, attendez que le déploiement se termine. Si le déploiement échoue, contactez l’assistance [Peering Microsoft](mailto:peering@microsoft.com). Un déploiement réussi apparaît comme illustré.

    > [!div class="mx-imgBorder"]
    > ![Peering success](../media/setup-direct-success.png)
