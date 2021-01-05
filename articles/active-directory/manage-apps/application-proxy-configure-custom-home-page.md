---
title: Personnaliser une page d’accueil pour les applications publiées - Proxy d’application Azure AD
description: Couvre les bases sur les connecteurs de proxy d’application Azure AD.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/23/2019
ms.author: kenwith
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 155a91ded6c814e2d868e8edd4572459460d006f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88642059"
---
# <a name="set-a-custom-home-page-for-published-apps-by-using-azure-ad-application-proxy"></a>Définir une page d’accueil personnalisée pour les applications publiées à l’aide du proxy d’application Azure AD

Cet article explique comment configurer une applications pour diriger un utilisateur vers une page d’accueil personnalisée. Lorsque vous publiez une application avec le proxy d’application, vous définissez une URL interne, mais parfois, ce n’est pas la page que l’utilisateur doit d’abord voir. Définissez une page d’accueil personnalisée afin que l’utilisateur tombe sur la page adéquate quand il accède à l’application. Un utilisateur voit la page d’accueil personnalisée que vous avez définie, qu’il accède à l’application à partir de Mes applications Azure Active Directory ou du lanceur d’applications Microsoft 365.

Quand un utilisateur lance l’application, il est dirigé par défaut vers l’URL du domaine racine de l’application publiée. La page d’arrivée est généralement définie comme URL de page d’accueil. Le module PowerShell Azure AD permet de définir une URL de la page d’accueil personnalisée lorsque vous souhaitez que l’utilisateur de l’application arrive sur une page spécifique au sein de l’application.

Voici un scénario expliquant pourquoi votre entreprise pourrait vouloir définir une page d’accueil personnalisée :

- Sur votre réseau d’entreprise, un utilisateur accède à `https://ExpenseApp/login/login.aspx` pour se connecter et accéder à votre application.
- Comme vous disposez d’autres ressources (comme des images) dont le proxy d’application a besoin pour accéder au niveau supérieur de la structure de dossiers, vous publiez l’application avec `https://ExpenseApp` en tant qu’URL interne.
- L’URL externe par défaut, `https://ExpenseApp-contoso.msappproxy.net`, n’envoie pas un utilisateur externe vers la page de connexion.
- Vous préférez définir `https://ExpenseApp-contoso.msappproxy.net/login/login.aspx` comme URL de la page d’accueil de sorte qu’un utilisateur externe voie d’abord la page de connexion.

