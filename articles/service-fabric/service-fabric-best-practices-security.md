---
title: Bonnes pratiques pour la sécurité Azure Service Fabric
description: Bonnes pratiques et considérations de conception pour garder les clusters et applications Azure Service Fabric sécurisés.
author: peterpogorski
ms.topic: conceptual
ms.date: 01/23/2019
ms.author: pepogors
ms.openlocfilehash: 90ffd1c01411982f56aed3332c499aa0c10b8a94
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86257605"
---
# <a name="azure-service-fabric-security"></a>Sécurité de Microsoft Azure Service Fabric 

Pour plus d’informations sur les [bonnes pratiques relatives à la sécurité d’Azure](../security/index.yml), passez en revue [Bonnes pratiques pour la sécurité Azure Service Fabric](../security/fundamentals/service-fabric-best-practices.md)

## <a name="key-vault"></a>Key Vault

[Azure Key Vault](../key-vault/index.yml) est le service de gestion des secrets recommandé pour les applications et les clusters Azure Service Fabric.
> [!NOTE]
> Si les certificats/secrets d’un coffre de clés sont déployés sur un groupe de machines virtuelles identiques en tant que secret d’un groupe de machines virtuelles identiques, le coffre de clés et le groupe de machines virtuelles identiques doivent être colocalisés.

## <a name="create-certificate-authority-issued-service-fabric-certificate"></a>Créer un certificat Service Fabric émis par une autorité de certification

Un certificat Azure Key Vault peut être créé ou importé dans un coffre de clés. Lors de la création d’un certificat Key Vault, la clé privée est créée dans le coffre de clés sans jamais être exposée au propriétaire du certificat. Voici les différentes façons de créer un certificat dans Key Vault :

- Créez un certificat auto-signé pour créer une paire de clés publique-privée et l’associer à un certificat. Le certificat est signé par sa propre clé. 
- Créez un certificat manuellement pour créer une paire de clés publique-privée et générer une demande de signature de certificat X.509. La demande de signature peut être signée par votre autorité de certification ou autorité d’inscription. Le certificat X509 signé peut être fusionné avec la paire de clés en attente afin de terminer la création du certificat KV dans Key Vault. Bien que cette méthode nécessite davantage d’étapes, elle offre une plus grande sécurité, car la clé privée est créée dans Key Vault et restreinte à ce dernier. Ceci est expliqué dans le diagramme ci-dessous. 

Pour plus d’informations, consultez [Méthodes de création de certificats Azure Key Vault ](../key-vault/certificates/create-certificate.md).

## <a name="deploy-key-vault-certificates-to-service-fabric-cluster-virtual-machine-scale-sets"></a>Déployer des certificats Key Vault sur les groupes de machines virtuelles identiques d’un cluster Service Fabric

