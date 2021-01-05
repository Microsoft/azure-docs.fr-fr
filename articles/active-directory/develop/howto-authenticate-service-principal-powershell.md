---
title: Créer une identité d’application Azure (PowerShell) | Azure
titleSuffix: Microsoft identity platform
description: Explique comment utiliser Azure PowerShell pour créer une application et un principal du service Azure Active Directory, et comment accorder à l’application l’accès aux ressources par le biais du contrôle d’accès en fonction du rôle. Il explique comment authentifier l’application avec un certificat.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev , devx-track-azurepowershell
ms.topic: how-to
ms.tgt_pltfrm: multiple
ms.date: 06/26/2020
ms.author: ryanwi
ms.reviewer: tomfitz
ms.openlocfilehash: 096b8ed60bf9880b6904ab952d4558939ca13574
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97652073"
---
# <a name="how-to-use-azure-powershell-to-create-a-service-principal-with-a-certificate"></a>Procédure : Utiliser Azure PowerShell pour créer un principal du service avec un certificat

Lorsque vous avez une application ou un script qui doit pouvoir accéder à des ressources, vous pouvez configurer une identité pour l’application et authentifier l’application avec ses propres informations d’identification. Cette identité est connue en tant que principal de service. Cette approche vous permet d’effectuer les opérations suivantes :

* Affecter à l’identité de l’application des autorisations différentes de vos propres autorisations. En règle générale, ces autorisations sont strictement limitées à ce que l’application doit faire.
* Utilisez un certificat pour l’authentification lors de l’exécution d’un script sans assistance.

> [!IMPORTANT]
> Au lieu de créer un principal du service, envisagez d’utiliser des identités managées pour les ressources Azure en ce qui concerne l’identité de votre application. Si votre code s’exécute sur un service qui prend en charge les identités managées et accède aux ressources qui prennent en charge l’authentification Azure Active Directory (Azure AD), les identités managées correspondent bien à vos besoins. Pour en savoir plus sur les identités gérées pour les ressources Azure, y compris les services qui les prennent actuellement en charge, consultez la rubrique [Que sont les identités gérées pour les ressources Azure ?](../managed-identities-azure-resources/overview.md).

Cet article explique comment créer un principal du service qui s’authentifie avec un certificat. Pour configurer un principal du service avec un mot de passe, consultez [Créer un principal du service Azure avec Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps).

Pour cet article, vous devez disposer de la [version la plus récente](/powershell/azure/install-az-ps) de PowerShell.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="required-permissions"></a>Autorisations requises

Pour réaliser les étapes décrites dans cet article, vous devez disposer des autorisations suffisantes dans Azure AD et votre abonnement Azure. Plus précisément, vous devez être en mesure de créer une application dans l’annuaire Azure AD et d’affecter un rôle au principal du service.

