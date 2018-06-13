---
title: Connectez Active Directory à Azure Active Directory. | Microsoft Docs
description: Azure AD Connect intègre vos répertoires locaux à Azure Active Directory. Cela vous permet de fournir une identité commune pour les applications Office 365, Azure et SaaS intégrées à Azure AD.
keywords: introduction à Azure AD Connect, présentation d’Azure AD Connect, qu’est-ce qu’Azure AD Connect, installation d’active directory
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 59bd209e-30d7-4a89-ae7a-e415969825ea
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 03/19/2018
ms.author: billmath
ms.openlocfilehash: 128a6bffe4e604eb4e02a10f13d561a336fc8c61
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/20/2018
ms.locfileid: "34354416"
---
# <a name="integrate-your-on-premises-directories-with-azure-active-directory"></a>Intégrer des répertoires locaux à Azure Active Directory
Azure AD Connect intègre vos répertoires locaux à Azure Active Directory. Cela vous permet de fournir une identité commune à vos utilisateurs pour les applications Office 365, Azure et SaaS intégrées à Azure AD. Cette rubrique vous guide dans les étapes de planification, de déploiement et opérationnelles. Elle comporte un ensemble de liens vers les rubriques associées à cette zone.

> [!IMPORTANT]
> [Azure AD Connect est la meilleure façon de connecter votre répertoire local avec Azure AD et Office 365. C’est l’occasion idéale d’effectuer la mise à niveau vers Azure AD Connect à partir de Microsoft Azure Active Directory Sync (DirSync) ou Azure AD Sync, car ces outils sont désormais dépréciés et ne sont plus pris en charge depuis le 13 avril 2017.](active-directory-aadconnect-dirsync-deprecated.md)  De plus :



> 
> - La synchronisation des utilisateurs à Azure AD est une **fonctionnalité gratuite**, et il n’est pas nécessaire de disposer d’un abonnement payant.
>- Les utilisateurs synchronisés **ne reçoivent *aucune* licence automatiquement**. Les administrateurs gardent le contrôle total sur l’attribution des licences. 
> - Microsoft recommande aux administrateurs informatiques de synchroniser tous les utilisateurs. Non seulement cela débloque les utilisateurs pour accéder à toute ressource intégrée Azure AD, mais cela permet aux administrateurs informatiques de visualiser de façon beaucoup plus large les applications auxquelles leurs utilisateurs accèdent. 

![Qu’est-ce qu’Azure AD Connect ?](media/active-directory-aadconnect/arch.png)

## <a name="why-use-azure-ad-connect"></a>Pourquoi utiliser Azure AD Connect
L’intégration de vos annuaires locaux avec Azure AD améliore la productivité de vos utilisateurs en leur fournissant une identité commune pour accéder aux ressources cloud et locales. Les utilisateurs et les organisations bénéficient des avantages suivants :

* Les utilisateurs peuvent utiliser une identité unique pour accéder aux applications locales et aux services de cloud comme Office 365.
* Outil unique offrant une expérience de déploiement simple pour la synchronisation et la connexion.
* Fournit les fonctionnalités les plus récentes pour vos scénarios. Azure AD Connect remplace les versions antérieures des outils d’intégration d’identité tels que DirSync et Azure AD Sync. Pour plus d’informations, consultez [Identité hybride : Comparaison des outils d’intégration d’annuaire](../active-directory-hybrid-identity-design-considerations-tools-comparison.md).

### <a name="how-azure-ad-connect-works"></a>Fonctionnement d’Azure AD Connect
Azure Active Directory Connect est constitué de trois composants principaux : les services de synchronisation, le composant Active Directory Federation Services facultatif et le composant d’analyse nommé [Azure AD Connect Health](../connect-health/active-directory-aadconnect-health.md).

<center>![Pile Azure AD Connect](./media/active-directory-aadconnect-how-it-works/AADConnectStack2.png)
</center>

