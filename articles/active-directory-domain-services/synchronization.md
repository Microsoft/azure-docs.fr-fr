---
title: Fonctionnement de la synchronisation dans Azure AD Domain Services | Microsoft Docs
description: Découvrez le fonctionnement du processus de synchronisation pour les objets et les informations d’identification d’un client Azure AD ou d’un environnement Active Directory Domain Services local à un domaine managé Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 57cbf436-fc1d-4bab-b991-7d25b6e987ef
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 41ba337765b4a0a93be52f08ae6656707cf7aa73
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618805"
---
# <a name="how-objects-and-credentials-are-synchronized-in-an-azure-active-directory-domain-services-managed-domain"></a>Synchronisation des objets et des informations d’identification dans un domaine managé Azure Active Directory Domain Services

Les objets et les informations d’identification d’un domaine managé Azure Active Directory Domain Services (Azure AD DS) peuvent être créés localement dans le domaine ou synchronisés à partir d’un locataire Azure Active Directory (Azure AD). Lorsque vous déployez Azure AD DS pour la première fois, une synchronisation automatique à sens unique est configurée et démarrée pour répliquer les objets à partir d’Azure AD. Cette synchronisation unidirectionnelle continue à s’exécuter en arrière-plan pour maintenir à jour le domaine managé Azure AD DS avec les modifications apportées par Azure AD. Aucune synchronisation n’est effectuée à partir d’Azure AD DS vers Azure AD.

Dans un environnement hybride, les objets et les informations d’identification d’un domaine AD DS local peuvent être synchronisés avec Azure AD à l’aide d’Azure AD Connect. Une fois que ces objets sont synchronisés avec succès sur Azure AD, la synchronisation en arrière-plan automatique rend ces objets et informations d’identification disponibles pour les applications utilisant le domaine managé.

Si l’environnement local AD DS et Azure AD sont configurés pour l’authentification fédérée avec ADFS, aucun hachage de mot de passe (actuel/valide) n’est disponible dans Azure DS. Il est possible que les comptes d’utilisateurs Azure AD créés avant l’implémentation de l’authentification fédérée disposent d’un ancien hachage de mot de passe, mais il est peu probable que celui-ci corresponde à un hachage de leur mot de passe local. De ce fait, Azure AD DS ne pourra pas valider les informations d’identification des utilisateurs.

Le diagramme suivant illustre le fonctionnement de la synchronisation entre Azure AD DS, Azure AD et un environnement AD DS local facultatif :

![Vue d’ensemble de la synchronisation dans un domaine managé Azure AD Domain Services](./media/active-directory-domain-services-design-guide/sync-topology.png)

## <a name="synchronization-from-azure-ad-to-azure-ad-ds"></a>Synchronisation à partir d’Azure AD vers Azure AD DS

Les comptes d’utilisateur, les appartenances aux groupes et les hachages des informations d’identification sont synchronisés dans une direction, d’Azure AD à Azure AD DS. Ce processus de synchronisation est automatique. Il est inutile de configurer, surveiller ou gérer le processus de synchronisation. La synchronisation initiale peut durer de quelques heures à quelques jours, en fonction du nombre d’objets contenus dans l’annuaire Azure AD. Une fois la synchronisation initiale terminée, les modifications apportées à Azure AD, telles que les modifications de mot de passe ou d’attribut, sont alors automatiquement synchronisées avec Azure AD DS.

Lorsqu’un utilisateur est créé dans Azure AD, il n’est pas synchronisé avec Azure AD DS tant qu’il n’a pas modifié son mot de passe dans Azure AD. Ce processus de changement du mot de passe entraîne la génération et le stockage dans Azure AD des hachages de mot de passe pour l’authentification Kerberos et NTLM. Les hachages de mot de passe sont nécessaires pour authentifier correctement un utilisateur dans Azure AD DS.

Le processus de synchronisation est unidirectionnel par nature. Il n’existe aucune synchronisation inverse des modifications d’Azure AD DS vers Azure AD. Un domaine managé est en grande partie en lecture seule, à l’exception des unités d’organisation personnalisées que vous pouvez créer. Vous ne pouvez pas changer les attributs utilisateur, les mots de passe utilisateur ou les appartenances aux groupes au sein d’un domaine managé.

## <a name="attribute-synchronization-and-mapping-to-azure-ad-ds"></a>Synchronisation des attributs et mappage avec Azure AD DS

