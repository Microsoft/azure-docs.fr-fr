---
title: Copie delta à partir d’une base de données à l’aide d’une table de contrôle
description: Découvrez comment utiliser un modèle de solution pour copier de façon incrémentielle uniquement les nouvelles lignes ou celles mises à jour dans une base de données avec Azure Data Factory.
services: data-factory
documentationcenter: ''
author: dearandyxu
ms.author: yexu
ms.reviewer: douglasl
manager: anandsub
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 12/09/2020
ms.openlocfilehash: 3f15ffd6f7ed784cce398d419e013a69c60289d8
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96921806"
---
# <a name="delta-copy-from-a-database-with-a-control-table"></a>Copie delta à partir d’une base de données avec une table de contrôle

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article décrit un modèle disponible pour charger de façon incrémentielle des lignes nouvelles ou mises à jour à partir d’une table de base de données vers Azure à l’aide d’une table de contrôle externe qui stocke une valeur limite supérieure.

Ce modèle nécessite que le schéma de base de données source contienne une colonne d’horodatage ou une clé d’incrémentation pour identifier les lignes nouvelles ou mises à jour.

>[!NOTE]
> Si vous avez une colonne d’horodatage dans votre base de données source pour identifier les lignes nouvelles ou mises à jour, mais ne souhaitez pas créer une table de contrôle externe à utiliser pour la copie delta, vous pouvez utiliser à la place l’[outil de copie de données Azure Data Factory](copy-data-tool.md) pour obtenir un pipeline. Cet outil utilise une heure planifiée de déclencheur en tant que variable pour lire les nouvelles lignes à partir de la base de données source.

## <a name="about-this-solution-template"></a>À propos de ce modèle de solution

Ce modèle récupère d’abord l’ancienne valeur limite supérieure, puis la compare à la valeur limite supérieure actuelle. Il copie ensuite uniquement les modifications apportées à la base de données source après comparaison des deux valeurs limites supérieures. Enfin, il stocke la nouvelle valeur limite supérieure dans une table de contrôle externe pour la prochaine opération de chargement de données delta.

Le modèle contient quatre activités :
- Une activité **Lookup** récupère l’ancienne valeur limite supérieure stockée dans une table de contrôle externe.
- Une autre activité **Lookup** récupère la valeur limite supérieure actuelle dans la base de données source.
- Une activité **Copy** copie uniquement les modifications de la base de données source vers le magasin de destination. La requête permettant d’identifier les modifications apportées à la base de données source ressemble à ceci : 'SELECT * FROM Data_Source_Table WHERE TIMESTAMP_Column > “last high-watermark” and TIMESTAMP_Column <= “current high-watermark”'.
- Une activité **SqlServerStoredProcedure** écrit la valeur limite supérieure actuelle dans une table de contrôle externe pour la prochaine opération de copie delta.

Le modèle définit les paramètres suivants :
- Le paramètre *Data_Source_Table_Name* est le nom de la table de la base de données source à partir de laquelle vous souhaitez charger des données.
- Le paramètre *Data_Source_WaterMarkColumn* est le nom de la colonne dans la table source qui permet d’identifier les lignes nouvelles ou mises à jour. Le type de cette colonne est généralement *datetime*, *INT* ou un type similaire.
- *Data_Destination_Container* indique l’emplacement racine où les données sont copiées dans votre magasin de destination.
- *Data_Destination_Directory* est le chemin du répertoire sous la racine de l’emplacement où les données sont copiées dans votre magasin de destination.
- *Data_Destination_Table_Name* est l’endroit où les données sont copiées dans votre magasin de destination (applicable quand « Azure Synapse Analytics » est sélectionné en tant que destination des données).
- *Data_Destination_Folder_Path* est l’endroit où les données sont copiées dans votre magasin de destination (applicable lorsque « Système de fichiers » ou « Azure Data Lake Storage Gen1 » est sélectionné comme destination des données).
- Le paramètre *Control_Table_Table_Name* indique la table de contrôle externe où la valeur de limite supérieure est stockée.
- Le paramètre *Control_Table_Column_Name* indique la colonne dans la table de contrôle externe où la valeur de limite supérieure est stockée.

## <a name="how-to-use-this-solution-template"></a>Utiliser ce modèle de solution

1. Explorez la table source que vous souhaitez charger, et définissez la colonne de limite supérieure qui peut être utilisée pour identifier les lignes nouvelles ou mises à jour. Le type de cette colonne peut être *datetime*, *INT* ou un type similaire. La valeur de cette colonne augmente à mesure que de nouvelles lignes sont ajoutées. À partir de l’exemple de table de source suivant (data_source_table), nous pouvons utiliser la colonne *LastModifytime* en tant que colonne de limite supérieure.

    ```sql
            PersonID    Name    LastModifytime
            1   aaaa    2017-09-01 00:56:00.000
            2   bbbb    2017-09-02 05:23:00.000
            3   cccc    2017-09-03 02:36:00.000
            4   dddd    2017-09-04 03:21:00.000
            5   eeee    2017-09-05 08:06:00.000
            6   fffffff 2017-09-06 02:23:00.000
            7   gggg    2017-09-07 09:01:00.000
            8   hhhh    2017-09-08 09:01:00.000
            9   iiiiiiiii   2017-09-09 09:01:00.000
    ```
    
