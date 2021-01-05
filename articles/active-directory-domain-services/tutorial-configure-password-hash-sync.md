---
title: Activer la synchronisation de hachage du mot de passe pour Azure AD Domain Services | Microsoft Docs
description: Dans ce tutoriel, vous allez apprendre à activer la synchronisation de hachage du mot de passe à l’aide d’Azure AD Connect sur un domaine managé Azure Active Directory Domain Services.
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: e83d8941d1be7fd36f53a881a21716252ad01954
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618346"
---
# <a name="tutorial-enable-password-synchronization-in-azure-active-directory-domain-services-for-hybrid-environments"></a>Tutoriel : Activer la synchronisation du mot de passe dans Azure Active Directory Domain Services pour les environnements hybrides

Pour les environnements hybrides, un locataire Azure Active Directory (Azure AD) peut être configuré pour se synchroniser avec un environnement Active Directory Domain Services (AD DS) local à l’aide d’Azure AD Connect. Par défaut, Azure AD Connect ne synchronise pas les hachages de mot de passe NTLM (NT LAN Manager) et Kerberos existants demandés pour Azure Active Directory Domain Services (Azure AD DS).

Pour utiliser Azure AD DS avec des comptes synchronisés à partir d’un environnement AD DS local, vous devez configurer Azure AD Connect afin de synchroniser ces hachages de mot de passe nécessaires pour l’authentification NTLM et Kerberos. Une fois Azure AD Connect configuré, un événement de création de compte ou de modification de mot de passe local synchronise également les hachages de mot de passe existants avec Azure AD.

Vous n’avez pas besoin d’effectuer ces étapes si vous utilisez des comptes cloud uniquement sans environnement AD DS local ou si vous utilisez une *forêt de ressources*. Pour les domaines managés qui utilisent une forêt de ressources, les hachages de mots de passe ne sont jamais synchronisés. L’authentification des comptes locaux réutilise la ou les approbations de forêt pour vos propres contrôleurs de domaine AD DS.

Dans ce tutoriel, vous allez voir comment :

> [!div class="checklist"]
> * Découvrir pourquoi les hachages de mot de passe NTLM et Kerberos existants sont nécessaires
> * Configurer la synchronisation de hachage du mot de passe existant pour Azure AD Connect

Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce didacticiel, vous avez besoin des ressources suivantes :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement et qui est synchronisé avec un annuaire local à l’aide d’Azure AD Connect.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
    * Si nécessaire, [activez Azure AD Connect pour la synchronisation de hachage du mot de passe][enable-azure-ad-connect].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, [créez et configurez un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].

## <a name="password-hash-synchronization-using-azure-ad-connect"></a>Synchronisation de hachage du mot de passe à l’aide d’Azure AD Connect

Azure AD Connect est utilisé pour synchroniser des objets tels que des comptes et des groupes d’utilisateurs à partir d’un environnement AD DS local dans un locataire Azure AD. Dans le cadre du processus, la synchronisation de hachage du mot de passe permet à des comptes d’utiliser le même mot de passe dans l’environnement AD DS local et dans Azure AD.

Pour authentifier les utilisateurs sur le domaine managé, Azure AD DS nécessite les hachages de mot de passe dans un format adapté à l’authentification NTLM et Kerberos. Azure AD ne stocke pas les hachages de mot de passe au format exigé pour l’authentification NTLM ou Kerberos tant que vous n’activez pas Azure AD DS pour votre locataire. Pour des raisons de sécurité, Azure AD ne stocke pas non plus d’informations d’identification de mot de passe sous forme de texte en clair. Par conséquent, Azure AD ne peut pas générer automatiquement ces hachages de mot de passe NTLM ou Kerberos en fonction des informations d’identification existantes des utilisateurs.

Azure AD Connect peut être configuré pour synchroniser les hachages de mot de passe NTLM ou Kerberos exigés pour Azure AD DS. Vérifiez que vous avez effectué les étapes nécessaires pour [activer Azure AD Connect pour la synchronisation de hachage de mot de passe][enable-azure-ad-connect]. Si vous aviez une instance d’Azure AD Connect, [téléchargez et mettez à jour la dernière version][azure-ad-connect-download] pour garantir que vous pouvez synchroniser les hachages de mot de passe existant pour NTLM et Kerberos. Cette fonctionnalité n’est pas disponible dans les versions antérieures d’Azure AD Connect ou avec l’outil DirSync existant. Azure AD Connect version *1.1.614.0* ou ultérieure est nécessaire.

