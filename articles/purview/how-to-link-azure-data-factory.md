---
title: Se connecter à Azure Data Factory
description: Cet article explique comment se connecter Azure Data Factory et Azure Purview pour suivre la traçabilité des données.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/22/2020
ms.openlocfilehash: 01af7b251c9ce3bfebb87016c85ea3efd9c0e8ac
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928764"
---
# <a name="how-to-connect-azure-data-factory-and-azure-purview"></a>Guide pratique pour se connecter à Azure Data Factory et Azure Purview

Ce document explique les étapes nécessaires pour connecter un compte Azure Data Factory à un compte Azure Purview afin de suivre la traçabilité des données. Il décrit également en détail l’étendue de la couverture et les modèles de traçabilité pris en charge.

## <a name="view-existing-data-factory-connections"></a>Afficher les connexions Data Factory existantes

Plusieurs fabriques de données Azure peuvent se connecter à un même Data Catalog Azure Purview pour envoyer des informations de traçabilité. La limite actuelle vous permet de connecter dix comptes Data Factory à la fois à partir du centre d’administration Purview. Pour afficher la liste des comptes Data Factory connectés à votre Data Catalog Purview, effectuez les étapes suivantes :

1. Sélectionnez **Centre de gestion** dans le volet de navigation gauche.
2. Sous **Connexions externes**, sélectionnez **Connexion Data Factory**.
3. La liste de connexions Data Factory s’affiche.

    :::image type="content" source="./media/how-to-link-azure-data-factory/data-factory-connection.png" alt-text="Capture d’écran montrant une liste de connexions Data Factory." lightbox="./media/how-to-link-azure-data-factory/data-factory-connection.png":::

4. Notez les différentes valeurs de l’**État** de connexion :

    - **Connecté** : la fabrique de données est connectée au catalogue de données.
    - **Déconnecté** : la fabrique de données a accès au catalogue, mais elle est connectée à un autre catalogue. En conséquence, la traçabilité des données ne sera pas automatiquement signalée au catalogue.
    - **Accès impossible** : l’utilisateur actuel n’a pas accès à la fabrique de données ; l’état de la connexion est donc inconnu.
 >[!Note]
 >Pour afficher les connexions Data Factory, vous devez détenir l’un des rôles Purview :
 >- Contributeur
 >- Propriétaire
 >- Lecteur
 >- Administrateur de l’accès utilisateur

## <a name="create-new-data-factory-connection"></a>Créer une connexion Data Factory

>[!Note]
>Pour ajouter ou supprimer les connexions Data Factory, vous devez détenir l’un des rôles Purview :
>- Propriétaire
>- Administrateur de l'accès utilisateur
>
> De plus, les utilisateurs doivent être le « contributeur » ou le « propriétaire » de la fabrique de données. 

Suivez les étapes ci-dessous pour connecter un compte Data Factory existant à votre Data Catalog Purview.

1. Sélectionnez **Centre de gestion** dans le volet de navigation gauche.
2. Sous **Connexions externes**, sélectionnez **Connexion Data Factory**.
3. Dans la page **Connexion Data Factory**, sélectionnez **Nouveau**.

4. Sélectionnez votre compte Data Factory dans la liste, puis **OK**. Vous pouvez également filtrer par nom d’abonnement pour affiner la liste.

    :::image type="content" source="./media/how-to-link-azure-data-factory/connect-data-factory.png" alt-text="Capture d’écran montrant comment connecter Azure Data Factory." lightbox="./media/how-to-link-azure-data-factory/connect-data-factory.png":::

    Certaines instances de Data Factory peuvent être désactivées si la fabrique de données est déjà connectée au compte Purview actif ou si elle n’a pas d’identité managée.

    Un message d’avertissement s’affiche si l’une des fabriques de données sélectionnées est déjà connectée à un autre compte Purview. Si vous sélectionnez OK, la connexion Data Factory à l’autre compte Purview sera déconnectée. Aucune confirmation supplémentaire n’est requise.


    :::image type="content" source="./media/how-to-link-azure-data-factory/warning-for-disconnect-factory.png" alt-text="Capture d’écran montrant un avertissement pour déconnecter Azure Data Factory." lightbox="./media/how-to-link-azure-data-factory/warning-for-disconnect-factory.png":::

