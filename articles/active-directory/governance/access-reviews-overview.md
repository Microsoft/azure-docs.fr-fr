---
title: Que sont les révisions d’accès ? - Azure Active Directory | Microsoft Docs
description: Avec les révisions d’accès Azure Active Directory, vous pouvez contrôler que l’appartenance à des groupes et l’accès aux applications répondent aux initiatives de gouvernance, de gestion des risques et de conformité de votre organisation.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.subservice: compliance
ms.date: 10/29/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.custom: contperf-fy21q1
ms.openlocfilehash: 08399c1b2d930abfdd5832cb102a789844931d61
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027694"
---
# <a name="what-are-azure-ad-access-reviews"></a>Présentation des révisions d’accès Azure AD

Les révisions d’accès Azure Active Directory (Azure AD) permettent aux organisations de gérer efficacement les appartenances à des groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les personnes appropriées continuent de bénéficier d’un accès.

Voici une vidéo qui donne une vue d’ensemble rapide des révisions d’accès :

>[!VIDEO https://www.youtube.com/embed/kDRjQQ22Wkk]

## <a name="why-are-access-reviews-important"></a>Pourquoi les révisions d’accès sont-elles importantes ?

Azure AD vous permet de collaborer avec des utilisateurs à l’intérieur de votre organisation ainsi qu’avec des utilisateurs externes. Les utilisateurs peuvent se joindre à des groupes, inviter des personnes, se connecter à des applications cloud et travailler à distance à partir de leurs appareils personnels ou professionnels. L’intérêt d’utiliser le libre-service a conduit à la nécessité d’avoir de meilleures fonctionnalités de gestion des accès.

- Quand de nouveaux employés arrivent, comment vérifier qu’ils disposent des accès dont ils ont besoin pour être productifs ?
- Quand des personnes changent d’équipe ou quittent l’entreprise, comment vérifier que leur ancien accès est bien supprimé ?
- Des droits d’accès excessifs peuvent entraîner des compromissions.
- Des droits d’accès excessifs peuvent également amener à auditer les résultats, car ils indiquent un manque de contrôle des accès.
- Vous devez inviter de façon proactive les propriétaires des ressources à vérifier régulièrement les utilisateurs qui ont accès à leurs ressources.

## <a name="when-should-you-use-access-reviews"></a>Quand utiliser les révisions d’accès ?

- **Trop d’utilisateurs dans des rôles privilégiés :** Il est judicieux de vérifier combien d’utilisateurs ont un accès d’administration, combien d’entre eux ont le rôle d’administrateur général, et s’il existe des invités ou des partenaires qui n’ont pas été supprimés après qu’une tâche d’administration leur a été attribuée. Vous pouvez recertifier les utilisateurs ayant fait l’objet d’une attribution de rôle dans les [rôles Azure AD](../privileged-identity-management/pim-how-to-perform-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json), comme les administrateurs généraux, ou [les rôles de ressources Azure](../privileged-identity-management/pim-resource-roles-perform-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json), comme les administrateur de l’accès utilisateur dans l’expérience [Azure AD Privileged Identity Management (PIM)](../privileged-identity-management/pim-configure.md).
- **Quand l’automatisation n’est pas possible :** Vous pouvez créer des règles pour l’appartenance dynamique sur les groupes de sécurité ou les groupes Microsoft 365, mais que se passe-t-il si les données des ressources humaines ne se trouvent pas dans Azure AD, ou si des utilisateurs ont toujours besoin d’un accès après avoir quitté le groupe pour entraîner les personnes qui les remplacent ? Vous pouvez alors créer une révision sur ce groupe pour que ceux qui ont encore besoin d’un accès puissent encore en bénéficier.
- **Quand un groupe est utilisé pour un nouvel objectif :** Si vous avez un groupe qui va être synchronisé avec Azure AD, ou si vous prévoyez d’activer l’application Salesforce pour tout le monde dans le groupe de l’équipe commerciale, il est utile de demander au propriétaire du groupe de passer en revue l’appartenance au groupe avant d’utiliser le groupe dans un autre contenu à risques.
- **Accès aux données critiques de l’entreprise :** pour certaines ressources, il peut être nécessaire à des fins d’audit de demander aux personnes en dehors du département informatique de se déconnecter régulièrement et de justifier la raison pour laquelle ils ont besoin d’un accès.
- **Pour gérer la liste d’exceptions d’une stratégie :** Dans l’idéal, tous les utilisateurs doivent respecter les stratégies d’accès pour sécuriser l’accès aux ressources de votre organisation. Toutefois, certains scénarios métiers nécessitent que vous fassiez des exceptions. En tant qu’administrateur informatique, vous pouvez gérer cette tâche, éviter d’oublier les exceptions à la stratégie et fournir aux auditeurs la preuve que ces exceptions sont révisées régulièrement.
- **Demander aux propriétaires de groupe de confirmer qu’ils ont encore besoin d’invités dans leurs groupes :** L’accès des employés peut être automatisé avec certains systèmes de gestion des identités et des accès (IAM) locaux, mais pas l’accès des invités. Si un groupe donne à des invités l’accès à du contenu sensible de l’entreprise, il est de la responsabilité du propriétaire du groupe de confirmer que les invités ont encore un besoin métier légitime de cet accès.
- **Procédez régulièrement à des révisions d’accès :** Vous pouvez configurer des révisions d’accès des utilisateurs récurrentes à des fréquences définies, hebdomadaires, mensuelles, trimestrielles ou annuelles, les réviseurs étant alors informés au début de chaque révision. Les réviseurs peuvent approuver ou refuser l’accès avec une interface conviviale et avec l’aide de recommandations intelligentes.

>[!NOTE]
>Si vous êtes prêts à essayer la révision d’accès, consultez [Créer une révision d’accès des groupes ou applications](create-access-review.md)

## <a name="where-do-you-create-reviews"></a>Où créer des révisions ?

Selon ce que vous voulez réviser, vous créez votre révision d’accès dans Révisions d’accès Azure AD, dans Applications d’entreprise Azure AD (en préversion) ou dans Azure AD PIM.

| Droits d’accès des utilisateurs | Les réviseurs peuvent être | Révision créée dans | Expérience des réviseurs |
| --- | --- | --- | --- |
| Membres des groupes de sécurité</br>Membres du groupe Office | Réviseurs spécifiés</br>Propriétaires de groupe</br>Révision indépendante | Révisions d’accès Azure AD</br>Groupes Azure AD | Panneau d’accès |
| Affecté à une application connectée | Réviseurs spécifiés</br>Révision indépendante | Révisions d’accès Azure AD</br>Applications d’entreprise Azure AD (en préversion) | Panneau d’accès |
| Rôle Azure AD | Réviseurs spécifiés</br>Révision indépendante | [Azure AD PIM](../privileged-identity-management/pim-how-to-start-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json) | Portail Azure |
| Rôle de ressource Azure | Réviseurs spécifiés</br>Révision indépendante | [Azure AD PIM](../privileged-identity-management/pim-resource-roles-start-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json) | Portail Azure |

## <a name="license-requirements"></a>Conditions de licence :

[!INCLUDE [Azure AD Premium P2 license](../../../includes/active-directory-p2-license.md)]

### <a name="how-many-licenses-must-you-have"></a>De combien de licences avez-vous besoin ?

Votre répertoire a besoin d’au moins autant de licences Azure AD Premium P2 que d’employés effectuant les tâches suivantes :

-   Utilisateurs membres auxquels le rôle de réviseur a été attribué
-   Utilisateurs membres qui effectuent une révision indépendante
-   Utilisateurs membres en tant que propriétaires de groupes qui effectuent une révision d’accès
-   Utilisateurs membres en tant que propriétaires d’applications qui effectuent une révision d’accès

Pour les utilisateurs invités, les besoins en licences dépendent du modèle de licence que vous utilisez. Toutefois, les activités des utilisateurs invités ci-dessous sont considérées comme une utilisation d’Azure AD Premium P2 :

-   Utilisateurs invités auxquels le rôle de réviseur a été attribué
-   Utilisateurs invités qui effectuent une révision indépendante
-   Utilisateurs invités en tant que propriétaires de groupes qui effectuent une révision d’accès
-   Utilisateurs invités en tant que propriétaires d’applications qui effectuent une révision d’accès


Les licences Azure AD P2 ne sont **pas** nécessaires pour les utilisateurs disposant des rôles Administrateur général ou Administrateur d’utilisateurs qui configurent des révisions d’accès et des paramètres, ou qui appliquent les décisions prises à partir des révisions.

L’accès des utilisateurs invités Azure AD est basé sur un modèle de facturation par utilisateurs actifs mensuels (MAU), qui remplace le modèle de facturation selon le rapport 1:5. Pour plus d’informations, consultez [Tarifs d’Azure AD External Identities](../external-identities/external-identities-pricing.md).

Pour plus d’informations sur les licences, consultez [Attribuer ou supprimer des licences à l’aide du portail Azure Active Directory](../fundamentals/license-users-groups.md).

### <a name="example-license-scenarios"></a>Exemples de scénarios de licence

Voici quelques exemples de scénarios de licence pour vous permettre de déterminer le nombre de licences dont vous devez disposer.

| Scénario | Calcul | Nombre de licences |
| --- | --- | --- |
| Un administrateur crée une révision d’accès du groupe A comportant 75 utilisateurs et 1 propriétaire du groupe, et affecte le propriétaire du groupe en tant que réviseur. | 1 licence pour le propriétaire du groupe en tant que réviseur | 1 |
| Un administrateur crée une révision d’accès du groupe B comportant 500 utilisateurs et 3 propriétaires de groupe, et affecte les 3 propriétaires du groupe en tant que réviseurs. | 3 licences pour chaque propriétaire du groupe en tant que réviseurs | 3 |
| Un administrateur crée une révision d’accès du groupe B comportant 500 utilisateurs. Effectue une révision indépendante. | 500 licences pour chaque utilisateur en tant qu’auto-réviseurs | 500 |
| Un administrateur crée une révision d’accès du groupe C comportant 50 utilisateurs membres et 25 utilisateurs invités. Effectue une révision indépendante. | 50 licences pour chaque utilisateur en tant qu’auto-réviseurs.* | 50 |
| Un administrateur crée une révision d’accès du groupe D comportant 6 utilisateurs membres et 108 utilisateurs invités. Effectue une révision indépendante. | 6 licences pour chaque utilisateur en tant qu’auto-réviseurs. Les utilisateurs invités sont facturés par utilisateur actif mensuel (MAU). Aucune licence supplémentaire n’est requise. *  | - |

\* Les tarifs des identités externes Azure AD (utilisateur invité) sont basés sur les utilisateurs actifs mensuels (MAU), c’est-à-dire le nombre d’utilisateurs uniques ayant une activité d’authentification au cours d’un mois civil. Ce modèle remplace le modèle de facturation selon le rapport 1:5, qui autorise jusqu’à cinq utilisateurs invités pour chaque licence de Azure AD Premium disponible dans votre locataire. Lorsque votre locataire est lié à un abonnement et que vous utilisez des fonctionnalités d’identités externes pour collaborer avec des utilisateurs invités, vous êtes automatiquement facturé à l’aide du modèle basé MAU. Pour plus d’informations, consultez Modèle de facturation pour les identités externes Azure AD.

## <a name="next-steps"></a>Étapes suivantes

- [Créer une révision d’accès de groupes ou d’applications](create-access-review.md)
- [Create an access review of users in an Azure AD administrative role](../privileged-identity-management/pim-how-to-start-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json) (Créer une révision d’accès pour les utilisateurs dans un rôle administratif d’Azure AD)
- [Réviser l’accès à des groupes ou à des applications](perform-access-review.md)
- [Effectuer une révision d’accès de groupes ou révisions d’accès des applications dans Azure AD](complete-access-review.md)
