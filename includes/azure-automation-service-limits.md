---
title: Fichier include
description: Fichier include
services: automation
author: mgoedtel
ms.service: automation
ms.topic: include
ms.date: 10/21/2020
ms.author: magoedte
ms.custom: include file
ms.openlocfilehash: 4a6b21d3782931e3bee4a13ff95e7f313dcc1af2
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92379752"
---
#### <a name="process-automation"></a>Automatisation de processus

| Ressource | Limite |Notes|
| --- | --- |---|
| Nombre maximum de nouveaux travaux pouvant être envoyés toutes les 30 secondes par compte Azure Automation (travaux non planifiés) |100 |Lorsque cette limite est atteinte, les demandes suivantes de création d’un travail échouent. Le client reçoit une réponse d’erreur.|
| Nombre maximum de travaux simultanés pendant la même instance de temps par compte Automation (travaux non planifiés) |200 |Lorsque cette limite est atteinte, les demandes suivantes de création d’un travail échouent. Le client reçoit une réponse d’erreur.|
| Taille maximum de stockage des métadonnées des tâches pour une période continue de 30 jours | 10 Go (environ 4 millions de tâches)|Lorsque cette limite est atteinte, les demandes suivantes de création d’un travail échouent. |
| Limite maximum du flux de travail|1 Mio|Un flux ne peut pas dépasser 1 Mo.|
| Nombre maximum de modules pouvant être importés toutes les 30 secondes par compte Automation |5 ||
| Taille maximum d’un module |100 Mo ||
| Durée d’exécution de la tâche, Niveau Gratuit |500 minutes par abonnement et par mois ||
| Quantité maximum d’espace disque autorisée par bac à sable<sup>1</sup> |1 Go |S’applique aux bacs à sable Azure uniquement.|
| Quantité maximum de mémoire affectée à un bac à sable <sup>1</sup> |400 Mo |S’applique aux bacs à sable Azure uniquement.|
| Nombre maximum de sockets réseau autorisé par bac à sable <sup>1</sup> |1 000 |S’applique aux bacs à sable Azure uniquement.|
| Durée d’exécution maximale autorisée par runbook <sup>1</sup> |3 heures |S’applique aux bacs à sable Azure uniquement.|
| Nombre maximum de comptes Automation dans un abonnement |Aucune limite ||
| Nombre maximum de groupes de Workers hybrides par compte Automation|4 000||
|Nombre maximum de travaux qui peuvent être exécutés simultanément sur un Runbook Worker hybride|50 ||
| Taille maximale des paramètres du travail de runbook   | 512 kilo-octets||
| Nombre maximum de paramètres du runbook   | 50|Vous pouvez passer une chaîne JSON ou XML sur un paramètre et l’analyser avec le runbook si vous atteignez la limite de 50 paramètres.|
| Taille maximale de la charge utile du webhook |  512 kilo-octets|
| Nombre maximal de jours de conservation des données de travail|30 jours|
| Taille maximale de l’état du workflow PowerShell |5 Mo| S’applique aux runbooks de workflow PowerShell durant le contrôle de workflow.|

<sup>1</sup>Un bac à sable est un environnement partagé qui peut être utilisé par plusieurs travaux. Les travaux qui utilisent le même bac à sable sont liés par les limitations de ressources du bac à sable.

#### <a name="change-tracking-and-inventory"></a>Suivi des modifications et inventaire

Le tableau suivant montre les limites des éléments suivis par machine pour Change Tracking.

| **Ressource** | **Limite**| **Remarques** |
|---|---|---|
|Fichier|500||
|Registre|250||
|Logiciels Windows|250|N’inclut pas les mises à jour logicielles.|
|Packages Linux|1250||
|Services|250||
|Daemon|250||

#### <a name="update-management"></a>Update Management

Le tableau suivant indique les limites d’Update Management.

| **Ressource** | **Limite**| **Remarques** |
|---|---|---|
|Nombre de machines par déploiement de mises à jour|1 000||