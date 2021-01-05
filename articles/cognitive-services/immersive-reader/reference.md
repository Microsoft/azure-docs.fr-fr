---
title: Référence du kit de développement logiciel (SDK) du lecteur immersif
titleSuffix: Azure Cognitive Services
description: Le kit SDK du Lecteur immersif contient une bibliothèque JavaScript qui vous permet d’intégrer le Lecteur immersif à votre application.
services: cognitive-services
author: metanMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: reference
ms.date: 06/20/2019
ms.author: metang
ms.openlocfilehash: f2f5c8193454a3b7fa6be1cea7a1236b613d6c8f
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92636525"
---
# <a name="immersive-reader-javascript-sdk-reference-v11"></a>Référence SDK JavaScript du lecteur immersif (v1.1)

Le kit SDK du Lecteur immersif contient une bibliothèque JavaScript qui vous permet d’intégrer le Lecteur immersif à votre application.

## <a name="functions"></a>Fonctions

Le SDK expose les fonctions suivantes :

- [`ImmersiveReader.launchAsync(token, subdomain, content, options)`](#launchasync)

- [`ImmersiveReader.close()`](#close)

- [`ImmersiveReader.renderButtons(options)`](#renderbuttons)

<br>

## <a name="launchasync"></a>launchAsync

Lance le lecteur immersif au sein d'un `iframe` de votre application web. Notez que la taille de votre contenu est limitée à un maximum de 50 Mo.

```typescript
launchAsync(token: string, subdomain: string, content: Content, options?: Options): Promise<LaunchResponse>;
```

#### <a name="launchasync-parameters"></a>Paramètres launchAsync

| Nom | Type | Description |
| ---- | ---- |------------ |
| `token` | string | Le jeton d’authentification Azure AD. Pour plus d’informations, consultez [Comment créer une ressource de Lecteur immersif](./how-to-create-immersive-reader.md). |
| `subdomain` | string | Sous-domaine personnalisé de votre ressource Lecteur immersif dans Azure. Pour plus d’informations, consultez [Comment créer une ressource de Lecteur immersif](./how-to-create-immersive-reader.md). |
| `content` | [Contenu](#content) | Objet dans lequel figure le contenu à afficher dans le lecteur immersif. |
| `options` | [Options](#options) | Options de configuration de certains comportements du lecteur immersif. facultatif. |

#### <a name="returns"></a>Retours

Retourne un `Promise<LaunchResponse>` qui se résout quand le Lecteur immersif est chargé. Le `Promise` est résolu en objet [`LaunchResponse`](#launchresponse).

#### <a name="exceptions"></a>Exceptions

L'élément `Promise` retourné est rejeté avec un objet [`Error`](#error) si le lecteur immersif ne se charge pas. Pour plus d’informations, consultez les [codes d’erreur](#error-codes).

<br>

## <a name="close"></a>close

Ferme le Lecteur immersif.

Par exemple, cette fonction est utile si ```hideExitButton: true``` est défini dans [options](#options) pour masquer le bouton Quitter. Un autre bouton (par exemple, la flèche retour d’un en-tête mobile) peut appeler cette fonction ```close``` quand l’utilisateur clique dessus.

```typescript
close(): void;
```

<br>

## <a name="immersive-reader-launch-button"></a>Bouton de lancement du Lecteur immersif

Le kit de développement logiciel (SDK) fournit le style par défaut du bouton de lancement du lecteur immersif. Utilisez l'attribut de classe `immersive-reader-button` pour activer ce style. Pour plus d’informations, consultez [Comment personnaliser le bouton Lecteur immersif](./how-to-customize-launch-button.md).

```html
<div class='immersive-reader-button'></div>
```

#### <a name="optional-attributes"></a>Attributs facultatifs

Utilisez les attributs suivants pour configurer l’apparence du bouton.

| Attribut | Description |
| --------- | ----------- |
| `data-button-style` | Définit le style du bouton. Peut être `icon`, `text` ou `iconAndText`. La valeur par défaut est `icon`. |
| `data-locale` | Définit les paramètres régionaux. Par exemple, `en-US` ou `fr-FR`. La valeur par défaut est l’anglais (`en`). |
| `data-icon-px-size` | Définit la taille de l’icône en pixels. La valeur par défaut est 20px. |

<br>

## <a name="renderbuttons"></a>renderButtons

La fonction ```renderButtons``` n’est pas nécessaire si vous utilisez les conseils figurant dans [Comment personnaliser le bouton Lecteur immersif](./how-to-customize-launch-button.md).

Cette fonction définit un style et met à jour les boutons du Lecteur immersif du document. Si ```options.elements``` est fourni, les boutons seront affichés dans chaque élément spécifié dans ```options.elements```. L’utilisation du paramètre ```options.elements``` est utile lorsque vous avez plusieurs sections dans votre document où lancer le Lecteur immersif, et que vous souhaitez bénéficier d’une méthode simplifiée pour afficher plusieurs boutons avec le même style, ou que vous souhaitez afficher les boutons selon un modèle de conception simple et cohérent. Pour utiliser cette fonction avec le paramètre des [options renderButtons](#renderbuttons-options), appelez ```ImmersiveReader.renderButtons(options: RenderButtonsOptions);``` lors du chargement de la page, comme illustré dans l’extrait de code ci-dessous. Sinon, les boutons sont affichés dans les éléments du document qui ont la classe ```immersive-reader-button``` comme illustré dans [Comment personnaliser le bouton Lecteur immersif](./how-to-customize-launch-button.md).

```typescript
// This snippet assumes there are two empty div elements in
// the page HTML, button1 and button2.
const btn1: HTMLDivElement = document.getElementById('button1');
const btn2: HTMLDivElement = document.getElementById('button2');
const btns: HTMLDivElement[] = [btn1, btn2];
ImmersiveReader.renderButtons({elements: btns});
```

Reportez-vous à [Attributs optionnels](#optional-attributes) ci-dessus, pour plus d’options de rendu. Pour utiliser ces options, ajoutez l’un des attributs optionnels à chaque ```HTMLDivElement``` dans le code HTML de votre page.

```typescript
renderButtons(options?: RenderButtonsOptions): void;
```

#### <a name="renderbuttons-parameters"></a>Paramètres renderButtons

| Nom | Type | Description |
| ---- | ---- |------------ |
| `options` | [Options renderButtons](#renderbuttons-options) | Options de configuration de certains comportements de la fonction renderButtons. facultatif. |

### <a name="renderbuttons-options"></a>Options renderButtons

Options de rendu des boutons du Lecteur immersif.

```typescript
{
    elements: HTMLDivElement[];
}
```

#### <a name="renderbuttons-options-parameters"></a>Paramètres d’options renderButtons

| Paramètre | Type | Description |
| ------- | ---- | ----------- |
| elements | HTMLDivElement[] | Éléments dans lesquels afficher les boutons du Lecteur immersif. |

##### `elements`
```Parameters
Type: HTMLDivElement[]
Required: false
```

<br>

## <a name="launchresponse"></a>LaunchResponse

Contient la réponse de l’appel à `ImmersiveReader.launchAsync`. Notez qu’une référence à `iframe` qui contient le lecteur immersif est accessible via `container.firstChild`.

```typescript
{
    container: HTMLDivElement;
    sessionId: string;
}
```

#### <a name="launchresponse-parameters"></a>Paramètres LaunchResponse

| Paramètre | Type | Description |
| ------- | ---- | ----------- |
| conteneur | HTMLDivElement | Élément HTML qui contient l’iframe du Lecteur immersif. |
| sessionID | Chaîne | Identificateur global unique de cette session, utilisé pour le débogage. |
 
## <a name="error"></a>Error

Contient des informations sur une erreur.

```typescript
{
    code: string;
    message: string;
}
```

#### <a name="error-parameters"></a>Paramètres d'erreur

| Paramètre | Type | Description |
| ------- | ---- | ----------- |
| code | String | Un code d’erreur dans un ensemble. Consultez [Codes d’erreur](#error-codes). |
| message | Chaîne | Représentation contrôlable de visu de l’erreur. |

#### <a name="error-codes"></a>Codes d’erreur

| Code | Description |
| ---- | ----------- |
| BadArgument | L’argument fourni est non valide, consultez le paramètre `message` de l’[erreur](#error). |
| Délai d'expiration | Le lecteur immersif n'a pas été chargé dans le délai spécifié. |
| TokenExpired | Le jeton fourni a expiré. |
| Throttled | Le taux d’appels maximal a été dépassé. |

<br>

## <a name="types"></a>Types

### <a name="content"></a>Contenu

Contient le contenu à afficher dans le lecteur immersif.

```typescript
{
    title?: string;
    chunks: Chunk[];
}
```

#### <a name="content-parameters"></a>Paramètres de contenu

| Nom | Type | Description |
| ---- | ---- |------------ |
| title | String | Texte de titre affiché en haut du Lecteur immersif (facultatif) |
| segments | [Chunk[]](#chunk) | Tableau de segments |

##### `title`
```Parameters
Type: String
Required: false
Default value: "Immersive Reader" 
```

##### `chunks`
```Parameters
Type: Chunk[]
Required: true
Default value: null 
```

<br>

### <a name="chunk"></a>Bloc

Bloc de données unique qui est passé dans le contenu du Lecteur immersif.

```typescript
{
    content: string;
    lang?: string;
    mimeType?: string;
}
```

#### <a name="chunk-parameters"></a>Paramètres de segment

| Nom | Type | Description |
| ---- | ---- |------------ |
| contenu | Chaîne | Chaîne qui contient le contenu envoyé au Lecteur immersif. |
| lang | Chaîne | Langue du texte. La valeur est au format d’étiquette d’identification de langue de la recommandation BCP 47 de l’IETF, par exemple en, es-ES. La langue est détectée automatiquement si elle n’est pas spécifiée. Voir [Langues prises en charge](#supported-languages). |
| mimeType | string | Les formats de texte brut, MathML, HTML et Microsoft Word DOCX sont pris en charge. Pour plus d’informations, consultez [Types MIME pris en charge](#supported-mime-types). |

##### `content`
```Parameters
Type: String
Required: true
Default value: null 
```

##### `lang`
```Parameters
Type: String
Required: false
Default value: Automatically detected 
```

##### `mimeType`
```Parameters
Type: String
Required: false
Default value: "text/plain"
```

#### <a name="supported-mime-types"></a>Types MIME pris en charge

| Type MIME | Description |
| --------- | ----------- |
| texte/brut | Texte brut. |
| texte/html | Contenu HTML. [En savoir plus](#html-support)|
| application/mathml+xml | Langage de balisage mathématique (MathML). [Plus d’informations](./how-to/display-math.md)
| application/vnd.openxmlformats-officedocument.wordprocessingml.document | Document au format .docx Microsoft Word.


<br>

## <a name="options"></a>Options

Contient les propriétés qui configurent certains comportements du lecteur immersif.

```typescript
{
    uiLang?: string;
    timeout?: number;
    uiZIndex?: number;
    useWebview?: boolean;
    onExit?: () => any;
    allowFullscreen?: boolean;
    hideExitButton?: boolean;
    cookiePolicy?: CookiePolicy;
    disableFirstRun?: boolean;
    readAloudOptions?: ReadAloudOptions;
    translationOptions?: TranslationOptions;
    displayOptions?: DisplayOptions;
    preferences?: string;
    onPreferencesChanged?: (value: string) => any;
    customDomain?: string;
}
```

#### <a name="options-parameters"></a>Paramètres d’options

| Nom | Type | Description |
| ---- | ---- |------------ |
| uiLang | Chaîne | Langue de l’interface utilisateur. La valeur est au format d’étiquette d’identification de langue de la recommandation BCP 47 de l’IETF, par exemple en, es-ES. Si elle n’est pas spécifiée, la langue du navigateur est définie par défaut. |
| délai d'expiration | Numéro | Durée (en millisecondes) avant l’échec de [launchAsync](#launchasync) avec une erreur de délai d’attente (15 000 ms par défaut). Ce délai d’attente s’applique uniquement au lancement initial de la page du Lecteur, où la réussite est observée lorsque la page du Lecteur s’ouvre et que le compteur démarre. L’ajustement du délai d’attente ne devrait pas être nécessaire. |
| uiZIndex | Numéro | Index-Z de l’iframe qui sera créé (la valeur par défaut est 1 000). |
| useWebview | Boolean| Utiliser une balise de vue web à la place d’un iframe, à des fins de compatibilité avec les applications Chrome (la valeur par défaut est false). |
| onExit | Fonction | S’exécute à la fermeture du Lecteur immersif. |
| allowFullscreen | Boolean | Possibilité de basculer en plein écran (la valeur par défaut est true). |
| hideExitButton | Boolean | Indique s’il faut ou non masquer la flèche du bouton de sortie du Lecteur immersif (la valeur par défaut est false). Cette valeur doit être true uniquement si un autre mécanisme est fourni pour quitter le Lecteur immersif (par exemple, la flèche vers la gauche d’une barre d’outils mobile). |
| cookiePolicy | [CookiePolicy](#cookiepolicy-options) | Paramètre d’utilisation des cookies du Lecteur immersif (la valeur par défaut est *CookiePolicy.Disable* ). Il incombe à l’application hôte d’obtenir les éventuels consentements nécessaires de l’utilisateur conformément à la stratégie de conformité des cookies de l’Union européenne. Consultez [Options de CookiePolicy](#cookiepolicy-options). |
| disableFirstRun | Boolean | Désactiver l’expérience de première exécution. |
| readAloudOptions | [ReadAloudOptions](#readaloudoptions) | Options de configuration de la lecture à voix haute. |
| translationOptions | [TranslationOptions](#translationoptions) | Options de configuration de la traduction. |
| displayOptions | [DisplayOptions](#displayoptions) | Options de configuration de la taille du texte, de la police, etc. |
| preferences | Chaîne | Chaîne retournée par onPreferencesChanged représentant les préférences de l’utilisateur dans le Lecteur immersif. Pour plus d’informations, consultez [Paramètres des réglages](#settings-parameters) et [Procédure – Stocker les préférences de l’utilisateur](./how-to-store-user-preferences.md). |
| onPreferencesChanged | Fonction | S’exécute lorsque les préférences de l’utilisateur ont changé. Pour plus d’informations, consultez [Procédure – Stocker les préférences de l’utilisateur](./how-to-store-user-preferences.md). |
| customDomain | Chaîne | Réservé à un usage interne. Domaine personnalisé dans lequel l’application web Lecteur immersif est hébergée (la valeur par défaut est null). |

##### `uiLang`
```Parameters
Type: String
Required: false
Default value: User's browser language 
```

##### `timeout`
```Parameters
Type: Number
Required: false
Default value: 15000
```

##### `uiZIndex`
```Parameters
Type: Number
Required: false
Default value: 1000
```

##### `onExit`
```Parameters
Type: Function
Required: false
Default value: null
```

##### `preferences`

> [!CAUTION]
> **IMPORTANT** N’essayez pas de modifier par programmation les valeurs de la chaîne `-preferences` envoyée à destination et en provenance de l’application Lecteur immersif, car cela peut provoquer un comportement inattendu entraînant une dégradation de l’expérience utilisateur pour vos clients. Les applications hôtes ne doivent jamais assigner de valeur personnalisée ou manipuler la chaîne `-preferences`. Quand vous utilisez l’option de chaîne `-preferences`, utilisez uniquement la valeur exacte qui a été retournée à partir de l’option de rappel de `-onPreferencesChanged`.

```Parameters
Type: String
Required: false
Default value: null
```

##### `onPreferencesChanged`
```Parameters
Type: Function
Required: false
Default value: null
```

##### `customDomain`
```Parameters
Type: String
Required: false
Default value: null
```

<br>

## <a name="readaloudoptions"></a>ReadAloudOptions

```typescript
type ReadAloudOptions = {
    voice?: string;
    speed?: number;
    autoplay?: boolean;
};
```

#### <a name="readaloudoptions-parameters"></a>Paramètres de ReadAloudOptions

| Nom | Type | Description |
| ---- | ---- |------------ |
| voice | Chaîne | Voix, « féminine » ou « masculine ». Notez que les langues ne prennent pas toutes en charge les deux genres. |
| speed | Numéro | Vitesse de lecture. Doit être comprise entre 0,5 et 2,5, bornes incluses. |
| autoPlay | Boolean | Démarrer automatiquement la lecture à voix haute au chargement du Lecteur immersif. |

##### `voice`
```Parameters
Type: String
Required: false
Default value: "Female" or "Male" (determined by language) 
Values available: "Female", "Male"
```

##### `speed`
```Parameters
Type: Number
Required: false
Default value: 1
Values available: 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2, 2.25, 2.5
```

> [!NOTE]
> En raison des limitations du navigateur, la lecture automatique n’est pas prise en charge dans Safari.

<br>

## <a name="translationoptions"></a>TranslationOptions

```typescript
type TranslationOptions = {
    language: string;
    autoEnableDocumentTranslation?: boolean;
    autoEnableWordTranslation?: boolean;
};
```

#### <a name="translationoptions-parameters"></a>Paramètres de TranslationOptions

| Nom | Type | Description |
| ---- | ---- |------------ |
| langage | Chaîne | Définit la langue de traduction. La valeur est au format d’étiquette d’identification de langue de la recommandation BCP 47 de l’IETF, par exemple fr-FR, es-MX, zh-Hans-CN. Requis pour activer automatiquement la traduction des mots ou du document. |
| autoEnableDocumentTranslation | Boolean | Traduire automatiquement le document complet. |
| autoEnableWordTranslation | Boolean | Activer automatiquement la traduction des mots. |

##### `language`
```Parameters
Type: String
Required: true
Default value: null 
Values available: See the Supported Languages section
```

<br>

## <a name="displayoptions"></a>DisplayOptions

```typescript
type DisplayOptions = {
    textSize?: number;
    increaseSpacing?: boolean;
    fontFamily?: string;
};
```

#### <a name="displayoptions-parameters"></a>Paramètres de DisplayOptions

| Nom | Type | Description |
| ---- | ---- |------------ |
| textSize | Numéro | Définit la taille du texte choisi. |
| increaseSpacing | Boolean | Définit si l’espacement du texte est activé ou désactivé. |
| fontFamily | Chaîne | Définit la police choisie (« Calibri », « ComicSans » ou « Sitka »). |

##### `textSize`
```Parameters
Type: Number
Required: false
Default value: 20, 36 or 42 (Determined by screen size)
Values available: 14, 20, 28, 36, 42, 48, 56, 64, 72, 84, 96
```

##### `fontFamily`
```Parameters
Type: String
Required: false
Default value: "Calibri"
Values available: "Calibri", "Sitka", "ComicSans"
```

<br>

## <a name="cookiepolicy-options"></a>Options de CookiePolicy

```typescript
enum CookiePolicy { Disable, Enable }
```

**Les paramètres listés ci-dessous sont fournis à titre d’information uniquement** . Le Lecteur immersif stocke ses paramètres ou les préférences de l’utilisateur dans des cookies. Cette option *cookiePolicy* **désactive** l’utilisation des cookies par défaut afin de se conformer aux lois de conformité des cookies de l’Union européenne. Pour réactiver les cookies et restaurer les fonctionnalités par défaut des préférences d’utilisateur du Lecteur immersif, vous devez vous assurer que votre application ou site web obtient le consentement explicite de l’utilisateur pour activer les cookies. Ensuite, pour réactiver les cookies dans le Lecteur immersif, vous devez définir explicitement l’option *cookiePolicy* sur *CookiePolicy.Enable* au lancement du Lecteur immersif. Le tableau ci-dessous décrit les paramètres que le Lecteur immersif stocke dans son cookie lorsque l’option *cookiePolicy* est activée.

#### <a name="settings-parameters"></a>Paramètres des réglages

| Paramètre | Type | Description |
| ------- | ---- | ----------- |
| textSize | Numéro | Définit la taille du texte choisi. |
| fontFamily | Chaîne | Définit la police choisie (« Calibri », « ComicSans » ou « Sitka »). |
| textSpacing | Numéro | Définit si l’espacement du texte est activé ou désactivé. |
| formattingEnabled | Boolean | Définit si la mise en forme HTML est activée ou désactivée. |
| thème | Chaîne | Définit le thème choisi (par exemple, « Clair », « Sombre », etc). |
| syllabificationEnabled | Boolean | Définit si la décomposition des mots en syllabes est activée ou désactivée. |
| nounHighlightingEnabled | Boolean | Définit si la mise en surbrillance des substantifs est activée ou désactivée. |
| nounHighlightingColor | Chaîne | Définit la couleur de mise en surbrillance des substantifs. |
| verbHighlightingEnabled | Boolean | Définit si la mise en surbrillance des verbes est activée ou désactivée. |
| verbHighlightingColor | Chaîne | Définit la couleur de mise en surbrillance des verbes. |
| adjectiveHighlightingEnabled | Boolean | Définit si la mise en surbrillance des adjectifs est activée ou désactivée. |
| adjectiveHighlightingColor | Chaîne | Définit la couleur de mise en surbrillance des adjectifs. |
| adverbHighlightingEnabled | Boolean | Définit si la mise en surbrillance des adverbes est activée ou désactivée. |
| adverbHighlightingColor | Chaîne | Définit la couleur de mise en surbrillance des adverbes. |
| pictureDictionaryEnabled | Boolean | Définit si le dictionnaire d’images est activé ou désactivé. |
| posLabelsEnabled | Boolean | Définit si les étiquettes de texte en exposants des éléments morphosyntaxiques sont activées ou désactivées.  |

<br>

## <a name="supported-languages"></a>Langues prises en charge

La fonctionnalité de traduction du Lecteur immersif prend en charge de nombreuses langues. Pour en savoir plus, consultez l’article [Prise en charge linguistique pour le Lecteur immersif](./language-support.md).

<br>

## <a name="html-support"></a>Prise en charge du langage HTML

Lorsque la mise en forme est activée, le contenu suivant est affiché au format HTML dans le Lecteur immersif.

| HTML | Contenu pris en charge |
| --------- | ----------- |
| Styles de police | Gras, Italique, Souligné, Code, Barré, Exposant, Indice |
| Listes non triées | Disque, Cercle, Carré |
| Listes triées | Décimal, Alphabet en majuscules, Alphabet en minuscules, Chiffres romains en majuscules, Chiffres romains en minuscules |

Les étiquettes non prises en charge sont affichées de manière comparable. Les images et les tables ne sont pas prises en charge pour l’instant.

<br>

## <a name="browser-support"></a>Prise en charge des navigateurs

Pour une meilleure expérience avec le lecteur immersif, utilisez les versions les plus récentes des navigateurs suivants.

* Microsoft Edge
* Internet Explorer 11
* Google Chrome
* Mozilla Firefox
* Apple Safari

<br>

## <a name="next-steps"></a>Étapes suivantes

* Explorer le [kit de développement logiciel (SDK) du lecteur immersif sur GitHub](https://github.com/microsoft/immersive-reader-sdk)
* [Démarrage rapide : Créer une application web qui lance le lecteur immersif (C#)](./quickstarts/client-libraries.md?pivots=programming-language-csharp)
