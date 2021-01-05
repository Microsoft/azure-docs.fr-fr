---
title: Méthode Translate de Translator
titleSuffix: Azure Cognitive Services
description: Comprendre les paramètres, les en-têtes et les messages de corps de la méthode Translate de d’Azure Cognitive Services Translator pour traduire du texte.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 08/06/2020
ms.author: swmachan
ms.openlocfilehash: 15f20bab91a4feab056a9f8cd6b012792f036be2
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95016545"
---
# <a name="translator-30-translate"></a>Translator 3.0 : Translate

Traduit du texte.

## <a name="request-url"></a>URL de la demande

Envoyez une demande `POST` à :

```HTTP
https://api.cognitive.microsofttranslator.com/translate?api-version=3.0
```

## <a name="request-parameters"></a>Paramètres de la demande

Les paramètres de demande transmis à la chaîne de requête sont les suivants :

### <a name="required-parameters"></a>Paramètres obligatoires

<table width="100%">
  <th width="20%">Paramètre de requête.</th>
  <th>Description</th>
  <tr>
    <td>api-version</td>
    <td><em>Paramètre obligatoire</em>.<br/>Version de l’API demandée par le client. La valeur doit être <code>3.0</code>.</td>
  </tr>
  <tr>
    <td>to</td>
    <td><em>Paramètre obligatoire</em>.<br/>Spécifie la langue du texte de sortie. La langue cible doit être l’une des <a href="./v3-0-languages.md">langues prises en charge</a> incluses dans l’étendue <code>translation</code>. Par exemple, utilisez <code>to=de</code> pour traduire en allemand.<br/>Il est possible de traduire en plusieurs langues simultanément en répétant le paramètre dans la chaîne de requête. Par exemple, utilisez <code>to=de&to=it</code> pour traduire en allemand et italien.</td>
  </tr>
</table>

### <a name="optional-parameters"></a>Paramètres facultatifs

