---
title: Se connecter en toute sécurité à un cluster Azure Service Fabric
description: Décrit comment authentifier l’accès client à un cluster Service Fabric et comment sécuriser les communications entre les clients et un cluster.
ms.topic: conceptual
ms.date: 01/29/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 2ce0325f7aebac92eb1e7deadd6f6875b75b3755
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89008809"
---
# <a name="connect-to-a-secure-cluster"></a>Se connecter à un cluster sécurisé

Lorsqu’un client se connecte à un nœud de cluster Service Fabric, il peut être authentifié et une communication sécurisée peut être établie à l’aide de la sécurité par certificat ou d’Azure Active Directory (AAD). Cette authentification garantit que seuls les utilisateurs autorisés puissent accéder au cluster et aux applications déployées, et effectuer des tâches de gestion.  La sécurité par certificat ou AAD doit avoir été précédemment activé sur le cluster à sa création.  Pour plus d’informations sur les scénarios de sécurité des clusters, consultez [Sécurité des clusters](service-fabric-cluster-security.md). Si vous vous connectez à un cluster sécurisé avec des certificats, [configurez le certificat client](service-fabric-connect-to-secure-cluster.md#connectsecureclustersetupclientcert) sur l’ordinateur qui se connecte au cluster. 

<a id="connectsecureclustercli"></a> 

## <a name="connect-to-a-secure-cluster-using-azure-service-fabric-cli-sfctl"></a>Se connecter à un cluster sécurisé à l’aide de l’interface CLI Azure Service Fabric (sfctl)

Il existe différentes manières de se connecter à un cluster sécurisé à l’aide de l’interface CLI Service Fabric (sfctl). Lorsque vous utilisez un certificat client pour l’authentification, les détails du certificat doivent correspondre à un certificat déployé sur les nœuds de cluster. Si votre certificat dispose d’autorités de certification, vous devez également spécifier les autorités de certification approuvées.

Vous pouvez vous connecter à un cluster à l’aide de la commande `sfctl cluster select`.

Les certificats clients peuvent être spécifiés de deux façons différentes : en tant que paire certificat/clé ou en tant que fichier PFX unique. Pour les fichiers PEM protégés par mot de passe, vous serez automatiquement invité à entrer le mot de passe. Si vous avez obtenu le certificat client sous forme de fichier PFX, commencez par convertir le fichier PFX en fichier PEM à l'aide de la commande suivante. 

```shell
openssl pkcs12 -in your-cert-file.pfx -out your-cert-file.pem -nodes -passin pass:your-pfx-password
```

Si votre fichier .pfx n'est pas protégé par mot de passe, utilisez -passin pass: en guise de dernier paramètre.

Pour spécifier le certificat client en tant que fichier .pem, spécifiez le chemin d’accès de fichier dans l’argument `--pem`. Par exemple :

```shell
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --pem ./client.pem
```

Les fichiers .pem protégés par mot de passe demanderont un mot de passe avant d’exécuter une commande.

Pour spécifier un certificat, la paire de clés utilise les arguments `--cert` et `--key` afin de spécifier les chemins d’accès de fichier vers chaque fichier respectif.

```shell
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --cert ./client.crt --key ./keyfile.key
```

Il arrive que les certificats utilisés pour sécuriser les clusters de test ou de développement ne parviennent pas à valider le certificat. Pour ignorer la vérification du certificat, spécifiez l’option `--no-verify`. Par exemple :

> [!WARNING]
> N’utilisez pas l’option `no-verify` lorsque vous vous connectez aux clusters Service Fabric de production.

```shell
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --pem ./client.pem --no-verify
```

En outre, vous pouvez spécifier des chemins d’accès aux répertoires de certificats d’autorité de certification approuvées, ou à des certificats individuels. Pour spécifier ces chemins d’accès, utilisez l’argument `--ca`. Par exemple :

```shell
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --pem ./client.pem --ca ./trusted_ca
```

Une fois connecté, vous pouvez normalement [exécuter d’autres commandes sfctl](service-fabric-cli.md) pour interagir avec le cluster.

<a id="connectsecurecluster"></a>

## <a name="connect-to-a-cluster-using-powershell"></a>Se connecter à un cluster à l’aide de PowerShell
Avant d’effectuer des opérations sur un cluster par le biais de PowerShell, établissez tout d’abord une connexion au cluster. La connexion au cluster est utilisée pour toutes les commandes suivantes dans la session PowerShell donnée.

### <a name="connect-to-an-unsecure-cluster"></a>Se connecter à un cluster non sécurisé

Pour vous connecter à un cluster non sécurisé, fournissez l’adresse de point de terminaison de cluster à la commande **Connect-ServiceFabricCluster** :

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 
```

### <a name="connect-to-a-secure-cluster-using-azure-active-directory"></a>Se connecter à un cluster sécurisé à l’aide d’Azure Active Directory

Pour vous connecter à un cluster sécurisé qui utilise Azure Active Directory pour autoriser un accès administrateur de cluster, fournissez l’empreinte de certificat du cluster et utilisez l’indicateur *AzureActiveDirectory*.  

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `
-ServerCertThumbprint <Server Certificate Thumbprint> `
-AzureActiveDirectory
```

### <a name="connect-to-a-secure-cluster-using-a-client-certificate"></a>Se connecter à un cluster sécurisé à l’aide d’un certificat client
Exécutez la commande PowerShell suivante pour vous connecter à un cluster sécurisé qui utilise des certificats clients pour autoriser l’accès administrateur. 

#### <a name="connect-using-certificate-common-name"></a>Se connecter à l’aide du nom commun du certificat
Fournissez le nom commun du certificat du cluster et celui du certificat client qui dispose des autorisations de gestion du cluster. Les détails du certificat doivent correspondre à un certificat sur les nœuds du cluster.

```powershell
Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCommonName <certificate common name>  `
    -FindType FindBySubjectName `
    -FindValue <certificate common name> `
    -StoreLocation CurrentUser `
    -StoreName My 
```
*ServerCommonName* est le nom commun du certificat de serveur installé sur les nœuds de cluster. *FindValue* est le nom commun du certificat client administrateur. Lorsque les paramètres sont renseignés, la commande ressemble à l’exemple suivant :
```powershell
$ClusterName= "sf-commonnametest-scus.southcentralus.cloudapp.azure.com:19000"
$certCN = "sfrpe2eetest.southcentralus.cloudapp.azure.com"

Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCommonName $certCN  `
    -FindType FindBySubjectName `
    -FindValue $certCN `
    -StoreLocation CurrentUser `
    -StoreName My 
```

#### <a name="connect-using-certificate-thumbprint"></a>Se connecter à l’aide d’une empreinte numérique du certificat
Fournissez l’empreinte de certificat du cluster et l’empreinte numérique du certificat client qui dispose des autorisations pour la gestion de cluster. Les détails du certificat doivent correspondre à un certificat sur les nœuds du cluster.

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `  
          -KeepAliveIntervalInSec 10 `  
          -X509Credential -ServerCertThumbprint <Certificate Thumbprint> `  
          -FindType FindByThumbprint -FindValue <Certificate Thumbprint> `  
          -StoreLocation CurrentUser -StoreName My
```

*ServerCertThumbprint* est l’empreinte du certificat du serveur installé sur les nœuds du cluster. *FindValue* est l’empreinte du certificat du client administrateur.  Lorsque les paramètres sont renseignés, la commande ressemble à l’exemple suivant :

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint clustername.westus.cloudapp.azure.com:19000 `  
          -KeepAliveIntervalInSec 10 `  
          -X509Credential -ServerCertThumbprint A8136758F4AB8962AF2BF3F27921BE1DF67F4326 `  
          -FindType FindByThumbprint -FindValue 71DE04467C9ED0544D021098BCD44C71E183414E `  
          -StoreLocation CurrentUser -StoreName My 
```

### <a name="connect-to-a-secure-cluster-using-windows-active-directory"></a>Connexion à un cluster sécurisé à l’aide de Windows Active Directory
Si votre cluster autonome est déployé à l’aide de la sécurité d’Active Directory, connectez-vous au cluster en ajoutant le commutateur « WindowsCredential ».

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `
          -WindowsCredential
```

<a id="connectsecureclusterfabricclient"></a>

## <a name="connect-to-a-cluster-using-the-fabricclient-apis"></a>Se connecter à un cluster à l’aide des API FabricClient
Le Kit de développement logiciel (SDK) Service Fabric fournit la classe [FabricClient](/dotnet/api/system.fabric.fabricclient) pour la gestion de cluster. Pour utiliser les API FabricClient, obtenez le package NuGet Microsoft.ServiceFabric.

### <a name="connect-to-an-unsecure-cluster"></a>Se connecter à un cluster non sécurisé

Pour vous connecter à un cluster à distance non sécurisé, créez une instance de FabricClient et fournissez l’adresse du cluster :

```csharp
FabricClient fabricClient = new FabricClient("clustername.westus.cloudapp.azure.com:19000");
```

Pour un code qui s’exécute à partir d’un cluster, par exemple dans un Reliable Service, créez un élément FabricClient *sans* spécifier l’adresse du cluster. FabricClient se connecte à la passerelle de gestion locale sur le nœud sur lequel le code est en cours d’exécution, évitant ainsi un tronçon réseau supplémentaire.

```csharp
FabricClient fabricClient = new FabricClient();
```

### <a name="connect-to-a-secure-cluster-using-a-client-certificate"></a>Se connecter à un cluster sécurisé à l’aide d’un certificat client

Les nœuds du cluster doivent présenter des certificats valides, dont le nom commun ou le nom DNS dans le SAN s’affichent dans la [propriété RemoteCommonNames](/dotnet/api/system.fabric.x509credentials) définie sur [FabricClient](/dotnet/api/system.fabric.fabricclient). Ce processus permet une authentification mutuelle entre le client et les nœuds du cluster.

```csharp
using System.Fabric;
using System.Security.Cryptography.X509Certificates;

string clientCertThumb = "71DE04467C9ED0544D021098BCD44C71E183414E";
string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string CommonName = "www.clustername.westus.azure.com";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var xc = GetCredentials(clientCertThumb, serverCertThumb, CommonName);
var fc = new FabricClient(xc, connection);

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}

static X509Credentials GetCredentials(string clientCertThumb, string serverCertThumb, string name)
{
    X509Credentials xc = new X509Credentials();
    xc.StoreLocation = StoreLocation.CurrentUser;
    xc.StoreName = "My";
    xc.FindType = X509FindType.FindByThumbprint;
    xc.FindValue = clientCertThumb;
    xc.RemoteCommonNames.Add(name);
    xc.RemoteCertThumbprints.Add(serverCertThumb);
    xc.ProtectionLevel = ProtectionLevel.EncryptAndSign;
    return xc;
}
```

### <a name="connect-to-a-secure-cluster-interactively-using-azure-active-directory"></a>Se connecter à un cluster sécurisé de manière interactive à l’aide d’Azure Active Directory

L’exemple suivant utilise Azure Active Directory pour l’identité du client et le certificat de serveur pour l’identité du serveur.

Une fenêtre de connexion interactive s’ouvre automatiquement une fois la connexion au cluster établie.

```csharp
string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var claimsCredentials = new ClaimsCredentials();
claimsCredentials.ServerThumbprints.Add(serverCertThumb);

var fc = new FabricClient(claimsCredentials, connection);

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}
```

### <a name="connect-to-a-secure-cluster-non-interactively-using-azure-active-directory"></a>Se connecter à un cluster sécurisé de manière non interactive à l’aide d’Azure Active Directory

L’exemple suivant est basé sur Microsoft.IdentityModel.Clients.ActiveDirectory, version 2.19.208020213.

Pour plus d’informations sur l’acquisition d’un jeton AAD, consultez [Microsoft.IdentityModel.Clients.ActiveDirectory](/dotnet/api/microsoft.identitymodel.clients.activedirectory?view=azure-dotnet).

```csharp
string tenantId = "C15CFCEA-02C1-40DC-8466-FBD0EE0B05D2";
string clientApplicationId = "118473C2-7619-46E3-A8E4-6DA8D5F56E12";
string webApplicationId = "53E6948C-0897-4DA6-B26A-EE2A38A690B4";

string token = GetAccessToken(
    tenantId,
    webApplicationId,
    clientApplicationId,
    "urn:ietf:wg:oauth:2.0:oob");

string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var claimsCredentials = new ClaimsCredentials();
claimsCredentials.ServerThumbprints.Add(serverCertThumb);
claimsCredentials.LocalClaims = token;

var fc = new FabricClient(claimsCredentials, connection);

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}

...

static string GetAccessToken(
    string tenantId,
    string resource,
    string clientId,
    string redirectUri)
{
    string authorityFormat = @"https://login.microsoftonline.com/{0}";
    string authority = string.Format(CultureInfo.InvariantCulture, authorityFormat, tenantId);
    var authContext = new AuthenticationContext(authority);

    var authResult = authContext.AcquireToken(
        resource,
        clientId,
        new UserCredential("TestAdmin@clustenametenant.onmicrosoft.com", "TestPassword"));
    return authResult.AccessToken;
}

```

### <a name="connect-to-a-secure-cluster-without-prior-metadata-knowledge-using-azure-active-directory"></a>Se connecter à un cluster sécurisé sans connaissance préalable des métadonnées à l’aide d’Azure Active Directory

L’exemple suivant utilise une procédure d’acquisition de jeton non interactive, mais vous pouvez suivre la même approche pour créer une expérience d’acquisition de jeton interactive personnalisée. Les métadonnées Azure Active Directory nécessaires à l’acquisition d’un jeton sont lues à partir de la configuration du cluster.

```csharp
string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var claimsCredentials = new ClaimsCredentials();
claimsCredentials.ServerThumbprints.Add(serverCertThumb);

var fc = new FabricClient(claimsCredentials, connection);

fc.ClaimsRetrieval += (o, e) =>
{
    return GetAccessToken(e.AzureActiveDirectoryMetadata);
};

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}

...

static string GetAccessToken(AzureActiveDirectoryMetadata aad)
{
    var authContext = new AuthenticationContext(aad.Authority);

    var authResult = authContext.AcquireToken(
        aad.ClusterApplication,
        aad.ClientApplication,
        new UserCredential("TestAdmin@clustenametenant.onmicrosoft.com", "TestPassword"));
    return authResult.AccessToken;
}

```

<a id="connectsecureclustersfx"></a>

## <a name="connect-to-a-secure-cluster-using-service-fabric-explorer"></a>Se connecter à un cluster sécurisé à l’aide de Service Fabric Explorer
Pour atteindre [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) pour un cluster donné, dirigez votre navigateur vers :

`http://<your-cluster-endpoint>:19080/Explorer`

L'URL complète est également disponible dans le volet des éléments essentiels du cluster dans le portail Azure.

Pour vous connecter à un cluster sécurisé sur Windows ou OS X à l’aide d’un navigateur, vous pouvez importer le certificat client et le navigateur vous demande le certificat à utiliser pour la connexion au cluster.  Sur les ordinateurs Linux, le certificat devra être importé à l’aide de paramètres du navigateur avancés (chaque navigateur a ses mécanismes) et pointer vers l’emplacement du certificat sur le disque. Pour plus d'informations, consultez [Configuration d'un certificat client](#connectsecureclustersetupclientcert).

### <a name="connect-to-a-secure-cluster-using-azure-active-directory"></a>Se connecter à un cluster sécurisé à l’aide d’Azure Active Directory

Pour vous connecter à un cluster sécurisé avec AAD, dirigez votre navigateur vers :

`https://<your-cluster-endpoint>:19080/Explorer`

Vous êtes automatiquement invité à vous connecter à AAD.

### <a name="connect-to-a-secure-cluster-using-a-client-certificate"></a>Se connecter à un cluster sécurisé à l’aide d’un certificat client

Pour vous connecter à un cluster sécurisé avec des certificats, pointez votre navigateur sur :

`https://<your-cluster-endpoint>:19080/Explorer`

Vous êtes automatiquement invité à sélectionner un certificat client.

<a id="connectsecureclustersetupclientcert"></a>

## <a name="set-up-a-client-certificate-on-the-remote-computer"></a>Configurer un certificat client sur l’ordinateur distant

Au moins deux certificats devraient être utilisés pour sécuriser le cluster, un pour le certificat du cluster et du serveur, et un autre pour l’accès client.  Nous vous recommandons d’utiliser également des certificats secondaires supplémentaires et des certificats d’accès client.  Pour sécuriser la communication entre un client et un nœud de cluster à l’aide de la sécurité par certificat, vous devez d’abord obtenir et installer le certificat client. Ce certificat peut être installé dans le magasin personnel de l’ordinateur local ou de l’utilisateur actuel.  Vous avez également besoin de l’empreinte numérique du certificat du serveur afin que le client puisse authentifier le cluster.

* Sur Windows : double-cliquez sur le fichier PFX et suivez les invites pour installer le certificat dans votre magasin personnel, `Certificates - Current User\Personal\Certificates`. Vous pouvez également utiliser la commande PowerShell :

    ```powershell
    Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My `
            -FilePath C:\docDemo\certs\DocDemoClusterCert.pfx `
            -Password (ConvertTo-SecureString -String test -AsPlainText -Force)
    ```

    S’il s’agit d’un certificat auto-signé, vous devez l’importer dans le magasin de « personnes autorisées » de votre ordinateur avant de pouvoir l’utiliser pour vous connecter à un cluster sécurisé.

    ```powershell
    Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\TrustedPeople `
    -FilePath C:\docDemo\certs\DocDemoClusterCert.pfx `
    -Password (ConvertTo-SecureString -String test -AsPlainText -Force)
    ```

* Sur Mac : double-cliquez sur le fichier PFX et suivez les invites pour installer le certificat dans votre application Keychain.

## <a name="next-steps"></a>Étapes suivantes

* [Processus de mise à niveau du cluster Service Fabric et attentes à votre égard](service-fabric-cluster-upgrade.md)
* [Gestion de vos applications Service Fabric dans Visual Studio](service-fabric-manage-application-in-visual-studio.md)
* [Présentation du modèle d’intégrité de Service Fabric](service-fabric-health-introduction.md)
* [Sécurité des applications et RunAs](service-fabric-application-runas-security.md)
* [Prise en main de l’interface de ligne de commande Service Fabric](service-fabric-cli.md)
