---
title: Gérer l’accès invité avec les révisions d’accès - Azure AD
description: Gérer les utilisateurs invités en tant que membres d’un groupe ou affectés à une application avec les révisions d’accès Azure Active Directory
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: 76aeb63feb006e6962efeb3687363f18acb59904
ms.sourcegitcommit: 6e2d37afd50ec5ee148f98f2325943bafb2f4993
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97746692"
---
# <a name="manage-guest-access-with-azure-ad-access-reviews"></a>Gérer l’accès invité avec les révisions d’accès Azure AD


Avec Azure Active Directory (Azure AD), vous pouvez facilement mettre en place une collaboration au-delà des limites de l’organisation à l’aide de la [fonctionnalité B2B d’Azure AD](../external-identities/what-is-b2b.md). Les utilisateurs invités depuis d’autres locataires peuvent être [invités par les administrateurs](../external-identities/add-users-administrator.md) ou par [d’autres utilisateurs](../external-identities/what-is-b2b.md). Cela s’applique également aux identités de réseaux sociaux tels que des comptes Microsoft.

Vous pouvez également facilement vous assurer que les utilisateurs invités disposent de droits d’accès appropriés. Vous pouvez demander directement aux invités ou bien à un décisionnaire de prendre part à une révision d’accès et de renouveler la certification (ou d’effectuer une « attestation ») pour l’accès invité. Les réviseurs peuvent donner leur avis sur les besoins de chaque utilisateur en matière d’accès continu, en fonction des suggestions d’Azure AD. Lorsqu’une révision d’accès est terminée, vous pouvez effectuer des modifications et supprimer l’accès des invités qui n’en ont plus besoin.

> [!NOTE]
> Ce document se concentre sur la revue de l’accès des utilisateurs invités. Si vous souhaitez réviser l’accès de tous les utilisateurs, et pas uniquement celui des invités, consultez l’article [Manage user access with access reviews](manage-user-access-with-access-reviews.md) (Gérer l’accès utilisateur à l’aide des révisions d’accès). Si vous voulez examiner l’appartenance d’un utilisateur à un rôle d’administrateur tel que le rôle Administrateur général, consultez [Démarrer une révision d’accès dans Azure AD Privileged Identity Management](../privileged-identity-management/pim-how-to-start-security-review.md).

## <a name="prerequisites"></a>Prérequis

- Azure AD Premium P2

