---
title: Résilience et haute disponibilité
titleSuffix: Azure Machine Learning
description: Découvrez comment rendre vos ressources Azure Machine Learning plus résistantes aux pannes à l’aide d’une configuration à haute disponibilité.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jhirono
author: jhirono
ms.reviewer: larryfr
ms.date: 09/16/2020
ms.openlocfilehash: 7a1a63893e6e2988fc5f21e84f21c74315d856b4
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93325480"
---
# <a name="increase-azure-machine-learning-resiliency"></a>Augmentez la résilience d’Azure Machine Learning



Dans cet article, découvrez comment rendre vos ressources Microsoft Azure Machine Learning plus résilientes à l’aide de configurations à haute disponibilité. Vous pouvez configurer les services Azure dont dépend Azure Machine Learning pour la haute disponibilité. Cet article identifie les services que vous pouvez configurer pour la haute disponibilité, ainsi que les liens vers des informations supplémentaires sur la configuration de ces ressources.

> [!NOTE]
> Azure Machine Learning lui-même n’offre pas d’option de récupération d’urgence.

## <a name="understand-azure-services-for-azure-machine-learning"></a>Comprendre les services Azure pour Azure Machine Learning

Azure Machine Learning dépend de plusieurs services Azure et possède plusieurs couches. Certains de ces services sont configurés dans votre abonnement (client). Vous êtes responsable de la configuration de la haute disponibilité de ces services. D’autres services sont créés dans un abonnement Microsoft et sont gérés par Microsoft. 

Les services Azure incluent :

* **Infrastructure Azure Machine Learning**  : un environnement géré par Microsoft pour un espace de travail Azure Machine Learning.

* **Ressources associées**  : Ressources approvisionnées dans votre abonnement lors de la création d’un espace de travail Azure Machine Learning. Ces ressources incluent le Stockage Azure, Azure Key Vault, Azure Container Registry et Application Insights. Vous êtes responsable de la configuration des paramètres de haute disponibilité pour ces ressources.
  * Le stockage par défaut contient des données telles que le modèle, les données du journal d’apprentissage et le jeu de données.
  * Key Vault possède des informations d’identification pour le Stockage Azure, Container Registry et les banques de données.
  * Container Registry dispose d’une image Docker pour les environnements de formation et d’inférence.
  * Application Insights est destiné à l’analyse d’Azure Machine Learning.

* **Ressources de calcul**  : Ressources que vous créez après le déploiement de l’espace de travail. Par exemple, vous pouvez créer une instance de calcul ou un cluster de calcul pour effectuer l'apprentissage d’un modèle Machine Learning.
  * Instance de calcul et cluster de calcul : environnements de développement de modèles gérés par Microsoft.
  * Autres ressources : Il s’agit des ressources de calcul Microsoft que vous pouvez joindre à Azure Machine Learning, comme Azure Kubernetes Service (AKS), Azure Databricks, Azure Container Instances et Azure HDInsight. Vous êtes responsable de la configuration des paramètres de haute disponibilité pour ces ressources.

* **Banques de données supplémentaires**  : Azure Machine Learning peut monter des magasins de données supplémentaires, comme le stockage Azure, Azure Data Lake Storage et Azure SQL Database pour les données d’apprentissage.  Ces banques de données sont configurées dans votre abonnement. Vous êtes responsable de la configuration de leurs paramètres de haute disponibilité.

Le tableau suivant répertorie les services Azure gérés par Microsoft, ceux qui sont gérés par vous et ceux qui sont hautement disponibles par défaut.

| Service | Géré par | Haute disponibilité par défaut |
| ----- | ----- | ----- |
| **Infrastructure Azure Machine Learning** | Microsoft | |
| **Ressources associées** |
| Stockage Azure | Vous | |
| Key Vault | Vous | ✓ |
| Container Registry | Vous | |
| Application Insights | Vous | N/D |
| **Ressources de calcul** |
| Instance de calcul | Microsoft |  |
| Cluster de calcul | Microsoft |  |
| Autres ressources de calcul telles que AKS, <br>Azure Databricks, Container Instances, HDInsight | Vous |  |
| **Banques de données supplémentaires** , comme le Stockage Azure, SQL Database,<br> Azure Database pour PostgreSQL, Azure Database pour MySQL, <br>système de fichiers Azure Databricks | Vous | |

Le reste de cet article décrit les actions que vous devez effectuer pour rendre chacun de ces services hautement disponible.

## <a name="associated-resources"></a>Ressources associées

