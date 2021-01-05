---
title: Actions du moteur de règles Standard pour Azure CDN | Microsoft Docs
description: Documentation de référence sur les actions du moteur de règles Standard pour Azure Content Delivery Network (Azure CDN).
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: article
ms.date: 08/04/2020
ms.author: allensu
ms.openlocfilehash: 051737a9f5e0d4092cda26a3f7ce3df1d7f535ef
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87760122"
---
# <a name="actions-in-the-standard-rules-engine-for-azure-cdn"></a>Actions du moteur de règles Standard pour Azure CDN

Dans le [moteur de règles Standard](cdn-standard-rules-engine.md) pour Azure Content Delivery Network (Azure CDN), une règle se compose d’une ou de plusieurs conditions de correspondance et d’une action. Cet article fournit les descriptions détaillées des actions que vous pouvez utiliser dans le moteur de règles Standard pour Azure CDN.

La deuxième partie d’une règle est une action. Une action définit le comportement appliqué au type de requête qu’une condition de correspondance ou un ensemble de conditions de correspondance identifie.

## <a name="actions"></a>Actions

Les actions suivantes peuvent être utilisées dans le moteur de règles Standard pour Azure CDN. 

### <a name="cache-expiration"></a>Expiration du cache

Utilisez cette action pour remplacer la valeur de durée de vie (TTL) du point de terminaison pour les requêtes spécifiées par les conditions de correspondance des règles.

#### <a name="required-fields"></a>Champs obligatoires

Comportement du cache |  Description              
---------------|----------------
Ignorer le cache | Lorsque cette option est sélectionnée et que la règle correspond, le contenu n’est pas mis en cache.
Écraser | Lorsque cette option est sélectionnée et que la règle correspond, la valeur TTL retournée par votre origine est remplacée par la valeur spécifiée dans l’action. Ce comportement est appliqué uniquement si la réponse est mise en cache. Pour l’en-tête de réponse de contrôle du cache avec les valeurs « no-cache », « private », « no-store », l’action n’est pas applicable.
Définir en cas d’absence | Lorsque cette option est sélectionnée et que la règle correspond, s’il n’y a aucune valeur TTL retournée par votre origine, la règle définit la TTL sur la valeur spécifiée dans l’action. Ce comportement est appliqué uniquement si la réponse est mise en cache. Pour l’en-tête de réponse de contrôle du cache avec les valeurs « no-cache », « private », « no-store », l’action n’est pas applicable.

#### <a name="additional-fields"></a>Champs supplémentaires

Jours | Heures | Minutes | Secondes
-----|-------|---------|--------
Int | Int | Int | Int 

### <a name="cache-key-query-string"></a>Chaîne de requête de clé de cache

Utilisez cette action pour modifier la clé de cache en fonction des chaînes de requête.

#### <a name="required-fields"></a>Champs obligatoires

Comportement | Description
---------|------------
Inclure | Lorsque cette option est sélectionnée et que la règle correspond, les chaînes de requête spécifiées dans les paramètres sont incluses lorsque la clé de cache est générée. 
Mettre en cache chaque URL unique | Lorsque cette option est sélectionnée et que la règle correspond, chaque URL unique a sa propre clé de cache. 
Exclure | Lorsque cette option est sélectionnée et que la règle correspond, les chaînes de requête spécifiées dans les paramètres sont exclues lorsque la clé de cache est générée.
Ignorer les chaînes de requête | Lorsque cette option est sélectionnée et que la règle correspond, les chaînes de requête ne sont pas prises en compte lorsque la clé de cache est générée. 

### <a name="modify-request-header"></a>Modifier l’en-tête de requête

Utilisez cette action pour modifier les en-têtes présents dans les requêtes envoyées à votre origine.

#### <a name="required-fields"></a>Champs obligatoires

