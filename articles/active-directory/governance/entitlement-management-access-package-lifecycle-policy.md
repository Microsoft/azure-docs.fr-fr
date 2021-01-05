---
title: Modifier les paramètres de cycle de vie d’un package d’accès dans la gestion des droits d’utilisation Azure AD – Azure Active Directory
description: Apprenez à modifier les paramètres d'informations sur le demandeur et de cycle de vie d'un package d'accès dans la fonctionnalité de gestion des droits d'utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4fd119238476b842a4dba1efeec86544ca263920
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90980094"
---
# <a name="change-lifecycle-settings-for-an-access-package-in-azure-ad-entitlement-management"></a>Modifier les paramètres de cycle de vie d’un package d’accès dans la gestion des droits d’utilisation Azure AD

En tant que gestionnaire de package d’accès, vous pouvez modifier les paramètres de cycle de vie d’un package d’accès à tout moment en modifiant une stratégie existante. Si vous modifiez la date d'expiration d'une stratégie, la date d'expiration des demandes en attente d'approbation ou approuvées ne change pas.

Cet article explique comment modifier les paramètres de cycle de vie d’un package d’accès existant.

## <a name="open-requestor-information"></a>Ouvrir les informations relatives au demandeur
Pour veiller à ce que les utilisateurs bénéficient d'un accès approprié à un package d'accès, des questions personnalisées peuvent être configurées et posées aux utilisateurs sollicitant l'accès à certains packages d'accès. Les options de configuration sont les suivantes : localisation, obligatoire/facultatif et formats texte/réponse à choix multiples. Les questions seront présentées aux demandeurs lorsqu'ils solliciteront l'accès au package, et les approbateurs pourront consulter leurs réponses afin de prendre une décision. Procédez comme suit pour configurer des questions dans un package d'accès :

## <a name="open-lifecycle-settings"></a>Ouvrir les paramètres de cycle de vie

Pour modifier les paramètres de cycle de vie d’un package d’accès, vous devez ouvrir la stratégie correspondante. Suivez ces étapes pour ouvrir les paramètres de cycle de vie pour un package d’accès.

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs ou gestionnaire de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Cliquez sur **Stratégies** puis sur la stratégie qui contient les paramètres de cycle de vie que vous souhaitez modifier.

    Le volet Détails de la stratégie s'ouvre au bas de la page.

    ![Package d’accès - Volet des détails de la stratégie](./media/entitlement-management-shared/policy-details.png)

1. Cliquez sur **Modifier** pour modifier la stratégie.

    ![Package d'accès - Modifier la stratégie](./media/entitlement-management-shared/policy-edit.png)

1. Cliquez sur l’onglet **Cycle de vie** pour ouvrir les paramètres de cycle de vie.

[!INCLUDE [Entitlement management lifecycle policy](../../../includes/active-directory-entitlement-management-lifecycle-policy.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Changer les paramètres de demande et d’approbation pour un package d’accès](entitlement-management-access-package-request-policy.md)