---
title: Gérer les autorisations des utilisateurs et des administrateurs – Azure Active Directory | Microsoft Docs
description: Apprenez à examiner et gérer les autorisations relatives à une application sur Azure AD. Par exemple, révoquez toutes les autorisations octroyées à une application.
services: active-directory
author: mimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 7/10/2020
ms.author: mimart
ms.reviewer: luleonpla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6ff97d0a69efbe624e959f92f5320f921476a306
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658976"
---
# <a name="take-action-on-overprivileged-or-suspicious-applications-in-azure-active-directory"></a>Mesures relatives aux applications surprivilégiées ou suspectes dans Azure Active Directory.

Apprenez à examiner et gérer les autorisations relatives à une application. Cet article propose différentes mesures pour sécuriser votre application en fonction du scénario. Ces mesures s’appliquent à toutes les applications qui ont été ajoutées à votre locataire Azure Active Directory (Azure AD), avec consentement utilisateur ou administrateur.

Pour plus d’informations sur le consentement des applications, consultez [Infrastructure de consentement d’Azure Active Directory](../develop/consent-framework.md).

## <a name="prerequisites"></a>Prérequis

Pour effectuer les actions suivantes, vous devez vous connecter en tant qu’administrateur général, administrateur d’application ou administrateur d’application cloud.

Pour restreindre l’accès aux applications, vous devez demander l’affectation d’utilisateurs, puis affecter des utilisateurs ou des groupes à l’application.  Pour plus d'informations, voir [Méthodes d'affectation d'utilisateurs et de groupes](./assign-user-or-group-access-portal.md).

Vous pouvez accéder au portail Azure AD pour obtenir des scripts PowerShell contextuels permettant d'appliquer les mesures.
 
1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général, administrateur d’application ou administrateur d’application cloud.
2. Sélectionnez **Azure Active Directory (Azure Active Directory)**  > **Enterprise applications (Applications d’entreprise)** .
3. Sélectionnez l’application dont vous souhaitez restreindre l’accès.
4. Sélectionnez **Autorisations**. Sur la barre de commandes, sélectionnez **Passer en revue les autorisations**.

![Capture d’écran de la fenêtre Passer en revue les autorisations.](./media/manage-application-permissions/review-permissions.png)


## <a name="control-access-to-an-application"></a>Contrôler l’accès à une application

Nous vous recommandons de restreindre l’accès à l’application en activant le paramètre **Affectation d’utilisateurs**.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général, administrateur d’application ou administrateur d’application cloud.
2. Sélectionnez **Azure Active Directory (Azure Active Directory)**  > **Enterprise applications (Applications d’entreprise)** .
3. Sélectionnez l’application dont vous souhaitez restreindre l’accès.
4. Sélectionnez **Propriétés**, puis définissez **Droits d’accès requis pour l’utilisateur** sur **Oui**.
5. Sélectionnez **Utilisateur et groupes**, puis supprimez les utilisateurs indésirables qui sont affectés à l’application.
6. Affectez des utilisateurs ou des groupes à l’application.

Si vous le souhaitez, vous pouvez supprimer tous les utilisateurs affectés à l’application à l’aide de PowerShell.

## <a name="revoke-all-permissions-for-an-application"></a>Révoquer toutes les autorisations relatives à une application

L’utilisation du script PowerShell révoque toutes les autorisations octroyées à cette application.

> [!NOTE]
> La révocation de l’autorisation actuellement octroyée n’empêchera pas les utilisateurs de consentir à nouveau à l’application. Si vous souhaitez empêcher les utilisateurs de donner leur consentement, consultez [Configurer la manière dont les utilisateurs consentent aux applications](configure-user-consent.md).

Si vous le souhaitez, vous pouvez désactiver l’application pour empêcher les utilisateurs d’y accéder et pour empêcher l’application d’accéder à vos données.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général, administrateur d’application ou administrateur d’application cloud.
2. Sélectionnez **Azure Active Directory (Azure Active Directory)**  > **Enterprise applications (Applications d’entreprise)** .
3. Sélectionnez l’application dont vous souhaitez restreindre l’accès.
4. Sélectionnez **Propriétés**, puis définissez **Connexion permise pour les utilisateurs ?** sur **Non**.

## <a name="investigate-a-suspicious-application"></a>Examiner une application suspecte

Nous vous recommandons de restreindre l’accès à l’application en activant le paramètre **Affectation d’utilisateurs**. Examinez ensuite les autorisations que les utilisateurs et les administrateurs ont accordées à l’application.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général, administrateur d’application ou administrateur d’application cloud.
3. Sélectionnez **Azure Active Directory (Azure Active Directory)**  > **Enterprise applications (Applications d’entreprise)** .
5. Sélectionnez l’application dont vous souhaitez restreindre l’accès.
6. Sélectionnez **Propriétés**, puis définissez **Droits d’accès requis pour l’utilisateur** sur **Oui**.
7. Sélectionnez **Autorisations** et vérifiez les autorisations consenties par l’administrateur et l’utilisateur.

