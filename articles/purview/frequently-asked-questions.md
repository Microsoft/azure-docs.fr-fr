---
title: Forum Aux Questions (FAQ)
description: Cet article répond aux questions fréquemment posées sur Azure Purview.
author: SunetraVirdi
ms.author: suvirdi
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 10/20/2020
ms.openlocfilehash: eca0b9986c4da30adeeb02bc3d90d1e3d2892df7
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550146"
---
# <a name="frequently-asked-questions-faq-about-azure-purview"></a>Forum Aux Questions (FAQ) sur Azure Purview

## <a name="overview"></a>Vue d’ensemble

Beaucoup d'organisations manquent de visibilité sur leurs données. Elles ont du mal à déterminer de quelles données elles disposent, où se trouvent celles-ci et comment accéder aux données pertinentes. Les données manquent de contexte (traçabilité, classification, métadonnées complètes, etc.), ce qui complique la tâche des utilisateurs professionnels en matière de recherche de données et d'utilisation appropriée de celles-ci. En conséquence, seule une petite partie des données collectées est utilisée pour prendre des décisions professionnelles éclairées. Enfin, l'identification des problèmes de sécurité relatifs aux données et la protection des données sensibles ne sont pas cohérentes. Cela nécessite du temps et des efforts continus, tout en préservant l'agilité des données.

Azure Purview est une solution de gouvernance des données qui permet aux clients d'acquérir des connaissances approfondies sur l'ensemble de leurs données tout en conservant le contrôle de leur utilisation. Avec Azure Purview, les organisations découvrent et organisent les données. Ils bénéficient d'insights sur leur patrimoine de données, et régissent l'accès aux données de manière centralisée.

## <a name="purpose-of-this-faq"></a>Objectif de ce FAQ

Ce FAQ répond aux questions fréquemment posées par les clients et par les équipes de terrain. Il vise à clarifier les questions relatives à Azure Purview et aux solutions connexes, comme Azure Data Catalog (ADC) Gen 2 (déconseillé) et Azure Information Protection.

### <a name="what-are-the-source-types-available-for-metadata-scanning-and-classification"></a>Quels sont les types de sources disponibles pour l'analyse et la classification des métadonnées ?

|Azure|Non-Azure|
|---------|---------|
|Stockage Blob Azure|Power BI|
|Azure Synapse Analytics (SQL DW)|SQL Server |
|Azure Cosmos DB|Teradata (disponible fin 2020)|
|Azure SQL Managed Instance|SAP ECC (disponible fin 2020)|
|Explorateur de données Azure|SAP S/4 HANA (disponible fin 2020)|
|Azure Data Lake Storage Gen1|Hive Metastore (disponible fin 2020)|
|Azure Data Lake Storage Gen2|--|
|Azure Files|--|
|Azure SQL Database|--|

### <a name="what-data-systemsprocessors-can-we-connect-and-get-lineage"></a>Quels processeurs/systèmes de données pouvons-nous connecter pour bénéficier d'une traçabilité ?

|Processeur/système de données 
|---------
|Azure Data Factory : activité de copie, activité de flux de données 
|Traçabilité personnalisée   
|Azure Data Share   
|Power BI    |
|SQL Server Integration Services  

### <a name="how-are-adc-gen-2-azure-information-protection-and-azure-purview-related"></a>Quel est le lien entre ADC Gen 2, Azure Information Protection et Azure Purview ?

Azure Purview s'appelait initialement ADC Gen 2, mais son étendue a depuis été élargie. Il englobe désormais les fonctionnalités de catalogue avancées d'ADC Gen 2 ainsi que les fonctionnalités de classification des données, d'étiquetage et de mise en œuvre des stratégies de conformité d'Azure Information Protection. Aujourd'hui, il est plus étroitement aligné sur la définition élargie de la gouvernance des données.

### <a name="what-happens-to-customers-using-adc-gen-1"></a>Qu'advient-il des clients qui utilisent ADC Gen 1 ?

Azure Purview est au centre de toutes les innovations de produits dans l'espace des solutions de catalogue de Microsoft. ADC Gen 1 restera pris en charge.

### <a name="can-customers-have-multiple-azure-purview-accounts-in-the-same-subscription"></a>Les clients peuvent-ils disposer de plusieurs comptes Azure Purview dans le cadre d'un même abonnement ?

Oui, nous prenons en charge de nombreux comptes Azure Purview au sein d'un même abonnement et d'un même locataire.

### <a name="can-i-run-adc-gen-1-and-azure-purview-in-parallel"></a>Puis-je exécuter ADC Gen 1 et Azure Purview en parallèle ?

Oui. Ces deux services sont indépendants.

### <a name="how-do-i-migrate-existing-adc-gen-1-data-assets-to-azure-purview"></a>Comment migrer des ressources de données ADC Gen 1 existantes vers Azure Purview ?

Utilisez les API Azure Purview pour les extraire d'ADC Gen 1 et les ingérer dans Azure Purview. Pour le glossaire, nous prenons en charge les outils basés sur CSV.

### <a name="how-do-i-encrypt-sensitive-data-for-sql-tables-using-azure-purview"></a>Comment chiffrer les données sensibles des tables SQL à l'aide d'Azure Purview ?

Le chiffrement des données est effectué au niveau de la source de données. Azure Purview stocke uniquement les métadonnées. Il ne propose pas d'aperçu des données.

### <a name="will-all-the-capabilities-of-adc-gen-2-exist-in-azure-purview"></a>Toutes les fonctionnalités d'ADC Gen 2 seront-elles disponibles dans Azure Purview ?

Oui.

<!--## Is the data lineage feature available in Azure Purview?

Yes, but it's limited to the Azure Data Factory connector.

