---
title: Configurer un certificat de chiffrement sur des clusters Windows
description: Découvrez comment configurer un certificat de chiffrement et chiffrer des secrets sur clusters Windows.
ms.topic: conceptual
ms.date: 01/04/2019
ms.openlocfilehash: eb4909d62a2627c368f24dab572b25c6f1df30ec
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82583295"
---
# <a name="set-up-an-encryption-certificate-and-encrypt-secrets-on-windows-clusters"></a>Configurer un certificat de chiffrement et chiffrer des secrets sur clusters Windows
Dans cet article, découvrez comment configurer un certificat de chiffrement et l’utiliser pour chiffrer des secrets sur clusters Windows. Pour les clusters Linux, consultez [Configurer un certificat de chiffrement et chiffrer des secrets sur des clusters Linux][secret-management-linux-specific-link].

[Azure Key Vault][key-vault-get-started] est utilisé ici comme un emplacement de stockage sécurisé pour des certificats et comme un moyen d’installer des certificats sur des clusters Service Fabric dans Azure. Si vous ne déployez pas dans Azure, il est inutile d’utiliser le coffre de clés pour gérer les secrets dans des applications Service Fabric. Toutefois, *l’utilisation de* secrets dans une application cloud est indépendante de la plateforme et permet ainsi un déploiement d’applications dans un cluster hébergé à n’importe quel endroit. 

## <a name="obtain-a-data-encipherment-certificate"></a>Obtenir un certificat de chiffrement de données
Un certificat de chiffrement de données est utilisé exclusivement pour le chiffrement et le déchiffrement des [paramètres][parameters-link] du fichier Settings.xml d’un service et des [variables d’environnement][environment-variables-link] du fichier ServiceManifest.xml d’un service. Il n’est pas utilisé pour l’authentification ou la signature du texte chiffré. Le certificat doit répondre aux exigences suivantes :

* Le certificat doit contenir une clé privée.
* Le certificat doit être créé pour l'échange de clés et pouvoir faire l'objet d'un export au format Personal Information Exchange (.pfx).
* L’utilisation d’une clé de certificat doit inclure le chiffrement de données (10) et ne doit pas inclure l’authentification du serveur ou l’authentification du client. 
  
  Par exemple, lorsque vous créez un certificat auto-signé à l’aide de PowerShell, l’indicateur `KeyUsage` doit être défini sur `DataEncipherment` :
  
  ```powershell
  New-SelfSignedCertificate -Type DocumentEncryptionCert -KeyUsage DataEncipherment -Subject mydataenciphermentcert -Provider 'Microsoft Enhanced Cryptographic Provider v1.0'
  ```

## <a name="install-the-certificate-in-your-cluster"></a>Installation du certificat dans votre cluster
Ce certificat doit être installé sur chaque nœud du cluster. Consultez la page concernant la [création d’un cluster à l’aide d’Azure Resource Manager][service-fabric-cluster-creation-via-arm] pour obtenir des instructions d’installation. 

## <a name="encrypt-application-secrets"></a>Chiffrement de secrets d’application
La commande PowerShell suivante est utilisée pour chiffrer un secret. Cette commande chiffre uniquement la valeur ; elle ne signe **pas** le texte chiffré. Vous devez utiliser le certificat de chiffrement qui est installé dans votre cluster afin de produire le texte chiffré pour les valeurs secrètes :

```powershell
Invoke-ServiceFabricEncryptText -CertStore -CertThumbprint "<thumbprint>" -Text "mysecret" -StoreLocation CurrentUser -StoreName My
```

La chaîne encodée en base 64 qui en résulte contient à la fois le texte chiffré secret et plus d’informations sur le certificat qui a été utilisé pour le chiffrement.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [Spécifier des secrets chiffrés dans une application][secret-management-specify-encrypted-secrets-link].

<!-- Links -->
[key-vault-get-started]:../key-vault/general/overview.md
[service-fabric-cluster-creation-via-arm]: service-fabric-cluster-creation-via-arm.md
[parameters-link]:service-fabric-how-to-parameterize-configuration-files.md
[environment-variables-link]: service-fabric-how-to-specify-environment-variables.md
[secret-management-linux-specific-link]: service-fabric-application-secret-management-linux.md
[secret-management-specify-encrypted-secrets-link]: service-fabric-application-secret-management.md#specify-encrypted-secrets-in-an-application
