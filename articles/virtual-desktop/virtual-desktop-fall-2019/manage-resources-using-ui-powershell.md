---
title: Déployer un outil de gestion pour Windows Virtual Desktop (classique) en utilisant le principal de service - Azure
description: Guide pratique pour l’outil de gestion pour Windows Virtual Desktop (classique) avec PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
ms.custom: devx-track-azurepowershell
manager: lizross
ms.openlocfilehash: d7219751d584eb458cded9f4e30cccb1439dfa1b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89069015"
---
# <a name="deploy-a-windows-virtual-desktop-classic-management-tool-with-powershell"></a>Déployer un outil de gestion de Windows Virtual Desktop (classique) avec PowerShell

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager.

Cet article vous indique comment déployer l’outil de gestion avec PowerShell.

## <a name="important-considerations"></a>Points importants à prendre en compte

Chaque abonnement du locataire Azure Active Directory (Azure AD) nécessite son propre déploiement de l’outil de gestion. Cet outil ne prend pas en charge les scénarios Azure AD B2B (Business-to-Business).

Cet outil de gestion est un exemple. Microsoft fournira des mises à jour importantes pour la qualité et la sécurité. [Le code source est disponible sur GitHub](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/wvd-management-ux/deploy). Que vous soyez client ou partenaire, nous vous encourageons à personnaliser l’outil pour répondre aux besoins de votre entreprise.

Les navigateurs suivants sont compatibles avec l’outil de gestion :

- Google Chrome version 68 ou ultérieure
- Microsoft Edge version 40.15063 ou ultérieure
- Mozilla Firefox version 52.0 ou ultérieure
- Safari version 10 ou ultérieure (macOS uniquement)

## <a name="what-you-need-to-deploy-the-management-tool"></a>Ce dont vous avez besoin pour déployer l’outil de gestion

Avant de déployer l’outil de gestion, vous avez besoin d’un utilisateur Azure Active Directory (Azure AD) pour créer une inscription d’application et déployer l’interface utilisateur de gestion. Cet utilisateur doit remplir les conditions suivantes :