Le tableau suivant répertorie certains attributs courants et la façon dont ils sont synchronisés avec Azure AD DS.

| Attribut dans Azure AD DS | Source | Notes |
|:--- |:--- |:--- |
| UPN | Attribut *UPN* de l’utilisateur dans votre locataire Azure AD | L’attribut UPN du locataire Azure AD est synchronisé tel quel pour Azure AD DS. La méthode la plus fiable pour se connecter à un domaine managé consiste à utiliser l’UPN. |
| SAMAccountName | Attribut *mailNickname* de l’utilisateur, dans le locataire Azure AD ou généré automatiquement | L’attribut *SAMAccountName* provient de l’attribut *mailNickname* dans le locataire Azure AD. Si plusieurs comptes d’utilisateurs ont le même attribut *mailNickname*, *SAMAccountName* est généré automatiquement. Si le paramètre *mailNickname* ou préfixe *UPN* dépasse 20 caractères, *SAMAccountName* est généré automatiquement pour répondre à la limite de 20 caractères sur les attributs *SAMAccountName*. |
| Mots de passe | Mot de passe utilisateur du locataire Azure AD | Les hachages de mot de passe hérités requis pour l’authentification NTLM ou Kerberos sont synchronisés à partir du locataire Azure AD. Si le locataire Azure AD est configuré pour la synchronisation hybride à l’aide d’Azure AD Connect, ces hachages de mot de passe proviennent de l’environnement AD DS local. |
| SID groupe/utilisateur principal | Généré automatiquement | Le SID principal pour les comptes de groupe/d’utilisateur est généré automatiquement dans Azure AD DS. Cet attribut ne correspond pas au SID de groupe/d’utilisateur principal de l’objet dans un environnement AD DS local. Cette incompatibilité est due au fait que le domaine managé a un espace de noms SID différent de celui du domaine AD DS local. |
| Historique des SID des utilisateurs et groupes | SID d’utilisateur et de groupe principal local | L'attribut *SidHistory* pour les utilisateurs et les groupes dans Azure AD DS est défini de sorte à correspondre au SID de groupe ou d’utilisateur principal correspondant dans un environnement AD DS local. Cette fonctionnalité permet de faciliter la migration des applications sur site vers Azure AD DS, étant donné que vous n’avez pas besoin de redéfinir les ACL des ressources. |

> [!TIP]
> **Se connecter au domaine managé en utilisant le format UPN** : l’attribut *SAMAccountName*, tel que `AADDSCONTOSO\driley`, peut être généré automatiquement pour certains comptes d’utilisateur dans un domaine managé. La valeur de *SAMAccountName* générée automatiquement pour l’utilisateur peut différer du préfixe UPN de ce dernier, donc n’est pas toujours un moyen fiable de se connecter.
>
> Par exemple, si plusieurs utilisateurs ont le même attribut *mailNickname* ou si des utilisateurs ont des préfixes UPN anormalement longs, la valeur *SAMAccountName* pour ces utilisateurs peut être générée automatiquement. Utilisez le format UPN, tel que `driley@aaddscontoso.com`, pour vous connecter de manière fiable à un domaine managé.

### <a name="attribute-mapping-for-user-accounts"></a>Mappage d’attributs pour les comptes d’utilisateur

Le tableau suivant illustre la façon dont certains attributs pour les objets utilisateur dans Azure AD sont synchronisés avec les attributs correspondants dans Azure AD DS.

| Attribut utilisateur dans Azure AD | Attribut utilisateur dans Azure AD DS |
|:--- |:--- |
| accountEnabled |userAccountControl (définit ou efface le bit ACCOUNT_DISABLED) |
| city |l |
| country |co |
| department |department |
| displayName |displayName |
| employeedId |employeeId |
| facsimileTelephoneNumber |facsimileTelephoneNumber |
| givenName |givenName |
| jobTitle |title |
| mail |mail |
| mailNickName |msDS-AzureADMailNickname |
| mailNickName |SAMAccountName (peut parfois être généré automatiquement) |
| manager |manager |
| mobile |mobile |
| objectid |msDS-AzureADObjectId |
| onPremiseSecurityIdentifier |sidHistory |
| passwordPolicies |userAccountControl (définit ou efface le bit DONT_EXPIRE_PASSWORD) |
| physicalDeliveryOfficeName |physicalDeliveryOfficeName |
| postalCode |postalCode |
| preferredLanguage |preferredLanguage |
| proxyAddresses | proxyAddresses |
| state |st |
| streetAddress |streetAddress |
| surname |sn |
| telephoneNumber |telephoneNumber |
| userPrincipalName |userPrincipalName |

