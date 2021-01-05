---
title: Intégrer des journaux à ArcSight à l’aide d’Azure Monitor | Microsoft Docs
description: Découvrez comment intégrer des journaux d’activité Azure Active Directory à ArcSight à l’aide d’Azure Monitor
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: b37bef0d-982e-4e28-86b2-6c61ca524ae1
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/19/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: fc7bcab04da005fd0d46d18e7b708dcb1c9d58e5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89230515"
---
# <a name="integrate-azure-active-directory-logs-with-arcsight-using-azure-monitor"></a>Intégrer des journaux d’activité Azure Active Directory à ArcSight à l’aide d’Azure Monitor

[Micro Focus ArcSight](https://software.microfocus.com/products/siem-security-information-event-management/overview) est une solution SIEM (Security Information and Event Management) qui vous permet de détecter les menaces de sécurité dans votre plateforme et d’y remédier. Vous pouvez désormais acheminer les journaux d’activité Azure Active Directory (Azure AD) vers ArcSight avec Azure Monitor à l’aide du connecteur ArcSight pour Azure AD. Cette fonctionnalité vous permet de surveiller votre locataire et de détecter les éléments en compromettant la sécurité à l’aide d’ArcSight.  

Dans cet article, vous allez apprendre à router les journaux d’activité Azure AD vers ArcSight à l’aide d’Azure Monitor. 

## <a name="prerequisites"></a>Prérequis

Pour utiliser cette fonctionnalité, vous avez besoin des éléments suivants :
* Un hub d’événements Azure contenant les journaux d’activité d’Azure AD. Découvrez comment [diffuser en continu vos journaux d’activité sur un hub d’événements](./tutorial-azure-monitor-stream-logs-to-event-hub.md). 
* Une instance configurée d’ArcSight Syslog NG Daemon SmartConnector (SmartConnector) ou ArcSight Load Balancer. Si les événements sont envoyés à ArcSight Load Balancer, ils sont ensuite envoyés SmartConnector par l’équilibreur de charge.

Téléchargez et ouvrez le [guide de configuration d’ArcSight SmartConnector pour l’Event Hub Azure Monitor](https://community.microfocus.com/t5/ArcSight-Connectors/SmartConnector-for-Microsoft-Azure-Monitor-Event-Hub/ta-p/1671292). Ce guide décrit les étapes à suivre pour installer et configurer ArcSight SmartConnector pour Azure Monitor. 

## <a name="integrate-azure-ad-logs-with-arcsight"></a>Intégrer les journaux d’activité Azure AD à ArcSight

1. Tout d’abord, suivez les étapes de la section **Conditions préalables** du guide de configuration. Cette section comprend les étapes suivantes :
    * Définissez les autorisations utilisateur dans Azure, afin de vous assurer qu’un utilisateur a le rôle **propriétaire** pour déployer et configurer le connecteur.
    * Ouvrez les ports sur le serveur avec Syslog NG Daemon SmartConnector, afin qu’il soit accessible depuis Azure. 
    * Le déploiement exécute un script Windows PowerShell, vous devez donc autoriser PowerShell à exécuter des scripts sur l’ordinateur où vous souhaitez déployer le connecteur.

2. Suivez les étapes de la section **Déploiement du connecteur** du guide de configuration pour déployer le connecteur. Cette section vous explique comment télécharger et extraire le connecteur, configurer les propriétés de l’application et exécuter le script de déploiement dans le dossier extrait. 

3. Utilisez les étapes décrites dans **Vérification du déploiement dans Azure** pour vous assurer que le connecteur est configuré et fonctionne correctement. Vérifiez les éléments suivants :
    * Les fonctions Azure requises sont créées dans votre abonnement Azure.
    * Les journaux d’activité Azure AD sont diffusés en continu vers la destination correcte. 
    * Les paramètres d’application de votre déploiement sont conservés dans les paramètres d’application d’Azure Function Apps. 
    * Un nouveau groupe de ressources pour ArcSight est créé dans Azure, avec une application Azure AD pour le connecteur ArcSight et des comptes de stockage contenant les fichiers mappés au format CEF.

4. Enfin, effectuez les étapes post-déploiement décrites dans la section **Configurations post-déploiement** du guide de configuration. Cette section explique comment effectuer une configuration supplémentaire si vous utilisez un plan App Service pour empêcher que les applications de fonction deviennent inactives après un certain délai d’attente, configurer la diffusion en continu des journaux de ressources à partir du hub d’événements et mettre à jour le certificat de magasins de clés SysLog NG Daemon SmartConnector pour l’associer avec le compte de stockage nouvellement créé.

5. Le guide de configuration explique également comment personnaliser les propriétés du connecteur dans Azure, et comment le mettre à niveau et le désinstaller. Il contient également une section sur les améliorations des performances, notamment sur la mise à niveau vers un [plan de consommation Azure](https://azure.microsoft.com/pricing/details/functions) et la configuration d’ArcSight Load Balancer si la charge de l’événement est supérieure à ce qu’un seul Syslog NG Daemon SmartConnector peut gérer.

## <a name="next-steps"></a>Étapes suivantes

[Guide de configuration d’ArcSight SmartConnector pour l’Event Hub Azure Monitor](https://community.microfocus.com/t5/ArcSight-Connectors/SmartConnector-for-Microsoft-Azure-Monitor-Event-Hub/ta-p/1671292)