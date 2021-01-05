---
title: Utiliser des groupes cloud pour gérer les attributions de rôles dans Azure Active Directory | Microsoft Docs
description: Affichez un aperçu des rôles Azure AD personnalisés pour la délégation de la gestion des identités. Gérez les attributions de rôles Azure dans le portail Azure, PowerShell ou l’API Graph.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: article
ms.date: 11/05/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 54988c8bbc4a9c3d448ac35f31e97e2d20228209
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007566"
---
# <a name="use-cloud-groups-to-manage-role-assignments-in-azure-active-directory-preview"></a>Utiliser des groupes cloud pour gérer les attributions de rôles dans Azure Active Directory (préversion)

Azure Active Directory (Azure AD) introduit une préversion publique dans laquelle vous pouvez attribuer un groupe cloud à des rôles intégrés Azure AD. Grâce à cette fonctionnalité, vous pouvez utiliser des groupes pour accorder un accès administrateur dans Azure AD avec le minimum d’effort de la part de vos administrateurs généraux et administrateurs de rôle privilégié.

Examinez cet exemple : Contoso a recruté des personnes pour toutes les zones géographiques afin de gérer et réinitialiser les mots de passe des employés de son organisation Azure AD. Au lieu de demander à un Administrateur de rôle privilégié ou à un Administrateur général d’attribuer le rôle Administrateur de mot de passe à chaque personne individuellement, il peut créer un groupe Contoso_Helpdesk_Administrators et l’attribuer au rôle. Lorsque des personnes rejoignent le groupe, elles se voient attribuer le rôle indirectement. Votre workflow de gouvernance actuel peut ensuite prendre en charge le processus d’approbation et l’audit de l’appartenance du groupe pour s’assurer que seuls les utilisateurs légitimes sont membres du groupe et donc affectés au rôle Administrateur de mot de passe.

## <a name="how-this-feature-works"></a>Principe de la fonctionnalité

Créez un nouveau groupe Microsoft 365 ou un groupe de sécurité dont la propriété « isAssignableToRole » a la valeur « true ». Vous pouvez également activer cette propriété lors de la création d’un groupe dans le portail Azure en activant l’option **Des rôles Azure AD peuvent être attribués au groupe**. Dans les deux cas, vous pouvez affecter le groupe à un ou plusieurs rôles Azure AD de la même façon que vous attribuez des rôles aux utilisateurs. Un maximum de 200 groupes assignables à un rôle peuvent être créés dans une seule organisation (locataire) Azure AD.

Si vous ne souhaitez pas que les membres du groupe disposent d’un accès permanent au rôle, vous pouvez utiliser Azure AD Privileged Identity Management. Assignez un groupe comme membre éligible d’un rôle Azure AD. Chaque membre du groupe peut alors faire activer son affectation pour le rôle auquel le groupe est assigné. Il peut ensuite activer son attribution de rôle pour une durée déterminée.

> [!Note]
> Vous devez disposer d’une version mise à jour de Privileged Identity Management pour être en mesure d’assigner un groupe à un rôle Azure AD via PIM. Vous pourriez être sur une version antérieure de PIM parce que votre organisation Azure AD exploite l’API Privileged Identity Management. Veuillez contacter l’alias pim_preview@microsoft.com pour déplacer votre organisation et mettre à jour votre API. Pour plus d’informations, consultez [Fonctionnalités et rôles Azure AD dans PIM](../privileged-identity-management/azure-ad-roles-features.md).

## <a name="why-we-enforce-creation-of-a-special-group-for-assigning-it-to-a-role"></a>Avantages de la création d’un groupe spécial pour l’attribution d’un rôle

Si un rôle est attribué à un groupe, tout administrateur informatique qui peut gérer l’appartenance au groupe peut également gérer indirectement l’appartenance à ce rôle. Par exemple, supposons qu’un groupe Contoso_User_Administrators est affecté au rôle Administrateur de compte d’utilisateur. Un Administrateur Exchange qui peut modifier l’appartenance au groupe peut s’ajouter au groupe Contoso_User_Administrators et devenir ainsi un Administrateur de compte d’utilisateur. Comme vous pouvez le voir, un administrateur peut élever ses privilèges d’une manière que vous n’aviez pas prévue.

Azure AD vous permet de protéger un groupe affecté à un rôle à l’aide d’une nouvelle propriété appelée isAssignableToRole pour les groupes. Seuls les groupes cloud dont la propriété isAssignableToRole a la valeur « true » au moment de la création peuvent être affectés à un rôle. Cette propriété est non modifiable ; une fois qu’un groupe a été créé avec cette propriété définie sur « true », il ne peut pas être modifié. Vous ne pouvez pas définir la propriété sur un groupe existant.
Nous avons conçu la manière dont les groupes sont affectés aux rôles afin d’éviter que ce genre de violation potentielle ne se produise :