### <a name="attribute-mapping-for-groups"></a>Mappage d’attributs pour les groupes

Le tableau suivant illustre la façon dont certains attributs pour les objets de groupe dans Azure AD sont synchronisés avec les attributs correspondants dans Azure AD DS.

| Attribut de groupe dans Azure AD | Attribut de groupe dans Azure AD DS |
|:--- |:--- |
| displayName |displayName |
| displayName |SAMAccountName (peut parfois être généré automatiquement) |
| mail |mail |
| mailNickName |msDS-AzureADMailNickname |
| objectid |msDS-AzureADObjectId |
| onPremiseSecurityIdentifier |sidHistory |
| proxyAddresses | proxyAddresses |
| securityEnabled |groupType |

## <a name="synchronization-from-on-premises-ad-ds-to-azure-ad-and-azure-ad-ds"></a>Synchronisation d’AD DS local vers Azure AD et Azure AD DS

Azure AD Connect est utilisé pour synchroniser les comptes d’utilisateur, les appartenances aux groupes et les hachages d’informations d’identification à partir d’un environnement AD DS local vers Azure AD. Les attributs des comptes d’utilisateur tels que l’UPN et l’identificateur de sécurité (SID) sont synchronisés en local. Pour vous connecter à l’aide d'Azure AD DS, les hachages de mot de passe hérités requis pour l'authentification NTLM et Kerberos sont également synchronisés avec Azure AD.

> [!IMPORTANT]
> Azure AD Connect doit uniquement être installé et configuré pour la synchronisation avec des environnements AD DS locaux. L’installation d’Azure AD Connect n’est pas prise en charge dans un domaine managé pour resynchroniser des objets sur Azure AD.

Si vous configurez l’écriture différée, les modifications d’Azure AD sont resynchronisées dans l’environnement AD DS local. Par exemple, si un utilisateur modifie son mot de passe à l’aide de la gestion des mots de passe en libre-service d’Azure AD, le mot de passe est mis à jour dans l’environnement AD DS local.

> [!NOTE]
> Utilisez toujours la version la plus récente d’Azure AD Connect pour vous assurer d'avoir les correctifs pour tous les bogues connus.

### <a name="synchronization-from-a-multi-forest-on-premises-environment"></a>Synchronisation d’un environnement local à plusieurs forêts

De nombreuses organisations disposent d’un environnement AD DS local relativement complexe qui comprend plusieurs forêts. Azure AD Connect prend en charge la synchronisation des utilisateurs, groupes et hachages d’informations d’identification à partir d’environnements à plusieurs forêts vers Azure AD.

Azure AD a un espace de noms bien plus simple et plat. Pour permettre aux utilisateurs d’accéder de manière fiable aux applications sécurisées par Azure AD, résolvez les conflits d’UPN entre comptes d’utilisateur dans des forêts différentes. Les domaines managés utilisent une structure d’unité d’organisation plate, similaire à Azure AD. Tous les comptes d’utilisateurs et les groupes sont stockés dans le conteneur *Utilisateurs AADDC*, en dépit de la synchronisation effectuée à partir de forêts ou de domaines locaux différents, même si vous avez configuré une structure d’unités d’organisation hiérarchique locale. Le domaine managé aplatit toutes les structures d’unités d’organisation hiérarchiques.

Comme nous l’avons vu précédemment, il n’existe aucune synchronisation d’Azure AD DS vers Azure AD. Vous pouvez [créer une unité d’organisation personnalisée](create-ou.md) dans Azure AD DS, puis des utilisateurs, des groupes ou des comptes de service au sein de ces unités d’organisation personnalisées. Aucun des objets créés dans les unités d’organisation personnalisées n’est de nouveau synchronisé avec Azure AD. Ces objets sont uniquement disponibles dans le domaine managé et ne sont pas visibles dans les applets de commande Azure AD PowerShell, l’API Microsoft Graph ou l’interface utilisateur de gestion Azure AD.

## <a name="what-isnt-synchronized-to-azure-ad-ds"></a>Éléments qui ne sont pas synchronisés avec Azure AD DS

Les objets ou attributs suivants ne sont pas synchronisés à partir d’un environnement AD DS local vers Azure AD ou Azure AD DS :

