---
title: Langage SSML - Service Speech
titleSuffix: Azure Cognitive Services
description: Utilisation du langage de balisage de synthèse vocale pour contrôler la prononciation et la prosodie dans la synthèse vocale.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/23/2020
ms.author: trbye
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: df6a4053eb70c02e27599bbd9086dfa32b0bcc65
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97508830"
---
# <a name="improve-synthesis-with-speech-synthesis-markup-language-ssml"></a>Améliorer la synthèse avec le langage de balisage de synthèse vocale (SSML, Speech Synthesis Markup Language)

Le langage de balisage de synthèse vocale (SSML) est un langage de balisage basé sur XML qui permet aux développeurs de spécifier la manière dont un texte en entrée est converti en parole synthétisée via le service de synthèse vocale. Comparé à du texte brut, le SSML permet aux développeurs de régler finement la tonalité, la prononciation, le débit, le volume et d’autres paramètres de la synthèse vocale. La ponctuation normale, telle que la pause après un point ou l’utilisation de l’intonation correcte quand une phrase se termine par un point d’interrogation, est traitée automatiquement.

L’implémentation par SSML du service Speech est basée sur le [langage SSML version 1.0](https://www.w3.org/TR/speech-synthesis) du World Wide Web Consortium.

> [!IMPORTANT]
> Les caractères chinois, japonais et coréens comptent pour deux en matière de facturation. Pour plus d’informations, voir la [tarification](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

## <a name="standard-neural-and-custom-voices"></a>Voix standard, neuronales et personnalisées

Choisissez parmi les voix standard et neuronales, ou créez une voix personnalisée propre à votre produit ou votre marque. Plus de 75 voix standard sont disponibles dans plus de 45 langues et paramètres régionaux, et 5 voix neurales sont disponibles dans 4 langues et paramètres régionaux. Pour obtenir la liste complète des langues, paramètres régionaux et voix (neuronales et standard) pris en charge, consultez [prise en charge linguistique](language-support.md).

Pour en savoir plus sur les voix standard, neuronales et personnalisées, voir [Vue d’ensemble de la synthèse vocale](text-to-speech.md).

## <a name="special-characters"></a>Caractères spéciaux

Lorsque vous utilisez SSML, n’oubliez pas que les caractères spéciaux, tels que les guillemets, les apostrophes et les crochets, doivent être placés dans une séquence d’échappement. Pour plus d’informations, consultez la page [Extensible Markup Language (XML) 1.0 : Annexe D](https://www.w3.org/TR/xml/#sec-entexpand).

## <a name="supported-ssml-elements"></a>Éléments SSML pris en charge

Chaque document SSML est créé avec des éléments SSML (ou les balises). Ces éléments sont utilisés pour ajuster la tonalité, la prosodie, le volume et d’autres paramètres. Les sections suivantes détaillent la manière dont chaque élément est utilisé, et quand il est obligatoire ou facultatif.  

> [!IMPORTANT]
> N’oubliez pas d’entourer les valeurs d’attribut de guillemets. Les normes pour un code XML bien formé valide exigent que les valeurs d’attribut soient placées entre guillemets. Par exemple, `<prosody volume="90">` est un élément bien formé valide, mais `<prosody volume=90>` ne l’est pas. Le SSML peut ne pas reconnaître des valeurs d’attribut non entourées de guillemets.

## <a name="create-an-ssml-document"></a>Créer un document SSML

`speak` est l’élément racine **requis** pour tous les documents SSML. L’élément `speak` contient des informations importantes, telles que la version, la langue et la définition de vocabulaire de balisage.

**Syntaxe**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="string"></speak>
```

**Attributs**

| Attribut | Description | Obligatoire/facultatif |
|-----------|-------------|---------------------|
| `version` | Indique la version de la spécification SSML utilisée pour interpréter le balisage de document. La version actuelle est 1.0. | Obligatoire |
| `xml:lang` | Spécifie la langue du document racine. La valeur peut contenir un code langue de deux lettres minuscules (par exemple, `en`), ou le code langue associé au code du pays ou de la région en majuscules (par exemple, `en-US`). | Obligatoire |
| `xmlns` | Spécifie l’URI du document définissant le vocabulaire de balisage (types d’éléments et noms d’attribut) du document SSML. L’URI en cours est http://www.w3.org/2001/10/synthesis. | Obligatoire |

## <a name="choose-a-voice-for-text-to-speech"></a>Choisir une voix de synthèse vocale

L’élément `voice` est obligatoire. Il spécifie la voix utilisée pour la synthèse vocale.

**Syntaxe**

```xml
<voice name="string">
    This text will get converted into synthesized speech.
</voice>
```

**Attributs**

| Attribut | Description | Obligatoire/facultatif |
|-----------|-------------|---------------------|
| `name` | Identifie la voix utilisée pour la sortie de synthèse vocale. Pour accéder à la liste complète des voix prises en charge, voir [Prise en charge des langues](language-support.md#text-to-speech). | Obligatoire |

**Exemple**

> [!NOTE]
> Cet exemple utilise la voix `en-US-AriaRUS`. Pour accéder à la liste complète des voix prises en charge, voir [Prise en charge des langues](language-support.md#text-to-speech).

```XML
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaRUS">
        This is the text that is spoken.
    </voice>
</speak>
```

## <a name="use-multiple-voices"></a>Utiliser plusieurs voix

Dans l’élément `speak`, vous pouvez spécifier plusieurs voix pour la sortie de synthèse vocale. Ces voix peuvent être dans différentes langues. Pour chaque voix, le texte doit être encapsulé dans un élément `voice`. 

**Attributs**

| Attribut | Description | Obligatoire/facultatif |
|-----------|-------------|---------------------|
| `name` | Identifie la voix utilisée pour la sortie de synthèse vocale. Pour accéder à la liste complète des voix prises en charge, voir [Prise en charge des langues](language-support.md#text-to-speech). | Obligatoire |

> [!IMPORTANT]
> Les voix multiples sont incompatibles avec la fonctionnalité de limite de mot. La fonctionnalité de limite de mot doit être désactivée pour pouvoir utiliser plusieurs voix.

### <a name="disable-word-boundary"></a>Désactiver la limite de mot

Selon la langue du Kit de développement logiciel (SDK) Speech, vous définirez la propriété `"SpeechServiceResponse_Synthesis_WordBoundaryEnabled"` sur `false` sur une instance de l’objet `SpeechConfig`.

# <a name="c"></a>[C#](#tab/csharp)

Pour plus d’informations, consultez <a href="https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.setproperty?view=azure-dotnet" target="_blank"> `SetProperty` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```csharp
speechConfig.SetProperty(
    "SpeechServiceResponse_Synthesis_WordBoundaryEnabled", "false");
```

# <a name="c"></a>[C++](#tab/cpp)

Pour plus d’informations, consultez <a href="https://docs.microsoft.com/cpp/cognitive-services/speech/speechconfig#setproperty" target="_blank"> `SetProperty` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```cpp
speechConfig->SetProperty(
    "SpeechServiceResponse_Synthesis_WordBoundaryEnabled", "false");
```

# <a name="java"></a>[Java](#tab/java)

Pour plus d’informations, consultez <a href="https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig.setproperty#com_microsoft_cognitiveservices_speech_SpeechConfig_setProperty_String_String_" target="_blank"> `setProperty` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```java
speechConfig.setProperty(
    "SpeechServiceResponse_Synthesis_WordBoundaryEnabled", "false");
```

# <a name="python"></a>[Python](#tab/python)

Pour plus d’informations, consultez <a href="https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python#set-property-by-name-property-name--str--value--str-" target="_blank"> `set_property_by_name` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```python
speech_config.set_property_by_name(
    "SpeechServiceResponse_Synthesis_WordBoundaryEnabled", "false");
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Pour plus d’informations, consultez <a href="https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?view=azure-node-latest&preserve-view=true#setproperty-string--string-" target="_blank"> `setProperty` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```javascript
speechConfig.setProperty(
    "SpeechServiceResponse_Synthesis_WordBoundaryEnabled", "false");
```

# <a name="objective-c"></a>[Objective-C](#tab/objectivec)

Pour plus d’informations, consultez <a href="https://docs.microsoft.com/objectivec/cognitive-services/speech/spxspeechconfiguration#setpropertytobyname" target="_blank"> `setPropertyTo` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```objectivec
[speechConfig setPropertyTo:@"false" byName:@"SpeechServiceResponse_Synthesis_WordBoundaryEnabled"];
```

# <a name="swift"></a>[Swift](#tab/swift)

Pour plus d’informations, consultez <a href="https://docs.microsoft.com/objectivec/cognitive-services/speech/spxspeechconfiguration#setpropertytobyname" target="_blank"> `setPropertyTo` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```swift
speechConfig!.setPropertyTo(
    "false", byName: "SpeechServiceResponse_Synthesis_WordBoundaryEnabled")
```

---

**Exemple**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaRUS">
        Good morning!
    </voice>
    <voice name="en-US-Guy24kRUS">
        Good morning to you too Aria!
    </voice>
</speak>
```

## <a name="adjust-speaking-styles"></a>Ajuster les styles oraux

> [!IMPORTANT]
> L’ajustement des styles oraux ne fonctionne qu’avec les voix neurales.

Par défaut, le service de synthèse vocale synthétise le texte à l’aide d’un style oral neutre pour les voix standard et neuronales. Avec les voix neurales, vous pouvez ajuster le ton pour exprimer différentes émotions, comme la joie, l’empathie ou le calme, ou bien optimiser la voix pour différents scénarios comme le service client, la diffusion d’informations et les assistants vocaux, à l’aide de l’élément `mstts:express-as`. Il s’agit d’un élément facultatif propre au service Speech.

Actuellement, des ajustements de style oral sont pris en charge pour ces voix neuronales :
* `en-US-AriaNeural`
* `en-US-JennyNeural`
* `en-US-GuyNeural`
* `zh-CN-XiaoxiaoNeural`
* `zh-CN-YunyangNeural`
* `zh-CN-YunyeNeural`
* `zh-CN-YunxiNeural` (version préliminaire)
* `zh-CN-XiaohanNeural` (version préliminaire)
* `zh-CN-XiaomoNeural` (version préliminaire)
* `zh-CN-XiaoxuanNeural` (version préliminaire)
* `zh-CN-XiaoruiNeural` (version préliminaire)

L’intensité du style d’élocution peut être modifiée pour mieux correspondre à votre cas d’usage. Vous pouvez spécifier un style plus fort ou plus doux avec `styledegree` pour rendre la voix plus expressive ou feutrée. 

Actuellement, des ajustements de style oral sont pris en charge pour ces voix neuronales :
* `zh-CN-XiaoxiaoNeural`

En plus d’ajuster les styles d’élocution et le degré de style, vous pouvez aussi ajuster le paramètre `role` afin que la voix imite un âge et un sexe différents. Par exemple, une voix masculine peut devenir plus aigüe et changer d’intonation pour imiter une voix féminine.

Actuellement, les ajustements de jeux de rôle sont pris en charge pour ces voix neuronales :
* `zh-CN-XiaomoNeural`
* `zh-CN-XiaoxuanNeural`

Les modifications ci-dessus sont appliquées au niveau de la phrase, et les styles et jeux de rôle varient selon la voix. Si un style ou jeu de rôle n’est pas pris en charge, le service retourne la voix dans le style d’élocution neutre par défaut. Vous pouvez voir les styles et jeux de rôle pris en charge pour chaque voix par le biais de l’[API Voice List](rest-text-to-speech.md#get-a-list-of-voices) ou celui de la plateforme de [création de contenu audio ](https://aka.ms/audiocontentcreation) sans code.

**Syntaxe**

```xml
<mstts:express-as style="string"></mstts:express-as>
```
```xml
<mstts:express-as style="string" styledegree="value"></mstts:express-as>
```
```xml
<mstts:express-as role="string" style="string"></mstts:express-as>
```
> [!NOTE]
> Pour le moment, `styledegree` prend uniquement en charge zh-CN-XiaoxiaoNeural. `role` prend uniquement en charge zh-CN-XiaomoNeural et zh-CN-XiaoxuanNeural.

**Attributs**

| Attribut | Description | Obligatoire/facultatif |
|-----------|-------------|---------------------|
| `style` | Spécifie le style oral. Actuellement, les styles oraux sont spécifiques à la voix. | Obligatoire en cas d’ajustement du style oral pour une voix neuronale. Si vous utilisez `mstts:express-as`, le style doit être fourni. Si une valeur non valide est fournie, cet élément est ignoré. |
| `styledegree` | Spécifie l’intensité du style d’élocution. **Valeurs acceptées** : 0,01 à 2 incluse. La valeur par défaut est 1, ce qui correspond à l’intensité de style prédéfinie. L’unité minimale est 0,01, ce qui aboutit à une légère tendance pour le style cible. La valeur 2 produit un doublement de l’intensité de style par défaut.  | Facultatif (Pour le moment, `styledegree` prend uniquement en charge zh-CN-XiaoxiaoNeural.)|
| `role` | Spécifie le jeu de rôle. La voix correspond à un âge et un sexe différents.  | Facultatif (Pour le moment, `role` prend uniquement en charge zh-CN-XiaomoNeural et zh-CN-XiaoxuanNeural.)|

Reportez-vous à ce tableau pour déterminer les styles oraux pris en charge pour chaque voix neuronale.

| Voix                   | Style                     | Description                                                 |
|-------------------------|---------------------------|-------------------------------------------------------------|
| `en-US-AriaNeural`      | `style="newscast-formal"` | Exprime un ton formel, confiant et autoritaire pour la diffusion des actualités |
|                         | `style="newscast-casual"` | Exprime un ton polyvalent et décontracté pour la diffusion d’informations générales        |
|                         | `style="customerservice"` | Exprime un ton convivial et pragmatique pour le support technique  |
|                         | `style="chat"`            | Exprime un ton informel et détendu                         |
|                         | `style="cheerful"`        | Exprime un ton positif et joyeux                         |
|                         | `style="empathetic"`      | Exprime une de la compassion et de la compréhension               |
| `en-US-JennyNeural`     | `style="customerservice"` | Exprime un ton convivial et pragmatique pour le support technique  |
|                         | `style="chat"`            | Exprime un ton informel et détendu                         |
|                         | `style="assistant"`       | Exprime un ton chaud et détendu pour les assistants numériques    |
|                         | `style="newscast"`        | Exprime un ton polyvalent et décontracté pour la diffusion d’informations générales   |
| `en-US-GuyNeural`       | `style="newscast"`        | Exprime un ton formel et professionnel pour la présentation des actualités |
| `zh-CN-XiaoxiaoNeural`  | `style="newscast"`        | Exprime un ton formel et professionnel pour la présentation des actualités |
|                         | `style="customerservice"` | Exprime un ton convivial et pragmatique pour le support technique  |
|                         | `style="assistant"`       | Exprime un ton chaud et détendu pour les assistants numériques    |
|                         | `style="chat"`            | Exprime un ton informel et détendu pour un échange de conversation (chit-chat)           |
|                         | `style="calm"`            | Exprime une attitude sympathique, calme et posée dans une conversation. Le ton, la hauteur de la voix, la prosodie sont bien plus uniformes par rapport aux autres types de voix.                                |
|                         | `style="cheerful"`        | Exprime un ton optimiste et enthousiaste, avec une plus grande hauteur de voix et énergie vocale.                         |
|                         | `style="sad"`             | Exprime un ton triste, avec une plus grande hauteur de voix, une intensité moindre et une énergie vocale plus faible. Les gémissements ou les pleurs pendant l’élocution indiquent généralement cette émotion.            |
|                         | `style="angry"`           | Exprime un ton fâché et énervé, avec une hauteur de voix plus basse, une intensité plus grande et une énergie vocale plus élevée. L’orateur est en colère, mécontent et offensé.       |
|                         | `style="fearful"`         | Exprime un ton effrayé et nerveux, avec une plus grande hauteur de voix, une plus grande énergie vocale et un débit plus rapide. L’orateur est dans un état de tension et de malaise.                          |
|                         | `style="disgruntled"`     | Exprime un ton dédaigneux et plaintif. L’expression de cette émotion témoigne du mécontentement et du mépris.              |
|                         | `style="serious"`         | Exprime un ton strict et autoritaire. L’orateur a souvent l’air sévère et peu détendu et sa cadence reste ferme.          |
|                         | `style="affectionate"`    | Exprime un ton chaleureux et affectueux, avec une hauteur de voix et une énergie vocale plus grandes. L’orateur attire l’attention de celui qui écoute. La « personnalité » de l’orateur est souvent d’une nature attachante.          |     
|                         | `style="gentle"`          | Exprime un ton doux, poli et agréable, avec une hauteur de voix et une énergie vocale moins élevées         |   
|                         | `style="lyrical"`         | Exprime les émotions d’une manière mélodique et sentimentale         |   
| `zh-CN-YunyangNeural`   | `style="customerservice"` | Exprime un ton convivial et pragmatique pour le support technique  | 
| `zh-CN-YunyeNeural`     | `style="calm"`            | Exprime une attitude sympathique, calme et posée dans une conversation. Le ton, la hauteur de la voix, la prosodie sont bien plus uniformes par rapport aux autres types de voix.    | 
|                         | `style="cheerful"`        | Exprime un ton optimiste et enthousiaste, avec une plus grande hauteur de voix et énergie vocale.                         |
|                         | `style="sad"`             | Exprime un ton triste, avec une plus grande hauteur de voix, une intensité moindre et une énergie vocale plus faible. Les gémissements ou les pleurs pendant l’élocution indiquent généralement cette émotion.            |
|                         | `style="angry"`           | Exprime un ton fâché et énervé, avec une hauteur de voix plus basse, une intensité plus grande et une énergie vocale plus élevée. L’orateur est en colère, mécontent et offensé.       |
|                         | `style="fearful"`         | Exprime un ton effrayé et nerveux, avec une plus grande hauteur de voix, une plus grande énergie vocale et un débit plus rapide. L’orateur est dans un état de tension et de malaise.                          |
|                         | `style="disgruntled"`     | Exprime un ton dédaigneux et plaintif. L’expression de cette émotion témoigne du mécontentement et du mépris.              |
|                         | `style="serious"`         | Exprime un ton strict et autoritaire. L’orateur a souvent l’air sévère et peu détendu et sa cadence reste ferme.          |
| `zh-CN-YunxiNeural`     | `style="cheerful"`        | Exprime un ton optimiste et enthousiaste, avec une plus grande hauteur de voix et énergie vocale.                         |
|                         | `style="sad"`             | Exprime un ton triste, avec une plus grande hauteur de voix, une intensité moindre et une énergie vocale plus faible. Les gémissements ou les pleurs pendant l’élocution indiquent généralement cette émotion.            |
|                         | `style="angry"`           | Exprime un ton fâché et énervé, avec une hauteur de voix plus basse, une intensité plus grande et une énergie vocale plus élevée. L’orateur est en colère, mécontent et offensé.       |
|                         | `style="fearful"`         | Exprime un ton effrayé et nerveux, avec une plus grande hauteur de voix, une plus grande énergie vocale et un débit plus rapide. L’orateur est dans un état de tension et de malaise.                          |
|                         | `style="disgruntled"`     | Exprime un ton dédaigneux et plaintif. L’expression de cette émotion témoigne du mécontentement et du mépris.              |
|                         | `style="serious"`         | Exprime un ton strict et autoritaire. L’orateur a souvent l’air sévère et peu détendu et sa cadence reste ferme.    |
|                         | `style="depressed"`       | Exprime un ton mélancolique et découragé avec une hauteur de voix plus basse et moins d’énergie    |
|                         | `style="embarrassed"`     | Exprime un ton incertain et hésitant quand l’orateur se sent mal à l’aise   |
| `zh-CN-XiaohanNeural`   | `style="cheerful"`        | Exprime un ton optimiste et enthousiaste, avec une plus grande hauteur de voix et énergie vocale.                         |
|                         | `style="sad"`             | Exprime un ton triste, avec une plus grande hauteur de voix, une intensité moindre et une énergie vocale plus faible. Les gémissements ou les pleurs pendant l’élocution indiquent généralement cette émotion.            |
|                         | `style="angry"`           | Exprime un ton fâché et énervé, avec une hauteur de voix plus basse, une intensité plus grande et une énergie vocale plus élevée. L’orateur est en colère, mécontent et offensé.       |
|                         | `style="fearful"`         | Exprime un ton effrayé et nerveux, avec une plus grande hauteur de voix, une plus grande énergie vocale et un débit plus rapide. L’orateur est dans un état de tension et de malaise.                          |
|                         | `style="disgruntled"`     | Exprime un ton dédaigneux et plaintif. L’expression de cette émotion témoigne du mécontentement et du mépris.              |
|                         | `style="serious"`         | Exprime un ton strict et autoritaire. L’orateur a souvent l’air sévère et peu détendu et sa cadence reste ferme.    |
|                         | `style="embarrassed"`     | Exprime un ton incertain et hésitant quand l’orateur se sent mal à l’aise   |
|                         | `style="affectionate"`    | Exprime un ton chaleureux et affectueux, avec une hauteur de voix et une énergie vocale plus grandes. L’orateur attire l’attention de celui qui écoute. La « personnalité » de l’orateur est souvent d’une nature attachante.          |     
|                         | `style="gentle"`          | Exprime un ton doux, poli et agréable, avec une hauteur de voix et une énergie vocale moins élevées         |   
| `zh-CN-XiaomoNeural`    | `style="cheerful"`        | Exprime un ton optimiste et enthousiaste, avec une plus grande hauteur de voix et énergie vocale.                         |
|                         | `style="angry"`           | Exprime un ton fâché et énervé, avec une hauteur de voix plus basse, une intensité plus grande et une énergie vocale plus élevée. L’orateur est en colère, mécontent et offensé.       |
|                         | `style="fearful"`         | Exprime un ton effrayé et nerveux, avec une plus grande hauteur de voix, une plus grande énergie vocale et un débit plus rapide. L’orateur est dans un état de tension et de malaise.                          |
|                         | `style="disgruntled"`     | Exprime un ton dédaigneux et plaintif. L’expression de cette émotion témoigne du mécontentement et du mépris.              |
|                         | `style="serious"`         | Exprime un ton strict et autoritaire. L’orateur a souvent l’air sévère et peu détendu et sa cadence reste ferme.    |
|                         | `style="depressed"`       | Exprime un ton mélancolique et découragé avec une hauteur de voix plus basse et moins d’énergie    |
|                         | `style="gentle"`          | Exprime un ton doux, poli et agréable, avec une hauteur de voix et une énergie vocale moins élevées         |  
| `zh-CN-XiaoxuanNeural`  | `style="cheerful"`        | Exprime un ton optimiste et enthousiaste, avec une plus grande hauteur de voix et énergie vocale.                         |
|                         | `style="angry"`           | Exprime un ton fâché et énervé, avec une hauteur de voix plus basse, une intensité plus grande et une énergie vocale plus élevée. L’orateur est en colère, mécontent et offensé.       |
|                         | `style="fearful"`         | Exprime un ton effrayé et nerveux, avec une plus grande hauteur de voix, une plus grande énergie vocale et un débit plus rapide. L’orateur est dans un état de tension et de malaise.                          |
|                         | `style="disgruntled"`     | Exprime un ton dédaigneux et plaintif. L’expression de cette émotion témoigne du mécontentement et du mépris.              |
|                         | `style="serious"`         | Exprime un ton strict et autoritaire. L’orateur a souvent l’air sévère et peu détendu et sa cadence reste ferme.    |
|                         | `style="depressed"`       | Exprime un ton mélancolique et découragé avec une hauteur de voix plus basse et moins d’énergie    |
|                         | `style="gentle"`          | Exprime un ton doux, poli et agréable, avec une hauteur de voix et une énergie vocale moins élevées         |   
| `zh-CN-XiaoruiNeural`    | `style="sad"`             | Exprime un ton triste, avec une plus grande hauteur de voix, une intensité moindre et une énergie vocale plus faible. Les gémissements ou les pleurs pendant l’élocution indiquent généralement cette émotion.            |
|                         | `style="angry"`           | Exprime un ton fâché et énervé, avec une hauteur de voix plus basse, une intensité plus grande et une énergie vocale plus élevée. L’orateur est en colère, mécontent et offensé.       |
|                         | `style="fearful"`         | Exprime un ton effrayé et nerveux, avec une plus grande hauteur de voix, une plus grande énergie vocale et un débit plus rapide. L’orateur est dans un état de tension et de malaise.                          |

Reportez-vous à ce tableau pour déterminer les rôles pris en charge pour chaque voix neuronale.

| Voix                   | Role                       | Description                                                 |
|-------------------------|----------------------------|-------------------------------------------------------------|
| `zh-CN-XiaomoNeural`    | `role="YoungAdultFemale"`  | La voix imite celle d’une jeune femme adulte.                 |
|                         | `role="OlderAdultMale"`    | La voix imite celle d’un homme adulte plus âgé.                   |
|                         | `role="Girl"`              | La voix imite celle d’une petite fille.                               |
|                         | `role="Boy"`               | La voix imite celle d’un petit garçon.                                |
| `zh-CN-XiaoxuanNeural`  | `role="YoungAdultFemale"`  | La voix imite celle d’une jeune femme adulte.                 |
|                         | `role="OlderAdultFemale"`  | La voix imite celle d’une femme adulte plus âgée.                 |
|                         | `role="OlderAdultMale"`    | La voix imite celle d’un homme adulte plus âgé.                   |

**Exemple**

Cet extrait de code SSML illustre la manière dont l’élément `<mstts:express-as>` est utilisé pour modifier le style oral en `cheerful`.

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
       xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
    <voice name="en-US-AriaNeural">
        <mstts:express-as style="cheerful">
            That'd be just amazing!
        </mstts:express-as>
    </voice>
</speak>
```

Cet extrait de code SSML illustre la manière dont l’attribut `styledegree` est utilisé pour modifier l’intensité du style d’élocution pour XiaoxiaoNeural.
```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
       xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="zh-CN">
    <voice name="zh-CN-XiaoxiaoNeural">
        <mstts:express-as style="sad" styledegree="2">
            快走吧，路上一定要注意安全，早去早回。
        </mstts:express-as>
    </voice>
</speak>
```

Cet extrait de code SSML illustre la manière dont l’attribut `role` est utilisé pour changer le jeu de rôle pour XiaomoNeural.
```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
       xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="zh-CN">
    <voice name="zh-CN-XiaomoNeural">
        女儿看见父亲走了进来，问道：
        <mstts:express-as role="YoungAdultFemale" style="calm">
            “您来的挺快的，怎么过来的？”
        </mstts:express-as>
        父亲放下手提包，说：
        <mstts:express-as role="OlderAdultMale" style="calm">
            “刚打车过来的，路上还挺顺畅。”
        </mstts:express-as>
    </voice>
</speak>
```

## <a name="add-or-remove-a-breakpause"></a>Ajouter ou supprimer une interruption/pause

Utilisez l’élément `break` pour insérer des pauses (ou des interruptions) entre des mots ou empêcher l’ajout automatique de pauses par le service de synthèse vocale.

> [!NOTE]
> Utilisez cet élément pour remplacer le comportement par défaut de la synthèse vocale (TTS) d’un mot ou d’une phrase dont le rendu en parole synthétisée ne semble pas naturel. Définissez `strength` sur `none` pour empêcher le service de synthèse vocale d’insérer automatiquement une interruption prosodique.

**Syntaxe**

```xml
<break strength="string" />
<break time="string" />
```

**Attributs**

| Attribut | Description | Obligatoire/facultatif |
|-----------|-------------|---------------------|
| `strength` | Spécifie la durée relative d’une pause à l’aide de l’une des valeurs suivantes :<ul><li>Aucun</li><li>x-weak</li><li>weak</li><li>medium (par défaut)</li><li>strong</li><li>x-strong</li></ul> | Facultatif |
| `time` | Spécifie la durée absolue d’une pause en secondes ou en millisecondes. Cette valeur doit être inférieure à 5 000 ms. Exemples de valeurs valides : `2s` et `500ms` | Facultatif |

| Puissance                      | Description |
|-------------------------------|-------------|
| Aucune, ou si aucune valeur fournie | 0 ms        |
| x-weak                        | 250 ms      |
| weak                          | 500 ms      |
| moyenne                        | 750 ms      |
| strong                        | 1000 ms     |
| x-strong                      | 1250 ms     |

**Exemple**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaNeural">
        Welcome to Microsoft Cognitive Services <break time="100ms" /> Text-to-Speech API.
    </voice>
</speak>
```
## <a name="add-silence"></a>Ajouter un silence

Utilisez l’élément `mstts:silence` pour insérer des pauses avant ou après le texte, ou entre 2 phrases adjacentes. 

> [!NOTE]
>La différence entre `mstts:silence` et `break` est que `break` peut être ajouté à n’importe quel endroit dans le texte, alors qu’un silence fonctionne uniquement au début ou à la fin du texte d’entrée, ou encore entre 2 phrases adjacentes.  


**Syntaxe**

```xml
<mstts:silence  type="string"  value="string"/>
```

**Attributs**

| Attribut | Description | Obligatoire/facultatif |
|-----------|-------------|---------------------|
| `type` | Spécifie l’emplacement auquel ajouter le silence : <ul><li>Leading (Début) : au début du texte </li><li>Tailing (Fin) : à la fin du texte </li><li>Sentenceboundary : entre des phrases adjacentes </li></ul> | Obligatoire |
| `Value` | Spécifie la durée absolue d’une pause en secondes ou en millisecondes. Cette valeur doit être inférieure à 5 000 ms. Exemples de valeurs valides : `2s` et `500ms` | Obligatoire |

**Exemple** : Dans cet exemple, `mtts:silence` est utilisé pour ajouter 200 ms de silence entre deux phrases.
```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">  
<voice name="en-US-AriaNeural"> 
<mstts:silence  type="Sentenceboundary" value="200ms"/> 
If we’re home schooling, the best we can do is roll with what each day brings and try to have fun along the way. 
A good place to start is by trying out the slew of educational apps that are helping children stay happy and smash their schooling at the same time. 
</voice> 
</speak> 
```

## <a name="specify-paragraphs-and-sentences"></a>Spécifier des paragraphes et des phrases

Les éléments `p` et `s` sont utilisés pour désigner respectivement des paragraphes et des phrases. En l’absence de ces éléments, le service de synthèse vocale détermine automatiquement la structure du document SSML.

L’élément `p` peut contenir du texte et les éléments suivants : `audio`, `break`, `phoneme`, `prosody`, `say-as`, `sub`, `mstts:express-as` et `s`.

L’élément `s` peut contenir du texte et les éléments suivants : `audio`, `break`, `phoneme`, `prosody`, `say-as`, `mstts:express-as` et `sub`.

**Syntaxe**

```XML
<p></p>
<s></s>
```

**Exemple**

```XML
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaRUS">
        <p>
            <s>Introducing the sentence element.</s>
            <s>Used to mark individual sentences.</s>
        </p>
        <p>
            Another simple paragraph.
            Sentence structure in this paragraph is not explicitly marked.
        </p>
    </voice>
</speak>
```

## <a name="use-phonemes-to-improve-pronunciation"></a>Utiliser des phonèmes pour améliorer la prononciation

L’élément `ph` est utilisé pour la prononciation phonétique dans des documents SSML. L’élément `ph` ne peut rien contenir d’autre que du texte. Fournissez toujours un discours contrôlable de visu comme solution de secours.

Les alphabets phonétiques sont constitués de phonèmes composés de lettres, de chiffres ou de caractères parfois combinés. Chaque phonème décrit un son vocal unique. Cela contraste avec l’alphabet latin où chaque lettre peut représenter plusieurs sons parlés. Considérez les différentes prononciations de la lettre « c » dans les mots « casser » et « cesser », ou les différentes prononciations de la combinaison de lettres « ch » dans les mots « chose » et « almanach ».

> [!NOTE]
> Actuellement, les étiquettes de phonèmes ne sont pas prises en charge pour ces 5 voix (et-EE-AnuNeural, ga-IE-OrlaNeural, lt-LT-OnaNeural, lv-LV-EveritaNeural et mt-MT-GarceNeural).

**Syntaxe**

```XML
<phoneme alphabet="string" ph="string"></phoneme>
```

**Attributs**

| Attribut | Description | Obligatoire/facultatif |
|-----------|-------------|---------------------|
| `alphabet` | Spécifie l’alphabet phonétique à utiliser lors de la synthèse de la prononciation de la chaîne dans l’attribut `ph`. La chaîne spécifiant l’alphabet doit être en lettres minuscules. Les alphabets que vous pouvez spécifier sont les suivants.<ul><li>`ipa` &ndash; <a href="https://en.wikipedia.org/wiki/International_Phonetic_Alphabet" target="_blank">Alphabet phonétique international <span class="docon docon-navigate-external x-hidden-focus"></span></a></li><li>`sapi` &ndash; [Alphabet phonétique du service Speech](speech-ssml-phonetic-sets.md)</li><li>`ups` &ndash;<a href="https://documentation.help/Microsoft-Speech-Platform-SDK-11/17509a49-cae7-41f5-b61d-07beaae872ea.htm" target="_blank"> Jeu de phonèmes universel</a></li></ul><br>L’alphabet s’applique uniquement au `phoneme` dans l’élément. | Facultatif |
| `ph` | Chaîne contenant des phonèmes spécifiant la prononciation du mot figurant dans l’élément `phoneme`. Si la chaîne spécifiée contient des phonèmes non reconnus, le service de synthèse vocale rejette la totalité du document SSML et ne produit aucune des sorties vocales spécifiées dans le document. | Requis en cas d’utilisation de phonèmes. |

**Exemples**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaRUS">
        <phoneme alphabet="ipa" ph="t&#x259;mei&#x325;&#x27E;ou&#x325;"> tomato </phoneme>
    </voice>
</speak>
```

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaRUS">
        <phoneme alphabet="sapi" ph="iy eh n y uw eh s"> en-US </phoneme>
    </voice>
</speak>
```

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaRUS">
        <s>His name is Mike <phoneme alphabet="ups" ph="JH AU"> Zhou </phoneme></s>
    </voice>
</speak>
```

## <a name="use-custom-lexicon-to-improve-pronunciation"></a>Utiliser un lexique personnalisé pour améliorer la prononciation

Parfois, le service de synthèse vocale ne peut pas prononcer un mot de façon correcte. Par exemple, le nom d’une société ou un terme médical. Les développeurs peuvent définir le mode de lecture des entités uniques dans SSML à l’aide des balises `phoneme` et `sub`. Toutefois, si vous devez définir le mode de lecture de plusieurs entités, vous pouvez créer un lexique personnalisé à l’aide de la balise `lexicon`.

> [!NOTE]
> Le lexique personnalisé prend actuellement en charge l’encodage UTF-8. 

> [!NOTE]
> Actuellement, le lexique personnalisé n’est pas pris en charge pour ces 5 voix (t-EE-AnuNeural, ga-IE-OrlaNeural, lt-LT-OnaNeural, lv-LV-EveritaNeural et mt-MT-GarceNeural).


**Syntaxe**

```XML
<lexicon uri="string"/>
```

**Attributs**

| Attribut | Description                               | Obligatoire/facultatif |
|-----------|-------------------------------------------|---------------------|
| `uri`     | Adresse du document PLS externe. | Obligatoire.           |

**Utilisation**

Pour définir le mode de lecture de plusieurs entités, vous pouvez créer un lexique personnalisé, qui est stocké sous forme de fichier .xml ou .pls. Voici un exemple de fichier .xml.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<lexicon version="1.0" 
      xmlns="http://www.w3.org/2005/01/pronunciation-lexicon"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
      xsi:schemaLocation="http://www.w3.org/2005/01/pronunciation-lexicon 
        http://www.w3.org/TR/2007/CR-pronunciation-lexicon-20071212/pls.xsd"
      alphabet="ipa" xml:lang="en-US">
  <lexeme>
    <grapheme>BTW</grapheme> 
    <alias>By the way</alias> 
  </lexeme>
  <lexeme>
    <grapheme> Benigni </grapheme> 
    <phoneme> bɛˈniːnji</phoneme>
  </lexeme>
</lexicon>
```

L’élément `lexicon` contient au moins un élément `lexeme`. Chaque élément `lexeme` contient au moins un élément `grapheme` et un ou plusieurs éléments `grapheme`, `alias` et `phoneme`. L’élément `grapheme` contient le texte décrivant l’<a href="https://www.w3.org/TR/pronunciation-lexicon/#term-Orthography" target="_blank">orthographe <span class="docon docon-navigate-external x-hidden-focus"></span></a>. Les éléments `alias` sont utilisés pour indiquer la prononciation d’un acronyme ou d’un terme abrégé. L’élément `phoneme` fournit du texte décrivant la façon dont le `lexeme` est prononcé.

Il est important de noter que vous ne pouvez pas définir directement la prononciation d’une phrase à l’aide du lexique personnalisé. Si vous devez définir la prononciation d’un acronyme ou d’un terme abrégé, fournissez d’abord un `alias`, puis associez `phoneme` à `alias`. Par exemple :

```xml
  <lexeme>
    <grapheme>Scotland MV</grapheme> 
    <alias>ScotlandMV</alias> 
  </lexeme>
  <lexeme>
    <grapheme>ScotlandMV</grapheme> 
    <phoneme>ˈskɒtlənd.ˈmiːdiəm.weɪv</phoneme>
  </lexeme>
```

Vous pouvez également fournir directement l’`alias` attendu pour l’acronyme ou le terme abrégé. Par exemple :
```xml
  <lexeme>
    <grapheme>Scotland MV</grapheme> 
    <alias>Scotland Media Wave</alias> 
  </lexeme>
```

> [!IMPORTANT]
> L’élément `phoneme` ne peut pas contenir d’espaces pour l’utilisation de l’IPA.

Pour plus d’informations sur le fichier de lexique personnalisé, consultez [Pronunciation Lexicon Specification (PLS) Version 1.0](https://www.w3.org/TR/pronunciation-lexicon/).

Ensuite, publiez votre fichier de lexique personnalisé. Bien qu’il n’y ait aucune restriction sur l’emplacement de ce fichier, nous vous recommandons d’utiliser le [Stockage Blob Azure](../../storage/blobs/storage-quickstart-blobs-portal.md).

Une fois que vous avez publié votre lexique personnalisé, vous pouvez le référencer à partir de votre SSML.

> [!NOTE]
> L’élément `lexicon` doit être dans l’élément `voice`.

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" 
          xmlns:mstts="http://www.w3.org/2001/mstts" 
          xml:lang="en-US">
    <voice name="en-US-AriaRUS">
        <lexicon uri="http://www.example.com/customlexicon.xml"/>
        BTW, we will be there probably at 8:00 tomorrow morning.
        Could you help leave a message to Robert Benigni for me?
    </voice>
</speak>
```

Lors de l’utilisation de ce lexique personnalisé, « BTW » est prononcé comme « by the way ». « Benigni » sera prononcé « bɛˈniːnji » avec l’alphabet phonétique international fourni.  

**Limitations**
- Taille du fichier : la limite maximale de la taille du fichier de lexique personnalisé est de 100 Ko. Au-delà, la demande de synthèse échoue.
- Actualisation du cache du lexique : le lexique personnalisé est mis en cache avec l’URI en tant que clé sur le service TTS lorsqu’il est chargé pour la première fois. Un lexique avec le même URI ne sera pas rechargé dans un délai de 15 minutes. Il est donc nécessaire de patienter au maximum 15 minutes avant qu’une modification du lexique personnalisé ne prenne effet.

**Jeux de phonèmes du service Speech**

Dans l’exemple ci-dessus, nous utilisons l’alphabet phonétique international, connu également sous le nom de jeu de phonèmes API. Nous suggérons aux développeurs d’utiliser l’API, car il s’agit de la norme internationale. Pour certains caractères de l’alphabet phonétique international, il existe une version « précomposée » et un version « décomposée » lorsqu’ils sont représentés en Unicode. Le lexique personnalisé ne prend en charge que les Unicode décomposés.

Sachant que l’API n’est pas facile à mémoriser, le service Speech définit un jeu de phonèmes pour sept langues (`en-US`, `fr-FR`, `de-DE`, `es-ES`, `ja-JP`, `zh-CN` et `zh-TW`).

Vous pouvez utiliser `sapi` comme valeur pour l’attribut `alphabet` avec des lexiques personnalisés, comme illustré ci-dessous :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<lexicon version="1.0" 
      xmlns="http://www.w3.org/2005/01/pronunciation-lexicon"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.w3.org/2005/01/pronunciation-lexicon
        http://www.w3.org/TR/2007/CR-pronunciation-lexicon-20071212/pls.xsd"
      alphabet="sapi" xml:lang="en-US">
  <lexeme>
    <grapheme>BTW</grapheme>
    <alias> By the way </alias>
  </lexeme>
  <lexeme>
    <grapheme> Benigni </grapheme>
    <phoneme> b eh 1 - n iy - n y iy </phoneme>
  </lexeme>
</lexicon>
```

Pour plus d’informations sur l’alphabet phonétique détaillé du service Speech, reportez-vous aux [jeux de phonèmes du service Speech](speech-ssml-phonetic-sets.md).

## <a name="adjust-prosody"></a>Ajuster la prosodie

L’élément `prosody` est utilisé pour spécifier des modifications apportées au timbre, au contour, à la tessiture, au débit, à la durée et au volume de la sortie de synthèse vocale. L’élément `prosody` peut contenir du texte et les éléments suivants : `audio`, `break`, `p`, `phoneme`, `prosody`, `say-as`, `sub` et `s`.

Étant donné que les valeurs d’attribut prosodique peuvent varier sur une vaste plage, le module de reconnaissance vocale interprète les valeurs affectées comme une suggestion de ce que les valeurs prosodiques réelles de la voix sélectionnée devraient être. Le service de synthèse vocale limite ou remplace les valeurs non prises en charge. Des valeurs non prises en charge sont, par exemple, une tonalité de 1 MHz ou un volume de 120.

**Syntaxe**

```XML
<prosody pitch="value" contour="value" range="value" rate="value" duration="value" volume="value"></prosody>
```

**Attributs**

| Attribut | Description | Obligatoire/facultatif |
|-----------|-------------|---------------------|
| `pitch` | Indique la tonalité de base pour le texte. Vous pouvez spécifier la tonalité comme suit :<ul><li>Valeur absolue, exprimée sous la forme d’un nombre suivi de « Hz » (Hertz). Par exemple : `<prosody pitch="600Hz">some text</prosody>`.</li><li>Valeur relative, exprimée sous la forme d’un nombre précédé du signe « + » ou « - » et suivi de « Hz » ou « st », qui spécifie l’importance d’un changement de tonalité. Par exemple, `<prosody pitch="+80Hz">some text</prosody>` ou `<prosody pitch="-2st">some text</prosody>`. « st » indique que l’unité de changement est le demi-ton, c’est-à-dire la moitié d’un ton sur l’échelle diatonique standard.</li><li>Valeur constante :<ul><li>x-low</li><li>low</li><li>moyenne</li><li>high</li><li>x-high</li><li>default</li></ul></li></ul> | Facultatif |
| `contour` |Le contour prend désormais en charge les voix neurales et standard. Le contour représente les changements de tonalité. Ces changements sont représentés sous la forme d’un tableau de cibles aux positions temporelles spécifiées dans la sortie vocale. Chaque cible est définie par des ensembles de paires de paramètres. Par exemple : <br/><br/>`<prosody contour="(0%,+20Hz) (10%,-2st) (40%,+10Hz)">`<br/><br/>La première valeur dans chaque paire de paramètres spécifie l’emplacement du changement de tonalité sous la forme d’un pourcentage de la durée du texte. La deuxième valeur spécifie la quantité de hausse ou de baisse de la tonalité, à l’aide d’une valeur relative ou une valeur d’énumération pour la tonalité (voir `pitch`). | Facultatif |
| `range` | Valeur représentant la plage de tonalités pour le texte. Vous pouvez exprimer `range` à l’aide des mêmes valeurs absolues, relatives ou d’énumération que celles utilisées pour décrire `pitch`. | Facultatif |
| `rate` | Indique la cadence d’énonciation du texte. Vous pouvez exprimer `rate` comme suit :<ul><li>Valeur relative exprimée sous forme de nombre agissant comme multiplicateur de la valeur par défaut. Par exemple, la valeur *1* n’entraîne aucun changement de cadence. La valeur *0.5* entraîne une réduction de moitié de la cadence. La valeur *3* entraîne un triplement de la cadence.</li><li>Valeur constante :<ul><li>x-slow</li><li>slow</li><li>moyenne</li><li>fast</li><li>x-fast</li><li>default</li></ul></li></ul> | Facultatif |
| `duration` | Période de temps qui doit s’écouler pendant que le service de synthèse vocale (TTS) lit le texte, exprimée en secondes ou millisecondes. Par exemple, *2 s* ou *1800 ms*. La durée prend uniquement en charge les voix standard.| Facultatif |
| `volume` | Indique le niveau de volume de la voix. Vous pouvez exprimer le volume comme suit :<ul><li>Valeur absolue, exprimée sous la forme d’un nombre dans la plage de 0,0 à 100,0, du *plus bas* au *plus fort*. Par exemple, 75. La valeur par défaut est 100,0.</li><li>Valeur relative, exprimée sous la forme d’un nombre précédé du signe « + » ou « - » spécifie une quantité de changement de volume. Par exemple, +10 ou -5.5.</li><li>Valeur constante :<ul><li>silent</li><li>x-soft</li><li>soft</li><li>moyenne</li><li>loud</li><li>x-loud</li><li>default</li></ul></li></ul> | Facultatif |

### <a name="change-speaking-rate"></a>Modifier le débit

La cadence d’élocution peut s’appliquer aux voix neurales et standard au niveau de la phrase ou du mot. 

**Exemple**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-GuyNeural">
        <prosody rate="+30.00%">
            Welcome to Microsoft Cognitive Services Text-to-Speech API.
        </prosody>
    </voice>
</speak>
```

### <a name="change-volume"></a>Modifier le volume

Les modifications de volume peuvent s’appliquer aux voix standard au niveau de la phrase ou du mot. Cependant, les modifications de volume ne s’appliquent aux voix neuronales qu’au niveau de la phrase.

**Exemple**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaRUS">
        <prosody volume="+20.00%">
            Welcome to Microsoft Cognitive Services Text-to-Speech API.
        </prosody>
    </voice>
</speak>
```

### <a name="change-pitch"></a>Modifier la tonalité

Les modifications de ton peuvent s’appliquer aux voix standard au niveau de la phrase ou du mot. Cependant, les modifications de ton ne s’appliquent aux voix neuronales qu’au niveau de la phrase.

**Exemple**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-Guy24kRUS">
        Welcome to <prosody pitch="high">Microsoft Cognitive Services Text-to-Speech API.</prosody>
    </voice>
</speak>
```

### <a name="change-pitch-contour"></a>Modifier le contour intonatif

> [!IMPORTANT]
> Les changements de contour de ton sont désormais pris en charge avec les voix neurales.

**Exemple**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaNeural">
        <prosody contour="(60%,-60%) (100%,+80%)" >
            Were you the only person in the room? 
        </prosody>
    </voice>
</speak>
```
## <a name="say-as-element"></a>élément say-as

`say-as` est un élément facultatif qui indique le type de contenu (nombre ou date, par exemple) du texte de l’élément. Il fournit des conseils au moteur de synthèse vocale sur la manière de prononcer le texte.

**Syntaxe**

```XML
<say-as interpret-as="string" format="digit string" detail="string"> <say-as>
```

**Attributs**

| Attribut | Description | Obligatoire/facultatif |
|-----------|-------------|---------------------|
| `interpret-as` | Indique le type de contenu du texte de l’élément. Pour obtenir la liste des types, consultez le tableau ci-dessous. | Obligatoire |
| `format` | Fournit des informations supplémentaires sur la mise en forme précise du texte de l’élément pour les types de contenu susceptibles de présenter des formats ambigus. SSML définit les formats des types de contenu qui les utilisent (voir le tableau ci-dessous). | Facultatif |
| `detail` | Indique le niveau de détail à prononcer. Par exemple, cet attribut peut demander à ce que le moteur de synthèse vocale prononce les signes de ponctuation. Aucune valeur standard n’est définie pour `detail`. | Facultatif |

<!-- I don't understand the last sentence. Don't we know which one Cortana uses? -->

Les types de contenu suivants sont pris en charge pour les attributs `interpret-as` et `format`. Incluez l’attribut `format` uniquement si `interpret-as` est défini sur date et heure.

| interpret-as | format | Interprétation |
|--------------|--------|----------------|
| `address` | | Le texte est prononcé sous forme d'adresse. Le moteur de synthèse vocale prononce :<br /><br />`I'm at <say-as interpret-as="address">150th CT NE, Redmond, WA</say-as>`<br /><br />Par exemple, « Je suis au 150e court nord est redmond washington. » |
| `cardinal`, `number` | | Le texte est prononcé sous forme de nombre cardinal. Le moteur de synthèse vocale prononce :<br /><br />`There are <say-as interpret-as="cardinal">3</say-as> alternatives`<br /><br />Par exemple, « Il existe trois alternatives ». |
| `characters`, `spell-out` | | Le texte est prononcé sous forme de lettres individuelles (épelées). Le moteur de synthèse vocale prononce :<br /><br />`<say-as interpret-as="characters">test</say-as>`<br /><br />Par exemple, « T E S T ». |
| `date` | dmy, mdy, ymd, ydm, ym, my, md, dm, d, m, y | Le texte est prononcé sous forme de date. L’attribut `format` spécifie le format de la date (*j=day (jour), m=month (mois) et y=year (année)* ). Le moteur de synthèse vocale prononce :<br /><br />`Today is <say-as interpret-as="date" format="mdy">10-19-2016</say-as>`<br /><br />Par exemple, « Nous sommes le 19 octobre 2016 ». |
| `digits`, `number_digit` | | Le texte est prononcé sous forme de séquence de chiffres individuels. Le moteur de synthèse vocale prononce :<br /><br />`<say-as interpret-as="number_digit">123456789</say-as>`<br /><br />Par exemple, « 1 2 3 4 5 6 7 8 9 ». |
| `fraction` | | Le texte est prononcé sous forme de nombre fractionnaire. Le moteur de synthèse vocale prononce :<br /><br /> `<say-as interpret-as="fraction">3/8</say-as> of an inch`<br /><br />Par exemple, « Trois huitièmes de pouce ». |
| `ordinal` | | Le texte est prononcé sous forme de nombre ordinal. Le moteur de synthèse vocale prononce :<br /><br />`Select the <say-as interpret-as="ordinal">3rd</say-as> option`<br /><br />Par exemple, « Sélectionnez la troisième option ». |
| `telephone` | | Le texte est prononcé sous forme de numéro de téléphone. L’attribut `format` peut contenir des chiffres correspondant à l’indicatif d’un pays. Par exemple, « 1 » pour les États-Unis ou « 39 » pour l’Italie. Le moteur de synthèse vocale peut utiliser ces informations pour guider la prononciation d’un numéro de téléphone. Le numéro de téléphone peut également inclure l’indicatif du pays qui, le cas échéant, est prioritaire sur l’indicatif du pays dans `format`. Le moteur de synthèse vocale prononce :<br /><br />`The number is <say-as interpret-as="telephone" format="1">(888) 555-1212</say-as>`<br /><br />Par exemple, « Mon numéro avec indicatif régional est huit huit huit cinq cinq cinq un deux un deux ». |
| `time` | hms12, hms24 | Le texte est prononcé sous forme d'heure. L’attribut `format` indique si l’heure correspond à l'horloge de 12 heures (hms12) ou 24 heures (hms24). Utilisez deux points pour séparer les nombres représentant les heures, les minutes et les secondes. Voici quelques exemples d'heure valides : 12:35, 1:14:32, 08:15 et 02:50:45. Le moteur de synthèse vocale prononce :<br /><br />`The train departs at <say-as interpret-as="time" format="hms12">4:00am</say-as>`<br /><br />Par exemple, « Le train part à 4 heures ». |

**Utilisation**

L’élément `say-as` peut uniquement contenir du texte.

**Exemple**

Le moteur de synthèse vocale prononce l’exemple ci-dessous comme suit : « Votre première requête portait sur une chambre le dix-neuf octobre deux mille dix, avec une arrivée à douze heure trente-cinq. »
 
```XML
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaRUS">
        <p>
        Your <say-as interpret-as="ordinal"> 1st </say-as> request was for <say-as interpret-as="cardinal"> 1 </say-as> room
        on <say-as interpret-as="date" format="mdy"> 10/19/2010 </say-as>, with early arrival at <say-as interpret-as="time" format="hms12"> 12:35pm </say-as>.
        </p>
    </voice>
</speak>
```

## <a name="add-recorded-audio"></a>Ajouter un audio enregistré

`audio` est un élément facultatif qui vous permet d’insérer un audio MP3 dans un document SSML. Le corps de l’élément audio peut contenir du texte brut ou un balisage SSML qui est prononcé si le fichier audio n’est pas disponible ou ne peut pas être lu. De plus, l’élément `audio` peut contenir du texte et les éléments suivants : `audio`, `break`, `p`, `s`, `phoneme`, `prosody`, `say-as` et `sub`.

Tout audio inclus dans le document SSML doit respecter les exigences suivantes :

* Le MP3 doit être hébergé sur un point de terminaison HTTPS accessible via Internet. HTTPS est requis et le domaine qui héberge le fichier MP3 doit présenter un certificat TLS/SSL approuvé valide.
* Le MP3 doit être un fichier MP3 valide (MPEG v2).
* La vitesse de transmission doit être de 48 Kbits/s.
* L’échantillonnage doit être de 16 000 Hz.
* La durée totale cumulée de tous les fichiers texte et audio dans une réponse unique ne peut pas dépasser 90 secondes.
* Le MP3 ne doit pas contenir d’informations propres à un client ou d’autres informations sensibles.

**Syntaxe**

```xml
<audio src="string"/></audio>
```

**Attributs**

| Attribut | Description                                   | Obligatoire/facultatif                                        |
|-----------|-----------------------------------------------|------------------------------------------------------------|
| `src`     | Spécifie l’emplacement/URL du fichier audio. | Obligatoire en cas d’utilisation de l’élément audio dans votre document SSML. |

**Exemple**

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-AriaRUS">
        <p>
            <audio src="https://contoso.com/opinionprompt.wav"/>
            Thanks for offering your opinion. Please begin speaking after the beep.
            <audio src="https://contoso.com/beep.wav">
                Could not play the beep, please voice your opinion now.
            </audio>
        </p>
    </voice>
</speak>
```

## <a name="add-background-audio"></a>Ajouter un arrière-plan audio

L’élément `mstts:backgroundaudio` vous permet d’ajouter de l’audio en arrière-plan à vos documents SSML (ou de combiner un fichier audio avec la synthèse vocale). Avec `mstts:backgroundaudio`, vous pouvez exécuter en boucle un fichier audio en arrière-plan, l’estomper au début de la synthèse vocale et l’estomper à la fin de la synthèse vocale.

Si l’audio en arrière-plan fourni est plus court que la synthèse vocale ou la diminution du son, il est répété en boucle. S’il est plus long que la synthèse vocale, il s’arrête lorsque la diminution du son est terminée.

Un seul fichier audio en arrière-plan est autorisé par document SSML. Toutefois, vous pouvez intercaler des balises `audio` dans l’élément `voice` pour ajouter de l’audio supplémentaire à votre document SSML.

**Syntaxe**

```XML
<mstts:backgroundaudio src="string" volume="string" fadein="string" fadeout="string"/>
```

**Attributs**

| Attribut | Description | Obligatoire/facultatif |
|-----------|-------------|---------------------|
| `src` | Spécifie l’emplacement/URL du fichier audio en arrière-plan. | Obligatoire en cas d’utilisation de l’audio en arrière-plan dans votre document SSML. |
| `volume` | Spécifie le volume du fichier audio en arrière-plan. Les **valeurs acceptées** vont de `0` à `100` inclus. La valeur par défaut est `1`. | Facultatif |
| `fadein` | Spécifie la durée de l’apparition en fondu de l’audio d’arrière-plan, en millisecondes. La valeur par défaut est `0`, ce qui équivaut à aucune apparition en fondu audio. Les **valeurs acceptées** vont de `0` à `10000` inclus.  | Facultatif |
| `fadeout` | Spécifie la durée de la disparition en fondu de l’audio d’arrière-plan, en millisecondes. La valeur par défaut est `0`, ce qui équivaut à aucune disparition en fondu audio. Les **valeurs acceptées** vont de `0` à `10000` inclus.  | Facultatif |

**Exemple**

```xml
<speak version="1.0" xml:lang="en-US" xmlns:mstts="http://www.w3.org/2001/mstts">
    <mstts:backgroundaudio src="https://contoso.com/sample.wav" volume="0.7" fadein="3000" fadeout="4000"/>
    <voice name="Microsoft Server Speech Text to Speech Voice (en-US, AriaRUS)">
        The text provided in this document will be spoken over the background audio.
    </voice>
</speak>
```

## <a name="next-steps"></a>Étapes suivantes

* [Prise en charge linguistique : voix, paramètres régionaux, langues](language-support.md)
