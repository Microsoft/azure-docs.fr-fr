---
title: Matrice de prise en charge du Centre de sauvegarde
description: Cet article résume les scénarios pris en charge par le Centre de sauvegarde pour chaque type de charge de travail
ms.topic: conceptual
ms.date: 09/07/2020
ms.openlocfilehash: 40582ddc1187082230e2887edcd24b3716811be2
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92173996"
---
# <a name="support-matrix-for-backup-center"></a>Matrice de prise en charge du Centre de sauvegarde

Le Centre de sauvegarde offre aux entreprises une interface unique pour [gérer, surveiller, exploiter et analyser les sauvegardes à grande échelle](backup-center-overview.md). Cet article résume les scénarios pris en charge par le Centre de sauvegarde pour chaque type de charge de travail.

## <a name="supported-scenarios"></a>Scénarios pris en charge

| **Catégorie** | **Scénario**  | **Charges de travail prises en charge**  | **Limites** |
| -------------| ------------- | ----------------------- |------------|
| Surveillance   | Afficher tous les travaux | <li> Machine virtuelle Azure <br><br> <li> Serveur Azure Database pour PostgreSQL | <li> 7 jours de travaux prêts à l'emploi. <br> <li> Chaque filtre/liste déroulante prend en charge un maximum de 1 000 éléments. Le Centre de sauvegarde peut donc être utilisé pour surveiller un maximum de 1 000 abonnements et de 1 000 coffres sur les différents locataires. |
| Surveillance | Afficher toutes les instances de sauvegarde | <li> Machine virtuelle Azure <br><br> <li> Serveur Azure Database pour PostgreSQL | Identique à ce qui précède |
| Surveillance | Afficher toutes les stratégies de sauvegarde | <li> Machine virtuelle Azure <br><br> <li> Serveur Azure Database pour PostgreSQL | Identique à ce qui précède |
| Surveillance | Afficher tous les coffres | <li> Machine virtuelle Azure <br><br> <li> Serveur Azure Database pour PostgreSQL | Identique à ce qui précède |
| Actions | Configurer une sauvegarde | <li> Machine virtuelle Azure <br><br> <li> Serveur Azure Database pour PostgreSQL | Reportez-vous aux matrices de prise en charge pour la [Sauvegarde de machines virtuelles Azure](./backup-support-matrix-iaas.md) et la [Sauvegarde d'un serveur Azure Database pour PostgreSQL](backup-azure-database-postgresql.md#support-matrix) |
| Actions | Restaurer une instance de sauvegarde | <li> Machine virtuelle Azure <br><br> <li> Serveur Azure Database pour PostgreSQL | Reportez-vous aux matrices de prise en charge pour la [Sauvegarde de machines virtuelles Azure](./backup-support-matrix-iaas.md) et la [Sauvegarde d'un serveur Azure Database pour PostgreSQL](backup-azure-database-postgresql.md#support-matrix) |
| Actions | Create vault | <li> Machine virtuelle Azure <br><br> <li> Serveur Azure Database pour PostgreSQL | Reportez-vous aux matrices de prise en charge pour le [coffre Recovery Services](./backup-support-matrix.md#vault-support) |
| Actions | Créer la stratégie de sauvegarde | <li> Machine virtuelle Azure <br><br> <li> Serveur Azure Database pour PostgreSQL | Reportez-vous aux matrices de prise en charge pour le [coffre Recovery Services](./backup-support-matrix.md#vault-support) |
| Actions | Exécuter une sauvegarde à la demande pour une instance de sauvegarde | <li> Machine virtuelle Azure <br><br> <li> Serveur Azure Database pour PostgreSQL | Reportez-vous aux matrices de prise en charge pour la [Sauvegarde de machines virtuelles Azure](./backup-support-matrix-iaas.md) et la [Sauvegarde d'un serveur Azure Database pour PostgreSQL](backup-azure-database-postgresql.md#support-matrix) |
| Actions | Arrêter la sauvegarde pour une instance de sauvegarde | <li> Machine virtuelle Azure <br><br> <li> Serveur Azure Database pour PostgreSQL | Reportez-vous aux matrices de prise en charge pour la [Sauvegarde de machines virtuelles Azure](./backup-support-matrix-iaas.md) et la [Sauvegarde d'un serveur Azure Database pour PostgreSQL](backup-azure-database-postgresql.md#support-matrix) |
| Insights | Afficher les rapports de sauvegarde | <li> Machine virtuelle Azure <br><br> <li> SQL dans une machine virtuelle Azure <br><br> <li> SAP HANA dans une machine virtuelle Azure <br><br> <li> Azure Files <br><br> <li> System Center Data Protection Manager <br><br> <li> Agent Azure Backup (MARS) <br><br> <li> Serveur Azure Backup (MABS) | Reportez-vous aux [scénarios pris en charge pour les Rapports de sauvegarde](./configure-reports.md#supported-scenarios) |
| Gouvernance | Afficher et attribuer des stratégies Azure intégrées et personnalisées sous la catégorie « Sauvegarde » | N/A | N/A |
| Gouvernance | Afficher les sources de données non configurées pour la sauvegarde | <li> Machine virtuelle Azure <br><br> <li> Serveur Azure Database pour PostgreSQL | N/A |

## <a name="unsupported-scenarios"></a>Scénarios non pris en charge

| **Catégorie** | **Scénario**  |
|--------------|---------------|
| Surveillance | Afficher les alertes à grande échelle |
| Actions | Configurer les paramètres de coffre à grande échelle |
| Actions | Exécuter un travail de restauration inter-région à partir du Centre de sauvegarde |

## <a name="next-steps"></a>Étapes suivantes

* [Passer en revue la matrice de prise en charge du service Sauvegarde Azure](./backup-support-matrix.md)
* [Passer en revue la matrice de prise en charge de la sauvegarde des machines virtuelles Azure](./backup-support-matrix-iaas.md)
* [Passer en revue la matrice de prise en charge de la sauvegarde d'un serveur Azure Database pour PostgreSQL](backup-azure-database-postgresql.md#support-matrix)