Le moyen le plus simple pour vérifier que votre compte dispose des autorisations adéquates est d’utiliser le portail. Consultez [Vérifier l’autorisation requise](howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

## <a name="assign-the-application-to-a-role"></a>Affecter l’application à un rôle
Pour accéder aux ressources de votre abonnement, vous devez affecter un rôle à l’application. Déterminez quel rôle fournit les autorisations appropriées pour l’application. Pour en savoir plus sur les rôles disponibles, consultez [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md).

Vous pouvez définir l’étendue au niveau de l’abonnement, du groupe de ressources ou de la ressource. Les autorisations sont héritées des niveaux inférieurs de l’étendue (par exemple, l’ajout d’une application au rôle *Lecteur* pour un groupe de ressources signifie qu’elle peut lire le groupe de ressources et toutes les ressources qu’il contient). Pour autoriser l’application à exécuter des actions telles que le redémarrage, le démarrage et l’arrêt d’instances, sélectionnez le rôle *Contributeur*.

## <a name="create-service-principal-with-self-signed-certificate"></a>Créer un principal du service avec un certificat auto-signé

L'exemple suivant aborde un scénario simple. Il utilise [New-AzADServicePrincipal](/powershell/module/az.resources/new-azadserviceprincipal) pour créer un principal du service avec un certificat auto-signé, et utilise [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) pour assigner le rôle de [Lecteur](../../role-based-access-control/built-in-roles.md#reader) au principal du service. L’attribution de rôle est étendue à votre abonnement Azure actuellement sélectionné. Pour sélectionner un autre abonnement, utilisez [Set-AzContext](/powershell/module/Az.Accounts/Set-AzContext).

> [!NOTE]
> Actuellement, l’applet de commande New-SelfSignedCertificate et le module PKI ne sont pas pris en charge dans PowerShell Core. 

```powershell
$cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" `
  -Subject "CN=exampleappScriptCert" `
  -KeySpec KeyExchange
$keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

$sp = New-AzADServicePrincipal -DisplayName exampleapp `
  -CertValue $keyValue `
  -EndDate $cert.NotAfter `
  -StartDate $cert.NotBefore
Sleep 20
New-AzRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $sp.ApplicationId
```

L’exemple reste en veille pendant 20 secondes pour laisser le temps au nouveau principal du service de se propager dans Azure AD. Si votre script ne patiente pas suffisamment longtemps, vous verrez une erreur indiquant : « Le principal {ID} n'existe pas dans le répertoire {DIR-ID} ». Pour résoudre cette erreur, patientez quelques instants puis réexécutez la commande **New-AzRoleAssignment**.

Vous pouvez étendre l’attribution de rôle à un groupe de ressources spécifiques à l’aide du paramètre **ResourceGroupName**. Vous pouvez également l’étendre à une ressource spécifique en utilisant les paramètres **ResourceType** et **ResourceName**. 

Si vous **n’avez pas Windows 10 ou Windows Server 2016**, vous devez télécharger le [générateur de certificats auto-signés](https://gallery.technet.microsoft.com/scriptcenter/Self-signed-certificate-5920a7c6/) depuis le Centre de scripts Microsoft. Extrayez son contenu et importez l’applet de commande dont vous avez besoin.

```powershell
# Only run if you could not use New-SelfSignedCertificate
Import-Module -Name c:\ExtractedModule\New-SelfSignedCertificateEx.ps1
```

Dans le script, remplacez les deux lignes suivantes pour générer le certificat.

```powershell
New-SelfSignedCertificateEx -StoreLocation CurrentUser `
  -Subject "CN=exampleapp" `
  -KeySpec "Exchange" `
  -FriendlyName "exampleapp"
$cert = Get-ChildItem -path Cert:\CurrentUser\my | where {$PSitem.Subject -eq 'CN=exampleapp' }
```

### <a name="provide-certificate-through-automated-powershell-script"></a>Fournir un certificat via un script PowerShell automatisé

Chaque fois que vous vous connectez en tant que principal de service, vous devez fournir l’ID de locataire du répertoire de votre application AD. Un locataire est une instance d’Azure AD.

```powershell
$TenantId = (Get-AzSubscription -SubscriptionName "Contoso Default").TenantId
$ApplicationId = (Get-AzADApplication -DisplayNameStartWith exampleapp).ApplicationId

$Thumbprint = (Get-ChildItem cert:\CurrentUser\My\ | Where-Object {$_.Subject -eq "CN=exampleappScriptCert" }).Thumbprint
Connect-AzAccount -ServicePrincipal `
  -CertificateThumbprint $Thumbprint `
  -ApplicationId $ApplicationId `
  -TenantId $TenantId
```

## <a name="create-service-principal-with-certificate-from-certificate-authority"></a>Créer un principal du service avec un certificat à partir de l’autorité de certification

L’exemple suivant utilise un certificat émis par une autorité de certification afin de créer le principal du service. L’attribution est étendue à l’abonnement Azure spécifié. Le principal du service est ajouté au rôle [Lecteur](../../role-based-access-control/built-in-roles.md#reader). Si une erreur se produit lors de l’attribution de rôle, il retente l’attribution.

```powershell
Param (
 [Parameter(Mandatory=$true)]
 [String] $ApplicationDisplayName,

 [Parameter(Mandatory=$true)]
 [String] $SubscriptionId,

 [Parameter(Mandatory=$true)]
 [String] $CertPath,

 [Parameter(Mandatory=$true)]
 [String] $CertPlainPassword
 )

 Connect-AzAccount
 Import-Module Az.Resources
 Set-AzContext -Subscription $SubscriptionId

 $CertPassword = ConvertTo-SecureString $CertPlainPassword -AsPlainText -Force

 $PFXCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($CertPath, $CertPassword)
 $KeyValue = [System.Convert]::ToBase64String($PFXCert.GetRawCertData())

 $ServicePrincipal = New-AzADServicePrincipal -DisplayName $ApplicationDisplayName
 New-AzADSpCredential -ObjectId $ServicePrincipal.Id -CertValue $KeyValue -StartDate $PFXCert.NotBefore -EndDate $PFXCert.NotAfter
 Get-AzADServicePrincipal -ObjectId $ServicePrincipal.Id 

 $NewRole = $null
 $Retries = 0;
 While ($NewRole -eq $null -and $Retries -le 6)
 {
    # Sleep here for a few seconds to allow the service principal application to become active (should only take a couple of seconds normally)
    Sleep 15
    New-AzRoleAssignment -RoleDefinitionName Reader -ServicePrincipalName $ServicePrincipal.ApplicationId | Write-Verbose -ErrorAction SilentlyContinue
    $NewRole = Get-AzRoleAssignment -ObjectId $ServicePrincipal.Id -ErrorAction SilentlyContinue
    $Retries++;
 }

 $NewRole
```

### <a name="provide-certificate-through-automated-powershell-script"></a>Fournir un certificat via un script PowerShell automatisé
Chaque fois que vous vous connectez en tant que principal de service, vous devez fournir l’ID de locataire du répertoire de votre application AD. Un locataire est une instance d’Azure AD.

```powershell
Param (

 [Parameter(Mandatory=$true)]
 [String] $CertPath,

 [Parameter(Mandatory=$true)]
 [String] $CertPlainPassword,

 [Parameter(Mandatory=$true)]
 [String] $ApplicationId,

 [Parameter(Mandatory=$true)]
 [String] $TenantId
 )

 $CertPassword = ConvertTo-SecureString $CertPlainPassword -AsPlainText -Force
 $PFXCert = New-Object `
  -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 `
  -ArgumentList @($CertPath, $CertPassword)
 $Thumbprint = $PFXCert.Thumbprint

 Connect-AzAccount -ServicePrincipal `
  -CertificateThumbprint $Thumbprint `
  -ApplicationId $ApplicationId `
  -TenantId $TenantId
```

L’ID de l’application et l’ID de l’abonné ne sont pas sensibles, vous pouvez donc les incorporer directement dans votre script. Pour récupérer l’ID client, utilisez :

```powershell
(Get-AzSubscription -SubscriptionName "Contoso Default").TenantId
```

Si vous avez besoin extraire l’ID de l’application, utilisez :

```powershell
(Get-AzADApplication -DisplayNameStartWith {display-name}).ApplicationId
```

## <a name="change-credentials"></a>Modifier les informations d’identification

Pour modifier les informations d’identification d’une application Active Directory, en raison d’une faille de sécurité ou de l’expiration des informations d’identification, utilisez les cmdlets [Remove-AzADAppCredential](/powershell/module/az.resources/remove-azadappcredential) et [New-AzADAppCredential](/powershell/module/az.resources/new-azadappcredential).

Pour supprimer toutes les informations d’identification d’une application, utilisez :

```powershell
Get-AzADApplication -DisplayName exampleapp | Remove-AzADAppCredential
```

Pour ajouter une valeur de certificat, créez un certificat auto-signé, comme indiqué dans cet article. Ensuite, utilisez :

```powershell
Get-AzADApplication -DisplayName exampleapp | New-AzADAppCredential `
  -CertValue $keyValue `
  -EndDate $cert.NotAfter `
  -StartDate $cert.NotBefore
```

## <a name="debug"></a>Débogage

Lors de la création d’un principal du service, vous pouvez rencontrer les erreurs suivantes :

* **« Authentication_Unauthorized »** ou **« Aucun abonnement trouvé dans le contexte. »** - Vous voyez cette erreur lorsque votre compte ne possède pas les [autorisations requises](#required-permissions) sur Azure AD pour inscrire une application. En règle générale, vous obtenez cette erreur lorsque seuls des utilisateurs administrateurs dans votre annuaire Azure Active Directory peuvent inscrire des applications et que votre compte n’est pas un compte d’administrateur. Demandez à votre administrateur de vous affecter à un rôle d’administrateur ou d’autoriser les utilisateurs ordinaires à inscrire des applications.

* Le message indique que votre compte **« n’est pas autorisé à effectuer l’action ’Microsoft.Authorization/roleAssignments/write’ sur l’étendue ’/subscriptions/{guid}’ »** - Vous voyez cette erreur lorsque votre compte ne dispose pas d’autorisations suffisantes pour affecter un rôle à une identité. Demandez à votre administrateur d’abonnement de vous ajouter au rôle Administrateur de l’accès utilisateur.

## <a name="next-steps"></a>Étapes suivantes

* Pour configurer un principal du service avec un mot de passe, consultez [Créer un principal du service Azure avec Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps).
* Pour obtenir une explication plus détaillée des applications et des principaux du service, consultez la rubrique [Objets principal du service et application](app-objects-and-service-principals.md).
* Pour plus d’informations sur l’authentification Azure AD, consultez la rubrique [Scénarios d’authentification pour Azure AD](./authentication-vs-authorization.md).
* Pour plus d’informations sur l’utilisation des inscriptions d’applications à l’aide de **Microsoft Graph**, consultez les informations de référence sur l’API [Applications](/graph/api/resources/application).
