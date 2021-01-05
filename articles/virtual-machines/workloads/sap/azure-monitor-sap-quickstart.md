---
title: Déployer Azure Monitor pour les solutions SAP avec le portail Azure
description: Déployer Azure Monitor pour les solutions SAP avec le portail Azure
author: sameeksha91
ms.author: sakhare
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: workloads
ms.date: 08/17/2020
ms.reviewer: cynthn
ms.openlocfilehash: c3b3848e4e4f7b0445f882265dbe66bb10b48833
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968585"
---
# <a name="deploy-azure-monitor-for-sap-solutions-with-azure-portal"></a>Déployer Azure Monitor pour les solutions SAP avec le portail Azure

Les ressources de Azure Monitor pour les solutions SAP peuvent être créées via le [portail Azure](https://azure.microsoft.com/features/azure-portal). Cette méthode fournit une interface utilisateur basée sur un navigateur pour déployer Azure Monitor pour les solutions SAP et configurer des fournisseurs.

## <a name="sign-in-to-azure-portal"></a>Se connecter au portail Azure

Connectez-vous au portail Azure sur https://portal.azure.com

## <a name="create-monitoring-resource"></a>Créer des ressources de surveillance

1. Sélectionnez **Solutions Azure Monitor pour solutions SAP** dans la **place de marché Azure**.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-1.png" alt-text="L’image montre l’offre de sélection Azure Monitor pour l’offre de solutions SAP à partir de la place de marché Azure." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-1.png":::

2. Sous l’onglet **De base**, indiquez les valeurs requises. Le cas échéant, vous pouvez utiliser un espace de travail Log Analytics existant.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-2.png" alt-text="Affichage des options de configuration du portail Azure." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-2.png":::

3. Lorsque vous sélectionnez un réseau virtuel, assurez-vous que les systèmes que vous souhaitez surveiller sont accessibles à partir de ce réseau virtuel. 

   > [!IMPORTANT]
   > Si vous sélectionnez **Partager** pour le partage de données avec Microsoft, nos équipes de support technique peuvent fournir un support supplémentaire.

## <a name="configure-providers"></a>Configurer des fournisseurs

### <a name="sap-hana-provider"></a>Fournisseur SAP HANA 

1. Sélectionnez l’onglet **Fournisseur** pour ajouter les fournisseurs que vous souhaitez configurer. Vous pouvez ajouter plusieurs fournisseurs l’un après l’autre ou les ajouter après avoir déployé la ressource de surveillance. 

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-3.png" alt-text="Affiche l’onglet Fournisseur pour ajouter des fournisseurs supplémentaires à votre Azure Monitor pour solutions SAP." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-3.png":::

2. Sélectionnez **Ajouter un fournisseur** et choisissez **SAP HANA** dans la liste déroulante. 

   > [!IMPORTANT]
   > Assurez-vous qu’un fournisseur SAP HANA est configuré pour le nœud « master » SAP HANA.

3. Entrez l’adresse IP privée du serveur HANA.

4. Entrez le nom du locataire de la base de données que vous voulez utiliser. Vous pouvez choisir n’importe quel locataire. Toutefois, nous vous recommandons d’utiliser **SYSTEMDB**, qui offre un ensemble plus vaste de zones de surveillance. 

5. Entrez le numéro de port SQL associé à votre base de données HANA. Le numéro de port doit être au format **[3]**  +  **[instance#]**  +  **[13]** . Par exemple, 30013. 

6. Entrez le nom d'utilisateur de la base de données à utiliser. Assurez-vous que les rôles de **surveillance** et de **lecture de catalogue** sont attribués à l’utilisateur de la base de données. 

7. Lorsque vous avez terminé, sélectionnez **Ajouter un fournisseur**. Continuez à ajouter des fournisseurs supplémentaires en fonction des besoins ou sélectionnez **Passer en revue + créer** pour terminer le déploiement.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-4.png" alt-text="Image des options de configuration lors de l’ajout d’informations sur le fournisseur." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-4.png":::

### <a name="high-availability-cluster-pacemaker-provider"></a>Fournisseur de cluster à haute disponibilité (Pacemaker)

1. Sélectionnez **Cluster à haute disponibilité (Pacemaker)** dans la liste déroulante. 

   > [!IMPORTANT]
   > Pour configurer le fournisseur de cluster à haute disponibilité (Pacemaker), assurez-vous que ha_cluster_provider est installé sur chaque nœud. Pour plus d’informations, consultez [Exportateur de cluster à haute disponibilité](https://github.com/ClusterLabs/ha_cluster_exporter#installation)

2. Entrez le point de terminaison Prometheus sous la forme http://IP:9664/metrics. 
 
3. Entrez l’ID système (SID), le nom d'hôte et le nom du cluster.

4. Lorsque vous avez terminé, sélectionnez **Ajouter un fournisseur**. Continuez à ajouter des fournisseurs supplémentaires en fonction des besoins ou sélectionnez **Passer en revue + créer** pour terminer le déploiement.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-5.png" alt-text="L’image montre les options relatives au fournisseur de stimulateur de cluster à haute disponibilité Pacemaker." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-5.png":::


### <a name="microsoft-sql-server-provider"></a>Fournisseur Microsoft SQL Server

1. Avant d’ajouter le fournisseur Microsoft SQL Server, vous devez exécuter le script suivant dans SQL Server Management Studio pour créer un utilisateur avec les autorisations appropriées nécessaires à la configuration du fournisseur.

   ```sql
   USE [<Database to monitor>]
   DROP USER [AMS]
   GO
   USE [master]
   DROP USER [AMS]
   DROP LOGIN [AMS]
   GO
   CREATE LOGIN [AMS] WITH PASSWORD=N'<password>', DEFAULT_DATABASE=[<Database to monitor>], DEFAULT_LANGUAGE=[us_english], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
   CREATE USER AMS FOR LOGIN AMS
   ALTER ROLE [db_datareader] ADD MEMBER [AMS]
   ALTER ROLE [db_denydatawriter] ADD MEMBER [AMS]
   GRANT CONNECT TO AMS
   GRANT VIEW SERVER STATE TO AMS
   GRANT VIEW SERVER STATE TO AMS
   GRANT VIEW ANY DEFINITION TO AMS
   GRANT EXEC ON xp_readerrorlog TO AMS
   GO
   USE [<Database to monitor>]
   CREATE USER [AMS] FOR LOGIN [AMS]
   ALTER ROLE [db_datareader] ADD MEMBER [AMS]
   ALTER ROLE [db_denydatawriter] ADD MEMBER [AMS]
   GO
   ``` 

2. Sélectionnez **Ajouter un fournisseur** et choisissez **Microsoft SQL Server** dans la liste déroulante. 

3. Renseignez les champs à l’aide des informations associées à votre Microsoft SQL Server. 

4. Lorsque vous avez terminé, sélectionnez **Ajouter un fournisseur**. Continuez à ajouter des fournisseurs supplémentaires en fonction des besoins ou sélectionnez **Passer en revue + créer** pour terminer le déploiement.

     :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-6.png" alt-text="L’image montre des informations relatives à l’ajout du fournisseur Microsoft SQL Server." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-6.png":::

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [Azure Monitor pour solutions SAP](azure-monitor-overview.md)
