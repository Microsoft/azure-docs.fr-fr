---
title: Utilisation d’Azure CDN avec CORS | Microsoft Docs
description: Découvrez comment utiliser Azure CDN (Content Delivery Network) avec CORS (Cross-Origin Resource Sharing).
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: 86740a96-4269-4060-aba3-a69f00e6f14e
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: f7edf790e526329dd285d03a31137a26220e52ee
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018645"
---
# <a name="using-azure-cdn-with-cors"></a>Utilisation d’Azure CDN avec CORS
## <a name="what-is-cors"></a>Présentation de CORS
CORS (Cross Origin Resource Sharing) est une fonctionnalité HTTP qui permet à une application web exécutée dans un domaine d’accéder aux ressources d’un autre domaine. Pour réduire le risque d’attaques de script entre sites, tous les navigateurs web modernes implémentent une restriction de sécurité appelée [stratégie de même origine](https://www.w3.org/Security/wiki/Same_Origin_Policy).  Celle-ci empêche une page web d’appeler des API dans un autre domaine.  CORS permet à une origine (le domaine d’origine) d’appeler des API dans un autre domaine de façon sécurisée.

## <a name="how-it-works"></a>Fonctionnement
Il existe deux types de demandes CORS : les *demandes simples* et *les demandes complexes*.

### <a name="for-simple-requests"></a>Pour les demandes simples :

1. Le navigateur envoie la demande CORS avec un en-tête de demande HTTP d’**origine** supplémentaire. La valeur de cet en-tête est l’origine qui a servi la page mère, définie comme la combinaison du *protocole,* du *domaine* et du *port*.  Lorsqu’une page du site https\://www.contoso.com tente d’accéder aux données d’un utilisateur dans l’origine fabrikam.com, l’en-tête de requête suivant est envoyé à fabrikam.com :

   `Origin: https://www.contoso.com`

2. Le serveur peut renvoyer les éléments suivants :

   * Un en-tête **Access-Control-Allow-Origin** indiquant le site d’origine autorisé. Par exemple :

     `Access-Control-Allow-Origin: https://www.contoso.com`

   * Un code d’erreur HTTP comme 403 si le serveur n’autorise pas la demande d’origine croisée après avoir vérifié l’en-tête d’origine.

   * Un en-tête **Access-Control-Allow-Origin** avec un caractère générique qui autorise toutes les origines :

     `Access-Control-Allow-Origin: *`

### <a name="for-complex-requests"></a>Pour les demandes complexes :

Une demande complexe est une demande CORS où le navigateur est nécessaire pour envoyer une *demande préliminaire* (c’est-à-dire une sonde préliminaire) avant d’envoyer la demande CORS. La demande préliminaire demande l’autorisation au serveur si la demande CORS d’origine peut se poursuivre et est une demande `OPTIONS` transmise à la même URL.

> [!TIP]
> Pour plus d’informations sur les flux CORS et les pièges les plus courants, consultez le [Authoritative guide to CORS (Cross-Origin Resource Sharing) for REST APIs (Guide de référence sur CORS (Cross-Origin Resource Sharing) pour les API REST)](https://www.moesif.com/blog/technical/cors/Authoritative-Guide-to-CORS-Cross-Origin-Resource-Sharing-for-REST-APIs/).
>
>

## <a name="wildcard-or-single-origin-scenarios"></a>Scénarios avec caractère générique ou à origine unique
CORS sur Azure CDN fonctionne automatiquement sans aucune configuration supplémentaire quand l’en-tête **Access-Control-Allow-Origin** est défini sur le caractère générique (*) ou une origine unique.  Le CDN met en cache la première réponse, et les demandes suivantes utilisent le même en-tête.

Si les demandes sont communiquées au CDN avant que CORS ne soit défini sur l’origine, vous devez vider le contenu de votre point de terminaison pour recharger le contenu avec l’en-tête **Access-Control-Allow-Origin**.

## <a name="multiple-origin-scenarios"></a>Scénarios avec plusieurs origines
Si une liste d’origines spécifique doit être autorisée pour CORS, les choses se compliquent un peu plus. Le problème se produit quand le CDN met en cache l’en-tête **Access-Control-Allow-Origin** pour la première origine CORS.  Quand une autre origine CORS envoie une demande ultérieure, le CDN utilise l’en-tête **Access-Control-Allow-Origin** mis en cache, qui ne correspond pas.  Il existe plusieurs façons de corriger cette situation.

### <a name="azure-cdn-standard-profiles"></a>Profils CDN Azure standard
Sur Azure CDN standard de Microsoft, vous pouvez créer une règle dans le [moteur de règles standard](cdn-standard-rules-engine-reference.md) pour vérifier l’en-tête **Origin** sur la demande. S’il s’agit d’une origine valide, votre règle définit l’en-tête **Access-Control-Allow-Origin** avec la valeur souhaitée. Dans ce cas, l’en-tête **Access-Control-Allow-Origin** issu du serveur d’origine du fichier est ignoré et le moteur de règles du CDN gère entièrement les origines CORS autorisées.

![Exemple de règles avec le moteur de règles standard](./media/cdn-cors/cdn-standard-cors.png)

> [!TIP]
> Vous pouvez ajouter des actions supplémentaires à votre règle pour modifier des en-têtes de réponse supplémentaires, par exemple **Access-Control-Allow-Methods**.
> 

Sur **Azure CDN Standard fourni par Akamai**, le seul mécanisme autorisant plusieurs origines sans recourir à l’origine avec caractère générique consiste à utiliser la [mise en cache de chaîne de requête](cdn-query-string.md). Activez le paramètre de chaîne de requête pour le point de terminaison CDN, puis utilisez une chaîne de requête unique pour les demandes à partir de chaque domaine autorisé. Ainsi, le CDN met en cache un objet distinct pour chaque chaîne de requête unique. Cette approche n’est pas idéale, toutefois, car plusieurs copies du même fichier sont mises en cache sur le CDN.  

### <a name="azure-cdn-premium-from-verizon"></a>CDN Azure Premium fourni par Verizon
À l’aide du moteur de règles Verizon Premium, vous devez [créer une règle](./cdn-verizon-premium-rules-engine.md) pour vérifier l’en-tête **Origin** dans la demande.  S’il s’agit d’une origine valide, votre règle définit l’en-tête **Access-Control-Allow-Origin** avec l’origine fournie dans la demande.  Si l’origine spécifiée dans l’en-tête **Origin** n’est pas autorisée, votre règle doit omettre l’en-tête **Access-Control-Allow-Origin**, ce qui amène le navigateur à rejeter la demande. 

Vous pouvez mettre en place cette procédure de deux façons avec le moteur de règles Premium. Dans les deux cas, l’en-tête **Access-Control-Allow-Origin** issu du serveur d’origine du fichier est ignoré et le moteur de règles du CDN gère entièrement les origines CORS autorisées.

#### <a name="one-regular-expression-with-all-valid-origins"></a>Une expression régulière avec toutes les origines valides
Dans ce cas, vous allez créer une expression régulière qui inclut toutes les origines que vous souhaitez autoriser : 

```http
https?:\/\/(www\.contoso\.com|contoso\.com|www\.microsoft\.com|microsoft.com\.com)$
```

> [!TIP]
> **CDN Azure Premium fourni par Verizon** utilise [PCRE (Perl Compatible Regular Expressions)](https://pcre.org/) comme moteur pour les expressions régulières.  Vous pouvez utiliser un outil tel que [Regular Expressions 101](https://regex101.com/) pour valider votre expression régulière.  Notez que le caractère « / » est valide dans les expressions régulières et n’a pas besoin d’être échappé ; toutefois, échapper ce caractère est une pratique recommandée et attendue par certains validateurs d’expression régulière.
> 
> 

Si l’expression régulière correspond, votre règle remplace l’en-tête **Access-Control-Allow-Origin** (le cas échéant) de l’origine par l’origine qui a envoyé la demande.  Vous pouvez également ajouter des en-têtes CORS, comme **Access-Control-Allow-Methods**.

![Exemple de règles avec expression régulière](./media/cdn-cors/cdn-cors-regex.png)

#### <a name="request-header-rule-for-each-origin"></a>Règle d’en-tête de demande pour chaque origine
Au lieu de recourir à des expressions régulières, vous pouvez créer une règle pour chaque origine à autoriser en utilisant la [condition de correspondance](/previous-versions/azure/mt757336(v=azure.100)#match-conditions) **Request Header Wildcard** (Caractère générique de l’en-tête de demande). Comme dans le cas de la méthode des expressions régulières, seul le moteur de règles définit les en-têtes CORS. 

![Exemple de règles sans expression régulière](./media/cdn-cors/cdn-cors-no-regex.png)

> [!TIP]
> Dans l’exemple ci-dessus, l’utilisation du caractère générique * indique au moteur de règles que HTTP et HTTPS satisfont tous les deux à la condition.
> 
>