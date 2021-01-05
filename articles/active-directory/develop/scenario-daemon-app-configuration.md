---
title: Configurer des applications démon appelant des API web - Plateforme d’identités Microsoft | Azure
description: Découvrir comment configurer le code de votre application démon qui appelle des API web (configuration d’application)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/19/2020
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: 6864502a9d338a786e1e77dbf9888a7818bb94e9
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95748646"
---
# <a name="daemon-app-that-calls-web-apis---code-configuration"></a>Application démon appelant des API web - Configuration du code

Découvrez comment configurer le code de votre application démon qui appelle des API web.

## <a name="msal-libraries-that-support-daemon-apps"></a>Bibliothèques MSAL qui prennent en charge les applications démon

Ces bibliothèques Microsoft prenant en charge les applications démon :

  Bibliothèque MSAL | Description
  ------------ | ----------
  ![MSAL.NET](media/sample-v2-code/logo_NET.png) <br/> MSAL.NET  | Les plateformes .NET Core et .NET Framework sont prises en charge pour la génération d’applications démon. (Les plateformes UWP, Xamarin.iOS et Xamarin.Android ne sont pas prises en charge, car elles sont utilisées pour créer des applications clientes publiques.)
  ![Python](media/sample-v2-code/logo_python.png) <br/> MSAL Python | Prise en charge des applications démon dans Python.
  ![Java](media/sample-v2-code/logo_java.png) <br/> MSAL Java | Prise en charge des applications démon dans Java.

## <a name="configure-the-authority"></a>Configurer l’autorité

Les applications démon utilisent des permissions d’application plutôt que des permissions déléguées. Le type de compte pris en charge ne peut donc pas être un compte dans un annuaire organisationnel ou un compte Microsoft personnel (par exemple, Skype, Xbox, Outlook.com). Aucun administrateur client n’octroie de consentement à une application démon pour un compte Microsoft personnel. Vous devez choisir des *comptes dans mon organisation* ou des *comptes dans une organisation*.

L’autorité spécifiée dans la configuration de l’application doit avoir des locataires (en spécifiant un ID de locataire ou un nom de domaine associé à votre organisation).

Même si vous souhaitez fournir un outil multilocataire, vous devez utiliser un ID de locataire ou un nom de domaine, et **non pas** `common` ou `organizations` avec ce flux, car le service ne peut pas déduire de manière fiable le locataire à utiliser.

## <a name="configure-and-instantiate-the-application"></a>Configurer et instancier l’application

Dans les bibliothèques MSAL, les informations d’identification de client (secret ou certificat) sont transmises en tant que paramètre de la construction d’application cliente confidentielle.

> [!IMPORTANT]
> Même si votre application est une application console qui s’exécute en tant que service, s’il s’agit d’une application démon, il devra s’agir d’une application cliente confidentielle.

### <a name="configuration-file"></a>Fichier de configuration

Le fichier de configuration définit les éléments suivants :

- L’instance cloud et l’ID de locataire, qui composent ensemble l’*autorité*.
- ID client que vous avez obtenu à partir de l’inscription de l’application ;
- secret client ou certificat.

# <a name="net"></a>[.NET](#tab/dotnet)

Voici un exemple de définition de la configuration dans un fichier [*appsettings.json*](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/master/1-Call-MSGraph/daemon-console/appsettings.json). Cet exemple provient de l’exemple de code [Console .NET Core (démon)](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2) sur GitHub.

```json
{
  "Instance": "https://login.microsoftonline.com/{0}",
  "Tenant": "[Enter here the tenantID or domain name for your Azure AD tenant]",
  "ClientId": "[Enter here the ClientId for your application]",
  "ClientSecret": "[Enter here a client secret for your application]",
  "CertificateName": "[Or instead of client secret: Enter here the name of a certificate (from the user cert store) as registered with your application]"
}
```

Vous fournissez un paramètre `ClientSecret` ou `CertificateName`. Ces paramètres sont exclusifs.

# <a name="python"></a>[Python](#tab/python)

