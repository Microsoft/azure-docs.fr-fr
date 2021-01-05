---
title: Vue d’ensemble des alertes classiques dans Azure Monitor
description: Les alertes classiques sont désormais déconseillées. Les alertes vous permettent d’analyser des mesures de ressources, événements ou journaux d’activité Azure, puis d’envoyer des alertes quand une condition spécifiée est remplie.
ms.topic: conceptual
ms.date: 05/19/2018
ms.subservice: alerts
ms.openlocfilehash: c7dfbd60b7a668b849c3150bb16e40285c345634
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015347"
---
# <a name="what-are-classic-alerts-in-microsoft-azure"></a>Que sont les alertes classiques dans Microsoft Azure ?

> [!NOTE]
> Cet article décrit comment créer des alertes de métriques classiques plus anciennes. Azure Monitor prend désormais en charge de [nouvelles alertes de métrique quasi en temps réel et une nouvelle expérience d’alertes](./alerts-overview.md). Les alertes classiques ne sont [mises hors service](./monitoring-classic-retirement.md), bien qu’elles soient toujours utilisées pour les ressources qui ne prennent pas encore en charge les nouvelles alertes.
>

Azure vous permet de définir des conditions sur des données et d’être informé quand les dernières données de monitoring répondent à ces conditions.

## <a name="old-and-new-alerting-capabilities"></a>Fonctionnalités de génération d’alertes nouvelles et anciennes

Par le passé, Azure Monitor, Application Insights, Log Analytics et Service Health avaient des fonctionnalités de génération d’alertes distinctes. Au fil du temps, Azure a amélioré et combiné l’interface utilisateur et les différentes méthodes de génération d’alertes. La consolidation est toujours en cours.

Vous pouvez afficher des alertes classiques uniquement sur l’écran utilisateur d’alertes classiques dans le Portail Azure. Pour obtenir cet écran, cliquez sur le bouton **Afficher les alertes classiques** de l’écran destiné aux alertes. 

 ![Choix d’alertes dans le Portail Azure](media/alerts-classic.overview/monitor-alert-screen2.png)

La nouvelle expérience utilisateur pour les alertes présente les avantages suivants par rapport à celle des alertes classiques :
- **Meilleur système de notification** : toutes les alertes plus récentes utilisent des groupes d’actions, qui sont des groupes nommés de notifications et d’actions qui peuvent être réutilisés dans plusieurs alertes. Les alertes métriques classiques et les alertes Log Analytics plus anciennes n’utilisent pas de groupes d’actions.
- **Une expérience de création unifiée** : la création de toutes les alertes pour les métriques, les journaux d’activité et le journal d’activité dans Azure Monitor, Log Analytics et Application Insights se fait à partir d’un même emplacement.
- **Affichage des alertes Log Analytics déclenchées dans le portail Azure** : à présent, vous pouvez également afficher les alertes Log Analytics déclenchées dans votre abonnement. Auparavant, elles se trouvaient dans un portail distinct.
- **Séparation des alertes déclenchées et des règles d’alerte** : les règles d’alerte (c’est-à-dire la définition des conditions qui déclenchent des alertes) et les alertes déclenchées (c’est-à-dire les instances d’activation de règles d’alerte) sont différenciées de manière à afficher de façon distincte les opérations et la configuration.
- **Un flux de travail amélioré** : la nouvelle expérience de création Alertes guide l’utilisateur tout au long du processus de configuration d’une règle d’alerte, ce qui facilite la découverte des éléments appropriés aux alertes.
- **Consolidation des alertes intelligentes** et **définition d’état d’alerte** : les alertes plus récentes incluent une fonctionnalité de regroupement automatique qui montre des alertes similaires ensemble pour réduire la surcharge de l’interface utilisateur. 

Les alertes de métrique plus récentes présentent les avantages suivants par rapport aux alertes de métrique classiques :
- **Latence améliorée** : les dernières alertes métriques peuvent être exécutées toutes les minutes. Les alertes métriques les plus anciennes s’exécutent toujours à une fréquence de 5 minutes. Pour les alertes plus récentes, le délai entre le moment où le problème survient et la notification ou l’action a été raccourci (de 3 à 5 minutes). Pour les alertes plus anciennes, comptez 5 à 15 minutes, en fonction du type.  Le délai des alertes de journal est généralement de 10 à 15 minutes en raison du temps nécessaire pour ingérer les journaux d’activité, mais les méthodes de traitement plus récentes permettent de réduire cette durée. 
- **Prise en charge des métriques multidimensionnelles** : vous pouvez définir des alertes sur des métriques dimensionnelles, ce qui vous permet d’analyser un segment intéressant de la métrique.
- **Contrôle renforcé des conditions des métriques** : vous pouvez définir des règles d’alerte plus riches. Les alertes plus récentes prennent en charge la surveillance des valeurs maximales, minimales, moyennes et totales des métriques.
- **Monitoring combiné de plusieurs métriques** : vous pouvez surveiller plusieurs métriques (actuellement jusqu’à deux) avec une seule règle. Une alerte est déclenchée si les deux métriques violent leurs seuils respectifs durant la période spécifiée.
- **Meilleur système de notification** : toutes les nouvelles alertes utilisent des [groupes d’actions](./action-groups.md), à savoir des groupes nommés de notifications et d’actions qui peuvent être réutilisés dans plusieurs alertes.  Les alertes métriques classiques et les alertes Log Analytics plus anciennes n’utilisent pas de groupes d’actions. 
- **Métriques issues des journaux d’activité** (préversion publique) : il est maintenant possible d’extraire les données des journaux destinées à Log Analytics, de les convertir en métriques Azure Monitor et de définir des alertes comme pour les autres métriques. Consultez la section [Taxonomie des alertes (classiques)]() pour découvrir la terminologie spécifique aux alertes classiques. 