Pour déployer des certificats d’un coffre de clés colocalisé sur un groupe de machines virtuelles identiques, utilisez le paramètre [osProfile](/rest/api/compute/virtualmachinescalesets/createorupdate#virtualmachinescalesetosprofile) du groupe de machines virtuelles identiques. Les propriétés du modèle Resource Manager sont les suivantes :

```json
"secrets": [
   {
       "sourceVault": {
           "id": "[parameters('sourceVaultValue')]"
       },
       "vaultCertificates": [
          {
              "certificateStore": "[parameters('certificateStoreValue')]",
              "certificateUrl": "[parameters('certificateUrlValue')]"
          }
       ]
   }
]
```

> [!NOTE]
> Le coffre doit être activé pour le déploiement du modèle Resource Manager.

## <a name="apply-an-access-control-list-acl-to-your-certificate-for-your-service-fabric-cluster"></a>Appliquer une liste de contrôle d’accès (ACL) à votre certificat pour votre cluster Service Fabric

Le serveur de publication des [extensions de groupe de machines virtuelles identiques](/cli/azure/vmss/extension?view=azure-cli-latest) Microsoft.Azure.ServiceFabric est utilisé pour configurer la sécurité de vos nœuds.
Pour appliquer une ACL à vos certificats pour les processus de votre cluster Service Fabric, utilisez les propriétés du modèle Resource Manager suivantes :

```json
"certificate": {
   "commonNames": [
       "[parameters('certificateCommonName')]"
   ],
   "x509StoreName": "[parameters('certificateStoreValue')]"
}
```

## <a name="secure-a-service-fabric-cluster-certificate-by-common-name"></a>Sécuriser un certificat de cluster Service Fabric par nom commun

Pour sécuriser votre cluster Service Fabric par `Common Name` de certificat, utilisez la propriété de modèle Resource Manager [certificateCommonNames](/rest/api/servicefabric/sfrp-model-clusterproperties#certificatecommonnames) comme ceci :

```json
"certificateCommonNames": {
    "commonNames": [
        {
            "certificateCommonName": "[parameters('certificateCommonName')]",
            "certificateIssuerThumbprint": "[parameters('certificateIssuerThumbprint')]"
        }
    ],
    "x509StoreName": "[parameters('certificateStoreValue')]"
}
```

> [!NOTE]
> Les clusters Service Fabric utilisent le premier certificat valide trouvé dans le magasin de certificats de votre hôte. Sur Windows, il s’agit du certificat dont la date d’expiration la plus récente correspond à votre nom commun et à votre empreinte d’émetteur.

Les domaines Azure, tels que *\<YOUR SUBDOMAIN\>.cloudapp.azure.com ou \<YOUR SUBDOMAIN\>.trafficmanager.net, appartiennent à Microsoft. Les autorités de certification n’émettent pas de certificats pour les domaines à des utilisateurs non autorisés. Pour qu’une autorité de certification émette un certificat portant ce nom commun, la plupart des utilisateurs doivent acheter un domaine auprès d’un bureau d’enregistrement ou disposer d’une autorisation d’administrateur de domaine.

Pour plus d’informations sur la façon de configurer le service DNS afin de résoudre votre domaine en adresse IP Microsoft, examinez comment configurer [Azure DNS pour héberger votre domaine](../dns/dns-delegate-domain-azure-dns.md).

> [!NOTE]
> Après avoir délégué vos serveurs de noms de domaine à vos serveurs de noms de zone Azure DNS, ajoutez les deux enregistrements suivants à votre zone DNS :
> - Un enregistrement « A » pour l’APEX de domaine qui N’EST PAS un `Alias record set` pour l’ensemble des adresses IP résolues par votre domaine personnalisé.
> - Un enregistrement « C » pour les sous-domaines Microsoft que vous avez provisionnés qui NE SONT PAS un `Alias record set`. Par exemple, vous pouvez utiliser le nom DNS de votre service Traffic Manager ou Load Balancer.

Pour mettre à jour votre portail afin d’afficher un nom DNS personnalisé pour le `"managementEndpoint"` de votre cluster Service Fabric, mettez à jour les propriétés du modèle Resource Manager du cluster :

```json
 "managementEndpoint": "[concat('https://<YOUR CUSTOM DOMAIN>:',parameters('nt0fabricHttpGatewayPort'))]",
```

## <a name="encrypting-service-fabric-package-secret-values"></a>Chiffrement des valeurs des secrets des packages Service Fabric

Les valeurs courantes chiffrées dans les packages Service Fabric sont, entre autres, les informations d’identification d’Azure Container Registry (ACR), les variables d’environnement, les paramètres et les clés de compte de stockage du plug-in Volume d’Azure.

Pour [configurer un certificat de chiffrement et chiffrer des secrets sur des clusters Windows](./service-fabric-application-secret-management-windows.md) :

Générez un certificat auto-signé pour chiffrer votre secret :

```powershell
New-SelfSignedCertificate -Type DocumentEncryptionCert -KeyUsage DataEncipherment -Subject mydataenciphermentcert -Provider 'Microsoft Enhanced Cryptographic Provider v1.0'
```

Suivez les instructions pour [déployer des certificats Key Vault sur les groupes de machines virtuelles identiques de votre cluster Service Fabric](#deploy-key-vault-certificates-to-service-fabric-cluster-virtual-machine-scale-sets).

Chiffrez votre secret à l’aide de la commande PowerShell suivante, puis mettez à jour votre manifeste d’application Service Fabric avec la valeur chiffrée :

``` powershell
Invoke-ServiceFabricEncryptText -CertStore -CertThumbprint "<thumbprint>" -Text "mysecret" -StoreLocation CurrentUser -StoreName My
```

Pour [configurer un certificat de chiffrement et chiffrer des secrets sur des clusters Linux.](./service-fabric-application-secret-management-linux.md) :

Générez un certificat auto-signé pour chiffrer vos secrets :

```bash
user@linux:~$ openssl req -newkey rsa:2048 -nodes -keyout TestCert.prv -x509 -days 365 -out TestCert.pem
user@linux:~$ cat TestCert.prv >> TestCert.pem
```

Suivez les instructions pour [déployer des certificats Key Vault sur les groupes de machines virtuelles identiques de votre cluster Service Fabric](#deploy-key-vault-certificates-to-service-fabric-cluster-virtual-machine-scale-sets).

Chiffrez votre secret à l’aide des commandes suivantes, puis mettez à jour votre manifeste d’application Service Fabric avec la valeur chiffrée :

```bash
user@linux:$ echo "Hello World!" > plaintext.txt
user@linux:$ iconv -f ASCII -t UTF-16LE plaintext.txt -o plaintext_UTF-16.txt
user@linux:$ openssl smime -encrypt -in plaintext_UTF-16.txt -binary -outform der TestCert.pem | base64 > encrypted.txt
```

Après avoir chiffré vos valeurs protégées, [spécifiez les secrets chiffrés dans l’application Service Fabric](./service-fabric-application-secret-management.md#specify-encrypted-secrets-in-an-application), et [déchiffrez les secrets chiffrés à partir du code de service](./service-fabric-application-secret-management.md#decrypt-encrypted-secrets-from-service-code).

## <a name="include-certificate-in-service-fabric-applications"></a>Inclure un certificat dans les applications Service Fabric

Pour permettre à votre application d’accéder aux secrets, incluez le certificat en ajoutant un élément **SecretsCertificate** au manifeste de l’application.

```xml
<ApplicationManifest … >
  ...
  <Certificates>
    <SecretsCertificate Name="MyCert" X509FindType="FindByThumbprint" X509FindValue="[YourCertThumbrint]"/>
  </Certificates>
</ApplicationManifest>
```
## <a name="authenticate-service-fabric-applications-to-azure-resources-using-managed-service-identity-msi"></a>Authentifier les applications Service Fabric auprès de ressources Azure à l’aide de MSI (Managed Service Identity)

Pour en savoir plus sur les identités managées pour les ressources Azure, consultez [Que sont les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/overview.md)
Les clusters Azure Service Fabric sont hébergés sur des groupes de machines virtuelles identiques qui prennent en charge [Managed Service Identity](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-managed-identities-for-azure-resources).
Pour obtenir la liste des services avec lesquels vous pouvez utiliser MSI pour l’authentification, consultez [Services Azure prenant en charge l’authentification Azure Active Directory](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).


Pour activer une identité managée affectée par le système pendant la création d’un groupe de machines virtuelles identiques ou dans un groupe de machines virtuelles identiques existant, déclarez la propriété `"Microsoft.Compute/virtualMachinesScaleSets"` suivante :

```json
"identity": { 
    "type": "SystemAssigned"
}
```
Pour plus d’informations, consultez [Que sont les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vmss.md#system-assigned-managed-identity)

Si vous avez créé une [identité managée affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-arm.md#create-a-user-assigned-managed-identity), déclarez la ressource suivante dans votre modèle pour l’affecter à votre groupe de machines virtuelles identiques. Remplacez `\<USERASSIGNEDIDENTITYNAME\>` par le nom de l’identité managée affectée par l’utilisateur que vous avez créée :

```json
"identity": {
    "type": "userAssigned",
    "userAssignedIdentities": {
        "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]": {}
    }
}
```

Pour que votre application Service Fabric puisse utiliser une identité managée, vous devez au préalable accorder des autorisations aux ressources Azure dont elle a besoin pour s’authentifier.
Les commandes suivantes accordent l’accès à une ressource Azure :

```bash
principalid=$(az resource show --id /subscriptions/<YOUR SUBSCRIPTON>/resourceGroups/<YOUR RG>/providers/Microsoft.Compute/virtualMachineScaleSets/<YOUR SCALE SET> --api-version 2018-06-01 | python -c "import sys, json; print(json.load(sys.stdin)['identity']['principalId'])")

az role assignment create --assignee $principalid --role 'Contributor' --scope "/subscriptions/<YOUR SUBSCRIPTION>/resourceGroups/<YOUR RG>/providers/<PROVIDER NAME>/<RESOURCE TYPE>/<RESOURCE NAME>"
```

Dans le code de votre application Service Fabric, [obtenez un jeton d’accès](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md#get-a-token-using-http) pour Azure Resource Manager à l’aide d’une REST similaire à la suivante :

```bash
access_token=$(curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true | python -c "import sys, json; print json.load(sys.stdin)['access_token']")

```

Votre application Service Fabric peut ensuite utiliser le jeton d’accès pour s’authentifier auprès de ressources Azure prenant en charge Active Directory.
L’exemple suivant montre comment faire pour une ressource Cosmos DB :

```bash
cosmos_db_password=$(curl 'https://management.azure.com/subscriptions/<YOUR SUBSCRIPTION>/resourceGroups/<YOUR RG>/providers/Microsoft.DocumentDB/databaseAccounts/<YOUR ACCOUNT>/listKeys?api-version=2016-03-31' -X POST -d "" -H "Authorization: Bearer $access_token" | python -c "import sys, json; print(json.load(sys.stdin)['primaryMasterKey'])")
```
## <a name="windows-security-baselines"></a>Lignes de base de la sécurité Windows
[Nous vous recommandons d’implémenter une configuration répondant aux normes du secteur largement connue et bien testée, telle que des lignes de base de sécurité Microsoft, plutôt que de créer une ligne de base vous-même](/windows/security/threat-protection/windows-security-baselines) ; une option pour l’approvisionnement sur vos groupes de machines virtuelles identiques consiste à utiliser le gestionnaire d’extension DSC (Desired State Configuration) pour configurer les machines virtuelles comme elles sont fournies en ligne, afin qu’elles exécutent le logiciel en production.

## <a name="azure-firewall"></a>Pare-feu Azure
[Pare-feu Azure est un service de sécurité réseau informatique managé qui protège vos ressources Réseau virtuel Azure. Il s’agit d’un service de pare-feu entièrement avec état, pourvu d’une haute disponibilité intégrée et de l’extensibilité du cloud sans limites.](../firewall/overview.md) ; cela permet de limiter le trafic HTTP/S sortant à une liste spécifiée des noms de domaine complet (FQDN), y compris les caractères génériques. Cette fonctionnalité ne nécessite pas d'arrêt TLS/SSL. Nous vous recommandons d’exploiter les [balises FQDN Pare-feu Azure](../firewall/fqdn-tags.md) pour Windows Update et pour activer le trafic réseau pour que les points de terminaison Microsoft Windows Update puissent transiter par votre pare-feu. [Déployer Pare-feu Azure à l’aide d’un modèle](../firewall/deploy-template.md) propose un exemple de définition de modèle de ressource Microsoft.Network/azureFirewalls. Les règles de pare-feu communes aux applications Service Fabric autorisent notamment les éléments suivants pour votre réseau virtuel de clusters :

- *download.microsoft.com
- *servicefabric.azure.com
- *.core.windows.net

Ces règles de pare-feu complètent vos groupes de sécurité réseau sortant autorisés, comprenant Service Fabric et Stockage, en tant que destinations autorisées depuis votre réseau virtuel.

## <a name="tls-12"></a>TLS 1.2

Microsoft [Azure recommande](https://azure.microsoft.com/updates/azuretls12/) à tous les clients d’effectuer une migration complète vers des solutions qui prennent en charge le protocole TLS (Transport Layer Security) 1.2 et de s’assurer que TLS 1.2 est utilisé par défaut.

Les services Azure, y compris [Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/microsoft-azure-service-fabric-6-3-refresh-release-cu1-notes/ba-p/791493), ont terminé le travail d’ingénierie visant à supprimer les dépendances envers les protocoles TLS 1.0/1.1, et offrent une prise en charge complète aux clients qui souhaitent que leurs charges de travail soient configurées pour accepter et établir uniquement des connexions TLS 1.2.

Les clients doivent configurer leurs charges de travail hébergées sur Azure et leurs applications locales qui interagissent avec les services Azure de sorte qu’elles utilisent TLS 1.2 par défaut. Voici comment [configurer des nœuds de cluster et des applications Service Fabric](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/Security/TLS%20Configuration.md) pour utiliser une version TLS spécifique.

## <a name="windows-defender"></a>Windows Defender 

Par défaut, l’antivirus Windows Defender est installé sur Windows Server 2016. Pour plus d’informations, consultez [Antivirus Windows Defender sur Windows Server 2016](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-on-windows-server-2016). L’interface utilisateur est installée par défaut sur certaines références SKU, mais elle n’est pas obligatoire. Pour réduire l’impact sur les performances et la consommation de ressources de Windows Defender, et si vos stratégies de sécurité vous permettent d’exclure des processus et des chemins pour les logiciels open-source, déclarez les propriétés suivantes du modèle Resource Manager de l’extension de groupe de machines virtuelles identiques pour exclure votre cluster Service Fabric des analyses :


```json
 {
    "name": "[concat('VMIaaSAntimalware','_vmNodeType0Name')]",
    "properties": {
        "publisher": "Microsoft.Azure.Security",
        "type": "IaaSAntimalware",
        "typeHandlerVersion": "1.5",
        "settings": {
            "AntimalwareEnabled": "true",
            "Exclusions": {
                "Paths": "[concat(parameters('svcFabData'), ';', parameters('svcFabLogs'), ';', parameters('svcFabRuntime'))]",
                "Processes": "Fabric.exe;FabricHost.exe;FabricInstallerService.exe;FabricSetup.exe;FabricDeployer.exe;ImageBuilder.exe;FabricGateway.exe;FabricDCA.exe;FabricFAS.exe;FabricUOS.exe;FabricRM.exe;FileStoreService.exe"
            },
            "RealtimeProtectionEnabled": "true",
            "ScheduledScanSettings": {
                "isEnabled": "true",
                "scanType": "Quick",
                "day": "7",
                "time": "120"
            }
        },
        "protectedSettings": null
    }
}
```

> [!NOTE]
> Si vous n’utilisez pas Windows Defender, reportez-vous à la documentation de votre logiciel anti-programme malveillant pour les règles de configuration. Windows Defender n’est pas pris en charge sur Linux.

## <a name="platform-isolation"></a>Isolement de la plateforme
Par défaut, les applications Service Fabric ont accès au runtime Service Fabric, qui se manifeste sous différentes formes : des [variables d’environnement](service-fabric-environment-variables-reference.md) pointant vers les chemins d’accès sur l’hôte correspondant à l’application et aux fichiers Fabric, un point de terminaison de communication inter-processus qui accepte les demandes spécifiques à l’application et le certificat client que Fabric prévoit que l’application utilisera pour s’authentifier. Dans l’éventualité que le service héberge lui-même du code non fiable, il est conseillé de désactiver cet accès au runtime SF, sauf si explicitement nécessaire. L’accès au runtime est supprimé via la déclaration suivante dans la section Stratégies du manifeste de l’application : 

```xml
<ServiceManifestImport>
    <Policies>
        <ServiceFabricRuntimeAccessPolicy RemoveServiceFabricRuntimeAccess="true"/>
    </Policies>
</ServiceManifestImport>

```

## <a name="next-steps"></a>Étapes suivantes

* Créez un cluster sur des machines virtuelles ou des ordinateurs exécutant Windows Server : [Création de clusters Service Fabric pour Windows Server](service-fabric-cluster-creation-for-windows-server.md).
* Créez un cluster sur des machines virtuelles ou des ordinateurs exécutant Linux : [Créer un cluster Linux](service-fabric-cluster-creation-via-portal.md).
* Découvrez les [options de support de Service Fabric](service-fabric-support.md).

[Image1]: ./media/service-fabric-best-practices/generate-common-name-cert-portal.png
