---
title: Configurer l’authentification Azure AD
description: Découvrez comment configurer l’authentification Azure Active Directory pour en faire le fournisseur d’identité de votre application App Service ou Azure Functions.
ms.assetid: 6ec6a46c-bce4-47aa-b8a3-e133baef22eb
ms.topic: article
ms.date: 04/14/2020
ms.custom: seodec18, fasttrack-edit, has-adal-ref
ms.openlocfilehash: 2968fd84febdd3b98aa5d8b42cbf3fb66cad2036
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289799"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-azure-ad-login"></a>Configurer votre application App Service ou Azure Functions pour utiliser une connexion Azure AD

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

Cet article vous montre comment configurer Azure App Service ou Azure Functions pour utiliser Azure Active Directory (Azure AD) comme fournisseur d’authentification.

> [!NOTE]
> La configuration rapide configure une inscription d’application AAD V1. Si vous souhaitez utiliser [Azure Active Directory v2.0](../active-directory/develop/v2-overview.md) (notamment [MSAL](../active-directory/develop/msal-overview.md)), suivez les [instructions de configuration avancée](#advanced).

Adoptez ces bonnes pratiques pour configurer votre application et l’authentification :

- Donnez à chaque application App Service ses propres autorisations et son propre consentement.
- Configurez chaque application App Service avec sa propre inscription.
- Évitez de partager des autorisations entre des environnements en utilisant des inscriptions d’application distinctes pour des emplacements de déploiement distincts. Dans le cadre des tests d’un nouveau code, cette pratique peut permettre d’éviter que des problèmes n’affectent l’application de production.

> [!NOTE]
> Cette fonctionnalité n’est pas disponible actuellement dans le plan de consommation Linux pour Azure Functions

## <a name="configure-with-express-settings"></a><a name="express"> </a>Configurer avec des paramètres express

> [!NOTE]
> L’option **Express** n’est pas disponible pour les clouds du secteur publique.

1. Dans le [Azure portal], recherchez et sélectionnez **App Services** , puis sélectionnez votre application.
2. Dans la barre de navigation gauche, sélectionnez **Authentification/Autorisation**  > **sur**.
3. Sélectionnez **Azure Active Directory** > **Express**.

   Si vous préférez choisir une inscription d’application existante :

   1. Cliquez sur **Sélectionner une application AD existante** , puis sur **Azure AD App**.
   2. Sélectionnez une inscription d’application existante et cliquez sur **OK**.

3. Cliquez sur **OK** pour inscrire l'application App Service auprès d'Azure Active Directory. Une nouvelle inscription d’application est créée.

    ![Paramètres Express d'Azure Active Directory](./media/configure-authentication-provider-aad/express-settings.png)

4. (Facultatif) Par défaut, App Service fournit une authentification, mais ne restreint pas l’accès autorisé au contenu et aux API de votre site. Vous devez autoriser les utilisateurs dans votre code d'application. Pour restreindre l’accès à l’application aux seuls utilisateurs authentifiés par Azure Active Directory, définissez **Action à exécuter quand une requête n’est pas authentifiée** sur **Se connecter avec Azure Active Directory**. Quand vous définissez cette fonctionnalité, votre application exige que toutes les demandes soient authentifiées. Elle redirige également toutes les demandes non authentifiées vers Azure Active Directory à des fins d’authentification.

    > [!CAUTION]
    > Cette manière de restreindre l’accès s’applique à tous les appels à votre application qui peuvent ne pas être souhaitables pour les applications qui ont une page d’accès publique disponible, comme dans de nombreuses applications monopages. Pour de telles applications, préférez **Autoriser les requêtes anonymes (aucune action)** . L’application démarre alors elle-même manuellement la connexion. Pour plus d’informations, consultez [Flux d’authentification](overview-authentication-authorization.md#authentication-flow).
5. Sélectionnez **Enregistrer**.

## <a name="configure-with-advanced-settings"></a><a name="advanced"></a>Configurer avec des paramètres avancés

Vous pouvez configurer les paramètres de l’application manuellement si vous souhaitez utiliser une inscription d’application à partir d’un locataire Azure AD différent. Pour effectuer cette configuration personnalisée :

1. Créer une inscription dans Azure AD.
2. Fournir certains des détails d’inscription à App Service.

### <a name="create-an-app-registration-in-azure-ad-for-your-app-service-app"></a><a name="register"></a>Créer une inscription d’application dans Azure AD pour votre application App Service

Vous avez besoin des informations suivantes quand vous configurez votre application App Service :

- ID client
- ID client
- Clé secrète client (facultative)
- URI d’ID d’application

Procédez comme suit :

1. Connectez-vous au [Azure portal], recherchez et sélectionnez **App Services** , puis sélectionnez votre application. Notez l’ **URL** de votre application. Vous allez l’utiliser pour configurer l’inscription de votre application Azure Active Directory.
1. Sélectionnez **Azure Active Directory** > **Inscriptions d’applications** > **Nouvelle inscription**.
1. Sur la page **Inscrire une application** , entrez un **Nom** pour votre inscription d'application.
1. Sous **URI de redirection** , sélectionnez **Web** , puis entrez `<app-url>/.auth/login/aad/callback`. Par exemple : `https://contoso.azurewebsites.net/.auth/login/aad/callback`.
1. Sélectionnez **Create** (Créer).
1. Une fois l’inscription d’application créée, copiez l’ **ID de l’application (client)** et l’ **ID de l’annuaire (locataire)** pour plus tard.
1. Sélectionnez **Authentification**. Sous **Octroi implicite** , activez **Jetons d’ID** pour autoriser les connexions utilisateur OpenID Connect à partir d’App Service.
1. (Facultatif) Sélectionnez **Personnalisation**. Dans **URL de la page d’accueil** , entrez l’URL de votre application App Service, puis sélectionnez **Enregistrer**.
1. Sélectionnez **Exposer une API** > **Définir**. Pour une application monolocataire, collez l’URL de votre application App Service et sélectionnez **Enregistrer**. Pour une application multilocataire, collez l’URL, qui est basée sur l’un des domaines vérifiés par un locataire, puis sélectionnez **Enregistrer**.

   > [!NOTE]
   > Cette valeur correspond à l' **URI d'ID d'application** de votre inscription d'application. Si votre application web nécessite un accès à une API dans le cloud, vous avez besoin de l’ **URI d’ID d’application** de l’application web lorsque vous configurez la ressource App Service cloud. Vous pouvez utiliser cette valeur, par exemple, si vous souhaitez que le service cloud accorde explicitement l’accès à l’application web.

1. sélectionner **Ajouter une étendue**.
   1. Dans **Nom de l’étendue** , entrez *user_impersonation*.
   1. Dans les zones de texte, entrez le nom et la description de l’étendue de consentement que vous voulez que les utilisateurs voient dans la page de consentement. Par exemple, entrez *Accéder à mon application*.
   1. Sélectionnez **Ajouter une étendue**.
1. (Facultatif) Pour créer une clé secrète client, sélectionnez **Certificats et secrets** > **Nouvelle clé secrète client** > **Ajouter**. Copiez la valeur de la clé secrète client qui s'affiche sur la page. Elle ne s’affichera plus.
1. (Facultatif) Pour ajouter plusieurs **URL de réponse** , sélectionnez **Authentification**.

### <a name="enable-azure-active-directory-in-your-app-service-app"></a><a name="secrets"> </a>Activez Azure Active Directory dans votre Azure App Service

1. Dans le [Azure portal], recherchez et sélectionnez **App Services** , puis sélectionnez votre application.
1. Dans le volet gauche, sous **Paramètres** , sélectionnez **Authentification/Autorisation** > **sur**.
1. (Facultatif) Par défaut, l’authentification App Service autorise l’accès non authentifié à votre application. Pour appliquer l'authentification utilisateur, définissez **Mesure à prendre quand une requête n’est pas authentifiée** , sur **Se connecter avec Azure Active Directory**.
1. Sous **Fournisseurs d’authentification** , cliquez sur **Azure Active Directory**.
1. Dans **Mode d'administration** , sélectionnez **Avancé** et configurez l'authentification App Service selon le tableau suivant :

    |Champ|Description|
    |-|-|
    |ID client| Utilisez l' **ID d’application (client)** de l’inscription de l’application. |
    |URL de l’émetteur| Utilisez `<authentication-endpoint>/<tenant-id>/v2.0`, puis remplacez *\<authentication-endpoint>* par le [point de terminaison d’authentification de votre environnement cloud](../active-directory/develop/authentication-national-cloud.md#azure-ad-authentication-endpoints) (par exemple, « https://login.microsoftonline.com  » pour Azure global), puis *\<tenant-id>* par l’ **ID du répertoire (locataire)** dans lequel l’inscription de l’application a été créée. Cette valeur sert à rediriger les utilisateurs vers le bon locataire Azure AD, mais aussi à télécharger les métadonnées appropriées pour déterminer les clés de signature de jetons et la valeur de revendication de l’émetteur de jeton qui conviennent, par exemple. Pour les applications qui utilisent Azure AD v1 et pour les applications Azure Functions, omettez `/v2.0` dans l’URL.|
    |Clé secrète client (facultative)| Utilisez la clé secrète client que vous avez générée lors de l’inscription de l’application.|
    |Audiences de jeton autorisées| S’il s’agit d’une application cloud ou serveur et que vous souhaitez autoriser les jetons d’authentification d’une application web, ajoutez l’ **URI d’ID d’application** de l’application web ici. **L’ID client** configuré est *toujours* implicitement considéré comme une audience autorisée. |

2. Sélectionnez **OK** , puis cliquez sur **Enregistrer**.

Vous êtes maintenant prêt à utiliser Azure Active Directory à des fins d’authentification dans votre application App Service.

## <a name="configure-a-native-client-application"></a>Configurer une application cliente native

Vous pouvez inscrire des clients natifs pour permettre une authentification au niveau des API Web hébergées dans votre application en utilisant une bibliothèque de client comme la **Bibliothèque d’authentification Active Directory**.

1. Dans le [Azure portal], sélectionnez **Active Directory** > **Inscriptions d’applications** > **Nouvelle inscription**.
1. Sur la page **Inscrire une application** , entrez un **Nom** pour votre inscription d'application.
1. Dans **URI de redirection** , sélectionnez **Client public (mobile et bureau)** et entrez l’URL `<app-url>/.auth/login/aad/callback`. Par exemple : `https://contoso.azurewebsites.net/.auth/login/aad/callback`.

    > [!NOTE]
    > Pour une application Microsoft Store, utilisez plutôt le [SID de package](/previous-versions/azure/app-service-mobile/app-service-mobile-dotnet-how-to-use-client-library#package-sid) en guise d’URI.
1. Sélectionnez **Create** (Créer).
1. Une fois l’inscription d’application créée, copiez la valeur de l’ **ID d’application (client)** .
1. Sélectionnez **Autorisations des API** > **Ajouter une autorisation** > **Mes API**.
1. Sélectionnez l’inscription d'application que vous avez créée précédemment pour votre application App Service. Si celle-ci n’apparaît pas, vérifiez que vous avez ajouté l’étendue **user_impersonation** dans [Créer une inscription d’application dans Azure AD pour votre application App Service](#register).
1. Sous **Autorisations déléguées** , sélectionnez **user_impersonation** , puis **Ajouter des autorisations**.

Vous avez maintenant configuré une application cliente native qui peut accéder à votre application App Service au nom d’un utilisateur.

## <a name="configure-a-daemon-client-application-for-service-to-service-calls"></a>Configurer une application cliente de démon pour des appels de service à service

Votre application peut acquérir un jeton pour appeler une API web hébergée dans votre App Service ou application de fonction pour son propre compte (et non pour le compte d’un utilisateur). Ce scénario est utile pour les applications de démon non interactives qui effectuent des tâches sans utilisateur connecté. Il utilise l’octroi d’[informations d’identification du client](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md) OAuth 2.0 standard.

1. Dans le [Azure portal], sélectionnez **Active Directory** > **Inscriptions d’applications** > **Nouvelle inscription**.
1. Dans la page **Inscrire une application** , entrez un **Nom** pour votre inscription d’application.
1. Pour une application démon, vous n’avez pas besoin d’un URI de redirection. Vous pouvez donc conserver cette zone vide.
1. Sélectionnez **Create** (Créer).
1. Une fois l’inscription d’application créée, copiez la valeur de l’ **ID d’application (client)** .
1. Sélectionnez **Certificats & secrets** > **Nouvelle clé secrète client** > **Ajouter**. Copiez la valeur de la clé secrète client qui s'affiche sur la page. Elle ne s’affichera plus.

Vous pouvez maintenant [demander un jeton d’accès à l’aide de l’ID client et de la clé secrète client](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md#first-case-access-token-request-with-a-shared-secret) en définissant le paramètre `resource` sur l’ **URI d’ID d’application** de l’application cible. Le jeton d’accès obtenu peut ensuite être présenté à l’application cible à l’aide de l’[en-tête d’autorisation OAuth 2.0](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md#use-the-access-token-to-access-the-secured-resource) standard. La fonction Authentification/autorisation App Service valide et utilise le jeton comme d’habitude pour indiquer à présent que l’appelant (en l’occurrence, une application, pas un utilisateur) est authentifié.

À l’heure actuelle, cela permet à _n’importe quelle_ application cliente de votre locataire Azure AD de demander un jeton d’accès et de s’authentifier auprès de l’application cible. Si vous souhaitez également appliquer un _autorisation_ pour n’autoriser que certaines applications clientes, vous devez effectuer une configuration supplémentaire.

1. [Définissez un rôle d’application](../active-directory/develop/howto-add-app-roles-in-azure-ad-apps.md) dans le manifeste de l’inscription d’application représentant l’App Service ou l’application de fonction que vous souhaitez protéger.
1. Sur l’inscription de l’application représentant le client qui doit être autorisé, sélectionnez **Autorisations de l’API** > **Ajouter une autorisation** > **Mes API**.
1. Sélectionnez l’inscription d’application que vous avez créée précédemment. Si vous ne voyez pas l’inscription d’application, assurez-vous que vous avez [ajouté un rôle d’application](../active-directory/develop/howto-add-app-roles-in-azure-ad-apps.md).
1. Sous **Autorisations d’application** , choisissez le rôle d’application que vous avez créé précédemment, puis sélectionnez **Ajouter des autorisations**.
1. Veillez à cliquer sur **Accorder un consentement administrateur** pour autoriser l’application cliente à demander l’autorisation.
1. Comme pour le scénario précédent (avant l’ajout de rôles), vous pouvez maintenant [demander un jeton d’accès](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md#first-case-access-token-request-with-a-shared-secret) pour la même `resource` cible, et le jeton d’accès inclut une revendication `roles` contenant les rôles d’application qui ont été autorisés pour l’application cliente.
1. Dans le code de l’App Service ou de l’application de fonction cibles, vous pouvez désormais vérifier que les rôles attendus sont présents dans le jeton (cela n’est pas effectué par la fonction Authentification/autorisation App Service). Pour plus d’informations, consultez la section [Revendications d’utilisateurs d’accès](app-service-authentication-how-to.md#access-user-claims).

Vous avez maintenant configuré une application cliente démon qui peut accéder à votre application App Service en utilisant sa propre identité.

## <a name="next-steps"></a><a name="related-content"> </a>Étapes suivantes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]
* [Tutoriel : Authentifier et autoriser des utilisateurs de bout en bout dans Azure App Service](tutorial-auth-aad.md)
<!-- URLs. -->

[Azure portal]: https://portal.azure.com/
