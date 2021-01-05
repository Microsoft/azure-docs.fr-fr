---
title: Autorisations dans Azure Advisor
description: Présentation des autorisations Advisor et de la façon dont elles peuvent bloquer votre capacité à configurer vos abonnements ou à reporter ou rejeter des recommandations.
ms.topic: article
ms.date: 04/03/2019
ms.openlocfilehash: 402a21c47c4cba8f747d5d4601f9c95034c99262
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91712933"
---
# <a name="permissions-in-azure-advisor"></a>Autorisations dans Azure Advisor

Azure Advisor fournit des recommandations basées sur l’utilisation et la configuration de vos ressources et abonnements Azure. Advisor utilise les [rôles intégrés](../role-based-access-control/built-in-roles.md) fournis par le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) pour gérer votre accès aux recommandations et aux fonctionnalités d’Advisor. 

## <a name="roles-and-their-access"></a>Les rôles et leur accès

Le tableau suivant définit les rôles et leurs accès au sein d’Advisor :

| **Rôle** | **Afficher les recommandations** | **Modifier une règle** | **Modifier la configuration de l’abonnement** | **Modifier la configuration du groupe de ressources**| **Ignorer et différer des recommandations**|
|---|:---:|:---:|:---:|:---:|:---:|
|Propriétaire de l’abonnement|**X**|**X**|**X**|**X**|**X**|
|Collaborateur de l’abonnement|**X**|**X**|**X**|**X**|**X**|
|Lecteur de l’abonnement|**X**|--|--|--|--|
|Propriétaire du groupe de ressources|**X**|--|--|**X**|**X**|
|Contributeur du groupe de ressources|**X**|--|--|**X**|**X**|
|Lecteur du groupe de ressources|**X**|--|--|--|--|
|Propriétaire de la ressource|**X**|--|--|--|**X**|
|Contributeur de la ressource|**X**|--|--|--|**X**|
|Lecteur de la ressource|**X**|--|--|--|--|

> [!NOTE]
> L’accès à l’affichage des recommandations dépend de votre accès à la ressource concernée par la recommandation.

## <a name="permissions-and-unavailable-actions"></a>Autorisations et actions non disponibles

Un manque d’autorisations appropriées peut bloquer votre capacité à effectuer des actions dans Advisor. Voici certains problèmes courants.

### <a name="unable-to-configure-subscriptions-or-resource-groups"></a>Impossible de configurer les abonnements ou les groupes de ressources

Lorsque vous tentez de configurer des abonnements ou des groupes de ressources dans Advisor, vous pouvez voir que l’option pour en inclure ou en exclure est désactivée. Cet état indique que vous n’avez pas un niveau d’autorisation suffisant pour ce groupe de ressources ou cet abonnement. Pour résoudre ce problème, découvrez comment [Accorder un accès utilisateur](../role-based-access-control/quickstart-assign-role-user-portal.md).

### <a name="unable-to-postpone-or-dismiss-a-recommendation"></a>Impossible de reporter ou ignorer une recommandation

Si vous recevez une erreur lorsque vous tentez de reporter ou ignorer une recommandation, c’est que vous ne disposez pas des autorisations suffisantes pour le faire. Vérifiez que vous disposez au moins d’un accès Contribueur à la ressource affectée par la recommandation que vous souhaitez différer ou ignorer. Pour résoudre ce problème, découvrez comment [Accorder un accès utilisateur](../role-based-access-control/quickstart-assign-role-user-portal.md).

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a présenté la façon dont Advisor utilise Azure RBAC pour contrôler les autorisations des utilisateurs. Il vous a aussi expliqué comment résoudre certains problèmes courants. Pour en savoir plus sur Advisor, consultez les ressources suivantes :

- [Présentation d’Azure Advisor](./advisor-overview.md)
- [Prise en main d’Azure Advisor](./advisor-get-started.md)
