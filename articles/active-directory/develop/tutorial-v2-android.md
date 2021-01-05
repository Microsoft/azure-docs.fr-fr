---
title: 'Tutoriel : Créer une application Android qui utilise la Plateforme d’identités Microsoft pour l’authentification | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, vous allez créer une application Android qui utilise la plateforme d’identités Microsoft pour connecter les utilisateurs et obtenir un jeton d’accès pour appeler l’API Microsoft Graph en leur nom.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 11/26/2019
ms.author: hahamil
ms.reviewer: brandwe
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 7ba15e66cca7baefdf8cca5cabd5e5d5b1e2c7f7
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507810"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-from-an-android-application"></a>Tutoriel : Connecter des utilisateurs et appeler l’API Microsoft Graph à partir d’une application Android

Dans ce tutoriel, vous allez créer une application Android qui s’intègre à la plateforme d’identités Microsoft afin de connecter des utilisateurs et obtenir un jeton d’accès pour appeler l’API Microsoft Graph.

À la fin de ce tutoriel, votre application acceptera les connexions de comptes Microsoft personnels (notamment outlook.com, live.com et d’autres) ainsi que de comptes professionnels ou scolaires de toute entreprise ou organisation utilisant Azure Active Directory.

Dans ce tutoriel : 

> [!div class="checklist"]
> * Créer un projet d’application Android dans *Android Studio*
> * Inscrire l'application sur le portail Azure
> * Ajouter du code pour prendre en charge la connexion et la déconnexion des utilisateurs
> * Ajouter du code pour appeler l’API Microsoft Graph.
> * Test de l'application

## <a name="prerequisites"></a>Prérequis

* Android Studio 3.5+

## <a name="how-this-tutorial-works"></a>Fonctionnement de ce tutoriel

![Fonctionnement de l’exemple d’application généré par ce tutoriel](../../../includes/media/active-directory-develop-guidedsetup-android-intro/android-intro.svg)

L’application utilisée dans ce tutoriel est destinée à connecter des utilisateurs et à obtenir des données au nom de ces derniers. Ces données sont accessibles par le biais d’une API protégée (l’API Microsoft Graph) qui nécessite une autorisation et est protégée par la plateforme d’identités Microsoft.

Plus précisément :

* Votre application connectera l’utilisateur via un navigateur ou Microsoft Authenticator et le portail d’entreprise Intune.
* L’utilisateur final acceptera les autorisations que votre application a demandées.
* Votre application émet un jeton d’accès pour l’API Microsoft Graph.
* Le jeton d’accès est inclus dans la requête HTTP adressée à l’API web.
* La réponse Microsoft Graph est traitée.