>[!Note]
>Nous prenons désormais en charge l’ajout de 10 fabriques de données maximum à la fois. Si vous souhaitez ajouter plus de 10 fabriques de données à la fois, veuillez créer un ticket de support.


### <a name="remove-data-factory-connections"></a>Supprimer des connexions Data Factory
Pour supprimer une connexion Data Factory, effectuez les étapes suivantes :

1. Dans la page **Connexion Data Factory**, sélectionnez le bouton **Supprimer** en regard d’une ou plusieurs connexions Data Factory.
1. Sélectionnez **Confirmer** dans la fenêtre contextuelle pour supprimer les connexions Data Factory sélectionnées.

    :::image type="content" source="./media/how-to-link-azure-data-factory/remove-data-factory-connection.png" alt-text="Capture d’écran montrant comment sélectionner des fabriques de données pour supprimer la connexion." lightbox="./media/how-to-link-azure-data-factory/remove-data-factory-connection.png":::

## <a name="configure-a-self-hosted-ir-to-collect-lineage-from-on-prem-sql"></a>Configurer un IR auto-hébergé pour recueillir la traçabilité à partir de SQL local

Le traçabilité de l’activité Copy Data Factory est disponible pour les bases de données SQL locales. Si vous exécutez un runtime d’intégration auto-hébergé pour le déplacement des données avec Azure Data Factory et que vous souhaitez capturer la traçabilité dans Azure Purview, vérifiez que la version est 4.8.7418.1 ou ultérieure. Pour plus d’informations sur le runtime d’intégration auto-hébergé, consultez [Créer et configurer un runtime d’intégration auto-hébergé](../data-factory/create-self-hosted-integration-runtime.md).

## <a name="supported-azure-data-factory-activities"></a>Activités Azure Data Factory prises en charge

Azure Purview capture la traçabilité du runtime à partir des activités Azure Data Factory suivantes :

- Copier des données
- Data Flow
- Exécuter le Package SSIS

> [!IMPORTANT]
> Azure Purview supprime la traçabilité si la source ou la destination utilise un système de stockage de données non pris en charge.

L’intégration entre Data Factory et Purview ne prend en charge qu’un sous-ensemble des systèmes de données pris en charge par Data Factory, comme décrit dans les sections suivantes.

### <a name="data-factory-copy-data-support"></a>Prise en charge de la copie de données Data Factory

| Système de stockage de données | Prise en charge en tant que source | Prise en charge en tant que récepteur |
| ------------------- | ------------------- | ----------------- |
| ADLS Gen1 (aucune prise en charge de JSON) | Oui | Oui (copie non binaire uniquement) |
| ADLS Gen2 (aucune prise en charge de JSON) | Oui | Oui |
| Objet blob Azure (aucune prise en charge de JSON) | Oui | Oui |
| Azure Cosmos DB (API SQL) | Oui | Oui |
| Azure Cosmos DB (API Mongo) | Oui | Oui |
| Recherche cognitive Azure | Oui | Oui |
| Explorateur de données Azure | Oui | Oui |
| Azure Database for Maria DB \* | Oui | Oui |
| Azure Database pour MYSQL \* | Oui | Oui |
| Azure Database pour PostgreSQL \* | Oui | Oui |
| Stockage Fichier Azure | Oui | Oui |
| Stockage de table Azure | Oui | Oui |
| Azure SQL Database \* | Oui | Oui |
| Azure SQL Database Managed Instance \* | Oui | Oui |
| Azure Synapse Analytics (anciennement SQL DW) \* | Oui | Oui |
| SQL Server local (SHIR requis) \* | Oui | Oui |
| Amazon S3 | Oui | Oui |
| Teradata | Oui | Oui |
| SAP s4 Hana | Oui | Oui |
| SAP ECC | Oui | Oui |
| Ruche | Oui | Oui |

