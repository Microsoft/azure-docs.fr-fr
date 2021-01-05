---
title: 'ML Studio (classique) : Importation de données d’apprentissage – Azure'
description: Découvrez comment importer vos données Azure Machine Learning Studio (classique) depuis différentes sources de données. Découvrez quels types de données et quels formats de données sont pris en charge.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: previous-author=heatherbshapiro, previous-ms.author=hshapiro
ms.date: 02/01/2019
ms.openlocfilehash: c69ed7b9e2aaa7113637868e1c1329ed2962b931
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96302776"
---
# <a name="import-your-training-data-into-azure-machine-learning-studio-classic-from-various-data-sources"></a>Importer vos données d’entraînement dans Azure Machine Learning Studio (classique) à partir de différentes sources de données

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Pour utiliser vos propres données dans Machine Learning Studio (classique) afin de développer et de tester une solution d’analyse prédictive, vous pouvez utiliser les données des ressources suivantes : 

* **Fichier local** : chargez à l’avance les données locales à partir de votre disque dur pour créer un module de jeu de données dans votre espace de travail
* **Sources de données en ligne** : utilisez le module [Importer les données][import-data] pour accéder aux données à partir d’une des nombreuses sources en ligne pendant que votre expérience s’exécute
* **Expérience Machine Learning Studio (classique)**  : utiliser les données qui ont été enregistrées en tant que jeu de données dans Machine Learning Studio (classique)
* [**Base de données SQL Server**](use-data-from-an-on-premises-sql-server.md) - Utiliser les données d’une base de données SQL Server sans avoir à copier les données manuellement

> [!NOTE]
> Un certain nombre d’exemples de jeux de données sont disponibles dans Machine Learning Studio (classique) et vous pouvez les utiliser comme données de formation. Pour plus d’informations, voir [Utilisation des exemples de jeux de données dans Azure Machine Learning Studio (classique)](use-sample-datasets.md).

## <a name="prepare-data"></a>Préparer les données

Machine Learning Studio (classique) est conçu pour travailler avec des données tabulaires ou rectangulaires, comme des données texte délimitées ou structurées à partir d’une base de données, bien que dans certains cas des données non rectangulaires puissent être utilisées.

Il est préférable que vos données soient relativement propres avant leur importation dans Studio (classique). Par exemple, vous souhaitez prendre en charge des problèmes tels que des chaînes sans guillemets.

Toutefois, des modules sont disponibles dans Studio (classique), qui permettent d’effectuer certaines manipulations de données dans votre expérience après importation de vos données. En fonction des algorithmes d’apprentissage automatique que vous allez utiliser, vous devrez décider comment gérer les problèmes structurels des données tels que des valeurs manquantes et des données fragmentées. Certains modules existent pour vous aider à régler ces problèmes. Rechercher dans la section **Transformation des données** de la palette des modules ceux qui exécutent ces fonctions.

À tout stade de votre expérience, vous pouvez voir ou télécharger les données générées par un module en cliquant sur le port de sortie. En fonction du module, différentes options de téléchargement peuvent être disponibles. Vous pouvez également afficher les données dans votre navigateur web dans Studio (classique).

## <a name="supported-data-formats-and-data-types"></a>Formats et types de données pris en charge

Vous pouvez importer un certain nombre de types de données dans votre expérience, selon le mécanisme que vous utilisez pour importer les données et l’emplacement d’où elles proviennent :

