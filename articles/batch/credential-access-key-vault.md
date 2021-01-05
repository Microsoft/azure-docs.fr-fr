---
title: Accéder de façon sécurisée à Key Vault avec Batch
description: Découvrez comment accéder par programmation à vos informations d’identification à partir de Key Vault à l’aide d’Azure Batch.
ms.topic: how-to
ms.date: 10/28/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b8b3d2655e79862c068aa48c29c7e89b7df85482
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350685"
---
# <a name="securely-access-key-vault-with-batch"></a>Accéder de façon sécurisée à Key Vault avec Batch

Dans cet article, vous allez apprendre à configurer des nœuds Batch pour accéder de façon sécurisée aux informations d’identification stockées dans [Azure Key Vault](../key-vault/general/overview.md). Il est sans intérêt de placer vos informations d’identification d’administrateur dans Key Vault, puis de coder les informations d’identification de manière irréversible pour accéder à Key Vault à partir d’un script. La solution consiste à utiliser un certificat qui accorde à vos nœuds Batch l’accès à Key Vault.

Pour vous authentifier auprès d’Azure Key Vault à partir d’un nœud Batch, vous avez besoin des éléments suivants :

- Des informations d’identification Azure Active Directory (Azure AD)
- Un certificat
- Un compte Batch
- Un pool Batch avec au moins un nœud

## <a name="obtain-a-certificate"></a>Obtenir un certificat

Si vous n’avez pas encore de certificat, le moyen le plus simple d’en obtenir un consiste à générer un certificat auto-signé à l’aide de l’outil en ligne de commande `makecert`.

Vous pouvez généralement trouver `makecert` dans ce chemin : `C:\Program Files (x86)\Windows Kits\10\bin\<arch>`. Ouvrez une invite de commandes en tant qu’administrateur et accédez à `makecert` à l’aide de l’exemple suivant.

```console
cd C:\Program Files (x86)\Windows Kits\10\bin\x64
```

Ensuite, utilisez l’outil `makecert` pour créer des fichiers de certificat auto-signé, appelés `batchcertificate.cer` et `batchcertificate.pvk`. Le nom commun (CN) utilisé n’est pas important pour cette application, mais il est utile de s’en servir comme d’une indication de la finalité du certificat.

```console
makecert -sv batchcertificate.pvk -n "cn=batch.cert.mydomain.org" batchcertificate.cer -b 09/23/2019 -e 09/23/2019 -r -pe -a sha256 -len 2048
```

Batch nécessite un fichier `.pfx`. Utilisez l’outil [pvk2pfx](/windows-hardware/drivers/devtest/pvk2pfx) pour convertir les fichiers `.cer` et `.pvk` créés par `makecert` en un seul fichier `.pfx`.

```console
pvk2pfx -pvk batchcertificate.pvk -spc batchcertificate.cer -pfx batchcertificate.pfx -po
```

## <a name="create-a-service-principal"></a>Créer un principal du service

L’accès à Key Vault est accordé à un **utilisateur** ou à un **principal de service**. Pour accéder programmatiquement à Key Vault, utilisez un [principal de service](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) avec le certificat créé à l’étape précédente. Le principal de service doit être dans le même locataire Azure AD que le coffre de clés.

```powershell
$now = [System.DateTime]::Parse("2020-02-10")
# Set this to the expiration date of the certificate
$expirationDate = [System.DateTime]::Parse("2021-02-10")
# Point the script at the cer file you created $cerCertificateFilePath = 'c:\temp\batchcertificate.cer'
$cer = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
$cer.Import($cerCertificateFilePath)
# Load the certificate into memory
$credValue = [System.Convert]::ToBase64String($cer.GetRawCertData())
# Create a new AAD application that uses this certificate
$newADApplication = New-AzureRmADApplication -DisplayName "Batch Key Vault Access" -HomePage "https://batch.mydomain.com" -IdentifierUris "https://batch.mydomain.com" -certValue $credValue -StartDate $now -EndDate $expirationDate
# Create new AAD service principal that uses this application
$newAzureAdPrincipal = New-AzureRmADServicePrincipal -ApplicationId $newADApplication.ApplicationId
```

Les URL de l’application ne sont pas importantes, car elles ne seront utilisées que pour l’accès à Key Vault.

