---
title: Meilleures pratiques relatives au niveau de performance – Azure Database pour MySQL
description: Cet article décrit les meilleures pratiques de monitoring et de réglage du niveau de performance d’une base de données Azure Database pour MySQL.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 11/23/2020
ms.openlocfilehash: c29c043a3af46086751629b31ce68217e7226442
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96354858"
---
# <a name="best-practices-for-optimal-performance-of-your-azure-database-for-mysql---single-server"></a>Meilleures pratiques pour un niveau de performance optimal d’Azure Database pour MySQL – Serveur unique

Découvrez les meilleures pratiques à respecter pour obtenir un niveau de performance optimal avec Azure Database pour MySQL – Serveur unique. Au fur et à mesure que nous ajouterons de nouvelles fonctionnalités à la plateforme, nous continuerons d’affiner les meilleures pratiques décrites dans cette section.

## <a name="physical-proximity"></a>Proximité physique

 Veillez à déployer l’application et la base de données dans la même région. Avant le début de l’exécution d’un benchmark du niveau de performance, effectuez un test rapide : déterminez le temps de réponse du réseau entre le client et la base de données à l’aide d’une simple requête SELECT 1. 

## <a name="accelerated-networking"></a>Mise en réseau accélérée

Utilisez les performances réseau accélérées pour le serveur d’applications si vous utilisez une machine virtuelle Azure, Azure Kubernetes ou App Services. Une mise en réseau accélérée permet d’opérer une virtualisation d’E/S d’une racine unique (SR-IOV) sur une machine virtuelle, ce qui améliore considérablement les performances de mise en réseau. Cette voie hautement performante court-circuite l’hôte à partir du chemin d’accès aux données, réduisant ainsi la latence, l’instabilité et l’utilisation du processeur pour servir les charges de travail réseau les plus exigeantes sur les types de machines virtuelles pris en charge.

## <a name="connection-efficiency"></a>Efficacité de la connexion

L’établissement d’une nouvelle connexion est toujours une tâche coûteuse et fastidieuse. Quand une application demande une connexion de base de données, elle ne crée pas de nouvelle connexion, mais classe par ordre de priorité l’allocation des connexions de base de données inactives existantes.  Voici quelques-unes des meilleures pratiques de connexion possibles :

- **ProxySQL** : utilisez [ProxySQL](https://proxysql.com/), qui offre un regroupement de connexions intégré et assure [l’équilibrage de la charge de travail](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/load-balance-read-replicas-using-proxysql-in-azure-database-for/ba-p/880042) sur plusieurs réplicas en lecture en fonction des besoins et à la demande, pour toute modification apportée au code de l’application.

- **Proxy de données Heimdall** : vous pouvez également tirer parti du proxy de données Heimdall, une solution de proxy propriétaire indépendante du fournisseur. Il prend en charge la mise en cache des requêtes et le fractionnement en lecture/écriture avec détection des décalages de réplication. Consultez également [Accélération du niveau de performance de MySQL avec le proxy Heimdall](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/accelerate-mysql-performance-with-the-heimdall-proxy/ba-p/1063349).  

- **Connexion persistante ou de longue durée** : si votre application présente des transactions courtes ou des requêtes d’une durée d’exécution en général inférieure à 5 ou 10 ms, remplacez les connexions courtes par des connexions persistantes. Ce changement n’implique que des modifications de code mineures. Il améliore cependant les performances de façon considérable dans de nombreux scénarios applicatifs courants. Veillez à définir le délai d’expiration ou à fermer la connexion lorsque la transaction est terminée.

- **Réplica** : si vous avez recours à un réplica, utilisez [ProxySQL](https://proxysql.com/) pour équilibrer la charge entre le serveur principal et le serveur de réplication secondaire accessible en lecture. Découvrez comment [configurer ProxySQL](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/scaling-an-azure-database-for-mysql-workload-running-on/ba-p/1105847).

## <a name="data-import-configurations"></a>Configurations de l’importation de données

- Vous pouvez faire passer temporairement votre instance à une taille de référence SKU supérieure avant de commencer une opération d’importation de données, puis effectuer un scale-down une fois l’importation réussie.
- Vous pouvez importer vos données avec un temps d’arrêt minimal avec [Azure Database Migration Service (DMS)](https://datamigration.microsoft.com/) pour les migrations en ligne ou hors connexion. 

## <a name="azure-database-for-mysql-memory-recommendations"></a>Recommandations relatives à la mémoire Azure Database pour MySQL

Concernant le niveau de performance d’Azure Database pour MySQL, l’une des meilleures pratiques consiste à allouer suffisamment de RAM pour que la plage de travail se trouve presque entièrement en mémoire. 

- Vérifiez si le pourcentage de mémoire utilisé atteint les [limites](https://docs.microsoft.com/azure/mysql/concepts-pricing-tiers) à l’aide des [métriques du serveur MySQL](https://docs.microsoft.com/azure/mysql/concepts-monitoring). 
- Configurez des alertes sur ces nombres afin de pouvoir prendre des mesures pour résoudre le problème quand les serveurs atteignent les limites. En fonction des limites définies, regardez si un scale-up de la référence SKU de base de données (taille de calcul supérieure ou meilleur niveau tarifaire) entraînerait une augmentation considérable du niveau de performance. 
- Effectuez un scale-up jusqu’à ce que votre niveau de performance ne chute plus après une opération de mise à l’échelle. Pour plus d’informations sur le suivi des métriques d’une instance de base de données, consultez [Métriques de base de données MySQL](https://docs.microsoft.com/azure/mysql/concepts-monitoring#metrics).

## <a name="next-steps"></a>Étapes suivantes

- [Meilleures pratiques relatives aux opérations de serveur d’Azure Database pour MySQL](concept-operation-excellence-best-practices.md) <br/>
- [Meilleures pratiques de monitoring d’Azure Database pour MySQL](concept-monitoring-best-practices.md)<br/>
- [Bien démarrer avec Azure Database pour MySQL](quickstart-create-mysql-server-database-using-azure-portal.md)<br/>
