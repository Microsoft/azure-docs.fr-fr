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
ms.openlocfilehash: 632490498b8dd13414657edb9518cd543ac07af6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81678823"
---
1. Dans la page **Créer un Peering**, sur l’onglet **Configuration**, remplissez les cases comme illustré ici.

    > [!div class="mx-imgBorder"]
    > ![Create a Peering page Configuration tab](../media/setup-direct-conf-tab.png)

    * Dans **Type d’appairage**, sélectionnez **Direct**.
    * Dans **Réseau Microsoft**, sélectionnez **AS8075**. Ne sélectionnez pas ASN 8069. Il est réservé aux applications spéciales et utilisé uniquement par [Peering Microsoft](mailto:peering@microsoft.com).
    * Sélectionnez **Gratuit de base** pour la **SKU**. Ne sélectionnez pas Premium Gratuit, car il est réservé aux applications spéciales.
    * Sélectionnez l’emplacement **Zone urbaine** dans lequel vous souhaitez convertir le Peering en ressource Azure. Si vous avez des connexions de Peering avec Microsoft dans l’emplacement **Zone urbaine** sélectionné qui ne sont pas converties en ressource Azure, elles seront répertoriées dans la section **Connexions de Peering** comme illustré. Vous pouvez maintenant convertir ces connexions de Peering en ressource Azure.

        > [!div class="mx-imgBorder"]
        > ![Peering connections list](../media/setup-directlegacy-conf-tab.png)

1. Si vous devez mettre à jour la bande passante, sélectionnez le bouton Modifier d’une ligne pour modifier les paramètres de connexion.

    > [!div class="mx-imgBorder"]
    > ![Edit button](../media/setup-directlegacy-conf-tab-edit.png)

    > [!NOTE]
    > Si vous souhaitez ajouter des connexions de Peering supplémentaires avec Microsoft dans l’emplacement **Zone urbaine** sélectionné, sélectionnez **Créer nouveau**. Pour plus d’informations, consultez [Créer ou modifier un Peering direct à l’aide du portail](../howto-direct-portal.md).
    >

1. Sélectionnez **Revoir + créer**. Notez que le portail effectue une validation de base des informations que vous avez entrées. Un ruban en haut de l’écran affiche le message *Exécution de la validation finale…* .

    > [!div class="mx-imgBorder"]
    > ![Peering Validation tab](../media/setup-direct-review-tab-validation.png)

1. Lorsque le message devient *Validation réussie*, vérifiez vos informations. Envoyez la demande en sélectionnant **Créer**. Pour modifier votre demande, sélectionnez **Précédent** et répétez les étapes.

    > [!div class="mx-imgBorder"]
    > ![Peering submission](../media/setup-direct-review-tab-submit.png)

1. Une fois que vous avez envoyé la demande, attendez que le déploiement se termine. Si le déploiement échoue, contactez l’assistance [Peering Microsoft](mailto:peering@microsoft.com). Un déploiement réussi apparaît comme illustré ici.

    > [!div class="mx-imgBorder"]
    > ![Peering success](../media/setup-direct-success.png)