> [!IMPORTANT]
> Azure Machine Learning ne prend pas en charge le basculement de compte de stockage par défaut à l’aide du stockage géo-redondant (GRS), du stockage géo-redondant interzone (GZRS), du stockage géo-redondant avec accès en lecture (RA-GRS) ou du stockage géo-redondant interzone avec accès en lecture (RA-GZRS).

Veillez à configurer les paramètres de haute disponibilité de chaque ressource en vous référant à la documentation suivante :

* **Stockage Azure** : Pour configurer les paramètres de haute disponibilité, consultez [Redondance du Stockage Azure](../storage/common/storage-redundancy.md).
* **Key Vault**  : Key Vault fournit par défaut une haute disponibilité et ne nécessite aucune action de l’utilisateur.  Consultez [Disponibilité et redondance d’Azure Key Vault](../key-vault/general/disaster-recovery-guidance.md).
* **Container Registry** (Registre de conteneurs) : Choisissez l’option de registre Premium pour la géo-réplication. Consultez [Géoréplication dans Azure Container Registry](../container-registry/container-registry-geo-replication.md).
* **Application Insights**  : Application Insights ne fournit pas de paramètres de haute disponibilité. Pour ajuster la période de rétention des données et les détails, consultez [Collecte, rétention et stockage des données dans Application Insights](../azure-monitor/app/data-retention-privacy.md#how-long-is-the-data-kept).

## <a name="compute-resources"></a>Ressources de calcul

Veillez à configurer les paramètres de haute disponibilité de chaque ressource en vous référant à la documentation suivante :

* **Azure Kubernetes Service**  : Consultez [Bonnes pratiques pour la continuité d’activité et la reprise d’activité dans AKS (Azure Kubernetes Services)](../aks/operator-best-practices-multi-region.md) et [Créer un cluster Azure Kubernetes Service (AKS) qui utilise des zones de disponibilité](../aks/availability-zones.md). Si le cluster AKS a été créé à l’aide de Studio Azure Machine Learning, du kit de développement logiciel (SDK) ou de l’interface CLI, la haute disponibilité inter-régions n’est pas prise en charge.
* **Azure Databricks**  : Consultez [Récupération d’urgence régionale pour les clusters Azure Databricks](/azure/databricks/scenarios/howto-regional-disaster-recovery).
* **Container Instances**  : Un orchestrateur est responsable du basculement. Consultez [Azure Container Instances et les orchestrateurs de conteneurs](../container-instances/container-instances-orchestrator-relationship.md).
* **HDInsight**  : Consultez [Services de haute disponibilité pris en charge par Azure HDInsight](../hdinsight/hdinsight-high-availability-components.md).

## <a name="additional-data-stores"></a>Banques de données supplémentaires

Veillez à configurer les paramètres de haute disponibilité de chaque ressource en vous référant à la documentation suivante :

* **Conteneur Blob Azure/ Azure Files/Data Lake Storage Gen2**  : Identique au stockage par défaut.
* **Data Lake Storage Gen1**  : Consultez [guide pour la haute disponibilité et récupération d'urgence pour Data Lake Storage Gen1](../data-lake-store/data-lake-store-disaster-recovery-guidance.md).
* **SQL Database**  : Consultez [Haute disponibilité pour Azure SQL Database et SQL Managed Instance](../azure-sql/database/high-availability-sla.md).
* **Azure Database pour PostgreSQL** : Consultez [Concepts de haute disponibilité dans Azure Database pour PostgreSQL (serveur unique)](../postgresql/concepts-high-availability.md).
* **Azure Database pour MySQL** : Consultez [Comprendre la continuité d’activité dans Azure Database pour MySQL](../mysql/concepts-business-continuity.md).
* **Système de fichiers Azure Databricks**  : Consultez [Récupération d’urgence régionale pour les clusters Azure Databricks](/azure/databricks/scenarios/howto-regional-disaster-recovery).

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

Si vous fournissez votre propre clé gérée par le client pour déployer un espace de travail Azure Machine Learning, Azure Cosmos DB est également configuré dans votre abonnement. Dans ce cas, vous êtes responsable de la configuration de ses paramètres de haute disponibilité. Consultez [Haute disponibilité avec Azure Cosmos DB](../cosmos-db/high-availability.md).

## <a name="next-steps"></a>Étapes suivantes

Pour déployer Azure Machine Learning avec les ressources associées avec vos paramètres de haute disponibilité, utilisez un [Modèle Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-machine-learning-advanced).