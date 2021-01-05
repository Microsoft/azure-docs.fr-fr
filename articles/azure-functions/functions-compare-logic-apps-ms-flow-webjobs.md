---
title: Options de la plateforme d’intégration et d’automatisation dans Azure
description: 'Comparez les services cloud Microsoft optimisés pour les tâches d’intégration : Power Automate, Logic Apps, Functions et WebJobs.'
ms.topic: overview
ms.date: 04/09/2018
ms.custom: mvc
ms.openlocfilehash: 2c17b311811fd2cba46db379fcc766c04dd9d56d
ms.sourcegitcommit: dc68a2c11bae2e9d57310d39fbed76628233fd7f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2020
ms.locfileid: "91403073"
---
# <a name="choose-the-right-integration-and-automation-services-in-azure"></a>Choisir les services d’intégration et d’automatisation appropriés dans Azure

Cet article compare les services de cloud computing Microsoft suivants :

* [Microsoft Power Automate](https://flow.microsoft.com/) (anciennement Microsoft Flow)
* [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/)
* [Azure Functions](https://azure.microsoft.com/services/functions/)
* [Azure App Service WebJobs](../app-service/webjobs-create.md)

Tous ces services peuvent résoudre des problèmes d’intégration et automatiser des processus métier. Ils peuvent tous définir des entrées, des actions, des conditions et des sorties. Vous pouvez exécuter chacun d’eux selon une planification ou à l’aide d’un déclencheur. Les services présentent des avantages qui leur sont propres. Cet article explique les différences entre eux. 

Si vous recherchez une comparaison plus générale entre Azure Functions et d’autres options de calcul Azure, consultez [Critères de sélection d’un service de calcul Azure](/azure/architecture/guide/technology-choices/compute-comparison) et [Choix d’une option de calcul Azure pour les microservices](/azure/architecture/microservices/design/compute-options).

## <a name="compare-microsoft-power-automate-and-azure-logic-apps"></a>Comparer Microsoft Power Automate et Azure Logic Apps

Power Automate et Logic Apps sont des services d’intégration *orientés concepteur* pouvant créer des workflows. Ils s’intègrent à différentes applications SaaS et d’entreprise. 

Power Automate repose sur Logic Apps. Ils partagent le même concepteur de workflows et les mêmes [connecteurs](../connectors/apis-list.md). 

Power Automate permet aux employés de bureau d’effectuer eux-mêmes des intégrations simples (par exemple, un processus d’approbation dans une bibliothèque de documents SharePoint) sans avoir à passer par les développeurs ou le service informatique. Logic Apps permet également d’effectuer des intégrations avancées (par exemple des processus B2B) en respectant les pratiques Azure DevOps et de sécurité requises au niveau de l’entreprise. Il est courant qu’un flux de travail métier se complexifie au fil du temps. Par conséquent, vous pouvez commencer par un flux et le convertir ensuite en application logique selon vos besoins.

Le tableau suivant vous aide à déterminer lequel des services Power Automate et Logic Apps répond le mieux à vos besoins d’intégration :

|  | Power Automate | Logic Apps |
| --- | --- | --- |
| **Utilisateurs** |Employés de bureau, utilisateurs de l’entreprise, administrateurs SharePoint |Intégrateurs et développeurs professionnels, professionnels de l’informatique |
| **Scénarios** |Libre-service |Intégrations avancées |
| **Outil de conception** |Dans le navigateur et application mobile, interface utilisateur uniquement |Dans le navigateur et [Visual Studio](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), [mode Code](../logic-apps/logic-apps-author-definitions.md) disponible |
| **Application Lifecycle Management (ALM)** |Concevoir et tester dans des environnements hors production, et passer en production lorsque vous êtes prêt |Azure DevOps : contrôle de code source, tests, support, automatisation et gestion simplifiée dans [Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md) |
| **Expérience administrateur** |Gérer le suivi des licences, les stratégies de protection contre la perte des données (DLP) et les environnements Power Automate : [Centre d’administration](https://admin.flow.microsoft.com) |Gérer les groupes de ressources, les connexions, les accès et la journalisation : [Azure portal](https://portal.azure.com) |
| **Sécurité** |Journaux d’audit de sécurité Microsoft 365, DLP, [chiffrement au repos](https://wikipedia.org/wiki/Data_at_rest#Encryption) pour les données sensibles |Assurance sécurité d’Azure : [Sécurité Azure](https://www.microsoft.com/en-us/trustcenter/Security/AzureSecurity), [Azure Security Center](https://azure.microsoft.com/services/security-center/), [journaux d’audit](https://azure.microsoft.com/blog/azure-audit-logs-ux-refresh/) |

## <a name="compare-azure-functions-and-azure-logic-apps"></a>Comparer Azure Functions et Azure Logic Apps

Functions et Logic Apps sont des services Azure qui proposent des charges de travail sans serveur. Azure Functions est un service de calcul serverless. Azure Logic Apps fournit des workflows serverless. Les deux services peuvent créer des *orchestrations* complexes. Une orchestration est une collection de fonctions ou d’étapes, appelées *actions* dans Logic Apps, qui sont exécutées pour accomplir une tâche complexe. Par exemple, pour traiter un lot de commandes, vous pourrez exécuter plusieurs instances d’une fonction en parallèle, attendre que toutes les instances se terminent, puis exécuter une fonction qui calcule un résultat de l’agrégat.

Pour Azure Functions, vous développez des orchestrations en écrivant du code et en utilisant l’[extension Durable Functions](durable/durable-functions-overview.md). Pour Logic Apps, vous créez des orchestrations en utilisant une interface graphique utilisateur ou en modifiant des fichiers config.

Vous pouvez mélanger et associer les services lorsque vous créez une orchestration, en appelant des fonctions à partir d’applications logiques et inversement. Vous pouvez opter pour l’un ou l’autre des services pour créer chacune de vos orchestrations en fonction des capacités de ceux-ci ou de vos préférences. Le tableau suivant liste certaines des principales différences entre ces services :

|  | Fonctions durables | Logic Apps |
| --- | --- | --- |
| **Développement** | Orienté code (impératif) | Orienté concepteur (déclaratif) |
| **Connectivité** | [Environ une douzaine de types de liaison intégrés](functions-triggers-bindings.md#supported-bindings), écriture de code pour les liaisons personnalisées | [Grande collection de connecteurs](../connectors/apis-list.md), [Enterprise Integration Pack pour les scénarios B2B](../logic-apps/logic-apps-enterprise-integration-overview.md), [intégration de connecteurs personnalisés](../logic-apps/custom-connector-overview.md) |
| **Actions** | Chaque activité est une fonction Azure ; écriture de code pour les fonctions de l’activité |[Grande collection d’actions prédéfinies](../logic-apps/logic-apps-workflow-actions-triggers.md)|
| **Surveillance** | [Azure Application Insights](../azure-monitor/app/app-insights-overview.md) | [Portail Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md), [journaux d’activité Azure Monitor](../logic-apps/monitor-logic-apps.md)|
| **Gestion** | [API REST](durable/durable-functions-http-api.md), [Visual Studio](/visualstudio/azure/vs-azure-tools-resources-managing-with-cloud-explorer?view=vs-2019) | [Portail Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md), [API REST](/rest/api/logic/), [PowerShell](/powershell/module/az.logicapp), [Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md) |
| **Contexte d’exécution** | Exécution possible [en local](functions-runtime-overview.md) ou dans le cloud | Exécution uniquement dans le cloud|

<a name="function"></a>

## <a name="compare-functions-and-webjobs"></a>Comparer Functions et WebJobs

Comme Azure Functions, Azure App Service WebJobs avec le Kit de développement logiciel (SDK) WebJobs est un service d’intégration *orienté code* conçu pour les développeurs. Les deux reposent sur [Azure App Service](../app-service/overview.md) et prennent en charge des fonctionnalités telles que [l’intégration du contrôle de code source](../app-service/deploy-continuous-deployment.md), [l’authentification](../app-service/overview-authentication-authorization.md) et la [surveillance avec l’intégration Application Insights](functions-monitoring.md).

### <a name="webjobs-and-the-webjobs-sdk"></a>WebJobs et Kit de développement logiciel (SDK) WebJobs

Vous pouvez utiliser la fonctionnalité *WebJobs* d’App Service pour exécuter un script ou du code dans le contexte d’une application web App Service. Le *Kit de développement logiciel (SDK) WebJobs* est une infrastructure conçue pour les WebJobs qui simplifie le code que vous écrivez pour répondre aux événements dans les services Azure. Par exemple, vous pouvez répondre à la création d’un objet blob d’image dans Stockage Azure en créant une image miniature. Le Kit de développement logiciel (SDK) WebJobs s’exécute comme une application de console .NET, que vous pouvez déployer sur un WebJob. 

Les WebJobs et le Kit de développement logiciel (SDK) WebJobs fonctionnent mieux ensemble, mais vous pouvez utiliser des WebJobs sans Kit de développement logiciel (SDK) WebJobs et vice versa. Un WebJob peut exécuter n’importe quel programme ou script s’exécutant dans le bac à sable App Service. Une application de console de Kit de développement logiciel (SDK) WebJobs peut s’exécuter là où les applications de console s’exécutent, tels que des serveurs locaux.

### <a name="comparison-table"></a>Tableau de comparaison

Azure Functions repose sur le Kit de développement logiciel (SDK) WebJobs, donc il partage beaucoup de déclencheurs d’événements et de connexions à d’autres services Azure. Voici quelques facteurs à prendre en compte quand vous devez choisir entre Azure Functions et WebJobs avec le SDK WebJobs :

|  | Fonctions | WebJobs avec le Kit de développement logiciel (SDK) WebJobs |
| --- | --- | --- |
|**[Modèle d’application serverless](https://azure.microsoft.com/solutions/serverless/) avec [mise à l’échelle automatique](functions-scale.md#how-the-consumption-and-premium-plans-work)**|✔||
|**[Développement et test dans un navigateur](functions-create-first-azure-function.md)** |✔||
|**[Paiement à l’utilisation](functions-scale.md#consumption-plan)**|✔||
|**[Intégration à Logic Apps](functions-twitter-email.md)**|✔||
| **Événements déclencheurs** |[Minuteur](functions-bindings-timer.md)<br>[Objets blob et files d’attente Stockage Azure](functions-bindings-storage-blob.md)<br>[Files d’attente et rubriques Azure Service Bus](functions-bindings-service-bus.md)<br>[Azure Cosmos DB](functions-bindings-cosmosdb.md)<br>[Azure Event Hubs](functions-bindings-event-hubs.md)<br>[HTTP/WebHook (GitHub, Slack)](functions-bindings-http-webhook.md)<br>[Azure Event Grid](functions-bindings-event-grid.md)|[Minuteur](functions-bindings-timer.md)<br>[Objets blob et files d’attente Stockage Azure](functions-bindings-storage-blob.md)<br>[Files d’attente et rubriques Azure Service Bus](functions-bindings-service-bus.md)<br>[Azure Cosmos DB](functions-bindings-cosmosdb.md)<br>[Azure Event Hubs](functions-bindings-event-hubs.md)<br>[Système de fichiers](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Files/FileTriggerAttribute.cs)|
| **Langues prises en charge**  |C#<br>F#<br>JavaScript<br>Java<br>Python<br>PowerShell |C#<sup>1</sup>|
|**Gestionnaires de package**|NPM et NuGet|NuGet<sup>2</sup>|

<sup>1</sup> WebJobs (sans le SDK WebJobs) prend en charge C#, Java, JavaScript, Bash, .cmd, .bat, PowerShell, PHP, TypeScript et Python, entre autres. Cette liste n’est pas exhaustive. Un WebJob peut exécuter n’importe quel programme ou script pouvant s’exécuter dans le bac à sable App Service.

<sup>2</sup> WebJobs (sans le SDK WebJobs) prend en charge NPM et NuGet.

### <a name="summary"></a>Résumé

Azure Functions apporte aux développeurs un gain de productivité supérieur par rapport à Azure App Service WebJobs. Il leur offre également davantage d’options pour les langages de programmation, les environnements de développement, l’intégration des services Azure et la tarification. Pour la plupart des scénarios, Azure Functions est le meilleur choix.

Voici deux scénarios pour lesquels WebJobs peut être le meilleur choix :

* Vous avez besoin de davantage de contrôle sur le code qui écoute les événements, l’objet `JobHost`. Azure Functions offre un nombre limité de méthodes de personnalisation du comportement `JobHost` dans le fichier [host.json](functions-host-json.md). Parfois, vous devez effectuer les opérations qui ne peuvent pas être spécifiées par une chaîne dans un fichier JSON. Par exemple, seul le Kit de développement logiciel (SDK) WebJobs vous permet de configurer une stratégie de nouvelles tentatives personnalisée pour Stockage Azure.
* Vous avez une application App Service pour laquelle vous souhaitez exécuter des extraits de code, et vous voulez les gérer ensemble dans le même environnement Azure DevOps.

Pour d’autres scénarios où vous souhaitez exécuter des extraits de code pour l’intégration de services Azure ou tiers, préférez Azure Functions à WebJobs avec le Kit de développement logiciel (SDK) WebJobs.

<a name="together"></a>

## <a name="power-automate-logic-apps-functions-and-webjobs-together"></a>Power Automate, Logic Apps, Functions et WebJobs ensemble

Vous n’êtes pas obligé de choisir un seul de ces services. Vous pouvez intégrer plusieurs services entre eux mais aussi des services externes.

Un flux peut appeler une application logique. Une application logique peut appeler une fonction, et une fonction peut appeler une application logique. Consultez par exemple [Créer une fonction qui s’intègre avec Azure Logic Apps](functions-twitter-email.md).

L’intégration entre Power Automate, Logic Apps et Functions continue de s’améliorer au fil du temps. Vous pouvez créer quelque chose dans un service et l’utiliser dans les autres services.

Vous pouvez obtenir des informations supplémentaires sur les services d’intégration par le biais des liens suivants :

* [Leveraging Azure Functions &amp; Azure App Service for integration scenarios (Tirer parti d’Azure Functions et Azure App Service pour les scénarios d’intégration), par Christopher Anderson](https://www.biztalk360.com/integrate-2016-resources/leveraging-azure-functions-azure-app-service-integration-scenarios/)
* [Integrations Made Simple (Les intégrations simplifiées), par Charles Lamanna](https://www.biztalk360.com/integrate-2016-resources/integrations-made-simple/)
* [Webcast en direct sur Logic Apps](https://aka.ms/logicappslive)
* [Questions fréquentes (FAQ) concernant Power Automate](/power-automate/frequently-asked-questions)

## <a name="next-steps"></a>Étapes suivantes

Commencez en créant votre premier flux, application logique ou application de fonction. Sélectionnez l’un des liens suivants :

* [Bien démarrer avec Power Automate](/power-automate/getting-started)
* [Créer une application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* [Créer votre première fonction Azure](functions-create-first-azure-function.md)