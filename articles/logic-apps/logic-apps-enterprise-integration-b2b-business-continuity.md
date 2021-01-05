---
title: Reprise d’activité pour des comptes d’intégration
description: Configurer vos comptes d’intégration et artefacts B2B avec la récupération d’urgence entre régions dans Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 04/10/2017
ms.openlocfilehash: 1e006dd690e9c008afada8a490da6c3238bc0791
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91565409"
---
# <a name="set-up-cross-region-disaster-recovery-for-integration-accounts-in-azure-logic-apps"></a>Configurer la reprise d’activité entre des régions pour des comptes d’intégration dans Azure Logic Apps

Les charges de travail B2B impliquent des transactions monétaires telles que des commandes et des factures. Pour les entreprises, une récupération rapide est essentielle pour respecter les SLA de niveau entreprise convenus avec leurs partenaires lors d’un sinistre. Cet article montre comment créer un plan de continuité des activités pour les charges de travail B2B. 

* Préparation à la récupération d’urgence 
* Basculer vers la région secondaire lors d’un sinistre 
* Revenir à la région primaire après sinistre

## <a name="disaster-recovery-readiness"></a>Préparation à la récupération d’urgence  

1. Identifiez une région secondaire et créer un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) dans cette région.

2. Ajoutez des partenaires, des schémas et des contrats pour les flux de messages requis où l’état d’exécution doit être répliqué sur le compte d’intégration de la région secondaire.

   > [!TIP]
   > Vérifiez la cohérence entre les régions dans la convention d’affectation de noms pour les artefacts de compte d’intégration. 

3. Pour extraire l’état d’exécution de la région primaire, créez une application logique dans la région secondaire. 

   L’application logique doit disposer d’un *déclencheur* et d’une *action*. 
   Le déclencheur doit se connecter au compte d’intégration de la région primaire tandis que l’action doit se connecter au compte d’intégration de la région secondaire. 
   Selon l’intervalle de temps, le déclencheur interroge la table d’état d’exécution de la région primaire et extrait les nouveaux enregistrements, le cas échéant. L’action les met à jour dans le compte d’intégration de la région secondaire. 
   Cela permet d’obtenir un état de runtime incrémentiel depuis la région primaire vers la région secondaire.

4. La continuité des activités dans le compte d’intégration Logic Apps est conçue pour prendre en charge des protocoles basés sur B2B : X12, AS2 et EDIFACT. Pour une procédure détaillée, sélectionnez les liens correspondants.

5. Il est recommandé de déployer toutes les ressources de la région primaire dans une région secondaire. 

   Les ressources de la région primaire incluent Azure SQL Database ou Azure Cosmos DB, Azure Service Bus/Azure Event Hubs (utilisés pour la messagerie), la gestion des API Azure et la fonctionnalité Logic Apps d’Azure App Service.   

6. Établissez une connexion entre la région primaire et la région secondaire. Pour extraire l’état d’exécution d’une région primaire, créez une application logique dans une région secondaire. 

   L’application logique doit disposer d’un déclencheur et d’une action. 
   Le déclencheur doit se connecter au compte d’intégration d’une région primaire. 
   Le déclencheur doit se connecter au compte d’intégration d’une région secondaire. 
   Selon l’intervalle de temps, le déclencheur interroge la table d’état d’exécution de la région primaire et extrait les nouveaux enregistrements, le cas échéant. 
   L’action les met à jour dans le compte d’intégration de la région secondaire. 
   Ce processus permet d’obtenir un état de runtime incrémentiel depuis la région primaire vers la région secondaire.

