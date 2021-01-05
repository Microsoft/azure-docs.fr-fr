---
title: FAQ sur Azure Security Center - Questions sur les agents Log Analytics existants
description: Ces Questions fréquentes (FAQ) et leurs réponses apportent des informations aux clients qui utilisent déjà l’agent Log Analytics et qui envisagent d’utiliser Azure Security Center, produit qui permet de prévenir, détecter et contrer les menaces.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/25/2020
ms.author: memildin
ms.openlocfilehash: 3a2854f4124a570de4fb467eb6a5d49a3214742e
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92341819"
---
# <a name="faq-for-customers-already-using-azure-monitor-logs"></a>FAQ pour les clients qui utilisent déjà les journaux Azure Monitor<a name="existingloganalyticscust"></a>

## <a name="does-security-center-override-any-existing-connections-between-vms-and-workspaces"></a>Est-ce que Security Center peut écraser les connexions existantes entre les machines virtuelles et les espaces de travail ?

Si l’agent Log Analytics est déjà installé sur une machine virtuelle comme une extension Azure, Security Center n’écrase pas la connexion à l’espace de travail existante. En revanche, Security Center utilise l’espace de travail existant. La machine virtuelle est protégée si la solution « Security » ou « SecurityCenterFree » a été installée dans l’espace de travail auquel elle fournit des rapports. 

Une solution de Security Center est installée sur l’espace de travail sélectionné dans l’écran Collection de données, si elle ne l’était pas déjà. La solution est appliquée uniquement aux machines virtuelles appropriées. Lorsque vous ajoutez une solution, elle est déployée automatiquement par défaut sur tous les agents Windows et Linux connectés à votre espace de travail Log Analytics. Le [ciblage de solution](../azure-monitor/insights/solution-targeting.md) permet d’appliquer une étendue à vos solutions.

> [!TIP]
> Si l’agent Log Analytics est directement installé sur la machine virtuelle (et non en tant qu'extension Azure), Security Center n'installe pas l’agent Log Analytics et la surveillance de la sécurité est limitée.

## <a name="does-security-center-install-solutions-on-my-existing-log-analytics-workspaces-what-are-the-billing-implications"></a>Est-ce que Security Center installe des solutions sur mes espaces de travail Log Analytics existants ? Quelles sont les conséquences sur la facturation ?
Lorsque Security Center identifie qu’une machine virtuelle est déjà connectée à un espace de travail que vous avez créé, Security Center active des solutions sur cet espace de travail en fonction de votre configuration tarifaire. Les solutions sont appliquées uniquement aux machines virtuelles Azure pertinentes, via le [ciblage de solution](../azure-monitor/insights/solution-targeting.md), de sorte que la facturation reste la même.

- **Azure Defender désactivé** : Security Center installe la solution « SecurityCenterFree » sur l’espace de travail. Vous ne serez pas facturé.
- **Azure Defender activé** : Security Center installe la solution « sécurité » sur l’espace de travail.

   ![Solutions sur l’espace de travail par défaut](./media/security-center-platform-migration-faq/solutions.png)

## <a name="i-already-have-workspaces-in-my-environment-can-i-use-them-to-collect-security-data"></a>Je dispose déjà d’espaces de travail dans mon environnement. Puis-je les utiliser pour collecter des données de sécurité ?
Si l’agent Log Analytics est déjà installé comme une extension Azure sur une machine virtuelle, Security Center utilise l’espace de travail connecté existant. Une solution de Security Center est installée sur l’espace de travail si elle ne l’était pas déjà. La solution est appliquée uniquement aux machines virtuelles appropriées via le [ciblage de solution](../azure-monitor/insights/solution-targeting.md).

Lorsque le service Security Center installe l’agent Log Analytics sur des machines virtuelles, il utilise le ou les espaces de travail par défaut créés par le service Security Center si celui-ci n’est pas pointé vers un espace de travail existant.

## <a name="i-already-have-security-solution-on-my-workspaces-what-are-the-billing-implications"></a>Je dispose déjà de la solution de sécurité sur mes espaces de travail. Quelles sont les conséquences sur la facturation ?
La solution Security & audit est utilisée pour activer **Azure Defender pour les serveurs**. Si la solution Security & Audit est déjà installée sur un espace de travail, Security Center utilise la solution existante. Il n’y aura aucune modification de la facturation.