---
title: Ingérer les données dans un pool SQL dédié
description: Découvrez comment ingérer des données dans un pool SQL dédié dans Azure Synapse Analytics.
services: synapse-analytics
author: djpmsft
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 11/03/2020
ms.author: daperlov
ms.reviewer: jrasnick
ms.openlocfilehash: afebb6a67b4c8a33e812c0e51ff225f16cd35356
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461475"
---
# <a name="ingest-data-into-a-dedicated-sql-pool"></a>Ingérer les données dans un pool SQL dédié

Cet article explique comment ingérer les données d’un compte de stockage Azure Data Lake Gen2 dans un pool SQL dédié à l’aide d’Azure Synapse Analytics.

## <a name="prerequisites"></a>Prérequis

* **Abonnement Azure** : Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.
* **Compte de stockage Azure** : Vous utilisez Azure Data Lake Storage Gen2 comme magasin de données *source*. Si vous ne possédez pas de compte de stockage, consultez l’article [Créer un compte Stockage Azure](../../storage/blobs/data-lake-storage-quickstart-create-account.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) pour découvrir comment procéder.
* **Azure Synapse Analytics** : Vous utilisez un pool SQL dédié en tant que magasin de données *récepteur*. Si vous ne disposez pas d’une instance Synapse Analytics, consultez [Créer un pool SQL dédié](../../azure-sql/database/single-database-create-quickstart.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) pour savoir comment procéder.

## <a name="create-linked-services"></a>Créez des services liés

Dans Azure Synapse Analytics, un service lié vous permet de définir vos informations de connexion à d’autres services. Dans cette section, vous allez ajouter un service lié Azure Synapse Analytics et Azure Data Lake Storage Gen2.

1. Ouvrez l'expérience utilisateur Azure Synapse Analytics et accédez à l’onglet **Gérer**.
1. Sous **Connexions externes**, sélectionnez **Services liés**.
1. Pour ajouter un service lié, sélectionnez **Nouveau**.
1. Sélectionnez la vignette Azure Data Lake Storage Gen2 dans la liste, puis sélectionnez **Continuer**.
1. Entrez vos informations d’identification d’authentification. Les types d’authentification actuellement pris en charge sont les suivants : clé de compte, principal de service et identité managée. Sélectionnez Tester la connexion pour vérifier que vos informations d’identification sont correctes. Lorsque vous avez terminé, sélectionnez **Créer**.
1. Répétez les étapes 3 à 5, mais au lieu d'Azure Data Lake Storage Gen2, sélectionnez la vignette Azure Synapse Analytics et entrez les informations d’identification de connexion correspondantes. Pour Azure Synapse Analytics, l’authentification SQL, l’identité managée et le principal de service sont actuellement pris en charge.

## <a name="create-pipeline"></a>Création d’un pipeline

Un pipeline contient le flux logique pour l’exécution d’un ensemble d’activités. Dans cette section, vous allez créer un pipeline contenant une activité de copie qui ingère des données d'ADLS Gen2 dans un pool SQL dédié.

1. Accédez à l’onglet **Intégrer**. Sélectionnez l’icône + à côté de l’en-tête Pipelines, puis sélectionnez **Pipeline**.
1. Dans le volet des activités, sous **Déplacer et transformer**, faites glisser **Copier les données** sur le canevas du pipeline.
1. Sélectionnez l’activité de copie, puis accédez à l’onglet **Source**. Sélectionnez **Nouveau** pour créer un jeu de données source.
1. Sélectionnez Azure Data Lake Storage Gen2 comme magasin de données, puis sélectionnez Continuer.
1. Sélectionnez DelimitedText comme format, puis sélectionnez Continuer.
1. Dans le volet Définir les propriétés, sélectionnez le service lié ADLS que vous avez créé. Spécifiez le chemin d’accès du fichier de vos données sources, puis spécifiez si la première ligne contient un en-tête. Vous pouvez importer le schéma à partir du magasin de fichiers ou d’un exemple de fichier. Sélectionnez OK lorsque vous avez terminé.
1. Accédez à l’onglet **Récepteur**. Sélectionnez **Nouveau** pour créer un jeu de données récepteur.
1. Sélectionnez Azure Synapse Analytics en tant que magasin de données, puis sélectionnez Continuer.
1. Dans le volet Définir les propriétés, sélectionnez le service lié Azure Synapse Analytics que vous avez créé. Si vous écrivez dans une table existante, sélectionnez-la dans la liste déroulante. Sinon, cochez **Modifier** et entrez le nom de la nouvelle table. Lorsque vous avez terminé, sélectionnez OK.
1. Si vous créez une table, activez **Créer automatiquement une table** dans le champ d'option.

## <a name="debug-and-publish-pipeline"></a>Déboguer et publier le pipeline

Une fois la configuration de votre pipeline terminée, avant de publier vos artefacts, vous pouvez exécuter un débogage pour vérifier que tout est correct.

1. Pour déboguer le pipeline, sélectionnez **Déboguer** dans la barre d’outils. L’état d’exécution du pipeline apparaît dans l’onglet **Sortie** au bas de la fenêtre. 
1. Une fois que le pipeline peut s’exécuter correctement, sélectionnez **Publier tout** dans la barre d’outils supérieure. Cette action publie les entités (jeux de données et pipelines) que vous avez créées dans le service Synapse Analytics.
1. Patientez jusqu’à voir le message **Publication réussie**. Pour afficher les messages de notification, sélectionnez le bouton avec l’icône de cloche en haut à droite. 


## <a name="trigger-and-monitor-the-pipeline"></a>Déclencher et surveiller le pipeline

Au cours de cette étape, vous déclenchez manuellement le pipeline publié à l’étape précédente. 

1. Sélectionnez **Ajouter déclencheur** dans la barre d’outils, puis **Déclencher maintenant**. Dans la page **Exécution du pipeline**, sélectionnez **Terminer**.  
1. Accédez à l’onglet **Surveiller** dans la barre latérale gauche. Vous voyez un pipeline qui est déclenché par un déclencheur manuel. Vous pouvez utiliser les liens dans la colonne **Actions** pour afficher les détails de l’activité et réexécuter le pipeline.
1. Pour afficher les exécutions d’activités associées à l’exécution du pipeline, sélectionnez le lien **Afficher les exécutions d’activités** dans la colonne **Actions**. Dans cet exemple, il n’y a qu’une seule activité, vous ne voyez donc qu’une seule entrée dans la liste. Pour plus de détails sur l’opération de copie, sélectionnez le lien **Détails** (icône en forme de lunettes) dans la colonne **Actions**. Sélectionnez **Exécutions de pipeline** au sommet de la page pour revenir à la vue des exécutions du pipeline. Sélectionnez **Actualiser** pour actualiser l’affichage.
1. Vérifiez que vos données sont correctement écrites dans le pool SQL dédié.


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’intégration de données pour Azure Synapse Analytics, consultez l’article [Ingestion de données dans Azure Data Lake Storage Gen2](data-integration-data-lake.md).
