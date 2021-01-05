---
title: Stratégies dans Gestion des API Azure | Microsoft Docs
description: Apprenez à créer, à modifier et à configurer des stratégies dans Gestion des API. Consultez des exemples de code et affichez des ressources disponibles supplémentaires.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/29/2017
ms.author: apimpm
ms.openlocfilehash: 37ac6369790ed526fd923819558863ae84432aed
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358114"
---
# <a name="policies-in-azure-api-management"></a>Stratégies dans Gestion des API Azure

Dans le service Gestion des API Azure (APIM), les stratégies sont une fonctionnalité puissante du système qui permet à l’éditeur de changer le comportement de l’API grâce à la configuration. Les stratégies sont un ensemble d'instructions qui sont exécutées dans l'ordre sur demande ou sur réponse d'une API. Les instructions les plus utilisées comprennent la conversion du format XML au format JSON et la limitation du débit d'appels pour restreindre la quantité d'appels entrants d'un développeur. De nombreuses autres stratégies sont disponibles.

Les stratégies sont appliquées au niveau de la passerelle qui se trouve entre le consommateur de l’API et l’API managée. La passerelle reçoit toutes les demandes et les transfère normalement sans les modifier à l’API sous-jacente. Cependant, une stratégie peut appliquer des modifications à la demande entrante et à la réponse sortante.

Les expressions de stratégie peuvent être utilisées comme valeurs d’attribut ou valeurs de texte dans l’une des stratégies de Gestion des API, sauf si la stratégie le spécifie autrement. Certaines stratégies, telles que les stratégies [Control flow][Control flow] et [Set variable][Set variable], sont basées sur des expressions de stratégie. Pour plus d’informations, consultez les rubriques [Stratégies avancées][Advanced policies] et [Expressions de stratégie][Policy expressions].

## <a name="understanding-policy-configuration"></a><a name="sections"> </a>Configuration de la stratégie

La définition de la stratégie est un simple document XML qui décrit une séquence d'instructions entrantes et sortantes. Le code XML peut être modifié directement dans la fenêtre de définition. Une liste d’instructions est fournie à droite. Les instructions applicables à la portée actuelle sont activées et mises en surbrillance.

Lorsque vous cliquez sur une instruction active, le code XML correspondant est inséré à l'emplacement du curseur dans la fenêtre de définition. 

> [!NOTE]
> Si la stratégie que vous souhaitez ajouter n’est pas activée, vérifiez que vous êtes dans l’étendue correcte pour cette stratégie. Chaque instruction de stratégie est conçue pour une utilisation dans certaines étendues et sections de la stratégie. Pour consulter les sections de la stratégie et les étendues pour une stratégie, consultez la section **Utilisation** de cette stratégie dans la [Référence de la stratégie][Policy Reference].
> 
> 

La configuration est composée des sections `inbound`, `backend`, `outbound` et `on-error`. La série d’instructions de stratégie spécifiées s’exécute dans l’ordre pour une demande et une réponse.

```xml
<policies>
  <inbound>
    <!-- statements to be applied to the request go here -->
  </inbound>
  <backend>
    <!-- statements to be applied before the request is forwarded to 
         the backend service go here -->
  </backend>
  <outbound>
    <!-- statements to be applied to the response go here -->
  </outbound>
  <on-error>
    <!-- statements to be applied if there is an error condition go here -->
  </on-error>
</policies> 
```

S'il existe une erreur lors du traitement d'une demande, les autres étapes des sections `inbound`, `backend` ou `outbound` sont ignorées et l'exécution passe aux instructions de la section `on-error`. En plaçant des instructions de stratégie dans la section `on-error`, vous pouvez consulter l'erreur à l'aide de la propriété `context.LastError`, inspecter et personnaliser la réponse à l'erreur à l'aide de la stratégie `set-body`, puis configurer ce qui se passe si une erreur se produit. Il existe des codes d'erreur pour les étapes intégrées et pour les erreurs qui peuvent se produire pendant le traitement d'instructions de stratégie. Pour plus d'informations, consultez [Gestion des erreurs dans les stratégies de gestion des API](./api-management-error-handling-policies.md).

## <a name="how-to-configure-policies"></a><a name="scopes"> </a>Configuration des stratégies

Pour plus d’informations sur la façon de configurer des stratégies, consultez [Définir ou modifier des stratégies](set-edit-policies.md).

## <a name="policy-reference"></a>Référence de stratégie

Pour obtenir la liste complète des instructions et des paramètres de stratégie, consultez la section [Référence de stratégie](./api-management-policies.md).

## <a name="policy-samples"></a>Exemples de stratégie

Consultez [Exemples de stratégie](./policy-reference.md) pour obtenir plus d’exemples de code.

## <a name="examples"></a>Exemples

### <a name="apply-policies-specified-at-different-scopes"></a>Appliquer des stratégies spécifiées à différentes portées

Si vous avez une stratégie configurée au niveau global et une stratégie configurée pour une API, dès que cette API est utilisée, les deux stratégies sont appliquées. Le service Gestion des API permet de trier de façon déterminée les instructions de stratégie combinées via l'élément de base. 

```xml
<policies>
    <inbound>
        <cross-domain />
        <base />
        <find-and-replace from="xyz" to="abc" />
    </inbound>
</policies>
```

Dans l'exemple de définition de stratégie ci-dessus, l'instruction `cross-domain` s'exécute avant toutes les autres stratégies de niveau supérieur, qui sont à leur tour suivies de la stratégie `find-and-replace`. 

### <a name="restrict-incoming-requests"></a>Limiter les demandes entrantes

Pour ajouter une nouvelle instruction afin de limiter les demandes entrantes à certaines adresses IP, placez le curseur juste à l’intérieur du contenu de l’élément XML `inbound`, puis cliquez sur l’instruction **Restrict caller IPs** (Limiter les adresses IP d’appelant).

![Restriction policies][policies-restrict]

Ceci ajoute un code XML à l'élément `inbound` , indiquant comment configurer l'instruction.

```xml
<ip-filter action="allow | forbid">
    <address>address</address>
    <address-range from="address" to="address"/>
</ip-filter>
```

Pour limiter les demandes entrantes et n'accepter que celles venant de l'adresse IP 1.2.3.4, modifiez le code XML comme suit :

```xml
<ip-filter action="allow">
    <address>1.2.3.4</address>
</ip-filter>
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation de stratégies, consultez les pages :

+ [Transform and protect your API](transform-api.md) (Transformer et protéger votre API)
+ [Référence de stratégie](./api-management-policies.md) pour obtenir la liste complète des instructions et des paramètres de stratégie
+ [Exemples de stratégie](./policy-reference.md)   

[Policy Reference]: ./api-management-policies.md
[Product]: api-management-howto-add-products.md
[API]: api-management-howto-add-products.md
[Operation]: ./mock-api-responses.md

[Advanced policies]: ./api-management-advanced-policies.md
[Control flow]: ./api-management-advanced-policies.md#choose
[Set variable]: ./api-management-advanced-policies.md#set-variable
[Policy expressions]: ./api-management-policy-expressions.md

[policies-restrict]: ./media/api-management-howto-policies/api-management-policies-restrict.png