<!-- ## How can I scan SQL Server on-premises? 

Use the self-host integration runtime capability. !-->

<!--### What is the difference between classification in Azure SQL Database and classification in Azure Purview?

|Azure SQL DB classification  |Azure Purview classification  |
|---------|---------|
|Classification is based on SQL metadata from system catalogs. |Classification is based on Azure Purview's sampling technique by using the system-defined or custom-defined regex pattern.|
|Custom classification is supported.     |Custom classification is supported.         |
|Doesn't use Microsoft 365 system classifiers out of the box.    | Uses Microsoft 365 system classifiers out of the box.        |
-->

### <a name="whats-the-difference-between-a-glossary-and-classification"></a>Quelle est la différence entre un glossaire et une classification ?

Un glossaire utilise une convention d'affectation de noms suivie par les utilisateurs non techniques/professionnels des données, également appelés consommateurs de données. Ces personnes sont des analystes d'entreprise ou des scientifiques des données qui utilisent Azure Purview pour rechercher certains types de données, en fonction de l'utilisation professionnelle. Par exemple, les analystes de la chaîne logistique peuvent avoir besoin de rechercher les termes *types de références SKU* et *détails de l'expédition*. Ils recherchent ces termes dans le glossaire pour trouver des données pertinentes.
La classification est une étiquette appliquée à une ressource de données au niveau de la table, de la colonne ou du fichier, qui identifie les données contenues dans la ressource. La classification peut être appliquée automatiquement ou manuellement, en fonction du type de données trouvées. En général, vous utilisez des étiquettes de classification pour déterminer si une ressource contient des données sensibles, et de quel type de données sensibles il s'agit.

### <a name="does-azure-purview-scan-and-classify-emails-pdfs-etc-in-my-sharepoint-and-onedrive"></a>Azure Purview analyse-t-il et classe-t-il les e-mails, les PDF, etc. dans mes instances de Sharepoint et OneDrive ?

L'analyse des sites et bibliothèques SharePoint locaux est assurée par le scanneur Azure Information Protection. Le scanneur peut être utilisé dans le cadre de l'abonnement Microsoft 365 d'un client avec les références SKU suivantes : AIP P1, EMS E3 et M365 E3. Si vous possédez l'une de ces références SKU, vous devez disposer des droits appropriés pour commencer à utiliser le scanneur Azure Information Protection.

<!--### What is the difference between classifications and sensitivity labels in Azure Purview?

Azure Purview's data governance solution is based on the Apache Atlas framework. As defined by Atlas, classification is a way to identify the contents of an asset (table or file) or an entity (table column or structured file). This classification becomes a metadata property that allows Azure Purview to understand the data within each asset and govern and protect them.

Sensitivity labels are a Microsoft 365 concept that resembles classification at the asset level. You create a label with a collection of classifications applied at the asset or entity level.

Atlas-centric customers will see no real distinction between classifications and labels. To these customers, everything is a classification and labels aren't needed.

Security-focused customers will see a distinction between classification and labeling, but only because in Microsoft 365 the classifications aren't exposed directly to the user; only labels are visible. So, similar to Atlas, Office 365 security customers don't need to deal with both entities.
-->

### <a name="what-is-the-compute-used-for-the-scan"></a>Quel est le calcul utilisé pour l'analyse ?
Il existe une infrastructure d'analyse gérée par Microsoft. Pour la plupart des ressources Azure/AWS que nous prenons en charge, vous n'avez pas besoin de déployer une infrastructure d'analyse.

### <a name="is-there-a-way-to-provision-azure-purview-via-azure-resource-manager-arm-template--cli--powershell"></a>Est-il possible d'approvisionner Azure Purview via le modèle Azure Resource Manager (ARM)/CLI/PowerShell ?

Oui, le modèle ARM est disponible.

<!--### Does Azure Purview support guest users in AAD?-->

### <a name="im-already-using-atlas-can-i-easily-move-to-azure-purview"></a>J'utilise déjà Atlas, puis-je facilement migrer vers Azure Purview ?

Azure Purview est compatible avec l'API Atlas. Si vous migrez depuis Atlas, nous vous recommandons de commencer par analyser vos sources de données à l'aide d'Azure Purview. Une fois que les ressources sont disponibles sur votre compte, vous pouvez utiliser des API Atlas similaires pour les intégrer, par exemple en mettant à jour les ressources ou en ajoutant une traçabilité personnalisée. Azure Purview modifie l'API Recherche pour utiliser Recherche Azure. Vous devriez donc pouvoir utiliser Recherche avancée.

### <a name="can-i-create-multiple-catalogs-in-my-tenant"></a>Puis-je créer plusieurs catalogues au sein de mon locataire ?

Oui, vous pouvez créer plusieurs comptes Azure Purview par abonnement et par locataire. N'hésitez pas à consulter la page [Gérer et augmenter les quotas de ressources avec Azure Purview](how-to-manage-quotas.md).

Des recommandations supplémentaires sur les cas où vous devez ou ne devez pas disposer de plusieurs comptes sont proposées dans [Meilleures pratiques de déploiement d'Azure Purview ](deployment-best-practices.md).

### <a name="can-i-register-multiple-tenants-within-a-single-azure-purview-account"></a>Puis-je inscrire plusieurs locataires sur un même compte Azure Purview ?

Non, actuellement, pour analyser la source de données d'un autre locataire, vous devez créer un compte Azure Purview distinct dans ce locataire.

### <a name="does-azure-purview-support-column-level-lineage"></a>Azure Purview prend-il en charge la traçabilité au niveau des colonnes ?

Oui, Azure Purview prend en charge la traçabilité au niveau des colonnes.