---
title: Attributs synchronisés par Azure AD Connect | Microsoft Docs
description: Répertorie les attributs qui sont synchronisés avec Azure Active Directory.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: c2bb36e0-5205-454c-b9b6-f4990bcedf51
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 04/15/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6ec05c4160c6502904644bf7035bda0bed66cc33
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94413188"
---
# <a name="azure-ad-connect-sync-attributes-synchronized-to-azure-active-directory"></a>Synchronisation d’Azure AD Connect : Attributs synchronisés avec Azure Active Directory
Cette rubrique répertorie les attributs synchronisés par Azure AD Connect Sync.  
Les attributs sont regroupés selon l’application Azure AD associée.

## <a name="attributes-to-synchronize"></a>Attributs à synchroniser
Une question fréquente concerne *la liste des attributs dont la synchronisation est obligatoire*. L’approche recommandée consiste à conserver les attributs par défaut pour qu’une liste d’adresses globale complète puisse être construite dans le cloud et afin de disposer de toutes les fonctionnalités dans les charges de travail Microsoft 365. Dans certains cas, votre organisation ne souhaitera pas synchroniser certains attributs sur le cloud, car ces attributs contiennent des données sensibles ou personnelles, comme dans cet exemple :  
![mauvais attributs](./media/reference-connect-sync-attributes-synchronized/badextensionattribute.png)

