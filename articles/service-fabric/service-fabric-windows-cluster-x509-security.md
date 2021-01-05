---
title: Sécuriser un cluster sur Windows à l’aide de certificats
description: Sécuriser les communications au sein d’un cluster local ou autonome Azure Service Fabric, ainsi qu’entre les clients et le cluster.
ms.topic: conceptual
ms.date: 10/15/2017
ms.openlocfilehash: 34ba457ce0f39705393962d5c5ec8fa11668f413
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94686121"
---
# <a name="secure-a-standalone-cluster-on-windows-by-using-x509-certificates"></a>Sécuriser un cluster autonome sur Windows à l’aide de certificats X.509
Cet article vous explique comment sécuriser la communication entre les différents nœuds de votre cluster Windows autonome. Il décrit également comment authentifier les clients qui se connectent à ce cluster à l’aide de certificats X.509. L’authentification garantit que seuls les utilisateurs autorisés peuvent accéder au cluster et aux applications déployées, et effectuer des tâches de gestion. La sécurité par certificat doit être activée sur le cluster lors de sa création.  

Pour en savoir plus sur la sécurité des clusters, telle que la sécurité nœud à nœud, la sécurité client à nœud et le contrôle d’accès en fonction du rôle, consultez [Scénarios relatifs à la sécurité des clusters](service-fabric-cluster-security.md).

