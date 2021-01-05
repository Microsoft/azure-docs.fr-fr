---
title: Variables HTTP pour le moteur de règles Azure CDN | Microsoft Docs
description: Découvrez les variables HTTP qui vous permettent d’obtenir des métadonnées sur les requêtes et réponses HTTP pour certaines fonctionnalités du moteur de règles. Utilisez des métadonnées pour modifier une demande/réponse.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/09/2018
ms.author: allensu
ms.openlocfilehash: a2d9fc98ba6f514afbd88e543a859a69e0fc6c6b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88192681"
---
# <a name="http-variables-for-azure-cdn-rules-engine"></a>Variables HTTP pour le moteur de règles Azure CDN
Les variables HTTP vous permettent de récupérer les métadonnées de requête et de réponse HTTP. Ces métadonnées peuvent ensuite servir à modifier une requête ou une réponse de façon dynamique. L’utilisation de variables HTTP est limitée aux fonctionnalités de moteur de règles suivantes :

- [Cache-Key Rewrite](https://docs.vdms.com/cdn/Content/HRE/F/Cache-Key-Rewrite.htm)
- [Modify Client Request Header](https://docs.vdms.com/cdn/Content/HRE/F/Modify-Client-Request-Header.htm)
- [Modify Client Response Header](https://docs.vdms.com/cdn/Content/HRE/F/Modify-Client-Response-Header.htm)
- [URL Redirect](https://docs.vdms.com/cdn/Content/HRE/F/URL-Redirect.htm)
- [URL Rewrite](https://docs.vdms.com/cdn/Content/HRE/F/URL-Rewrite.htm)

## <a name="definitions"></a>Définitions
Le tableau suivant décrit les variables HTTP prises en charge. Une valeur vide est retournée lorsque les métadonnées géographiques (par exemple, code postal) ne sont pas disponibles pour une requête particulière.


| Nom | Variable | Description | Exemple de valeur |
| ---- | -------- | ----------- | ------------ |
| ASN (demandeur) | %{geo_asnum} | Indique le numéro de système autonome du demandeur. <br /><br />**Déprécié :** %{virt_dst_asnum}. <br />Cette variable est dépréciée au profit de %{geo_asnum}. Bien que les règles utilisant cette variable dépréciée continuent à fonctionner, vous devez les mettre à jour pour utiliser la nouvelle variable. | AS15133 |
| Ville (demandeur) | %{geo_city} | Indique la ville du demandeur. | Los Angeles |
| Continent (demandeur) | %{geo_continent} | Indique le continent du demandeur au travers de son abréviation. <br />Les valeurs autorisées sont : <br />AF : Afrique<br />AS : Asia<br />EU : Europe<br />NA : Amérique du Nord<br />OC : Océanie<br />SA : Amérique du Sud<br /><br />**Déprécié :** %{virt_dst_continent}. <br />Cette variable est dépréciée au profit de %{geo_continent}. <br />Bien que les règles utilisant cette variable dépréciée continuent à fonctionner, vous devez les mettre à jour pour utiliser la nouvelle variable.| N/A |
| Valeur du cookie | %{cookie_Cookie} | Retourne la valeur correspondant à la clé de cookie identifiée par le terme Cookie. | Exemple d’utilisation : <br />%{cookie__utma}<br /><br />Exemple de valeur :<br />111662281.2.10.1222100123 |
| Pays/région (demandeur) | %{geo_country} | Indique le pays/la région d’origine du demandeur au travers de son préfixe international. <br />**Déprécié :** %{virt_dst_country}. <br /><br />Cette variable est dépréciée au profit de %{geo_country}. Bien que les règles utilisant cette variable dépréciée continuent à fonctionner, vous devez les mettre à jour pour utiliser la nouvelle variable. | US |
| Designated Market Area (demandeur) | %{geo_dma_code} |Spécifie le marché des médias du demandeur au moyen de son code de région. <br /><br />Ce champ est uniquement applicable aux requêtes provenant des États-Unis.| 745 |
| Méthode de requête HTTP | %{request_method} | Indique la méthode de requête HTTP. | GET |
| Code d’état HTTP | %{status} | Indique le code d’état HTTP de la réponse. | 200 |
| Adresse IP (demandeur) | %{virt_dst_addr} | Indique l’adresse IP du demandeur. | 192.168.1.1 |
| Latitude (demandeur) | %{geo_latitude} | Indique la latitude du demandeur. | 34.0995 |
| Longitude (demandeur) | %{geo_longitude} | Indique la longitude du demandeur. | -118.4143 |
| Metropolitan Statistical Area (demandeur) | %{geo_metro_code} | Indique la zone métropolitaine du demandeur. <br /><br />Ce champ est uniquement applicable aux requêtes provenant des États-Unis.<br />| 745 |
| Port (demandeur) | %{virt_dst_port} | Indique le port éphémère du demandeur. | 55885 |
| Code postal (demandeur) | %{geo_postal_code} | Indique le code postal du demandeur. | 90210 |
| Chaîne de requête trouvée | %{is_args} | La valeur de cette variable change selon que la demande contient une chaîne de requête.<br /><br />- Chaîne de requête trouvée : ?<br />- Aucune chaîne de requête : NULL | ? |
| Paramètre de chaîne de requête trouvé | %{is_amp} | La valeur de cette variable change selon que la demande contient au moins un paramètre de chaîne de requête.<br /><br />- Paramètre trouvé : &<br />- Aucun paramètre : NULL | & |
| Valeur de paramètre de chaîne de requête | %{arg_&lt;parameter&gt;} | Retourne la valeur correspondant au paramètre de chaîne de requête identifié par le terme &lt;parameter&gt;. | Exemple d’utilisation : <br />%{arg_language}<br /><br />Exemple de paramètre de chaîne de requête : <br />?language=en<br /><br />Exemple de valeur : en |
| Valeur de chaîne de requête | %{query_string} | Indique la valeur de chaîne de requête entière définie dans l’URL de requête. |key1=val1&key2=val2&key3=val3 |
| Domaine du point d’accès | %{referring_domain} | Indique le domaine défini dans l’en-tête de requête Referrer. | <www.google.com> |
| Région (demandeur) | %{geo_region} | Indique la région du demandeur (par exemple, état ou province) au travers de son abréviation alphanumérique. | CA |
| Valeur d’en-tête de requête | %{http_RequestHeader} | Retourne la valeur correspondant à l’en-tête de requête identifié par le terme RequestHeader. <br /><br />Si le nom de l’en-tête de requête contient un tiret (par exemple, User-Agent), remplacez-le par un trait de soulignement (par exemple, User_Agent).| Exemple d’utilisation : %{http_Connection}<br /><br />Exemple de valeur : Keep-Alive | 
| Hôte de la requête | %{host} | Indique l’hôte défini dans l’URL de requête. | <www.mondomaine.com> |
| Protocole de requête | %{request_protocol} | Indique le protocole de la requête. | HTTP/1.1 |
| Modèle de requête | %{scheme} | Indique le schéma de la requête. |http |
| URI de requête (relatif) | %{request_uri} | Indique le chemin d’accès relatif, y compris la chaîne de requête, défini dans l’URI de requête. | /marketing/foo.js?loggedin=true |
| URI de requête (relatif sans chaîne de requête) | %{uri} | Indique le chemin d’accès relatif vers le contenu demandé. <br /><br/>Informations essentielles :<br />- Ce chemin d’accès relatif exclut la chaîne de requête.<br />- Ce chemin d’accès relatif reflète les réécritures d’URL. Une URL est réécrite dans les conditions suivantes :<br />  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- Fonctionnalité de réécriture d’URL : cette fonctionnalité réécrit le chemin d’accès relatif défini dans l’URI de requête.<br />    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- Edge CNAME URL (URL du CNAME de périmètre) : ce type de requête est réécrit sur l’URL du CDN correspondant. |/800001/corigin/rewrittendir/foo.js |
| URI de demande | %{request} | Décrit la requête. <br />Syntaxe : &lt;Méthode HTTP&gt; &lt;chemin relatif&gt; &lt;protocole HTTP&gt; | GET /marketing/foo.js?loggedin=true HTTP/1.1 |
| Valeur d’en-tête de réponse | %{resp_&lt;ResponseHeader&gt;} | Retourne la valeur correspondant à l’en-tête de réponse identifié par le terme &lt;ResponseHeader&gt;. <br /><br />Si le nom de l’en-tête de réponse contient un tiret (par exemple, User-Agent), remplacez-le par un trait de soulignement (par exemple, User_Agent). | Exemple d’utilisation : %{resp_Content_Length}<br /><br />Exemple de valeur : 100 |

## <a name="usage"></a>Usage
Le tableau suivant décrit la syntaxe permettant de spécifier les variables HTTP.


| Syntaxe | Exemple | Description |
| ------ | -------- | ---------- |
| %{&lt;HTTPVariable&gt;} | %{host} | Utilisez la syntaxe suivante pour obtenir la valeur entière correspondant à la variable &lt;HTTPVariable&gt; spécifiée. |
| %{&lt;HTTPVariableDelimiter&gt;} | %{host,} | Utilisez la syntaxe suivante pour définir la casse de la valeur entière correspondant à la variable &lt;HTTPVariableDelimiter&gt; spécifiée. |
| %{&lt;HTTPVariableDelimiterExpression&gt;} | %{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$2.$3:80} | Utilisez une expression régulière pour &lt;HTTPVariableDelimiterExpression&gt; afin de remplacer, supprimer ou manipuler la valeur d’une variable HTTP. |

Les noms de variables HTTP prennent uniquement en charge des traits de soulignement et les caractères alphabétiques. Convertissez les caractères non pris en charge en traits de soulignement.

## <a name="delimiter-reference"></a>Référence de délimiteur
Un délimiteur peut être spécifié après une variable HTTP pour apporter l’un des effets suivants :

- Transformer la valeur associée à la variable.

     Exemple : Convertit la valeur entière en minuscules.

- Supprimer la valeur associée à la variable.

- Manipuler la valeur associée à la variable.

     Exemple : Utilise des expressions régulières pour modifier la valeur associée à la variable HTTP.

Les délimiteurs sont décrits dans le tableau suivant.

| Délimiteur | Description |
| --------- | ----------- |
| := | Indique qu’une valeur par défaut sera assignée à la variable lorsqu’elle est : <br />- en-tête manquante ; <br />- valeur de l’en-tête définie sur Null. |
| :+ | Indique qu’une valeur par défaut sera assignée à la variable lorsqu’une valeur lui aura été assignée. |
| : | Indique qu’une sous-chaîne de la valeur assignée à la variable sera développée. |
| # | Indique que le modèle spécifié après ce délimiteur doit être supprimé lorsqu’il se trouve au début de la valeur associée à la variable. |
| % | Indique que le modèle spécifié après ce délimiteur doit être supprimé lorsqu’il se trouve à la fin de la valeur associée à la variable. <br />Cette définition s’applique uniquement lorsque le symbole % est utilisé comme délimiteur. |
| / | Délimite une variable HTTP ou un modèle. |
| // | Recherche et remplace toutes les instances du modèle spécifié. |
| /= | Recherche, copie et réécrit toutes les occurrences du modèle spécifié. |
| , | Convertit la valeur associée à la variable HTTP en minuscules. |
| ^ | Convertit la valeur associée à la variable HTTP en majuscules. |
| ,, | Convertit toutes les instances du caractère spécifié dans la valeur associée à la variable HTTP en minuscules. |
| ^^ | Convertit toutes les instances du caractère spécifié dans la valeur associée à la variable HTTP en majuscules. |

## <a name="exceptions"></a>Exceptions
Le tableau suivant décrit les circonstances dans lesquelles le texte spécifié n’est pas traité comme une variable HTTP.

| Condition | Description | Exemple |
| --------- | ----------- | --------|
| Échappement du symbole % | Le symbole de pourcentage peut faire l’objet d’un échappement au moyen d’une barre oblique inverse. <br />L’exemple de valeur à droite sera considéré comme une valeur littérale et non comme une variable HTTP.| \%{host} |
| Variables inconnues | Une chaîne vide est toujours retournée pour les variables inconnues. | % {unknown_variable} |
| Caractères ou syntaxe non valides | Les variables qui contiennent des caractères ou une syntaxe non valides sont traitées comme des valeurs littérales. <br /><br />Exemple 1 : la valeur spécifiée contient un caractère non valide (par exemple, -). <br /><br />Exemple 2 : la valeur spécifiée contient un double ensemble d’accolades. <br /><br />Exemple 3 : il manque une accolade fermante à la valeur spécifiée.<br /> | Exemple 1 : %{resp_user-agent} <br /><br />Exemple 2 : %{{host}} <br /><br />Exemple 3 : %{host |
| Nom de la variable manquant | Une valeur Null est toujours retournée lorsqu’une variable n’est pas spécifiée. | %{} |
| Caractères de fin | Les caractères de fin d’une variable sont traités comme des valeurs littérales. <br />L’exemple de valeur à droite contient une accolade de fin qui sera traitée comme une valeur littérale. | %{host}} |

## <a name="setting-default-header-values"></a>Paramétrage des valeurs d’en-tête par défaut
Une valeur par défaut peut être assignée à un en-tête dans les conditions suivantes :
- en-tête manquant/non défini ;
- valeur de l’en-tête définie sur Null.

Le tableau suivant décrit comment définir une valeur par défaut.

| Condition | Syntaxe | Exemple | Description |
| --------- | ------ | --------| ----------- |
| Définition d’une valeur par défaut pour un en-tête dans les conditions suivantes : <br /><br />- en-tête manquant ; <br /><br />- valeur d’en-tête définie sur Null.| %{Variable:=Value} | %{http_referrer:=unspecified} | L’en-tête Referrer est uniquement défini sur *unspecified* quand il est manquant ou défini avec la valeur Null. Aucune action n’est effectuée s’il a été défini. |
| Définition d’un en-tête à une valeur par défaut lorsqu’il est manquant. | %{Variable=Value} | %{http_referrer=unspecified} | L’en-tête Referrer est uniquement défini sur *unspecified* quand il est manquant. Aucune action n’est effectuée s’il a été défini. |
| Définition d’une valeur par défaut pour un en-tête quand il ne satisfait aucune des conditions suivantes : <br /><br />- en-tête manquante ;<br /><br /> - valeur de l’en-tête définie sur Null. | %{Variable:+Value} | %{http_referrer:+unspecified} | L’en-tête Referrer est uniquement défini sur *unspecified* lorsqu’une valeur lui a été assignée. Aucune action n’est effectuée s’il est manquant ou défini avec la valeur Null. |

## <a name="manipulating-variables"></a>Manipulation de variables
Les variables peuvent être manipulées comme suit :

- Développement de sous-chaînes
- Suppression de modèles

### <a name="substring-expansion"></a>Développement de sous-chaîne
Par défaut, une variable se développe à sa valeur complète. Utilisez la syntaxe suivante pour développer uniquement une sous-chaîne de la valeur de la variable.

`%<Variable>:<Offset>:<Length>}`

Informations essentielles :

- La valeur assignée au terme Offset détermine le caractère de début de la sous-chaîne :

     - Positive : le caractère de début de la sous-chaîne est calculé à partir du premier caractère de la chaîne.
     - Zéro : le caractère de début de la sous-chaîne est le premier caractère de la chaîne.
     - Négatif : le caractère de début de la sous-chaîne est calculé à partir du dernier caractère de la chaîne.

- La longueur de la sous-chaîne est déterminée par le terme *Length* :

     - Omise : l’omission du terme Length permet à la sous-chaîne d’inclure tous les caractères entre le caractère de début et la fin de la chaîne.
     - Positive : détermine la longueur de la sous-chaîne à partir du premier caractère vers la droite.
     - Négatif : détermine la longueur de la sous-chaîne à partir du premier caractère vers la gauche.

#### <a name="example"></a>Exemple :

L’exemple suivant s’appuie sur l’exemple d’URL de requête suivant :

https:\//cdn.mydomain.com/folder/marketing/myconsultant/proposal.html

La chaîne suivante illustre différentes méthodes de manipulation des variables :

https:\//www%{http_host:3}/mobile/%{request_uri:7:10}/%{request_uri:-5:-8}.htm

En fonction de l’exemple d’URL de requête, la manipulation de variables ci-dessus produit la valeur suivante :

https:\//www.mydomain.com/mobile/marketing/proposal.htm


### <a name="pattern-removal"></a>Suppression du modèle
Le texte correspondant à un modèle spécifique peut être supprimé du début ou de la fin de la valeur d’une variable.

| Syntaxe | Action |
| ------ | ------ |
| %{Variable#Pattern} | Supprime le texte lorsque le modèle spécifié est localisé au début de la valeur d’une variable. |
| %{Variable%Pattern} | Supprime le texte lorsque le modèle spécifié est localisé à la fin de la valeur d’une variable. |

#### <a name="example"></a>Exemple :

Dans cet exemple de scénario, la variable *request_uri* est définie sur :

`/800001/myorigin/marketing/product.html?language=en-US`

Le tableau suivant illustre le fonctionnement de cette syntaxe.

| Exemple de syntaxe | Résultats | Description |
| ------------- | ------- | --- |
| %{request_uri#/800001}/customerorigin | /customerorigin/myorigin/marketing/product.html?language=en-US | Étant donné que la variable commence par le modèle, ce dernier a été remplacé. |
| %{request_uri%html}htm | /800001/myorigin/marketing/product.html?language=en-US | Étant donné que la variable ne se termine pas par le modèle, il n’y a aucune modification.|

### <a name="find-and-replace"></a>Rechercher et remplacer
La syntaxe de recherche et de remplacement est décrite dans le tableau suivant.

| Syntaxe | Action |
| ------ | ------ |
| %{Variable/Find/Replace} | Recherche et remplace la première occurrence du modèle spécifié. |
| %{Variable//Find/Replace} | Recherche et remplace toutes les occurrences du modèle spécifié. |
| %{Variable^} |Convertit la valeur entière en majuscules. |
| %{Variable^Find} | Convertit la première occurrence du modèle spécifié en majuscules. |
| %{Variable,} | Convertit la valeur entière en minuscules. |
| %{Variable,Find} | Convertit la première occurrence du modèle spécifié en minuscules. |

### <a name="find-and-rewrite"></a>Rechercher et réécrire
Pour une variante de la recherche et du remplacement, utilisez le texte correspondant au modèle spécifié lors de sa réécriture. La syntaxe de recherche et de réécriture est décrite dans le tableau suivant.

| Syntaxe | Action |
| ------ | ------ |
| %{Variable/=Find/Rewrite} | Recherche, copie et réécrit toutes les occurrences du modèle spécifié. |
| %{Variable/^Find/Rewrite} | Recherche, copie et réécrit le modèle spécifié lorsqu’il apparaît au début de la variable. |
| %{Variable/$Find/Rewrite} | Recherche, copie et réécrit le modèle spécifié lorsqu’il apparaît à la fin de la variable. |
| %{Variable/Find} | Recherche et supprime toutes les occurrences du modèle spécifié. |

Informations essentielles :

- Développez le texte correspondant au modèle spécifié en indiquant un signe dollar suivi d’un nombre entier (par exemple, $1).

- Plusieurs modèles peuvent être spécifiés. L’ordre dans lequel le modèle est spécifié détermine l’entier qui lui sera assigné. Dans l’exemple suivant, le premier modèle correspond à « www. », le deuxième modèle correspond au domaine de second niveau, et le troisième modèle correspond au domaine de niveau supérieur :

    `%{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$2.$3:80}`

- La valeur réécrite peut se composer de n’importe quelle combinaison de texte et de ces espaces réservés.

    Dans l’exemple précédent, le nom d’hôte est réécrit de la manière suivante : `cdn.$2.$3:80` (par exemple, cdn.mydomain.com:80).

- La casse d’un espace réservé de modèle (par exemple, $1) peut être modifiée au moyen des indicateurs suivants :
     - U : majuscules pour la valeur développée.

         Exemple de syntaxe :

         `%{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$U2.$3:80}`

     - L : minuscules pour la valeur développée.

         Exemple de syntaxe :

         `%{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$L2.$3:80}`

- Un opérateur doit être spécifié avant le modèle. L’opérateur spécifié détermine le comportement de capture de modèle :

     - `=`: indique que toutes les occurrences du modèle spécifié doivent être capturées et réécrites.
     - `^`: indique que seul le texte commençant par le modèle spécifié est capturé.
     - `$`: indique que seul le texte se terminant par le modèle spécifié est capturé.
 
- Si vous omettez la valeur */Rewrite* valeur, le texte correspondant au modèle est supprimé.


