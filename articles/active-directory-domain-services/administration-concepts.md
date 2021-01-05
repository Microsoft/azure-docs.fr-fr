---
title: Concepts de gestion pour Azure AD Domain Services | Microsoft Docs
description: En savoir plus sur l’administration d’un domaine managé Azure Active Directory Domain Services et le comportement des comptes d’utilisateur et des mots de passe
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 06/05/2020
ms.author: justinha
ms.openlocfilehash: 76af477af8290634d2658c58574b8f96cff7b1fb
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620339"
---
# <a name="management-concepts-for-user-accounts-passwords-and-administration-in-azure-active-directory-domain-services"></a>Concepts de gestion pour les comptes d’utilisateur, les mots de passe et l’administration dans Azure Active Directory Domain Services

Lorsque vous créez et exécutez un domaine managé Azure Active Directory Domain Services (AD DS), il existe des différences de comportement par rapport à un environnement AD DS local traditionnel. Vous utilisez les mêmes outils d’administration dans Azure AD DS en tant que domaine automanagé, mais vous ne pouvez pas accéder directement aux contrôleurs de domaine (DC). Il existe également des différences de comportement pour les stratégies de mot de passe et les hachages de mot de passe en fonction de la source de la création du compte d’utilisateur.

Cet article conceptuel explique en détail comment administrer un domaine managé et le comportement différent des comptes d’utilisateur en fonction de la façon dont ils sont créés.

## <a name="domain-management"></a>Gestion de domaine

Un domaine managé est un espace de noms DNS associé à un annuaire. Dans un domaine managé, les contrôleurs de domaine (DC) qui contiennent toutes les ressources telles que les utilisateurs et les groupes, les informations d’identification et les stratégies font partie du service managé. Pour la redondance, deux contrôleurs de domaine sont créés dans le cadre d’un domaine managé. Vous ne pouvez pas vous connecter à ces contrôleurs de domaine pour effectuer des tâches de gestion. Au lieu de cela, vous créez une machine virtuelle de gestion qui est jointe au domaine managé, puis vous installez vos outils de gestion AD DS standard. Vous pouvez utiliser les composants logiciels enfichables du Centre d’administration Active Directory ou de MMC (Microsoft Management Console) comme les objets DNS ou de stratégie de groupe.

## <a name="user-account-creation"></a>Création d’un compte d'utilisateur

Les comptes d’utilisateurs peuvent être créés dans un domaine managé de plusieurs façons. La plupart des comptes d’utilisateurs sont synchronisés à partir d’Azure AD, qui peuvent également inclure un compte d’utilisateur synchronisé à partir d’un environnement AD DS local. Vous pouvez également créer manuellement des comptes directement dans un domaine managé. Certaines caractéristiques, telles que la synchronisation de mot de passe initiale ou la stratégie de mot de passe, se comportent différemment selon le mode et l’emplacement de création des comptes d’utilisateur.

* Le compte d’utilisateur peut être synchronisé à partir d’Azure AD. Cela comprend les comptes d’utilisateurs uniquement dans le cloud créés directement dans Azure AD, et les comptes d’utilisateur hybrides synchronisés à partir d’un environnement AD DS local à l’aide d’Azure AD Connect.
    * La majorité des comptes d’utilisateur dans un domaine managé sont créés par le biais du processus de synchronisation à partir d’Azure AD.
* Le compte d’utilisateur peut être créé manuellement dans un domaine managé et n’existe pas dans Azure AD.
    * Si vous avez besoin de créer des comptes de service pour des applications qui s’exécutent uniquement dans un domaine managé, vous pouvez les créer manuellement dans le domaine managé. La synchronisation étant unidirectionnelle à partir d’Azure AD, les comptes d’utilisateurs créés dans un domaine managé ne sont pas resynchronisés avec Azure AD.

## <a name="password-policy"></a>Stratégie de mot de passe

Azure AD DS comprend une stratégie de mot de passe par défaut qui définit des paramètres pour les éléments tels que le verrouillage de compte, l’âge maximum du mot de passe et la complexité du mot de passe. Les paramètres tels que la stratégie de verrouillage de compte s’appliquent à tous les utilisateurs dans un domaine managé quelle que soit la façon dont l’utilisateur a été créé comme indiqué dans la section précédente. Certains paramètres, comme la longueur minimale et la complexité du mot de passe, s’appliquent uniquement aux utilisateurs créés directement dans un domaine managé.

Vous pouvez créer vos propres stratégies de mot de passe personnalisées pour remplacer la stratégie par défaut dans un domaine managé. Ces stratégies personnalisées peuvent ensuite être appliquées à des groupes d’utilisateurs spécifiques en fonction des besoins.

Pour plus d’informations sur les différences dans la façon dont les stratégies de mot de passe sont appliquées en fonction de la source de la création de l’utilisateur, consultez [Stratégies de mot de passe et de verrouillage de compte sur les domaines managés][password-policy].

