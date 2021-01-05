---
title: Notification de maintenance planifiée - Azure Database for MariaDB
description: Cet article décrit la fonctionnalité de notification de maintenance planifiée dans Azure Database for MariaDB
author: ambhatna
ms.author: ambhatna
ms.service: mariadb
ms.topic: conceptual
ms.date: 10/21/2020
ms.openlocfilehash: c290236dfe7e88999847f8cb0d66b2d3c868c1ab
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96453288"
---
# <a name="planned-maintenance-notification-in-azure-database-for-mariadb"></a>Notification de maintenance planifiée dans Azure Database for MariaDB

Apprenez à anticiper les événements de maintenance planifiée pour votre base de données Azure Database for MariaDB.

## <a name="what-is-a-planned-maintenance"></a>Qu’est-ce qu’une maintenance planifiée ?

Le service Azure Database for MariaDB effectue une mise à jour corrective automatisée du matériel, du système d’exploitation et du moteur de base de données sous-jacents. Le correctif comprend de nouvelles fonctionnalités de service et de sécurité, et des mises à jour logicielles. Pour le moteur MariaDB, la mise à niveau des versions mineures est automatique et incluse dans le cadre du cycle de mise à jour corrective. Aucune action de l’utilisateur ni aucun paramètre de configuration ne sont requis pour la mise à jour corrective. Le correctif est testé largement et déployé à l’aide de pratiques de déploiement sécurisé.

Une maintenance planifiée est une fenêtre de maintenance lorsque ces mises à jour de service sont déployées sur des serveurs dans une région Azure donnée. Lors de la maintenance planifiée, un événement de notification est créé pour informer les clients lorsque la mise à jour du service est déployée dans la région Azure hébergeant leurs serveurs. La durée minimale entre deux maintenances planifiées est de 30 jours. Vous recevez une notification de la prochaine fenêtre de maintenance 72 heures à l’avance.

## <a name="planned-maintenance---duration-and-customer-impact"></a>Maintenance planifiée : durée et impact sur le client

Une maintenance planifiée pour une région Azure donnée est généralement prévue pour une exécution de 15 heures. La durée de cette fenêtre comprend également une marge pour exécuter un plan de restauration si nécessaire. Pendant la maintenance planifiée, il peut y avoir des redémarrages ou des basculements de serveur de base de données, ce qui peut entraîner une brève indisponibilité des serveurs de base de données pour les utilisateurs finaux. Les serveurs Azure Database for MariaDB s’exécutent dans des conteneurs. Les redémarrages de serveurs de base de données sont donc généralement rapides, et s’exécutent normalement en 60-120 secondes. L’ensemble de l’événement de maintenance planifiée, y compris chaque redémarrage de serveur, est analysé avec soin par l’équipe d’ingénierie. Le temps de basculement du serveur dépend du temps de récupération de la base de données, ce qui peut entraîner un temps de mise en ligne de la base de données plus long si vous avez des activités transactionnelles importantes sur le serveur au moment du basculement. Pour éviter des temps de redémarrage plus longs, il est recommandé d’éviter toute transaction longue (chargement en masse) pendant les événements de maintenance planifiée.

En résumé, bien que l’événement de maintenance planifiée s’exécute pendant 15 heures, l’impact sur un serveur individuel est généralement de 60 secondes en fonction de l’activité transactionnelle sur ce serveur. Une notification est envoyée 72 heures calendaires avant le démarrage de la maintenance planifiée, suivie d’une autre quand la maintenance en cours pour une région donnée.

## <a name="how-can-i-get-notified-of-planned-maintenance"></a>Comment puis-je être averti de la maintenance planifiée ?

Vous pouvez utiliser la fonctionnalité de notifications de maintenance planifiée pour recevoir des alertes pour un événement de maintenance planifiée à venir. Vous recevrez la notification concernant la maintenance à venir 72 heures calendaires avant l’événement et une autre quand la maintenance est en cours pour une région donnée.

### <a name="planned-maintenance-notification"></a>Notification de maintenance planifiée

> [!IMPORTANT]
> Les notifications de maintenance planifiée sont actuellement disponibles en préversion dans toutes les régions, **sauf** USA Centre-Ouest.

Les **notifications de maintenance planifiée** vous permettent de recevoir des alertes concernant l’événement de maintenance planifiée à venir sur Azure Database for MariaDB. Ces notifications sont intégrées à la maintenance planifiée de [Service Health](../service-health/overview.md) et vous permettent d’afficher toutes les tâches de maintenance planifiée de vos abonnements dans un même emplacement. Cela permet également d’adapter la notification au public approprié pour divers groupes de ressources, car des contacts différents peuvent être responsables de différentes ressources. Vous recevez la notification concernant la maintenance à venir 72 heures calendaires avant l’événement.

