---
title: Surveillance de la FAQ sur les alertes et les rapports
description: Dans cet article, découvrez les réponses aux questions les plus fréquentes sur les alertes de supervision Sauvegarde Azure et les rapports de Sauvegarde Azure.
ms.reviewer: srinathv
ms.topic: conceptual
ms.date: 07/08/2019
ms.openlocfilehash: cf6929b9b926a6e6469f3fa789a19e60d5883d21
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89181491"
---
# <a name="azure-backup-monitoring-alert---faq"></a>Alerte de supervision de Sauvegarde Azure - Questions fréquentes (FAQ)

Cet article répond aux questions courantes sur la supervision et la création de rapports avec Sauvegarde Azure.

## <a name="configure-azure-backup-reports"></a>Configurer les rapports de la Sauvegarde Azure

### <a name="how-do-i-check-if-reporting-data-has-started-flowing-into-a-log-analytics-la-workspace"></a>Comment vérifier si les données de rapports ont commencé à arriver dans un espace de travail Log Analytics (LA) ?

Accédez à l’espace de travail LA que vous avez configuré. Accédez à l’élément de menu **Journaux** et exécutez la requête `CoreAzureBackup | take 1`. Si vous voyez un enregistrement retourné, cela signifie que les données ont commencé à arriver dans l’espace de travail. Le Push de données initial peut prendre jusqu’à 24 heures.

### <a name="what-is-the-frequency-of-data-push-to-an-la-workspace"></a>Quelle est la fréquence de Push de données à un espace de travail LA ?

Les données de diagnostic provenant du coffre sont injectées dans l’espace de travail Log Analytics avec un certain décalage. Chaque événement arrive dans l’espace de travail Log Analytics 20 à 30 minutes après son envoi (push) depuis le coffre Recovery Services. Voici plus d’informations sur le décalage :

* Dans toutes les solutions, les alertes intégrées du service de sauvegarde sont envoyées (push) dès qu’elles sont créées. Elles apparaissent donc généralement dans l’espace de travail Log Analytics après 20 à 30 minutes.
* Dans toutes les solutions, les travaux de sauvegarde et les travaux de restauration à la demande sont envoyés (push) dès qu’ils se terminent.
* Pour toutes les solutions, à l’exception de la sauvegarde SQL, les travaux de sauvegarde planifiés sont envoyés (push) dès qu’ils se terminent.
* Pour la sauvegarde SQL, comme des sauvegardes de journal peuvent se produire toutes les 15 minutes, les informations pour tous les travaux de sauvegarde planifiés terminés, y compris les journaux, sont traitées par lot et envoyées (push) toutes les 6 heures.
* Dans toutes les solutions, d’autres informations, comme l’élément de sauvegarde, la stratégie de sauvegarde, les points de récupération de sauvegarde, le stockage de sauvegarde, etc., sont envoyées (push) au moins une fois par jour.
* Une modification dans la configuration de sauvegarde (comme la stratégie de changement ou de modification) déclenche un envoi (push) de toutes les informations de sauvegarde associées.

### <a name="how-long-can-i-retain-reporting-data"></a>Combien de temps puis-je conserver les données des rapports ?

Après avoir créé un espace de travail LA, vous pouvez choisir de conserver les données pendant deux ans maximum. Par défaut, un espace de travail LA conserve les données pendant 31 jours.

### <a name="will-i-see-all-my-data-in-reports-after-i-configure-the-la-workspace"></a>Verrai-je toutes mes données dans des rapports après avoir configuré l’espace de travail LA ?

 Toutes les données générées après la configuration des paramètres de diagnostic sont envoyées (push) à l’espace de travail LA et sont disponibles dans les rapports. Les travaux En cours ne sont pas transmis pour la création de rapports. Une fois que la tâche se termine ou échoue, elle est envoyée aux rapports.

### <a name="can-i-view-reports-across-vaults-and-subscriptions"></a>Puis-je afficher des rapports sur différents coffres et abonnements ?

Oui, vous pouvez afficher des rapports sur différents coffres et abonnements, ainsi que sur différentes régions. Vos données peuvent résider dans un espace de travail LA unique ou dans un groupe d’espaces de travail LA.

### <a name="can-i-view-reports-across-tenants"></a>Puis-je afficher les rapports de différents locataires ?

Si vous êtes un utilisateur [Azure Lighthouse](https://azure.microsoft.com/services/azure-lighthouse/) doté d’un accès délégué aux abonnements ou aux espaces de travail LA de vos clients, vous pouvez utiliser des rapports de sauvegarde pour consulter les données de tous vos locataires.

## <a name="recovery-services-vault"></a>Coffre Recovery Services

### <a name="how-long-does-it-take-for-the-azure-backup-agent-job-status-to-reflect-in-the-portal"></a>Combien de temps faut-il pour que l’état du travail de l’agent Sauvegarde Azure apparaisse dans le portail ?

Une quinzaine de minutes peuvent être nécessaires avant que l’état du travail de l’agent Sauvegarde Azure ne s’affiche dans le portail Azure.

### <a name="when-a-backup-job-fails-how-long-does-it-take-to-raise-an-alert"></a>En cas d’échec d’un travail de sauvegarde, au bout de combien de temps l’alerte est-elle déclenchée ?

Une alerte est générée dans les 20 minutes qui suivent l’échec de Sauvegarde Azure.

### <a name="is-there-a-case-where-an-email-wont-be-sent-if-notifications-are-configured"></a>Est-il possible qu’aucun e-mail ne soit envoyé alors que les notifications sont activées ?

Oui. Dans les cas suivants, les notifications ne sont pas envoyées :

* Si les notifications sont configurées sur une base horaire, et qu’une alerte est déclenchée et résolue dans l’heure
* Si un travail est annulé
* Si un travail de sauvegarde secondaire a échoué, parce que le travail de sauvegarde principal est en cours

## <a name="next-steps"></a>Étapes suivantes

Lire les autres forums aux questions :

* [Questions courantes](backup-azure-vm-backup-faq.md) sur la sauvegarde des machines virtuelles Azure.
* [Questions courantes](backup-azure-file-folder-backup-faq.md) sur l’agent Sauvegarde Azure.
