---
title: Maintenance planifiée - Azure Database pour MySQL - Serveur flexible
description: Cet article décrit la fonction de maintenance planifiée dans Azure Database pour MySQL - Serveur flexible.
author: niklarin
ms.author: nlarin
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: a2e99440a7c8f33eee9d3c9fe2276ac3868ff4b6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91331758"
---
# <a name="scheduled-maintenance-in-azure-database-for-mysql--flexible-server"></a>Maintenance planifiée dans Azure Database pour MySQL – Serveur flexible

Azure Database pour MySQL - Serveur flexible effectue une maintenance périodique pour garantir la sécurité, la stabilité et la mise à jour de votre base de données managée. Au cours de la maintenance, le serveur récupère des fonctionnalités, des mises à jour et des correctifs.

> [!IMPORTANT]
> Azure Database pour MySQL - Serveur flexible est en préversion.

## <a name="select-a-maintenance-window"></a>Sélectionner une fenêtre de maintenance

Vous pouvez planifier la maintenance pour un jour spécifique de la semaine et une fenêtre de temps durant cette journée. Vous pouvez aussi laisser le système choisir un jour et une heure pour vous automatiquement. Dans les deux cas, le système vous alerte cinq jours avant d’exécuter une opération de maintenance. Le système indique également quand la maintenance est démarrée et quand elle est terminée.

Les notifications relatives à la maintenance planifiée à venir peuvent être :

* Envoyées par e-mail à une adresse spécifique.
* Envoyées par e-mail à un rôle Azure Resource Manager.
* Envoyées dans un message texte (SMS) à des appareils mobiles.
* Envoyées en tant que notification à une application Azure.
* Remises sous forme de message vocal.

Quand vous spécifiez des préférences de planification de la maintenance, vous pouvez choisir un jour de la semaine et une fenêtre de temps. Si vous n’en spécifiez pas, le système choisit des heures entre 23h00 et 7h00 dans le fuseau horaire de la région de votre serveur. Vous pouvez définir différentes planifications pour chaque serveur flexible dans votre abonnement Azure.

> [!IMPORTANT]
> Normalement, il faut compter au moins 30 jours entre les événements de maintenance planifiés réussis pour un serveur.
>
> Toutefois, dans le cas d’une mise à jour critique urgente telle qu’une vulnérabilité grave, la fenêtre de notification peut être inférieure à cinq jours. La mise à jour critique peut être appliquée à votre serveur même si une maintenance planifiée réussie a été effectuée au cours des 30 derniers jours.

Vous pouvez mettre à jour les paramètres de planification à tout moment. Si une maintenance est planifiée pour votre serveur flexible et que vous mettez à jour les préférences de planification, l’événement actuel se poursuivra comme prévu et la modification des paramètres de planification prendra effet dès la fin de l’opération.

Dans de rares cas, l’événement de maintenance peut être annulé par le système ou ne peut pas se terminer correctement. Dans ce cas, le système créera une notification sur l’événement de maintenance annulé ou ayant échoué, respectivement. La prochaine tentative d’exécution de la maintenance sera planifiée en fonction des paramètres de planification actuels et vous recevrez une notification cinq jours à l’avance.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [modifier la planification de la maintenance](how-to-maintenance-portal.md).
* Découvrez comment [recevoir des notifications sur la maintenance à venir](../../service-health/service-notifications.md) à l’aide d’Azure Service Health.
* Découvrez comment [configurer des alertes concernant les événements de maintenance planifiée à venir](../../service-health/resource-health-alert-monitor-guide.md).
