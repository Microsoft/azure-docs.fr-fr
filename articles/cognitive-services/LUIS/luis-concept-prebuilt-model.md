---
title: Modèles prédéfinis – LUIS
titleSuffix: Azure Cognitive Services
description: Les modèles prédéfinis fournissent des domaines, des intentions, des énoncés et des entités. Vous pouvez démarrer votre application avec un domaine prédéfini ou ajouter un domaine approprié à votre application ultérieurement.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 10/10/2019
ms.openlocfilehash: 6642e59c2957b298d54bc587853752b9fce74686
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95019141"
---
# <a name="prebuilt-models"></a>Modèles prédéfinis

Les modèles prédéfinis fournissent des domaines, des intentions, des énoncés et des entités. Vous pouvez démarrer votre application avec un modèle prédéfini ou ajouter un modèle approprié à votre application ultérieurement. 

## <a name="types-of-prebuilt-models"></a>Types de modèles prédéfinis

LUIS fournit trois types de modèles prédéfinis. Chaque modèle peut être ajouté à votre application à tout moment. 

|Type de modèle|Includes|
|--|--|
|[Domaine](luis-reference-prebuilt-domains.md)|Intentions, énoncés, entités|
|Intentions|Intentions, énoncés|
|[Entités](luis-reference-prebuilt-entities.md)|Entités uniquement| 

## <a name="prebuilt-domains"></a>Domaines prédéfinis

Language Understanding (LUIS) fournit des *domaines prédéfinis*, à savoir des modèles préformés d’[intentions](luis-how-to-add-intents.md) et d’[entités](luis-concept-entity-types.md) qui fonctionnent conjointement pour des domaines ou des catégories communes d’applications clientes. 

Les domaines prédéfinis sont formés et prêts à êtres ajoutés à votre application LUIS. Les intentions et les entités d’un domaine prédéfini sont entièrement personnalisables, une fois que vous les avez ajoutées à votre application. 

> [!TIP]
> Les intentions et les entités d’un domaine prédéfini fonctionnent le mieux ensemble. Il est préférable de combiner des intentions et des entités du même domaine lorsque cela est possible.
> Le domaine prédéfini Utilitaires comprend des intentions que vous pouvez personnaliser pour une utilisation dans n’importe quel domaine. Par exemple, vous pouvez ajouter `Utilities.Repeat` à votre application et lui apprendre à reconnaître les actions que les utilisateurs souhaitent répéter dans votre application. 

### <a name="changing-the-behavior-of-a-prebuilt-domain-intent"></a>Modification du comportement d’une intention de domaine prédéfini

Vous constaterez peut-être qu’un domaine prédéfini contient une intention semblable à une intention que vous souhaitez avoir dans votre application LUIS, mais avec un comportement différent. Par exemple, le domaine prédéfini **Lieux** fournit une intention `MakeReservation` pour effectuer une réservation au restaurant, mais vous voulez que votre application utilise cette intention pour faire des réservations d’hôtel. Dans ce cas, vous pouvez modifier le comportement de cette intention en ajoutant des exemples d’énoncés à l’intention dans le but d’effectuer des réservations d’hôtel, puis effectuer à nouveau l’apprentissage de l’application. 

Vous trouverez une liste complète des domaines prédéfinis dans la [référence des domaines prédéfinis](./luis-reference-prebuilt-domains.md).

## <a name="prebuilt-intents"></a>Intentions prédéfinies

LUIS fournit les intentions prédéfinies et leurs énoncés pour chacun de ses domaines prédéfinis. Les intentions peuvent être ajoutées sans ajouter le domaine entier. Pour ajouter une intention, il faut ajouter une intention et ses énoncés à votre application. Le nom de l’intention et la liste d’énoncés peuvent être modifiés.  

## <a name="prebuilt-entities"></a>Entités prédéfinies

LUIS comprend un ensemble d’entités prédéfinies pour reconnaître des types courants d’informations telles que des dates, heures, nombres, mesures et devises. La prise en charge de l’entité prédéfinie varie selon la culture de votre application LUIS. Pour obtenir la liste complète des entités prédéfinies que LUIS prend en charge, notamment en lien avec la culture, voir la [documentation de référence sur les entités prédéfinies](./luis-reference-prebuilt-entities.md).

Quand une entité prédéfinie est incluse dans votre application, ses prédictions sont incluses dans votre application publiée. Le comportement des entités prédéfinies est préformé et **ne peut pas** être modifié. 

> [!NOTE]
> L’entité **builtin.datetime** est désapprouvée. Elle est remplacée par l’entité [**builtin.datetimev2**](luis-reference-prebuilt-datetimev2.md), qui fournit la reconnaissance des plages de dates et d’heures, ainsi qu’une reconnaissance améliorée des dates et heures ambiguës.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [ajouter des entités prédéfinies](./howto-add-prebuilt-models.md) à votre application.