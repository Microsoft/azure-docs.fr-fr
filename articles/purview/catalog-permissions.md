---
title: Autorisations du catalogue (préversion)
description: Cet article donne une vue d’ensemble de la configuration du contrôle d’accès en fonction du rôle (RBAC) dans Azure Purview
author: yarong
ms.author: yarong
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 10/20/2020
ms.openlocfilehash: e87c9efc910d08307d40d42e58f8272a01902a41
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550206"
---
# <a name="role-based-access-control-in-azure-purviews-data-plane"></a>Contrôle d’accès en fonction du rôle dans le plan de données d’Azure Purview

Cet article décrit comment le contrôle d’accès en fonction du rôle (RBAC) est implémenté dans le [plan de données](../azure-resource-manager/management/control-plane-and-data-plane.md#data-plane) d’Azure Purview.

> [!IMPORTANT]
> Le principal qui a créé un compte Purview reçoit automatiquement toutes les autorisations de plan de données, quels que soient les rôles de plan de données dans lesquels ils peuvent ou non se trouver. Pour qu’un autre utilisateur fasse quoi que ce soit dans Azure Purview, il doit se trouver dans au moins l’un des rôles de plan de données prédéfinis.

## <a name="azure-purviews-pre-defined-data-plane-roles"></a>Rôles de plan de données prédéfinis d’Azure Purview

Azure Purview définit un ensemble de rôles de plan de données prédéfinis qui peuvent être utilisés pour contrôler qui peut accéder à quoi, dans Azure Purview. Ces rôles sont les suivants :

* **Rôle de lecteur de données Purview** : peut accéder au portail Purview et lire tout le contenu dans Azure Purview à l’exception des liaisons d’analyse.
* **Rôle de conservateur de données Purview** : peut accéder au portail Purview et lire tout le contenu dans Azure Purview, à l’exception des liaisons d’analyse, peut modifier les informations sur les ressources, peut modifier les définitions de classification et les termes de glossaire, et peut appliquer des classifications et des termes de glossaire aux ressources.
* **Rôle d’administrateur de source de données Purview** : n’a pas accès au portail Purview (l’utilisateur doit également se trouver dans les rôles Lecteur de données ou Conservateur de données) et peut gérer tous les aspects de l’analyse des données dans Azure Purview, mais ne dispose pas d’un accès en lecture ou en écriture au contenu dans Azure Purview en plus de ceux liés à l’analyse.

## <a name="understanding-how-to-use-azure-purviews-data-plane-roles"></a>Comprendre comment utiliser les rôles de plan de données d’Azure Purview

Lors de la création d’un compte Azure Purview, le créateur sera traité comme s’il appartient aux deux rôles Conservateur de données Purview et Administrateur de source de données Purview. Cependant, le créateur du compte n’est pas affecté à ces rôles dans le magasin de rôles. Azure Purview reconnaît que le principal est le créateur du compte et étend ces fonctionnalités à celui-ci en fonction de son identité.

Tous les autres utilisateurs peuvent utiliser le compte Azure Purview seulement s’ils figurent dans au moins un de ces rôles. Cela signifie que lors de la création d’un compte Azure Purview, seul le créateur peut accéder au compte ou utiliser ses API jusqu’à ce qu’il soit placé dans un ou plusieurs des rôles prédéfinis.

Notez que le rôle Administrateur de source de données Purview a deux scénarios pris en charge. Le premier scénario concerne les utilisateurs qui sont des lecteurs de données ou des conservateurs de données Purview qui doivent également être en mesure de créer des analyses. Ces utilisateurs doivent avoir les deux rôles, au moins Lecteur de données Purview ou Conservateur de données Purview, ainsi qu’être affectés au rôle Administrateur de source de données Purview.

L’autre scénario pour Administrateur de source de données Purview concerne les processus de programmation, tels que les principaux du service, qui doivent être en mesure de configurer et de surveiller les analyses, mais ne doivent avoir accès à aucune des données du catalogue.

Ce scénario peut être implémenté en plaçant le principal du service dans le rôle Administrateur de source de données Purview sans être placé dans l’un des deux autres rôles. Le principal n’a pas accès au portail Purview, mais ce n’est pas un problème parce qu’il s’agit d’un principal de programmation et qu’il communique uniquement via des API.

## <a name="putting-users-into-roles"></a>Placement d’utilisateurs dans des rôles

La première chose à faire après la création d’un compte Azure Purview consiste donc à affecter des personnes à ces rôles.

L’attribution de rôle est gérée via [RBAC d’Azure](../role-based-access-control/overview.md).

Seuls deux rôles de plan de contrôle intégrés dans Azure peuvent affecter des rôles d’utilisateurs, ceux-ci sont soit des propriétaires, soit des administrateurs d’accès utilisateur. Par conséquent, pour placer des personnes dans ces rôles pour Azure Purview, vous devez trouver une personne qui est propriétaire ou administrateur de l’accès utilisateur ou qui le devient.

### <a name="an-example-of-assigning-someone-to-a-role"></a>Exemple d’affectation d’une personne à un rôle

1. Accédez à https://portal.azure.com et accédez à votre compte Azure Purview.
1. Sur le côté gauche, cliquez sur « Contrôle d’accès (IAM) ».
1. Suivez ensuite les instructions générales fournies [ici](../role-based-access-control/quickstart-assign-role-user-portal.md#create-a-resource-group).

## <a name="role-definitions-and-actions"></a>Définitions de rôles et actions

Un rôle est défini comme une collection d’actions. Reportez-vous [ici](../role-based-access-control/role-definitions.md) pour plus d’informations sur la façon dont les rôles sont définis. Et [ici](../role-based-access-control/built-in-roles.md) pour les définitions de rôle pour les rôles d’Azure Purview.

## <a name="getting-added-to-a-data-plane-role-in-an-azure-purview-account"></a>Ajout à un rôle de plan de données dans un compte Azure Purview

Si vous souhaitez avoir accès à un compte Azure Purview pour pouvoir utiliser son studio ou appeler ses API, vous devez l’ajouter à un rôle de plan de données Azure Purview. Les seules personnes habilitées sont celles qui sont propriétaires ou administrateurs de l’accès utilisateur sur le compte Azure Purview. Pour la plupart des utilisateurs, l’étape suivante consiste à trouver un administrateur local qui peut aider à trouver les personnes permettant d’avoir accès.

En ce qui concerne les utilisateurs qui ont accès au [Portail Azure](https://portal.azure.com) de leur entreprise, ils peuvent rechercher le compte Azure Purview particulier qu’ils souhaitent rejoindre, cliquer sur l’onglet « Contrôle d’accès (IAM) » et découvrir qui sont les propriétaires ou les administrateurs de l’accès utilisateur (UAA). Toutefois, notez que dans certains cas, les groupes Azure Active Directory ou les principaux de service peuvent être utilisés comme propriétaires ou UAA, auquel cas il n’est pas possible de les contacter directement. Au lieu de cela, il faut trouver un administrateur pouvant fournir une aide.

## <a name="who-should-be-assigned-to-what-role"></a>Qui doit être affecté à quelle rôle ?

|Scénario d'utilisateur|Rôle(s) approprié(s)|
|-------------|-----------------|
|J’ai juste besoin de trouver des ressources, je ne veux rien modifier|Rôle de lecteur de données Purview|
|Je dois modifier les informations sur les ressources, y placer des classifications, les associer à des entrées de glossaire, etc.|Rôle de conservateur de données Purview|
|Je dois modifier le glossaire ou configurer de nouvelles définitions de classification|Rôle de conservateur de données Purview|
|Le principal du service de mon application doit transmettre (push) des données à Azure Purview|Rôle de conservateur de données Purview|
|Je dois configurer des analyses via Purview Studio|Rôle d’administrateur de source de données Purview plus au moins un rôle de lecteur de données Purview ou un rôle de conservateur de données Purview|
|Je dois activer un principal de service ou une autre identité de programmation pour configurer et surveiller les analyses dans Azure Purview sans autoriser l’identité de programmation à accéder aux informations du catalogue |Rôle d’administrateur de source de données Purview|
|Je dois placer les utilisateurs dans des rôles dans Azure Purview | Propriétaire ou Administrateur de l’accès utilisateur |

Passez à l’article suivant pour apprendre à ajouter un principal de sécurité à un rôle.

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un compte Azure Purview](create-catalog-portal.md)