## <a name="password-hashes"></a>Hachages de mot de passe

Pour authentifier les utilisateurs sur le domaine managé, Azure AD DS nécessite les hachages de mot de passe dans un format adapté à l’authentification NTLM (NT LAN Manager) et Kerberos. Azure AD ne génère pas et ne stocke pas les hachages de mot de passe au format nécessaire pour l’authentification NTLM ou Kerberos tant que vous n’activez pas Azure AD DS pour votre locataire. Pour des raisons de sécurité, Azure AD ne stocke pas non plus d’informations d’identification de mot de passe sous forme de texte en clair. Par conséquent, Azure AD ne peut pas générer automatiquement ces hachages de mot de passe NTLM ou Kerberos en fonction des informations d’identification existantes des utilisateurs.

Pour les comptes d’utilisateurs cloud uniquement, les utilisateurs doivent changer leur mot de passe avant de pouvoir utiliser le domaine managé. Ce processus de changement du mot de passe entraîne la génération et le stockage dans Azure AD des hachages de mot de passe pour l’authentification Kerberos et NTLM. Le compte n’est pas synchronisé entre Azure AD et Azure AD DS tant que le mot de passe n’a pas été modifié.

Pour les utilisateurs synchronisés à partir d’un environnement AD DS local à l’aide d’Azure AD Connect, [activez la synchronisation des hachages de mot de passe ][hybrid-phs].

> [!IMPORTANT]
> Azure AD Connect synchronise uniquement les hachages de mot de passe hérités lorsque vous activez Azure AD DS pour votre client Azure AD. Les hachages de mot de passe hérités ne sont pas utilisés si vous utilisez uniquement Azure AD Connect pour synchroniser un environnement AD DS local avec Azure AD.
>
> Si vos applications héritées n’utilisent pas l’authentification NTLM ou les liaisons simples LDAP, nous vous recommandons de désactiver la synchronisation de hachage de mot de passe NTLM pour Azure AD DS. Pour plus d’informations, consultez [Désactiver les suites de chiffrement faible et la synchronisation des hachages des informations d’identification NTLM][secure-domain].

Une fois configurés de façon appropriée, les hachages de mot de passe utilisables sont stockés dans le domaine managé. Si vous supprimez le domaine managé, tout hachage de mot de passe stocké à ce stade est également supprimé. Les informations d’identification synchronisées dans Azure AD ne peuvent pas être réutilisées si vous créez par la suite un domaine managé : vous devez reconfigurer la synchronisation de hachage de mot de passe pour stocker à nouveau les hachages de mot de passe. Les machines virtuelles ou les utilisateurs auparavant joints à un domaine ne pourront pas s’authentifier immédiatement : Azure AD doit générer et stocker les hachages de mot de passe dans le nouveau domaine managé. Pour plus d’informations, consultez [Processus de synchronisation du hachage de mot de passe pour Azure AD DS et Azure AD Connect][azure-ad-password-sync].

> [!IMPORTANT]
> Azure AD Connect doit uniquement être installé et configuré pour la synchronisation avec des environnements AD DS locaux. L’installation d’Azure AD Connect n’est pas prise en charge dans un domaine managé pour resynchroniser des objets sur Azure AD.

## <a name="forests-and-trusts"></a>Forêts et approbations

Une *forêt* est une construction logique utilisée par Active Directory Domain Services (AD DS) pour regrouper un ou plusieurs *domaines*. Les domaines stockent alors les objets pour un utilisateur ou des groupes et fournissent des services d’authentification.

Dans Azure AD DS, la forêt ne contient qu’un seul domaine. Les forêts AD DS locales contiennent souvent de nombreux domaines. Dans les grandes organisations, en particulier après des fusions et acquisitions, vous pouvez vous retrouver avec plusieurs forêts locales qui contiennent chacune plusieurs domaines.

Par défaut, un domaine managé est créé en tant que forêt d’*utilisateurs*. Ce type de forêt synchronise tous les objets d’Azure AD, notamment les comptes d’utilisateur créés dans un environnement AD DS local. Les comptes d’utilisateur peuvent directement s’authentifier auprès du domaine managé, par exemple pour se connecter à une machine virtuelle jointe à un domaine. Une forêt d’utilisateurs fonctionne lorsque les hachages de mot de passe peuvent être synchronisés et que les utilisateurs n’utilisent pas de méthode de connexion exclusive, comme l’authentification par carte à puce.

Dans une forêt Azure AD DS de *ressources*, les utilisateurs s’authentifient sur une forêt à *approbation* unique à partir de leur AD DS local. Avec cette approche, les objets utilisateur et les hachages de mot de passe ne sont pas synchronisés avec Azure AD DS. Les objets utilisateur et les informations d’identification existent uniquement dans l’instance AD DS locale. Cette approche permet aux entreprises d’héberger des ressources et des plateformes d’application dans Azure qui dépendent de l’authentification classique, par exemple LDAPS, Kerberos ou NTLM, en éliminant les problèmes et craintes en matière d’authentification.

