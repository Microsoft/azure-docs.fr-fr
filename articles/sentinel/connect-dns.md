---
title: Connecter des données DNS dans Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter un serveur de noms de domaine (DNS) s’exécutant sur Windows à Azure Sentinel en installant un agent sur l’ordinateur DNS.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 77af84f9-47bc-418e-8ce2-4414d7b58c0c
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/24/2019
ms.author: yelevin
ms.openlocfilehash: 7701fc6d90fd9ebc7ec29f0ffdd7d050c58c036c
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655661"
---
# <a name="connect-your-domain-name-server"></a>Connecter votre serveur de noms de domaine

> [!IMPORTANT]
> Le connecteur de données DNS dans Azure Sentinel est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Vous pouvez vous connecter n’importe quel serveur DNS (Domain Name Server) s’exécutant sous Windows à Azure Sentinel. Pour cela, vous devez installer un agent sur l’ordinateur DNS. À l’aide des journaux DNS, vous pouvez obtenir des informations sur la sécurité, les performances et les opérations dans l’infrastructure DNS de votre organisation, en collectant, analysant et mettant en corrélation des analyses et des journaux d’audit, ainsi que d’autres des données connexes des serveurs DNS.

Lorsque vous activez la connexion du journal DNS, vous pouvez :
- Identifier les clients qui tentent de résoudre des noms de domaine malveillant
- Identifier les enregistrements de ressources obsolètes
- Identifier les noms de domaine fréquemment interrogés et les clients DNS communiquant
- Afficher la charge de la demande sur les serveurs DNS
- Afficher les échecs d’inscription DNS dynamique

## <a name="connected-sources"></a>Sources connectées

Le tableau suivant décrit les sources connectées qui sont prises en charge par cette solution :

| **Source connectée** | **Support** | **Description** |
| --- | --- | --- |
| [Agents Windows](../azure-monitor/platform/agent-windows.md) | Oui | La solution collecte des informations DNS auprès d’agents Windows. |
| [Agents Linux](../azure-monitor/learn/quick-collect-linux-computer.md) | Non | La solution ne collecte aucune information DNS auprès d’agents Linux directs. |
| [Groupe d’administration de Microsoft System Center Operations Manager](../azure-monitor/platform/om-agents.md) | Oui | La solution collecte des informations DNS auprès d’agents dans un groupe d’administration d’Operations Manager connecté. Une connexion directe entre l’agent Operations Manager et Azure Monitor n’est pas obligatoire. Les données sont transférées du groupe d’administration à l’espace de travail Log Analytics. |
| [Compte Azure Storage](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace) | Non | Le stockage Azure n’est pas utilisé par la solution. |

### <a name="data-collection-details"></a>Détails sur la collecte de données

La solution collecte des données liées aux inventaires DNS et aux événements DNS sur les serveurs DNS sur lesquels un agent Log Analytics est installé. Les données liées aux inventaires, comme le nombre de serveurs, de zones et d’enregistrements de ressources DNS, sont collectées en exécutant les applets de commande PowerShell de DNS. Les données sont mises à jour tous les deux jours. Les données liées aux événements sont collectées quasiment en temps réel à partir des [journaux d’analyse et d’audit](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn800669(v=ws.11)#enhanc) fournis par le biais de la fonctionnalité améliorée de diagnostics et de journalisation DNS de Windows Server 2012 R2.


## <a name="connect-your-dns-appliance"></a>Connexion à votre appliance DNS

1. Dans le portail Azure Sentinel, sélectionnez **Connecteurs de données** et choisissez la vignette **DNS (préversion)** .
1. Si vos ordinateurs DNS se trouvent dans Azure :
    1. Cliquez sur **Installer l’agent sur la machine de virtuelle Windows Azure**.
    1. Dans la liste **Machines virtuelles**, sélectionnez l’ordinateur DNS que vous souhaitez diffuser en continu dans Azure Sentinel. Assurez-vous qu’il s’agit d’une machine virtuelle Windows.
    1. Dans la fenêtre qui s’ouvre pour cette machine virtuelle, cliquez sur **Connecter**.  
    1. Cliquez sur **Activer** dans la fenêtre **Connecteur DNS**. 

2. Si votre ordinateur DNS n’est pas une machine virtuelle Azure :
    1. Cliquez sur **Installer l’agent sur les machines non Azure**.
    1. Dans la fenêtre **Agent direct**, sélectionnez **Télécharger l’agent Windows (64 bits)** ou **Télécharger l’agent Windows (32 bits)** .
    1. Installez l’agent sur votre ordinateur DNS. Copiez l’**ID de l’espace de travail**, la **clé primaire** et la **clé secondaire**, et utilisez-les lorsque vous y êtes invité pendant l’installation.

3. Pour utiliser le schéma pertinent dans Log Analytics pour les journaux DNS, recherchez **DnsEvents**.

## <a name="validate"></a>Valider 

Dans Log Analytics, recherchez le schéma **DnsEvents** et assurez-vous qu’il existe des événements.

## <a name="troubleshooting"></a>Dépannage

Si les requêtes de recherche ne s’affichent pas dans Azure Sentinel, effectuez les étapes suivantes pour qu’elles s’affichent correctement :
1. Activez les [journaux DNS Analytics sur vos serveurs](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn800669(v=ws.11)).
2. Assurez-vous que des événements apparaissent sous DNSEvents dans votre liste de collections Log Analytics.
3. Activez [Azure DNS Analytics](../azure-monitor/insights/dns-analytics.md).
4. Dans Azure DNS Analytics, sous **Configuration**, modifiez les paramètres souhaités, enregistrez-les, puis rétablissez-les si nécessaire et enregistrez-les à nouveau.
5. Vérifiez que les requêtes s’affichent à présent dans Azure DNS Analytics.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter les appliances locales DNS à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).