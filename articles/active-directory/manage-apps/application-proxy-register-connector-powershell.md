---
title: Installation sans assistance du connecteur Azure AD App Proxy | Documents Microsoft
description: Explique comment effectuer une installation silencieuse du connecteur du Proxy d’application Azure AD pour offrir un accès à distance sécurisé à vos applications locales.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/24/2020
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro, has-adal-ref
ms.collection: M365-identity-device-management
ms.openlocfilehash: f46071d15db57f64eb60d9267d4081a2b0b74a50
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658432"
---
# <a name="create-an-unattended-installation-script-for-the-azure-ad-application-proxy-connector"></a>Créer un script d’installation sans assistance pour le connecteur de proxy d’application Azure AD

Cette rubrique vous aide à créer un script Windows PowerShell qui active l’installation sans assistance et l’inscription de votre connecteur de proxy d’application Azure AD.

Cette fonctionnalité est utile lorsque vous souhaitez :

* Installer le connecteur sur des serveurs Windows sur lesquels aucune interface utilisateur n’est activée ou auxquels vous ne pouvez pas accéder avec le Bureau à distance
* Installer et inscrire beaucoup de connecteurs à la fois.
* Intégrer l’installation et l’inscription du connecteur dans le cadre d’une autre procédure.
* Créer une image de serveur standard qui contient les exécutables du connecteur, mais qui n’est pas inscrite.

Pour que le [connecteur de proxy d’application](application-proxy-connectors.md) puisse fonctionner, vous devez l’inscrire auprès de votre répertoire Azure AD à l’aide d’un identifiant d’administrateur d’application et d’un mot de passe. En général, ces informations sont saisies lors de l’installation du connecteur dans une boîte de dialogue contextuelle, mais vous pouvez utiliser PowerShell pour automatiser ce processus.

Une installation sans assistance comporte deux étapes. Tout d’abord, vous devez installer le connecteur. Ensuite, vous devez inscrire le connecteur auprès d’Azure Active Directory.

