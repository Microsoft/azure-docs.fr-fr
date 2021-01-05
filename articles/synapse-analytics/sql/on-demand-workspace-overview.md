---
title: Pool SQL serverless
description: Découvrez le pool SQL serverless dans Azure Synapse Analytics.
services: synapse analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 34ce36f0ff348f896b7c2ea680c113b5e9e4ea09
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96463170"
---
# <a name="serverless-sql-pool-in-azure-synapse-analytics"></a>Pool SQL serverless dans Azure Synapse Analytics 

Chaque espace de travail Azure Synapse Analytics est fourni avec des points de terminaison de pool SQL serverless que vous pouvez utiliser pour interroger les données du lac.

Le pool SQL serverless est un service de requête sur les données de votre lac de données. Il vous permet d’accéder à vos données par le biais des fonctionnalités suivantes :
 
- Une syntaxe T-SQL familière pour interroger des données sur place sans avoir besoin de les copier ou de les charger dans un magasin spécialisé 
- Une connectivité intégrée via l’interface T-SQL qui offre un large éventail d’outils décisionnels et d’outils de requête ad hoc, y compris les pilotes les plus populaires 

Le pool SQL serverless est un système de traitement de données distribué, conçu pour des fonctions de calcul et de données à grande échelle. Le pool SQL serverless vous permet d’analyser vos données Big Data en quelques secondes ou quelques minutes, selon la charge de travail. Grâce à la tolérance de panne de l’exécution intégrée des requêtes, le système fournit des taux de réussite et de fiabilité élevés, même pour les requêtes longues qui impliquent des jeux de données volumineux.

Le pool SQL serverless est serverless : il n’y a donc pas d’infrastructure à configurer ni de clusters à gérer. Pour ce service, un point de terminaison par défaut est fourni dans chaque espace de travail Azure Synapse. Vous pouvez donc commencer à interroger les données dès que l’espace de travail est créé. 

Les ressources réservées n’impliquent aucuns frais. Seules les données traitées par les requêtes que vous exécutez vous sont facturées. Par conséquent, ce modèle est un véritable modèle de paiement à l’utilisation.  

Si vous utilisez Apache Spark pour Azure Synapse dans votre pipeline de données, pour la préparation, le nettoyage ou l’enrichissement des données, vous pouvez [interroger les tables externes Spark](develop-storage-files-spark-tables.md) que vous avez créées durant ce processus, directement à partir du pool SQL serverless. Utilisez [Private Link](../security/how-to-connect-to-workspace-with-private-links.md) pour placer votre point de terminaison de pool SQL serverless dans le [réseau virtuel de votre espace de travail managé](../security/synapse-workspace-managed-vnet.md).  

## <a name="serverless-sql-pool-benefits"></a>Avantages du pool SQL serverless

Si vous devez explorer les données du lac de données, obtenir des insights à partir de celles-ci ou optimiser votre pipeline de transformation de données existant, vous pouvez tirer parti de l’utilisation du pool SQL serverless. Il convient aux scénarios suivants :

- Découverte et exploration de base : comprenez rapidement les données de différents formats (Parquet, CSV, JSON) présentes dans votre lac de données, afin de planifier l’extraction d’insights à partir de celles-ci.
- Entrepôt de données logique : fournissez une abstraction relationnelle pour les données brutes ou disparates, sans déplacer ni transformer ces données, afin de toujours avoir une vue de vos données qui soit actuelle.
- Transformation des données : méthode simple, scalable et performante pour transformer les données d’un lac à l’aide de T-SQL, en vue de les envoyer vers des outils décisionnels ou autre, ou en vue de les charger dans un magasin de données relationnelles (bases de données Synapse SQL, Azure SQL Database, etc.).

Différents rôles professionnels peuvent tirer parti du pool SQL serverless :