- Seuls les Administrateurs généraux et les Administrateurs de rôle privilégié peuvent créer un groupe assignable à un rôle (avec la propriété « isAssignableToRole » activée).
- Il ne peut pas s’agir d’un groupe dynamique Azure AD ; autrement dit, il doit avoir le type d’appartenance « Assigned ». L’alimentation automatisée de groupes dynamiques peut entraîner l’ajout d’un compte indésirable au groupe et don son affectation au rôle.
- Par défaut, seuls les Administrateurs généraux et les Administrateurs de rôle privilégié peuvent gérer l’appartenance à un groupe assignable à un rôle, mais vous pouvez déléguer la gestion des groupes assignables à un rôle en leur ajoutant des propriétaires de groupe.
- Pour empêcher une élévation des privilèges, les informations d’identification des membres et propriétaires d’un groupe assignable à un rôle ne peuvent être modifiées que par un Administrateur d’authentification privilégié ou un Administrateur général.
- Aucune imbrication. Un groupe ne peut pas être ajouté en tant que membre d’un groupe assignable à un rôle.

## <a name="limitations"></a>Limites

Les scénarios suivants ne sont pas pris en charge pour le moment :  

- Affecter des groupes locaux à des rôles Azure AD (intégrés ou personnalisés)

## <a name="known-issues"></a>Problèmes connus

- La fonctionnalité **Activer le lancement intermédiaire pour la connexion utilisateur managée** ne prend pas en charge l’affectation par le biais du groupe.
- *Clients titulaires d’une licence Azure AD P2 uniquement :* n’affectez pas un groupe actif à un rôle à la fois par le biais d’Azure AD et de Privileged Identity Management (PIM). En particulier, n’affectez pas un rôle à un groupe assignable à un rôle lors de sa création *et* à l’aide de PIM plus tard. Cela entraînera des problèmes où les utilisateurs ne pourront pas voir leurs attributions de rôles actives dans PIM, ainsi que l’impossibilité de supprimer ces attributions PIM. Les attributions éligibles ne sont pas visées dans ce scénario. Si vous tentez d’effectuer cette affectation, vous pourriez constater des comportements inattendus, par exemple :
  - L’heure de fin de l’attribution de rôle peut s’afficher de manière incorrecte.
  - Dans le portail PIM, la rubrique **Mes rôles** ne peut afficher qu’une seule attribution de rôle, quel que soit le nombre de méthodes par lesquelles l’affectation est accordée (par le biais d’un ou de plusieurs groupes, directement ou non).
- *Clients titulaires d’une licence Azure AD P2 uniquement :* même après avoir supprimé le groupe, celui-ci apparaît toujours comme membre éligible du rôle dans l’interface utilisateur PIM. Fonctionnellement, il n’y a aucun problème ; il s’agit simplement d’un problème de cache dans le portail Azure.  
- Utilisez le nouveau [Centre d'administration Exchange](https://admin.exchange.microsoft.com/) pour les attributions de rôles via l'appartenance de groupe. L'ancien Centre d'administration Exchange ne prend pas encore en charge cette fonctionnalité. Les cmdlets Exchange PowerShell fonctionnent normalement.
- Le portail Azure Information Protection (le portail classique) ne reconnaît pas encore l’appartenance au rôle par le biais d’un groupe. Vous pouvez [migrer vers la plateforme unifiée d’étiquetage de confidentialité](/azure/information-protection/configure-policy-migrate-labels), puis utiliser le centre Security & Compliance d’Office 365 pour gérer les rôles à l’aide des affectations de groupe.

Nous travaillons à corriger ces problèmes.

## <a name="required-license-plan"></a>Plan de licence obligatoire

Pour utiliser cette fonctionnalité, vous devez disposer d’une licence Azure AD Premium P1 dans votre organisation Azure AD. Afin d’utiliser également Privileged Identity Management pour l’activation de rôle juste-à-temps, vous devez disposer d’une licence Azure AD Premium P2. Pour trouver la licence appropriée à vos besoins, consultez [Comparaison des fonctionnalités mises à la disposition générale des plans Gratuit et Premium](../fundamentals/active-directory-whatis.md#what-are-the-azure-ad-licenses).

## <a name="next-steps"></a>Étapes suivantes

- [Créer un groupe assignable à un rôle](groups-create-eligible.md)
- [Attribuer un rôle à un groupe assignable à un rôle](groups-assign-role.md)
