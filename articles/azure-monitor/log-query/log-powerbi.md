---
title: Intégration de Log Analytics à Power BI et Excel
description: Comment envoyer des résultats de Log Analytics à Power BI
ms.subservice: logs
ms.topic: conceptual
author: roygalMS
ms.author: roygal
ms.date: 11/03/2020
ms.openlocfilehash: a4e2faf87ac3dddf91e6945343a46b02df72db0a
ms.sourcegitcommit: 4bee52a3601b226cfc4e6eac71c1cb3b4b0eafe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94507370"
---
# <a name="log-analytics-integration-with-power-bi"></a>Intégration de Log Analytics à Power BI

Cet article se concentre sur les moyens de transmettre des données de Log Analytics à Microsoft Power BI afin de créer des tableaux de bord et des rapports plus attrayants. 

## <a name="background"></a>Arrière-plan 

Les journaux Azure Monitor constituent une plateforme qui fournit une solution complète d’ingestion des journaux. [Azure Monitor Log Analytics](../platform/data-platform.md#) est l’interface qui permet d’interroger ces journaux. Pour plus d’informations sur l’ensemble de la plateforme de données Azure Monitor, y compris Log Analytics, consultez [Plateforme de données Azure Monitor](../platform/data-platform.md). 

Microsoft Power BI est une plateforme de visualisation de données de Microsoft. Pour savoir comment démarrer, consultez la [page d’accueil de Power BI](https://powerbi.microsoft.com/). 


En général, vous pouvez utiliser des fonctionnalités de Power BI gratuites pour intégrer et créer des rapports et des tableaux de bord attrayants.

Des fonctionnalités plus avancées peuvent nécessiter l’achat d’un compte Power BI Pro ou Premium. Ces fonctionnalités sont les suivantes : 
 - Partage de votre travail 
 - Actualisations planifiées
 - Applications Power BI 
 - Dataflows et actualisation incrémentielle 

Pour plus d’informations, découvrez [les tarifs et les fonctionnalités de Power BI](https://powerbi.microsoft.com/pricing/). 

## <a name="integrating-queries"></a>Intégration de requêtes  

Power BI utilise le [langage de requête M](/powerquery-m/power-query-m-language-specification/) comme langage de requête principal. 

Les requêtes Log Analytics peuvent être exportées vers M et utilisées directement dans Power BI. Après avoir exécuté une requête, sélectionnez **Exporter vers Power BI (requête M)** à partir du bouton **Exporter** dans la barre d’action supérieure de l’interface utilisateur de Log Analytics.

:::image type="content" source="media/log-powerbi-excel/export-query2.png" alt-text="Requête Log Analytics présentant les éléments du menu déroulant Exporter" border="true":::

Log Analytics crée un fichier .txt contenant le code M qui peut être utilisé directement dans Power BI.

## <a name="connecting-your-logs-to-a-dataset"></a>Connexion de vos journaux à un jeu de données 

Un jeu de données Power BI est une source de données prête pour la création de rapports et la visualisation. Pour connecter une requête Log Analytics à un jeu de données, copiez le code M exporté à partir de Log Analytics dans une requête vide au sein de Power BI. 

Pour plus d’informations, consultez [Compréhension des jeux de données Power BI](/power-bi/service-datasets-understand/). 

## <a name="collect-data-with-power-bi-dataflows"></a>Collecter des données avec des dataflows Power BI 

Les dataflows Power BI vous permettent également de collecter et de stocker des données. Pour plus d’informations, consultez [Dataflows Power BI](/power-bi/service-dataflows-overview).

Un dataflow est un type de procédure « ETL cloud » conçue pour vous aider à collecter et à préparer vos données. Un jeu de données est le « modèle » conçu pour vous aider à connecter différentes entités et à les modéliser en fonction de vos besoins.

## <a name="incremental-refresh"></a>Actualisation incrémentielle 

Les jeux de données Power BI et les dataflows Power BI ont une option d’actualisation incrémentielle. Les dataflows Power BI et les jeux de données Power BI prennent en charge cette fonctionnalité, mais vous avez besoin de Power BI Premium pour l’utiliser.  


L’actualisation incrémentielle exécute de petites requêtes et met à jour de plus petites quantités de données par exécution au lieu de réingérer systématiquement toutes les données quand vous exécutez la requête. Vous avez la possibilité d’enregistrer de grandes quantités de données, mais vous ajoutez un nouvel incrément de données chaque fois que la requête est exécutée. Ce comportement est idéal pour les rapports de plus longue durée.

L’actualisation incrémentielle dans Power BI s’appuie sur l’existence d’une *DateHeure* archivée dans le jeu de résultats. Avant de configurer l’actualisation incrémentielle, assurez-vous qu’au moins une *DateHeure* est archivée dans le jeu de résultats de votre requête Log Analytics. 

Pour plus d’informations et pour savoir comment configurer l’actualisation incrémentielle, consultez [Jeux de données et actualisation incrémentielle dans Power BI](/power-bi/service-premium-incremental-refresh) et [Dataflows et actualisation incrémentielle dans Power BI](/power-bi/service-dataflows-incremental-refresh).

## <a name="reports-and-dashboards"></a>Rapports et tableaux de bord

Une fois vos données envoyées à Power BI, vous pouvez continuer à utiliser Power BI pour créer des rapports et des tableaux de bord.

Pour plus d’informations, consultez [ce guide sur la création de vos premiers modèle et rapport Power BI](/learn/modules/build-your-first-power-bi-report/).  

## <a name="excel-integration"></a>Intégration d’Excel

Vous pouvez utiliser la même intégration M que celle utilisée dans Power BI pour effectuer une intégration à une feuille de calcul Excel. Pour plus d’informations, reportez-vous à ce [guide sur l’intégration à Excel](https://support.microsoft.com/office/import-data-from-external-data-sources-power-query-be4330b3-5356-486c-a168-b68e9e616f5a) puis collez la requête M exportée à partir de Log Analytics.

Vous trouverez des informations supplémentaires dans [Intégrer Log Analytics et Excel](log-excel.md).

## <a name="next-steps"></a>Étapes suivantes

Démarrez avec les [requêtes Log Analytics](log-query-overview.md).