* Synchronisation : ce composant est chargé de créer des utilisateurs, des groupes et autres objets. Il permet également de s’assurer que les informations d’identité relatives aux utilisateurs et aux groupes dans votre environnement local correspondent à celles qui se trouvent dans le cloud.
* AD FS : ceci est une partie facultative d’Azure AD Connect, qui peut être utilisée pour configurer un environnement hybride à l’aide d’une infrastructure AD FS locale. Cette partie permet aux organisations de faire face à des déploiements complexes, par exemple l’authentification unique de jonction de domaine, l’application de la stratégie de connexion AD ou l’utilisation de cartes à puce ou d’une solution tierce d’authentification multifacteur.
* Analyse d’intégrité : Azure AD Connect Health peut assurer une surveillance robuste et offrir l’accès à un emplacement central dans le portail Azure pour afficher ces activités. Pour plus d’informations, voir [Azure Active Directory Connect Health](../connect-health/active-directory-aadconnect-health.md).

## <a name="install-azure-ad-connect"></a>Installer Azure AD Connect

> [!IMPORTANT]
> Microsoft ne prend pas en charge la modification ou l’utilisation de la synchronisation Azure AD Connect en dehors des actions qui sont documentées de façon formelle. Ces actions peuvent entraîner un état de synchronisation Azure AD Connect incohérent ou non pris en charge. Par conséquent, Microsoft ne peut pas fournir de support technique pour ces déploiements.

