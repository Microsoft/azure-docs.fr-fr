---
title: Créer un cluster à l’aide du nom commun du certificat
description: Découvrez comment créer un cluster Service Fabric en utilisant un nom commun de certificat à partir d’un modèle.
ms.topic: conceptual
ms.date: 09/06/2019
ms.openlocfilehash: c852b40d35f936753d3c16420159676da239b6c6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86246433"
---
# <a name="deploy-a-service-fabric-cluster-that-uses-certificate-common-name-instead-of-thumbprint"></a>Déployer un cluster Service Fabric utilisant un nom commun de certificat au lieu d’une empreinte
Deux certificats ne peuvent pas avoir la même empreinte, ce qui complique la gestion ou le renouvellement des certificats de cluster. Toutefois, plusieurs certificats peuvent avoir le même nom commun ou objet.  Un cluster utilisant des noms communs de certificat facilite grandement la gestion des certificats. Cet article décrit le déploiement d’un cluster Service Fabric pour utiliser le nom commun d’un certificat à la place de son empreinte.
 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="get-a-certificate"></a>Obtenir un certificat
Commencez par obtenir un certificat auprès d’une [autorité de certification (AC)](https://wikipedia.org/wiki/Certificate_authority).  Le nom commun du certificat doit être pour votre domaine personnalisé et doit être acheté auprès d’un bureau d’enregistrement de domaine. Par exemple, « azureservicefabricbestpractices.com ». Les personnes qui ne sont pas employées par Microsoft ne peuvent pas approvisionner les certificats pour les domaines Microsoft et vous ne pouvez donc pas utiliser les noms DNS de votre équilibreur de charge ou de Traffic Manager comme noms communs pour votre certificat. Vous devez approvisionner une [zone Azure DNS](../dns/dns-delegate-domain-azure-dns.md) si votre domaine personnalisé doit être résolu dans Azure. Vous devez également déclarer le domaine personnalisé que vous possédez comme « managementEndpoint » de votre cluster si vous souhaitez que le portail affiche l’alias de domaine personnalisé pour votre cluster.

À des fins de test, vous pouvez obtenir un certificat signé par une autorité de certification auprès d’une autorité de certification ouverte ou gratuite.

> [!NOTE]
> Les certificats auto-signés, notamment ceux qui sont générés lors du déploiement d’un cluster Service Fabric dans le portail Azure, ne sont pas pris en charge. 

## <a name="upload-the-certificate-to-a-key-vault"></a>Charger le certificat vers un coffre de clés
Dans Azure, un cluster Service Fabric est déployé sur un groupe de machines virtuelles identiques.  Chargez le certificat vers un coffre de clés.  Lors du déploiement du cluster, le certificat est installé sur le groupe de machines virtuelles identiques sur lequel le cluster s’exécute.

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser -Force

$SubscriptionId  =  "<subscription ID>"

# Sign in to your Azure account and select your subscription
Login-AzAccount -SubscriptionId $SubscriptionId

$region = "southcentralus"
$KeyVaultResourceGroupName  = "mykeyvaultgroup"
$VaultName = "mykeyvault"
$certFilename = "C:\users\sfuser\myclustercert.pfx"
$certname = "myclustercert"
$Password  = "P@ssw0rd!123"

# Create new Resource Group 
New-AzResourceGroup -Name $KeyVaultResourceGroupName -Location $region

# Create the new key vault
$newKeyVault = New-AzKeyVault -VaultName $VaultName -ResourceGroupName $KeyVaultResourceGroupName -Location $region -EnabledForDeployment 
$resourceId = $newKeyVault.ResourceId 

# Add the certificate to the key vault.
$PasswordSec = ConvertTo-SecureString -String $Password -AsPlainText -Force
$KVSecret = Import-AzKeyVaultCertificate -VaultName $vaultName -Name $certName  -FilePath $certFilename -Password $PasswordSec

$CertificateThumbprint = $KVSecret.Thumbprint
$CertificateURL = $KVSecret.SecretId
$SourceVault = $resourceId
$CommName    = $KVSecret.Certificate.SubjectName.Name

Write-Host "CertificateThumbprint    :"  $CertificateThumbprint
Write-Host "CertificateURL           :"  $CertificateURL
Write-Host "SourceVault              :"  $SourceVault
Write-Host "Common Name              :"  $CommName    
```

## <a name="download-and-update-a-sample-template"></a>Télécharger et mettre à jour un exemple de modèle
Cet article utilise l’exemple de modèle [cluster sécurisé à 5 nœuds](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure) et des paramètres du modèle. Téléchargez les fichiers *azuredeploy.json* et *azuredeploy.parameters.json* sur votre ordinateur.

### <a name="update-parameters-file"></a>Mettre à jour le fichier de paramètres
Tout d’abord, ouvrez le fichier *azuredeploy.parameters.json* dans un éditeur de texte, puis ajoutez la valeur de paramètre suivante :
```json
"certificateCommonName": {
    "value": "myclustername.southcentralus.cloudapp.azure.com"
},
"certificateIssuerThumbprint": {
    "value": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
},
```

Ensuite, définissez les valeurs des paramètres *certificateCommonName*, *sourceVaultValue* et *certificateUrlValue* sur celles retournées par le script précédent :
```json
"certificateCommonName": {
    "value": "myclustername.southcentralus.cloudapp.azure.com"
},
"certificateIssuerThumbprint": {
    "value": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
},
"sourceVaultValue": {
  "value": "/subscriptions/<subscription>/resourceGroups/testvaultgroup/providers/Microsoft.KeyVault/vaults/testvault"
},
"certificateUrlValue": {
  "value": "https://testvault.vault.azure.net:443/secrets/testcert/5c882b7192224447bbaecd5a46962655"
},
```

### <a name="update-the-template-file"></a>Mettre à jour le fichier de modèle
Ouvrez le fichier *azuredeploy.json* dans un éditeur de texte et effectuez trois mises à jour pour prendre en charge le nom commun du certificat.

1. Dans la section **parameters**, ajoutez un paramètre *certificateCommonName* :
    ```json
    "certificateCommonName": {
      "type": "string",
      "metadata": {
        "description": "Certificate Commonname"
      }
    },
    "certificateIssuerThumbprint": {
      "type": "string",
      "metadata": {
        "description": "Certificate Authority Issuer Thumpbrint for Commonname cert"
      }
    },
    ```

    Pensez également à supprimer le paramètre *certificateThumbprint*, car il n’est sans doute plus utile.

2. Définissez la valeur de la variable *sfrpApiVersion* sur « 2018-02-01 » :
    ```json
    "sfrpApiVersion": "2018-02-01",
    ```

3. Dans la ressource **Microsoft.Compute/virtualMachineScaleSets**, mettez à jour l’extension de machine virtuelle pour utiliser le nom commun dans les paramètres du certificat au lieu de l’empreinte.  Dans **virtualMachineProfile**->**extensionProfile**->**extensions**->**properties**->**settings**->**certificate**, ajoutez 
    ```json
       "commonNames": [
        "[parameters('certificateCommonName')]"
       ],
    ```

    et supprimez `"thumbprint": "[parameters('certificateThumbprint')]",`.

    ```json
    "virtualMachineProfile": {
      "extensionProfile": {
        "extensions": [
          {
            "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
            "properties": {
              "type": "ServiceFabricNode",
              "autoUpgradeMinorVersion": true,
              "protectedSettings": {
                "StorageAccountKey1": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key1]",
                "StorageAccountKey2": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key2]"
              },
              "publisher": "Microsoft.Azure.ServiceFabric",
              "settings": {
                "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                "nodeTypeRef": "[variables('vmNodeType0Name')]",
                "dataPath": "D:\\SvcFab",
                "durabilityLevel": "Bronze",
                "enableParallelJobs": true,
                "nicPrefixOverride": "[variables('subnet0Prefix')]",
                "certificate": {
                  "commonNames": [
                     "[parameters('certificateCommonName')]"
                  ],
                  "x509StoreName": "[parameters('certificateStoreValue')]"
                }
              },
              "typeHandlerVersion": "1.0"
            }
          },
    ```

4. Dans la ressource **Microsoft.ServiceFabric/clusters**, mettez à jour la version de l’API vers « 2018-02-01 ».  Ajoutez également un paramètre **certificateCommonNames** avec une propriété **commonNames** et supprimez le paramètre **certificate** (avec la propriété d’empreinte) comme dans l’exemple suivant :
   ```json
   {
       "apiVersion": "2018-02-01",
       "type": "Microsoft.ServiceFabric/clusters",
       "name": "[parameters('clusterName')]",
       "location": "[parameters('clusterLocation')]",
       "dependsOn": [
       "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]"
       ],
       "properties": {
       "addonFeatures": [
           "DnsService",
           "RepairManager"
       ],        
       "certificateCommonNames": {
           "commonNames": [
           {
               "certificateCommonName": "[parameters('certificateCommonName')]",
               "certificateIssuerThumbprint": "[parameters('certificateIssuerThumbprint')]"
           }
           ],
           "x509StoreName": "[parameters('certificateStoreValue')]"
       },
       ...
   ```
   > [!NOTE]
   > Le champ « certificateIssuerThumbprint » permet de spécifier les émetteurs de certificats attendus avec le nom commun d’un objet donné. Ce champ accepte une énumération séparée par des virgules d’empreintes numériques SHA-1. Il est à noter qu'il s’agit d’un renforcement de la validation du certificat : dans le cas où l’émetteur est vide ou non spécifié, le certificat est accepté pour l’authentification si sa chaîne peut être générée et se retrouve dans une racine approuvée par le validateur. Lorsque l’émetteur est spécifié, le certificat est accepté si l’empreinte de son émetteur direct correspond à l’une des valeurs spécifiées dans ce champ, que la racine soit ou non approuvée. Il faut savoir qu’une infrastructure à clé publique (PKI) peut utiliser différentes autorités de certification pour émettre des certificats portant sur le même objet ; il est donc important de spécifier toutes les empreintes d’émetteur attendues pour un objet donné.
   >
   > Il est recommandé de spécifier l’émetteur ; même s’il continue de fonctionner lorsqu’il est omis (pour les certificats en chaînage sur une racine approuvée), ce comportement comporte des limitations et risque d’être progressivement supprimé dans un avenir proche. Sachez également que les clusters déployés dans Azure et sécurisés avec des certificats X509 émis par une infrastructure PKI et déclarés par objet ne sont pas forcément validables par le service Azure Service Fabric (pour les communications de cluster à service) si la stratégie de certificat de l’infrastructure n’est pas détectable, disponible et accessible. 

## <a name="deploy-the-updated-template"></a>Déployer le modèle mis à jour
Redéployez le modèle mis à jour après avoir apporté les modifications.

```powershell
# Variables.
$groupname = "testclustergroup"
$clusterloc="southcentralus"  
$id="<subscription ID"

# Sign in to your Azure account and select your subscription
Login-AzAccount -SubscriptionId $id 

# Create a new resource group and deploy the cluster.
New-AzResourceGroup -Name $groupname -Location $clusterloc

New-AzResourceGroupDeployment -ResourceGroupName $groupname -TemplateParameterFile "C:\temp\cluster\AzureDeploy.Parameters.json" -TemplateFile "C:\temp\cluster\AzureDeploy.json" -Verbose
```

## <a name="next-steps"></a>Étapes suivantes
* Découvrez plus en détail la [sécurité des clusters](service-fabric-cluster-security.md).
* Découvrez comment [renouveler un certificat de cluster](service-fabric-cluster-rollover-cert-cn.md).
* [Mettre à jour et gérer les certificats de cluster](service-fabric-cluster-security-update-certs-azure.md)
* Simplifiez la gestion des certificats en apprenant à [modifier un cluster pour qu’il passe de l’utilisation d’une empreinte de certificat à l’utilisation d’un nom commun](service-fabric-cluster-change-cert-thumbprint-to-cn.md).

[image1]: .\media\service-fabric-cluster-change-cert-thumbprint-to-cn\PortalViewTemplates.png