Dans ce cas, commencez par la liste des attributs dans cette rubrique et identifiez ceux qui contiennent des informations sensibles ou personnelles et ne peuvent pas être synchronisés. Ensuite, désélectionnez ces attributs lors de l’installation à l’aide de [l’application Azure AD et du filtrage des attributs](how-to-connect-install-custom.md#azure-ad-app-and-attribute-filtering).

> [!WARNING]
> Lorsque vous désélectionnez des attributs, prenez garde à ne désélectionner que ceux qu’il est absolument impossible de synchroniser. Désélectionner d’autres attributs peut avoir un impact négatif sur les fonctionnalités.
>
>

## <a name="microsoft-365-apps-for-enterprise"></a>Microsoft 365 Apps for enterprise
| Nom de l'attribut | Utilisateur | Commentaire |
| --- |:---:| --- |
| accountEnabled |X |Détermine si un compte est activé. |
| cn |X | |
| displayName |X | |
| objectSID |X |propriété mécanique. Identificateur d’utilisateur Active Directory utilisé pour maintenir la synchronisation entre Azure AD et Active Directory. |
| pwdLastSet |X |propriété mécanique. Permet de savoir quand invalider les jetons déjà émis. S’applique à la synchronisation du hachage de mot de passe, l’authentification directe et la fédération. |
|samAccountName|X| |
| sourceAnchor |X |propriété mécanique. Identificateur immuable pour maintenir la relation entre ADDS et Azure AD. |
| usageLocation |X |propriété mécanique. Pays/Région de l’utilisateur. Utilisé pour l’attribution de licence. |
| userPrincipalName |X |Le nom UPN est l’ID de connexion de l’utilisateur. Le plus souvent, identique à la valeur [mail]. |

## <a name="exchange-online"></a>Exchange Online
| Nom de l'attribut | Utilisateur | Contact | Groupe | Commentaire |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |Détermine si un compte est activé. |
| assistant |X |X | | |
| altRecipient |X | | |Nécessite Azure AD Connect 1.1.552.0 ou version ultérieure. |
| authOrig |X |X |X | |
| c |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| société |X |X | | |
| countryCode |X |X | | |
| department |X |X | | |
| description | | |X | |
| displayName |X |X |X | |
| dLMemRejectPerms |X |X |X | |
| dLMemSubmitPerms |X |X |X | |
| extensionAttribute1 |X |X |X | |
| extensionAttribute10 |X |X |X | |
| extensionAttribute11 |X |X |X | |
| extensionAttribute12 |X |X |X | |
| extensionAttribute13 |X |X |X | |
| extensionAttribute14 |X |X |X | |
| extensionAttribute15 |X |X |X | |
| extensionAttribute2 |X |X |X | |
| extensionAttribute3 |X |X |X | |
| extensionAttribute4 |X |X |X | |
| extensionAttribute5 |X |X |X | |
| extensionAttribute6 |X |X |X | |
| extensionAttribute7 |X |X |X | |
| extensionAttribute8 |X |X |X | |
| extensionAttribute9 |X |X |X | |
| facsimiletelephonenumber |X |X | | |
| givenName |X |X | | |
| homePhone |X |X | | |
| info |X |X |X |Cet attribut n'est actuellement pas utilisé pour les groupes. |
| Initials |X |X | | |
| l |X |X | | |
| legacyExchangeDN |X |X |X | |
| mailNickName |X |X |X | |
| managedBy | | |X | |
| manager |X |X | | |
| member | | |X | |
| mobile |X |X | | |
| msDS-HABSeniorityIndex |X |X |X | |
| msDS-PhoneticDisplayName |X |X |X | |
| msExchArchiveGUID |X | | | |
| msExchArchiveName |X | | | |
| msExchAssistantName |X |X | | |
| msExchAuditAdmin |X | | | |
| msExchAuditDelegate |X | | | |
| msExchAuditDelegateAdmin |X | | | |
| msExchAuditOwner |X | | | |
| msExchBlockedSendersHash |X |X | | |
| msExchBypassAudit |X | | | |
| msExchBypassModerationLink | | |X |Disponible dans Azure AD Connect version 1.1.524.0 |
| msExchCoManagedByLink | | |X | |
| msExchDelegateListLink |X | | | |
| msExchELCExpirySuspensionEnd |X | | | |
| msExchELCExpirySuspensionStart |X | | | |
| msExchELCMailboxFlags |X | | | |
| msExchEnableModeration |X | |X | |
| msExchExtensionCustomAttribute1 |X |X |X |Cet attribut n'est actuellement pas utilisé par Exchange Online. |
| msExchExtensionCustomAttribute2 |X |X |X |Cet attribut n'est actuellement pas utilisé par Exchange Online. |
| msExchExtensionCustomAttribute3 |X |X |X |Cet attribut n'est actuellement pas utilisé par Exchange Online. |
| msExchExtensionCustomAttribute4 |X |X |X |Cet attribut n'est actuellement pas utilisé par Exchange Online. |
| msExchExtensionCustomAttribute5 |X |X |X |Cet attribut n'est actuellement pas utilisé par Exchange Online. |
| msExchHideFromAddressLists |X |X |X | |
| msExchImmutableID |X | | | |
| msExchLitigationHoldDate |X |X |X | |
| msExchLitigationHoldOwner |X |X |X | |
| msExchMailboxAuditEnable |X | | | |
| msExchMailboxAuditLogAgeLimit |X | | | |
| msExchMailboxGuid |X | | | |
| msExchModeratedByLink |X |X |X | |
| msExchModerationFlags |X |X |X | |
| msExchRecipientDisplayType |X |X |X | |
| msExchRecipientTypeDetails |X |X |X | |
| msExchRemoteRecipientType |X | | | |
| msExchRequireAuthToSendTo |X |X |X | |
| msExchResourceCapacity |X | | | |
| msExchResourceDisplay |X | | | |
| msExchResourceMetaData |X | | | |
| msExchResourceSearchProperties |X | | | |
| msExchRetentionComment |X |X |X | |
| msExchRetentionURL |X |X |X | |
| msExchSafeRecipientsHash |X |X | | |
| msExchSafeSenderHash |X |X | | |
| msExchSenderHintTranslations |X |X |X | |
| msExchTeamMailboxExpiration |X | | | |
| msExchTeamMailboxOwners |X | | | |
| msExchTeamMailboxSharePointUrl |X | | | |
| msExchUserHoldPolicies |X | | | |
| msOrg-IsOrganizational | | |X | |
| objectSID |X | |X |propriété mécanique. Identificateur d’utilisateur Active Directory utilisé pour maintenir la synchronisation entre Azure AD et Active Directory. |
| oOFReplyToOriginator | | |X | |
| otherFacsimileTelephone |X |X | | |
| otherHomePhone |X |X | | |
| otherTelephone |X |X | | |
| pager |X |X | | |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| proxyAddresses |X |X |X | |
| publicDelegates |X |X |X | |
| pwdLastSet |X | | |propriété mécanique. Permet de savoir quand invalider les jetons déjà émis. Utilisé par la synchronisation de mot de passe et par la fédération. |
| reportToOriginator | | |X | |
| reportToOwner | | |X | |
| sn |X |X | | |
| sourceAnchor |X |X |X |propriété mécanique. Identificateur immuable pour maintenir la relation entre ADDS et Azure AD. |
| st |X |X | | |
| streetAddress |X |X | | |
| targetAddress |X |X | | |
| telephoneAssistant |X |X | | |
| telephoneNumber |X |X | | |
| thumbnailPhoto |X |X | |synchronisé une seule fois d’Azure AD vers Exchange Online, après quoi Exchange Online devient la source d’autorité pour cet attribut, et les modifications ultérieures ne peuvent pas être synchronisées à partir d’un site local. Pour plus d’informations, consultez la ([base de connaissances](https://support.microsoft.com/help/3062745/user-photos-aren-t-synced-from-the-on-premises-environment-to-exchange)).|
| title |X |X | | |
| unauthOrig |X |X |X | |
| usageLocation |X | | |propriété mécanique. Pays/Région de l’utilisateur. Utilisé pour l’attribution de licence. |
| userCertificate |X |X | | |
| userPrincipalName |X | | |Le nom UPN est l’ID de connexion de l’utilisateur. Le plus souvent, identique à la valeur [mail]. |
| userSMIMECertificates |X |X | | |
| wWWHomePage |X |X | | |

## <a name="sharepoint-online"></a>SharePoint Online
| Nom de l'attribut | Utilisateur | Contact | Groupe | Commentaire |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |Détermine si un compte est activé. |
| authOrig |X |X |X | |
| c |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| société |X |X | | |
| countryCode |X |X | | |
| department |X |X | | |
| description |X |X |X | |
| displayName |X |X |X | |
| dLMemRejectPerms |X |X |X | |
| dLMemSubmitPerms |X |X |X | |
| extensionAttribute1 |X |X |X | |
| extensionAttribute10 |X |X |X | |
| extensionAttribute11 |X |X |X | |
| extensionAttribute12 |X |X |X | |
| extensionAttribute13 |X |X |X | |
| extensionAttribute14 |X |X |X | |
| extensionAttribute15 |X |X |X | |
| extensionAttribute2 |X |X |X | |
| extensionAttribute3 |X |X |X | |
| extensionAttribute4 |X |X |X | |
| extensionAttribute5 |X |X |X | |
| extensionAttribute6 |X |X |X | |
| extensionAttribute7 |X |X |X | |
| extensionAttribute8 |X |X |X | |
| extensionAttribute9 |X |X |X | |
| facsimiletelephonenumber |X |X | | |
| givenName |X |X | | |
| hideDLMembership | | |X | |
| homePhone |X |X | | |
| info |X |X |X | |
| Initials |X |X | | |
| ipPhone |X |X | | |
| l |X |X | | |
| mail |X |X |X | |
| mailNickName |X |X |X | |
| managedBy | | |X | |
| manager |X |X | | |
| member | | |X | |
| middleName |X |X | | |
| mobile |X |X | | |
| msExchTeamMailboxExpiration |X | | | |
| msExchTeamMailboxOwners |X | | | |
| msExchTeamMailboxSharePointLinkedBy |X | | | |
| msExchTeamMailboxSharePointUrl |X | | | |
| objectSID |X | |X |propriété mécanique. Identificateur d’utilisateur Active Directory utilisé pour maintenir la synchronisation entre Azure AD et Active Directory. |
| oOFReplyToOriginator | | |X | |
| otherFacsimileTelephone |X |X | | |
| otherHomePhone |X |X | | |
| otherIpPhone |X |X | | |
| otherMobile |X |X | | |
| otherPager |X |X | | |
| otherTelephone |X |X | | |
| pager |X |X | | |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| postOfficeBox |X |X | |Cet attribut n’est actuellement pas utilisé par SharePoint Online. |
| preferredLanguage |X | | | |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |propriété mécanique. Permet de savoir quand invalider les jetons déjà émis. S’applique à la synchronisation du hachage de mot de passe, l’authentification directe et la fédération. |
| reportToOriginator | | |X | |
| reportToOwner | | |X | |
| sn |X |X | | |
| sourceAnchor |X |X |X |propriété mécanique. Identificateur immuable pour maintenir la relation entre ADDS et Azure AD. |
| st |X |X | | |
| streetAddress |X |X | | |
| targetAddress |X |X | | |
| telephoneAssistant |X |X | | |
| telephoneNumber |X |X | | |
| thumbnailPhoto |X |X | |synchronisé une seule fois d’Azure AD vers Exchange Online, après quoi Exchange Online devient la source d’autorité pour cet attribut, et les modifications ultérieures ne peuvent pas être synchronisées à partir d’un site local. Pour plus d’informations, consultez la ([base de connaissances](https://support.microsoft.com/help/3062745/user-photos-aren-t-synced-from-the-on-premises-environment-to-exchange)).|
| title |X |X | | |
| unauthOrig |X |X |X | |
| url |X |X | | |
| usageLocation |X | | |propriété mécanique. Pays/Région de l’utilisateur
. Utilisé pour l’attribution de licence. |
| userPrincipalName |X | | |Le nom UPN est l’ID de connexion de l’utilisateur. Le plus souvent, identique à la valeur [mail]. |
| wWWHomePage |X |X | | |

## <a name="teams-and-skype-for-business-online"></a>Teams et Skype for Business Online
| Nom de l'attribut | Utilisateur | Contact | Groupe | Commentaire |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |Détermine si un compte est activé. |
| c |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| société |X |X | | |
| department |X |X | | |
| description |X |X |X | |
| displayName |X |X |X | |
| facsimiletelephonenumber |X |X |X | |
| givenName |X |X | | |
| homePhone |X |X | | |
| ipPhone |X |X | | |
| l |X |X | | |
| mail |X |X |X | |
| mailNickName |X |X |X | |
| managedBy | | |X | |
| manager |X |X | | |
| member | | |X | |
| mobile |X |X | | |
| msExchHideFromAddressLists |X |X |X | |
| msRTCSIP-ApplicationOptions |X | | | |
| msRTCSIP-DeploymentLocator |X |X | | |
| msRTCSIP-Line |X |X | | |
| msRTCSIP-OptionFlags |X |X | | |
| msRTCSIP-OwnerUrn |X | | | |
| msRTCSIP-PrimaryUserAddress |X |X | | |
| msRTCSIP-UserEnabled |X |X | | |
| objectSID |X | |X |propriété mécanique. Identificateur d’utilisateur Active Directory utilisé pour maintenir la synchronisation entre Azure AD et Active Directory. |
| otherTelephone |X |X | | |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| preferredLanguage |X | | | |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |propriété mécanique. Permet de savoir quand invalider les jetons déjà émis. S’applique à la synchronisation du hachage de mot de passe, l’authentification directe et la fédération. |
| sn |X |X | | |
| sourceAnchor |X |X |X |propriété mécanique. Identificateur immuable pour maintenir la relation entre ADDS et Azure AD. |
| st |X |X | | |
| streetAddress |X |X | | |
| telephoneNumber |X |X | | |
| thumbnailPhoto |X |X | |synchronisé une seule fois d’Azure AD vers Exchange Online, après quoi Exchange Online devient la source d’autorité pour cet attribut, et les modifications ultérieures ne peuvent pas être synchronisées à partir d’un site local. Pour plus d’informations, consultez la ([base de connaissances](https://support.microsoft.com/help/3062745/user-photos-aren-t-synced-from-the-on-premises-environment-to-exchange)).|
| title |X |X | | |
| usageLocation |X | | |propriété mécanique. Pays/Région de l’utilisateur. Utilisé pour l’attribution de licence. |
| userPrincipalName |X | | |Le nom UPN est l’ID de connexion de l’utilisateur. Le plus souvent, identique à la valeur [mail]. |
| wWWHomePage |X |X | | |

## <a name="azure-rms"></a>Azure RMS
| Nom de l'attribut | Utilisateur | Contact | Groupe | Commentaire |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |Détermine si un compte est activé. |
| cn |X | |X |Nom commun ou alias. Le plus souvent, il s’agit du préfixe de valeur [mail]. |
| displayName |X |X |X |Chaîne qui représente le nom affiché souvent comme nom convivial (prénom nom). |
| mail |X |X |X |Adresse de messagerie complète. |
| member | | |X | |
| objectSID |X | |X |propriété mécanique. Identificateur d’utilisateur Active Directory utilisé pour maintenir la synchronisation entre Azure AD et Active Directory. |
| proxyAddresses |X |X |X |propriété mécanique. Utilisé par Azure AD. Contient toutes les adresses de messagerie secondaires pour l’utilisateur. |
| pwdLastSet |X | | |propriété mécanique. Permet de savoir quand invalider les jetons déjà émis. |
| sourceAnchor |X |X |X |propriété mécanique. Identificateur immuable pour maintenir la relation entre ADDS et Azure AD. |
| usageLocation |X | | |propriété mécanique. Pays/Région de l’utilisateur. Utilisé pour l’attribution de licence. |
| userPrincipalName |X | | |Ce nom UPN est l’ID de connexion de l’utilisateur. Le plus souvent, identique à la valeur [mail]. |

## <a name="intune"></a>Intune
| Nom de l'attribut | Utilisateur | Contact | Groupe | Commentaire |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |Détermine si un compte est activé. |
| c |X |X | | |
| cn |X | |X | |
| description |X |X |X | |
| displayName |X |X |X | |
| mail |X |X |X | |
| mailNickName |X |X |X | |
| member | | |X | |
| objectSID |X | |X |propriété mécanique. Identificateur d’utilisateur Active Directory utilisé pour maintenir la synchronisation entre Azure AD et Active Directory. |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |propriété mécanique. Permet de savoir quand invalider les jetons déjà émis. S’applique à la synchronisation du hachage de mot de passe, l’authentification directe et la fédération. |
| sourceAnchor |X |X |X |propriété mécanique. Identificateur immuable pour maintenir la relation entre ADDS et Azure AD. |
| usageLocation |X | | |propriété mécanique. Pays/Région de l’utilisateur. Utilisé pour l’attribution de licence. |
| userPrincipalName |X | | |Le nom UPN est l’ID de connexion de l’utilisateur. Le plus souvent, identique à la valeur [mail]. |

## <a name="dynamics-crm"></a>Dynamics CRM
| Nom de l'attribut | Utilisateur | Contact | Groupe | Commentaire |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |Détermine si un compte est activé. |
| c |X |X | | |
| cn |X | |X | |
| co |X |X | | |
| société |X |X | | |
| countryCode |X |X | | |
| description |X |X |X | |
| displayName |X |X |X | |
| facsimiletelephonenumber |X |X | | |
| givenName |X |X | | |
| l |X |X | | |
| managedBy | | |X | |
| manager |X |X | | |
| member | | |X | |
| mobile |X |X | | |
| objectSID |X | |X |propriété mécanique. Identificateur d’utilisateur Active Directory utilisé pour maintenir la synchronisation entre Azure AD et Active Directory. |
| physicalDeliveryOfficeName |X |X | | |
| postalCode |X |X | | |
| preferredLanguage |X | | | |
| pwdLastSet |X | | |propriété mécanique. Permet de savoir quand invalider les jetons déjà émis. S’applique à la synchronisation du hachage de mot de passe, l’authentification directe et la fédération. |
| sn |X |X | | |
| sourceAnchor |X |X |X |propriété mécanique. Identificateur immuable pour maintenir la relation entre ADDS et Azure AD. |
| st |X |X | | |
| streetAddress |X |X | | |
| telephoneNumber |X |X | | |
| title |X |X | | |
| usageLocation |X | | |propriété mécanique. Pays/Région de l’utilisateur. Utilisé pour l’attribution de licence. |
| userPrincipalName |X | | |Le nom UPN est l’ID de connexion de l’utilisateur. Le plus souvent, identique à la valeur [mail]. |

## <a name="3rd-party-applications"></a>Applications tierces
Ce groupe est un ensemble d'attributs utilisés comme attributs nécessaires au minimum pour une application ou une charge de travail générique. Il peut être utilisé pour des charges de travail non répertoriées dans une section ou pour une application hors applications Microsoft. Il est utilisé explicitement pour les applications suivantes :

* Yammer (seul l’utilisateur est consommé)
* [Scénarios de collaboration transorganisationnelle B2B (Business-to-Business) hybride proposés par des ressources comme SharePoint](/sharepoint/create-b2b-extranet)

Ce groupe est un ensemble d’attributs qui peut être utilisé si l’annuaire Azure AD n’est pas utilisé pour prendre en charge Microsoft 365, Dynamics ou Intune. Il comporte un petit ensemble d’attributs de base. Notez que l’authentification unique ou le provisionnement de certaines applications tierces requièrent la configuration de la synchronisation des attributs en plus des attributs décrits ici. Les spécifications de l’application sont décrites dans le [tutoriel sur l’application SaaS](../saas-apps/tutorial-list.md) pour chaque application.

| Nom de l'attribut | Utilisateur | Contact | Groupe | Commentaire |
| --- |:---:|:---:|:---:| --- |
| accountEnabled |X | | |Détermine si un compte est activé. |
| cn |X | |X | |
| displayName |X |X |X | |
| employeeID |X |  |  | |
| givenName |X |X | | |
| mail |X | |X | |
| managedBy | | |X | |
| mailNickName |X |X |X | |
| member | | |X | |
| objectSID |X | | |propriété mécanique. Identificateur d’utilisateur Active Directory utilisé pour maintenir la synchronisation entre Azure AD et Active Directory. |
| proxyAddresses |X |X |X | |
| pwdLastSet |X | | |propriété mécanique. Permet de savoir quand invalider les jetons déjà émis. S’applique à la synchronisation du hachage de mot de passe, l’authentification directe et la fédération. |
| sn |X |X | | |
| sourceAnchor |X |X |X |propriété mécanique. Identificateur immuable pour maintenir la relation entre ADDS et Azure AD. |
| usageLocation |X | | |propriété mécanique. Pays/Région de l’utilisateur. Utilisé pour l’attribution de licence. |
| userPrincipalName |X | | |Le nom UPN est l’ID de connexion de l’utilisateur. Le plus souvent, identique à la valeur [mail]. |

## <a name="windows-10"></a>Windows 10
Un ordinateur (appareil) Windows 10 joint à un domaine synchronisera certains attributs sur Azure AD. Pour plus d'informations sur les scénarios, consultez [Connecter des appareils joints au domaine à Azure AD pour des expériences Windows 10](../devices/hybrid-azuread-join-plan.md). Ces attributs sont toujours synchronisés et Windows 10 n'apparaît pas comme une application que vous pouvez désélectionner. Un ordinateur appartenant au domaine Windows 10 est identifié par l'attribut userCertificate.

| Nom de l'attribut | Appareil | Commentaire |
| --- |:---:| --- |
| accountEnabled |X | |
| deviceTrustType |X |Valeur codée en dur pour les ordinateurs appartenant au domaine. |
| displayName |X | |
| ms-DS-CreatorSID |X |Également appelé registeredOwnerReference. |
| objectGUID |X |Également appelé deviceID. |
| objectSID |X |Également appelé onPremisesSecurityIdentifier. |
| operatingSystem |X |Également appelé deviceOSType. |
| operatingSystemVersion |X |Également appelé deviceOSVersion. |
| userCertificate |X | |

Ces attributs pour **l'utilisateur** s'ajoutent aux autres applications que vous avez sélectionnées.  

| Nom de l'attribut | Utilisateur | Commentaire |
| --- |:---:| --- |
| domainFQDN |X |Également appelé dnsDomainName. Par exemple, contoso.com. |
| domainNetBios |X |Également appelé netBiosName. Par exemple, CONTOSO. |
| msDS-KeyCredentialLink |X |Une fois l’utilisateur inscrit dans Windows Hello Entreprise. | 

## <a name="exchange-hybrid-writeback"></a>Écriture différée d’Exchange hybride
Ces attributs sont écrits en différé depuis Azure AD vers Active Directory local quand vous choisissez d’activer **Exchange hybride**. Selon votre version d’Exchange, il est possible que moins d’attributs soient synchronisés.

| Nom d’attribut (instance locale d’AD) | Nom d’attribut (interface utilisateur Connect) | Utilisateur | Contact | Groupe | Commentaire |
| --- |:---:|:---:|:---:| --- |---|
| msDS-ExternalDirectoryObjectID| ms-DS-External-Directory-Object-Id |X | | |Dérivé de cloudAnchor dans Azure AD. Cet attribut est une nouveauté dans Exchange 2016 et Windows Server 2016 AD. |
| msExchArchiveStatus| ms-Exch-ArchiveStatus |X | | |Archive en ligne : Permet aux clients d’archiver le courrier. |
| msExchBlockedSendersHash| ms-Exch-BlockedSendersHash |X | | |Filtrage : Réécrit le filtrage local, les données sécurisées en ligne et les données des expéditeurs bloqués provenant des clients. |
| msExchSafeRecipientsHash| ms-Exch-SafeRecipientsHash  |X | | |Filtrage : Réécrit le filtrage local, les données sécurisées en ligne et les données des expéditeurs bloqués provenant des clients. |
| msExchSafeSenderHash| ms-Exch-SafeSendersHash  |X | | |Filtrage : Réécrit le filtrage local, les données sécurisées en ligne et les données des expéditeurs bloqués provenant des clients. |
| msExchUCVoiceMailSettings| ms-Exch-UCVoiceMailSettings |X | | |Activer la messagerie unifiée (MU) - Messagerie vocale en ligne : Utilisée par l’intégration de Microsoft Lync Server pour indiquer à Lync Server local que l’utilisateur dispose de la messagerie vocale dans les services en ligne. |
| msExchUserHoldPolicies| ms-Exch-UserHoldPolicies |X | | |Conservation pour litige : Permet aux services cloud de déterminer quels utilisateurs sont sous conservation pour litige. |
| proxyAddresses| proxyAddresses |X |X |X |Seule l’adresse x500 d’Exchange Online est insérée. |
| publicDelegates| ms-Exch-Public-Delegates  |X | | |Permet à une boîte aux lettres Exchange Online d’obtenir des droits SendOnBehalfTo sur les boîtes aux lettres Exchange sur site des utilisateurs. Nécessite Azure AD Connect 1.1.552.0 ou version ultérieure. |

## <a name="exchange-mail-public-folder"></a>Dossier public de messagerie Exchange
Ces attributs sont synchronisés de l’annuaire Active Directory local vers Azure AD quand vous choisissez d’activer le **dossier public d’Exchange Mail**.

| Nom de l'attribut | Dossier public | Commentaire |
| --- | :---:| --- |
| displayName | X |  |
| mail | X |  |
| msExchRecipientTypeDetails | X |  |
| objectGUID | X |  |
| proxyAddresses | X |  |
| targetAddress | X |  |

## <a name="device-writeback"></a>Écriture différée des appareils
Les objets d’appareil sont créés dans Active Directory. Ces objets peuvent être des appareils joints à Azure AD ou d’ordinateurs Windows 10 appartenant au domaine.

| Nom de l'attribut | Appareil | Commentaire |
| --- |:---:| --- |
| altSecurityIdentities |X | |
| displayName |X | |
| dn |X | |
| msDS-CloudAnchor |X | |
| msDS-DeviceID |X | |
| msDS-DeviceObjectVersion |X | |
| msDS-DeviceOSType |X | |
| msDS-DeviceOSVersion |X | |
| msDS-DevicePhysicalIDs |X | |
| msDS-KeyCredentialLink |X |Uniquement avec le schéma AD de Windows Server 2016 |
| msDS-IsCompliant |X | |
| msDS-IsEnabled |X | |
| msDS-IsManaged |X | |
| msDS-RegisteredOwner |X | |

## <a name="notes"></a>Notes
* Quand vous utilisez un ID secondaire, l’attribut local userPrincipalName est synchronisé avec l’attribut Azure AD onPremisesUserPrincipalName. L’attribut de l’ID secondaire, par exemple mail, sera synchronisé avec l’attribut Azure AD userPrincipalName.
* Dans la liste ci-dessus, le type d'objet **Utilisateur** s'applique également au type d'objet **iNetOrgPerson**.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur la configuration de la [synchronisation Azure AD Connect](how-to-connect-sync-whatis.md) .

En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).