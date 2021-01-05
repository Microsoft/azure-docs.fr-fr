---
title: Gérer plusieurs locataires dans Azure Sentinel en tant que fournisseur de services de sécurité gérée | Microsoft Docs
description: Intégration et gestion de plusieurs locataires dans Azure Sentinel en tant que fournisseur de services de sécurité gérée (MSSP) à l’aide d’Azure Lighthouse.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/29/2020
ms.author: yelevin
ms.openlocfilehash: 011ddb883c028a954a8b0683c220bf6341eddb66
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91578142"
---
# <a name="manage-multiple-tenants-in-azure-sentinel-as-an-mssp"></a>Gérer plusieurs locataires dans Azure Sentinel en tant que MSSP

Si vous êtes un fournisseur de services de sécurité gérée (MSSP, managed security service provider) et que vous utilisez [Azure Lighthouse](../lighthouse/overview.md) pour offrir des services de centre d’opérations de sécurité à vos clients, vous pouvez gérer les ressources Azure Sentinel de vos clients directement depuis votre propre locataire Azure, sans avoir à vous connecter au locataire du client. 

## <a name="prerequisites"></a>Prérequis

- [Intégrez Azure Lighthouse](../lighthouse/how-to/onboard-customer.md).

- Pour que le processus fonctionne correctement, votre locataire (le locataire MSSP) doit disposer de fournisseurs de ressources Azure Sentinel inscrits sur au moins un abonnement. En outre, les fournisseurs de ressources doivent être inscrits pour chacun des locataires de vos clients. Si vous avez inscrit Azure Sentinel dans votre locataire et vos clients dans le leur, vous êtes prêt à commencer. Pour vérifier l’inscription, procédez comme suit :

    1. Sélectionnez **Abonnements** dans le portail Azure, puis sélectionnez un abonnement approprié dans le menu.

    1. Dans le menu de navigation de l’écran d’abonnement, sous **Paramètres**, sélectionnez **Fournisseurs de ressources**.

    1. À partir de l’écran ***nom de l’abonnement* | Fournisseurs de ressources**, recherchez et sélectionnez *Microsoft.OperationalInsights* et *Microsoft.SecurityInsights*, puis vérifiez la colonne **État**. Si l’état du fournisseur est *NotRegistered*, sélectionnez **Inscrire**.
    
        :::image type="content" source="media/multiple-tenants-service-providers/check-resource-provider.png" alt-text="Vérification des fournisseurs de ressources":::

## <a name="how-to-access-azure-sentinel-in-managed-tenants"></a>Accéder à Azure Sentinel dans des locataires gérés

1. Sous **Répertoire + abonnement**, sélectionnez les répertoires délégués (répertoire = locataire) et les abonnements où se trouvent les espaces de travail Azure Sentinel de votre client.

    :::image type="content" source="media/multiple-tenants-service-providers/directory-subscription.png" alt-text="Vérification des fournisseurs de ressources":::

1. Ouvrez Azure Sentinel. Vous voyez tous les espaces de travail des abonnements sélectionnés. Vous pourrez les utiliser en toute transparence comme n’importe quel espace de travail de votre propre locataire.

> [!NOTE]
> Vous ne pourrez pas déployer de connecteurs dans Azure Sentinel à partir d’un espace de travail managé. Pour déployer un connecteur, vous devez vous connecter directement au locataire sur lequel vous souhaitez le déployer et vous authentifier avec les autorisations nécessaires.

## <a name="next-steps"></a>Étapes suivantes

Ce document vous a permis d’apprendre à gérer plusieurs locataires Azure Sentinel en toute transparence. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).