<table width="100%">
  <th width="20%">Paramètre de requête.</th>
  <th>Description</th>
  <tr>
    <td>de</td>
    <td><em>Paramètre facultatif</em>.<br/>Spécifie la langue du texte d’entrée. Trouvez les langues disponibles pour la traduction en recherchant <a href="./v3-0-languages.md">langues prises en charge</a> à l’aide de l’étendue <code>translation</code>. Si le paramètre <code>from</code> n’est pas spécifié, une détection automatique de la langue est appliquée pour déterminer la langue source. <br/><br/>Vous devez utiliser le paramètre <code>from</code> au lieu de la détection automatique lors de l’utilisation de la fonctionnalité de <a href="/azure/cognitive-services/translator/dynamic-dictionary">dictionnaire dynamique</a>.</td>
  </tr>  
  <tr>
    <td>textType</td>
    <td><em>Paramètre facultatif</em>.<br/>Définit si le texte en cours de traduction est au format texte brut ou HTML. Tout code HTML doit être un élément bien formé et complet. Les valeurs possibles sont : <code>plain</code> (par défaut) ou <code>html</code>.</td>
  </tr>
  <tr>
    <td>catégorie</td>
    <td><em>Paramètre facultatif</em>.<br/>Chaîne spécifiant la catégorie (domaine) de la traduction. Ce paramètre est utilisé pour obtenir des traductions d’un système personnalisé créé avec <a href="../customization.md">Custom Translator</a>. Ajoutez l’ID de catégorie de vos <a href="/azure/cognitive-services/translator/custom-translator/how-to-create-project#view-project-details">détails de projet</a> Custom Translator à ce paramètre pour utiliser votre système personnalisé déployé. La valeur par défaut est <code>general</code>.</td>
  </tr>
  <tr>
    <td>ProfanityAction</td>
    <td><em>Paramètre facultatif</em>.<br/>Spécifie comment les vulgarités doivent être traitées dans les traductions. Les valeurs possibles sont : <code>NoAction</code> (valeur par défaut), <code>Marked</code> ou <code>Deleted</code>. Pour comprendre comment traiter les vulgarités, voir <a href="#handle-profanity">Gestion de la vulgarité</a>.</td>
  </tr>
  <tr>
    <td>ProfanityMarker</td>
    <td><em>Paramètre facultatif</em>.<br/>Spécifie comment vulgarités doit être marquées dans les traductions. Les valeurs possibles sont : <code>Asterisk</code> (par défaut) ou <code>Tag</code>. Pour comprendre comment traiter les vulgarités, voir <a href="#handle-profanity">Gestion de la vulgarité</a>.</td>
  </tr>
  <tr>
    <td>includeAlignment</td>
    <td><em>Paramètre facultatif</em>.<br/>Spécifie s’il faut inclure une projection d’alignement du texte source vers le texte traduit. Les valeurs possibles sont <code>true</code> ou <code>false</code> (par défaut). </td>
  </tr>
  <tr>
    <td>includeSentenceLength</td>
    <td><em>Paramètre facultatif</em>.<br/>Spécifie s’il faut inclure des limites de longueur de phrase aux texte d’entrée et au texte traduit. Les valeurs possibles sont <code>true</code> ou <code>false</code> (par défaut).</td>
  </tr>
  <tr>
    <td>suggestedFrom</td>
    <td><em>Paramètre facultatif</em>.<br/>Spécifie une langue de base si la langue du texte d’entrée ne peut pas être identifiée. La détection automatique de la langue est appliquée en cas d’omission du paramètre <code>from</code>. Si la détection échoue, la langue <code>suggestedFrom</code> est présupposée.</td>
  </tr>
  <tr>
    <td>fromScript</td>
    <td><em>Paramètre facultatif</em>.<br/>Spécifie le script du texte d’entrée.</td>
  </tr>
  <tr>
    <td>toScript</td>
    <td><em>Paramètre facultatif</em>.<br/>Spécifie le script du texte traduit.</td>
  </tr>
  <tr>
    <td>allowFallback</td>
    <td><em>Paramètre facultatif</em>.<br/>Spécifie que le service est autorisé à revenir à un système général quand il n’existe pas de système personnalisé. Les valeurs possibles sont : <code>true</code> (par défaut) ou <code>false</code>.<br/><br/><code>allowFallback=false</code> spécifie que la traduction doit utiliser uniquement les systèmes entraînés pour le <code>category</code> spécifié par la requête. Si une traduction de langue X en langue Y exige un chaînage via une langue relais E, tous les systèmes présents dans la chaîne (X->E et E->Y) doivent être personnalisés et avoir la même catégorie. Si aucun système n’est trouvé avec une catégorie spécifique, la requête retourne le code d’état 400. <code>allowFallback=true</code> spécifie que le service est autorisé à revenir à un système général quand il n’existe pas de système personnalisé.
</td>
  </tr>
</table> 

Les en-têtes de demande sont les suivants :

<table width="100%">
  <th width="20%">headers</th>
  <th>Description</th>
  <tr>
    <td>En-tête(s) d’authentification</td>
    <td><em>En-tête de demande obligatoire</em>.<br/>Voir les <a href="/azure/cognitive-services/translator/reference/v3-0-reference#authentication">options disponibles pour l’authentification</a>.</td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td><em>En-tête de demande obligatoire</em>.<br/>Spécifie le type de contenu de la charge utile.<br/> La valeur acceptée est <code>application/json; charset=UTF-8</code>.</td>
  </tr>
  <tr>
    <td>Content-Length</td>
    <td><em>En-tête de demande obligatoire</em>.<br/>Longueur du corps de la demande.</td>
  </tr>
  <tr>
    <td>X-ClientTraceId</td>
    <td><em>Facultatif</em>.<br/>GUID généré par le client pour identifier de façon unique la demande. Vous pouvez omettre cet en-tête si vous incluez l’ID de trace dans la chaîne de requête à l’aide d’un paramètre de requête appelé <code>ClientTraceId</code>.</td>
  </tr>
</table> 

## <a name="request-body"></a>Corps de la demande

