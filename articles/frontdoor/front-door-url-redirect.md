---
title: Azure Front Door – Redirection d’URL | Microsoft Docs
description: Cet article vous aide à comprendre comment Azure Front Door prend en charge la redirection d'URL pour ses règles d'acheminement.
services: front-door
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: duau
ms.openlocfilehash: 61077c7900530fd4c5be64054bedd9c5d087fe77
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91442058"
---
# <a name="url-redirect"></a>Redirection d’URL
Azure Front Door peut rediriger le trafic à chacun des niveaux suivants : protocole, nom d'hôte, chemin d'accès, chaîne de requête. Ces fonctionnalités peuvent être configurées pour des microservices individuels dans la mesure où la redirection est basée sur le chemin d'accès. Cela peut permettre de simplifier la configuration de l'application en optimisant l'utilisation des ressources, et de prendre en charge de nouveaux scénarios de redirection, notamment la redirection globale et basée sur le chemin d'accès.
</br>

:::image type="content" source="./media/front-door-url-redirect/front-door-url-redirect.png" alt-text="Redirection d’URL Azure Front Door":::

## <a name="redirection-types"></a>Types de redirection
Un type de redirection définit le code d’état de réponse pour aider les clients à comprendre l’objectif de la redirection. Les types suivants de redirection sont pris en charge :

- **301 Déplacé de façon permanente** : Indique que la ressource cible a été attribuée à un nouvel URI permanent. Toute référence future à cette ressource utilisera l'un des URI inclus. Utilisez le code d’état 301 pour la redirection du protocole HTTP vers HTTPS. 
- **302 Trouvé** : Indique que la ressource cible se trouve temporairement sous un autre URI. Étant donné que la redirection peut changer à l'occasion, le client doit continuer à utiliser l'URI de requête effectif pour les demandes ultérieures.
- **307 Redirection temporaire** : Indique que la ressource cible se trouve temporairement sous un autre URI. L'agent utilisateur NE DOIT PAS modifier la méthode de demande s'il procède à une redirection automatique vers cet URI. Étant donné que la redirection peut changer au fil du temps, le client doit continuer à utiliser l’URI de requête en effet d’origine pour les requêtes ultérieures.
- **308 Redirection permanente** : Indique que la ressource cible a été attribuée à un nouvel URI permanent. Toute référence future à cette ressource devra utiliser l'un des URI inclus.

## <a name="redirection-protocol"></a>Protocole de redirection
Vous pouvez définir le protocole qui sera utilisé pour la redirection. Les cas d'usage les plus courants de la fonctionnalité de redirection sont la redirection du trafic HTTP vers HTTPS.

- **HTTPS uniquement** : Définissez le protocole sur HTTPS uniquement si vous cherchez à rediriger le trafic HTTP vers HTTPS. Azure Front Door recommande de toujours définir la redirection vers HTTPS uniquement.
- **HTTP uniquement** : Redirige la requête entrante vers HTTP. Utilisez cette valeur uniquement si vous souhaitez conserver votre trafic HTTP tel qu’il est, à savoir non chiffré.
- **Faire correspondre la requête** : Cette option conserve le protocole utilisé par la requête entrante. Par conséquent, une requête HTTP reste HTTP et une requête HTTPS reste une redirection HTTPS.

## <a name="destination-host"></a>Hôte de destination
Dans le cadre de la configuration d’un routage de redirection, vous pouvez également modifier le nom d’hôte ou le domaine pour la requête de redirection. Vous pouvez définir ce champ pour modifier le nom d’hôte dans l’URL pour la redirection, ou bien conserver le nom d’hôte de la demande entrante. Ainsi, avec ce champ, vous pouvez rediriger toutes les requêtes envoyées sur `https://www.contoso.com/*` vers `https://www.fabrikam.com/*`.

## <a name="destination-path"></a>Chemin de destination
Pour les cas où vous souhaiteriez remplacer le segment de chemin d’accès d’une URL dans le cadre de la redirection, vous pouvez définir ce champ avec la nouvelle valeur de chemin d’accès. Sinon, vous pouvez choisir de conserver la valeur de chemin d’accès dans le cadre de la redirection. Ainsi, avec ce champ, vous pouvez rediriger toutes les requêtes envoyées à `https://www.contoso.com/\*` vers `https://www.contoso.com/redirected-site`.

## <a name="query-string-parameters"></a>Paramètres de chaîne de requête
Vous pouvez également remplacer les paramètres de chaîne de requête dans l’URL redirigée. Pour remplacer une chaîne de requête existante de l'URL de requête entrante, définissez ce champ sur « Remplacer », puis définissez la valeur appropriée. Sinon, vous pouvez conserver le jeu initial de chaînes de requête en définissant le champ sur « Conserver ». Par exemple, à l’aide de ce champ, vous pouvez rediriger tout le trafic envoyé à `https://www.contoso.com/foo/bar` vers `https://www.contoso.com/foo/bar?&utm_referrer=https%3A%2F%2Fwww.bing.com%2F`. 

## <a name="destination-fragment"></a>Fragment de destination
Le fragment de destination est la partie de l'URL figurant après « # », que les navigateurs utilisent pour accéder à une section spécifique d'une page web. Vous pouvez définir ce champ pour ajouter un fragment à l’URL de redirection.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer une porte d’entrée](quickstart-create-front-door.md).
- Découvrez [comment fonctionne Front Door](front-door-routing-architecture.md).