* Texte brut (.txt)
* Comma-separated values (CSV) avec un en-tête (.csv) ou sans (. nh.csv)
* Tab-separated values (TSV) avec un en-tête (.tsv) ou sans (. nh.tsv)
* Fichier Excel
* Table Azure
* Table hive
* Base de données SQL
* Valeurs OData
* Données SVMLight (.svmlight) (voir la [définition SVMLight](http://svmlight.joachims.org/) pour les informations relatives au format)
* Données Attribute Relation File Format (ARFF) (.arff) (voir la [définition ARFF](https://www.cs.waikato.ac.nz/ml/weka/arff.html) pour les informations relatives au format)
* Fichier zip (.zip)
* Fichier d’espace de travail ou d’objet R (.RData)

Si vous importez des données dans un format tel que ARFF qui inclut des métadonnées, Studio (classique) utilise celles-ci pour définir le titre et le type de données de chaque colonne.

Si vous importez des données dans des formats tels que TSV ou CSV qui n’incluent pas ces métadonnées, Studio (classique) déduit le type de données de chaque colonne en échantillonnant les données. Si les données n’ont pas non plus de titre de colonne, Studio (classique) fournit des noms par défaut.

Vous pouvez spécifier de manière explicite ou modifier les titres et les types de données pour les colonnes à l’aide du module [Modifier les métadonnées][edit-metadata].

Les types de données reconnus par Studio (classique) sont les suivants :

* String
* Integer
* Double
* Boolean
* DateTime
* TimeSpan

Studio utilise un type de données interne appelé ***table de données** _ pour passer des données entre les modules. Vous pouvez convertir de manière explicite vos données dans un format de table de données à l’aide du module [Convertir en jeu de données][convert-to-dataset].

Tout module qui accepte des formats autres que la table de données convertit silencieusement les données en table de données avant de les passer au module suivant.

Au besoin, vous pouvez convertir à nouveau le format de table de données au format CSV, TSV, ARFF ou SVMLight à l’aide d’autres modules de conversion.
Recherchez dans la section _ *Conversion des formats de données** de la palette des modules ceux qui exécutent ces fonctions.

## <a name="data-capacities"></a>Capacités des données

Les modules de Machine Learning Studio (classique) prennent en charge les jeux de données d’une taille maximale de 10 Go de données numériques denses pour les scénarios d’utilisation courants. Si un module accepte plusieurs entrées, le total de toutes les tailles d’entrée est de 10 Go. Vous pouvez échantillonner des jeux de données plus importants par le biais de requêtes Hive ou Azure SQL Database ou encore via un prétraitement Learning by Counts avant d'importer les données.  

Les types de données suivants peuvent être développés en jeux de données plus importants au moment de la normalisation des fonctionnalités et ils sont limités à moins de 10 Go :

* Partiellement alloué
* Par catégorie
* Chaînes
* Données binaires

Les modules suivants sont limités à des jeux de données inférieurs à 10 Go :

* modules de recommandation
* Module SMOTE (Synthetic Minority Oversampling Technique)
* Modules de scripts : R, Python, SQL
* Modules dont la taille des données de sortie peut être supérieure à la taille des données d’entrée, comme Join ou Feature Hashing
* Validation croisée, réglage des hyperparamètres de modèle, régression ordinale et plusieurs classes de un contre tous, lorsque le nombre d’itérations est très élevé

Pour les tailles de jeux de données supérieures à quelques gigaoctets, chargez les données sur Stockage Azure ou Azure SQL Database. Vous pouvez aussi utiliser Azure HDInsight au lieu d’effectuer un chargement directement à partir d’un fichier local.

Vous trouverez des informations sur les données d’image dans la référence du module [Importer des Images](/azure/machine-learning/studio-module-reference/import-images#bkmk_Notes).

## <a name="import-from-a-local-file"></a>Importer à partir d’un fichier local

Vous pouvez charger un fichier de données à partir de votre disque dur pour vous en servir comme données de formation dans Studio (classique). Lorsque vous importez un fichier de données, vous créez un module de jeu de données prêt à être utilisé dans des expériences au sein de votre espace de travail.

Pour importer des données à partir d’un disque dur local, procédez comme suit :

1. Cliquez sur **+NOUVEAU** en bas de la fenêtre de Studio (classique).
2. Sélectionnez **JEU DE DONNÉES** et **DEPUIS UN FICHIER LOCAL**.
3. Dans la boîte de dialogue **Télécharger un nouveau jeu de données** , recherchez le fichier que vous souhaitez télécharger.
4. Saisissez un nom, identifiez le type de données puis saisissez éventuellement une description. Une description est recommandée : elle vous permet d’enregistrer des caractéristiques relatives aux données que vous souhaitez mémoriser pour une utilisation future.
5. La case à cocher **Il s'agit de la nouvelle version d'un jeu de donnée existant** vous permet de mettre à jour un jeu de données existant avec de nouvelles données. Pour ce faire, cliquez simplement sur cette case à cocher, puis saisissez le nom d’un jeu de données existant.

![Charger un nouveau jeu de données](./media/import-data/upload-dataset-from-local-file.png)

Le temps de téléchargement dépend de la taille de vos données et de la vitesse de votre connexion au service. Si vous savez que le téléchargement du fichier prendra du temps, vous pouvez faire autre chose dans Studio (classique) en attendant. Cependant, la fermeture du navigateur avant la fin du chargements entraîne l’échec de celui-ci.

Une fois que vos données sont téléchargées, elles sont stockées dans un module de jeu de données et sont disponibles pour n'importe quelle expérience dans votre espace de travail.

Quand vous éditez une expérimentation, les jeux de données que vous avez chargés apparaissent sous **Mes jeux de données** dans la liste **Jeux de données enregistrés** de la palette du module. Vous pouvez glisser-déplacer le jeu de données dans le canevas de l’expérience en vue d’affiner l’analyse et Machine Learning.

## <a name="import-from-online-data-sources"></a>Importer à partir de sources de données en ligne

En utilisant le module [Importer des données][import-data], votre expérience peut importer des données à partir de diverses sources en ligne en cours d’exécution.

> [!NOTE]
> Cet article fournit des informations générales sur le module [Importer les données][import-data]. Pour plus d’informations sur les types de données auxquelles vous pouvez accéder, les formats, les paramètres et les réponses aux questions courantes, consultez la rubrique de référence pour le module [Importer les données][import-data].

Le module [Importer des données][import-data] vous permet d’accéder aux données d’une des sources en ligne en cours d’expérience :

* Une URL web avec HTTP
* Hadoop avec HiveQL
* Stockage Blob Azure
* Table Azure
* Azure SQL Database. SQL Managed Instance ou SQL Server
* Un fournisseur de flux de données, actuellement, OData
* Azure Cosmos DB

Ces données d’apprentissage étant chargées en cours d’expérience, elles ne sont disponibles que dans le cadre de celle-ci. En revanche, des données stockées dans un module de jeu de données sont disponibles pour toute expérience au sein de votre espace de travail.

Pour accéder à des sources de données en ligne dans votre expérience Studio (classique), ajoutez le module [Importer des données][import-data] module à votre expérience. Sélectionnez ensuite **Lancer l’Assistant Importation de données** sous **Propriétés** afin d’obtenir des instructions pas à pas pour sélectionner et configurer la source de données. Vous pouvez également sélectionner manuellement une **source de données** sous **Propriétés** et fournir les paramètres nécessaires pour accéder aux données.

Les sources de données en ligne prises en charge sont détaillées dans le tableau ci-dessous. Ce tableau récapitule également les formats de fichier pris en charge et les paramètres qui sont utilisés pour accéder aux données.

> [!IMPORTANT]
> Actuellement, les modules [Importer les données][import-data] et [Exporter les données][export-data] peuvent lire et écrire des données uniquement à partir d’un stockage Azure créé à l’aide du modèle de déploiement classique. En d’autres termes, le nouveau type de compte de stockage d’objets blob Azure qui offre un niveau d’accès au stockage chaud ou un niveau d’accès au stockage froid n’est pas encore pris en charge.
>
> En règle générale, les comptes de stockage Azure que vous avez peut-être créés avant que cette option de service ne soit disponible ne devraient pas être affectés.
> Si vous avez besoin de créer un nouveau compte, sélectionnez **Classique** comme modèle de déploiement, ou utilisez Resource Manager et, comme **Type de compte**, sélectionnez **Usage général** plutôt que **Stockage Blob**.
>
> Pour plus d’informations, consultez [Stockage Blob Azure : niveaux de stockage chauds et froids](../../storage/blobs/storage-blob-storage-tiers.md).

### <a name="supported-online-data-sources"></a>Sources de données en ligne prises en charge
Le module **Importer les données** d’Azure Machine Learning Studio (classique) prend en charge les sources de données suivantes :

| source de données | Description | Paramètres |
| --- | --- | --- |
| URL Web via HTTP |Lit les données au format CSV (valeurs séparées par des virgules), TSV (valeurs séparées par des tabulations), ARFF (format de fichier de relation d’attribut) et SVM-light (Machines vectorielles (SVM clair)), à partir de n’importe quelle URL web qui utilise le protocole HTTP |<b>URL</b> : Indique le nom complet du fichier, notamment l’URL du site et le nom de fichier, avec n’importe quelle extension. <br/><br/><b>Format de données</b> : Spécifie un des formats de données pris en charge : CSV, TSV, ARFF ou SVM-light. Lorsque les données ont une ligne d’en-tête, elle est utilisée pour attribuer des noms de colonne. |
| Hadoop/HDFS |Lit les données à partir d’un stockage distribué dans Hadoop. Spécifiez les données souhaitées à l’aide de HiveQL, langage de requête de type SQL. Vous pouvez également utiliser HiveQL pour agréger et effectuer un filtrage des données avant d’ajouter les données à Studio (classique). |<b>Requête de base de données Hive</b> : Spécifie la requête Hive utilisée pour générer les données.<br/><br/><b>URI du serveur HCatalog</b> : Spécifie le nom de votre cluster en utilisant le format *&lt;nom de votre cluster&gt;. azurehdinsight.net.*<br/><br/><b>Nom du compte utilisateur Hadoop</b> : Spécifie le nom du compte utilisateur Hadoop utilisé pour configurer le cluster.<br/><br/><b>Mot de passe du compte utilisateur Hadoop</b> : Spécifie les informations d’identification utilisées lors de la configuration du cluster. Pour plus d’informations, consultez [Création de clusters Hadoop dans HDInsight](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md).<br/><br/><b>Emplacement des données de sortie</b> : Indique si les données sont stockées dans un système de fichiers distribué Hadoop (HDFS) ou dans Azure. <br/><ul>Si vous stockez des données de sortie dans HDFS, spécifiez l'URI du serveur HDFS. (Veillez à utiliser le nom du cluster HDInsight sans le préfixe HTTPS://). <br/><br/>Si vous stockez vos données de sortie dans Azure, vous devez spécifier le nom du compte de stockage Azure, la clé d’accès au stockage et le nom du conteneur de stockage.</ul> |
| Base de données SQL |Lit les données stockées dans Azure SQL Database, SQL Managed Instance ou dans une base de données SQL Server s’exécutant sur une machine virtuelle Azure. |<b>Nom du serveur de base de données</b> : Spécifie le nom du serveur sur lequel la base de données s’exécute.<br/><ul>S’il s’agit d'une base de données Azure SQL, entrez le nom du serveur généré. En général, il se présente sous la forme *&lt;generated_identifier&gt;.database.windows.net.* <br/><br/>Dans le cas d’un serveur SQL hébergé sur une machine virtuelle Azure, entrez *tcp:&lt;Virtual Machine DNS Name&gt;, 1433*</ul><br/><b>Nom de la base de données</b> : Spécifie le nom de la base de données sur le serveur. <br/><br/><b>Nom du compte utilisateur du serveur</b> : Spécifie un nom d’utilisateur pour un compte disposant des autorisations d’accès à la base de données. <br/><br/><b>Mot de passe du compte d’utilisateur du serveur</b> : Spécifie le mot de passe du compte d’utilisateur.<br/><br/><b>Requête de base de données</b> : entrez une instruction SQL qui décrit les données que vous souhaitez lire. |
| Base de données SQL locale |Lit les données stockées dans une base de données SQL. |<b>Passerelle de données</b> : Spécifie le nom de la passerelle de gestion de données installée sur un ordinateur à partir duquel elle peut accéder à votre base de données SQL Server. Pour plus d’informations sur la configuration de la passerelle, consultez [Effectuer des analyses avancées avec Azure Machine Learning Studio (classique) en utilisant les données d’un serveur SQL](use-data-from-an-on-premises-sql-server.md).<br/><br/><b>Nom du serveur de base de données</b> : Spécifie le nom du serveur sur lequel la base de données s’exécute.<br/><br/><b>Nom de la base de données</b> : Spécifie le nom de la base de données sur le serveur. <br/><br/><b>Nom du compte utilisateur du serveur</b> : Spécifie un nom d’utilisateur pour un compte disposant des autorisations d’accès à la base de données. <br/><br/><b>Nom d’utilisateur et mot de passe</b> : Cliquez sur <b>Entrer les valeurs</b> pour saisir les informations d’identification de votre base de données. Vous pouvez utiliser l’authentification intégrée Windows ou l’authentification SQL Server en fonction de la configuration de votre serveur SQL Server.<br/><br/><b>Requête de base de données</b> : entrez une instruction SQL qui décrit les données que vous souhaitez lire. |
| table Azure |Lit les données du service de tableau dans le stockage Azure.<br/><br/>Si vous lisez rarement de grandes quantités de données, utilisez le Service de Tableau Azure. Il fournit une solution de stockage ultra disponible, flexible, non relationnelle (NoSQL), économique et hautement évolutive. |Les options du module **Importer les données** changent selon que vous accédez à des informations publiques ou à un compte de stockage privé qui nécessite des informations d’identification de connexion. Cela est déterminé par le <b>Type d'authentification</b> qui peut avoir la valeur « PublicOrSAS » ou « Compte », chacun d'entre eux possède son propre ensemble de paramètres. <br/><br/><b>URI de signature d’accès partagé (SAP) ou public</b> : Les paramètres sont les suivants :<br/><br/><ul><b>URI de table</b> : Spécifie l’URL SAP ou publique de la table.<br/><br/><b>Spécifie les lignes à analyser pour rechercher les noms de propriété</b> : Les valeurs sont <i>TopN</i> pour analyser le nombre spécifié de lignes ou <i>ScanAll</i> pour obtenir toutes les lignes de la table. <br/><br/>Si les données sont homogènes et prévisibles, nous vous recommandons de sélectionner *TopN* et d’entrer un chiffre pour N. Pour les tables volumineuses, ceci peut accélérer les temps de lecture.<br/><br/>Si les données sont structurées avec des ensembles de propriétés qui varient en fonction de la profondeur et la position de la table, choisissez l’option *ScanAll* pour analyser toutes les lignes. Cela garantit l’intégrité de la propriété obtenue et de la conversion des métadonnées.<br/><br/></ul><b>Compte de stockage privé</b> : Les paramètres sont les suivants : <br/><br/><ul><b>Nom du compte</b> : Spécifie le nom du compte contenant la table à lire.<br/><br/><b>Clé du compte</b> : Spécifie la clé de stockage associée au compte.<br/><br/><b>Nom de la table</b> : Spécifie le nom de la table contenant les données à lire.<br/><br/><b>Lignes à analyser pour rechercher les noms de propriété</b> : Les valeurs sont <i>TopN</i> pour analyser le nombre spécifié de lignes ou <i>ScanAll</i> pour obtenir toutes les lignes de la table.<br/><br/>Si les données sont homogènes et prévisibles, nous vous recommandons de sélectionner *TopN* et d’entrer un chiffre pour N. Pour les tables volumineuses, ceci peut accélérer les temps de lecture.<br/><br/>Si les données sont structurées avec des ensembles de propriétés qui varient en fonction de la profondeur et la position de la table, choisissez l’option *ScanAll* pour analyser toutes les lignes. Cela garantit l’intégrité de la propriété obtenue et de la conversion des métadonnées.<br/><br/> |
| Stockage Blob Azure |Lit les données stockées dans le service d'objet Blob dans le stockage Azure, notamment les images, le texte non structuré ou les données binaires.<br/><br/>Vous pouvez utiliser le service Blob pour exposer publiquement des données ou pour stocker en privé des données d'application. Vous pouvez accéder à vos données depuis n’importe où grâce à des connexions HTTP ou HTTPS. |Les options du module **Importer les données** changent selon que vous accédez à des informations publiques ou à un compte de stockage privé qui nécessite des informations d’identification de connexion. Cela est déterminé par le <b>Type d’authentification</b> qui peut avoir la valeur « PublicOrSAS » ou « Compte ».<br/><br/><b>URI de signature d’accès partagé (SAP) ou public</b> : Les paramètres sont les suivants :<br/><br/><ul><b>URI</b> : Spécifie l’URL SAP ou publique du Stockage Blob.<br/><br/><b>Format du fichier</b> : Spécifie le format des données dans le service BLOB. Les formats pris en charge sont CSV, TSV et ARFF.<br/><br/></ul><b>Compte de stockage privé</b> : Les paramètres sont les suivants : <br/><br/><ul><b>Nom du compte</b> : Spécifie le nom du compte contenant l’objet Blob à lire.<br/><br/><b>Clé du compte</b> : Spécifie la clé de stockage associée au compte.<br/><br/><b>Chemin d’accès au conteneur, répertoire ou Blob</b> : Spécifie le nom de l’objet Blob contenant les données à lire.<br/><br/><b>Format du fichier Blob</b> : Spécifie le format des données dans le service Blob. Les formats de données pris en charge sont CSV, TSV, ARFF, CSV avec un codage spécifié et Excel. <br/><br/><ul>Si le format est CSV ou TSV, veillez à indiquer si le fichier contient une ligne d’en-tête.<br/><br/>Vous pouvez utiliser l'option Excel pour lire des données à partir de classeurs Excel. Dans l'option <i>format de données Excel</i>, indiquez si les données se trouvent dans une plage de feuille de calcul Excel ou un tableau Excel. Dans l’option <i>Feuille de calcul Excel ou table intégrée</i>, spécifiez le nom de la feuille ou table à lire.</ul><br/> |
| Fournisseur de flux de données |Lit les données d’un fournisseur de flux pris en charge. Actuellement, seul le protocole Open Data Protocol (OData) est pris en charge. |<b>Type de contenu de données</b> : Spécifie le format OData.<br/><br/><b>URL source</b> : Spécifie l’URL complète du flux de données. <br/>Par exemple, l’URL suivante lit dans l’exemple de base de données Northwind : https://services.odata.org/northwind/northwind.svc/ |

## <a name="import-from-another-experiment"></a>Importer à partir d’une autre expérience

Il se peut que vous ayez parfois besoin d’obtenir un résultat intermédiaire à partir d’une expérience et de l’utiliser dans le cadre d’une autre expérience. Pour ce faire, vous enregistrez le module en tant que jeu de données :

1. Cliquez sur la sortie du module que vous souhaitez enregistrer en tant que jeu de données.
2. Cliquez sur **Enregistrer comme jeu de données**.
3. Lorsque vous y êtes invité, saisissez un nom et une description qui vous permet d'identifier facilement le jeu de données.
4. Cliquez sur la coche **OK** .

Lorsque l'enregistrement est terminé, le jeu de données sera disponible pour être utilisé dans n'importe quelle expérience dans votre espace de travail. Vous pouvez le trouver dans la liste **Jeux de données enregistrés** dans la palette des modules.

## <a name="next-steps"></a>Étapes suivantes

[Déployer des services web Azure Machine Learning Studio utilisant les modules d’importation et d’exportation des données](web-services-that-use-import-export-modules.md)


<!-- Module References -->
[import-data]: /azure/machine-learning/studio-module-reference/import-data
[export-data]: /azure/machine-learning/studio-module-reference/export-data


<!-- Module References -->
[convert-to-dataset]: /azure/machine-learning/studio-module-reference/convert-to-dataset
[edit-metadata]: /azure/machine-learning/studio-module-reference/edit-metadata
[import-data]: /azure/machine-learning/studio-module-reference/import-data