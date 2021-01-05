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
ms.openlocfilehash: 268703081a2a40e8bcc665889eaeaf8edd673bfd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81680943"
---
1. Dans la page **Créer un Peering**, sous l’onglet **Configuration**, remplissez les cases comme illustré.

    > [!div class="mx-imgBorder"]
    > ![Create a Peering page Exchange peering type](../media/setup-exchange-conf-tab.png)

    * Dans **Type de Peering**, sélectionnez **Exchange**.
    * Sélectionnez **Gratuit de base** pour la **SKU**.
    * Sélectionnez l’emplacement **Zone urbaine** comme emplacement où vous souhaitez configurer le Peering.

        > [!NOTE]
        > Si vous avez déjà des connexions de Peering avec Microsoft à l’emplacement **Zone urbaine** sélectionné et que vous utilisez le portail pour la première fois pour configurer le Peering dans cet emplacement, vos connexions de Peering existantes sont répertoriées dans la section **Connexions de Peering** comme illustré. Microsoft convertira automatiquement ces connexions de Peering en ressource Azure afin que vous puissiez les gérer avec les nouvelles connexions de façon centralisée. Pour plus d’informations, consultez [Convertir un Peering Exchange hérité en ressource Azure à l’aide du portail](../howto-legacy-exchange-portal.md).
        >

1. Sous **Connexions Peering**, sélectionnez **Créer nouveau** afin d’ajouter une ligne pour chaque nouvelle connexion que vous souhaitez configurer.

    * Pour configurer ou modifier les paramètres de connexion, sélectionnez le bouton Modifier d’une ligne.

        > [!div class="mx-imgBorder"]
        > ![Edit button](../media/setup-exchange-conf-tab-edit.png)

    * Pour supprimer une ligne, sélectionnez **…**  > **Supprimer**.

        > [!div class="mx-imgBorder"]
        > ![Delete button](../media/setup-exchange-conf-tab-delete.png)

    * Vous devez fournir tous les paramètres d’une connexion, comme illustré ici.

         > [!div class="mx-imgBorder"]
         > ![Exchange Peering Connection page](../media/setup-exchange-conf-tab-connection.png)

        1. Sélectionnez la **fonctionnalité de peering** dans laquelle la connexion doit être configurée.
        1. Dans les cases **Adresse IPv4** et **Adresse IPv6**, entrez respectivement les adresses IPv4 et IPv6 qui seraient configurées dans les routeurs Microsoft à l’aide de la commande neighbor.
        1. Entrez le nombre de préfixes IPv4 et IPv6 que vous publierez dans les cases **Nombre maximum d’adresses IPv4 publiées** et **Nombre maximum d’adresses IPv6 publiées** respectivement.
        1. Sélectionnez **OK** pour enregistrer vos paramètres de connexion.

1. Répétez l’étape pour ajouter d’autres connexions dans toute installation où Microsoft est colocalisée avec votre réseau, au sein de la **zone urbaine** sélectionnée précédemment.

1. Après avoir ajouté toutes les connexions requises, sélectionnez **Vérifier + créer**.

    > [!div class="mx-imgBorder"]
    > ![Peering Configuration tab](../media/setup-exchange-conf-tab-final.png)

1. Notez que le portail exécute la validation de base des informations que vous avez entrées. Un ruban en haut de l’écran affiche le message *Exécution de la validation finale…* .

    > [!div class="mx-imgBorder"]
    > ![Peering Validation tab](../media/setup-direct-review-tab-validation.png)

1. Lorsque le message devient *Validation réussie*, vérifiez vos informations. Envoyez la demande en sélectionnant **Créer**. Pour modifier votre demande, sélectionnez **Précédent** et répétez les étapes.

    > [!div class="mx-imgBorder"]
    > ![Peering submission](../media/setup-exchange-review-tab-submit.png)

1. Une fois que vous avez envoyé la demande, attendez que le déploiement se termine. Si le déploiement échoue, contactez l’assistance [Peering Microsoft](mailto:peering@microsoft.com). Un déploiement réussi apparaît comme illustré ici.

    > [!div class="mx-imgBorder"]
    > ![Peering success](../media/setup-direct-success.png)
