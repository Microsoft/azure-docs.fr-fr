---
title: Enregistrements DNS du centre de données Azure utilisés par Azure Automation | Microsoft Docs
description: Cet article fournit les enregistrements DNS qu’exigent les fonctionnalités d’Azure Automation lors de la restriction de la communication à une région Azure spécifique hébergeant ce compte Automation.
services: automation
ms.subservice: process-automation
ms.date: 11/25/2020
ms.topic: conceptual
ms.openlocfilehash: b4b8f48afc75c0a96937575bdad5bb884d0cb4d8
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183717"
---
# <a name="dns-records-for-azure-regions-used-by-azure-automation"></a>Enregistrements DNS pour les régions Azure utilisées par Azure Automation

Le service [Azure Automation](../automation-intro.md) utilise un certain nombre d’enregistrements DNS pour permettre aux fonctionnalités de se connecter au service. Si vous avez un compte Automation défini pour une région spécifique, vous pouvez limiter les communications à ce centre de données régional. Vous devrez peut-être connaître ces enregistrements pour permettre aux fonctionnalités suivantes d’Automation d’opérer quand elles sont hébergées derrière un pare-feu :

* Runbook Worker hybride
* Configuration de l’état
* Webhooks

>[!NOTE]
>L’inscription du Runbook Worker hybride Linux échoue avec les nouveaux enregistrements, sauf s’il s’agit de la version 1.6.10.2 ou d’une version supérieure. Vous devez effectuer une mise à niveau vers une version plus récente de l’[agent Log Analytics pour Linux](../../azure-monitor/platform/agent-linux.md) afin que l’ordinateur reçoive une version à jour du rôle de travail et utilise ces nouveaux enregistrements. Les ordinateurs existants continueront de fonctionner sans problème.  

## <a name="dns-records-per-region"></a>Enregistrements DNS par région

Le tableau suivant indique l’enregistrement DNS pour chaque région.