Nous mettons tout en œuvre pour fournir une **notification de maintenance planifiée** avec un préavis de 72 heures pour tous les événements. Toutefois, en cas de correctifs critiques ou de sécurité, des notifications peuvent être envoyées dans un délai plus proche de l’événement ou être omises.

Vous pouvez consulter la notification de maintenance planifiée sur le portail Azure ou configurer des alertes pour recevoir des notifications. 

### <a name="check-planned-maintenance-notification-from-azure-portal"></a>Vérifier les notifications de maintenance planifiée à partir du portail Azure

1. Dans le [portail](https://portal.azure.com) Azure, sélectionnez **Intégrité du service**.
2. Sélectionnez l’onglet **Maintenance planifiée**
3. Sélectionnez **Abonnement**, puis la **Région et le **Service** pour lesquels vous souhaitez vérifier la notification de maintenance planifiée. 
   
### <a name="to-receive-planned-maintenance-notification"></a>Pour recevoir une notification de maintenance planifiée

1. Dans le [portail](https://portal.azure.com), sélectionnez **Intégrité du service**.
2. Dans la section **Alertes**, sélectionnez **Alertes d’intégrité**.
3. Sélectionnez **+ Ajouter une alerte d’intégrité de service** et renseignez les champs.
4. Renseignez les champs obligatoires. 
5. Choisissez le **type d’événement**, puis sélectionnez **Maintenance planifiée** ou **Tout sélectionner**.
6. Dans **Groupes d’actions**, définissez la manière dont vous souhaitez recevoir l’alerte (obtenir un e-mail, déclencher une application logique, etc.).  
7. Vérifiez que l’activation de la règle lors de la création est définie sur Oui.
8. Sélectionnez **Créer une règle d’alerte** pour terminer votre alerte.

Pour obtenir des instructions détaillées sur la création d’**alertes d’intégrité de service**, consultez [Créer des alertes de journal d’activité sur les notifications de service](../service-health/alerts-activity-log-service-notifications-portal.md).

## <a name="can-i-cancel-or-postpone-planned-maintenance"></a>Puis-je annuler ou reporter la maintenance planifiée ?

La maintenance est nécessaire pour garantir la sécurité, la stabilité et la conformité de votre serveur. L’événement de maintenance planifiée ne peut pas être annulé ou repoussé. Une fois la notification envoyée à une région Azure donnée, il est impossible d’effectuer des modifications du calendrier de mise à jour corrective pour un serveur individuel de cette région. Le correctif est déployé pour toute la région à la fois. Azure Database pour for MariaDB est conçu pour une application cloud native qui ne nécessite pas de contrôle granulaire ou de personnalisation du service.

## <a name="are-all-the-azure-regions-patched-at-the-same-time"></a>Toutes les régions Azure sont-elles mises à jour en même temps ?

Non, toutes les régions Azure reçoivent les correctifs à un moment précis de la fenêtre de déploiement. La fenêtre de déploiement s’étend généralement de 17 h à 8 h heure locale le lendemain dans une région Azure donnée. Les régions Azure jumelées géographiquement reçoivent les correctifs des jours différents. Pour la haute disponibilité et la continuité de l’activité des serveurs de base de données, il est recommandé de tirer parti des [réplicas en lecture inter-régions](./concepts-read-replicas.md#cross-region-replication).

## <a name="retry-logic"></a>Logique de nouvelle tentative

Une erreur temporaire, aussi connue sous le nom de défaut temporaire, est une erreur qui se résout d’elle-même. Des [erreurs temporaires](./concepts-connectivity.md#transient-errors) peuvent se produire pendant la maintenance. La plupart de ces événements sont automatiquement corrigés par le système en moins de 60 secondes. Les erreurs temporaires doivent être gérées à l’aide de la [logique de nouvelle tentative](./concepts-connectivity.md#handling-transient-errors).


## <a name="next-steps"></a>Étapes suivantes

- Pour toute question ou suggestion ayant trait à l’utilisation d’Azure Database for MariaDB, envoyez un e-mail à l’équipe Azure Database for MariaDB à l’adresse AskAzureDBforMariaDB@service.microsoft.com.
- Consultez le [guide pratique pour configurer des alertes](howto-alert-metric.md) pour savoir comment créer une alerte sur une métrique.
- [Résoudre les problèmes de connexion à la base de données Azure Database for MariaDB](howto-troubleshoot-common-connection-issues.md)
- [Gérer les erreurs temporaires et se connecter efficacement à Azure Database for MariaDB](concepts-connectivity.md)