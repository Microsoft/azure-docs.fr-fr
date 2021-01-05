---
title: 'Tutoriel : Utiliser le mode d’appareil partagé avec la bibliothèque d’authentification Microsoft (MSAL) pour Android | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, vous allez découvrir comment préparer un appareil Android pour qu’il s’exécute en mode partagé et comment exécuter une application pour travailleurs en première ligne.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 1/15/2020
ms.author: hahamil
ms.reviewer: brandwe
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 981d3a0c5d01d70625fc0d022318c5bc866f23a0
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95756398"
---
# <a name="tutorial-use-shared-device-mode-in-your-android-application"></a>Tutoriel : Utiliser le mode d’appareil partagé dans votre application Android

Dans ce tutoriel, les développeurs Android et les administrateurs de locataires Azure Active Directory (Azure AD) découvrent le code, l’application Authenticator et les paramètres de locataire nécessaires à l’activation du mode d’appareil partagé pour une application Android.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Télécharger un exemple de code
> * Activer et détecter le mode d’appareil partagé
> * Détecter le mode monocompte ou multicompte
> * Détecter un commutateur utilisateur, et activer la connexion et la déconnexion globales
> * Configurer le locataire et inscrire l’application dans le portail Azure
> * Configurer un appareil Android en mode d’appareil partagé
> * Exécution de l'exemple d'application

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="developer-guide"></a>Guide du développeur

Cette section du tutoriel fournit des conseils aux développeurs concernant l’implémentation du mode d’appareil partagé dans une application Android à l’aide de la bibliothèque d’authentification Microsoft (MSAL). Consultez le [tutoriel MSAL Android](./tutorial-v2-android.md) pour voir comment intégrer MSAL à votre application Android, connecter un utilisateur, appeler Microsoft Graph et déconnecter un utilisateur.

### <a name="download-the-sample"></a>Télécharger l’exemple

