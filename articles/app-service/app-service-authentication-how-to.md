---
title: Utilisation avancée de l’authentification et de l’autorisation
description: Apprenez à personnaliser les paramètres d’authentification et d’autorisation dans App Service pour différents scénarios et à obtenir des revendications d’utilisateur et des jetons distincts.
ms.topic: article
ms.date: 07/08/2020
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 85fd7fdba4c62f4837a419af44c83f7e46cb9e39
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96601779"
---
# <a name="advanced-usage-of-authentication-and-authorization-in-azure-app-service"></a>Utilisation avancée des paramètres d’authentification et d’autorisation dans Azure App Service

Cet article vous explique comment personnaliser les [paramètres d’authentification et d’autorisation intégrés dans App Service](overview-authentication-authorization.md), et comment gérer les identités à partir de votre application. 

Pour commencer rapidement, consultez l’un des didacticiels suivants :

* [Tutoriel : Authentifier et autoriser des utilisateurs de bout en bout dans Azure App Service](tutorial-auth-aad.md)
* [Comment configurer votre application pour utiliser une connexion Azure Active Directory](configure-authentication-provider-aad.md)
* [Comment configurer votre application pour utiliser une connexion Facebook](configure-authentication-provider-facebook.md)
* [Comment configurer votre application pour utiliser une connexion Google](configure-authentication-provider-google.md)
* [Comment configurer votre application pour utiliser une connexion par compte Microsoft](configure-authentication-provider-microsoft.md)
* [Comment configurer votre application pour utiliser une connexion Twitter](configure-authentication-provider-twitter.md)
* [Guide pratique pour configurer une application de sorte qu’elle se connecte avec un fournisseur OpenID Connect (préversion)](configure-authentication-provider-openid-connect.md)
* [Configurer votre application pour se connecter à l’aide d’Apple (préversion)](configure-authentication-provider-apple.md)

## <a name="use-multiple-sign-in-providers"></a>Utiliser plusieurs fournisseurs de connexion

La configuration du portail n’offre pas de solution clé en main pour présenter à vos utilisateurs plusieurs fournisseurs de connexion (telles que Facebook et Twitter). Toutefois, il est relativement simple d’ajouter cette fonctionnalité à votre application. Voici la procédure à suivre :

Tout d’abord, sur la page **Authentification / Autorisation** du portail Azure, configurez chaque fournisseur d’identité que vous souhaitez activer.

Sous **Mesure à prendre quand une demande n’est pas authentifiée**, sélectionnez **Autoriser les requêtes anonymes (aucune action)** .

Dans la page de connexion, la barre de navigation ou tout autre emplacement de votre application, ajoutez un lien de connexion pour chacun des fournisseurs que vous avez activés (`/.auth/login/<provider>`). Par exemple :

```html
<a href="/.auth/login/aad">Log in with Azure AD</a>
<a href="/.auth/login/microsoftaccount">Log in with Microsoft Account</a>
<a href="/.auth/login/facebook">Log in with Facebook</a>
<a href="/.auth/login/google">Log in with Google</a>
<a href="/.auth/login/twitter">Log in with Twitter</a>
<a href="/.auth/login/apple">Log in with Apple</a>
```

Lorsque l’utilisateur clique sur l’un des liens, la page de connexion respective s’ouvre pour que l’utilisateur se connecte.

Pour rediriger l’utilisateur post-connexion vers une URL personnalisée, utilisez le paramètre de chaîne de requête `post_login_redirect_url` (à ne pas confondre avec l’URI de redirection de votre configuration de fournisseur d’identité). Par exemple, pour diriger l’utilisateur vers `/Home/Index` après sa connexion, utilisez le code HTML suivant :

```html
<a href="/.auth/login/<provider>?post_login_redirect_url=/Home/Index">Log in</a>
```

## <a name="validate-tokens-from-providers"></a>Valider les jetons des fournisseurs

