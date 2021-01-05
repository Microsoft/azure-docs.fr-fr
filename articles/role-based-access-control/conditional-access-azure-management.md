---
title: Gérer l’accès à la gestion Azure avec l’accès conditionnel dans Azure AD
description: Découvrez l’utilisation de l’accès conditionnel dans Azure AD pour gérer l’accès à la gestion Azure.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: skwan
ms.assetid: 0adc8b11-884e-476c-8c43-84f9bf12a34b
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/15/2019
ms.author: rolyon
ms.reviewer: skwan
ms.openlocfilehash: 2a52635dbaa7a76034f3a535b099320a901e8c07
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83758773"
---
# <a name="manage-access-to-azure-management-with-conditional-access"></a>Gérer l’accès à la gestion Azure avec l’accès conditionnel

> [!CAUTION]
> Vous devez comprendre le fonctionnement de l’accès conditionnel avant de configurer une stratégie pour gérer l’accès à la gestion Azure. Veillez à ne pas créer de conditions susceptibles de bloquer votre propre accès au portail.

L’accès conditionnel dans Azure Active Directory (Azure AD) contrôle l’accès aux applications cloud en fonction de conditions spécifiées par vos soins. Pour autoriser l’accès, vous créez des stratégies d’accès conditionnel qui autorisent ou interdisent l’accès selon que les exigences définies dans la stratégie sont respectées ou non. 

En règle générale, vous utilisez l’accès conditionnel pour contrôler l’accès à vos applications cloud. Vous pouvez également définir des stratégies pour contrôler l’accès à la gestion Azure en fonction de certaines conditions (par exemple, le risque de connexion, l’emplacement ou l’appareil) et pour appliquer des exigences, telles que l’authentification multifacteur.

Pour créer une stratégie pour la gestion Azure, sélectionnez **Gestion Microsoft Azure** sous **Applications cloud** quand vous choisissez l’application à laquelle appliquer la stratégie.

![Accès conditionnel pour la gestion Azure](./media/conditional-access-azure-management/conditional-access-azure-mgmt.png)

La stratégie que vous créez s’applique à tous les points de terminaison de gestion Azure, notamment :

- Portail Azure
- Fournisseur Azure Resource Manager
- API Gestion des services classiques
- Azure PowerShell
- Portail de l’administrateur d’abonnements Visual Studio
- Azure DevOps
- Portail Azure Data Factory

Notez que la stratégie s’applique à Azure PowerShell, qui appelle l’API Azure Resource Manager. Elle ne s’applique pas à [Azure AD PowerShell](/powershell/azure/active-directory/install-adv2), qui appelle Microsoft Graph.

Pour plus d’informations sur le mode de configuration d’un exemple de stratégie pour activer l’accès conditionnel pour la gestion de Microsoft Azure, consultez l’article [Accès conditionnel : Exiger MFA pour la gestion Azure](../active-directory/conditional-access/howto-conditional-access-policy-azure-management.md).
