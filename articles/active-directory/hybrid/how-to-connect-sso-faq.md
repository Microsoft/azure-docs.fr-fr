---
title: 'Azure AD Connect : Authentification unique transparente - Questions fréquentes | Microsoft Docs'
description: Réponse à des questions fréquentes sur l’authentification unique transparente Azure Active Directory.
services: active-directory
keywords: Qu’est-ce qu’Azure AD Connect, Installation d’Active Directory, Composants requis pour Azure AD, SSO, Authentification unique
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/07/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9a3e2dd1ac58c89060fc3e09e97617cb2a68d8ac
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97094571"
---
# <a name="azure-active-directory-seamless-single-sign-on-frequently-asked-questions"></a>Authentification unique transparente Azure Active Directory : Forum aux questions

Dans cet article, nous répondons au forum aux questions sur l’authentification unique et transparente Azure Active Directory. N’hésitez pas à le consulter régulièrement, du contenu nouveau y est fréquemment ajouté.

**Q : Avec quelles méthodes de connexion l'authentification unique transparente est-elle compatible ?**

L’authentification unique transparente peut être combinée avec la [synchronisation de hachage de mot de passe](how-to-connect-password-hash-synchronization.md) et [l’authentification directe](how-to-connect-pta.md). Toutefois cette fonctionnalité ne peut pas être utilisée avec les services de fédération Active Directory (ADFS).

**Q : La fonctionnalité d'authentification unique transparente est-elle gratuite ?**

L’authentification unique transparente est une fonctionnalité gratuite et il n’est pas utile de disposer des éditions payantes d’Azure AD pour l’utiliser.

**Q : L'authentification unique transparente est-elle disponible dans le [cloud Microsoft Azure Allemagne](https://www.microsoft.de/cloud-deutschland) et dans le [cloud Microsoft Azure Government](https://azure.microsoft.com/features/gov/) ?**

L’authentification unique fluide est disponible pour le cloud Azure Government. Pour plus d’informations, consultez [Considérations sur les identités hybrides pour Azure Government](./reference-connect-government-cloud.md).

**Q : Quelles sont les applications qui tirent parti des paramètres `domain_hint` et `login_hint` de l'authentification unique transparente ?**

Vous trouverez ci-dessous une liste non exhaustive des applications qui envoient ces paramètres à Azure AD et permettent donc aux utilisateurs de se connecter de manière silencieuse à l’aide de l’authentification unique transparente (par ex. les utilisateurs n’ont pas besoin d’entrer leur nom d’utilisateur ou mot de passe) :

| Nom de l'application | URL d’application à utiliser |
| -- | -- |
| Panneau d’accès | https:\//myapps.microsoft.com/contoso.com |
| Outlook sur le Web | https:\//outlook.office365.com/contoso.com |
| Portails Office 365 | https:\//portal.office.com?domain_hint=contoso.com, https:\//www.office.com?domain_hint=contoso.com |