> [!IMPORTANT]
> Azure AD Connect doit uniquement être installé et configuré pour la synchronisation avec des environnements AD DS locaux. L’installation d’Azure AD Connect n’est pas prise en charge dans un domaine managé Azure AD DS pour resynchroniser des objets sur Azure AD.

## <a name="enable-synchronization-of-password-hashes"></a>Activer la synchronisation des hachages de mot de passe

Une fois Azure AD Connect installé et configuré pour se synchroniser avec Azure AD, configurez la synchronisation du hachage de mot de passe existante pour NTLM et Kerberos. Un script PowerShell est utilisé pour configurer les paramètres obligatoires, puis démarrer une synchronisation de mot de passe complète avec Azure AD. Une fois ce processus de synchronisation du hachage de mot de passe Azure AD Connect terminé, les utilisateurs peuvent se connecter aux applications avec Azure AD DS qui utilisent des hachages de mot de passe NTLM ou Kerberos existants.

1. Sur l’ordinateur sur lequel Azure AD Connect est installé, dans le menu Démarrer, ouvrez **Azure AD Connect > Service de synchronisation** .
1. Sélectionnez l’onglet **Connecteurs**. Les informations de connexion utilisées pour établir la synchronisation entre l’environnement AD DS local et Azure AD sont listées.

    Le **Type** indique *Windows Azure Active Directory (Microsoft)* pour le connecteur Azure AD ou *Active Directory Domain Services* pour le connecteur AD DS local. Notez les noms de connecteurs à utiliser dans le script PowerShell à l’étape suivante.

    ![Lister les noms de connecteurs dans Sync Service Manager](media/tutorial-configure-password-hash-sync/service-sync-manager.png)

    Dans cet exemple de capture d’écran, les connecteurs suivants sont utilisés :

    * Le connecteur Azure AD est nommé *contoso.onmicrosoft.com - AAD*
    * Le connecteur AD DS local est nommé *onprem.contoso.com*

1. Copiez et collez le script PowerShell suivant sur l’ordinateur sur lequel Azure AD Connect est installé. Le script déclenche une synchronisation de mot de passe complète qui inclut les hachages de mot de passe existants. Mettez à jour les variables `$azureadConnector` et `$adConnector` avec les noms de connecteurs notés à l’étape précédente.

    Exécutez ce script sur chaque forêt AD pour synchroniser les hachages de mot de passe NTLM et Kerberos de compte local avec Azure AD.

    ```powershell
    # Define the Azure AD Connect connector names and import the required PowerShell module
    $azureadConnector = "<CASE SENSITIVE AZURE AD CONNECTOR NAME>"
    $adConnector = "<CASE SENSITIVE AD DS CONNECTOR NAME>"
    
    Import-Module "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync\ADSync.psd1"
    Import-Module "C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\AdSyncConfig.psm1"

    # Create a new ForceFullPasswordSync configuration parameter object then
    # update the existing connector with this new configuration
    $c = Get-ADSyncConnector -Name $adConnector
    $p = New-Object Microsoft.IdentityManagement.PowerShell.ObjectModel.ConfigurationParameter "Microsoft.Synchronize.ForceFullPasswordSync", String, ConnectorGlobal, $null, $null, $null
    $p.Value = 1
    $c.GlobalParameters.Remove($p.Name)
    $c.GlobalParameters.Add($p)
    $c = Add-ADSyncConnector -Connector $c

    # Disable and re-enable Azure AD Connect to force a full password synchronization
    Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $azureadConnector -Enable $false
    Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $azureadConnector -Enable $true
    ```

    En fonction de la taille de votre annuaire en termes de nombre de comptes et de groupes, la synchronisation des hachages de mot de passe existants avec Azure AD peut prendre un certain temps. Après avoir été synchronisés avec Azure AD, les mots de passe sont ensuite synchronisés avec le domaine managé.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Découvrir pourquoi les hachages de mot de passe NTLM et Kerberos existants sont nécessaires
> * Configurer la synchronisation de hachage du mot de passe existant pour Azure AD Connect

> [!div class="nextstepaction"]
> [Découvrir comment fonctionne la synchronisation dans un domaine managé Azure AD Domain Services](synchronization.md)

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[enable-azure-ad-connect]: ../active-directory/hybrid/how-to-connect-install-express.md

<!-- EXTERNAL LINKS -->
[azure-ad-connect-download]: https://www.microsoft.com/download/details.aspx?id=47594