## <a name="classic-alerts-on-azure-monitor-data"></a>Alertes classiques pour des données Azure Monitor
Deux types d’alertes classiques sont disponibles : les alertes de métriques et les alertes de journal d’activité.

* **Alertes de métrique classiques** : ces alertes se déclenchent quand la valeur d’une métrique spécifiée dépasse le seuil défini. L’alerte génère une notification quand le seuil est atteint et que la condition d’alerte est remplie. À ce stade, l’alerte est considérée comme « Activée ». Elle génère une autre notification quand elle est « résolue », c’est-à-dire quand le seuil est de nouveau atteint et que la condition n’est plus remplie.

* **Alertes classiques du journal d’activité** : alertes du journal de streaming qui se déclenchent lorsqu’une entrée d’événement du journal d’activité correspond à vos critères de filtre. Ces alertes ne possèdent que l’état « Activé ». Le moteur d’alerte ne fait qu’appliquer les critères de filtre aux nouveaux événements. Il ne recherche pas d’anciennes entrées. Ces alertes peuvent vous informer quand un nouvel incident d’état du service se produit ou quand un utilisateur ou une application effectue une opération dans votre abonnement (par exemple, « Supprimer la machine virtuelle »).

Pour les données des journaux de ressources disponibles dans Azure Monitor, acheminez les données dans Log Analytics et utilisez une alerte de requête de journal. Log Analytics utilise maintenant la [nouvelle méthode d’alerte](./alerts-overview.md) 

Le diagramme suivant récapitule les sources de données d’Azure Monitor, et explique comment créer des alertes à partir de ces données.

![Alertes expliquées](media/alerts-classic.overview/Alerts_Overview_Resource_v5.png)

## <a name="taxonomy-of-alerts-classic"></a>Taxonomie des alertes (classiques)
Azure utilise les termes suivants pour décrire les alertes classiques et leurs fonctions :
* **Alerte** : définition de critères (une ou plusieurs règles ou conditions) qui est activée lorsque les critères sont remplis.
* **Actif** : état indiquant que les critères définis par une alerte classique sont remplis.
* **Résolu** : état indiquant que les critères définis par une alerte classique ne sont plus remplis après l’avoir été.
* **Notification** : action entreprise quand une alerte classique devient active.
* **Action** : appel émis vers le destinataire d’une notification (par exemple, en envoyant un e-mail ou en publiant une URL Webhook). Les notifications peuvent généralement déclencher plusieurs actions.

## <a name="how-do-i-receive-a-notification-from-an-azure-monitor-classic-alert"></a>Comment recevoir une notification d’une alerte classique Azure Monitor ?
Auparavant, les alertes Azure des différents services utilisaient leurs propres méthodes de notification intégrées. 

Azure Monitor a créé un regroupement de notifications réutilisable appelé *groupe d’actions*. Les groupes d’actions spécifient un ensemble de destinataires pour une notification. Chaque fois qu’une alerte est activée, qui référence le groupe d’actions, tous les destinataires reçoivent la notification. Les groupes d’actions vous permettent de réutiliser un groupe de destinataires (par exemple, votre liste d’ingénieurs d’astreinte) pour plusieurs objets d’alerte. Les groupes d’actions prennent en charge les notifications via la publication d’une URL de webhook, en plus des notifications par adresse e-mail, numéro de SMS et plusieurs autres actions.  Pour plus d’informations, consultez [Créer et gérer des groupes d’actions sur le Portail Azure](./action-groups.md). 

Les alertes de journal d’activité classiques plus anciennes utilisent des groupes d’actions.

Toutefois, les anciennes alertes de métrique n’utilisent pas les groupes d’actions. Au lieu de cela, vous pouvez configurer les actions suivantes : 
- envoyer des notifications par courrier électronique à l’administrateur de service, aux coadministrateurs ou aux adresses e-mail supplémentaires que vous spécifiez ;
- appeler un webhook, qui permet de lancer des actions d’automatisation supplémentaires.

Les webhooks autorisent l’automatisation et la correction avec, par exemple :
- Runbook Azure Automation
- Fonction Azure
- Application logique Azure
- Un service tiers

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez obtenir des informations sur les règles d’alerte et leur configuration avec :

* En savoir plus sur les [métriques](data-platform.md)
* Configurer les [alertes métriques classiques via le Portail Azure](alerts-classic-portal.md)
* Configurer les [alertes métriques classiques PowerShell](alerts-classic-portal.md)
* Configurer [l’interface de ligne de commande (CLI) des alertes métriques classiques](alerts-classic-portal.md)
* Configurer les [l’API REST Azure Monitor des alertes métriques](/rest/api/monitor/alertrules)
* En savoir plus sur les [journaux d’activité](platform-logs-overview.md)
* Configurer les [alertes de journal d’activité par le biais du portail Azure](activity-log-alerts.md)
* Configurer les [alertes de journal d’activité au moyen de Resource Manager](alerts-activity-log.md)
* Consulter le [schéma de webhook d’alerte de journal d’activité](activity-log-alerts-webhook.md)
* En savoir plus sur les [groupes d’actions](action-groups.md)
* Configurer des [alertes plus récentes](alerts-metric.md)