La continuité des activités dans le compte d’intégration Logic Apps prend en charge les protocoles B2B X12, AS2 et EDIFACT. Pour obtenir des instructions détaillées sur l’utilisation de X12 et AS2, consultez les sections de cet articles consacrées à [X12](../logic-apps/logic-apps-enterprise-integration-b2b-business-continuity.md#x12) et [AS2](../logic-apps/logic-apps-enterprise-integration-b2b-business-continuity.md#as2).

## <a name="fail-over-to-a-secondary-region-during-a-disaster-event"></a>Basculer vers une région secondaire lors d’un sinistre

Lors d’un sinistre, lorsque la région primaire n’est pas disponible pour la continuité des activités, dirigez le trafic vers la région secondaire. Une région secondaire permet à une entreprise de récupérer rapidement ses fonctions de manière à respecter ses objectifs de point de récupération et de temps de récupération convenus avec ses partenaires. Elle contribue en outre à réduire les efforts nécessaires pour basculer d’une région à l’autre. 

Il existe une latence attendue lorsque des numéros de contrôle sont copiés depuis une région primaire vers une région secondaire. Pour éviter d’envoyer des numéros de contrôle générés en double aux partenaires lors d’un sinistre, il est recommandé d’augmenter les numéros de contrôle dans les accords de région secondaire au moyen [d’applets de commande PowerShell](/powershell/module/azurerm.logicapp/set-azurermintegrationaccountgeneratedicn?view=azurermps-6.13.0).

## <a name="fall-back-to-a-primary-region-post-disaster-event"></a>Revenir à une région primaire après un sinistre

Pour revenir à une région primaire lorsqu’elle est de nouveau disponible, procédez comme suit :

1. Cessez d’accepter les messages des partenaires dans la région secondaire.  

2. Augmentez les numéros de contrôle générés pour tous les accords de région primaire au moyen [d’applets de commande PowerShell](/powershell/module/azurerm.logicapp/set-azurermintegrationaccountgeneratedicn?view=azurermps-6.13.0).  

3. Dirigez le trafic de la région secondaire vers la région primaire.

4. Vérifiez que l’application logique créée dans la région secondaire pour extraire l’état d’exécution de la région primaire est activée.

## <a name="x12"></a>X 12 

La continuité des activités pour les documents EDI X12 documents repose sur les numéros de contrôle :

> [!TIP]
> Vous pouvez également utiliser le [modèle de démarrage rapide X12](https://azure.microsoft.com/resources/templates/201-logic-app-b2b-disaster-recovery-replication/) pour créer des applications logiques. La création d’un compte d’intégration primaire et d’un compte d’intégration secondaire est nécessaire pour utiliser le modèle. Le modèle permet de créer deux applications logiques, une pour les numéros de contrôle reçus et l’autre pour les numéros de contrôle générés. Les déclencheurs et actions respectifs sont créés dans les applications logiques, ce qui permet de connecter le déclencheur connecté au compte d’intégration primaire et de connecter l’action au compte d’intégration secondaire.

**Composants requis**

Pour activer la récupération d’urgence pour les messages entrants, sélectionnez les options de vérification de doublons dans les paramètres de réception de l’accord X12.

![Sélectionnez les paramètres de vérification des doublons](./media/logic-apps-enterprise-integration-b2b-business-continuity/dupcheck.png)  

1. Créez une [application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md) dans la région secondaire.    

2. Lancez une recherche sur **X12** et sélectionnez **X12 - Lors de la modification d’un numéro de contrôle**.   

   ![Recherchez X12](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn1.png)

   Le déclencheur vous invite à établir une connexion à un compte d’intégration. 
   Le déclencheur doit être connecté au compte d’intégration d’une région primaire.

3. Entrez un nom de connexion, sélectionnez votre *compte d’intégration de la région primaire* dans la liste et cliquez sur **Créer**.   

   ![Capture d’écran montrant où entrer un nom de connexion et sélectionner votre compte d’intégration de région primaire. ](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn2.png)

4. Le paramètre **DateTime pour démarrer la synchronisation des numéros de contrôle** est facultatif. La **Fréquence** peut être définie sur **Jour**, **Heure**, **Minute** ou **Seconde** avec un intervalle.   

   ![Capture d’écran montrant le paramètre DateTime pour démarrer la synchronisation des numéros de contrôle.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn3.png)

5. Sélectionnez **Nouvelle étape** > **Ajouter une action**.

   ![Capture d’écran montrant le bouton Nouvelle étape et l’option Ajouter une action.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn4.png)

6. Lancez une recherche sur **X12** et sélectionnez **X12 - Ajouter ou mettre à jour des numéros de contrôle**.   

   ![Ajoutez ou mettez à jour les numéros de contrôle](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn5.png)

7. Pour connecter une action à un compte d’intégration d’une région secondaire, sélectionnez **Modifier la connexion** > **Ajouter une nouvelle connexion** pour obtenir la liste des comptes d’intégration disponibles. Entrez un nom de connexion, sélectionnez votre *compte d’intégration de la région secondaire* dans la liste et cliquez sur **Créer**. 

   ![Capture d’écran montrant où ajouter un nom de compte d’intégration de région secondaire.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn6.png)

8. Basculez vers les entrées brutes en cliquant sur l’icône située dans le coin supérieur droit.

   ![Capture d’écran montrant l’icône à sélectionner pour basculer vers les entrées brutes.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12rawinputs.png)

9. Sélectionnez le corps à partir du sélecteur de contenu dynamique et enregistrez l’application logique.

   ![Champs de contenu dynamique](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn7.png)

   Selon l’intervalle de temps, le déclencheur interroge la table des numéros de contrôle de la région primaire et extrait les nouveaux enregistrements. 
   L’action les met à jour dans le compte d’intégration de la région secondaire. 
   S’il n’y a aucune mise à jour, le déclencheur affiche l’état **Ignoré**.   

   ![Table des numéros de contrôle](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12recevicedcn8.png)

En fonction de l’intervalle de temps, l’état d’exécution incrémentiel est dupliqué d’une région primaire à une région secondaire. Lors d’un sinistre, lorsque la région primaire n’est pas disponible, dirigez le trafic vers la région secondaire pour la continuité des activités. 

## <a name="edifact"></a>EDIFACT 

La continuité des activités pour les documents EDI EDIFACT repose sur les numéros de contrôle.

**Composants requis**

Pour activer la récupération d’urgence pour les messages entrants, sélectionnez les options de vérification de doublons dans les paramètres de réception de l’accord EDIFACT.

![Capture d’écran montrant les options de vérification de doublons dans les paramètres de réception de l’accord EDIFACT.](./media/logic-apps-enterprise-integration-b2b-business-continuity/edifactdupcheck.png)  

1. Créez une [application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md) dans la région secondaire.    

2. Lancez une recherche sur **EDIFACT** et sélectionnez **EDIFACT - Lors de la modification d’un numéro de contrôle**.

   ![Recherchez EDIFACT](./media/logic-apps-enterprise-integration-b2b-business-continuity/edifactcn1.png)

   Le déclencheur vous invite à établir une connexion à un compte d’intégration. 
   Le déclencheur doit être connecté au compte d’intégration d’une région primaire. 

3. Entrez un nom de connexion, sélectionnez votre *compte d’intégration de la région primaire* dans la liste et cliquez sur **Créer**.    

   ![Nom du compte d’intégration de la région primaire](./media/logic-apps-enterprise-integration-b2b-business-continuity/X12CN2.png)

4. Le paramètre **DateTime pour démarrer la synchronisation des numéros de contrôle** est facultatif. La **Fréquence** peut être définie sur **Jour**, **Heure**, **Minute** ou **Seconde** avec un intervalle.    

   ![Capture d’écran montrant les paramètres DateTime et Fréquence.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn3.png)

6. Sélectionnez **Nouvelle étape** > **Ajouter une action**.    

   ![Capture d’écran montrant où sélectionner Ajouter une action.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn4.png)

7. Lancez une recherche sur **EDIFACT** et sélectionnez **EDIFACT - Ajouter ou mettre à jour des numéros de contrôle**.   

   ![Capture d’écran montrant où ajouter ou mettre à jour des numéros de contrôle.](./media/logic-apps-enterprise-integration-b2b-business-continuity/EdifactChooseAction.png)

8. Pour connecter une action à un compte d’intégration d’une région secondaire, sélectionnez **Modifier la connexion** > **Ajouter une nouvelle connexion** pour obtenir la liste des comptes d’intégration disponibles. Entrez un nom de connexion, sélectionnez votre *compte d’intégration de la région secondaire* dans la liste et cliquez sur **Créer**.

   ![Créez un nom de compte d’intégration de région secondaire.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn6.png)

9. Basculez vers les entrées brutes en cliquant sur l’icône située dans le coin supérieur droit.

   ![Capture d’écran qui met en surbrillance l’icône à sélectionner lorsque vous souhaitez basculer vers les entrées brutes.](./media/logic-apps-enterprise-integration-b2b-business-continuity/Edifactrawinputs.png)

10. Sélectionnez le corps à partir du sélecteur de contenu dynamique et enregistrez l’application logique.   

   ![Capture d’écran montrant le sélecteur de contenu dynamique dans lequel vous pouvez sélectionner le corps.](./media/logic-apps-enterprise-integration-b2b-business-continuity/X12CN7.png)

   Selon l’intervalle de temps, le déclencheur interroge la table des numéros de contrôle de la région primaire et extrait les nouveaux enregistrements.
   L’action les met à jour dans le compte d’intégration de la région secondaire. 
   S’il n’y a aucune mise à jour, le déclencheur affiche l’état **Ignoré**.

   ![Table des numéros de contrôle](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12recevicedcn8.png)

En fonction de l’intervalle de temps, l’état d’exécution incrémentiel est dupliqué d’une région primaire à une région secondaire. Lors d’un sinistre, lorsque la région primaire n’est pas disponible, dirigez le trafic vers la région secondaire pour la continuité des activités. 

## <a name="as2"></a>AS2 

La continuité des activités pour les documents qui utilisent le protocole AS2 est basée sur l’ID de message et la valeur MIC.

> [!TIP]
> Vous pouvez également utiliser le [modèle de démarrage rapide AS2](https://github.com/Azure/azure-quickstart-templates/pull/3302) pour créer des applications logiques. La création d’un compte d’intégration primaire et d’un compte d’intégration secondaire est nécessaire pour utiliser le modèle. Ce modèle permet de créer une application logique qui comporte un déclencheur et une action. L’application logique crée une connexion entre le déclencheur et un compte d’intégration primaire et entre l’action et un compte d’intégration secondaire.

1. Créez une [application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md) dans la région secondaire.  

2. Recherchez **AS2** et sélectionnez **AS2 - Lorsqu’une valeur MIC est créée**.   

   ![Recherchez AS2](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid1.png)

   Un déclencheur vous invite à établir une connexion à un compte d’intégration. 
   Le déclencheur doit être connecté au compte d’intégration d’une région primaire. 
   
3. Entrez un nom de connexion, sélectionnez votre *compte d’intégration de la région primaire* dans la liste et cliquez sur **Créer**.

   ![Capture d’écran montrant où entrer un nom de connexion lorsqu’une valeur MIC est créée.](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid2.png)

4. Le paramètre **DateTime de démarrage de la synchronisation des valeurs MIC** est facultatif. La **Fréquence** peut être définie sur **Jour**, **Heure**, **Minute** ou **Seconde** avec un intervalle.   

   ![Date/heure et fréquence](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid3.png)

5. Sélectionnez **Nouvelle étape** > **Ajouter une action**.  

   ![Nouvelle étape, puis Ajouter une action](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid4.png)

6. Recherchez **AS2** et sélectionnez **AS2 - Ajouter ou mettre à jour les contenus MIC**.  

   ![Ajout ou mise à jour d’un MIC](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid5.png)

7. Pour connecter une action à un compte d’intégration secondaire, sélectionnez **Modifier la connexion** > **Ajouter une nouvelle connexion** pour obtenir la liste des comptes d’intégration disponibles. Entrez un nom de connexion, sélectionnez votre *compte d’intégration de la région secondaire* dans la liste et cliquez sur **Créer**.

   ![Nom du compte d’intégration de la région secondaire](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid6.png)

8. Basculez vers les entrées brutes en cliquant sur l’icône située dans le coin supérieur droit.

   ![Basculez vers des entrées brutes](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2rawinputs.png)

9. Sélectionnez le corps à partir du sélecteur de contenu dynamique et enregistrez l’application logique.   

   ![Contenu dynamique](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid7.png)

   Selon l’intervalle de temps, le déclencheur interroge la table de la région primaire et extrait les nouveaux enregistrements. L’action les met à jour dans le compte d’intégration de la région secondaire. 
   S’il n’y a aucune mise à jour, le déclencheur affiche l’état **Ignoré**.  

   ![Table de la région primaire](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid8.png)

En fonction de l’intervalle de temps, l’état d’exécution incrémentiel est dupliqué de la région primaire à la région secondaire. Lors d’un sinistre, lorsque la région primaire n’est pas disponible, dirigez le trafic vers la région secondaire pour la continuité des activités. 

## <a name="next-steps"></a>Étapes suivantes

[Superviser les messages B2B avec les journaux d’activité Azure Monitor](../logic-apps/monitor-b2b-messages-log-analytics.md)