Vous pouvez trouver le téléchargement d’Azure AD Connect sur le [Centre de téléchargement Microsoft](http://go.microsoft.com/fwlink/?LinkId=615771).

| Solution | Scénario |
| --- | --- |
| Avant de commencer : [Matériel et conditions préalables](active-directory-aadconnect-prerequisites.md) |<li>Étapes à suivre avant de commencer à installer Azure AD Connect.</li> |
| [Paramètres Express](active-directory-aadconnect-get-started-express.md) |<li>Si vous ne disposez que d’une seule forêt Active Directory, cette option est recommandée.</li> <li>Connexion de l’utilisateur avec le même mot de passe à l’aide de la synchronisation du mot de passe.</li> |
| [Paramètres personnalisés](active-directory-aadconnect-get-started-custom.md) |<li>Utilisés lorsque vous disposez de plusieurs forêts. Prise en charge de nombreuses [topologies](active-directory-aadconnect-topologies.md) locales.</li> <li>Personnalisez votre option de connexion, par exemple l’authentification directe, ADFS pour la fédération, ou utilisez un fournisseur d’identité tiers.</li> <li>Personnalisez les fonctionnalités de synchronisation, telles que le filtrage et l’écriture différée.</li> |
| [Mise à niveau à partir de DirSync](active-directory-aadconnect-dirsync-upgrade-get-started.md) |<li>Utilisé lorsque vous disposez d’un serveur DirSync existant déjà en cours d’exécution.</li> |
| [Mise à niveau à partir d’Azure AD Sync ou d’Azure AD Connect](active-directory-aadconnect-upgrade-previous-version.md) |<li>Il existe plusieurs méthodes différentes, en fonction de vos préférences.</li> |

[Après l’installation](active-directory-aadconnect-whats-next.md) , vous devez vérifier que tout fonctionne comme prévu et affecter des licences aux utilisateurs.

### <a name="next-steps-to-install-azure-ad-connect"></a>Étapes suivantes pour installer Azure AD Connect
|Rubrique |Lien|  
| --- | --- |
|Téléchargez Azure AD Connect | [Téléchargez Azure AD Connect](http://go.microsoft.com/fwlink/?LinkId=615771)|
|Installation à l’aide des paramètres Express | [Installation rapide pour Azure AD Connect](./active-directory-aadconnect-get-started-express.md)|
|Installation à l’aide des paramètres personnalisés | [Installation personnalisée d’Azure AD Connect](./active-directory-aadconnect-get-started-custom.md)|
|Effectuer une mise à niveau à partir de DirSync | [Effectuer une mise à niveau à partir de l’outil de synchronisation Azure AD (DirSync)](./active-directory-aadconnect-dirsync-upgrade-get-started.md)|
|Après l’installation | [Vérification de l’installation et affectation des licences ](active-directory-aadconnect-whats-next.md)|

### <a name="learn-more-about-install-azure-ad-connect"></a>En savoir plus sur l’installation d’Azure AD Connect
Il peut également être judicieux de se préparer aux préoccupations [opérationnelles](active-directory-aadconnectsync-operations.md) . Vous pouvez souhaiter disposer d’un serveur de secours afin de pouvoir facilement basculer en cas de [sinistre](active-directory-aadconnectsync-operations.md#disaster-recovery). Si vous envisagez d’effectuer des modifications de configuration fréquentes, vous devriez planifier un serveur en [mode de préproduction](active-directory-aadconnectsync-operations.md#staging-mode) .

|Rubrique |Lien|  
| --- | --- |
|Topologies prises en charge | [Topologies pour Azure AD Connect](active-directory-aadconnect-topologies.md)|
|Principes de conception | [Principes de conception Azure AD Connect](active-directory-aadconnect-design-concepts.md)|
|Comptes utilisés pour l’installation | [Autorisations et informations d’identification Azure AD Connect](./active-directory-aadconnect-accounts-permissions.md)|
|Planification opérationnelle | [Azure Connect AD sync : tâches et examen opérationnels](active-directory-aadconnectsync-operations.md)|
|Options de connexion utilisateur | [Options de connexion de l’utilisateur via Azure AD Connect](active-directory-aadconnect-user-signin.md)|

## <a name="configure-sync-features"></a>Configuration de fonctionnalités de synchronisation
Azure AD Connect est doté de plusieurs fonctionnalités que vous pouvez activer ou qui sont activées par défaut. Certaines fonctionnalités peuvent parfois nécessiter une configuration supplémentaire dans des topologies et scénarios spécifiques.

[filtrage](active-directory-aadconnectsync-configure-filtering.md) est utilisé lorsque vous souhaitez limiter le nombre d’objets synchronisés sur Azure AD. Par défaut, tous les utilisateurs, contacts, groupes et ordinateurs Windows 10 sont synchronisés. Vous pouvez modifier le filtrage en fonction des domaines, des unités d’organisation ou des attributs.

La [synchronisation de hachage du mot de passe](active-directory-aadconnectsync-implement-password-hash-synchronization.md) synchronise le hachage du mot de passe dans Active Directory sur Azure AD. Cela permet à l’utilisateur final d’utiliser le même mot de passe en local et dans le cloud, mais uniquement de le gérer dans un seul emplacement. Dans la mesure où cela utilise votre Active Directory local en tant qu’autorité, vous pourrez également utiliser votre propre stratégie de mot de passe.

[écriture différée du mot de passe](../authentication/quickstart-sspr.md) permettra à vos utilisateurs de modifier et de réinitialiser leurs mots de passe dans le cloud et d’appliquer votre stratégie de mot de passe locale.

[L’écriture différée d’appareils](active-directory-aadconnect-feature-device-writeback.md) autorise un appareil inscrit dans Azure AD à bénéficier de l’écriture différée dans Active Directory en local afin de pouvoir être utilisé pour l’accès conditionnel.

La fonctionnalité de [prévention des suppressions accidentelles](active-directory-aadconnectsync-feature-prevent-accidental-deletes.md) est activée par défaut et protège votre répertoire du cloud d’un grand nombre de suppressions en même temps. Par défaut, elle permet 500 suppressions par exécution. Vous pouvez modifier ce paramètre en fonction de la taille de votre organisation.

La [mise à niveau automatique](active-directory-aadconnect-feature-automatic-upgrade.md) est activée par défaut pour une installation rapide des paramètres. Elle garantit qu’Azure AD Connect est toujours à jour avec la dernière version.

### <a name="next-steps-to-configure-sync-features"></a>Étapes suivantes pour configurer des fonctionnalités de synchronisation
|Rubrique |Lien|  
| --- | --- |
|Configurer le filtrage | [Azure AD Connect Sync : Configurer le filtrage](active-directory-aadconnectsync-configure-filtering.md)|
|Synchronisation de hachage de mot de passe | [Azure AD Connect Sync : implémenter la synchronisation de hachage du mot de passe](active-directory-aadconnectsync-implement-password-hash-synchronization.md)|
|Réécriture du mot de passe | [Prise en main de la gestion de mot de passe](../authentication/quickstart-sspr.md)|
|L’écriture différée d’appareils | [Activation de l’écriture différée des appareils dans Azure AD Connect](active-directory-aadconnect-feature-device-writeback.md)|
|prévention des suppressions accidentelles | [Azure AD Connect Sync : Prévention des suppressions accidentelles](active-directory-aadconnectsync-feature-prevent-accidental-deletes.md)|
|Mise à jour automatique | [Azure AD Connect : Mise à niveau automatique](active-directory-aadconnect-feature-automatic-upgrade.md)|

## <a name="customize-azure-ad-connect-sync"></a>Personnaliser Azure AD Connect Sync
Azure Connect AD Sync est doté d’une configuration par défaut qui est destinée à fonctionner pour la plupart des clients et des topologies. Toutefois, il existe toujours des situations dans lesquelles la configuration par défaut ne fonctionne pas et doit être ajustée. Il est possible d’apporter les modifications documentées dans cette section, ainsi que dans les rubriques connexes.

Si vous n’avez jamais travaillé avec une topologie de synchronisation auparavant, vous devriez commencer par assimiler les notions de base et les termes utilisés décrits dans les [concepts techniques](active-directory-aadconnectsync-technical-concepts.md). Azure AD Connect est l’évolution de MIIS2003, ILM2007 et FIM2010. Bien que certains éléments soient identiques, beaucoup de choses ont changé.

La [configuration par défaut](active-directory-aadconnectsync-understanding-default-configuration.md) suppose la présence possible de plusieurs forêts. Dans ces topologies, un objet utilisateur peut être représenté comme un contact dans une autre forêt. L’utilisateur peut également disposer d’une boîte aux lettres liée dans une autre forêt de ressources. Le comportement de la configuration par défaut est décrit dans [Utilisateurs et contacts](active-directory-aadconnectsync-understanding-users-and-contacts.md).

Le modèle de configuration dans la synchronisation est appelé [Approvisionnement déclaratif](active-directory-aadconnectsync-understanding-declarative-provisioning-expressions.md). Les flux des attributs avancés utilisent des [fonctions](active-directory-aadconnectsync-functions-reference.md) pour exprimer les transformations des attributs. Vous pouvez afficher et examiner l’intégralité de la configuration à l’aide des outils fournis avec Azure AD Connect. Si vous devez apporter des modifications à la configuration, assurez-vous de suivre les [meilleures pratiques](active-directory-aadconnectsync-best-practices-changing-default-configuration.md) afin que les nouvelles versions soient plus faciles à adopter.

### <a name="next-steps-to-customize-azure-ad-connect-sync"></a>Étapes suivantes pour personnaliser Azure AD Connect Sync
|Rubrique |Lien|  
| --- | --- |
|Tous les articles sur la synchronisation Azure AD Connect | [Synchronisation d’Azure AD Connect](active-directory-aadconnectsync-whatis.md)|
|concepts techniques | [Azure AD Connect Sync : Concepts techniques](active-directory-aadconnectsync-technical-concepts.md)|
|Présentation de la configuration par défaut | [Azure AD Connect Sync : présentation de la configuration par défaut](active-directory-aadconnectsync-understanding-default-configuration.md)|
|Présentation des utilisateurs et des contacts | [Azure AD Connect Sync : Présentation des utilisateurs et des contacts](active-directory-aadconnectsync-understanding-users-and-contacts.md)|
|Approvisionnement déclaratif | [Azure AD Connect Sync : présentation des expressions d’approvisionnement déclaratif](active-directory-aadconnectsync-understanding-declarative-provisioning-expressions.md)|
|Modifier la configuration par défaut | [Meilleures pratiques pour la modification de la configuration par défaut](active-directory-aadconnectsync-best-practices-changing-default-configuration.md)|

## <a name="configure-federation-features"></a>Configuration de fonctionnalités de fédération

Azure AD Connect fournit plusieurs fonctionnalités qui simplifient la fédération avec Azure AD à l’aide d’AD FS, ainsi que la gestion de l’approbation de fédération. Azure AD Connect prends en charge AD FS sur Windows Server 2012R2 et les versions ultérieures.

[Mettez à jour le certificat SSL de la batterie de serveurs AD FS](active-directory-aadconnectfed-ssl-update.md), même si vous n’utilisez pas Azure AD Connect pour gérer l’approbation de votre fédération.

[Ajoutez un serveur AD FS](active-directory-aadconnect-federation-management.md#addadfsserver) à votre batterie de serveurs pour l’étendre selon vos besoins.

[Réparez l’approbation](active-directory-aadconnect-federation-management.md#repairthetrust) avec Azure AD en quelques clics.

ADFS peut être configuré de manière à prendre en charge [plusieurs domaines](active-directory-aadconnect-multiple-domains.md). Par exemple, vous pouvez avoir plusieurs domaines principaux que vous devez utiliser pour la fédération.

Si votre serveur ADFS n’a pas été configuré pour mettre à jour automatiquement les certificats Azure AD ou si vous utilisez une solution non ADFS, vous recevrez une notification lorsque vous devrez [mettre à jour des certificats](active-directory-aadconnect-o365-certs.md).

### <a name="next-steps-to-configure-federation-features"></a>Étapes suivantes pour configurer des fonctionnalités de fédération
|Rubrique |Lien|  
| --- | --- |
|Tous les articles AD FS | [Fédération avec Azure AD Connect](active-directory-aadconnectfed-whatis.md)|
|Configuration d’ADFS avec des sous-domaines | [Prise en charge de plusieurs domaines pour la fédération avec Azure AD](active-directory-aadconnect-multiple-domains.md)|
|Gérer la batterie AD FS | [Gestion AD FS et personnalisation avec Azure AD Connect](active-directory-aadconnect-federation-management.md)|
|Mise à jour manuelle de certificats de fédération | [Renouvellement des certificats de fédération pour Office 365 et Azure AD](active-directory-aadconnect-o365-certs.md)|

## <a name="more-information-and-references"></a>Plus d’informations et de références
|Rubrique |Lien|  
| --- | --- |
|Historique des versions | [Historique des versions](active-directory-aadconnect-version-history.md)|
|Comparatif DirSync, Azure ADSync et Azure AD Connect | [Comparaison des outils d’intégration de répertoire](../active-directory-hybrid-identity-design-considerations-tools-comparison.md)|
|Liste de compatibilité non-ADFS pour Azure AD | [Liste de compatibilité de fédération Azure AD](active-directory-aadconnect-federation-compatibility.md)|
|Configuration d’un IdP SAML 2.0|[Utilisation d’un fournisseur d’identité (IdP) SAML 2.0 pour l’authentification unique](active-directory-aadconnect-federation-saml-idp.md)|
|Attributs synchronisés | [Attributs synchronisés](active-directory-aadconnectsync-attributes-synchronized.md)|
|Surveillance à l’aide d’Azure AD Connect Health | [Azure AD Connect Health](../connect-health/active-directory-aadconnect-health.md)|
|Forum Aux Questions (FAQ) | [FAQ Azure AD Connect](active-directory-aadconnect-faq.md)|

**Ressources supplémentaires**

Présentation d’Ignite 2015 sur l'extension de vos répertoires locaux dans le cloud.

> [!VIDEO https://channel9.msdn.com/Events/Ignite/2015/BRK3862/player]
> 
> 

