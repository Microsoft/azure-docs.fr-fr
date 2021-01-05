---
title: Azure Front Door – Mise en cache | Microsoft Docs
description: Cet article explique le comportement d’Azure Front Door avec les règles de routage ayant la mise en cache activée.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/29/2020
ms.author: duau
ms.openlocfilehash: 1a8064c3ff89c0bc8b0ceb5249492b912c219ce8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91535829"
---
# <a name="caching-with-azure-front-door"></a>Mise en cache avec Azure Front Door
Le document suivant explique comment spécifier les comportements d’une porte d’entrée à l’aide de règles de routage ayant la mise en cache activée. Front Door est un réseau de distribution de contenu (CDN) moderne qui, outre l’accélération de site dynamique et l’équilibrage de charge, prend en charge les comportements de mise en cache comme n’importe quel autre CDN.

## <a name="delivery-of-large-files"></a>Distribution de fichiers volumineux
Azure Front Door permet de distribuer des fichiers volumineux sans aucune plafond de taille de fichier. Front Door utilise une technique appelée segmentation d’objets. Quand un fichier volumineux est demandé, la porte d’entrée récupère les petits éléments du fichier auprès du backend. Après avoir reçu une demande de fichier complet ou de plage d’octets, l’environnement Front Door demande le fichier du serveur principale par blocs de 8 Mo.

</br>Dès qu’un bloc parvient à l’environnement Front Door, il est mis en cache et immédiatement servi à l’utilisateur. La porte d’entrée se prépare ensuite à récupérer le bloc suivant en parallèle. Cette prérécupération est l’assurance que le contenu a un bloc d’avance sur l’utilisateur, ce qui réduit la latence. Ce processus se poursuit jusqu’à ce que le fichier entier soit téléchargé (si nécessaire) ou que le client ferme la connexion.

