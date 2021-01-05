---
title: Demander une augmentation de quota et obtenir de l’aide
description: Comment créer une demande de support dans le portail Azure pour Azure Synapse Analytics. Demandez une augmentation de quota ou obtenez de l’aide pour la résolution des problèmes.
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/10/2020
author: kevinvngo
ms.author: kevin
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: de45e338b0b863dc2364af399a6991f56658b0e7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85212273"
---
# <a name="request-quota-increases-and-get-support-for-azure-synapse-analytics"></a>Demander une augmentation du quota et obtenir de l’aide pour Azure Synapse Analytics

Cet article explique comment envoyer un ticket de support dans le portail Azure pour Azure Synapse Analytics. Ce processus vous permet de demander une augmentation de quota ou de soumettre une demande de support technique à l’équipe de support technique.

## <a name="create-a-support-ticket"></a>Création d’un ticket de support

Suivez les étapes ci-après pour créer une demande de support à partir du portail Azure pour Azure Synapse Analytics.

1. Dans le menu [Portail Azure](https://portal.azure.com), sélectionnez **Aide + support**.

   ![Lien « Aide + support »](./media/sql-data-warehouse-get-started-create-support-ticket/help-plus-support.png)


1. Dans **Aide + support**, sélectionnez **Nouvelle demande de support**.

    ![Créer une demande de support](./media/sql-data-warehouse-get-started-create-support-ticket/new-support-request.png)

1. Consultez votre [plan de support Azure](https://azure.microsoft.com/support/plans/?WT.mc_id=Support_Plan_510979/).

   * La **gestion de la facturation, des abonnements et des quotas** sont pris en charge à tous les niveaux de support.
   * Les problèmes **couverts par la garantie de réparation et d’assistance** sont pris en charge dans les plans de support suivants : [Developer](https://azure.microsoft.com/support/plans/developer/), [Standard](https://azure.microsoft.com/support/plans/standard/), [Professional Direct](https://azure.microsoft.com/support/plans/prodirect/) ou [Premier](https://azure.microsoft.com/support/plans/premier/). Les problèmes couverts par la garantie de réparation et d'assistance sont des problèmes rencontrés par les clients lors de l'utilisation d'Azure et qui peuvent être raisonnablement imputés à Microsoft.
   * Les **services de tutorat** et d’**expertise pour développeur** sont disponibles aux niveaux de support [Professional Direct](https://azure.microsoft.com/support/plans/prodirect/) et [Premier](https://azure.microsoft.com/support/plans/premier/).

   Si vous avez un plan de support Premier, vous pouvez également signaler les problèmes liés à Azure Synapse Analytics sur le [portail en ligne Microsoft Premier](https://premier.microsoft.com/). Consultez les [plans de support Azure](https://azure.microsoft.com/support/plans/?WT.mc_id=Support_Plan_510979/) pour en savoir plus sur les différents plans disponibles, y compris leur portée, leur temps de réponse, leur tarification, etc.  Pour accéder aux questions fréquemment posées sur le support Azure, consultez la page [FAQ du support Azure](https://azure.microsoft.com/support/faq/).

1. Pour **Type de problème**, sélectionnez le type de problème approprié. Pour les problèmes couverts par la garantie de réparation et d’assistance, sélectionnez **Technique**. Pour les demandes d’augmentation de quota, sélectionnez **Limites du service et de l’abonnement (quotas)** .

   ![Sélectionner un type de problème](./media/sql-data-warehouse-get-started-create-support-ticket/select-quota-issue-type.png)  

   > [!NOTE]
   > Le reste de cet article se concentre sur les demandes d’augmentation de quota. Toutefois, vous pouvez également sélectionner **Technique** ici pour les demandes de support pour la résolution de problèmes. Si vous sélectionnez **Technique**, vous êtes invité à fournir un résumé, puis à identifier un type de problème en sélectionnant **Sélectionner un type de problème**. Il se peut que des solutions s’affichent pour vous aider à résoudre votre problème. Si les solutions présentées ne résolvent pas votre problème, sélectionnez **Suivant : Détails** et remplissez le formulaire pour envoyer le ticket de support.

1. Pour les demandes d’augmentation de quota, sélectionnez **Azure Synapse Analytics** pour le **Type de quota**. Ensuite, sélectionnez **Next: Solutions >>** .

   ![Sélectionner un type de quota](./media/sql-data-warehouse-get-started-create-support-ticket/select-quota-type.png)

1. Dans la fenêtre **Détails**, sélectionnez **Entrer les détails** pour entrer des informations supplémentaires.

   ![Lien « Fournir des détails »](./media/sql-data-warehouse-get-started-create-support-ticket/provide-details-link.png)

## <a name="quota-request-types"></a>Types de demande de quota

Quand vous sélectionnez **Entrer les détails**, la fenêtre **Détails du quota** s’affiche. Elle vous permet d’ajouter des informations. Les sections suivantes décrivent les différentes demandes de quota disponibles pour Azure Synapse Analytics.

### <a name="synapse-sql-pool-data-warehouse-units-dwus-per-server"></a>Pool de serveurs SQL Data Warehouse Units (DWU) par serveur

Suivez les étapes ci-après pour demander une augmentation du nombre de DWU par serveur.

1. Sélectionnez le type de quota de **pool Synapse SQL DWU par serveur**.

1. Sélectionnez la **ressource** pour laquelle vous souhaitez appliquer l’augmentation du quota à l’aide de la liste déroulante.

1. Entrez votre nouveau quota dans la section **Quota de la requête**.

1. Sélectionnez **Enregistrer et continuer**.

   ![Détails du quota de DWU](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details-dwus.png)


### <a name="servers-per-subscription"></a>Serveurs par abonnement

Suivez les étapes ci-après pour demander une augmentation du nombre de serveurs par abonnement.

1. Sélectionnez le type de quota **Serveurs SQL par abonnement**.

1. Dans la liste **Emplacement**, sélectionnez la région Azure à utiliser. Le quota s’entend par abonnement dans chaque région.

1. Dans le champ **Demander un quota**, entrez le nombre maximal de serveurs demandé dans cette région.

   ![Détails du quota de serveurs](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details-servers.png)



1. Sélectionnez **Enregistrer et continuer**.

Certains types d’offres ne sont pas disponibles dans toutes les régions. L'erreur suivante peut apparaître :

![Erreur d’accès à une région](./media/sql-data-warehouse-get-started-create-support-ticket/region-access-error.png)

### <a name="enable-subscription-access-to-a-region"></a>Activer l’accès d’un abonnement à une région

Pour activer l’accès à la région pour un abonnement, vous devez effectuer les étapes suivantes :  

1. Sélectionnez le type de quota **Accès à la région du pool Synapse SQL (entrepôt de données)** .

1. Sélectionnez la région en choisissant un **Emplacement** dans la liste déroulante.

1. Indiquez vos exigences en matière de performances de DWU dans la section **Exigences de DWU**.

1. Saisissez votre **Description des besoins métier**. 

1. Sélectionnez **Enregistrer et continuer**.

![Accès à une région](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details-region.png)


### <a name="for-other-quota-requests"></a>Pour toute autre demande de quota

Sélectionnez **Autre demande de quota** dans le menu déroulant pour les autres types de quota :

![Autres détails de quota](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details-whitelisting.png)

## <a name="submit-your-request"></a>Soumettre votre requête

La dernière étape consiste à renseigner les détails restants de votre demande support SQL Database. Ensuite, sélectionnez **Next: Vérifier + créer >>** .

![Consulter les détails de la création](./media/sql-data-warehouse-get-started-create-support-ticket/review-create-details.png)

Vérifiez les détails de la demande, puis cliquez sur **Créer** pour l’envoyer.

![Créer un ticket](./media/sql-data-warehouse-get-started-create-support-ticket/create-ticket.png)

## <a name="monitor-a-support-ticket"></a>Surveiller un ticket de support

Une fois votre demande de support envoyée, l’équipe de support Azure vous contactera. Pour vérifier l’état et les détails de votre demande, sélectionnez **Toutes les demandes de support** dans le tableau de bord.

![Vérification du statut](./media/sql-data-warehouse-get-started-create-support-ticket/monitor-ticket.png)

## <a name="other-resources"></a>Autres ressources

Vous pouvez également contacter la communauté Azure Synapse Analytics sur [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-synapse+or+azure-sql-data-warehouse) ou par le biais de la [page de questions Microsoft Q&A pour Azure Synapse Analytics](https://docs.microsoft.com/answers/topics/azure-synapse-analytics.html).

