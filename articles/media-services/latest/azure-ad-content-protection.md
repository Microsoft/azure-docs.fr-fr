---
title: Protection de contenu de bout en bout avec Azure AD
description: Cet article explique comment protéger votre contenu avec Azure Media Services et Azure Active Directory.
services: media-services
documentationcenter: ''
author: willzhan
manager: femila
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-js
ms.openlocfilehash: 9415d66c49992bc31f773dec908a861f1126e714
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92427199"
---
# <a name="tutorial-end-to-end-content-protection-using-azure-ad"></a>Tutoriel : Protection de contenu de bout en bout avec Azure AD

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Avec ce tutoriel et l’exemple de lecteur fourni, vous pouvez configurer un sous-système de protection de contenu multimédia de bout en bout sur Azure Media Services (AMS) et Azure Active Directory (AAD) en vue de diffuser du contenu multimédia à l’aide de tous les systèmes de chiffrement DRM/AES-128, protocoles de streaming, codecs et formats de conteneurs pris en charge par AMS. L’exemple est suffisamment générique pour permettre un accès sécurisé à toute API REST protégée par OAuth 2 via le flux du code d’autorisation avec PKCE (le service de distribution de licences Azure Media Services fait partie de ces API). Il fonctionne également pour l’API Microsoft Graph et pour toute API REST personnalisée et sécurisée à l’aide du code d’autorisation OAuth 2. Il s’agit du document associé à l’[exemple de code](https://github.com/Azure-Samples/media-services-content-protection-azure-ad).

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
>
> * Connaître les exigences d’authentification
> * Comprendre le fonctionnement de l’application
> * Inscrire une application de ressource back-end
> * Inscrire une application cliente
> * Configurer la stratégie de clé de contenu du compte Media Services et les stratégies de streaming
> * Configurer l’application de lecteur

Si vous n’avez pas d’abonnement Azure Media Services, créez un [compte d’essai gratuit Azure](https://azure.microsoft.com/free/), puis un compte Media Services.

### <a name="duration"></a>Duration
Une fois que les prérequis technologiques seront prêts à être exécutés, ce tutoriel vous prendra environ deux heures.

## <a name="prerequisites"></a>Prérequis

Nous allons utiliser les versions et les concepts suivants. Nous vous recommandons donc de vous familiariser avec ceux-ci avant de suivre ce tutoriel.

### <a name="prerequisite-knowledge"></a>Connaissances requises

Même si ce n’est pas indispensable, nous vous recommandons de vous familiariser avec les concepts suivants avant de commencer ce tutoriel :

* La gestion des droits numériques (DRM)
* [Azure Media Services (AMS) v3](./media-services-overview.md)
* Les [stratégies de clé de contenu](content-key-policy-concept.md) AMS utilisant l’API AMS v3, le portail Azure ou l’[outil Azure Media Services Explorer (AMSE)](https://github.com/Azure/Azure-Media-Services-Explorer)
* Les points de terminaison Azure AD v2 sur la [plateforme d’identité Microsoft](../../active-directory/develop/index.yml)
* L’authentification cloud moderne comme [OAuth 2.0 et OpenID Connect](../../active-directory/develop/active-directory-v2-protocols.md)
  * Le [flux du code d’autorisation dans OAuth 2.0](../../active-directory/develop/v2-oauth2-auth-code-flow.md) et la raison pour laquelle PKCE est nécessaire
  * [Autorisations déléguées et autorisations d’application](../../active-directory/develop/developer-glossary.md#permissions)
* Le [jeton JWT](../../active-directory/develop/access-tokens.md), ses revendications et la substitution de la clé de signature (inclus dans l’exemple).

### <a name="prerequisite-code-and-installations"></a>Installations et code prérequis

* L’exemple de code. Téléchargez l’[exemple de code](https://github.com/Azure-Samples/media-services-content-protection-azure-ad).
* Une installation de Visual Studio Code. Téléchargez Visual Studio Code ici : [https://code.visualstudio.com/download](https://code.visualstudio.com/download).
* Une installation de Node.js. Téléchargez Node.js ici : [https://nodejs.org](https://nodejs.org). NPM est fourni avec l’installation.
* Un [abonnement Azure](https://azure.microsoft.com/free/).
* Un compte Azure Media Services (AMS).
* @azure/msal-browser v2.0, qui fait partie de la famille des SDK de la [bibliothèque MSAL](../../active-directory/develop/msal-overview.md) pour les différentes plateformes clientes
* La dernière version du [lecteur multimédia Azure](https://github.com/Azure-Samples/azure-media-player-samples) (inclus dans l’exemple).
* Des informations d’identification FPS Apple, si vous souhaitez inclure la gestion des droits numériques (DRM) FairPlay et le certificat d’application hébergé avec CORS, qui est accessible via JavaScript côté client.

> [!IMPORTANT]
> Ce tutoriel utilise .NET pour créer la restriction de stratégie de clé de contenu.  Si vous n’êtes pas développeur .NET, mais souhaitez essayer Node.js pour vous connecter à Azure Media Services, lisez [Se connecter à l’API Media Services v3 – Node.js](configure-connect-nodejs-howto.md). Il existe également un module Node.js permettant de gérer automatiquement la substitution de clé. Pour plus d’informations, reportez-vous au [module passport-ad](https://github.com/AzureAD/passport-azure-ad) de Node.js.

## <a name="consider-the-authentication-and-authorization-requirements"></a>Tenir compte des exigences d’authentification et d’autorisation

La conception du sous-système présente quelques difficultés. En effet, celui-ci comprend plusieurs parties mobiles et il implique des contraintes d’application cliente. En outre, il nécessite une substitution de la clé Azure AD qui a lieu toutes les six semaines.

L’application monopage qui est utilisée dans ce tutoriel prend en compte les difficultés engendrées par les exigences d’authentification et les restrictions qui suivent. Il utilise :

* Des points de terminaison Azure AD v2, car les points de terminaison de la plateforme de développement Azure AD v1 sont remplacés par les points de terminaison de la plateforme d’identité Microsoft v2.
* Le flux du code d’autorisation, car le flux d’octroi implicite OAuth 2 est désormais déprécié.
* Une application soumise aux contraintes suivantes :
    * Un client public ne peut pas masquer le secret client.  L’utilisation du flux de code d’autorisation seul exige le masquage du secret client. Pour cette raison, le flux du code d’autorisation avec PKCE est utilisé.
    * Une application basée sur un navigateur qui est soumise à un bac à sable (sandbox) de sécurité du navigateur (contrainte CORS/Preflight).
    * Une application basée sur un navigateur utilisant un code JavaScript moderne et soumise à des contraintes de sécurité JavaScript (accessibilité des en-têtes personnalisés, ID de corrélation).

## <a name="understand-the-subsystem-design"></a>Comprendre la conception du sous-système

La conception du sous-système est illustrée dans le diagramme suivant.  Le sous-système comprend trois couches :

* Une couche back-office (en noir) pour la configuration de la stratégie de clé de contenu et la publication de contenu pour le streaming
* Des points de terminaison publics (en bleu) accessibles aux joueurs/clients et qui fournissent l’authentification, l’autorisation, la licence DRM et le contenu chiffré
* Une application de lecteur (en bleu clair) qui intègre tous les composants et :
    * gère l’authentification utilisateur via Azure AD.
    * gère l’acquisition access_token à partir d’Azure AD.
    * reçoit le manifeste et le contenu chiffré provenant d’AMS/CDN.
    * acquiert une licence DRM à partir d’Azure Media Services.
    * gère le déchiffrement, le décodage et l’affichage du contenu.

![Capture d’écran montrant l’analyse des jetons JWT](media/aad-ams-content-protection/subsystem.svg)

Pour plus d’informations sur le sous-système, consultez [Concevoir un système de protection de contenu multi-DRM avec contrôle d’accès](./design-multi-drm-system-with-access-control.md).

## <a name="understand-the-single-page-app"></a>Comprendre l’application monopage

L’application de lecteur est une application monopage, qui a été développée dans Visual Studio Code à l’aide des éléments suivants :

* Node.js avec ES 6 JavaScript
* @azure/msal-browser 2.0 bêta
* Le SDK du lecteur multimédia Azure
* Le flux OAuth 2 pour les points de terminaison Azure AD v2 (plateforme d’identité Microsoft)

L’application de lecteur monopage exécute les actions suivantes :

* Authentification utilisateur pour les utilisateurs natifs du locataire et les utilisateurs invités provenant d’autres locataires AAD ou comptes MSA. Les utilisateurs peuvent choisir de se connecter via une fenêtre contextuelle du navigateur ou une redirection (pour les navigateurs qui n’autorisent pas les fenêtres contextuelles tels que Safari).
* Acquisition de `access_token` via le flux du code d’autorisation avec PKCE.
* Renouvellement de `access_token` (les jetons émis par AAD sont valides pendant 1 heure), pour lequel `refresh_token` est également acquis.
* Analyse des jetons JWT (`access_token` et `id_token`) pour le test ou l’inspection.
* Acquisition de licences DRM pour les trois systèmes DRM ou la clé de contenu AES-128.
* Streaming de contenu avec différentes combinaisons de systèmes DRM, protocoles de streaming et formats de conteneurs. Une chaîne au bon format est générée pour chaque combinaison.
* Déchiffrement, décodage et affichage.
* Appels d’API Microsoft Graph à des fins de résolution des problèmes. <!--See more details in the subsection Shortest path: testing my protected asset in my subscription with your hosted player app and underlying tenant. -->

Voici l’écran pour la connexion, l’acquisition de jetons, le renouvellement des jetons et l’affichage des jetons :

 ![Écran pour la connexion, l’acquisition de jetons, le renouvellement des jetons et l’affichage des jetons](media/aad-ams-content-protection/token-acquisition.png)

Voici l’écran d’analyse des jetons JWT (access_token ou id_token) :

![Capture d’écran montrant l’analyse des jetons JWT.](media/aad-ams-content-protection/parsing-jwt-tokens.png)

Voici l’écran permettant de tester du contenu protégé avec différentes combinaisons de systèmes DRM/AES, protocoles de streaming et formats de conteneurs :

![Capture d’écran montrant le test d’un contenu protégé avec différentes combinaisons de systèmes DRM ou AES comparés à des protocoles de streaming et à des formats de conteneurs](media/aad-ams-content-protection/testing-protected-content.png)
-->

<!-- You can see a hosted version of the sample at [https://aka.ms/ott](https://aka.ms/ott)-->

## <a name="choose-an-azure-ad-tenant"></a>Choisir un locataire Azure AD

> [!NOTE]
> À ce stade, il est supposé que vous vous êtes connecté au portail Azure et que vous disposez au moins d’un locataire Azure AD.

Choisissez un locataire Azure AD à utiliser pour l’exemple de bout en bout. Deux options s'offrent à vous :

* Utiliser un locataire Azure AD existant. Tout abonnement Azure doit avoir un locataire Azure AD. Toutefois, un locataire Azure AD peut être utilisé par plusieurs abonnements Azure.
* Utiliser un nouveau locataire Azure AD qui *n’est pas* utilisé par un abonnement Azure. Si vous choisissez d’utiliser un nouveau locataire, le compte de service multimédia et l’exemple d’application de lecteur doivent se trouver dans un abonnement Azure qui utilise un autre locataire Azure AD. Vous disposez ainsi d’une certaine flexibilité. Par exemple, vous pouvez utiliser votre propre locataire Azure AD, mais aussi le compte de service multimédia du client dans l’abonnement Azure du client.

## <a name="register-the-backend-resource-app"></a>Inscrire l’application de ressource back-end

1. Accédez au locataire Azure AD que vous avez choisi ou créé.
1. Dans le menu, sélectionnez **Azure Active Directory**.
1. Dans le menu, sélectionnez **Inscriptions d’applications**.
1. Cliquez sur **+ Nouvelle inscription**.
1. Nommez l’application *LicenseDeliveryResource2* (où 2 indique les points de terminaison AAD v2).
1. Sélectionnez **Comptes dans ce répertoire d’organisation uniquement ([ *nom de votre locataire* ] uniquement - Locataire unique)** . Si vous souhaitez activer l’accès à plusieurs locataires, sélectionnez l’une des autres options multilocataires.
1. L’ **URI de redirection** est facultatif et peut être modifié ultérieurement.
1. Cliquez sur **S'inscrire**. La vue Inscriptions d’applications s’affiche.
1. Dans le menu, sélectionnez **Manifeste**. La vue Manifeste s’affiche.
1. Remplacez la valeur `accessTokenAcceptedVersion` par *2* (sans guillemets).
1. Remplacez la valeur `groupMembershipClaims` par *"SecurityGroup"* (avec des guillemets).
1. Cliquez sur **Enregistrer**.
1. Dans le menu, sélectionnez **Exposer une API**. La vue Ajouter une étendue s’affiche (Azure fournit un URI d’ID d’application. Toutefois, si vous souhaitez le modifier, vous pouvez le faire dans le champ URI d’ID d’application).
1. Cliquez sur **Enregistrer et continuer**. La vue changera. Pour chacun des paramètres de la colonne Paramètre du tableau ci-dessous, entrez la valeur de la colonne Valeur, puis cliquez sur **Ajouter une étendue**.

| Paramètre | Valeur | Description |
| ------- | ----- | ----------- |
| Nom de l’étendue | *DRM.License.Delivery* | Nom de l’étendue qui s’affiche quand l’accès à cette API est demandé, et qui s’affiche dans les jetons d’accès lorsque l’étendue a été accordée à une application cliente. Celui-ci doit être unique au sein de l’application. Pour générer le nom, il est recommandé d’utiliser « resource.operation.constraint » comme modèle. |
| Qui peut donner son consentement ? | *Administrateurs et utilisateurs* | Détermine si les utilisateurs peuvent accepter cette étendue dans les annuaires où le consentement de l’utilisateur est activé. |
| Nom d’affichage du consentement administrateur | *Remise de licence DRM* | Nom de l’étendue dans l’écran de consentement lorsque les administrateurs acceptent cette étendue. |
| Description du consentement de l’administrateur** | *Étendue de la ressource back-end de distribution des licences DRM* | Description détaillée de l’étendue qui s’affiche lorsque les administrateurs du locataire développent une étendue dans l’écran de consentement. |
| Nom d’affichage du consentement de l’utilisateur | *DRM.License.Delivery* | Nom de l’étendue dans l’écran de consentement lorsque les utilisateurs acceptent cette étendue. |
| Description du consentement de l’utilisateur | *Étendue de la ressource back-end de distribution des licences DRM* | Il s’agit d’une description détaillée de l’étendue qui s’affiche lorsque les utilisateurs développent une étendue sur l’écran de consentement. |
| State | *Activé* | Détermine si cette étendue peut être demandée par les clients. Affectez-lui la valeur « Disabled » pour les étendues que vous ne souhaitez pas rendre visibles des clients. Seules les étendues désactivées peuvent être supprimées. En outre, nous vous recommandons d’attendre au moins une semaine après la désactivation d’une étendue pour la supprimer, afin d’être sûr qu’elle n’est plus utilisée par aucun client. |

## <a name="register-the-client-app"></a>Inscrire l’application cliente

1. Accédez au locataire Azure AD que vous avez choisi ou créé.
1. Dans le menu, sélectionnez **Azure Active Directory**.
1. Dans le menu, sélectionnez **Inscriptions d’applications**.
1. Cliquez sur **+ Nouvelle inscription**.
1. Attribuez un nom à l’application cliente, par exemple, *AMS AAD Content Protection*.
1. Sélectionnez **Comptes dans ce répertoire d’organisation uniquement ([ *nom de votre locataire* ] uniquement - Locataire unique)** . Si vous souhaitez activer l’accès à plusieurs locataires, sélectionnez l’une des autres options multilocataires.
1. L’ **URI de redirection** est facultatif et peut être modifié ultérieurement.
1. Cliquez sur **S'inscrire**.
1. Dans le menu, sélectionnez **Autorisations de l’API**.
1. Cliquez sur **+ Ajouter une autorisation**. La vue Demander des autorisations d’API s’ouvre alors.
1. Cliquez sur l’onglet **Mon API** , puis sélectionnez l’application *LicenseDeliveryResource2* que vous avez créée dans la dernière section.
1. Cliquez sur la flèche DRM, puis sélectionnez l’autorisation *DRM.License.Delivery*.
1. Cliquez sur **Ajouter des autorisations**. La vue Ajouter des autorisations se ferme alors.
1. Dans le menu, sélectionnez **Manifeste**. La vue Manifeste s’affiche.
1. Recherchez et ajoutez les paires de valeurs suivantes à l’attribut `replyUrlsWithType` :

   ```json
   "replyUrlsWithType": [
        {
            "url": "https://npmwebapp.azurewebsites.net/",
            "type": "SPA"
        },
        {
            "url": "http://localhost:3000/",
            "type": "SPA"
        }
    ],
   ```

    > [!NOTE]
    > À ce stade, vous ne disposez pas encore de l’URL de votre application de lecteur.  Si vous exécutez l’application à partir de votre serveur hôte localhost, vous ne pourrez utiliser que la paire de valeurs localhost. Une fois que vous avez déployé votre application de lecteur, vous pouvez y ajouter l’entrée avec l’URL déployée.  Si vous oubliez de le faire, un message d’erreur s’affichera dans la page de connexion Azure AD.

1. Cliquez sur **Enregistrer**.
1. Enfin, pour vérifier que votre configuration est correcte, sélectionnez **Authentification**.  La vue Authentification s’affiche. Votre application cliente sera listée comme une application monopage, l’URI de redirection sera listé et le type d’octroi sera Flux du code d’autorisation avec PKCE.

### <a name="set-up-the-media-services-account-content-key-policy-and-streaming-policies"></a>Configurer la stratégie de clé de contenu du compte Media Services et les stratégies de streaming

**Cette section part du principe que vous êtes un développeur .NET et que vous êtes familiarisé avec l’utilisation de l’API AMS v3.**

> [!NOTE]
> Désormais, il n’est plus possible d’utiliser le portail Azure pour la configuration de la stratégie de la clé du compte Media Services, car il ne prend pas en charge l’utilisation d’une clé de signature de jeton asymétrique avec OpenID-Config.  La configuration doit prendre en charge la substitution de clé Azure AD, car le jeton Azure AD émis est signé par une clé asymétrique et cette clé est substituée toutes les six semaines. Par conséquent, ce tutoriel utilise .NET et l’API AMS v3.

La configuration de la [stratégie de clé de contenu](content-key-policy-concept.md) et celle des [stratégies de streaming](streaming-policy-concept.md) pour DRM et AES-128 s’appliquent.  Remplacez `ContentKeyPolicyRestriction` dans la stratégie de clé de contenu.

Voici le code .NET permettant de créer la restriction de stratégie de clé de contenu.

```dotnetcli
ContentKeyPolicyRestriction objContentKeyPolicyRestriction;

//use Azure Active Directory OpenId discovery document, supporting key rollover
objContentKeyPolicyRestriction = new ContentKeyPolicyTokenRestriction()
    {
        OpenIdConnectDiscoveryDocument = ConfigAccessor.AppSettings["ida_AADOpenIdDiscoveryDocument"]
    };

string audience = ConfigAccessor.AppSettings["ida_audience"];
string issuer   = ConfigAccessor.AppSettings["ida_issuer"];

ContentKeyPolicyTokenRestriction objContentKeyPolicyTokenRestriction = (ContentKeyPolicyTokenRestriction)objContentKeyPolicyRestriction;
objContentKeyPolicyTokenRestriction.Audience             = audience;
objContentKeyPolicyTokenRestriction.Issuer               = issuer;
objContentKeyPolicyTokenRestriction.RestrictionTokenType = ContentKeyPolicyRestrictionTokenType.Jwt;

objContentKeyPolicyRestriction = objContentKeyPolicyTokenRestriction;

return objContentKeyPolicyRestriction;
```

Remplacez les valeurs `ida_AADOpenIdDiscoveryDocument`, `ida_audience` et `ida_issuer` dans le code ci-dessus. Pour trouver les valeurs de ces éléments dans le portail Azure :

1. Sélectionnez le locataire AAD que vous avez utilisé précédemment, cliquez sur **Inscriptions d’applications** dans le menu, puis cliquez sur le lien **Points de terminaison**.
1. Sélectionnez puis copiez la valeur du champ **Document de métadonnées OpenIdConnect** dans le code en tant que valeur `ida_AADOpenIdDiscoveryDocument`.
1. La valeur `ida_audience` est l’ID de l’application (client) de l’application inscrite *LicenseDeliveryResource2*.
1. La valeur `ida_issuer` est l’URL `https://login.microsoftonline.com/[tenant_id]/v2.0`. Remplacez [ *tenant_id* ] par l’ID de votre locataire.

## <a name="set-up-the-sample-player-app"></a>Configurer l’exemple d’application de lecteur

Si vous ne l’avez pas déjà fait, clonez ou téléchargez l’application à partir du dépôt source : [https://github.com/Azure-Samples/media-services-content-protection-azure-ad](https://github.com/Azure-Samples/media-services-content-protection-azure-ad).

Vous disposez de deux options pour configurer l’application de lecteur :

* Personnalisation minimale (en remplaçant uniquement certaines valeurs de chaînes constantes telles que client_id, tenant_id et l’URL de streaming). Toutefois, vous devez utiliser Visual Studio Code et Node.js.
* Si vous préférez utiliser un autre IDE et une autre plateforme web comme ASP.NET Core, vous pouvez placer les fichiers de pages web, les fichiers JavaScript et le fichier CSS dans votre projet, puisque l’application de lecteur proprement dite n’utilise pas de code côté serveur.

### <a name="option-1"></a>Option 1 :

1. Démarrez Visual Studio Code.
1. Pour ouvrir le projet, cliquez sur Fichier -> Ouvrir le dossier -> puis recherchez et sélectionnez le dossier parent du fichier *package.json*.
1. Ouvrez le fichier JavaScript *public/javascript/constants.js*.
1. Remplacez `OAUTH2_CONST.CLIENT_ID` par le `client_id` de votre application cliente inscrite dans le locataire AAD.  Vous trouverez le `client_id` dans la section Vue d’ensemble de l’application inscrite dans le portail Azure. Remarque : il s’agit de l’ID client, et non de l’ID d’objet.
1. Remplacez `OAUTH2_CONST.TENANT_ID` par le `tenant_id` de votre locataire Azure AD. Pour trouver votre `tenant_id`, vous pouvez cliquer sur Azure Active Directory. Le tenant_id s’affichera dans la section Vue d’ensemble.
1. Remplacez `OAUTH2_CONST.SCOPE` par l’étendue que vous avez ajoutée à votre application cliente inscrite. Vous pouvez trouver l’étendue en accédant à l’application cliente inscrite à partir du menu **Inscriptions d’applications** , puis en sélectionnant votre application cliente :
    1. Sélectionnez votre application cliente, cliquez sur le menu **Autorisations de l’API** , puis sélectionnez l’étendue *DRM.License.Delivery* sous l’autorisation d’API *LicenseDeliveryResource2*. L’autorisation doit être au format suivant : *api://df4ed433-dbf0-4da6-b328-e1fe05786db5/DRM.License.Delivery*. **Important !** Conservez l’espace devant `offline_access` dans `OAUTH2_CONST.SCOPE`.
1. Remplacez les deux chaînes constantes de `AMS_CONST`, comme indiqué ci-dessous. L’une est l’URL de streaming protégée de votre ressource de test, tandis que l’autre est l’URL du certificat d’application FPS permettant d’inclure le cas de test FairPlay. Sinon, vous pouvez conserver les valeurs de `AMS_CONST.APP_CERT_URL`. Ensuite, cliquez sur **Enregistrer**.

```javascript
//defaults in ams.js
class AMS_CONST {
    static get DEFAULT_URL() {
        return "https://eventgridmediaservice-usw22.streaming.media.azure.net/9591e337-ae90-420e-be30-1da36c06665b/MicrosoftElite01.ism/manifest(format=mpd-time-csf,encryption=cenc)";
    }
    //FairPlay application cert URL
    static get APP_CERT_URL() {
     return `${window.location.href}cert/FPSAC.cer`;
    }
}
```

Pour effectuer des tests localement :

1. Dans Visual Studio Code (VSC), sélectionnez **Afficher** dans le menu principal, puis **Terminal**.
1. Si vous n’avez pas encore installé npm, à l’invite de commandes, entrez `npm install`.
1. À l’invite de commandes, entrez `npm start` (si npm ne démarre pas, essayez de modifier l’annuaire par `npmweb` en entrant `cd npmweb` à l’invite de commandes).
1. Utilisez un navigateur pour accéder à `http://localhost:3000`.

En fonction du navigateur que vous utilisez, choisissez une combinaison adaptée de système DRM/AES, protocole de streaming et format de conteneur afin d’effectuer des tests après la connexion (acquisition de `access_token`). Si vous effectuez des tests dans Safari sur un système macOS, vous devez cocher l’option API de redirection, car Safari n’autorise pas les fenêtres contextuelles. La plupart des autres navigateurs autorisent à la fois les fenêtres contextuelles et les redirections.

### <a name="option-2"></a>Option 2 :

Si vous envisagez d’utiliser une autre plateforme IDE/Web et/ou un serveur web comme IIS sur votre ordinateur de développement, copiez les fichiers suivants dans un nouveau répertoire de votre serveur web local. Les chemins ci-dessous indiquent l’endroit où ils se trouvent dans le code que vous avez téléchargé.

* *views/index.ejs* (remplacez le suffixe par .html)
* *views/jwt.ejs* (remplacez le suffixe par .html)
* *views/info.ejs* (remplacez le suffixe par html)
* *public/* * (fichiers JavaScript, CSS, images, comme indiqué ci-dessous)

1. Copiez les fichiers figurant dans le dossier *view* situé à la racine du nouveau répertoire.
1. Copiez les *dossiers* qui se trouvent dans le dossier *public* situé à la racine du nouveau répertoire.
1. Remplacez les extensions des fichiers `.ejs` par `.html` (aucune variable côté serveur n’est utilisée pour que vous puissiez la remplacer sans prendre de risques).
1. Ouvrez *index.html* dans VSC (ou dans un autre éditeur de code), puis modifiez les chemins `<script>` et `<link>` afin qu’ils reflètent l’emplacement des fichiers.  Si vous avez suivi les étapes précédentes, il vous suffit de supprimer le `\` dans le chemin.  Par exemple, `<script type="text/javascript" src="/javascript/constants.js"></script>` devient `<script type="text/javascript" src="javascript/constants.js"></script>`.
1. Personnalisez les constantes dans le fichier *javascript/constants.js* , comme dans l’option 1.

## <a name="common-customer-scenarios"></a>Scénarios clients courants

Maintenant que vous avez suivi ce tutoriel et que vous disposez d’un sous-système opérationnel, vous pouvez essayer de le modifier pour les scénarios clients suivants :

### <a name="azure-role-based-access-control-azure-rbac-for-license-delivery-via-azure-ad-group-membership"></a>Contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour la distribution des licences via l’appartenance à un groupe Azure AD

Jusqu’à présent, le système autorise tout utilisateur pouvant se connecter à obtenir une licence valide et à lire le contenu protégé.

Il est courant qu’un sous-ensemble d’utilisateurs authentifiés soient autorisés à regarder du contenu, tandis que d’autres ne le sont pas. C’est le cas, par exemple, quand un client propose à la fois des abonnements de base et des abonnements Premium pour son contenu vidéo. Les clients qui ont un abonnement de base ne peuvent pas regarder du contenu nécessitant un abonnement Premium. Voici les étapes supplémentaires que vous devez effectuer pour répondre à cette exigence :

#### <a name="set-up-the-azure-ad-tenant"></a>Configurer le locataire Azure AD

1. Configurez deux comptes dans votre locataire. Vous pourriez les nommer *premium_user* et *basic_user*.
1. Créez un groupe d’utilisateurs et appelez-le *PremiumGroup*.
1. Ajoutez *premium_user* à *PremiumGroup* en tant que membre, mais n’ajoutez pas *basic_user* au groupe.
1. Prenez note de l’ **ID d’objet** de *PremiumGroup*.

#### <a name="set-up-the-media-services-account"></a>Configurer le compte Media Services

Modifiez `ContentKeyPolicyRestriction` (comme indiqué dans la section ci-dessus) en ajoutant une revendication nommée *groups* , où `ida_EntitledGroupObjectId` a comme valeur l’ID d’objet *PremiumGroup*  :

```dotnetcli

var tokenClaims = new ContentKeyPolicyTokenClaim[] { new ContentKeyPolicyTokenClaim("groups", ConfigAccessor.AppSettings["ida_EntitledGroupObjectId"])
//, other claims, if any.
};

if (tokenClaims != null && tokenClaims.Length > 0)
{
     objContentKeyPolicyTokenRestriction.RequiredClaims = new List<ContentKeyPolicyTokenClaim>(tokenClaims);
}
```

La revendication *groups* est membre d’un [ensemble de revendications restreint](../../active-directory/develop/active-directory-claims-mapping.md#claim-sets) dans Azure AD.

#### <a name="test"></a>Test

1. Connectez-vous avec le compte *premium_user*. Vous devez être en mesure de lire le contenu protégé.
1. Connectez-vous avec le compte *basic_user*. Vous devez recevoir une erreur indiquant que la vidéo est chiffrée, mais qu’il n’existe pas de clé pour la déchiffrer. Si vous affichez les événements, les erreurs et les téléchargements avec la liste déroulante située en bas de l’affichage des diagnostics concernant le lecteur, le message d’erreur doit indiquer un échec d’acquisition de licence en raison d’une valeur manquante pour la revendication de groupes dans le JWT émis par le point de terminaison de jeton Azure AD.

### <a name="supporting-multiple-media-service-accounts-across-multiple-subscriptions"></a>Prise en charge de plusieurs comptes de service multimédia (dans plusieurs abonnements)

Un client peut avoir plusieurs comptes de service multimédia dans un ou plusieurs abonnements Azure. Par exemple, un client peut avoir un compte de service multimédia comme serveur de production principal, un autre comme serveur secondaire de sauvegarde, et un autre pour le développement et les tests.

Tout ce que vous avez à faire est de vous assurer que vous utilisez le même jeu de paramètres que celui que vous avez utilisé dans la section Configurer le compte Media Services lors de la création de `ContentKeyPolicyRestriction` dans tous les comptes de service multimédia.

### <a name="supporting-a-customer-its-vendors-andor-subsidiaries-across-multiple-aad-tenants"></a>Prise en charge d’un client, de ses fournisseurs et/ou de ses filiales au sein de plusieurs locataires AAD

En tant qu’utilisateurs de la solution, les filiales, fournisseurs et partenaires d’un client peuvent résider dans différents locataires AAD, comme `mycustomer.com`, `mysubsidiary.com` et `myparther.com`. Même si cette solution s’appuie sur un seul locataire AAD, tel que `mycustomer.com`, vous pouvez l’utiliser avec les utilisateurs d’autres locataires.

En utilisant `mycustomer.com` pour cette solution, ajoutez un utilisateur issu de `mypartner.com` en tant qu’utilisateur invité dans le locataire `mycustomer.com`. Vérifiez que l’utilisateur `mypartner.com` a activé le compte invité. Le compte invité peut être un autre locataire AAD ou un compte `outlook.com`.

Notez que les utilisateurs invités de `mypartner.com`, après avoir été activés dans `mycustomer.com`, sont toujours authentifiés par le biais de leur propre locataire AAD ou du locataire AAD d’origine (`mypartner.com`). Toutefois, `access_token` est émis par `mycustomer.com`.

### <a name="supporting-a-customer-tenantsubscription-with-a-setup-in-your-subscriptiontenant"></a>Prise en charge d’un locataire/abonnement client dans votre abonnement/locataire

Vous pouvez utiliser votre configuration pour tester le contenu protégé dans le compte de service multimédia ou dans l’abonnement de votre client. Vous devez le configurer avec un locataire Azure AD et un compte de service multimédia dans un même abonnement. Le compte de service multimédia du client se trouverait dans son abonnement Azure avec son propre locataire Azure AD.

1. Ajoutez le compte de votre client à votre locataire en tant que compte invité.
1. Collaborez avec votre client en vue de préparer du contenu protégé dans son compte de service multimédia, en fournissant les trois paramètres listés dans la section Configurer le compte Media Services.

Votre client pourra alors accéder à votre configuration, se connecter avec le compte invité et tester son propre contenu protégé. Vous pouvez également vous connecter avec votre propre compte et tester le contenu de votre client.

Votre exemple de solution peut être configuré dans un locataire Microsoft avec un abonnement Microsoft ou dans un locataire personnalisé avec un abonnement Microsoft. L’instance Azure Media Services peut provenir d’un autre abonnement appartenant à son locataire.

## <a name="clean-up-resources"></a>Nettoyer les ressources

> [!WARNING]
> Si vous ne pensez pas continuer à utiliser cette application, supprimez les ressources que vous avez créées dans le cadre de ce tutoriel. Si vous ne le faites pas, les ressources vous seront facturées.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Démarrage rapide : Chiffrer du contenu](encrypt-content-quickstart.md)