Cet exemple utilise la bibliothèque d’authentification Microsoft pour Android (MSAL) pour implémenter l’authentification : [com.microsoft.identity.client](https://javadoc.io/doc/com.microsoft.identity.client/msal).

MSAL renouvelle automatiquement les jetons, fournit une authentification unique (SSO) entre les autres applications de l’appareil et gère les comptes.

> [!NOTE]
> Ce tutoriel présente des exemples simplifiés montrant comment utiliser MSAL pour Android. Par souci de simplicité, il utilise uniquement le mode monocompte. Pour explorer des scénarios plus complexes, consultez cet [exemple de code fonctionnel](https://github.com/Azure-Samples/ms-identity-android-java/) complet sur GitHub.

## <a name="create-a-project"></a>Création d’un projet
Si vous n’avez pas encore d’application Android, effectuez les étapes suivantes pour configurer un nouveau projet.

1. Ouvrez Android Studio, puis sélectionnez **Démarrer un nouveau projet Android Studio**.
2. Sélectionnez **Activité de base**, puis **Suivant**.
3. Donnez un nom à votre application.
4. Enregistrez le nom du package. Vous allez l’entrer plus tard dans le portail Azure.
5. Remplacez le langage **Kotlin** par **Java**.
6. Affectez à **Niveau d’API minimal** la valeur **API 19** ou une valeur supérieure, puis cliquez sur **Terminer**.
7. Dans la vue du projet, dans la liste déroulante, choisissez **Projet** pour afficher les fichiers projet sources et non-sources, ouvrez **app/build.gradle**, puis affectez à `targetSdkVersion` la valeur `28`.

## <a name="integrate-with-microsoft-authentication-library"></a>Intégrer à l’aide de la bibliothèque d’authentification Microsoft

### <a name="register-your-application"></a>Inscrivez votre application

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
1. Entrez un **nom** pour votre application. Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
1. Sélectionnez **Inscription**.
1. Sous **Gérer**, sélectionnez **Authentification** > **Ajouter une plateforme** > **Android**.
1. Entrez le nom du package de votre projet. Si vous avez téléchargé le code, cette valeur est `com.azuresamples.msalandroidapp`.
1. Dans la section **Hachage de signature** de la page **Configurer votre application Android**, sélectionnez **Création d’un hachage de signature de développement**. et copiez la commande KeyTool à utiliser pour votre plateforme.

   > [!Note]
   > Keytool.exe est installé en même temps que le kit de développement Java (JDK). Vous devez également installer l’outil OpenSSL pour exécuter la commande KeyTool. Pour plus d’informations, reportez-vous à la [documentation Android sur la génération d’une clé](https://developer.android.com/studio/publish/app-signing#generate-key).

1. Entrez le **Hachage de signature** généré par KeyTool.
1. Sélectionnez **Configurer** et enregistrez la **Configuration MSAL** qui apparaît dans la page **Configuration Android** pour pouvoir l’entrer plus tard quand vous devrez configurer votre application.  
1. Sélectionnez **Terminé**.

### <a name="configure-your-application"></a>Configuration de votre application

1. Dans le volet de projet d’Android Studio, accédez à **app\src\main\res**.
1. Cliquez avec le bouton droit sur **res**, puis choisissez **Nouveau** > **Répertoire**. Entrez `raw` en tant que nouveau nom de répertoire, puis cliquez sur **OK**.
1. Dans **app** > **src** > **main** > **res** > **raw**, créez un fichier JSON appelé `auth_config_single_account.json`, puis collez la configuration MSAL que vous avez enregistrée.

    Sous l’URI de redirection, collez :
    ```json
      "account_mode" : "SINGLE",
    ```
    Votre fichier de configuration doit ressembler à l’exemple suivant :
    ```json
    {
      "client_id" : "0984a7b6-bc13-4141-8b0d-8f767e136bb7",
      "authorization_user_agent" : "DEFAULT",
      "redirect_uri" : "msauth://com.azuresamples.msalandroidapp/1wIqXSqBj7w%2Bh11ZifsnqwgyKrY%3D",
      "broker_redirect_uri_registered" : true,
      "account_mode" : "SINGLE",
      "authorities" : [
        {
          "type": "AAD",
          "audience": {
            "type": "AzureADandPersonalMicrosoftAccount",
            "tenant_id": "common"
          }
        }
      ]
    }
   ```

   >[!NOTE]
   >Ce tutoriel montre uniquement comment configurer une application en mode monocompte. Consultez la documentation pour plus d’informations sur le [mode monocompte ou multicompte](./single-multi-account.md) et la [configuration de votre application](./msal-configuration.md).

4. Dans **app** > **src** > **main** > **AndroidManifest.xml**, ajoutez l’activité `BrowserTabActivity` ci-dessous au corps de l’application. Cette entrée permet à Microsoft de rappeler votre application une fois l’authentification terminée :

    ```xml
    <!--Intent filter to capture System Browser or Authenticator calling back to our app after sign-in-->
    <activity
        android:name="com.microsoft.identity.client.BrowserTabActivity">
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="msauth"
                android:host="Enter_the_Package_Name"
                android:path="/Enter_the_Signature_Hash" />
        </intent-filter>
    </activity>
    ```

    Remplacez le nom de package que vous avez inscrit dans le portail Azure pour la valeur `android:host=`.
    Remplacez le hachage de clé que vous avez inscrit dans le portail Azure pour la valeur `android:path=`. Le hachage de signature **ne doit pas** être encodé en tant qu’URL. Assurez-vous de la présence d’un caractère `/` de début dans votre hachage de signature.
    >[!NOTE]
    >Le « nom du package » avec lequel vous remplacerez la valeur `android:host` doit ressembler à ce qui suit : « com.azuresamples.msalandroidapp ». Le « hachage de signature » avec lequel vous remplacerez votre valeur `android:path` doit ressembler à : « /1wIqXSqBj7w+h11ZifsnqwgyKrY= ». Vous pourrez également trouver ces valeurs dans le panneau Authentification de l’inscription de votre application. Notez que votre URI de redirection ressemblera à : « msauth://com.azuresamples.msalandroidapp/1wIqXSqBj7w%2Bh11ZifsnqwgyKrY%3D ». Le hachage de signature est encodé avec une URL à la fin de cette valeur, mais le hachage de signature **ne doit pas** être encodé avec une URL dans votre valeur `android:path`.

## <a name="use-msal"></a>Utiliser MSAL

### <a name="add-msal-to-your-project"></a>Ajouter MSAL à votre projet

1. Dans la fenêtre de projet Android Studio, accédez à **app** > **src** > **build.gradle** et ajoutez :

    ```gradle
    repositories{
        jcenter()
    }
    dependencies{
        implementation 'com.microsoft.identity.client:msal:2.+'
        implementation 'com.microsoft.graph:microsoft-graph:1.5.+'
    }
    packagingOptions{
        exclude("META-INF/jersey-module-version")
    }
    ```
    [Informations complémentaires sur le kit SDK Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-java/)

### <a name="required-imports"></a>Importations nécessaires

Ajoutez le code suivant au début de **app** > **src** > **main**> **java** > **com.example(yourapp)**  > **MainActivity.java**

```java
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import com.google.gson.JsonObject;
import com.microsoft.graph.authentication.IAuthenticationProvider; //Imports the Graph sdk Auth interface
import com.microsoft.graph.concurrency.ICallback;
import com.microsoft.graph.core.ClientException;
import com.microsoft.graph.http.IHttpRequest;
import com.microsoft.graph.models.extensions.*;
import com.microsoft.graph.requests.extensions.GraphServiceClient;
import com.microsoft.identity.client.AuthenticationCallback; // Imports MSAL auth methods
import com.microsoft.identity.client.*;
import com.microsoft.identity.client.exception.*;
```

## <a name="instantiate-publicclientapplication"></a>Instancier PublicClientApplication
#### <a name="initialize-variables"></a>Initialiser les variables
```java
private final static String[] SCOPES = {"Files.Read"};
/* Azure AD v2 Configs */
final static String AUTHORITY = "https://login.microsoftonline.com/common";
private ISingleAccountPublicClientApplication mSingleAccountApp;

private static final String TAG = MainActivity.class.getSimpleName();

/* UI & Debugging Variables */
Button signInButton;
Button signOutButton;
Button callGraphApiInteractiveButton;
Button callGraphApiSilentButton;
TextView logTextView;
TextView currentUserTextView;
```

### <a name="oncreate"></a>onCreate
Dans la classe `MainActivity`, reportez-vous à la méthode onCreate() suivante pour instancier MSAL à l’aide de `SingleAccountPublicClientApplication`.

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    initializeUI();

    PublicClientApplication.createSingleAccountPublicClientApplication(getApplicationContext(),
            R.raw.auth_config_single_account, new IPublicClientApplication.ISingleAccountApplicationCreatedListener() {
                @Override
                public void onCreated(ISingleAccountPublicClientApplication application) {
                    mSingleAccountApp = application;
                    loadAccount();
                }
                @Override
                public void onError(MsalException exception) {
                    displayError(exception);
                }
            });
}
```

### <a name="loadaccount"></a>loadAccount

```java
//When app comes to the foreground, load existing account to determine if user is signed in
private void loadAccount() {
    if (mSingleAccountApp == null) {
        return;
    }

    mSingleAccountApp.getCurrentAccountAsync(new ISingleAccountPublicClientApplication.CurrentAccountCallback() {
        @Override
        public void onAccountLoaded(@Nullable IAccount activeAccount) {
            // You can use the account data to update your UI or your app database.
            updateUI(activeAccount);
        }

        @Override
        public void onAccountChanged(@Nullable IAccount priorAccount, @Nullable IAccount currentAccount) {
            if (currentAccount == null) {
                // Perform a cleanup task as the signed-in account changed.
                performOperationOnSignOut();
            }
        }

        @Override
        public void onError(@NonNull MsalException exception) {
            displayError(exception);
        }
    });
}
```

### <a name="initializeui"></a>initializeUI
Soyez à l’écoute des boutons et appelez les méthodes ou consignez les erreurs en conséquence.
```java
private void initializeUI(){
        signInButton = findViewById(R.id.signIn);
        callGraphApiSilentButton = findViewById(R.id.callGraphSilent);
        callGraphApiInteractiveButton = findViewById(R.id.callGraphInteractive);
        signOutButton = findViewById(R.id.clearCache);
        logTextView = findViewById(R.id.txt_log);
        currentUserTextView = findViewById(R.id.current_user);

        //Sign in user
        signInButton.setOnClickListener(new View.OnClickListener(){
            public void onClick(View v) {
                if (mSingleAccountApp == null) {
                    return;
                }
                mSingleAccountApp.signIn(MainActivity.this, null, SCOPES, getAuthInteractiveCallback());
            }
        });

        //Sign out user
        signOutButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mSingleAccountApp == null){
                    return;
                }
                mSingleAccountApp.signOut(new ISingleAccountPublicClientApplication.SignOutCallback() {
                    @Override
                    public void onSignOut() {
                        updateUI(null);
                        performOperationOnSignOut();
                    }
                    @Override
                    public void onError(@NonNull MsalException exception){
                        displayError(exception);
                    }
                });
            }
        });

        //Interactive
        callGraphApiInteractiveButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mSingleAccountApp == null) {
                    return;
                }
                mSingleAccountApp.acquireToken(MainActivity.this, SCOPES, getAuthInteractiveCallback());
            }
        });

        //Silent
        callGraphApiSilentButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mSingleAccountApp == null){
                    return;
                }
                mSingleAccountApp.acquireTokenSilentAsync(SCOPES, AUTHORITY, getAuthSilentCallback());
            }
        });
    }
