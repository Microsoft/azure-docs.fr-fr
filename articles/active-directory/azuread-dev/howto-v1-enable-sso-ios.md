---
title: Activation de l’authentification unique entre applications sur iOS à l’aide de la bibliothèque ADAL | Microsoft Docs
description: En savoir plus sur l’utilisation des fonctionnalités de votre Kit de développement logiciel (SDK) ADAL pour activer l’authentification unique sur l’ensemble de vos applications.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: how-to
ms.date: 09/24/2018
ms.author: ryanwi
ms.reviewer: brandwe
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 08b018082c753b9524cb12a72d637fe5458d9114
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85383697"
---
# <a name="how-to-enable-cross-app-sso-on-ios-using-adal"></a>Procédure : Activer l’authentification unique entre applications sur iOS à l’aide de la bibliothèque ADAL

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

L’authentification unique (SSO) permet aux utilisateurs de n’entrer leurs informations d’identification qu’une seule fois. Ces informations d’identification s’appliquent alors automatiquement aux applications et aux plateformes que d’autres applications peuvent utiliser (par exemple, des comptes Microsoft ou un compte professionnel de Microsoft 365), quel que soit le serveur de publication.

La plateforme d’identité de Microsoft, avec les kits de développement logiciel (SDK), simplifie l’activation de l’authentification unique dans votre propre suite d’applications, ou avec la fonctionnalité de répartiteur et des applications Authenticator, sur l’ensemble de l’appareil.

Dans cette procédure, vous allez découvrir comment configurer le Kit de développement logiciel (SDK) dans votre application afin de fournir l’authentification unique à vos clients.

Cette procédure s’applique à :

* Azure Active Directory (Azure Active Directory)
* Azure Active Directory B2C
* Azure Active Directory B2B
* Accès conditionnel Azure Active Directory

## <a name="prerequisites"></a>Prérequis

Cette procédure suppose que vous savez comment :