</br>Pour plus d’informations sur la demande de plage d’octets, consultez [RFC 7233](https://web.archive.org/web/20171009165003/http://www.rfc-base.org/rfc-7233.html).
Front Door met en cache les blocs au fur et à mesure de leur réception, ce qui évite d’avoir à mettre l’intégralité du fichier dans le cache Front Door. Les demandes subséquentes du fichier ou de plages d’octets sont servies à partir du cache. Si certains blocs ne sont pas mis en cache, ils sont demandés au serveur principal via une prérécupération. Cette optimisation s’appuie sur la capacité du serveur principal à prendre en charge les demandes de plages d’octets. Si le serveur principal ne prend pas en charge les demandes de plages d’octets, cette optimisation n’est pas effective.

## <a name="file-compression"></a>Compression de fichiers
Front Door peut compresser de manière dynamique le contenu en périphérie, ce qui a pour effet de réduire la taille et le délai de la réponse envoyée à vos clients. Tous les fichiers peuvent faire l’objet d’une compression. Cependant, un fichier doit être de type MIME pour être éligible pour la compression. À l’heure actuelle, Front Door n’autorise pas la modification de cette liste. Voici la liste actuelle :</br>
- "application/eot"
- "application/font"
- "application/font-sfnt"
- "application/javascript"
- "application/json"
- "application/opentype"
- "application/otf"
- "application/pkcs7-mime"
- "application/truetype"
- "application/ttf",
- "application/vnd.ms-fontobject"
- "application/xhtml+xml"
- "application/xml"
- "application/xml+rss"
- "application/x-font-opentype"
- "application/x-font-truetype"
- "application/x-font-ttf"
- "application/x-httpd-cgi"
- "application/x-mpegurl"
- "application/x-opentype"
- "application/x-otf"
- "application/x-perl"
- "application/x-ttf"
- "application/x-javascript"
- "font/eot"
- "font/ttf"
- "font/otf"
- "font/opentype"
- "image/svg+xml"
- "text/css"
- "text/csv"
- "text/html"
- "text/javascript"
- "text/js", "text/plain"
- "text/richtext"
- "text/tab-separated-values"
- "text/xml"
- "text/x-script"
- "text/x-component"
- "text/x-java-source"

Par ailleurs, la taille du fichier doit être comprise entre 1 Ko et 8 Mo (inclus).

Ces profils prennent en charge les encodages de compression suivants :
- [Gzip (GNU zip)](https://en.wikipedia.org/wiki/Gzip)
- [Brotli](https://en.wikipedia.org/wiki/Brotli)

Si une demande prend en charge la compression gzip et Brotli, la compression Brotli est prioritaire.</br>
Quand une demande portant sur un élément multimédia spécifie une compression et qu’il en résulte une absence dans le cache, Front Door compresse l’élément multimédia directement sur le serveur POP. Ensuite, le fichier compressé est servi à partir du cache. L’élément qui en résulte est retourné avec un encodage de transfert : segmenté.

## <a name="query-string-behavior"></a>Comportement des chaînes de requête
Avec Front Door, vous pouvez contrôler la manière dont les fichiers sont mis en cache pour une requête web qui contient une chaîne de requête. Dans une requête web contenant une chaîne de requête, la chaîne de requête représente la partie de la demande qui apparaît après le point d’interrogation (?). Une chaîne de requête peut contenir une ou plusieurs paires clé-valeur où le nom du champ et sa valeur sont séparés par un signe égal (=). Chaque paire clé-valeur est séparée par une esperluette (&). Par exemple : `http://www.contoso.com/content.mov?field1=value1&field2=value2`. S’il existe plusieurs paires clé-valeur dans la chaîne de requête d’une demande, leur ordre n’a pas d’importance.
- **Ignorer les chaînes de requête** : Dans ce mode, Front Door transmet les chaînes de requête du demandeur au backend à la première demande et met en cache l’élément multimédia. Toutes les demandes subséquentes portant sur l’élément multimédia, qui sont traitées par l’environnement Front Door, ignorent les chaînes de requête tant que l’élément multimédia mis en cache n’est pas arrivé à expiration.

- **Mettre en cache chaque URL unique** : dans ce mode, chaque demande contenant une URL unique, y compris la chaîne de requête, est traitée comme une ressource unique avec son propre cache. Par exemple, la réponse du backend à une demande pour `www.example.ashx?q=test1` est mise en cache dans l’environnement Front Door et retournée pour les mises en cache subséquentes avec la même chaîne de requête. Une demande pour `www.example.ashx?q=test2` est mise en cache en tant que ressource distincte avec son propre paramètre de durée de vie.

## <a name="cache-purge"></a>Vidage du cache

Front Door met en cache les ressources tant que leur durée de vie (TTL) n’est pas arrivée à expiration. Chaque fois qu’un client demande un élément multimédia dont la durée de vie a expiré, l’environnement Front Door récupère une nouvelle copie mise à jour de l’élément multimédia pour servir la demande, puis stocke le cache actualisé.

La bonne pratique pour vous assurer que vos utilisateurs obtiennent toujours la dernière copie de vos éléments multimédia consiste à établir une version de vos ressources pour chaque mise à jour et à les publier en tant que nouvelles URL. Front Door récupère immédiatement les nouveaux éléments multimédias pour les demandes suivantes des clients. Il est parfois souhaitable de vider le contenu mis en cache sur tous les nœuds de périmètre et de tous les forcer à récupérer de nouveaux éléments multimédias mis à jour. Cela peut être dû à des mises à jour de votre application web, ou à la nécessité de mettre à jour rapidement les éléments multimédias contenant des informations incorrectes.

Sélectionnez les ressources que vous souhaitez supprimer définitivement des nœuds de périphérie. Pour effacer toutes les ressources, sélectionnez **Vider tout**. Dans le cas contraire, dans **Chemin d’accès**, entrez le chemin d’accès de chaque ressource que vous souhaitez supprimer définitivement.

Ces formats sont pris en charge dans les listes de chemins d’accès à vider :

- **Vidage à chemin unique** : videz une ou plusieurs ressources individuelles en spécifiant leur chemin complet (sans le protocole ni le domaine) avec l’extension de fichier, par exemple /pictures/strasbourg.png.
- **Vidage de caractère générique** : l’astérisque (\*) peut être utilisé comme caractère générique. Videz tous les dossiers, sous-dossiers et fichiers d’un point de terminaison en indiquant /\* dans le chemin ou videz tous les sous-dossiers et fichiers d’un certain dossier en spécifiant le dossier suivi de /\*, par exemple /pictures/\*.
- **Vidage du domaine racine** : videz la racine du point de terminaison avec « / » dans le chemin d’accès.

> [!NOTE]
> **Vidage des domaines génériques** : La spécification des chemins d’accès mis en cache pour le vidage, telle que décrite dans cette section, ne s’applique pas aux domaines génériques associés à Front Door. Actuellement, nous ne prenons pas en charge le vidage direct des domaines génériques. Vous pouvez vider les chemins d’accès de sous-domaines spécifiques en indiquant ce sous-domaine et le chemin de vidage. Par exemple, si Front Door a `*.contoso.com`, je peux supprimer définitivement les ressources de mon sous-domaine `foo.contoso.com` en saisissant `foo.contoso.com/path/*`. Actuellement, la spécification de noms d’hôtes dans le chemin de contenu de vidage est limitée aux sous-domaines des domaines génériques, le cas échéant.
>

Les vidages du cache dans la porte d’entrée ne respectent pas la casse. Par ailleurs, ils ne prennent pas en compte les chaînes de requête, ce qui signifie que le vidage d’une URL n’a pas pour effet de vider toutes ses variantes constituées de chaînes de requête. 

## <a name="cache-expiration"></a>Expiration du cache
L’ordre suivant des en-têtes sert à déterminer la durée de stockage d’un élément dans le cache :</br>
1. Cache-Control: s-maxage=\<seconds>
2. Cache-Control: max-age=\<seconds>
3. Expire : \<http-date>

Les en-têtes de réponse Cache-Control qui indiquent que la réponse n’est pas mise en cache, comme Cache-Control: private, Cache-Control: no cache et Cache-Control: no-store, sont respectés.  En l’absence de Cache-Control, par défaut, Front Door met en cache la ressource pendant une durée X, où X est choisi de manière aléatoire entre 1 et 3 jours.

## <a name="request-headers"></a>En-têtes de requête

Les en-têtes de demande suivants ne sont pas transférés à un serveur principal quand la mise en cache est utilisée.
- Content-Length
- Transfer-Encoding

## <a name="cache-duration"></a>Durée du cache

La durée du cache peut être configurée à la fois dans le concepteur Front Door et dans Rules Engine. La durée du cache définie dans le concepteur Front Door correspond à la durée minimale du cache. Ce remplacement ne fonctionnera pas si l’en-tête de contrôle du cache de l’origine a une durée de vie supérieure à la valeur de remplacement. 

La durée du cache définie via le moteur de règles est un véritable remplacement de cache, ce qui signifie qu’elle utilise la valeur de remplacement, quelle que soit l’en-tête de réponse d’origine.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer une porte d’entrée](quickstart-create-front-door.md).
- Découvrez [comment fonctionne Front Door](front-door-routing-architecture.md).