- Les ingénieurs des données peuvent explorer le lac, transformer et préparer les données à l’aide de ce service, et simplifier leurs pipelines de transformation des données. Pour plus d’informations, consultez ce [tutoriel](tutorial-data-analyst.md).
- Les scientifiques des données peuvent rapidement comprendre le contenu et la structure des données du lac, grâce à des fonctionnalités telles que OPENROWSET et l’inférence de schéma automatique.
- Les analystes de données peuvent [explorer les données et les tables externes Spark](develop-storage-files-spark-tables.md) créées par les scientifiques des données ou les ingénieurs de données en utilisant le langage T-SQL bien connu ou leurs outils favoris qui peuvent se connecter au pool SQL serverless.
- Les professionnels du décisionnel peuvent rapidement [créer des rapports Power BI à partir des données du lac](tutorial-connect-power-bi-desktop.md) et des tables Spark.

## <a name="how-to-start-using-serverless-sql-pool"></a>Comment commencer à utiliser un pool SQL serverless

Un point de terminaison de pool SQL serverless est fourni dans chacun des espaces de travail Azure Synapse. Vous pouvez créer un espace de travail et commencer à interroger les données instantanément à l’aide des outils que vous connaissez.

## <a name="client-tools"></a>Outils clients

Le pool SQL serverless permet aux outils de requêtes SQL ad hoc et aux outils décisionnels existants d’exploiter les données du lac. Étant donné qu’il fournit une syntaxe T-SQL que vous connaissez déjà, tous les outils capables d’établir des connexions TDS SQL peuvent [se connecter à Synapse SQL et interroger ses données](connect-overview.md) à la demande. Vous pouvez vous connecter avec Azure Data Studio et exécuter des requêtes ad hoc, ou vous connecter avec Power BI pour obtenir des insights en quelques minutes.

## <a name="t-sql-support"></a>Prise en charge de T-SQL

Le pool SQL serverless offre une surface d’exposition de requête T-SQL qui a été légèrement améliorée et étendue sur certains points afin de prendre en charge l’interrogation des données semi-structurées et non structurées. De plus, certains aspects du langage T-SQL ne sont pas pris en charge en raison de la conception du pool SQL serverless ; par exemple, la fonctionnalité DML n’est pas prise en charge.

- La charge de travail peut être organisée à l’aide de concepts familiers :
- Bases de données : le point de terminaison du pool SQL serverless peut avoir plusieurs bases de données.
- Schémas : dans une base de données, il existe un ou plusieurs groupes d’appropriation d’objets appelés « schémas ».
- Les vues
- Ressources externes : sources de données, formats de fichiers et tables

La sécurité peut être appliquée à l’aide des éléments suivants :

- Connexions et utilisateurs
- Informations d’identification pour contrôler l’accès aux comptes de stockage
- Autorisations Grant, Deny et Revoke au niveau de chaque objet
- Intégration d’Azure Active Directory

T-SQL pris en charge :

- L’intégralité de la surface d’exposition [SELECT](/sql/t-sql/queries/select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) est prise en charge, y compris la majorité des fonctions SQL
- CETAS - CREATE EXTERNAL TABLE AS SELECT
- Instructions DDL relatives aux vues et à la sécurité uniquement

Le pool SQL serverless n’a pas de stockage local : seuls les objets de métadonnées sont stockés dans les bases de données. Par conséquent, le langage T-SQL qui est associé aux concepts suivants n’est pas pris en charge :

- Tables
- Déclencheurs
- Vues matérialisées
- Instructions DDL autres que celles relatives aux vues et à la sécurité
- Instructions DML

### <a name="extensions"></a>Extensions

Pour faciliter l’interrogation sur place des données qui se trouvent dans des fichiers du lac de données, le pool SQL serverless étend la fonction [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) existante en y ajoutant les fonctionnalités suivantes :