```

> [!Important]
> La déconnexion de MSAL entraîne la suppression de toutes les informations connues sur un utilisateur dans l’application. Toutefois, l’utilisateur a toujours une session active sur son appareil. Si l’utilisateur tente de se reconnecter, il peut éventuellement voir l’IU de connexion, mais il n’est pas obligé de ressaisir ses informations d’identification, car la session sur l’appareil est toujours active.

### <a name="getauthinteractivecallback"></a>getAuthInteractiveCallback
Rappel utilisé pour les demandes interactives

```java
private AuthenticationCallback getAuthInteractiveCallback() {
    return new AuthenticationCallback() {
        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            /* Successfully got a token, use it to call a protected resource - MSGraph */
            Log.d(TAG, "Successfully authenticated");
            /* Update UI */
            updateUI(authenticationResult.getAccount());
            /* call graph */
            callGraphAPI(authenticationResult);
        }

        @Override
        public void onError(MsalException exception) {
            /* Failed to acquireToken */
            Log.d(TAG, "Authentication failed: " + exception.toString());
            displayError(exception);
        }
        @Override
        public void onCancel() {
            /* User canceled the authentication */
            Log.d(TAG, "User cancelled login.");
        }
    };
}
```

### <a name="getauthsilentcallback"></a>getAuthSilentCallback
Rappel utilisé pour les demandes silencieuses
```java
private SilentAuthenticationCallback getAuthSilentCallback() {
    return new SilentAuthenticationCallback() {
        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            Log.d(TAG, "Successfully authenticated");
            callGraphAPI(authenticationResult);
        }
        @Override
        public void onError(MsalException exception) {
            Log.d(TAG, "Authentication failed: " + exception.toString());
            displayError(exception);
        }
    };
}
```

## <a name="call-microsoft-graph-api"></a>Appeler l’API Microsoft Graph

Le code suivant montre comment appeler l’API Graph à l’aide du kit SDK Graph.

### <a name="callgraphapi"></a>callGraphAPI

```java
private void callGraphAPI(IAuthenticationResult authenticationResult) {

    final String accessToken = authenticationResult.getAccessToken();

    IGraphServiceClient graphClient =
            GraphServiceClient
                    .builder()
                    .authenticationProvider(new IAuthenticationProvider() {
                        @Override
                        public void authenticateRequest(IHttpRequest request) {
                            Log.d(TAG, "Authenticating request," + request.getRequestUrl());
                            request.addHeader("Authorization", "Bearer " + accessToken);
                        }
                    })
                    .buildClient();
    graphClient
            .me()
            .drive()
            .buildRequest()
            .get(new ICallback<Drive>() {
                @Override
                public void success(final Drive drive) {
                    Log.d(TAG, "Found Drive " + drive.id);
                    displayGraphResult(drive.getRawObject());
                }

                @Override
                public void failure(ClientException ex) {
                    displayError(ex);
                }
            });
}
```

## <a name="add-ui"></a>Ajouter une IU
### <a name="activity"></a>Activité
Si vous souhaitez modéliser votre interface utilisateur en dehors de ce tutoriel, les méthodes suivantes fournissent un guide pour la mise à jour du texte et l’écoute des boutons.

#### <a name="updateui"></a>updateUI
Activez/désactivez les boutons en fonction de l’état de connexion et du texte défini.
```java
private void updateUI(@Nullable final IAccount account) {
    if (account != null) {
        signInButton.setEnabled(false);
        signOutButton.setEnabled(true);
        callGraphApiInteractiveButton.setEnabled(true);
        callGraphApiSilentButton.setEnabled(true);
        currentUserTextView.setText(account.getUsername());
    } else {
        signInButton.setEnabled(true);
        signOutButton.setEnabled(false);
        callGraphApiInteractiveButton.setEnabled(false);
        callGraphApiSilentButton.setEnabled(false);
        currentUserTextView.setText("");
        logTextView.setText("");
    }
}
```
#### <a name="displayerror"></a>displayError
```java
private void displayError(@NonNull final Exception exception) {
       logTextView.setText(exception.toString());
   }
