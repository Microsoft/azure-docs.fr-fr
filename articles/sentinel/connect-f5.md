---
title: Connecter des données ASM F5 à Azure Sentinel | Microsoft Docs
description: Découvrez comment utiliser le connecteur de données ASM F5 pour tirer vos journaux ASM F5 dans Azure Sentinel. Affichez les données de l’affichage ASM F5 dans des classeurs, créez des alertes et améliorez l’investigation.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 0001cad6-699c-4ca9-b66c-80c194e439a5
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/20/2020
ms.author: yelevin
ms.openlocfilehash: 81a0f0e01827c48518f2eb1fba062e9e6536d9df
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655695"
---
# <a name="connect-f5-asm-to-azure-sentinel"></a>Connecter ASM F5 à Azure Sentinel

Cet article explique comment utiliser le connecteur de données ASM F5 pour tirer (pull) facilement vos journaux ASM F5 dans Azure Sentinel. Cela vous permet d’afficher des données ASM F5 dans des classeurs, de les utiliser pour créer des alertes personnalisées et de les incorporer pour améliorer l’investigation. Les données ASM F5 dans Azure Sentinel vous permettent d'obtenir davantage d'informations sur la sécurité de l’application web au sein de votre organisation et améliorent vos fonctionnalités d’opérations de sécurité. 

## <a name="configure-your-f5-asm-to-send-cef-messages"></a>Configurer votre ASM F5 pour envoyer des messages CEF

1. Suivez les instructions dans [Configuration F5 de la journalisation des événements de sécurité des applications](https://techdocs.f5.com/kb/en-us/products/big-ip_asm/manuals/product/asm-implementations-11-5-0/12.html) et configurez la journalisation à distance, en suivant les lignes directrices ci-dessous :
   - Définissez le **Type de stockage distant** sur **CEF**.
   - Définissez le **Protocole** sur **TCP**.
   - Définissez l'**adresse IP** sur l'adresse IP du serveur Syslog.
   - Définissez le **numéro de port** sur **514** ou le port que vous avez défini pour l'agent.
   - Vous pouvez définir la **Taille maximale de chaîne de requête** sur la taille que vous avez définie dans votre agent.

1. Pour utiliser le schéma pertinent dans Log Analytics pour les événements CEF, recherchez `CommonSecurityLog`.

1. Passez à [l’étape 3 : Valider la connectivité](connect-cef-verify.md).


## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous découvrirez comment connecter ASM F5 à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](./tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.