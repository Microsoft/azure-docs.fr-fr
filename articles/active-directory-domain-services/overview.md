---
title: Présentation des services de domaine Azure Active Directory | Microsoft Docs
description: Dans cette vue d’ensemble, découvrez ce qui est fourni par Azure Active Directory Domain Services et comment l’utiliser dans votre organisation pour fournir des services d’identité aux applications et services dans le cloud.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: overview
ms.date: 12/03/2020
ms.author: justinha
ms.custom: contperf-fy21q1
ms.openlocfilehash: 99a207810f51949b0da0e78b1cb0e0a7f8bc95e3
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027337"
---
# <a name="what-is-azure-active-directory-domain-services"></a>Présentation d’Azure Active Directory Domain Services

Azure Active Directory Domain Services (AD DS) fournit des services de domaine managé, comme la jonction de domaine, la stratégie de groupe, le protocole LDAP et l’authentification Kerberos/NTLM. Vous utilisez ces services de domaine sans avoir à déployer, gérer et apporter des correctifs aux contrôleurs de domaine dans le cloud.

Un domaine managé Azure AD DS vous permet d’exécuter des applications héritées dans le cloud qui ne peuvent pas utiliser les méthodes d'authentification modernes ou pour lesquelles vous ne voulez pas que les recherches de répertoire retournent systématiquement à un environnement AD DS local. Vous pouvez effectuer un lift-and-shift de ces applications héritées de votre environnement local à un domaine managé sans devoir gérer l’environnement AD DS dans le cloud.

Azure AD DS s’intègre à votre locataire Azure AD existant. Cette intégration permet aux utilisateurs de se connecter au service et aux applications connectés au domaine managé à l’aide de leurs informations d’identification existantes. Vous pouvez également utiliser des comptes d’utilisateurs et des groupes existants pour sécuriser l’accès aux ressources. Ces fonctionnalités fournissent une migration lift-and-shift plus simple des ressources locales vers Azure.

> [!div class="nextstepaction"]
> [Pour commencer, créez un domaine managé Azure AD DS à l’aide du portail Azure][tutorial-create].

## <a name="how-does-azure-ad-ds-work"></a>Fonctionnement d’Azure AD DS

Quand vous créez un domaine managé Azure AD DS, vous définissez un espace de noms unique. Cet espace de noms est le nom de domaine, par exemple *aaddscontoso.com*. Deux contrôleurs de domaine Windows Server sont ensuite déployés dans votre région Azure sélectionnée. Ce déploiement de contrôleurs de domaine est appelé jeu de réplicas.

Vous n’avez pas besoin de gérer, configurer ou mettre à jour ces contrôleurs de domaine. La plateforme Azure gère les contrôleurs de domaine comme faisant partie du domaine managé, y compris les sauvegardes et le chiffrement au repos utilisant Azure Disk Encryption.

Un domaine managé est configuré pour effectuer une synchronisation unidirectionnelle à partir d’Azure AD, afin de fournir l’accès à un ensemble centralisé d’utilisateurs, de groupes et d’informations d’identification. Vous pouvez créer des ressources directement dans le domaine managé, mais elles ne sont pas resynchronisées sur Azure AD. Dans Azure, les applications, les services et les machines virtuelles qui se connectent au domaine managé peuvent ensuite utiliser des fonctionnalités AD DS courantes telles que la jonction de domaine, la stratégie de groupe, LDAP et l’authentification Kerberos/NTLM.

Dans un environnement hybride avec un environnement AD DS local, [Azure AD Connect][azure-ad-connect] synchronise les informations d’identité avec Azure AD, qui sont ensuite synchronisées avec le domaine managé.

![Synchronisation dans Azure AD Domain Services avec Azure AD et AD DS en local à l’aide d’AD Connect](./media/active-directory-domain-services-design-guide/sync-topology.png)

Azure AD DS réplique les informations d’identité à partir d’Azure AD et fonctionne donc avec des clients Azure AD qui sont uniquement cloud ou synchronisés avec un environnement local (AD DS). Le même ensemble de fonctionnalités Azure AD DS existe pour les deux environnements.

* Si vous disposez d’un environnement AD DS local existant, vous pouvez synchroniser les informations des comptes d’utilisateurs pour fournir une identité cohérente aux utilisateurs. Pour en savoir plus, consultez [Synchronisation des objets et des informations d’identification dans un domaine managé][synchronization].
* Pour les environnements cloud uniquement, vous n’avez pas besoin d’un environnement AD DS local traditionnel pour utiliser les services d’identité centralisés d’Azure AD DS.

Vous pouvez étendre un domaine managé pour avoir plusieurs jeux de réplicas par locataire Azure AD. Les jeux de réplicas peuvent être ajoutés à n’importe quel réseau virtuel appairé dans toute région Azure prenant en charge Azure AD DS. D’autres jeux de réplicas dans des régions Azure différentes assurent la récupération d’urgence géographique pour les applications héritées si une région Azure est mise hors connexion. Les jeux de réplicas sont actuellement en préversion. Pour plus d’informations, consultez [Concepts et caractéristiques des jeux de réplicas pour les domaines managés][concepts-replica-sets].

La vidéo suivante fournit une vue d’ensemble de la façon dont Azure AD DS s’intègre à vos applications et charges de travail pour fournir des services d’identité dans le cloud :

<br />

