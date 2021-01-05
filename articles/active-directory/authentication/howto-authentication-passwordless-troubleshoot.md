---
title: Problèmes connus et résolution des problèmes de clés de sécurité FIDO2 hybrides – Azure Active Directory
description: En savoir plus sur certains problèmes connus et sur les méthodes de résolution des problèmes de connexion par clé de sécurité FIDO2 hybride sans mot de passe en utilisant Azure Active Directory (préversion)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: troubleshooting
ms.date: 08/19/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: aakapo
ms.collection: M365-identity-device-management
ms.openlocfilehash: ea034974e85febcffeb2494d87b666a39e524eb1
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743273"
---
# <a name="troubleshooting-for-hybrid-deployments-of-fido2-security-keys-in-azure-ad-preview"></a>Résolution des problèmes liés aux déploiements hybrides de clés de sécurité FIDO2 dans Azure AD (préversion)

Cet article aborde les foires aux questions relatives aux appareils avec jointure hybride Azure AD et à la connexion sans mot de passe aux ressources locales. Grâce à cette fonctionnalité sans mot de passe, vous pouvez activer l’authentification Azure AD sur appareils Windows 10 pour les appareils avec jointure hybride Azure AD utilisant des clés de sécurité FIDO2. Les utilisateurs peuvent se connecter à Windows sur leurs appareils à l’aide d’informations d’identification modernes telles que des clés FIDO2 et accéder à des ressources traditionnelles basées sur Active Directory Domain Services (AD DS) avec une expérience d’authentification unique (SSO) transparente pour leurs ressources locales.

Pour les utilisateurs d’un environnement hybride, les scénarios suivants sont pris en charge :

* Se connecter à des appareils avec jointure hybride Azure AD à l’aide de clés de sécurité FIDO2 et bénéficier d’un accès SSO aux ressources locales.
* Se connecter à des appareils joints à Azure AD à l’aide de clés de sécurité FIDO2 et bénéficier d’un accès SSO aux ressources locales.

Pour vous familiariser avec les clés de sécurité FIDO2 et l’accès hybride aux ressources locales, consultez les articles suivants :

* [Clés de sécurité sans mot de passe](howto-authentication-passwordless-security-key.md)
* [Windows 10 sans mot de passe](howto-authentication-passwordless-security-key-windows.md)
* [Ressources locales sans mot de passe](howto-authentication-passwordless-security-key-on-premises.md)

> [!NOTE]
> Les clés de sécurité FIDO2 sont une fonctionnalité d’évaluation publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="known-issues"></a>Problèmes connus

