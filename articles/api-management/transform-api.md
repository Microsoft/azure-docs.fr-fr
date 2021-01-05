---
title: Tutoriel - Transformer et protéger votre API dans Gestion des API Azure | Microsoft Docs
description: Dans ce tutoriel, vous découvrez comment protéger votre API dans Gestion des API avec des stratégies de transformation et de limitation (limitation du débité).
author: vladvino
ms.service: api-management
ms.custom: mvc
ms.topic: tutorial
ms.date: 09/28/2020
ms.author: apimpm
ms.openlocfilehash: 979bdaa1e0dac4f45a321abda2a208f46983f9cd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010229"
---
# <a name="tutorial-transform-and-protect-your-api"></a>Tutoriel : Transformer et protéger votre API

Le tutoriel vous montre comment transformer votre API de façon à ce qu’elle ne révèle pas d’informations sur le back-end privé. Par exemple, vous voulez masquer les informations relatives à la pile de technologies qui s’exécutent sur le back-end. Vous pouvez aussi souhaiter masquer les URL d’origine qui apparaissent dans le corps de la réponse HTTP de l’API et les rediriger au lieu de cela vers la passerelle de Gestion des API.

Ce tutoriel vous montre aussi combien il est facile d’ajouter une protection à votre API back-end en configurant une limite de débit avec Gestion des API Azure. Par exemple, vous pouvez limiter le débit des appels de l’API pour que l’API ne soit pas surutilisée par les développeurs. Pour plus d’informations, consultez [Stratégies de Gestion des API](api-management-policies.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> -   Transformer une API pour supprimer des en-têtes de réponse
> -   Remplacer les URL d’origine dans le corps de la réponse de l’API par les URL de la passerelle APIM
> -   Protéger une API en ajoutant une stratégie de limite de débit (limitation)
> -   Tester les transformations

:::image type="content" source="media/transform-api/api-management-management-console.png" alt-text="Stratégies dans le portail":::

## <a name="prerequisites"></a>Prérequis

-   Apprenez la [terminologie relative à Gestion des API Azure](api-management-terminology.md).
-   Comprendre le [concept des stratégies dans Gestion des API Azure](api-management-howto-policies.md).
-   Suivez ce guide de démarrage rapide : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).
-   Effectuez également toutes les étapes du tutoriel suivant : [Importer et publier votre première API](import-and-publish.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="transform-an-api-to-strip-response-headers"></a>Transformer une API pour supprimer des en-têtes de réponse

Cette section montre comment masquer les en-têtes HTTP que vous ne voulez pas montrer à vos utilisateurs. Cet exemple montre comment supprimer les en-têtes suivants dans la réponse HTTP :

-   **X-Powered-By**
-   **X-AspNet-Version**

### <a name="test-the-original-response"></a>Tester la réponse d’origine

Pour consulter la réponse d’origine :

1. Dans votre instance du service Gestion des API, sélectionnez **API**.
1. Sélectionnez **Demo Conference API** dans votre liste d’API.
1. Sélectionnez l’onglet **Test**, en haut de l’écran.
1. Sélectionnez l’opération **GetSpeakers**, puis sélectionnez **Envoyer**.

La réponse d’origine doit se présenter comme suit :

:::image type="content" source="media/transform-api/original-response.png" alt-text="Réponse de l’API d’origine":::

Comme vous pouvez le voir, la réponse comprend les en-têtes **X-AspNet-Version** et **X-Powered-By**.

### <a name="set-the-transformation-policy"></a>Définir la stratégie de transformation

1. Sélectionnez **Demo Conference API** > **Conception** > **Toutes les opérations**.
4. Dans la section **Traitement sortant**, sélectionnez l’icône Éditeur de code ( **</>** ).

   :::image type="content" source="media/transform-api/04-ProtectYourAPI-01-SetPolicy-Outbound.png" alt-text="Accéder à la stratégie sortante" border="false":::

1. Placez le curseur à l’intérieur de l’élément **&lt;sortant&gt;** , puis sélectionnez **Afficher les extraits de code** dans le coin supérieur droit.
1. Dans la fenêtre de droite, sous **Stratégies de transformation**, sélectionnez deux fois **Définir l’en-tête HTTP** (pour insérer deux extraits de code de stratégie).

   :::image type="content" source="media/transform-api/transform-api.png" alt-text="Définir la stratégie d’en-tête HTTP":::

1. Modifiez votre code **\<outbound>** afin qu’il ressemble à ceci :

   ```
   <set-header name="X-Powered-By" exists-action="delete" />
   <set-header name="X-AspNet-Version" exists-action="delete" />
   ```

   :::image type="content" source="media/transform-api/set-policy.png" alt-text="Définir l’en-tête HTTP":::

1. Sélectionnez **Enregistrer**.

## <a name="replace-original-urls-in-the-body-of-the-api-response-with-apim-gateway-urls"></a>Remplacer les URL d’origine dans le corps de la réponse de l’API par les URL de la passerelle APIM

Cette section montre comment protéger vos URL d’origine qui apparaissent dans le corps de la réponse HTTP de l’API, en les redirigeant au lieu de cela vers la passerelle Gestion des API.

### <a name="test-the-original-response"></a>Tester la réponse d’origine

Pour consulter la réponse d’origine :

1. Sélectionnez **Demo Conference API** > **Tester**.
1. Sélectionnez l’opération **GetSpeakers**, puis sélectionnez **Envoyer**.

    Comme vous pouvez le voir, la réponse comprend les URL de back-end d’origine :

    :::image type="content" source="media/transform-api/original-response2.png" alt-text="URL d’origine dans la réponse":::


### <a name="set-the-transformation-policy"></a>Définir la stratégie de transformation

1.  Sélectionnez **Demo Conference API** > **Toutes les opérations** > **Conception**.
1.  Dans la section **Traitement sortant**, sélectionnez l’icône Éditeur de code ( **</>** ).
1.  Placez le curseur à l’intérieur de l’élément **&lt;sortant&gt;** , puis sélectionnez **Afficher les extraits de code** dans le coin supérieur droit.
1.  Dans la fenêtre de droite, sous **Stratégies de transformation**, sélectionnez **Masquer les URL dans le contenu**. 
1.  Sélectionnez **Enregistrer**.

## <a name="protect-an-api-by-adding-rate-limit-policy-throttling"></a>Protéger une API en ajoutant une stratégie de limite de débit (limitation)

Cette section vous montre comment ajouter une protection pour votre API principale, en configurant les limites de débit. Par exemple, vous pouvez limiter le débit des appels de l’API pour que l’API ne soit pas surutilisée par les développeurs. Dans cet exemple, la limite est fixée à 3 appels par intervalle de 15 secondes, pour chaque ID d’abonnement. Après 15 secondes, un développeur peut de nouveau tenter d’appeler l’API.

1.  Sélectionnez **Demo Conference API** > **Toutes les opérations** > **Conception**.
1.  Dans la section **Traitement entrant**, sélectionnez l’icône Éditeur de code ( **</>** ).
1.  Placez le curseur à l’intérieur de l’élément **&lt;entrant&gt;** , puis sélectionnez **Afficher les extraits de code** dans le coin supérieur droit.

    :::image type="content" source="media/transform-api/04-ProtectYourAPI-01-SetPolicy-Inbound.png" alt-text="Définir une stratégie entrante" border="false":::

1.  Dans la fenêtre de droite, sous **Stratégies de restriction des accès**, cliquez sur **+ Limite le débit des appels par clé**.
1.  Remplacez votre code **rate-limit-by-key** (dans l’élément **\<inbound\>** ) par le code suivant :

    ```
    <rate-limit-by-key calls="3" renewal-period="15" counter-key="@(context.Subscription.Id)" />
    ```

## <a name="test-the-transformations"></a>Tester les transformations

À ce stade, si vous examinez le code dans l’éditeur de code, vos stratégies se présentent comme ceci :

   ```
   <policies>
      <inbound>
        <rate-limit-by-key calls="3" renewal-period="15" counter-key="@(context.Subscription.Id)" />
        <base />
      </inbound>
      <backend>
        <base />
      </backend>
      <outbound>
        <set-header name="X-Powered-By" exists-action="delete" />
        <set-header name="X-AspNet-Version" exists-action="delete" />
        <redirect-content-urls />
        <base />
      </outbound>
      <on-error>
        <base />
      </on-error>
   </policies>
   ```

Le reste de cette section est dédié au test des transformations de stratégies définies dans cet article.

### <a name="test-the-stripped-response-headers"></a>Tester les en-têtes de réponse supprimés

1. Sélectionnez **Demo Conference API** > **Tester**.
1. Sélectionnez l’opération **GetSpeakers**, puis sélectionnez **Envoyer**.

    Comme vous pouvez le voir, les en-têtes ont été retirés :

    :::image type="content" source="media/transform-api/final-response1.png" alt-text="En-têtes de réponse retirés":::

### <a name="test-the-replaced-url"></a>Tester l’URL remplacé

1. Sélectionnez **Demo Conference API** > **Tester**.
1. Sélectionnez l’opération **GetSpeakers**, puis sélectionnez **Envoyer**.

    Comme vous pouvez le constater, l’URL a été remplacée.

    :::image type="content" source="media/transform-api/final-response2.png" alt-text="URL remplacée":::

### <a name="test-the-rate-limit-throttling"></a>Tester la limite de débit (limitation)

1. Sélectionnez **Demo Conference API** > **Tester**.
1. Sélectionnez l’opération **GetSpeakers**. Sélectionnez **Envoyer** trois fois dans une ligne.

    Après avoir envoyé la requête 3 fois, vous recevez une réponse **429 Trop de requêtes**.

    :::image type="content" source="media/transform-api/test-throttling.png" alt-text="Trop de requêtes":::

1. Attendez environ 15 secondes, puis resélectionnez **Envoyer**. Cette fois, vous devriez obtenir une réponse **200 OK**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
>
> -   Transformer une API pour supprimer des en-têtes de réponse
> -   Remplacer les URL d’origine dans le corps de la réponse de l’API par les URL de la passerelle APIM
> -   Protéger une API en ajoutant une stratégie de limite de débit (limitation)
> -   Tester les transformations

Passez au tutoriel suivant :

> [!div class="nextstepaction"]
> [Surveiller votre API](api-management-howto-use-azure-monitor.md)
