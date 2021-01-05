---
title: Cas d’utilisation - Profilage des utilisateurs
description: Découvrez comment Azure Data Factory permet de créer un workflow piloté par les données (pipeline) pour définir le profil des clients de leurs jeux.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 313050035694eaeabaefc8e09383ec3a887eb32b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96495920"
---
# <a name="use-case---customer-profiling"></a>Cas d’utilisation - Profilage des utilisateurs
Azure Data Factory est l’un des nombreux services permettant d’implémenter la solution d’accélérateurs Cortana Intelligence Suite.  Pour plus d’informations sur Cortana Intelligence, consultez [Cortana Intelligence Suite](https://www.microsoft.com/cortanaanalytics). Dans ce document, nous décrivons un cas d'utilisation simple pour vous aider à comprendre comment Azure Data Factory permet de résoudre les problèmes d'analyse courants.

## <a name="scenario"></a>Scénario
Contoso est une société qui crée des jeux pour plusieurs plateformes : des consoles de jeux, des appareils portatifs et des ordinateurs personnels (PC). À mesure que les joueurs jouent à ces jeux, des volumes importants de données de journalisation sont générés et suivent les modèles d’utilisation, le style de jeu et les préférences de l’utilisateur.  Combiné à des données démographiques, régionales et de produits, Contoso peut effectuer une analyse pour les guider sur la façon d’améliorer l’expérience des joueurs et leur recommander des mises à niveau et des achats intégrés. 

L’objectif de Contoso est d’identifier des opportunités de vente incitative/croisée basées sur l’historique du jeu des joueurs et d’ajouter de nouvelles fonctionnalités intéressantes, pour stimuler la croissance de l’entreprise et offrir une meilleure expérience aux clients. Pour ce cas d’utilisation, nous prenons l’exemple d’un développeur de jeux. L’entreprise souhaite optimiser ses jeux en fonction du comportement des joueurs. Ces principes s’appliquent à toute entreprise cherchant à fidéliser ses clients autour de ses produits et services et à améliorer leur expérience d’achat.

Dans cette solution, Contoso souhaite évaluer l’efficacité d’une campagne marketing qu'elle a lancée récemment. Nous commençons avec des journaux d’activité de jeu bruts, les traitons et les enrichissons avec des données de géolocalisation, les joignons avec des données de référence publicitaires et enfin les copions dans Azure SQL Database pour analyser l’impact de la campagne.

## <a name="deploy-solution"></a>Déployer la solution
Pour accéder à ce cas d’utilisation simple et l’essayer, il vous suffit d’un [abonnement Azure](https://azure.microsoft.com/pricing/free-trial/), d’un [compte de stockage d’objets Blob Azure](../../storage/common/storage-account-create.md) et d’une [base de données Azure SQL Database](../../azure-sql/database/single-database-create-quickstart.md). Vous déployez le pipeline de profilage de client à partir du panneau **Exemples de pipelines** sur la page d’accueil de votre Data Factory.

1. Créez une fabrique de données ou ouvrez une fabrique de données existante. Pour connaître les étapes de création d’une fabrique de données, consultez [Copie de données Blob Storage vers une base de données SQL à l’aide de Data Factory](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
2. Dans le panneau **DATA FACTORY** de la fabrique de données, cliquez sur la vignette **Exemples de pipelines**.

    ![Vignette Exemples de pipelines](./media/data-factory-samples/SamplePipelinesTile.png)
3. Dans le panneau **Exemples de pipelines**, cliquez sur le **Profilage des utilisateurs** que vous souhaitez déployer.

    ![Panneau Exemples de pipelines](./media/data-factory-samples/SampleTile.png)
4. Spécifiez les paramètres de configuration de l’exemple. Par exemple, votre clé et votre nom de compte de stockage Azure, le nom du serveur SQL logique, la base de données, l’ID d’utilisateur et le mot de passe.

    ![Panneau Exemple](./media/data-factory-samples/SampleBlade.png)
5. Une fois que vous avez fini de spécifier les paramètres de configuration, cliquez sur **Créer** pour créer/déployer les exemples de pipelines et les tables/services liés utilisés par les pipelines.
6. L’état du déploiement s’affiche sur l’exemple de mosaïque sur laquelle vous avez cliqué dans le panneau **Exemples de pipelines**.

    ![état du déploiement](./media/data-factory-samples/DeploymentStatus.png)
7. Lorsque vous voyez le message **Déploiement réussi** sur la mosaïque de l’exemple, fermez le panneau **Exemples de pipelines**.  
8. Dans le panneau **DATA FACTORY**, vous pouvez voir que les services liés, les jeux de données et les pipelines sont ajoutés à votre fabrique de données.  

    ![Panneau Data Factory](./media/data-factory-samples/DataFactoryBladeAfter.png)

## <a name="solution-overview"></a>Vue d'ensemble de la solution
Ce cas d'utilisation simple peut servir d’exemple montrant comment utiliser Azure Data Factory pour recevoir, préparer, transformer, analyser et publier des données.

![Workflow de bout en bout](./media/data-factory-customer-profiling-usecase/EndToEndWorkflow.png)

Cette figure montre comment les pipelines de données apparaissent dans l’interface utilisateur du portail Azure après leur déploiement.

1. Le pipeline **PartitionGameLogsPipeline** lit les événements de jeu bruts depuis un stockage d’objets blob et crée des partitions basées sur l’année, le mois et le jour.
2. Le pipeline **EnrichGameLogsPipeline** associe des événements de jeu partitionnés à des données de référence de code géographique et les enrichit en mappant les adresses IP avec les emplacements géographiques correspondants.
3. Le pipeline **AnalyzeMarketingCampaignPipeline** utilise les données enrichies et les traite avec les données de publicité pour créer la sortie finale contenant l’efficacité de la campagne marketing.

Dans cet exemple, Data Factory sert à orchestrer les activités qui copient les données d’entrée, transforment et traitent les données, et à générer les données finales dans une base de données Azure SQL.  Vous pouvez également visualiser le réseau des pipelines de données, les gérer et surveiller leur état à partir de l'interface utilisateur.

## <a name="benefits"></a>Avantages
En optimisant l’analyse du profil de ses utilisateurs et en l’alignant sur ses objectifs commerciaux, le développeur de jeux peut recueillir rapidement des modèles d’utilisation et analyser l’efficacité de ses campagnes marketing.