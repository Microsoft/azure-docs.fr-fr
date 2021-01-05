---
title: Importation de données Azure Log Analytics dans Power BI | Microsoft Docs
description: Power BI est un service Microsoft d’analyse commerciale basé sur le cloud qui fournit de riches fonctions de visualisation et de rapport afin de faciliter l’analyse de différents jeux de données.  Cet article explique comment configurer l'importation de données Log Analytics dans Power BI et les configurer pour s'actualiser automatiquement.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/01/2019
ms.openlocfilehash: 53277f64c3d1b03572732157756da1fececbcd43
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184567"
---
# <a name="import-azure-monitor-log-data-into-power-bi"></a>Importation de données de journal Azure Monitor dans Power BI


[Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-get-started/) est un service Microsoft d’analyse commerciale basé sur le cloud qui fournit de riches fonctions de visualisation et de rapport afin de faciliter l’analyse de différents jeux de données.  Vous pouvez importer les résultats d’une requête Log Azure Monitor dans un jeu de données Power BI afin de bénéficier de ses fonctionnalités, comme la combinaison de données provenant de différentes sources et le partage de rapports sur le web et sur des appareils mobiles.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="overview"></a>Vue d’ensemble
Pour importer les données d’un [espace de travail Log Analytics](manage-access.md) d’Azure Monitor dans Power BI, créez un jeu de données dans Power BI à partir d’une [requête de recherche dans les journaux](../log-query/log-query-overview.md) dans Azure Monitor.  La requête est exécutée chaque fois que le jeu de données est actualisé.  Vous pouvez ensuite créer des rapports Power BI qui utilisent les informations du jeu de données.  Pour créer le jeu de données dans Power BI, vous exportez votre requête depuis Log Analytics vers le [langage Power Query (M)](/powerquery-m/power-query-m-language-specification).  Vous l'utilisez ensuite pour créer une requête dans Power BI Desktop, avant de la publier dans Power BI en tant que jeu de données.  Les détails de ce processus sont décrits ci-dessous.

![Log Analytics vers Power BI](media/powerbi/overview.png)

## <a name="export-query"></a>Exporter une requête
Commencez par créer une [requête de journal](../log-query/log-query-overview.md) qui renvoie les données que vous voulez ajouter au jeu de données Power BI.  Vous exportez ensuite cette requête dans le [langage Power Query (M)](/powerquery-m/power-query-m-language-specification) qui peut être utilisé par Power BI Desktop.

1. [Créez la requête de journal Log Analytics](../log-query/log-analytics-tutorial.md) pour extraire les informations de votre jeu de données.
2. Sélectionnez **Exporter** > **Requête Power BI (M)** .  Cela a pour effet d’exporter la requête vers un fichier texte intitulé **PowerBIQuery.txt**. 

    ![Exporter une recherche de journal](media/powerbi/export-analytics.png)

3. Ouvrez le fichier texte et copiez son contenu.

## <a name="import-query-into-power-bi-desktop"></a>Importation d'une requête dans Power BI Desktop
Power BI Desktop est une application de bureau qui vous permet de créer des jeux de données et des rapports pouvant être publiés sur Power BI.  Vous pouvez également l’utiliser pour créer une requête à l’aide du langage Power Query exporté depuis Azure Monitor. 

1. Installez [Power BI Desktop](https://powerbi.microsoft.com/desktop/) si vous ne l'avez pas déjà, puis ouvrez l'application.
2. Sélectionnez **Obtenir les données** > **Requête vide** pour ouvrir une nouvelle requête.  Sélectionnez ensuite **Éditeur avancé** puis collez le contenu du fichier dans la requête. Cliquez sur **Done**.

    ![Requête Power BI Desktop](media/powerbi/desktop-new-query.png)

5. La requête s'exécute puis les résultats s'affichent.  Vous pouvez être invité à entrer vos informations d'identification pour vous connecter à Azure.  
6. Entrez un nom descriptif pour la requête.  La valeur par défaut est **Query1**. Cliquez sur **Fermer et appliquer** pour ajouter le jeu de données au rapport.

    ![Nom Power BI Desktop](media/powerbi/desktop-results.png)



## <a name="publish-to-power-bi"></a>Publier vers Power BI
Lorsque vous publiez sur Power BI, un jeu de données et un rapport sont créés.  Si vous créez un rapport dans Power BI Desktop, celui-ci sera publié avec vos données.  Sinon, un rapport vide sera créé.  Vous pouvez modifier le rapport dans Power BI, ou en créer un basé sur le jeu de données.

1. Créez un rapport basé sur vos données.  Consultez la [documentation Power BI Desktop](/power-bi/desktop-report-view) si vous avez besoin d'aide.  
1. Lorsque vous être prêt à envoyer le rapport à Power BI, cliquez sur **Publier**.  
1. À l'invite, sélectionnez une destination dans votre compte Power BI.  À moins de vouloir choisir une destination spécifique, utilisez **Mon espace de travail**.

    ![Publication Power BI Desktop](media/powerbi/desktop-publish.png)

1. Une fois la publication terminée, cliquez sur **Ouvrir dans Power BI** pour ouvrir Power BI avec votre nouveau jeu de données.


### <a name="configure-scheduled-refresh"></a>Configuration d'une actualisation planifiée
Le jeu de données créé dans Power BI contiendra les mêmes informations que celles que vous avez déjà consultées dans Power BI Desktop.  Vous devez régulièrement actualiser le jeu de données pour réexécuter la requête et la remplir avec les dernières informations d’Azure Monitor.  

1. Cliquez sur l'espace de travail dans lequel vous avez chargé votre rapport puis sélectionnez le menu **Jeux de données**. 
1. Sélectionnez le menu contextuel en regard de votre nouveau jeu de données puis choisissez **Paramètres**. 
1. Sous **Informations d'identification de la source de données**, vous devriez voir un message indiquant que les informations d'identification ne sont pas valides.  Cela est dû au fait que vous n'avez pas encore fourni d'informations d'identification pour le jeu de données à utiliser lors de l'actualisation de ses informations.  
1. Cliquez sur **Modifier les informations d’identification** et spécifiez les informations d’identification pour accéder à l’espace de travail Log Analytics dans Azure Monitor. Si vous avez besoin d’une authentification à deux facteurs, sélectionnez **OAuth2** pour demander à la **méthode d’authentification** une connexion à l’aide de vos informations d’identification.

    ![Planification Power BI](media/powerbi/powerbi-schedule.png)

5. Sous **Actualisation planifiée**, activez l'option pour **conserver vos données à jour**.  Vous pouvez aussi modifier la **fréquence d'actualisation** et une ou plusieurs heures spécifiques pour exécuter l'actualisation.

    ![Actualisation Power BI](media/powerbi/powerbi-schedule-refresh.png)



## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment les [recherches de journaux](../log-query/log-query-overview.md) peuvent vous aider à générer des requêtes pouvant être exportées vers Power BI.
* Découvrez comment utiliser [Power BI](https://powerbi.microsoft.com) pour créer des visualisations basées sur des exportations Azure Monitor.