- Il doit avoir l’autorisation de créer des ressources dans votre abonnement Azure.
- Il doit avoir l’autorisation de créer une application Azure AD. Suivez ces étapes pour vérifier si votre utilisateur dispose des autorisations nécessaires en suivant les instructions fournies dans [Autorisations nécessaires](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

Une fois que vous avez déployé et configuré l’outil de gestion, nous vous recommandons de demander à un utilisateur de lancer l’interface utilisateur de gestion pour vérifier que tout fonctionne correctement. L’utilisateur qui lance l’interface utilisateur de gestion doit avoir une attribution de rôle qui lui permet de voir ou de modifier le locataire Windows Virtual Desktop.

## <a name="set-up-powershell"></a>Configurer PowerShell

Commencez par vous connecter aux modules PowerShell Az et Azure AD. Voici comment se connecter :

1. Ouvrez PowerShell en tant qu’administrateur, puis accédez au répertoire où vous avez enregistré les scripts PowerShell.
2. Connectez-vous à Azure avec un compte disposant des autorisations Propriétaire ou Contributeur sur l’abonnement Azure que vous prévoyez d’utiliser pour créer l’outil de gestion en exécutant l’applet de commande suivante :

    ```powershell
    Login-AzAccount
    ```

3. Exécutez l’applet de commande suivante pour vous connecter à Azure AD avec le même compte que celui utilisé pour le module PowerShell Az :

    ```powershell
    Connect-AzureAD
    ```

4. Ensuite, accédez au dossier où vous avez enregistré les deux scripts PowerShell provenant du dépôt GitHub RDS-Templates.

Gardez ouverte la fenêtre PowerShell que vous avez utilisée pour vous connecter pour pouvoir exécuter d’autres applets de commande PowerShell pendant que vous êtes connecté.

## <a name="create-an-azure-active-directory-app-registration"></a>Créer une inscription d’application Azure Active Directory

Pour déployer et configurer correctement l’outil de gestion, vous devez d’abord télécharger les scripts PowerShell suivants à partir du [référentiel GitHub RDS-Template](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/wvd-management-ux/deploy/scripts).

```powershell
Set-Location -Path "c:\temp"
$uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-management-ux/deploy/scripts/createWvdMgmtUxAppRegistration.ps1"
Invoke-WebRequest -Uri $uri -OutFile ".\createWvdMgmtUxAppRegistration.ps1"
$uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-management-ux/deploy/scripts/updateWvdMgmtUxApiUrl.ps1"
Invoke-WebRequest -Uri $uri -OutFile ".\updateWvdMgmtUxApiUrl.ps1"
```

Exécutez les commandes suivantes pour créer l’inscription d’application avec les autorisations d’API nécessaires :

```powershell
$appName = Read-Host -Prompt "Enter a unique name for the management tool's app registration. The name can't contain spaces or special characters."
$azureSubscription = Get-AzSubscription | Out-GridView -PassThru
$subscriptionId = $azureSubscription.Id
Get-AzSubscription -SubscriptionId $subscriptionId | Select-AzSubscription

.\createWvdMgmtUxAppRegistration.ps1 -AppName $appName -SubscriptionId $subscriptionId
```

Maintenant que vous avez terminé l’inscription de l’application Azure AD, vous pouvez déployer l’outil de gestion.

## <a name="deploy-the-management-tool"></a>Déployer l’outil de gestion

Exécutez les commandes PowerShell suivantes pour déployer l’outil de gestion et l’associer au principal de service que vous venez de créer :

```powershell
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateParameters = @{}
$templateParameters.Add('isServicePrincipal', $true)
$templateParameters.Add('azureAdminUserPrincipalNameOrApplicationId', $ServicePrincipalCredentials.UserName)
$templateParameters.Add('azureAdminPassword', $servicePrincipalCredentials.Password)
$templateParameters.Add('applicationName', $appName)

Get-AzSubscription -SubscriptionId $subscriptionId | Select-AzSubscription
New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateUri "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-management-ux/deploy/mainTemplate.json" `
    -TemplateParameterObject $templateParameters `
    -Verbose
```

Une fois que vous avez créé l’application web, vous devez ajouter un URI de redirection à l’application Azure AD pour que les utilisateurs puissent se connecter correctement.

## <a name="set-the-redirect-uri"></a>Définie l’URI de redirection

Exécutez les commandes PowerShell suivantes pour récupérer l’URL de l’application web et la définir en tant qu’URI de redirection de l’authentification (également appelée URL de réponse) :

```powershell
$webApp = Get-AzWebApp -ResourceGroupName $resourceGroupName -Name $appName
$redirectUri = "https://" + $webApp.DefaultHostName + "/"
Get-AzureADApplication -All $true | where { $_.AppId -match $servicePrincipalCredentials.UserName } | Set-AzureADApplication -ReplyUrls $redirectUri
```

Maintenant que vous avez ajouté un URI de redirection, vous devez ensuite mettre à jour l’URL de l’API pour que l’outil de gestion puisse interagir avec le service back-end d’API.

## <a name="update-the-api-url-for-the-web-application"></a>Mettre à jour l’URL de l’API pour l’application web

Exécutez le script suivant pour mettre à jour la configuration de l’URL de l’API dans le front-end de l’application web :

```powershell
.\updateWvdMgmtUxApiUrl.ps1 -AppName $appName -SubscriptionId $subscriptionId
```

Maintenant que vous avez entièrement configuré l’application web de l’outil de gestion, il est temps de vérifier l’application Azure AD et de fournir un consentement.

## <a name="verify-the-azure-ad-application-and-provide-consent"></a>Vérifier l’application Azure AD et fournir un consentement

Pour vérifier l’application Azure AD et fournir un consentement :

1. Ouvrez votre navigateur Internet et connectez-vous au [Portail Azure](https://portal.azure.com/) avec votre compte d’administrateur.
2. Dans la barre de recherche en haut du portail Azure, recherchez **Inscriptions d’applications** et sélectionnez l’élément sous **Services**.
3. Sélectionnez **Toutes les applications** et recherchez le nom unique de l’application que vous avez fourni pour le script PowerShell dans [Créer une inscription d’application Azure Active Directory](#create-an-azure-active-directory-app-registration).
4. Dans le volet gauche du navigateur, sélectionnez **Authentification** et vérifiez que l’URI de redirection est identique à l’URL de l’application Web pour l’outil de gestion, comme illustré dans l’image suivante.

   [ ![Page d’authentification avec l’URI de redirection entré](../media/management-ui-redirect-uri-inline.png) ](../media/management-ui-redirect-uri-expanded.png#lightbox)

5. Dans le volet gauche, sélectionnez **Autorisations de l’API** pour vérifier que les autorisations ont été ajoutées. Si vous êtes un administrateur général, sélectionnez le bouton **Accorder un consentement d’administrateur pour `tenantname`** et suivez les invites de la boîte de dialogue pour fournir le consentement administrateur pour votre organisation.

    [ ![La page des autorisations de l’API](../media/management-ui-permissions-inline.png) ](../media/management-ui-permissions-expanded.png#lightbox)

Vous pouvez maintenant commencer à utiliser l’outil de gestion.

## <a name="use-the-management-tool"></a>Utiliser l’outil de gestion

Maintenant que vous avez configuré l’outil de gestion, vous pouvez le lancer à tout moment, où que vous soyez. Voici comment lancer l’outil :

1. Ouvrez l’URL de l’application web dans un navigateur web. Si vous ne vous souvenez pas de l’URL, vous pouvez vous connecter à Azure, rechercher le service d’application que vous avez déployé pour l’outil de gestion, puis sélectionner l’URL.
2. Connectez-vous avec vos informations d’identification Windows Virtual Desktop.

   > [!NOTE]
   > Si vous n’avez pas accordé le consentement de l’administrateur lors de la configuration de l’outil de gestion, chaque utilisateur qui se connecte doit fournir son propre consentement d’utilisateur pour pouvoir utiliser l’outil.

3. Quand vous êtes invité à choisir un groupe de locataires, sélectionnez **Groupe de locataires par défaut** dans la liste déroulante.
4. Quand vous sélectionnez **Groupe de locataires par défaut**, un menu doit s’afficher du côté gauche de votre fenêtre. Dans ce menu, recherchez le nom de votre groupe de locataires et sélectionnez-le.

   > [!NOTE]
   > Si vous avez un groupe de locataires personnalisé, entrez le nom manuellement au lieu de choisir dans la liste déroulante.

## <a name="report-issues"></a>Signaler des problèmes

Si vous rencontrez des problèmes avec l’outil de gestion ou d’autres outils Windows Virtual Desktop, suivez les instructions de [Modèles Azure Resource Manager pour les services Bureau à distance](https://github.com/Azure/RDS-Templates/blob/master/README.md) pour les signaler sur GitHub.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert comment déployer et vous connecter à l’outil de gestion, vous pouvez découvrir comment utiliser Azure Service Health pour superviser les problèmes des services et les alertes d’intégrité. Pour plus d’informations, consultez notre [Tutoriel de configuration des alertes de service](set-up-service-alerts-2019.md).