Le corps de la demande est un tableau JSON. Chaque élément du tableau est un objet JSON avec une propriété de chaîne nommée `Text`, qui représente la chaîne à traduire.

```json
[
    {"Text":"I would really like to drive your car around the block a few times."}
]
```

Les limites suivantes s'appliquent :

* Le tableau ne peut pas compter plus de 100 éléments.
* L’intégralité du texte inclus dans la demande ne peut pas dépasser 10 000 caractères, espaces comprises.

## <a name="response-body"></a>Response body

Une réponse correcte est un tableau JSON avec un résultat pour chaque chaîne dans le tableau d’entrée. Un objet de résultat inclut les propriétés suivantes :

  * `detectedLanguage`: objet décrivant la langue détectée au moyen des propriétés suivantes :

      * `language`: chaîne représentant le code de la langue détectée.

      * `score`: valeur flottante indiquant le niveau de confiance dans le résultat. Le score est compris entre zéro et un, un score faible indiquant un niveau de confiance bas.

    La propriété `detectedLanguage` n’est présente dans l’objet de résultat que quand la détection automatique de la langue est demandée.

  * `translations`: tableau des résultats de la traduction. La taille du tableau correspond au nombre de langues cibles spécifié par le paramètre de requête `to`. Chaque élément dans le tableau inclut ce qui suit :

    * `to`: chaîne représentant le code de langue de la langue cible.

    * `text`: chaîne fournissant le texte traduit.

    * `transliteration`: objet fournissant le texte traduit dans le script spécifié par le paramètre `toScript`.

      * `script`: chaîne spécifiant le script cible.   

      * `text`: chaîne fournissant le texte traduit dans le script cible.

    L’objet `transliteration` n’est pas inclus si une translittération n’a pas lieu.

    * `alignment`: objet avec une propriété de chaîne unique nommée `proj`, qui mappe le texte d’entrée au texte traduit. Les informations d’alignement ne sont fournies que quand le paramètre de requête `includeAlignment` est `true`. L’alignement est renvoyé en tant que valeur de chaîne au format suivant : `[[SourceTextStartIndex]:[SourceTextEndIndex]–[TgtTextStartIndex]:[TgtTextEndIndex]]`.  Le signe deux-points sépare les index de début et de fin, le signe tiret sépare les langues, et une espace sépare les mots. Un mot peut s’aligner avec zéro, un ou plusieurs mots dans l’autre langue, et les mots alignés peuvent ne pas être contigus. Quand aucune information d’alignement n’est disponible, l’élément d’alignement est vide. Pour obtenir un exemple et les restrictions, voir [Obtenir les informations d’alignement](#obtain-alignment-information).

    * `sentLen`: objet retournant les limites de longueur de phrase dans les textes d’entrée et de sortie.

      * `srcSentLen`: tableau d’entiers représentant les longueurs des phrases dans le texte d’entrée. La longueur du tableau correspond au nombre de phrases, et les valeurs sont les longueurs des phrases.

      * `transSentLen`:  tableau d’entiers représentant les longueurs des phrases dans le texte traduit. La longueur du tableau correspond au nombre de phrases, et les valeurs sont les longueurs des phrases.

    Les limites de longueur de phrase ne sont incluses que si le paramètre de requête `includeSentenceLength` est `true`.

  * `sourceText`: objet avec une propriété de chaîne unique nommée `text`, qui fournit le texte d’entrée dans le script par défaut de la langue source. La propriété `sourceText` est présente uniquement quand l’entrée est exprimée dans un script qui n’est pas le script habituel pour la langue. Par exemple, si l’entrée est de l’arabe écrit dans un script latin, `sourceText.text` est le même texte en arabe converti en script arabe.

Des exemples de réponses JSON sont fournis dans la section [exemples](#examples).

## <a name="response-headers"></a>En-têtes de réponse

<table width="100%">
  <th width="20%">headers</th>
  <th>Description</th>
    <tr>
    <td>X-RequestId</td>
    <td>Valeur générée par le service pour identifier la demande. Elle sert à des fins de dépannage.</td>
  </tr>
  <tr>
    <td>X-MT-System</td>
    <td>Spécifie le type de système qui a été utilisé pour la traduction pour chaque langue de destination (« to ») demandée pour la traduction. La valeur est une liste de chaînes séparées par des virgules. Chaque chaîne indique un type :<br/><ul><li>Personnalisé : la requête inclut un système personnalisé, et au moins un système personnalisé a été utilisé lors de la traduction.</li><li>Équipe : toutes les autres requêtes.</li></td>
  </tr>
</table> 

## <a name="response-status-codes"></a>Codes d’état de réponse

Voici les codes d’état HTTP qu’une demande peut retourner. 

<table width="100%">
  <th width="20%">Code d’état</th>
  <th>Description</th>
  <tr>
    <td>200</td>
    <td>Réussite.</td>
  </tr>
  <tr>
    <td>400</td>
    <td>L’un des paramètres de requête est manquant ou non valide. Corrigez les paramètres de demande avant de réessayer.</td>
  </tr>
  <tr>
    <td>401</td>
    <td>Il n’a pas été possible d’authentifier la demande. Vérifiez que les informations d’identification sont spécifiées et valides.</td>
  </tr>
  <tr>
    <td>403</td>
    <td>La demande n’est pas autorisée. Vérifiez le message d’erreur détaillé. Cela indique souvent que toutes les traductions gratuites fournies avec un abonnement d’essai ont été utilisées.</td>
  </tr>
  <tr>
    <td>408</td>
    <td>La requête n’a pas pu être satisfaite car il manque une ressource. Vérifiez le message d’erreur détaillé. L’utilisation d’un <code>category</code> personnalisé indique souvent que le système de traduction personnalisée n’est pas encore disponible pour traiter des requêtes. La requête doit être retentée après un délai d’attente (par exemple, 1 minute).</td>
  </tr>
  <tr>
    <td>429</td>
    <td>Le serveur a rejeté la requête, car le client a dépassé les limites de requête.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>Une erreur inattendue s’est produite. Si l’erreur persiste, signalez-la en fournissant les informations suivantes : date et heure de la défaillance, identificateur de la demande dans l’en-tête de réponse,<code>X-RequestId</code> et identificateur du client dans l’en-tête de demande <code>X-ClientTraceId</code>.</td>
  </tr>
  <tr>
    <td>503</td>
    <td>Serveur temporairement indisponible. Relancez la requête. Si l’erreur persiste, signalez-la en fournissant les informations suivantes : date et heure de la défaillance, identificateur de la demande dans l’en-tête de réponse,<code>X-RequestId</code> et identificateur du client dans l’en-tête de demande <code>X-ClientTraceId</code>.</td>
  </tr>
</table> 

Si une erreur se produit, la requête renvoie également une réponse d'erreur JSON. Le code d’erreur est un nombre à 6 chiffres qui combine le code d’état HTTP à 3 chiffres et un nombre à 3 chiffres qui sert à catégoriser plus précisément l’erreur. Vous trouverez les codes d’erreur les plus courants sur la [page Référence de Translator v3](./v3-0-reference.md#errors). 

## <a name="examples"></a>Exemples

### <a name="translate-a-single-input"></a>Traduire une entrée unique

Cet exemple montre comment traduire une phrase unique de l’anglais en chinois simplifié.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'Hello, what is your name?'}]"
```

Le corps de la réponse est le suivant :

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"}
        ]
    }
]
```