Éventuellement, à l’aide de PowerShell, vous pouvez :

- Supprimer tous les utilisateurs attribués pour les empêcher de se connecter à l’application.
- Invalider les jetons d’actualisation des utilisateurs qui ont accès à l’application.
- Révoquer toutes les autorisations relatives à l’application.

Sinon ,vous pouvez désactiver l’application pour bloquer l’accès des utilisateurs et empêcher celle-ci d’accéder à vos données.


## <a name="disable-a-malicious-application"></a>Désactiver une application malveillante 

Nous vous recommandons de désactiver l’application pour bloquer l’accès des utilisateurs et empêcher l’application d’accéder à vos données. Si vous supprimez l’application, les utilisateurs peuvent de nouveau consentir à celle-ci et octroyer l’accès à vos données.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général, administrateur d’application ou administrateur d’application cloud.
2. Sélectionnez **Azure Active Directory (Azure Active Directory)**  > **Enterprise applications (Applications d’entreprise)** .
3. Sélectionnez l’application dont vous souhaitez restreindre l’accès.
4. Sélectionnez **Propriétés**, puis copiez l’ID d’objet.

### <a name="powershell-commands"></a>Commandes PowerShell


Récupérez l’ID d’objet du principal de service.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général, administrateur d’application ou administrateur d’application cloud.
2. Sélectionnez **Azure Active Directory (Azure Active Directory)**  > **Enterprise applications (Applications d’entreprise)** .
3. Sélectionnez l’application dont vous souhaitez restreindre l’accès.
4. Sélectionnez **Propriétés**, puis copiez l’ID d’objet.

```powershell
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
    $sp.ObjectId
```
Supprimez tous les utilisateurs affectés à l’application.
 ```powershell
    Connect-AzureAD

    # Get Service Principal using objectId
    $sp = Get-AzureADServicePrincipal -ObjectId "<ServicePrincipal objectID>"

    # Get Azure AD App role assignments using objectId of the Service Principal
    $assignments = Get-AzureADServiceAppRoleAssignment -ObjectId $sp.ObjectId -All $true

    # Remove all users and groups assigned to the application
    $assignments | ForEach-Object {
        if ($_.PrincipalType -eq "User") {
            Remove-AzureADUserAppRoleAssignment -ObjectId $_.PrincipalId -AppRoleAssignmentId $_.ObjectId
        } elseif ($_.PrincipalType -eq "Group") {
            Remove-AzureADGroupAppRoleAssignment -ObjectId $_.PrincipalId -AppRoleAssignmentId $_.ObjectId
        }
    }
 ```

Révoquez les autorisations octroyées à l’application.

```powershell
    Connect-AzureAD

    # Get Service Principal using objectId
    $sp = Get-AzureADServicePrincipal -ObjectId "<ServicePrincipal objectID>"

    # Get all delegated permissions for the service principal
    $spOAuth2PermissionsGrants = Get-AzureADOAuth2PermissionGrant -All $true| Where-Object { $_.clientId -eq $sp.ObjectId }

    # Remove all delegated permissions
    $spOAuth2PermissionsGrants | ForEach-Object {
        Remove-AzureADOAuth2PermissionGrant -ObjectId $_.ObjectId
    }

    # Get all application permissions for the service principal
    $spApplicationPermissions = Get-AzureADServiceAppRoleAssignedTo -ObjectId $sp.ObjectId -All $true | Where-Object { $_.PrincipalType -eq "ServicePrincipal" }

    # Remove all delegated permissions
    $spApplicationPermissions | ForEach-Object {
        Remove-AzureADServiceAppRoleAssignment -ObjectId $_.PrincipalId -AppRoleAssignmentId $_.objectId
    }
```
Invalidez les jetons d’actualisation.
```powershell
        Connect-AzureAD

        # Get Service Principal using objectId
        $sp = Get-AzureADServicePrincipal -ObjectId "<ServicePrincipal objectID>"

        # Get Azure AD App role assignments using objectID of the Service Principal
        $assignments = Get-AzureADServiceAppRoleAssignment -ObjectId $sp.ObjectId -All $true | Where-Object {$_.PrincipalType -eq "User"}

        # Revoke refresh token for all users assigned to the application
        $assignments | ForEach-Object {
            Revoke-AzureADUserAllRefreshToken -ObjectId $_.PrincipalId
        }
```
## <a name="next-steps"></a>Étapes suivantes
- [Gérer le consentement pour les applications et évaluer les demandes de consentement](manage-consent-requests.md)
- [Configurer le consentement de l'utilisateur](configure-user-consent.md)
- [Configurer le workflow du consentement administrateur](configure-admin-consent-workflow.md)