* Configurer votre application à l’aide du portail hérité pour Azure AD. Pour plus d’informations, consultez [Inscrire une application](../develop/quickstart-register-app.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json)
* Intégrer votre application à [Kit de développement logiciel (SDK) iOS d’Azure AD](https://github.com/AzureAD/azure-activedirectory-library-for-objc).

## <a name="single-sign-on-concepts"></a>Concepts de l’authentification unique

### <a name="identity-brokers"></a>Répartiteurs d’identité

Sur l’ensemble des plateformes mobiles, Microsoft fournit des applications qui prennent en charge le portage des informations d’identification entre les applications de différents fournisseurs, ainsi que des fonctionnalités avancées nécessitant un emplacement unique sécurisé de validation des informations d’identification. Elles sont appelées des **répartiteurs**.

Sur iOS et Android, les répartiteurs sont fournis par le biais d’applications téléchargeables que les clients installent indépendamment ou font l’objet d’un push vers l’appareil par une entreprise qui gère certains ou la totalité des appareils de ses employés. Les répartiteurs gèrent la sécurité d’une partie des applications ou de l’intégralité de l’appareil, en fonction de la configuration de l’administrateur informatique. Dans Windows, cette fonctionnalité est fournie par un sélecteur de compte intégré au système d’exploitation, désigné techniquement sous l’appellation « Répartiteur d’authentification web ».

### <a name="patterns-for-logging-in-on-mobile-devices"></a>Modèles de connexion sur les appareils mobiles

L’accès aux informations d’identification sur les appareils respecte deux modèles de base :

* Connexions assistées sans répartiteur
* Connexions assistées avec répartiteur

#### <a name="non-broker-assisted-logins"></a>Connexions assistées sans répartiteur

Les connexions assistées sans répartiteur correspondent à des expériences de connexion intervenant en ligne avec l’application et utilisant le stockage local sur l’appareil pour cette application. Ce stockage peut être partagé entre les applications, mais les informations d’identification sont étroitement liées à l’application ou à la suite d’applications qui les utilisent. Vous avez probablement eu cette expérience sur de nombreuses applications mobiles, lorsque vous devez saisir un nom d’utilisateur et un mot de passe.

Ces connexions présentent les avantages suivants :

* L’expérience utilisateur existe intégralement au sein de l’application.
* Les informations d’identification peuvent être partagées entre les applications qui sont signées par le même certificat, ce qui constitue une expérience de connexion unique à votre suite d’applications.
* Le contrôle de connexion est fourni à l’application avant et après la connexion.

Ces connexions présentent les inconvénients suivants :

* L’utilisateur ne peut pas profiter de l’authentification unique sur l’ensemble des applications utilisant l’identité Microsoft, uniquement sur celles que votre application a configurées.
* Votre application ne peut pas être utilisée avec des fonctionnalités commerciales plus avancées, comme l’accès conditionnel ; elle ne peut pas non plus utiliser la suite de produits Intune.
* Votre application ne peut pas prendre en charge l’authentification par certificat des utilisateurs professionnels.

Voici une représentation de la manière dont les Kits de développement logiciel (SDK) fonctionnent avec le stockage partagé de vos applications, pour l’activation de l’authentification unique :

```
+------------+ +------------+  +-------------+
|            | |            |  |             |
|   App 1    | |   App 2    |  |   App 3     |
|            | |            |  |             |
|            | |            |  |             |
+------------+ +------------+  +-------------+
| ADAL SDK  |  |  ADAL SDK  |  |  ADAL SDK   |
+------------+-+------------+--+-------------+
|                                            |
|            App Shared Storage              |
+--------------------------------------------+
```

#### <a name="broker-assisted-logins"></a>Connexions assistées avec répartiteur

Les connexions assistées avec répartiteur sont des expériences de connexion se produisant au sein de l’application de répartiteur, qui utilisent le stockage et la sécurité de ce composant pour partager l’ensemble des applications sur l’appareil qui applique la plateforme d’identité. Vos applications s’appuient sur le répartiteur pour connecter les utilisateurs. Sur iOS et Android, ces répartiteurs sont fournis par le biais d’applications téléchargeables que les clients installent indépendamment ou peuvent être transmis (push) à l’appareil par une entreprise qui gère l’appareil de ses employés. Comme exemple de ce type d’application, citons Microsoft Authenticator sur iOS. Dans Windows, cette fonctionnalité est fournie par un sélecteur de compte intégré au système d’exploitation, désigné techniquement sous l’appellation « Répartiteur d’authentification web ».

L’expérience, qui varie en fonction des plateformes, peut parfois perturber les utilisateurs en cas de gestion inappropriée. Vous connaissez probablement davantage ce modèle si vous avez installé l’application Facebook et que vous utilisez Facebook Connect depuis une autre application. La plateforme d’identité utilise le même modèle.

Sur iOS, une animation de transition s’affiche. Votre application est transmise à l’arrière-plan, tandis que les applications Microsoft Authenticator sont mises en avant-plan, ce qui permet à l’utilisateur de choisir son compte de connexion.

Sur Android et Windows, le sélecteur de compte s’affiche dans la partie supérieure de l’application ; l’utilisateur est ainsi moins perturbé.

#### <a name="how-the-broker-gets-invoked"></a>Appel du répartiteur

Si un répartiteur compatible, tel que l’application Microsoft Authenticator, est installé sur l’appareil, les Kits de développement logiciel (SDK) effectuent automatiquement pour vous l’opération d’appel du répartiteur lorsqu’un utilisateur souhaite se connecter à l’aide d’un compte de la plateforme d’identité. Il peut s’agir d’un compte personnel Microsoft, d’un compte professionnel ou scolaire, ou d’un compte que vous fournissez et hébergez dans Microsoft Azure à l’aide de nos produits B2C et B2B.

#### <a name="how-we-ensure-the-application-is-valid"></a>Comment s’assurer que l’application est valide

Pour sécuriser les connexions assistées avec répartiteur, il est impératif de s’assurer de l’identité de l’application appelant le répartiteur. iOS et Android ne fournissent pas d’identificateurs uniques valides uniquement pour une application donnée. Ainsi, des applications malveillantes peuvent « usurper » l’identité d’une application légitime en utilisant son identificateur et recevoir les jetons destinés à l’application légitime. Pour être sûrs de toujours communiquer avec la bonne application au moment de l’exécution, nous demandons aux développeurs de fournir un URI de redirection (redirectURI) personnalisé lorsqu’ils inscrivent leur application auprès de Microsoft. Nous expliquons ci-dessous comment les développeurs doivent créer cet URI de redirection. Cet URI de redirection personnalisé contient l’ID d’offre groupée de l’application et est garanti comme étant propre à l’application par l’App Store. Lorsqu’une application appelle le répartiteur, celui-ci demande au système d’exploitation iOS de lui fournir l’ID d’offre groupée qui a appelé le répartiteur. Le répartiteur fournit cet l’ID d’offre groupée à Microsoft dans le cadre de l’appel à notre système d’identité. Si l’ID d’offre groupée de l’application ne correspond pas à l’ID d’offre groupée fourni par le développeur au moment de l’inscription, nous refusons l’accès aux jetons de la ressource dont l’application a fait la demande. Cette vérification permet de s’assurer que seule l’application inscrite par le développeur reçoit les jetons.

**Le développeur détermine si le kit de développement logiciel (SDK) appelle le répartiteur ou utilise un flux assisté sans répartiteur.** Toutefois, si le développeur choisit de ne pas avoir recours au flux assisté avec répartiteur, il ne peut pas utiliser les informations d’identification d’authentification unique déjà saisies par l’utilisateur sur l’appareil et il empêche toute utilisation de l’application avec des fonctions commerciales fournies par Microsoft à ses clients, comme l’accès conditionnel, les fonctionnalités de gestion Intune et l’authentification par certificat.

Ces connexions présentent les avantages suivants :

* L’utilisateur profite de l’authentification unique sur l’ensemble de ses applications, quel que soit le fournisseur.
* Votre application peut utiliser des fonctionnalités commerciales plus avancées, comme l’accès conditionnel ou la suite de produits Intune.
* Votre application peut prendre en charge l’authentification par certificat des utilisateurs professionnels.
* L’expérience de connexion est bien plus sécurisée, dans la mesure où les identités de l’application et de l’utilisateur sont vérifiées par l’application du répartiteur à l’aide d’algorithmes de sécurité et d’un chiffrement supplémentaires.

Ces connexions présentent les inconvénients suivants :

* Dans iOS, l’utilisateur est déplacé à l’extérieur de l’expérience applicative pendant la sélection des informations d’identification.
* Perte de la capacité de gérer l’expérience de connexion des clients au sein de l’application.

Voici une représentation de la manière dont les Kits de développement logiciel (SDK) fonctionnent avec les applications de répartiteur pour prendre en charge l’authentification unique :

```
+------------+ +------------+   +-------------+
|            | |            |   |             |
|   App 1    | |   App 2    |   |   Someone   |
|            | |            |   |    Else's   |
|            | |            |   |     App     |
+------------+ +------------+   +-------------+
| Azure SDK  | | Azure SDK  |   | Azure SDK   |
+-----+------+-+-----+------+-  +-------+-----+
      |              |                  |
      |       +------v------+           |
      |       |             |           |
      |       | Microsoft   |           |
      +-------> Broker      |^----------+
              | Application
              |             |
              +-------------+
              |             |
              |   Broker    |
              |   Storage   |
              |             |
              +-------------+
```

## <a name="enabling-cross-app-sso-using-adal"></a>Activation de l’authentification unique entre applications à l’aide de la bibliothèque ADAL

Ici, nous utilisons le Kit de développement logiciel ADAL iOS pour effectuer les opérations suivantes :

* Activer l’authentification unique assistée sans répartiteur pour votre suite d’applications
* Activer la prise en charge de l’authentification unique assistée avec répartiteur

### <a name="turning-on-sso-for-non-broker-assisted-sso"></a>Activation de l’authentification unique assistée sans répartiteur

Les Kits de développement logiciel (SDK) prennent en charge une grande partie de la complexité de l’authentification unique assistée sans répartiteur entre les applications. Cela inclut l’identification de l’utilisateur approprié dans le cache et la gestion d’une liste d’utilisateurs connectés dans laquelle effectuer votre recherche.

Pour activer l’authentification unique sur l’ensemble des applications que vous possédez, vous devez effectuer les opérations suivantes :

1. Vérifiez que l’ensemble de vos applications utilisent le même ID client ou ID d’application.
2. Vérifiez que l’ensemble de vos applications partagent le même certificat de signature fourni par Apple, de manière à ce que vous puissiez partager les trousseaux.
3. Demandez la même éligibilité de trousseau pour l’ensemble de vos applications.
4. Indiquez aux Kits de développement logiciel le trousseau partagé que vous souhaitez utiliser.

#### <a name="using-the-same-client-id--application-id-for-all-the-applications-in-your-suite-of-apps"></a>Utilisation du même ID client/ID d’application pour l’ensemble des applications de votre suite d’applications

Pour que votre plateforme d’identité puisse savoir que le partage des jetons est autorisé entre vos applications, elles doivent toutes partager le même ID client ou d’application. Il s’agit de l’identificateur unique qui vous a été fourni lorsque vous avez inscrit votre première application dans le portail.

Les URI redirigées vous permettent d’identifier différentes applications utilisant le même ID d’application sur un service Microsoft Identity. Chaque application peut posséder plusieurs URI de redirection inscrits sur le portail d’intégration. Chacune des applications de votre suite présente un URI de redirection propre. Voici une représentation possible :

URI de redirection App1 : `x-msauth-mytestiosapp://com.myapp.mytestapp`

URI de redirection App2 : `x-msauth-mytestiosapp://com.myapp.mytestapp2`

URI de redirection App3 : `x-msauth-mytestiosapp://com.myapp.mytestapp3`

....

Ces éléments sont imbriqués sous le même ID client/ID d’application et recherchés en fonction de l’URI de redirection que vous nous transmettez dans votre configuration du Kit de développement logiciel (SDK).

```
+-------------------+
|                   |
|  Client ID        |
+---------+---------+
          |
          |           +-----------------------------------+
          |           |  App 1 Redirect URI               |
          +----------^+                                   |
          |           +-----------------------------------+
          |
          |           +-----------------------------------+
          +----------^+  App 2 Redirect URI               |
          |           |                                   |
          |           +-----------------------------------+
          |
          +----------^+-----------------------------------+
                      |  App 3 Redirect URI               |
                      |                                   |
                      +-----------------------------------+

```

Le format de ces URI de redirection est expliqué plus bas. Vous pouvez utiliser l’URI de redirection de votre choix, sauf si vous souhaitez prendre en charge le répartiteur. Le cas échéant, la configuration est similaire à ce qui précède*

#### <a name="create-keychain-sharing-between-applications"></a>Créer le partage de trousseau entre les applications

L’activation du partage de trousseau n’entre pas dans le périmètre de ce document. Le sujet est abordé par Apple, dans le document [Adding Capabilities](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html) (Ajout de fonctionnalités). L’essentiel est que vous déterminiez le nom que vous souhaitez attribuer à votre trousseau et que vous ajoutiez cette fonctionnalité dans l’ensemble de vos applications.

Une fois que vous disposez des droits appropriés, un fichier nommé `entitlements.plist` doit s’afficher dans votre répertoire de projet. Le cas échéant, il présente un contenu de ce type :

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "https://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>keychain-access-groups</key>
    <array>
        <string>$(AppIdentifierPrefix)com.myapp.mytestapp</string>
        <string>$(AppIdentifierPrefix)com.myapp.mycache</string>
    </array>
</dict>
</plist>
```

Une fois que l’éligibilité au trousseau a été activée dans chacune de vos applications et que vous êtes prêt à utiliser l’authentification unique, communiquez votre trousseau au Kit de développement logiciel (SDK) d’identité en utilisant le paramètre suivant dans votre `ADAuthenticationSettings`, à l’aide du paramètre ci-dessous :

```
defaultKeychainSharingGroup=@"com.myapp.mycache";
```

> [!WARNING]
> Lorsque vous partagez un trousseau entre vos applications, chaque application peut supprimer les utilisateurs, ou au pire l’ensemble des jetons de votre application. Cela aura un impact particulièrement désastreux si vous possédez des applications qui s’appuient sur les jetons pour exécuter les tâches d’arrière-plan. Le partage du trousseau nécessite de votre part une précaution infinie avec l’ensemble des opérations de suppression effectuées dans les kits de développement logiciel d’identité.

Et voilà ! Le Kit de développement logiciel (SDK) partage désormais les informations d’identification dans l’ensemble de vos applications. La liste des utilisateurs sera également partagée entre toutes les instances d’application.

### <a name="turning-on-sso-for-broker-assisted-sso"></a>Activation de l’authentification unique assistée avec répartiteur

La capacité d’une application à utiliser un répartiteur installé sur l’appareil est **désactivée par défaut**. Pour utiliser votre application avec le répartiteur, vous devez effectuer une configuration supplémentaire et ajouter du code à votre application.

Voici la procédure à suivre :

1. Activez le mode répartiteur dans votre appel du code d’application vers le Kit de développement logiciel MS.
2. Établissez un nouvel URI de redirection à fournir à l’application et à votre inscription d’application.
3. Enregistrez un nouveau schéma d’URL.
4. Ajoutez une autorisation à votre fichier info.plist.

#### <a name="step-1-enable-broker-mode-in-your-application"></a>Étape 1 : Activer le mode répartiteur dans votre application

La capacité de votre application à utiliser le répartiteur est activée lorsque vous créez le contexte, ou la configuration initiale, de votre objet d’authentification. Pour ce faire, définissez votre type d’informations d’identification dans votre code :

```
/*! See the ADCredentialsType enumeration definition for details */
@propertyADCredentialsType credentialsType;
```
Le paramètre `AD_CREDENTIALS_AUTO` autorise le Kit de développement logiciel à essayer d’appeler le répartiteur, tandis que `AD_CREDENTIALS_EMBEDDED` l’empêche de le faire.

#### <a name="step-2-registering-a-url-scheme"></a>Étape 2 : Enregistrer un nouveau schéma d’URL

La plateforme d’identité utilise des URL pour appeler le répartiteur, avant de rendre le contrôle à votre application. Pour terminer cet aller-retour, vous devez disposer d’un schéma d’URL inscrit pour votre application et dont la plateforme d’identité a connaissance. Il peut s’agir d’un ajout à un autre schéma d’application précédemment inscrit avec votre application.

> [!WARNING]
> Nous vous recommandons de personnaliser au maximum le schéma d’URL, ceci pour réduire la probabilité qu’une autre application utilise le même schéma d’URL. Apple n’applique pas l’unicité des schémas d’URL inscrits dans le magasin d’applications.

Voici un exemple de l’affichage dans votre configuration de projet. Vous pouvez également définir en XCode :

```
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLName</key>
        <string>com.myapp.mytestapp</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>x-msauth-mytestiosapp</string>
        </array>
    </dict>
</array>
```

#### <a name="step-3-establish-a-new-redirect-uri-with-your-url-scheme"></a>Étape 3 : Établissez un nouvel URI de redirection avec votre schéma d’URL

Pour garantir que nous renvoyons toujours les jetons d’identification à l’application appropriée, nous devons nous assurer de respecter une procédure de rappel de votre application pouvant être vérifiée par votre système d’exploitation iOS. Le système d’exploitation iOS signale aux applications de répartiteur Microsoft l’ID d’offre de l’application appelante. Il ne peut pas être falsifié par une application non fiable. Par conséquent, nous tirons parti de cette donnée et de l’URI de notre application de répartiteur pour vérifier que les jetons sont renvoyés à l’application appropriée. Nous vous demandons d’établir cet URI de redirection unique dans votre application et de le définir en tant qu’URI de redirection dans notre portail des développeurs.

Votre URI de redirection doit présenter la forme appropriée suivante :

`<app-scheme>://<your.bundle.id>`

ex : *x-msauth-mytestiosapp://com.myapp.mytestapp*

Cet URI de direction doit être spécifié dans l’inscription de votre application avec le [portail Azure](https://portal.azure.com/). Pour plus d’informations sur l’inscription d’applications Azure AD, consultez [Intégration avec Azure Active Directory](../develop/active-directory-how-to-integrate.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

##### <a name="step-3a-add-a-redirect-uri-in-your-app-and-dev-portal-to-support-certificate-based-authentication"></a>Étape 3a : Ajouter un URI de redirection dans votre application et le portail de développement pour prendre en charge l’authentification par certificat

Pour assurer la prise en charge de l’authentification par certificat, vous devez inscrire un second « msauth » dans votre application et le [portail Azure](https://portal.azure.com/) afin de prendre en charge l’authentification par certificat, si vous souhaitez ajouter cette prise en charge dans votre application.

`msauth://code/<broker-redirect-uri-in-url-encoded-form>`

exemple : *msauth://code/x-msauth-mytestiosapp%3A%2F%2Fcom.myapp.mytestapp*

#### <a name="step-4-add-a-configuration-parameter-to-your-app"></a>Étape 4 : Ajouter un paramètre de configuration à votre application

La bibliothèque ADAL utilise –canOpenURL: pour vérifier si le répartiteur est installé sur l’appareil. Dans iOS 9 et versions ultérieures, Apple a verrouillé la liste des schémas dans lesquels une application peut lancer une requête. Il vous faudra ajouter msauth à la section LSApplicationQueriesSchemes de votre `info.plist file`.

```
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>msauth</string>
    </array>

```

### <a name="youve-configured-sso"></a>Vous avez configuré l’authentification unique !

Désormais, le Kit de développement logiciel (SDK) d’identité partage automatiquement les informations d’identification entre vos applications et appelle l’éventuel répartiteur existant sur l’appareil.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur le [Protocole SAML d’authentification unique](../develop/single-sign-on-saml-protocol.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json)