Le `translations` tableau inclut un élément qui fournit la traduction de l’élément de texte dans l’entrée.

### <a name="translate-a-single-input-with-language-auto-detection"></a>Traduire une entrée unique avec détection automatique de la langue

Cet exemple montre comment traduire une phrase unique de l’anglais en chinois simplifié. La demande ne spécifie pas la langue d’entrée. La détection automatique de la langue source est utilisée à la place.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'Hello, what is your name?'}]"
```

Le corps de la réponse est le suivant :

```
[
    {
        "detectedLanguage": {"language": "en", "score": 1.0},
        "translations":[
            {"text": "你好, 你叫什么名字？", "to": "zh-Hans"}
        ]
    }
]
```
La réponse est similaire à la réponse de l’exemple précédent. Étant donné que la détection automatique de la langue a été demandée, la réponse inclut également des informations sur la langue détectée pour le texte d’entrée. La détection automatique de la langue fonctionne mieux avec un texte d’entrée plus long.

### <a name="translate-with-transliteration"></a>Traduire avec translittération

Étendons l’exemple précédent en ajoutant la translittération. La requête suivante demande une traduction chinoise écrite en script Latin.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=zh-Hans&toScript=Latn" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'Hello, what is your name?'}]"
```

Le corps de la réponse est le suivant :