Dans une connexion dirigée par le client, l'application connecte manuellement l'utilisateur au fournisseur, puis soumet le jeton d'authentification à App Service pour validation (voir [Flux d'authentification](overview-authentication-authorization.md#authentication-flow)). Cette validation proprement dite ne vous octroie pas l'accès aux ressources souhaitées, mais une validation réussie vous conférera un jeton de session que vous pourrez utiliser pour accéder aux ressources de l'application. 

Pour valider le jeton du fournisseur, l'application App Service doit d'abord être configurée avec le fournisseur souhaité. Au moment de l'exécution, après avoir récupéré le jeton d'authentification auprès de votre fournisseur, envoyez-le à `/.auth/login/<provider>` pour validation. Par exemple : 

```
POST https://<appname>.azurewebsites.net/.auth/login/aad HTTP/1.1
Content-Type: application/json

{"id_token":"<token>","access_token":"<token>"}
```

Le format du jeton varie légèrement selon le fournisseur. Pour plus de détails, consultez le tableau suivant :

| Valeur du fournisseur | Requis dans le corps de la demande | Commentaires |
|-|-|-|
| `aad` | `{"access_token":"<access_token>"}` | |
| `microsoftaccount` | `{"access_token":"<token>"}` | La propriété `expires_in` est facultative. <br/>Lors de la demande de jeton à partir des services Live, demandez toujours l'étendue `wl.basic`. |
| `google` | `{"id_token":"<id_token>"}` | La propriété `authorization_code` est facultative. Lorsqu'elle est spécifiée, elle peut également être accompagnée de la propriété `redirect_uri`. |
| `facebook`| `{"access_token":"<user_access_token>"}` | Utilisez un [jeton d'accès utilisateur](https://developers.facebook.com/docs/facebook-login/access-tokens) valide à partir de Facebook. |
| `twitter` | `{"access_token":"<access_token>", "access_token_secret":"<acces_token_secret>"}` | |
| | | |

Si le jeton du fournisseur est validé, l'API renvoie un `authenticationToken` dans le corps de la réponse. Il s'agit de votre jeton de session. 

```json
{
    "authenticationToken": "...",
    "user": {
        "userId": "sid:..."
    }
}
```

Une fois en possession de ce jeton de session, vous pouvez accéder aux ressources d'application protégées en ajoutant l'en-tête `X-ZUMO-AUTH` à vos requêtes HTTP. Par exemple : 

```
GET https://<appname>.azurewebsites.net/api/products/1
X-ZUMO-AUTH: <authenticationToken_value>
```

## <a name="sign-out-of-a-session"></a>Déconnexion d’une session

Les utilisateurs peuvent initier une déconnexion en envoyant une requête `GET` au point de terminaison `/.auth/logout` de l’application. La requête `GET` effectue les opérations suivantes :

- Efface les cookies d’authentification de la session active.
- Supprime du magasin de jetons les jetons de l’utilisateur actuel.
- Pour Azure Active Directory et Google, effectue une déconnexion côté serveur sur le fournisseur d’identité.

Voici un lien de déconnexion simple dans une page web :

```html
<a href="/.auth/logout">Sign out</a>
```

Par défaut, une déconnexion réussie redirige le client vers l’URL `/.auth/logout/done`. Vous pouvez modifier la page de redirection après déconnexion en ajoutant le paramètre de requête `post_logout_redirect_uri`. Par exemple :

```
GET /.auth/logout?post_logout_redirect_uri=/index.html
```

Il est recommandé d’[encoder](https://wikipedia.org/wiki/Percent-encoding) la valeur de `post_logout_redirect_uri`.

Lorsque vous utilisez une URL complète, celle-ci doit être hébergée dans le même domaine, ou configurée comme URL de redirection externe autorisée pour votre application. Dans l’exemple suivant, pour rediriger vers l’URL `https://myexternalurl.com` qui n’est pas hébergée dans le même domaine :

```
GET /.auth/logout?post_logout_redirect_uri=https%3A%2F%2Fmyexternalurl.com
```

Exécutez la commande suivante dans [Azure Cloud Shell](../cloud-shell/quickstart.md) :

```azurecli-interactive
az webapp auth update --name <app_name> --resource-group <group_name> --allowed-external-redirect-urls "https://myexternalurl.com"
```

## <a name="preserve-url-fragments"></a>Conserver les fragments d’URL

Lorsque des utilisateurs se connectent à votre application, ils veulent être redirigés vers la même section de la même page, par exemple, `/wiki/Main_Page#SectionZ`. Toutefois, étant donné que des [fragments d’URL](https://wikipedia.org/wiki/Fragment_identifier) (par exemple, `#SectionZ`) ne sont jamais envoyés au serveur, ils ne sont pas conservés par défaut après que la connexion OAuth est terminée et redirige vers votre application. Les utilisateurs vivent ensuite une expérience médiocre quand ils ont besoin d’accéder à nouveau à l’ancre souhaitée. Cette limitation s’applique à toutes les solutions d’authentification côté serveur.

Dans l’authentification App Service, vous pouvez conserver des fragments d’URL dans le cadre de la connexion OAuth. Pour ce faire, définissez un paramètre d’application nommé `WEBSITE_AUTH_PRESERVE_URL_FRAGMENT` sur `true`. Vous pouvez le faire dans le [portail Azure](https://portal.azure.com), ou simplement exécuter la commande suivante dans [Azure Cloud Shell](../cloud-shell/quickstart.md) :

```azurecli-interactive
az webapp config appsettings set --name <app_name> --resource-group <group_name> --settings WEBSITE_AUTH_PRESERVE_URL_FRAGMENT="true"
```

## <a name="access-user-claims"></a>Accéder aux revendications d’utilisateur

App Service transmet des revendications d’utilisateur à votre application au moyen d’en-têtes spéciaux. Les demandes externes ne sont pas autorisées à définir ces en-têtes. Ces derniers ne s’affichent donc que s’ils sont définis par App Service. Voici quelques exemples d’en-têtes :

* X-MS-CLIENT-PRINCIPAL-NAME
* X-MS-CLIENT-PRINCIPAL-ID

Tout code, quels que soient le langage ou l’infrastructure utilisés, peut trouver les informations qu’il recherche dans ces en-têtes. Dans le cas d’applications ASP.NET 4.6, le paramètre **ClaimsPrincipal** est automatiquement défini sur les valeurs appropriées. Toutefois, ASP.NET Core ne fournit pas d’intergiciel d’authentification qui s’intègre aux revendications d’utilisateur App Service. Pour une solution de contournement, consultez [MaximeRouiller.Azure.AppService.EasyAuth](https://github.com/MaximRouiller/MaximeRouiller.Azure.AppService.EasyAuth).

Si le [magasin de jetons](overview-authentication-authorization.md#token-store) est activé pour votre application, vous pouvez également obtenir des informations supplémentaires sur l’utilisateur authentifié en appelant `/.auth/me`. Les Kits de développement logiciel (SDK) serveur de Mobile Apps offrent des méthodes d’assistance permettant de manipuler ces données. Pour plus d’informations, consultez [Comment utiliser le Kit de développement logiciel Node.js dans Azure Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk#howto-tables-getidentity), et [Utiliser le Kit de développement logiciel (SDK) de serveur principal .NET pour Azure Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk#user-info).

## <a name="retrieve-tokens-in-app-code"></a>Récupérer des jetons dans le code d’application

À partir de votre code serveur, les jetons spécifiques au fournisseur sont ajoutés dans l’en-tête de requête, afin que vous puissiez y accéder facilement. Le tableau suivant présente les noms d’en-tête de jeton possibles :

| Fournisseur | Noms des en-têtes |
|-|-|
| Azure Active Directory | `X-MS-TOKEN-AAD-ID-TOKEN` <br/> `X-MS-TOKEN-AAD-ACCESS-TOKEN` <br/> `X-MS-TOKEN-AAD-EXPIRES-ON`  <br/> `X-MS-TOKEN-AAD-REFRESH-TOKEN` |
| Jeton Facebook | `X-MS-TOKEN-FACEBOOK-ACCESS-TOKEN` <br/> `X-MS-TOKEN-FACEBOOK-EXPIRES-ON` |
| Google | `X-MS-TOKEN-GOOGLE-ID-TOKEN` <br/> `X-MS-TOKEN-GOOGLE-ACCESS-TOKEN` <br/> `X-MS-TOKEN-GOOGLE-EXPIRES-ON` <br/> `X-MS-TOKEN-GOOGLE-REFRESH-TOKEN` |
| Compte Microsoft | `X-MS-TOKEN-MICROSOFTACCOUNT-ACCESS-TOKEN` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-EXPIRES-ON` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-AUTHENTICATION-TOKEN` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-REFRESH-TOKEN` |
| Twitter | `X-MS-TOKEN-TWITTER-ACCESS-TOKEN` <br/> `X-MS-TOKEN-TWITTER-ACCESS-TOKEN-SECRET` |
|||

À partir de votre code client (par exemple, une application mobile ou un navigateur JavaScript), envoyez une requête `GET` HTTP à `/.auth/me` (le [magasin de jetons](overview-authentication-authorization.md#token-store) doit être activé). La réponse JSON retournée contient les jetons spécifiques au fournisseur.

> [!NOTE]
> Les jetons d’accès permettent d’accéder aux ressources du fournisseur. Ils ne s’affichent donc que si vous configurez votre fournisseur avec une clé secrète client. Pour savoir comment obtenir des jetons d’actualisation, consultez Actualiser des jetons d’accès.

## <a name="refresh-identity-provider-tokens"></a>Actualiser des jetons de fournisseur d’identité

Lorsque le jeton d'accès de votre fournisseur (et non le [jeton de session](#extend-session-token-expiration-grace-period)) expire, vous devez réauthentifier l’utilisateur avant de réutiliser ce jeton. Vous pouvez éviter l’expiration du jeton en effectuant un appel `GET` au point de terminaison `/.auth/refresh` de votre application. Lorsqu’il est appelé, App Service actualise automatiquement les jetons d’accès dans le [magasin de jetons](overview-authentication-authorization.md#token-store) pour l’utilisateur authentifié. Les demandes de jeton suivantes effectuées via le code de votre application permettent d’obtenir les jetons actualisés. Toutefois, pour que l’actualisation des jetons soit effective, le magasin de jetons doit contenir les [jetons d’actualisation](https://auth0.com/learn/refresh-tokens/) pour votre fournisseur. La procédure pour obtenir des jetons d’actualisation est fournie par chaque fournisseur. La liste suivante en fournit toutefois un bref résumé :

- **Google** : ajouter un paramètre de chaîne de requête `access_type=offline` à votre appel d’API `/.auth/login/google`. Si vous utilisez le kit de développement logiciel Mobile Apps, vous pouvez ajouter le paramètre à l’une des surcharges `LogicAsync` (voir [Google Refresh Tokens](https://developers.google.com/identity/protocols/OpenIDConnect#refresh-tokens) (Jetons d’actualisation Google)).
- **Facebook** : ne fournit pas de jetons d’actualisation. Les jetons de longue durée expirent au bout de 60 jours (voir [Facebook Expiration and Extension of Access Tokens](https://developers.facebook.com/docs/facebook-login/access-tokens/expiration-and-extension) (Expiration et prolongation des jetons d’accès Facebook)).
- **Twitter** : les jetons d’accès n’expirent pas (voir les [questions fréquentes sur Twitter OAuth](https://developer.twitter.com/en/docs/authentication/faq)).
- **Compte Microsoft** : au moment de [configurer les paramètres d’authentification de compte Microsoft](configure-authentication-provider-microsoft.md), sélectionnez l’étendue `wl.offline_access`.
- **Azure Active Directory** : Dans [https://resources.azure.com](https://resources.azure.com), effectuez les étapes suivantes :
    1. En haut de la page, sélectionnez **Lecture/écriture**.
    2. Dans le navigateur de gauche, accédez à **subscriptions** > ** _\<subscription\_name_** > **resourceGroups** > **_ \<resource\_group\_name> _** > **providers** > **Microsoft.Web** > **sites** > **_ \<app\_name>_** > **config** > **authsettings**. 
    3. Cliquez sur **Modifier**.
    4. Modifiez la propriété suivante. Remplacez la valeur _\<app\_id>_ par l’ID d’application Azure Active Directory du service auquel vous voulez accéder.

        ```json
        "additionalLoginParams": ["response_type=code id_token", "resource=<app_id>"]
        ```

    5. Cliquez sur **Put**. 

Une fois que votre fournisseur est configuré, vous pouvez [rechercher le jeton d’actualisation et l’heure d’expiration pour le jeton d’accès](#retrieve-tokens-in-app-code) dans le magasin de jetons. 

Pour actualiser votre jeton d’accès à tout moment, il vous suffit d’appeler `/.auth/refresh` dans n’importe quel langage. L’extrait de code suivant utilise jQuery pour actualiser vos jetons d’accès à partir d’un client JavaScript.

```javascript
function refreshTokens() {
  let refreshUrl = "/.auth/refresh";
  $.ajax(refreshUrl) .done(function() {
    console.log("Token refresh completed successfully.");
  }) .fail(function() {
    console.log("Token refresh failed. See application logs for details.");
  });
}
```

Si un utilisateur révoque les autorisations accordées à votre application, votre appel à `/.auth/me` peut échouer avec une réponse `403 Forbidden`. Pour diagnostiquer les erreurs, consultez vos journaux d’activité d’application pour obtenir des informations supplémentaires.

## <a name="extend-session-token-expiration-grace-period"></a>Étendre la période de grâce d’expiration de jeton de session

La session authentifiée expire au bout de 8 heures. Après expiration d’une session authentifiée, une période de grâce de 72 heures s’applique par défaut. Au cours de cette période de grâce, vous pouvez actualiser le jeton de session via App Service sans réauthentifier l’utilisateur. Vous pouvez simplement appeler `/.auth/refresh` lorsque jeton de session n’est plus valide. Vous n’avez pas besoin de vérifier vous-même les dates d’expiration des jetons. À la fin de la période de grâce de 72 heures, l’utilisateur doit se connecter à nouveau pour obtenir un jeton de session valide.

Si le délai de 72 heures n’est pas suffisamment long pour vous, vous pouvez étendre cette fenêtre d’expiration. Étendre la fenêtre d’expiration sur une longue période peut avoir une incidence majeure sur la sécurité (par exemple lorsqu’un jeton d’authentification est divulgué ou volé). Nous vous recommandons donc de conserver la valeur par défaut de 72 heures ou de définir la période d’extension sur la valeur minimale.

Pour étendre la fenêtre d’expiration par défaut, exécutez la commande suivante dans [Cloud Shell](../cloud-shell/overview.md).

```azurecli-interactive
az webapp auth update --resource-group <group_name> --name <app_name> --token-refresh-extension-hours <hours>
```

> [!NOTE]
> La période de grâce s’applique uniquement à la session App Service authentifiée, et non aux jetons fournis par les fournisseurs d’identité. Il n’existe aucune période de grâce pour les jetons de fournisseur ayant expiré. 
>

## <a name="limit-the-domain-of-sign-in-accounts"></a>Restreindre le domaine des comptes de connexion

Les options Compte Microsoft et Azure Active Directory permettent de se connecter à partir de multiples domaines. Par exemple, l’option Compte Microsoft prend en charge les comptes _outlook.com_, _live.com_ et _hotmail.com_. Azure Active Directory prend en charge un nombre illimité de domaines personnalisés pour les comptes de connexion. Toutefois, vous pouvez accélérer l’accès de vos utilisateurs directement à votre propre page de connexion Azure AD (par exemple, `contoso.com`). Pour suggérer le nom de domaine des comptes de connexion, procédez comme suit.

Dans [https://resources.azure.com](https://resources.azure.com), accédez à **subscriptions**> ** _\<subscription\_name_** > **resourceGroups** > **_ \<resource\_group\_name> _** > **providers** > **Microsoft.Web** > **sites** > **_ \<app\_name> _** > **config** > **authsettings**. 

Cliquez sur **Modifier**, modifiez la propriété suivante, puis cliquez sur **Put**. Veillez à remplacer la valeur _\<domain\_name>_ par le domaine souhaité.

```json
"additionalLoginParams": ["domain_hint=<domain_name>"]
```

Ce paramètre ajoute le paramètre de chaîne de requête `domain_hint` à l’URL de redirection de la connexion. 

> [!IMPORTANT]
> Le client peut supprimer le paramètre `domain_hint` après avoir reçu l’URL de redirection, puis de se connecter avec un autre domaine. Bien que cette fonction soit pratique, il ne s’agit pas d’une fonctionnalité de sécurité.
>

## <a name="authorize-or-deny-users"></a>Autoriser ou refuser des utilisateurs

Bien que App Service se charge du cas d’autorisation le plus simple (c’est-à-dire le rejet des demandes non authentifiées), votre application peut nécessiter un comportement d’autorisation plus précis, tel que la limitation de l’accès à un groupe spécifique d’utilisateurs. Dans certains cas, vous devez écrire un code d’application personnalisé pour autoriser ou refuser l’accès à l’utilisateur connecté. Dans d’autres cas, App Service ou votre fournisseur d’identité peut être en mesure de vous aider sans modification du code.

- [Au niveau du serveur](#server-level-windows-apps-only)
- [Au niveau du fournisseur d’identité](#identity-provider-level)
- [Au niveau de l’application](#application-level)

### <a name="server-level-windows-apps-only"></a>Au niveau du serveur (applications Windows uniquement)

Pour toute application Windows, vous pouvez définir le comportement d’autorisation du serveur web IIS en modifiant le fichier *Web.config*. Les applications Linux n’utilisent pas IIS et ne peuvent pas être configurées à l’aide du fichier *Web.config*.

1. Accédez à `https://<app-name>.scm.azurewebsites.net/DebugConsole`.

1. Dans l’explorateur de navigateur de vos fichiers App Service, accédez à *site/wwwroot*. S’il n’existe pas de fichier *Web.config*, créez-en un en sélectionnant **+**  > **Nouveau fichier**. 

1. Sélectionnez le crayon correspondant à *Web. config* pour le modifier. Ajoutez le code de configuration suivant, puis cliquez sur **Enregistrer**. Si le fichier *Web.config* existe, ajoutez simplement l’élément `<authorization>` avec tout ce qui se trouve dans celui-ci. Ajoutez les comptes que vous souhaitez autoriser dans l’élément `<allow>`.

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
       <system.web>
          <authorization>
            <allow users="user1@contoso.com,user2@contoso.com"/>
            <deny users="*"/>
          </authorization>
       </system.web>
    </configuration>
    ```

### <a name="identity-provider-level"></a>Au niveau du fournisseur d’identité

Le fournisseur d’identité peut fournir un autorisation clé en main. Par exemple :

- Pour [Azure App Service](configure-authentication-provider-aad.md), vous pouvez [gérer l’accès au niveau entreprise](../active-directory/manage-apps/what-is-access-management.md) directement dans Azure AD. Pour obtenir des instructions, voir [Comprendre comment retirer l’accès d’un utilisateur à une application](../active-directory/manage-apps/methods-for-removing-user-access.md).
- Pour [Google](configure-authentication-provider-google.md) , les projets d’API Google appartenant à une [organisation](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy#organizations) peuvent être configurés de façon à autoriser l’accès uniquement aux utilisateurs de votre organisation (voir la [page de support **Setting up OAuth 2.0** de Google](https://support.google.com/cloud/answer/6158849?hl=en)).

### <a name="application-level"></a>Niveau d’application

Si l’un des autres niveaux ne fournit pas l’autorisation dont vous avez besoin, ou si votre plateforme ou fournisseur d’identité ne sont pas pris en charge, vous devez écrire du code personnalisé afin d’autoriser les utilisateurs en fonction de leurs [revendications](#access-user-claims).

## <a name="configure-using-a-file-preview"></a><a name="config-file"> </a>Configuration à l’aide d’un fichier (préversion)

Vos paramètres d’authentification peuvent éventuellement être configurés à l’aide d’un fichier fourni par votre déploiement. Certaines fonctionnalités en préversion de l’authentification/autorisation App Service l’exigent.

> [!IMPORTANT]
> N’oubliez pas que la charge utile de votre application et, par conséquent, ce fichier peuvent passer d’un environnement à un autre, comme avec les [emplacements](./deploy-staging-slots.md). Si vous souhaitez épingler une inscription d’application différente pour chaque emplacement, ce qui est probablement le cas, continuez à appliquer la méthode de configuration standard au lieu d’utiliser le fichier de configuration.

### <a name="enabling-file-based-configuration"></a>Activation de la configuration à l’aide d’un fichier

> [!CAUTION]
> Lors de la préversion, le fait d’activer la configuration à l’aide d’un fichier a pour effet de désactiver la gestion de la fonctionnalité d’authentification/autorisation App Service de l’application par le biais de certains clients, comme le Portail Azure, Azure CLI et Azure PowerShell.

1. Créez un fichier JSON pour votre configuration à la racine de votre projet (déployé sur D:\home\site\wwwroot dans votre application web/de fonction). Renseignez la configuration de votre choix suivant les [informations de référence de la configuration à l’aide d’un fichier](#configuration-file-reference). Si vous modifiez une configuration Azure Resource Manager existante, veillez à traduire les propriétés capturées dans la collection `authsettings` dans votre fichier de configuration.

2. Modifiez la configuration existante, capturée dans les API [Azure Resource Manager](../azure-resource-manager/management/overview.md) sous `Microsoft.Web/sites/<siteName>/config/authsettings`. Pour cela, vous pouvez utiliser un [modèle Azure Resource Manager](../azure-resource-manager/templates/overview.md) ou un outil comme [Azure Resource Explorer](https://resources.azure.com/). Au sein de la collection authsettings, vous devez définir trois propriétés (et éventuellement en supprimer d’autres) :

    1.  Définissez `enabled` sur « true ».
    2.  Définissez `isAuthFromFile` sur « true ».
    3.  Définissez `authFilePath` sur le nom du fichier (par exemple, « auth.json »).

> [!NOTE]
> Le format de `authFilePath` varie d’une plateforme à l’autre. Sur Windows, les chemins d’accès relatifs et absolus sont pris en charge. Les chemins d’accès relatifs sont recommandés. Pour Linux, seuls des chemins d’accès absolus sont actuellement pris en charge. Par conséquent, la valeur du paramètre doit être « /Home/site/wwwroot/auth.JSON » ou une valeur similaire.

Une fois cette mise à jour de la configuration effectuée, le contenu du fichier permet de définir le comportement de l’authentification/autorisation App Service pour ce site. Si vous souhaitez un jour revenir à la configuration Azure Resource Manager, réaffectez à `isAuthFromFile` la valeur « false ».

### <a name="configuration-file-reference"></a>Informations de référence sur le fichier de configuration

Tous les secrets auquel le fichier de configuration fait référence doivent être stockés en tant que [paramètres d’application](./configure-common.md#configure-app-settings). Vous pouvez donner le nom de votre choix aux paramètres. Veillez simplement à ce que les références du fichier de configuration utilisent les mêmes clés.

Voici toutes les configuration possibles dans le fichier :

```json
{
    "platform": {
        "enabled": <true|false>
    },
    "globalValidation": {
        "unauthenticatedClientAction": "RedirectToLoginPage|AllowAnonymous|Return401|Return403",
        "redirectToProvider": "<default provider alias>",
        "excludedPaths": [
            "/path1",
            "/path2"
        ]
    },
    "httpSettings": {
        "requireHttps": <true|false>,
        "routes": {
            "apiPrefix": "<api prefix>"
        },
        "forwardProxy": {
            "convention": "NoProxy|Standard|Custom",
            "customHostHeaderName": "<host header value>",
            "customProtoHeaderName": "<proto header value>"
        }
    },
    "login": {
        "routes": {
            "logoutEndpoint": "<logout endpoint>"
        },
        "tokenStore": {
            "enabled": <true|false>,
            "tokenRefreshExtensionHours": "<double>",
            "fileSystem": {
                "directory": "<directory to store the tokens in if using a file system token store (default)>"
            },
            "azureBlobStorage": {
                "sasUrlSettingName": "<app setting name containing the sas url for the Azure Blob Storage if opting to use that for a token store>"
            }
        },
        "preserveUrlFragmentsForLogins": <true|false>,
        "allowedExternalRedirectUrls": [
            "https://uri1.azurewebsites.net/",
            "https://uri2.azurewebsites.net/",
            "url_scheme_of_your_app://easyauth.callback"
        ],
        "cookieExpiration": {
            "convention": "FixedTime|IdentityDerived",
            "timeToExpiration": "<timespan>"
        },
        "nonce": {
            "validateNonce": <true|false>,
            "nonceExpirationInterval": "<timespan>"
        }
    },
    "identityProviders": {
        "azureActiveDirectory": {
            "enabled": <true|false>,
            "registration": {
                "openIdIssuer": "<issuer url>",
                "clientId": "<app id>",
                "clientSecretSettingName": "APP_SETTING_CONTAINING_AAD_SECRET",
            },
            "login": {
                "loginParameters": [
                    "paramName1=value1",
                    "paramName2=value2"
                ]
            },
            "validation": {
                "allowedAudiences": [
                    "audience1",
                    "audience2"
                ]
            }
        },
        "facebook": {
            "enabled": <true|false>,
            "registration": {
                "appId": "<app id>",
                "appSecretSettingName": "APP_SETTING_CONTAINING_FACEBOOK_SECRET"
            },
            "graphApiVersion": "v3.3",
            "login": {
                "scopes": [
                    "public_profile",
                    "email"
                ]
            },
        },
        "gitHub": {
            "enabled": <true|false>,
            "registration": {
                "clientId": "<client id>",
                "clientSecretSettingName": "APP_SETTING_CONTAINING_GITHUB_SECRET"
            },
            "login": {
                "scopes": [
                    "profile",
                    "email"
                ]
            }
        },
        "google": {
            "enabled": true,
            "registration": {
                "clientId": "<client id>",
                "clientSecretSettingName": "APP_SETTING_CONTAINING_GOOGLE_SECRET"
            },
            "login": {
                "scopes": [
                    "profile",
                    "email"
                ]
            },
            "validation": {
                "allowedAudiences": [
                    "audience1",
                    "audience2"
                ]
            }
        },
        "twitter": {
            "enabled": <true|false>,
            "registration": {
                "consumerKey": "<consumer key>",
                "consumerSecretSettingName": "APP_SETTING_CONTAINING TWITTER_CONSUMER_SECRET"
            }
        },
        "apple": {
            "enabled": <true|false>,
            "registration": {
                "clientId": "<client id>",
                "clientSecretSettingName": "APP_SETTING_CONTAINING_APPLE_SECRET"
            },
            "login": {
                "scopes": [
                    "profile",
                    "email"
                ]
            }
        },
        "openIdConnectProviders": {
            "<providerName>": {
                "enabled": <true|false>,
                "registration": {
                    "clientId": "<client id>",
                    "clientCredential": {
                        "clientSecretSettingName": "<name of app setting containing client secret>"
                    },
                    "openIdConnectConfiguration": {
                        "authorizationEndpoint": "<url specifying authorization endpoint>",
                        "tokenEndpoint": "<url specifying token endpoint>",
                        "issuer": "<url specifying issuer>",
                        "certificationUri": "<url specifying jwks endpoint>",
                        "wellKnownOpenIdConfiguration": "<url specifying .well-known/open-id-configuration endpoint - if this property is set, the other properties of this object are ignored, and authorizationEndpoint, tokenEndpoint, issuer, and certificationUri are set to the corresponding values listed at this endpoint>"
                    }
                },
                "login": {
                    "nameClaimType": "<name of claim containing name>",
                    "scopes": [
                        "openid",
                        "profile",
                        "email"
                    ],
                    "loginParameterNames": [
                        "paramName1=value1",
                        "paramName2=value2"
                    ],
                }
            },
            //...
        }
    }
}
```

## <a name="pin-your-app-to-a-specific-authentication-runtime-version"></a>Épingler votre application à une version spécifique du runtime d’authentification

Lorsque vous activez l’authentification/l’autorisation, l’intergiciel (middleware) de la plateforme est injecté dans votre pipeline de requêtes HTTP, comme décrit dans la [vue d’ensemble des fonctionnalités](overview-authentication-authorization.md#how-it-works). Cet intergiciel (middleware) de plateforme est régulièrement mis à jour avec de nouvelles fonctionnalités et des améliorations dans le cadre des mises à jour de routine de la plateforme. Par défaut, votre application web ou de fonction s’exécutera sur la dernière version de cet intergiciel (middleware) de plateforme. Ces mises à jour automatiques sont toujours à compatibilité descendante. Toutefois, dans les rares cas où cette mise à jour automatique introduit un problème de runtime pour votre application web ou de fonction, vous pouvez revenir temporairement à la version précédente de l’intergiciel (middleware). Cet article explique comment épingler temporairement une application à une version spécifique de l’intergiciel (middleware) d’authentification.

### <a name="automatic-and-manual-version-updates"></a>Mises à jour de versions automatiques et manuelles 

Vous pouvez épingler votre application à une version spécifique de l’intergiciel (middleware) de plateforme en définissant un paramètre `runtimeVersion` pour l’application. Votre application s’exécute toujours sur la version la plus récente, sauf si vous choisissez de la réépingler explicitement à une version spécifique. Plusieurs versions sont prises en charge à la fois. Si vous épinglez votre application à une version non valide qui n’est plus prise en charge, votre application utilisera la version la plus récente à la place. Pour toujours exécuter la version la plus récente, définissez `runtimeVersion` sur ~1. 

### <a name="view-and-update-the-current-runtime-version"></a>Afficher et mettre à jour la version actuelle du runtime

Vous pouvez modifier la version du runtime utilisée par votre application. La nouvelle version du runtime doit prendre effet après le redémarrage de l’application. 

#### <a name="view-the-current-runtime-version"></a>Afficher la version actuelle du runtime

Vous pouvez afficher la version actuelle de l’intergiciel (middleware) d’authentification de la plateforme à l’aide d’Azure CLI ou via l’un des points de terminaison HTTP de la version intégrés dans votre application.

##### <a name="from-the-azure-cli"></a>Dans Azure CLI

À l’aide d’Azure CLI, affichez la version actuelle de l’intergiciel (middleware) avec la commande [az webapp auth show](/cli/azure/webapp/auth?view=azure-cli-latest&preserve-view=true#az-webapp-auth-show).

```azurecli-interactive
az webapp auth show --name <my_app_name> \
--resource-group <my_resource_group>
```

Dans ce code, remplacez `<my_app_name>` par le nom de votre application. Remplacez également `<my_resource_group>` par le nom du groupe de ressources de votre application.

Le champ `runtimeVersion` est affiché dans la sortie de l’interface CLI. Il ressemble à l’exemple de sortie suivant, qui a été tronqué pour une meilleure lisibilité : 
```output
{
  "additionalLoginParams": null,
  "allowedAudiences": null,
    ...
  "runtimeVersion": "1.3.2",
    ...
}
```

##### <a name="from-the-version-endpoint"></a>À partir du point de terminaison de version

Vous pouvez également atteindre le point de terminaison /.auth/version sur une application pour afficher la version actuelle de l’intergiciel (middleware) sur laquelle l’application s’exécute. Cela ressemble à l’exemple de sortie suivant :
```output
{
"version": "1.3.2"
}
```

#### <a name="update-the-current-runtime-version"></a>Mettre à jour la version actuelle du runtime

À l’aide d’Azure CLI, vous pouvez mettre à jour le paramètre `runtimeVersion` dans l’application avec la commande [az webapp auth update](/cli/azure/webapp/auth?view=azure-cli-latest&preserve-view=true#az-webapp-auth-update).

```azurecli-interactive
az webapp auth update --name <my_app_name> \
--resource-group <my_resource_group> \
--runtime-version <version>
```

Remplacez `<my_app_name>` par le nom de votre application. Remplacez également `<my_resource_group>` par le nom du groupe de ressources de votre application. Par ailleurs, remplacez `<version>` par une version valide du runtime 1.x ou `~1` pour la version la plus récente. Vous trouverez les notes de publication sur les différentes versions du runtime [ici] (https://github.com/Azure/app-service-announcements) pour déterminer la version sur laquelle épingler.

Vous pouvez exécuter cette commande à partir de [Azure Cloud Shell](../cloud-shell/overview.md) en choisissant **Essayer** dans l’exemple de code qui précède. Vous pouvez également utiliser [Azure CLI en local](/cli/azure/install-azure-cli) pour exécuter cette commande après avoir lancé la commande [az login](/cli/azure/reference-index#az-login) pour vous connecter.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Authentifier et autoriser des utilisateurs de bout en bout](tutorial-auth-aad.md)
