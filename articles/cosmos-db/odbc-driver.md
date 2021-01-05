---
title: Se connecter à Azure Cosmos DB à l’aide d’outils d’analyse décisionnelle
description: Découvrez comment utiliser le pilote ODBC Azure Cosmos DB pour créer des tables et des vues afin d’afficher les données normalisées dans BI et dans un logiciel d’analyse de données.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/02/2019
ms.author: sngun
ms.openlocfilehash: e7d6a67f5322c5bb640430f66ccb0917f6faada1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96003495"
---
# <a name="connect-to-azure-cosmos-db-using-bi-analytics-tools-with-the-odbc-driver"></a>Se connecter à Azure Cosmos DB à l’aide d’outils d’analyse décisionnelle avec le pilote ODBC
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le pilote ODBC Azure Cosmos DB vous permet de vous connecter à Azure Cosmos DB à l’aide d’outils d’analyse décisionnelle comme SQL Server Integration Services, Power BI Desktop et Tableau, pour analyser et créer une représentation visuelle de vos données Azure Cosmos DB dans ces solutions.

Le pilote ODBC Azure Cosmos DB est conforme à ODBC 3.8 et prend en charge la syntaxe ANSI SQL-92. Le pilote offre de puissantes fonctionnalités pour vous aider à renormaliser les données dans Azure Cosmos DB. Grâce à ce pilote, vous pouvez représenter les données dans Azure Cosmos DB sous forme de tables et de vues. Il vous permet d’effectuer des opérations SQL dans des tables et des vues, notamment des regroupements par requêtes, des insertions, des mises à jour et des suppressions.

> [!NOTE]
> La connexion à Azure Cosmos DB avec le pilote ODBC est actuellement prise en charge uniquement pour les comptes de l’API SQL Azure Cosmos DB.

## <a name="why-do-i-need-to-normalize-my-data"></a>Pourquoi dois-je normaliser mes données ?
Azure Cosmos DB est une base de données sans schéma, ce qui permet le développement rapide d’applications et donne la possibilité d’effectuer une itération sur les modèles de données sans être limité à un schéma strict. Une même base de données Azure Cosmos peut contenir des documents JSON de différentes structures. C’est une solution idéale pour le développement rapide d’applications, mais si vous souhaitez analyser et créer des rapports de vos données à l’aide d’outils d’analyse de données et décisionnels, les données doivent souvent être aplaties et respecter un schéma spécifique.

C’est là qu’intervient le pilote ODBC. Grâce au pilote ODBC, vous pouvez à présent renormaliser les données d’Azure Cosmos DB dans des tables et des vues adaptées à vos besoins d’analytique données et de création de rapports. Les schémas renormalisés n’ont aucun impact sur les données sous-jacentes et il n’est pas obligatoire pour les développeurs de les respecter ; au lieu de cela, ils permettent de tirer parti d’outils compatibles ODBC pour accéder aux données. Désormais, votre base de données Azure Cosmos ne sera pas uniquement l’un des outils favoris de votre équipe de développement. Vos analystes de données vont l’adorer eux aussi.

Familiarisons-nous avec le pilote ODBC.

## <a name="step-1-install-the-azure-cosmos-db-odbc-driver"></a><a id="install"></a>Étape 1 : Installer le pilote ODBC Azure Cosmos DB

