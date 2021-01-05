---
title: Connecter des données Illusive Attack Management System à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter des données Illusive Attack Management System à Azure Sentinel.
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
ms.date: 07/17/2020
ms.author: yelevin
ms.openlocfilehash: ea7953be25473357f7ed572fa8b76076edc6f75a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87038115"
---
# <a name="connect-your-illusive-attack-management-system-to-azure-sentinel"></a>Connecter votre solution Illusive Attack Management System à Azure Sentinel

> [!IMPORTANT]
> Le connecteur de données Illusive Attack Management System dans Azure Sentinel est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Cet article explique comment connecter votre solution [Illusive Attack Management System](https://www.illusivenetworks.com/technology/platform/attack-detection-system) à Azure Sentinel. Le connecteur Illusive Attack Management System vous permet de partager les données d’analyse de la surface d’attaque et les journaux d’incident Illusive avec Azure Sentinel et de voir ces informations dans des tableaux de bord dédiés qui vous permettent d’obtenir des insights sur le risque de la surface d’attaque de votre organisation (tableau de bord ADS) et d’effectuer le suivi des mouvements latéraux non autorisés dans le réseau de votre organisation (tableau de bord ADS).

> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="forward-illusive-attack-management-system-logs-to-the-syslog-agent"></a>Transférer les journaux Illusive Attack Management System vers l’agent Syslog  

Configurez la solution Illusive Attack Management System pour transférer les messages Syslog au format CEF à votre espace de travail Azure par le biais de l’agent Syslog.

1. Connectez-vous à la console Illusive, puis accédez à Paramètres-> Création de rapports.

1. Recherchez les serveurs Syslog.

1. Fournissez les informations suivantes :
   - Nom d’hôte : Adresse IP de l’agent Linux Syslog ou nom d’hôte sous forme de nom de domaine complet
   - Port : 514
   - Protocole : TCP
   - Messages d’audit : Envoyer des messages d’audit au serveur

1. Pour ajouter le serveur Syslog, cliquez sur Ajouter.

1. Pour utiliser le schéma approprié dans **Journaux** pour Illusive Attack Management System, recherchez **CommonSecurityLog**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter Illusive Attack Management System à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.
