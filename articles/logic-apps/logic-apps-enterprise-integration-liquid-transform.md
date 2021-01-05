---
title: Convertir du code JSON et XML avec des modèles Liquid
description: Transformez du code JSON et XML en utilisant des modèles Liquid en tant que mappages dans Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, daviburg, logicappspm
ms.topic: article
ms.date: 07/31/2020
ms.openlocfilehash: 0362c9ed4f736474dbd49e1bfaf1373e0f48acd6
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94992707"
---
# <a name="transform-json-and-xml-using-liquid-templates-as-maps-in-azure-logic-apps"></a>Transformer du code JSON et XML en utilisant des modèles Liquid en tant que mappages dans Azure Logic Apps

Lorsque vous voulez effectuer des transformations de code JSON de base dans vos applications logiques, vous pouvez utiliser des [opérations de données](../logic-apps/logic-apps-perform-data-operations.md) natives telles que **Compose** ou **Parse JSON**. Pour des transformations avancées et complexes de code JSON vers JSON, dotées d’éléments tels que des itérations, des flux de contrôle et des variables, créez et utilisez des modèles qui décrivent ces transformations à l’aide du langage de modèle open source [Liquid](https://shopify.github.io/liquid/). Vous pouvez également [effectuer d’autres transformations](#other-transformations), par exemple, de JSON en texte, de XML vers JSON et de XML en texte.

Avant de pouvoir effectuer une transformation Liquid dans votre application logique, vous devez d’abord créer un modèle Liquid qui définit le mappage souhaité. Ensuite, vous [chargez le modèle en tant que mappage](../logic-apps/logic-apps-enterprise-integration-maps.md) dans votre [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md). Une fois que vous avez ajouté l’action **Transformer de JSON en JSON - Liquid** à votre application logique, vous pouvez sélectionner le modèle Liquid comme mappage pour l’action à utiliser.

Cet article vous explique comment effectuer ces tâches :

* Créez un modèle Liquid.
* Ajoutez le modèle à votre compte d’intégration.
* Ajoutez l’action de transformation Liquid à votre application logique.
* Sélectionnez le modèle en tant que mappage que vous souhaitez utiliser.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* Un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)

