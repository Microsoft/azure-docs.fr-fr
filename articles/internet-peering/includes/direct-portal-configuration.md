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
ms.openlocfilehash: fda22346a44388248e37473bc7891b8a130569c4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81681059"
---
1. Dans la page **Créer un Peering**, sur l’onglet **Configuration**, remplissez les cases comme illustré ici.

    * Dans **Type d’appairage**, sélectionnez **Direct**.
    * Dans **Réseau Microsoft**, sélectionnez **AS8075**. Ne créez pas de Peering avec ASN 8069. Il est réservé aux applications spéciales et utilisé uniquement par [Peering Microsoft](mailto:peering@microsoft.com).
    * Sélectionnez **Gratuit de base** pour la **SKU**. Ne sélectionnez pas Premium Gratuit, car il est réservé aux applications spéciales.
    * Sélectionnez l’emplacement **Zone urbaine** comme emplacement où vous souhaitez configurer le Peering.

        > [!NOTE]
        > Si vous avez déjà des connexions de Peering avec Microsoft à l’emplacement **Zone urbaine** sélectionné et que vous utilisez le Portail Azure pour la première fois pour configurer le Peering dans cet emplacement, vos connexions de Peering existantes sont répertoriées dans la section **Connexions de Peering**, comme illustré. Microsoft convertira automatiquement ces connexions de Peering en ressource Azure afin que vous puissiez les gérer avec les nouvelles connexions de façon centralisée. Pour plus d’informations, consultez [Convertir un Peering direct hérité en ressource Azure à l’aide du portail](../howto-legacy-direct-portal.md).
        >

1. Sous **Connexions de Peering**, sélectionnez **Créer nouveau** afin d’ajouter une ligne pour chaque nouvelle connexion que vous souhaitez configurer.

    * Pour configurer ou modifier les paramètres de connexion, sélectionnez le bouton Modifier d’une ligne.

        > [!div class="mx-imgBorder"]
        > ![Edit button](../media/setup-direct-conf-tab-edit.png)
    
    * Pour supprimer une ligne, sélectionnez **…**  > **Supprimer**.

        > [!div class="mx-imgBorder"]
        > ![Delete button](../media/setup-direct-conf-tab-delete.png)

    * Vous devez fournir tous les paramètres d’une connexion, comme illustré ici.

         > [!div class="mx-imgBorder"]
         > ![Direct Peering Connection page](../media/setup-direct-conf-tab-connection.png)

        1. Sélectionnez la **fonctionnalité de peering** dans laquelle la connexion doit être configurée.
        1. **Fournisseur d’adresses de session** est utilisé pour déterminer qui fournit le sous-réseau nécessaire à l’établissement d’une session BGP entre votre réseau et Microsoft. Si vous pouvez fournir le sous-réseau, sélectionnez **Homologue**. Sinon, sélectionnez **Microsoft**. [Peering Microsoft](mailto:peering@microsoft.com) vous contactera alors. Si vous choisissez cette option, Microsoft prendra plus de temps à traiter la demande de Peering. Dans certains cas, Microsoft peut ne pas être en mesure de fournir des sous-réseaux, ce qui entraînera un refus de la demande.
        1. Si vous avez sélectionné **Fournisseur d’adresses de session** en tant qu’**Homologue**, entrez les adresses IPv4 et IPv6 ainsi que les masques de préfixe dans les cases **Préfixe IPv4 de session** et **Préfixe IPv6 de session**, respectivement.
        1. Entrez le nombre de préfixes IPv4 et IPv6 que vous publierez dans les cases **Nombre maximum d’adresses IPv4 publiées** et **Nombre maximum d’adresses IPv6 publiées**, respectivement.
        1. Ajustez le curseur **Bande passante totale** pour refléter la bande passante de la connexion.
        1. Sélectionnez **Enregistrer** pour enregistrer vos paramètres de connexion.

1. Répétez l’étape précédente pour ajouter d’autres connexions dans toute installation où Microsoft est colocalisée avec votre réseau, au sein de la **zone urbaine** sélectionnée précédemment.

1. Après avoir ajouté toutes les connexions requises, sélectionnez **Vérifier + créer**.

    > [!div class="mx-imgBorder"]
    > ![Peering Configuration tab final](../media/setup-direct-conf-tab-final.png)

1. Notez que le portail effectue une validation de base des informations que vous avez entrées. Un ruban en haut de l’écran affiche le message *Exécution de la validation finale…* .

    > [!div class="mx-imgBorder"]
    > ![Peering Validation tab](../media/setup-direct-review-tab-validation.png)

1. Lorsque le message devient *Validation réussie*, vérifiez vos informations. Envoyez la demande en sélectionnant **Créer**. Pour modifier votre demande, sélectionnez **Précédent** et répétez les étapes.

    > [!div class="mx-imgBorder"]
    > ![Peering submission](../media/setup-direct-review-tab-submit.png)

1. Une fois que vous avez envoyé la demande, attendez que le déploiement se termine. Si le déploiement échoue, contactez l’assistance [Peering Microsoft](mailto:peering@microsoft.com). Un déploiement réussi apparaît comme illustré ici.

    > [!div class="mx-imgBorder"]
    > ![Peering success](../media/setup-direct-success.png)
