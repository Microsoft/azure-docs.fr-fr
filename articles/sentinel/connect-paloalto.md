---
title: Connecter des données Palo Alto Networks à Azure Sentinel | Microsoft Docs
description: Découvrez comment utiliser le connecteur de données Palo Alto Networks pour connecter facilement vos journaux Palo Alto Networks à Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: a4b21d67-1a72-40ec-bfce-d79a8707b4e1
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: 245db436fc3216fe5c8c8f51c50c0ac03190f9eb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85564557"
---
# <a name="connect-palo-alto-networks-to-azure-sentinel"></a>Connecter Palo Alto Networks à Azure Sentinel



Cet article explique comment connecter votre appliance Palo Alto Networks à Azure Sentinel. Le connecteur de données Palo Alto Networks vous permet de connecter facilement vos journaux Palo Alto Networks à Azure Sentinel, de consulter des tableaux de bord, de créer des alertes personnalisées et d’améliorer les enquêtes. L’utilisation de Palo Alto Networks sur Azure Sentinel vous permet d’obtenir davantage d’informations sur l’utilisation d’Internet de votre organisation et améliore ses fonctionnalités de sécurité. 


## <a name="forward-palo-alto-networks-logs-to-the-syslog-agent"></a>Transférer les journaux Palo Alto Networks à l’agent Syslog

Configurez Palo Alto Networks pour transférer les messages Syslog au format CEF à votre espace de travail Azure par le biais de l’agent Syslog :
1.  Accédez à [Common Event Format (CEF) Configuration Guides](https://docs.paloaltonetworks.com/resources/cef) (Guides de configuration CEF (Common Event Format) et téléchargez le PDF correspondant au type de votre appliance. Suivez les instructions qui s’y trouvent pour configurer votre appliance Palo Alto Networks et collecter des événements CEF. 

1.  Accédez à [Configure Syslog monitoring](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/configure-syslog-monitoring) (Configurer la surveillance Syslog) et suivez les étapes 2 et 3 pour configurer l’envoi d’événement CEF depuis votre appliance Palo Alto Networks vers Azure Sentinel.

    1. Veillez à définir le **Syslog server format** (Format du serveur Syslog) sur **BSD**.

       > [!NOTE]
       > Les opérations copier/coller depuis le PDF peuvent modifier le texte et insérer des caractères aléatoires. Pour éviter ce souci, copiez le texte dans un éditeur et supprimez tous caractères qui pourraient poser problème au format du journal avant de les coller, comme montré dans cet exemple.
 
        ![Problème de copie de texte CEF](./media/connect-cef/paloalto-text-prob1.png)

1. Pour utiliser le schéma approprié dans Log Analytics pour les événements Palo Alto Networks, recherchez **CommonSecurityLog**.

1. Passez à [l’étape 3 : Valider la connectivité](connect-cef-verify.md).




## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter les appliances Palo Alto Networks à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.