>[!NOTE]
>Bien que la liste des enregistrements DNS d’Automation fournie ici ait été supprimée, ils restent fonctionnels pour vous permettre de migrer vers les nouveaux enregistrements répertoriés sous [prise en charge de Liaison privée](#support-for-private-link) et d’éviter l’échec de vos processus d’automation.

| **Région** | **Enregistrement DNS** |
| --- | --- |
| Centre de l’Australie |ac-jobruntimedata-prod-su1.azure-automation.net</br>ac-agentservice-prod-1.azure-automation.net |
| Australie Est |ae-jobruntimedata-prod-su1.azure-automation.net</br>ae-agentservice-prod-1.azure-automation.net |
| Sud-Est de l’Australie |ase-jobruntimedata-prod-su1.azure-automation.net</br>ase-agentservice-prod-1.azure-automation.net |
| Centre du Canada |cc-jobruntimedata-prod-su1.azure-automation.net</br>cc-agentservice-prod-1.azure-automation.net |
| Inde centrale |cid-jobruntimedata-prod-su1.azure-automation.net</br>cid-agentservice-prod-1.azure-automation.net |
| USA Est 2 |eus2-jobruntimedata-prod-su1.azure-automation.net</br>eus2-agentservice-prod-1.azure-automation.net |
| Japon Est |jpe-jobruntimedata-prod-su1.azure-automation.net</br>jpe-agentservice-prod-1.azure-automation.net |
| Europe Nord |ne-jobruntimedata-prod-su1.azure-automation.net</br>ne-agentservice-prod-1.azure-automation.net |
| États-Unis - partie centrale méridionale |scus-jobruntimedata-prod-su1.azure-automation.net</br>scus-agentservice-prod-1.azure-automation.net |
| Asie Sud-Est |sea-jobruntimedata-prod-su1.azure-automation.net</br>sea-agentservice-prod-1.azure-automation.net|
| Sud du Royaume-Uni | uks-jobruntimedata-prod-su1.azure-automation.net</br>uks-agentservice-prod-1.azure-automation.net |
| Gouvernement américain - Virginie | usge-jobruntimedata-prod-su1.azure-automation.us<br>usge-agentservice-prod-1.azure-automation.us |
| Centre-USA Ouest | wcus-jobruntimedata-prod-su1.azure-automation.net</br>wcus-agentservice-prod-1.azure-automation.net |
| Europe Ouest |we-jobruntimedata-prod-su1.azure-automation.net</br>we-agentservice-prod-1.azure-automation.net |
| USA Ouest 2 |wus2-jobruntimedata-prod-su1.azure-automation.net</br>wus2-agentservice-prod-1.azure-automation.net |

### <a name="support-for-private-link"></a>Prise en charge de Liaison privée

Pour prendre en charge [Liaison privée](../../private-link/private-link-overview.md) dans Azure Automation, les enregistrements DNS pour chaque centre de données pris en charge ont été mis à jour. Au lieu d’URL spécifiques d’une région, les URL sont spécifiques du compte Automation.

| **Région** | **Enregistrement DNS** |
| --- | --- |
| Centre-USA Ouest |`https://<accountId>.webhook.wcus.azure-automation.net`<br>`https://<accountId>.agentsvc.wcus.azure-automation.net`<br>`https://<accountId>.jrds.wcus.azure-automation.net` |
| USA Ouest |`https://<accountId>.webhook.wus.azure-automation.net`<br>`https://<accountId>.agentsvc.wus.azure-automation.net`<br>`https://<accountId>.jrds.wus.azure-automation.net` |
| USA Ouest 2 |`https://<accountId>.webhook.wus2.azure-automation.net`<br>`https://<accountId>.agentsvc.wus2.azure-automation.net`<br>`https://<accountId>.jrds.wus2.azure-automation.net` |
| USA Centre |`https://<accountId>.webhook.cus.azure-automation.net`<br>`https://<accountId>.agentsvc.cus.azure-automation.net`<br>`https://<accountId>.jrds.cus.azure-automation.net` |
| États-Unis - partie centrale méridionale |`https://<accountId>.webhook.scus.azure-automation.net`<br>`https://<accountId>.agentsvc.scus.azure-automation.net`<br>`https://<accountId>.jrds.scus.azure-automation.net` |
| Centre-Nord des États-Unis |`https://<accountId>.webhook.ncus.azure-automation.net`<br>`https://<accountId>.agentsvc.ncus.azure-automation.net`<br>`https://<accountId>.jrds.ncus.azure-automation.net` |
| USA Est |`https://<accountId>.webhook.eus.azure-automation.net`<br>`https://<accountId>.agentsvc.eus.azure-automation.net`<br>`https://<accountId>.jrds.eus.azure-automation.net` |
| USA Est 2 |`https://<accountId>.webhook.eus2.azure-automation.net`<br>`https://<accountId>.agentsvc.eus2.azure-automation.net`<br>`https://<accountId>.jrds.eus2.azure-automation.net` |
| Centre du Canada |`https://<accountId>.webhook.cc.azure-automation.net`<br>`https://<accountId>.agentsvc.cc.azure-automation.net`<br>`https://<accountId>.jrds.cc.azure-automation.net` |
| Europe Ouest |`https://<accountId>.webhook.we.azure-automation.net`<br>`https://<accountId>.agentsvc.we.azure-automation.net`<br>`https://<accountId>.jrds.we.azure-automation.net` |
| Europe Nord |`https://<accountId>.webhook.ne.azure-automation.net`<br>`https://<accountId>.agentsvc.ne.azure-automation.net`<br>`https://<accountId>.jrds.ne.azure-automation.net` |
| Asie Sud-Est |`https://<accountId>.webhook.sea.azure-automation.net`<br>`https://<accountId>.agentsvc.sea.azure-automation.net`<br>`https://<accountId>.jrds.sea.azure-automation.net` |
| Asie Est |`https://<accountId>.webhook.ea.azure-automation.net`<br>`https://<accountId>.agentsvc.ea.azure-automation.net`<br>`https://<accountId>.jrds.ea.azure-automation.net` |
| Inde centrale |`https://<accountId>.webhook.cid.azure-automation.net`<br>`https://<accountId>.agentsvc.cid.azure-automation.net`<br>`https://<accountId>.jrds.cid.azure-automation.net` |
| Japon Est |`https://<accountId>.webhook.jpe.azure-automation.net`<br>`https://<accountId>.agentsvc.jpe.azure-automation.net`<br>`https://<accountId>.jrds.jpe.azure-automation.net` |
| Centre de la Corée |`https://<accountId>.webhook.kc.azure-automation.net`<br>`https://<accountId>.agentsvc.kc.azure-automation.net`<br>`https://<accountId>.jrds.kc.azure-automation.net` |
| Sud-Est de l’Australie |`https://<accountId>.webhook.ase.azure-automation.net`<br>`https://<accountId>.agentsvc.ase.azure-automation.net`<br>`https://<accountId>.jrds.ase.azure-automation.net` |
| Australie Est |`https://<accountId>.webhook.ae.azure-automation.net`<br>`https://<accountId>.agentsvc.ae.azure-automation.net`<br>`https://<accountId>.jrds.ae.azure-automation.net` |
| Centre de l’Australie |`https://<accountId>.webhook.ac.azure-automation.net`<br>`https://<accountId>.agentsvc.ac.azure-automation.net`<br>`https://<accountId>.jrds.ac.azure-automation.net` |
| Sud du Royaume-Uni |`https://<accountId>.webhook.uks.azure-automation.net`<br>`https://<accountId>.agentsvc.uks.azure-automation.net`<br>`https://<accountId>.jrds.uks.azure-automation.net` |
| France Centre |`https://<accountId>.webhook.fc.azure-automation.net`<br>`https://<accountId>.agentsvc.fc.azure-automation.net`<br>`https://<accountId>.jrds.fc.azure-automation.net` |
| Afrique du Sud Nord |`https://<accountId>.webhook.san.azure-automation.net`<br>`https://<accountId>.agentsvc.san.azure-automation.net`<br>`https://<accountId>.jrds.san.azure-automation.net` |
| Brésil Sud |`https://<accountId>.webhook.brs.azure-automation.net`<br>`https://<accountId>.agentsvc.brs.azure-automation.net`<br>`https://<accountId>.jrds.brs.azure-automation.net` |
| Chine du Nord |`https://<accountId>.webhook.bjb.azure-automation.cn`<br>`https://<accountId>.agentsvc.bjb.azure-automation.cn`<br>`https://<accountId>.jrds.bjb.azure-automation.cn` |
| Chine Nord 2 |`https://<accountId>.webhook.bjs2.azure-automation.cn`<br>`https://<accountId>.agentsvc.bjs2.azure-automation.cn`<br>`https://<accountId>.jrds.bjs2.azure-automation.cn` |
| Chine orientale 2 |`https://<accountId>.webhook.sha2.azure-automation.cn`<br>`https://<accountId>.agentsvc.sha2.azure-automation.cn`<br>`https://<accountId>.jrds.sha2.azure-automation.cn` |
| Gouvernement américain - Virginie |`https://<accountId>.webhook.usge.azure-automation.us`<br>`https://<accountId>.agentsvc.usge.azure-automation.us`<br>`https://<accountId>.jrds.usge.azure-automation.us` |
| Gouvernement des États-Unis – Texas |`https://<accountId>.webhook.ussc.azure-automation.us`<br>`https://<accountId>.agentsvc.ussc.azure-automation.us`<br>`https://<accountId>.jrds.ussc.azure-automation.us` |
| Gouvernement des États-Unis – Arizona |`https://<accountId>.webhook.phx.azure-automation.us`<br>`https://<accountId>.agentsvc.phx.azure-automation.us`<br>`https://<accountId>.jrds.phx.azure-automation.us` |

Remplacez `<accountId>` dans l’enregistrement DNS par le GUID qui représente votre ID de compte Automation dans la valeur **URL**. Vous pouvez récupérer l’ID requis en sélectionnant **Clés** sous **Paramètres de compte** dans le portail Azure.

![Compte Automation - page de la clé primaire](./media/automation-region-dns-records/automation-account-keys.png)

Copiez la valeur après *accounts/* dans le champ **URL** - `https://<GUID>.agentsvc.<region>.azure-automation.net/accounts/<GUID>`

> [!NOTE]
> Tous les enregistrements DNS de Webhook et agentservice ont été mis à jour en fonction du nouveau style d’enregistrement DNS pour prendre en charge Private Link. En ce qui concerne les enregistrements DNS JRDS, l’ancien style et le nouveau style d’enregistrement DNS sont pris en charge. Si vous n’utilisez pas Private Link, vous voyez uniquement l’ancien style d’enregistrement DNS, tandis que ceux qui utilisent Private Link voient le nouveau style d’enregistrement DNS.

Il est recommandé d’utiliser les adresses répertoriées lors de la définition d’[exceptions](../automation-runbook-execution.md#exceptions). Pour obtenir la liste des adresses IP régionales plutôt que celle des noms des régions, téléchargez le fichier JSON à partir du Centre de téléchargement Microsoft pour les environnements cloud suivant :

* [Plages d’adresses IP et étiquettes de service – Azure public](https://www.microsoft.com/download/details.aspx?id=56519)
* [Plages d’adresses IP et étiquettes de service – Azure Government](https://www.microsoft.com/download/details.aspx?id=57063)
* [Plages d’adresses IP et étiquettes de service – Azure Allemagne](https://www.microsoft.com/download/details.aspx?id=57064)
* [Plages d’adresses IP et étiquettes de service – Azure Chine Vianet 21](https://www.microsoft.com/download/details.aspx?id=57062)

Le fichier d’adresses IP répertorie les plages d’adresses IP qui sont utilisées dans les centres de données Microsoft Azure. Il comprend des plages de calcul, SQL et de stockage, et reflète les plages actuellement déployées ainsi que toutes les futures modifications des plages d’adresses IP. Les nouvelles plages figurant dans le fichier ne sont pas utilisées dans les centres de données avant une semaine minimum.

Il est recommandé de télécharger le nouveau fichier d’adresses IP chaque semaine. Ensuite, mettez à jour votre site afin qu’il identifie correctement les services en cours d’exécution dans Azure.

> [!NOTE]
> Si vous utilisez Azure ExpressRoute, n’oubliez pas que le fichier d’adresses IP est utilisé pour mettre à jour la publication BGP (Border Gateway Protocol) de l’espace Azure la première semaine de chaque mois.

## <a name="next-steps"></a>Étapes suivantes

* Pour savoir comment résoudre les problèmes de vos Runbook Workers hybrides, consultez [Résoudre les problèmes liés à la fonctionnalité Runbook Worker hybride](../troubleshoot/hybrid-runbook-worker.md#general).

* Pour savoir comment résoudre les problèmes liés à State Configuration, consultez [Résoudre les problèmes liés à State Configuration](../troubleshoot/desired-state-configuration.md).