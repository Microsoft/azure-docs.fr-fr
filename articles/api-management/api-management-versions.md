---
title: Versions dans Gestion des API Azure | Microsoft Docs
description: Découvrez le concept des versions dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: johndowns
ms.service: api-management
ms.topic: article
ms.date: 06/12/2020
ms.author: jodowns
ms.custom: fasttrack-new
ms.openlocfilehash: 578bb511175d88a1507af9520265a1acd068b27c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094152"
---
# <a name="versions-in-azure-api-management"></a>Versions dans Gestion des API Azure

Les versions vous permettent de présenter des groupes d’API connexes à vos développeurs. Vous pouvez utiliser des versions pour traiter en toute sécurité les changements cassants dans votre API. Les clients peuvent choisir d’utiliser la nouvelle version de votre API lorsqu’ils sont prêts, tandis que les clients existants continuent d’utiliser une version antérieure. Les versions sont différenciées par un identificateur de version (qui correspond à toute valeur de chaîne de votre choix), et un schéma de contrôle de version permet aux clients d’identifier la version d’une API qu’ils souhaitent utiliser.

Dans la plupart des cas, chaque version d’API peut être considérée comme sa propre API indépendante. Deux versions d’API distinctes peuvent avoir différents ensembles d’opérations et des stratégies spécifiques.

Grâce aux versions, vous pouvez :

- Publier plusieurs versions de votre API au même moment.
- Utiliser un chemin d’accès, une chaîne de requête ou un en-tête pour faire la différence entre les versions.
- Utiliser la valeur de chaîne de votre choix pour identifier votre version, qui peut être un nombre, une date ou un nom.
- Afficher vos versions d’API regroupées sur le portail des développeurs.
- Prendre une API existante (sans version) et en créer une nouvelle version de celle-ci sans arrêter les clients existants.

[Prenez en main les versions en suivant notre procédure pas à pas.](./api-management-get-started-publish-versions.md)

## <a name="versioning-schemes"></a>Schémas de contrôle de version

Différents développeurs d’API ont des exigences différentes en matière de contrôle de version. Gestion des API Azure n’impose pas une approche unique pour le contrôle de version, mais offre à la place plusieurs options.

### <a name="path-based-versioning"></a>Contrôle de version basé sur le chemin d’accès

Lorsque le schéma de contrôle de version par chemin d’accès est utilisé, l’identificateur de version doit être inclus dans le chemin d’URL de toutes les demandes d’API.

Par exemple, `https://apis.contoso.com/products/v1` et `https://apis.contoso.com/products/v2` peuvent faire référence à la même API `products`, mais aux versions `v1` et `v2` respectivement.

Le format d’une URL de demande d’API lorsque le contrôle de version est basé sur l’en-tête est le suivant : `https://{yourDomain}/{apiName}/{versionIdentifier}/{operationId}`.

### <a name="header-based-versioning"></a>Contrôle de version basé sur l’en-tête

Lorsque le schéma de contrôle de version par en-tête est utilisé, l’identificateur de version doit être inclus dans un en-tête de requête HTTP pour toutes les demandes d’API. Vous pouvez spécifier le nom de l’en-tête de requête HTTP.

Par exemple, vous pouvez créer un en-tête personnalisé nommé `Api-Version`, et les clients peuvent spécifier `v1` ou `v2` dans la valeur de cet en-tête.

### <a name="query-string-based-versioning"></a>Contrôle de version basé sur une chaîne de requête

Lorsque le schéma de contrôle de version par chaîne de requête est utilisé, l’identificateur de version doit être inclus dans un paramètre de chaîne de requête pour toutes les demandes d’API. Vous pouvez spécifier le nom du paramètre de chaîne de requête.

Le format d’une URL de demande d’API lorsque le contrôle de version est basé sur une chaîne de requête est le suivant : `https://{yourDomain}/{apiName}/{operationId}?{queryStringParameterName}={versionIdentifier}`.

Par exemple, `https://apis.contoso.com/products?api-version=v1` et `https://apis.contoso.com/products/api-version=v2` peuvent faire référence à la même API `products`, mais aux versions `v1` et `v2` respectivement.

## <a name="original-versions"></a>Versions d’origine

Si vous ajoutez une version à une API sans version, une version `Original` est automatiquement créée et répond sur l’URL par défaut, sans qu’un identificateur de version soit spécifié. La version `Original` permet aux appelants existants de ne pas être interrompus par le processus d’ajout d’une version. Si vous créez une nouvelle API avec des versions activées au démarrage, une version `Original` n’est pas créée.

## <a name="how-versions-are-represented"></a>Représentation des versions

Gestion des API Azure gère une ressource appelée *jeu de versions*, qui représente un ensemble de versions pour une API logique unique. Lorsque vous utilisez le portail Azure pour gérer les versions, vous ne voyez pas le jeu de versions, mais, si vous interagissez avec votre service Gestion des API en utilisant PowerShell, des modèles Resource Manager ou l’API Azure Resource Manager, vous pouvez consulter et gérer les jeux de versions directement. Un jeu de versions contient le nom complet de l’API avec version, ainsi que le [schéma de contrôle de version utilisé](#versioning-schemes) pour diriger les demandes vers les versions spécifiées.

Chaque version d’une API est gérée comme sa propre ressource d’API, qui est ensuite associée à un jeu de versions. Un jeu de versions peut contenir des API avec des opérations ou des stratégies très différentes, ce qui reflète le fait que vous pouvez apporter des modifications importantes entre les versions de votre API.

### <a name="migrating-a-non-versioned-api-to-a-versioned-api"></a>Migration d’une API sans version vers une API avec version

Lorsque vous utilisez le portail Azure pour activer le contrôle de version sur une API existante, les modifications suivantes sont apportées à vos ressources Gestion des API :

 * Un jeu de versions est créé.
 * La version existante est conservée et [configurée comme la version d’API `Original`](#original-versions). L’API est associée au jeu de versions, mais ne nécessite pas la spécification d’un identificateur de version.
 * La nouvelle version est créée en tant que nouvelle API et associée au jeu de versions. Cette nouvelle API doit être accessible à l’aide du schéma de contrôle de version et de l’identificateur.

## <a name="versions-and-revisions"></a>Versions et révisions

Les versions et les révisions constituent des fonctionnalités distinctes. Chaque version peut présenter plusieurs révisions, tout comme il peut exister une API sans version. Vous pouvez utiliser des révisions sans utiliser de versions ou inversement. En général, les versions sont utilisées pour distinguer les versions d’API avec changements cassants, tandis que les révisions peuvent être utilisées pour les changements mineurs et non cassants apportés à une API.

Si vous pensez que votre révision présente des changements cassants ou si vous souhaitez formellement faire évoluer votre révision en version bêta/test, vous pouvez créer une version à partir d’une révision. À l’aide du portail Azure, cliquez sur « Créer une version à partir de la révision » dans le menu contextuel de révision sous l’onglet Révisions.

## <a name="developer-portal"></a>Portail des développeurs

Le [portail des développeurs](./api-management-howto-developer-portal.md) répertorie chaque version d’une API séparément.

![Portail des développeurs de Gestion des API affichant une liste des API avec version](media/api-management-versions/portal-list.png)

Les détails d’une API affichent également une liste de toutes les versions de cette API. Une version `Original` s’affiche sans identificateur de version.

![Portail des développeurs de Gestion des API affichant les détails d’une API et une liste des versions de cette API](media/api-management-versions/portal-details.png)

> [!TIP]
> Vous devez ajouter les versions de l’API à un produit pour qu’elles soient visibles sur le portail des développeurs.