* Des connaissances de base sur le [langage de gabarit Liquid](https://shopify.github.io/liquid/)

  > [!NOTE]
  > L’action **Transformer de JSON en JSON - Liquid** suit l’[implémentation DotLiquid pour Liquid](https://github.com/dotliquid/dotliquid), qui diffère dans des cas spécifiques de l’[implémentation Shopify pour Liquid](https://shopify.github.io/liquid). Pour plus d’informations, consultez [Considérations relatives aux modèles Liquid](#liquid-template-considerations).

## <a name="create-the-template"></a>Créer le modèle

1. Créez le modèle Liquid à utiliser comme mappage pour la transformation de code JSON. Vous pouvez utiliser l’outil d’édition de votre choix.

   Pour cet exemple, créez l’exemple de modèle Liquid décrit dans cette section :

   ```json
   {%- assign deviceList = content.devices | Split: ', ' -%}

   {
      "fullName": "{{content.firstName | Append: ' ' | Append: content.lastName}}",
      "firstNameUpperCase": "{{content.firstName | Upcase}}",
      "phoneAreaCode": "{{content.phone | Slice: 1, 3}}",
      "devices" : [
         {%- for device in deviceList -%}
            {%- if forloop.Last == true -%}
            "{{device}}"
            {%- else -%}
            "{{device}}",
            {%- endif -%}
         {%- endfor -%}
      ]
   }
   ```

1. Enregistrez le modèle à l’aide de l’extension `.liquid`. Cet exemple utilise `SimpleJsonToJsonTemplate.liquid`.

## <a name="upload-the-template"></a>Charger le modèle

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec les informations d’identification de votre compte Azure.

1. Dans la zone de recherche du portail Azure, entrez `integration accounts` et sélectionnez **Comptes d’intégration**.

   ![Rechercher « Comptes d’intégration »](./media/logic-apps-enterprise-integration-liquid-transform/find-integration-accounts.png)

1. Recherchez et sélectionnez votre compte d’intégration.

   ![Sélectionner le compte d’intégration](./media/logic-apps-enterprise-integration-liquid-transform/select-integration-account.png)

1. Dans le volet **Vue d’ensemble**, sous **Composants**, sélectionnez **Mappages**.

    ![Sélectionner la vignette « Mappages »](./media/logic-apps-enterprise-integration-liquid-transform/select-maps-tile.png)

1. Dans le volet **Mappages**, sélectionnez **Ajouter** et fournissez les informations suivantes pour votre mappage :

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | **Nom** | `JsonToJsonTemplate` | Nom de votre mappage (« JsontoJsonTemplate » dans cet exemple) |
   | **Type de mappage** | **liquid** | Type de votre mappage. Pour une transformation JSON à JSON, vous devez sélectionner **liquid**. |
   | **Map** | `SimpleJsonToJsonTemplate.liquid` | Fichier de modèle ou de mappage Liquid existant à utiliser pour la transformation (« SimpleJsonToJsonTemplate.liquid » dans cet exemple). Pour trouver ce fichier, vous pouvez utiliser le sélecteur de fichiers. Pour connaître les limites de taille qui s'appliquent aux mappages, consultez [Limites et configuration](../logic-apps/logic-apps-limits-and-config.md#artifact-capacity-limits). |
   |||

   ![Ajouter un modèle Liquid](./media/logic-apps-enterprise-integration-liquid-transform/add-liquid-template.png)

## <a name="add-the-liquid-transformation-action"></a>Ajouter l’action de transformation Liquid

1. Dans le portail Azure, procédez comme suit pour [créer une application logique vierge](../logic-apps/quickstart-create-first-logic-app-workflow.md).

1. Dans le Concepteur d’application logique, ajoutez le [déclencheur Requête](../connectors/connectors-native-reqres.md#add-request) à votre application logique.

1. Sous le déclencheur, choisissez **Nouvelle étape**. Dans la zone de recherche, entrez `liquid` comme filtre, puis sélectionnez cette action : **Transformer de JSON en JSON - Liquid**

   ![Rechercher et sélectionner l’action Liquid](./media/logic-apps-enterprise-integration-liquid-transform/search-action-liquid.png)

1. Ouvrez la liste **Mappage**, puis sélectionnez votre modèle Liquid (« JsonToJsonTemplate » dans cet exemple).

   ![Sélectionner le mappage](./media/logic-apps-enterprise-integration-liquid-transform/select-map.png)

   Si la liste de mappages est vide, votre application logique n’est probablement pas liée à votre compte d’intégration. 
   Pour lier votre application logique au compte d’intégration associé au modèle ou au mappage Liquid, effectuez les étapes suivantes :

   1. Dans le menu de votre application logique, sélectionnez **Paramètres de flux de travail**.

   1. Dans la liste **Sélectionner un compte d’intégration**, sélectionnez votre compte d’intégration et sélectionnez **Enregistrer**.

      ![Lier une application logique à un compte d’intégration](./media/logic-apps-enterprise-integration-liquid-transform/link-integration-account.png)

1. Ajoutez maintenant la propriété **Contenu** à cette action. Ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez la propriété **Contenu**.

   ![Ajouter la propriété « Contenu » à l’action](./media/logic-apps-enterprise-integration-liquid-transform/add-content-property-to-action.png)

1. Pour définir la valeur de la propriété **Contenu**, cliquez à l’intérieur de la zone **Contenu** afin que la liste de contenu dynamique s’affiche. Sélectionnez le jeton **Corps**, qui représente la sortie du contenu du corps à partir du déclencheur.

   ![Sélectionner le jeton « Corps » pour la valeur de propriété « Contenu »](./media/logic-apps-enterprise-integration-liquid-transform/select-body.png)

   Une fois que vous avez terminé, l’action ressemble à cet exemple :

   ![Action « Transformer de JSON en JSON » terminée](./media/logic-apps-enterprise-integration-liquid-transform/finished-transform-action.png)

## <a name="test-your-logic-app"></a>Tester votre application logique

Publiez l’entrée JSON dans votre application logique en utilisant [Postman](https://www.getpostman.com/postman) ou un outil similaire. La sortie JSON transformée à partir de votre application logique ressemble à ceci :

![Exemple de sortie](./media/logic-apps-enterprise-integration-liquid-transform/example-output-jsontojson.png)

<a name="template-considerations"></a>

## <a name="liquid-template-considerations"></a>Considérations relatives aux modèles Liquid

* Les modèles Liquid suivent les [limites de taille de fichier pour les mappages](../logic-apps/logic-apps-limits-and-config.md#artifact-capacity-limits) dans Azure Logic Apps.

* L’action **Transformer de JSON en JSON - Liquid** suit l’[implémentation de DotLiquid pour Liquid](https://github.com/dotliquid/dotliquid). Cette implémentation est un port vers le .NET Framework à partir de l’[implémentation Shopify pour Liquid](https://shopify.github.io/liquid/) et diffère dans des [cas spécifiques](https://github.com/dotliquid/dotliquid/issues).

  Voici les principales différences :

  * L’action **Transformer de JSON en JSON - Liquid** génère en mode natif une chaîne, qui peut inclure du code JSON, XML, HTML, etc. L’action Liquid indique uniquement que la sortie de texte attendue du modèle Liquid est une chaîne JSON. L’action indique à votre application logique d’analyser l’entrée en tant qu’objet JSON et applique un wrapper afin que Liquid puisse interpréter la structure JSON. Après la transformation, l’action indique à votre application logique d’analyser la sortie de texte de Liquid en retour vers JSON.

    DotLiquid ne comprend pas de façon native le code JSON. Veillez donc à échapper la barre oblique inverse (`\`) et tout autre caractère JSON réservé.

  * Si votre modèle utilise des [filtres Liquid](https://shopify.github.io/liquid/basics/introduction/#filters), veillez à respecter les [conventions de nommage DotLiquid et C#](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Designers#filter-and-output-casing), qui utilisent la *casse des phrases*. Pour toutes les transformations Liquid, assurez-vous que les noms des filtres dans votre modèle utilisent également la casse des phrases. Sinon, les filtres ne fonctionneront pas.

    Par exemple, lorsque vous utilisez le filtre `replace`, utilisez `Replace` et non pas `replace`. La même règle s’applique si vous essayez des exemples sur le site [d’essai en ligne de DotLiquid](http://dotliquidmarkup.org/try-online). Pour plus d’informations, consultez les [filtres Shopify Liquid](https://shopify.dev/docs/themes/liquid/reference/filters) et les [filtres DotLiquid Liquid](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Developers#create-your-own-filters). La spécification Shopify inclut des exemples pour chaque filtre. Ainsi, à des fins de comparaison, vous pouvez essayer ces exemples sur le site d’[essai en ligne de DotLiquid](http://dotliquidmarkup.org/try-online).

  * Le filtre `json` des filtres d’extension Shopify est actuellement [non implémenté dans DotLiquid](https://github.com/dotliquid/dotliquid/issues/384). En règle générale, vous pouvez utiliser ce filtre pour préparer la sortie de texte pour l’analyse des chaînes JSON, mais vous devez utiliser le filtre `Replace` à la place.

  * Le filtre `Replace` standard dans l’[implémentation DotLiquid](https://github.com/dotliquid/dotliquid/blob/b6a7d992bf47e7d7dcec36fb402f2e0d70819388/src/DotLiquid/StandardFilters.cs#L425) utilise la [correspondance d’expression régulière (RegEx)](/dotnet/standard/base-types/regular-expression-language-quick-reference), tandis que l’[implémentation Shopify](https://shopify.github.io/liquid/filters/replace/) utilise la [correspondance de chaîne simple](https://github.com/Shopify/liquid/issues/202). Les deux implémentations fonctionnent apparemment de la même façon jusqu’à ce que vous utilisiez un caractère RegEx réservé ou un caractère d’échappement dans le paramètre match.

    Par exemple, pour échapper le caractère d’échappement de barre oblique inverse RegEx réservé (`\`), utilisez `| Replace: '\\', '\\'` et non pas `| Replace: '\', '\\'`. Ces exemples montrent comment le filtre `Replace` se comporte différemment lorsque vous essayez d’échapper le caractère de barre oblique inverse. Cette version fonctionne correctement :

    `{ "SampleText": "{{ 'The quick brown fox "jumped" over the sleeping dog\\' | Replace: '\\', '\\' | Replace: '"', '\"'}}"}`

    Avec le résultat suivant :

    `{ "SampleText": "The quick brown fox \"jumped\" over the sleeping dog\\\\"}`

    Tandis que cette version échoue :

    `{ "SampleText": "{{ 'The quick brown fox "jumped" over the sleeping dog\\' | Replace: '\', '\\' | Replace: '"', '\"'}}"}`

    Avec cette erreur :

    `{ "SampleText": "Liquid error: parsing "\" - Illegal \ at end of pattern."}`

    Pour plus d’informations, consultez [Replace standard filter uses RegEx pattern matching...](https://github.com/dotliquid/dotliquid/issues/385).

  * Le filtre `Sort` dans l’[implémentation de DotLiquid](https://github.com/dotliquid/dotliquid/blob/b6a7d992bf47e7d7dcec36fb402f2e0d70819388/src/DotLiquid/StandardFilters.cs#L326) trie les éléments d’un tableau ou d’une collection par propriété, mais avec les différences suivantes :<p>

    * Suit le [comportement sort_natural de Shopify](https://shopify.github.io/liquid/filters/sort_natural/) et non pas le [comportement sort de Shopify](https://shopify.github.io/liquid/filters/sort/).

    * Trie uniquement dans l’ordre alphanumérique des chaînes. Pour plus d’informations, consultez [Numeric sort](https://github.com/Shopify/liquid/issues/980).

    * Utilise un ordre *ne respectant pas la casse*, et non pas un ordre respectant la casse. Pour plus d’informations, consultez [Sort filter does not follow casing behavior from Shopify’s specification]( https://github.com/dotliquid/dotliquid/issues/393).

<a name="other-transformations"></a>

## <a name="other-transformations-using-liquid"></a>Autres transformations utilisant Liquid

Liquid n’est pas limité aux seules transformations de code JSON. Vous pouvez également utiliser Liquid pour effectuer d’autres transformations, par exemple :

* [de JSON en texte](#json-text)
* [de XML vers JSON](#xml-json)
* [de XML en texte](#xml-text)

<a name="json-text"></a>

### <a name="transform-json-to-text"></a>Transformer du JSON en texte

Voici le modèle Liquid utilisé pour cet exemple :

```json
{{content.firstName | Append: ' ' | Append: content.lastName}}
```

Voici les exemples d’entrées et de sorties :

![Exemple de sortie JSON en texte](./media/logic-apps-enterprise-integration-liquid-transform/example-output-jsontotext.png)

<a name="xml-json"></a>

### <a name="transform-xml-to-json"></a>Transformer du XML en JSON

Voici le modèle Liquid utilisé pour cet exemple :

``` json
[{% JSONArrayFor item in content -%}
      {{item}}
  {% endJSONArrayFor -%}]
```

La boucle `JSONArrayFor` est un mécanisme de boucle personnalisé pour les entrées XML, qui permet de créer des charges utiles JSON sans virgule de fin. En outre, la condition `where` de ce mécanisme de boucle personnalisé utilise le nom de l’élément XML à des fins de comparaison, plutôt que la valeur de l’élément comme d’autres filtres Liquid. Pour plus d’informations, consultez une [présentation approfondie de la stratégie set-body - collections d’éléments](https://azure.microsoft.com/blog/deep-dive-on-set-body-policy).

Voici les exemples d’entrées et de sorties :

![Exemple de sortie XML en JSON](./media/logic-apps-enterprise-integration-liquid-transform/example-output-xmltojson.png)

<a name="xml-text"></a>

### <a name="transform-xml-to-text"></a>Transformer du XML en texte

Voici le modèle Liquid utilisé pour cet exemple :

``` json
{{content.firstName | Append: ' ' | Append: content.lastName}}
```

Voici les exemples d’entrées et de sorties :

![Exemple de sortie XML en texte](./media/logic-apps-enterprise-integration-liquid-transform/example-output-xmltotext.png)

## <a name="next-steps"></a>Étapes suivantes

* [Langage et exemples Shopify Liquid](https://shopify.github.io/liquid/basics/introduction/)
* [DotLiquid](http://dotliquidmarkup.org/)
* [Essai en ligne de DotLiquid](http://dotliquidmarkup.org/try-online)
* [Page GitHub de DotLiquid](https://github.com/dotliquid/dotliquid)
* [Page GitHub des problèmes relatifs à DotLiquid](https://github.com/dotliquid/dotliquid/issues/)
* En savoir plus sur les [mappages](../logic-apps/logic-apps-enterprise-integration-maps.md)
