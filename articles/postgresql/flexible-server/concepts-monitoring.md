---
title: Supervision et métriques - Azure Database pour PostgreSQL - Serveur flexible
description: Cet article décrit les fonctionnalités de supervision et de métriques disponibles dans Azure Database pour PostgreSQL - Serveur flexible.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: 1519e0b5cef6055cf8d8b0aded0d8ad323d548a2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91707844"
---
# <a name="monitor-metrics-on-azure-database-for-postgresql---flexible-server"></a>Superviser les métriques dans Azure Database pour PostgreSQL - Serveur flexible

> [!IMPORTANT]
> Azure Database pour PostgreSQL – Serveur flexible est en préversion

La surveillance des données relatives à vos serveurs vous aide à résoudre les problèmes et à optimiser votre charge de travail. Azure Database pour PostgreSQL propose différentes options de supervision pour fournir un insight sur le comportement de votre serveur.

## <a name="metrics"></a>Mesures
Azure Database pour PostgreSQL propose diverses métriques qui donnent des informations sur le comportement des ressources prenant en charge le serveur MySQL. Chaque métrique est émise selon une fréquence d’une minute et est conservée jusqu’à [93 jours dans l’historique](../../azure-monitor/platform/data-platform-metrics.md#retention-of-metrics). Vous pouvez configurer des alertes basées sur les métriques. Les autres options incluent la configuration d’actions automatisées, l’exécution d’analytique avancée et l’archivage de l’historique. Pour plus d’informations, consultez [Vue d’ensemble des mesures dans Microsoft Azure](../../azure-monitor/platform/data-platform-metrics.md).

### <a name="list-of-metrics"></a>Liste des métriques
Les métriques suivantes sont disponibles pour le serveur flexible PostgreSQL :


|Métrique|Nom d’affichage de la métrique|Unité|Description|
|---|---|---|---|
| active_connections | Connexions actives | Count | Le nombre de connexions à votre serveur. | 
| backup_storage_used | Stockage de sauvegarde utilisé | Octets | Quantité de stockage de sauvegarde utilisée. La métrique représente le total du stockage consommé par l’ensemble des sauvegardes de base de données complètes, sauvegardes différentielles et sauvegardes de journaux conservées en fonction de la période de rétention de sauvegarde définie pour le serveur. La fréquence des sauvegardes est gérée par le service. Pour le stockage géo-redondant, l’utilisation du stockage de sauvegarde est le double de celle du stockage localement redondant. |
| connections_failed | Connexions ayant échoué | Count | Connexions ayant échoué. |
| connections_succeeded | Connexions ayant abouti | Count | Connexions ayant abouti. |
| cpu_credits_consumed | Crédits de processeur consommés | Count | Nombre de crédits utilisés par le serveur flexible. Applicable au niveau Expansible. |
| cpu_credits_remaining | Crédits de processeurs restants | Count | Nombre total de crédits disponibles pour l’expansion. Applicable au niveau Expansible. |
| cpu_percent | Pourcentage d’UC | Pourcentage | Pourcentage de processeur en cours d’utilisation. | 
| disk_queue_depth | Longueur de file d’attente de disque | Count | Nombre d’opérations d’E/S en attente sur le disque de données. |
| iops | E/S par seconde | Count | Nombre d’opérations d’E/S sur disque par seconde. |
| maximum_used_transactionIDs | Nombre maximal d’ID de transaction utilisés | Count | ID de transaction maximal en cours d’utilisation. |
| memory_percent | Pourcentage de mémoire | Pourcentage | Pourcentage de mémoire en cours d’utilisation. |
| network_bytes_egress | Network Out | Octets | Quantité de trafic réseau sortant. |
| network_bytes_ingress | Network In | Octets | Quantité de trafic réseau entrant. |
| read_iops | E/S par seconde en lecture | Nombre | Nombre d’opérations de lecture d’E/S de disque de données par seconde. |
| read_throughput | Débit de lecture | Octets | Octets lus par seconde à partir du disque. |
| storage_free | Espace de stockage libre | Octets | Quantité d’espace de stockage disponible. |
| storage_percent | Pourcentage de stockage | Pourcentage | Pourcentage d’espace de stockage utilisé. Le stockage utilisé par le service peut inclure les fichiers de base de données, les journaux d’activité des transactions et les journaux d’activité du serveur.|
| storage_used | Stockage utilisé | Octets | Pourcentage d’espace de stockage utilisé. Le stockage utilisé par le service peut inclure les fichiers de base de données, les journaux d’activité des transactions et les journaux d’activité du serveur. |
| txlogs_storage_used | Stockage des journaux des transactions utilisé | Octets | Quantité d’espace de stockage utilisée par les journaux des transactions. | 
| write_throughput | Débit d’écriture | Octets | Octets écrits par seconde sur disque. |
| write_iops | E/S par seconde en écriture | Nombre | Nombre d’opérations d’écriture d’E/S de disque de données par seconde. |

## <a name="server-logs"></a>Journaux d’activité du serveur
Azure Database pour PostgreSQL vous permet de configurer et d’accéder aux journaux standard de Postgres. Pour en savoir plus sur les journaux, consultez la [documentation sur les concepts de journalisation](concepts-logging.md).
