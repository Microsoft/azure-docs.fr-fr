---
title: Fonctionnement de la mise en cache | Microsoft Docs
description: La mise en cache est le processus de stockage local des données afin que les requêtes futures relatives à ces dernières soient plus rapidement accessibles.
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
ms.date: 04/30/2018
ms.author: allensu
ms.openlocfilehash: a226682c2580a871e1b2fc4db71f369f3bcc3abb
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010161"
---
# <a name="how-caching-works"></a>Comment la mise en cache fonctionne

Cet article fournit une vue d’ensemble des concepts généraux de mise en cache et de la manière dont [Microsoft Azure Content Delivery Network (CDN)](cdn-overview.md) utilise la mise en cache pour améliorer le niveau de performance. Si vous souhaitez en savoir plus sur la manière de personnaliser le comportement de mise en cache sur votre point de terminaison CDN, consultez [Contrôle du comportement de mise en cache du CDN Azure avec règles de mise en cache](cdn-caching-rules.md) et [Contrôle du comportement de mise en cache du CDN Azure avec des chaînes de requête](cdn-query-string.md).

## <a name="introduction-to-caching"></a>Introduction de la mise en cache

La mise en cache est le processus de stockage local des données afin que les requêtes futures relatives à ces dernières soient plus rapidement accessibles. Dans le type le plus courant de mise en cache, la mise en cache du navigateur web, un navigateur web stocke localement des copies des données statiques sur un disque dur local. À l’aide de la mise en cache, le navigateur web peut éviter d’effectuer plusieurs allers-retours au serveur et, au lieu de cela, accéder localement aux mêmes données, ce qui permet d’économiser du temps et des ressources. La mise en cache est bien adaptée à la gestion locale de données statiques de petite taille, telles que les images statiques, les fichiers CSS et les fichiers JavaScript.

De même, la mise en cache est utilisée par un réseau de distribution de contenu sur des serveurs Edge proches de l’utilisateur afin d’éviter le retour de requêtes à leur l’origine et la réduction de la latence de l’utilisateur final. Contrairement à un cache de navigateur web, qui est utilisé uniquement pour un seul utilisateur, le CDN utilise un cache partagé. Dans un cache CDN partagé, un fichier demandé par un utilisateur sera ultérieurement accessibles à d’autres utilisateurs, ce qui réduit considérablement le nombre de requêtes au serveur d’origine.

Les ressources dynamiques qui changent fréquemment ou sont propres à un utilisateur individuel ne peuvent pas être mises en cache. Toutefois, ces types de ressources peuvent bénéficier de l’optimisation de l’accélération de site dynamique (DSA) sur Microsoft Azure Content Delivery Network pour améliorer leurs performances.

La mise en cache peut avoir lieu à plusieurs niveaux entre le serveur d’origine et l’utilisateur final :

- Serveur web : utilise un cache partagé (pour plusieurs utilisateurs).
- Réseau de distribution de contenu : utilise un cache partagé (pour plusieurs utilisateurs).
- Fournisseur de services Internet (ISP) : utilise un cache partagé (pour plusieurs utilisateurs).
- Navigateur web : utilise un cache privé (pour un utilisateur).

