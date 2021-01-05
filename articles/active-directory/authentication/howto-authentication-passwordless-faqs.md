---
title: FAQ relatives au déploiement de clés de sécurité FIDO2 hybrides – Azure Active Directory
description: En savoir plus sur la foire aux questions concernant la connexion par clé de sécurité FIDO2 hybride sans mot de passe en utilisant Azure Active Directory (préversion)
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
ms.openlocfilehash: 98cb990ede7c4d6e261bba05b0b8c97d758e6c32
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743528"
---
# <a name="deployment-frequently-asked-questions-faqs-for-hybrid-fido2-security-keys-in-azure-ad-preview"></a>Foires aux questions (FAQ) relatives au déploiement de clés de sécurité FIDO2 hybrides dans Azure AD (préversion)

Cet article aborde les foires aux questions (FAQ) relatives aux appareils avec jointure hybride Azure AD et la connexion sans mot de passe aux ressources locales. Grâce à cette fonctionnalité sans mot de passe, vous pouvez activer l’authentification Azure AD sur appareils Windows 10 pour les appareils avec jointure hybride Azure AD utilisant des clés de sécurité FIDO2. Les utilisateurs peuvent se connecter à Windows sur leurs appareils à l’aide d’informations d’identification modernes telles que des clés FIDO2 et accéder à des ressources traditionnelles basées sur Active Directory Domain Services (AD DS) avec une expérience d’authentification unique (SSO) transparente pour leurs ressources locales.

Pour les utilisateurs d’un environnement hybride, les scénarios suivants sont pris en charge :

* Se connecter à des appareils avec jointure hybride Azure AD à l’aide de clés de sécurité FIDO2 et bénéficier d’un accès SSO aux ressources locales.
* Se connecter à des appareils joints à Azure AD à l’aide de clés de sécurité FIDO2 et bénéficier d’un accès SSO aux ressources locales.

Pour vous familiariser avec les clés de sécurité FIDO2 et l’accès hybride aux ressources locales, consultez les articles suivants :

* [Clés de sécurité FIDO2 sans mot de passe](howto-authentication-passwordless-security-key.md)
* [Windows 10 sans mot de passe](howto-authentication-passwordless-security-key-windows.md)
* [En local sans mot de passe](howto-authentication-passwordless-security-key-on-premises.md)

> [!NOTE]
> Les clés de sécurité FIDO2 sont une fonctionnalité d’évaluation publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="security-keys"></a>Clés de sécurité

