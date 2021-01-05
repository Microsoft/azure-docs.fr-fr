---
title: Monitorage des performances d’une base de données multilocataire partitionnée
description: Surveiller et gérer les performances d’une instance Azure SQL Database multi-locataire partitionnée dans une application SaaS multi-locataire
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: d37bf2c84b74dba76e5d1921ed67072af7f6c328
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790897"
---
# <a name="monitor-and-manage-performance-of-sharded-multi-tenant-azure-sql-database-in-a-multi-tenant-saas-app"></a>Surveiller et gérer les performances d’une instance Azure SQL Database multi-locataire partitionnée dans une application SaaS multi-locataire
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Ce didacticiel aborde plusieurs scénarios de gestion de performance clés utilisés dans les applications SaaS. Les fonctionnalités intégrées de surveillance et d’alerte Azure SQL Database sont illustrés à l’aide d’un générateur de charge destiné à simuler l’activité de plusieurs bases de données multi-locataires partitionnées.

L’application de base de données multi-locataire SaaS Wingtip Tickets utilise un modèle de données multi-locataires partitionnées, où les données du lieu (locataire) peuvent être réparties par ID de locataire sur plusieurs bases de données. Comme de nombreuses applications SaaS, le modèle de charge de travail de locataire anticipé est imprévisible et sporadique. En d’autres termes, les ventes de tickets peuvent se produire à tout moment. Pour tirer parti de ce modèle d’utilisation de base de données type, vous pouvez augmenter ou réduire la taille des bases de données pour optimiser le coût d’une solution. Avec ce type de modèle, il est important de surveiller l’utilisation des ressources des bases de données pour veiller à ce que les charges soient raisonnablement équilibrées entre éventuellement plusieurs bases de données. Vous devez également veiller à ce que les bases de données aient des ressources appropriées et qu’elles n’atteignent pas les limites [DTU](purchasing-models.md#dtu-based-purchasing-model). Ce didacticiel explore plusieurs moyens de surveiller et de gérer des bases de données et montre comment prendre des mesures correctives en réponse aux variations de la charge de travail.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> 
> * Simuler l’utilisation sur une base de données multi-locataire partitionnée en exécutant un générateur de charge fourni
> * Surveiller la base de données à mesure qu’elle répond à l’augmentation de la charge
> * Augmenter la taille de la base de données en réponse à la charge accrue sur la base de données
> * Provisionner un locataire dans une base de données à un seul locataire

Pour suivre ce didacticiel, vérifiez que les prérequis suivants sont remplis :

* L’application de base de données multilocataire SaaS Wingtip Tickets est déployée. Pour procéder à un déploiement en moins de cinq minutes, consultez [Déployer et explorer l’application de base de données multi-locataire SaaS Wingtip Tickets](./saas-multitenantdb-get-started-deploy.md)
* Azure PowerShell est installé. Pour plus d’informations, voir [Bien démarrer avec Azure PowerShell](/powershell/azure/get-started-azureps).

## <a name="introduction-to-saas-performance-management-patterns"></a>Présentation des modèles de gestion de la performance SaaS

La gestion des performances des bases de données se compose des opérations suivantes : compilation et analyse des données de performances, puis réaction à ces données en ajustant les paramètres afin de conserver un temps de réponse acceptable pour votre application. 

### <a name="performance-management-strategies"></a>Stratégies de gestion des performances

* Pour éviter de devoir analyser manuellement les performances, il est plus efficace de **définir des alertes qui se déclenchent si les bases de données s’éloignent des limites normales** .
* Pour répondre aux fluctuations à court terme de la taille de calcul d’une base de données, **vous pouvez augmenter ou diminuer le niveau DTU** . Si cette fluctuation est régulière ou prévisible, **vous pouvez planifier la mise à l’échelle automatique de la base de données** . Par exemple, diminuez la taille du pool lorsque vous savez que votre charge de travail est faible, disons la nuit ou le week-end.
* Pour répondre aux fluctuations à plus long terme ou aux changements de locataires, **vous pouvez déplacer des locataires spécifiques dans d’autres bases de données** .
* Pour répondre aux augmentations à court terme de la charge de locataires *spécifiques* , **vous pouvez sortir ceux-ci d’une base de données et leur attribuer une taille de calcul spécifique** . Une fois que la charge est réduite, vous pouvez remettre le locataire dans la base de données multi-locataire. Si vous en avez connaissance à l’avance, les locataires peuvent être déplacés de manière préemptive pour vérifier que la base de données a toujours les ressources dont elle a besoin et éviter l’impact sur d’autres locataires de la base de données multilocataire. Si ce besoin est prévisible, par exemple un lieu accueillant un événement populaire avec une vente de tickets à grande échelle, ce comportement de gestion peut être intégré à l’application.

Le [portail Azure](https://portal.azure.com) offre des fonctionnalités intégrées de surveillance et d’alerte sur la plupart des ressources. Pour SQL Database, la surveillance et les alertes sont disponibles pour les bases de données. Ces fonctionnalités de surveillance et d’alertes intégrées sont propres à la ressource. Par conséquent, il est pratique de les utiliser pour un petit nombre de ressources, mais pas pour de nombreuses ressources.

Pour les scénarios à volume important où vous travaillez avec de nombreuses ressources, vous pouvez utiliser les [journaux Azure Monitor](https://azure.microsoft.com/services/log-analytics/). Il s’agit d’un service Azure distinct offrant l’analytique des journaux émis rassemblés dans un espace de travail Log Analytics. Les journaux Azure Monitor peuvent collecter des données de télémétrie à partir de nombreux services, et être utilisés pour interroger et définir des alertes.

## <a name="get-the-wingtip-tickets-saas-multi-tenant-database-application-source-code-and-scripts"></a>Obtenir les scripts et le code source de l’application de base de données multi-locataire SaaS Wingtip Tickets

Les scripts et le code source de l’application de base de données multilocataire Wingtip Tickets SaaS sont disponibles dans le dépôt GitHub [WingtipTicketsSaaS-MultitenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB). Consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md) avant de télécharger et de débloquer les scripts Wingtip Tickets SaaS.

## <a name="provision-additional-tenants"></a>Approvisionner des locataires supplémentaires

Pour une bonne compréhension du fonctionnement de la gestion et de la surveillance des performances à grande échelle, ce didacticiel nécessite la présence de plusieurs locataires dans une base de données multi-locataire partitionnée.

Si vous avez déjà provisionné un lot de locataires dans le cadre d’un didacticiel précédent, passez à la section [Simuler l’utilisation sur toutes les bases de données de locataire](#simulate-usage-on-all-tenant-databases).

1. Ouvrez... **Modules d’apprentissage** Analyse et gestion des performances\\\\Demo-PerformanceMonitoringAndManagement.ps1\\ dans *PowerShell ISE* . Gardez ce script ouvert, car vous exécuterez plusieurs scénarios au cours de ce didacticiel.
1. Définissez **$DemoScenario** = **1** , _Approvisionner un lot de locataires_
1. Appuyez sur **F5** pour exécuter le script.

En quelques minutes, le script déploie 17 locataires dans la base de données multi-locataire. 

Le script *New-TenantBatch* crée des locataires avec des clés de locataire uniques au sein de la base de données multi-locataire partitionnée et les initialise avec le nom du locataire et le type de lieu. Ce comportement est cohérent avec la manière dont l’application approvisionne un nouveau locataire. 

## <a name="simulate-usage-on-all-tenant-databases"></a>Simuler l’utilisation sur toutes les bases de données de locataire

Le script *Demo-PerformanceMonitoringAndManagement.ps1* simule une charge de travail s’exécutant sur la base de données multi-locataire. La charge est générée à l’aide de l’un des scénarios de charge disponibles :

| Démonstration | Scénario |
|:--|:--|
| 2 | Générer une charge d’intensité normale (environ 30 DTU) |
| 3 | Générer une charge avec des pics plus longs par locataire|
| 4 | Générer une charge avec des pics de DTU plus élevés par locataire (environ 70 DTU)|
| 5 | Générer une intensité élevée (environ 90 DTU) sur un locataire unique, plus une charge d’intensité normale sur tous les autres locataires |

Le générateur de charge applique une charge CPU *synthétique* à chaque base de données de locataire. Le générateur démarre un travail pour chaque base de données de locataire, qui appelle périodiquement une procédure stockée qui génère la charge. Les niveaux de charge (exprimés en DTU), la durée et les intervalles varient selon les bases de données pour simuler l’activité d’un locataire imprévisible.

1. Ouvrez... **Modules d’apprentissage** Analyse et gestion des performances\\\\Demo-PerformanceMonitoringAndManagement.ps1\\ dans *PowerShell ISE* . Gardez ce script ouvert, car vous exécuterez plusieurs scénarios au cours de ce didacticiel.
1. Définissez **$DemoScenario** = **2** , _Générer une charge d’intensité normale_ .
1. Appuyez sur **F5** pour appliquer une charge à tous vos locataires.

L’application de base de données multi-locataire SaaS Wingtip Tickets est une application SaaS, et dans le monde réel, la charge se trouvant sur une application SaaS est généralement sporadique et imprévisible. Pour simuler cette situation, le générateur de charge produit une charge aléatoire répartie sur tous les locataires. Plusieurs minutes étant nécessaires pour que le modèle de charge émerge, laissez au générateur de charge environ 3 à 5 minutes avant d’essayer de surveiller la charge dans les sections suivantes.

> [!IMPORTANT]
> Le générateur de charge s’exécute en tant que série de travaux dans une nouvelle fenêtre PowerShell. Si vous fermez la session, le générateur de charge s’arrête. Le générateur de charge reste à l’état *job-invoking* où il génère la charge sur les nouveaux clients approvisionnés après le démarrage du générateur. Utilisez *Ctrl-C* pour arrêter l’appel de nouvelles tâches et de quitter le script. Le générateur de charge continue de s’exécuter, mais uniquement sur les clients existants.

## <a name="monitor-resource-usage-using-the-azure-portal"></a>Surveiller l’utilisation des ressources via le portail Azure

Pour surveiller l’utilisation de ressources qui résulte de la charge appliquée, dans le portail, ouvrez la base de données multi-locataire **tenants1** contenant les locataires :

1. Ouvrez le [portail Azure](https://portal.azure.com), puis accédez au serveur *tenants1-mt-&lt;USER&gt;* .
1. Faites défiler vers le bas, recherchez les bases de données, puis cliquez sur **tenants1** . Cette base de données multi-locataire partitionnée contient tous les locataires créés jusqu’à présent.

![Graphique de base de données](./media/saas-multitenantdb-performance-monitoring/multitenantdb.png)

Observez le graphique **DTU** .

## <a name="set-performance-alerts-on-the-database"></a>Définir des alertes de performance sur la base de données

Définissez une alerte sur la base de données, qui se déclenche quand \>l’utilisation atteint 75 % comme suit :

1. Ouvrez la base de données *tenants1* (sur le serveur *tenants1-mt-&lt;USER&gt;* ) dans le [portail Azure](https://portal.azure.com).
1. Cliquez sur **Règles d’alerte** , puis sur **+ Ajouter une alerte** .

   ![ajouter une alerte](./media/saas-multitenantdb-performance-monitoring/add-alert.png)

1. Entrez un nom, par exemple **Nombre élevé de DTU** ,
1. Définissez les valeurs suivantes :
   * **Métrique = pourcentage DTU**
   * **Condition = supérieur à**
   * **Seuil = 75** .
   * **Période = Au cours des 30 dernières minutes**
1. Ajoutez une adresse e-mail à la zone *Adresse(s) e-mail administrateur supplémentaire(s)* , puis cliquez sur **OK** .

   ![définir une alerte](./media/saas-multitenantdb-performance-monitoring/set-alert.png)

## <a name="scale-up-a-busy-database"></a>Augmenter la taille d’une base de données occupée

Si le niveau de charge d’une base de données augmente jusqu’à atteindre la limite maximale de la base de données et l’utilisation de 100 % des DTU, le niveau de performance de la base de données est affecté, ce qui peut ralentir les temps de réponse des requêtes.

**À court terme** , songez à augmenter la taille de la base de données pour fournir des ressources supplémentaires, ou à supprimer des locataires de la base de données multi-locataire (en les faisant passer de la base de données à une base de données autonome).

**À long terme** , envisagez d’optimiser les requêtes ou l’utilisation de l’index pour améliorer les performances des bases de données. En fonction de la sensibilité de l’application aux problèmes de performances, il est recommandé d’augmenter la taille d’une base de données avant que celui-ci atteigne l’utilisation de 100 % des DTU. Utilisez une alerte pour vous avertir à l’avance.

Vous pouvez simuler une base de données occupée en augmentant la charge produite par le générateur. Cela a pour effet de demander aux locataires de fonctionner au maximum plus fréquemment et plus longtemps et d’augmenter la charge sur la base de données multi-locataire sans changer les besoins des locataires individuels. Vous pouvez facilement augmenter la taille de la base de données dans le portail ou à partir de PowerShell. Cet exercice utilise le portail.

1. Définissez *$DemoScenario* = **3** , _Générer une charge avec des pics plus longs et plus fréquents par base de données_ pour augmenter l’intensité de la charge globale sur la base de données sans changer le pic de charge exigé par chaque locataire.
1. Appuyez sur **F5** pour appliquer une charge à toutes vos bases de données de locataire.
1. Accédez à la base de données **tenants1** dans le portail Azure.

Surveillez l’augmentation de l’utilisation de DTU de la base de données sur le graphique du haut. L’affichage de la nouvelle charge supérieure peut prendre quelques minutes, mais vous devez rapidement voir que la base de données commence à atteindre une utilisation maximale, et qu’à mesure que la charge se stabilise dans le nouveau modèle, la base de données est rapidement surchargée.

1. Pour augmenter la taille de la base de données, cliquez sur **Niveau tarifaire (mise à l’échelle de DTU)** dans le panneau des paramètres.
1. Définissez le paramètre **DTU** avec la valeur **100** . 
1. Cliquez sur **Appliquer** pour envoyer la demande de mise à l’échelle de la base de données.

Revenez à **tenants1** > **Vue d’ensemble** pour afficher les graphiques de surveillance. Surveillez l’effet produit par la fourniture de davantage de ressources à la base de données (bien qu’avec quelques locataires et une charge aléatoire, il n’est pas toujours facile de voir l’effet de façon concluante avant un certain temps d’exécution). Quand vous examinez les graphiques, gardez à l’esprit le fait que 100 % sur le graphique du haut représente maintenant 100 DTU, alors que sur le graphique du bas, 100 % correspond toujours à 50 DTU.

La base de données reste en ligne et entièrement disponible tout au long du processus. Le code d’application doit toujours être écrit de façon à retenter les connexions ignorées et à ce que la reconnexion à la base de données ait lieu.

## <a name="provision-a-new-tenant-in-its-own-database"></a>Approvisionner un nouveau locataire dans sa propre base de données 

Le modèle multi-locataire partitionné vous permet de choisir s’il faut provisionner un nouveau locataire dans une base de données multi-locataire avec d’autres locataires, ou provisionner le locataire dans sa propre base de données. Quand vous provisionnez un locataire dans sa propre base de données, celui-ci tire parti de l’isolation inhérente à la base de données distincte, ce qui vous permet de gérer le niveau de performance de ce locataire indépendamment des autres, de le restaurer indépendamment des autres, etc. Par exemple, vous pouvez placer les clients standard ou ceux profitant d’un essai gratuit dans une base de données multi-locataire, et placer les clients premium dans des bases de données individuelles.  Si vous créez des bases de données à locataire unique isolées, vous pouvez continuer de les gérer collectivement dans un pool élastique pour optimiser les coûts des ressources.

Si vous avez déjà provisionné un nouveau locataire dans sa propre base de données, ignorez les étapes suivantes.

1. Dans **PowerShell ISE** , ouvrez …\\Learning Modules\\ProvisionTenants\\*Demo-ProvisionTenants.ps1* . 
1. Modifiez **$TenantName = "Salix Salsa"** et **$VenueType  = "dance"** .
1. Définissez **$Scenario** = **2** , _Provisionner un locataire dans une base de données à locataire unique_
1. Appuyez sur **F5** pour exécuter le script.

Le script provisionne ce locataire dans une base de données distincte, inscrit la base de données et le locataire auprès du catalogue, puis ouvre la page des événements du locataire dans le navigateur. Quand vous actualisez la page Event Hub, vous pouvez voir que « Salix Salsa » a été ajouté comme lieu.

## <a name="manage-performance-of-an-individual-database"></a>Gérer les performances d’une base de données individuelle

Si un locataire unique dans une base de données multi-locataire fait l’objet d’une charge élevée soutenue, il peut avoir tendance à dominer les ressources de la base de données et à impacter d’autres locataires dans la même base de données. Si l’activité est susceptible de continuer pendant un certain temps, le locataire peut être déplacé temporairement hors de la base de données et dans sa propre base de données à locataire unique. Cela permet au locataire de disposer des ressources supplémentaires dont il a besoin et d’être complètement isolé des autres locataires.

Cet exercice simule l’effet de Salix Salsa qui subit une charge élevée quand des tickets sont mis en vente pour un événement populaire.

1. Ouvrez le script …\\*Demo-PerformanceMonitoringAndManagement.ps1* .
1. Définissez **$DemoScenario = 5** , _Générer une charge normale et une charge élevée sur un locataire unique (environ 90 DTU)._
1. Définissez **$SingleTenantName = Salix Salsa** .
1. Exécutez le script en appuyant sur **F5** .

Accédez au portail et accédez à **salixsalsa** > **Vue d’ensemble** pour afficher les graphiques de surveillance. 

## <a name="other-performance-management-patterns"></a>Autres modèles de gestion des performances

**Mise à l’échelle libre-service par le locataire**

Comme la mise à l’échelle est une tâche facilement appelée par le biais de l’API Gestion, vous pouvez aisément configurer la possibilité de mettre à l’échelle des bases de données de locataire dans votre application côté locataire et la proposer en tant que fonctionnalité de votre service SaaS. Par exemple, laissez les locataires administrer la mise à l’échelle en liant ces opérations à leur facturation.

**Augmentation ou réduction de la taille d’une base de données selon une planification pour une correspondance avec les modèles d’utilisation**

Quand l’utilisation globale du locataire suit des modèles d’utilisation prévisibles, vous pouvez utiliser Azure Automation pour mettre à l’échelle une base de données selon une planification. Par exemple, diminuez la taille d’une base de données après 18 h et avant 6 h en semaine quand vous savez que les besoins en ressources baissent.

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Simuler l’utilisation sur une base de données multi-locataire partitionnée en exécutant un générateur de charge fourni
> * Surveiller la base de données à mesure qu’elle répond à l’augmentation de la charge
> * Augmenter la taille de la base de données en réponse à la charge accrue sur la base de données
> * Provisionner un locataire dans une base de données à un seul locataire

## <a name="additional-resources"></a>Ressources supplémentaires

<!--* [Additional tutorials that build upon the Wingtip Tickets SaaS Multi-tenant Database application deployment](saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)-->
* [Azure Automation](../../automation/automation-intro.md)