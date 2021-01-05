---
title: Personnaliser un modèle de personne avec le site web Video Indexer
titleSuffix: Azure Media Services
description: Découvrez comment personnaliser un modèle de personne avec le site web Video Indexer.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 08/06/2020
ms.author: kumud
ms.openlocfilehash: 9f5a18cc51aa92b53a5391404b1e9d3fc800bb47
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91356619"
---
# <a name="customize-a-person-model-with-the-video-indexer-website"></a>Personnaliser un modèle de personne avec le site web Video Indexer

Video Indexer prend en charge la reconnaissance de célébrités dans du contenu vidéo. La fonctionnalité de reconnaissance de célébrités couvre environ un million de visages en fonction de la source de données couramment demandée comme IMDB, Wikipedia et les principaux influenceurs LinkedIn. Pour une présentation détaillée, consultez [Customize a Person model in Video Indexer](customize-person-model-overview.md) (Personnaliser un modèle de personne dans Video Indexer).

Vous pouvez utiliser le site web Video Indexer pour modifier des visages détectés dans une vidéo, comme décrit dans cette rubrique. Vous pouvez également utiliser l’API, comme décrit dans [Personnaliser un modèle de personne avec l’API Video Indexer](customize-person-model-with-api.md).

## <a name="central-management-of-person-models-in-your-account"></a>Gestion centrale des modèles de personne dans votre compte

1. Pour afficher, modifier et supprimer les modèles de personne dans votre compte, accédez au site web de Video Indexer et connectez-vous.

2. Sélectionnez le bouton de personnalisation du modèle de contenu en haut à droite de la page.

    ![Capture d’écran montrant Video Indexer avec le bouton Personnalisation du modèle de contenu sélectionné.](./media/customize-face-model/content-model-customization.png)

3. Sélectionnez l’onglet Personne.

    Vous voyez le modèle de personne par défaut dans votre compte. Le modèle de personne par défaut conserve les visages vous avez modifiés dans les insights de vos vidéos pour lesquels vous n’avez pas spécifié de modèle de personne personnalisé lors de l’indexation.

    Si vous avez créé d’autres modèles de personne, ils sont également listés sur cette page.

    ![Capture d’écran montrant la boîte de dialogue Personnalisation du modèle de contenu.](./media/customize-face-model/content-model-customization-people-tab.png)

## <a name="create-a-new-person-model"></a>Créer un nouveau modèle de personne

1. Cliquez sur le bouton **+ Ajouter un modèle**.

    ![Capture d’écran montrant la boîte de dialogue Personnalisation du modèle de contenu avec l’onglet Personnes sélectionné.](./media/customize-face-model/add-new-person.png)

2. Entrez le nom du modèle, puis sélectionnez la coche en regard du nom.

    ![Capture d’écran montrant la boîte de dialogue Personnalisation du modèle de contenu avec un nom entré pour le modèle.](./media/customize-face-model/add-new-person2.png)

    Vous avez créé un nouveau modèle de personne. Vous pouvez maintenant ajouter des visages au nouveau modèle de personne.

3. Sélectionnez le bouton de liste déroulante et choisissez **+ Ajouter une personne**.

    ![Capture d’écran montrant la boîte de dialogue Personnalisation du modèle de contenu avec un menu contextuel pour votre nouveau modèle et l’option Ajouter une personne.](./media/customize-face-model/add-new-person3.png)

## <a name="add-a-new-person-to-a-person-model"></a>Ajouter une nouvelle personne à un modèle de personne

> [!NOTE]
> Video Indexer vous permet d’ajouter plusieurs personnes portant le même nom dans un modèle de personne. Toutefois, nous vous recommandons de donner des noms à chaque personne de votre modèle pour plus d’utilisabilité et de clarté.

1. Pour ajouter un nouveau visage à un modèle de personne, sélectionnez le bouton de menu de liste en regard du modèle de personne auquel vous voulez ajouter le visage.