* [Mon organisation requiert une authentification à deux facteurs pour accéder aux ressources. Que puis-je faire pour prendre en charge cette exigence ?](#my-organization-requires-multi-factor-authentication-to-access-resources-what-can-i-do-to-support-this-requirement)
* [Où puis-je trouver des clés de sécurité FIDO2 conformes ?](#where-can-i-find-compliant-fido2-security-keys)
* [Que dois-je faire si je perds ma clé de sécurité ?](#what-if-i-lose-my-security-key)
* [Comment les données sont-elles protégées sur la clé de sécurité FIDO2 ?](#how-is-the-data-protected-on-the-fido2-security-key)
* [Comment l’inscription des clés de sécurité FIDO2 fonctionne-t-il ?](#how-does-the-registering-of-fido2-security-keys-work)
* [Existe-t-il un moyen pour les administrateurs d’approvisionner les clés directement pour les utilisateurs ?](#is-there-a-way-for-admins-to-provision-the-keys-for-the-users-directly)

### <a name="my-organization-requires-multi-factor-authentication-to-access-resources-what-can-i-do-to-support-this-requirement"></a>Mon organisation requiert une authentification multifacteur pour accéder aux ressources. Que puis-je faire pour prendre en charge cette exigence ?

Les clés de sécurité FIDO2 sont disponibles dans différents facteurs de forme. Contactez le fabricant de l’appareil pour savoir comment le paramétrer afin d’utiliser un code PIN ou un accès biométrique comme deuxième facteur. Pour obtenir la liste des fournisseurs pris en charge, consultez [Fournisseurs de clés de sécurité FIDO2](concept-authentication-passwordless.md#fido2-security-key-providers).

### <a name="where-can-i-find-compliant-fido2-security-keys"></a>Où puis-je trouver des clés de sécurité FIDO2 conformes ?

Pour obtenir la liste des fournisseurs pris en charge, consultez [Fournisseurs de clés de sécurité FIDO2](concept-authentication-passwordless.md#fido2-security-key-providers).

### <a name="what-if-i-lose-my-security-key"></a>Que se passe-t-il si je perds ma clé de sécurité ?

Vous pouvez supprimer des clés dans le portail Azure en accédant à la page **Informations relatives à la sécurité** et en supprimant les clés de sécurité FIDO2.

### <a name="how-is-the-data-protected-on-the-fido2-security-key"></a>Comment les données sont-elles protégées sur la clé de sécurité FIDO2 ?

Les clés de sécurité FIDO2 ont des enclaves sécurisées qui protègent les clés privées qui y sont stockées. Une clé de sécurité FIDO2 a également des propriétés anti-martèlement intégrées, comme dans Windows Hello, où vous ne pouvez pas extraire la clé privée.

### <a name="how-does-the-registering-of-fido2-security-keys-work"></a>Comment l’inscription des clés de sécurité FIDO2 fonctionne-t-il ?

Pour plus d’informations sur l’inscription et l’utilisation de clés de sécurité FIDO2, consultez [Activer la connexion par clé de sécurité sans mot de passe](howto-authentication-passwordless-security-key.md).

### <a name="is-there-a-way-for-admins-to-provision-the-keys-for-the-users-directly"></a>Existe-t-il un moyen pour les administrateurs d’approvisionner les clés directement pour les utilisateurs ?

Non, pas pour l’instant.

## <a name="prerequisites"></a>Prérequis

* [Cette fonctionnalité fonctionne-t-elle en l’absence de connexion Internet ?](#does-this-feature-work-if-theres-no-internet-connectivity)
* [Quels sont les points de terminaison spécifiques qui doivent être ouverts pour Azure AD ?](#what-are-the-specific-end-points-that-are-required-to-be-open-to-azure-ad)
* [Comment identifier le type de jonction de domaine (jointure Azure AD ou jointure hybride Azure AD) pour mon appareil Windows 10 ?](#how-do-i-identify-the-domain-join-type-azure-ad-joined-or-hybrid-azure-ad-joined-for-my-windows-10-device)
* [Quelle est la recommandation en matière de nombre de contrôleurs de domaine qui doivent être corrigés ?](#whats-the-recommendation-on-the-number-of-dcs-that-should-be-patched)
* [Puis-je déployer le fournisseur d’informations d’identification FIDO2 sur un appareil uniquement local ?](#can-i-deploy-the-fido2-credential-provider-on-an-on-premises-only-device)
* [La connexion par clé de sécurité FIDO2 ne fonctionne pas pour mon compte Administrateur de domaine ou d’autres comptes à privilèges élevés. Pourquoi ?](#fido2-security-key-sign-in-isnt-working-for-my-domain-admin-or-other-high-privilege-accounts-why)

### <a name="does-this-feature-work-if-theres-no-internet-connectivity"></a>Cette fonctionnalité fonctionne-t-elle en l’absence de connexion Internet ?

La connectivité Internet est une condition préalable à l’activation de cette fonctionnalité. La première fois qu’un utilisateur se connecte à l’aide de clés de sécurité FIDO2, il doit disposer d’une connexion Internet. Pour les événements de connexion suivants, la connexion mise en cache doit fonctionner et permettre à l’utilisateur de s’authentifier sans connexion à Internet.

Pour une expérience cohérente, assurez-vous que les appareils disposent d’un accès Internet et d’une ligne de vue sur les contrôleurs de domaine.

### <a name="what-are-the-specific-end-points-that-are-required-to-be-open-to-azure-ad"></a>Quels sont les points de terminaison spécifiques qui doivent être ouverts pour Azure AD ?

Les points de terminaison suivants sont nécessaires pour l’inscription et l’authentification :

* * *.microsoftonline.com*
* * *.microsoftonline-p.com*
* * *.msauth.net*
* * *.msauthimages.net*
* * *.msecnd.net*
* * *.msftauth.net*
* * *.msftauthimages.net*
* * *.phonefactor.net*
* *enterpriseregistration.windows.net*
* *management.azure.com*
* *policykeyservice.dc.ad.msft.net*
* *secure.aadcdn.microsoftonline-p.com*

Pour obtenir la liste complète des points de terminaison nécessaires à l’utilisation des produits en ligne de Microsoft, consultez [URL et plages d’adresses IP Office 365](/microsoft-365/enterprise/urls-and-ip-address-ranges).

### <a name="how-do-i-identify-the-domain-join-type-azure-ad-joined-or-hybrid-azure-ad-joined-for-my-windows-10-device"></a>Comment identifier le type de jonction de domaine (jointure Azure AD ou jointure hybride Azure AD) pour mon appareil Windows 10 ?

Pour vérifier si l’appareil client Windows 10 possède le type de jonction de domaine approprié, utilisez la commande suivante :

```console
Dsregcmd/status
```

L’exemple de sortie suivant montre que l’appareil a une jointure Azure AD, car *AzureADJoined* est défini sur *YES* :

```output
+---------------------+
| Device State        |
+---------------------+

AzureADJoined: YES
EnterpriseJoined: NO
DomainedJoined: NO
```

L’exemple de sortie suivant montre que l’appareil a une jointure hybride Azure AD, car *DomainedJoined* est défini sur *YES*. La valeur *DomainName* est également illustrée ci-dessous :

```output
+---------------------+
| Device State        |
+---------------------+

AzureADJoined: YES
EnterpriseJoined: NO
DomainedJoined: YES
DomainName: CONTOSO
```

Sur un contrôleur de domaine Windows Server 2016 ou 2019, vérifiez que les patchs suivants sont appliqués. Si nécessaire, exécutez Windows Update pour les installer :

* Windows Server 2016 :[KB4534307](https://support.microsoft.com/help/4534307/windows-10-update-kb4534307)
* Windows Server 2019 : [KB4534321](https://support.microsoft.com/help/4534321/windows-10-update-kb4534321)

À partir d’un appareil client, exécutez la commande suivante pour vérifier la connectivité à un contrôleur de domaine approprié avec les patchs installés :

```console
nltest /dsgetdc:<domain> /keylist /kdc
```

### <a name="whats-the-recommendation-on-the-number-of-dcs-that-should-be-patched"></a>Quelle est la recommandation en matière de nombre de contrôleurs de domaine qui doivent être corrigés ?

Nous vous recommandons d’appliquer le patch à la majorité de vos contrôleurs de domaine Windows Server 2016 ou 2019 afin qu’ils puissent traiter la charge de requêtes d’authentification de votre organisation.

Sur un contrôleur de domaine Windows Server 2016 ou 2019, vérifiez que les patchs suivants sont appliqués. Si nécessaire, exécutez Windows Update pour les installer :

* Windows Server 2016 :[KB4534307](https://support.microsoft.com/help/4534307/windows-10-update-kb4534307)
* Windows Server 2019 : [KB4534321](https://support.microsoft.com/help/4534321/windows-10-update-kb4534321)

### <a name="can-i-deploy-the-fido2-credential-provider-on-an-on-premises-only-device"></a>Puis-je déployer le fournisseur d’informations d’identification FIDO2 sur un appareil uniquement local ?

Non, cette fonctionnalité n’est pas prise en charge pour un appareil uniquement local. Le fournisseur d’informations d’identification FIDO2 n’apparaît pas.

### <a name="fido2-security-key-sign-in-isnt-working-for-my-domain-admin-or-other-high-privilege-accounts-why"></a>La connexion par clé de sécurité FIDO2 ne fonctionne pas pour mon compte Administrateur de domaine ou d’autres comptes à privilèges élevés. Pourquoi ?

La stratégie de sécurité par défaut n’accorde pas d’autorisation Azure AD pour signer des comptes à privilèges élevés sur des ressources locales.

Pour débloquer les comptes, utilisez **Utilisateurs et ordinateurs Active Directory** pour modifier la propriété *msDS-NeverRevealGroup* d’*Azure AD Kerberos Computer object (CN=AzureADKerberos,OU=Domain Controllers,\<domain-DN>)* .

## <a name="under-the-hood"></a>Sous le capot

* [Comment Azure AD Kerberos est-il lié à mon environnement Active Directory Domain Services local ?](#how-is-azure-ad-kerberos-linked-to-my-on-premises-active-directory-domain-services-environment)
* [Où puis-je voir ces objets serveur Kerberos qui sont créés dans AD et publiés dans Azure AD ?](#where-can-i-view-these-kerberos-server-objects-that-are-created-in-ad-ds-and-published-in-azure-ad)
* [Pourquoi la clé publique n’est-elle pas inscrite sur un AD DS local afin de ne pas dépendre d’Internet ?](#why-cant-we-have-the-public-key-registered-to-on-premises-ad-ds-so-there-is-no-dependency-on-the-internet)
* [Comment les clés sont-elles alternées sur l’objet serveur Kerberos ?](#how-are-the-keys-rotated-on-the-kerberos-server-object)
* [Pourquoi avons-nous besoin d’Azure AD Connect ? Réécrit-t-il des informations sur AD DS à partir d’Azure AD ?](#why-do-we-need-azure-ad-connect-does-it-write-any-info-back-to-ad-ds-from-azure-ad)
* [À quoi ressemble la requête/réponse HTTP lors d’une requête PRT + TGT partielle ?](#what-does-the-http-requestresponse-look-like-when-requesting-prt-partial-tgt)

### <a name="how-is-azure-ad-kerberos-linked-to-my-on-premises-active-directory-domain-services-environment"></a>Comment Azure AD Kerberos est-il lié à mon environnement Active Directory Domain Services local ?

Il existe deux parties : l’environnement AD DS local et le locataire Azure AD.

**Active Directory Domain Services (AD DS)**

Le serveur Azure AD Kerberos est représenté dans un environnement AD DS local sous la forme d’un objet contrôleur de domaine (DC). Cet objet DC est constitué de plusieurs objets :

* *CN=AzureADKerberos,OU=Domain Controllers,\<domain-DN>*
    
    Objet *Computer* qui représente un contrôleur de domaine en lecture seule (RODC) dans AD DS. Aucun ordinateur n’est associé à cet objet. Au lieu de cela, il s’agit d’une représentation logique d’un DC.

* *CN=krbtgt_AzureAD,CN=Users,\<domain-DN>*

    Objet *User* qui représente une clé de chiffrement TGT (Ticket Granting Ticket) Kerberos RODC.

* *CN=900274c4-b7d2-43c8-90ee-00a9f650e335,CN=AzureAD,CN=System,\<domain-DN>*

    Objet *ServiceConnectionPoint* qui stocke les métadonnées relatives aux objets de serveur Azure AD Kerberos. Les outils d’administration utilisent cet objet pour identifier et localiser les objets de serveur Azure AD Kerberos.

**Azure Active Directory**

L’objet serveur Azure AD Kerberos est représenté dans Azure AD en tant qu’objet *KerberosDomain*. Chaque environnement AD DS local est représenté par un objet *KerberosDomain* unique dans le locataire Azure AD.

Par exemple, vous pouvez avoir une forêt AD DS avec deux domaines tels que *contoso.com* et *fabrikam.com*. Si vous autorisez Azure AD à émettre des TGT Kerberos pour l’ensemble de la forêt, il y aura deux objets *KerberosDomain* dans Azure AD, un objet pour *contoso.com* et l’autre pour *fabrikam.com*.

Si vous avez plusieurs forêts AD DS, vous aurez un objet *KerberosDomain* pour chaque domaine dans chaque forêt.

### <a name="where-can-i-view-these-kerberos-server-objects-that-are-created-in-ad-ds-and-published-in-azure-ad"></a>Où puis-je voir ces objets serveur Kerberos qui sont créés dans AD DS et publiés dans Azure AD ?

Pour afficher tous les objets, utilisez les cmdlets PowerShell du serveur Azure AD Kerberos incluses dans la dernière version d’Azure AD Connect.

Pour plus d’informations, notamment des instructions sur la façon d’afficher les objets, consultez [Créer un objet serveur Kerberos](howto-authentication-passwordless-security-key-on-premises.md#create-kerberos-server-object).

### <a name="why-cant-we-have-the-public-key-registered-to-on-premises-ad-ds-so-there-is-no-dependency-on-the-internet"></a>Pourquoi la clé publique n’est-elle pas inscrite sur un AD DS local afin de ne pas dépendre d’Internet ?

Nous avons reçu des commentaires sur la complexité du modèle de déploiement pour Windows Hello Entreprise. Nous voulions donc simplifier le modèle de déploiement sans avoir à utiliser de certificats ni de PKI (FIDO2 n’utilise pas de certificats).

### <a name="how-are-the-keys-rotated-on-the-kerberos-server-object"></a>Comment les clés sont-elles alternées sur l’objet serveur Kerberos ?

Comme pour tout autre contrôleur de domaine, les clés *krbtgt* de chiffrement du serveur Azure AD Kerberos doivent être régulièrement alternées. Il est recommandé de suivre la même planification que celle utilisée pour alterner toutes les autres clés *krbtgt* d’AD DS.

> [!NOTE]
> Bien qu’il existe d’autres outils pour alterner les clés *krbtgt*, vous devez [utiliser les cmdlets PowerShell pour alterner les clés *krbtgt*](howto-authentication-passwordless-security-key-on-premises.md#rotating-the-azure-ad-kerberos-server-key) de votre serveur Azure AD Kerberos. Cette méthode permet de s’assurer que les clés sont mises à jour à la fois dans l’environnement AD DS local et dans Azure AD.

### <a name="why-do-we-need-azure-ad-connect-does-it-write-any-info-back-to-ad-ds-from-azure-ad"></a>Pourquoi avons-nous besoin d’Azure AD Connect ? Réécrit-t-il des informations sur AD DS à partir d’Azure AD ?

Azure AD Connect n’écrit pas d’informations d’Azure AD sur AD DS. L’utilitaire comprend le module PowerShell pour créer l’objet serveur Kerberos dans AD DS et le publier dans Azure AD.

### <a name="what-does-the-http-requestresponse-look-like-when-requesting-prt-partial-tgt"></a>À quoi ressemble la requête/réponse HTTP lors d’une requête PRT + TGT partielle ?

La requête HTTP est une demande de jeton d’actualisation principal (PRT, Primary Refresh Token) standard. Cette demande de PRT comprend une revendication indiquant qu’un ticket TGT (Ticket Granting Ticket) Kerberos est nécessaire.

| Revendication | Valeur | Description                             |
|-------|-------|-----------------------------------------|
| tgt   | true  | La revendication indique que le client a besoin d’un TGT. |

Azure AD combine la clé de client chiffrée et la mémoire tampon de message dans la réponse PRT en tant que propriétés supplémentaires. La charge utile est chiffrée à l’aide de la clé de session de l’appareil Azure AD.

| Champ              | Type   | Description  |
|--------------------|--------|--------------|
| tgt_client_key     | string | Clé client encodée en base64 (secret). Il s’agit de la clé secrète client utilisée pour protéger le TGT. Dans ce scénario sans mot de passe, la clé secrète client est générée par le serveur dans le cadre de chaque requête TGT, puis retournée au client dans la réponse. |
| tgt_key_type       | int    | Type de clé AD DS local utilisé pour la clé client et la clé de session Kerberos inclus dans le KERB_MESSAGE_BUFFER. |
| tgt_message_buffer | string | KERB_MESSAGE_BUFFER encodé en base64. |

## <a name="next-steps"></a>Étapes suivantes

Pour vous familiariser avec les clés de sécurité FIDO2 et l’accès hybride aux ressources locales, consultez les articles suivants :

* [Clés de sécurité FIDO2 sans mot de passe](howto-authentication-passwordless-security-key.md)
* [Windows 10 sans mot de passe](howto-authentication-passwordless-security-key-windows.md)
* [En local sans mot de passe](howto-authentication-passwordless-security-key-on-premises.md)
