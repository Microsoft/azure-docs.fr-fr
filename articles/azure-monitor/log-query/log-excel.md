---
title: Intégrer Log Analytics et Excel
description: Obtenir une requête Log Analytics dans Excel et actualiser les résultats dans Excel.
ms.subservice: logs
ms.topic: conceptual
author: roygalMS
ms.author: roygal
ms.date: 11/03/2020
ms.openlocfilehash: d903d1bb16ba3576d0092979f1cc6b82fac1c0be
ms.sourcegitcommit: 4bee52a3601b226cfc4e6eac71c1cb3b4b0eafe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94507375"
---
# <a name="integrate-log-analytics-and-excel"></a>Intégrer Log Analytics et Excel

Vous pouvez intégrer Azure Monitor Log Analytics et Microsoft Excel à l’aide d’une requête M et de l’API Log Analytics.  Cette intégration vous permet d’envoyer 500 000 enregistrements à Excel.

> [!NOTE]
> Excel étant une application cliente locale, les limitations matérielles et logicielles locales ont un impact sur ses performances et sa capacité à traiter de grands ensembles de données.

## <a name="create-your-m-query-in-log-analytics"></a>Créer votre requête M dans Log Analytics 

1. **Créez et exécutez votre requête** dans Log Analytics comme vous le feriez normalement. Ne vous inquiétez pas si vous atteignez la limite de 10 000 enregistrements dans l’interface utilisateur.  Nous vous recommandons d’utiliser des dates relatives, telles que la fonction « ago » ou le sélecteur d’heure de l’interface utilisateur. Ainsi, Excel actualise le jeu de données approprié.
  
2. **Exporter une requête** : Une fois que vous êtes satisfait de la requête et de ses résultats, exportez la requête vers M en utilisant le choix de menu **Exporter vers Power BI (requête M)** de Log Analytics sous le menu *Exporter* :

:::image type="content" source="media/log-excel/export-query.png" alt-text="Requête Log Analytics avec les données et l’option d’exportation" border="true":::



En choisissant cette option, vous téléchargez un fichier .txt contenant le code M que vous pouvez utiliser dans Excel.

La requête présentée ci-dessus exporte le code M suivant. Voici un exemple de code M exporté pour la requête dans notre exemple :

```m
/*
The exported Power Query Formula Language (M Language ) can be used with Power Query in Excel
and Power BI Desktop.
For Power BI Desktop follow the instructions below: 
1) Download Power BI Desktop from https://powerbi.microsoft.com/desktop/
2) In Power BI Desktop select: 'Get Data' -> 'Blank Query'->'Advanced Query Editor'
3) Paste the M Language script into the Advanced Query Editor and select 'Done'
*/

let AnalyticsQuery =
let Source = Json.Document(Web.Contents("https://api.loganalytics.io/v1/workspaces/ddcfc599-cae0-48ee-9026-fffffffffffff/query", 
[Query=[#"query"="

Heartbeat 
| summarize dcount(ComputerIP) by bin(TimeGenerated, 1h)    
| render timechart
",#"x-ms-app"="OmsAnalyticsPBI",#"timespan"="P1D",#"prefer"="ai.response-thinning=true"],Timeout=#duration(0,0,4,0)])),
TypeMap = #table(
{ "AnalyticsTypes", "Type" }, 
{ 
{ "string",   Text.Type },
{ "int",      Int32.Type },
{ "long",     Int64.Type },
{ "real",     Double.Type },
{ "timespan", Duration.Type },
{ "datetime", DateTimeZone.Type },
{ "bool",     Logical.Type },
{ "guid",     Text.Type },
{ "dynamic",  Text.Type }
}),
DataTable = Source[tables]{0},
Columns = Table.FromRecords(DataTable[columns]),
ColumnsWithType = Table.Join(Columns, {"type"}, TypeMap , {"AnalyticsTypes"}),
Rows = Table.FromRows(DataTable[rows], Columns[name]), 
Table = Table.TransformColumnTypes(Rows, Table.ToList(ColumnsWithType, (c) => { c{0}, c{3}}))
in
Table
in AnalyticsQuery
```

## <a name="connect-query-to-excel"></a>Connexion d’une requête à Excel 

Pour importer la requête. 

1. Ouvrez Microsoft Excel. 
1. Dans le ruban, accédez au menu **Données**. Sélectionnez **Obtenir des données**. Dans **Autres sources**, sélectionnez **Requête vide** :
 
   :::image type="content" source="media/log-excel/excel-import-blank-query.png" alt-text="Option Importer à partir d’une requête vide dans Excel" border="true":::

1. Dans cette fenêtre Power Query, sélectionnez **Éditeur avancé**.

   :::image type="content" source="media/log-excel/advanced-editor.png" alt-text="Éditeur de requête avancé Excel" border="true":::

 
1. Remplacez le texte dans l’éditeur avancé par la requête exportée à partir de Log Analytics :

   :::image type="content" source="media/log-excel/advanced-editor-2.png" alt-text="Création d’une requête vide" border="true":::
 
1. Sélectionnez **Terminé**, puis **Charger et fermer**. Excel exécute la requête à l’aide de l’API Log Analytics et le jeu de résultats s’affiche alors.
 

   :::image type="content" source="media/log-excel/excel-query-result.png" alt-text="Résultats de la requête dans Excel" border="true":::

##  <a name="refreshing--data"></a>Actualisation des données

Vous pouvez actualiser vos données directement à partir d’Excel. Dans le groupe de menus **Données** du ruban Excel, sélectionnez le bouton **Actualiser**.
 
## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les intégrations d’Excel avec des sources de données externes, consultez [importer des données à partir de sources de données externes (Power Query)](https://support.office.com/article/import-data-from-external-data-sources-power-query-be4330b3-5356-486c-a168-b68e9e616f5a).