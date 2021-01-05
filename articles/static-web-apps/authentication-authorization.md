---
title: Authentification et autorisation pour les applications Azure Static Web Apps
description: Apprenez à utiliser différents fournisseurs d’autorisation pour sécuriser votre application statique.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: e95cd313d341844eabf4f5c5feae8a8ca3dc9c2e
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91826541"
---
# <a name="authentication-and-authorization-for-azure-static-web-apps-preview"></a>Authentification et autorisation pour les applications Azure Static Web Apps - Préversion

Azure Static Web Apps simplifie l’expérience d’authentification en gérant l’authentification auprès des fournisseurs suivants :

- Azure Active Directory
- GitHub
- Facebook
- Google<sup>1</sup>
- Twitter

Les [invitations](#invitations) spécifiques au fournisseur associent les utilisateurs à des rôles, et les utilisateurs autorisés bénéficient d’un accès aux [itinéraires](routes.md) via les règles définies dans le fichier _routes.json_.

Tous les fournisseurs d’authentification sont activés par défaut. Pour restreindre un fournisseur d’authentification, [bloquez l’accès](#block-an-authorization-provider) avec une règle d’acheminement personnalisée.

Les rubriques relatives à l’authentification et à l’autorisation ont beaucoup de points communs les concepts de routage. Veillez à lire le [guide relatif au routage](routes.md) avec cet article.

## <a name="roles"></a>Rôles

Tous les utilisateurs qui accèdent à une application web statique appartiennent à un ou plusieurs rôles.  Les utilisateurs peuvent appartenir à deux rôles intégrés :

- **Anonyme** : tous les utilisateurs appartiennent automatiquement au rôle _anonyme_.
- **Authentifié** : tous les utilisateurs qui sont connectés appartiennent au rôle _authentifié_ .

Au-delà des rôles intégrés, vous pouvez créer des rôles, les affecter à des utilisateurs via des invitations et les référencer dans le fichier _routes.json_.

## <a name="role-management"></a>Gestion des rôles

### <a name="add-a-user-to-a-role"></a>Ajouter un utilisateur à un rôle

Pour ajouter des utilisateurs à votre site web, vous générez des invitations qui vous permettent d’associer les utilisateurs à des rôles spécifiques. Les rôles sont définis et gérés dans le fichier _routes.json_.

<a name="invitations" id="invitations"></a>

#### <a name="create-an-invitation"></a>Créer une invitation

Les invitations sont spécifiques à chaque fournisseur d’autorisation. Tenez compte des besoins de votre application lorsque vous sélectionnez les fournisseurs à utiliser. Certains fournisseurs exposent l’adresse e-mail de l’utilisateur, tandis que d’autres fournissent uniquement le nom d’utilisateur sur le site.

<a name="provider-user-details" id="provider-user-details"></a>

| Fournisseur d’autorisation | Expose  |
| ---------------------- | ----------------- |
| Azure Active Directory | de l’adresse de messagerie     |
| Facebook               | de l’adresse de messagerie     |
| GitHub                 | username          |
| Google<sup>1</sup>     | de l’adresse de messagerie     |
| Twitter                | username          |

1. Accédez à la ressource Web Apps statique dans le [Portail Azure](https://portal.azure.com).
1. Sous _Paramètres_, cliquez sur l’onglet **Gestion des rôles**.
1. Cliquez sur le bouton **Inviter**.
1. Sélectionnez un _fournisseur d’autorisation_ dans la liste des options.
1. Ajoutez le nom d’utilisateur ou l’adresse e-mail du destinataire dans la zone _Détails sur l’invité_.
    - Pour GitHub et Twitter, entrez le nom d’utilisateur. Pour tous les autres, entrez l’adresse e-mail du destinataire.
1. Sélectionnez le domaine de votre site statique dans la liste déroulante _Domaine_.
    - Le domaine que vous sélectionnez est le domaine qui apparaît dans l’invitation. Si vous avez un domaine personnalisé associé à votre site, vous souhaiterez probablement choisir ce domaine personnalisé.
1. Ajoutez une liste séparée par des virgules de noms de rôles dans la zone _Rôle_.
1. Entrez le nombre d’heures pendant lesquelles vous souhaitez que l’invitation reste valide.
    - La limite maximale possible est de 168 heures, soit 7 jours.
1. Cliquez sur le bouton **Générer**.
1. Copiez le lien dans la zone _Lien d’invitation_.
1. Envoyez le lien d’invitation à la personne à laquelle vous accordez l’accès à votre application.

Quand l’utilisateur clique sur le lien dans l’invitation, il est invité à se connecter avec le compte correspondant. Une fois la connexion établie, l’utilisateur est associé aux rôles sélectionnés.

> [!CAUTION]
> Vérifiez que vos règles d’acheminement ne sont pas en conflit avec les fournisseurs d’authentification sélectionnés. Le blocage d’un fournisseur avec une règle d’acheminement empêche les utilisateurs d’accepter des invitations.

### <a name="update-role-assignments"></a>Mettre à jour les attributions de rôles

1. Accédez à la ressource Web Apps statique dans le [Portail Azure](https://portal.azure.com).
1. Sous _Paramètres_, cliquez sur l’onglet **Gestion des rôles**.
1. Cliquez sur l’utilisateur dans la liste.
1. Modifiez la liste des rôles dans la zone _Rôle_.
1. Cliquez sur le bouton **Mettre à jour**.

### <a name="remove-user"></a>Supprimer un utilisateur

1. Accédez à la ressource Web Apps statique dans le [Portail Azure](https://portal.azure.com).
1. Sous _Paramètres_, cliquez sur l’onglet **Gestion des rôles**.
1. Recherchez l’utilisateur dans la liste.
1. Activez la case à cocher sur la ligne de l’utilisateur.
1. Cliquez sur le bouton **Supprimer** .

Lorsque vous supprimez un utilisateur, vous devez garder à l’esprit les considérations suivantes :

1. La suppression d’un utilisateur invalide ses autorisations.
1. La propagation globale peut prendre quelques minutes.
1. Si l’utilisateur est de nouveau ajouté à l’application, le [`userId` change](user-information.md).

## <a name="remove-personal-identifying-information"></a>Supprimer les informations d’identification personnelle

Lorsque vous accordez un consentement à une application en tant qu’utilisateur final, l’application a accès à votre adresse e-mail ou à votre nom d’utilisateur, selon le fournisseur d’identité utilisé. Une fois ces informations fournies, le propriétaire de l’application décide comment traiter les informations d’identification personnelle.

Les utilisateurs finaux doivent contacter l’administrateur de chaque application Web pour révoquer ces informations.

Pour supprimer les informations d’identification personnelle de la plateforme Azure Static Web Apps et empêcher la plateforme de fournir ces informations lors de demandes ultérieures, envoyez une demande à l’aide de l’URL suivante :

```url
https://identity.azurestaticapps.net/.auth/purge/<AUTHENTICATION_PROVIDER_NAME>
```

Pour empêcher la plateforme de fournir ces informations lors de demandes ultérieures envoyées aux applications, envoyez une demande à l’URL suivante :

```url
https://<WEB_APP_DOMAIN_NAME>/.auth/purge/<AUTHENTICATION_PROVIDER_NAME>
```

## <a name="system-folder"></a>Dossier système

Azure Static Web Apps utilise le dossier système `/.auth` pour fournir l’accès aux API associées aux autorisations. Au lieu d’exposer les itinéraires situés dans le dossier `/.auth` directement aux utilisateurs finaux, créez des [règles d’acheminement](routes.md) pour créer des URL conviviales.

## <a name="login"></a>Connexion

Utilisez le tableau suivant pour rechercher l’itinéraire de connexion spécifique au fournisseur.

| Fournisseur d’autorisation | Itinéraire de connexion             |
| ---------------------- | ----------------------- |
| Azure Active Directory | `/.auth/login/aad`      |
| Facebook               | `/.auth/login/facebook` |
| GitHub                 | `/.auth/login/github`   |
| Google<sup>1</sup>     | `/.auth/login/google`   |
| Twitter                | `/.auth/login/twitter`  |

Par exemple, pour vous connecter avec GitHub, vous pouvez inclure un lien de connexion comme dans l’extrait de code suivant :

```html
<a href="/.auth/login/github">Login</a>
```

Si vous choisissez de prendre en charge plusieurs fournisseurs, vous devez exposer un lien spécifique au fournisseur pour chacun d’entre eux sur votre site web.

Vous pouvez utiliser une [règle d’acheminement](routes.md) pour mapper un fournisseur par défaut sur un itinéraire convivial comme _/login_.

```json
{
  "route": "/login",
  "serve": "/.auth/login/github"
}
```

### <a name="post-login-redirect"></a>Redirection après connexion

Si vous souhaitez qu’un utilisateur retourne à une page spécifique après la connexion, indiquez une URL dans le paramètre de chaîne de requête `post_login_redirect_uri`.


## <a name="logout"></a>Logout

L’itinéraire `/.auth/logout` déconnecte les utilisateurs du site web. Vous pouvez ajouter un lien vers votre navigation de site pour permettre à l’utilisateur de se déconnecter comme illustré dans l’exemple suivant.

```html
<a href="/.auth/logout">Log out</a>
```

Vous pouvez utiliser une [règle d’acheminement](routes.md) pour mapper un itinéraire convivial comme _/login_.

```json
{
  "route": "/logout",
  "serve": "/.auth/logout"
}
```

### <a name="post-logout-redirect"></a>Redirection après la déconnexion

Si vous souhaitez qu’un utilisateur retourne à une page spécifique après la déconnexion, indiquez une URL dans le paramètre de chaîne de requête `post_logout_redirect_uri`.

## <a name="block-an-authorization-provider"></a>Bloquer un fournisseur d’autorisation

Vous souhaitez peut-être empêcher votre application d’utiliser un fournisseur d’autorisation. Par exemple, votre application peut choisir de n’utiliser que les [fournisseurs qui exposent les adresses e-mail](#provider-user-details).

Pour bloquer un fournisseur, vous pouvez créer des [règles d’acheminement](routes.md) pour retourner un message 404 pour les demandes d’itinéraire bloqué spécifique au fournisseur. Par exemple, pour restreindre Twitter en tant que fournisseur, ajoutez la règle d’acheminement suivante.

```json
{
  "route": "/.auth/login/twitter",
  "statusCode": "404"
}
```

## <a name="restrictions"></a>Restrictions

Consultez [l’article sur les quotas](quotas.md) pour connaître les restrictions et les limitations générales.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Accès à l’authentification de l’utilisateur et aux données d’autorisation](user-information.md)

<sup>1</sup> Expiration imminente du certificat externe.
