---
title: Utiliser AppAuth dans une application iOS
titleSuffix: Azure AD B2C
description: Guide pratique pour créer une application iOS qui utilise AppAuth avec Azure Active Directory B2C pour gérer les identités utilisateur et authentifier les utilisateurs.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/30/2018
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 6064bd2c62922abea44508b8bf6cdfa3e7ecbc92
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953302"
---
# <a name="azure-ad-b2c-sign-in-using-an-ios-application"></a>Azure Active Directory B2C : Se connecter à l’aide d’une application iOS

La plateforme d’identité Microsoft utilise des normes ouvertes telles que OAuth2 et OpenID Connect. L’utilisation d’un protocole standard ouvert offre plusieurs options de développeur lors de la sélection d’une bibliothèque à intégrer à nos services. Nous proposons cette procédure pas à pas et d’autres similaires pour aider les développeurs à écrire des applications qui se connectent à la plateforme Microsoft Identity. La plupart des bibliothèques qui implémentent [la spécification RFC6749 OAuth2](https://tools.ietf.org/html/rfc6749) sont en mesure de se connecter à la plateforme Microsoft Identity.

> [!WARNING]
> Microsoft ne fournit pas de correctifs pour les bibliothèques tierces et ne les a pas vérifiées. Cet exemple utilise une bibliothèque tierce appelée AppAuth dont la compatibilité a été testée dans des scénarios de base avec Azure AD B2C. Les problèmes et les demandes de fonctionnalités doivent être soumis au projet open source de la bibliothèque. Pour plus d’informations, consultez [cet article](../active-directory/develop/reference-v2-libraries.md).
>
>

Si vous découvrez OAuth2 ou OpenID Connect, cet exemple de configuration n’est peut-être pas très parlant pour vous. Nous vous recommandons de consulter une brève [vue d’ensemble du protocole que nous avons décrit ici](protocols-overview.md).

## <a name="get-an-azure-ad-b2c-directory"></a>Obtention d'un répertoire Azure AD B2C
Avant de pouvoir utiliser Azure AD B2C, vous devez créer un répertoire ou un client. Un répertoire est un conteneur destiné à recevoir tous vos utilisateurs, applications, groupes et autres. Si vous n’en possédez pas déjà un, [créez un répertoire B2C](tutorial-create-tenant.md) avant de continuer.

## <a name="create-an-application"></a>Créer une application

Inscrivez ensuite une application dans votre locataire Azure AD B2C. Vous fournissez ainsi à Azure AD les informations nécessaires pour communiquer avec votre application en toute sécurité.

[!INCLUDE [active-directory-b2c-appreg-native](../../includes/active-directory-b2c-appreg-native.md)]

Enregistrez l’**ID d’application (client)** pour l’utiliser dans une étape ultérieure.

Enregistrez également votre URI de redirection personnalisé pour l’utiliser dans une étape ultérieure. Par exemple : `com.onmicrosoft.contosob2c.exampleapp://oauth/redirect`.

## <a name="create-your-user-flows"></a>Créer vos flux utilisateur
Dans Azure AD B2C, chaque expérience utilisateur est définie par un [flux utilisateur](user-flow-overview.md). Cette application offre une expérience unique en matière d'identité en combinant connexion et inscription. Lorsque vous créez le flux utilisateur, veillez à effectuer les actions suivantes :

* Sous **Sign-up attributes** (Attributs d’abonnement), sélectionnez l’attribut **Nom complet**.  Vous pouvez sélectionner d’autres attributs également.
* Sous **Revendications d’applications**, sélectionnez le **Nom d’affichage** et l’**ID d’objet de l’utilisateur** des revendications. Vous pouvez aussi sélectionner d’autres revendications.
* Copier le **nom** de chaque flux utilisateur après sa création. Lorsque vous enregistrez le flux utilisateur, son nom porte le préfixe `b2c_1_`.  Vous aurez besoin du nom du flux utilisateur.

Une fois vos flux d’utilisateurs créés, vous pouvez générer votre application.

## <a name="download-the-sample-code"></a>Télécharger l’exemple de code
Nous avons fourni un exemple fonctionnel qui utilise AppAuth avec Azure AD B2C [sur GitHub](https://github.com/Azure-Samples/active-directory-ios-native-appauth-b2c). Vous pouvez télécharger le code et l’exécuter. Pour utiliser votre propre client Azure AD B2C, suivez les instructions du fichier [README.md](https://github.com/Azure-Samples/active-directory-ios-native-appauth-b2c/blob/master/README.md).

Cet exemple a été créé en suivant les instructions du fichier README par le [projet iOS AppAuth sur GitHub](https://github.com/openid/AppAuth-iOS). Pour plus d’informations sur le fonctionnement de l’exemple et la bibliothèque, consultez le fichier README AppAuth sur GitHub.

## <a name="modifying-your-app-to-use-azure-ad-b2c-with-appauth"></a>Modification de votre application pour utiliser Azure AD B2C avec AppAuth

> [!NOTE]
> AppAuth prend en charge iOS 7 et versions ultérieures.  Cependant, pour prendre en charge les connexions sociales sur Google, SFSafariViewController qui requiert iOS 9 ou version ultérieure est nécessaire.
>

### <a name="configuration"></a>Configuration

Vous pouvez configurer la communication avec Azure AD B2C en spécifiant les URI du point de terminaison d’autorisation et du point de terminaison de jeton.  Pour générer ces URI, vous avez besoin des informations suivantes :
* ID client (par exemple, contoso.onmicrosoft.com)
* Nom du flux utilisateur (par exemple, B2C\_1\_SignUpIn)

L’URI du point de terminaison de jeton peut être généré en remplaçant l’\_ID client et le nom de la stratégie\_ dans l’URL suivante :

```objc
static NSString *const tokenEndpoint = @"https://<Tenant_name>.b2clogin.com/te/<Tenant_ID>/<Policy_Name>/oauth2/v2.0/token";
```

L’URI du point de terminaison d’autorisation peut être généré en remplaçant l’\_ID client et le nom de la stratégie\_ dans l’URL suivante :

```objc
static NSString *const authorizationEndpoint = @"https://<Tenant_name>.b2clogin.com/te/<Tenant_ID>/<Policy_Name>/oauth2/v2.0/authorize";
```

Exécutez le code suivant pour créer votre objet AuthorizationServiceConfiguration :

```objc
OIDServiceConfiguration *configuration =
    [[OIDServiceConfiguration alloc] initWithAuthorizationEndpoint:authorizationEndpoint tokenEndpoint:tokenEndpoint];
// now we are ready to perform the auth request...
```

### <a name="authorizing"></a>Autorisation

Après la configuration ou la récupération d’une configuration de service d’autorisation, une demande d’autorisation peut être créée. Pour créer la demande, vous avez besoin des informations suivantes :

* ID client (ID d’APPLICATION) que vous avez enregistré précédemment. Par exemple : `00000000-0000-0000-0000-000000000000`.
* URI de redirection personnalisé que vous avez enregistré précédemment. Par exemple : `com.onmicrosoft.contosob2c.exampleapp://oauth/redirect`.

Les deux éléments doivent avoir été enregistrés là où vous avez [inscrit votre application](#create-an-application).

```objc
OIDAuthorizationRequest *request =
    [[OIDAuthorizationRequest alloc] initWithConfiguration:configuration
                                                  clientId:kClientId
                                                    scopes:@[OIDScopeOpenID, OIDScopeProfile]
                                               redirectURL:[NSURL URLWithString:kRedirectUri]
                                              responseType:OIDResponseTypeCode
                                      additionalParameters:nil];

AppDelegate *appDelegate = (AppDelegate *)[UIApplication sharedApplication].delegate;
appDelegate.currentAuthorizationFlow =
    [OIDAuthState authStateByPresentingAuthorizationRequest:request
                                   presentingViewController:self
                                                   callback:^(OIDAuthState *_Nullable authState, NSError *_Nullable error) {
        if (authState) {
            NSLog(@"Got authorization tokens. Access token: %@", authState.lastTokenResponse.accessToken);
            [self setAuthState:authState];
        } else {
            NSLog(@"Authorization error: %@", [error localizedDescription]);
            [self setAuthState:nil];
        }
    }];
```

Pour configurer votre application pour gérer la redirection vers l’URI avec le schéma personnalisé, vous devez mettre à jour la liste de « schémas d’URL » dans votre Info.pList :
* Ouvrez Info.pList.
* Survolez une ligne comme « Bundle OS Type Code », puis cliquez sur le symbole \+.
* Renommez la nouvelle ligne « Types d’URL ».
* Cliquez sur la flèche à gauche de « Types d’URL » pour ouvrir l’arborescence.
* Cliquez sur la flèche à gauche de « Élément 0 » pour ouvrir l’arborescence.
* Renommez le premier élément sous l’élément 0 « Modèles d’URL ».
* Cliquez sur la flèche à gauche de « Modèles d’URL » pour ouvrir l’arborescence.
* Dans la colonne « Valeur », il y a un champ vide à gauche de « Élément 0 » sous « Modèles d’URL ».  Définissez la valeur sur le modèle unique de votre application.  La valeur doit correspondre au modèle utilisé dans redirectURL lors de la création de l’objet OIDAuthorizationRequest.  Dans l’exemple, le schéma « com.onmicrosoft.fabrikamb2c.exampleapp » est utilisé.

Reportez-vous au [guide d’AppAuth](https://openid.github.io/AppAuth-iOS/) pour le reste du processus. Si vous avez besoin de prendre rapidement en main une application, consultez [cet exemple](https://github.com/Azure-Samples/active-directory-ios-native-appauth-b2c). Suivez les étapes du fichier [README.md](https://github.com/Azure-Samples/active-directory-ios-native-appauth-b2c/blob/master/README.md) pour entrer votre propre configuration Azure AD B2C.

Nous sommes ouverts aux commentaires et suggestions ! Si vous avez des difficultés avec cet article, ou si vous avez des conseils pour améliorer ce contenu, faites-nous part de vos commentaires en bas de la page. En cas de demandes liées aux fonctionnalités, utilisez [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160596-b2c).