>[!VIDEO https://www.youtube.com/embed/T1Nd9APNceQ]

Pour voir des scénarios de déploiement d’Azure AD DS en action, vous pouvez explorer les exemples suivants :

* [Azure AD DS pour les organisations hybrides](scenarios.md#azure-ad-ds-for-hybrid-organizations)
* [Azure AD DS pour les organisations cloud uniquement](scenarios.md#azure-ad-ds-for-cloud-only-organizations)

## <a name="azure-ad-ds-features-and-benefits"></a>Fonctionnalités et avantages d’Azure AD DS

Pour fournir des services d’identité aux applications et aux machines virtuelles dans le cloud, Azure AD DS est entièrement compatible avec un environnement AD DS traditionnel pour des opérations telles que la jonction de domaine, le protocole LDAP sécurisé (LDAPS), la stratégie de groupe, la gestion de DNS ainsi que la prise en charge des lectures et liaisons LDAP. La prise en charge de l’écriture LDAP est disponible pour les objets créés dans le domaine managé, mais pas pour les ressources synchronisées à partir d’Azure AD.

Pour en savoir plus sur les options d’identité, [ comparez Azure AD DS avec Azure AD, AD DS sur des machines virtuelles Azure et AD DS en local][compare].

Les fonctionnalités suivantes d’Azure AD DS simplifient les opérations de déploiement et de gestion :

* **Expérience de déploiement simplifiée :** Azure AD DS est activé pour votre locataire Azure AD à l’aide d’un seul Assistant dans le Portail Azure.
* **Intégration à Azure AD :** Les comptes d’utilisateur, les appartenances aux groupes et les informations d’identification sont automatiquement disponibles à partir de votre locataire Azure AD. Les nouveaux utilisateurs, groupes ou modifications apportées aux attributs à partir de votre locataire Azure AD ou de votre environnement AD DS local sont automatiquement synchronisés avec Azure AD DS.
    * Les comptes d’annuaires externes liés à votre annuaire Azure AD ne sont pas disponibles dans Azure AD DS. Les informations d’identification ne sont pas disponibles pour ces annuaires externes et ne peuvent donc pas être synchronisées dans un domaine managé.
* **Utilisation de vos informations d’identification/mots de passe d’entreprise :** Les mots de passe des utilisateurs dans Azure AD DS sont identiques à ceux dans votre locataire Azure AD. Les utilisateurs peuvent utiliser leurs informations d’identification d’entreprise pour joindre des machines à des domaines, se connecter de manière interactive ou via le bureau à distance et s’authentifier sur le domaine managé.
* **Authentification NTLM et Kerberos :** Grâce à la prise en charge de l’authentification NTLM et Kerberos, vous pouvez déployer des applications qui reposent sur l’authentification intégrée de Windows.
* **Haute disponibilité :** Azure AD DS inclut plusieurs contrôleurs de domaine, qui fournissent une haute disponibilité pour votre domaine managé. Cette haute disponibilité garantit le temps de fonctionnement du service et la résilience aux défaillances.
    * Dans les régions qui prennent en charge les [Zones de disponibilité Azure][availability-zones], ces contrôleurs de domaine sont également répartis entre les zones pour une résilience supplémentaire.
    * Les [jeux de replicas][concepts-replica-sets] permettent également de proposer la récupération d’urgence géographique pour les applications héritées si une région Azure est mise hors connexion.

Voici quelques aspects majeurs d’un domaine managé :

* Le domaine géré est un domaine autonome. et non une extension du domaine local.
    * Si nécessaire, vous pouvez créer des approbations de forêt sortantes unidirectionnelles entre Azure AD DS et un environnement AD DS local. Pour plus d’informations, consultez [Concepts et fonctionnalités de la forêt de ressources pour Azure AD DS][ forest-trusts].
* Votre équipe informatique n’a pas à gérer, à appliquer les correctifs ni à analyser les contrôleurs de domaine pour ce domaine managé.

Pour les environnements hybrides qui exécutent AD DS localement, vous n’avez pas besoin de gérer la réplication Active Directory sur le domaine managé. Les comptes d’utilisateur, les appartenances aux groupes et les informations d’identification de votre annuaire local sont synchronisés avec Azure AD via [Azure AD Connect][azure-ad-connect]. Ces comptes utilisateur, appartenances aux groupes et informations d’identification sont automatiquement disponibles dans le domaine géré.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les différentes entre Azure AD DS et d’autres solutions d’identité et sur le fonctionnement de la synchronisation, consultez les articles suivants :

* [Comparer Azure AD DS avec Azure AD, Active Directory Domain Services sur des machines virtuelles Azure et Active Directory Domain Services en local][compare]
* [Comprendre comment la solution Azure AD Domain Services est synchronisée avec votre répertoire Azure AD][synchronization]
* Pour découvrir comment administrer un domaine managé, consultez [Concepts de gestion pour les comptes d’utilisateur, les mots de passe et l’administration dans Azure AD DS][administration-concepts].

Pour commencer, [créez un domaine managé à l’aide du portail Azure][tutorial-create].

<!-- INTERNAL LINKS -->
[compare]: compare-identity-solutions.md
[synchronization]: synchronization.md
[tutorial-create]: tutorial-create-instance.md
[azure-ad-connect]: ../active-directory/hybrid/whatis-azure-ad-connect.md
[password-hash-sync]: ../active-directory/hybrid/how-to-connect-password-hash-synchronization.md
[availability-zones]: ../availability-zones/az-overview.md
[forest-trusts]: concepts-resource-forest.md
[administration-concepts]: administration-concepts.md
[synchronization]: synchronization.md
[concepts-replica-sets]: concepts-replica-sets.md
