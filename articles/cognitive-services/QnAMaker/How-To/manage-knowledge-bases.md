---
title: Gérer des bases de connaissances – QnA Maker
description: QnA Maker vous permet de gérer vos bases de connaissances en donnant accès aux paramètres et au contenu de la base de connaissances.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 03/18/2020
ms.openlocfilehash: c7bb79cbd67ba2fb151641ddeeb31b54e399b906
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938254"
---
# <a name="create-knowledge-base-and-manage-settings"></a>Créer une base de connaissances et gérer les paramètres

QnA Maker vous permet de gérer vos bases de connaissances en donnant accès aux paramètres et aux sources de données de la base de connaissances.

## <a name="prerequisites"></a>Prérequis

> [!div class="checklist"]
> * Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.
> * Une [ressource QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) créée dans le portail Azure. Rappelez-vous l’ID Azure Active Directory, l’abonnement et le nom de ressource QnA que vous avez sélectionnés au moment de créer la ressource.

## <a name="create-a-knowledge-base"></a>Créer une base de connaissances

1. Connectez-vous au portail [QnAMaker.ai](https://QnAMaker.ai) avec vos informations d’identification Azure.

1. Dans le portail QnA Maker, sélectionnez **Créer une base de connaissances**.

1. Dans la page **Créer**, ignorez l’**étape 1** si vous disposez déjà de votre ressource QnA Maker.

    Si vous n’avez pas encore créé la ressource, sélectionnez **Créer un service QnA**. Vous êtes dirigé vers le [portail Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) pour configurer un service QnA Maker dans votre abonnement. Rappelez-vous l’ID Azure Active Directory, l’abonnement et le nom de ressource QnA que vous avez sélectionnés au moment de créer la ressource.

    Lorsque vous avez terminé de créer la ressource dans le portail Azure, revenez au portail QnA Maker, actualisez la page du navigateur et passez à l’**étape 2**.

1. À l’**étape 3**, sélectionnez votre annuaire Active Directory, votre abonnement, votre service (ressource) et la langue de toutes les bases de connaissances créées dans le service.

   ![Capture d’écran de la sélection d’une base de connaissances du service QnA Maker](../media/qnamaker-quickstart-kb/qnaservice-selection.png)

1. À l’**étape 3**, nommez votre base de connaissances `My Sample QnA KB`.

1. À l’**étape 4**, configurez les paramètres à l’aide du tableau suivant :

    |Paramètre|Valeur|
    |--|--|
    |**Enable multi-turn extraction from URLs, .pdf or .docx files** (Activer l’extraction multitour à partir d’URL, de fichier .pdf ou .docx)|Activé|
    |**Texte de réponse par défaut**| `Quickstart - default answer not found.`|
    |**+ Ajouter une URL**|`https://azure.microsoft.com/en-us/support/faq/`|
    |**Échange de conversation (chit-chat)**|Sélectionnez **Professional**.|

1. À l’**étape 5**, sélectionnez **Create your KB** (Créer votre base de connaissances).

    Le processus d’extraction prend quelques minutes pour lire le document et identifier les questions et réponses.

    Une fois que QnA Maker a créé la base de connaissances, la page **Knowledge base** (Base de connaissances) s’ouvre. Vous pouvez modifier le contenu de la base de connaissances dans cette page.

## <a name="edit-knowledge-base"></a>Modifier la base de connaissances

1.  Sélectionnez **Mes bases de connaissances** dans la barre de navigation supérieure.

       Vous pouvez voir tous les services, que vous avez créés ou qui ont été partagés avec vous, triés dans l’ordre décroissant de la date de **dernière modification**.

       ![Mes bases de connaissances](../media/qnamaker-how-to-edit-kb/my-kbs.png)

1. Sélectionnez une base de connaissances particulière pour y apporter des modifications.

1.  Sélectionnez **Paramètres**. La liste suivante contient des champs que vous pouvez modifier.

       |Objectif|Action|
       |--|--|
       |Ajouter une URL|Vous pouvez ajouter de nouvelles URL pour ajouter du nouveau contenu de FAQ à la base de connaissances en cliquant sur le lien **Manage knowledge base -> '+ Add URL'** (Gérer la base de connaissances -> « + Ajouter une URL »).|
       |Supprimer une URL|Vous pouvez supprimer des URL existantes en sélectionnant l’icône supprimer, la corbeille.|
       |Actualiser le contenu|Si vous souhaitez que la base de connaissances analyse le contenu le plus récent des URL existantes, cochez la case **Refresh** (Actualiser). Cela met à jour la base de connaissances avec le contenu d’URL le plus récent à une seule reprise. Cela ne revient pas à définir une planification régulière des mises à jour.|
       |Ajouter un fichier|Vous pouvez ajouter un document pris en charge pour qu’il fasse partie de la base de connaissances en sélectionnant **Manage knowledge base** (Gérer la base de connaissance), puis -> **+ Add File** (+ Ajouter un fichier).|
    |Importer|Vous pouvez également importer n’importe quelle base de connaissances existante en sélectionnant le bouton **Importer la base de connaissances**. |
    |Update|La mise à jour de la base de connaissances dépend du **niveau tarifaire de gestion** utilisé lors de la création du service QnA Maker associé à votre base de connaissances. Vous pouvez également mettre à jour le niveau de gestion à partir du portail Azure si nécessaire.

  1. Une fois que vous avez fini d’apporter des modifications à la base de connaissances, sélectionnez **Enregistrer et entraîner** dans l’angle supérieur droit de la page afin de conserver les modifications.

       ![Enregistrer et former](../media/qnamaker-how-to-edit-kb/save-and-train.png)

       >[!CAUTION]
       >Si vous quittez la page avant de sélectionner **Save and train** (Enregistrer et former), toutes les modifications seront perdues.



## <a name="manage-large-knowledge-bases"></a>Gérer de grandes bases de connaissances

* **Groupes de source de données** : les questions et réponses sont regroupées en fonction de la source de données dont elles ont été extraites. Vous pouvez développer ou réduire la source de données.

    ![Utilisez la barre de source de données de QnA Maker pour réduire et développer les questions et réponses de source de données](../media/qnamaker-how-to-edit-kb/data-source-grouping.png)

* **Rechercher dans la Base de connaissances** : vous pouvez opérer une recherche dans la base de connaissances en tapant dans la zone de texte en haut de la table de la Base de connaissances. Cliquez sur Entrée pour effectuer des recherches sur le contenu de la question, de la réponse ou des métadonnées. Cliquez sur l’icône X pour supprimer le filtre de recherche.

    ![Utilisez la zone de recherche de QnA Maker au-dessus des questions et réponses pour réduire la vue aux seuls éléments correspondant au filtre](../media/qnamaker-how-to-edit-kb/search-paginate-group.png)

* **Pagination** : parcourez rapidement les sources de données pour gérer de grandes bases de connaissances.

    ![Utilisez les fonctionnalités de pagination de QnA Maker au-dessus des questions et réponses pour faire défiler les pages de questions et réponses](../media/qnamaker-how-to-edit-kb/pagination.png)

## <a name="delete-knowledge-bases"></a>Supprimer les bases de connaissances

La suppression d’une base de connaissances (KB) est une opération définitive. Elle ne peut pas être annulée. Avant de supprimer une base de connaissances, exportez-la à partir de la page **Paramètres** page du portail QnA Maker.

Si vous partagez votre base de connaissances avec des collaborateurs,] (collaboration-knowledge-base.md), puis la supprimez, tout le monde perd l’accès à celle-ci.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur la [gestion de la langue](../index.yml) de toutes les bases de connaissances dans une ressource.

* Modifier les paires de Q/R
* Gérer les ressources Azure utilisées par QnA Maker