```
[
    {
        "detectedLanguage":{"language":"en","score":1.0},
        "translations":[
            {
                "text":"你好, 你叫什么名字？",
                "transliteration":{"script":"Latn", "text":"nǐ hǎo , nǐ jiào shén me míng zì ？"},
                "to":"zh-Hans"
            }
        ]
    }
]
```

Le résultat de la traduction inclut à présent une `transliteration` propriété qui fournit le texte traduit en caractères latins.

### <a name="translate-multiple-pieces-of-text"></a>Traduire plusieurs éléments de texte

La traduction de plusieurs chaînes en une fois nécessite simplement de spécifier un tableau de chaînes dans le corps de la demande.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'Hello, what is your name?'}, {'Text':'I am fine, thank you.'}]"
```

La réponse contient la traduction de tous les éléments de texte dans le même ordre que dans la requête.
Le corps de la réponse est le suivant :

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"}
        ]
    },            
    {
        "translations":[
            {"text":"我很好，谢谢你。","to":"zh-Hans"}
        ]
    }
]
```

### <a name="translate-to-multiple-languages"></a>Traduire en plusieurs langues

Cet exemple montre comment traduire une même entrée en plusieurs langues en utilisant une seule requête.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans&to=de" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'Hello, what is your name?'}]"
```

Le corps de la réponse est le suivant :

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"},
            {"text":"Hallo, was ist dein Name?","to":"de"}
        ]
    }
]
```

### <a name="handle-profanity"></a>Traiter la vulgarité

Le service Translator conserve normalement dans la traduction toute vulgarité présente dans la source. Le degré de vulgarité et le contexte qui rend des mots vulgaires diffèrent selon les cultures. Par conséquent, le degré de vulgarité dans la langue cible peut être amplifié ou réduit.

Si vous souhaitez éviter toute vulgarité dans la traduction, indépendamment de la présence de vulgarité dans le texte source, vous disposez d’une option de filtrage de la vulgarité. L’option vous permet de spécifier si vous souhaitez que la vulgarité soit supprimée, que les vulgarités soient marquées à l’aide de balises appropriées (ce qui vous offre la possibilité d’ajouter votre propre post-traitement), ou qu’aucune action ne soit appliquée. Les valeurs acceptées de `ProfanityAction` sont `Deleted`, `Marked` et `NoAction` (par défaut).

<table width="100%">
  <th width="20%">ProfanityAction</th>
  <th>Action</th>
  <tr>
    <td><code>NoAction</code></td>
    <td>Il s'agit du comportement par défaut. La vulgarité de la source est reflétée dans la cible.<br/><br/>
    <strong>Exemple de source (japonais)</strong> : 彼はジャッカスです。<br/>
    <strong>Exemple de traduction (français)</strong> : Il est un imbécile.
    </td>
  </tr>
  <tr>
    <td><code>Deleted</code></td>
    <td>Les mots vulgaires sont retirés de la cible sans remplacement.<br/><br/>
    <strong>Exemple de source (japonais)</strong> : 彼はジャッカスです。<br/>
    <strong>Exemple de traduction (français)</strong> : C’est un.
    </td>
  </tr>
  <tr>
    <td><code>Marked</code></td>
    <td>Les mots vulgaires sont remplacés par un marqueur dans la sortie. Le marqueur varie selon le paramètre <code>ProfanityMarker</code>.<br/><br/>
