---
title: Guide pratique pour utiliser des valeurs nommées dans les stratégies Gestion des API Azure
description: Découvrez comment utiliser des valeurs nommées dans les stratégies Gestion des API Azure. Les valeurs nommées peuvent contenir des chaînes littérales et des expressions de stratégie.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/08/2020
ms.author: apimpm
ms.openlocfilehash: 3f317276ae92e6121d519553b7883677dab89705
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87852189"
---
# <a name="how-to-use-named-values-in-azure-api-management-policies"></a>Guide pratique pour utiliser des valeurs nommées dans les stratégies Gestion des API Azure

Les stratégies Gestion des API sont une fonctionnalité puissante du système qui permet au portail Azure de modifier le comportement de l’API grâce à la configuration. Les stratégies sont un ensemble d'instructions qui sont exécutées dans l'ordre sur demande ou sur réponse d'une API. Les instructions de la stratégie peuvent être construites à l’aide de valeurs de texte littéral, d’expressions de stratégie et de valeurs nommées.

Chaque instance du service Gestion des API possède une collection de paires clé/valeur, appelées « valeurs nommées », qui s’appliquent de manière globale à l’instance du service. Il n’existe aucune limite imposée quant au nombre d’éléments que peut contenir une collection. Les valeurs nommées peuvent être utilisées pour gérer les valeurs de chaîne constantes dans l’ensemble des stratégies et de la configuration des API. Chaque valeur nommée peut avoir les attributs suivants :

| Attribut      | Type            | Description                                                                                                                            |
| -------------- | --------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `Display name` | string          | Utilisé pour référencer la valeur nommée dans les stratégies. Chaîne de 1 à 256 caractères. Seuls les lettres, les chiffres, les points et les tirets sont autorisés. |
| `Value`        | string          | Valeur réelle. Ne peut pas être vide ni se composer uniquement d’espaces blancs. Longueur maximale de 4 096 caractères.                                        |
| `Secret`       | boolean         | Détermine si la valeur est un secret et doit être chiffrée.                                                               |
| `Tags`         | tableau de chaînes | Utilisé pour filtrer la liste de valeurs nommées. Jusqu’à 32 étiquettes.                                                                                    |

![Valeurs nommées](./media/api-management-howto-properties/named-values.png)

Les valeurs nommées peuvent contenir des chaînes littérales et des [expressions de stratégie](./api-management-policy-expressions.md). Par exemple, la valeur de `Expression` est une expression de stratégie qui retourne une chaîne contenant la date et l’heure actuelles. La valeur nommée `Credential` est marquée en tant que secret. Par défaut, cette valeur n’est donc pas affichée.

| Nom       | Valeur                      | Secret | Balises          |
| ---------- | -------------------------- | ------ | ------------- |
| Valeur      | 42                         | False  | vital-numbers |
| Informations d'identification | ••••••••••••••••••••••     | True   | security      |
| Expression | @(DateHeure.Now.ToString()) | False  |               |

