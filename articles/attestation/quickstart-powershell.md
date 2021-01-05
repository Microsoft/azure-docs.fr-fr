---
title: Configurer Azure Attestation avec Azure PowerShell
description: Guide pratique pour installer et configurer un fournisseur d’attestations à l’aide d’Azure PowerShell
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: 6050dd43b91299cd776d1121d7398f46e95bc096
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92678197"
---
# <a name="quickstart-set-up-azure-attestation-with-azure-powershell"></a>Démarrage rapide : Configurer Azure Attestation avec Azure PowerShell

Suivez les étapes ci-dessous pour créer et configurer un fournisseur d’attestations à l’aide d’Azure PowerShell. Pour plus d’informations sur l’installation et l’exécution d’Azure PowerShell, consultez [Vue d’ensemble d’Azure PowerShell](/powershell/azure/?view=azps-2.8.0&viewFallbackFrom=azps-2.4.0).

Notez que PowerShell Gallery a déprécié les versions 1.0 et 1.1 du protocole TLS (Transport Layer Security). TLS 1.2 ou une version ultérieure est recommandé. Vous risquez donc de recevoir les erreurs suivantes :

- AVERTISSEMENT : Impossible de résoudre la source du package « https://www.powershellgallery.com/api/v2  »
- PackageManagement\Install-Package : Aucune correspondance n’a été trouvée pour les critères de recherche et le nom de module spécifiés 

Pour continuer à interagir avec PowerShell Gallery, exécutez la commande suivante avant les commandes Install-Module

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 
```

## <a name="install-azattestation-powershell-module"></a>Installer le module PowerShell Az.Attestation

Sur une machine dotée d’Azure PowerShell, installez le module PowerShell Az.Attestation, qui contient des applets de commande pour Azure Attestation.  

### <a name="initial-installation"></a>Installation initiale

Terminez toutes les fenêtres PowerShell existantes.

Pour effectuer l’installation pour « l’utilisateur actuel », lancez une fenêtre PowerShell sans élévation de privilèges et exécutez la commande suivante :

```powershell
Install-Module -Name Az.Attestation -AllowClobber -Scope CurrentUser
```

Pour effectuer l’installation pour « tous les utilisateurs », lancez une fenêtre PowerShell avec élévation de privilèges et exécutez la commande suivante :

```powershell
Install-Module -Name Az.Attestation -AllowClobber -Scope AllUsers
```

Fermez la console PowerShell avec élévation de privilèges.

### <a name="update-the-installation"></a>Mettre à jour l’installation

Terminez toutes les fenêtres PowerShell existantes.

Pour effectuer une mise à jour pour « l’utilisateur actuel », lancez une fenêtre PowerShell sans élévation de privilèges et exécutez la commande suivante :

```powershell
Update-Module -Name Az.Attestation
```

Pour effectuer une mise à jour pour « tous les utilisateurs », lancez une fenêtre PowerShell avec élévation de privilèges et exécutez la commande suivante :

```powershell
Update-Module -Name Az.Attestation
```

Fermez la console PowerShell avec élévation de privilèges.

### <a name="get-installed-modules"></a>Obtenir les modules installés

Version minimale des modules Az nécessaires pour prendre en charge les opérations d’attestation :
- Az 4.5.0
- Az.Accounts 1.9.2
- Az.Attestation 0.1.8

Exécutez la commande ci-dessous pour vérifier la version installée de tous les modules Az : 

```powershell
Get-InstalledModule
```
Si les versions ne correspondent pas à la configuration minimale requise, exécutez des commandes Update-Module.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à Azure dans la console PowerShell (sans privilèges d’accès élevés).

```powershell
Connect-AzAccount
```

Si nécessaire, basculez vers l’abonnement à utiliser pour Azure Attestation.

```powershell
Set-AzContext -Subscription <subscription id>  
```

## <a name="register-microsoftattestation-resource-provider"></a>Inscrire le fournisseur de ressources Microsoft.Attestation

Inscrivez le fournisseur de ressources Microsoft.Attestation dans l’abonnement. Pour plus d’informations sur les fournisseurs de ressources Azure et sur la configuration et la gestion des fournisseurs de ressources, consultez [Fournisseurs et types de ressources Azure](../azure-resource-manager/management/resource-providers-and-types.md). Notez que l’inscription d’un fournisseur de ressources n’est nécessaire qu’une seule fois pour un abonnement.

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Attestation
```
## <a name="regional-availability-of-azure-attestation"></a>Disponibilité régionale d’Azure Attestation

```powershell
(Get-AzResourceProvider -ProviderNamespace Microsoft.Attestation)[0].Locations
```

## <a name="create-an-azure-resource-group"></a>Création d’un groupe de ressources Azure

Créez un groupe de ressources pour le fournisseur d’attestations. Notez que d’autres ressources Azure (y compris une machine virtuelle avec une instance d’application cliente) peuvent être placées dans le même groupe de ressources.

```powershell
$location = "uksouth" 
$attestationResourceGroup = "<attestation provider resource group name>"
New-AzResourceGroup -Name $attestationResourceGroup -Location $location 
```

## <a name="create-and-manage-an-attestation-provider"></a>Créer et gérer un fournisseur d’attestations

New-AzAttestation crée un fournisseur d’attestations.

```powershell
$attestationProvider = "<attestation provider name>" 
New-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Location $location
```

PolicySignerCertificateFile est un fichier qui spécifie un jeu de clés de signature approuvées. Si un nom de fichier est spécifié pour le paramètre PolicySignerCertificateFile, le fournisseur d’attestations peut être configuré uniquement avec des stratégies au format JWT signé. Sinon, la stratégie peut être configurée dans du texte ou dans un format JWT non signé.

