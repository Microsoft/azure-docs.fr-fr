---
title: Faire des demandes avec Postman
titleSuffix: Azure Digital Twins
description: Découvrez comment configurer et utiliser Postman pour tester les API Azure Digital Twins.
ms.author: baanders
author: baanders
ms.service: digital-twins
services: digital-twins
ms.topic: how-to
ms.date: 11/10/2020
ms.openlocfilehash: fe79797aeef2b0c148aece0319d6b340b663b184
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763721"
---
# <a name="how-to-use-postman-to-send-requests-to-the-azure-digital-twins-apis"></a>Comment utiliser Postman pour envoyer des demandes aux API Azure Digital Twins

[Postman](https://www.getpostman.com/) est un outil de test REST qui fournit des fonctionnalités clés de requête HTTP dans une interface utilisateur graphique Bureau basée sur un plug-in. Vous pouvez l’utiliser pour créer des requêtes HTTP et envoyer celles-ci aux [API REST Azure Digital Twins](how-to-use-apis-sdks.md).

Cet article décrit comment configurer le [client REST Postman](https://www.getpostman.com/) pour interagir avec les API Azure Digital Twins en procédant comme suit :

1. Utilisez l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true) pour obtenir un jeton du porteur que vous utiliserez pour effectuer des demandes API dans Postman.
1. Créez une collection Postman et configurez le client REST Postman afin qu’il utilise votre jeton du porteur pour s’authentifier.
1. Utilisez le client Postman configuré pour créer et envoyer une demande aux API Azure Digital Twins.

## <a name="prerequisites"></a>Prérequis

Pour utiliser Postman pour accéder aux API Azure Digital Twins, vous devez configurer une instance Azure Digital Twins et télécharger Postman. Le reste de cette section vous guidera dans ces étapes.

### <a name="set-up-azure-digital-twins-instance"></a>Configurer l’instance Azure Digital Twins

[!INCLUDE [digital-twins-prereq-instance.md](../../includes/digital-twins-prereq-instance.md)]

### <a name="download-postman"></a>Téléchargez Postman.

Téléchargez ensuite la version de bureau du client Postman. Accédez à [*www.getpostman.com/apps*](https://www.getpostman.com/apps) et suivez les invites pour télécharger l’application.

## <a name="get-bearer-token"></a>Obtenir un jeton du porteur

À présent que vous avez configuré Postman et votre instance Azure Digital Twins, vous devez obtenir un jeton du porteur que les demandes Postman peuvent utiliser pour autoriser les API Azure Digital Twins.

Il existe plusieurs façons d’obtenir ce jeton. Dans le cadre de cet article, vous allez utiliser l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true) pour vous connecter à votre compte Azure et obtenir un jeton de cette façon.

Si vous disposez d’une interface de ligne de commande Azure [installée localement](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true), vous pouvez démarrer une invite de commandes sur votre ordinateur pour exécuter les commandes suivantes.
Dans le cas contraire, vous pouvez ouvrir une fenêtre [Azure Cloud Shell](https://shell.azure.com) dans votre navigateur et y exécuter les commandes.

1. Tout d’abord, assurez-vous que vous êtes connecté à Azure avec les informations d’identification appropriées, en exécutant la commande suivante :

    ```azurecli-interactive
    az login
    ```

1. Ensuite, utilisez la commande [az account get-access-token](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest&preserve-view=true#az_account_get_access_token) pour obtenir un jeton du porteur ayant accès au service Azure Digital Twins.

    ```azurecli-interactive
    az account get-access-token --resource 0b07f429-9f4b-4714-9392-cc5e8e80c8b0
    ```

1. Copiez la valeur de `accessToken` dans le résultat, puis enregistrez-la pour l’utiliser dans la section suivante. Il s’agit de la **valeur de jeton** que vous allez fournir à Postman pour authentifier vos demandes.

    :::image type="content" source="media/how-to-use-postman/console-access-token.png" alt-text="Affichage d’une fenêtre de console locale présentant le résultat de la commande az account get-access-token. L’un des champs du résultat est appelé accessToken, et son exemple de valeur (commençant par ey) est en surbrillance.":::

>[!TIP]
>Ce jeton est valide pendant au minimum cinq minutes et au maximum 60 minutes. Si le temps alloué ne suffit pas pour le jeton actuel, vous pouvez répéter les étapes décrites dans cette section pour en obtenir un nouveau.

## <a name="set-up-postman-collection-and-authorization"></a>Configurer une collection et une autorisation Postman

Ensuite, configurez Postman pour effectuer des demandes API.
Ces étapes ayant lieu dans votre application Postman locale, ouvrez celle-ci sur votre ordinateur.

### <a name="create-a-postman-collection"></a>Créer une collection Postman

Les demandes dans Postman sont enregistrées dans des **collections** (groupes de demandes). Lorsque vous créez une collection pour regrouper vos demandes, vous pouvez appliquer des paramètres communs à de nombreuses demandes à la fois. Cela peut sensiblement simplifier l’autorisation si vous envisagez de créer plusieurs demandes aux API Azure Digital Twins, car vous ne devez configurer l’authentification qu’une seule fois pour l’ensemble de la collection.

1. Pour créer une collection, appuyez sur le bouton *+ Nouvelle collection*.

    :::image type="content" source="media/how-to-use-postman/postman-new-collection.png" alt-text="Affichage d’une fenêtre de Postman venant de s’ouvrir. Le bouton « Nouveau collection » est en surbrillance":::

1. Dans la fenêtre *CRÉER UNE COLLECTION* qui suit, entrez un **Nom** et une **Description** facultative pour votre collection.

Ensuite, passez à la section suivante pour ajouter un jeton du porteur à la collection à des fins d’autorisation.

### <a name="add-authorization-token-and-finish-collection"></a>Ajouter un jeton d’autorisation et terminer la collection

1. Dans la boîte de dialogue *CRÉER UNE COLLECTION*, accédez à l’onglet *Autorisation*. C’est là que vous allez placer la **valeur de jeton** que vous avez recueillie dans la section [Obtenir un jeton du porteur](#get-bearer-token) afin de l’utiliser pour toutes les demandes API dans votre collection.

    :::image type="content" source="media/how-to-use-postman/postman-authorization.png" alt-text="Fenêtre de Postman « CRÉER UNE COLLECTION », affichant l’onglet « Autorisation ».":::

1. Définissez le *Type* sur _**OAuth 2.0**_, puis collez votre jeton d’accès dans la zone *Jeton d’accès*.

    :::image type="content" source="media/how-to-use-postman/postman-paste-token.png" alt-text="Fenêtre de Postman « CRÉER UNE COLLECTION », affichant l’onglet « Autorisation ». Le type « OAuth 2.0 » est sélectionné, et la zone Jeton d’accès dans laquelle la valeur du jeton d’accès peut être collée est en surbrillance.":::

1. Après avoir collé votre jeton du porteur, appuyez sur *Créer* pour terminer la création de votre collection.

Votre nouvelle collection est désormais visible dans l’affichage principal de Postman, sous *Collections*.

:::image type="content" source="media/how-to-use-postman/postman-post-collection.png" alt-text="Affichage de la fenêtre principale de Postman. Le collection que vous venez de créer est en surbrillance sous l’onglet « Collections ».":::

## <a name="create-a-request"></a>Créer une demande

Une fois les étapes précédentes terminées, vous pouvez créer des demandes aux API Azure Digital Twins.

1. Pour créer une demande, appuyez sur le bouton *+ Nouveau*.

    :::image type="content" source="media/how-to-use-postman/postman-new-request.png" alt-text="Affichage de la fenêtre principale de Postman. Le bouton « Nouveau » est en surbrillance":::

1. Choisissez *Demande*.

    :::image type="content" source="media/how-to-use-postman/postman-new-request-2.png" alt-text="Affichage des options que vous pouvez sélectionner pour créer quelque chose. L’option « Demande » est en surbrillance":::

1. Cette action ouvre la fenêtre *Enregistrer la demande* dans laquelle vous pouvez entrer un nom et une description facultative pour votre demande, ainsi que choisir la collection à laquelle elle appartient. Entrez les détails et enregistrez la demande dans la collection que vous avez créée précédemment.

    :::row:::
        :::column:::
            :::image type="content" source="media/how-to-use-postman/postman-save-request.png" alt-text="Affichage de la fenêtre « Enregistrer la demande » dans laquelle vous pouvez remplir les champs décrits. Le bouton « Enregistrer dans la collection Azure Digital Twins » est en surbrillance":::
        :::column-end:::
        :::column:::
        :::column-end:::
    :::row-end:::

Vous pouvez maintenant afficher votre demande dans la collection, et la sélectionner pour extraire ses détails modifiables.

:::image type="content" source="media/how-to-use-postman/postman-request-details.png" alt-text="Affichage de la fenêtre principale de Postman. La collection Azure Digital Twins est développée, et la demande « Query twins » (Interroger des jumeaux) est en surbrillance. Les détails de la demande s’affichent au centre de la page." lightbox="media/how-to-use-postman/postman-request-details.png":::

### <a name="set-request-details"></a>Définir les détails d’une demande

Pour envoyer une demande Postman à l’une des API Azure Digital Twins, vous avez besoin de l’URL de l’API et d’informations sur les détails requis. Vous trouverez ces informations dans la [documentation de référence sur l’API REST Azure Digital Twins](/rest/api/azure-digitaltwins/).

Pour continuer avec un exemple de requête, cet article utilise l’API de requête (et sa [documentation de référence](/rest/api/digital-twins/dataplane/query/querytwins)) pour interroger tous les jumeaux numériques dans une instance.

1. Trouvez l’URL et le type de la demande dans la documentation de référence. Pour l’API de requête, il s’agit actuellement de *POST `https://digitaltwins-hostname/query?api-version=2020-10-31`* .
1. Dans Postman, définissez le type de la demande, puis entrez son URL en remplissant les espaces réservés dans l’URL de manière appropriée. C’est là que vous allez utiliser le **nom d’hôte** de votre instance, évoqué dans la section [*Conditions préalables*](#prerequisites).
    
   :::image type="content" source="media/how-to-use-postman/postman-request-url.png" alt-text="Dans les détails de la nouvelle demande, l’URL de requête de la documentation de référence a été renseignée dans la zone URL de la demande." lightbox="media/how-to-use-postman/postman-request-url.png":::
    
1. Vérifiez que les paramètres indiqués pour la demande sous l’onglet *Params* correspondent à ceux décrits dans la documentation de référence. Pour cette demande dans Postman, le paramètre `api-version` a été automatiquement renseigné lors de l’entrée de l’URL de la demande à l’étape précédente. Pour l’API de requête, comme il s’agit du seul paramètre requis, cette étape est accomplie.
1. Sous l’onglet *Autorisation*, définissez le *Type* sur *Hériter du parent*. Cela indique que cette demande utilisera l’authentification que vous avez configurée précédemment pour l’ensemble de la collection.
1. Vérifiez que les en-têtes affichés pour la demande sous l’onglet *En-têtes* correspondent à ceux décrits dans la documentation de référence. Pour cette demande, plusieurs en-têtes ont été remplis automatiquement. Pour l’API de requête, aucune des options d’en-tête n’étant requise, cette étape est accomplie.
1. Vérifiez que le corps affiché pour la demande sous l’onglet *Corps* correspond à celui décrit dans la documentation de référence. Pour l’API de requête, un corps JSON est requis pour fournir le texte de la requête. Voici un exemple de corps pour cette demande qui interroge tous les jumeaux numériques dans l’instance :

   :::image type="content" source="media/how-to-use-postman/postman-request-body.png" alt-text="Dans les détails de la nouvelle demande, l’onglet Corps est affiché. Il contient un corps JSON brut avec une requête « SELECT * FROM DIGITALTWINS »."lightbox="media/how-to-use-postman/postman-request-body.png":::

   Pour plus d’informations sur la création de requêtes Azure Digital Twins, consultez [*Guide pratique : Interroger le graphique de jumeaux*](how-to-query-graph.md).

1. Consultez la documentation de référence pour connaître les autres champs susceptibles d’être requis pour votre type de demande. Pour l’API de requête, toutes les exigences ayant été satisfaites dans la demande Postman, cette étape est accomplie.
1. Utilisez le bouton *Envoyer* pour envoyer votre demande terminée.
   :::image type="content" source="media/how-to-use-postman/postman-request-send.png" alt-text="Près des détails de la nouvelle demande, le bouton Envoyer est en surbrillance." lightbox="media/how-to-use-postman/postman-request-send.png":::

Après l’envoi de la demande, les détails de la réponse s’affichent dans la fenêtre Postman sous la demande. Vous pouvez afficher le code d’état de la réponse et tout texte de corps.

:::image type="content" source="media/how-to-use-postman/postman-request-response.png" alt-text="Sous les détails de la demande envoyée, les détails de la réponse sont en surbrillance. Il y a un État 200 OK et un texte de corps décrivant les jumeaux numériques que la requête a retournés." lightbox="media/how-to-use-postman/postman-request-response.png":::

Vous pouvez également comparer la réponse aux données de réponse attendues dans la documentation de référence pour vérifier le résultat ou en savoir plus sur les erreurs qui se produisent.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les API Digital Twins, lisez [*Guide pratique : Utiliser les API et les kits de développement logiciel (SDK) Azure Digital Twins*](how-to-use-apis-sdks.md) ou consultez la [documentation de référence pour les API REST](/rest/api/azure-digitaltwins/).