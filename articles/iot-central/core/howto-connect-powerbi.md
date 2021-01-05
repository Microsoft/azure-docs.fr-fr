---
title: Visualiser des données Azure IoT Central dans un tableau de bord Power BI | Microsoft Docs
description: Utilisez la solution Power BI pour Azure IoT Central pour visualiser et analyser vos données IoT Central.
ms.service: iot-central
services: iot-central
author: viv-liu
ms.author: viviali
ms.date: 10/4/2019
ms.topic: conceptual
ms.openlocfilehash: 191b57b08ba04844824dd5cf26875c21e494c5ef
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92123335"
---
# <a name="visualize-and-analyze-your-azure-iot-central-data-in-a-power-bi-dashboard"></a>Visualiser et analyser des données Azure IoT Central dans un tableau de bord Power BI

*Cette rubrique s’applique aux administrateurs et aux développeurs de solutions.*

:::image type="content" source="media/howto-connect-powerbi/iot-continuous-data-export.png" alt-text="Pipeline de solution Power BI":::

La Solution Power BI pour Azure IoT Central V3 vous permet de créer un tableau de bord Power BI puissant, qui effectue le monitoring des performances de vos appareils IoT. Ce tableau de bord Power BI vous permet d’effectuer les opérations suivantes :

- suivre la quantité de données envoyées par les appareils au fil du temps ;
- comparer des volumes de données entre différents flux de télémétrie ;
- filtrer les données envoyées par des appareils spécifiques ;
- afficher les données de télémétrie les plus récentes dans une table.

Cette solution configure un pipeline qui lit les données à partir de l’[Exportation de données continue](./howto-export-data.md) de votre compte de stockage d’objets blob Azure. Ce pipeline utilise Azure Functions, Azure Data Factory et Azure SQL Database pour traiter et transformer les données. Vous pouvez visualiser et analyser les données dans un rapport Power BI que vous téléchargez en tant que fichier PBIX. Toutes ces ressources étant créées dans votre abonnement Azure, vous pouvez personnaliser chaque composant en fonction de vos besoins.

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes décrites dans ce guide pratique, vous avez besoin d’un abonnement Azure actif. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

La configuration de la solution requiert les ressources suivantes :

- Application IoT Central version 3. Pour savoir comment vérifier la version de votre application, consultez [À propose de votre application](./howto-get-app-info.md). Pour savoir comment créer une application IoT Central, consultez [Créer une application Azure IoT Central](./quick-deploy-iot-central.md).
- Exportation de données continue configurée pour exporter la télémétrie, les appareils et les modèles d’appareils vers le service Stockage Blob Azure. Pour plus d’informations, consultez [Comment exporter des données IoT vers des destinations dans Azure](howto-export-data.md).
  - Assurez-vous que seule votre application IoT Central exporte des données vers le conteneur d’objets blob.
  - Vos [appareils doivent envoyer des messages encodés JSON](../../iot-hub/iot-hub-devguide-messages-d2c.md). Les appareils doivent spécifier `contentType:application/JSON` et `contentEncoding:utf-8` ou `contentEncoding:utf-16` ou `contentEncoding:utf-32` dans les propriétés système du message.
- Power BI Desktop (dernière version) Consultez [Téléchargements Power BI](https://powerbi.microsoft.com/downloads/).
- Power BI Pro (si vous souhaitez partager le tableau de bord avec d’autres utilisateurs).

> [!NOTE]
> Si vous utilisez une application de IoT Central version 2, consultez [Visualiser et analyser des données Azure IoT Central dans un tableau de bord Power BI](/previous-versions/azure/iot-central/core/howto-connect-powerbi) sur le site de documentation des versions précédentes.

## <a name="install"></a>Installer

Pour configurer le pipeline, accédez à la page [Solution Power BI pour Azure IoT Central V3](https://appsource.microsoft.com/product/web-apps/iot-central.power-bi-solution-iot-central) sur le site **Microsoft AppSource**. Sélectionnez **Obtenir maintenant**, puis suivez les instructions.

Lorsque vous ouvrez le fichier PBIX, veillez à lire et suivre les instructions de la page de garde. Ces instructions décrivent comment connecter votre rapport à votre base de données SQL.

## <a name="report"></a>Rapport

Le fichier PBIX contient le rapport **Appareils et de télémétrie** qui donne une vue historique de la télémétrie envoyée par les appareils. Il présente les différents types de télémétries, ainsi que la télémétrie la plus récente envoyée par les appareils.

:::image type="content" source="media/howto-connect-powerbi/report.png" alt-text="Pipeline de solution Power BI":::

## <a name="pipeline-resources"></a>Ressources de pipeline

Vous pouvez accéder à toutes les ressources Azure qui composent le pipeline dans le portail Azure. Toutes les ressources se trouvent dans le groupe de ressources que vous avez créé lors de la configuration du pipeline.

:::image type="content" source="media/howto-connect-powerbi/azure-deployment.png" alt-text="Pipeline de solution Power BI":::

La liste suivante décrit le rôle de chaque ressource dans le pipeline :

### <a name="azure-functions"></a>Azure Functions

L’application Azure Function déclenche chaque fois qu’IoT Central écrit un nouveau fichier dans le stockage d’objets blob. Les fonctions extraient les données des blobs de télémétrie, d’appareils et de modèles d’appareils pour remplir les tables SQL intermédiaires qu’Azure Data Factory utilise.

### <a name="azure-data-factory"></a>Azure Data Factory

Azure Data Factory se connecte à une base de données SQL en tant que service lié. Il exécute des procédures stockée pour traiter les données, et stocke celles-ci dans des tables d’analyse.

Azure Data Factory s’exécute toutes les 15 minutes pour transformer le dernier lot de données à charger dans les tables SQL (nombre minimal actuel pour le **Déclencheur de fenêtre bascule**).

### <a name="azure-sql-database"></a>Azure SQL Database

Azure Data Factory génère un ensemble de tables d’analyse pour Power BI. Vous pouvez explorer ces schémas dans Power BI et les utiliser pour générer vos propres visualisations.

## <a name="estimated-costs"></a>Coûts estimés

La page [Solution Power BI pour Azure IoT Central V3](https://appsource.microsoft.com/product/web-apps/iot-central.power-bi-solution-iot-central) sur le site Microsoft AppSource contient un lien vers un estimateur de coût pour les ressources que vous déployez.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à visualiser vos données dans Power BI, l’étape suivante suggérée consiste à découvrir [Comment gérer les appareils](howto-manage-devices.md).