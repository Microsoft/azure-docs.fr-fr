---
title: 'Démarrage rapide : Ajouter la connexion Microsoft sur une application web Java | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce guide de démarrage rapide, apprenez à implémenter la connexion Microsoft dans une application web Java avec OpenID Connect.
services: active-directory
author: sangonzal
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/09/2019
ms.author: sagonzal
ms.custom: aaddev, scenarios:getting-started, languages:Java, devx-track-java
ms.openlocfilehash: e188c00840a4d043e94f94f9db565e2d4e06aaba
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031060"
---
# <a name="quickstart-add-sign-in-with-microsoft-to-a-java-web-app"></a>Démarrage rapide : Ajouter la connexion avec Microsoft à une application web Java

Dans ce guide de démarrage rapide, vous téléchargez et exécutez un exemple de code qui montre comment une application web Java peut connecter des utilisateurs et appeler l’API Microsoft Graph. Les utilisateurs de n’importe quelle organisation Azure Active Directory (Azure AD) peuvent se connecter à l’application.

 Consultez [Fonctionnement de l’exemple](#how-the-sample-works) pour obtenir une illustration.

## <a name="prerequisites"></a>Prérequis

Pour exécuter cet exemple, vous avez besoin des éléments suivants :

- [Kit de développement Java (JDK)](https://openjdk.java.net/) version 8 ou ultérieure, et [Maven](https://maven.apache.org/).

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-app"></a>Inscrire et télécharger votre application de démarrage rapide
> Vous avez deux options pour démarrer votre application du guide de démarrage rapide : express (Option 1) ou manuel (Option 2)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Option 1 : Inscrire et configurer automatiquement votre application, puis télécharger votre exemple de code
>
> 1. Accédez à l’expérience de démarrage rapide [Portail Azure - Inscriptions d’applications](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/JavaQuickstartPage/sourceType/docs).
> 1. Entrez un nom pour votre application, puis sélectionnez **Inscrire**.
> 1. Suivez les instructions de l’expérience de démarrage rapide du portail pour télécharger le code d’application configuré automatiquement.
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>Option n°2 : Inscrire et configurer manuellement vos application et exemple de code
>
> #### <a name="step-1-register-your-application"></a>Étape 1 : Inscrivez votre application
>
> Pour inscrire votre application et ajouter manuellement les informations d’inscription de l’application à votre application, suivez ces étapes :
>
> 1. Connectez-vous au [portail Azure](https://portal.azure.com).
> 1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
> 1. Recherchez et sélectionnez **Azure Active Directory**.
> 1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
> 1. Entrez un **nom** pour votre application (par exemple, `java-webapp`). Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
> 1. Sélectionnez **Inscription**.
> 1. Dans la page **Vue d’ensemble**, notez les valeurs **ID d’application (client)** et **ID d’annuaire (locataire)** pour une utilisation ultérieure.
> 1. Sous **Gérer**, sélectionnez **Authentification**.
> 1. Sélectionnez **Ajouter une plateforme** > **Web**.
> 1. Dans la section **URI de redirection**, ajoutez `https://localhost:8443/msal4jsample/secure/aad`.
> 1. Sélectionnez **Configurer**.
> 1. Dans la section **Web**, ajoutez `https://localhost:8443/msal4jsample/graph/me` comme seconde **URI de redirection**.
> 1. Sous **Gérer**, sélectionnez **Certificats et secrets**. Dans la section **Secrets client**, sélectionnez **Nouveau secret client**.
> 1. Tapez une description de clé (par exemple, un secret d’application), laissez l’expiration par défaut et sélectionnez **Ajouter**.
> 1. Notez la **Valeur** de la **clé secrète client** pour une utilisation ultérieure.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Étape 1 : Configurer votre application dans le portail Azure
>
> Pour que l’exemple de code de ce guide de démarrage rapide fonctionne, vous devez :
>
> 1. Ajouter des URL de réponse comme `https://localhost:8443/msal4jsample/secure/aad` et `https://localhost:8443/msal4jsample/graph/me`
> 1. Créer un secret client.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Apporter ces modifications pour moi]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Déjà configuré](media/quickstart-v2-aspnet-webapp/green-check.png) Votre application est configurée avec ces attributs.

#### <a name="step-2-download-the-code-sample"></a>Étape 2 : Télécharger l’exemple de code
> [!div renderon="docs"]
> [Télécharger l'exemple de code](https://github.com/Azure-Samples/ms-identity-java-webapp/archive/master.zip)

> [!div class="sxs-lookup" renderon="portal"]
> Téléchargez le projet et extrayez le fichier zip dans un dossier local proche du dossier racine (par exemple, **C:\Azure-Samples**)
>
> Pour utiliser https avec localhost, complétez les propriétés server.ssl.key. Pour générer un certificat auto-signé, utilisez l’utilitaire keytool (inclus dans JRE).
>
>  ```
>   Example:
>   keytool -genkeypair -alias testCert -keyalg RSA -storetype PKCS12 -keystore keystore.p12 -storepass password
>
>   server.ssl.key-store-type=PKCS12
>   server.ssl.key-store=classpath:keystore.p12
>   server.ssl.key-store-password=password
>   server.ssl.key-alias=testCert
>   ```
>   Placez le fichier keystore généré dans le dossier « resources ».

> [!div renderon="portal" id="autoupdate" class="sxs-lookup nextstepaction"]
> [Téléchargez l’exemple de code](https://github.com/Azure-Samples/ms-identity-java-webapp/archive/master.zip).

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
> #### <a name="step-3-configure-the-code-sample"></a>Étape 3 : Configurer l’exemple de code
> 1. Extrayez le fichier zip dans un dossier local.
> 1. Si vous utilisez un environnement de développement intégré, ouvrez l’exemple dans votre IDE habituel (facultatif).
> 1. Ouvrez le fichier application.properties, qui se trouve dans le dossier src/main/resources/ et remplacez la valeur des champs *aad.clientId*, *aad.authority* et *aad.secretKey* par les valeurs respectives de **ID d’application**, **ID de locataire** et **Secret client** comme suit :
>
>    ```file
>    aad.clientId=Enter_the_Application_Id_here
>    aad.authority=https://login.microsoftonline.com/Enter_the_Tenant_Info_Here/
>    aad.secretKey=Enter_the_Client_Secret_Here
>    aad.redirectUriSignin=https://localhost:8443/msal4jsample/secure/aad
>    aad.redirectUriGraph=https://localhost:8443/msal4jsample/graph/me
>    aad.msGraphEndpointHost="https://graph.microsoft.com/"
>    ```
> Où :
>
> - `Enter_the_Application_Id_here` - est l’ID de l’application pour l’application que vous avez inscrite.
> - `Enter_the_Client_Secret_Here` - correspond au **secret du client** que vous avez créé dans **Certificats et secrets** pour l’application que vous avez inscrite.
> - `Enter_the_Tenant_Info_Here` – est la valeur **ID de l’annuaire (locataire)** de l’application que vous avez inscrite.
> 1. Pour utiliser https avec localhost, complétez les propriétés server.ssl.key. Pour générer un certificat auto-signé, utilisez l’utilitaire keytool (inclus dans JRE).
>
>  ```
>   Example:
>   keytool -genkeypair -alias testCert -keyalg RSA -storetype PKCS12 -keystore keystore.p12 -storepass password
>
>   server.ssl.key-store-type=PKCS12
>   server.ssl.key-store=classpath:keystore.p12
>   server.ssl.key-store-password=password
>   server.ssl.key-alias=testCert
>   ```
>   Placez le fichier keystore généré dans le dossier « resources ».


> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-run-the-code-sample"></a>Étape 3 : Exécuter l’exemple de code
> [!div renderon="docs"]
> #### <a name="step-4-run-the-code-sample"></a>Étape 4 : Exécuter l’exemple de code

Pour exécuter le projet, vous pouvez :

L’exécuter directement à partir de votre IDE en utilisant le serveur Spring Boot incorporé ou bien, l’empaqueter dans un fichier WAR à l’aide de [maven](https://maven.apache.org/plugins/maven-war-plugin/usage.html) et le déployer sur une solution de conteneur J2EE comme [Apache Tomcat](http://tomcat.apache.org/).

##### <a name="running-from-ide"></a>Exécuter à partir d’un IDE

Si vous exécutez l’application web à partir d’un environnement de développement intégré (IDE), sélectionnez Exécuter, puis accédez à la page d’accueil du projet. Pour cet exemple, l’URL de la page d’accueil standard est https://localhost:8443.

1. Sur la première page, sélectionnez le bouton **Connexion** pour rediriger vers Azure Active Directory et inviter l’utilisateur à entrer ses informations d’identification.

1. Une fois l’utilisateur authentifié, il est redirigé vers *https://localhost:8443/msal4jsample/secure/aad* . Ils sont désormais connectés et la page affiche des informations sur le compte connecté. L’exemple d’interface utilisateur comporte les boutons suivants :
    - *Se déconnecter* : déconnecte l’utilisateur actuel de l’application et le redirige vers la page d’accueil.
    - *Afficher les infos sur l’utilisateur* : Acquiert un jeton pour Microsoft Graph et appelle Microsoft Graph avec une requête contenant le jeton, qui retourne des informations de base sur l’utilisateur connecté.

##### <a name="running-from-tomcat"></a>Exécuter à partir de Tomcat

Si vous souhaitez déployer l’exemple web sur Tomcat, vous devrez apporter quelques modifications au code source.

1. Ouvrez ms-identity-java-webapp/pom.xml
    - Sous `<name>msal-web-sample</name>` ajoutez `<packaging>war</packaging>`

2. Ouvrez ms-identity-java-webapp/src/main/java/com.microsoft.azure.msalwebsample/MsalWebSampleApplication

    - Supprimez tout le code source et remplacez par ce qui suit :

   ```Java
    package com.microsoft.azure.msalwebsample;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.builder.SpringApplicationBuilder;
    import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

    @SpringBootApplication
    public class MsalWebSampleApplication extends SpringBootServletInitializer {

     public static void main(String[] args) {
      SpringApplication.run(MsalWebSampleApplication.class, args);
     }

     @Override
     protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
      return builder.sources(MsalWebSampleApplication.class);
     }
    }
   ```

3.   Le port HTTP par défaut de Tomcat est 8080, même si une connexion HTTPS sur le port 8443 est nécessaire. Pour configurer cela :
        - Accédez à tomcat/conf/server.xml
        - Recherchez la balise `<connector>`, puis remplacez le connecteur existant par :

        ```xml
        <Connector
                   protocol="org.apache.coyote.http11.Http11NioProtocol"
                   port="8443" maxThreads="200"
                   scheme="https" secure="true" SSLEnabled="true"
                   keystoreFile="C:/Path/To/Keystore/File/keystore.p12" keystorePass="KeystorePassword"
                   clientAuth="false" sslProtocol="TLS"/>
        ```

4. Ouvrez une invite de commandes, accédez au dossier racine de cet exemple (où se trouve le fichier pom.xml), puis exécutez `mvn package` pour générer le projet.
    - Un fichier `msal-web-sample-0.1.0.war` est généré dans votre répertoire /targets.
    - Renommez ce fichier `msal4jsample.war`.
    - Déployez ce fichier war à l’aide de Tomcat ou de toute autre solution de conteneur J2EE.
        - Pour déployer, copiez le fichier msal4jsample.war dans le dossier `/webapps/` de votre installation Tomcat, puis démarrez le serveur Tomcat.

5. Une fois déployé, accédez à https://localhost:8443/msal4jsample dans votre navigateur.


> [!IMPORTANT]
> Cette application de démarrage rapide utilise un secret client pour s’identifier en tant que client confidentiel. Le secret client étant ajouté en texte brut à vos fichiers projet, il est recommandé, pour des raisons de sécurité, d’utiliser un certificat au lieu d’un secret client avant de considérer l’application comme application de production. Pour plus d’informations sur l’utilisation d’un certificat, consultez [Informations d’identification de certificat pour l’authentification d’application](./active-directory-certificate-credentials.md).

## <a name="more-information"></a>Informations complémentaires

### <a name="how-the-sample-works"></a>Fonctionnement de l’exemple
![Fonctionnement de l’exemple d’application généré par ce guide de démarrage rapide](media/quickstart-v2-java-webapp/java-quickstart.svg)

### <a name="getting-msal"></a>Obtention de MSAL

MSAL for Java (MSAL4J) est la bibliothèque Java qui est utilisée pour connecter les utilisateurs et pour demander des jetons permettant d’accéder à une API protégée par la plateforme d’identités Microsoft.

Ajoutez MSAL4J à votre application en utilisant Maven ou Gradle pour gérer vos dépendances. Pour cela, apportez les modifications suivantes au fichier pom.xml (Maven) ou build.gradle (Gradle) de votre application.

Dans pom.xml :

```XML
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>msal4j</artifactId>
    <version>1.0.0</version>
</dependency>
```

Dans build.gradle :

```$xslt
compile group: 'com.microsoft.azure', name: 'msal4j', version: '1.0.0'
```

### <a name="msal-initialization"></a>Initialisation MSAL

Ajoutez la référence à MSAL for Java en ajoutant le code suivant au début du fichier où vous allez utiliser MSAL4J :

```Java
import com.microsoft.aad.msal4j.*;
```

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir une présentation plus approfondie de la génération d’applications web qui connectent des utilisateurs à la plateforme d’identités Microsoft, passez à notre série de scénarios en plusieurs parties :

> [!div class="nextstepaction"]
> [Scénario : application web qui connecte les utilisateurs](scenario-web-app-sign-user-overview.md?tabs=java)
