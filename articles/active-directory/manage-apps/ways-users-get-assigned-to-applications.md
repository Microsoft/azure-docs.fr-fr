---
title: Comprendre comment les utilisateurs sont affectés aux applications dans Azure Active Directory
description: Comprendre comment les utilisateurs sont affectés à une application qui utilise Azure Active Directory pour la gestion des identités.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: kenwith
ms.openlocfilehash: b9786f9d9da363f15bd2f59390d5dddf86bc1bf9
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658848"
---
# <a name="understand-how-users-are-assigned-to-apps-in-azure-active-directory"></a>Comprendre comment les utilisateurs sont affectés aux applications dans Azure Active Directory
Cet article vous explique comment les utilisateurs sont affectés à une application dans votre client.

## <a name="how-do-users-get-assigned-to-an-application-in-azure-ad"></a>Comment les utilisateurs sont-ils affectés à une application dans Azure AD ?
Pour qu’un utilisateur puisse accéder à une application, il doit tout d’abord être affecté à cette application d’une certaine manière. L’affectation peut être effectuée par un administrateur, un délégué de l’entreprise ou, parfois, par l’utilisateur lui-même. Voici une description des méthodes permettant d’affecter des utilisateurs à des applications :

*  Un administrateur [affecte un utilisateur](./assign-user-or-group-access-portal.md) directement à l’application
*  Un administrateur [affecte un groupe](./assign-user-or-group-access-portal.md) dont l’utilisateur est membre à l’application, y compris :
    * Un groupe synchronisé localement
    * Un groupe de sécurité statique créé dans le cloud
    * Un [groupe de sécurité dynamique](../enterprise-users/groups-dynamic-membership.md) créé dans le cloud
    * Un groupe Microsoft 365 créé dans le cloud
    * Le groupe [Tous les utilisateurs](../fundamentals/active-directory-groups-create-azure-portal.md)
*  Un administrateur active l’[accès aux applications en libre-service](./manage-self-service-access.md) pour autoriser un utilisateur à ajouter une application à l’aide de la fonctionnalité **Ajouter une application** de [Mes applications](../user-help/my-apps-portal-end-user-access.md) **sans approbation de l’entreprise**.
*  Un administrateur active l’[accès aux applications en libre-service](./manage-self-service-access.md) pour autoriser un utilisateur à ajouter une application à l’aide de la fonctionnalité **Ajouter une application** de [Mes accès](../user-help/my-apps-portal-end-user-access.md), mais uniquement **après approbation d’un ensemble sélectionné d’approbateurs d’entreprise**.
*  Un administrateur active la [gestion des groupes en libre-service](../enterprise-users/groups-self-service-management.md) pour permettre à un utilisateur de joindre un groupe auquel une application est affectée, **sans approbation d’entreprise**
*  Un administrateur active la [gestion des groupes en libre-service](../enterprise-users/groups-self-service-management.md) pour permettre à un utilisateur de joindre un groupe auquel une application est affectée, mais uniquement **après approbation d’un ensemble sélectionné d’approbateurs d’entreprise**
*  Un administrateur affecte une licence à un utilisateur pour une première application, par exemple [Microsoft 365](https://products.office.com/)
*  Un administrateur affecte une licence à un groupe dont l’utilisateur est membre pour une première application, par exemple [Microsoft 365](https://products.office.com/)
*  Un [administrateur donne son consentement à une application](../develop/howto-convert-app-to-be-multi-tenant.md) pour être utilisée par tous les utilisateurs, puis un utilisateur se connecte à l’application
* Un utilisateur [donne lui-même son consentement à une application](../develop/howto-convert-app-to-be-multi-tenant.md) en se connectant à l’application

## <a name="next-steps"></a>Étapes suivantes
* [Série de guides de démarrage rapide sur la gestion des applications](view-applications-portal.md)
* [Qu’est-ce que la gestion des applications ?](what-is-application-management.md)
* [Qu’est-ce que l’authentification unique ?](what-is-single-sign-on.md)