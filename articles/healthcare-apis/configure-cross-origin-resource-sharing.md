---
title: Configurer le partage de ressources CORS (cross-origin) dans l’API Azure pour FHIR
description: Cet article explique comment configurer le partage de ressources CORS (cross-origin) dans l’API Azure pour FHIR.
author: matjazl
ms.author: matjazl
ms.date: 3/11/2019
ms.topic: reference
ms.service: healthcare-apis
ms.subservice: fhir
ms.openlocfilehash: dc4c034b0821f1fe5ecb940591fca77d61edc3ce
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84870939"
---
# <a name="configure-cross-origin-resource-sharing-in-azure-api-for-fhir"></a>Configurer le partage de ressources CORS (cross-origin) dans l’API Azure pour FHIR

L’API Azure pour FHIR prend en charge le [partage des ressources cross-origin (CORS)](https://wikipedia.org/wiki/Cross-Origin_Resource_Sharing). Ce dernier permet de configurer des paramètres afin que les applications d’un domaine (origine) puissent accéder aux ressources d’un domaine différent par le biais d’une requête inter-domaines.

Le partage des ressources cross-origin (CORS) est souvent utilisé dans une application à page unique qui doit appeler une API RESTful sur un autre domaine.

Pour configurer un paramètre de partage des ressources cross-origin (CORS) dans l’API Azure pour FHIR, spécifiez les paramètres suivants :

- **Origines (Access-Control-Allow-Origin)** . Liste de domaines autorisés à effectuer des requêtes cross-origin sur l’API Azure pour FHIR. Chaque domaine (origine) doit être entré dans une ligne distincte. Vous pouvez entrer un astérisque (*) pour autoriser les appels à partir de n’importe quel domaine, toutefois, cela n’est pas recommandé dans la mesure ou il existe un risque de sécurité.

- **En-têtes (Access-Control-Allow-Headers)** . Liste d’en-têtes contenus dans la requête d’origine. Pour autoriser tous les en-têtes, saisissez un astérisque (*).

- **Méthodes (Access-Control-Allow-Methods)** . Méthodes autorisées (PUT, GET, POST etc.) dans un appel d’API. Choisissez **Sélectionner tout** pour toutes les méthodes.

- **Âge maximal (Access-Control-Max-Age)** . Valeur en secondes pour mettre en cache les résultats de requête préliminaire pour Access-Control-Allow-Headers et Access-Control-Allow-Methods.

- **Autoriser les informations d’identification (Access-Control-Allow-Credentials)** . Les requêtes CORS n’incluent normalement pas de cookies permettant d’empêcher les attaques par [falsification de requête intersites](https://en.wikipedia.org/wiki/Cross-site_request_forgery). Si vous sélectionnez ce paramètre, la requête peut être effectuée de manière à inclure des informations d’identification, telles que des cookies. Vous ne pouvez pas configurer ce paramètre si vous avez déjà défini Origines sur un astérisque (*).

![Paramètres du partage des ressources cross-origin (CORS)](media/cors/cors.png)

>[!NOTE]
>Vous ne pouvez pas spécifier de paramètres différents pour différentes origines de domaine. Tous les paramètres (**En-têtes**, **Méthodes**, **Âge maximal** et **Autoriser les informations d’identification**) s’appliquent à toutes les origines spécifiées dans le paramètre Origines.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à configurer le partage de ressources CORS (cross-origin) dans l’API Azure pour FHIR. Ensuite, déployez une API Azure pour FHIR complètement managée :
 
>[!div class="nextstepaction"]
>[Déployer l’API Azure pour FHIR](fhir-paas-portal-quickstart.md)