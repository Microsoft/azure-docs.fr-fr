---
title: Exemples et scénarios courants
description: Trouver des exemples, des scénarios courants, des didacticiels et des procédures pas à pas courants relatifs à Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 02/28/2020
ms.openlocfilehash: 5f7afe044ab4f782f2028598c2c56e75edf95860
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92317374"
---
# <a name="common-scenarios-examples-tutorials-and-walkthroughs-for-azure-logic-apps"></a>Scénarios, exemples, didacticiels et procédures pas à pas courants relatifs à Azure Logic Apps

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) vous permet d’orchestrer et d’intégrer différents services en fournissant [des centaines de connecteurs prêts à l’emploi](../connectors/apis-list.md), des connecteurs SQL Server ou SAP à Azure Cognitive Services. Le service Logic Apps est un service « sans serveur ». Ainsi, vous n’avez pas à vous soucier de la mise à l’échelle ni des instances. Il vous suffit de définir le workflow avec un déclencheur et les actions exécutées par ce workflow. La plateforme sous-jacente gère la mise à l’échelle, la disponibilité et les performances. Le service Logic Apps se révèle d’une grande utilité pour les cas d’usage et les scénarios impliquant la coordination de plusieurs actions dans plusieurs systèmes et services.

Pour vous aider à en savoir plus sur les capacités et les modèles pris en charge par Azure Logic Apps, cet article décrit des points de départ courants, des exemples et des scénarios.

## <a name="common-starting-points-for-logic-app-workflows"></a>Points de départ courants pour les workflows d’application logique