2. Créez une table de contrôle dans SQL Server ou Azure SQL Database pour stocker la valeur limite supérieure pour le chargement de données delta. Dans l’exemple suivant, le nom de la table de contrôle est *watermarktable*. Dans ce tableau, *WatermarkValue* est la colonne qui stocke la valeur limite supérieure, et son type est *datetime*.

    ```sql
            create table watermarktable
            (
            WatermarkValue datetime,
            );
            INSERT INTO watermarktable
            VALUES ('1/1/2010 12:00:00 AM')
    ```
    
3. Créez une procédure stockée dans la même instance SQL Server ou Azure SQL Database que celle utilisée pour créer la table de contrôle. La procédure stockée sert à écrire la nouvelle valeur limite supérieure dans la table de contrôle externe pour la prochaine opération de chargement de données delta.

    ```sql
            CREATE PROCEDURE update_watermark @LastModifiedtime datetime
            AS

            BEGIN

                UPDATE watermarktable
                SET [WatermarkValue] = @LastModifiedtime 

            END
    ```
    
4. Accédez au modèle **Copie delta à partir d’une base de données**. Créez une **nouvelle** connexion à la base de données source à partir de laquelle vous copiez des données.

    ![Créer une connexion à la table source](media/solution-template-delta-copy-with-control-table/DeltaCopyfromDB_with_ControlTable4.png)

5. Créez une **nouvelle** connexion au magasin de données de destination vers lequel vous copiez les données.

    ![Créer une connexion à la table de destination](media/solution-template-delta-copy-with-control-table/DeltaCopyfromDB_with_ControlTable5.png)

6. Créez une **nouvelle** connexion à la table de contrôle externe et à la procédure stockée que vous avez créées aux étapes 2 et 3.

    ![Créer une connexion au magasin de données de la table de contrôle](media/solution-template-delta-copy-with-control-table/DeltaCopyfromDB_with_ControlTable6.png)

7. Sélectionnez **Utiliser ce modèle**.
    
8. Vous voyez le pipeline disponible, comme dans l’exemple suivant :
  
    ![Passer en revue le pipeline](media/solution-template-delta-copy-with-control-table/DeltaCopyfromDB_with_ControlTable8.png)

9. Sélectionnez **Procédure stockée**. Pour **Nom de la procédure stockée**, choisissez **[dbo].[update_watermark]** . Sélectionnez **Paramètre d’importation**, puis sélectionnez **Ajouter du contenu dynamique**.  

    ![Définir l’activité Procédure stockée](media/solution-template-delta-copy-with-control-table/DeltaCopyfromDB_with_ControlTable9.png)  

10. Écrivez le contenu **\@{activity(’LookupCurrentWaterMark’).output.firstRow.NewWatermarkValue}** , puis sélectionnez **Terminer**.  

    ![Écrire le contenu pour les paramètres de la procédure stockée](media/solution-template-delta-copy-with-control-table/DeltaCopyfromDB_with_ControlTable10.png)       
     
11. Sélectionnez **Déboguer**, entrez les **Paramètres**, puis sélectionnez **Terminer**.

    ![Sélectionner **Déboguer**](media/solution-template-delta-copy-with-control-table/DeltaCopyfromDB_with_ControlTable11.png)

12. Des résultats similaires à l’exemple suivant s’affichent :

    ![Vérifier le résultat](media/solution-template-delta-copy-with-control-table/DeltaCopyfromDB_with_ControlTable12.png)

13. Vous pouvez créer des lignes dans votre table source. Voici un exemple en langage SQL pour créer des lignes :

    ```sql
            INSERT INTO data_source_table
            VALUES (10, 'newdata','9/10/2017 2:23:00 AM')

            INSERT INTO data_source_table
            VALUES (11, 'newdata','9/11/2017 9:01:00 AM')
    ```

14. Pour réexécuter le pipeline, sélectionnez **Déboguer**, entrez les **Paramètres**, puis sélectionnez **Terminer**.

    Vous pouvez voir que seules les nouvelles lignes ont été copiées dans la destination.

15. (Facultatif :) Si vous avez choisi Azure Synapse Analytics comme destination des données, vous devez également fournir une connexion à un Stockage Blob Azure pour la mise en lots, conformément aux exigences d’Azure Synapse Analytics Polybase. Le modèle génère un chemin d’accès au conteneur pour vous. Après l’exécution du pipeline, vérifiez si le conteneur a été créé dans le stockage d’objets Blob.
    
    ![Configurer Polybase](media/solution-template-delta-copy-with-control-table/DeltaCopyfromDB_with_ControlTable15.png)
    
## <a name="next-steps"></a>Étapes suivantes

- [Copier en bloc à partir d’une base de données à l’aide d’une table de contrôle avec Azure Data Factory](solution-template-bulk-copy-with-control-table.md)
- [Copier des fichiers issus de plusieurs conteneurs avec Azure Data Factory](solution-template-copy-files-multiple-containers.md)
