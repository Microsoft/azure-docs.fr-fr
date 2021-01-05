---
title: Gérer la protection intelligente contre le tracking dans Safari | Azure
titleSuffix: Microsoft identity platform
description: Authentification d’application monopage quand les cookies tiers ne sont plus autorisés.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 05/19/2020
ms.author: hirsin
ms.reviewer: kkrishna
ms.custom: aaddev
ms.openlocfilehash: 064c9a00e1cd7c139f3f42a053dcf8a5db13f161
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92104578"
---
# <a name="handle-itp-in-safari-and-other-browsers-where-third-party-cookies-are-blocked"></a>Gérer la protection intelligente contre le tracking dans Safari et d’autres navigateurs où les cookies tiers sont bloqués

Aujourd’hui, de nombreux navigateurs bloquent les cookies tiers, des cookies sur des demandes adressées à des domaines autres que celui affiché dans la barre du navigateur. Cela rompt le flux implicite et nécessite de nouveaux modèles d’authentification pour la connexion des utilisateurs. Dans la plateforme d’identité Microsoft, nous utilisons le flux d’autorisation avec PKCE, et actualisons les jetons pour maintenir les utilisateurs connectés quand des cookies tiers sont bloqués.

## <a name="what-is-intelligent-tracking-protection-itp"></a>Qu’est-ce que la protection intelligente contre le tracking ?