Chaque application logique commence avec un seul et unique [*déclencheur*](../logic-apps/logic-apps-overview.md#logic-app-concepts), qui démarre votre workflow d’application logique et transmet des données. Certains connecteurs fournissent des déclencheurs, qui présentent les types suivants :

* *Déclencheurs d'interrogation* : vérifient régulièrement l'existence de nouvelles données dans un point de terminaison de service. Lorsque de nouvelles données existent, le déclencheur crée et exécute une nouvelle instance de workflow en utilisant ces données en guise d’entrée.

* *Déclencheurs d'émission* : écoutent les données au niveau d'un point de terminaison de service et attendent qu'un événement spécifique se produise. Lorsque cet événement se produit, le déclencheur s’active aussitôt en créant et exécutant une instance de workflow qui utilise toutes les données disponibles en tant qu’entrée.

Voici des exemples qui décrivent les déclencheurs couramment utilisés :

* Déclencheurs d’*interrogation* :

  * Le [déclencheur **Périodicité**](../connectors/connectors-native-recurrence.md) vous permet de définir les date et heure de début et la périodicité du déclenchement de votre application logique. Par exemple, vous pouvez sélectionner les jours de la semaine et les heures de la journée auxquels votre application logique doit se déclencher. Pour plus d’informations, consultez les rubriques suivantes :<p>

    * [Créer et exécuter des tâches, processus et workflows automatisés récurrents avec Azure Logic Apps](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md)
    * [Tutoriel : Créer des workflows automatisés périodiques basés sur la planification à l’aide d’Azure Logic Apps](../logic-apps/tutorial-build-schedule-recurring-logic-app-workflow.md)

  * Le déclencheur **Lors de la réception d’un e-mail** permet à votre application logique de vérifier l’existence de nouveaux e-mails à partir de n’importe quel fournisseur de messagerie pris en charge par Logic Apps, par exemple, [Office 365 Outlook](../connectors/connectors-create-api-office365-outlook.md), [Gmail](/connectors/gmail/), [Outlook.com](/connectors/outlook/), etc.

    > [!IMPORTANT]
    > Si vous souhaitez utiliser le connecteur Gmail, seuls les comptes professionnels G-Suite peuvent utiliser ce connecteur sans restriction dans les applications logiques. Si vous disposez d’un compte de consommateur Gmail, vous pouvez utiliser ce connecteur uniquement avec certains services approuvés par Google, ou vous pouvez [créer une application cliente Google pour servir lors de l’authentification avec votre connecteur Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Pour plus d’informations, consultez [Stratégies de confidentialité et de sécurité des données pour les connecteurs Google dans Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

    Pour plus d’informations, consultez les rubriques suivantes :<p>

    * [Tutoriel : Créer des workflows automatisés basés sur l’approbation à l’aide d’Azure Logic Apps](../logic-apps/tutorial-process-mailing-list-subscriptions-workflow.md)
    * [Tutoriel : Automatiser les tâches de traitement des e-mails avec Azure Logic Apps, Azure Functions et Stockage Azure](../logic-apps/tutorial-process-email-attachments-workflow.md)

  * Le [déclencheur **HTTP**](../connectors/connectors-native-http.md) peut appeler un point de terminaison de service sur HTTP ou HTTPS. Pour plus d’informations, consultez [Appeler, déclencher ou imbriquer des workflows avec des points de terminaison HTTP](../logic-apps/logic-apps-http-endpoint.md).

* Déclencheurs d’*émission* :

  * Le [déclencheur de **requête**](../connectors/connectors-native-reqres.md) peut recevoir des requêtes HTTPS entrantes.

  * Le [déclencheur **HTTP Webhook**](../connectors/connectors-native-webhook.md) s’abonne au point de terminaison d’un service en inscrivant une *URL de rappel* auprès de ce service. De cette façon, le service peut tout simplement informer le déclencheur lorsque l’événement spécifié se produit, ce qui évite au déclencheur d’avoir à interroger le service.

Lorsque l’événement spécifié se produit, le déclencheur s’active, ce qui crée une instance de workflow d’application logique et exécute les actions dans le workflow. Vous pouvez accéder à toutes les données du déclencheur dans l’ensemble du workflow. Par exemple, le déclencheur **En cas de nouveau tweet** sur Twitter transmet le contenu du tweet dans l’exécution de l’application logique. Pour commencer à utiliser Azure Logic Apps, reportez-vous aux rubriques de démarrage rapide suivantes :

* [Démarrage rapide : Créer votre premier workflow automatisé avec Azure Logic Apps – Portail Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* [Démarrage rapide : Créer des tâches, des processus et des workflows automatisés avec Azure Logic Apps – Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)
* [Démarrage rapide : Créer et gérer des workflows automatisés d’application logique à l’aide de Visual Studio Code](../logic-apps/quickstart-create-logic-apps-visual-studio-code.md)

## <a name="control-flow-and-error-handling-capabilities"></a>Capacités de flux de contrôle et de gestion des erreurs

Les applications logiques incluent de puissantes capacités de flux de contrôle avancé, comme des conditions, des commutateurs, des boucles et des étendues. Pour garantir la résilience de vos solutions, vous pouvez également implémenter la gestion des erreurs et des exceptions dans vos flux de travail.

* Exécuter des actions différentes en fonction [d’instructions conditionnelles](../logic-apps/logic-apps-control-flow-conditional-statement.md) et [d’instructions switch](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [Répéter les étapes ou traiter les éléments dans des tableaux et collections avec des boucles](../logic-apps/logic-apps-control-flow-loops.md)
* [Regrouper des actions avec des étendues](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
* [Ajouter la gestion des erreurs et des exceptions dans un workflow](../logic-apps/logic-apps-exception-handling.md)
* [Cas d’usage : montre comment un prestataire de soins de santé utilise la gestion des exceptions d'application logique pour les flux de travail HL7 FHIR](../logic-apps/logic-apps-scenario-error-and-exception-handling.md)

## <a name="create-custom-apis-and-connectors"></a>Créer des API et des connecteurs personnalisés

Pour les systèmes et services pour lesquels aucun connecteur n’a été publié, vous pouvez également étendre les applications logiques.

* [Créer des API personnalisées à appeler à partir d’Azure Logic Apps](../logic-apps/logic-apps-create-api-app.md)
* [Rechercher régulièrement les nouvelles données ou les nouveaux événements à l’aide du modèle de déclencheur d’interrogation](../logic-apps/logic-apps-create-api-app.md#polling-triggers)
* [Attendre et écouter les nouvelles données ou les nouveaux événements avec le modèle de déclencheur Webhook](../logic-apps/logic-apps-create-api-app.md#webhook-triggers)
* [Effectuer des tâches longues avec le modèle d’action d’interrogation](../logic-apps/logic-apps-create-api-app.md#async-pattern)
* [Effectuer des tâches longues avec le modèle d’action Webhook](../logic-apps/logic-apps-create-api-app.md#webhook-actions)
* [Connecteurs personnalisés dans Azure Logic Apps](../logic-apps/custom-connector-overview.md)

## <a name="build-business-to-business-b2b-solutions"></a>Créer des solutions interentreprises (B2B)

Pour les solutions d’intégration d’entreprise et une communication fluide entre les organisations, vous pouvez générer des workflows évolutifs automatisés pour ces scénarios en utilisant Enterprise Integration Pack (EIP) avec Azure Logic Apps. Les organisations peuvent échanger des messages électroniques, même si elles utilisent des formats et des protocoles différents. EIP transforme différents formats en un format que les systèmes de votre organisation peuvent traiter et prend en charge les protocoles standard, notamment AS2, X12, EDIFACT et RosettaNet. Pour créer ces solutions, créez un compte d’intégration, qui est une ressource Azure distincte offrant un conteneur sécurisé, évolutif et facile à gérer pour les artefacts que vous définissez et utilisez avec vos workflows d’application logique. Par exemple, les artefacts incluent les partenaires commerciaux, les contrats, les mappages, les schémas, les certificats et les configurations de lots.

* [Vue d’ensemble : Solutions d’intégration d’entreprise B2B avec Azure Logic Apps et Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md)
* [Créer et gérer des comptes d’intégration pour les intégrations d’entreprise B2B dans Azure Logic Apps](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)

## <a name="access-azure-virtual-network-resources"></a>Accéder aux ressources du réseau virtuel Azure

Parfois, vos applications logiques et vos comptes d’intégration doivent accéder à des ressources sécurisées, comme des machines virtuelles et d’autres systèmes ou services, qui se trouvent dans un réseau virtuel Azure. Pour configurer cet accès, vous pouvez créer un environnement de service d’intégration (ISE, integration service environment) dans lequel vous pouvez générer et exécuter vos applications logiques. Un ISE est une instance isolée et privée du service Logic Apps qui utilise des ressources dédiées, comme le stockage, et s’exécute séparément du service Logic Apps multilocataire « mondial ». La séparation entre votre instance privée isolée et l’instance mondiale publique permet également de réduire l’impact que les autres locataires Azure peuvent avoir sur les performances de vos applications, ce que l’on appelle également l’effet « voisins bruyants ».

* [Vue d’ensemble : Accéder à des ressources de réseau virtuel Azure à partir d’Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md)
* [Se connecter à des réseaux virtuels Azure à partir d’Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment.md)

## <a name="deploy-manage-and-monitor-logic-apps"></a>Déployer, gérer et surveiller des applications logiques

Vous pouvez développer et déployer des applications logiques entièrement avec Visual Studio, Azure DevOps ou tout autre outil de génération automatisée et contrôle de code source. Pour prendre en charge le déploiement de flux de travail et de connexions dépendantes d’un modèle de ressource, les applications logiques utilisent des modèles de déploiement de ressources Azure. Les outils Visual Studio génèrent automatiquement ces modèles, que vous pouvez archiver dans le contrôle de code source à des fins de contrôle de version. Pour la notification et les journaux de diagnostic relatifs à l’état d’exécution des flux de travail, Azure Logic Apps fournit également la supervision et des alertes.

### <a name="deploy"></a>Déployer

* [Démarrage rapide : Créer des tâches, des processus et des workflows automatisés avec Azure Logic Apps – Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)
* [Vue d’ensemble : Automatiser le déploiement d’applications logiques](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md)
* [Créer des modèles Azure Resource Manager afin d’automatiser le déploiement pour le service Azure Logic Apps](../logic-apps/logic-apps-create-azure-resource-manager-templates.md)
* [Déployer des modèles Azure Resource Manager pour Azure Logic Apps](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md)
* [Exemple : Se connecter à des files d’attente Azure Service Bus à partir d’Azure Logic Apps et déployer avec Azure Pipelines dans Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-azure-service-bus-queues-from-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Exemple : Se connecter à des comptes Stockage Azure à partir d'Azure Logic Apps et déployer avec Azure Pipelines dans Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-azure-storage-accounts-from-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Exemple : Configurer une action Function App pour Azure Logic Apps et déployer avec Azure Pipelines dans Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/set-up-an-azure-function-app-action-for-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Exemple : Se connecter à un compte d'intégration à partir d'Azure Logic Apps et déployer avec Azure Pipelines dans Azure DevOps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-an-integration-account-from-azure-logic-apps-and-deploy-by-using-azure-devops-pipelines/)
* [Exemple : Orchestrer Azure Pipelines à l’aide d’Azure Logic Apps](/samples/azure-samples/azure-logic-apps-pipeline-orchestration/azure-devops-orchestration-with-logic-apps/)

### <a name="manage"></a>Gérer

* [Gérer des applications logiques avec Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md)
* [Créer et gérer des comptes d’intégration pour les intégrations d’entreprise B2B](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)
* [Gérer un environnement de service d’intégration dans Azure Logic Apps](../logic-apps/ise-manage-integration-service-environment.md)

### <a name="monitor"></a>Superviser

* [Surveiller l’état d’exécution, examiner l’historique des déclencheurs et configurer des alertes pour Azure Logic Apps](../logic-apps/monitor-logic-apps.md)
* [Configurer les journaux d’activité Azure Monitor et collecter des données de diagnostic pour Azure Logic Apps](../logic-apps/monitor-logic-apps-log-analytics.md)
* [Configurer les journaux d’activité Azure Monitor et collecter des données de diagnostic pour des messages B2B dans Azure Logic Apps](../logic-apps/monitor-b2b-messages-log-analytics.md)
* [Afficher et créer des requêtes de surveillance et de suivi dans des journaux d’activité Azure Monitor pour Azure Logic Apps](../logic-apps/create-monitoring-tracking-queries.md)

## <a name="handle-content-types-conversions-and-transformations"></a>Gérer les types de contenu, les conversions et les transformations

Vous pouvez accéder à plusieurs types de contenu, les convertir et les transformer à l’aide des nombreuses fonctions du [langage de définition de flux de travail](./logic-apps-workflow-definition-language.md) d’Azure Logic Apps. Par exemple, vous pouvez effectuer des conversions entre une chaîne, JSON et XML avec les expressions de flux de travail `@json()` et `@xml()`. Le moteur Logic Apps préserve les types de contenu pour prendre en charge le transfert de contenu entre les services sans perte.

* [Gérer les types de contenu dans Azure Logic Apps](../logic-apps/logic-apps-content-type.md), comme `application/`, `application/octet-stream` et `multipart/formdata`
* [Guide de référence sur l’utilisation des fonctions dans les expressions pour Azure Logic Apps et Power Automate](../logic-apps/workflow-definition-language-functions-reference.md)
* [Schéma du langage de définition du flux de travail pour Azure Logic Apps](../logic-apps/logic-apps-workflow-definition-language.md)

## <a name="other-integrations-and-capabilities"></a>Autres intégrations et fonctionnalités

Azure Logic Apps s’intègre à de nombreux services, comme Azure Functions, la Gestion des API Azure, Azure App Service et les points de terminaison HTTP personnalisés (par exemple, REST et SOAP).

* [Appeler Azure Functions à partir d’Azure Logic Apps](../logic-apps/logic-apps-azure-functions.md)
* [Tutoriel : Appeler ou déclencher des applications logiques à l’aide d’Azure Functions et d’Azure Service Bus](../logic-apps/logic-apps-scenario-function-sb-trigger.md)
* [Tutoriel : Créer un tableau de bord des insights client en streaming avec Azure Logic Apps et Azure Functions](../logic-apps/logic-apps-scenario-social-serverless.md)
* [Tutoriel : Créer une fonction qui s'intègre à Azure Logic Apps et à Azure Cognitive Services pour analyser le sentiment lié à la publication Twitter](../azure-functions/functions-twitter-email.md)
* [Tutoriel : Créer un tableau de bord social reposant sur l’intelligence artificielle à l’aide de Logic Apps et Power BI](https://aka.ms/logicappsdemo)
* [Tutoriel : Superviser les modifications d’une machine virtuelle avec Azure Event Grid et Azure Logic Apps](../event-grid/monitor-virtual-machine-changes-event-grid-logic-app.md)
* [Tutoriel : Notifications et surveillance à distance IoT à l'aide d'Azure Logic Apps, avec connexion de votre hub IoT et de votre boîte aux lettres](../iot-hub/iot-hub-monitoring-notifications-with-azure-logic-apps.md)
* [Blog : Appeler des services SOAP à l’aide d’Azure Logic Apps](/archive/blogs/logicapps/using-soap-services-with-logic-apps)

## <a name="end-to-end-scenarios"></a>Scénarios de bout en bout

* [Livre blanc : Intégration de la gestion de bout en bout des dossiers aux services Azure comme Logic Apps](https://aka.ms/enterprise-integration-e2e-case-management-utilities-logic-apps)

## <a name="customer-stories"></a>Témoignages client

Découvrez comment Azure Logic Apps, ainsi que d’autres services Azure et produits Microsoft, ont aidé [ces sociétés](https://aka.ms/logic-apps-customer-stories) à améliorer leur flexibilité et à se concentrer sur leurs activités principales en simplifiant, organisant, automatisant et orchestrant des processus complexes.

## <a name="next-steps"></a>Étapes suivantes

* Découvrir les [connecteurs pour Logic Apps](../connectors/apis-list.md)
* Découvrir des [scénarios d’intégration d’entreprise B2B avec Azure Logic Apps](../logic-apps/logic-apps-enterprise-integration-overview.md)