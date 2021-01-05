---
title: Charger des données à partir de SAP Business Warehouse
description: Utiliser Azure Data Factory pour copier des données à partir de SAP Business Warehouse (BW)
services: data-factory
author: linda33wj
ms.author: jingwang
manager: shwang
ms.reviewer: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 05/22/2019
ms.openlocfilehash: bad9a706c5289966334af26eacbfa41c418b7ab5
ms.sourcegitcommit: d95cab0514dd0956c13b9d64d98fdae2bc3569a0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91360801"
---
# <a name="copy-data-from-sap-business-warehouse-by-using-azure-data-factory"></a>Copier des données de SAP Business Warehouse à l’aide d’Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article explique comment utiliser Azure Data Factory pour copier des données de SAP Business Warehouse (BW) via Open Hub vers Azure Data Lake Storage Gen2. Vous pouvez utiliser un processus similaire pour copier des données vers d’autres [banques de données de récepteur prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

> [!TIP]
> Pour des informations générales sur la copie des données à partir de SAP BW, y compris sur le flux d’intégration et d’extraction delta de SAP BW Open Hub, consultez l’article [Copier des données à partir de SAP Business Warehouse via Open Hub à l’aide d’Azure Data Factory](connector-sap-business-warehouse-open-hub.md).

## <a name="prerequisites"></a>Prérequis