> [!NOTE]
> Au lieu de valeurs nommées stockées dans un service Gestion des API, vous pouvez utiliser les valeurs stockées dans le service [Azure Key Vault](https://azure.microsoft.com/services/key-vault/), comme illustré dans cet [exemple](https://github.com/Azure/api-management-policy-snippets/blob/master/examples/Look%20up%20Key%20Vault%20secret%20using%20Managed%20Service%20Identity.policy.xml).

## <a name="to-add-and-edit-a-named-value"></a>Pour ajouter et modifier une valeur nommée

![Ajouter une valeur nommée](./media/api-management-howto-properties/add-property.png)

1. Sélectionnez **API** sous **Gestion des API**.
2. Sélectionnez **Valeurs nommées**.
3. Appuyez sur **+ Ajouter**.

    Nom et Valeur doivent être renseignés. Si la valeur est un secret, cochez la case _Il s’agit d’une clé secrète_. Entrez une ou plusieurs balises facultatives pour aider à organiser vos valeurs nommées, puis cliquez sur Enregistrer.

4. Cliquez sur **Créer**.

Une fois la valeur nommée créée, vous pouvez la modifier en cliquant dessus. Si vous modifiez le nom de valeur nommée, toutes les stratégies qui font référence à cette valeur nommée sont automatiquement mises à jour pour utiliser le nouveau nom.

## <a name="to-delete-a-named-value"></a>Pour supprimer une valeur nommée

Pour supprimer une valeur nommée, cliquez sur **Supprimer** en regard de la valeur nommée à supprimer.

> [!IMPORTANT]
> Si la valeur nommée est référencée par des stratégies, vous ne pouvez pas la supprimer correctement tant que vous ne l’avez pas supprimée de toutes les stratégies qui l’utilisent.

## <a name="to-search-and-filter-named-values"></a>Pour rechercher et filtrer des valeurs nommées

L’onglet **Valeurs nommées** inclut des fonctionnalités de recherche et de filtrage pour vous aider à gérer vos valeurs nommées. Pour filtrer la liste de valeurs nommées par nom, entrez un terme à rechercher dans la zone de texte **Propriétés de recherche** . Pour afficher toutes les valeurs nommées, effacez la zone de texte **Propriétés de recherche**, puis appuyez sur Entrée.

Pour filtrer la liste par balise, entrez une ou plusieurs balises dans la zone de texte **Filtrer par balises** . Pour afficher toutes les valeurs nommées, effacez la zone de texte **Filtrer par balises**, puis appuyez sur Entrée.

## <a name="to-use-a-named-value"></a>Pour utiliser une valeur nommée

Pour utiliser une valeur nommée dans une stratégie, placez son nom dans une paire d’accolades telle que `{{ContosoHeader}}`, comme illustré dans l’exemple suivant :

```xml
<set-header name="{{ContosoHeader}}" exists-action="override">
  <value>{{ContosoHeaderValue}}</value>
</set-header>
```

Dans cet exemple, `ContosoHeader` est utilisé comme nom d’en-tête d’une stratégie `set-header`, et `ContosoHeaderValue` comme valeur de cet en-tête. Lorsque cette stratégie est évaluée lors d’une demande ou d’une réponse à la passerelle Gestion des API, `{{ContosoHeader}}` et `{{ContosoHeaderValue}}` sont remplacés par leurs valeurs respectives.

Les valeurs nommées peuvent être utilisées comme attribut complet ou valeurs d’élément, comme indiqué dans l’exemple précédent. Elles peuvent également être insérées dans ou combinés avec une partie d’une expression de texte littéral, comme illustré dans l’exemple suivant : `<set-header name = "CustomHeader{{ContosoHeader}}" ...>`

Les valeurs nommées peuvent également contenir des expressions de stratégie. Dans l’exemple suivant, `ExpressionProperty` est utilisé.

```xml
<set-header name="CustomHeader" exists-action="override">
    <value>{{ExpressionProperty}}</value>
</set-header>
```

Lorsque cette stratégie est évaluée, `{{ExpressionProperty}}` est remplacé par sa valeur : `@(DateTime.Now.ToString())`. Étant donné que la valeur est une expression de stratégie, l’expression est évaluée et la stratégie poursuit son exécution.

Vous pouvez tester cette opération dans le portail des développeurs en appelant une opération qui a une stratégie dont l’étendue inclut des valeurs nommées. Dans l’exemple suivant, une opération est appelée avec les deux stratégies `set-header` de l’exemple précédent incluant des valeurs nommées. Notez que la réponse contient deux en-têtes personnalisés configurés à l’aide de stratégies et de valeurs nommées.

![Portail des développeurs][api-management-send-results]

Si vous examinez le [suivi de l’inspecteur d’API](api-management-howto-api-inspector.md) pour un appel qui inclut les deux exemples de stratégies précédents incluant des valeurs nommées, vous pouvez voir les deux stratégies `set-header` avec les valeurs nommées insérées, ainsi que l’évaluation de l’expression de stratégie pour la valeur nommée contenant l’expression de stratégie.

![Suivi de l’inspecteur d’API][api-management-api-inspector-trace]

Alors que les valeurs nommées peuvent contenir des expressions de stratégie, elles ne peuvent pas contenir d’autres valeurs nommées. Si le texte contenant une référence de valeur nommée est utilisé pour une valeur, telle que `Text: {{MyProperty}}`, cette référence ne sera pas résolue et remplacée.

## <a name="next-steps"></a>Étapes suivantes

-   En savoir plus sur l’utilisation des stratégies
    -   [Stratégies dans Gestion des API](api-management-howto-policies.md)
    -   [Référence de stratégie](./api-management-policies.md)
    -   [Expressions de stratégie](./api-management-policy-expressions.md)

[api-management-send-results]: ./media/api-management-howto-properties/api-management-send-results.png
[api-management-properties-filter]: ./media/api-management-howto-properties/api-management-properties-filter.png
[api-management-api-inspector-trace]: ./media/api-management-howto-properties/api-management-api-inspector-trace.png
