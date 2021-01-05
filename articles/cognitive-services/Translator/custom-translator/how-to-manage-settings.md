---
title: Comment gérer les paramètres ? - Custom Translator
titleSuffix: Azure Cognitive Services
description: Guide pratique pour gérer les paramètres, créer l’espace de travail, partager l’espace de travail et gérer la clé d’abonnement dans Custom Translator.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: e556ca2051fd75eb9bf6411c5479e63554606d48
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94368336"
---
# <a name="how-to-manage-settings"></a>Comment gérer les paramètres

Dans la page de paramètres de Custom Translator, vous pouvez créer un espace de travail, modifier votre clé d’abonnement Translator et supprimer l’espace de travail.

Pour accéder à la page des paramètres :

1. Se connecter au portail [Custom Translator](https://portal.customtranslator.azure.ai/).
2. Dans le portail Custom Translator, cliquez sur l’icône d’engrenage dans la barre latérale.

    ![Lien de paramétrage](media/how-to/how-to-settings.png)

## <a name="associating-translator-subscription"></a>Association d’abonnements au Traducteur

Vous devez avoir une clé d’abonnement à Translator associée à votre espace de travail pour entrainer ou déployer des modèles.

Si vous n’avez pas d’abonnement, procédez comme suit :

1. Abonnez-vous pour créer une ressource Translator. Suivez la [procédure d’inscription à Translator](../translator-how-to-signup.md) pour vous abonner et acquérir une clé Translator.
2. Notez la clé de votre abonnement à Translator. Key1 ou Key2 sont acceptables.
3. Retournez au portail Custom Translator.

## <a name="create-a-new-workspace"></a>Créer un espace de travail

1. Cliquez sur le bouton « + Créer un espace de travail » dans la barre latérale « Custom Translator ».

    ![Créer un espace de travail](media/how-to/create-new-workspace.png)

2. Dans la boîte de dialogue, entrez le nom du nouvel espace de travail.
3. Cliquez sur « Suivant ».
4. Choisissez le type d’abonnement.
5. Sélectionnez la région de l’abonnement. La région doit correspondre à la région sélectionnée lors de la création de la clé de ressource Translator.
6. Entrez la clé de votre abonnement à Translator, puis cliquez sur le bouton « Enregistrer ».

    ![Créer une boîte de dialogue d’espace de travail](media/how-to/create-new-workspace-dialog.png)

>[!Note]
>Custom Translator ne prend pas en charge la création d’espace de travail pour la ressource de l’API de traduction de texte Translator Text (ou clé d’abonnement Azure) créée dans [Réseau virtuel activé](../../../api-management/api-management-using-with-vnet.md).

### <a name="modify-existing-key"></a>Modifier une clé existante

1. Accédez à la page « Paramètres » de votre espace de travail.
2. Cliquez sur « Changer la clé ».

    ![Comment ajouter une clé d’abonnement](media/how-to/how-to-add-subscription-key.png)

3. Dans la boîte de dialogue, entrez la clé de votre abonnement à Translator, puis cliquez sur le bouton « Enregistrer ».

    ![Boîte de dialogue Ajout d’une clé d’abonnement](media/how-to/how-to-add-subscription-key-dialog.png)

## <a name="manage-your-workspace"></a>Gérer votre espace de travail

Un espace de travail est une zone de travail pour la composition et la création de votre système de traduction personnalisé. Un espace de travail peut contenir plusieurs projets, modèles et documents.

Si différentes parties de votre travail doivent être partagées avec différentes personnes, il peut être utile de créer plusieurs espaces de travail.

## <a name="share-your-workspace"></a>Partager votre espace de travail

Dans Custom Translator, vous pouvez partager votre espace de travail avec d’autres utilisateurs si certaines parties de votre travail doivent être partagées avec différentes personnes.

1. Accédez à la page « Paramètres » de l’espace de travail.
2. Cliquez sur le bouton « Ajouter des personnes » dans la section « Paramètres de partage ».

    ![Partager l’espace de travail](media/how-to/share-workspace.png)

3. Dans la boîte de dialogue, entrez une liste séparée par des virgules des adresses e-mail avec lesquelles vous souhaitez partager cet espace de travail. Assurez-vous que vous partagez bien avec l’adresse e-mail que la personne concernée utilise pour se connecter à Custom Translator. Ensuite, sélectionnez le niveau d’autorisation de partage approprié et cliquez sur le bouton « Enregistrer ».

    ![Boîte de dialogue Partager l’espace de travail](media/how-to/share-workspace-dialog.png)

4. Si votre espace de travail a toujours le nom par défaut « Mon espace de travail », vous devrez le modifier avant de le partager.
5. Cliquez sur « Enregistrer ».

## <a name="sharing-permissions"></a>Autorisations de partage

1. **Lecteur :** un lecteur dans l’espace de travail sera en mesure d’afficher toutes les informations dans l’espace de travail.

2. **Éditeur :** un éditeur dans l’espace de travail sera en mesure d’ajouter des documents, de former des modèles et de supprimer des documents et des projets. Il peut ajouter une clé d’abonnement, mais ne peut pas modifier les personnes avec lesquelles l’espace de travail est partagé, supprimer l’espace de travail ou modifier son nom.

3. **Propriétaire :** un propriétaire dispose d’autorisations complètes pour l’espace de travail.

## <a name="change-sharing-permission"></a>Autorisation de modification du partage

Quand un espace de travail est partagé, la section « Paramètres de partage » affiche toutes les adresses e-mail avec lesquelles il est partagé. Vous pouvez modifier une autorisation de partage existante pour chaque adresse e-mail si vous avez un accès propriétaire à l’espace de travail.

1. Dans la section « Paramètres de partage » de chaque e-mail, un menu déroulant montre le niveau d’autorisation actuel.

2. Cliquez sur le menu déroulant et sélectionnez le nouveau niveau d’autorisation que vous souhaitez affecter à cette adresse e-mail.

    ![Paramètres d’autorisation de partage](media/how-to/sharing-permission-settings.png)

## <a name="pin-your-workspace"></a>Épingler votre espace de travail

Le premier espace de travail que vous avez créé est épinglé par défaut. Chaque fois que vous vous connectez, votre espace de travail épinglé s’affiche lors du chargement du site. Si vous avez créé de nombreux espaces de travail et que vous souhaitez faire de l’un d’entre eux votre espace de travail par défaut lorsque vous vous connectez, vous devez l’épingler.

1. Dans la barre latérale, cliquez sur le nom de l’espace de travail que vous souhaitez épingler.
2. Accédez à la page « Paramètres » de votre espace de travail.
3. Cliquez sur l’icône d’épingle.

    ![Épingler l’espace de travail](media/how-to/how-to-pin-workspace.png)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer votre espace de travail et vos projets](workspace-and-project.md).