> [!Note]
> La fonctionnalité de traçabilité implique certaines surcharges de performances dans l’activité de copie Data Factory. Ceux qui configurent des connexions Data Factory dans Purview constateront peut-être que certaines tâches de copie prennent davantage de temps. Le plus souvent, l’impact est négligeable, voire nul. Veuillez contacter le support en indiquant la comparaison de durée si les travaux de copie prennent beaucoup plus de temps que d’habitude.

### <a name="data-factory-data-flow-support"></a>Prise en charge de Data Flow dans Data Factory

| Système de stockage de données | Prise en charge |
| ------------------- | ------------------- | ----------------- |
| ADLS Gen1 | Oui |
| ADLS Gen2 | Oui |
| Objets blob Azure | Oui |
| Azure SQL Database \* | Oui |
| Azure Synapse Analytics (anciennement SQL DW) \* | Oui |

### <a name="data-factory-execute-ssis-package-support"></a>Prise en charge de l’exécution d’un package SSIS dans Data Factory

| Système de stockage de données | Prise en charge |
| ------------------- | ------------------- | ----------------- |
| Objets blob Azure | Oui |
| ADLS Gen1 | Oui |
| ADLS Gen2 | Oui |
| Azure SQL Database \* | Oui |
| Azure SQL Database Managed Instance \*| Oui |
| Azure Synapse Analytics (anciennement SQL DW) \* | Oui |
| SQL Server local \* | Oui |
| Stockage Fichier Azure | Oui |

*\* Pour les scénarios SQL (Azure et locaux), Azure Purview ne prend pas en charge les procédures stockées ou les scripts pour la traçabilité ou l’analyse. La traçabilité est limitée aux sources de table et de vue uniquement.*

> [!Note]
> Azure Data Lake Storage Gen2 est maintenant en disponibilité générale. Nous vous recommandons de commencer à l’utiliser dès aujourd'hui. Pour plus d’informations, consultez la [page du produit](https://azure.microsoft.com/en-us/services/storage/data-lake-storage/).

## <a name="supported-lineage-patterns"></a>Modèles de traçabilité pris en charge

