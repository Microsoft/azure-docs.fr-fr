---
title: Sources de données prises en charge par Azure Analysis Services | Microsoft Docs
description: Décrit les sources de données et les connecteurs pris en charge pour les modèles de données tabulaires 1200 et versions ultérieures dans Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 08/21/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 61efc7719b071ff4e8e5c0e07534b72a2883aff1
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96458872"
---
# <a name="data-sources-supported-in-azure-analysis-services"></a>Sources de données prises en charge dans Azure Analysis Services

Les sources de données et connecteurs figurant dans Obtenir des données ou l’Assistant Importation de table de Visual Studio avec les projets Analysis Services s’affichent à la fois pour Azure Analysis Services et SQL Server Analysis Services. Cependant, toutes les sources de données et tous les connecteurs affichés ne sont pas pris en charge dans Azure Analysis Services. Les types de sources de données auxquelles vous pouvez vous connecter dépendent de nombreux facteurs, comme le niveau de compatibilité du modèle, les connecteurs de données disponibles, le type d’authentification et la prise en charge de la passerelle de données locale. Les tableaux suivants décrivent les sources de données prises en charge pour Azure Analysis Services.

## <a name="azure-data-sources"></a>Sources de données Azure

|Source de données  |En mémoire  |DirectQuery  |Notes |
|---------|---------|---------|---------|
|Azure SQL Database      |   Oui      |    Oui      |<sup>[2](#azprovider)</sup>, <sup>[3](#azsqlmanaged)</sup>|
|Azure Synapse Analytics (SQL DW)      |   Oui      |   Oui       |<sup>[2](#azprovider)</sup>|
|Stockage Blob Azure      |   Oui       |    Non      | <sup>[1](#tab1400a)</sup> |
|Stockage de table Azure     |   Oui       |    Non      | <sup>[1](#tab1400a)</sup>|
|Azure Cosmos DB     |  Oui        |  Non        |<sup>[1](#tab1400a)</sup> |
|Azure Data Lake Store Gen1      |   Oui       |    Non      |<sup>[1](#tab1400a)</sup> |
|Azure Data Lake Store Gen2       |   Oui       |    Non      |<sup>[1](#tab1400a)</sup>, <sup>[5](#gen2)</sup>|
|Azure HDInsight HDFS    |     Oui     |   Non       |<sup>[1](#tab1400a)</sup> |
|Azure HDInsight Spark     |   Oui       |   Non       |<sup>[1](#tab1400a)</sup>, <sup>[4](#databricks)</sup>|
||||

**Remarques :**

<a name="tab1400a">1</a> - Modèles tabulaires 1400 ou supérieurs uniquement.  
<a name="azprovider">2</a> – Quand elle est spécifiée en tant que source de données *fournisseur* dans des modèles tabulaires 1200 et supérieurs, les modèles en mémoire et DirectQuery nécessitent MSOLEDBSQL de Microsoft OLE DB Driver pour SQL Server (recommandé), SQL Server Native Client 11.0 ou le Fournisseur de données .NET Framework pour SQL Server.  
<a name="azsqlmanaged">3</a> – Azure SQL Managed Instance est pris en charge. Comme l'instance gérée SQL s'exécute dans Azure VNet avec une adresse IP privée, le point de terminaison public doit être activé sur l'instance. Si cette option n’est pas activée, une [passerelle de données locale](analysis-services-gateway.md) est nécessaire.  
<a name="databricks">4</a> – Azure Databricks avec connecteur Spark n’est actuellement pas pris en charge.  
<a name="gen2">5</a> – Le connecteur ADLS Gen2 n’est pas pris en charge actuellement. Cependant, le connecteur Stockage Blob Azure peut être utilisé avec une source de données ADLS Gen2.

## <a name="other-data-sources"></a>Autres sources de données

|Source de données | En mémoire | DirectQuery |Notes   |
|  --- | --- | --- | --- |
|Base de données Access     |  Oui | Non |  |
|Active Directory     |  Oui | Non | <sup>[6](#tab1400b)</sup>  |
|Analysis Services     |  Oui | Non |  |
|Système de la plateforme d'analyse     |  Oui | Non |  |
|Fichier CSV  |Oui | Non |  |
|Dynamics 365     |  Oui | Non | <sup>[6](#tab1400b)</sup> |
|Classeur Excel     |  Oui | Non |  |
|Exchange      |  Oui | Non | <sup>[6](#tab1400b)</sup> |
|Dossier      |Oui | Non | <sup>[6](#tab1400b)</sup> |
|IBM Informix  |Oui | Non |  |
|Document JSON      |  Oui | Non | <sup>[6](#tab1400b)</sup> |
|Lignes issues d’un fichier binaire      | Oui | Non | <sup>[6](#tab1400b)</sup> |
|Base de données MySQL     | Oui | Non |  |
|Flux OData      |  Oui | Non | <sup>[6](#tab1400b)</sup> |
|Requête ODBC     | Oui | Non |  |
|OLE DB     |   Oui | Non |  |
|Oracle  | Oui  |Oui  | <sup>[9](#oracle)</sup> |
|Base de données PostgreSQL   | Oui | Non | <sup>[6](#tab1400b)</sup> |
|Objets Salesforce|  Oui | Non | <sup>[6](#tab1400b)</sup> |
|Rapports Salesforce |Oui | Non | <sup>[6](#tab1400b)</sup> |
|SAP HANA     |  Oui | Non |  |
|SAP Business Warehouse    |  Oui | Non | <sup>[6](#tab1400b)</sup> |
|Liste SharePoint      |   Oui | Non | <sup>[6](#tab1400b)</sup>, <sup>[11](#filesSP)</sup> |
|SQL Server |Oui   | Oui  | <sup>[7](#sqlim)</sup>, <sup>[8](#instgw)</sup> |
|SQL Server Data Warehouse |Oui   | Oui  | <sup>[7](#sqlim)</sup>, <sup>[8](#instgw)</sup> |
|Base de données Sybase     |  Oui | Non |  |
|Teradata | Oui  | Oui  | <sup>[10](#teradata)</sup> |
|Fichier TXT  |Oui | Non |  |
|Table XML    |  Oui | Non | <sup>[6](#tab1400b)</sup> |
| | | |

**Remarques :**  
<a name="tab1400b">6</a> – Modèles tabulaires 1400 ou supérieurs uniquement.  
<a name="sqlim">7</a> – Quand elle est spécifiée en tant que source de données *fournisseur* dans des modèles tabulaires 1200 et supérieurs, spécifiez MSOLEDBSQL de Microsoft OLE DB Driver pour SQL Server (recommandé), SQL Server Native Client 11.0 ou le Fournisseur de données .NET Framework pour SQL Server.  
<a name="instgw">8</a> – Si vous spécifiez MSOLEDBSQL comme fournisseur de données, il peut être nécessaire de télécharger et installer [Microsoft OLE DB Driver pour SQL Server](/sql/connect/oledb/oledb-driver-for-sql-server) sur le même ordinateur que la passerelle de données locale.  
<a name="oracle">9</a> – Pour les modèles tabulaires 1200 ou pour une source de données *fournisseur* de modèles tabulaires 1400 et supérieurs, spécifiez Oracle Data Provider for .NET. S’il est spécifié comme source de données structurées, assurez-vous d’[activer le fournisseur managé Oracle](#enable-oracle-managed-provider).   
<a name="teradata">10</a> – Pour les modèles tabulaires 1200 ou pour une source de données *fournisseur* de modèles tabulaires 1400 et supérieurs, spécifiez Teradata Data Provider for .NET.  
<a name="filesSP">11</a> – Les fichiers contenus dans SharePoint en local ne sont pas pris en charge.

La connexion aux sources de données locales à partir d’un serveur Azure Analysis Services nécessite une [passerelle locale](analysis-services-gateway.md). Lorsque vous utilisez une passerelle, des fournisseurs 64 bits sont requis.

## <a name="understanding-providers"></a>Présentation des fournisseurs

Quand vous créez des projets de modèle tabulaire 1400 et supérieurs dans Visual Studio, par défaut, vous ne spécifiez pas de fournisseur de données au moment d’établir une connexion à une source de données à l’aide de **Obtenir des données**. Les modèles tabulaires 1400 et supérieurs utilisent des connecteurs [Power Query](/power-query/power-query-what-is-power-query) pour gérer les connexions, les requêtes de données et les mashups entre la source de données et Analysis Services. Ils sont parfois appelés connexions de source de données *structurée* dans le sens où les paramètres des propriétés de connexion sont définis automatiquement. Toutefois, vous pouvez activer des sources de données héritées pour un projet de modèle dans Visual Studio. Dans ce cas, vous pouvez utiliser l’**Assistant Importation de table** pour vous connecter à certaines sources de données traditionnellement prises en charge dans les modèles tabulaires 1200 et inférieurs en tant que sources de données *héritées* ou *fournisseur*. Quand il est spécifié en tant que source de données fournisseur, vous pouvez spécifier un fournisseur de données particulier et d’autres propriétés de connexion avancées. Par exemple, vous pouvez vous connecter à une instance SQL Server Data Warehouse ou même à une base de données Azure SQL Database en guise de source de données héritée. Vous pouvez alors sélectionner fournisseur de données MSOLEDBSQL OLE DB Driver pour SQL Server. Dans ce cas, la sélection d’un fournisseur de données OLE DB peut offrir de meilleures performances que le connecteur Power Query. 

Quand l’Assistant Importation de table de Visual Studio est utilisé, un fournisseur de données est nécessaire pour se connecter à une source de données. Un fournisseur de données par défaut est alors sélectionné automatiquement. Vous pouvez changer de fournisseur de données si nécessaire. Le type de fournisseur choisi peut dépendre des performances, que le modèle utilise le stockage en mémoire ou DirectQuery, et de la plateforme Analysis Services sur laquelle vous déployez votre modèle.

### <a name="specify-provider-data-sources-in-tabular-1400-and-higher-model-projects"></a>Spécifier des sources de données fournisseur dans les projets de modèle tabulaire 1400 et supérieur

Pour activer des sources de données fournisseur, dans Visual Studio, cliquez sur **Outils** > **Options** > **Analysis Services Tabular** > **Importation des données**, sélectionnez **Activer les sources de données héritées**.

![Activer les sources de données héritées](media/analysis-services-datasource/aas-enable-legacy-datasources.png)

Une fois les sources de données héritées activées, dans **Explorateur de modèles tabulaires**, cliquez avec le bouton droit sur **Sources de données** > **Importer depuis une source de données (hérité)** .

![Sources de données héritées dans l’Explorateur de modèles tabulaires](media/analysis-services-datasource/aas-import-legacy-datasources.png)

À l’instar des projets de modèle tabulaire 1200, utilisez l’**Assistant Importation de table** pour vous connecter à une source de données. Dans la page de connexion, cliquez sur **Avancé**. Spécifiez le fournisseur de données et les autres paramètres de connexion dans **Définir les propriétés avancées**.

![Sources de données héritées, propriétés avancées](media/analysis-services-datasource/aas-import-legacy-advanced.png)

## <a name="impersonation"></a>Emprunt d'identité
Dans certains cas, il peut être nécessaire de spécifier un autre compte d’emprunt d’identité. Le compte d’emprunt d’identité peut être spécifié dans Visual Studio ou SQL Server Management Studio (SSMS).

Pour les sources de données locales :

* Si vous utilisez l’authentification SQL, l’emprunt d’identité doit être le compte de service.
* Si vous utilisez l’authentification Windows, définissez l’utilisateur/le mot de passe Windows. Pour SQL Server, l’authentification Windows avec un compte d’emprunt d’identité spécifique est pris en charge pour les modèles de données en mémoire uniquement.

Pour les sources de données cloud :

* Si vous utilisez l’authentification SQL, l’emprunt d’identité doit être le compte de service.

## <a name="oauth-credentials"></a>Informations d’identification OAuth

Pour les modèles tabulaires au niveau de compatibilité 1400 ou plus utilisant le mode en mémoire, Azure SQL Database, Azure Synapse, Dynamics 365 et la liste SharePoint prennent en charge les informations d’identification OAuth. Azure Analysis Services gère l’actualisation des jetons pour les sources de données OAuth afin d’éviter un dépassement des délais d’expiration pour les opérations d’actualisation de longue durée. Pour générer des jetons valides, définissez les informations d'identification à l'aide de Power Query.

Le mode de requête directe n’est pas compatible avec les informations d’identification OAuth.

## <a name="enable-oracle-managed-provider"></a>Activer le fournisseur managé Oracle

Dans certains cas, les requêtes DAX vers une source de données Oracle peuvent retourner des résultats inattendus. Cela peut être dû au fournisseur utilisé pour la connexion de la source de données.

Comme décrit dans la section [Compréhension des fournisseurs](#understanding-providers), les modèles tabulaires se connectent aux sources de données en tant que source de données *structurées* ou source de données *fournisseur*. Pour les modèles disposant d’une source de données Oracle spécifiée comme source de données fournisseur, assurez-vous que le fournisseur spécifié est le fournisseur de données Oracle pour .NET (Oracle.DataAccess.Client). 

Si la source de données Oracle est spécifiée comme source de données structurées, activez la propriété de serveur **MDataEngine\UseManagedOracleProvider**. Définir cette propriété assure la connexion de votre modèle à la source de données Oracle à l’aide du fournisseur de données Oracle recommandé comme fournisseur managé .NET.
 
Pour activer le fournisseur managé Oracle :

1. Dans SQL Server Management Studio, connectez-vous à votre serveur.
2. Créez une requête XMLA avec le script suivant. Remplacez **ServerName** par le nom complet du serveur, puis exécutez la requête.

    ```xml
    <Alter AllowCreate="true" ObjectExpansion="ObjectProperties" xmlns="http://schemas.microsoft.com/analysisservices/2003/engine">
        <Object />
        <ObjectDefinition>
            <Server xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:ddl2="http://schemas.microsoft.com/analysisservices/2003/engine/2" xmlns:ddl2_2="http://schemas.microsoft.com/analysisservices/2003/engine/2/2" 
    xmlns:ddl100_100="http://schemas.microsoft.com/analysisservices/2008/engine/100/100" xmlns:ddl200="http://schemas.microsoft.com/analysisservices/2010/engine/200" xmlns:ddl200_200="http://schemas.microsoft.com/analysisservices/2010/engine/200/200" 
    xmlns:ddl300="http://schemas.microsoft.com/analysisservices/2011/engine/300" xmlns:ddl300_300="http://schemas.microsoft.com/analysisservices/2011/engine/300/300" xmlns:ddl400="http://schemas.microsoft.com/analysisservices/2012/engine/400" 
    xmlns:ddl400_400="http://schemas.microsoft.com/analysisservices/2012/engine/400/400" xmlns:ddl500="http://schemas.microsoft.com/analysisservices/2013/engine/500" xmlns:ddl500_500="http://schemas.microsoft.com/analysisservices/2013/engine/500/500">
                <ID>ServerName</ID>
                <Name>ServerName</Name>
                <ServerProperties>
                    <ServerProperty>
                        <Name>MDataEngine\UseManagedOracleProvider</Name>
                        <Value>1</Value>
                    </ServerProperty>
                </ServerProperties>
            </Server>
        </ObjectDefinition>
    </Alter>
    ```

3. Redémarrez le serveur.


## <a name="next-steps"></a>Étapes suivantes

* [Passerelle locale](analysis-services-gateway.md)
* [Gérer votre serveur](analysis-services-manage.md)