---
title: Utilisation d’Azure Sentinel avec Azure Web Application Firewall
description: Cet article explique comment utiliser Azure Sentinel avec Azure Web Application Firewall (WAF).
services: web-application-firewall
author: TreMansdoerfer
ms.service: web-application-firewall
ms.date: 10/12/2020
ms.author: victorh
ms.topic: how-to
ms.openlocfilehash: 525ec334e73ca010d319b40ab864d08dae32f493
ms.sourcegitcommit: 83610f637914f09d2a87b98ae7a6ae92122a02f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91997114"
---
# <a name="using-azure-sentinel-with-azure-web-application-firewall"></a>Utilisation d’Azure Sentinel avec Azure Web Application Firewall

Azure Web Application Firewall (WAF), associé à Azure Sentinel, peut assurer la gestion des événements relatifs aux informations de sécurité pour les ressources WAF. Azure Sentinel fournit une analytique de sécurité grâce à Log Analytics, ce qui vous permet de décomposer et d’afficher facilement vos données WAF. À l’aide de Sentinel, vous pouvez accéder à des classeurs prédéfinis et les modifier pour les adapter aux besoins de votre organisation. Le classeur peut afficher l’analytique pour WAF sur Azure Content Delivery Network (CDN), WAF sur Azure Front Door et WAF sur Application Gateway dans plusieurs abonnements et espaces de travail.

## <a name="waf-log-analytics-categories"></a>Catégories d’analytique des journaux d’activité WAF

L’analytique des journaux d’activité WAF se répartit en plusieurs catégories :  

- Toutes les actions WAF effectuées 
- Les 40 principales adresses URI de requête bloquées 
- Les 50 principaux déclencheurs d’événements  
- Messages dans le temps 
- Détails complets du message 
- Événements d’attaque par messages  
- Événements d’attaque dans le temps 
- Filtre d’ID de suivi 
- Messages d’ID de suivi 
- Les 10 premières adresses IP attaquées 
- Messages d’attaque des adresses IP 

## <a name="waf-workbook-examples"></a>Exemples de classeur WAF

Les exemples de classeurs WAF suivants montrent des exemples de données :

:::image type="content" source="media//waf-sentinel/waf-actions-filter.png" alt-text="Filtre d’actions WAF":::

:::image type="content" source="media//waf-sentinel/top-50-event-trigger.png" alt-text="Filtre d’actions WAF":::

:::image type="content" source="media//waf-sentinel/attack-events.png" alt-text="Filtre d’actions WAF":::

:::image type="content" source="media//waf-sentinel/top-10-attacking-ip-address.png" alt-text="Filtre d’actions WAF":::

## <a name="launch-a-waf-workbook"></a>Lancer un classeur WAF

Le classeur WAF fonctionne pour tous les pare-feu d’applications web Azure Front Door, Application Gateway et CDN. Avant de connecter les données de ces ressources, l’analytique des journaux d’activité doit être activée sur votre ressource. 

Afin d’activer l’analytique des journaux d’activité pour chaque ressource, accédez à votre ressource Azure Front Door, Application Gateway ou CDN :

1. Sélectionnez **Paramètres de diagnostic**.
2. Sélectionnez **+ Ajouter le paramètre de diagnostic**. 
3. Dans la page Paramètres de diagnostic :
   1. Tapez un nom. 
   1. Sélectionnez **Envoyer à Log Analytics**. 
   1. Choisissez l’espace de travail de destination du journal. 
   1. Sélectionnez les types de journaux que vous souhaitez analyser :
      1. Application Gateway : « ApplicationGatewayAccessLog » et « ApplicationGatewayFirewallLog »
      1. Azure Front Door : « FrontDoorAccessLog » et « FrontDoorFirewallLog »
      1. CDN : « AzureCdnAccessLog »
   1. Sélectionnez **Enregistrer**.

   :::image type="content" source="media//waf-sentinel/diagnostics-setting.png" alt-text="Filtre d’actions WAF":::

4. Sur la page d’accueil Azure, saisissez **Azure Sentinel** dans la barre de recherche, puis sélectionnez la ressource **Azure Sentinel**. 
2. Sélectionnez un espace de travail déjà actif ou créez-en un dans Sentinel. 
3. Dans le volet gauche, sous **Configuration**, sélectionnez **Connecteurs de données**.
4. Recherchez **Pare-feu d’applications web Microsoft** et sélectionnez **Pare-feu d’applications web Microsoft (WAF)** . En bas à droite, sélectionnez **Ouvrir la page des connecteurs**.

   :::image type="content" source="media//waf-sentinel/data-connectors.png" alt-text="Filtre d’actions WAF":::

8. Si vous ne l’avez pas déjà fait, suivez les instructions sous **Configuration** pour chaque ressource WAF pour laquelle vous souhaitez obtenir des données d’analytique des journaux d’activité.
6. Une fois que vous avez terminé la configuration des ressources WAF individuelles, sélectionnez l’onglet **Étapes suivantes**. Sélectionnez l’un des classeurs recommandés. Ce classeur utilisera toutes les données d’analytique des journaux d’activité qui ont été activées précédemment. Un classeur WAF fonctionnel doit maintenant exister pour vos ressources WAF.

   :::image type="content" source="media//waf-sentinel/waf-workbooks.png" alt-text="Filtre d’actions WAF":::


## <a name="next-steps"></a>Étapes suivantes

- [En savoir plus sur Azure Sentinel](../sentinel/overview.md)
- [En savoir plus sur les classeurs Azure Monitor](../azure-monitor/platform/workbooks-overview.md)