* **Exlure les attributs :** Vous pouvez choisir d’exclure certains attributs de la synchronisation avec Azure AD à partir d’un environnement AD DS local à l’aide d’Azure AD Connect. Ces attributs exclus ne sont alors pas disponibles dans Azure AD DS.
* **Stratégies de groupe :** Les stratégies de groupe configurées dans un environnement AD DS local ne sont pas synchronisées avec Azure AD DS.
* **Dossier Sysvol :** Le contenu du dossier *Sysvol* dans un environnement AD DS local n’est pas synchronisé avec Azure AD DS.
* **Objets ordinateur :** Les objets ordinateur pour les ordinateurs joints à un environnement AD DS local ne sont pas synchronisés avec Azure AD DS. Ces ordinateurs n’ont pas de relation d’approbation avec le domaine managé et n’appartiennent qu’à l’environnement AD DS local. Dans Azure AD DS, seuls les objets ordinateur pour les ordinateurs que vous avez explicitement joints au domaine managé s’affichent.
* **Attributs SidHistory des utilisateurs et groupes :** les SID de l’utilisateur principal et du groupe principal d’un environnement AD DS local sont synchronisés avec Azure AD DS. Toutefois, les attributs *SidHistory* existants pour les utilisateurs et les groupes ne sont pas synchronisés de l’environnement AD DS vers Azure AD DS.
* **Structures d’unités d’organisation (OU) :** Les unités d’organisation définies dans un environnement AD DS local ne sont pas synchronisées avec Azure AD DS. Il existe deux unités d’organisation intégrées dans Azure AD DS : une pour les utilisateurs et une pour les ordinateurs. Le domaine managé a une structure d’unité d’organisation plate. Vous pouvez choisir de [créer une unité d’organisation personnalisée dans votre domaine managé](create-ou.md).

## <a name="password-hash-synchronization-and-security-considerations"></a>Considérations relatives à la sécurité et à la synchronisation de hachage du mot de passe

Lorsque vous activez Azure AD DS, les hachages de mot de passe hérités pour l’authentification NTLM + Kerberos sont requis. Azure AD ne stocke pas les mots de passe en texte clair. Par conséquent, ces hachages ne peuvent pas être générés automatiquement pour les comptes d'utilisateur existants. Une fois générés et stockés, les hachages de mot de passe compatibles NTLM et Kerberos sont toujours stockés sous forme chiffrée dans Azure AD.

Les clés de chiffrement sont uniques pour chaque locataire Azure AD. Ces hachages sont chiffrés afin que seul Azure AD DS ait accès aux clés de déchiffrement. Aucun autre service ni composant d’Azure AD n’a accès aux clés de déchiffrement.

Les hachages de mot de passe hérités sont ensuite synchronisés à partir d’Azure AD dans les contrôleurs de domaine pour un domaine managé. Les disques de ces contrôleurs de domaine managé dans Azure AD DS sont chiffrés à l’arrêt. Ces hachages de mot de passe sont stockés et sécurisés sur ces contrôleurs de domaine de la même manière que les mots de passe stockés et sécurisés dans un environnement AD DS local.

Pour les environnements Azure AD cloud uniquement, les [utilisateurs doivent réinitialiser/modifier leur mot de passe](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) afin que les hachages de mot de passe requis soient générés et stockés dans Azure AD. Pour tout compte d’utilisateur cloud créé dans Azure AD après l’activation d’Azure AD Domain Services, les hachages de mot de passe sont générés et stockés dans des formats compatibles NTLM et Kerberos. Tous les comptes d’utilisateur cloud doivent modifier leur mot de passe pour pouvoir être synchronisés avec Azure AD DS.

Pour les comptes d’utilisateurs hybrides synchronisés à partir d’un environnement AD DS local à l’aide d’Azure AD Connect, vous devez [configurer Azure AD Connect pour synchroniser les hachages de mot de passe dans des formats compatibles NTLM et Kerberos](tutorial-configure-password-hash-sync.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les détails de la synchronisation de mot de passe, consultez [Fonctionnement de la synchronisation de hachage du mot de passe avec Azure AD Connect](../active-directory/hybrid/how-to-connect-password-hash-synchronization.md?context=/azure/active-directory-domain-services/context/azure-ad-ds-context).

Pour commencer à utiliser Azure AD DS, [créez un domaine managé](tutorial-create-instance.md).