> [!NOTE]
> Si vous octroyez aux utilisateurs un accès aux applications publiées, celles-ci sont affichées dans [Mes applications](../user-help/my-apps-portal-end-user-access.md) et le [lanceur d’applications Office 365](https://www.microsoft.com/microsoft-365/blog/2016/09/27/introducing-the-new-office-365-app-launcher/).

## <a name="before-you-start"></a>Avant de commencer

Avant de définir l’URL de page d’accueil, n’oubliez pas les exigences suivantes :

- Le chemin d’accès que vous spécifiez doit être un chemin d’accès de sous-domaine de l’URL du domaine racine.

  Par exemple, si l’URL du domaine racine est `https://apps.contoso.com/app1/`, l’URL de la page d’accueil que vous configurez doit commencer par `https://apps.contoso.com/app1/`.

- Si vous apportez une modification à l’application publiée, ce changement peut réinitialiser la valeur de l’URL de page d’accueil. Ultérieurement, lorsque vous mettez à jour l’application, vous devez revérifier et, si nécessaire, mettre à jour l’URL de page d’accueil.

Vous pouvez définir l’URL de la page d’accueil via le portail Azure ou à l’aide de PowerShell.

## <a name="change-the-home-page-in-the-azure-portal"></a>Changer la page d’accueil du portail Azure

Pour modifier l’URL de la page d’accueil de votre application via le portail Azure AD, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur.
1. Sélectionnez **Azure Active Directory**, puis **Inscriptions d’applications**. La liste des applications inscrites s’affiche.
1. Choisissez votre application dans la liste. Une page présentant les détails de l’application inscrite s’affiche.
1. Dans **Gérer**, sélectionnez **Personnalisation**.
1. Mettez à jour le champ **URL de la page d’accueil** avec votre nouveau chemin d’accès.

   ![Page Personnalisation pour une application inscrite, affichant le champ URL de la page d’accueil](media/application-proxy-configure-custom-home-page/app-proxy-app-branding.png)

1. Sélectionnez **Enregistrer**.

## <a name="change-the-home-page-with-powershell"></a>Changer la page d’accueil à l’aide de PowerShell

Pour configurer la page d’accueil d’une application à l’aide de PowerShell, vous devez :

1. Installez le module Azure AD PowerShell.
1. Rechercher la valeur ObjectID de l’application.
1. Mettre à jour l’URL de la page d’accueil de l’application à l’aide de commandes PowerShell.

### <a name="install-the-azure-ad-powershell-module"></a>Installer le module Azure AD PowerShell

Avant de définir une URL de page d’accueil personnalisée à l’aide de PowerShell, installez le module Azure AD PowerShell. Vous pouvez télécharger le package à partir de [PowerShell Gallery](https://www.powershellgallery.com/packages/AzureAD/2.0.2.16), qui utilise le point de terminaison de l’API Graph.

Pour installer le package, procédez comme suit :

1. Ouvrez une fenêtre PowerShell standard, puis exécutez la commande suivante :

   ```powershell
   Install-Module -Name AzureAD
   ```

    Si vous l’exécutez en tant que non-administrateur, utilisez l’option `-scope currentuser`.

1. Pendant l’installation, sélectionnez **Y** pour installer deux packages depuis Nuget.org. Les deux packages sont requis.

### <a name="find-the-objectid-of-the-app"></a>Rechercher l’ObjectID de l’application

Pour obtenir l’ObjectID de l’application, recherchez l’application par son nom d’affichage ou sa page d’accueil.

1. Dans la même fenêtre PowerShell, importez le module Azure AD.

   ```powershell
   Import-Module AzureAD
   ```

1. Connectez-vous au module Azure AD en tant qu’administrateur client.

   ```powershell
   Connect-AzureAD
   ```

1. Recherchez l’application. Cet exemple utilise PowerShell pour rechercher l’ObjectID en recherchant l’application pas un nom d’affichage de `SharePoint`.

   ```powershell
   Get-AzureADApplication | Where-Object { $_.DisplayName -eq "SharePoint" } | Format-List DisplayName, Homepage, ObjectId
   ```

   Vous devez obtenir un résultat semblable à celui qui est illustré ici. Copiez le GUID ObjectID à utiliser dans la section suivante.

   ```console
   DisplayName : SharePoint
   Homepage    : https://sharepoint-iddemo.msappproxy.net/
   ObjectId    : 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4
   ```

   Vous pouvez également extraire la liste de toutes les applications, rechercher dans celle-ci l’application avec un nom d’affichage ou une page d’accueil spécifiques, et copier l’ObjectID de l’application une fois celle-ci trouvée.

   ```powershell
   Get-AzureADApplication | Format-List DisplayName, Homepage, ObjectId
   ```

### <a name="update-the-home-page-url"></a>Mettre à jour l’URL de la page d’accueil

Créez l’URL de la page d’accueil et mettez à jour votre application avec cette valeur. Continuez à utiliser la même fenêtre PowerShell ou, si vous utilisez une nouvelle fenêtre PowerShell, connectez-vous au module Azure AD à l’aide de `Connect-AzureAD`. Exécutez ensuite les opérations qui suivent :

1. Créez une variable pour contenir la valeur ObjectID que vous avez copiée dans la section précédente (remplacez la valeur ObjectID utilisée dans cet exemple SharePoint par la valeur ObjectID de votre application).

   ```powershell
   $objguid = "8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4"
   ```

1. Vérifiez que vous avez la bonne application en exécutant la commande suivante. La sortie doit être identique à celle que vous avez vue dans la section précédente ([Rechercher l’ObjectID de l’application](#find-the-objectid-of-the-app)).

   ```powershell
   Get-AzureADApplication -ObjectId $objguid | Format-List DisplayName, Homepage, ObjectId
   ```

1. Créez un objet d’application vide pour conserver les modifications que vous souhaitez apporter.

   ```powershell
   $appnew = New-Object "Microsoft.Open.AzureAD.Model.Application"
   ```

1. Définissez l’URL de page d’accueil sur la valeur souhaitée. La valeur doit correspondre à un chemin d’accès de sous-domaine de l’application publiée. Par exemple, si vous changez l’URL de la page d’accueil de `https://sharepoint-iddemo.msappproxy.net/` en `https://sharepoint-iddemo.msappproxy.net/hybrid/`, les utilisateurs de l’application accèdent directement à la page d’accueil personnalisée.

   ```powershell
   $homepage = "https://sharepoint-iddemo.msappproxy.net/hybrid/"
   ```

1. Actualisez la page d’accueil.

   ```powershell
   Set-AzureADApplication -ObjectId $objguid -Homepage $homepage
   ```

1. Pour vérifier que la modification a réussi, réexécutez la commande suivante de l’étape 2.

   ```powershell
   Get-AzureADApplication -ObjectId $objguid | Format-List DisplayName, Homepage, ObjectId
   ```

   Dans notre exemple, la sortie doit maintenant apparaître comme suit :

   ```console
   DisplayName : SharePoint
   Homepage    : https://sharepoint-iddemo.msappproxy.net/hybrid/
   ObjectId    : 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4
   ```

1. Redémarrez l’application pour vérifier que la page d’accueil s’affiche en tant que premier écran, comme prévu.

> [!NOTE]
> Les modifications que vous apportez à l’application peuvent réinitialiser l’URL de page d’accueil. Si l’URL de votre page d’accueil est réinitialisée, répétez les étapes décrites dans cette section pour la rétablir.

## <a name="next-steps"></a>Étapes suivantes

- [Activer l’accès distant pour SharePoint avec le proxy d’application Azure AD](application-proxy-integrate-with-sharepoint-server.md)
- [Tutoriel : Ajouter une application locale pour un accès à distance via le proxy d’application d’Azure Active Directory](application-proxy-add-on-premises-application.md)
