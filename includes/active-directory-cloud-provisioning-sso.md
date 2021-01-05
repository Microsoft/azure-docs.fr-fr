---
author: billmath
ms.service: active-directory
ms.subservice: cloud-provisioning
ms.topic: include
ms.date: 10/16/2019
ms.author: billmath
ms.openlocfilehash: 6d95e40623f17a39145778a2fc067dccc68fd872
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95560027"
---
## <a name="steps-to-enable-single-sign-on"></a>Procédure d’activation de l’authentification unique
L’approvisionnement cloud fonctionne avec l’authentification unique.  Il n’existe actuellement aucune option permettant d’activer l’authentification unique lors de l’installation de l’agent. Toutefois, vous pouvez utiliser les étapes ci-dessous pour activer l’authentification unique et l’utiliser. 

### <a name="step-1-download-and-extract-azure-ad-connect-files"></a>Étape 1 : Télécharger et extraire des fichiers Azure AD Connect
1.  Commencez par télécharger la dernière version d’[Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594).
2.  Ouvrez une invite de commandes en utilisant des privilèges Administrateur, puis accédez au msi que vous venez de télécharger.
3.  Exécutez la commande suivante : `msiexec /a C:\filepath\AzureADConnect.msi /qb TARGETDIR=C:\filepath\extractfolder`
4. Modifiez FilePath et extractfolder de façon à ce qu’ils correspondent au chemin d’accès de votre fichier et au nom de votre dossier d’extraction.  Le contenu doit maintenant se trouver dans le dossier d’extraction.

### <a name="step-2-import-the-seamless-sso-powershell-module"></a>Étape 2 : Importer le module PowerShell Authentification unique (SSO) transparente

1. Téléchargez et installez [Azure AD PowerShell V2](/powershell/azure/active-directory/overview).
2. Accédez au dossier `%programfiles%\Microsoft Azure Active Directory Connect`.
3. Importez le module PowerShell Authentification unique (SSO) transparente à l’aide de la commande suivante : `Import-Module .\AzureADSSO.psd1`.

### <a name="step-3-get-the-list-of-active-directory-forests-on-which-seamless-sso-has-been-enabled"></a>Étape 3 : Obtenir la liste des forêts Azure Directory dans lesquelles l’authentification unique (SSO) transparente a été activée

1. Exécutez PowerShell ISE en tant qu’administrateur. Dans PowerShell, appelez `New-AzureADSSOAuthenticationContext`. Lorsque vous y êtes invité, fournissez les informations d’identification de l’administrateur général de votre locataire.
2. Appelez `Get-AzureADSSOStatus`. Cette commande vous fournit la liste des forêts Azure Directory (examinez la liste « Domaines ») dans lesquelles cette fonctionnalité a été activée.

### <a name="step-4-enable-seamless-sso-for-each-active-directory-forest"></a>Étape 4 : Activer l’authentification unique (SSO) transparente pour chaque forêt Azure Directory

1. Appelez `Enable-AzureADSSOForest`. Quand vous y êtes invité, entrez les informations d’identification d’administrateur de domaine pour la forêt Azure Directory souhaitée.

   > [!NOTE]
   >Le nom d’utilisateur des informations d’identification de l’administrateur de domaine doit être entré au format de nom de compte SAM (contoso\johndoe ou contoso.com\johndoe). Nous utilisons la partie domaine du nom d’utilisateur pour localiser le contrôleur de domaine de l’administrateur de domaine à l’aide de DNS.

   >[!NOTE]
   >Le compte administrateur de domaine utilisé ne doit pas être membre du groupe Utilisateurs protégés. Sinon, l’opération échoue.

2. Répétez l'étape précédente pour chaque forêt Azure Directory dans laquelle vous souhaitez configurer la fonctionnalité.

### <a name="step-5-enable-the-feature-on-your-tenant"></a>Étape 5. Activer la fonctionnalité pour votre locataire

Pour activer la fonctionnalité pour votre locataire, appelez `Enable-AzureADSSO -Enable $true`.