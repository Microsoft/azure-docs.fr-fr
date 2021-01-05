---
title: Actions du moteur de règles Azure Front Door
description: Cet article fournit la liste des différentes actions que vous pouvez effectuer avec le moteur de règles Azure Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/29/2020
ms.author: duau
ms.openlocfilehash: ff61af192471bcfc9bdb9f1ce3970d5c22f39579
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91569785"
---
# <a name="azure-front-door-rules-engine-actions"></a>Actions du moteur de règles Azure Front Door

Dans le [moteur de règles AFD](front-door-rules-engine.md), une règle se compose de zéro, d’une ou de plusieurs conditions de correspondance et d’actions. Cet article fournit les descriptions détaillées des actions que vous pouvez utiliser dans le moteur de règles AFD.

Une action définit le comportement appliqué au type de requête qu’une condition de correspondance ou un ensemble de conditions de correspondance identifie. Dans le moteur de règles AFD, une règle peut contenir jusqu’à cinq actions. Une seule d’entre elles peut être une action de remplacement de configuration de route (transférer ou rediriger).

Les actions suivantes sont prêtes à être utilisées dans le moteur de règles Azure Front Door.  

## <a name="modify-request-header"></a>Modifier l’en-tête de requête

Utilisez cette action pour modifier les en-têtes présents dans les requêtes envoyées à votre origine.

### <a name="required-fields"></a>Champs obligatoires

Action | Nom de l’en-tête HTTP | Valeur
-------|------------------|------
Ajouter | Quand cette option est sélectionnée et que la règle est en correspondance, l’en-tête spécifié dans **Nom de l’en-tête** est ajouté à la requête avec la valeur spécifiée. Si l’en-tête est déjà présent, la valeur sera ajoutée à la valeur existante. | String
Remplacer | Quand cette option est sélectionnée et que la règle est en correspondance, l’en-tête spécifié dans **Nom de l’en-tête** est ajouté à la requête avec la valeur spécifiée. Si l’en-tête est déjà présent, la valeur spécifiée remplace la valeur existante. | String
DELETE | Quand cette option est sélectionnée avec des règles en correspondance et que l’en-tête spécifié dans la règle est présent, l’en-tête est supprimé de la requête. | String

## <a name="modify-response-header"></a>Modifier l’en-tête de réponse

Utilisez cette action pour modifier les en-têtes présents dans les réponses retournées à vos clients.

### <a name="required-fields"></a>Champs obligatoires

Action | Nom de l’en-tête HTTP | Valeur
-------|------------------|------
Ajouter | Quand cette option est sélectionnée et que la règle est en correspondance, l’en-tête spécifié dans **Nom de l’en-tête** est ajouté à la réponse en utilisant la **Valeur** spécifiée. Si l’en-tête est déjà présent, **Valeur** est ajoutée à la valeur existante. | String
Remplacer | Lorsque cette option est sélectionnée et que la règle correspond, l’en-tête spécifié dans **Nom de l’en-tête** est ajouté à la réponse à l’aide de la **Valeur** spécifiée. Si l’en-tête est déjà présent, **Valeur** remplace la valeur existante. | String
DELETE | Quand cette option est sélectionnée et que la règle correspond à l’en-tête spécifié dans la règle, l’en-tête est supprimé de la réponse. | String

## <a name="route-configuration-overrides"></a>Remplacements de configuration de route 

### <a name="route-type-redirect"></a>Type de route : Rediriger

Utilisez cette action pour rediriger les clients vers une nouvelle URL. 

#### <a name="required-fields"></a>Champs obligatoires

