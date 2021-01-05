---
title: Gérer les stratégies personnalisées avec PowerShell
titleSuffix: Azure AD B2C
description: Utilisez la cmdlet PowerShell Azure Active Directory (Azure AD) pour la gestion programmatique de vos stratégies personnalisées Azure AD B2C. Créez, lisez, mettez à jour et supprimez des stratégies personnalisées avec PowerShell.
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/14/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: fcc482e215e646fec20516f35641bd05398d2f2d
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928713"
---
# <a name="manage-azure-ad-b2c-custom-policies-with-azure-powershell"></a>Gérer les stratégies personnalisées Azure AD B2C avec Azure PowerShell

Azure PowerShell fournit différentes cmdlet pour la gestion basée sur ligne de commande et sur script des stratégies personnalisées dans votre locataire Azure AD B2C. Apprenez à utiliser le module PowerShell Azure AD pour :

* Dresser la liste des stratégies personnalisées dans un locataire Azure AD B2C
* Télécharger une stratégie à partir d'un locataire
* Mettre à jour une stratégie existante en écrasant son contenu
* Charger une nouvelle stratégie dans votre locataire Azure AD B2C
* Supprimer une stratégie personnalisée d'un locataire

## <a name="prerequisites"></a>Conditions préalables requises

* [Locataire Azure AD B2C](tutorial-create-tenant.md) et informations d'identification d'un utilisateur de l'annuaire disposant du rôle [Administrateur de stratégies B2C IEF](../active-directory/roles/permissions-reference.md#b2c-ief-policy-administrator)
* [Stratégies personnalisées](custom-policy-get-started.md) téléchargées sur votre locataire
* [Azure AD PowerShell pour Graph **- Module disponible en préversion**](/powershell/azure/active-directory/install-adv2)

## <a name="connect-powershell-session-to-b2c-tenant"></a>Connecter la session PowerShell au locataire B2C

Pour utiliser des stratégies personnalisées dans votre locataire Azure AD B2C, vous devez d'abord connecter votre session PowerShell au locataire à l'aide de la commande [Connect-AzureAD][Connect-AzureAD].

Exécutez la commande suivante, en remplaçant `{b2c-tenant-name}` par le nom de votre locataire Azure AD B2C. Connectez-vous à l'aide d'un compte de l'annuaire auquel le rôle [Administrateur de stratégies B2C IEF](../active-directory/roles/permissions-reference.md#b2c-ief-policy-administrator) a été attribué.

```PowerShell
Connect-AzureAD -Tenant "{b2c-tenant-name}.onmicrosoft.com"
```

Exemple de sortie de commande affichant une connexion réussie :

```Console
PS C:\> Connect-AzureAD -Tenant "contosob2c.onmicrosoft.com"

Account               Environment TenantId                             TenantDomain                 AccountType
-------               ----------- --------                             ------------                 -----------
azureuser@contoso.com AzureCloud  00000000-0000-0000-0000-000000000000 contosob2c.onmicrosoft.com   User
```

## <a name="list-all-custom-policies-in-the-tenant"></a>Dresser la liste de toutes les stratégies personnalisées du locataire

La découverte de stratégies personnalisées permet à un administrateur Azure AD B2C d'examiner, de gérer et d'ajouter une logique métier à ses opérations. Utilisez la commande [Get-AzureADMSTrustFrameworkPolicy][Get-AzureADMSTrustFrameworkPolicy] pour renvoyer la liste des ID des stratégies personnalisées d'un locataire Azure AD B2C.

```PowerShell
Get-AzureADMSTrustFrameworkPolicy
```

Exemple de sortie de commande :

```Console
PS C:\> Get-AzureADMSTrustFrameworkPolicy

Id
--
B2C_1A_TrustFrameworkBase
B2C_1A_TrustFrameworkExtensions
B2C_1A_signup_signin
B2C_1A_ProfileEdit
B2C_1A_PasswordReset
```

## <a name="download-a-policy"></a>Télécharger une stratégie

Après avoir examiné la liste des ID de stratégies, vous pouvez cibler une stratégie spécifique à l'aide de la commande [Get-AzureADMSTrustFrameworkPolicy][Get-AzureADMSTrustFrameworkPolicy] afin de télécharger son contenu.

```PowerShell
Get-AzureADMSTrustFrameworkPolicy [-Id <policyId>]
```

Dans cet exemple, la stratégie correspondant à l'ID *B2C_1A_signup_signin* est téléchargée :

```Console
PS C:\> Get-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin
<TrustFrameworkPolicy xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06" PolicySchemaVersion="0.3.0.0" TenantId="contosob2c.onmicrosoft.com" PolicyId="B2C_1A_signup_signin" PublicPolicyUri="http://contosob2c.onmicrosoft.com/B2C_1A_signup_signin" TenantObjectId="00000000-0000-0000-0000-000000000000">
  <BasePolicy>
    <TenantId>contosob2c.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```

Pour modifier le contenu de la stratégie localement, dirigez la sortie de la commande vers un fichier à l'aide de l'argument `-OutputFilePath`, puis ouvrez le fichier dans l'éditeur de votre choix.

Exemple de commande envoyant une sortie vers un fichier :

```PowerShell
# Download and send policy output to a file
Get-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin -OutputFilePath C:\RPPolicy.xml
```

## <a name="update-an-existing-policy"></a>Mettre à jour une stratégie existante