Clonez l’[exemple d’application](https://github.com/Azure-Samples/ms-identity-android-java/) à partir de GitHub. L’exemple a la possibilité de fonctionner en [mode monocompte ou multicompte](./single-multi-account.md).

### <a name="add-the-msal-sdk-to-your-local-maven-repository"></a>Ajouter le SDK MSAL à votre dépôt Maven local

Si vous n’utilisez pas l’exemple d’application, ajoutez la bibliothèque MSAL en tant que dépendance dans votre fichier build.gradle, comme suit :

```gradle
dependencies{
  implementation 'com.microsoft.identity.client.msal:1.0.+'
}
```

### <a name="configure-your-app-to-use-shared-device-mode"></a>Configurer votre application pour utiliser le mode d’appareil partagé

Reportez-vous à la [documentation de configuration](./msal-configuration.md) pour plus d’informations sur la configuration de votre fichier de configuration.

Affectez à `"shared_device_mode_supported"` la valeur `true` dans votre fichier de configuration MSAL.

Vous n’envisagez peut-être pas de prendre en charge le mode multicompte. Cela peut être le cas si vous n’utilisez pas un appareil partagé et si l’utilisateur peut se connecter à l’application avec plusieurs comptes en même temps. Dans ce cas, affectez à `"account_mode"` la valeur `"SINGLE"`. Cette valeur garantit que votre application obtient toujours `ISingleAccountPublicClientApplication`, ce qui simplifie considérablement l’intégration de MSAL. La valeur par défaut de `"account_mode"` est `"MULTIPLE"`. Il est donc important de modifier cette valeur dans le fichier de configuration si vous utilisez le mode `"single account"`.

Voici un exemple de fichier auth_config.json inclus dans le répertoire **app**>**main**>**res**>**raw** de l’exemple d’application :

```json
{
 "client_id":"Client ID after app registration at https://aka.ms/MobileAppReg",
 "authorization_user_agent":"DEFAULT",
 "redirect_uri":"Redirect URI after app registration at https://aka.ms/MobileAppReg",
 "account_mode":"SINGLE",
 "broker_redirect_uri_registered": true,
 "shared_device_mode_supported": true,
 "authorities":[
  {
   "type":"AAD",
   "audience":{
     "type": "AzureADandPersonalMicrosoftAccount",
     "tenant_id":"common"
   }
  }
 ]
}
```

### <a name="detect-shared-device-mode"></a>Détecter le mode d’appareil partagé

Le mode d’appareil partagé vous permet de configurer des appareils Android de sorte qu’ils soient partagés par plusieurs employés, tout en gérant ces appareils avec des identités Microsoft. Les employés peuvent se connecter à leurs appareils et accéder rapidement aux informations sur les clients. Quand ils ont terminé leur journée ou leur tâche, ils peuvent se déconnecter de toutes les applications sur l’appareil partagé en un seul clic et l’appareil est immédiatement prêt pour être utilisé par l’employé suivant.

Utilisez `isSharedDevice()` pour déterminer si une application est en cours d’exécution sur un appareil qui est en mode d’appareil partagé. Votre application peut utiliser cet indicateur pour déterminer si elle doit modifier l’expérience utilisateur en conséquence.

Voici un extrait de code qui montre comment utiliser `isSharedDevice()`.  Il provient de la classe `SingleAccountModeFragment` dans l’exemple d’application :

```Java
deviceModeTextView.setText(mSingleAccountApp.isSharedDevice() ?"Shared" :"Non-Shared");
```

### <a name="initialize-the-publicclientapplication-object"></a>Initialiser l’objet PublicClientApplication

Si vous définissez `"account_mode":"SINGLE"` dans le fichier de configuration MSAL, vous pouvez caster en toute sécurité l’objet d’application retourné en `ISingleAccountPublicCLientApplication`.

```java
private ISingleAccountPublicClientApplication mSingleAccountApp;

/*Configure your sample app and save state for this activity*/
PublicClientApplication.create(this.getApplicationCOntext(),
  R.raw.auth_config,
  new PublicClientApplication.ApplicationCreatedListener(){
  @Override
  public void onCreated(IPublicClientApplication application){
  mSingleAccountApp = (ISingleAccountPublicClientApplication)application;
  loadAccount();
  }
  @Override
  public void onError(MsalException exception{
  /*Fail to initialize PublicClientApplication */
  }
});
```

### <a name="detect-single-vs-multiple-account-mode"></a>Détecter le mode monocompte ou multicompte

Si vous écrivez une application destinée à être uniquement utilisée par des travailleurs en première ligne sur un appareil partagé, nous vous recommandons de prendre en charge uniquement le mode monocompte. Il inclut la plupart des applications concentrées sur les tâches, comme les applications de dossiers médicaux, les applications de facturation et la plupart des applications métier. Votre travail de développement s’en trouvera simplifié, car de nombreuses fonctionnalités du SDK n’auront pas besoin d’être adaptées.

Si votre application prend en charge plusieurs comptes, ainsi que le mode d’appareil partagé, vous devez effectuer une vérification de type et caster dans l’interface appropriée comme indiqué ci-dessous.

```java
private IPublicClientApplication mApplication;

        if (mApplication instanceOf IMultipleAccountPublicClientApplication) {
          IMultipleAccountPublicClientApplication multipleAccountApplication = (IMultipleAccountPublicClientApplication) mApplication;
          ...
        } else if (mApplication instanceOf    ISingleAccountPublicClientApplication) {
           ISingleAccountPublicClientApplication singleAccountApplication = (ISingleAccountPublicClientApplication) mApplication;
            ...
        }
```

### <a name="get-the-signed-in-user-and-determine-if-a-user-has-changed-on-the-device"></a>Obtenir l’utilisateur connecté et déterminer si un utilisateur a changé sur l’appareil

La méthode `loadAccount` récupère le compte de l’utilisateur connecté. La méthode `onAccountChanged` détermine si l’utilisateur connecté a changé et, le cas échéant, procède à un nettoyage :

```java
private void loadAccount()
{
  mSingleAccountApp.getCurrentAccountAsync(new ISingleAccountPublicClientApplication.CurrentAccountCallback()
  {
    @Overide
    public void onAccountLoaded(@Nullable IAccount activeAccount)
    {
      if (activeAccount != null)
      {
        signedInUser = activeAccount;
        mSingleAccountApp.acquireTokenSilentAsync(SCOPES,"http://login.microsoftonline.com/common",getAuthSilentCallback());
      }
    }
    @Override
    public void on AccountChanged(@Nullable IAccount priorAccount, @Nullable Iaccount currentAccount)
    {
      if (currentAccount == null)
      {
        //Perform a cleanup task as the signed-in account changed.
        updateSingedOutUI();
      }
    }
    @Override
    public void onError(@NonNull Exception exception)
    {
    }
  }
}
```

### <a name="globally-sign-in-a-user"></a>Connecter globalement un utilisateur

Le code suivant connecte un utilisateur sur l’appareil à d’autres applications qui utilisent MSAL avec l’application Authenticator :

```java
private void onSignInClicked()
{
  mSingleAccountApp.signIn(getActivity(), SCOPES, null, getAuthInteractiveCallback());
}
```

### <a name="globally-sign-out-a-user"></a>Déconnecter globalement un utilisateur

L’exemple suivant supprime le compte connecté et efface les jetons mis en cache de l’application, mais également de l’appareil en mode d’appareil partagé :

```java
private void onSignOutClicked()
{
  mSingleAccountApp.signOut(new ISingleAccountPublicClientApplication.SignOutCallback()
  {
    @Override
    public void onSignOut()
    {
      updateSignedOutUI();
    }
    @Override
    public void onError(@NonNull MsalException exception)
    {
      /*failed to remove account with an exception*/
    }
  });
}
```

## <a name="administrator-guide"></a>Guide de l’administrateur

Les étapes suivantes décrivent la configuration de votre application dans le portail Azure et le passage de votre appareil en mode d’appareil partagé.

### <a name="register-your-application-in-azure-active-directory"></a>Inscrire votre application dans Azure Active Directory

Tout d’abord, inscrivez votre application dans le locataire de votre organisation. Ensuite, fournissez les valeurs ci-dessous dans auth_config.json afin que votre application s’exécute correctement.

Pour plus d’informations sur la procédure à suivre, reportez-vous à [Inscrire votre application](./tutorial-v2-android.md#register-your-application).

> [!NOTE]
> Quand vous inscrivez votre application, utilisez le guide de démarrage rapide situé dans la partie gauche, puis sélectionnez **Android**. Vous accédez alors à une page dans laquelle vous êtes invité à fournir le **nom du package** et le **code de hachage de la signature** de votre application. Ces informations sont très importantes pour garantir le fonctionnement de la configuration de votre application. Vous recevez ensuite un objet de configuration que vous pouvez utiliser pour votre application que vous allez couper et coller dans votre fichier auth_config.json.

:::image type="content" source="media/tutorial-v2-shared-device-mode/register-app.png" alt-text="Configurer votre page d’application Android dans le portail Azure - Démarrage rapide":::

Vous devez sélectionner **Apporter cette modification pour moi**, puis fournir les valeurs demandées dans le cadre du guide de démarrage rapide dans le portail Azure. Après cela, nous allons générer tous les fichiers de configuration dont vous avez besoin.

:::image type="content" source="media/tutorial-v2-shared-device-mode/config-info.png" alt-text="Configurer votre page de projet dans le portail Azure - Démarrage rapide":::

## <a name="set-up-a-tenant"></a>Configurer un locataire

À des fins de test, configurez ce qui suit dans votre locataire : au moins deux employés, un seul administrateur d’appareil cloud et un seul administrateur général. Dans le portail Azure, définissez l’administrateur d’appareil cloud en modifiant des rôles organisationnels. Dans le portail Azure, accédez à vos rôles organisationnels en sélectionnant **Azure Active Directory** > **Rôles et administrateurs**  > **Administrateur d’appareil cloud**. Ajoutez les utilisateurs qui peuvent mettre un appareil en mode partagé.

## <a name="set-up-an-android-device-in-shared-mode"></a>Configurer un appareil Android en mode partagé

### <a name="download-the-authenticator-app"></a>Télécharger l’application Authenticator

Téléchargez l’application Microsoft Authenticator à partir de Google Play Store. Si vous avez déjà téléchargé l’application, vérifiez qu’il s’agit de la version la plus récente.

### <a name="authenticator-app-settings--registering-the-device-in-the-cloud"></a>Paramètres de l’application Authenticator et inscription de l’appareil dans le cloud

Lancez l’application Authenticator et accédez à la page du compte principal. Une fois que vous voyez la page **Ajouter un compte**, vous êtes prêt à rendre l’appareil partagé.

:::image type="content" source="media/tutorial-v2-shared-device-mode/authenticator-add-account.png" alt-text="Écran d’ajout de compte dans Authentificator":::

Accédez au volet **Paramètres** à l’aide de la barre de menus de droite. Sélectionnez **Inscription d’appareil** sous **Comptes professionnels et scolaires**.

:::image type="content" source="media/tutorial-v2-shared-device-mode/authenticator-settings.png" alt-text="Écran de paramètres d’Authenticator":::

Lorsque vous cliquez sur ce bouton, vous êtes invité à autoriser l’accès aux contacts de l’appareil. Ceci est dû à l’intégration de compte d’Android sur l’appareil. Choisissez **Autoriser**.

:::image type="content" source="media/tutorial-v2-shared-device-mode/authenticator-allow-screen.png" alt-text="Écran de confirmation d’autorisation d’accès Authenticator":::

L’administrateur d’appareil cloud doit entrer son adresse e-mail professionnelle sous **Ou inscrire en tant qu’appareil partagé**. Cliquez ensuite sur le bouton **Inscrire en tant qu’appareil partagé**, puis entrez ses informations d’identification.

:::image type="content" source="media/tutorial-v2-shared-device-mode/register-device.png" alt-text="Écran d’inscription de l’appareil dans l’application":::

:::image type="content" source="media/tutorial-v2-shared-device-mode/sign-in.png" alt-text="Capture d’écran de l’application montrant la page de connexion Microsoft":::

L’appareil est maintenant en mode partagé.

:::image type="content" source="media/tutorial-v2-shared-device-mode/shared-device-mode-screen.png" alt-text="Écran de l’application montrant que le mode d’appareil partagé est activé":::

 Toutes les connexions et déconnexions sur l’appareil sont globales, ce qui signifie qu’elles s’appliquent à toutes les applications intégrées à MSAL et Microsoft Authenticator sur l’appareil. Vous pouvez maintenant déployer des applications sur l’appareil qui utilise les fonctionnalités du mode d’appareil partagé.

## <a name="view-the-shared-device-in-the-azure-portal"></a>Voir l’appareil partagé dans le portail Azure

Une fois que vous avez passé un appareil en mode partagé, il est connu de votre organisation et suivi dans son locataire. Vous pouvez voir vos appareils partagés en examinant le **type de jonction** dans le panneau Azure Active Directory de votre portail Azure.

:::image type="content" source="media/tutorial-v2-shared-device-mode/registered-device-screen.png" alt-text="Volet Tous les appareils affiché dans le portail Azure":::

## <a name="running-the-sample-app"></a>Exécution de l’exemple d’application

L’exemple d’application est une application simple qui appelle l’API Graph de votre organisation. À la première exécution, vous êtes invité à donner votre consentement, car l’application est nouvelle pour votre compte d’employé.

:::image type="content" source="media/tutorial-v2-shared-device-mode/run-app-permissions-requested.png" alt-text="Écran d’informations sur la configuration de l’application":::

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur l’utilisation de la bibliothèque d’authentification Microsoft et du mode d’appareil partagé sur les appareils Android :

> [!div class="nextstepaction"]
> [Mode d’appareil partagé pour les appareils Android](msal-android-shared-devices.md)