Azure Purview prend en charge plusieurs modèles de traçabilité. Les données de traçabilité générées sont basées sur le type de source et de récepteur utilisé dans les activités Data Factory. Bien que Data Factory prenne en charge plus de 80 sources et récepteurs, Azure Purview prend uniquement en charge un sous-ensemble, comme indiqué dans [Activités Azure Data Factory prises en charge](#supported-azure-data-factory-activities).

Pour configurer Data Factory de façon à envoyer des informations de traçabilité, consultez [Bien démarrer avec la traçabilité](catalog-lineage-user-guide.md#get-started-with-lineage).

Voici quelques autres méthodes permettant de rechercher des informations dans la vue de traçabilité :

- Sous l’onglet **Traçabilité**, pointez sur des formes pour afficher un aperçu des informations supplémentaires sur la ressource dans l’info-bulle.
- Sélectionnez le nœud ou la périphérie pour afficher le type de ressource auquel il appartient ou pour basculer d’une ressource à une autre.
- Les colonnes d’un jeu de données sont affichées sur le côté gauche de l’onglet **Traçabilité**. Pour plus d’informations sur la traçabilité au niveau des colonnes, consultez [Traçabilité au niveau des colonnes](catalog-lineage-user-guide.md#column-level-lineage).

### <a name="data-lineage-for-11-operations"></a>Traçabilité des données pour les opérations 1:1

Le modèle le plus courant pour capturer la traçabilité des données consiste à déplacer les données d’un jeu de données d’entrée unique vers un jeu de données de sortie unique, avec un processus entre les deux.

Voici un exemple de ce modèle :

- 1 source/entrée : *Customer* (table SQL)
- 1 récepteur/sortie : *Customer1.csv* (objet blob Azure)
- 1 processus : *CopyCustomerInfo1\#Customer1.csv* (activité Copy Data Factory)

:::image type="content" source="./media/how-to-link-azure-data-factory/adf-copy-lineage.png" alt-text="Capture d’écran montrant la traçabilité pour une opération de copie un-à-un Data Factory." lightbox="./media/how-to-link-azure-data-factory/adf-copy-lineage.png":::

### <a name="data-movement-with-11-lineage-and-wildcard-support"></a>Déplacement des données avec traçabilité 1:1 et prise en charge des caractères génériques

Un autre scénario courant pour capturer la traçabilité consiste à utiliser un caractère générique pour copier les fichiers d’un jeu de données d’entrée unique vers un jeu de données de sortie unique. Le caractère générique permet à l’activité de copie d’établir une correspondance avec plusieurs fichiers pour la copie à l’aide d’une partie commune du nom de fichier. Azure Purview capture la traçabilité au niveau du fichier pour chaque fichier copié par l’activité de copie correspondante.

Voici un exemple de ce modèle :

* Source/entrée : *CustomerCall\*.csv* (chemin ADLS Gen2)
* Récepteur/sortie : *CustomerCall\*.csv* (fichier blob Azure)
* 1 processus : *CopyGen2ToBlob\#CustomerCall.csv* (activité Copy Data Factory)  

:::image type="content" source="./media/how-to-link-azure-data-factory/adf-copy-lineage-wildcard.png" alt-text="Capture d’écran montrant la traçabilité pour une opération de copie un-à-un avec prise en charge des caractères génériques." lightbox="./media/how-to-link-azure-data-factory/adf-copy-lineage-wildcard.png":::

### <a name="data-movement-with-n1-lineage"></a>Déplacement des données avec traçabilité n:1

Vous pouvez utiliser des activités Data Flow pour effectuer des opérations de données telles que la fusion, la jointure, et ainsi de suite. Plusieurs jeux de données sources peuvent être utilisés pour produire un jeu de données cible. Dans cet exemple, Azure Purview capture la traçabilité au niveau du fichier pour des fichiers d’entrée spécifiques dans une table SQL qui fait partie d’une activité Data Flow.

Voici un exemple de ce modèle :

* 2 sources/entrées : *Customer.csv*, *Sales.parquet* (chemin ADLS Gen2)
* 1 récepteur/sortie : *Company data* (table Azure SQL)
* 1 processus : *DataFlowBlobsToSQL* (activité Data Flow Data Factory)

:::image type="content" source="./media/how-to-link-azure-data-factory/adf-data-flow-lineage.png" alt-text="Capture d’écran montrant la traçabilité pour une opération n-à-un Data Flow ADF." lightbox="./media/how-to-link-azure-data-factory/adf-data-flow-lineage.png":::

### <a name="lineage-for-resource-sets"></a>Traçabilité pour les ensembles de ressources

Un ensemble de ressources est un objet logique dans le catalogue qui représente de nombreux fichiers de partition dans le stockage sous-jacent. Pour plus d’informations, consultez [Présentation des ensembles de ressources](concept-resource-sets.md). Quand Azure Purview capture la traçabilité à partir d’Azure Data Factory, il applique les règles pour normaliser les fichiers de partition et créer un objet logique unique.

Dans l’exemple suivant, un ensemble de ressources Azure Data Lake Gen2 est généré à partir d’un objet blob Azure :

* 1 source/entrée : *Employee\_management.csv* (objet blob Azure)
* 1 récepteur/sortie : *Employee\_management.csv* (Azure Data Lake Gen 2)
* 1 processus : *CopyBlobToAdlsGen2\_RS* (activité Copy Data Factory)

:::image type="content" source="./media/how-to-link-azure-data-factory/adf-resource-set-lineage.png" alt-text="Capture d’écran montrant la traçabilité d’un ensemble de ressources." lightbox="./media/how-to-link-azure-data-factory/adf-resource-set-lineage.png":::

## <a name="next-steps"></a>Étapes suivantes

- [Guide de l’utilisateur sur la traçabilité Data Catalog](catalog-lineage-user-guide.md)
- [Lien vers Azure Data Share pour la traçabilité](how-to-link-azure-data-share.md)