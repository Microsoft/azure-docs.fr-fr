---
title: Itinéraires dans Azure Static Web Apps
description: Découvrez les règles d’acheminement principales et comment sécuriser les itinéraires avec des rôles.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: 8abbe575e855347714c19c40155d890af484d5d6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91822328"
---
# <a name="routes-in-azure-static-web-apps-preview"></a>Itinéraires dans Azure Static Web Apps - Préversion

Le routage dans Azure Static Web Apps définit les règles d’acheminement principales et le comportement d’autorisation pour le contenu statique et les API<sup>1</sup>. Les règles sont définies sous la forme d’un tableau de règles dans le fichier _routes.json_.

- Le fichier _routes.json_ doit exister à la racine du dossier de l’artefact de build de l’application.
- Les règles sont exécutées dans l’ordre dans lequel elles apparaissent dans le tableau `routes`.
- L’évaluation de la règle s’arrête à la première correspondance. Les règles d’acheminement ne sont pas enchaînées.
- Les rôles sont définis dans le fichier _routes.json_ et les utilisateurs sont associés aux rôles via des [invitations](authentication-authorization.md).
- Vous avez un contrôle total sur les noms de rôle.

La rubrique relative au routage a de nombreux points communs avec les concepts d’authentification et d’autorisation. Lisez le Guide d’[authentification et d’autorisation](authentication-authorization.md), ainsi que cet article.