Champ | Description 
------|------------
Type de redirection | Sélectionnez le type de réponse à renvoyer au demandeur : Trouvé (302), Déplacé (301), Redirection temporaire (307) et Redirection permanente (308).
Protocole de redirection | Requête correspondante, HTTP ou HTTPS.
Hôte de destination | Sélectionnez le nom d’hôte vers lequel vous souhaitez rediriger la requête. Laissez vide pour conserver l’hôte entrant.
Chemin de destination | Définissez le chemin d’accès à utiliser dans la redirection. Laissez vide pour conserver le chemin d’accès entrant.  
Chaîne de requête | Définissez la chaîne de requête utilisée dans la redirection. Laissez vide pour conserver la chaîne de requête entrante. 
Fragment de destination | Définissez le fragment à utiliser dans la redirection. Laissez vide pour conserver le fragment entrant. 


### <a name="route-type-forward"></a>Type de route : Transférer

Utilisez cette action pour transférer les clients vers une nouvelle URL. Cette action contient aussi des sous-actions pour les réécritures et mises en cache d’URL. 

Champ | Description 
------|------------
Pool principal | Sélectionnez le pool de back-ends à remplacer et traitez les requêtes : ceci va aussi montrer tous vos pools de back-ends préconfigurés qui sont actuellement dans votre profil Front Door. 
Protocole de transfert | Requête correspondante, HTTP ou HTTPS.
Réécrire URL | Utilisez cette action pour réécrire le chemin d’accès d’une requête en cours d’acheminement vers votre origine. Si cette option est activée, voir ci-dessous les autres champs obligatoires
Mise en cache | Activé, Désactivé. Voir ci-dessous pour les autres champs obligatoires si cette option est activée. 

#### <a name="url-rewrite"></a>Réécrire URL

Utilisez ce paramètre pour configurer un **chemin de transfert personnalisé** facultatif à utiliser lors de la construction d’une requête à transférer au back-end.

Champ | Description 
------|------------
Chemin de transfert personnalisé | Définissez le chemin vers lequel transférer les requêtes. 

#### <a name="caching"></a>Mise en cache

Utilisez ces paramètres pour contrôler la façon dont les fichiers sont mis en cache pour les demandes qui contiennent des chaînes de requête. Indique si votre contenu doit être mis en cache en fonction de tous les paramètres ou des paramètres sélectionnés. Vous pouvez utiliser des paramètres supplémentaires pour remplacer la valeur de la durée de vie (TTL) pour contrôler la durée pendant laquelle le contenu reste dans le cache. Pour forcer la mise en cache comme action, définissez le champ de mise en cache sur « Activé ». Quand vous forcez la mise en cache, les options suivantes apparaissent : 

Comportement du cache |  Description              
---------------|----------------
Ignorer les chaînes de requête | Une fois que la ressource est mise en cache, toutes les demandes qui suivent ignorent les chaînes de requête jusqu’à l’expiration de la ressource mise en cache.
Mettre en cache chaque URL unique | Chaque demande contenant une URL unique, y compris la chaîne de requête, est traitée comme une ressource unique avec son propre cache.
Ignorer les chaînes de requête spécifiées | Les chaînes de requête d’URL de demande listées dans le paramètre « Paramètres de requête » sont ignorées pour la mise en cache.
Inclure les chaînes de requête spécifiées | Les chaînes de requête d’URL de demande listées dans le paramètre « Paramètres de requête » sont utilisées pour la mise en cache.

Champs supplémentaires |  Description 
------------------|---------------
Compression dynamique | Front Door peut compresser dynamiquement le contenu en périphérie, ce qui a pour effet de réduire la taille et le délai de la réponse.
Paramètres de requête | Une liste séparée par des virgules des paramètres autorisés (ou non autorisés) à utiliser comme base pour la mise en cache.
Durée du cache | Durée d’expiration du cache en jours, heures, minutes, secondes. Toutes les valeurs doivent être des entiers. 

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment configurer votre premier [Moteur de règles](front-door-tutorial-rules-engine.md). 
- En savoir plus sur les [Conditions de correspondance du moteur de règles](front-door-rules-engine-match-conditions.md)
- En savoir plus sur le [Moteur de règles Azure Front Door](front-door-rules-engine.md)