1. Sélectionnez **+ Ajouter une personne** dans le menu.

    ![Capture d’écran montrant la boîte de dialogue Personnalisation du modèle de contenu prête pour l’ajout d’une personne avec le menu contextuel.](./media/customize-face-model/add-new-face.png)

    Une fenêtre contextuelle vous invitera à fournir les informations relatives à la personne. Tapez le nom de la personne, puis sélectionnez la coche.

    ![Capture d’écran montrant les détails d’une personne.](./media/customize-face-model/add-new-face2.png)

    Vous pouvez ensuite choisir à partir de l’Explorateur de fichiers ou glisser et déplacer les images de visage du visage concerné. Video Indexer accepte tous les types de fichiers d’image standard (p. ex. : JPG, PNG, etc.).

    Video Indexer peut détecter les occurrences de cette personne dans les futures vidéos que vous indexerez et dans les vidéos actuelles que vous aviez déjà indexées, en utilisant le modèle de personne auquel vous avez ajouté ce nouveau visage. La reconnaissance de la personne dans vos vidéos actuelles peut prendre un certain temps pour entrer en vigueur, car il s’agit d’un processus de traitement par lots.

## <a name="rename-a-person-model"></a>Renommer un modèle de personne

Vous pouvez renommer tous les modèles de personne dans votre compte, y compris le modèle de personne par défaut. Même si vous renommez votre modèle de personne par défaut, il sera toujours considéré comme le modèle de personne par défaut dans votre compte.

1. Sélectionnez le bouton de menu de liste en regard du modèle de personne que vous voulez renommer.
2. Sélectionnez **Renommer** dans le menu.

    ![Capture d’écran montrant la boîte de dialogue Personnalisation du modèle de contenu avec un menu contextuel contenant l’option permettant de renommer un modèle de personne. ](./media/customize-face-model/rename-person.png)

3. Sélectionnez le nom actuel du modèle et tapez votre nouveau nom.

    ![Capture d’écran montrant le modèle de personne renommé.](./media/customize-face-model/rename-person2.png)

4. Sélectionnez la coche du modèle à renommer.

## <a name="delete-a-person-model"></a>Supprimer un modèle de personne

Vous pouvez supprimer les modèles de personne que vous avez créés dans votre compte. Vous ne pouvez cependant pas supprimer votre modèle de personne par défaut.

1. Sélectionnez **Supprimer** dans le menu.

    ![Capture d’écran montrant la boîte de dialogue Personnalisation du modèle de contenu avec un menu contextuel contenant l’option permettant de supprimer un modèle de personne.](./media/customize-face-model/delete-person.png)

    Une fenêtre contextuelle s’affiche et vous informe que cette action va supprimer le modèle de personne et toutes les personnes et les fichiers qu’il contient. Cette action ne peut pas être annulée.

    ![Capture d’écran montrant une boîte de dialogue de confirmation pour la suppression du modèle de personne.](./media/customize-face-model/delete-person2.png)

1. Si vous êtes sûr, sélectionnez à nouveau Supprimer.

> [!NOTE]
> Les vidéos existantes qui ont été indexées en utilisant ce modèle de personne (maintenant supprimé) ne prendront pas en charge la possibilité de mettre à jour les noms des visages qui s’affichent dans la vidéo. Vous pourrez modifier les noms des visages dans ces vidéos seulement après les avoir réindexées en utilisant un autre modèle de personne. Si vous réindexez sans spécifier un modèle de personne, le modèle par défaut est utilisé.

## <a name="manage-existing-people-in-a-person-model"></a>Gérer la personne existante dans un modèle de personne

Pour visualiser le contenu d’un de vos modèles de personne, sélectionnez la flèche en regard du nom du modèle de personne. La liste déroulante affiche toutes les personnes de ce modèle de personne particulier. Si vous sélectionnez le bouton de menu de liste en regard de chaque personne, vous pouvez voir les options Gérer, Renommer et Supprimer.  

![Capture d’écran montrant un menu contextuel contenant des options pour gérer, renommer et supprimer.](./media/customize-face-model/manage-people.png)

### <a name="rename-a-person"></a>Renommer une personne

1. Pour renommer une personne dans votre modèle de personne, sélectionnez le bouton de menu de liste, puis choisissez **Renommer** dans le menu de liste.
1. Sélectionnez le nom actuel de la personne et tapez votre nouveau nom.
1. Sélectionnez la coche : la personne est alors renommée.