Quand vous générez un client confidentiel avec des secrets client, le fichier config [parameters.json](https://github.com/Azure-Samples/ms-identity-python-daemon/blob/master/1-Call-MsGraph-WithSecret/parameters.json) de l’exemple [Démon Python](https://github.com/Azure-Samples/ms-identity-python-daemon) se présente comme suit :

```Json
{
  "authority": "https://login.microsoftonline.com/<your_tenant_id>",
  "client_id": "your_client_id",
  "scope": [ "https://graph.microsoft.com/.default" ],
  "secret": "The secret generated by AAD during your confidential app registration",
  "endpoint": "https://graph.microsoft.com/v1.0/users"
}
```

Quand vous générez un client confidentiel avec des certificats, le fichier config [parameters.json](https://github.com/Azure-Samples/ms-identity-python-daemon/blob/master/2-Call-MsGraph-WithCertificate/parameters.json) de l’exemple [Démon Python](https://github.com/Azure-Samples/ms-identity-python-daemon) se présente comme suit :

```Json
{
  "authority": "https://login.microsoftonline.com/<your_tenant_id>",
  "client_id": "your_client_id",
  "scope": [ "https://graph.microsoft.com/.default" ],
  "thumbprint": "790E... The thumbprint generated by AAD when you upload your public cert",
  "private_key_file": "server.pem",
  "endpoint": "https://graph.microsoft.com/v1.0/users"
}
```

# <a name="java"></a>[Java](#tab/java)

```Java
 private final static String CLIENT_ID = "";
 private final static String AUTHORITY = "https://login.microsoftonline.com/<tenant>/";
 private final static String CLIENT_SECRET = "";
 private final static Set<String> SCOPE = Collections.singleton("https://graph.microsoft.com/.default");
```

---

### <a name="instantiate-the-msal-application"></a>Instancier l’application MSAL

Pour instancier l’application MSAL, vous devez ajouter, référencer ou importer le package MSAL (selon le langage de programmation).

La construction est différente selon que vous utilisez des certificats ou des secrets clients (ou, en tant que scénario avancé, des assertions signées).

#### <a name="reference-the-package"></a>Référencer le package

Faites référence au package MSAL dans le code de votre application.

# <a name="net"></a>[.NET](#tab/dotnet)

Ajoutez le package NuGet [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client) à votre application, puis ajoutez une directive `using` dans votre code pour le référencer.

Dans MSAL.NET, l’application cliente confidentielle est représentée par l’interface `IConfidentialClientApplication`.

```csharp
using Microsoft.Identity.Client;
IConfidentialClientApplication app;
```

# <a name="python"></a>[Python](#tab/python)

```python
import msal
import json
import sys
import logging
```

# <a name="java"></a>[Java](#tab/java)

```java
import com.microsoft.aad.msal4j.ClientCredentialFactory;
import com.microsoft.aad.msal4j.ClientCredentialParameters;
import com.microsoft.aad.msal4j.ConfidentialClientApplication;
import com.microsoft.aad.msal4j.IAuthenticationResult;
import com.microsoft.aad.msal4j.IClientCredential;
import com.microsoft.aad.msal4j.MsalException;
import com.microsoft.aad.msal4j.SilentParameters;
```

---

#### <a name="instantiate-the-confidential-client-application-with-a-client-secret"></a>Instancier l’application cliente confidentielle avec un secret client

Voici le code permettant d’instancier l’application cliente confidentielle avec un secret client :

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
           .WithClientSecret(config.ClientSecret)
           .WithAuthority(new Uri(config.Authority))
           .Build();
```

`Authority` est une concaténation de l’instance cloud et de l’ID de locataire, par exemple `https://login.microsoftonline.com/contoso.onmicrosoft.com` ou `https://login.microsoftonline.com/eb1ed152-0000-0000-0000-32401f3f9abd`. Dans le fichier *appsettings.json* indiqué dans la section [Fichier de configuration](#configuration-file), ces éléments sont représentés respectivement par les valeurs `Instance` et `Tenant`.

Dans l’exemple de code d’où l’extrait de code précédent est tiré, `Authority` est une propriété dans la classe [AuthenticationConfig](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/ffc4a9f5d9bdba5303e98a1af34232b434075ac7/1-Call-MSGraph/daemon-console/AuthenticationConfig.cs#L61-L70) et elle est définie comme suit :

```csharp
/// <summary>
/// URL of the authority
/// </summary>
public string Authority
{
    get
    {
        return String.Format(CultureInfo.InvariantCulture, Instance, Tenant);
    }
}
```

# <a name="python"></a>[Python](#tab/python)

```Python
# Pass the parameters.json file as an argument to this Python script. E.g.: python your_py_file.py parameters.json
config = json.load(open(sys.argv[1]))

# Create a preferably long-lived app instance that maintains a token cache.
app = msal.ConfidentialClientApplication(
    config["client_id"], authority=config["authority"],
    client_credential=config["secret"],
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )
```

# <a name="java"></a>[Java](#tab/java)

```Java
IClientCredential credential = ClientCredentialFactory.createFromSecret(CLIENT_SECRET);

ConfidentialClientApplication cca =
        ConfidentialClientApplication
                .builder(CLIENT_ID, credential)
                .authority(AUTHORITY)
                .build();
```

---

#### <a name="instantiate-the-confidential-client-application-with-a-client-certificate"></a>Instancier l’application cliente confidentielle avec un certificat client

Voici le code permettant de générer une application avec un certificat :

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
X509Certificate2 certificate = ReadCertificate(config.CertificateName);
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
    .WithCertificate(certificate)
    .WithAuthority(new Uri(config.Authority))
    .Build();
```

# <a name="python"></a>[Python](#tab/python)

```Python
# Pass the parameters.json file as an argument to this Python script. E.g.: python your_py_file.py parameters.json
config = json.load(open(sys.argv[1]))

# Create a preferably long-lived app instance that maintains a token cache.
app = msal.ConfidentialClientApplication(
    config["client_id"], authority=config["authority"],
    client_credential={"thumbprint": config["thumbprint"], "private_key": open(config['private_key_file']).read()},
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )
```

# <a name="java"></a>[Java](#tab/java)

Dans MSAL Java, il existe deux générateurs permettant d’instancier l’application cliente confidentielle avec des certificats :

```Java

InputStream pkcs12Certificate = ... ; /* Containing PCKS12-formatted certificate*/
string certificatePassword = ... ;    /* Contains the password to access the certificate */

IClientCredential credential = ClientCredentialFactory.createFromCertificate(pkcs12Certificate, certificatePassword);

ConfidentialClientApplication cca =
        ConfidentialClientApplication
                .builder(CLIENT_ID, credential)
                .authority(AUTHORITY)
                .build();
```

or

```Java
PrivateKey key = getPrivateKey(); /* RSA private key to sign the assertion */
X509Certificate publicCertificate = getPublicCertificate(); /* x509 public certificate used as a thumbprint */

IClientCredential credential = ClientCredentialFactory.createFromCertificate(key, publicCertificate);

ConfidentialClientApplication cca =
        ConfidentialClientApplication
                .builder(CLIENT_ID, credential)
                .authority(AUTHORITY)
                .build();
```

---

#### <a name="advanced-scenario-instantiate-the-confidential-client-application-with-client-assertions"></a>Scénario avancé : Instancier l’application cliente confidentielle avec des assertions clientes

# <a name="net"></a>[.NET](#tab/dotnet)

À la place d’un secret client ou d’un certificat, l’application cliente confidentielle peut aussi prouver son identité à l’aide d’assertions clientes.

MSAL.NET possède deux méthodes pour fournir des assertions signées à l’application cliente confidentielle :

- `.WithClientAssertion()`
- `.WithClientClaims()`

Lorsque vous utilisez `WithClientAssertion`, vous devez fournir un jeton JWT signé. Ce scénario avancé est détaillé dans [Assertions clientes](msal-net-client-assertions.md).

```csharp
string signedClientAssertion = ComputeAssertion();
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
                                          .WithClientAssertion(signedClientAssertion)
                                          .Build();
```

Lorsque vous utilisez `WithClientClaims`, MSAL.NET génère une assertion signée qui contient les revendications attendues par Azure AD en plus des revendications clientes supplémentaires que vous souhaitez envoyer.
Ce code illustre comment procéder :

```csharp
string ipAddress = "192.168.1.2";
var claims = new Dictionary<string, string> { { "client_ip", ipAddress } };
X509Certificate2 certificate = ReadCertificate(config.CertificateName);
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
                                          .WithAuthority(new Uri(config.Authority))
                                          .WithClientClaims(certificate, claims)
                                          .Build();
```

Encore une fois, pour plus d’informations, consultez [Assertions clientes](msal-net-client-assertions.md).

# <a name="python"></a>[Python](#tab/python)

Dans MSAL Python, vous pouvez fournir des revendications de client à l’aide des revendications qui seront signées par la clé privée de cette `ConfidentialClientApplication`.

```Python
# Pass the parameters.json file as an argument to this Python script. E.g.: python your_py_file.py parameters.json
config = json.load(open(sys.argv[1]))

# Create a preferably long-lived app instance that maintains a token cache.
app = msal.ConfidentialClientApplication(
    config["client_id"], authority=config["authority"],
    client_credential={"thumbprint": config["thumbprint"], "private_key": open(config['private_key_file']).read()},
    client_claims = {"client_ip": "x.x.x.x"}
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )
```

Pour plus d’informations, consultez la documentation de référence de MSAL Python relative à [ConfidentialClientApplication](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.__init__).

# <a name="java"></a>[Java](#tab/java)

```Java
IClientCredential credential = ClientCredentialFactory.createFromClientAssertion(assertion);

ConfidentialClientApplication cca =
        ConfidentialClientApplication
                .builder(CLIENT_ID, credential)
                .authority(AUTHORITY)
                .build();
```

---

## <a name="next-steps"></a>Étapes suivantes

# <a name="net"></a>[.NET](#tab/dotnet)

Passez à l’article suivant de ce scénario, [Acquérir un jeton pour l’application](./scenario-daemon-acquire-token.md?tabs=dotnet).

# <a name="python"></a>[Python](#tab/python)

Passez à l’article suivant de ce scénario, [Acquérir un jeton pour l’application](./scenario-daemon-acquire-token.md?tabs=python).

# <a name="java"></a>[Java](#tab/java)

Passez à l’article suivant de ce scénario, [Acquérir un jeton pour l’application](./scenario-daemon-acquire-token.md?tabs=java).

---
