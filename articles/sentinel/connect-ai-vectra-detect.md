---
title: Connecter des données d’AI Vectra Detect à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter des données d’AI Vectra Detect à Azure Sentinel.
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
ms.date: 06/21/2020
ms.author: yelevin
ms.openlocfilehash: c57f4a49ac89f1347fc88f8bacddce3abab7e44e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87038220"
---
# <a name="connect-ai-vectra-detect-to-azure-sentinel"></a>Connectez AI Vectra Detect à Azure Sentinel

> [!IMPORTANT]
> Le connecteur de données AI Vectra Detect dans Azure Sentinel est actuellement disponible en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Cet article explique comment connecter votre appliance [AI Vectra Detect](https://www.vectra.ai/product/cognito-detect) à Azure Sentinel. Le connecteur de données AI Vectra Detect vous permet de placer facilement vos données AI Vectra Detect dans Azure Sentinel, ce qui vous permet ensuite de les consulter dans des classeurs, de les utiliser pour créer des alertes personnalisées et de les incorporer pour améliorer l’investigation.

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="configure-your-ai-vectra-detect-appliance-to-send-cef-messages"></a>Configurer votre appliance AI Vectra Detect pour envoyer des messages CEF  

Configurez AI Vectra Detect pour transférer les messages Syslog au format CEF à votre espace de travail Log Analytics via le redirecteur Syslog que vous avez configuré dans [Étape 1 : Déployez le redirecteur de journal](connect-cef-agent.md).

1. À partir de l’interface Vectra, accédez à Paramètres > Notifications, puis choisissez Modifier la configuration Syslog. Suivez les instructions ci-dessous pour configurer la connexion :

    - Ajoutez une nouvelle destination (le nom d’hôte du [redirecteur de journal](connect-cef-agent.md))
    - Définissez le port sur **514**
    - Définissez le protocole sur **UDP**
    - Définissez le format sur **CEF**
    - Définissez les types de journaux (sélectionnez tous les types de journaux disponibles)
    - Dans le menu Paramètres, cliquez sur **Enregistrer**

2. Vous pouvez cliquer sur le bouton **Test** pour forcer l’envoi de certains événements de test au redirecteur de journal.

3. Pour utiliser le schéma approprié dans Log Analytics pour les événements AI Vectra Detect, recherchez **CommonSecurityLog**.

4. Passez à [l’étape 3 : Valider la connectivité](connect-cef-verify.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter des appliances AI Vectra Detect à Azure Sentinel. Pour tirer pleinement parti des fonctionnalités intégrées à ce connecteur de données, cliquez sur l’onglet **Étapes suivantes** de la page du connecteur de données. Vous y trouverez des exemples de requêtes prêtes à l’emploi pour commencer à trouver des informations utiles.

Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.