```

#### <a name="displaygraphresult"></a>displayGraphResult

```java
private void displayGraphResult(@NonNull final JsonObject graphResponse) {
      logTextView.setText(graphResponse.toString());
  }
```
#### <a name="performoperationonsignout"></a>performOperationOnSignOut
Méthode permettant de mettre à jour le texte dans l’interface utilisateur pour refléter la déconnexion.

```java
private void performOperationOnSignOut() {
    final String signOutText = "Signed Out.";
    currentUserTextView.setText("");
    Toast.makeText(getApplicationContext(), signOutText, Toast.LENGTH_SHORT)
            .show();
}
```
### <a name="layout"></a>Mise en page

Exemple de fichier `activity_main.xml` permettant d’afficher des boutons et des zones de texte.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#FFFFFF"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:paddingTop="5dp"
        android:paddingBottom="5dp"
        android:weightSum="10">

        <Button
            android:id="@+id/signIn"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="5"
            android:gravity="center"
            android:text="Sign In"/>

        <Button
            android:id="@+id/clearCache"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="5"
            android:gravity="center"
            android:text="Sign Out"
            android:enabled="false"/>

    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:orientation="horizontal">

        <Button
            android:id="@+id/callGraphInteractive"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="5"
            android:text="Get Graph Data Interactively"
            android:enabled="false"/>

        <Button
            android:id="@+id/callGraphSilent"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="5"
            android:text="Get Graph Data Silently"
            android:enabled="false"/>
    </LinearLayout>

    <TextView
        android:text="Getting Graph Data..."
        android:textColor="#3f3f3f"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="5dp"
        android:id="@+id/graphData"
        android:visibility="invisible"/>

    <TextView
        android:id="@+id/current_user"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginTop="20dp"
        android:layout_weight="0.8"
        android:text="Account info goes here..." />

    <TextView
        android:id="@+id/txt_log"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginTop="20dp"
        android:layout_weight="0.8"
        android:text="Output goes here..." />
</LinearLayout>
```

## <a name="test-your-app"></a>Test de l'application

### <a name="run-locally"></a>Exécution locale

Générez et déployez l’application sur un appareil de test ou un émulateur. Vous devez pouvoir vous connecter et obtenir des jetons pour les comptes Azure AD ou les comptes personnels Microsoft.

Une fois que vous êtes connecté, l’application affiche les données retournées par le point de terminaison `/me` Microsoft Graph.
PR 4
### <a name="consent"></a>Consentement

La première fois qu’un utilisateur se connecte à votre application, Microsoft Identity l’invitera à accepter les autorisations demandées. Certains locataires Azure AD ont désactivé le consentement de l’utilisateur, ce qui oblige les administrateurs à donner leur consentement au nom de tous les utilisateurs. Pour prendre en charge ce scénario, vous devez créer votre propre locataire ou recevoir le consentement de l’administrateur.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez l’objet d’application que vous avez créé à l’étape [Inscrivez votre application](#register-your-application).

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur la création d’applications mobiles qui appellent des API web protégées dans notre série de scénarios en plusieurs parties.

> [!div class="nextstepaction"]
> [Scénario : Application mobile appelant des API web](scenario-mobile-overview.md)
