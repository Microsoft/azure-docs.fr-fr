---
title: Fonctionnalités du moteur de règles Azure CDN de Verizon Premium
description: Documentation de référence sur les fonctionnalités du moteur de règles Azure CDN de Verizon Premium.
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: article
ms.date: 06/02/2020
ms.author: allensu
ms.openlocfilehash: 0ea4f167b992ccfbc4156ac06c8f636d2ef4a355
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020413"
---
# <a name="azure-cdn-from-verizon-premium-rules-engine-features"></a>Fonctionnalités du moteur de règles Azure CDN de Verizon Premium

Cet article comprend les descriptions détaillées des fonctionnalités du [moteur de règles](cdn-verizon-premium-rules-engine.md) Azure Content Delivery Network (CDN).

La troisième partie d’une règle est la fonctionnalité. Une fonctionnalité définit le type d’action appliqué au type de requête qui est identifié par un ensemble de conditions de correspondance.

## <a name="azure-cdn-from-verizon-premium-rules-engine-features-reference"></a><a name="top"></a>Informations de référence sur les fonctionnalités du moteur de règles Azure CDN de Verizon Premium

Les types de fonctionnalités disponibles sont les suivants :

* [Accès](#access)
* [Mise en cache](#caching)
* [Commentaire](#comment)
* [En-têtes](#headers)
* [Journaux d’activité](#logs)
* [Optimize](#optimize)
* [Origine](#origin)
* [Spécialité](#specialty)
* [URL](#url)
* [Pare-feu d’applications web](#waf)

### <a name="access"></a><a name="access"></a>Accès

Ces fonctionnalités sont conçues pour contrôler l’accès au contenu.

| Nom       | Objectif                                                           |
|------------|-------------------------------------------------------------------|
| [Deny Access (403)](https://docs.vdms.com/cdn/Content/HRE/F/Deny-Access-403.htm) | Détermine si toutes les requêtes sont rejetées avec une réponse 403 Interdit. |
| [Token Auth](https://docs.vdms.com/cdn/Content/HRE/F/Token-Auth.htm) | Détermine si l’authentification basée sur les jetons est appliquée à une requête. |
| [Token Auth Denial Code](https://docs.vdms.com/cdn/Content/HRE/F/Token-Auth-Denial-Code.htm) | Détermine le type de réponse à retourner à un utilisateur quand une requête est refusée en raison de l’authentification basée sur les jetons. |
| [Token Auth Ignore URL Case](https://docs.vdms.com/cdn/Content/HRE/F/Token-Auth-Ignore-URL-Case.htm) | Détermine si les comparaisons d’URL effectuées par l’authentification basée sur les jetons respectent la casse. |
| [Token Auth Parameter](https://docs.vdms.com/cdn/Content/HRE/F/Token-Auth-Parameter.htm) | Détermine si le paramètre de chaîne de requête de l’authentification basée sur les jetons doit être renommé. |

**[Retour au début](#top)**

### <a name="caching"></a><a name="caching"></a>Mise en cache

Ces fonctionnalités sont conçues pour personnaliser la mise en cache du contenu.

| Nom       | Objectif                                                           |
|------------|-------------------------------------------------------------------|
| [Bandwidth Parameters](https://docs.vdms.com/cdn/Content/HRE/F/Bandwidth-Parameters.htm) | Détermine si les paramètres de limitation de bande passante (c.-à-d., ec_rate et ec_prebuf) sont actifs. |
| [Bandwidth Throttling](https://docs.vdms.com/cdn/Content/HRE/F/Bandwidth-Throttling.htm) | Limite la bande passante pour la réponse fournie par nos serveurs Edge. |
| [Bypass Cache](https://docs.vdms.com/cdn/Content/HRE/F/Bypass-Cache.htm) | Détermine si la requête peut exploiter notre technologie de mise en cache. |
| [Cache-Control Header Treatment](https://docs.vdms.com/cdn/Content/HRE/F/Cache-Control-Header-Treatment.htm) |  Contrôle la génération des en-têtes de contrôle de cache par le serveur Edge quand la fonctionnalité Âge maximal externe est active. |
| [Cache-Key Query String](https://docs.vdms.com/cdn/Content/HRE/F/Cache-Key-Query-String.htm) | Détermine si la **clé de cache** _ inclut ou exclut les paramètres de chaîne de requête associés à une demande. <br> __ Chemin d’accès relatif qui identifie de façon unique un composant à des fins de mise en cache.  Nos serveurs Edge utilisent ce chemin d’accès relatif lors de la vérification du contenu mis en cache.  Par défaut, une clé de cache ne contient pas de paramètres de chaîne de requête._ |
| [Cache-Key Rewrite](https://docs.vdms.com/cdn/Content/HRE/F/Cache-Key-Rewrite.htm) | Réécrit la clé de cache associée à une requête. |
| [Complete Cache Fill](https://docs.vdms.com/cdn/Content/HRE/F/Complete-Cache-Fill.htm) | Détermine ce qui se passe quand une requête génère une absence de cache partielle sur un serveur Edge. |
| [Compress File Types](https://docs.vdms.com/cdn/Content/HRE/F/Compress-File-Types.htm) | Définit les formats de fichier à compresser sur le serveur. | 
| [Default Internal Max-Age](https://docs.vdms.com/cdn/Content/HRE/F/Default-Internal-Max-Age.htm) | Détermine l’intervalle d’âge maximal par défaut pour la revalidation du cache entre le serveur Edge et le serveur d’origine. |
| [Expires Header Treatment](https://docs.vdms.com/cdn/Content/HRE/F/Expires-Header-Treatment.htm) | Contrôle la génération de l’expiration des en-têtes par un serveur Edge quand la fonctionnalité Âge maximal externe est active. |
| [External Max-Age](https://docs.vdms.com/cdn/Content/HRE/F/External-Max-Age.htm) | Détermine l’intervalle d’âge maximal pour la revalidation du cache entre le navigateur et le serveur Edge. |
| [Force Internal Max-Age](https://docs.vdms.com/cdn/Content/HRE/F/Force-Internal-Max-Age.htm) | Détermine l’intervalle d’âge maximal pour la revalidation du cache entre le serveur Edge et le serveur d’origine. |
| [H.264 Support (HTTP Progressive Download)](https://docs.vdms.com/cdn/Content/HRE/F/H.264-Support-HPD.htm) | Détermine les types de formats de fichier H.264 qui peuvent être utilisés pour diffuser du contenu. |
| [H.264 Support Video Seek Params](https://docs.vdms.com/cdn/Content/HRE/F/H.264-Support-VSP-HPD.htm) | Remplace les noms affectés aux paramètres qui contrôlent la recherche via le support H.264 lors de l’utilisation du téléchargement progressif HTTP. |
| [Honor No-Cache Request](https://docs.vdms.com/cdn/Content/HRE/F/Honor-No-Cache-Request.htm) | Détermine si les requêtes non-cache d’un client HTTP sont transmises au serveur d’origine. |
| [Ignore Origin No-Cache](https://docs.vdms.com/cdn/Content/HRE/F/Ignore-Origin-No-Cache.htm) | Détermine si notre CDN ignore certaines directives remises par un serveur d’origine. |
| [Ignore Unsatisfiable Ranges](https://docs.vdms.com/cdn/Content/HRE/F/Ignore-Unsatisfiable-Ranges.htm) | Détermine la réponse à retourner aux clients quand une requête génère un code d’état 416 Plage demandée non satisfaisante. |
| [Internal Max-Stale](https://docs.vdms.com/cdn/Content/HRE/F/Internal-Max-Stale.htm) | Contrôle la durée après l’expiration normale d’une ressource mise en cache pendant laquelle cette ressource peut être remise depuis un serveur Edge quand ce serveur Edge ne parvient pas à la revalider avec le serveur d’origine. |
| [Partial Cache Sharing](https://docs.vdms.com/cdn/Content/HRE/F/Partial-Cache-Sharing.htm) | Détermine si une requête peut générer du contenu partiellement mis en cache. |
| [Prevalidate Cached Content](https://docs.vdms.com/cdn/Content/HRE/F/Prevalidate-Cached-Content.htm) | Détermine si du contenu mis en cache peut faire l’objet d’une revalidation anticipée avant l’expiration de sa durée de vie. |
| [Refresh Zero-Byte Cache Files](https://docs.vdms.com/cdn/Content/HRE/F/Refresh-Zero-Byte-Cache-Files.htm) | Détermine la façon dont nos serveurs Edge gèrent la requête d’un client HTTP liée à une ressource de cache de 0 octet. |
| [Set Cacheable Status Codes](https://docs.vdms.com/cdn/Content/HRE/F/Set-Cacheable-Status-Codes.htm) | Définit l’ensemble des codes d’état qui peuvent générer du contenu mis en cache. |
| [Stale Content Delivery on Error](https://docs.vdms.com/cdn/Content/HRE/F/Stale-Content-Delivery-on-Error.htm) | Détermine si du contenu mis en cache qui a expiré est remis quand une erreur se produit pendant la revalidation du cache ou l’extraction du contenu demandé à partir du serveur d’origine du client. | 
| [Stale While Revalidate](https://docs.vdms.com/cdn/Content/HRE/F/Stale-While-Revalidate.htm) | Améliore les performances en permettant à nos serveurs Edge de rendre obsolète le client pour le demandeur pendant la revalidation. |

**[Retour au début](#top)**

### <a name="comment"></a><a name="comment"></a>Commentaire

La fonctionnalité Commentaire permet d’ajouter une remarque dans une règle.

**[Retour au début](#top)**

### <a name="headers"></a><a name="headers"></a>En-têtes

Ces fonctionnalités sont conçues pour ajouter, modifier ou supprimer des en-têtes de la demande ou réponse.

| Nom       | Objectif                                                           |
|------------|-------------------------------------------------------------------|
| [Age Response Header](https://docs.vdms.com/cdn/Content/HRE/F/Age-Response-Header.htm) | Détermine si un en-tête de réponse Age est inclus dans la réponse envoyée au demandeur. |
| [Debug Cache Response Headers](https://docs.vdms.com/cdn/Content/HRE/F/Debug-Cache-Response-Headers.htm) | Détermine si une réponse peut inclure l’[en-tête de réponse X-EC-Debug](https://docs.vdms.com/cdn/Content/Knowledge_Base/X_EC_Debug.htm) qui fournit des informations sur la stratégie de cache du composant demandé. |
| [Modify Client Request Header](https://docs.vdms.com/cdn/Content/HRE/F/Modify-Client-Request-Header.htm) | Remplace, complète ou supprime un en-tête dans une requête. |
| [Modify Client Response Header](https://docs.vdms.com/cdn/Content/HRE/F/Modify-Client-Response-Header.htm) | Remplace, complète ou supprime un en-tête dans une réponse. |
| [Set Client IP Custom Header](https://docs.vdms.com/cdn/Content/HRE/F/Set-Client-IP-Custom-Header.htm) | Permet d’ajouter l’adresse IP du client demandeur à la requête sous la forme d’en-tête de requête personnalisé. |

**[Retour au début](#top)**

### <a name="logs"></a><a name="logs"></a>Journaux d’activité

Ces fonctionnalités sont conçues pour personnaliser les données stockées dans les fichiers journaux bruts.

| Nom       | Objectif                                                           |
|------------|-------------------------------------------------------------------|
| [Custom Log Field 1](https://docs.vdms.com/cdn/Content/HRE/F/Custom-Log-Field-1.htm) | Détermine le format et le contenu à attribuer au champ de fichier journal personnalisé dans un fichier journal brut. |
| [Log Query String](https://docs.vdms.com/cdn/Content/HRE/F/Log-Query-String.htm) | Détermine si une chaîne de requête est stockée avec l’URL dans les journaux d’activité d’accès. |

**[Retour au début](#top)**

### <a name="optimize"></a><a name="optimize"></a>Optimiser

Ces fonctionnalités déterminent si une requête subit les optimisations fournies par Edge Optimizer.

| Nom       | Objectif                                                           |
|------------|-------------------------------------------------------------------|
| [Edge Optimizer](https://docs.vdms.com/cdn/Content/HRE/F/Edge-Optimizer.htm) | Détermine si Edge Optimizer peut être appliqué à une requête. |
| [Edge Optimizer – Instancier la configuration](https://docs.vdms.com/cdn/Content/HRE/F/Edge-Optimizer-Instantiate-Configuration.htm) | Instancie ou active la configuration Edge Optimizer associée à un site. |

**[Retour au début](#top)**

### <a name="origin"></a><a name="origin"></a>Origine

Ces fonctionnalités sont conçues pour contrôler la manière dont le CDN communique avec un serveur d’origine.

| Nom       | Objectif                                                           |
|------------|-------------------------------------------------------------------|
| [Maximum Keep-Alive Requests](https://docs.vdms.com/cdn/Content/HRE/F/Maximum-Keep-Alive-Requests.htm) | Définit le nombre maximal de requêtes pour une connexion toujours active avant sa fermeture. |
| [Proxy Special Headers](https://docs.vdms.com/cdn/Content/HRE/F/Proxy-Special-Headers.htm) | Définit l’ensemble des [en-têtes de requête spécifiques de CDN](https://docs.vdms.com/cdn/Content/Knowledge_Base/Request-Format.htm#RequestHeaders) qui seront transmis d’un serveur Edge à un serveur d’origine. |

**[Retour au début](#top)**

### <a name="specialty"></a><a name="specialty"></a>Spécialité

Ces fonctionnalités sont des fonctionnalités avancées que seuls les utilisateurs expérimentés doivent utiliser.

| Nom       | Objectif                                                           |
|------------|-------------------------------------------------------------------|
| [Cacheable HTTP Methods](https://docs.vdms.com/cdn/Content/HRE/F/Cacheable-HTTP-Methods.htm) | Détermine l’ensemble des méthodes HTTP supplémentaires pouvant être mises en cache sur notre réseau. |
| [Cacheable Request Body Size](https://docs.vdms.com/cdn/Content/HRE/F/Cacheable-Request-Body-Size.htm) | Définit le seuil permettant de déterminer si une réponse POST peut être mise en cache. |
| [QUIC](https://docs.vdms.com/cdn/Content/HRE/F/QUIC.htm) | Détermine si le client est informé que notre service CDN prend en charge QUIC. |
| [Optimisation de la diffusion en continu](https://docs.vdms.com/cdn/Content/HRE/F/Streaming-Optimization.htm) | Ajuste votre configuration de mise en cache pour optimiser les performances des streams en direct et réduire la charge sur le serveur d’origine. |
| [User Variable](https://docs.vdms.com/cdn/Content/HRE/F/User-Variable.htm) | Affecte une valeur à une variable définie par l’utilisateur qui est transmise à votre solution de traitement du trafic personnalisée. |

**[Retour au début](#top)**

### <a name="url"></a><a name="url"></a>URL

Ces fonctionnalités permettent de rediriger ou de réécrire une requête vers une URL différente.

| Nom       | Objectif                                                           |
|------------|-------------------------------------------------------------------|
| [Follow Redirects](https://docs.vdms.com/cdn/Content/HRE/F/Follow-Redirects.htm) | Détermine si les requêtes peuvent être redirigées vers le nom d’hôte défini dans l’en-tête Location retourné par un serveur d’origine du client. |
| [URL Redirect](https://docs.vdms.com/cdn/Content/HRE/F/URL-Redirect.htm) | Redirige les requêtes via l’en-tête Location. |
| [URL Rewrite](https://docs.vdms.com/cdn/Content/HRE/F/URL-Rewrite.htm) | Réécrit l’URL de la requête. |

**[Retour au début](#top)**

### <a name="web-application-firewall"></a><a name="waf"></a>Pare-feu d’applications web

La fonctionnalité [Pare-feu d’applications web](https://docs.vdms.com/cdn/Content/HRE/F/Web_Application_Firewall.htm) détermine si une requête est filtrée par un pare-feu d’applications web.

**[Retour au début](#top)**

Pour connaître les dernières fonctionnalités, consultez la [Documentation du moteur de règles Verizon](https://docs.vdms.com/cdn/index.html#Quick_References/HRE_QR.htm#Actions).

## <a name="next-steps"></a>Étapes suivantes

- [Informations de référence du moteur de règles](cdn-verizon-premium-rules-engine-reference.md)
- [Expressions conditionnelles du moteur de règles](cdn-verizon-premium-rules-engine-reference-conditional-expressions.md)
- [Conditions de correspondance du moteur de règles](cdn-verizon-premium-rules-engine-reference-match-conditions.md)
- [Remplacement du comportement HTTP à l’aide du moteur de règles](cdn-verizon-premium-rules-engine.md)
- [Vue d’ensemble d’Azure CDN](cdn-overview.md)
