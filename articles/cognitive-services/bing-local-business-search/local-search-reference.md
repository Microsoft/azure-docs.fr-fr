---
title: Informations de référence sur l’API Recherche d’entreprises locales Bing v7
titleSuffix: Azure Cognitive Services
description: Cet article donne des détails techniques sur les objets de réponse, les paramètres de la requête et les en-têtes qui affectent les résultats de recherche.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-local-business
ms.topic: conceptual
ms.date: 11/01/2018
ms.author: rosh
ms.openlocfilehash: 9791d99598fe3d043c42a37e2f4993edd6c5b3ba
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96487131"
---
# <a name="bing-local-business-search-api-v7-reference"></a>Informations de référence sur l’API Recherche d’entreprises locales Bing v7

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche d’entreprises locales envoie une requête de recherche à Bing pour obtenir des résultats sur les restaurants, les hôtels et d’autres entreprises locales. Pour trouver des lieux, la requête peut spécifier le nom de l’entreprise locale ou une catégorie (par exemple, les restaurants autour de moi). Les résultats d’entités incluent des personnes, des lieux ou d’autres éléments. Dans ce contexte, un lieu correspond à une entité de type entreprise, un État, un pays/région, etc.  

Cette section donne des détails techniques sur les objets de réponse, les paramètres de la requête et les en-têtes qui affectent les résultats de recherche. Pour obtenir des exemples montrant comment effectuer des requêtes, voir [Démarrage rapide C# sur la Recherche d’entreprises locales](quickstarts/local-quickstart.md) ou [Démarrage rapide Java sur la Recherche d’entreprises locales](quickstarts/local-search-java-quickstart.md). 
  
Pour plus d’informations sur les en-têtes que doivent comporter les demandes, voir [En-têtes](#headers).  
  
Pour plus d’informations sur les paramètres de requête que doivent comporter les demandes, voir [Paramètres de la requête](#query-parameters).  
  
Pour plus d’informations sur les objets JSON que comporte la réponse, voir [Objets de la réponse](#response-objects).

Pour plus d’informations sur l’utilisation autorisée et l’affichage des résultats, voir [Conditions d’utilisation et d’affichage](../bing-web-search/use-display-requirements.md).


  
## <a name="endpoint"></a>Point de terminaison  
Pour demander les résultats en matière d’entreprises locales, envoyez une demande GET à : 

``` 
https://api.cognitive.microsoft.com/bing/v7.0/localbusinesses/search

```
  
La requête doit utiliser le protocole HTTPS.  
  
> [!NOTE]
> La longueur maximale de l’URL est de 2 048 caractères. Pour que votre URL ne dépasse pas la limite, la longueur maximale de vos paramètres de requête doit être inférieure à 1 500 caractères. Si l’URL dépasse 2 048 caractères, le serveur retourne une erreur 404 (Introuvable).  
  
  
## <a name="headers"></a>headers  
Voici les en-têtes possibles d’une demande et d’une réponse.  
  
|En-tête|Description|  
|------------|-----------------|  
|Acceptation|En-tête de demande facultatif.<br /><br /> Le type de média par défaut est application/json. Pour spécifier que la réponse utilise [JSON-LD](https://json-ld.org/), donnez la valeur application/ld+json à l’en-tête Accept.|  
|<a name="acceptlanguage"></a>Accept-Language|En-tête de demande facultatif.<br /><br /> Liste délimitée par des virgules des langues à utiliser pour les chaînes d’interface utilisateur. Elle est triée par ordre de préférence décroissant. Pour plus d’informations, notamment le format attendu, voir [RFC2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html).<br /><br /> Cet en-tête et le paramètre de requête [setLang](#setlang) s’excluent mutuellement &mdash; ne spécifiez pas les deux.<br /><br /> Si vous définissez cet en-tête, vous devrez également spécifier le paramètre de requête cc. Pour déterminer pour quel marché les résultats devront être retournés, Bing utilise la première langue prise en charge qu’il trouve dans la liste et la combine avec la valeur du paramètre `cc`. Si la liste ne comporte pas de langue prise en charge, Bing recherche la langue et le marché les plus proches qui prennent en charge la demande, ou utilise un marché agrégé ou par défaut pour les résultats. Pour identifier le marché utilisé par Bing, voir l’en-tête BingAPIs-Market.<br /><br /> N’utilisez cet en-tête et le paramètre de requête `cc` que si vous spécifiez plusieurs langues. Sinon, utilisez les paramètres de requête [mkt](#mkt) et [setLang](#setlang).<br /><br /> Une chaîne d’interface utilisateur est une chaîne utilisée comme étiquette dans une interface utilisateur. Les objets de réponse JSON en comportent quelques-unes. Les liens vers les propriétés Bing.com dans les objets de la réponse s’appliquent à la langue spécifiée.|  
|<a name="market"></a>BingAPIs-Market|En-tête de réponse.<br /><br /> Marché utilisé par la demande. La forme est \<languageCode\>-\<countryCode\>. Par exemple, en-US.|  
|<a name="traceid"></a>BingAPIs-TraceId|En-tête de réponse.<br /><br /> ID de l’entrée du journal contenant les détails de la demande. Lorsqu’une erreur se produit, capturez cet ID. Si vous ne parvenez pas à identifier ou à résoudre le problème, précisez cet ID avec les autres informations envoyées à l’équipe de support.|  
|<a name="subscriptionkey"></a>Ocp-Apim-Subscription-Key|En-tête de demande requis.<br /><br /> Clé d’abonnement reçue lors de l’inscription à ce service dans [Cognitive Services](https://www.microsoft.com/cognitive-services/).|  
|<a name="pragma"></a>Pragma|En-tête de demande facultatif.<br /><br /> Par défaut, Bing retourne le contenu en cache, s’il est disponible. Pour éviter cela, définissez l’en-tête Pragma sur no-cache (par exemple, Pragma: no-cache).
|<a name="useragent"></a>User-Agent|En-tête de demande facultatif.<br /><br /> Agent utilisateur à l’origine de la requête. Bing utilise l’agent utilisateur pour offrir une expérience optimisée aux utilisateurs mobiles. Nous vous conseillons de toujours indiquer cet en-tête, bien qu’il soit facultatif.<br /><br /> L’agent utilisateur doit correspondre à la chaîne envoyée par n’importe quel navigateur couramment utilisé. Pour plus d’informations sur les agents utilisateurs, voir [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html).<br /><br /> Voici quelques exemples de chaînes de l’agent utilisateur.<br /><ul><li>Windows Phone&mdash;Mozilla/5.0 (compatible ; MSIE 10.0 ; Windows Phone 8.0 ; Trident/6.0 ; IEMobile/10.0 ; ARM ; Touch ; NOKIA ; Lumia 822)<br /><br /></li><li>Android&mdash;Mozilla/5.0 (Linux ; U ; Android 2.3.5 ; en-us ; SCH-I500 Build/GINGERBREAD) AppleWebKit/533.1 (KHTML ; comme Gecko) Version/4.0 Mobile Safari/533.1<br /><br /></li><li>iPhone&mdash;Mozilla/5.0 (iPhone ; CPU iPhone OS 6_1 comme Mac OS X) AppleWebKit/536.26 (KHTML ; comme Gecko) Mobile/10B142 iPhone4;1 BingWeb/3.03.1428.20120423<br /><br /></li><li>PC&mdash;Mozilla/5.0 (Windows NT 6.3 ; WOW64 ; Trident/7.0 ; Touch ; rv:11.0) comme Gecko<br /><br /></li><li>iPad&mdash;Mozilla/5.0 (iPad ; CPU OS 7_0 comme Mac OS X) AppleWebKit/537.51.1 (KHTML, comme Gecko) Version/7.0 Mobile/11A465 Safari/9537.53</li></ul>|
|<a name="clientid"></a>X-MSEdge-ClientID|En-tête de demande et de réponse facultatif.<br /><br /> Bing utilise cet en-tête pour garantir aux utilisateurs un comportement cohérent d’un appel d’API Bing à l’autre. Bing propose souvent de nouvelles fonctionnalités et améliorations, et se sert de l’ID client comme d’une clé pour attribuer le trafic aux différentes versions d’évaluation. Si vous n’assignez pas le même ID client à un utilisateur d’une demande à l’autre, Bing est susceptible d’affecter cet utilisateur à plusieurs versions d’évaluation en conflit, ce qui risque de nuire à l’expérience utilisateur. Par exemple, si la deuxième demande comporte une attribution de version d’évaluation différente de la première, l’expérience se révélera peut-être inattendue. De même, Bing peut utiliser l’ID client pour personnaliser les résultats web d’après l’historique de recherche correspondant à cet ID et ainsi proposer à l’utilisateur une expérience plus riche.<br /><br /> Bing utilise également cet en-tête pour aider à améliorer le classement des résultats en analysant l’activité générée par un ID client. L’amélioration de la pertinence permet d’obtenir des résultats de meilleure qualité de la part des API Bing et, en retour, des taux de clic plus élevés pour le consommateur des API.<br /><br /> **IMPORTANT :** Il est vivement recommandé d’indiquer cet en-tête, bien qu’il soit facultatif. Grâce à la persistance de l’ID client dans plusieurs demandes pour une même combinaison appareil/utilisateur final, (1) le consommateur des API bénéficie d’une expérience utilisateur cohérente et (2) le taux de clic est plus élevé du fait des résultats de meilleure qualité provenant des API Bing.<br /><br /> Voici les règles d’utilisation de base qui s’appliquent à cet en-tête.<br /><ul><li>Chaque utilisateur de votre application sur l’appareil doit avoir un ID client unique, généré par Bing.<br /><br/>Si vous n’insérez pas cet en-tête dans la demande, Bing génère un ID et le retourne dans l’en-tête de réponse X-MSEdge-ClientID. La première fois que l’utilisateur utilise votre application sur cet appareil est la seule fois où vous ne devez PAS inclure cet en-tête dans la demande.<br /><br/></li><li>Utilisez l’ID client pour chaque requête d’API Bing qu’effectue votre application pour cet utilisateur sur l’appareil.<br /><br/></li><li>**ATTENTION :** Vérifiez que cet ID client ne peut pas être associé à des informations authentifiables sur le compte d’utilisateur.</li><br/><li>Conservez l’ID client. Pour conserver l’identifiant dans une application de navigateur, utilisez un cookie HTTP persistant qui garantit l’utilisation de cet identifiant dans toutes les sessions. N’utilisez pas de cookie de session. Dans le cas d’autres applications, comme des applications mobiles, utilisez le stockage persistant de l’appareil pour conserver cet identifiant.<br /><br/>La prochaine fois que l’utilisateur utilisera votre application sur cet appareil, vous récupérerez l’ID client que vous aurez conservé.</li></ul><br /> **REMARQUE :** Les réponses de Bing ne comportent pas forcément cet en-tête. Si elles l’incluent, capturez l’ID client et utilisez-le pour toutes les demandes Bing suivantes concernant l’utilisateur sur cet appareil.<br /><br /> **REMARQUE :** Si vous insérez l’en-tête X-MSEdge-ClientID, n’incluez pas les cookies dans la requête.|  
|<a name="clientip"></a>X-MSEdge-ClientIP|En-tête de demande facultatif.<br /><br /> Adresse IPv4 ou IPv6 de l’appareil client. L’adresse IP est utilisée pour découvrir l’emplacement de l’utilisateur. Bing utilise les informations de localisation pour déterminer le comportement de recherche approprié.<br /><br /> **REMARQUE :** Nous vous conseillons de toujours indiquer cet en-tête et l’en-tête X-Search-Location, bien qu’ils soient facultatifs.<br /><br /> N’obfusquez pas l’adresse (par exemple, en remplaçant le dernier octet par 0). Cela aurait pour effet d’éloigner la localisation de l’emplacement réel de l’appareil, ce qui pourrait conduire Bing à retourner des résultats erronés.|  
|<a name="location"></a>X-Search-Location|En-tête de demande facultatif.<br /><br /> Liste délimitée par des points-virgules de paires clé/valeur qui décrivent la situation géographique du client. Bing utilise les informations de localisation pour déterminer le comportement de recherche approprié et retourner le contenu local pertinent. Spécifiez la paire clé/valeur sous la forme \<key\>:\<value\>. Voici les clés permettant de spécifier l’emplacement de l’utilisateur.<br /><br /><ul><li>lat &mdash; Latitude de l’emplacement du client, en degrés. Elle doit être supérieure ou égale à -90,0 et inférieure ou égale à +90,0. Les valeurs négatives indiquent les latitudes sud et les valeurs positives les latitudes nord.<br /><br /></li><li>long &mdash; Longitude de l’emplacement du client, en degrés. Elle doit être supérieure ou égale à -180,0 et inférieure ou égale à +180,0. Les valeurs négatives indiquent les longitudes occidentales et les valeurs positives les longitudes orientales.<br /><br /></li><li>re &mdash; Rayon, en mètres, qui spécifie la précision horizontale des coordonnées. Transmettez la valeur retournée par le service de localisation de l’appareil. Voici quelques valeurs courantes : 22 m pour le GPS/Wi-Fi, 380 m pour la triangulation des tours cellulaires et 18 000 m pour la recherche IP inversée.<br /><br /></li><li>ts &mdash; Horodatage UTC UNIX correspondant au moment où le client était à cette position. (L’horodatage UNIX est le nombre de secondes écoulées depuis le 1er janvier 1970.)<br /><br /></li><li>head &mdash; Facultatif. Direction relative du client en déplacement. Spécifiez-la en degrés, entre 0 et 360, dans le sens horaire par rapport au nord géographique. N’indiquez cette clé que si la clé `sp` est différente de zéro.<br /><br /></li><li>sp &mdash; Vélocité (vitesse) horizontale, en mètres par seconde, à laquelle se déplace l’appareil du client.<br /><br /></li><li>alt &mdash; Altitude de l’appareil du client, en mètres.<br /><br /></li><li>are &mdash; Facultatif. Rayon, en mètres, qui spécifie la précision verticale des coordonnées. La valeur par défaut est 50 km. N’indiquez cette clé que si vous spécifiez la clé `alt`.<br /><br /></li></ul> **REMARQUE :** Bien que la plupart des clés soient facultatives, plus vous fournirez d’informations, plus les résultats de localisation seront précis.<br /><br /> **REMARQUE :** Nous vous conseillons de toujours indiquer la situation géographique de l’utilisateur, bien qu’elle soit facultative. C’est particulièrement important si l’adresse IP du client ne reflète pas exactement l’emplacement physique de l’utilisateur (par exemple, si le client utilise un VPN). Pour des résultats optimaux, précisez cet en-tête et l’en-tête X-MSEdge-ClientIP ; au minimum, indiquez cet en-tête.|

> [!NOTE] 
> N’oubliez pas que les conditions d’utilisation imposent le respect de toutes les lois en vigueur, y compris en ce qui concerne l’utilisation de ces en-têtes. Par exemple, dans certaines juridictions, comme en Europe, il faut obtenir le consentement de l’utilisateur pour pouvoir placer certains dispositifs de suivi sur les appareils des utilisateurs.
  

## <a name="query-parameters"></a>Paramètres de requête  
La demande peut comporter les paramètres de requête suivants. Consultez la colonne Requis pour savoir lesquels sont obligatoires. Vous devez encoder les paramètres de requête au format URL.  
  
  
|Nom|Valeur|Type|Obligatoire|  
|----------|-----------|----------|--------------|
|<a name="count"></a>count|Nombre de résultats à retourner, en commençant par l’index spécifié par le paramètre `offset`.|String|Non|   
|<a name="localCategories"></a>localCategories|Liste des options qui définissent la recherche par catégorie d’entreprise.  Voir [Recherche par catégories d’entreprises locales](local-categories.md).|String|Non|  
|<a name="mkt"></a>mkt|Marché d’où proviennent les résultats. <br /><br />Pour connaître la liste des valeurs de marché possibles, voir Codes de marché.<br /><br /> **REMARQUE :** Actuellement, l’API Recherche d’entreprises locales ne prend en charge que le marché et la langue en-us.<br /><br />|String|Oui|
|<a name="offset"></a>offset|Index de début des résultats spécifiés par le paramètre `count`.|Integer|Non|  
|<a name="query"></a>q|Critère de recherche de l’utilisateur.|String|Non|  
|<a name="responseformat"></a>responseFormat|Type de média à utiliser pour la réponse. Voici les valeurs possibles. Elles ne sont pas sensibles à la casse.<br /><ul><li>JSON</li><li>JSONLD</li></ul><br /> La valeur par défaut est JSON. Pour plus d’informations sur les objets JSON que contient la réponse, voir [Objets de la réponse](#response-objects).<br /><br />  Si vous spécifiez JsonLd, le corps de la réponse comporte les objets JSON-LD contenant les résultats de la recherche. Pour plus d’informations sur la spécification JSON-LD, voir [JSON-LD](https://json-ld.org/).|String|Non|  
|<a name="safesearch"></a>safeSearch|Filtre utilisé pour filtrer le contenu pour adultes. Voici les valeurs possibles. Elles ne sont pas sensibles à la casse.<br /><ul><li>Désactivé &mdash; Retourner les pages web comportant du texte, des images ou des vidéos pour adultes.<br /><br/></li><li>Modéré &mdash; Retourner les pages web comportant du texte pour adultes, mais pas des images ou des vidéos pour adultes.<br /><br/></li><li>Strict &mdash; Ne pas retourner de pages web comportant du texte, des images ou des vidéos pour adultes.</li></ul><br /> La valeur par défaut est Modéré.<br /><br /> **REMARQUE :** Si la demande provient d’un marché où la stratégie de Bing en matière de contenu pour adultes exige que `safeSearch` ait la valeur Strict, Bing ignore la valeur `safeSearch` et utilise Strict.<br/><br/>**REMARQUE :** Si vous utilisez l’opérateur de requête `site:`, il est possible que la réponse présente du contenu pour adultes, et ce quel que soit le paramètre de requête `safeSearch` défini. N’utilisez `site:` que si vous connaissez le contenu du site et si votre scénario accepte le contenu pour adultes. |String|Non|  
|<a name="setlang"></a>setLang|Langue à utiliser pour les chaînes de l’interface utilisateur. Spécifiez la langue en utilisant le code de langue ISO 639-1 à deux lettres. Par exemple, celui de l’anglais est EN. La valeur par défaut est EN (anglais).<br /><br /> Nous vous conseillons de toujours indiquer la langue, bien qu’elle soit facultative. En général, on définit `setLang` sur la langue spécifiée par `mkt`, sauf si l’utilisateur souhaite que les chaînes de l’interface utilisateur soient affichées dans une autre langue.<br /><br /> Ce paramètre et l’en-tête [Accept-Language](#acceptlanguage) s’excluent mutuellement &mdash; ne spécifiez pas les deux.<br /><br /> Une chaîne d’interface utilisateur est une chaîne utilisée comme étiquette dans une interface utilisateur. Les objets de réponse JSON en comportent quelques-unes. En outre, les liens vers les propriétés Bing.com dans les objets de la réponse s’appliquent à la langue spécifiée.|String|Non| 


## <a name="response-objects"></a>Objets de la réponse  
Voici les objets de réponse JSON que peut inclure la réponse. Si la demande aboutit, l’objet de niveau supérieur dans la réponse est l’objet [SearchResponse](#searchresponse). Si la demande échoue, l’objet de niveau supérieur est l’objet [ErrorResponse](#errorresponse).


|Object|Description|  
|------------|-----------------|  
|[Place](#place)|Informations relatives à une entreprise locale, comme un restaurant ou un hôtel.|  

  
### <a name="error"></a>Error  
Définit l’erreur qui s’est produite.  
  
|Élément|Description|Type|  
|-------------|-----------------|----------|  
|<a name="error-code"></a>code|Code d’erreur identifiant la catégorie de l’erreur. Pour connaître la liste des codes possibles, voir [Codes d’erreur](#error-codes).|String|  
|<a name="error-message"></a>message|Description de l’erreur.|String|  
|<a name="error-moredetails"></a>moreDetails|Description de l’erreur comportant des informations supplémentaires.|String|  
|<a name="error-parameter"></a>parameter|Paramètre de requête de la demande ayant provoqué l’erreur.|String|  
|<a name="error-subcode"></a>subCode|Code identifiant l’erreur. Par exemple, si `code` est InvalidRequest, `subCode` peut être ParameterInvalid ou ParameterInvalidValue. |String|  
|<a name="error-value"></a>value|Valeur du paramètre de requête qui n’était pas valide.|String|  
  

### <a name="errorresponse"></a>ErrorResponse  
Objet de niveau supérieur figurant dans la réponse en cas d’échec de la demande.  
  
|Nom|Valeur|Type|  
|----------|-----------|----------|  
|_type|Indicateur de type.|String|  
|<a name="errors"></a>errors|Liste des erreurs qui décrivent les raisons pour lesquelles la demande a échoué.|[Error](#error)[]|  

  
  
### <a name="license"></a>Licence  
Définit la licence sous laquelle il est possible d’utiliser le texte ou la photo.  
  
|Nom|Valeur|Type|  
|----------|-----------|----------|  
|name|Nom de la licence.|String|  
|url|URL vers un site web sur lequel l’utilisateur trouvera plus d’informations sur la licence.<br /><br /> Utilisez le nom et l’URL pour créer un lien hypertexte.|String|  


### <a name="link"></a>Lien  
Définit les composants d’un lien hypertexte.  
  
|Nom|Valeur|Type|  
|----------|-----------|----------|  
|_type|Indicateur de type.|String|  
|text|Texte d’affichage.|String|  
|url|URL. Utilisez l’URL et le texte d’affichage pour créer un lien hypertexte.|String|  
  


  
### <a name="organization"></a>Organisation  
Définit un éditeur.  
  
Notez qu’un éditeur peut indiquer son nom, son site web ou les deux.  
  
|Nom|Valeur|Type|  
|----------|-----------|----------|  
|name|Nom de l’éditeur.|String|  
|url|URL du site web de l’éditeur.<br /><br /> Notez que l’éditeur peut ne pas indiquer de site web.|String|  
  
  

### <a name="place"></a>Emplacement  
Informations relatives à une entreprise locale, comme un restaurant ou un hôtel.  
  
|Nom|Valeur|Type|  
|----------|-----------|----------|  
|_type|Indicateur de type, qui peut avoir les valeurs suivantes :<br /><br /><ul><li>Hotel</li><li>LocalBusiness<br /></li><li>Restaurant</ul><li>|String|  
|address|Adresse postale où se trouve l’entité.|PostalAddress|  
|entityPresentationInfo|Informations supplémentaires sur l’entité (par exemple, indicateurs permettant de déterminer le type de l’entité : restaurant, hôtel, etc.). Le champ `entityScenario` est défini sur ListItem.|EntityPresentationInfo|  
|name|Nom de l’entité.|String|  
|telephone|Numéro de téléphone de l’entité.|String|  
|url|URL du site web de l’entité.<br /><br /> Utilisez-la avec le nom de l’entité pour créer un lien hypertexte qui amène l’utilisateur sur le site web de l’entité.|String|  
|webSearchUrl|URL des résultats de recherche de Bing sur ce lieu.|String| 
  
  
### <a name="querycontext"></a>QueryContext  
Définit le contexte de requête utilisé par Bing pour la demande.  
  
|Élément|Description|Type|  
|-------------|-----------------|----------|  
|adultIntent|Valeur booléenne qui indique si la requête spécifiée est destinée à des adultes. La valeur est **true** si c’est le cas, **false** sinon.|Boolean|  
|alterationOverrideQuery|Chaîne de requête à utiliser pour forcer Bing à utiliser la chaîne d’origine. Par exemple, si la chaîne de requête est *saling downwind*, la chaîne de requête de remplacement sera *+saling downwind*. N’oubliez pas d’encoder la chaîne de requête, ce qui donne *%2Bsaling+downwind*.<br /><br /> Ce champ n’est précisé que si la chaîne de requête d’origine contient une faute d’orthographe.|String|  
|alteredQuery|Chaîne de requête utilisée par Bing pour exécuter la requête. Bing utilise la chaîne de requête modifiée si la chaîne de requête d’origine contenait des fautes d’orthographe. Par exemple, si la chaîne de requête est `saling downwind`, la chaîne de requête modifiée sera `sailing downwind`.<br /><br /> Ce champ n’est précisé que si la chaîne de requête d’origine contient une faute d’orthographe.|String|  
|askUserForLocation|Valeur booléenne qui indique si Bing exige que l’emplacement de l’utilisateur donne des résultats précis. Si vous avez spécifié l’emplacement de l’utilisateur à l’aide des en-têtes [X-MSEdge-ClientIP](#clientip) et [X-Search-Location](#location), vous pouvez ignorer ce champ.<br /><br /> Dans le cas des requêtes pour lesquelles l’emplacement de l’utilisateur doit fournir des résultats précis, comme « météo du jour » ou « restaurants proches », ce champ est défini sur **true**.<br /><br /> Dans le cas des requêtes qui précisent l’emplacement (par exemple, « météo Seattle »), ce champ est défini sur **false**. Ce champ est également défini sur **false** pour les requêtes qui ne dépendent pas de la localisation, comme « meilleurs ventes ».|Boolean|  
|originalQuery|Chaîne de requête telle que spécifiée dans la demande.|String|  

### <a name="identifiable"></a>Identifiable

|Nom|Valeur|Type|  
|-------------|-----------------|----------|
|id|Identificateur de ressource.|String|
 
### <a name="rankinggroup"></a>RankingGroup
Définit un groupe de résultats de la recherche, par exemple, mainline.

|Nom|Valeur|Type|  
|-------------|-----------------|----------|
|items|Liste de résultats de la recherche à afficher dans le groupe.|RankingItem|

### <a name="rankingitem"></a>RankingItem
Définit un élément de résultat de recherche à afficher.

|Nom|Valeur|Type|  
|-------------|-----------------|----------|
|resultIndex|Index base zéro de l’élément de la réponse à afficher. Si l’élément ne comporte pas ce champ, affiche tous les éléments de la réponse. Par exemple, affiche tous les articles dans la réponse Actualités.|Integer|
|answerType|Réponse qui contient l’élément à afficher. Par exemple, Actualités.<br /><br />Utilisez le type pour trouver la réponse dans l’objet SearchResponse. Le type est le nom d’un champ SearchResponse.<br /><br /> Toutefois, n’utilisez le type de réponse que si cet objet inclut le champ de valeur ; sinon, ignorez-le.|String|
|textualIndex|Index de la réponse dans textualAnswers à afficher.| Entier non signé|
|value|ID qui identifie une réponse ou un élément de réponse à afficher. Si l’ID identifie une réponse, affiche tous les éléments de la réponse.|Identifiable|

### <a name="rankingresponse"></a>RankingResponse  
Définit où le contenu doit être placé sur la page de résultats de la recherche et dans quel ordre.  
  
|Nom|Valeur|  
|----------|-----------|  
|<a name="ranking-mainline"></a>mainline|Résultats de la recherche à afficher dans la partie principale.|  
|<a name="ranking-pole"></a>pole|Résultats de la recherche qui doivent être les plus visibles (par exemple, affichés au-dessus de la partie principale et de l’encadré).|  
|<a name="ranking-sidebar"></a>sidebar|Résultats de la recherche à afficher dans l’encadré.| 

### <a name="searchresponse"></a>SearchResponse  
Définit l’objet de niveau supérieur figurant dans la réponse en cas de réussite de la demande.  
  
Notez que, si le service suspecte une attaque par déni de service, la demande réussira (le code d’état HTTP est 200 OK) ; toutefois, le corps de la réponse sera vide.  
  
|Nom|Valeur|Type|  
|----------|-----------|----------|  
|_type|Indicateur de type, défini sur SearchResponse.|String|  
|places|Liste d’entités pertinentes pour la requête de recherche.|Objet JSON|  
|queryContext|Objet contenant la chaîne de requête utilisée par Bing pour la demande.<br /><br /> Il comporte la chaîne de requête entrée par l’utilisateur. La chaîne de requête utilisée par Bing pour la requête peut également être une chaîne de requête modifiée, s’il y avait une faute d’orthographe dans la chaîne de requête.|[QueryContext](#querycontext)|  


## <a name="error-codes"></a>Codes d’erreur

Voici les codes d’état HTTP qu’une demande peut retourner.  
  
|Code d’état|Description|  
|-----------------|-----------------|  
|200|Réussite.|  
|400|L’un des paramètres de requête est manquant ou non valide.|  
|401|La clé d’abonnement est manquante ou non valide.|  
|403|L’utilisateur est authentifié (par exemple, il a utilisé une clé d’abonnement valide), mais il n’est pas autorisé à accéder à la ressource demandée.<br /><br /> Bing peut également retourner cet état si l’appelant a dépassé son quota mensuel de requêtes.|  
|410|La demande a utilisé le protocole HTTP au lieu du protocole HTTPS. HTTPS est le seul protocole pris en charge.|  
|429|L’appelant a dépassé son quota de requêtes par seconde.|  
|500|Événement serveur inattendu.|

Si la demande échoue, la réponse comporte un objet [ErrorResponse](#errorresponse) qui contient une liste d’objets [Error](#error) décrivant l’origine de l’erreur. Si l’erreur est liée à un paramètre, le champ `parameter` identifie celui qui pose problème. De même, si l’erreur est liée à une valeur de paramètre, le champ `value` identifie la valeur non valide.

```json
{
  "_type": "ErrorResponse", 
  "errors": [
    {
      "code": "InvalidRequest", 
      "subCode": "ParameterMissing", 
      "message": "Required parameter is missing.", 
      "parameter": "q" 
    }
  ]
}

{
  "_type": "ErrorResponse", 
  "errors": [
    {
      "code": "InvalidAuthorization", 
      "subCode": "AuthorizationMissing", 
      "message": "Authorization is required.", 
      "moreDetails": "Subscription key is not recognized."
    }
  ]
}
```

Voici les valeurs possibles de code d’erreur et de sous-code d’erreur.

|Code|SubCode|Description
|-|-|-
|ServerError|UnexpectedError<br/>ResourceError<br/>NotImplemented|Le code d’état HTTP est 500.
|InvalidRequest|ParameterMissing<br/>ParameterInvalidValue<br/>HttpNotAllowed<br/>Bloqué|Bing retourne InvalidRequest à chaque fois que la demande comporte une partie non valide. Par exemple, un paramètre obligatoire est manquant ou une valeur de paramètre n’est pas valide.<br/><br/>Si l’erreur est ParameterMissing ou ParameterInvalidValue, le code d’état HTTP est 400.<br/><br/>Si vous utilisez le protocole HTTP au lieu du protocole HTTPS, Bing retourne HttpNotAllowed, et le code d’état HTTP est 410.
|RateLimitExceeded|Aucun sous-code|Bing retourne RateLimitExceeded chaque fois que vous dépassez votre quota de requêtes par seconde (QPS) ou par mois (QPM).<br/><br/>Si vous dépassez votre QPS, Bing retourne le code d’état HTTP 429 ; si vous dépassez votre QPM, Bing retourne le code d’état 403.
|InvalidAuthorization|AuthorizationMissing<br/>AuthorizationRedundancy|Bing retourne InvalidAuthorization s’il ne parvient pas à identifier l’appelant. Par exemple, l’en-tête `Ocp-Apim-Subscription-Key` est manquant ou la clé d’abonnement n’est pas valide.<br/><br/>Il y a redondance si plusieurs méthodes d’authentification sont spécifiées.<br/><br/>Si l’erreur est InvalidAuthorization, le code d’état HTTP est 401.
|InsufficientAuthorization|AuthorizationDisabled<br/>AuthorizationExpired|Bing retourne InsufficientAuthorization lorsque l’appelant n’est pas autorisé à accéder à la ressource. Cela peut se produire si la clé d’abonnement a été désactivée ou a expiré. <br/><br/>Si l’erreur est InsufficientAuthorization, le code d’état HTTP est 403.

## <a name="next-steps"></a>Étapes suivantes
- [Démarrage rapide de la Recherche d’entreprises locales](quickstarts/local-quickstart.md)
- [Démarrage rapide Java de la Recherche d’entreprises locales](quickstarts/local-search-java-quickstart.md)
- [Démarrage rapide Node de la Recherche d’entreprises locales](quickstarts/local-search-node-quickstart.md)
- [Démarrage rapide Python de la Recherche d’entreprises locales](quickstarts/local-search-python-quickstart.md)