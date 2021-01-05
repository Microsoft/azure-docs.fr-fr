---
title: Gestion des sessions - Outil Microsoft de modélisation des menaces - Azure | Microsoft Docs
description: En savoir plus sur l’atténuation des menaces exposées dans l’outil Threat Modeling Tool, possible grâce à la gestion des sessions. Voir les informations sur l’atténuation et les exemples de code.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.custom: has-adal-ref, devx-track-js, devx-track-csharp
ms.openlocfilehash: a1f4d4a3bb78da82753d651e1a73cf244096d5df
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94518099"
---
# <a name="security-frame-session-management"></a>Infrastructure de sécurité : Gestion des sessions
| Produit/Service | Article |
| --------------- | ------- |
| **Azure AD**    | <ul><li>[Implémenter une déconnexion appropriée à l’aide de méthodes ADAL lors de l’utilisation d’Azure AD](#logout-adal)</li></ul> |
| **Appareil IoT** | <ul><li>[Utiliser des durées de vie limitées pour les jetons SaS générés](#finite-tokens)</li></ul> |
| **Azure Document DB** | <ul><li>[Utiliser des durées de vie de jeton minimales pour les jetons de ressource générés](#resource-tokens)</li></ul> |
| **ADFS** | <ul><li>[Implémenter une déconnexion appropriée à l’aide de méthodes WsFederation lors de l’utilisation d’ADFS](#wsfederation-logout)</li></ul> |
| **Serveur d’identité** | <ul><li>[Implémenter une déconnexion appropriée lors de l’utilisation d’IdentityServer](#proper-logout)</li></ul> |
| **Application Web** | <ul><li>[Utilisation requise de cookies sécurisés par les applications disponibles par le biais de HTTPS](#https-secure-cookies)</li><li>[Élément httpOnly requis dans la définition des cookies pour toutes les applications basées sur HTTP](#cookie-definition)</li><li>[Prévenir les attaques de falsification de requête intersites (CSRF, Cross Site Request Forgery) sur les pages web ASP.NET](#csrf-asp)</li><li>[Configurer la durée de vie d’inactivité d’une session](#inactivity-lifetime)</li><li>[Implémenter une déconnexion appropriée de l’application](#proper-app-logout)</li></ul> |
| **API Web** | <ul><li>[Prévenir les attaques de falsification de requête intersites (CSRF, Cross Site Request Forgery) sur les API Web ASP.NET](#csrf-api)</li></ul> |

## <a name="implement-proper-logout-using-adal-methods-when-using-azure-ad"></a><a id="logout-adal"></a>Implémenter une déconnexion appropriée à l’aide de méthodes ADAL lors de l’utilisation d’Azure AD

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Azure AD | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Si l’application s’appuie sur le jeton d’accès émis par Azure AD, le gestionnaire d’événements de déconnexion doit appeler |

### <a name="example"></a> Exemple
```csharp
HttpContext.GetOwinContext().Authentication.SignOut(OpenIdConnectAuthenticationDefaults.AuthenticationType, CookieAuthenticationDefaults.AuthenticationType)
```

### <a name="example"></a>Exemple
Il est également nécessaire de détruire la session de l’utilisateur en appelant la méthode Session.Abandon(). La méthode ci-après présente une implémentation sécurisée de la déconnexion d’un utilisateur :
```csharp
    [HttpPost]
        [ValidateAntiForgeryToken]
        public void LogOff()
        {
            string userObjectID = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value;
            AuthenticationContext authContext = new AuthenticationContext(Authority + TenantId, new NaiveSessionCache(userObjectID));
            authContext.TokenCache.Clear();
            Session.Clear();
            Session.Abandon();
            Response.SetCookie(new HttpCookie("ASP.NET_SessionId", string.Empty));
            HttpContext.GetOwinContext().Authentication.SignOut(
                OpenIdConnectAuthenticationDefaults.AuthenticationType,
                CookieAuthenticationDefaults.AuthenticationType);
        } 
```

## <a name="use-finite-lifetimes-for-generated-sas-tokens"></a><a id="finite-tokens"></a>Utiliser des durées de vie limitées pour les jetons SaS générés

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Appareil IoT | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Les jetons SaS générés pour l’authentification auprès d’Azure IoT Hub doivent présenter une période d’expiration limitée dans le temps. Conservez les durées de vie des jetons SaS sur une valeur minimale pour limiter la durée pendant laquelle ces jetons peuvent être relus dans le cas où leur intégrité a été compromise.|

## <a name="use-minimum-token-lifetimes-for-generated-resource-tokens"></a><a id="resource-tokens"></a>Utiliser des durées de vie de jeton minimales pour les jetons de ressource générés

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Azure Document DB | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Définissez la période de validité d’un jeton de ressource sur une valeur minimale. Les jetons de ressource ont une durée de validité par défaut d'une heure.|

## <a name="implement-proper-logout-using-wsfederation-methods-when-using-adfs"></a><a id="wsfederation-logout"></a>Implémenter une déconnexion appropriée à l’aide de méthodes WsFederation lors de l’utilisation d’ADFS

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | ADFS | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Si l’application s’appuie sur un jeton STS (service d’émission de jeton de sécurité) émis par les services ADFS (Active Directory Federation Services), le gestionnaire d’événements de déconnexion doit appeler la méthode WSFederationAuthenticationModule.FederatedSignOut() pour déconnecter l’utilisateur. En outre, la session actuelle doit être détruite, et la valeur de jeton de session doit être réinitialisée et annulée.|

### <a name="example"></a>Exemple
```csharp
        [HttpPost, ValidateAntiForgeryToken]
        [Authorization]
        public ActionResult SignOut(string redirectUrl)
        {
            if (!this.User.Identity.IsAuthenticated)
            {
                return this.View("LogOff", null);
            }

            // Removes the user profile.
            this.Session.Clear();
            this.Session.Abandon();
            HttpContext.Current.Response.Cookies.Add(new System.Web.HttpCookie("ASP.NET_SessionId", string.Empty)
                {
                    Expires = DateTime.Now.AddDays(-1D),
                    Secure = true,
                    HttpOnly = true
                });

            // Signs out at the specified security token service (STS) by using the WS-Federation protocol.
            Uri signOutUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Issuer);
            Uri replyUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Realm);
            if (!string.IsNullOrEmpty(redirectUrl))
            {
                replyUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Realm + redirectUrl);
            }
           //     Signs out of the current session and raises the appropriate events.
            var authModule = FederatedAuthentication.WSFederationAuthenticationModule;
            authModule.SignOut(false);
        //     Signs out at the specified security token service (STS) by using the WS-Federation
        //     protocol.            
            WSFederationAuthenticationModule.FederatedSignOut(signOutUrl, replyUrl);
            return new RedirectResult(redirectUrl);
        }
```

## <a name="implement-proper-logout-when-using-identity-server"></a><a id="proper-logout"></a>Implémenter une déconnexion appropriée lors de l’utilisation d’IdentityServer

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | IdentityServer | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [IdentityServer3-Federated sign out](https://identityserver.github.io/Documentation/docsv2/advanced/federated-signout.html) |
| **Étapes** | IdentityServer prend en charge une possibilité de fédération avec des fournisseurs d’identité externes. Lorsqu’un utilisateur se déconnecte d’un fournisseur d’identité en amont, selon le protocole utilisé, la déconnexion de l’utilisateur peut parfois être signalée par une notification. Cela permet à IdentityServer d’informer ses clients pour que ceux-ci puissent également déconnecter l’utilisateur. Pour plus d’informations sur les détails d’implémentation, consultez la section relative aux références dans la documentation.|

## <a name="applications-available-over-https-must-use-secure-cookies"></a><a id="https-secure-cookies"></a>Utilisation requise de cookies sécurisés par les applications disponibles par le biais de HTTPS

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | EnvironmentType - OnPrem |
| **Informations de référence**              | [httpCookies, élément (Schéma des paramètres ASP.NET)](/previous-versions/dotnet/netframework-4.0/ms228262(v=vs.100)), [HttpCookie.Secure Property (Propriété HttpCookie.Secure)](/dotnet/api/system.web.httpcookie.secure) |
| **Étapes** | Normalement, les cookies sont uniquement accessibles au domaine auquel ils ont été étendus. Malheureusement, la définition du terme « domaine » n’inclut pas le protocole, de sorte que les cookies créés par le biais du protocole HTTPS sont accessibles par l’intermédiaire de HTTP. L’attribut « secure » indique donc au navigateur que le cookie doit uniquement être accessible via HTTPS. Assurez-vous que tous les cookies définis sur HTTPS utilisent l’attribut **secure**. Il est possible d’appliquer cette exigence dans le fichier web.config en définissant l’attribut requireSSL sur la valeur true. Cette approche est recommandée, car elle appliquera l’attribut **secure** à tous les cookies actuels et futurs sans nécessiter aucune modification de code supplémentaire.|

### <a name="example"></a>Exemple
```csharp
<configuration>
  <system.web>
    <httpCookies requireSSL="true"/>
  </system.web>
</configuration>
```
Le paramètre est appliqué même en cas d’utilisation du protocole HTTP pour accéder à l’application. Si le protocole HTTP est utilisé, ce paramètre arrête l’application, car les cookies étant définis avec l’attribut secure, le navigateur ne les renvoie pas à l’application.

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Web Forms, MVC5 |
| **Attributs**              | EnvironmentType - OnPrem |
| **Informations de référence**              | N/A  |
| **Étapes** | Lorsque l’application web constitue la partie de confiance et que le fournisseur d’identité est le serveur ADFS, il est possible de configurer l’attribut secure du jeton FedAuth en définissant l’élément requireSSL sur la valeur True dans la section `system.identityModel.services` du fichier web.config :|

### <a name="example"></a>Exemple
```csharp
  <system.identityModel.services>
    <federationConfiguration>
      <!-- Set requireSsl=true; domain=application domain name used by FedAuth cookies (Ex: .gdinfra.com); -->
      <cookieHandler requireSsl="true" persistentSessionLifetime="0.0:20:0" />
    ....  
    </federationConfiguration>
  </system.identityModel.services>
```

## <a name="all-http-based-application-should-specify-http-only-for-cookie-definition"></a><a id="cookie-definition"></a>Élément httpOnly requis dans la définition des cookies pour toutes les applications basées sur HTTP

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Secure Cookie Attribute (Attribut de cookie sécurisé)](https://en.wikipedia.org/wiki/HTTP_cookie#Secure_cookie) |
| **Étapes** | Pour atténuer les risques de divulgation d’informations découlant d’une attaque par exécution de scripts intersites (XSS), le nouvel attribut httpOnly a été introduit pour les cookies et est pris en charge par tous les principaux navigateurs. L’attribut indique qu’un cookie n’est pas accessible par le biais d’un script. En utilisant des cookies HttpOnly, une application web réduit les risques que des informations sensibles contenues dans le cookie soient volées par l’intermédiaire d’un script et envoyées au site web d’un attaquant. |

### <a name="example"></a>Exemple
Toutes les applications basées sur HTTP qui utilisent des cookies doivent spécifier l’attribut HttpOnly dans la définition des cookies en implémentant la configuration suivante dans le fichier web.config :
```XML
<system.web>
.
.
   <httpCookies requireSSL="false" httpOnlyCookies="true"/>
.
.
</system.web>
```

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Web Forms |
| **Attributs**              | N/A  |
| **Informations de référence**              | [FormsAuthentication.RequireSSL Property (Propriété FormsAuthentication.RequireSSL)](/dotnet/api/system.web.security.formsauthentication.requiressl) |
| **Étapes** | La valeur de propriété RequireSSL est définie dans le fichier de configuration d’une application ASP.NET à l’aide de l’attribut requireSSL de l’élément de configuration. Dans le fichier web.config de votre application ASP.NET, vous pouvez définir l’attribut requireSSL afin de spécifier si TLS (Transport Layer Security), anciennement appelé SSL (Secure Sockets Layer), est requis pour renvoyer le cookie d’authentification par formulaire au serveur.|

### <a name="example"></a>Exemple 
L’exemple de code ci-après définit l’attribut requireSSL dans le fichier web.config.
```XML
<authentication mode="Forms">
  <forms loginUrl="member_login.aspx" cookieless="UseCookies" requireSSL="true"/>
</authentication>
```

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | MVC5 |
| **Attributs**              | EnvironmentType - OnPrem |
| **Informations de référence**              | [Windows Identity Foundation (WIF) Configuration – Part II (Configuration de Windows Identity Foundation (WIF) - Partie II)](/archive/blogs/alikl/windows-identity-foundation-wif-configuration-part-ii-cookiehandler-chunkedcookiehandler-customcookiehandler) |
| **Étapes** | Pour définir l’attribut httpOnly pour les cookies FedAuth, définissez l’attribut hideFromCsript sur la valeur True. |

### <a name="example"></a>Exemple
Le code ci-après présente la configuration appropriée :
```XML
<federatedAuthentication>
<cookieHandler mode="Custom"
                       hideFromScript="true"
                       name="FedAuth"
                       path="/"
                       requireSsl="true"
                       persistentSessionLifetime="25">
</cookieHandler>
</federatedAuthentication>
```

## <a name="mitigate-against-cross-site-request-forgery-csrf-attacks-on-aspnet-web-pages"></a><a id="csrf-asp"></a>Prévenir les attaques de falsification de requête intersites (CSRF, Cross Site Request Forgery) sur les pages web ASP.NET

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | La falsification de requête intersites (CSRF ou XSRF) est un type d’attaque dans lequel une personne malveillante peut exécuter des actions dans le contexte de sécurité de la session établie d’un autre utilisateur sur un site web. L’objectif est de modifier ou de supprimer du contenu, si le site web ciblé s’appuie exclusivement sur les cookies de session pour authentifier la demande reçue. Un attaquant pourrait exploiter cette faille de sécurité en faisant en sorte que le navigateur d’un autre utilisateur charge une URL avec une commande à partir d’un site vulnérable auquel l’utilisateur est déjà connecté. Un attaquant dispose de nombreuses méthodes pour parvenir à ce résultat, notamment en hébergeant un autre site web qui charge une ressource à partir du serveur vulnérable, ou en incitant l’utilisateur à cliquer sur un lien. Il est possible d’éviter cette attaque en faisant en sorte que le serveur envoie un jeton supplémentaire au client, qu’il demande au client d’inclure ce jeton dans toutes les demandes ultérieures et qu’il vérifie que toutes ces demandes comprennent un jeton s’appliquant à la session actuelle, par exemple en utilisant AntiForgeryToken ou ViewState ASP.NET. |

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | MVC5, MVC6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [XSRF/CSRF Prevention in ASP.NET MVC and Web Pages (Prévention des attaques XSRF/CSRF dans les pages MVC et Web ASP.NET)](https://www.asp.net/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages) |
| **Étapes** | Formulaires anti-CSRF et MVC ASP.NET - Utilisez la méthode d’assistance `AntiForgeryToken` sur les vues ; placez un élément `Html.AntiForgeryToken()` dans le formulaire, par exemple,|

### <a name="example"></a> Exemple
```csharp
@using (Html.BeginForm("UserProfile", "SubmitUpdate")) { 
    @Html.ValidationSummary(true) 
    @Html.AntiForgeryToken()
    <fieldset> 
```

### <a name="example"></a> Exemple
```csharp
<form action="/UserProfile/SubmitUpdate" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="saTFWpkKN0BYazFtN6c4YbZAmsEwG0srqlUqqloi/fVgeV2ciIFVmelvzwRZpArs" />
    <!-- rest of form goes here -->
</form>
```

### <a name="example"></a>Exemple
Au même moment, Html.AntiForgeryToken() fournit au visiteur un cookie appelé __RequestVerificationToken, présentant la même valeur que la valeur masquée aléatoire indiquée ci-dessus. Ensuite, pour valider une publication de formulaire entrante, ajoutez le filtre [ValidateAntiForgeryToken] à la méthode d’action cible. Par exemple :
```
[ValidateAntiForgeryToken]
public ViewResult SubmitUpdate()
{
// ... etc.
}
```
Un filtre d’autorisation qui vérifie les éléments suivants :
* La demande entrante comporte un cookie appelé __RequestVerificationToken.
* La demande entrante comporte une entrée `Request.Form` appelée __RequestVerificationToken
* Ce cookie et ces valeurs `Request.Form` correspondent. Si tout va bien, la demande s’exécute normalement. Dans le cas contraire, un échec d’autorisation survient avec le message « Un jeton anti-contrefaçon requis n’a pas été fourni ou n’est pas valide ». 

### <a name="example"></a>Exemple
Anti-CSRF et AJAX : le jeton de formulaire peut se révéler problématique pour les requêtes AJAX, car une requête AJAX risque d’envoyer des données JSON, et non des données de formulaire HTML. L’une des solutions consiste à envoyer les jetons dans un en-tête HTTP personnalisé. Le code ci-après utilise la syntaxe Razor pour générer les jetons, puis ajoute les jetons à une demande AJAX. 
```csharp
<script>
    @functions{
        public string TokenHeaderValue()
        {
            string cookieToken, formToken;
            AntiForgery.GetTokens(null, out cookieToken, out formToken);
            return cookieToken + ":" + formToken;                
        }
    }

    $.ajax("api/values", {
        type: "post",
        contentType: "application/json",
        data: {  }, // JSON data goes here
        dataType: "json",
        headers: {
            'RequestVerificationToken': '@TokenHeaderValue()'
        }
    });
</script>
```

### <a name="example"></a>Exemple
Lorsque vous traitez la demande, extrayez les jetons de l’en-tête de la demande. Ensuite, appelez la méthode AntiForgery.Validate pour valider les jetons. Si les jetons ne sont pas valides, la méthode Validate lève une exception.
```csharp
void ValidateRequestHeader(HttpRequestMessage request)
{
    string cookieToken = "";
    string formToken = "";

    IEnumerable<string> tokenHeaders;
    if (request.Headers.TryGetValues("RequestVerificationToken", out tokenHeaders))
    {
        string[] tokens = tokenHeaders.First().Split(':');
        if (tokens.Length == 2)
        {
            cookieToken = tokens[0].Trim();
            formToken = tokens[1].Trim();
        }
    }
    AntiForgery.Validate(cookieToken, formToken);
}
```

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Web Forms |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Utilisation des fonctionnalités intégrées ASP.NET pour repousser les attaques Web](/previous-versions/dotnet/articles/ms972969(v=msdn.10)#securitybarriers_topic2) |
| **Étapes** | Il est possible de prévenir les attaques CSRF dans les applications reposant sur WebForm en définissant l’élément ViewStateUserKey sur une chaîne aléatoire qui varie selon chaque utilisateur (identifiant utilisateur ou, mieux encore, identifiant de session). Pour un certain nombre de raisons techniques et sociales, l’identifiant de session constitue un bien meilleur choix, car ce type d’identifiant est imprévisible, arrive à expiration et varie selon chaque utilisateur.|

### <a name="example"></a>Exemple
Voici le code qui doit apparaître dans toutes vos pages :
```csharp
void Page_Init (object sender, EventArgs e) {
   ViewStateUserKey = Session.SessionID;
   :
}
```

## <a name="set-up-session-for-inactivity-lifetime"></a><a id="inactivity-lifetime"></a>Configurer la durée de vie d’inactivité d’une session

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [HttpSessionState.Timeout Property (Propriété HttpSessionState.Timeout)](/dotnet/api/system.web.sessionstate.httpsessionstate.timeout) |
| **Étapes** | Un événement d’expiration de session survient lorsqu’un utilisateur n’exécute aucune action sur un site web au cours d’un intervalle de temps donné (défini par le serveur web). Côté serveur, cet événement redéfinit l’état de la session utilisateur comme non valide (par exemple, « plus utilisé ») et demande au serveur web de détruire la session (en supprimant toutes les données qu’elle contient). L’exemple de code ci-après définit l’attribut d’expiration de session sur 15 minutes dans le fichier web.config.|

### <a name="example"></a>Exemple
```XML 
<configuration>
  <system.web>
    <sessionState mode="InProc" cookieless="true" timeout="15" />
  </system.web>
</configuration>
```

## <a name="enable-threat-detection-on-azure-sql"></a><a id="threat-detection"></a>Activer la détection des menaces sur SQL Azure

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Web Forms |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Élément Forms pour l'authentification (schéma des paramètres ASP.NET)](/previous-versions/dotnet/netframework-4.0/1d3t3c61(v=vs.100)) |
| **Étapes** | Définissez le délai d’expiration du cookie du ticket d’authentification par formulaire sur 15 minutes.|

### <a name="example"></a>Exemple
```XML
<forms  name=".ASPXAUTH" loginUrl="login.aspx"  defaultUrl="default.aspx" protection="All" timeout="15" path="/" requireSSL="true" slidingExpiration="true"/>
</forms>
```

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Web Forms, MVC5 |
| **Attributs**              | EnvironmentType - OnPrem |
| **Informations de référence**              | [asdeqa](https://skf.azurewebsites.net/Mitigations/Details/wefr) |
| **Étapes** | Quand l’application web est la partie de confiance et qu’ADFS est le service STS, la durée de vie des cookies d’authentification (jetons FedAuth) peut être définie par la configuration suivante dans le fichier web.config :|

### <a name="example"></a> Exemple
```XML
  <system.identityModel.services>
    <federationConfiguration>
      <!-- Set requireSsl=true; domain=application domain name used by FedAuth cookies (Ex: .gdinfra.com); -->
      <cookieHandler requireSsl="true" persistentSessionLifetime="0.0:15:0" />
      <!-- Set requireHttps=true; -->
      <wsFederation passiveRedirectEnabled="true" issuer="http://localhost:39529/" realm="https://localhost:44302/" reply="https://localhost:44302/" requireHttps="true"/>
      <!--
      Use the code below to enable encryption-decryption of claims received from ADFS. Thumbprint value varies based on the certificate being used.
      <serviceCertificate>
        <certificateReference findValue="4FBBBA33A1D11A9022A5BF3492FF83320007686A" storeLocation="LocalMachine" storeName="My" x509FindType="FindByThumbprint" />
      </serviceCertificate>
      -->
    </federationConfiguration>
  </system.identityModel.services>
```

### <a name="example"></a>Exemple
Définissez également la durée de vie du jeton de revendication SAML émis par ADFS sur 15 minutes en exécutant la commande PowerShell ci-après sur le serveur ADFS :
```csharp
Set-ADFSRelyingPartyTrust -TargetName "<RelyingPartyWebApp>" -ClaimsProviderName @("Active Directory") -TokenLifetime 15 -AlwaysRequireAuthentication $true
```

## <a name="implement-proper-logout-from-the-application"></a><a id="proper-app-logout"></a>Implémenter une déconnexion appropriée de l’application

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Effectuez une déconnexion appropriée de l’application lorsque l’utilisateur appuie sur le bouton de déconnexion. Lors de la déconnexion, l’application doit détruire la session de l’utilisateur et également réinitialiser et annuler la valeur du cookie de session, ainsi que la valeur du cookie d’authentification. En outre, lorsque plusieurs sessions sont associées à une seule identité d’utilisateur, elles doivent être terminées de manière collective côté serveur au moment de l’arrivée à expiration ou de la déconnexion. Enfin, assurez-vous que la fonctionnalité de déconnexion est accessible sur chaque page. |

## <a name="mitigate-against-cross-site-request-forgery-csrf-attacks-on-aspnet-web-apis"></a><a id="csrf-api"></a>Prévenir les attaques de falsification de requête intersites (CSRF, Cross Site Request Forgery) sur les API Web ASP.NET

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | La falsification de requête intersites (CSRF ou XSRF) est un type d’attaque dans lequel une personne malveillante peut exécuter des actions dans le contexte de sécurité de la session établie d’un autre utilisateur sur un site web. L’objectif est de modifier ou de supprimer du contenu, si le site web ciblé s’appuie exclusivement sur les cookies de session pour authentifier la demande reçue. Un attaquant pourrait exploiter cette faille de sécurité en faisant en sorte que le navigateur d’un autre utilisateur charge une URL avec une commande à partir d’un site vulnérable auquel l’utilisateur est déjà connecté. Un attaquant dispose de nombreuses méthodes pour parvenir à ce résultat, notamment en hébergeant un autre site web qui charge une ressource à partir du serveur vulnérable, ou en incitant l’utilisateur à cliquer sur un lien. Il est possible d’éviter cette attaque en faisant en sorte que le serveur envoie un jeton supplémentaire au client, qu’il demande au client d’inclure ce jeton dans toutes les demandes ultérieures et qu’il vérifie que toutes ces demandes comprennent un jeton s’appliquant à la session actuelle, par exemple en utilisant AntiForgeryToken ou ViewState ASP.NET. |

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | MVC5, MVC6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Preventing Cross-Site Request Forgery (CSRF) Attacks in ASP.NET Web API (Prévenir les attaques de falsification de requête intersites (CSRF, Cross Site Request Forgery) dans les API Web ASP.NET)](https://www.asp.net/web-api/overview/security/preventing-cross-site-request-forgery-csrf-attacks) |
| **Étapes** | Anti-CSRF et AJAX : le jeton de formulaire peut se révéler problématique pour les requêtes AJAX, car une requête AJAX risque d’envoyer des données JSON, et non des données de formulaire HTML. L’une des solutions consiste à envoyer les jetons dans un en-tête HTTP personnalisé. Le code ci-après utilise la syntaxe Razor pour générer les jetons, puis ajoute les jetons à une demande AJAX. |

### <a name="example"></a> Exemple
```Javascript
<script>
    @functions{
        public string TokenHeaderValue()
        {
            string cookieToken, formToken;
            AntiForgery.GetTokens(null, out cookieToken, out formToken);
            return cookieToken + ":" + formToken;                
        }
    }
    $.ajax("api/values", {
        type: "post",
        contentType: "application/json",
        data: {  }, // JSON data goes here
        dataType: "json",
        headers: {
            'RequestVerificationToken': '@TokenHeaderValue()'
        }
    });
</script>
```

### <a name="example"></a>Exemple
Lorsque vous traitez la demande, extrayez les jetons de l’en-tête de la demande. Ensuite, appelez la méthode AntiForgery.Validate pour valider les jetons. Si les jetons ne sont pas valides, la méthode Validate lève une exception.
```csharp
void ValidateRequestHeader(HttpRequestMessage request)
{
    string cookieToken = "";
    string formToken = "";

    IEnumerable<string> tokenHeaders;
    if (request.Headers.TryGetValues("RequestVerificationToken", out tokenHeaders))
    {
        string[] tokens = tokenHeaders.First().Split(':');
        if (tokens.Length == 2)
        {
            cookieToken = tokens[0].Trim();
            formToken = tokens[1].Trim();
        }
    }
    AntiForgery.Validate(cookieToken, formToken);
}
```

### <a name="example"></a>Exemple
Formulaires anti-CSRF et MVC ASP.NET - Utilisez la méthode d’assistance AntiForgeryToken sur les vues ; placez un élément Html.AntiForgeryToken() dans le formulaire, par exemple,
```csharp
@using (Html.BeginForm("UserProfile", "SubmitUpdate")) { 
    @Html.ValidationSummary(true) 
    @Html.AntiForgeryToken()
    <fieldset> 
}
```

### <a name="example"></a>Exemple
Le résultat de l’exemple ci-dessus devrait ressembler à ceci :
```csharp
<form action="/UserProfile/SubmitUpdate" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="saTFWpkKN0BYazFtN6c4YbZAmsEwG0srqlUqqloi/fVgeV2ciIFVmelvzwRZpArs" />
    <!-- rest of form goes here -->
</form>
```

### <a name="example"></a>Exemple
Au même moment, Html.AntiForgeryToken() fournit au visiteur un cookie appelé __RequestVerificationToken, présentant la même valeur que la valeur masquée aléatoire indiquée ci-dessus. Ensuite, pour valider une publication de formulaire entrante, ajoutez le filtre [ValidateAntiForgeryToken] à la méthode d’action cible. Par exemple :
```
[ValidateAntiForgeryToken]
public ViewResult SubmitUpdate()
{
// ... etc.
}
```
Un filtre d’autorisation qui vérifie les éléments suivants :
* La demande entrante comporte un cookie appelé __RequestVerificationToken.
* La demande entrante comporte une entrée `Request.Form` appelée __RequestVerificationToken
* Ce cookie et ces valeurs `Request.Form` correspondent. Si tout va bien, la demande s’exécute normalement. Dans le cas contraire, un échec d’autorisation survient avec le message « Un jeton anti-contrefaçon requis n’a pas été fourni ou n’est pas valide ».

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | MVC5, MVC6 |
| **Attributs**              | Fournisseur d’identité - ADFS, Fournisseur d’identité - Azure AD |
| **Informations de référence**              | [Secure a Web API with Individual Accounts and Local Login in ASP.NET Web API 2.2 (Sécuriser une API Web avec des comptes individuels et une connexion locale dans API Web ASP.NET 2.2)](https://www.asp.net/web-api/overview/security/individual-accounts-in-web-api) |
| **Étapes** | Si l’API Web est sécurisée à l’aide d’OAuth 2.0, elle attend un jeton du porteur dans l’en-tête de la demande d’autorisation et n’accorde l’accès à la demande que si le jeton est valide. Contrairement à l’authentification basée sur les cookies, les navigateurs n’attachent pas les jetons du porteur aux demandes. Le client effectuant la demande doit attacher explicitement le jeton du porteur dans l’en-tête de la demande. Par conséquent, dans le cas des API Web ASP.NET protégées à l’aide d’OAuth 2.0, les jetons du porteur sont considérés comme un moyen de défense contre les attaques CSRF. Notez que si la partie MVC de l’application utilise l’authentification par formulaire (autrement dit, des cookies), l’application web MVC doit recourir à des jetons anti-contrefaçon. |

### <a name="example"></a>Exemple
L’API Web doit être informée qu’elle doit UNIQUEMENT se fier aux jetons du porteur, et non aux cookies. Cette opération peut être effectuée par le biais de la configuration ci-après dans la méthode `WebApiConfig.Register` :

```csharp
config.SuppressDefaultHostAuthentication();
config.Filters.Add(new HostAuthenticationFilter(OAuthDefaults.AuthenticationType));
```

La méthode SuppressDefaultHostAuthentication indique à l’API Web d’ignorer toute authentification survenant avant que la requête n’atteigne le pipeline d’API Web, soit par Internet Information Services (IIS), soit par l’intergiciel (middleware) OWIN. De cette façon, nous pouvons limiter l’API Web pour qu’elle n’effectue l’authentification qu’à l’aide des jetons du porteur.