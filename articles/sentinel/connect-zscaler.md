---
title: Connecter des données Zscaler à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter des données Zscaler à Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: d788219e36ac94b1d13b2f4819c3e546622ddff1
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655253"
---
# <a name="connect-zscaler-internet-access-to-azure-sentinel"></a>Connecter Zscaler Internet Access à Azure Sentinel

Cet article explique comment connecter votre appliance Zscaler Internet Access à Azure Sentinel. Le connecteur de données Zscaler vous permet de connecter facilement vos journaux Zscaler Internet Access (ZIA) à Azure Sentinel, de consulter des tableaux de bord, de créer des alertes personnalisées et d’améliorer l’investigation. En utilisant Zscaler sur Azure Sentinel, vous obtenez davantage d’insights sur l’usage d’Internet dans votre organisation, ce qui vous aide à améliorer la sécurité des opérations. 


## <a name="configure-your-zscaler-to-send-cef-messages"></a>Configurer votre appliance Zscaler pour envoyer des messages CEF

1. Sur l’appliance Zscaler, vous devez définir ces valeurs pour que l’appliance envoie les journaux nécessaires dans le format approprié à l’agent Azure Sentinel Syslog, en fonction de l’agent Log Analytics. Vous pouvez modifier ces paramètres dans votre appliance, à condition de les modifier également dans le démon Syslog sur l'agent Azure Sentinel.
    - Protocole = TCP
    - Port = 514
    - Format = CEF
    - Adresse IP : assurez-vous d’envoyer les messages CEF à l’adresse IP de la machine virtuelle dédiée à cet effet.
 Pour plus d’informations, consultez le [Guide de déploiement Zscaler et Azure Sentinel](https://aka.ms/ZscalerCEFInstructions).
 
   > [!NOTE]
   > Cette solution prend en charge Syslog RFC 3164 ou RFC 5424.


1. Pour utiliser le schéma pertinent dans Log Analytics pour les événements CEF, recherchez `CommonSecurityLog`.
1. Passez à [l’étape 3 : Valider la connectivité](connect-cef-verify.md).


## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter Zscaler Internet Access à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](./tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.