* [Les utilisateurs ne parviennent pas à se connecter à l’aide de clés de sécurité FIDO2, car Reconnaissance des visages Windows Hello est trop rapide et constitue le mécanisme de connexion par défaut](#users-are-unable-to-sign-in-using-fido2-security-keys-as-windows-hello-face-is-too-quick-and-is-the-default-sign-in-mechanism)
* [Les utilisateurs ne peuvent pas utiliser les clés de sécurité FIDO2 immédiatement après avoir créé une machine avec jointure hybride Azure AD](#users-arent-able-to-use-fido2-security-keys-immediately-after-they-create-a-hybrid-azure-ad-joined-machine)
* [Les utilisateurs ne peuvent pas effectuer l’authentification unique auprès de leur ressource réseau NTLM après s’être connecté avec une clé de sécurité FIDO2, et ils reçoivent une invite à entrer des informations d’identification](#users-are-unable-to-get-sso-to-my-ntlm-network-resource-after-signing-in-with-a-fido2-security-key-and-receiving-a-credential-prompt)

### <a name="users-are-unable-to-sign-in-using-fido2-security-keys-as-windows-hello-face-is-too-quick-and-is-the-default-sign-in-mechanism"></a>Les utilisateurs ne parviennent pas à se connecter à l’aide de clés de sécurité FIDO2, car Reconnaissance des visages Windows Hello est trop rapide et constitue le mécanisme de connexion par défaut

Reconnaissance des visages Windows Hello est la meilleure expérience prévue pour un appareil sur lequel un utilisateur est inscrit. Les clés de sécurité FIDO2 sont destinées à être utilisées sur des appareils partagés ou lorsque l’inscription Windows Hello Entreprise est un obstacle.

Si Reconnaissance des visages Windows Hello empêche les utilisateurs d’essayer le scénario de connexion par clé de sécurité FIDO2, les utilisateurs peuvent désactiver la connexion par reconnaissance des visages Windows Hello en supprimant l’inscription de la reconnaissance des visages dans **Paramètres > Options de connexion**.

### <a name="users-arent-able-to-use-fido2-security-keys-immediately-after-they-create-a-hybrid-azure-ad-joined-machine"></a>Les utilisateurs ne peuvent pas utiliser les clés de sécurité FIDO2 immédiatement après avoir créé une machine avec jointure hybride Azure AD

Après le processus de jonction de domaine et de redémarrage sur une nouvelle installation de machine avec jointure hybride Azure AD, vous devez vous connecter avec un mot de passe et attendre la synchronisation de la stratégie avant de pouvoir utiliser une clé de sécurité FIDO2 pour vous connecter.

Ce comportement est une limitation connue des appareils joints à un domaine et n’est pas spécifique aux clés de sécurité FIDO2.

Pour vérifier l’état actuel, utilisez la commande `dsregcmd /status`. Vérifiez qu’*AzureAdJoined* et *DomainJoined* ont la valeur *YES*.

### <a name="users-are-unable-to-get-sso-to-my-ntlm-network-resource-after-signing-in-with-a-fido2-security-key-and-receiving-a-credential-prompt"></a>Les utilisateurs ne peuvent pas effectuer l’authentification unique auprès de leur ressource réseau NTLM après s’être connecté avec une clé de sécurité FIDO2, et ils reçoivent une invite à entrer des informations d’identification

Assurez-vous qu’un nombre suffisant de contrôleurs de domaine sont corrigés pour répondre dans les temps afin de servir votre demande de ressource. Pour vérifier si vous pouvez voir un serveur qui exécute la fonctionnalité, passez en revue la sortie de `nltest /dsgetdc:<dc name> /keylist /kdc`.

Si vous voyez un contrôleur de domaine doté de cette fonctionnalité, le mot de passe de l’utilisateur a peut-être été modifié depuis qu’il s’est connecté, ou il y a un autre problème. Rassemblez les journaux comme indiqué dans la section suivante pour que l’équipe du support technique de Microsoft puisse déboguer le problème.

## <a name="troubleshoot"></a>Dépanner

Il existe deux aspects à dépanner : les [problèmes liés au client Windows](#windows-client-issues) ou les [problèmes de déploiement](#deployment-issues).

### <a name="windows-client-issues"></a>Problèmes liés au client Windows

Pour collecter des données permettant de résoudre les problèmes de connexion à Windows ou d’accès aux ressources locales à partir d’appareils Windows 10, procédez comme suit :

1. Ouvrez l’application **Hub de commentaires**. Assurez-vous que votre nom se trouve dans le coin inférieur gauche de l’application, puis sélectionnez **Créer un nouvel élément de commentaires**.

    Pour le type d’élément de commentaires, choisissez *Problème*.

1. Sélectionnez la catégorie *Sécurité et confidentialité*, puis la sous-catégorie *FIDO*.
1. Cochez la case *Envoyer les fichiers joints et les diagnostics à Microsoft avec mes commentaires*.
1. Sélectionnez *Recréer mes problèmes*, puis *Démarrer la capture*.
1. Verrouillez et déverrouillez la machine avec la clé de sécurité FIDO2. Si le problème se produit, essayez de la déverrouiller avec d’autres informations d’identification.
1. Revenez à **Hub de commentaires**, sélectionnez **Arrêter la capture**, puis envoyez vos commentaires.
1. Accédez à la page *Commentaires*, puis à l’onglet *Mes commentaires*. Sélectionnez les commentaires que vous avez envoyés récemment.
1. Sélectionnez le bouton *Partager* dans le coin supérieur droit pour obtenir un lien vers les commentaires. Incluez ce lien lorsque vous ouvrez un dossier de support ou répondez à l’ingénieur affecté à un dossier de support existant.

Les journaux d’événements et les informations de clé de Registre suivants sont collectés :

**Clés de Registre**

* *HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\FIDO [\*]*
* *HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\PasswordForWork\* [\*]*
* *HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Policies\PasswordForWork\* [\*]*

**Informations de diagnostic**

* Transfert mémoire du noyau actif
* Informations du package AppX de collecte
* Fichiers de contexte UIF

**Journaux d’événements**

* *%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-AAD%40Operational.evtx*
* *%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-WebAuthN%40Operational.evtx*
* *%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-HelloForBusiness%40Operational.evtx*

### <a name="deployment-issues"></a>Problèmes de déploiement

Pour résoudre les problèmes de déploiement du serveur Kerberos Azure AD, utilisez le nouveau module PowerShell fourni avec Azure AD Connect.

#### <a name="viewing-the-logs"></a>Affichage des journaux

Les cmdlets PowerShell pour serveur Kerberos Azure AD utilisent la même journalisation que l’Assistant Azure AD Connect standard. Pour afficher les informations ou les détails de l’erreur à partir des cmdlets, procédez comme suit :

1. Sur le serveur Azure AD Connect, accédez à `C:\ProgramData\AADConnect\`. Ce dossier est masqué par défaut.
1. Ouvrez et consultez le fichier `trace-*.log` le plus récent situé dans le répertoire.

#### <a name="viewing-the-azure-ad-kerberos-server-objects"></a>Visualisation des objets serveur Kerberos Azure AD

Pour afficher les objets serveur Kerberos Azure AD et vérifier que tout est en ordre, procédez comme suit :

1. Sur le serveur Azure AD Connect, ouvrez PowerShell et accédez à `C:\Program Files\Microsoft Azure Active Directory Connect\AzureADKerberos\`.
1. Exécutez les commandes PowerShell suivantes pour afficher le serveur Kerberos Azure AD à partir d’Azure AD et de l’instance AD DS locale.

    Remplacez *corp.contoso.com* par le nom de votre domaine AD DS local.

    ```powershell
    Import-Module ".\AzureAdKerberos.psd1"

    # Specify the on-premises AD DS domain.
    $domain = "corp.contoso.com"

    # Enter an Azure Active Directory Global Admin username and password.
    $cloudCred = Get-Credential

    # Enter a Domain Admin username and password.
    $domainCred = Get-Credential

    # Get the Azure AD Kerberos Server Object
    Get-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential
    $domainCred
    ```

La commande affiche les propriétés du serveur Kerberos Azure AD à partir d’Azure AD et de l’instance AD DS locale. Passez en revue les propriétés pour vérifier que tout est en ordre. Utilisez le tableau ci-dessous pour vérifier les propriétés.

Le premier ensemble de propriétés provient des objets de l’environnement AD DS local. La seconde moitié (les propriétés qui commencent par *Cloud**) proviennent de l’objet serveur Kerberos dans Azure AD :

| Property           | Description  |
|--------------------|--------------|
| Id                 | *ID* unique de l’objet contrôleur de domaine AD DS. |
| DomainDnsName      | Nom de domaine DNS du domaine AD DS. |
| ComputerAccount    | Objet compte d’ordinateur de l’objet serveur Kerberos Azure AD (le contrôleur de domaine). |
| UserAccount        | Objet compte d’utilisateur désactivé contenant la clé de chiffrement du TGT du serveur Kerberos Azure AD. Le nom de domaine de ce compte est *CN=krbtgt_AzureAD,CN=Users,<Domain-DN>* . |
| KeyVersion         | Version de clé de la clé de chiffrement du TGT du serveur Kerberos Azure AD. La version est attribuée lors de la création de la clé. La version est ensuite incrémentée à chaque rotation de la clé. Les incréments sont basés sur des métadonnées de réplication et seront probablement supérieurs à un.<br /><br /> Par exemple, la *KeyVersion* initiale pourrait être *192272*. À la première rotation de la clé, la version peut passer à *212621*.<br /><br /> L’élément important à vérifier est que la *KeyVersion* de l’objet local et la *CloudKeyVersion* de l’objet cloud sont identiques. |
| KeyUpdatedOn       | Date et heure auxquelles la clé de chiffrement du TGT du serveur Kerberos Azure AD a été mise à jour ou créée. |
| KeyUpdatedFrom     | Contrôleur de domaine où la clé de chiffrement du TGT du serveur Kerberos Azure AD a été mise à jour pour la dernière fois. |
| CloudId            | *ID* de l’objet Azure AD. Doit correspondre à l’*ID* ci-dessus. |
| CloudDomainDnsName | *DomainDnsName* de l’objet Azure AD. Doit correspondre à la valeur *DomainDnsName* ci-dessus. |
| CloudKeyVersion    | *KeyVersion* de l’objet Azure AD. Doit correspondre à la valeur *KeyVersion* ci-dessus. |
| CloudKeyUpdatedOn  | *KeyUpdatedOn* de l’objet Azure AD. Doit correspondre à la valeur *KeyUpdatedOn* ci-dessus. |

## <a name="next-steps"></a>Étapes suivantes

Pour vous familiariser avec les clés de sécurité FIDO2 et l’accès hybride aux ressources locales, consultez les articles suivants :

* [Clés de sécurité FIDO2 sans mot de passe](howto-authentication-passwordless-security-key.md)
* [Windows 10 sans mot de passe](howto-authentication-passwordless-security-key-windows.md)
* [En local sans mot de passe](howto-authentication-passwordless-security-key-on-premises.md)