Pour <code>ProfanityMarker=Asterisk</code>, les mots vulgaires sont remplacés par <code>***</code> :<br/>
    <strong>Exemple de source (japonais)</strong> : 彼はジャッカスです。<br/>
    <strong>Exemple de traduction (français)</strong> : C’est un \*\*\*.<br/><br/>
Pour <code>ProfanityMarker=Tag</code>, les mots vulgaires sont entourés de balises XML &lt;profanity&gt; et &lt;/profanity&gt; :<br/>
    <strong>Exemple de source (japonais)</strong> : 彼はジャッカスです。<br/>
    <strong>Exemple de traduction (français)</strong> : C’est un &lt;profanity&gt;con&lt;/profanity&gt;.
  </tr>
</table> 

Par exemple :

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de&profanityAction=Marked" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'This is a freaking good idea.'}]"
```
Cette demande renvoie :

```
[
    {
        "translations":[
            {"text":"Das ist eine *** gute Idee.","to":"de"}
        ]
    }
]
```

Comparez à :

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de&profanityAction=Marked&profanityMarker=Tag" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'This is a freaking good idea.'}]"
```

Cette dernière demande renvoie :

```
[
    {
        "translations":[
            {"text":"Das ist eine <profanity>verdammt</profanity> gute Idee.","to":"de"}
        ]
    }
]
```

### <a name="translate-content-with-markup-and-decide-whats-translated"></a>Traduire du contenu incluant un balisage et décider ce qui est traduit

Il est courant de traduire du contenu incluant un balisage, tel que le contenu d’une page HTML ou d’un document XML. Lors de la traduction de contenu incluant des balises, incluez le paramètre de requête `textType=html`. De plus, il est parfois utile d’exclure du contenu spécifique de la traduction. Vous pouvez utiliser l’attribut `class=notranslate` pour spécifier le contenu qui doit rester dans la langue d’origine. Dans l’exemple suivant, le contenu du premier élément `div` ne sera pas traduit, tandis que le contenu du deuxième élément `div` le sera.

```
<div class="notranslate">This will not be translated.</div>
<div>This will be translated. </div>
```

Voici un exemple de demande.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans&textType=html" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'<div class=\"notranslate\">This will not be translated.</div><div>This will be translated.</div>'}]"
```

La réponse est la suivante :

```
[
    {
        "translations":[
            {"text":"<div class=\"notranslate\">This will not be translated.</div><div>这将被翻译。</div>","to":"zh-Hans"}
        ]
    }
]
```

### <a name="obtain-alignment-information"></a>Obtenir les informations d’alignement

L’alignement est retourné en tant que valeur de chaîne au format suivant pour chaque mot de la source. Les informations pour chaque mot sont séparées par un espace, y compris pour les langues non séparées par des espaces (scripts) comme le chinois :

[[SourceTextStartIndex]\:[SourceTextEndIndex]–[TgtTextStartIndex]\:[TgtTextEndIndex]] *

Exemple de chaîne d’alignement : « 0:0-7:10 1:2-11:20 3:4-0:3 3:4-4:6 5:5-21:21 ».

En d’autres termes, le signe deux-points sépare les index de début et de fin, le signe tiret sépare les langues et un espace sépare les mots. Un mot peut s’aligner avec zéro, un ou plusieurs mots dans l’autre langue, et les mots alignés peuvent ne pas être contigus. Quand aucune information d’alignement n’est disponible, l’élément Alignment est vide. La méthode ne retourne aucune erreur dans ce cas.

Pour recevoir les informations d’alignement, spécifiez `includeAlignment=true` sur la chaîne de requête.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=fr&includeAlignment=true" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'The answer lies in machine translation.'}]"
```

La réponse est la suivante :

```
[
    {
        "translations":[
            {
                "text":"La réponse se trouve dans la traduction automatique.",
                "to":"fr",
                "alignment":{"proj":"0:2-0:1 4:9-3:9 11:14-11:19 16:17-21:24 19:25-40:50 27:37-29:38 38:38-51:51"}
            }
        ]
    }
]
```