## <a name="which-certificates-do-you-need"></a>De quels certificats avez-vous besoin ?
Pour commencer, [téléchargez le package Service Fabric pour Windows Server](service-fabric-cluster-creation-for-windows-server.md#download-the-service-fabric-for-windows-server-package) sur l’un des nœuds de votre cluster. Dans le package téléchargé se trouve un fichier ClusterConfig.X509.MultiMachine.json. Ouvrez le fichier et consultez la section relative à la sécurité dans la section Propriétés :

```JSON
"security": {
    "metadata": "The Credential type X509 indicates this cluster is secured by using X509 certificates. The thumbprint format is d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
    "ClusterCredentialType": "X509",
    "ServerCredentialType": "X509",
    "CertificateInformation": {
        "ClusterCertificate": {
            "Thumbprint": "[Thumbprint]",
            "ThumbprintSecondary": "[Thumbprint]",
            "X509StoreName": "My"
        },        
        "ClusterCertificateCommonNames": {
            "CommonNames": [
            {
                "CertificateCommonName": "[CertificateCommonName]",
                "CertificateIssuerThumbprint": "[Thumbprint1,Thumbprint2,Thumbprint3,...]"
            }
            ],
            "X509StoreName": "My"
        },
        "ClusterCertificateIssuerStores": [
            {
                "IssuerCommonName": "[IssuerCommonName]",
                "X509StoreNames" : "Root"
            }
        ],
        "ServerCertificate": {
            "Thumbprint": "[Thumbprint]",
            "ThumbprintSecondary": "[Thumbprint]",
            "X509StoreName": "My"
        },
        "ServerCertificateCommonNames": {
            "CommonNames": [
            {
                "CertificateCommonName": "[CertificateCommonName]",
                "CertificateIssuerThumbprint": "[Thumbprint1,Thumbprint2,Thumbprint3,...]"
            }
            ],
            "X509StoreName": "My"
        },
        "ServerCertificateIssuerStores": [
            {
                "IssuerCommonName": "[IssuerCommonName]",
                "X509StoreNames" : "Root"
            }
        ],
        "ClientCertificateThumbprints": [
            {
                "CertificateThumbprint": "[Thumbprint]",
                "IsAdmin": false
            },
            {
                "CertificateThumbprint": "[Thumbprint]",
                "IsAdmin": true
            }
        ],
        "ClientCertificateCommonNames": [
            {
                "CertificateCommonName": "[CertificateCommonName]",
                "CertificateIssuerThumbprint": "[Thumbprint1,Thumbprint2,Thumbprint3,...]",
                "IsAdmin": true
            }
        ],
        "ClientCertificateIssuerStores": [
            {
                "IssuerCommonName": "[IssuerCommonName]",
                "X509StoreNames": "Root"
            }
        ]
        "ReverseProxyCertificate": {
            "Thumbprint": "[Thumbprint]",
            "ThumbprintSecondary": "[Thumbprint]",
            "X509StoreName": "My"
        },
        "ReverseProxyCertificateCommonNames": {
            "CommonNames": [
                {
                "CertificateCommonName": "[CertificateCommonName]"
                }
            ],
            "X509StoreName": "My"
        }
    }
},
```

Cette section décrit les certificats requis pour sécuriser votre cluster Windows autonome. Si vous spécifiez un certificat de cluster, affectez à ClusterCredentialType la valeur _X509_. Si vous spécifiez un certificat de serveur pour des connexions externes, donnez à ServerCredentialType la valeur _X509_. Bien que ce ne soit pas obligatoire, nous vous recommandons d’avoir ces deux certificats pour que votre cluster soit correctement sécurisé. Si vous définissez ces valeurs sur *X509*, vous devez également spécifier les certificats correspondants, sinon, Service Fabric lèvera une exception. Dans certains scénarios, vous souhaiterez peut-être ne spécifier que _ClientCertificateThumbprints_ ou _ReverseProxyCertificate_. Dans ces scénarios, vous n’avez pas besoin de définir _ClusterCredentialType_ ou _ServerCredentialType_ sur _X509_.


> [!NOTE]
> Un [Thumbprint](https://en.wikipedia.org/wiki/Public_key_fingerprint) est l’identité principale d’un certificat. Lisez la section [Retrieve a thumbprint of a certificate](/dotnet/framework/wcf/feature-details/how-to-retrieve-the-thumbprint-of-a-certificate) (Récupérer l’empreinte numérique d’un certificat) pour déterminer l’empreinte des certificats que vous créez.
> 
> 

Le tableau suivant répertorie les certificats dont vous aurez besoin pour la configuration de votre cluster :

| **Paramètre CertificateInformation** | **Description** |
| --- | --- |
| ClusterCertificate |Recommandé pour un environnement de test. Ce certificat est requis pour sécuriser les communications entre les nœuds sur un cluster. Vous pouvez utiliser deux certificats différents : un certificat principal et un certificat secondaire pour la mise à niveau. Définissez l’empreinte du certificat principal dans la section Thumbprint, et celui du certificat secondaire dans les variables ThumbprintSecondary. |
| ClusterCertificateCommonNames |Recommandé pour un environnement de production. Ce certificat est requis pour sécuriser les communications entre les nœuds sur un cluster. Vous pouvez utiliser un ou deux noms communs de certificat de cluster. CertificateIssuerThumbprint correspond à l’empreinte de l’émetteur de ce certificat. Si plusieurs certificats portant le même nom sont utilisés, vous pouvez spécifier plusieurs empreintes d’émetteur.|
| ClusterCertificateIssuerStores |Recommandé pour un environnement de production. Ce certificat correspond à l’émetteur du certificat de cluster. Au lieu de spécifier l’empreinte numérique de l’émetteur sous ClusterCertificateCommonNames, vous pouvez fournir le nom commun de l’émetteur et le nom de magasin correspondant sous cette section.  Cela facile la substitution des certificats d’émetteur de cluster. Plusieurs émetteurs peuvent être spécifiés si plus d’un certificat de cluster est utilisé. Une valeur IssuerCommonName vide a pour effet d’autoriser tous les certificats dans les magasins correspondants spécifiés sous X509StoreNames.|
| ServerCertificate |Recommandé pour un environnement de test. Ce certificat est présenté au client lorsqu’il tente de se connecter à ce cluster. Pour plus de commodité, vous pouvez choisir d’utiliser le même certificat pour les éléments ClusterCertificate et ServerCertificate. Vous pouvez utiliser deux certificats de serveur différents : un certificat principal et un certificat secondaire pour la mise à niveau. Définissez l’empreinte du certificat principal dans la section Thumbprint, et celui du certificat secondaire dans les variables ThumbprintSecondary. |
| ServerCertificateCommonNames |Recommandé pour un environnement de production. Ce certificat est présenté au client lorsqu’il tente de se connecter à ce cluster. CertificateIssuerThumbprint correspond à l’empreinte de l’émetteur de ce certificat. Si plusieurs certificats portant le même nom sont utilisés, vous pouvez spécifier plusieurs empreintes d’émetteur. Pour plus de commodité, vous pouvez choisir d’utiliser le même certificat pour les éléments ClusterCertificateCommonNames et ServerCertificateCommonNames. Vous pouvez utiliser un ou deux noms communs de certificat de serveur. |
| ServerCertificateIssuerStores |Recommandé pour un environnement de production. Ce certificat correspond à l’émetteur du certificat de serveur. Au lieu de spécifier l’empreinte numérique de l’émetteur sous ServerCertificateCommonNames, vous pouvez fournir le nom commun de l’émetteur et le nom de magasin correspondant sous cette section.  Cela facile la substitution des certificats d’émetteur de serveur. Plusieurs émetteurs peuvent être spécifiés si plus d’un certificat de serveur est utilisé. Une valeur IssuerCommonName vide a pour effet d’autoriser tous les certificats dans les magasins correspondants spécifiés sous X509StoreNames.|
| ClientCertificateThumbprints |Installez ce jeu de certificats sur les clients authentifiés. Il peut y avoir plusieurs certificats clients installés sur les machines qui seront autorisées à accéder au cluster. Définissez l’empreinte de chaque certificat dans la variable CertificateThumbprint. Si vous affectez la valeur *true* à IsAdmin, le client sur lequel ce certificat est installé peut effectuer des activités de gestion d’administrateur sur le cluster. Si IsAdmin est défini sur *false*, le client ayant ce certificat peut effectuer les actions uniquement autorisées pour les droits d’accès utilisateur, généralement en lecture seule. Pour plus d’informations sur les rôles, consultez [Contrôle d’accès en fonction du rôle Service Fabric](service-fabric-cluster-security.md#service-fabric-role-based-access-control). |
| ClientCertificateCommonNames |Définissez le nom commun du premier certificat client pour CertificateCommonName. L’élément CertificateIssuerThumbprint est l’empreinte numérique relative à l’émetteur de ce certificat. Pour en savoir plus sur les noms communs et l’émetteur, consultez la section [Work with certificates](/dotnet/framework/wcf/feature-details/working-with-certificates) (Utilisation des certificats). |
| ClientCertificateIssuerStores |Recommandé pour un environnement de production. Ce certificat correspond à l’émetteur du certificat client (rôles administrateur et non-administrateur). Au lieu de spécifier l’empreinte numérique de l’émetteur sous ClientCertificateCommonNames, vous pouvez fournir le nom commun de l’émetteur et le nom de magasin correspondant sous cette section.  Cela facile la substitution des certificats d’émetteur de client. Plusieurs émetteurs peuvent être spécifiés si plus d’un certificat client est utilisé. Une valeur IssuerCommonName vide a pour effet d’autoriser tous les certificats dans les magasins correspondants spécifiés sous X509StoreNames.|
| ReverseProxyCertificate |Recommandé pour un environnement de test. Ce certificat facultatif peut être spécifié si vous souhaitez sécuriser votre [proxy inversé](service-fabric-reverseproxy.md). Assurez-vous que reverseProxyEndpointPort est défini dans nodeTypes, si vous utilisez ce certificat. |
| ReverseProxyCertificateCommonNames |Recommandé pour un environnement de production. Ce certificat facultatif peut être spécifié si vous souhaitez sécuriser votre [proxy inversé](service-fabric-reverseproxy.md). Assurez-vous que reverseProxyEndpointPort est défini dans nodeTypes, si vous utilisez ce certificat. |

Voici un exemple de configuration de cluster dans lequel les certificats de client, de serveur et de cluster ont été fournis. Pour les certificats cluster/serveur/reverseProxy, l’empreinte et le nom commun ne peuvent pas être configurés conjointement pour le même type de certificat.

 ```JSON
 {
    "name": "SampleCluster",
    "clusterConfigurationVersion": "1.0.0",
    "apiVersion": "10-2017",
    "nodes": [{
        "nodeName": "vm0",
        "metadata": "Replace the localhost below with valid IP address or FQDN",
        "iPAddress": "10.7.0.5",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r0",
        "upgradeDomain": "UD0"
    }, {
        "nodeName": "vm1",
        "metadata": "Replace the localhost with valid IP address or FQDN",
        "iPAddress": "10.7.0.4",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r1",
        "upgradeDomain": "UD1"
    }, {
        "nodeName": "vm2",
        "iPAddress": "10.7.0.6",
        "metadata": "Replace the localhost with valid IP address or FQDN",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r2",
        "upgradeDomain": "UD2"
    }],
    "properties": {
        "diagnosticsStore": {
        "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
        "dataDeletionAgeInDays": "7",
        "storeType": "FileShare",
        "IsEncrypted": "false",
        "connectionstring": "c:\\ProgramData\\SF\\DiagnosticsStore"
        },
        "security": {
            "metadata": "The Credential type X509 indicates this cluster is secured by using X509 certificates. The thumbprint format is d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
            "ClusterCredentialType": "X509",
            "ServerCredentialType": "X509",
            "CertificateInformation": {
                "ClusterCertificateCommonNames": {
                  "CommonNames": [
                    {
                      "CertificateCommonName": "myClusterCertCommonName"
                    }
                  ],
                  "X509StoreName": "My"
                },
                "ClusterCertificateIssuerStores": [
                    {
                        "IssuerCommonName": "ClusterIssuer1",
                        "X509StoreNames" : "Root"
                    },
                    {
                        "IssuerCommonName": "ClusterIssuer2",
                        "X509StoreNames" : "Root"
                    }
                ],
                "ServerCertificateCommonNames": {
                  "CommonNames": [
                    {
                      "CertificateCommonName": "myServerCertCommonName",
                      "CertificateIssuerThumbprint": "7c fc 91 97 13 16 8d ff a8 ee 71 2b a2 f4 62 62 00 03 49 0d"
                    }
                  ],
                  "X509StoreName": "My"
                },
                "ClientCertificateThumbprints": [{
                    "CertificateThumbprint": "c4 c18 8e aa a8 58 77 98 65 f8 61 4a 0d da 4c 13 c5 a1 37 6e",
                    "IsAdmin": false
                }, {
                    "CertificateThumbprint": "71 de 04 46 7c 9e d0 54 4d 02 10 98 bc d4 4c 71 e1 83 41 4e",
                    "IsAdmin": true
                }]
            }
        },
        "reliabilityLevel": "Bronze",
        "nodeTypes": [{
            "name": "NodeType0",
            "clientConnectionEndpointPort": "19000",
            "clusterConnectionEndpointPort": "19001",
            "leaseDriverEndpointPort": "19002",
            "serviceConnectionEndpointPort": "19003",
            "httpGatewayEndpointPort": "19080",
            "applicationPorts": {
                "startPort": "20001",
                "endPort": "20031"
            },
            "ephemeralPorts": {
                "startPort": "20032",
                "endPort": "20062"
            },
            "isPrimary": true
        }
         ],
        "fabricSettings": [{
            "name": "Setup",
            "parameters": [{
                "name": "FabricDataRoot",
                "value": "C:\\ProgramData\\SF"
            }, {
                "name": "FabricLogRoot",
                "value": "C:\\ProgramData\\SF\\Log"
            }]
        }]
    }
}
 ```

## <a name="certificate-rollover"></a>Substitution de certificat
Lorsque vous utilisez le nom commun du certificat au lieu de l’empreinte, la substitution de certificat ne requiert aucune mise à niveau de configuration de cluster. Pour les mises à niveau d’empreintes d’émetteur, vérifiez que la nouvelle liste d’empreintes a des éléments communs avec l’ancienne. Vous devez tout d’abord effectuer une mise à niveau de la configuration avec les nouvelles empreintes d’émetteur, puis installer les nouveaux certificats (certificat de serveur/cluster et certificats d’émetteur) dans le magasin. Conservez l’ancien certificat de l’émetteur dans le magasin de certificats au moins deux heures après avoir installé le nouveau certificat d’émetteur.
Si vous utilisez des magasins d’émetteur, la substitution du certificat d’émetteur ne nécessite aucune mise à niveau de configuration. Installez le nouveau certificat d’émetteur avec une date d’expiration plus récente dans le magasin de certificats correspondant, puis supprimez l’ancien certificat d’émetteur après quelques heures.

## <a name="acquire-the-x509-certificates"></a>Acquérir des certificats X.509
Pour sécuriser les communications à l’intérieur du cluster, vous devez d’abord obtenir des certificats X.509 pour vos nœuds de cluster. En outre, pour limiter les connexions à ce cluster aux ordinateurs/utilisateurs autorisés, vous devez obtenir et installer des certificats pour les ordinateurs clients.

Utilisez un certificat X.509 signé par une [autorité de certification](https://en.wikipedia.org/wiki/Certificate_authority) pour sécuriser les clusters exécutant des charges de travail de production. Pour plus d’informations sur la façon d’obtenir ces certificats, consultez [How to obtain a certificate](/dotnet/framework/wcf/feature-details/how-to-obtain-a-certificate-wcf) (Comment obtenir un certificat). 

Le certificat doit posséder un certain nombre de propriétés pour fonctionner correctement :

* Le fournisseur de certificat doit être **Microsoft Enhanced RSA and AES Cryptographic Provider**

* Lorsque vous créez une clé RSA, assurez-vous que la clé est de **2 048 bits**.

* La valeur de l’extension d’utilisation de la clé est **signature numérique, chiffrement de la clé (a0)**

* Les valeurs d’extension d’utilisation avancée de la clé sont **l’authentification du serveur** (OID : 1.3.6.1.5.5.7.3.1) et **l’authentification du client** (OID : 1.3.6.1.5.5.7.3.2)

Pour les clusters que vous utilisez à des fins de test, vous pouvez choisir d’utiliser un certificat auto-signé.

Si vous avez d’autres questions, consultez les [questions fréquemment posées sur les certificats](./cluster-security-certificate-management.md#troubleshooting-and-frequently-asked-questions).

## <a name="optional-create-a-self-signed-certificate"></a>Facultatif : Créer un certificat auto-signé
Pour créer un certificat auto-signé qui peut être sécurisé correctement, l’une des solutions consiste à utiliser le script CertSetup.ps1 contenu dans le dossier du Kit de développement logiciel (SDK) Service Fabric dans le répertoire C:\Program Files\Microsoft SDKs\Service Fabric\ClusterSetup\Secure. Modifiez ce fichier pour changer le nom par défaut du certificat. (Recherchez la valeur CN = ServiceFabricDevClusterCert.) Exécutez ce script en tant que `.\CertSetup.ps1 -Install`.

Maintenant, exportez le certificat vers un fichier .pfx avec un mot de passe protégé. Tout d’abord, récupérez l’empreinte du certificat. 
1. Dans le menu **Démarrer**, exécutez **Gérer les certificats d’ordinateur**. 

2. Accédez au dossier **Ordinateur local\Personnel** et recherchez le certificat que vous avez créé. 

3. Double-cliquez sur le certificat pour l’ouvrir, sélectionnez l’onglet **Détails** et faites défiler jusqu’au champ **Thumbprint**. 

4. Supprimez les espaces et copiez la valeur d’empreinte dans la commande PowerShell ci-dessous. 

5. Remplacez la valeur `String` par un mot de passe suffisamment sécurisé et exécutez les commandes suivantes dans PowerShell :

   ```powershell   
   $pswd = ConvertTo-SecureString -String "1234" -Force –AsPlainText
   Get-ChildItem -Path cert:\localMachine\my\<Thumbprint> | Export-PfxCertificate -FilePath C:\mypfx.pfx -Password $pswd
   ```

6. Pour afficher les détails d’un certificat installé sur l’ordinateur, exécutez la commande PowerShell suivante :

   ```powershell
   $cert = Get-Item Cert:\LocalMachine\My\<Thumbprint>
   Write-Host $cert.ToString($true)
   ```

Si vous disposez d’un abonnement Azure, vous pouvez également suivre les étapes de [Créer un cluster Service Fabric à l’aide d’Azure Resource Manager](service-fabric-cluster-creation-via-arm.md).

## <a name="install-the-certificates"></a>Installer les certificats
Une fois que vous avez les certificats, vous pouvez les installer sur les nœuds du cluster. La dernière version de Windows PowerShell 3.x doit être installée sur vos nœuds. Répétez cette procédure sur chaque nœud, pour les certificats de cluster et de serveur et chaque certificat secondaire éventuel.

1. Copiez le ou les fichiers .pfx sur le nœud.

2. Ouvrez une fenêtre PowerShell en tant qu’administrateur, puis entrez les commandes suivantes. Remplacez *$pswd* par le mot de passe que vous avez utilisé pour créer ce certificat. Remplacez *$PfxFilePath* par le chemin complet du fichier .pfx copié sur ce nœud.
   
    ```powershell
    $pswd = "1234"
    $PfxFilePath ="C:\mypfx.pfx"
    Import-PfxCertificate -Exportable -CertStoreLocation Cert:\LocalMachine\My -FilePath $PfxFilePath -Password (ConvertTo-SecureString -String $pswd -AsPlainText -Force)
    ```
3. Maintenant, définissez le contrôle d’accès sur ce certificat afin que le processus Service Fabric, qui s’exécute sous le compte Service réseau, puisse l’utiliser en exécutant le script suivant. Spécifiez l’empreinte du certificat et **SERVICE RÉSEAU** comme compte de service. Vous pouvez vérifier que les ACL du certificat sont correctes en ouvrant le certificat dans **Démarrer** > **Gérer les certificats d’ordinateur** et en examinant **Toutes les tâches** > **Gérer les clés privées**.
   
    ```powershell
    param
    (
    [Parameter(Position=1, Mandatory=$true)]
    [ValidateNotNullOrEmpty()]
    [string]$pfxThumbPrint,
   
    [Parameter(Position=2, Mandatory=$true)]
    [ValidateNotNullOrEmpty()]
    [string]$serviceAccount
    )
   
    $cert = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object -FilterScript { $PSItem.ThumbPrint -eq $pfxThumbPrint; }
   
    # Specify the user, the permissions, and the permission type
    $permission = "$($serviceAccount)","FullControl","Allow" # "NT AUTHORITY\NetworkService" is the service account
    $accessRule = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $permission
   
    # Location of the machine-related keys
    $keyPath = Join-Path -Path $env:ProgramData -ChildPath "\Microsoft\Crypto\RSA\MachineKeys"
    $keyName = $cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName
    $keyFullPath = Join-Path -Path $keyPath -ChildPath $keyName
   
    # Get the current ACL of the private key
    $acl = (Get-Item $keyFullPath).GetAccessControl('Access')
   
    # Add the new ACE to the ACL of the private key
    $acl.SetAccessRule($accessRule)
   
    # Write back the new ACL
    Set-Acl -Path $keyFullPath -AclObject $acl -ErrorAction Stop
   
    # Observe the access rights currently assigned to this certificate
    get-acl $keyFullPath| fl
    ```
4. Répétez les étapes précédentes pour chaque certificat de serveur. Vous pouvez également utiliser ces étapes pour installer les certificats clients sur les ordinateurs qui seront autorisés à accéder au cluster.

## <a name="create-the-secure-cluster"></a>Créer le cluster sécurisé
Après avoir configuré la section relative à la sécurité du fichier ClusterConfig.X509.MultiMachine.json, vous pouvez passer à la section [Création du cluster](service-fabric-cluster-creation-for-windows-server.md#create-the-cluster) pour configurer les nœuds et créer le cluster autonome. N’oubliez pas d’utiliser le fichier ClusterConfig.X509.MultiMachine.json lors de la création du cluster. Voici un exemple de commande :

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.X509.MultiMachine.json
```

Une fois que le cluster Windows autonome sécurisé s’exécute correctement et que vous avez configuré les clients authentifiés pour qu’ils s’y connectent, suivez les étapes de la section [Se connecter à un cluster à l’aide de PowerShell](service-fabric-connect-to-secure-cluster.md#connect-to-a-cluster-using-powershell) pour vous y connecter. Par exemple :

```powershell
$ConnectArgs = @{  ConnectionEndpoint = '10.7.0.5:19000';  X509Credential = $True;  StoreLocation = 'LocalMachine';  StoreName = "MY";  ServerCertThumbprint = "057b9544a6f2733e0c8d3a60013a58948213f551";  FindType = 'FindByThumbprint';  FindValue = "057b9544a6f2733e0c8d3a60013a58948213f551"   }
Connect-ServiceFabricCluster $ConnectArgs
```

Vous pouvez ensuite exécuter d’autres commandes PowerShell pour travailler avec ce cluster. Par exemple, vous pouvez exécuter [Get-ServiceFabricNode](/powershell/module/servicefabric/get-servicefabricnode?view=azureservicefabricps) pour afficher la liste des nœuds sur ce cluster sécurisé.


Pour supprimer le cluster, connectez-vous au nœud du cluster dans lequel vous avez téléchargé le package Service Fabric, ouvrez une ligne de commande et accédez au dossier du package. Exécutez à présent la commande suivante :

```powershell
.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.X509.MultiMachine.json
```

> [!NOTE]
> Une configuration incorrecte des certificats peut empêcher l’affichage du cluster pendant le déploiement. Pour diagnostiquer les problèmes de sécurité, ouvrez le groupe de l’Observateur d’événements **Journaux des applications et des services** > **Microsoft-Service Fabric**.
> 
> 