En outre, les utilisateurs bénéficient d'une expérience de connexion silencieuse si une application envoie des demandes de connexion à des points de terminaison d'Azure AD configurés en tant que locataires (https:\//login.microsoftonline.com/contoso.com/<..> ou https:\//login.microsoftonline.com/<ID_locataire>/<..>) au lieu du point de terminaison commun d'Azure AD, à savoir https:\//login.microsoftonline.com/common/<...>. Vous trouverez ci-dessous une liste non exhaustive d’applications qui rendent ces types de requêtes de connexion.

| Nom de l'application | URL d’application à utiliser |
| -- | -- |
| SharePoint Online | https:\//contoso.sharepoint.com |
| Portail Azure | https:\//portal.azure.com/contoso.com |

Dans les tables ci-dessus, remplacez « contoso.com » par votre nom de domaine pour obtenir les URL d’application de votre abonné.

Si vous souhaitez que d’autres applications utilisent notre expérience d’authentification en mode silencieux, faites-le nous savoir dans la section des commentaires.

**Q : L'authentification unique transparente prend-elle en charge `Alternate ID` comme nom d'utilisateur à la place de `userPrincipalName` ?**

Oui. L’authentification unique transparente prend en charge `Alternate ID` comme nom d’utilisateur dans Azure AD Connect comme indiqué [ici](how-to-connect-install-custom.md). Toutes les applications Microsoft 365 ne prennent pas en charge `Alternate ID`. Reportez-vous à la documentation de l’application qui vous intéresse pour avoir des précisions sur sa prise en charge.

**Q : Quelle est la différence entre l'expérience d'authentification unique fournie par [Azure AD Join](../devices/overview.md) et l'authentification unique transparente ?**

[Azure AD Join](../devices/overview.md) fournit l’authentification unique aux utilisateurs si leurs appareils sont inscrits auprès d’Azure AD. Ces appareils ne doivent pas nécessairement être joints au domaine. L’authentification unique est fournie à l’aide de *jetons d’actualisation principaux* ou *PRTs*, et non Kerberos. L’expérience utilisateur est optimale sur des appareils Windows 10. L’authentification unique s’effectue automatiquement sur le navigateur Microsoft Edge. Elle fonctionne également sur Chrome avec une extension de navigateur.

Vous pouvez utiliser Azure AD Join et l’authentification unique transparente sur votre client. Ces deux fonctionnalités sont complémentaires. Si les deux fonctionnalités sont activées, l’authentification unique à partir d’Azure AD Join est prioritaire sur l’authentification unique transparente.

**Q : Je souhaite inscrire des appareils non Windows 10 auprès d’Azure AD sans utiliser les services AD FS. Puis-je plutôt utiliser l'authentification unique transparente ?**

Oui, ce scénario nécessite la version 2.1 ou ultérieure du [client workplace-join](https://www.microsoft.com/download/details.aspx?id=53554).

**Q : Comment puis-je substituer la clé de déchiffrement Kerberos du compte d'ordinateur `AZUREADSSO` ?**

Il est important de fréquemment substituer la clé de déchiffrement Kerberos du `AZUREADSSO` compte d’ordinateur (c’est-à-dire Azure AD) créé dans votre forêt AD locale.

>[!IMPORTANT]
>Nous vous recommandons vivement de substituer la clé de déchiffrement Kerberos au moins tous les 30 jours.

Procédez comme suit sur le serveur local où vous exécutez Azure AD Connect :

   > [!NOTE]
   >Pour les étapes ci-dessous, vous aurez besoin des informations d’identification de l’administrateur de domaine et de l’administrateur général.

   **Étape 1. Obtenir la liste des forêts AD dans lesquelles l’authentification unique transparente a été activée**

   1. Commencez par télécharger et installer [Azure AD PowerShell](/powershell/azure/active-directory/overview).
   2. Accédez au dossier `%programfiles%\Microsoft Azure Active Directory Connect`.
   3. Importez le module PowerShell Authentification unique (SSO) transparente à l’aide de la commande suivante : `Import-Module .\AzureADSSO.psd1`.
   4. Exécutez PowerShell ISE en tant qu’administrateur. Dans PowerShell, appelez `New-AzureADSSOAuthenticationContext`. Cette commande doit afficher une fenêtre contextuelle dans laquelle vous devez entrer vos informations d’identification d’administrateur général de locataire.
   5. Appelez `Get-AzureADSSOStatus | ConvertFrom-Json`. Cette commande vous fournit la liste des forêts AD (examinez la liste « Domaines ») dans lesquelles cette fonctionnalité a été activée.

   **Étape 2. Mettez à jour la clé de déchiffrement Kerberos sur chaque forêt AD à laquelle elle a été attribuée.**

   1. Appelez `$creds = Get-Credential`. Quand vous y êtes invité, entrez les informations d’identification d’administrateur de domaine pour la forêt AD souhaitée.

   > [!NOTE]
   >Le nom d’utilisateur des informations d’identification de l’administrateur de domaine doit être entré au format de nom de compte SAM (contoso\johndoe ou contoso.com\johndoe). Nous utilisons la partie domaine du nom d’utilisateur pour localiser le contrôleur de domaine de l’administrateur de domaine à l’aide de DNS.

   >[!NOTE]
   >Le compte administrateur de domaine utilisé ne doit pas être membre du groupe Utilisateurs protégés. Sinon, l’opération échoue.

   2. Appelez `Update-AzureADSSOForest -OnPremCredentials $creds`. Cette commande met à jour la clé de déchiffrement de Kerberos pour le `AZUREADSSO` compte de l’ordinateur et la forêt AD spécifique et dans Azure AD.
   
   >[!NOTE]
   >Si vous n’êtes pas un administrateur de domaine et que vous avez reçu des autorisations par l’administrateur de domaine, vous devez appeler `Update-AzureADSSOForest -OnPremCredentials $creds -PreserveCustomPermissionsOnDesktopSsoAccount`
   
   3. Répétez les étapes précédentes pour chaque forêt AD dans laquelle vous avez configuré la fonctionnalité.
   
  >[!NOTE]
   >Si vous mettez à jour une forêt autre que celle d’Azure AD Connect, assurez-vous que la connectivité au serveur du catalogue global (TCP 3268 et TCP 3269) est disponible.

   >[!IMPORTANT]
   >Assurez-vous de _ne pas_ exécuter la `Update-AzureADSSOForest` commande plusieurs fois. Dans le cas contraire, la fonctionnalité cesse de fonctionner jusqu’à ce que les tickets Kerberos de vos utilisateurs expirent et soient régénérés par votre annuaire Active Directory local.

**Q : Comment désactiver l'authentification unique transparente ?**

   **Étape 1. Désactiver la fonctionnalité pour votre locataire**

   **Option A : Désactiver à l’aide d’Azure AD Connect**
    
   1. Exécutez Azure AD Connect, cliquez sur la **page Modifier la connexion utilisateur** puis sur **Suivant**.
   2. Désactivez l’option **Activer l’authentification unique**. Suivez les instructions de l’Assistant.

   À la fin de l’Assistant, l’authentification unique fluide sera désactivée pour votre locataire. Toutefois, le message suivant s’affichera :

   "L’authentification unique est désormais désactivée, mais des étapes manuelles supplémentaires restent à effectuer pour terminer le nettoyage. En savoir plus."

   Pour terminer le processus de nettoyage, suivez les étapes 2 et 3 sur le serveur local où vous exécutez Azure AD Connect.

   **Option B : Désactiver à l’aide de PowerShell**

   Exécutez les étapes suivantes sur le serveur local où vous exécutez Azure AD Connect :

   1. Commencez par télécharger et installer [Azure AD PowerShell](/powershell/azure/active-directory/overview).
   2. Accédez au dossier `%programfiles%\Microsoft Azure Active Directory Connect`.
   3. Importez le module PowerShell Authentification unique (SSO) transparente à l’aide de la commande suivante : `Import-Module .\AzureADSSO.psd1`.
   4. Exécutez PowerShell ISE en tant qu’administrateur. Dans PowerShell, appelez `New-AzureADSSOAuthenticationContext`. Cette commande doit afficher une fenêtre contextuelle dans laquelle vous devez entrer vos informations d’identification d’administrateur général de locataire.
   5. Appelez `Enable-AzureADSSO -Enable $false`.
   
   À ce stade, l’authentification unique fluide est désactivée, mais les domaines restent configurés au cas où vous souhaiteriez réactiver l’authentification unique fluide. Si vous souhaitez supprimer complètement les domaines de la configuration de l’authentification unique fluide, appelez l’applet de commande suivante après avoir terminé l’étape 5 ci-dessus : `Disable-AzureADSSOForest -DomainFqdn <fqdn>`.

   >[!IMPORTANT]
   >La désactivation de l’authentification unique fluide à l’aide de PowerShell ne modifiera pas l’état dans Azure AD Connect. L’authentification unique fluide apparaîtra comme étant activée dans la page **Modifier la connexion utilisateur**.

   **Étape 2. Obtenir la liste des forêts AD dans lesquelles l’authentification unique transparente a été activée**

   Suivez les tâches 1 à 4 ci-dessous si vous avez désactivé l’authentification unique fluide à l’aide d’Azure AD Connect. Si vous avez désactivé l’authentification unique fluide à l’aide de PowerShell, passez directement à la tâche 5 ci-dessous.

   1. Commencez par télécharger et installer [Azure AD PowerShell](/powershell/azure/active-directory/overview).
   2. Accédez au dossier `%programfiles%\Microsoft Azure Active Directory Connect`.
   3. Importez le module PowerShell Authentification unique (SSO) transparente à l’aide de la commande suivante : `Import-Module .\AzureADSSO.psd1`.
   4. Exécutez PowerShell ISE en tant qu’administrateur. Dans PowerShell, appelez `New-AzureADSSOAuthenticationContext`. Cette commande doit afficher une fenêtre contextuelle dans laquelle vous devez entrer vos informations d’identification d’administrateur général de locataire.
   5. Appelez `Get-AzureADSSOStatus | ConvertFrom-Json`. Cette commande vous fournit la liste des forêts AD (examinez la liste « Domaines ») dans lesquelles cette fonctionnalité a été activée.

   **Étape 3. Supprimez manuellement le compte d'ordinateur `AZUREADSSO` de chaque forêt AD répertoriée.**

## <a name="next-steps"></a>Étapes suivantes

- [**Démarrage rapide**](how-to-connect-sso-quick-start.md) : Découvrez l’authentification unique transparente Azure AD.
- [**Immersion technique**](how-to-connect-sso-how-it-works.md) : découvrez comment fonctionne cette fonctionnalité.
- [**Résolution des problèmes**](tshoot-connect-sso.md) : découvrez comment résoudre les problèmes courants susceptibles de survenir avec cette fonctionnalité.
- [**UserVoice**](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) : pour le dépôt de nouvelles demandes de fonctionnalités.