En général, chaque cache gère l’actualisation de ses propres ressources et effectue une validation lorsqu’un fichier est périmé. Ce comportement est défini dans la spécification de mise en cache HTTP, [RFC 7234](https://tools.ietf.org/html/rfc7234).

### <a name="resource-freshness"></a>Actualisation des ressources

Une ressource mise en cache pouvant être obsolète ou périmée (par rapport à la ressource correspondante sur le serveur d’origine), il est important pour tout mécanisme de mise en cache de contrôler quand le contenu est actualisé. Pour réduire la consommation de temps et de bande passante, une ressource mise en cache n’est pas comparée à la version sur le serveur d’origine lors de chaque accès. Au lieu de cela, tant qu’une ressource mise en cache est considérée comme actuelle, elle est censée être la version la plus récente et est envoyée directement au client. Une ressource mise en cache est considérée comme actualisée lorsque son âge est inférieur à la période ou à l’âge définis par un paramètre de cache. Par exemple, lorsqu’un navigateur recharge une page web, il vérifie que chaque ressource mise en cache sur votre disque dur est actualisée et la charge. Si la ressource n’est pas actualisée (périmée), une copie à jour est chargée à partir du serveur.

### <a name="validation"></a>Validation

Si une ressource est considérée comme périmée, le serveur d’origine est invité à la valider, autrement dit, à déterminer si les données du cache correspondent toujours à celles du serveur d’origine. Si le fichier a été modifié sur le serveur d’origine, le cache met à jour sa version de la ressource. Sinon, si la ressource est actualisée, les données sont fournies directement à partir du cache sans validation préalable.

### <a name="cdn-caching"></a>Mise en cache du CDN

La mise en cache fait partie intégrante de la façon dont un CDN fonctionne pour accélérer la distribution et réduire la charge d’origine des ressources statiques, telles que des images, des polices et des vidéos. Dans la mise en cache du CDN, les ressources statiques sont stockées de manière sélective sur des serveurs placés stratégiquement plus près d’un utilisateur, et elle présente les avantages suivants :

- Étant donné que la majorité du trafic web est statique (par exemple, des images, des polices et des vidéos), la mise en cache du CDN réduit la latence du réseau en déplaçant le contenu plus près de l’utilisateur, ce qui réduit la distance de déplacement des données.

- En déchargeant le travail sur un CDN, elle peut réduire le trafic réseau et la charge sur le serveur d’origine. Cela réduit les coûts et les ressources requises pour l’application, même en présence d’un grand nombre d’utilisateurs.

Comme dans un navigateur web, vous pouvez contrôler comment la mise en cache est effectuée dans un CDN en envoyant des en-têtes de la directive du cache. Les en-têtes de la directive du cache sont des en-têtes HTTP qui sont généralement ajoutés par le serveur d’origine. Bien que la plupart de ces en-têtes aient été initialement conçus pour traiter la mise en cache dans des navigateurs clients, ils sont désormais également utilisés par tous les caches intermédiaires, tels que les CDN. 

Deux en-têtes peuvent être utilisés pour définir l’actualisation du cache : `Cache-Control` et `Expires`. `Cache-Control` est plus récent et est prioritaire sur `Expires`, le cas échéant. Il existe également deux types d’en-tête utilisés pour la validation (appelés validateurs) : `ETag` et `Last-Modified`. `ETag` est plus récent et est prioritaire sur `Last-Modified` si les deux sont définis.  

## <a name="cache-directive-headers"></a>En-têtes de la directive du cache

> [!IMPORTANT]
> Par défaut, un point de terminaison Azure CDN optimisé pour DSA ignore les en-têtes de la directive du cache et la mise en cache. En ce qui concerne les profils **Azure CDN Standard de Verizon** et **Azure CDN Standard d’Akamai**, vous pouvez ajuster la manière dont un point de terminaison Azure CDN traite ces en-têtes en utilisant des [règles de mise en cache CDN](cdn-caching-rules.md) pour activer la mise en cache. Dans le cas des profils **Azure CDN Premium de Verizon**, vous utiliserez pour cela le [moteur de règles](./cdn-verizon-premium-rules-engine.md).

Azure CDN prend en charge les en-têtes de la directive de cache HTTP suivants, qui définissent la durée et le partage du cache.

**Cache-Control :**
- Introduit dans HTTP 1.1 pour permettre aux éditeurs web de mieux contrôler leur contenu et de traiter les limitations de l’en-tête `Expires`.
- Remplace l’en-tête `Expires` si lui et `Cache-Control` sont définis.
- Lorsqu’elle est utilisée dans une requête HTTP du client vers les emplacements des points de présence (POP) de CDN, la directive `Cache-Control` est ignorée par l’ensemble des profils Azure CDN, par défaut.
- Lorsqu’elle est utilisée dans une réponse HTTP du client vers les emplacements POP de CDN :
     - **Azure CDN Standard/Premium de Verizon** et **Azure CDN Standard de Microsoft** prennent en charge l’ensemble des directives `Cache-Control`.
     - **Azure CDN Standard d’Akamai** prend uniquement en charge les directives `Cache-Control` suivantes ; toutes les autres sont ignorées :
         - `max-age`: Un cache peut stocker le contenu pendant le nombre de secondes spécifié. Par exemple : `Cache-Control: max-age=5`. Cette directive spécifie le maximum de temps durant lequel le contenu est considéré comme actualisé.
         - `no-cache`: Mise en cache du contenu, qui doit cependant être validé avant chaque distribution à partir du cache. Équivaut à `Cache-Control: max-age=0`.
         - `no-store`: Ne jamais mettre le contenu en cache. Supprimer le contenu s’il a été préalablement stocké.

**Expires :**
- En-tête hérité introduit dans HTTP 1.0 ; pris en charge pour la compatibilité descendante.
- Utilise une heure d’expiration basés sur une date avec une précision à la seconde. 
- Semblable à `Cache-Control: max-age`.
- Utilisé lorsque `Cache-Control` n’existe pas.

**Pragma :**
   - Par défaut, pas honoré par Azure CDN.
   - En-tête hérité introduit dans HTTP 1.0 ; pris en charge pour la compatibilité descendante.
   - Utilisé comme en-tête de requête client avec la directive suivante : `no-cache`. Cette directive indique au serveur de fournir une version actualisée de la ressource.
   - `Pragma: no-cache` équivaut à `Cache-Control: no-cache`.

## <a name="validators"></a>Validateurs

Lorsque le cache est périmé, les validateurs de cache HTTP sont utilisés pour comparer la version mise en cache d’un fichier avec la version sur le serveur d’origine. **Azure CDN Standard/Premium de Verizon** prend en charge les validateurs `ETag` et `Last-Modified` par défaut, tandis qu’**Azure CDN Standard de Microsoft** et **Azure CDN Standard d’Akamai** prennent en charge uniquement `Last-Modified` par défaut.

**ETag :**
- **Azure CDN Standard/Premium de Verizon** prend en charge `ETag` par défaut, ce qui n’est pas le cas d’**Azure CDN Standard de Microsoft** et de **Azure CDN Standard d’Akamai**.
- `ETag` définit une chaîne unique pour chaque fichier et chaque version d’un fichier. Par exemple : `ETag: "17f0ddd99ed5bbe4edffdd6496d7131f"`.
- Introduit dans HTTP 1.1 et plus actuel que `Last-Modified`. Utile lorsque la date de dernière modification est difficile à déterminer.
- Prend en charge à la fois la validation forte et la validation faible ; toutefois, Azure CDN prend en charge uniquement la validation forte. Pour la validation forte, les deux représentations de la ressource doivent être identiques à l’octet près. 
- Un cache valide un fichier qui utilise `ETag` en envoyant un en-tête `If-None-Match` dont la requête contient un ou plusieurs validateurs `ETag`. Par exemple : `If-None-Match: "17f0ddd99ed5bbe4edffdd6496d7131f"`. Si la version du serveur correspond à un validateur `ETag` dans la liste, il envoie le code d’état 304 (non modifié) dans sa réponse. Si la version est différente, le serveur répond avec le code d’état 200 (OK) et la ressource mise à jour.

**Last-Modified :**
- Pour **Azure CDN Standard/Premium de Verizon** uniquement, la directive `Last-Modified` est utilisée si `ETag` ne fait pas partie de la réponse HTTP. 
- Spécifie la date et l’heure auxquelles le serveur d’origine a déterminé la dernière modification de la ressource. Par exemple : `Last-Modified: Thu, 19 Oct 2017 09:28:00 GMT`.
- Un cache valide un fichier en utilisant `Last-Modified` en envoyant un en-tête `If-Modified-Since` dont la requête contient une date et une heure. Le serveur d’origine compare la date avec l’en-tête `Last-Modified` de la ressource la plus récente. Si la ressource n’a pas été modifiée depuis l’heure spécifiée, le serveur renvoie le code d’état 304 (non modifié) dans sa réponse. Si la ressource a été modifiée, le serveur retourne le code d’état 200 (OK) et la ressource mise à jour.

## <a name="determining-which-files-can-be-cached"></a>Déterminer quels fichiers peuvent être mis en cache

Toutes les ressources ne peuvent pas être mises en cache. Le tableau suivant montre quelles ressources peuvent être mises en cache, en fonction du type de réponse HTTP. Les ressources fournies avec des réponses HTTP ne remplissant pas toutes ces conditions ne peuvent pas être mises en cache. Dans le cas **d’Azure CDN Premium de Verizon** uniquement, il est possible d’utiliser le moteur de règles pour personnaliser certaines de ces conditions.

|                       | Azure CDN de Microsoft          | Azure CDN de Verizon | Azure CDN d’Akamai        |
|-----------------------|-----------------------------------|------------------------|------------------------------|
| **Codes d’état HTTP** | 200, 203, 206, 300, 301, 410, 416 | 200                    | 200, 203, 300, 301, 302, 401 |
| **Méthodes HTTP**      | GET, HEAD                         | GET                    | GET                          |
| **Limites de taille de fichiers**  | 300 Go                            | 300 Go                 | - Optimisation de la livraison web générale : 1,8 Go<br />- Optimisation de la diffusion multimédia en continu : 1,8 Go<br />- Optimisation des fichiers volumineux : 150 Go |

Pour permettre l’activation de la mise en cache d’**Azure CDN Standard de Microsoft** sur une ressource, le serveur d’origine doit prendre en charge les requêtes HTTP HEAD et GET et les valeurs content-length doivent être identiques pour l’ensemble des réponses HTTP HEAD et GET associées à la ressource. Dans le cas d’une requête HEAD, le serveur d’origine doit prendre en charge la requête et répondre avec les en-têtes qu’il aurait utilisé s’il avait reçu une requête GET.

## <a name="default-caching-behavior"></a>Comportement de mise en cache par défaut

Le tableau suivant décrit le comportement de mise en cache par défaut des produits Azure CDN et de leurs optimisations.

|    | Microsoft : Livraison web générale | Verizon : Livraison web générale | Verizon : DSA | Akamai : Livraison web générale | Akamai : DSA | Akamai : Téléchargement de fichiers volumineux | Akamai : diffusion multimédia en continu générale ou VOD |
|------------------------|--------|-------|------|--------|------|-------|--------|
| **Honorer l’origine**       | Oui    | Oui   | Non   | Oui    | Non   | Oui   | Oui    |
| **Durée de cache CDN** | 2 jours |7 jours | None | 7 jours | None | 1 jour | 1 an |

**Honorer l’origine** : indique s’il faut honorer les en-têtes de la directive du cache pris en charge s’il y en a dans la réponse HTTP du serveur d’origine.

**Durée de cache du CDN** : indique le temps pendant lequel une ressource est mise en cache sur Azure CDN. Toutefois, si **Honorer l’origine** est Oui et que la réponse HTTP du serveur d’origine inclut l’en-tête de la directive de cache `Expires` ou `Cache-Control: max-age`, Azure CDN utilise la valeur de la durée spécifiée par l’en-tête à la place. 

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment personnaliser et remplacer le comportement de mise en cache par défaut sur CDN par le biais de règles de mise en cache, consultez [Contrôler le comportement de mise en cache d’Azure CDN avec des règles de mise en cache](cdn-caching-rules.md). 
- Pour savoir comment utiliser des chaînes de requête pour contrôler le comportement de mise en cache, consultez [Contrôler le comportement de mise en cache d’Azure CDN avec des chaînes de requête](cdn-query-string.md).