Apple Safari intègre une fonctionnalité de protection de la confidentialité activée par défaut, nommée [protection intelligente contre le tracking](https://webkit.org/tracking-prevention-policy/) ou *ITP* (Intelligent Tracking Protection). La protection intelligente contre le tracking bloque les cookies « tiers », c’est-à-dire des cookies sur des demandes inter-domaines.

Une forme courante de tracking, ou suivi, de l’utilisateur consiste à charger un iframe sur un site tiers en arrière-plan, et à utiliser des cookies pour mettre en corrélation l’utilisateur sur Internet. Malheureusement, ce modèle est également la manière standard d’implémenter le [flux implicite](v2-oauth2-implicit-grant-flow.md) dans des applications monopages. Quand un navigateur bloque des cookies tiers pour empêcher le suivi d’un utilisateur, les applications monopages sont également bloquées.

Safari n’est pas seul navigateur à bloquer les cookies tiers pour améliorer la confidentialité des utilisateurs. Brave bloque les cookies tiers par défaut, et Chromium (la plateforme sur laquelle s’appuient Google Chrome et Microsoft Edge) a annoncé l’arrêt prochain de la prise en charge des cookies tiers.

La solution décrite dans cet article fonctionne dans tous ces navigateurs, ou partout où des cookies tiers sont bloqués.

## <a name="overview-of-the-solution"></a>Vue d’ensemble de la solution

Pour continuer à authentifier les utilisateurs dans des applications monopages, les développeurs d’applications doivent utiliser le [flux de code d’autorisation](v2-oauth2-auth-code-flow.md). Dans le flux de code d’authentification, le fournisseur d’identité émet un code, et l’application monopage échange le code contre un jeton d’accès et un jeton d’actualisation. Lorsque l’application requiert des jetons supplémentaires, elle peut utiliser le [flux de jeton d’actualisation](v2-oauth2-auth-code-flow.md#refresh-the-access-token) pour obtenir de nouveaux jetons. MSAL.js 2,0, la bibliothèque de la plateforme d’identité Microsoft pour les applications monopages, implémente le flux de code d’autorisation pour les applications monopages et, moyennant des mises à jour mineures, constitue un remplacement instantané pour MSAL.js 1.x.

Pour la plateforme d’identité Microsoft, les applications monopages et les clients natifs suivent des recommandations de protocole similaires :

* Utilisation d’une [vérification de code PKCE](https://tools.ietf.org/html/rfc7636)
    * PKCE est *requis* pour les applications monopages sur la plateforme d’identité Microsoft. PKCE est *recommandé* pour les clients natifs et confidentiels.
* Pas d’utilisation de clé secrète client

Les applications monopages ont deux restrictions supplémentaires :

* [L’URI de redirection doit être marqué en tant que type `spa`](v2-oauth2-auth-code-flow.md#redirect-uri-setup-required-for-single-page-apps) pour activer CORS sur les points de terminaison de connexion.
* Les jetons d’actualisation émis via le flux de code d’autorisation vers les URI de redirection `spa` ont une durée de vie de 24 heures au lieu de 90 jours.

:::image type="content" source="media/v2-oauth-auth-code-spa/active-directory-oauth-code-spa.svg" alt-text="Diagramme montrant le flot du code d’autorisation OAuth 2 entre une application à page unique et le point de terminaison du service d’émission de jeton de sécurité." border="false":::

## <a name="performance-and-ux-implications"></a>Implications sur les performances et l’expérience utilisateur

Certaines applications utilisant le flux implicite tentent de se connecter sans effectuer de redirection en ouvrant un iframe de connexion à l’aide de `prompt=none`. Dans la plupart des navigateurs, cette requête répond avec des jetons pour l’utilisateur actuellement connecté (en supposant que le consentement a déjà été donné). Ce modèle signifie que les applications n’ont pas besoin d’une redirection de page complète pour connecter l’utilisateur, ce qui améliore les performances et l’expérience utilisateur, car l’utilisateur visitant la page web et est déjà connecté. Étant donné que `prompt=none` dans un iframe n’est plus une option quand les cookies tiers sont bloqués, les applications doivent visiter la page de connexion dans un cadre de niveau supérieur pour obtenir l’émission d’un code d’autorisation.

Il existe deux façons d’établir une connexion :

* **Redirections de page complète**
    * Lors du premier chargement de l’application monopage, rediriger l’utilisateur vers la page de connexion s’il n’existe aucune session (ou si la session a expiré). Le navigateur de l’utilisateur accède à la page de connexion, présente les cookies contenant la session utilisateur, puis redirige vers l’application avec le code et les jetons dans un fragment.
    * La redirection a pour effet que l’application monopage est chargée deux fois. Suivez les meilleures pratiques de mise en cache des applications monopages afin que l’application ne soit pas entièrement téléchargée deux fois.
    * Songez à instaurer une séquence de préchargement dans l’application, qui vérifie l’existence d’une session de connexion et redirige vers la page de connexion avant que l’application décompresse et exécute entièrement la charge utile JavaScript.
* **Fenêtres contextuelles**
    * Si l’expérience utilisateur (UX) d’une redirection de page complète ne fonctionne pas pour l’application, envisagez d’utiliser une fenêtre contextuelle pour gérer l’authentification.
    * Quand la fenêtre contextuelle achève de redigirer vers l’application après l’authentification, le code dans le gestionnaire de redirection stocke le code et les jetons dans un stockage local à l’usage de l’application. MSAL.js prend en charge les fenêtres contextuelles pour l’authentification, comme la plupart des bibliothèques.
    * Les navigateurs réduisant la prise en charge des fenêtres contextuelles, ils se peut qu’ils ne soient pas l’option la plus fiable. Une interaction de l’utilisateur avec l’application monopage avant la création de la fenêtre contextuelle peut être nécessaire pour répondre aux besoins du navigateur.

>[!NOTE]
> Apple [décrit une méthode de fenêtre contextuelle](https://webkit.org/blog/8311/intelligent-tracking-prevention-2-0/) comme étant un correctif de compatibilité temporaire pour donner à la fenêtre d’origine l’accès à des cookies tiers. Si Apple peut supprimer ce transfert d’autorisations à l’avenir, cela n’aura pas d’incidence sur les conseils prodigués ici. Ici, la fenêtre contextuelle est utilisée en tant que vecteur de navigation interne vers la page de connexion afin qu’une session soit trouvée et qu’un code d’authentification puisse être fourni. Cela devrait continuer à fonctionner à l’avenir.

### <a name="a-note-on-iframe-apps"></a>Remarque sur les applications iframe

Un modèle courant dans les applications web consiste à utiliser un iframe pour incorporer une application dans une autre. Le cadre de niveau supérieur gère l’authentification de l’utilisateur, et l’application hébergée dans l’iframe a la certitude que l’utilisateur est connecté, en extrayant des jetons de manière silencieuse à l’aide du flux implicite. L’acquisition silencieuse du jeton ne fonctionne plus quand les cookies tiers sont bloqués. L’application incorporée dans l’iframe doit basculer vers l’utilisation de fenêtres contextuelles pour accéder à la session de l’utilisateur, car elle ne peut pas accéder à la page de connexion.

## <a name="security-implications-of-refresh-tokens-in-the-browser"></a>Implications pour la sécurité des jetons d’actualisation dans le navigateur

L’émission de jetons d’actualisation à destination du navigateur est considérée comme un problème de sécurité. Des attaques par scripts de site à site ou des packages JS compromis peuvent voler un jeton d’actualisation et l’utiliser à distance jusqu’à ce qu’il expire ou soit révoqué. Afin de réduire le risque de vol de jetons d’actualisation, les jetons émis pour les applications monopages ont une durée de vie limitée à 24 heures. Après 24 heures, l’application doit acquérir un nouveau code d’autorisation en visitant la page de connexion via un cadre de niveau supérieur.

Ce modèle de jeton d’actualisation à durée de vie limitée a été choisi en tant que un compromis entre la sécurité et une expérience utilisateur dégradée. À défaut de jetons d’actualisation ou de cookies tiers, le flux de code d’autorisation (tel que recommandé par le [projet des meilleures pratiques actuelles de sécurité OAuth](https://tools.ietf.org/html/draft-ietf-oauth-security-topics-14)) devient onéreux quand de nouveaux jetons ou des jetons supplémentaires sont requis. Une redirection de page complète ou une fenêtre contextuelle sont requises pour chaque jeton, chaque fois qu’un jeton expire (généralement toutes les heures pour les jetons de plateforme d’identité Microsoft).

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur le [flux de code d’autorisation](v2-oauth2-auth-code-flow.md).

Essayez le flux de code d’autorisation avec le [démarrage rapide MSAL.js 2.0](quickstart-v2-javascript-auth-code.md).
