---
title: Échelle et hébergement dans Azure Functions
description: Découvrez comment choisir entre le plan Consommation et le plan Premium d’Azure Functions.
ms.assetid: 5b63649c-ec7f-4564-b168-e0a74cb7e0f3
ms.topic: conceptual
ms.date: 08/17/2020
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: f41354630f4885a30bd5c036495b216a2cc05599
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96167792"
---
# <a name="azure-functions-scale-and-hosting"></a>Échelle et hébergement dans Azure Functions

Lorsque vous créez une Function App dans Azure, vous devez choisir un plan d’hébergement pour votre application. Trois plans d’hébergement De base sont disponibles pour Azure Functions : [plan Consommation](#consumption-plan), [plan Premium](#premium-plan) et [plan dédié (App Service)](#app-service-plan). Tous les plans d’hébergement sont en disponibilité générale sur les machines virtuelles Linux et Windows.

Le plan d’hébergement que vous choisissez détermine les comportements suivants :

* La mise à l’échelle de votre Function App.
* Les ressources disponibles pour chaque instance de Function App.
* La prise en charge des fonctionnalités avancées, notamment la connectivité du réseau virtuel Azure.

Les plans Consommation et Premium ajoutent automatiquement de la puissance de calcul lors de l’exécution de votre code. Le cas échéant, votre application bénéficie d’un scale-out afin de gérer la charge et d’un scale-in lorsque le code n’est plus en cours d’exécution. En outre, avec le plan Consommation, vous n’avez pas à payer pour des machines virtuelles inactives ni à réserver de la capacité à l’avance.  

Le plan Premium fournit des fonctionnalités supplémentaires, telles que des instances de calcul premium, la possibilité de conserver les instances chaudes indéfiniment et la connectivité de réseau virtuel.

Le plan App Service vous permet d’exploiter l’infrastructure dédiée que vous gérez. La mise à l’échelle de votre application de fonction ne dépend pas des événements, un scale-in jusqu’à zéro est donc impossible. ([Always On](#always-on) doit être activé.)

Pour une comparaison détaillée des différents plans d’hébergement (notamment l’hébergement basé sur Kubernetes), consultez la [section Comparaison des plans d’hébergement](#hosting-plans-comparison).

## <a name="consumption-plan"></a>Plan de consommation

Quand vous utilisez le plan Consommation, les instances de l’hôte Azure Functions sont ajoutées et supprimées de façon dynamique en fonction du nombre d’événements entrants. Ce plan serverless est mis à l’échelle automatiquement, et vous êtes facturé pour les ressources de calcul uniquement lorsque vos fonctions sont exécutées. Dans un plan Consommation, le délai d’attente de l’exécution d’une fonction arrive à expiration après une période configurable.

La facturation est basée sur le nombre d’exécutions, la durée d’exécution et la mémoire utilisée. L’utilisation est agrégée pour toutes les fonctions d’une même application de fonction. Pour plus d’informations, consultez la page [Tarification d’Azure Functions](https://azure.microsoft.com/pricing/details/functions/).

Le plan d’hébergement par défaut (le plan Consommation) présente les avantages suivants :

* Paiement uniquement à l’exécution de vos fonctions
* Effectuer un scale-out automatique même pendant les périodes de charge élevée

Les Function App d’une même région peuvent être affectées au même plan Consommation. Avoir plusieurs applications qui s’exécutent dans le même plan Consommation n’entraîne aucun inconvénient ou impact. L’attribution de plusieurs applications au même plan Consommation n’affecte pas la résilience, la scalabilité ni la fiabilité de chaque application.

Pour en savoir plus sur la façon d’estimer les coûts pour une exécution dans le cadre d’un plan Consommation, consultez [Compréhension des coûts d’un plan Consommation](functions-consumption-costs.md).

## <a name="premium-plan"></a><a name="premium-plan"></a>Plan Premium

Quand vous utilisez le plan Premium, les instances de l’hôte Azure Functions sont ajoutées et supprimées en fonction du nombre d’événements entrants, comme avec le plan Consommation.  Le plan Premium prend en charge les fonctionnalités suivantes :

* Les instances perpétuellement chaudes permettent d’éviter les démarrages à froid
* Connectivité des réseaux virtuels
* Durée d’exécution illimitée (60 minutes garanties)
* Tailles d’instance premium (instances à un, deux et quatre cœurs)
* Tarification plus prévisible
* Allocation d’application à haute densité pour les plans avec plusieurs Function App

Pour en savoir plus sur la création d’une application de fonction dans un plan Premium, consultez le [Plan Premium Azure Functions](functions-premium-plan.md).

Au lieu d’une facturation en fonction des exécutions et de la mémoire consommées, la facturation du plan Premium se base sur le nombre de cœurs-seconde et la mémoire alloués sur les instances.  Il n’y a pas de frais d’exécution avec le plan Premium. Au moins une instance doit être allouée en permanence par plan. Il en résulte un coût mensuel minimum par plan actif, que la fonction soit active ou inactive. Gardez à l’esprit que toutes les applications de fonction d’un plan Premium partagent les instances allouées.

Envisagez le plan Premium d’Azure Functions dans les situations suivantes :

* Vos applications de fonction s’exécutent en continu ou presque.
* Vous disposez d’un grand nombre d’exécutions de petite taille et avez une facture d’exécution élevée, mais également une facture de Go par seconde faible dans le plan Consommation.
* Vous avez besoin de plus d’options de mémoire ou de processeur que celles qui sont proposées dans le plan Consommation.
* Votre code exige une [durée d’exécution supérieure à celle qui est autorisée](#timeout) dans le plan Consommation.
* Vous avez besoin de fonctionnalités qui sont uniquement disponibles dans un plan Premium, notamment la connectivité de réseau virtuel. 

## <a name="dedicated-app-service-plan"></a><a name="app-service-plan"></a>Plan (App Service) dédié

Vos Function App sont exécutées sur les mêmes machines virtuelles dédiées que d’autres applications App Service (références De base, Standard, Premium et Isolé).

Envisagez un plan App Service dans les situations suivantes :

* Vous disposez de machines virtuelles existantes, sous-utilisées qui exécutent déjà d’autres instances App Service.
* Vous souhaitez fournir une image personnalisée sur laquelle vos fonctions s’exécuteront.

Le coût des Function App dans un plan App Service est le même que pour d’autres ressources App Service, par exemple les applications web. Pour plus d’informations sur le fonctionnement du plan App Service, consultez l’article [Présentation détaillée des plans d’Azure App Service](../app-service/overview-hosting-plans.md).

À l’aide d’un plan App Service, vous pouvez effectuer manuellement un scale-out en ajoutant des instances de machine virtuelle supplémentaires. Vous pouvez également activer la mise à l’échelle automatique, bien que celle-ci soit plus lente que la mise à l’échelle élastique du plan Premium. Pour plus d’informations, consultez [Mettre à l’échelle le nombre d’instances manuellement ou automatiquement](../azure-monitor/platform/autoscale-get-started.md?toc=%2fazure%2fapp-service%2ftoc.json). Vous pouvez également effectuer une montée en puissance en choisissant un autre plan App Service. Pour plus d’informations, consultez [Faire monter en puissance une application web dans Azure](../app-service/manage-scale-up.md). 

Lorsque vous exécutez des fonctions JavaScript dans un plan App Service, vous devez choisir un plan qui comporte moins de processeurs virtuels. Pour plus d’informations, consultez [Choisir des plans App Service à cœur unique](functions-reference-node.md#choose-single-vcpu-app-service-plans). 
<!-- Note: the portal links to this section via fwlink https://go.microsoft.com/fwlink/?linkid=830855 --> 

L’exécution dans une instance [App Service Environment](../app-service/environment/intro.md) vous permet d’isoler entièrement vos fonctions et de tirer parti d’un nombre d’instances plus élevé que dans un plan App Service.

### <a name="always-on"></a><a name="always-on"></a> Always On

Si vous utilisez un plan App Service, vous devez activer le paramètre **Always On** afin que l’application de fonction s’exécute correctement. Dans un plan App Service, comme le runtime des fonctions devient inactif après quelques minutes d’inactivité, seuls des déclencheurs HTTP peuvent « relancer » vos fonctions. Le paramètre Always On est disponible uniquement dans un plan App Service. Dans un plan Consommation, la plateforme active automatiquement les applications de fonction.

[!INCLUDE [Timeout Duration section](../../includes/functions-timeout-duration.md)]


Même lorsque le paramètre Always On est activé, le délai d’exécution des fonctions individuelles est contrôlé par le paramètre `functionTimeout` dans le fichier projet [host.json](functions-host-json.md#functiontimeout).

## <a name="determine-the-hosting-plan-of-an-existing-application"></a>Déterminer le plan d’hébergement d’une application existante

Pour identifier le plan d’hébergement utilisé par votre application de fonction, consultez la section **Plan App Service** dans l’onglet **Vue d’ensemble** correspondant à l’application de fonction dans le [portail Azure](https://portal.azure.com). Pour afficher le niveau tarifaire, sélectionnez le nom **Plan App Service**, puis sélectionnez **Propriétés** dans le volet de gauche.

![Afficher le plan de mise à l’échelle dans le portail](./media/functions-scale/function-app-overview-portal.png)

Vous pouvez également utiliser l’interface de ligne de commande Azure pour identifier le plan utilisé tel que suit :

```azurecli-interactive
appServicePlanId=$(az functionapp show --name <my_function_app_name> --resource-group <my_resource_group> --query appServicePlanId --output tsv)
az appservice plan list --query "[?id=='$appServicePlanId'].sku.tier" --output tsv
```  

Lorsque la sortie de cette commande est `dynamic`, votre application de fonction utilise le plan Consommation. Lorsque la sortie de cette commande est `ElasticPremium`, votre Function App utilise le plan Premium. Les autres valeurs représentent un plan App Service avec des niveaux différents.

## <a name="storage-account-requirements"></a>Conditions requises pour le compte de stockage

Dans n’importe quel plan, une Function App nécessite un compte de stockage Azure général prenant en charge les stockages Blob, File d’attente, Fichiers et Table Azure. Cela s’explique par le fait qu’Azure Functions utilise le stockage Azure pour des opérations, telles que la gestion des déclencheurs et la journalisation des exécutions de fonctions. Certains comptes de stockage ne prennent toutefois pas en charge les files d’attente et les tables. Ces comptes, qui incluent les comptes de stockage Blob uniquement (notamment le stockage Premium) et les comptes de stockage à usage général avec la réplication de stockage redondant interzone (ZRS), ne sont pas inclus dans la liste de **comptes de stockage** existants que vous pouvez sélectionner lorsque vous créez une application de fonction.

Le même compte de stockage que celui utilisé par votre application de fonction peut également être utilisé par vos déclencheurs et liaisons pour stocker vos données d’application. Toutefois, pour les opérations gourmandes en stockage, il est préférable d’utiliser un compte de stockage distinct.  

Plusieurs applications de fonction peuvent partager le même compte de stockage sans aucun problème. (Cela peut notamment être le cas lorsque vous développez plusieurs applications dans votre environnement local à l'aide de l'Émulateur de stockage Azure, qui agit comme un compte de stockage unique.) 

<!-- JH: Does using a Premium Storage account improve perf? -->

Pour en savoir plus sur les types de compte de stockage, consultez [Présentation des services Stockage Azure](../storage/common/storage-introduction.md#core-storage-services).

### <a name="in-region-data-residency"></a>Résidence des données dans la région

Lorsque cela est nécessaire pour que toutes les données clients restent dans une seule région, le compte de stockage associé à l’application de fonction doit être un compte avec une [redondance dans la région](../storage/common/storage-redundancy.md).  Un compte de stockage redondant dans la région doit également être utilisé avec [Azure Durable Functions](./durable/durable-functions-perf-and-scale.md#storage-account-selection) pour Durable Functions.

Les autres données clients gérées par la plateforme ne seront stockées au sein de la région que si elles sont hébergées dans une instance Internal Load Balancer App Service Environment (ou ILB ASE).  Pour plus d’informations, consultez la section relative à la [redondance de zone ASE](../app-service/environment/zone-redundancy.md#in-region-data-residency).

## <a name="how-the-consumption-and-premium-plans-work"></a>Fonctionnement des plans Consommation et Premium

Dans les plans Consommation et Premium, l’infrastructure d’Azure Functions met à l’échelle les ressources processeur et mémoire en ajoutant des instances de l’hôte Functions selon le nombre d’événements sur lesquels ses fonctions sont déclenchées. Chaque instance de l’hôte Functions dans le plan Consommation est limitée à 1,5 Go de mémoire et un seul processeur.  Une instance de l’hôte constitue l’intégralité de la Function App, ce qui signifie que toutes les fonctions dans une Function App partagent des ressources au sein d’une instance et sont mises à l’échelle simultanément. Les applications de fonction qui partagent le même plan Consommation sont mises à l’échelle indépendamment.  Dans le plan Premium, la taille de votre plan détermine la mémoire et le processeur disponibles pour toutes les applications de ce plan, sur cette instance.  

Les fichiers de code de fonction sont stockés dans des partages Azure Files du compte de stockage principal de la fonction. Lorsque vous supprimez le compte de stockage principal de l’application de fonction, les fichiers de code de fonction sont supprimés et ne peuvent pas être récupérés.

### <a name="runtime-scaling"></a>Mise à l’échelle du runtime

Azure Functions utilise un composant appelé *contrôleur de mise à l’échelle* pour surveiller la fréquence des événements et déterminer s’il convient d’effectuer un scale-out ou un scale-in. Le contrôleur de mise à l’échelle utilise une méthode heuristique pour chaque type de déclencheur. Par exemple, si vous utilisez un déclencheur de stockage File d’attente Azure, il est mis à l’échelle en fonction de la longueur de la file d’attente et de l’âge du plus ancien message en file d’attente.

L’unité d’échelle pour Azure Functions est la Function App. Quand les instances de l’application de fonction font l’objet d’une augmentation de taille, des ressources supplémentaires sont allouées pour exécuter plusieurs instances de l’hôte Azure Functions. À l’inverse, quand la demande de calcul est réduite, le contrôleur de mise à l’échelle supprime des instances de l’hôte de fonction. Le nombre d’instances est finalement *réduit à zéro* si aucune fonction n’est exécutée dans une application de fonction.

![Contrôleur de mise à l’échelle surveillant les événements et créant des instances](./media/functions-scale/central-listener.png)

### <a name="cold-start"></a>Démarrage à froid

Une fois que votre application de fonction a été inactive pendant quelques minutes, la plateforme peut effectuer un scale-down à zéro du nombre d’instances sur lesquelles votre application s’exécute. La prochaine requête présente la latence supplémentaire de la mise à l’échelle de zéro à un. Cette latence est appelée _démarrage à froid_. Le nombre de dépendances que votre application de fonction doit charger peut avoir un impact sur le temps de démarrage à froid. Le démarrage à froid est plus problématique pour les opérations synchrones, telles que les déclencheurs HTTP, qui doivent retourner une réponse. Si les démarrages à froid ont un impact sur vos fonctions, envisagez l’exécution dans un plan Premium ou dans un plan dédié avec activation de l’option Always-on.   

### <a name="understanding-scaling-behaviors"></a>Présentation des comportements de mise à l’échelle

La mise à l’échelle peut varier en fonction de certains facteurs et selon le déclencheur et le langage sélectionnés. Il est nécessaire de connaître certaines subtilités relatives aux comportements de mise à l’échelle :

* Une application de fonction peut faire l’objet d’un scale-out jusqu’à 200 instances au maximum. Une seule instance, par contre, peut traiter plusieurs messages ou requêtes à la fois, ainsi il n’y a pas de limite définie sur le nombre d’exécutions simultanées.  Vous pouvez [spécifier une valeur maximale inférieure](#limit-scale-out) pour limiter l’échelle en fonction des besoins.
* Pour les déclencheurs HTTP, de nouvelles instances sont allouées, au plus, une fois par seconde.
* Pour les déclencheurs non HTTP, de nouvelles instances sont allouées, au plus, une fois toutes les 30 secondes. La mise à l’échelle est plus rapide lors de l’exécution dans [plan Premium](#premium-plan).
* Pour les déclencheurs Service Bus, utilisez des _Droits de gestion_ sur les ressources pour une mise à l’échelle d’une efficacité optimale. Avec des _Droits d’écoute_, la mise à l’échelle n’est pas aussi précise parce que la longueur de la file d’attente ne peut pas être utilisée pour informer des décisions de mise à l’échelle. Pour en savoir plus sur la définition de droits dans les stratégies d’accès Service Bus, consultez [Stratégie d’autorisation d’accès partagé](../service-bus-messaging/service-bus-sas.md#shared-access-authorization-policies).
* Pour les déclencheurs Event Hub, consultez les [conseils de mise à l’échelle](functions-bindings-event-hubs-trigger.md#scaling) dans l’article de référence. 

### <a name="limit-scale-out"></a>Limiter le scale-out

Vous souhaiterez peut-être limiter le nombre d’instances jusqu’auquel une application effectue un scale-out.  C’est le cas le plus fréquent lorsqu’un composant en aval comme une base de données a un débit limité.  Par défaut, les fonctions du plan de consommation effectuent un scale-out jusqu’à 200 instances, et les fonctions du plan Premium jusqu’à 100 instances.  Vous pouvez spécifier une valeur maximale inférieure pour une application spécifique en modifiant la valeur `functionAppScaleLimit`.  Le `functionAppScaleLimit` peut être défini sur 0 ou sur Null pour une utilisation sans restriction, ou sur une valeur valide comprise entre 1 et le maximum de l’application.

```azurecli
az resource update --resource-type Microsoft.Web/sites -g <resource_group> -n <function_app_name>/config/web --set properties.functionAppScaleLimit=<scale_limit>
```

### <a name="best-practices-and-patterns-for-scalable-apps"></a>Bonnes pratiques et modèles pour les applications scalables

Nombreux sont les aspects d’une application de fonction qui impactent sa bonne mise à l’échelle, notamment la configuration de l’hôte, l’encombrement du runtime et l’efficacité des ressources.  Pour plus d’informations, consultez la [section sur l’extensibilité dans l’article Considérations relatives aux performances](functions-best-practices.md#scalability-best-practices). Vous devez également savoir ce qu’il se passe au niveau des connexions lors de la mise à l’échelle de votre application de fonction. Pour plus d’informations, consultez [How to manage connections in Azure Functions](manage-connections.md) (Comment gérer des connexions dans Azure Functions).

Pour plus d’informations sur la mise à l’échelle en Python et Node.js, consultez le [Guide des développeurs Python sur Azure Functions - Mise à l’échelle et concurrence](functions-reference-python.md#scaling-and-performance) et le [Guide des développeurs Node.js sur Azure Functions - Mise à l’échelle et concurrence](functions-reference-node.md#scaling-and-concurrency).

### <a name="billing-model"></a>Modèle de facturation

La facturation des différents plans est décrite en détail dans la [page Tarification d’Azure Functions](https://azure.microsoft.com/pricing/details/functions/). L’utilisation est agrégée au niveau de l’application de fonction et compte uniquement la durée d’exécution du code de fonction. Les unités de facturation sont les suivantes :

* **Consommation des ressources en gigaoctet/seconde (Go/s)** . Calcul effectué d’après une combinaison de la taille de la mémoire et de la durée d’exécution pour toutes les fonctions d’une application de fonction. 
* **Exécutions**. Comptées chaque fois qu’une fonction est exécutée en réponse à un déclencheur d’événements.

Vous trouverez des requêtes et des informations utiles pour vous aider à comprendre votre facture de consommation [dans la FAQ sur la facturation](https://github.com/Azure/Azure-Functions/wiki/Consumption-Plan-Cost-Billing-FAQ).

[Azure Functions pricing page]: https://azure.microsoft.com/pricing/details/functions

## <a name="hosting-plans-comparison"></a>Comparaison des plans d’hébergement

Le tableau de comparaison suivant présente tous les aspects importants pour aider à déterminer le choix du plan d’hébergement de l’application Azure Functions :

### <a name="plan-summary"></a>Résumé du plan
| | |
| --- | --- |  
|**[Plan de consommation](#consumption-plan)**| Mettez à l’échelle automatiquement et ne payez les ressources de calcul que lorsque vos fonctions sont en cours d’exécution. Dans le plan de consommation, les instances de l’hôte Functions sont ajoutées et supprimées de façon dynamique en fonction du nombre d’événements entrants.<br/> ✔ Plan d’hébergement par défaut.<br/>✔ Paiement uniquement en cas d’exécution de vos fonctions.<br/>✔ Scale-out automatique, même pendant les périodes de charge élevée.|  
|**[Plan Premium](#premium-plan)**|Alors que la mise à l’échelle automatique dépend de la demande, utilisez des rôles de travail préparés pour exécuter des applications sans délai après leur inactivité, procédez à l’exécution sur des instances plus puissantes et connectez-vous à des réseaux virtuels. Envisagez le plan Azure Functions Premium dans les cas suivants, en plus de toutes les fonctionnalités du plan App Service : <br/>✔Vos applications de fonction s’exécutent en continu ou presque.<br/>✔ Vous disposez d’un grand nombre d’exécutions de petite taille et avez une facture d’exécution élevée, mais également une facture de Go par seconde faible dans le plan de consommation.<br/>✔Vous avez besoin de plus d’options de mémoire ou de processeur que celles qui sont proposées dans le plan de consommation.<br/>✔Votre code exige une durée d’exécution supérieure à celle qui est autorisée dans le plan de consommation.<br/>✔ Vous avez besoin de fonctionnalités qui sont uniquement disponibles dans un plan Premium, notamment la connectivité de réseau virtuel.|  
|**[Plan dédié](#app-service-plan)** <sup>1</sup>|Exécutez vos fonctions au sein d’un plan App Service aux tarifs habituels du plan App Service. Adapté aux opérations de longue durée et en cas de nécessité de coûts et d’une mise à l’échelle plus prédictifs. Envisagez un plan App Service dans les situations suivantes :<br/>✔ Vous disposez de machines virtuelles existantes sous-utilisées qui exécutent déjà d’autres instances App Service.<br/>✔ Vous souhaitez fournir une image personnalisée sur laquelle exécuter vos fonctions.|  
|**[ASE](#app-service-plan)** <sup>1</sup>|App Service Environment(ASE) est une fonctionnalité d’App Service qui fournit un environnement totalement isolé et dédié pour l’exécution sécurisée de vos applications App Service à grande échelle. Les environnements ASE conviennent aux charges de travail d’application qui nécessitent : <br/>✔ Une très grande échelle.<br/>✔ Une isolation totale du calcul et un accès réseau sécurisé.<br/>✔ Une utilisation élevée de la mémoire.|  
| **[Kubernetes](functions-kubernetes-keda.md)** | Kubernetes offre un environnement totalement isolé et dédié qui s’exécute sur la plateforme Kubernetes.  Kubernetes convient aux charges de travail d’application qui nécessitent : <br/>✔ Des exigences matérielles personnalisées.<br/>✔ Une isolation et un accès réseau sécurisé.<br/>✔ Une possibilité d’exécution dans un environnement hybride ou multi-cloud.<br/>✔ Une exécution en parallèle de services et d’applications Kubernetes existants.|  

<sup>1</sup> Pour connaître les limites spécifiques des différentes options du plan App Service, consultez les [limites du plan App Service](../azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits).

### <a name="operating-systemruntime"></a>Système d'exploitation/runtime

| | Linux<sup>1</sup><br/>Code uniquement | Windows<sup>2</sup><br/>Code uniquement | Linux<sup>1,3</sup><br/>Conteneur Docker |
| --- | --- | --- | --- |
| **[Plan de consommation](#consumption-plan)** | .NET Core<br/>Node.js<br/>Java<br/>Python | .NET Core<br/>Node.js<br/>Java<br/>PowerShell Core | Aucune prise en charge  |
| **[Plan Premium](#premium-plan)** | .NET Core<br/>Node.js<br/>Java<br/>Python|.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core |.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core<br/>Python  | 
| **[Plan dédié](#app-service-plan)** <sup>4</sup> | .NET Core<br/>Node.js<br/>Java<br/>Python|.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core |.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core<br/>Python |
| **[ASE](#app-service-plan)** <sup>4</sup> | .NET Core<br/>Node.js<br/>Java<br/>Python |.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core  |.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core<br/>Python | 
| **[Kubernetes](functions-kubernetes-keda.md)** | n/a | n/a |.NET Core<br/>Node.js<br/>Java<br/>PowerShell Core<br/>Python |

<sup>1</sup>Linux est le seul système d'exploitation pris en charge pour votre pile d'exécution Python.  
<sup>2</sup>Windows est le seul système d'exploitation pris en charge pour votre pile d'exécution PowerShell.   
<sup>3</sup>Linux est le seul système d’exploitation pris en charge pour les conteneurs Docker.
<sup>4</sup> Pour connaître les limites spécifiques des différentes options du plan App Service, consultez les [limites du plan App Service](../azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits).

### <a name="scale"></a>Scale

| | Scale-out | Nombre maximal d’instances |
| --- | --- | --- |
| **[Plan de consommation](#consumption-plan)** | Basé sur les événements. Effectuer un scale-out automatique, même pendant les périodes de charge élevée L’infrastructure Azure Functions met automatiquement à l’échelle les ressources processeur et mémoire en ajoutant des instances de l’hôte Functions selon le nombre d’événements en fonction desquels ses fonctions sont déclenchées. | 200 |
| **[Plan Premium](#premium-plan)** | Basé sur les événements. Effectuer un scale-out automatique, même pendant les périodes de charge élevée L’infrastructure Azure Functions met automatiquement à l’échelle les ressources processeur et mémoire en ajoutant des instances de l’hôte Functions selon le nombre d’événements en fonction desquels ses fonctions sont déclenchées. |100|
| **[Plan dédié](#app-service-plan)** <sup>1</sup> | Manuel/Mise à l’échelle automatique |10-20|
| **[ASE](#app-service-plan)** <sup>1</sup> | Manuel/Mise à l’échelle automatique |100 |
| **[Kubernetes](functions-kubernetes-keda.md)**  | Mise à l’échelle automatique basée sur les événements pour les clusters Kubernetes qui utilisent [KEDA](https://keda.sh). | Varie&nbsp;en fonction du&nbsp;cluster.&nbsp;&nbsp;|

<sup>1</sup> Pour connaître les limites spécifiques des différentes options du plan App Service, consultez les [limites du plan App Service](../azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits).

### <a name="cold-start-behavior"></a>Comportement du démarrage à froid

|    |    | 
| -- | -- |
| **[Plan de&nbsp;consommation](#consumption-plan)** | Les applications peuvent être mises à l’échelle jusqu’à zéro en cas d’inactivité pendant une certaine période, ce qui signifie que certaines requêtes peuvent présenter une latence supplémentaire au démarrage.  Le plan de consommation offre des optimisations pour réduire le temps de démarrage à froid, notamment en extrayant à partir de fonctions d’espace réservé préparées qui exécutent déjà les processus de langage et d’hôte de fonction. |
| **[Plan Premium](#premium-plan)** | Les instances perpétuellement chaudes permettent d’éviter les démarrages à froid. |
| **[Plan dédié](#app-service-plan)** <sup>1</sup> | En cas d’exécution dans un plan dédié, l’hôte Functions peut s’exécuter en continu, ce qui signifie que le démarrage à froid n’est pas vraiment un problème. |
| **[ASE](#app-service-plan)** <sup>1</sup> | En cas d’exécution dans un plan dédié, l’hôte Functions peut s’exécuter en continu, ce qui signifie que le démarrage à froid n’est pas vraiment un problème. |
| **[Kubernetes](functions-kubernetes-keda.md)**  | Dépend de la configuration KEDA. Les applications peuvent être configurées de manière à toujours s’exécuter et à ne jamais présenter de démarrage à froid, ou de manière à être mises à l’échelle jusqu’à zéro, ce qui se traduit par un démarrage à froid lors de nouveaux événements. 

<sup>1</sup> Pour connaître les limites spécifiques des différentes options du plan App Service, consultez les [limites du plan App Service](../azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits).

### <a name="service-limits"></a>Limites du service

[!INCLUDE [functions-limits](../../includes/functions-limits.md)]

### <a name="networking-features"></a>Fonctionnalités réseau

[!INCLUDE [functions-networking-features](../../includes/functions-networking-features.md)]

### <a name="billing"></a>Facturation

| | | 
| --- | --- |
| **[Plan de consommation](#consumption-plan)** | Ne payez que la durée d’exécution de vos fonctions. La facturation est basée sur le nombre d’exécutions, la durée d’exécution et la mémoire utilisée. |
| **[Plan Premium](#premium-plan)** | Le plan Premium se base sur le nombre de cœurs-seconde et la mémoire utilisée sur les instances nécessaires et préparées. Au moins une instance par plan doit être chaude en permanence. Ce plan offre une tarification plus prévisible. |
| **[Plan dédié](#app-service-plan)** <sup>1</sup> | Le coût des Function App dans un plan App Service est le même que pour d’autres ressources App Service, par exemple les applications web.|
| **[ASE](#app-service-plan)** <sup>1</sup> | Un tarif fixe mensuel couvre l’infrastructure d’un ASE et ne change pas avec la taille de l’ASE. Chaque processeur virtuel de plan App Service présente aussi un coût. Toutes les applications hébergées dans un environnement App Service sont dans la référence (SKU) de prix Isolée. |
| **[Kubernetes](functions-kubernetes-keda.md)**| Vous ne payez que le coût de votre cluster Kubernetes et aucune facturation supplémentaire pour Functions. Votre application de fonction s’exécute comme une charge de travail d’application sur votre cluster, tout comme une application normale. |

<sup>1</sup> Pour connaître les limites spécifiques des différentes options du plan App Service, consultez les [limites du plan App Service](../azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits).

## <a name="next-steps"></a>Étapes suivantes

+ [Démarrage rapide : Créer un projet Azure Functions avec Visual Studio Code](./create-first-function-vs-code-csharp.md)
+ [Technologies de déploiement dans Azure Functions](functions-deployment-technologies.md) 
+ [Guide du développeur Azure Functions](functions-reference.md)