---
title: Qu’est-ce qu’Azure SQL Edge ?
description: Découvrez ce qu’est Azure SQL Edge
keywords: présentation de SQL Edge, qu'est-ce que SQL Edge, vue d'ensemble de SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: a74e12a5e9bdd2dfdbc6ac07b66798e517f6f426
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93395068"
---
# <a name="what-is-azure-sql-edge"></a>Qu’est-ce qu’Azure SQL Edge ?

Azure SQL Edge est un moteur de base de données relationnelle optimisé et conçu pour les déploiements IoT et IoT Edge. Il offre des fonctionnalités permettant de créer une couche de traitement et de stockage des données hautes performances pour les solutions et applications IoT. Azure SQL Edge propose des fonctionnalités qui permettent de diffuser, de traiter et d'analyser des données relationnelles et non relationnelles, telles que des données JSON, des données graphiques et des données de série chronologique, ce qui en fait un choix idéal pour une variété d'applications IoT modernes.

Azure SQL Edge est basé sur les dernières versions du [moteur de base de données SQL Server](/sql/sql-server/sql-server-technical-documentation), qui fournit des performances, une sécurité et des fonctionnalités de traitement des requêtes de pointe. Comme Azure SQL Edge utilise le même moteur que [SQL Server](/sql/sql-server/sql-server-technical-documentation) et [Azure SQL](../azure-sql/index.yml), il offre la même surface de programmation Transact-SQL (T-SQL), ce qui facilite et accélère le développement d’applications ou de solutions, et fournit une portabilité directe des applications entre les appareils IoT Edge, les centres de données et le cloud.

Vidéo de présentation d’Azure SQL Edge sur Channel 9 :
> [!VIDEO https://channel9.msdn.com/Shows/Data-Exposed/What-is-Azure-SQL-Edge/player]

## <a name="deployment-models"></a>Modèles de déploiement

Azure SQL Edge prend en charge deux modes de déploiement.

- Déploiement connecté via Azure IoT Edge : Azure SQL Edge est disponible sur la Place de marché Azure et peut être déployé en tant que module pour [Azure IoT Edge](../iot-edge/about-iot-edge.md). Pour plus d'informations, consultez [Déployer Azure SQL Edge](deploy-portal.md).<br>

![Diagramme de présentation de SQL Edge](media/overview/overview.png)

- Déploiement déconnecté : Les images conteneur Azure SQL Edge peuvent être tirées du hub Docker, puis déployées en tant que conteneur Docker autonome ou sur un cluster Kubernetes. Pour plus d’informations, consultez [Déployer Azure SQL Edge avec Docker](disconnected-deployment.md) et [Déployer un conteneur Azure SQL Edge dans Kubernetes](deploy-kubernetes.md).

## <a name="editions-of-sql-edge"></a>Éditions de SQL Edge

Deux éditions ou plans logiciels DE SQL Edge sont disponibles. Ces éditions incluent des ensembles de fonctionnalités identiques et diffèrent uniquement en termes de droits d’utilisation et de quantité d’UC/de mémoire qu’elles prennent en charge.

   |**Planification**  |**Description**  |
   |---------|---------|
   |Azure SQL Edge Développeur  |  Référence SKU de développement uniquement, chaque conteneur SQL Edge est limité à 4 cœurs et à 32 Go de mémoire.  |
   |Azure SQL Edge    |  Référence SKU de production, chaque conteneur SQL Edge est limité à 8 cœurs et à 64 Go de mémoire. |

## <a name="pricing-and-availability"></a>Tarification et disponibilité

Azure SQL Edge est maintenant en disponibilité générale. Pour plus d'informations sur la tarification et la disponibilité dans les régions, consultez [Azure SQL Edge](https://azure.microsoft.com/services/sql-edge/).

> [!IMPORTANT]
> Pour comprendre les différences de fonctionnalités entre Azure SQL Edge et SQL Server, ainsi que les différences entre les diverses options d'Azure SQL Edge, consultez [Fonctionnalités d'Azure SQL Edge prises en charge](features.md).

## <a name="streaming-capabilities"></a>Fonctionnalités de diffusion en continu  

Azure SQL Edge fournit des fonctionnalités de diffusion en continu intégrées pour l'analyse en temps réel et les traitements d'événements complexes. La fonctionnalité de diffusion en continu repose sur les mêmes constructions qu'[Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md) et sur des fonctionnalités semblables à celles d'[Azure Stream Analytics sur IoT Edge](../stream-analytics/stream-analytics-edge.md).

Le moteur de diffusion en continu d'Azure SQL Edge est conçu pour ce qui suit : latence faible, résilience, utilisation efficace de la bande passante et conformité. 

Pour plus d'informations sur la diffusion de données en continu dans SQL Edge, consultez [Streaming de données](stream-data.md).

## <a name="machine-learning-and-artificial-intelligence-capabilities"></a>Fonctionnalités de Machine Learning et d’intelligence artificielle

Azure SQL Edge fournit des fonctionnalités de Machine Learning et d'analyse intégrées via le runtime ONNX (Open Neural Network Exchange) au format ouvert, qui permet d'échanger des modèles de Deep Learning et de réseau neuronal entre différentes infrastructures. Pour plus d’informations sur ONNX, cliquez [ici](https://onnx.ai/). Le runtime ONNX permet de développer des modèles avec le langage ou les outils de votre choix, et ceux-ci peuvent ensuite être convertis au format ONNX pour être exécutés dans SQL Edge. Pour plus d'informations, consultez [Machine Learning et intelligence artificielle avec ONNX dans SQL Edge](onnx-overview.md).

## <a name="working-with-azure-sql-edge"></a>Utiliser Azure SQL Edge

Azure SQL Edge simplifie le développement et la gestion des applications et vous fait gagner en productivité. Les utilisateurs peuvent utiliser tous les outils et compétences familiers pour créer des applications et des solutions exceptionnelles répondant à leurs besoins en IoT Edge. L'utilisateur peut développer dans SQL Edge à l'aide d'outils tels que :

- [Portail Azure](https://portal.azure.com/) : application web dédiée à la gestion de tous les services Azure.
- [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms/) : application cliente téléchargeable gratuitement dédiée à la gestion des infrastructures SQL, allant de SQL Server à SQL Database.
- [SQL Server Data Tools dans Visual Studio](/sql/ssdt/download-sql-server-data-tools-ssdt/) : application cliente téléchargeable gratuitement, dédiée au développement de bases de données relationnelles SQL Server, de bases de données SQL, de packages Integration Services, de modèles de données Analysis Services et de rapports Reporting Services.
- [Azure Data Studio](/sql/azure-data-studio/what-is/) : outil de base de données multiplateforme téléchargeable gratuitement pour les professionnels des données qui utilisent la famille Microsoft de plateformes de données locales et cloud sous Windows, MacOS et Linux.
- [Visual Studio Code](https://code.visualstudio.com/docs) : éditeur de code open source téléchargeable gratuitement pour Windows, macOS et Linux. Il prend en charge les extensions, notamment l’[extension mssql](https://aka.ms/mssql-marketplace) pour l’exécution de requêtes dans Microsoft SQL Server, Azure SQL Database et Azure Synapse Analytics.


## <a name="next-steps"></a>Étapes suivantes

- [Déployer SQL Edge via le portail Azure](deploy-portal.md)
- [Machine Learning et intelligence artificielle avec SQL Edge](onnx-overview.md)
- [Générer une solution IoT de bout en bout avec SQL Edge](tutorial-deploy-azure-resources.md)