### <a name="delete-a-person"></a>Supprimer une personne

1. Pour supprimer une personne dans votre modèle de personne, sélectionnez le bouton de menu de liste, puis choisissez **Supprimer** dans le menu de liste.
1. Une fenêtre contextuelle vous indique que cette action va supprimer la personne et que cette opération ne peut pas être annulée.
1. Sélectionnez à nouveau **Supprimer** pour supprimer la personne du modèle de personne.

### <a name="manage-a-person"></a>Gérer une personne

Si vous sélectionnez **Gérer**, vous voyez la fenêtre **Détails de la personne** avec tous les visages à partir desquels ce modèle de personne est formé. Ces visages proviennent des occurrences de cette personne dans des vidéos qui utilisent ce modèle de personne ou d’images que vous avez chargées manuellement.

> [!TIP]
> Vous pouvez accéder à la fenêtre **Détails de la personne** en cliquant sur le nom de la personne ou en cliquant sur **Gérer**, comme indiqué ci-dessus.

#### <a name="add-a-face"></a>Ajouter un visage

Vous pouvez ajouter plus de visages à la personne en sélectionnant **Ajouter des images**.

#### <a name="delete-a-face"></a>Supprimer un visage

Sélectionnez l’image que vous souhaitez supprimer et cliquez sur **Supprimer**.

#### <a name="rename-and-delete-the-person"></a>Renommer et supprimer la personne 

Vous pouvez utiliser le volet de gestion pour renommer la personne et supprimer la personne du modèle de personne.

## <a name="use-a-person-model-to-index-a-video"></a>Utiliser un modèle de personne pour indexer une vidéo

Vous pouvez utiliser un modèle de personne pour indexer votre nouvelle vidéo en attribuant le modèle de personne lors du chargement de la vidéo.

Pour utiliser votre modèle de personne sur une nouvelle vidéo, effectuez les étapes suivantes :

1. Sélectionnez le bouton **Charger** en haut de la page.

    ![Capture d’écran montrant le bouton Charger.](./media/customize-face-model/upload.png)

1. Déposez votre fichier vidéo dans le cercle ou accédez à votre fichier.
1. Sélectionnez la flèche **Options avancées**.

    ![Capture d’écran montrant la boîte de dialogue Charger votre fichier multimédia.](./media/customize-face-model/upload2.png)

1. Sélectionnez la liste déroulante, puis le modèle de personne que vous avez créé.

    ![Capture d’écran montrant la boîte de dialogue Charger votre fichier multimédia avec la valeur Par défaut sélectionnée pour le modèle Personnes dans les Options avancées.](./media/customize-face-model/upload3.png)

1. Sélectionnez l’option **Charger** dans le bas de la page : votre nouvelle vidéo est alors indexée en utilisant votre modèle de personne.

Si vous ne spécifiez pas de modèle de personne lors du chargement, Video Indexer indexe la vidéo en utilisant le modèle de personne par défaut de votre compte.

## <a name="use-a-person-model-to-reindex-a-video"></a>Utiliser un modèle de personne pour réindexer une vidéo

Pour utiliser un modèle de personne pour réindexer une vidéo de votre collection, accédez aux vidéos du compte sur la page d’accueil de Video Indexer et pointez sur le nom de la vidéo que vous souhaitez réindexer.

Les options vous permettant de modifier, de supprimer et de réindexer votre vidéo s’affichent.

1. Sélectionnez l’option pour réindexer votre vidéo.

    ![Capture d’écran montrant les vidéos du compte et l’option permettant de réindexer votre vidéo.](./media/customize-face-model/reindex.png)

    Vous pouvez maintenant sélectionner le modèle de personne avec lequel réindexer votre vidéo.
1. Sélectionnez la liste déroulante, puis le modèle de personne que vous souhaitez utiliser.

    ![Capture d’écran montrant les options disponibles pour une vidéo, dont la sélection d’un modèle Personnes.](./media/customize-face-model/reindex2.png)

