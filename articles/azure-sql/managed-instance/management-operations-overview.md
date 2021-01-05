---
title: Vue d’ensemble des opérations de gestion
titleSuffix: Azure SQL Managed Instance
description: Découvrez la durée des opérations de gestion Azure SQL Managed Instance et les meilleures pratiques.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: overview
author: urosmil
ms.author: urmilano
ms.reviewer: sstein, MashaMSFT
ms.date: 07/10/2020
ms.openlocfilehash: 2da7311e61aa39be69a6a0a29eff686baaad7ebf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91323190"
---
# <a name="overview-of-azure-sql-managed-instance-management-operations"></a>Vue d’ensemble des opérations de gestion Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Azure SQL Managed Instance fournit des opérations de gestion que vous pouvez utiliser pour déployer automatiquement de nouvelles instances managées, mettre à jour les propriétés des instances et supprimer des instances quand vous n’en avez plus besoin.

## <a name="what-are-management-operations"></a>Présentation des opérations de gestion

Toutes les opérations de gestion peuvent être classées comme suit :

- Déploiement d’instance (création d’instance)
- Mise à jour d’instance (modification des propriétés d’une instance, par exemple, par exemple les vCores ou le stockage réservé)
- Suppression d’instance

Pour prendre en charge les [déploiements au sein des réseaux virtuels Azure](../../virtual-network/virtual-network-for-azure-services.md) et assurer l’isolation et la sécurité des clients, SQL Managed Instance s’appuie sur des [clusters virtuels](connectivity-architecture-overview.md#high-level-connectivity-architecture). Le cluster virtuel représente un ensemble dédié de machines virtuelles isolées déployées dans le sous-réseau du réseau virtuel du client. Fondamentalement, chaque déploiement d’instance managée sur un sous-réseau vide génère une nouvelle structure de cluster virtuel.

Les opérations de gestion suivantes sur les instances managées peuvent avoir un impact sur le cluster virtuel sous-jacent. Les changements qui impactent le cluster virtuel sous-jacent peuvent affecter la durée des opérations de gestion, car le déploiement de machines virtuelles supplémentaires s’accompagne d’une surcharge que vous devez prendre en compte pour planifier de nouveaux déploiements ou des mises à jour d’instances managées existantes.


## <a name="duration"></a>Duration

La durée des opérations sur le cluster virtuel peut varier, mais elle a généralement la durée la plus longue. 

Voici les valeurs généralement attendues, selon les données de télémétrie de service existantes :

- **Création de cluster virtuel** :  La création constitue une étape synchrone dans les opérations de gestion d’instance. <br/> **90 % des opérations se terminent dans les quatre heures**.
- **Redimensionnement de cluster virtuel (expansion ou réduction)**  : L’expansion est une étape synchrone, tandis que la réduction est effectuée de façon asynchrone (sans incidence sur la durée des opérations de gestion des instances). <br/>**90 % des expansions de cluster nécessitent moins de deux heures trente**.
- **Suppression de cluster virtuel** : La suppression est une étape asynchrone, mais elle peut également être [lancée manuellement](virtual-cluster-delete.md) sur un cluster virtuel vide, auquel cas elle s’exécute de façon synchrone. <br/>**90 % des suppressions de cluster virtuel ne prennent pas plus d’une heure trente**.

Par ailleurs, la gestion des instances peut également comporter l’une des opérations suivantes sur les bases de données hébergées, qui allongent les durées d’exécution :

- **Attachement de fichiers de base de données à partir du Stockage Azure** :  Une étape synchrone, comme le scale-up ou le scale-down du calcul (vCores), ou du stockage dans le niveau de service Usage général. <br/>**90 % de ces opérations ne prennent pas plus de cinq minutes**.
- **Amorçage de groupe de disponibilité Always On** : étape synchrone. Exemples : mise à l’échelle du calcul (vCore) ou du stockage dans le niveau de service Critique pour l’entreprise, ou changement de niveau de service de Usage général à Critique pour l’entreprise (et inversement). La durée de cette opération est proportionnelle à la taille totale de la base de données, ainsi qu’à l’activité actuelle de la base de données (nombre de transactions actives). L’activité de la base de données lors de la mise à jour d’une instance peut faire varier considérablement la durée totale. <br/>**90 % de ces opérations s’exécutent à 220 Go/heure ou plus**.

Les tableaux suivants récapitulent les opérations et les durées globales habituelles, en fonction de la catégorie de l’opération :

**Catégorie : Déploiement**

|Opération  |Segment de longue durée  |Durée estimée  |
|---------|---------|---------|
|Première instance sur un sous-réseau vide|Création de cluster virtuel|90 % des opérations se terminent dans les 4 heures.|
|Première instance d’une autre génération de matériel sur un sous-réseau non vide (par exemple, première instance de Génération 5 sur un sous-réseau avec des instances de Génération 4)|Création de cluster virtuel<sup>1</sup>|90 % des opérations se terminent dans les 4 heures.|
|Création d’instance suivante sur le sous-réseau non vide (2e instance, 3e instance, et ainsi de suite.)|Redimensionnement de cluster virtuel|90 % des opérations se terminent dans les deux heures trente.|
| | | 

<sup>1</sup> Le cluster virtuel est créé en fonction de la génération du matériel.

**Catégorie : Mise à jour**

|Opération  |Segment de longue durée  |Durée estimée  |
|---------|---------|---------|
|Modification de propriété d’instance (mot de passe administrateur, connexion Azure AD, indicateur Azure Hybrid Benefit)|N/A|Jusqu’à une minute.|
|Scale-up ou scale-down du stockage d’instance (niveau de service Usage général)|Attachement de fichiers de base de données|90 % des opérations ne prennent pas plus de cinq minutes.|
|Scale-up et scale-down du stockage d’instance (niveau de service Critique pour l’entreprise)|- Redimensionnement de cluster virtuel<br>- Amorçage de groupe de disponibilité Always On|90 % des opérations se terminent dans les 2,5 heures + durée nécessaire pour amorcer toutes les bases de données (220 Go/heure).|
|Scale-up et scale-down de la capacité de calcul des instances (vCores) (Usage général)|- Redimensionnement de cluster virtuel<br>- Attachement de fichiers de base de données|90 % des opérations se terminent dans les deux heures trente.|
|Scale-up et scale-down de la capacité de calcul des instances (vCores) (Critique pour l’entreprise)|- Redimensionnement de cluster virtuel<br>- Amorçage de groupe de disponibilité Always On|90 % des opérations se terminent dans les 2,5 heures + durée nécessaire pour amorcer toutes les bases de données (220 Go/heure).|
|Modification du niveau de service de l’instance (Usage général vers Critique pour l’entreprise et vice versa)|- Redimensionnement de cluster virtuel<br>- Amorçage de groupe de disponibilité Always On|90 % des opérations se terminent dans les 2,5 heures + durée nécessaire pour amorcer toutes les bases de données (220 Go/heure).|
| | | 

**Catégorie : Suppression**

|Opération  |Segment de longue durée  |Durée estimée  |
|---------|---------|---------|
|Suppression d’instance|Sauvegarde de la fin du journal pour toutes les bases de données|90 % des opérations se terminent en une minute.<br>Remarque : Si la dernière instance du sous-réseau est supprimée, cette opération va planifier la suppression du cluster virtuel au bout de 12 heures.<sup>1</sup>|
|Suppression de cluster virtuel (en tant qu’opération lancée par l’utilisateur)|Suppression de cluster virtuel|90 % des opérations ne nécessitent pas plus d’une heure trente.|
| | | 

<sup>1</sup>12 heures est la configuration actuelle, mais elle est sujette à modification dans le futur. Si vous devez supprimer un cluster virtuel plus tôt (par exemple pour libérer le sous-réseau), consultez [Supprimer un sous-réseau après avoir supprimé une instance managée](virtual-cluster-delete.md).

## <a name="instance-availability"></a>Disponibilité des instances

SQL Managed Instance **est disponible pendant les opérations de mise à jour**, à l’exception d’un court temps d’arrêt dû au basculement qui se produit à la fin de la mise à jour. Elle dure généralement jusqu’à 10 secondes, même en cas de transactions longues interrompues, grâce à la [récupération de base de données accélérée](../accelerated-database-recovery.md).

SQL Managed Instance n’est pas accessible aux applications clientes pendant les opérations de déploiement et de suppression.

> [!IMPORTANT]
> Nous vous déconseillons de mettre à l’échelle la capacité de calcul ou de stockage d’Azure SQL Managed Instance, ou de changer le niveau de service en même temps que des transactions de longue durée (importation de données, travaux de traitement des données, regénération d’index, etc.). Le basculement de la base de données à la fin de l’opération annule toutes les transactions en cours. 

## <a name="management-operations-steps"></a>Étapes des opérations de gestion

Les opérations de gestion sont constituées de plusieurs étapes. Avec l’[introduction de l’API Opérations](management-operations-monitor.md), ces étapes sont exposées pour un sous-ensemble d’opérations (déploiement et mise à jour). L’opération de déploiement se compose de trois étapes, tandis que l’opération de mise à jour est effectuée en six étapes. Pour plus d’informations sur la durée des opérations, consultez la section [Durée des opérations de gestion](#duration). Les étapes sont listées par ordre d’exécution.

### <a name="managed-instance-deployment-steps"></a>Étapes de déploiement d’une instance managée

|Nom de l’étape  |Description de l’étape  |
|----|---------|
|Validation des demandes |Les paramètres soumis sont validés. En cas d’erreur de configuration, l’opération échoue avec une erreur. |
|Redimensionnement / création de cluster virtuel |Selon l’état du sous-réseau, le cluster virtuel est créé ou redimensionné. |
|Démarrage de la nouvelle instance SQL |Le processus SQL est démarré sur le cluster virtuel déployé. |

### <a name="managed-instance-update-steps"></a>Étapes de mise à jour d’une instance managée

|Nom de l’étape  |Description de l’étape  |
|----|---------|
|Validation des demandes | Les paramètres soumis sont validés. En cas d’erreur de configuration, l’opération échoue avec une erreur. |
|Redimensionnement / création de cluster virtuel |Selon l’état du sous-réseau, le cluster virtuel est créé ou redimensionné. |
|Démarrage de la nouvelle instance SQL | Le processus SQL est démarré sur le cluster virtuel déployé. |
|Amorçage des fichiers de base de données / attachement de fichiers de base de données |Selon le type de l’opération de mise à jour, l’amorçage de la base de données ou l’attachement de fichiers de base de données est effectué. |
|Préparation du basculement et basculement |Une fois les données amorcées ou les fichiers de base de données rattachés, le système est préparé pour le basculement. Quand tout est défini, le basculement est effectué **avec un temps d’arrêt réduit**. |
|Nettoyage de l’ancienne instance SQL |Suppression de l’ancien processus SQL du cluster virtuel |

> [!NOTE]
> Suite à la mise à l’échelle des instances, le cluster virtuel sous-jacent passe par le processus de libération de la capacité inutilisée et par une défragmentation possible de la capacité, ce qui peut affecter les instances qui n’ont pas participé aux opérations de création et de mise à l’échelle. 


## <a name="management-operations-cross-impact"></a>Impact sur les opérations de gestion

Les opérations de gestion effectuées sur une instance gérée sont susceptibles d’avoir une incidence sur celles des instances placées dans le même cluster virtuel :

- Les **opérations de restauration longues** dans un cluster virtuel vont mettre en suspens les opérations de création ou de mise à l’échelle d’une autre instance dans le même sous-réseau.<br/>**Exemple :** Si une opération de restauration longue est en cours d’exécution et qu’il existe une demande de création ou de mise à l’échelle dans le même sous-réseau, cette demande va prendre plus de temps à s’exécuter, car elle attend la fin de l’opération de restauration avant de continuer.

- L’opération suivante de **création ou de mise à l’échelle d’une instance** est mise en attente par l’opération de création ou de mise à l’échelle de l’instance lancée précédemment qui a lancé un redimensionnement du cluster virtuel.<br/>**Exemple :** Dans le cas de plusieurs demandes de création et/ou de mise à l’échelle dans le même sous-réseau sous le même cluster virtuel, si l’une d’elles lance un redimensionnement du cluster virtuel, toutes les demandes soumises 5 minutes ou plus après la demande d’opération initiale vont durer plus longtemps qu’attendu, car ces demandes doivent attendre la fin de ce redimensionnement pour reprendre.

- Les **opérations de création/mise à l’échelle soumises dans un délai de 5 minutes** sont traitées par lot et exécutées en parallèle.<br/>**Exemple :** Un seul redimensionnement de cluster virtuel est effectué pour toutes les opérations soumises dans un délai de 5 minutes (à compter de l’exécution de la première demande d’opération). Si une autre demande est soumise plus de 5 minutes après la première, elle attend la fin du redimensionnement du cluster virtuel avant de commencer à s’exécuter.

> [!IMPORTANT]
> Les opérations de gestion mises en suspens en raison d’une autre opération en cours sont reprises automatiquement une fois que les conditions permettant de continuer sont remplies. Aucune action de l’utilisateur n’est nécessaire pour reprendre les opérations de gestion temporairement suspendues.

## <a name="monitoring-management-operations"></a>Supervision des opérations de gestion

Pour découvrir comment superviser la progression et l’état de l’opération de gestion, consultez [Supervision des opérations de gestion](management-operations-monitor.md).

## <a name="canceling-management-operations"></a>Annulation des opérations de gestion

Pour découvrir comment annuler une opération de gestion, consultez [Annulation des opérations de gestion](management-operations-cancel.md).


## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment créer votre première instance managée, consultez le [Guide de démarrage rapide](instance-create-quickstart.md).
- Pour une liste des fonctionnalités et leur comparaison, consultez [Fonctionnalités SQL courantes](../database/features-comparison.md).
- Pour plus d’informations sur la configuration du réseau virtuel, consultez [Configuration de réseau virtuel SQL Managed Instance](connectivity-architecture-overview.md).
- Pour obtenir un guide de démarrage rapide qui crée une instance managée et restaure une base de données à partir d’un fichier de sauvegarde, consultez [Créer une instance managée](instance-create-quickstart.md).
- Pour accéder à un tutoriel expliquant comment utiliser Azure Database Migration Service pour la migration, consultez [Migration SQL Managed Instance à l’aide d’Azure Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
