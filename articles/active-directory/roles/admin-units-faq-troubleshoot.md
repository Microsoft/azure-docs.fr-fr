---
title: Résolution des problèmes d’unités administratives et FAQ – Azure Active Directory | Microsoft Docs
description: Examinez les unités administratives pour accorder des autorisations d’étendue restreinte dans Azure Active Directory.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.subservice: roles
ms.workload: identity
ms.date: 11/04/2020
ms.author: curtand
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: f585be2057bda19038fff1066e7864c6796576c0
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93394677"
---
# <a name="azure-ad-administrative-units-troubleshooting-and-faq"></a>Unités administratives Azure AD : Dépannage et FAQ

Pour un contrôle administratif plus précis dans Azure Active Directory (Azure AD), vous pouvez assigner des utilisateurs à un rôle Azure AD dont l’étendue est limitée à une ou plusieurs unités administratives. Pour obtenir des exemples de scripts PowerShell pour les tâches courantes, consultez [Utiliser des unités administratives](/powershell/azure/active-directory/working-with-administrative-units?view=azureadps-2.0&preserve-view=true).

## <a name="frequently-asked-questions"></a>Forum aux questions

**Q : Pourquoi ne puis-je pas créer d’unité administrative ?**

**R :** Seul un *administrateur général* ou un *administrateur de rôle privilégié* peut créer une unité administrative dans Azure AD. Assurez-vous que l’utilisateur qui tente de créer l’unité administrative se voit attribuer le rôle d’ *administrateur général* ou d’ *administrateur de rôle privilégié*.

**Q : J’ai ajouté un groupe à une unité administrative. Pourquoi les membres du groupe n’apparaissent-ils toujours pas ?**

**R :** Lorsque vous ajoutez un groupe à une unité administrative, cela n’entraîne pas l’ajout de tous les membres du groupe. Les utilisateurs doivent être directement assignés à l’unité administrative.

**Q : Je viens d’ajouter (ou de supprimer) un membre de l’unité administrative. Pourquoi le membre n’apparaît-il pas (ou pourquoi est-il toujours affiché) dans l’interface utilisateur ?**

**R :** Parfois, quelques minutes peuvent s’écouler avant que la volet **Unités administratives** reflète l’ajout ou la suppression d’un ou de plusieurs membres d’une unité administrative. Vous pouvez également accéder directement aux propriétés de la ressource associée et voir si l’action a été accomplie. Pour plus d’informations sur les utilisateurs et les groupes dans les unités administratives, consultez [Afficher une liste d’unités administratives pour un utilisateur](admin-units-add-manage-users.md) et [Afficher une liste d’unités administratives pour un groupe](admin-units-add-manage-groups.md).

**Q : Je suis administrateur de mot de passe délégué sur une unité administrative. Pourquoi ne puis-je pas réinitialiser le mot de passe d’un utilisateur spécifique ?**

**R :** En tant qu’administrateur d’une unité administrative, vous pouvez uniquement réinitialiser le mot de passe des utilisateurs assignés à votre unité administrative. Assurez-vous que l’utilisateur dont vous ne parvenez pas à réinitialiser le mot de passe appartient à l’unité administrative à laquelle vous avez été assigné. Si l’utilisateur appartient bien à la même unité administrative, mais que vous ne pouvez toujours pas réinitialiser son mot de passe, vérifiez les rôles attribués à l’utilisateur. 

Pour empêcher une élévation de privilèges, un administrateur assigné à une unité administrative ne peut pas réinitialiser le mot de passe d’un utilisateur dont le rôle s’étend à l’organisation entière.

**Q : Pourquoi les unités administratives sont-elles nécessaires ? Nous n’avons pas pu utiliser des groupes de sécurité pour définir une étendue.**

**R :** Les groupes de sécurité ont un modèle d’objectif et d’autorisation existant. Par exemple, un *administrateur d’utilisateurs* peut gérer l’appartenance de tous les groupes de sécurité de l’organisation Azure AD. Le rôle peut utiliser des groupes pour gérer l’accès aux applications telles que Salesforce. Un *administrateur d’utilisateurs* ne doit pas être en mesure de gérer le modèle de délégation proprement dit, ce qui serait le cas si des groupes de sécurité étaient étendus pour prendre en charge des scénarios de « regroupement des ressources ». 

Les unités administratives, telles les unités d’organisation dans Windows Server Active Directory, sont conçues pour permettre de déterminer l’étendue de l’administration d’un vaste éventail d’objets annuaire. Des groupes de sécurité peuvent eux-mêmes être membres d’étendues de ressources. L’utilisation de groupes de sécurité pour définir l’ensemble des groupes de sécurité qu’un administrateur peut gérer risque de prêter à confusion.

**Q : Qu’implique le fait d’ajouter un groupe à une unité administrative ?**

**R :** L’ajout d’un groupe à une unité administrative a pour effet d’intégrer ce groupe dans l’étendue de gestion d’un *administrateur d’utilisateurs* dont l’étendue définie est également cette unité administrative. Les administrateurs d’utilisateurs de l’unité administrative peuvent gérer le nom et l’appartenance du groupe lui-même. Cela n’a pas pour effet d’accorder à l’ *administrateur d’utilisateurs* des autorisations permettant de gérer les utilisateurs du groupe (par exemple, pour réinitialiser leur mot de passe). Pour accorder à l’ *administrateur d’utilisateurs* la capacité de gérer les utilisateurs, il faut que ceux-ci soient des membres directs de l’unité administrative.

**Q : Une ressource (utilisateur ou groupe) peut-elle être membre de plusieurs unités administratives ?**

**R :** Oui, une ressource peut être membre de plusieurs unités administratives. Elle peut être gérée par tous les administrateurs à l’échelle de l’organisation ou des unités administratives qui disposent d’autorisations sur la ressource.

**Q : Les unités administratives sont-elles disponibles dans les organisations B2C ?**

**R :** Non, les unités administratives ne sont pas disponibles pour les organisations B2C.

**Q : Les unités administratives imbriquées sont-elles prises en charge ?**

**R :** Non, les unités administratives imbriquées ne sont pas prises en charge.

**Q : Les unités administratives sont-elles prises en charge dans PowerShell et l’API Graph ?**

**R :** Oui. Vous trouverez des informations sur la prise en charge des unités administratives dans la [documentation relative aux cmdlets PowerShell](/powershell/module/Azuread/?view=azureadps-2.0&preserve-view=true) et les [exemples de scripts](/powershell/azure/active-directory/working-with-administrative-units?view=azureadps-2.0&preserve-view=true).

Recherchez la prise en charge du [type de ressource administrativeUnit](/graph/api/resources/administrativeunit?view=graph-rest-1.0&preserve-view=true) dans Microsoft Graph.

## <a name="next-steps"></a>Étapes suivantes

- [Limiter l’étendue des rôles à l’aide d’unités administratives](administrative-units.md)
- [Gestion des unités administratives](admin-units-manage.md)
