---
title: Feuille de route pour l’installation d’Azure AD Connect et d’Azure AD Connect Health. | Microsoft Docs
description: Ce document fournit une vue d’ensemble des options et des chemins d’installation disponibles pour l’installation d’Azure AD Connect et de Connect Health.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/18/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: ae575aa6544a174a70eb8ea4749566e8660280e2
ms.sourcegitcommit: 642988f1ac17cfd7a72ad38ce38ed7a5c2926b6c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94873265"
---
# <a name="azure-ad-connect-and-azure-ad-connect-health-installation-roadmap"></a>Feuille de route pour l’installation d’Azure AD Connect et d’Azure AD Connect Health

## <a name="install-azure-ad-connect"></a>Installer Azure AD Connect

> [!IMPORTANT]
> Microsoft ne prend pas en charge la modification ou l’utilisation de la synchronisation Azure AD Connect en dehors des actions qui sont documentées de façon formelle. Ces actions peuvent entraîner un état de synchronisation Azure AD Connect incohérent ou non pris en charge. Par conséquent, Microsoft ne peut pas fournir de support technique pour ces déploiements.

Vous pouvez trouver le téléchargement d’Azure AD Connect sur le [Centre de téléchargement Microsoft](https://go.microsoft.com/fwlink/?LinkId=615771).

| Solution | Scénario |
| --- | --- |
| Avant de commencer : [Matériel et conditions préalables](how-to-connect-install-prerequisites.md) |<li>Étapes à suivre avant de commencer à installer Azure AD Connect.</li> |
| [Paramètres Express](how-to-connect-install-express.md) |<li>Si vous ne disposez que d’une seule forêt Active Directory, cette option est recommandée.</li> <li>Connexion de l’utilisateur avec le même mot de passe à l’aide de la synchronisation du mot de passe.</li> |
| [Paramètres personnalisés](how-to-connect-install-custom.md) |<li>Utilisés lorsque vous disposez de plusieurs forêts. Prise en charge de nombreuses [topologies](plan-connect-topologies.md) locales.</li> <li>Personnalisez votre option de connexion, par exemple l’authentification directe, ADFS pour la fédération, ou utilisez un fournisseur d’identité tiers.</li> <li>Personnalisez les fonctionnalités de synchronisation, telles que le filtrage et l’écriture différée.</li> |
| [Mise à niveau à partir de DirSync](how-to-dirsync-upgrade-get-started.md) |<li>Utilisé lorsque vous disposez d’un serveur DirSync existant déjà en cours d’exécution.</li> |
| [Mise à niveau à partir d’Azure AD Sync ou d’Azure AD Connect](how-to-upgrade-previous-version.md) |<li>Il existe plusieurs méthodes différentes, en fonction de vos préférences.</li> |

[Après l’installation](how-to-connect-post-installation.md) , vous devez vérifier que tout fonctionne comme prévu et affecter des licences aux utilisateurs.

### <a name="next-steps-to-install-azure-ad-connect"></a>Étapes suivantes pour installer Azure AD Connect
|Rubrique |Lien|  
| --- | --- |
|Téléchargez Azure AD Connect | [Téléchargez Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771)|
|Installation à l’aide des paramètres Express | [Installation rapide pour Azure AD Connect](./how-to-connect-install-express.md)|
|Installation à l’aide des paramètres personnalisés | [Installation personnalisée d’Azure AD Connect](./how-to-connect-install-custom.md)|
|Effectuer une mise à niveau à partir de DirSync | [Effectuer une mise à niveau à partir de l’outil de synchronisation Azure AD (DirSync)](./how-to-dirsync-upgrade-get-started.md)|
|Après l’installation | [Vérification de l’installation et affectation des licences](how-to-connect-post-installation.md)|

### <a name="learn-more-about-install-azure-ad-connect"></a>En savoir plus sur l’installation d’Azure AD Connect
Il peut également être judicieux de se préparer aux préoccupations [opérationnelles](./how-to-connect-sync-staging-server.md) . Vous pouvez souhaiter disposer d’un serveur de secours afin de pouvoir facilement basculer en cas de [sinistre](how-to-connect-sync-staging-server.md#disaster-recovery). Si vous envisagez d’effectuer des modifications de configuration fréquentes, vous devriez planifier un serveur en [mode intermédiaire](how-to-connect-sync-staging-server.md) .

|Rubrique |Lien|  
| --- | --- |
|Topologies prises en charge | [Topologies pour Azure AD Connect](plan-connect-topologies.md)|
|Principes de conception | [Principes de conception Azure AD Connect](plan-connect-design-concepts.md)|
|Comptes utilisés pour l’installation | [Autorisations et informations d’identification Azure AD Connect](reference-connect-accounts-permissions.md)|
|Planification opérationnelle | [Synchronisation Azure AD Connect : tâches opérationnelles et examen](./how-to-connect-sync-staging-server.md)|
|Options de connexion utilisateur | [Options de connexion de l’utilisateur via Azure AD Connect](plan-connect-user-signin.md)|

## <a name="configure-sync-features"></a>Configuration des fonctionnalités de synchronisation
Azure AD Connect est doté de plusieurs fonctionnalités que vous pouvez activer ou qui sont activées par défaut. Certaines fonctionnalités peuvent parfois nécessiter une configuration supplémentaire dans des topologies et scénarios spécifiques.

[filtrage](how-to-connect-sync-configure-filtering.md) est utilisé lorsque vous souhaitez limiter le nombre d’objets synchronisés sur Azure AD. Par défaut, tous les utilisateurs, contacts, groupes et ordinateurs Windows 10 sont synchronisés. Vous pouvez modifier le filtrage en fonction des domaines, des unités d’organisation ou des attributs.

La [synchronisation de hachage du mot de passe](how-to-connect-password-hash-synchronization.md) synchronise le hachage du mot de passe dans Active Directory sur Azure AD. Cela permet à l’utilisateur final d’utiliser le même mot de passe en local et dans le cloud, mais uniquement de le gérer dans un seul emplacement. Dans la mesure où cela utilise votre Active Directory local en tant qu’autorité, vous pourrez également utiliser votre propre stratégie de mot de passe.

[écriture différée du mot de passe](../authentication/tutorial-enable-sspr.md) permettra à vos utilisateurs de modifier et de réinitialiser leurs mots de passe dans le cloud et d’appliquer votre stratégie de mot de passe locale.

La [réécriture d’appareil](how-to-connect-device-writeback.md) autorise un appareil inscrit dans Azure AD à bénéficier de la réécriture dans Active Directory en local afin de pouvoir être utilisé pour l’accès conditionnel.

La fonctionnalité de [prévention des suppressions accidentelles](how-to-connect-sync-feature-prevent-accidental-deletes.md) est activée par défaut et protège votre répertoire du cloud d’un grand nombre de suppressions en même temps. Par défaut, elle permet 500 suppressions par exécution. Vous pouvez modifier ce paramètre en fonction de la taille de votre organisation.

La [mise à niveau automatique](how-to-connect-install-automatic-upgrade.md) est activée par défaut pour une installation rapide des paramètres. Elle garantit qu’Azure AD Connect est toujours à jour avec la dernière version.

### <a name="next-steps-to-configure-sync-features"></a>Étapes suivantes pour configurer des fonctionnalités de synchronisation
|Rubrique |Lien|  
| --- | --- |
|Configurer le filtrage | [Synchronisation Azure AD Connect : Configurer le filtrage](how-to-connect-sync-configure-filtering.md)|
|Synchronisation de hachage de mot de passe | [Synchronisation de hachage de mot de passe](how-to-connect-password-hash-synchronization.md)|
|Authentification directe | [Authentification directe](how-to-connect-pta.md)
|Réécriture du mot de passe | [Prise en main de la gestion de mot de passe](../authentication/tutorial-enable-sspr.md)|
|Écriture différée des appareils | [Activation de l’écriture différée des appareils dans Azure AD Connect](how-to-connect-device-writeback.md)|
|prévention des suppressions accidentelles | [Synchronisation Azure AD Connect : prévention des suppressions accidentelles](how-to-connect-sync-feature-prevent-accidental-deletes.md)|
|Mise à jour automatique | [Azure AD Connect : mise à niveau automatique](how-to-connect-install-automatic-upgrade.md)|

## <a name="customize-azure-ad-connect-sync"></a>Personnaliser Azure AD Connect Sync
Azure Connect AD Sync est doté d’une configuration par défaut qui est destinée à fonctionner pour la plupart des clients et des topologies. Toutefois, il existe toujours des situations dans lesquelles la configuration par défaut ne fonctionne pas et doit être ajustée. Il est possible d’apporter les modifications documentées dans cette section, ainsi que dans les rubriques connexes.

Si vous n’avez jamais travaillé avec une topologie de synchronisation auparavant, vous devriez commencer par assimiler les notions de base et les termes utilisés décrits dans les [concepts techniques](how-to-connect-sync-technical-concepts.md). Azure AD Connect est l’évolution de MIIS2003, ILM2007 et FIM2010. Bien que certains éléments soient identiques, beaucoup de choses ont changé.

La [configuration par défaut](concept-azure-ad-connect-sync-default-configuration.md) suppose la présence possible de plusieurs forêts. Dans ces topologies, un objet utilisateur peut être représenté comme un contact dans une autre forêt. L’utilisateur peut également disposer d’une boîte aux lettres liée dans une autre forêt de ressources. Le comportement de la configuration par défaut est décrit dans [Utilisateurs et contacts](concept-azure-ad-connect-sync-user-and-contacts.md).

Le modèle de configuration dans la synchronisation est appelé [Approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md). Les flux des attributs avancés utilisent des [fonctions](reference-connect-sync-functions-reference.md) pour exprimer les transformations des attributs. Vous pouvez afficher et examiner l’intégralité de la configuration à l’aide des outils fournis avec Azure AD Connect. Si vous devez apporter des modifications à la configuration, assurez-vous de suivre les [meilleures pratiques](how-to-connect-sync-best-practices-changing-default-configuration.md) afin que les nouvelles versions soient plus faciles à adopter.

### <a name="next-steps-to-customize-azure-ad-connect-sync"></a>Étapes suivantes pour personnaliser Azure AD Connect Sync
|Rubrique |Lien|  
| --- | --- |
|Tous les articles sur la synchronisation Azure AD Connect | [Synchronisation d’Azure AD Connect](how-to-connect-sync-whatis.md)|
|Concepts techniques | [Synchronisation Azure AD Connect : concepts techniques](how-to-connect-sync-technical-concepts.md)|
|Présentation de la configuration par défaut | [Synchronisation Azure AD Connect : comprendre la configuration par défaut](concept-azure-ad-connect-sync-default-configuration.md)|
|Présentation des utilisateurs et des contacts | [Synchronisation Azure AD Connect : présentation des utilisateurs et des contacts](concept-azure-ad-connect-sync-user-and-contacts.md)|
|Approvisionnement déclaratif | [Synchronisation Azure AD Connect : comprendre les expressions de provisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)|
|Modifier la configuration par défaut | [Meilleures pratiques pour la modification de la configuration par défaut](how-to-connect-sync-best-practices-changing-default-configuration.md)|

## <a name="configure-federation-features"></a>Configuration de fonctionnalités de fédération

Azure AD Connect fournit plusieurs fonctionnalités qui simplifient la fédération avec Azure AD à l’aide d’AD FS, ainsi que la gestion de l’approbation de fédération. Azure AD Connect prends en charge AD FS sur Windows Server 2012R2 et les versions ultérieures.

[Mettez à jour le certificat TLS/SSL de la batterie de serveurs AD FS](how-to-connect-fed-ssl-update.md), même si vous n’utilisez pas Azure AD Connect pour gérer l’approbation de votre fédération.

[Ajoutez un serveur AD FS](how-to-connect-fed-management.md#addadfsserver) à votre batterie de serveurs pour l’étendre selon vos besoins.

[Réparez l’approbation](how-to-connect-fed-management.md#repairthetrust) avec Azure AD en quelques clics.

ADFS peut être configuré de manière à prendre en charge [plusieurs domaines](how-to-connect-install-multiple-domains.md). Par exemple, vous pouvez avoir plusieurs domaines principaux que vous devez utiliser pour la fédération.

Si votre serveur ADFS n’a pas été configuré pour mettre à jour automatiquement les certificats Azure AD ou si vous utilisez une solution non ADFS, vous recevrez une notification lorsque vous devrez [mettre à jour des certificats](how-to-connect-fed-o365-certs.md).

### <a name="next-steps-to-configure-federation-features"></a>Étapes suivantes pour configurer des fonctionnalités de fédération
|Rubrique |Lien|  
| --- | --- |
|Tous les articles AD FS | [Fédération avec Azure AD Connect](how-to-connect-fed-whatis.md)|
|Configuration d’ADFS avec des sous-domaines | [Prise en charge de plusieurs domaines pour la fédération avec Azure AD](how-to-connect-install-multiple-domains.md)|
|Gérer la batterie AD FS | [Gestion AD FS et personnalisation avec Azure AD Connect](how-to-connect-fed-management.md)|
|Mise à jour manuelle des certificats de fédération | [Renouvellement des certificats de fédération pour Microsoft 365 et Azure AD](how-to-connect-fed-o365-certs.md)|


## <a name="get-started-with-azure-ad-connect-health"></a>Démarrer avec Azure AD Connect Health
Pour vous familiariser avec Azure AD Connect Health, procédez comme suit :

1. Accédez à [Obtenir Azure AD Premium](../fundamentals/active-directory-get-started-premium.md) ou [Démarrer une version d’évaluation](https://azure.microsoft.com/trial/get-started-active-directory/).
2. [Téléchargez et installez les agents Azure AD Connect Health](#download-and-install-azure-ad-connect-health-agent) sur vos serveurs d’identité.
3. Affichez le tableau de bord Azure AD Connect Health disponible à l’adresse [https://aka.ms/aadconnecthealth](https://aka.ms/aadconnecthealth).

> [!NOTE]
> Avant de pouvoir consulter des données sur votre tableau de bord Azure AD Connect Health, il vous faudra installer les agents Azure AD Connect Health sur vos serveurs cibles.
>
>

## <a name="download-and-install-azure-ad-connect-health-agent"></a>Téléchargement et installation de l’agent Azure AD Connect Health
* Vérifiez que vous [disposez de la configuration requise](how-to-connect-health-agent-install.md#requirements) pour Azure AD Connect Health.
* Prise en main d’Azure AD Connect Health pour AD FS
    * [Téléchargez l’agent Azure AD Connect Health pour AD FS.](https://go.microsoft.com/fwlink/?LinkID=518973)
    * [Consultez les instructions d’installation](how-to-connect-health-agent-install.md#install-the-agent-for-ad-fs).
* Prise en main d’Azure AD Connect Health pour la synchronisation
    * [Téléchargez et installez la dernière version d’Azure AD Connect](https://go.microsoft.com/fwlink/?linkid=615771). L’agent d’intégrité pour la synchronisation sera ajouté dans le cadre de l’installation d’Azure AD Connect (version 1.0.9125.0 ou ultérieure).
* Prise en main d’Azure AD Connect Health pour AD DS
    * [Téléchargez l’agent Azure AD Connect Health pour AD DS](https://go.microsoft.com/fwlink/?LinkID=820540).
    * [Consultez les instructions d’installation](how-to-connect-health-agent-install.md#install-the-agent-for-azure-ad-ds).


## <a name="azure-ad-connect-health-portal"></a>Portail Azure AD Connect Health
Ce portail vous permet d’afficher des alertes, de surveiller les performances et d’analyser les utilisations. L’URL https://aka.ms/aadconnecthealth vous permet d’accéder au panneau principal d’Azure AD Connect Health. Considérez les panneaux comme des fenêtres. Dans le panneau principal, vous pouvez voir **Démarrage rapide**, les services offerts dans Azure AD Connect Health et d’autres options de configuration. Consultez la capture d’écran suivante et la brève explication qui l’accompagne. Après le déploiement des agents, le service de contrôle d’intégrité s’identifie automatiquement pour les services qu’Azure AD Connect Health analyse.

> [!NOTE]
> Pour plus d’informations sur les licences, consultez la [FAQ Azure AD Connect Health](reference-connect-health-faq.md) ou la [page de tarification d’Azure AD](https://aka.ms/aadpricing).
    
![portail Azure AD Connect Health](./media/whatis-hybrid-identity-health/portalsidebar.png)

* **Démarrage rapide** : quand vous sélectionnez cette option, le panneau **Démarrage rapide** s’ouvre. Vous pouvez télécharger l’agent Azure AD Connect Health en sélectionnant **Obtenir les outils**. Vous pouvez également accéder à la documentation et fournir des commentaires.
* **Azure Active Directory Connect (synchronisation)**  : cette option affiche les serveurs Azure AD Connect actuellement supervisés par Azure AD Connect Health. L’entrée **Erreurs de synchronisation** affiche les erreurs de synchronisation de base de votre premier service de synchronisation intégré par catégories. Quand vous sélectionnez l’entrée **Services de synchronisation**, le panneau qui s’ouvre affiche des informations sur vos serveurs Azure AD Connect. Pour en savoir plus sur les fonctionnalités, consultez [Utilisation d’Azure AD Connect Health pour la synchronisation](how-to-connect-health-sync.md).
* **Active Directory Federation Services** : cette option affiche tous les services AD FS supervisés par Azure AD Connect Health. Lorsque vous sélectionnez une instance, le panneau qui s’ouvre fournit des informations sur cette instance de service. Il comporte une vue d’ensemble, les propriétés, les alertes, les données de surveillance et une analyse de l’utilisation. Pour en savoir plus sur les fonctionnalités, consultez [Utilisation d’Azure AD Connect Health avec AD FS](how-to-connect-health-adfs.md).
* **Active Directory Domain Services** : cette option affiche toutes les forêts AD DS supervisées par Azure AD Connect Health. Lorsque vous sélectionnez une forêt, le panneau qui s’ouvre fournit des informations sur cette forêt. Ces informations incluent une vue d’ensemble des informations essentielles, le tableau de bord des contrôleurs de domaine, le tableau de bord d’état de réplication, des alertes et la surveillance. Pour en savoir plus sur les fonctionnalités, consultez [Utilisation d’Azure AD Connect Health avec AD DS](how-to-connect-health-adds.md).
* **Configurer** : cette section inclut des options permettant d’activer ou de désactiver les éléments suivants :

   - La **mise à jour automatique** de l’agent Azure AD Connect Health vers la dernière version : l’agent Azure AD Connect Health est automatiquement mis à jour dès que de nouvelles versions sont disponibles. Cette option est activée par défaut.
   - **Accès aux données** à partir de l’intégrité de l’annuaire Azure AD par Microsoft uniquement à des fins de dépannage : si cette option est activée, Microsoft peut accéder aux mêmes données que l’utilisateur. Ces informations peuvent être utiles pour la résolution des problèmes et pour fournir l’assistance nécessaire. Cette option est désactivée par défaut
* **Contrôle d’accès en fonction du rôle (IAM)** est la section permettant de gérer l’accès aux données Connect Health en fonction du rôle. 

## <a name="next-steps"></a>Étapes suivantes

- [Matériel et conditions préalables](how-to-connect-install-prerequisites.md) 
- [Paramètres Express](how-to-connect-install-express.md)
- [Paramètres personnalisés](how-to-connect-install-custom.md)
- [Synchronisation de hachage de mot de passe](how-to-connect-password-hash-synchronization.md)|
- [Authentification directe](how-to-connect-pta.md)
- [Fédération avec Azure AD Connect](how-to-connect-fed-whatis.md)
- [Installation des agents Azure AD Connect Health](how-to-connect-health-agent-install.md) 
- [Synchronisation d’Azure AD Connect](how-to-connect-sync-whatis.md)