1. Téléchargez les pilotes correspondant à votre environnement :

    | Programme d’installation | Systèmes d’exploitation pris en charge| 
    |---|---| 
    |[Microsoft Azure Cosmos DB ODBC 64-bit.msi](https://aka.ms/cosmos-odbc-64x64) pour Windows 64 bits| Versions 64 bits de Windows 8.1 ou version ultérieure, Windows 8, Windows 7, Windows Server 2012 R2, Windows Server 2012 et Windows Server 2008 R2.| 
    |[Microsoft Azure Cosmos DB ODBC 32x64-bit.msi](https://aka.ms/cosmos-odbc-32x64) pour 32 bits sur Windows 64 bits| Versions 64 bits de Windows 8.1 ou version ultérieure, Windows 8, Windows 7, Windows XP, Windows Vista, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 et Windows Server 2003.| 
    |[Microsoft Azure Cosmos DB ODBC 32-bit.msi](https://aka.ms/cosmos-odbc-32x32) pour Windows 32 bits|Versions 32 bits de Windows 8.1 ou version ultérieure, Windows 8, Windows 7, Windows XP et Windows Vista.|

    Exécutez le fichier msi localement pour lancer l’**Assistant d’installation du pilote ODBC Microsoft Azure Cosmos DB**. 

1. Terminez l’assistant d’installation en utilisant l’entrée par défaut pour installer le pilote ODBC.

1. Ouvrez l’application **Administrateur de sources de données ODBC** sur votre ordinateur. Vous pouvez le faire en tapant **sources de données ODBC** dans la zone de recherche Windows. 
    Vous pouvez confirmer l’installation du pilote en cliquant dans l’onglet **Pilotes** pour vérifier que le **pilote ODBC Microsoft Azure Cosmos DB** est répertorié.

    :::image type="content" source="./media/odbc-driver/odbc-driver.png" alt-text="Administrateur de la source de données ODBC Azure Cosmos DB":::

## <a name="step-2-connect-to-your-azure-cosmos-database"></a><a id="connect"></a>Étape 2 : Vous connecter à votre base de données Azure Cosmos

1. Après l’[installation du pilote ODBC Azure Cosmos DB](#install), dans la fenêtre **Administrateur de sources de données ODBC**, cliquez sur **Ajouter**. Vous pouvez créer un DSN utilisateur ou système. Dans cet exemple, vous allez créer un DSN utilisateur.

1. Dans la fenêtre **Créer une nouvelle source de données**, sélectionnez **Microsoft Azure Cosmos DB ODBC Driver (Pilote ODBC Microsoft Azure Cosmos DB)** , puis cliquez sur **Terminer**.

1. Dans la fenêtre **Azure Cosmos DB ODBC Driver SDN Setup (Configuration DSN du pilote ODBC Azure Cosmos DB)** , indiquez les informations suivantes : 

    :::image type="content" source="./media/odbc-driver/odbc-driver-dsn-setup.png" alt-text="Fenêtre de configuration DSN du pilote ODBC Azure Cosmos DB":::
    - **Nom de source de données** : le nom convivial de votre DSN ODBC. Ce nom étant spécifique à votre compte Azure Cosmos DB, choisissez-le de manière appropriée si vous possédez plusieurs comptes.
    - **Description** : courte description de la source de données.
    - **Hôte** : URI de votre compte Azure Cosmos DB. Vous pouvez récupérer cette information sur la page des clés Azure Cosmos DB du portail Azure, comme illustré dans la capture d’écran suivante. 
    - **Clé d’accès** : clé primaire ou secondaire, en lecture-écriture ou en lecture seule, affichée sur la page des clés Azure Cosmos DB du portail Azure, comme illustré dans la capture d’écran suivante. Nous vous recommandons d'utiliser la clé en lecture seule si le DSN sert au traitement des données en lecture seule et à la création de rapports.
    :::image type="content" source="./media/odbc-driver/odbc-cosmos-account-keys.png" alt-text="Page des clés Azure Cosmos DB":::
    - **Chiffrer la clé d’accès pour** : sélectionnez l’option optimale en fonction des utilisateurs de cet ordinateur. 
    
1. Cliquez sur le bouton **Test** pour vérifier que vous pouvez vous connecter à votre compte Azure Cosmos DB. 

1.  Cliquez sur **Options avancées** et définissez les valeurs suivantes :
    *  **Version de l’API REST** : Sélectionnez la [version de l’API REST](/rest/api/cosmos-db/) pour vos opérations. La valeur par défaut est 2015-12-16. Si vous avez des conteneurs avec de [grandes clés de partition](large-partition-keys.md) et que vous avez besoin de l’API REST version 2018-12-31 :
        - Tapez **2018-12-31** comme version de l’API REST
        - Dans le menu **Démarrer**, tapez « regedit » pour rechercher et ouvrir l’application **Éditeur du Registre**.
        - Dans l’Éditeur du Registre, accédez au chemin suivant : **Computer\HKEY_LOCAL_MACHINE\SOFTWARE\ODBC\ODBC.INI**
        - Créez une nouvelle sous-clé avec le même nom que votre DSN, p. ex. « Contoso Account ODBC DSN ».
        - Accédez à la sous-clé « Contoso Account ODBC DSN ».
        - Cliquez avec le bouton droit pour ajouter une nouvelle valeur **String** :
            - Nom de la valeur : **IgnoreSessionToken**
            - Données de la valeur : **1**
            :::image type="content" source="./media/odbc-driver/cosmos-odbc-edit-registry.png" alt-text="Paramètres de l’Éditeur du Registre":::
    - **Cohérence des requêtes** : sélectionnez le [niveau de cohérence](consistency-levels.md) de vos opérations. La valeur par défaut est Session.
    - **Nombre de tentatives** : entrez le nombre de tentatives d’une opération si la demande initiale n’aboutit pas en raison d’une limitation du débit service.
    - **Fichier de schéma** : Vous avez plusieurs possibilités.
        - Par défaut, si vous ne modifiez pas cette entrée (vide), le pilote analyse la première page des données de tous les conteneurs afin de déterminer le schéma de chaque conteneur. Cette opération est appelée Mappage de conteneur. Si aucun fichier de schéma n’est défini, le pilote doit effectuer l’analyse pour chaque session de pilote, ce qui peut allonger le délai de démarrage d’une application avec le DSN. Nous vous recommandons de toujours associer un fichier de schéma à un DSN.
        - Si vous disposez déjà d’un fichier de schéma (peut-être un fichier que vous avez créé à l’aide de l’Éditeur de schéma), cliquez sur **Parcourir**, recherchez votre fichier, cliquez sur **Enregistrer**, puis sur **OK**.
        - Si vous souhaitez créer un nouveau schéma, cliquez sur **OK**, puis sur **Éditeur de schéma** dans la fenêtre principale. Accédez ensuite à l’Éditeur de schéma pour plus d’informations. Après la création du nouveau fichier de schéma, pensez à revenir à la fenêtre **Options avancées** pour l’inclure.

1. Une fois que vous avez terminé et fermé la fenêtre de **configuration DSN du pilote ODBC Azure Cosmos DB**, le DSN du nouvel utilisateur est ajouté à l’onglet DSN utilisateur.

    :::image type="content" source="./media/odbc-driver/odbc-driver-user-dsn.png" alt-text="Nouveau nom de source de données ODBC Azure Cosmos DB dans l’onglet Nom de source de données utilisateur":::

## <a name="step-3-create-a-schema-definition-using-the-container-mapping-method"></a><a id="#container-mapping"></a>Étape 3 : Créer une définition de schéma à l’aide de la méthode de mappage de conteneur

Il existe deux types de méthodes d’échantillonnage que vous pouvez utiliser : le **mappage de conteneur** et les **délimiteurs de table**. Une session d’échantillonnage peut utiliser ces deux méthodes d’échantillonnage, mais chaque conteneur ne peut utiliser qu’une méthode d’échantillonnage spécifique. Les étapes ci-dessous créent un schéma pour les données d’un ou plusieurs conteneurs à l’aide de la méthode de mappage de conteneur. Cette méthode d’échantillonnage récupère les données dans la page d’un conteneur pour déterminer la structure de ces données. Elle transpose un conteneur en table côté ODBC. Cette méthode d’échantillonnage est rapide et efficace lorsque les données d’un conteneur sont homogènes. Si un conteneur contient des données hétérogènes, nous vous recommandons d’utiliser la [méthode de mappage par délimiteurs de table](#table-mapping), car elle fournit une méthode d’échantillonnage plus robuste pour déterminer les structures des données du conteneur. 

1. Après avoir terminé les étapes 1 à 4 de la rubrique [Vous connecter à votre base de données Azure Cosmos](#connect), cliquez sur **Éditeur de schéma** dans la fenêtre **Configuration DSN du pilote ODBC Azure Cosmos DB**.

    :::image type="content" source="./media/odbc-driver/odbc-driver-schema-editor.png" alt-text="Bouton Éditeur de schéma dans la fenêtre de configuration du nom de source de données du pilote ODBC Azure Cosmos DB":::
1. Dans la fenêtre **Éditeur de schéma**, cliquez sur **Créer**.
    La fenêtre **Générer le schéma** affiche tous les conteneurs du compte Azure Cosmos DB. 

1. Sélectionnez un ou plusieurs conteneurs à échantillonner, puis cliquez sur **Échantillonner**. 

1. Dans l’onglet **Mode Création**, la base de données, le schéma et la table sont représentés. Dans la vue de la table, l’analyse affiche l’ensemble des propriétés associées aux noms de colonne (Nom SQL, Nom de la Source, etc.).
    Pour chaque colonne, vous pouvez modifier le nom de la colonne SQL, le type SQL, la longueur SQL (le cas échéant), l’échelle (le cas échéant), la précision (le cas échéant) et la valeur Nullable.
    - Vous pouvez définir **Masquer la colonne** sur **true** si vous souhaitez exclure cette colonne des résultats de la requête. Les colonnes marquées Masquer la colonne = true ne sont pas retournées pour la sélection et la projection, bien qu’elles fassent toujours partie du schéma. Par exemple, vous pouvez masquer toutes les propriétés système Azure Cosmos DB requises commençant par « _ ».
    - La colonne **id** est le seul champ qui ne peut pas être masqué car elle sert de clé primaire dans le schéma normalisé. 

1. Une fois que vous avez terminé la définition du schéma, cliquez sur **Fichier** | **Enregistrer**, accédez au répertoire d’enregistrement du schéma, puis cliquez sur **Enregistrer**.

1. Pour utiliser ce schéma avec un nom de source de données (DSN), ouvrez la **fenêtre de configuration du DSN du pilote ODBC Azure Cosmos DB** (par le biais de l’Administrateur de sources de données ODBC), cliquez sur **Options avancées**, puis, dans la boîte de dialogue **Fichier de schéma**, accédez au schéma enregistré. L’enregistrement d’un fichier de schéma dans une source de données existante modifie la connexion de DSN afin de définir l’étendue des données et de la structure définie par le schéma.

## <a name="step-4-create-a-schema-definition-using-the-table-delimiters-mapping-method"></a><a id="table-mapping"></a>Étape 4 : Créer une définition de schéma à l’aide de la méthode de mappage des délimiteurs de table

Il existe deux types de méthodes d’échantillonnage que vous pouvez utiliser : le **mappage de conteneur** et les **délimiteurs de table**. Une session d’échantillonnage peut utiliser ces deux méthodes d’échantillonnage, mais chaque conteneur ne peut utiliser qu’une méthode d’échantillonnage spécifique. 

Les étapes suivantes créent un schéma pour les données d’un ou plusieurs conteneurs à l’aide de la méthode de mappage par **délimiteurs de table**. Nous vous recommandons d’utiliser cette méthode d’échantillonnage lorsque vos conteneurs contiennent des données hétérogènes. Vous pouvez utiliser cette méthode pour définir l’étendue de l’échantillonnage sur un ensemble d’attributs et ses valeurs correspondantes. Par exemple, si un document contient une propriété « Type », vous pouvez étendre l’échantillonnage aux valeurs de cette propriété. Le résultat final de l’échantillonnage serait un ensemble de tables pour chacune des valeurs du type que vous avez spécifié. Par exemple, Type = Voiture produira une table Voiture tandis que Type = Avion produira une table Avion.

1. Après avoir terminé les étapes 1 à 4 de la rubrique [Vous connecter à votre base de données Azure Cosmos](#connect), cliquez sur **Éditeur de schéma** dans la fenêtre Configuration DSN du pilote ODBC Azure Cosmos DB.

1. Dans la fenêtre **Éditeur de schéma**, cliquez sur **Créer**.
    La fenêtre **Générer le schéma** affiche tous les conteneurs du compte Azure Cosmos DB. 

1. Sélectionnez un conteneur sous l’onglet **Exemple de vue**, dans la colonne **Définition de mappage** du conteneur, puis cliquez sur **Modifier**. Puis, dans la fenêtre **Définition de mappage**, sélectionnez la méthode **Délimiteurs de table**. Faites ensuite ce qui suit :

    a. Dans le champ **Attributs**, tapez le nom d’une propriété de délimiteur. Il s’agit d’une propriété de votre document que vous souhaitez étendre à l’échantillonnage, par exemple Ville. Appuyez ensuite sur Entrée. 

    b. Si vous souhaitez uniquement étendre l’échantillonnage certaines valeurs de l’attribut que vous avez entré précédemment, sélectionnez l’attribut dans la zone de sélection, entrez une valeur dans la zone **Valeur**, par exemple Seattle, puis appuyez sur Entrée. Vous pouvez continuer à ajouter d’autres valeurs pour les attributs. Assurez-vous simplement que l’attribut approprié est sélectionné lorsque vous entrez des valeurs.

    Par exemple, si vous incluez une valeur **Attributs** de type Ville et que vous souhaitez limiter votre table pour inclure uniquement les lignes dont la valeur Ville est New York et Dubaï, vous devez entrer Ville dans la zone Attributs et New York puis Dubaï dans la zone **Valeurs**.

1. Cliquez sur **OK**. 

1. Après avoir mappé les définitions des conteneurs à échantillonner, dans la fenêtre **Éditeur de schéma**, cliquez sur **Échantillonner**.
     Pour chaque colonne, vous pouvez modifier le nom de la colonne SQL, le type SQL, la longueur SQL (le cas échéant), l’échelle (le cas échéant), la précision (le cas échéant) et la valeur Nullable.
    - Vous pouvez définir **Masquer la colonne** sur **true** si vous souhaitez exclure cette colonne des résultats de la requête. Les colonnes marquées Masquer la colonne = true ne sont pas retournées pour la sélection et la projection, bien qu’elles fassent toujours partie du schéma. Par exemple, vous pouvez masquer toutes les propriétés système Azure Cosmos DB requises commençant par `_`.
    - La colonne **id** est le seul champ qui ne peut pas être masqué car elle sert de clé primaire dans le schéma normalisé. 

1. Une fois que vous avez terminé la définition du schéma, cliquez sur **Fichier** | **Enregistrer**, accédez au répertoire d’enregistrement du schéma, puis cliquez sur **Enregistrer**.

1. Dans la fenêtre de **configuration du DSN du pilote ODBC Azure Cosmos DB**, cliquez sur **Options avancées**. Puis, dans la fenêtre **Fichier de schéma**, accédez au fichier de schéma enregistré et cliquez sur **OK**. Cliquez à nouveau sur **OK** pour enregistrer le DSN. Cette opération enregistre dans le DSN le schéma que vous avez créé. 

## <a name="optional-set-up-linked-server-connection"></a>(Facultatif) Configurer une connexion à un serveur lié

Vous pouvez interroger Azure Cosmos DB à partir de SQL Server Management Studio (SSMS) en configurant une connexion à un serveur lié.

1. Créez une source de données système, nommée par exemple `SDS Name`, en suivant les instructions de [l’étape 2](#connect).

1. [Installez SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) et connectez-vous au serveur. 

1. Dans l’éditeur de requête SSMS, créez un objet de serveur lié `DEMOCOSMOS` pour la source de données avec les commandes suivantes. Remplacez `DEMOCOSMOS` par le nom de votre serveur lié, et `SDS Name` par le nom de votre source de données système.

    ```sql
    USE [master]
    GO
    
    EXEC master.dbo.sp_addlinkedserver @server = N'DEMOCOSMOS', @srvproduct=N'', @provider=N'MSDASQL', @datasrc=N'SDS Name'
    
    EXEC master.dbo.sp_addlinkedsrvlogin @rmtsrvname=N'DEMOCOSMOS', @useself=N'False', @locallogin=NULL, @rmtuser=NULL, @rmtpassword=NULL
    
    GO
    ```
    
Pour voir le nom du nouveau serveur lié, actualisez la liste Serveurs liés.

:::image type="content" source="./media/odbc-driver/odbc-driver-linked-server-ssms.png" alt-text="Serveur lié dans SSMS":::

### <a name="query-linked-database"></a>Interroger une base de données liée

Pour interroger la base de données liée, entrez une requête SSMS. Dans cet exemple, la requête effectue une sélection dans la table du conteneur nommé `customers`:

```sql
SELECT * FROM OPENQUERY(DEMOCOSMOS, 'SELECT *  FROM [customers].[customers]')
```

exécutez la requête. Le résultat devrait se présenter ainsi :

```
attachments/  1507476156    521 Bassett Avenue, Wikieup, Missouri, 5422   "2602bc56-0000-0000-0000-59da42bc0000"   2015-02-06T05:32:32 +05:00 f1ca3044f17149f3bc61f7b9c78a26df
attachments/  1507476156    167 Nassau Street, Tuskahoma, Illinois, 5998   "2602bd56-0000-0000-0000-59da42bc0000"   2015-06-16T08:54:17 +04:00 f75f949ea8de466a9ef2bdb7ce065ac8
attachments/  1507476156    885 Strong Place, Cassel, Montana, 2069       "2602be56-0000-0000-0000-59da42bc0000"   2015-03-20T07:21:47 +04:00 ef0365fb40c04bb6a3ffc4bc77c905fd
attachments/  1507476156    515 Barwell Terrace, Defiance, Tennessee, 6439     "2602c056-0000-0000-0000-59da42bc0000"   2014-10-16T06:49:04 +04:00      e913fe543490432f871bc42019663518
attachments/  1507476156    570 Ruby Street, Spokane, Idaho, 9025       "2602c156-0000-0000-0000-59da42bc0000"   2014-10-30T05:49:33 +04:00 e53072057d314bc9b36c89a8350048f3
```

> [!NOTE]
> Le serveur Cosmos DB lié ne prend pas en charge les noms en quatre parties. Un message d’erreur de ce type est retourné :

```
Msg 7312, Level 16, State 1, Line 44

Invalid use of schema or catalog for OLE DB provider "MSDASQL" for linked server "DEMOCOSMOS". A four-part name was supplied, but the provider does not expose the necessary interfaces to use a catalog or schema.
``` 

## <a name="optional-creating-views"></a>(Facultatif) Création de vues
Vous pouvez définir et créer des vues dans le cadre du processus d’échantillonnage. Ces vues sont équivalentes aux vues SQL. Elles sont en lecture seule et affichent les sélections et les projections de la requête SQL Azure Cosmos DB définie. 

Pour créer une vue de vos données, dans la fenêtre **Éditeur de schéma**, dans la colonne **View Definitions** (Définitions de vue), cliquez sur **Add** (Ajouter) sur la ligne du conteneur à échantillonner. 

:::image type="content" source="./media/odbc-driver/odbc-driver-create-view.png" alt-text="Création d’une vue des données":::


Puis, dans la fenêtre **View Definitions** (Définitions de la vue), procédez comme suit :

1. Cliquez sur **New** (Nouveau), entrez un nom pour la vue, par exemple, EmployeesfromSeattleView, puis cliquez **OK**.

1. Dans la fenêtre **Modifier l’affichage**, entrez une requête Azure Cosmos DB. Utilisez obligatoirement une [requête SQL Azure Cosmos DB](./sql-query-getting-started.md), par exemple `SELECT c.City, c.EmployeeName, c.Level, c.Age, c.Manager FROM c WHERE c.City = "Seattle"`, puis cliquez sur **OK**.

    :::image type="content" source="./media/odbc-driver/odbc-driver-create-view-2.png" alt-text="Ajout d’une requête lors de la création d’une vue":::


Vous pouvez créer autant de vues que vous le souhaitez. Une fois que vous avez terminé la définition des vues, vous pouvez échantillonner les données. 

## <a name="step-5-view-your-data-in-bi-tools-such-as-power-bi-desktop"></a>Étape 5 : Affichage de vos données dans des outils décisionnels comme Power BI Desktop

Vous pouvez utiliser votre nouveau DSN pour vous connecter à Azure Cosmos DB avec n’importe quel outil compatible ODBC. Cette étape vous indique simplement comment vous connecter à Power BI Desktop et créer une visualisation Power BI.

1. Ouvrez Power BI Desktop.

1. Cliquez sur **Get Data** (Obtenir les données).

    :::image type="content" source="./media/odbc-driver/odbc-driver-power-bi-get-data.png" alt-text="Obtenir les données dans Power BI Desktop":::

1. Dans la fenêtre **Get Data** (Obtenir les données), cliquez sur **Other** (Autre)  | **ODBC** | **Connect** (Se connecter).

    :::image type="content" source="./media/odbc-driver/odbc-driver-power-bi-get-data-2.png" alt-text="Choix de la source de données ODBC dans l’option Obtenir les données de Power BI":::

1. Dans la fenêtre **From ODBC** (Depuis ODBC), sélectionnez le nom de source de données que vous avez créé, puis cliquez sur **OK**. Vous pouvez laisser les entrées **Options avancées** vides.

   :::image type="content" source="./media/odbc-driver/odbc-driver-power-bi-get-data-3.png" alt-text="Choisir le nom de la source de données de l’option Obtenir des données de Power BI":::

1. Dans la fenêtre **Accéder à une source de données à l’aide d’un pilote ODBC**, sélectionnez **Par défaut ou Personnalisé** , puis cliquez sur **Connecter**. Vous n’avez pas besoin d’inclure les **propriétés de la chaîne d’informations d’identification**.

1. Dans la fenêtre du **navigateur**, dans le volet gauche, développez la base de données, le schéma, puis sélectionnez la table. Le volet des résultats inclut les données en utilisant le schéma que vous avez créé.

    :::image type="content" source="./media/odbc-driver/odbc-driver-power-bi-get-data-4.png" alt-text="Sélection de la table dans l’option Obtenir les données de Power BI":::

1. Pour visualiser les données dans Power BI Desktop, cochez la case en regard du nom de la table, puis cliquez sur **Charger**.

1. Dans Power BI Desktop, à l’extrême gauche, sélectionnez l’onglet Données :::image type="icon" source="./media/odbc-driver/odbc-driver-data-tab.png"::: pour confirmer que vos données ont été importées. 

1. Vous pouvez désormais créer des éléments visuels à l’aide de Power BI en cliquant sur l’onglet :::image type="icon" source="./media/odbc-driver/odbc-driver-report-tab.png":::, sur **Nouvel élément visuel**, puis en personnalisation votre mosaïque. Pour plus d’informations sur la création de visualisations dans Power BI Desktop, consultez [Types de visualisation dans Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-visualization-types-for-reports-and-q-and-a/). 

## <a name="troubleshooting"></a>Dépannage

Si l’erreur suivante s’affiche, vérifiez que les valeurs **Hôte** et **Clé d’accès** que vous avez copiées sur le portail Azure à l’[étape 2](#connect) sont correctes, puis réessayez. Utilisez les boutons de copie à droite des valeurs **Hôte** et **Clé d’accès** sur le portail Azure pour copier les valeurs correctes.

```output
[HY000]: [Microsoft][Azure Cosmos DB] (401) HTTP 401 Authentication Error: {"code":"Unauthorized","message":"The input authorization token can't serve the request. Please check that the expected payload is built as per the protocol, and check the key being used. Server used the following payload to sign: 'get\ndbs\n\nfri, 20 jan 2017 03:43:55 gmt\n\n'\r\nActivityId: 9acb3c0d-cb31-4b78-ac0a-413c8d33e373"}
```


## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Azure Cosmos DB, consultez la page [Bienvenue dans Azure Cosmos DB](introduction.md).