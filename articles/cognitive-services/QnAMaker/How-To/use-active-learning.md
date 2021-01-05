---
title: Utiliser l’apprentissage actif avec la base de connaissances – QnA Maker
description: Apprenez à améliorer la qualité de votre base de connaissances avec l'apprentissage actif. Examinez, acceptez ou refusez, ajoutez sans supprimer ou modifier les questions existantes.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 03/18/2020
ms.openlocfilehash: 1ac5e32b454cfc1adafb1f54b01d2a1a302908a4
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96600433"
---
# <a name="use-active-learning-to-improve-your-knowledge-base"></a>Utiliser l’apprentissage actif pour améliorer votre base de connaissances

L’[apprentissage actif](../Concepts/active-learning-suggestions.md) vous permet d’améliorer la qualité de votre base de connaissances en suggérant des questions alternatives. Les envois des utilisateurs sont pris en compte et s’affichent sous forme de suggestions dans la liste des questions alternatives. Vous avez la possibilité d’ajouter ces suggestions en tant que questions alternatives ou de les rejeter.

Votre base de connaissances ne change pas automatiquement. Vous devez accepter les suggestions afin que toute modification prenne effet. Ces suggestions ajoutent des questions, mais elles ne modifient pas les questions existantes pas plus qu’elles ne les suppriment.


## <a name="upgrade-runtime-version-to-use-active-learning"></a>Mettre à niveau votre version de runtime pour utiliser l’apprentissage actif

L’apprentissage actif est pris en charge dans la version 4.4.0 du runtime et dans les versions ultérieures. Si votre base de connaissances a été créée dans une version antérieure, [mettez à niveau votre runtime](set-up-qnamaker-service-azure.md#get-the-latest-runtime-updates) pour utiliser cette fonctionnalité.

## <a name="turn-on-active-learning-for-alternate-questions"></a>Activer l’apprentissage actif pour les questions alternatives

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

L’apprentissage actif est désactivé par défaut. Activez-le pour afficher les suggestions de questions. Une fois l’apprentissage actif activé, vous devez envoyer des informations de l’application cliente vers QnA Maker. Pour plus d’informations, consultez [Flux architectural pour utiliser les API Train et GenerateAnswer à partir d’un bot](improve-knowledge-base.md#architectural-flow-for-using-generateanswer-and-train-apis-from-a-bot).

1. Sélectionnez **Publier** pour publier la base de connaissances. Les requêtes d’apprentissage actif sont collectées à partir du point de terminaison de prédiction de l’API GenerateAnswer uniquement. Les requêtes vers le volet Test du portail QnA Maker n’affectent pas l’apprentissage actif.

1. Pour activer l’apprentissage actif, cliquez sur votre **Nom**, puis accédez à [**Paramètres du service**](https://www.qnamaker.ai/UserSettings) dans le portail QnA Maker, dans l’angle supérieur droit.

    ![Activez les alternatives de question suggérées de l’apprentissage actif à partir de la page Paramètres du service. Sélectionnez votre nom d’utilisateur dans le menu en haut à droite, puis sélectionnez Paramètres du service.](../media/improve-knowledge-base/Endpoint-Keys.png)


1. Recherchez le service QnA Maker, puis activez la stratégie **Apprentissage actif**.

    > [!div class="mx-imgBorder"]
    > [![Activation de la fonctionnalité d’apprentissage actif sur la page Paramètres du service. Si vous n’êtes pas en mesure d’activer/de désactiver la fonctionnalité, vous devrez peut-être mettre à niveau votre service.](../media/improve-knowledge-base/turn-active-learning-on-at-service-setting.png)](../media/improve-knowledge-base/turn-active-learning-on-at-service-setting.png#lightbox)

    > [!Note]
    > La version exacte sur l’image précédente est présentée à titre d’exemple uniquement. Votre version peut être différente.

    Une fois que la fonctionnalité **Apprentissage actif** est activée, la base de connaissances suggère de nouvelles questions à intervalles réguliers en fonction des questions soumises par l’utilisateur. Vous pouvez désactiver la stratégie **Apprentissage actif** en basculant de nouveau le paramètre.
    
# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Par défaut, l’apprentissage actif est **activé** dans QnA Maker managé (préversion). Pour voir les autres questions suggérées, [utilisez les options Affichage](../How-To/improve-knowledge-base.md#view-suggested-questions) sur la page Modifier.

---

## <a name="review-suggested-alternate-questions"></a>Passer en revue les questions alternatives suggérées

[Passez en revue les questions alternatives suggérées](improve-knowledge-base.md) dans la page **Edition** de chaque base de connaissances.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une base de connaissances](./manage-knowledge-bases.md)