Les informations d’alignement commencent par `0:2-0:1`, ce qui signifie que les trois premiers caractères dans le texte source (`The`) mappent aux deux premiers caractères dans le texte traduit (`La`).

#### <a name="limitations"></a>Limites
L'obtention des informations d'alignement est une fonctionnalité expérimentale dédiée au prototypage de la recherche et des expériences avec des mappages de phrases potentiels. Nous pourrions décider de ne plus la prendre en charge à l'avenir. Voici quelques restrictions notables pour lesquelles les alignements ne sont pas pris en charge :

* L’alignement n’est pas disponible pour du texte au format HTML, par exemple textType=html
* L’alignement est renvoyé uniquement pour un sous-ensemble de paires de langues :
  - Anglais vers/depuis n’importe quelle autre langue, à l’exception du chinois traditionnel, du cantonais (traditionnel) ou du serbe (cyrillique).
  - du japonais au coréen ou inversement.
  - du japonais au chinois simplifié et du chinois simplifié au japonais. 
  - du chinois simplifié au chinois traditionnel et du chinois traditionnel au chinois simplifié. 
* Vous ne recevrez pas d’alignement si la phrase est une traduction définie. Des traductions définies sont, par exemple, « Ceci est un test », « Je t’aime » et d’autres phrases extrêmement fréquentes.
* L'alignement n'est pas possible lorsque vous appliquez l'une des approches visant à empêcher la traduction comme décrit [ici](../prevent-translation.md)

### <a name="obtain-sentence-boundaries"></a>Obtenir les limites de longueur de phrase

Pour recevoir des informations sur la longueur des phrases dans le texte source et le texte traduit, spécifiez `includeSentenceLength=true` dans la chaîne de requête.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=fr&includeSentenceLength=true" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'The answer lies in machine translation. The best machine translation technology cannot always provide translations tailored to a site or users like a human. Simply copy and paste a code snippet anywhere.'}]"
```

La réponse est la suivante :

```
[
    {
        "translations":[
            {
                "text":"La réponse se trouve dans la traduction automatique. La meilleure technologie de traduction automatique ne peut pas toujours fournir des traductions adaptées à un site ou des utilisateurs comme un être humain. Il suffit de copier et coller un extrait de code n’importe où.",
                "to":"fr",
                "sentLen":{"srcSentLen":[40,117,46],"transSentLen":[53,157,62]}
            }
        ]
    }
]
```

### <a name="translate-with-dynamic-dictionary"></a>Traduire avec un dictionnaire dynamique

Si vous connaissez déjà la traduction que vous souhaitez appliquer à un mot ou à une phrase, vous pouvez la fournir en tant que balisage dans la demande. Le dictionnaire dynamique n’est sûr que pour des noms propres comme les noms de personnes et les noms de produits.

Le balisage à fournir utilise la syntaxe suivante.

``` 
<mstrans:dictionary translation="translation of phrase">phrase</mstrans:dictionary>
```

Par exemple, considérez la phrase : « Le mot wordomatic est une entrée de dictionnaire ». Pour conserver le mot _wordomatic_ dans la traduction, envoyez la demande suivante :

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json; charset=UTF-8" -d "[{'Text':'The word <mstrans:dictionary translation=\"wordomatic\">word or phrase</mstrans:dictionary> is a dictionary entry.'}]"
```

Le résultat est le suivant :

```
[
    {
        "translations":[
            {"text":"Das Wort \"wordomatic\" ist ein Wörterbucheintrag.","to":"de"}
        ]
    }
]
```

Cette fonctionnalité opère de la même façon avec `textType=text` ou `textType=html`. Elle doit être utilisée avec parcimonie. La façon appropriée et de loin préférable de personnaliser une traduction consiste à utiliser Custom Translator. Custom Translator utilise totalement le contexte et les probabilités statistiques. Si vous avez ou pouvez vous permettre de créer des données d’apprentissage qui montrent votre mot ou phrase en contexte, vous obtenez de bien meilleurs résultats. [En savoir plus sur Custom Translator](../customization.md).