Pour obtenir des détails, consultez [l’exemple de fichier de routage](#example-route-file).

## <a name="location"></a>Emplacement

Le fichier _routes.json_ doit exister à la racine du dossier de l’artefact de build de l’application. Si votre application web comprend une étape de compilation qui copie les fichiers compilés d’un dossier donné vers votre dossier d’artefacts de build, le fichier _routes.json_ doit exister dans ce dossier.

Le tableau suivant liste l’emplacement approprié pour placer votre fichier _routes.json_ pour un certain nombre de bibliothèques et de frameworks front-end.

|Framework/bibliothèque | Emplacement  |
|---------|----------|
| Angular | _assets_   |
| React   | _public_  |
| Svelte  | _public_   |
| Vue     | _public_ |
| Blazor  | _wwwroot_ |

Le tableau ci-dessus est uniquement représentatif de quelques bibliothèques et frameworks compatibles avec Azure Static Web Apps. Pour plus d’informations, consultez [Configurer des bibliothèques et des frameworks front-end](./front-end-frameworks.md).

## <a name="defining-routes"></a>Définition des itinéraires

Les itinéraires sont définis dans le fichier _routes.json_ sous la forme d’un tableau de règles d’acheminement sur la propriété `routes`. Chaque règle est composée d’un modèle d’itinéraire, ainsi que d’une ou plusieurs des propriétés facultatives de la règle. Pour obtenir des exemples d’utilisation, consultez l’[exemple de fichier de routage](#example-route-file).

| Propriété de la règle  | Obligatoire | Valeur par défaut | Commentaire                                                      |
| -------------- | -------- | ------------- | ------------------------------------------------------------ |
| `route`        | Oui      | n/a          | Modèle d’itinéraire demandé par l’appelant.<ul><li>[Les caractères génériques](#wildcards) sont pris en charge à la fin des chemins d’accès d’itinéraire. Par exemple, l’itinéraire _admin/\*_ correspond à n’importe quel itinéraire sous le chemin d’accès _admin_.<li>Le fichier d’itinéraire par défaut est _index.html_.</ul>|
| `serve`        | Non       | n/a          | Définit le fichier ou le chemin d’accès retourné par la demande. Le chemin d’accès et le nom du fichier peuvent être différents du chemin d’accès demandé. Si une valeur `serve` n’est pas définie, le chemin d’accès demandé est utilisé. Les paramètres de la chaîne de requête ne sont pas pris en charge ; les valeurs `serve` doivent pointer vers des fichiers réels.  |
| `allowedRoles` | Non       | anonyme     | Tableau de noms de rôles. <ul><li>Les caractères valides sont `a-z`, `A-Z`, `0-9` et `_`.<li>Le rôle intégré `anonymous` s’applique à tous les utilisateurs non authentifiés.<li>Le rôle intégré `authenticated` s’applique à tous les utilisateurs connectés.<li>Les utilisateurs doivent appartenir à au moins un rôle.<li>Les rôles sont mis en correspondance sur une base _OU_. Si un utilisateur se trouve dans l’un des rôles de la liste, l’accès est accordé.<li>Les utilisateurs individuels sont associés à des rôles par le biais des [invitations](authentication-authorization.md).</ul> |
| `statusCode`   | Non       | 200           | La réponse de [code d’état HTTP](https://wikipedia.org/wiki/List_of_HTTP_status_codes) pour la demande. |

## <a name="securing-routes-with-roles"></a>Sécurisation des itinéraires avec des rôles

Les itinéraires sont sécurisés en y ajoutant un ou plusieurs noms de rôles dans le tableau `allowedRoles` de la règle. Pour obtenir des exemples d’utilisation, consultez l’[exemple de fichier de routage](#example-route-file).

Par défaut, chaque utilisateur appartient au rôle `anonymous` intégré et tous les utilisateurs connectés sont membres du rôle `authenticated`. Par exemple, pour limiter un itinéraire aux seuls utilisateurs authentifiés, ajoutez le rôle `authenticated` intégré au groupe `allowedRoles`.

```json
{
  "route": "/profile",
  "allowedRoles": ["authenticated"]
}
```

Vous pouvez créer des rôles en fonction des besoins dans le tableau `allowedRoles`. Pour limiter un itinéraire uniquement aux administrateurs, vous pouvez définir un rôle `administrator` dans le tableau `allowedRoles`.

```json
{
  "route": "/admin",
  "allowedRoles": ["administrator"]
}
```

- Vous disposez d’un contrôle total sur les noms de rôles ; il n’existe aucune liste à laquelle vos rôles doivent adhérer.
- Les utilisateurs individuels sont associés à des rôles par le biais des [invitations](authentication-authorization.md).

## <a name="wildcards"></a>Caractères génériques

Les règles génériques correspondent à toutes les requêtes dans un modèle d’itinéraire donné. Si vous définissez une valeur de `serve` dans votre règle, le fichier ou le chemin d’accès nommé est pris en charge comme réponse.

Par exemple, pour implémenter des itinéraires pour une application de calendrier, vous pouvez mapper toutes les URL qui se trouvent sous l’itinéraire _calendrier_ pour un seul fichier.

```json
{
  "route": "/calendar/*",
  "serve": "/calendar.html"
}
```

Le fichier _calendar.html_ peut ensuite utiliser le routage côté client pour offrir une vue différente des variantes d’URL, comme `/calendar/january/1`, `/calendar/2020` et `/calendar/overview`.

Vous pouvez également sécuriser les itinéraires avec des caractères génériques. Dans l’exemple suivant, tout fichier demandé dans le chemin d’accès _admin_ nécessite un utilisateur authentifié membre du rôle _administrateur_.

```json
{
  "route": "/admin/*",
  "allowedRoles": ["administrator"]
}
```

> [!NOTE]
> Les demandes de fichiers référencés par un fichier servi sont évaluées uniquement pour les vérifications d’authentification. Par exemple, les demandes à un fichier CSS sous un chemin d’accès générique servent le fichier CSS et non ce qui est défini comme fichier `serve`. Le fichier CSS est pris en charge à condition que l’utilisateur réponde aux exigences d’authentification requises.

## <a name="fallback-routes"></a>Itinéraires de secours

Les applications monopages, qu’elles utilisent des bibliothèques ou des frameworks JavaScript front-end, ou des plateformes WebAssembly comme Blazor, s’appuient souvent sur le routage côté client pour la navigation dans les applications web. Ces règles d’acheminement côté client mettent à jour l’emplacement de la fenêtre du navigateur sans effectuer de demande au serveur. Si vous actualisez la page ou si vous accédez directement aux emplacements générés par les règles d’acheminement côté client, un itinéraire de secours côté serveur est requis pour servir la page HTML appropriée.

Voici un exemple classique d’itinéraire de secours :

```json
{
  "routes": [
    {
      "route": "/*",
      "serve": "/index.html",
      "statusCode": 200
    }
  ]
}
```

L’itinéraire de secours doit être listé en dernier dans vos règles d’acheminement, car il intercepte toutes les requêtes non interceptées par les règles définies précédemment.

## <a name="redirects"></a>Redirection

Vous pouvez utiliser les codes d’état HTTP [301](https://en.wikipedia.org/wiki/HTTP_301) et [302](https://en.wikipedia.org/wiki/HTTP_302) pour rediriger les demandes d’un itinéraire vers un autre.

Par exemple, la règle suivante crée une redirection 301 de _old-page.html_ vers _new-page.html_.

```json
{
  "route": "/old-page.html",
  "serve": "/new-page.html",
  "statusCode": 301
}
```

Les redirections fonctionnent également avec les chemins d’accès qui ne définissent pas de fichiers distincts.

```json
{
  "route": "/about-us",
  "serve": "/about",
  "statusCode": 301
}
```

## <a name="custom-error-pages"></a>Pages d’erreur personnalisées

Les utilisateurs peuvent rencontrer plusieurs situations qui peuvent provoquer une erreur. À l’aide du tableau `platformErrorOverrides`, vous pouvez fournir une expérience personnalisée en réponse à ces erreurs. Reportez-vous à l’[exemple de fichier de route](#example-route-file) pour le placement du tableau dans le fichier _routes.json_.

> [!NOTE]
> Une fois qu’une requête atteint le niveau des remplacements de la plateforme, les règles de routage ne sont pas réexécutées.

Le tableau ci-après répertorie les remplacements d’erreur de plateforme disponibles :

| Type d’erreur  | Code d'état HTTP | Description |
|---------|---------|---------|
| `NotFound` | 404  | La page est introuvable sur le serveur. |
| `Unauthenticated` | 401 | L’utilisateur n’est pas connecté avec un [fournisseur d’authentification](authentication-authorization.md). |
| `Unauthorized_InsufficientUserInformation` | 401 | Le compte de l’utilisateur sur le fournisseur d’authentification n’est pas configuré pour exposer les données requises. Cette erreur peut par exemple se produire dans les situations où l’application demande au fournisseur d’authentification l’adresse e-mail de l’utilisateur, mais que l’utilisateur a choisi de restreindre l’accès à son adresse e-mail. |
| `Unauthorized_InvalidInvitationLink` | 401 | L’invitation a expiré ou l’utilisateur a suivi un lien d’invitation généré pour un autre destinataire.  |
| `Unauthorized_MissingRoles` | 401 | L’utilisateur n’est pas membre d’un rôle requis. |
| `Unauthorized_TooManyUsers` | 401 | Le site a atteint le nombre maximal d’utilisateurs et le serveur limite les ajouts supplémentaires. Cette erreur est exposée au client, car il n’existe aucune limite quant au nombre d’[invitations](authentication-authorization.md) vous pouvez générer, et certains utilisateurs peuvent ne jamais accepter leur invitation.|
| `Unauthorized_Unknown` | 401 | Un problème inconnu s’est produite lors de la tentative d’authentification de l’utilisateur. Cette erreur peut être due au fait que l’utilisateur n’est pas reconnu, car il n’a pas accordé son consentement à l’application.|

## <a name="custom-mime-types"></a>Types MIME personnalisés

L’objet `mimeTypes`, répertorié au même niveau que le tableau `routes`, vous permet d’associer des [types MIME](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types) à des extensions de fichier.

```json
{
    "routes": [],
    "mimeTypes": {
        "custom": "text/html"
    }
}
```

Dans l’exemple ci-dessus, tous les fichiers avec l’extension `.custom` sont traités avec le type MIME `text/html`.

Les considérations suivantes sont importantes quand vous utilisez des types MIME :

- Les clés ne peuvent pas être de type Null ou vides, ni contenir plus de 50 caractères
- Les valeurs ne peuvent pas être de type Null ou vides, ni contenir plus de 1 000 caractères

> [!NOTE]
> Comme Static Web Apps comprend les applications Blazor et les types MIME attendus pour les fichiers WASM et DLL, vous n’avez pas besoin d’ajouter des mappages pour eux.

## <a name="default-headers"></a>En-têtes par défaut

L’objet `defaultHeaders`, répertorié au même niveau que le tableau `routes`, vous permet d’ajouter, de modifier ou de supprimer des [en-têtes de réponse](https://developer.mozilla.org/docs/Web/HTTP/Headers).

Le fait de fournir une valeur pour un en-tête ajoute ou modifie celui-ci. Si vous fournissez une valeur vide, l’en-tête n’est pas envoyé au client.

```json
{
    "routes": [],
    "defaultHeaders": {
      "content-security-policy": "default-src https: 'unsafe-eval' 'unsafe-inline'; object-src 'none'",
      "cache-control": "must-revalidate, max-age=6000",
      "x-dns-prefetch-control": ""
    }
}
```

Dans l’exemple ci-dessus, un nouvel en-tête `content-security-policy` est ajouté, le `cache-control` modifie la valeur par défaut du serveur et l’en-tête `x-dns-prefectch-control` est supprimé.

Les considérations suivantes sont importantes quand vous utilisez des en-têtes :

- Les clés ne peuvent pas être de type Null ou vides.
- Les valeurs de type Null ou vides suppriment un en-tête du traitement.
- Les clés ou les valeurs ne doivent pas dépasser 8 000 caractères.
- Les en-têtes définis sont traités avec toutes les requêtes.
- Les en-têtes définis dans _routes.json_ s’appliquent uniquement au contenu statique. Vous pouvez personnaliser les en-têtes de réponse d’un point de terminaison d’API dans le code de la fonction.

## <a name="example-route-file"></a>Exemple de fichier de routage

L’exemple suivant montre comment créer des règles d’acheminement pour le contenu statique et les API dans un fichier _routes.json_. Certains itinéraires utilisent le [dossier système _/.auth_](authentication-authorization.md) qui accède aux points de terminaison liés à l’authentification.

```json
{
  "routes": [
    {
      "route": "/profile",
      "allowedRoles": ["authenticated"]
    },
    {
      "route": "/admin/*",
      "allowedRoles": ["administrator"]
    },
    {
      "route": "/api/admin",
      "allowedRoles": ["administrator"]
    },
    {
      "route": "/customers/contoso",
      "allowedRoles": ["administrator", "customers_contoso"]
    },
    {
      "route": "/login",
      "serve": "/.auth/login/github"
    },
    {
      "route": "/.auth/login/twitter",
      "statusCode": "404"
    },
    {
      "route": "/logout",
      "serve": "/.auth/logout"
    },
    {
      "route": "/calendar/*",
      "serve": "/calendar.html"
    },
    {
      "route": "/specials",
      "serve": "/deals",
      "statusCode": 301
    }
  ],
  "platformErrorOverrides": [
    {
      "errorType": "NotFound",
      "serve": "/custom-404.html"
    },
    {
      "errorType": "Unauthenticated",
      "statusCode": "302",
      "serve": "/login"
    }
  ],
  "defaultHeaders": {
    "content-security-policy": "default-src https: 'unsafe-eval' 'unsafe-inline'; object-src 'none'"
  },
  "mimeTypes": {
      "custom": "text/html"
  }
}
```

Les exemples suivants décrivent ce qui se produit lorsqu’une demande correspond à une règle.

| Demande... | Résultat... |
|--|--|--|
| _/profile_ | Les utilisateurs authentifiés reçoivent le fichier _/profile/index.html_. Utilisateurs non authentifiés redirigés vers _/login_. |
| _/admin/reports_ | Les utilisateurs authentifiés dans le rôle _administrateurs_ reçoivent le fichier _/admin/reports/index.html_. Les utilisateurs authentifiés qui ne se trouvent pas dans le rôle _administrateurs_ reçoivent une erreur 401 <sup>2</sup>. Utilisateurs non authentifiés redirigés vers _/login_. |
| _/api/admin_ | Les demandes des utilisateurs authentifiés dans le rôle _administrateurs_ sont envoyées à l’API. Les utilisateurs authentifiés qui ne se trouvent pas dans le rôle _administrateurs_ et les utilisateurs non authentifiés reçoivent une erreur 401. |
| _/customers/contoso_ | Les utilisateurs authentifiés qui appartiennent aux rôles _administrateurs_ ou _clients\_contoso_ reçoivent le fichier _/customers/contoso/index.html_<sup>2</sup>. Les utilisateurs authentifiés qui ne se trouvent pas dans le rôle _administrateurs_ ou _clients\_contoso_ reçoivent une erreur 401. Utilisateurs non authentifiés redirigés vers _/login_. |
| _/login_ | Les utilisateurs non authentifiés sont invités à s’authentifier auprès de GitHub. |
| _/.auth/login/twitter_ | L’autorisation avec Twitter est désactivée. Le serveur répond avec une erreur 404. |
| _/logout_ | Les utilisateurs sont déconnectés de tous les fournisseurs d’authentification. |
| _/calendar/2020/01_ | Le navigateur reçoit le fichier _/calendar.html_. |
| _/specials_ | Le navigateur est redirigé vers _/deals_. |
| _/unknown-folder_ | Le fichier _/custom-404.html_ est envoyé. |
| Fichiers avec l’extension `.custom` | Sont pris en charge avec le type MIME `text/html` |

Toutes les réponses incluent les en-têtes `content-security-policy` avec une valeur de `default-src https: 'unsafe-eval' 'unsafe-inline'; object-src 'none'`.

<sup>1</sup> Les règles de routage pour les fonctions d’API prennent uniquement en charge les [redirections](#redirects) et la [sécurisation des itinéraires avec des rôles](#securing-routes-with-roles).

<sup>2</sup> Vous pouvez fournir une page d’erreur personnalisée en définissant une règle `Unauthorized_MissingRoles` dans le tableau `platformErrorOverrides`.

## <a name="restrictions"></a>Restrictions

- Le fichier _routes.json_ ne peut pas être supérieur à 100 Ko
- Le fichier _routes.json_ prend en charge un maximum de 50 rôles

Consultez [l’article sur les quotas](quotas.md) pour connaître les restrictions et les limitations générales.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Configuration de l’authentification et de l’autorisation](authentication-authorization.md)
