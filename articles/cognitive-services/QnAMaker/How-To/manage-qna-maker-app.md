---
title: Gérer une application QnA Maker - QnA Maker
description: QnA Maker permet aux utilisateurs de collaborer sur une base de connaissances. QnA Maker vous donne la possibilité d'améliorer la qualité de votre base de connaissances grâce à l'apprentissage actif. Examinez, acceptez ou refusez, ajoutez sans supprimer ou modifier les questions existantes.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: 9c042d044f5ceba5a64d6bd7dfefa34bbc69b107
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353202"
---
# <a name="manage-qna-maker-app"></a>Gérer une application QnA Maker

QnA Maker vous permet de collaborer avec différents auteurs et éditeurs de contenu en vous donnant la possibilité de limiter l'accès des collaborateurs en fonction de leur rôle.
Découvrez-en plus sur les [concepts d'authentification des collaborateurs QnA Maker](../Concepts/role-based-access-control.md).

Vous pouvez également améliorer la qualité de votre base de connaissances en suggérant des questions alternatives grâce à l'[apprentissage actif](../Concepts/active-learning-suggestions.md). Les envois des utilisateurs sont pris en compte et apparaissent sous forme de suggestions dans la liste des questions alternatives. Vous avez la possibilité d’ajouter ces suggestions en tant que questions alternatives ou de les rejeter.

Votre base de connaissances ne change pas automatiquement. Vous devez accepter les suggestions afin que toute modification prenne effet. Ces suggestions ajoutent des questions, mais elles ne modifient pas les questions existantes pas plus qu’elles ne les suppriment.

## <a name="add-azure-role-based-access-control-azure-rbac"></a>Ajouter le Contrôle d'accès en fonction du rôle Azure (Azure RBAC)

QnA Maker permet aux utilisateurs de collaborer sur toutes les bases de connaissances dans la même ressource QnA Maker. Cette fonctionnalité est fournie avec le [Contrôle d'accès en fonction du rôle Azure (Azure RBAC)](../../../role-based-access-control/role-assignments-portal.md).

## <a name="access-at-the-qna-maker-resource-level"></a>Accès au niveau de la ressource QnA Maker

Vous ne pouvez pas partager une base de connaissances particulière dans un service QnA Maker. Si vous souhaitez que le contrôle d’accès soit plus granulaire, envisagez de répartir vos bases de connaissances sur différentes ressources QnA Maker, puis ajoutez des rôles à chaque ressource.

## <a name="add-a-role-to-a-resource"></a>Ajouter un rôle à une ressource

### <a name="add-a-user-account-to-the-qna-maker-resource"></a>Ajouter un compte d’utilisateur à la ressource QnA Maker

Les étapes suivantes utilisent le rôle de collaborateur, cependant tous les [rôles](../reference-role-based-access-control.md) peuvent être ajoutés à l’aide de ces étapes

1. Connectez-vous au portail [Microsoft Azure](https://portal.azure.com/) et accédez à votre ressource QnA Maker.

    ![Liste des ressources QnA Maker](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-resource-list.png)

1. Accédez à l’onglet **Contrôle d’accès (IAM)** .

    ![IAM pour QnA Maker](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-iam.png)

1. Sélectionnez **Ajouter**.

    ![Ajoutez IAM à QnA Maker](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-iam-add.png)

1. Sélectionnez un rôle dans la liste suivante :

    |Role|
    |--|
    |Propriétaire|
    |Contributeur|
    |Lecteur QnA Maker Cognitive Services|
    |Éditeur QnA Maker Cognitive Services|
    |Utilisateur Cognitive Services|

    :::image type="content" source="../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-add-role-iam.png" alt-text="IAM pour QnA Maker, ajouter un rôle.":::

1. Saisissez l’adresse e-mail de l’utilisateur et cliquez sur **Enregistrer**.

    ![Ajoutez une adresse e-mail IAM à QnA Maker](../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-iam-add-email.png)

### <a name="view-qna-maker-knowledge-bases"></a>Afficher les bases de connaissances QnA Maker

Lorsque la personne avec laquelle vous avez partagé votre service QnA Maker se connecte au [portail QnA Maker](https://qnamaker.ai), elle peut voir toutes les bases de connaissances de ce service en fonction de leur rôle.

Lorsqu’elle sélectionne une base de connaissances, son rôle actuel sur cette ressource QnA Maker est visible à côté du nom de la base de connaissances.

:::image type="content" source="../media/qnamaker-how-to-collaborate-knowledge-base/qnamaker-knowledge-base-role-name.png" alt-text="Capture d’écran de la base de connaissances en mode d’édition avec un nom de rôle entre parenthèses à côté du nom de la base de connaissances dans l’angle supérieur gauche de la page web.":::

## <a name="upgrade-runtime-version-to-use-active-learning"></a>Mettre à niveau votre version de runtime pour utiliser l’apprentissage actif

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

L’apprentissage actif est pris en charge dans la version 4.4.0 du runtime et dans les versions ultérieures. Si votre base de connaissances a été créée dans une version antérieure, [mettez à niveau votre runtime](set-up-qnamaker-service-azure.md#get-the-latest-runtime-updates) pour utiliser cette fonctionnalité.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Dans QnA Maker managé (préversion), étant donné que le runtime est hébergé par le service QnA Maker lui-même, il n’est pas nécessaire de mettre à niveau manuellement le runtime.

---

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