1. Sélectionnez le bouton **Réindexer** : votre vidéo est alors réindexée en utilisant votre modèle de personne.

Toutes les modifications que vous apportez aux visages détectés et reconnus dans la vidéo que vous venez de réindexer seront enregistrées dans le modèle de personne que vous utilisez pour réindexer la vidéo.

## <a name="managing-people-in-your-videos"></a>Gestion des personnes de vos vidéos

Vous pouvez gérer les visages détectés et les personnes reconnues dans les vidéos que vous indexez en modifiant et en supprimant les visages.

La suppression d’un visage supprime un visage spécifique des insights de la vidéo.

La modification d’un visage renomme un visage qui est détecté et éventuellement reconnu dans votre vidéo. Lorsque vous modifiez un visage dans votre vidéo, ce nom est enregistré comme une entrée de la personne dans le modèle de personne qui a été attribué à la vidéo pendant le chargement et l’indexation.

Si vous n’affectez pas de modèle de personne à la vidéo lors du chargement, votre modification est enregistrée dans le modèle de personne par défaut de votre compte.

### <a name="edit-a-face"></a>Modifier un visage

> [!NOTE]
> Si un modèle de personne a deux ou plusieurs personnes différentes portant le même nom, vous ne serez pas en mesure de marquer ce nom dans les vidéos qui utilisent ce modèle de personne. Vous pourrez apporter des modifications seulement aux personnes qui partagent ce nom dans l’onglet Personne de la page de personnalisation du modèle de contenu dans Video Indexer. C’est pour cette raison qu’il est recommandé d’affecter un nom unique à chaque personne dans votre modèle de personne.

1. Accédez au site web Video Indexer et connectez-vous.
1. Recherchez une vidéo que vous souhaitez afficher et modifier dans votre compte.
1. Pour modifier un visage dans votre vidéo, accédez à l’onglet Insights, puis sélectionnez l’icône de crayon en haut à droite de la fenêtre.

    ![Capture d’écran montrant une vidéo avec un visage inconnu à sélectionner.](./media/customize-face-model/edit-face.png)

1. Sélectionnez un des visages détectés et changez son nom, « Unknown #X » (Inconnu n°X), ou le nom précédemment affecté au visage.
1. Après avoir tapé le nouveau nom, sélectionnez l’icône de coche en regard du nouveau nom. Cette action permet d’enregistrer le nouveau nom, ainsi que de reconnaître et de nommer toutes les occurrences de ce visage dans les autres vidéos actuelles et les vidéos que vous chargerez par la suite. La reconnaissance du visage dans vos autres vidéos actuelles peut prendre un certain temps pour entrer en vigueur, car il s’agit d’un processus de traitement par lots.

Si vous nommez un visage avec le nom d’une personne qui existe dans le modèle de personne utilisé par la vidéo, les images des visages détectés dans cette vidéo de cette personne seront fusionnées avec ce qui se trouve déjà dans le modèle. Si vous donnez un nouveau nom à un visage, une nouvelle entrée de personne est créée dans le modèle de personne que la vidéo utilise.

![Capture d’écran montrant un visage avec un nom ajouté.](./media/customize-face-model/edit-face2.png)

### <a name="delete-a-face"></a>Supprimer un visage

Pour supprimer un visage détecté dans votre vidéo, accédez au volet Insights, puis sélectionnez l’icône de crayon en haut à droite du volet. Sélectionnez l’option **Supprimer** sous le nom du visage. Cette action supprime le visage détecté de la vidéo. Le visage de la personne sera toujours détecté dans les autres vidéos où il apparaît, mais vous pouvez aussi le supprimer de ces vidéos une fois qu’elles ont été indexées.

La personne, si elle avait été nommée, continue également d’exister dans le modèle de personne qui a été utilisé pour indexer la vidéo d’où vous avez supprimé le visage, sauf si vous supprimez spécifiquement la personne du modèle de personne.

![Supprimer un visage dans votre vidéo](./media/customize-face-model/delete-face.png)

## <a name="next-steps"></a>Étapes suivantes

[Personnaliser le modèle de personne à l’aide des API](customize-person-model-with-api.md)
