---
title: Connecter des données Check Point à Azure Sentinel | Microsoft Docs
description: Configurez Check Point pour transférer les messages Syslog au format CEF à votre espace de travail Azure Sentinel par le biais de l’agent Syslog.
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
ms.openlocfilehash: 32cbdabef4d89c7fabb47a52ebf0589690b7ab33
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85566053"
---
# <a name="connect-check-point-to-azure-sentinel"></a>Connecter Check Point à Azure Sentinel



Cet article explique comment connecter votre appliance Check Point à Azure Sentinel. Le connecteur de données Check Point vous permet de connecter facilement vos journaux Check Point à Azure Sentinel, de consulter des tableaux de bord, de créer des alertes personnalisées et d’améliorer les enquêtes. L’utilisation de Check Point sur Azure Sentinel vous permettra d’obtenir davantage d’informations sur l’utilisation d’Internet au sein de votre organisation et améliorera ses fonctionnalités de sécurité. 

## <a name="forward-check-point-logs-to-the-syslog-agent"></a>Transférer les journaux Check Point à l’agent Syslog

Configurez Check Point pour transférer les messages Syslog au format CEF à votre espace de travail Azure par le biais de l’agent Syslog.

1. Accédez à la section [Check Point Log Export](https://aka.ms/asi-syslog-checkpoint-forwarding) (Exportation de journaux Check Point).
1. Faites défiler la page jusqu’au lien **Basic Deployment** (Déploiement de base) et suivez les instructions pour configurer la connexion, en suivant les recommandations suivantes :
   - Définissez le **port Syslog** sur la valeur **514**, ou sur la valeur du port défini sur l’agent.
     - Remplacez le **nom** et **l’adresse IP du serveur cible** dans la CLI par le nom et l’adresse IP de l’agent Syslog.
     - Définissez le format sur **CEF**.
1. Si vous utilisez la version R77.30 ou R80.10, faites défiler la page jusqu’à la section **Installations** et suivez les instructions pour installer un exportateur de journal correspondant à votre version.
1. Passez à [l’étape 3 : Valider la connectivité](connect-cef-verify.md).
 

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter des appliances Check Point à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- [Valider la connectivité](connect-cef-verify.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.