## <a name="grant-rights-to-key-vault"></a>Accorder des droits à Key Vault

Le principal de service créé à l’étape précédente doit avoir l’autorisation de récupérer les secrets de Key Vault. L’autorisation peut être accordée sur le [Portail Azure](../key-vault/general/assign-access-policy-portal.md) ou avec la commande PowerShell ci-dessous.

```powershell
Set-AzureRmKeyVaultAccessPolicy -VaultName 'BatchVault' -ServicePrincipalName '"https://batch.mydomain.com' -PermissionsToSecrets 'Get'
```

## <a name="assign-a-certificate-to-a-batch-account"></a>Attribuer un certificat à un compte Batch

Créez un pool Batch, accédez à l’onglet des certificats dans le pool, puis attribuez le certificat que vous avez créé. Le certificat est maintenant sur tous les nœuds Batch.

Attribuons à présent le certificat au compte Batch. Batch pourra ainsi l’affecter aux pools, puis aux nœuds. Pour ce faire, la méthode la plus simple consiste à accéder à votre compte Batch dans le portail, à accéder à **Certificats**, puis à sélectionner **Ajouter**. Téléchargez le fichier `.pfx` que vous avez généré et indiquez le mot de passe. Une fois l’opération terminée, le certificat est ajouté à la liste et vous pouvez vérifier l’empreinte.

Désormais, quand vous créez un pool Batch, vous pouvez accéder à **Certificats** au sein du pool et attribuer à ce dernier le certificat que vous avez créé. Quand vous effectuez cette opération, assurez-vous de sélectionner **LocalMachine** pour l’emplacement du magasin. Le certificat est chargé sur tous les nœuds Batch dans le pool.

## <a name="install-azure-powershell"></a>Installation d’Azure PowerShell

Si vous envisagez d’accéder à Key Vault à l’aide de scripts PowerShell sur vos nœuds, il est nécessaire que la bibliothèque Azure PowerShell soit installée. Si la version 5 de WMF (Windows Management Framework) est installée sur vos nœuds, vous pouvez utiliser la commande install-module pour la télécharger. Si vous utilisez des nœuds non dotés de WMF 5, le moyen le plus simple de l’installer consiste à regrouper le fichier `.msi` Azure PowerShell avec vos fichiers Batch, puis à appeler le programme d’installation au début de votre script de démarrage Batch. Pour plus de détails, consultez cet exemple :

```powershell
$psModuleCheck=Get-Module -ListAvailable -Name Azure -Refresh
if($psModuleCheck.count -eq 0) {
    $psInstallerPath = Join-Path $downloadPath "azure-powershell.3.4.0.msi" Start-Process msiexec.exe -ArgumentList /i, $psInstallerPath, /quiet -wait
}
```

## <a name="access-key-vault"></a>Accéder au coffre de clés

Vous pouvez maintenant accéder à Key Vault dans des scripts qui s’exécutent sur vos nœuds Batch. Pour que vous puissiez accéder à Key Vault à partir d’un script, il suffit que ce dernier s’authentifie auprès d’Azure AD à l’aide du certificat. Pour effectuer cette opération dans PowerShell, utilisez les exemples de commandes suivants. Spécifiez le GUID approprié pour l’**Empreinte**, l’**ID de l’application** (ID de votre principal de service) et l’**ID de locataire** (locataire dans lequel se trouve votre principal de service).

```powershell
Add-AzureRmAccount -ServicePrincipal -CertificateThumbprint -ApplicationId
```

Une fois authentifié, accédez à Key Vault comme vous le feriez normalement.

```powershell
$adminPassword=Get-AzureKeyVaultSecret -VaultName BatchVault -Name batchAdminPass
```

Ce sont là les informations d’identification à utiliser dans votre script.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Azure Key Vault](../key-vault/general/overview.md).
- Consultez la [Base de référence de sécurité Azure pour Batch](security-baseline.md).
- Découvrez les fonctionnalités de Batch, notamment la [configuration de l’accès aux nœuds de calcul](pool-endpoint-configuration.md), les [nœuds de calcul Linux](batch-linux-nodes.md) et les [points de terminaison privés](private-connectivity.md).