```powershell
New-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Location $location -PolicySignersCertificateFile "C:\test\policySignersCertificates.pem"
```

Pour obtenir un exemple de fichier PolicySignersCertificateFile, consultez [Exemples de certificat du signataire de stratégie](policy-signer-examples.md).

Get-AzAttestation récupère les propriétés du fournisseur d’attestations telles que status et AttestURI. Prenez note de la propriété AttestURI, car vous en aurez besoin plus tard.

```azurepowershell
Get-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup  
```

La commande ci-dessus doit produire une sortie semblable à celle ci-dessous : 

```
Id:/subscriptions/MySubscriptionID/resourceGroups/MyResourceGroup/providers/Microsoft.Attestation/attestationProviders/MyAttestationProvider
Location: MyLocation
ResourceGroupName: MyResourceGroup
Name: MyAttestationProvider
Status: Ready
TrustModel: AAD
AttestUri: https://MyAttestationProvider.us.attest.azure.net 
Tags: 
TagsTable: 
```

Les fournisseurs d’attestations peuvent être supprimés à l’aide de l’applet de commande Remove-AzAttestation.  

```powershell
Remove-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup
```

## <a name="policy-management"></a>Gestion des stratégies

Afin de gérer les stratégies, un utilisateur Azure AD doit avoir les autorisations suivantes pour « Actions » :
- Microsoft.Attestation/attestationProviders/attestation/read
- Microsoft.Attestation/attestationProviders/attestation/write
- Microsoft.Attestation/attestationProviders/attestation/delete

Ces autorisations peuvent être attribuées à un utilisateur AD par le biais d’un rôle tel que « Propriétaire » (autorisations génériques), « Contributeur » (autorisations génériques) ou « Contributeur d’attestation » (autorisations spécifiques pour Azure Attestation uniquement).  

Afin de lire les stratégies, un utilisateur Azure AD doit avoir l’autorisation suivante pour « Actions » :
- Microsoft.Attestation/attestationProviders/attestation/read

Cette autorisation peut être attribuée à un utilisateur AD par le biais d’un rôle tel que « Lecteur » (autorisations génériques) ou « Lecteur d’attestation » (autorisations spécifiques pour Azure Attestation uniquement).

Les applets de commande PowerShell ci-dessous permettent de gérer les stratégies pour un fournisseur d’attestations (un environnement TEE à la fois).

Get-AzAttestationPolicy retourne la stratégie actuelle pour l’environnement TEE spécifié. L’applet de commande affiche la stratégie à la fois dans le texte et le format JWT de la stratégie.

```powershell
$teeType = "<tee Type>"
Get-AzAttestationPolicy   -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Tee $teeType 
```

Les types d’environnements d’exécution de confiance (TEE, Trusted Execution Environment) pris en charge sont « SgxEnclave », « OpenEnclave » et « VbsEnclave ».

Set-AttestationPolicy définit une nouvelle stratégie pour l’environnement TEE spécifié. L’applet de commande accepte une stratégie au format texte ou JWT et est contrôlée par le paramètre PolicyFormat. « Text » est la valeur par défaut de PolicyFormat. 

```powershell
$policyFormat = "<policy format>"
$policy=Get-Content -path "C:\test\policy.txt" -Raw
Set-AzAttestationPolicy   -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Tee $teeType -Policy $policy -PolicyFormat $policyFormat 
```

Si PolicySignerCertificateFile est fourni lors de la création d’un fournisseur d’attestations, les stratégies peuvent être configurées uniquement au format JWT signé. Sinon, la stratégie peut être configurée dans du texte ou dans un format JWT non signé.

La stratégie d’attestation au format JWT doit contenir une revendication nommée « AttestationPolicy ». Pour la stratégie signée, le JWT doit être signé avec une clé privée correspondant à l’un des certificats de signataire de stratégie existants.

Pour obtenir des exemples de stratégie, consultez [Exemples de stratégie d’attestation](policy-examples.md).

Reset-AzAttestationPolicy rétablit la valeur par défaut de la stratégie pour l’environnement TEE spécifié.

```powershell
Reset-AzAttestationPolicy -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Tee $teeType 
```

## <a name="policy-signer-certificates-management"></a>Gestion des certificats du signataire de stratégie

Les applets de commande PowerShell ci-dessous permettent de gérer les certificats du signataire de stratégie pour un fournisseur d’attestations :

```powershell
Get-AzAttestationPolicySigners -Name $attestationProvider -ResourceGroupName $attestationResourceGroup

Add-AzAttestationPolicySigner -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Signer <signer>

Remove-AzAttestationPolicySigner -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Signer <signer>
```

Le certificat du signataire de stratégie est un jeton JWT signé avec une revendication nommée « maa-policyCertificate ». La valeur de la revendication est un objet JWK qui contient la clé de signature approuvée à ajouter. Le jeton JWT doit être signé avec une clé privée correspondant à l’un des certificats de signataire de stratégie existants.

Notez que toute manipulation sémantique du certificat du signataire de stratégie doit être effectuée en dehors de PowerShell. En ce qui concerne PowerShell, il s’agit d’une simple chaîne.

Pour obtenir un exemple de certificat du signataire de stratégie, consultez [Exemples de certificat de signataire de stratégie](policy-signer-examples.md).

Pour plus d’informations sur les applets de commande et leurs paramètres, consultez [Applets de commande PowerShell pour Azure Attestation](/powershell/module/az.attestation/?view=azps-4.3.0#attestation). 

## <a name="next-steps"></a>Étapes suivantes

- [Guide pratique pour créer et signer une stratégie d’attestation](author-sign-policy.md)
- [Attester une enclave SGX à l’aide d’exemples de code](/samples/browse/?expanded=azure&terms=attestation)