Action | Nom de l’en-tête HTTP | Valeur
-------|------------------|------
Ajouter | Lorsque cette option est sélectionnée et que la règle correspond, l’en-tête spécifié dans **Nom de l’en-tête** est ajouté à la requête avec la valeur spécifiée. Si l’en-tête est déjà présent, la valeur sera ajoutée à la valeur existante. | String
Remplacer | Lorsque cette option est sélectionnée et que la règle correspond, l’en-tête spécifié dans **Nom de l’en-tête** est ajouté à la requête avec la valeur spécifiée. Si l’en-tête est déjà présent, la valeur spécifiée remplace la valeur existante. | String
DELETE | Lorsque cette option est sélectionnée, que la règle correspond et que l’en-tête spécifié dans la règle est présent, l’en-tête est supprimé de la requête. | String

### <a name="modify-response-header"></a>Modifier l’en-tête de réponse

Utilisez cette action pour modifier les en-têtes présents dans les réponses retournées à vos clients.

#### <a name="required-fields"></a>Champs obligatoires

Action | Nom de l’en-tête HTTP | Valeur
-------|------------------|------
Ajouter | Lorsque cette option est sélectionnée et que la règle correspond, l’en-tête spécifié dans **Nom de l’en-tête** est ajouté à la réponse à l’aide de la **Valeur** spécifiée. Si l’en-tête est déjà présent, **Valeur** est ajoutée à la valeur existante. | String
Remplacer | Lorsque cette option est sélectionnée et que la règle correspond, l’en-tête spécifié dans **Nom de l’en-tête** est ajouté à la réponse à l’aide de la **Valeur** spécifiée. Si l’en-tête est déjà présent, **Valeur** remplace la valeur existante. | String
DELETE | Lorsque cette option est sélectionnée, que la règle correspond et que l’en-tête spécifié dans la règle est présent, l’en-tête est supprimé de la réponse. | String

### <a name="url-redirect"></a>Redirection d’URL

Utilisez cette action pour rediriger les clients vers une nouvelle URL. 

#### <a name="required-fields"></a>Champs obligatoires

Champ | Description 
------|------------
Type | Sélectionnez le type de réponse à renvoyer au demandeur : Trouvé (302), Déplacé (301), Redirection temporaire (307) et Redirection permanente (308).
Protocol | Requête correspondante, HTTP ou HTTPS.
HostName | Sélectionnez le nom d’hôte vers lequel vous souhaitez rediriger la requête. Laissez vide pour conserver l’hôte entrant.
Path | Définissez le chemin d’accès à utiliser dans la redirection. Laissez vide pour conserver le chemin d’accès entrant.  
Chaîne de requête | Définissez la chaîne de requête utilisée dans la redirection. Laissez vide pour conserver la chaîne de requête entrante. 
Fragment | Définissez le fragment à utiliser dans la redirection. Laissez vide pour conserver le fragment entrant. 

Nous vous recommandons vivement d’utiliser une URL absolue. L’utilisation d’une URL relative peut rediriger les URL Azure CDN vers un chemin d’accès non valide. 

### <a name="url-rewrite"></a>Réécrire URL

Utilisez cette action pour réécrire le chemin d’accès d’une requête en cours d’acheminement vers votre origine.

#### <a name="required-fields"></a>Champs obligatoires

Champ | Description 
------|------------
Modèle source | Définissez le modèle source dans le chemin d’accès de l’URL à remplacer. Actuellement, le modèle source utilise une correspondance basée sur un préfixe. Pour correspondre à tous les chemins d’accès d’URL, utilisez une barre oblique ( **/** ) comme valeur de modèle source.
Destination | Définissez le chemin d’accès de destination à utiliser lors de la réécriture. Le chemin d’accès de destination remplace le modèle source.
Conserver le chemin d’accès sans correspondance | S’il est défini sur **Oui**, le chemin d’accès restant après le modèle source est ajouté au nouveau chemin d’accès de destination. 

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble d’Azure CDN](cdn-overview.md)
- [Informations de référence sur le moteur de règles Standard](cdn-standard-rules-engine-reference.md)
- [Conditions de correspondance du moteur de règles Standard](cdn-standard-rules-engine-match-conditions.md)
- [Appliquer HTTPS en utilisant le moteur de règles Standard](cdn-standard-rules-engine.md)