- **Azure Data Factory** : Si vous n’en avez pas encore, suivez ces étapes pour [créer une fabrique de données](quickstart-create-data-factory-portal.md#create-a-data-factory).

- **Destination SAP BW Open Hub (OHD) avec type de destination « Database Table » (Table de base de données)** : Pour créer une OHD ou pour vérifier que votre OHD est correctement configurée pour l’intégration Data Factory, consultez la section [Configuration des destinations SAP BW Open Hub](#sap-bw-open-hub-destination-configurations) de cet article.

- **L’utilisateur SAP BW a besoin des autorisations suivantes** :

  - Autorisation relative au protocole RFC (Remote Function Calls) et à SAP BW.
  - Autorisations relatives à l’activité « Exécuter » de l’objet d’autorisation **S_SDSAUTH**.

- **Un [runtime d’intégration auto-hébergé (IR)](concepts-integration-runtime.md#self-hosted-integration-runtime) avec SAP .NET Connector 3.0**. Suivez ces étapes de configuration :

  1. Installez et inscrivez le runtime d’intégration auto-hébergé, version 3.13 ou ultérieure. (Cette procédure est décrite plus loin dans cet article.)

  2. Téléchargez la version [64 bits de SAP Connector for Microsoft .NET 3.0](https://support.sap.com/en/product/connectors/msnet.html) à partir du site web de SAP et installez-la sur l’ordinateur doté du runtime d’intégration auto-hébergé. Pendant l’installation, veillez à sélectionner l’option **Install Assemblies to GAC** (Installer les assemblys dans le GAC) de la boîte de dialogue **Optional setup steps** (Étapes de configuration facultatives), comme illustré dans l’image suivante :

     ![Boîte de dialogue Set up SAP .NET Connector (Configurer SAP .NET)](media/connector-sap-business-warehouse-open-hub/install-sap-dotnet-connector.png)

## <a name="do-a-full-copy-from-sap-bw-open-hub"></a>Effectuer une copie complète à partir de SAP BW Open Hub

Accédez à votre fabrique de données dans le Portail Azure. Sélectionnez **Créer et surveiller** pour ouvrir l’interface utilisateur de Data Factory dans un onglet séparé.

1. Dans la page **Prise en main**, sélectionnez **Copier des données** pour ouvrir l’outil Copier des données.

2. Dans la page **Propriétés**, spécifiez le **Nom de la tâche**, puis sélectionnez **Suivant**.

3. Dans la page **Banque de données sources**, sélectionnez **+ Créer une connexion**. Sélectionnez **SAP BW Open Hub** dans la galerie des connecteurs, puis sélectionnez **Continuer**. Vous pouvez taper **SAP** dans la zone de recherche pour filtrer les connecteurs.

4. Dans la page de **spécification de la connexion SAP BW Open Hub**, procédez comme suit pour créer une connexion.

   ![Créer la page Service lié de SAP BW Open Hub](media/load-sap-bw-data/create-sap-bw-open-hub-linked-service.png)

   1. Dans la liste **Se connecter via le runtime d’intégration**, sélectionnez un runtime d’intégration auto-hébergé existant. Si vous n’en avez pas encore, choisissez d’en créer un.

      Pour créer un runtime d’intégration auto-hébergé, sélectionnez **+Nouveau**, puis **Auto-hébergé**. Saisissez un **Nom**, et sélectionnez **Suivant**. Sélectionnez **Installation rapide** pour une installation sur l’ordinateur actuel, ou suivez les étapes **d’installation manuelle** fournies.

      Comme indiqué dans les [prérequis](#prerequisites), assurez-vous que SAP Connector for Microsoft .NET 3.0 est installé sur l’ordinateur exécutant le runtime d’intégration auto-hébergé.

   2. Renseignez le **Nom du serveur**, le **Numéro du système**, l’**ID client,** la **Langue** (si différente de **EN**), le **Nom d’utilisateur** et le **Mot de passe** associés à SAP BW.

   3. Sélectionnez **Tester la connexion** pour vérifier les paramètres, puis sélectionnez **Terminer**.

   4. Une connexion est créée. Sélectionnez **Suivant**.

5. Dans la page de **sélection des destinations Open Hub**, parcourez les destinations Open Hub disponibles dans votre instance SAP BW. Sélectionnez l’OHD à partir de laquelle copier les données, puis sélectionnez **Suivant**.

   ![Table de sélection des destinations SAP BW Open Hub](media/load-sap-bw-data/select-sap-bw-open-hub-table.png)

6. Spécifiez un filtre, si besoin. Si votre OHD contient uniquement des données provenant d’une exécution de processus de transfert de données (DTP) unique avec un seul ID de demande, ou si vous êtes sûr que votre DTP est terminé et que vous souhaitez copier les données, décochez la case **Exclude Last Request** (Exclure la dernière demande).

   Pour en savoir plus sur ces paramètres, reportez-vous à la section [Configuration des destinations SAP BW Open Hub](#sap-bw-open-hub-destination-configurations) de cet article. Sélectionnez **Validate** (Valider) pour vérifier quelles données seront renvoyées. Sélectionnez ensuite **Suivant**.

   ![Configurer le filtre SAP BW Open Hub](media/load-sap-bw-data/configure-sap-bw-open-hub-filter.png)

7. Dans la page **Banque de données de destination**, sélectionnez **+Créer une connexion** > **Azure Data Lake Storage Gen2** > **Continuer**.

8. Dans la page **Spécifier la connexion Azure Data Lake Storage**, procédez comme suit pour créer une connexion.

   ![Créer une page Service lié ADLS Gen2](media/load-sap-bw-data/create-adls-gen2-linked-service.png)

   1. Sélectionnez votre compte activé pour Data Lake Storage Gen2 dans la liste déroulante **Nom**.
   2. Sélectionnez **Terminer** pour créer la connexion. Sélectionnez ensuite **Suivant**.

9. Dans la page de **sélection du fichier ou dossier de sortie**, saisissez **copyfromopenhub** dans le champ du nom du dossier de sortie. Sélectionnez ensuite **Suivant**.

   ![Page de sélection du dossier de sortie](media/load-sap-bw-data/choose-output-folder.png)

10. Dans la page **Paramètres de format de fichier**, sélectionnez **Suivant** pour utiliser les paramètres par défaut.

    ![Page de spécification du format de récepteur](media/load-sap-bw-data/specify-sink-format.png)

11. Dans la page **Paramètres**, développez les **Paramètres de performances**. Entrez une valeur de **degré de parallélisme de copie** comme 5 pour charger les données à partir de SAP BW en parallèle. Sélectionnez ensuite **Suivant**.

    ![Configurer les paramètres de copie](media/load-sap-bw-data/configure-copy-settings.png)

12. Vérifiez les paramètres dans la page de **résumé**. Sélectionnez ensuite **Suivant**.

13. Dans la page **Déploiement**, sélectionnez **Surveiller** pour surveiller le pipeline.

    ![Page Déploiement](media/load-sap-bw-data/deployment.png)

14. Notez que l’onglet **Surveiller** sur la gauche de la page est sélectionné automatiquement. La colonne **Actions** comprend les liens permettant d’afficher les détails de l’exécution de l’activité et de réexécuter le pipeline.

    ![Vue de surveillance du pipeline](media/load-sap-bw-data/pipeline-monitoring.png)

15. Pour afficher les exécutions d’activités associées à l’exécution du pipeline, sélectionnez **Afficher les exécutions d’activités** dans la colonne **Actions**. Il n’y a qu’une seule activité (activité de copie) dans le pipeline ; vous ne voyez donc qu’une seule entrée. Pour revenir à l’affichage des exécutions du pipeline, sélectionnez le lien **Pipelines** affiché en haut de la fenêtre. Sélectionnez **Actualiser** pour actualiser la liste.

    ![Écran de surveillance des activités](media/load-sap-bw-data/activity-monitoring.png)

16. Pour surveiller les détails d’exécution de chaque activité de copie, sélectionnez le lien **Détails**, l’icône de lunettes, sous **Actions** dans la vue de surveillance des activités. Les informations disponibles incluent le volume de données copié de la source vers le récepteur, le débit des données, les étapes et la durée de l’exécution, ainsi que les configurations utilisées.

    ![Détails de surveillance des activités](media/load-sap-bw-data/activity-monitoring-details.png)

17. Pour afficher l’**ID de demande maximale**, revenez à la vue de surveillance des activités et sélectionnez **Sortie** sous **Actions**.

    ![Écran de sortie d’activité](media/load-sap-bw-data/activity-output.png)

    ![Vue des détails de sortie de l’activité](media/load-sap-bw-data/activity-output-details.png)

## <a name="incremental-copy-from-sap-bw-open-hub"></a>Copie incrémentielle à partir de SAP BW Open Hub

> [!TIP]
> Consultez l’article [Flux d’extraction delta de SAP BW Open Hub Connector](connector-sap-business-warehouse-open-hub.md#delta-extraction-flow) pour savoir comment le connecteur SAP BW Open Hub de Data Factory copie les données incrémentielles à partir de SAP BW. Cet article peut également vous aider à comprendre la configuration de base du connecteur.

Continuons la configuration de la copie incrémentielle à partir de SAP BW Open Hub.

La copie incrémentielle utilise un mécanisme de « limite supérieure » basé sur l’**ID de demande**. Cet ID est généré automatiquement dans la destination SAP BW Open Hub par le processus DTP de transfert de données. Le diagramme qui suit explique ce flux de travail :

![Diagramme du flux de travail de copie incrémentielle](media/load-sap-bw-data/incremental-copy-workflow.png)

Dans la page **Prise en main** de la fabrique de données, sélectionnez **Créer un pipeline à partir d’un modèle** pour utiliser le modèle intégré.

1. Recherchez **SAP BW** et sélectionnez le modèle **Incremental copy from SAP BW to Azure Data Lake Storage Gen2** (Copie incrémentielle à partir de SAP BW vers Azure Data Lake Storage Gen2). Ce modèle permet de copier les données dans Azure Data Lake Storage Gen2. Vous pouvez utiliser un flux de travail similaire pour copier les données vers d’autres types de récepteurs.

2. Dans la page principale du modèle, sélectionnez ou créez les trois connexions suivantes, puis choisissez **Utiliser ce modèle** dans le coin inférieur droit de la fenêtre.

   - **Stockage Blob Azure** : Dans cette procédure pas-à-pas, nous utilisons le stockage Blob Azure pour stocker la limite supérieure, c’est-à-dire l’*ID de demande de données maximales copiées*.
   - **SAP BW Open Hub** : Il s’agit de la source à partir de laquelle copier les données. Reportez-vous à la procédure pas-à-pas précédente concernant la copie complète pour connaître la configuration détaillée.
   - **Azure Data Lake Storage Gen2** : Il s’agit du récepteur vers lequel copier les données. Reportez-vous à la procédure pas-à-pas précédente concernant la copie complète pour connaître la configuration détaillée.

   ![Modèle pour la copie incrémentielle à partir de SAP BW](media/load-sap-bw-data/incremental-copy-from-sap-bw-template.png)

3. Ce modèle génère un pipeline contenant les trois activités suivantes, chaînées en cas de réussite : *Recherche*, *Copier des données* et *Web*.

   Accédez à l’onglet **Paramètres** du pipeline. Il contient tous les paramètres de configuration que vous devez fournir.

   ![Configuration de la copie incrémentielle à partir de SAP BW](media/load-sap-bw-data/incremental-copy-from-sap-bw-pipeline-config.png)

   - **SAPOpenHubDestinationName** : Spécifiez le nom de la table Open Hub à partir de laquelle copier les données.

   - **Data_Destination_Container** : spécifiez la destination du conteneur Azure Data Lake Storage Gen2 vers lequel copier les données. Si le conteneur n’existe pas, l’activité de copie Data Factory en crée un lors de l’exécution.
  
   - **Data_Destination_Directory** : spécifiez le chemin d’accès du dossier sous le conteneur Azure Data Lake Storage Gen2 vers lequel copier les données. Si le chemin d’accès n’existe pas, l’activité de copie Data Factory en crée un lors de l’exécution.
  
   - **HighWatermarkBlobContainer** : spécifiez le conteneur dans lequel stocker la valeur de limite supérieure.

   - **HighWatermarkBlobDirectory** : spécifiez le chemin d’accès du dossier sous le conteneur dans lequel stocker la valeur de limite supérieure.

   - **HighWatermarkBlobName** : Spécifiez le nom de l’objet blob de stockage de la valeur de limite supérieure, par exemple `requestIdCache.txt`. Dans le stockage Blob, accédez au chemin d’accès HighWatermarkBlobContainer+HighWatermarkBlobDirectory+HighWatermarkBlobName, tel que *container/path/requestIdCache.txt*. Créez un fichier avec un contenu 0.

      ![Contenu de l’objet blob](media/load-sap-bw-data/blob.png)

   - **LogicAppURL** : Dans ce modèle, nous utilisons WebActivity pour appeler Azure Logic Apps et définir la valeur de limite supérieure dans le stockage Blob. Sinon, vous pouvez utiliser Azure SQL Database pour la stocker. Utilisez une activité de procédure stockée pour mettre à jour la valeur.

      Vous devez d’abord créer une application logique, comme le montre l’image suivante. Ensuite, collez l’**URL HTTP POST**.

      ![Configuration des applications logiques](media/load-sap-bw-data/logic-app-config.png)

      1. Accédez au portail Azure. Sélectionnez un nouveau service **Logic Apps**. Sélectionnez **+Application logique vide** pour accéder au **Concepteur Logic Apps**.

      2. Créez un déclencheur **Lors de la réception d’une demande HTTP**. Spécifiez le corps de la demande HTTP comme suit :

         ```json
         {
            "properties": {
               "sapOpenHubMaxRequestId": {
                  "type": "string"
               }
            },
            "type": "object"
         }
         ```

      3. Ajoutez une action **Créer un objet blob**. Pour **Chemin d’accès du dossier** et **Nom d’objet Blob**, utilisez les mêmes valeurs que celles configurées précédemment au niveau des paramètres *HighWatermarkBlobContainer+HighWatermarkBlobDirectory* et *HighWatermarkBlobName*.

      4. Sélectionnez **Enregistrer**. Ensuite, copiez la valeur de l’**URL HTTP POST** à utiliser dans le pipeline Data Factory.

4. Après avoir fourni les paramètres du pipeline Data Factory, sélectionnez **Déboguer** > **Terminer** pour exécuter la validation de la configuration. Sinon, sélectionnez **Publier** pour publier toutes les modifications, puis choisissez **Ajouter un déclencheur** pour effectuer l’exécution.

## <a name="sap-bw-open-hub-destination-configurations"></a>Configuration des destinations SAP BW Open Hub

Cette section présente la configuration côté SAP BW pour l’utilisation du connecteur SAP BW Open Hub dans Data Factory pour copier des données.

### <a name="configure-delta-extraction-in-sap-bw"></a>Configurer l’extraction delta dans SAP BW

Si vous avez besoin d’exécuter à la fois une copie historique et incrémentielle, ou simplement une copie incrémentielle, configurez l’extraction delta dans SAP BW.

1. Créez la destination Open Hub. Vous pouvez créer la destination Open Hub dans SAP Transaction RSA1, qui crée automatiquement le processus de transformation et de transfert de données nécessaire. Utilisez les paramètres suivants :

   - **ObjectType** : Vous pouvez utiliser n’importe quel type d’objet. Ici, nous utilisons **InfoCube** comme exemple.
   - **Type de destination** : Sélectionnez **Database Table** (Table de base de données).
   - **Key of the Table** (Clé de la table) : Sélectionnez **Technical Key** (Clé technique).
   - **Extraction** : Sélectionnez **Keep Data and Insert Records into Table** (Conserver les données et insérer les enregistrements dans la table).

   ![Boîte de dialogue de création de l’extraction delta vers la destination SAP BW Open Hub](media/load-sap-bw-data/create-sap-bw-ohd-delta.png)

   ![Boîte de dialogue de création de l’extraction delta2 vers la destination SAP BW Open Hub](media/load-sap-bw-data/create-sap-bw-ohd-delta2.png)

   Vous pouvez augmenter le nombre de processus de travail SAP exécutés en parallèle pour le transfert de données :

   ![La capture d’écran montre la boîte de dialogue Settings for Parallel Processing (Paramètres pour le traitement parallèle) où vous pouvez sélectionner le nombre de processus parallèles pour le processus de transfert des données.](media/load-sap-bw-data/create-sap-bw-ohd-delta3.png)

2. Planifiez le transfert de données dans des chaînes de processus.

   Un transfert de données delta pour un cube fonctionne uniquement si les lignes nécessaires n’ont pas été compressées. Assurez-vous que la compression du cube BW n’est pas en cours d’exécution avant le transfert de données vers la table Open Hub. Pour ce faire, le plus simple consiste à intégrer le processus DTP dans vos chaînes de processus existantes. Dans l’exemple suivant, le processus DTP (vers l’OHD) est inséré dans la chaîne de processus entre les étapes *Ajuster* (cumul d’agrégats) et *Réduire* (compression du cube).

   ![Diagramme de création de flux de chaîne de processus SAP BW](media/load-sap-bw-data/create-sap-bw-process-chain.png)

### <a name="configure-full-extraction-in-sap-bw"></a>Configurer l’extraction complète dans SAP BW

En plus de l’extraction delta, vous voudrez peut-être effectuer une extraction complète du même fournisseur SAP BW. Cela s’applique généralement si vous souhaitez effectuer une copie complète et non incrémentielle, ou si vous souhaitez [resynchroniser l’extraction delta](#resync-delta-extraction).

Vous ne pouvez pas définir plusieurs processus de transfert de données pour la même destination Open Hub. Par conséquent, vous devez créer une OHD supplémentaire avant l’extraction delta.

![Créer une OHD SAP BW pour copie complète](media/load-sap-bw-data/create-sap-bw-ohd-full.png)

Pour une destination OHD destinée à une copie complète, choisissez des options différentes de celles de l’extraction delta :

- Au niveau de l’OHD : Définissez l’option **Extraction** sur **Delete Data and Insert Records** (Supprimer les données et insérer les enregistrements). Sinon, les données seront extraites autant de fois que vous répéterez le transfert des données dans une chaîne de processus BW.

- Au niveau du processus de transfert des données : Définissez **Extraction Mode** (Extraction Mode) sur **Full** (Complet). Vous devez modifier le paramètre du processus DTP créé automatiquement de **Delta** à **Full** (Complet) immédiatement après la création de la destination OHD, comme indiqué dans cette image :

   ![Boîte de dialogue de création d’une OHD SAP BW configurée pour une extraction « Complète »](media/load-sap-bw-data/create-sap-bw-ohd-full2.png)

- Au niveau du connecteur BW Open Hub de Data Factory : Désactivez l’option **Exclude last request** (Exclure la dernière demande). Dans le cas contraire, aucune donnée ne sera extraite.

Vous exécutez généralement le processus de transfert des données complet manuellement. Sinon, vous pouvez créer une chaîne de processus pour le transfert de données complet. Il s’agit généralement d’une chaîne distincte indépendante de vos chaînes de processus existantes. Dans les deux cas, *assurez-vous que le processus DTP est terminé avant de commencer l’extraction avec la copie Data Factory*. Sinon, seules des données partielles seront copiées.

### <a name="run-delta-extraction-the-first-time"></a>Première exécution de l’extraction delta

La première extraction delta est techniquement une *extraction complète*. Par défaut, le connecteur SAP BW Open Hub exclut la dernière demande lors de la copie des données. Lors de la première extraction delta, aucune donnée n’est extraite par l’activité de copie Data Factory avant qu’un processus DTP ne génère des données delta dans la table avec un ID de demande distinct. Deux méthodes permettent d’éviter ce scénario :

- Désactivez l’option **Exclude last request** (Exclure la dernière demande) lors de la première extraction delta. Assurez-vous que le premier processus de transfert de données delta est terminé avant de commencer la première extraction delta.
-  Utilisez la procédure de resynchronisation de l’extraction delta, comme indiqué dans la section suivante.

### <a name="resync-delta-extraction"></a>Resynchroniser l’extraction delta

Les scénarios suivants modifient les données dans les cubes SAP BW, mais ils ne sont pas pris en compte par le processus de transfert de données delta :

- Suppression sélective SAP BW (des lignes avec n’importe quelle condition de filtre)
- Suppression des demandes SAP BW (des demandes défectueuses)

Une destination SAP Open Hub n’est pas une cible de données contrôlée par un mini-data warehouse (dans tous les packages de prise en charge SAP BW depuis 2015). Par conséquent, vous pouvez supprimer les données d’un cube sans modifier les données dans la destination OHD. Vous devez ensuite resynchroniser les données du cube avec Data Factory :

1. Exécutez une extraction complète dans Data Factory (en utilisant un processus DTP complet dans SAP).
2. Supprimez toutes les lignes de la table Open Hub pour le processus DTP delta.
3. Définissez l’état du processus DTP delta sur **Fetched** (Extraites).

Après cela, tous les processus DTP delta et extractions delta Data Factory fonctionneront comme prévu.

Pour définir l’état du processus DTP delta sur **Fetched** (Extraites), vous pouvez utiliser l’option suivante pour exécuter le transfert de données delta manuellement :

*Aucun transfert de données ; État Delta dans Source : Récupéré*

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la prise en charge du connecteur SAP BW Open Hub :

> [!div class="nextstepaction"]
>[Connecteur SAP Business Warehouse Open Hub](connector-sap-business-warehouse-open-hub.md)
