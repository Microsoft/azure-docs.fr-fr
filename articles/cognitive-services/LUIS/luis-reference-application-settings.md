---
title: Paramètres d’application - LUIS
description: Les paramètres d’application pour des applications Language Understanding Azure Cognitive Services sont stockés dans l’application et le portail.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 05/04/2020
ms.openlocfilehash: 0578e3c3c952a475c6beb01ffcf354e19eda6e26
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91319161"
---
# <a name="app-and-version-settings"></a>Paramètres d’application et de version

Ces paramètres sont stockés dans l’application [exportée](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c40) et mis à jour avec les API REST ou le portail LUI.

Le fait de modifier vos paramètres de version d’application réinitialise le statut d’entraînement de votre application sur « Non entraînée ».

[!INCLUDE [App and version settings](includes/app-version-settings.md)]


Une référence de texte et des exemples incluent ce qui suit :

* [Ponctuation](#punctuation-normalization)
* [Signes diacritiques](#diacritics-normalization)

## <a name="diacritics-normalization"></a>Normalisation des signes diacritiques

Les énoncés suivants montrent l’impact de la normalisation des signes diacritiques sur les énoncés :

|Avec les signes diacritiques définis sur false|Avec les signes diacritiques définis sur true|
|--|--|
|`quiero tomar una piña colada`|`quiero tomar una pina colada`|
|||

### <a name="language-support-for-diacritics"></a>Prise en charge linguistique pour les signes diacritiques

#### <a name="brazilian-portuguese-pt-br-diacritics"></a>Signes diacritiques pour le Portugais (Brésil) `pt-br`

|Signes diacritiques définis sur false|Signes diacritiques définis sur true|
|-|-|
|`á`|`a`|
|`â`|`a`|
|`ã`|`a`|
|`à`|`a`|
|`ç`|`c`|
|`é`|`e`|
|`ê`|`e`|
|`í`|`i`|
|`ó`|`o`|
|`ô`|`o`|
|`õ`|`o`|
|`ú`|`u`|
|||

#### <a name="dutch-nl-nl-diacritics"></a>Signes diacritiques pour le Néerlandais `nl-nl`

|Signes diacritiques définis sur false|Signes diacritiques définis sur true|
|-|-|
|`á`|`a`|
|`à`|`a`|
|`é`|`e`|
|`ë`|`e`|
|`è`|`e`|
|`ï`|`i`|
|`í`|`i`|
|`ó`|`o`|
|`ö`|`o`|
|`ú`|`u`|
|`ü`|`u`|
|||

#### <a name="french-fr--diacritics"></a>Signes diacritiques pour le Français `fr-`

Cela inclut les sous-cultures Français et Canadien.

|Signes diacritiques définis sur false|Signes diacritiques définis sur true|
|--|--|
|`é`|`e`|
|`à`|`a`|
|`è`|`e`|
|`ù`|`u`|
|`â`|`a`|
|`ê`|`e`|
|`î`|`i`|
|`ô`|`o`|
|`û`|`u`|
|`ç`|`c`|
|`ë`|`e`|
|`ï`|`i`|
|`ü`|`u`|
|`ÿ`|`y`|

#### <a name="german-de-de-diacritics"></a>Signes diacritiques pour l’Allemand `de-de`

|Signes diacritiques définis sur false|Signes diacritiques définis sur true|
|--|--|
|`ä`|`a`|
|`ö`|`o`|
|`ü`|`u`|

#### <a name="italian-it-it-diacritics"></a>Signes diacritiques pour l’Italien `it-it`

|Signes diacritiques définis sur false|Signes diacritiques définis sur true|
|--|--|
|`à`|`a`|
|`è`|`e`|
|`é`|`e`|
|`ì`|`i`|
|`í`|`i`|
|`î`|`i`|
|`ò`|`o`|
|`ó`|`o`|
|`ù`|`u`|
|`ú`|`u`|

#### <a name="spanish-es--diacritics"></a>Signes diacritiques pour l’Espagnol `es-`

Cela inclut les sous-cultures Mexicain et Espagnol.

|Signes diacritiques définis sur false|Signes diacritiques définis sur true|
|-|-|
|`á`|`a`|
|`é`|`e`|
|`í`|`i`|
|`ó`|`o`|
|`ú`|`u`|
|`ü`|`u`|
|`ñ`|`u`|

## <a name="punctuation-normalization"></a>Normalisation des signes de ponctuation

Les énoncés suivants montrent l’impact de la ponctuation sur les énoncés :

|Avec la ponctuation définie sur False|Avec la ponctuation définie sur True|
|--|--|
|`Hmm..... I will take the cappuccino`|`Hmm I will take the cappuccino`|
|||

### <a name="punctuation-removed"></a>Ponctuation supprimée

La ponctuation suivante est supprimée quand `NormalizePunctuation` a la valeur true.

|Ponctuation|
|--|
|`-`|
|`.`|
|`'`|
|`"`|
|`\`|
|`/`|
|`?`|
|`!`|
|`_`|
|`,`|
|`;`|
|`:`|
|`(`|
|`)`|
|`[`|
|`]`|
|`{`|
|`}`|
|`+`|
|`¡`|

## <a name="next-steps"></a>Étapes suivantes

* Découvrez les [concepts](luis-concept-utterance.md#utterance-normalization-for-diacritics-and-punctuation) des signes diacritiques et de la ponctuation.