Après avoir modifié un fichier de stratégie que vous avez créé ou téléchargé, vous pouvez publier la stratégie mise à jour sur Azure AD B2C à l'aide de la commande [Set-AzureADMSTrustFrameworkPolicy][Set-AzureADMSTrustFrameworkPolicy].

Si vous utilisez la commande `Set-AzureADMSTrustFrameworkPolicy` avec l'ID d'une stratégie qui existe déjà dans votre locataire Azure AD B2C, le contenu de cette stratégie est écrasé.

```PowerShell
Set-AzureADMSTrustFrameworkPolicy [-Id <policyId>] -InputFilePath <inputpolicyfilePath> [-OutputFilePath <outputFilePath>]
```

Exemple de commande :

```PowerShell
# Update an existing policy from file
Set-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin -InputFilePath C:\B2C_1A_signup_signin.xml
```

Pour accéder à d'autres exemples, consultez les informations de référence relatives à la commande [Set-AzureADMSTrustFrameworkPolicy][Set-AzureADMSTrustFrameworkPolicy].

## <a name="upload-a-new-policy"></a>Charger une nouvelle stratégie

Lorsque vous apportez une modification à une stratégie personnalisée qui est en production, vous pouvez publier plusieurs versions de cette stratégie pour des scénarios de secours ou de test A/B. Vous pouvez également effectuer une copie d'une stratégie existante, y apporter quelques petites modifications, puis la charger en tant que nouvelle stratégie à utiliser par une autre application.

Utilisez la commande [New-AzureADMSTrustFrameworkPolicy][New-AzureADMSTrustFrameworkPolicy] pour charger une nouvelle stratégie :

```PowerShell
New-AzureADMSTrustFrameworkPolicy -InputFilePath <inputpolicyfilePath> [-OutputFilePath <outputFilePath>]
```

Exemple de commande :

```PowerShell
# Add new policy from file
New-AzureADMSTrustFrameworkPolicy -InputFilePath C:\SignUpOrSignInv2.xml
```

## <a name="delete-a-custom-policy"></a>Supprimer une stratégie personnalisée

Pour maintenir le cycle de vie des opérations, nous vous recommandons de supprimer régulièrement les stratégies personnalisées inutilisées. Par exemple, n'hésitez pas à supprimer les anciennes versions des stratégies après avoir procédé à une migration vers un nouvel ensemble de stratégies et vérifié le bon fonctionnement des nouvelles stratégies. En outre, si vous tentez de publier un ensemble de stratégies personnalisées et que vous recevez un message d'erreur, il peut être judicieux de supprimer les stratégies créées avec la version défectueuse.

Utilisez la commande [Remove-AzureADMSTrustFrameworkPolicy][Remove-AzureADMSTrustFrameworkPolicy] pour supprimer une stratégie de votre locataire.

```PowerShell
Remove-AzureADMSTrustFrameworkPolicy -Id <policyId>
```

Exemple de commande :

```PowerShell
# Delete an existing policy
Remove-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin
```

## <a name="troubleshoot-policy-upload"></a>Résoudre les problèmes liés au chargement d'une stratégie

Lorsque vous essayez de publier une nouvelle stratégie personnalisée ou de mettre à jour une stratégie existante, une mise en forme XML incorrecte et des erreurs liées à la chaîne d'héritage du fichier de stratégie peuvent faire échouer la validation.

Par exemple, voici une tentative de mise à jour d'une stratégie dont le contenu présente une mise en forme XML incorrecte (la sortie est tronquée par souci de concision) :

```Console
PS C:\> Set-AzureADMSTrustFrameworkPolicy -Id B2C_1A_signup_signin -InputFilePath C:\B2C_1A_signup_signin.xml
Set-AzureADMSTrustFrameworkPolicy : Error occurred while executing PutTrustFrameworkPolicy
Code: AADB2C
Message: Validation failed: 1 validation error(s) found in policy "B2C_1A_SIGNUP_SIGNIN" of tenant "contosob2c.onmicrosoft.com".Schema validation error found at line
14 col 55 in policy "B2C_1A_SIGNUP_SIGNIN" of tenant "contosob2c.onmicrosoft.com": The element 'OutputClaims' in namespace
'http://schemas.microsoft.com/online/cpim/schemas/2013/06' cannot contain text. List of possible elements expected: 'OutputClaim' in namespace
'http://schemas.microsoft.com/online/cpim/schemas/2013/06'.
...
```

Pour plus d'informations sur la résolution des problèmes de stratégies personnalisées, consultez [Résoudre les problèmes liés aux stratégies personnalisées Azure AD B2C et à Identity Experience Framework](./troubleshoot-custom-policies.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations sur l'utilisation de PowerShell pour déployer des stratégies personnalisées dans le cadre d'un pipeline d'intégration/livraison continue (CI/CD), consultez [Déployer des stratégies personnalisées à partir d'un pipeline Azure DevOps](deploy-custom-policies-devops.md).

<!-- LINKS - External -->
[Connect-AzureAD]: /powershell/module/azuread/get-azureadmstrustframeworkpolicy
[Get-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/get-azureadmstrustframeworkpolicy
[New-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/new-azureadmstrustframeworkpolicy
[Remove-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/remove-azureadmstrustframeworkpolicy
[Set-AzureADMSTrustFrameworkPolicy]: /powershell/module/azuread/set-azureadmstrustframeworkpolicy