Pour plus d’informations sur les types de forêts dans Azure AD DS, consultez [Qu’est-ce que des forêts de ressources ?][concepts-forest] et [Comment faire fonctionner des approbations de forêts dans Azure AD DS ?][concepts-trust]

## <a name="azure-ad-ds-skus"></a>Références SKU Azure AD DS

Dans Azure AD DS, les performances et fonctionnalités disponibles sont basées sur la référence SKU. Vous sélectionnez une référence SKU lorsque vous créez le domaine managé et pouvez changer de références SKU au fil des besoins de votre entreprise, une fois le domaine managé déployé. Le tableau suivant présente les références SKU disponibles ainsi que leurs différences :

| Nom de la référence (SKU)   | Nombre maximal d'objets | Fréquence de sauvegarde | Nombre maximal d’approbations de forêts sortantes |
|------------|----------------------|------------------|----|
| Standard   | Illimité            | Tous les 7 jours     | 0  |
| Entreprise | Illimité            | Tous les 3 jours     | 5  |
| Premium    | Illimité            | Quotidien            | 10 |

Avant ces références SKU Azure AD DS, un modèle de facturation basé sur le nombre d’objets (comptes d’utilisateurs et d’ordinateurs) dans le domaine managé était utilisé. Il n’existe plus de tarification variable en fonction du nombre d’objets du domaine managé.

Pour plus d’informations, consultez la [page de tarification Azure AD DS][pricing].

### <a name="managed-domain-performance"></a>Performances du domaine managé

Les performances du domaine varient selon la manière dont l’authentification est implémentée pour une application. Des ressources de calcul supplémentaires peuvent contribuer à améliorer le temps de réponse aux requêtes et réduire le délai des opérations de synchronisation. Plus le niveau de référence SKU augmente, plus les ressources de calcul disponibles pour le domaine managé augmentent aussi. Surveillez les performances de vos applications et planifiez les ressources requises.

Si les besoins de votre entreprise ou de vos applications évoluent et qu'il vous faut une puissance de calcul supplémentaire pour votre domaine managé, vous pouvez opter pour une autre référence SKU.

### <a name="backup-frequency"></a>Fréquence de sauvegarde

La fréquence de sauvegarde détermine la fréquence de prise d'un instantané du domaine managé. Les sauvegardes relèvent d'un processus automatisé managé par la plateforme Azure. En cas de problème lié à votre domaine managé, le support Azure peut vous aider à procéder à une restauration à partir d'une sauvegarde. La synchronisation étant unidirectionnelle *depuis* Azure AD, les problèmes survenant dans un domaine managé n’ont pas d’impact sur Azure AD ou les environnements et fonctionnalités AD DS locaux.

Plus le niveau de référence SKU augmente, plus la fréquence de ces instantanés de sauvegarde augmente aussi. Examinez les besoins de votre entreprise et l’objectif de point de récupération afin de déterminer la fréquence de sauvegarde requise pour votre domaine managé. Si les besoins de votre entreprise ou de vos applications évoluent et que vous avez besoin de sauvegardes plus fréquentes, vous pouvez opter pour une autre référence SKU.

### <a name="outbound-forest-trusts"></a>Approbations de forêt sortantes

La section précédente a décrit en détail les approbations de forêts sortantes unidirectionnelles d’un domaine managé vers un environnement AD DS local. La référence SKU détermine nombre maximal d’approbations de forêt que vous pouvez créer pour un domaine managé. Examinez les exigences de votre entreprise et de vos applications pour déterminer le nombre d’approbations dont vous avez réellement besoin, puis choisissez la référence SKU Azure AD DS appropriée. Là encore, si les exigences de votre entreprise évoluent et qu'il vous faut créer des approbations de forêts supplémentaires, vous pouvez opter pour une autre référence SKU.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer, [créons un domaine managé Azure AD DS][create-instance].

<!-- INTERNAL LINKS -->
[password-policy]: password-policy.md
[hybrid-phs]: tutorial-configure-password-hash-sync.md#enable-synchronization-of-password-hashes
[secure-domain]: secure-your-domain.md
[azure-ad-password-sync]: ../active-directory/hybrid/how-to-connect-password-hash-synchronization.md#password-hash-sync-process-for-azure-ad-domain-services
[create-instance]: tutorial-create-instance.md
[tutorial-create-instance-advanced]: tutorial-create-instance-advanced.md
[concepts-forest]: concepts-resource-forest.md
[concepts-trust]: concepts-forest-trust.md

<!-- EXTERNAL LINKS -->
[pricing]: https://azure.microsoft.com/pricing/details/active-directory-ds/