Pour plus d’informations, consultez [Conditions de licence](access-reviews-overview.md#license-requirements).

## <a name="create-and-perform-an-access-review-for-guests"></a>Créer et exécuter une révision d’accès pour des invités

Tout d’abord, un des rôles suivants doit vous être attribué :
- Administrateur général
- Administrateur d’utilisateurs
- (Préversion) Propriétaire de groupe de sécurité AAD ou M365 pour le groupe à examiner

Accédez ensuite à la [page Gouvernance des identités](https://portal.azure.com/#blade/Microsoft_AAD_ERM/DashboardBlade/) pour vous assurer que les révisions d’accès sont prêtes pour votre organisation.

Azure AD permet plusieurs scénarios de revue des utilisateurs invités.

Vous pouvez réviser :

 - Un groupe dans Azure AD comprenant un ou plusieurs invités comme membres.
 - Une application connectée à Azure AD ayant un ou plusieurs utilisateurs invités attribués. 

Vous pouvez alors décider de demander à chaque invité de revoir leur propre accès ou de demander à un ou plusieurs utilisateurs de revoir l’accès de chaque invité.

 Ces scénarios sont traités dans les sections suivantes.

### <a name="ask-guests-to-review-their-own-membership-in-a-group"></a>Demander aux invités de revoir leur propre appartenance à un groupe

Vous pouvez utiliser les révisions d’accès pour vous assurer que les utilisateurs qui ont été invités et ajoutés à un groupe ont toujours besoin de leur accès. Vous pouvez facilement demander aux invités de revoir leur propre appartenance à ce groupe.

1. Pour créer une révision d’accès pour le groupe, choisissez d’y inclure uniquement les utilisateurs invités et faites-leur faire la révision eux-mêmes. Pour plus d’informations, consultez [Créer une révision d’accès de groupes ou d’applications](create-access-review.md).

2. Demandez à chaque invité de revoir sa propre appartenance. Par défaut, chaque invité ayant accepté une invitation recevra un message électronique de la part d’Azure AD comprenant un lien vers la révision d’accès. Les instructions d’Azure AD pour les invités concernent la [révision d’accès aux groupes ou aux applications](perform-access-review.md).

3. Une fois que les réviseurs ont fait part de leur avis, arrêtez la révision d’accès et appliquez les modifications. Pour plus d’informations, consultez [Effectuer une révision d’accès de groupes ou d’applications](complete-access-review.md).

4. En plus des utilisateurs indiquant n’avoir plus besoin d’un accès continu, vous pouvez également retirer les utilisateurs n’ayant pas répondu. Il est probable que les utilisateurs qui ne répondent pas ne reçoivent plus de courrier électronique.

5. Si le groupe n’est pas utilisé pour la gestion des accès, vous pouvez également retirer les utilisateurs qui n’ont pas été sélectionnés pour participer à la révision car ils n’ont pas accepté leur invitation. Cela peut être dû à une faute de frappe dans l’adresse e-mail de l’utilisateur invité. Si un groupe est utilisé comme liste de distribution, il est possible que certains utilisateurs invités n’aient pas été sélectionnés pour participer parce qu’ils sont des objets contact.

### <a name="ask-a-sponsor-to-review-a-guests-membership-in-a-group"></a>Demander à un commanditaire de revoir l’appartenance d’un invité à un groupe

Vous pouvez demander à un commanditaire, tel que le propriétaire d’un groupe, d’évaluer si un invité a toujours besoin d’appartenir à un groupe.

1. Pour créer une révision d’accès pour le groupe, choisissez d’y inclure uniquement les utilisateurs invités. Spécifiez ensuite un ou plusieurs réviseurs. Pour plus d’informations, consultez [Créer une révision d’accès de groupes ou d’applications](create-access-review.md).

2. Demandez aux réviseurs de donner leur avis. Par défaut, chacun d’eux reçoit un e-mail provenant d’Azure AD et contenant un lien vers le panneau d’accès, dans lequel ils peuvent [réviser l’accès à des groupes ou à des applications](perform-access-review.md).

3. Une fois que les réviseurs ont fait part de leur avis, arrêtez la révision d’accès et appliquez les modifications. Pour plus d’informations, consultez [Effectuer une révision d’accès de groupes ou d’applications](complete-access-review.md).

### <a name="ask-guests-to-review-their-own-access-to-an-application"></a>Demander aux invités de revoir leur propre accès à une application

Vous pouvez utiliser les révisions d’accès pour vous assurer que les utilisateurs qui ont été invités pour une application spécifique ont toujours besoin de leur accès. Vous pouvez facilement demander aux invités de revoir eux-mêmes leur besoin d’accès.

1. Pour créer une révision d’accès pour l’application, choisissez d’y inclure uniquement les invités et de leur faire revoir leur accès eux-mêmes. Pour plus d’informations, consultez [Créer une révision d’accès de groupes ou d’applications](create-access-review.md).

2. Demandez à chaque invité de revoir son propre accès à l’application. Par défaut, chaque invité ayant accepté une invitation recevra un message électronique de la part d’Azure AD. Ce message contient un comprenant un lien vers la révision d’accès dans le volet d’accès de votre organisation. Les instructions d’Azure AD pour les invités concernent la [révision d’accès aux groupes ou aux applications](perform-access-review.md).

3. Une fois que les réviseurs ont fait part de leur avis, arrêtez la révision d’accès et appliquez les modifications. Pour plus d’informations, consultez [Effectuer une révision d’accès de groupes ou d’applications](complete-access-review.md).

4. En plus des utilisateurs indiquant n’avoir plus besoin d’un accès continu, vous pouvez également retirer les utilisateurs invités n’ayant pas répondu. Il est probable que les utilisateurs qui ne répondent pas ne reçoivent plus de courrier électronique. Vous pouvez également supprimer les utilisateurs invités qui n’ont pas été sélectionnées pour participer, surtout s’ils n’ont pas été récemment invités. N’ayant pas accepté leur invitation, ces utilisateurs n’avaient donc pas accès à l’application. 

### <a name="ask-a-sponsor-to-review-a-guests-access-to-an-application"></a>Demander à un commanditaire de revoir l’accès d’un invité à une application

Vous pouvez demander à un commanditaire, tel que le propriétaire d’une application, de revoir le besoin de l’invité à disposer d’un accès continu à l’application.

1. Pour créer une révision d’accès pour l’application, choisissez d’y inclure uniquement les invités. Définissez ensuite un ou plusieurs utilisateurs comme réviseurs. Pour plus d’informations, consultez [Créer une révision d’accès de groupes ou d’applications](create-access-review.md).

2. Demandez aux réviseurs de donner leur avis. Par défaut, chacun d’eux reçoit un e-mail provenant d’Azure AD et contenant un lien vers le panneau d’accès, dans lequel ils peuvent [réviser l’accès à des groupes ou à des applications](perform-access-review.md).

3. Une fois que les réviseurs ont fait part de leur avis, arrêtez la révision d’accès et appliquez les modifications. Pour plus d’informations, consultez [Effectuer une révision d’accès de groupes ou d’applications](complete-access-review.md).

### <a name="ask-guests-to-review-their-need-for-access-in-general"></a>Demander aux invités de revoir leurs besoins d’accès en général

Dans certaines organisations, les invités peuvent ne pas être conscients de leur appartenance à un groupe.

> [!NOTE]
> Dans les versions antérieures du portail Azure, les utilisateurs de type Invité ne pouvaient pas disposer d’un accès administratif. Dans certains cas, il est possible qu’un administrateur de votre répertoire ait défini la valeur UserType d’un invité sur Membre à l’aide de PowerShell. Si cette modification avait auparavant été apportée dans votre répertoire, il est possible que la requête précédente n’inclue pas tous les utilisateurs invités qui disposaient auparavant de droits d’accès administratif. Dans ce cas, vous devez modifier le type d’utilisateur de l’invité ou inclure manuellement l’invité dans le groupe.

1. Créez un groupe de sécurité dans Azure AD avec les invités comme membres, si un groupe approprié n’existe pas déjà. Par exemple, vous pouvez créer un groupe pour lequel l’appartenance des invités est gérée manuellement. Ou bien, vous pouvez créer un groupe dynamique doté d’un nom tel que « Invités de Contoso » pour les utilisateurs dans le client Contoso dont la valeur d’attribut UserType est configurée sur Invité.  Pour plus d’efficacité, assurez-vous que le groupe est principalement formé d’invités. Ne sélectionnez pas un groupe dont les utilisateurs sont des membres, car ils n’ont pas besoin d’être examinés.  En outre, n’oubliez pas qu’un utilisateur invité membre du groupe peut voir les autres membres du groupe.

2. Pour créer une révision d’accès pour ce groupe, choisissez les membres comme réviseurs. Pour plus d’informations, consultez [Créer une révision d’accès de groupes ou d’applications](create-access-review.md).

3. Demandez à chaque invité de revoir sa propre appartenance. Par défaut, chaque invité ayant accepté une invitation recevra un message électronique de la part d’Azure AD comprenant un lien vers la révision d’accès dans le panneau d’accès de votre organisation. Les instructions d’Azure AD pour les invités concernent la [révision d’accès aux groupes ou aux applications](perform-access-review.md).  Ces invités qui n’ont pas accepté leur invitation apparaîtront dans les résultats de la révision comme « Non averti ».

4. Une fois que les réviseurs ont fait part de leur avis, arrêtez la révision d’accès. Pour plus d’informations, consultez [Effectuer une révision d’accès de groupes ou d’applications](complete-access-review.md).

5. Retirez l’accès invité aux invités ayant été refusés, n’ayant pas terminé la révision ou bien n’ayant pas accepté leur invitation. Si certains des invités sont des contacts sélectionnés pour participer à la révision ou s’ils n’avaient précédemment pas accepté une invitation, vous pouvez désactiver leur compte à l’aide du portail Azure ou de PowerShell. Si l’invité n’a plus besoin d’un accès et n’est pas un contact, vous pouvez supprimer son objet utilisateur de votre répertoire à l’aide du portail Azure ou de PowerShell pour supprimer l’objet utilisateur invité.

## <a name="next-steps"></a>Étapes suivantes

[Créer une révision d’accès de groupes ou d’applications](create-access-review.md)