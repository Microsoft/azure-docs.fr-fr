---
title: Résoudre les problèmes de connexion dans Azure Active Directory Domain Services | Microsoft Docs
description: Découvrez comment résoudre les erreurs courantes de connexion utilisateur dans les services Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 85b261b8754f35c7705690d15671144b858c0a43
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618567"
---
# <a name="troubleshoot-account-sign-in-problems-with-an-azure-active-directory-domain-services-managed-domain"></a>Résoudre les problèmes de connexion au compte avec un domaine managé Azure Active Directory Domain Services

Les raisons les plus courantes pour lesquelles un compte utilisateur ne peut pas se connecter à un domaine Azure AD DS (Azure Active Directory) incluent les scénarios suivants :

* [Le compte n’est pas encore synchronisé avec Azure AD DS.](#account-isnt-synchronized-into-azure-ad-ds-yet)
* [Azure AD DS ne dispose pas des hachages de mot de passe pour permettre au compte de se connecter.](#azure-ad-ds-doesnt-have-the-password-hashes)
* [Le compte est verrouillé.](#the-account-is-locked-out)

> [!TIP]
> Azure AD DS ne peut pas synchroniser les informations d’identification des comptes qui sont externes au locataire Azure AD. Les utilisateurs externes ne peuvent pas se connecter au domaine managé Azure AD DS.

## <a name="account-isnt-synchronized-into-azure-ad-ds-yet"></a>Le compte n’est pas encore synchronisé avec Azure AD DS

Selon la taille de votre répertoire, la mise à disposition des comptes d’utilisateurs et des hachages d’informations d’identification dans le domaine managé peut prendre un certain temps. Pour les annuaires de grande taille, cette synchronisation unidirectionnelle initiale à partir d’Azure AD peut prendre entre quelques heures et un ou deux jours. Patientez suffisamment longtemps avant d’effectuer une nouvelle tentative d’authentification.

Pour les environnements hybrides qui utilisent Azure AD Connect pour synchroniser les données d’annuaire locales avec Azure AD, assurez-vous que vous utilisez la dernière version d’Azure AD Connect et que vous avez [configuré Azure AD Connect pour effectuer une synchronisation complète après avoir activé Azure AD DS][azure-ad-connect-phs]. Si vous désactivez Azure AD DS et que vous le réactivez, vous devez suivre ces étapes à nouveau.

Si vous continuez d’avoir des problèmes avec des comptes qui ne se synchronisent pas via Azure AD Connect, redémarrez le service Azure AD Sync. À partir de l’ordinateur sur lequel Azure AD Connect est installé, ouvrez une fenêtre d’invite de commande et exécutez les commandes suivantes :

```console
net stop 'Microsoft Azure AD Sync'
net start 'Microsoft Azure AD Sync'
```

## <a name="azure-ad-ds-doesnt-have-the-password-hashes"></a>Azure AD DS ne dispose pas des hachages de mot de passe

Azure AD ne génère pas et ne stocke pas les hachages de mot de passe au format nécessaire pour l’authentification NTLM ou Kerberos tant que vous n’activez pas Azure AD DS pour votre locataire. Pour des raisons de sécurité, Azure AD ne stocke pas non plus d’informations d’identification de mot de passe sous forme de texte en clair. Par conséquent, Azure AD ne peut pas générer automatiquement ces hachages de mot de passe NTLM ou Kerberos en fonction des informations d’identification existantes des utilisateurs.

### <a name="hybrid-environments-with-on-premises-synchronization"></a>Environnements hybrides avec synchronisation locale

Pour les environnements hybrides utilisant Azure AD Connect pour la synchronisation à partir d’un environnement AD DS local, vous pouvez générer et synchroniser localement les hachages de mots de passe NTLM ou Kerberos requis dans Azure AD. Après avoir créé un domaine managé, [activez la synchronisation du hachage de mot de passe pour Azure Active Directory Domain Services][azure-ad-connect-phs]. Sans terminer cette étape de synchronisation de hachage de mot de passe, vous ne pouvez pas vous connecter à un compte à l’aide d’un domaine managé. Si vous désactivez Azure AD DS et que vous le réactivez, vous devez suivre ces étapes à nouveau.

Pour plus d’informations, consultez la rubrique concernant [le fonctionnement de la synchronisation de hachage du mot de passe pour Azure AD DS][phs-process].

### <a name="cloud-only-environments-with-no-on-premises-synchronization"></a>Environnements cloud uniquement sans synchronisation locale

Les domaines managés sans synchronisation locale (comptes Azure AD uniquement) doivent également générer les hachages de mot de passe NTLM ou Kerberos requis. Si un compte cloud uniquement ne peut pas se connecter, le mot de passe a-t-il été mis à jour après l’activation d’Azure AD DS ?

* **Non, le mot de passe n’a pas été modifié.**
    * [Modifiez le mot de passe du compte][enable-user-accounts] pour générer les hachages de mot de passe requis, puis attendez 15 minutes avant d’essayer de vous reconnecter.
    * Si vous désactivez Azure AD DS, puis le réactivez, chaque compte doit suivre à nouveau la procédure de changement du mot de passe et de génération du hachage de mot de passe requise.
* **Oui, le mot de passe a été modifié.**
    * Essayez de vous connecter en utilisant le format *UPN*, tel que `driley@aaddscontoso.com`, et non pas le format *SAMAccountName*, tel que `AADDSCONTOSO\deeriley`.
    * Le format *SAMAccountName* peut être généré automatiquement pour les utilisateurs dont le préfixe UPN est trop long ou identique à un autre utilisateur sur le domaine managé. Le format *UPN* garantit des données uniques au sein d’Azure AD.

## <a name="the-account-is-locked-out"></a>Le compte est verrouillé

Un compte utilisateur dans un domaine managé est verrouillé lorsqu’un seuil défini pour les tentatives de connexion infructueuses a été atteint. Ce comportement de verrouillage de compte est conçu pour vous protéger contre les tentatives répétées de connexion par force brute qui pourraient indiquer une attaque numérique automatisée.

Par défaut, si cinq tentatives de mot de passe incorrectes ont lieu en l’espace de deux minutes, le compte est verrouillé pendant 30 minutes.

Pour plus d’informations et pour savoir comment résoudre les problèmes de blocage de compte, consultez la rubrique [Troubleshoot account lockout problems in Azure AD DS][troubleshoot-account-lockout] (Résolution des problèmes de blocage de compte dans Azure AD DS).

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez toujours des problèmes pour joindre votre machine virtuelle au domaine managé, [demandez de l’aide et ouvrez un ticket de support pour Azure Active Directory][azure-ad-support].

<!-- INTERNAL LINKS -->
[troubleshoot-account-lockout]: troubleshoot-account-lockout.md
[azure-ad-connect-phs]: ./tutorial-configure-password-hash-sync.md
[enable-user-accounts]:  tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds
[phs-process]: ../active-directory/hybrid/how-to-connect-password-hash-synchronization.md#password-hash-sync-process-for-azure-ad-domain-services
[azure-ad-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md