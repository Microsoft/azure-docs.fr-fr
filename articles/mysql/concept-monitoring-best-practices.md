---
title: Meilleures pratiques de monitoring – Azure Database pour MySQL
description: Cet article décrit les meilleures pratiques de monitoring d’une base de données Azure Database pour MySQL.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 11/23/2020
ms.openlocfilehash: 59301e26f4d42056322ba5a7cdaff1555c531096
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96354857"
---
# <a name="best-practices-for-monitoring-azure-database-for-mysql--single-server"></a>Meilleures pratiques de monitoring d’Azure Database pour MySQL – Serveur unique

Découvrez les meilleures pratiques que vous pouvez appliquer pour superviser les opérations de votre base de données et éviter que le niveau de performance ne soit compromis au fil de l’augmentation de la taille des données. Au fur et à mesure que nous ajouterons de nouvelles fonctionnalités à la plateforme, nous continuerons d’affiner les meilleures pratiques décrites dans cette section.

## <a name="layout-of-the-current-monitoring-toolkit"></a>Structure du kit de ressources de monitoring actuel

Azure Database pour MySQL offre des outils et des méthodes permettant de superviser facilement l’utilisation d’une base de données, d’y ajouter ou d’en supprimer des ressources (processeur, mémoire ou E/S), de résoudre ses problèmes potentiels et d’améliorer son niveau de performance. Vous pouvez [suivre régulièrement les métriques de performances](concepts-monitoring.md#metrics) pour voir les valeurs moyennes, maximales et minimales sur différents intervalles de temps.

Vous pouvez [configurer des alertes](howto-alert-on-metric.md#create-an-alert-rule-on-a-metric-from-the-azure-portal) pour un seuil de métrique, afin de recevoir une notification si le serveur atteint ces limites et de prendre les mesures appropriées.  

Supervisez le serveur de base de données pour veiller à ce que les ressources affectées à la base de données puissent gérer la charge de travail applicative. Si la base de données atteint ses limites de ressources, envisagez les opérations suivantes :
    * Identifier et optimiser les principales requêtes consommatrices de ressources. 
    * Ajouter plus de ressources par une mise à niveau du niveau de service.

### <a name="cpu-utilization"></a>Utilisation du processeur
Supervisez l’utilisation du processeur et observez si la base de données épuise les ressources associées. Si l’utilisation du processeur est supérieure ou égale à 90 %, effectuez un scale-up du calcul en accroissant le nombre de vCore ou passez au niveau tarifaire suivant.  Vérifiez que vous obtenez le débit et la concurrence attendus en cas de scale-up ou de scale-down du processeur. 

### <a name="memory"></a>Mémoire 
La quantité de mémoire disponible pour le serveur de base de données est proportionnelle au [nombre de vCore](concepts-pricing-tiers.md). Assurez-vous que la mémoire est suffisante pour la charge de travail. Testez la charge de votre application afin de vérifier que la mémoire est suffisante pour les opérations de lecture et d’écriture. Si la consommation de mémoire de la base de données dépasse fréquemment un seuil défini, c’est le signe que vous devez mettre à niveau votre instance en augmentant le nombre de vCore ou en passant à un niveau de performance supérieur. Utilisez le [Magasin des requêtes](concepts-query-store.md) et les [recommandations relatives aux performances des requêtes](concepts-performance-recommendations.md) pour identifier les requêtes les plus longues et les plus souvent exécutées. Explorez les opportunités d’optimisation. 

### <a name="storage"></a>Stockage 
C’est la [quantité de stockage](howto-create-manage-server-portal.md#scale-compute-and-storage) provisionnée pour le serveur MySQL qui détermine ses IOPS. Le stockage utilisé par le service inclut les fichiers de base de données, les journaux des transactions, les journaux du serveur et les instantanés de sauvegarde. Veillez à ce que l’espace disque consommé ne dépasse pas constamment 85 % de l’espace disque total provisionné, auquel cas vous devrez supprimer ou archiver des données du serveur de base de données pour libérer de l’espace. 

### <a name="network-traffic"></a>Trafic réseau 

**Débit de réception du réseau, débit de transmission réseau** : débit du trafic réseau en provenance et à destination de l’instance MySQL en mégabits par seconde. Vous devez évaluer le débit requis pour le serveur et superviser en permanence le trafic si le débit est moins élevé que prévu. 

### <a name="database-connections"></a>Connexions de base de données 
**Connexions de base de données** : le nombre de sessions clientes connectées à Azure Database pour MySQL doit respecter les [limites de connexion associées à la taille de la référence SKU sélectionnée](concepts-server-parameters.md#max_connections). 


## <a name="next-steps"></a>Étapes suivantes

- [Meilleures pratiques relatives au niveau de performance d’Azure Database pour MySQL](concept-performance-best-practices.md)
- [Meilleures pratiques relatives aux opérations de serveur d’Azure Database pour MySQL](concept-operation-excellence-best-practices.md)
