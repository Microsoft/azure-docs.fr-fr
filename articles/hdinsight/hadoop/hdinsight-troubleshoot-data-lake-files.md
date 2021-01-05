---
title: Impossible d’accéder aux fichiers de stockage Data Lake dans Azure HDInsight
description: Impossible d’accéder aux fichiers de stockage Data Lake dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 08/13/2019
ms.openlocfilehash: 8bac53cd08629e8b0a9cb91e596856c0ae6b5a2f
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289117"
---
# <a name="unable-to-access-data-lake-storage-files-in-azure-hdinsight"></a>Impossible d’accéder aux fichiers de stockage Data Lake dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue-acl-verification-failed"></a>Problème : Échec de la vérification d’ACL

Vous recevez un message d’erreur similaire au suivant :

```
LISTSTATUS failed with error 0x83090aa2 (Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.).
```

### <a name="cause"></a>Cause

L’utilisateur peut avoir révoqué les autorisations de principal du service (SP) sur les fichiers/dossiers.

### <a name="resolution"></a>Résolution

1. Vérifiez que le SP dispose d’autorisations « x » pour parcourir le chemin d’accès. Pour plus d’informations, consultez [Autorisations](https://hdinsight.github.io/ClusterCRUD/ADLS/adls-create-permission-setup.html). Exemple de commande `dfs` pour vérifier l’accès aux fichiers/dossiers dans le compte de stockage Data Lake :

    ```
    hdfs dfs -ls /<path to check access>
    ```

1. Configurez les autorisations nécessaires pour accéder au chemin d’accès en fonction de l’opération de lecture/écriture effectuée. Consultez cette page pour obtenir les autorisations nécessaires aux différentes opérations de système de fichiers.

---

## <a name="issue-service-principal-certificate-expiry"></a>Problème : Expiration du certificat du principal du service

Vous recevez un message d’erreur similaire au suivant :

```
Token Refresh failed - Received invalid http response: 500
```

### <a name="cause"></a>Cause

Le certificat fourni pour l’accès du principal du service a peut-être expiré.

1. SSH dans le nœud principal. Vérifiez l’accès au compte de stockage à l’aide de la commande `dfs` suivante :

    ```
    hdfs dfs -ls /
    ```

1. Vérifiez que le message d’erreur est similaire au résultat suivant :

    ```
    {"stderr": "-ls: Token Refresh failed - Received invalid http response: 500, text = Response{protocol=http/1.1, code=500, message=Internal Server Error, url=http://gw0-abccluster.24ajrd4341lebfgq5unsrzq0ue.fx.internal.cloudapp.net:909/api/oauthtoken}}...
    ```

1. Obtenez une des URL de `core-site.xml property` - `fs.azure.datalake.token.provider.service.urls`.

1. Exécutez la commande curl suivante pour récupérer le jeton OAuth.

    ```
    curl gw0-abccluster.24ajrd4341lebfgq5unsrzq0ue.fx.internal.cloudapp.net:909/api/oauthtoken
    ```

1. Le résultat d’un principal du service valide doit être semblable à ce qui suit :

    ```
    {"AccessToken":"MIIGHQYJKoZIhvcNAQcDoIIGDjCCBgoCAQA…….","ExpiresOn":1500447750098}
    ```

1. Si le certificat du principal du service a expiré, vous obtenez un résultat semblable à ce qui suit :

    ```
    Exception in OAuthTokenController.GetOAuthToken: 'System.InvalidOperationException: Error while getting the OAuth token from AAD for AppPrincipalId 23abe517-2ffd-4124-aa2d-7c224672cae2, ResourceUri https://management.core.windows.net/, AADTenantId https://login.windows.net/80abc8bf-86f1-41af-91ab-2d7cd011db47, ClientCertificateThumbprint C49C25705D60569884EDC91986CEF8A01A495783 ---> Microsoft.IdentityModel.Clients.ActiveDirectory.AdalServiceException: AADSTS70002: Error validating credentials. AADSTS50012: Client assertion contains an invalid signature. **[Reason - The key used is expired.**, Thumbprint of key used by client: 'C49C25705D60569884EDC91986CEF8A01A495783', Found key 'Start=08/03/2016, End=08/03/2017, Thumbprint=C39C25705D60569884EDC91986CEF8A01A4956D1', Configured keys: [Key0:Start=08/03/2016, End=08/03/2017, Thumbprint=C39C25705D60569884EDC91986CEF8A01A4956D1;]]
    Trace ID: e4d34f1c-a584-47f5-884e-1235026d5000
    Correlation ID: a44d870e-6f23-405a-8b23-9b44aebfa4bb
    Timestamp: 2017-10-06 20:44:56Z ---> System.Net.WebException: The remote server returned an error: (401) Unauthorized.
    at System.Net.HttpWebRequest.GetResponse()
    at Microsoft.IdentityModel.Clients.ActiveDirectory.HttpWebRequestWrapper.<GetResponseSyncOrAsync>d__2.MoveNext()
    ```

1. Toutes les autres erreurs liées à des erreurs/un certificat Azure Active Directory peuvent être reconnues en exécutant une commande ping sur l’URL de la passerelle pour récupérer le jeton OAuth.

1. Si vous obtenez l’erreur suivante lors d’une tentative d’accès à ADLS à partir du cluster HDI. Vérifiez si le certificat a expiré en suivant les étapes mentionnées ci-dessus.

    ```
    Error: java.lang.IllegalArgumentException: Token Refresh failed - Received invalid http response: 500, text = Response{protocol=http/1.1, code=500, message=Internal Server Error, url=http://clustername.hmssomerandomstringc.cx.internal.cloudapp.net:909/api/oauthtoken}
    ```

### <a name="resolution"></a>Résolution

Créez un nouveau certificat ou attribuez un certificat existant à l’aide du script PowerShell suivant :

```powershell
$clusterName = 'CLUSTERNAME'
$resourceGroupName = 'RGNAME'
$subscriptionId = 'SUBSCRIPTIONID'
$appId = 'APPLICATIONID'
$generateSelfSignedCert = $false
$addNewCertKeyCredential = $true
$certFilePath = 'NEW_CERT_PFX_LOCAL_PATH'
$certPassword = Read-Host "Enter Certificate Password"

if($generateSelfSignedCert)
{
    Write-Host "Generating new SelfSigned certificate"
    
    $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=hdinsightAdlsCert" -KeySpec KeyExchange
    $certBytes = $cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword);
    $certString = [System.Convert]::ToBase64String($certBytes)
}
else
{

    Write-Host "Reading the cert file from path $certFilePath"

    $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($certFilePath, $certPassword)
    $certString = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes($certFilePath))
}


Login-AzureRmAccount

if($addNewCertKeyCredential)
{
    Write-Host "Creating new KeyCredential for the app"

    $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())
    
    New-AzureRmADAppCredential -ApplicationId $appId -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore

    Write-Host "Waiting for 30 seconds for the permissions to get propagated"

    Start-Sleep -s 30
}

Select-AzureRmSubscription -SubscriptionId $subscriptionId

Write-Host "Updating the certificate on HDInsight cluster."

Invoke-AzureRmResourceAction `
    -ResourceGroupName $resourceGroupName `
    -ResourceType 'Microsoft.HDInsight/clusters' `
    -ResourceName $clusterName `
    -ApiVersion '2015-03-01-preview' `
    -Action 'updateclusteridentitycertificate' `
    -Parameters @{ ApplicationId = $appId.ToString(); Certificate = $certString; CertificatePassword = $certPassword.ToString() } `
    -Force

```

Pour attribuer un certificat existant, créer un certificat, préparez le fichier .pfx et le mot de passe. Associez le certificat au principal du service avec lequel le cluster a été créé et préparez l’AppId.

Exécutez la commande PowerShell après avoir remplacé les paramètres par les valeurs réelles.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]