[Interrogation de plusieurs fichiers ou dossiers](query-data-storage.md#query-multiple-files-or-folders)

[Format de fichier PARQUET](query-data-storage.md#query-parquet-files)

[Options supplémentaires pour l’utilisation d’un texte délimité (marque de fin de champ, marque de fin de ligne, caractère d’échappement)](query-data-storage.md#query-csv-files)

[Lire un sous-ensemble choisi de colonnes](query-data-storage.md#read-a-chosen-subset-of-columns)

[Inférence de schéma](query-data-storage.md#schema-inference)

[Fonction filename](query-data-storage.md#filename-function)

[Fonction filepath](query-data-storage.md#filepath-function)

[Utilisation de types complexes et de structures de données imbriquées ou répétées](query-data-storage.md#work-with-complex-types-and-nested-or-repeated-data-structures)

## <a name="security"></a>Sécurité

Le pool SQL serverless offre des mécanismes permettant de sécuriser l’accès à vos données.

### <a name="azure-active-directory-integration-and-multi-factor-authentication"></a>Intégration d’Azure Active Directory et authentification multifacteur

Le pool SQL serverless vous permet de gérer de manière centralisée les identités des utilisateurs de bases de données et d’autres services Microsoft avec l’[intégration d’Azure Active Directory](../../azure-sql/database/authentication-aad-configure.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json). Cette fonctionnalité simplifie la gestion des autorisations et améliore la sécurité. Azure Active Directory (Azure AD) prend en charge l’[authentification multifacteur](../../azure-sql/database/authentication-mfa-ssms-configure.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) (MFA) pour augmenter la sécurité des données et des applications, ainsi qu’un processus d’authentification unique.

#### <a name="authentication"></a>Authentification

L’authentification du pool SQL serverless fait référence à la façon dont les utilisateurs prouvent leur identité quand ils se connectent au point de terminaison. Deux types d’authentifications sont pris en charge :

- **Authentification SQL**

  Cette méthode d’authentification utilise un nom d’utilisateur et un mot de passe.

- **Authentification Azure Active Directory** :

  Cette méthode d’authentification utilise les identités managées par Azure Active Directory. Pour les utilisateurs Azure AD, l’authentification multifacteur peut être activée. Utilisez l’authentification Active Directory (sécurité intégrée) [dans la mesure du possible](/sql/relational-databases/security/choose-an-authentication-mode?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

#### <a name="authorization"></a>Autorisation

Le terme « autorisation » fait référence aux actions qu’un utilisateur peut exécuter dans une base de données du pool SQL serverless ; elle est contrôlée par les appartenances aux rôles et par les autorisations au niveau objet de la base de données de votre compte d’utilisateur.

Quand l’authentification SQL est utilisée, l’utilisateur SQL existe seulement dans le pool SQL serverless et les autorisations sont limitées aux objets présents dans le pool SQL serverless. L’accès aux objets sécurisables d’autres services (comme Stockage Azure) ne peut pas être accordé directement à l’utilisateur SQL, car il n’existe que dans l’étendue du pool SQL serverless. Pour accéder aux fichiers, l’utilisateur SQL doit utiliser l’un des [types d’autorisations pris en charge](develop-storage-files-storage-access-control.md#supported-storage-authorization-types).

Si l’authentification Azure AD est utilisée, un utilisateur peut se connecter au pool SQL serverless et à d’autres services, comme Stockage Azure, et peut accorder des autorisations à l’utilisateur Azure AD.

### <a name="access-to-storage-accounts"></a>Accès aux comptes de stockage

Un utilisateur connecté au service de pool SQL serverless doit être autorisé à accéder aux fichiers présents dans Stockage Azure et à les interroger. Le pool SQL serverless prend en charge les types d’autorisation suivants :

- Une **signature d’accès partagé (SAS)** fournit un accès délégué aux ressources du compte de stockage. Avec une signature d’accès partagé, vous pouvez accorder aux clients l’accès aux ressources d’un compte de stockage sans partager les clés du compte. Une signature d’accès partagé vous fournit un contrôle précis sur le type d’accès que vous accordez aux clients qui disposent de la signature SAS : intervalle de validité, autorisations accordées, plage d’adresses IP acceptée, protocole accepté (https/http).

- L’**identité de l’utilisateur** (également appelée « pass-through ») est un type d’autorisation où l’identité de l’utilisateur Azure AD qui s’est connecté au pool SQL serverless est utilisée pour autoriser l’accès aux données. Avant d’accéder aux données, l’administrateur du stockage Azure doit accorder des autorisations à l’utilisateur Azure AD pour accéder aux données. Ce type d’autorisation utilise l’utilisateur Azure AD qui s’est connecté au pool SQL serverless : il n’est donc pas pris en charge pour les types d’utilisateurs SQL.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur la connexion aux points de terminaison et sur l’interrogation des fichiers, consultez les articles suivants : 
- [Se connecter au point de terminaison](connect-overview.md)
- [Interroger des fichiers](develop-storage-files-overview.md)