> [!IMPORTANT]
> Si vous installez le connecteur pour le cloud Azure Government, suivez les [prérequis](../hybrid/reference-connect-government-cloud.md#allow-access-to-urls) et les [étapes d’installation](../hybrid/reference-connect-government-cloud.md#install-the-agent-for-the-azure-government-cloud). Cela nécessite l’activation de l’accès à un autre ensemble d’URL et un paramètre supplémentaire pour exécuter l’installation.

## <a name="install-the-connector"></a>Installation du connecteur
Effectuez les étapes suivantes pour installer le connecteur sans l’inscrire :

1. Ouvrez une invite de commandes.
2. Exécutez la commande suivante, dans laquelle /q signifie « installation silencieuse ». Une installation silencieuse ne vous invite pas à accepter le Contrat de Licence Utilisateur Final.

   ```
   AADApplicationProxyConnectorInstaller.exe REGISTERCONNECTOR="false" /q
   ```

## <a name="register-the-connector-with-azure-ad"></a>Inscription du connecteur sur Azure Active Directory
Deux méthodes permettent d’inscrire le connecteur :

* Inscription du connecteur à l’aide d’un objet d’informations d’identification Windows PowerShell
* Inscription du connecteur à l’aide d’un jeton créé hors connexion

### <a name="register-the-connector-using-a-windows-powershell-credential-object"></a>Inscription du connecteur à l’aide d’un objet d’informations d’identification Windows PowerShell
1. Créez un objet d’informations d’identification Windows PowerShell `$cred` qui contient un nom d’utilisateur et un mot de passe d’administration pour votre annuaire. Exécutez la commande suivante, en remplaçant *\<username\>* et *\<password\>* :

   ```powershell
   $User = "<username>"
   $PlainPassword = '<password>'
   $SecurePassword = $PlainPassword | ConvertTo-SecureString -AsPlainText -Force
   $cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $User, $SecurePassword
   ```
2. Accédez à **C:\Program Files\Microsoft AAD App Proxy Connector** et exécutez le script suivant en utilisant l’objet `$cred` que vous avez créé :

   ```powershell
   .\RegisterConnector.ps1 -modulePath "C:\Program Files\Microsoft AAD App Proxy Connector\Modules\" -moduleName "AppProxyPSModule" -Authenticationmode Credentials -Usercredentials $cred -Feature ApplicationProxy
   ```

### <a name="register-the-connector-using-a-token-created-offline"></a>Inscription du connecteur à l’aide d’un jeton créé hors connexion
1. Créez un jeton hors connexion à l’aide de la classe AuthenticationContext en utilisant les valeurs de cet extrait de code ou les applets de commande PowerShell ci-dessous :

   **Utilisation de C# :**

   ```csharp
   using System;
   using System.Linq;
   using System.Collections.Generic;
   using Microsoft.Identity.Client;

   class Program
   {
   #region constants
   /// <summary>
   /// The AAD authentication endpoint uri
   /// </summary>
   static readonly string AadAuthenticationEndpoint = "https://login.microsoftonline.com/common/oauth2/v2.0/authorize";

   /// <summary>
   /// The application ID of the connector in AAD
   /// </summary>
   static readonly string ConnectorAppId = "55747057-9b5d-4bd4-b387-abf52a8bd489";
 
   /// <summary>
   /// The AppIdUri of the registration service in AAD
   /// </summary>
   static readonly string RegistrationServiceAppIdUri = "https://proxy.cloudwebappproxy.net/registerapp/user_impersonation";

   #endregion

   #region private members
   private string token;
   private string tenantID;
   #endregion

   public void GetAuthenticationToken()
   {
    
   IPublicClientApplication clientApp = PublicClientApplicationBuilder
      .Create(ConnectorAppId)
      .WithDefaultRedirectUri() // will automatically use the default Uri for native app
      .WithAuthority(AadAuthenticationEndpoint)
      .Build();

      AuthenticationResult authResult = null;
            
      IAccount account = null;

      IEnumerable<string> scopes = new string[] { RegistrationServiceAppIdUri };

      try
      {
       authResult = await clientApp.AcquireTokenSilent(scopes, account).ExecuteAsync();
      }
       catch (MsalUiRequiredException ex)
      {
       authResult = await clientApp.AcquireTokenInteractive(scopes).ExecuteAsync();
      }


      if (authResult == null || string.IsNullOrEmpty(authResult.AccessToken) || string.IsNullOrEmpty(authResult.TenantId))
      {
       Trace.TraceError("Authentication result, token or tenant id returned are null");
       throw new InvalidOperationException("Authentication result, token or tenant id returned are null");
      }

      token = authResult.AccessToken;
      tenantID = authResult.TenantId;
      }
      ```

   **Utilisation de PowerShell :**

   ```powershell
   # Load MSAL (Tested with version 4.7.1) 

   Add-Type -Path "..\MSAL\Microsoft.Identity.Client.dll" 
        
   # The AAD authentication endpoint uri
        
   $authority = "https://login.microsoftonline.com/common/oauth2/v2.0/authorize"

   #The application ID of the connector in AAD

   $connectorAppId = "55747057-9b5d-4bd4-b387-abf52a8bd489";

   #The AppIdUri of the registration service in AAD
   $registrationServiceAppIdUri = "https://proxy.cloudwebappproxy.net/registerapp/user_impersonation"

   # Define the resources and scopes you want to call 

   $scopes = New-Object System.Collections.ObjectModel.Collection["string"] 

   $scopes.Add($registrationServiceAppIdUri)

   $app = [Microsoft.Identity.Client.PublicClientApplicationBuilder]::Create($connectorAppId).WithAuthority($authority).WithDefaultRedirectUri().Build()

   [Microsoft.Identity.Client.IAccount] $account = $null

   # Acquiring the token 

   $authResult = $null

   $authResult = $app.AcquireTokenInteractive($scopes).WithAccount($account).ExecuteAsync().ConfigureAwait($false).GetAwaiter().GetResult()

   # Check AuthN result
   If (($authResult) -and ($authResult.AccessToken) -and ($authResult.TenantId)) {
        
   $token = $authResult.AccessToken
   $tenantId = $authResult.TenantId

   Write-Output "Success: Authentication result returned."
        
   }
   Else {
         
   Write-Output "Error: Authentication result, token or tenant id returned with null."
        
   } 
   ```

2. Une fois que vous disposez du jeton, créez une chaîne SecureString en utilisant le jeton :

   ```powershell
   $SecureToken = $Token | ConvertTo-SecureString -AsPlainText -Force
   ```

3. Exécuter la commande Windows PowerShell suivante, en remplaçant \<tenant GUID\> par votre ID de répertoire :

   ```powershell
   .\RegisterConnector.ps1 -modulePath "C:\Program Files\Microsoft AAD App Proxy Connector\Modules\" -moduleName "AppProxyPSModule" -Authenticationmode Token -Token $SecureToken -TenantId <tenant GUID> -Feature ApplicationProxy
   ```

## <a name="next-steps"></a>Étapes suivantes
* [Publier des applications avec votre propre nom de domaine](application-proxy-configure-custom-domain.md)
* [Activer l’authentification unique](application-proxy-configure-single-sign-on-with-kcd.md)
* [Résoudre les problèmes rencontrés avec le proxy d’application](application-proxy-troubleshoot.md)