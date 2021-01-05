---
title: Plan Premium Azure Functions
description: Détails et options de configuration (VNet, pas de démarrage à froid, durée d’exécution illimitée) pour le plan Azure Functions Premium.
author: jeffhollan
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: jehollan
ms.custom:
- references_regions
- fasttrack-edit
- devx-track-azurecli
ms.openlocfilehash: 7efcff5709995898a6ec950dfea6450f7e0dd48d
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92736805"
---
# <a name="azure-functions-premium-plan"></a>Plan Premium Azure Functions

Le plan Azure Functions Premium (parfois appelé plan Elastic Premium) est une option d'hébergement destinée aux applications de fonction. Il présente des avantages tels que la connectivité réseau virtuel, l’absence de démarrage à froid et du matériel de pointe.  Plusieurs applications de fonction peuvent être déployées sur le même plan Premium, et celui-ci vous permet de configurer la taille d’instance de calcul, la taille du plan de base et taille maximale de plan.  Pour une comparaison entre le plan Premium et d’autres types de plans et d’hébergements, voir [options de mise à l’échelle et d’hébergement de fonction](functions-scale.md).

## <a name="create-a-premium-plan"></a>Créer un plan Premium

[!INCLUDE [functions-premium-create](../../includes/functions-premium-create.md)]

Vous pouvez également créer un plan Premium à l’aide de la commande [az functionapp plan create](/cli/azure/functionapp/plan#az-functionapp-plan-create) dans Azure CLI. L’exemple suivant crée un plan de niveau _Élastique Premium 1_  :

```azurecli-interactive
az functionapp plan create --resource-group <RESOURCE_GROUP> --name <PLAN_NAME> \
--location <REGION> --sku EP1
```

Dans cet exemple, remplacez `<RESOURCE_GROUP>` par votre groupe de ressources `<PLAN_NAME>` et par le nom de votre plan, qui doit être unique dans le groupe de ressources. Spécifiez une [`<REGION>` prise en charge](https://azure.microsoft.com/global-infrastructure/services/?products=functions). Pour créer un plan Premium qui prend en charge Linux, ajoutez l’option `--is-linux`.

Une fois le plan créé, vous pouvez utiliser [az functionapp create](/cli/azure/functionapp#az-functionapp-create) pour créer votre application de fonction. Dans le portail, le plan et l’application sont créés en même temps. Pour obtenir un exemple de script Azure CLI complet, consultez [Créer une application de fonction sur le plan Premium](scripts/functions-cli-create-premium-plan.md).

## <a name="features"></a>Fonctionnalités

Les fonctionnalités suivantes sont disponibles pour les applications de fonction déployées sur un plan Premium.

### <a name="always-ready-instances"></a>Instances toujours prêtes

Si aucun événement ou aucune exécution ne se produisent aujourd’hui dans le plan Consommation, il se peut que votre application effectue un scale-in jusqu’à zéro instance. Quand un nouvel événement se produit, une nouvelle instance doit être spécialisée avec votre application s’exécutant sur celle-ci.  La spécialisation d’une nouvelle instance peut prendre un certain temps en fonction de l’application.  Cette latence supplémentaire du premier appel est souvent appelée démarrage à froid d’application.

Dans le plan Premium, vous pouvez faire en sorte que votre application soit toujours prête sur un nombre spécifié d’instances.  Le nombre maximal d’instances toujours prêtes est de 20.  Quand l’application commence à déclencher des événements, ceux-ci sont d’abord routés vers les instances toujours prêtes.  Dès que la fonction devient active, des instances supplémentaires sont initialisées en tant que mémoire tampon.  Cette mémoire tampon empêche le démarrage à froid des nouvelles instances nécessaires pendant la mise à l’échelle.  Ces instances mises en mémoire tampon s’appellent des [instances préchauffées](#pre-warmed-instances).  L’association des instances toujours prêtes combinées à une mémoire tampon préchauffée permet en quelque sorte à votre application d’éliminer le démarrage à froid.

> [!NOTE]
> Chaque plan Premium dispose en permanence d’au moins une instance active (facturée).

Vous pouvez configurer le nombre d’instances toujours prêtes sur le portail Azure en sélectionnant votre **application de fonction** , en accédant à l’onglet **Fonctionnalités de la plateforme** , puis en sélectionnant les options **Monter en charge** . Dans la fenêtre de modification de l’application de fonction, les instances toujours prêtes sont propres à cette application.

![Paramètres de mise à l’échelle élastique](./media/functions-premium-plan/scale-out.png)

Vous pouvez aussi configurer des instances toujours prêtes pour une application avec Azure CLI.

```azurecli-interactive
az resource update -g <resource_group> -n <function_app_name>/config/web --set properties.minimumElasticInstanceCount=<desired_always_ready_count> --resource-type Microsoft.Web/sites
```

#### <a name="pre-warmed-instances"></a>Instances préchauffées

Les instances préchauffées correspondent au nombre d’instances initialisées en tant que mémoire tampon à l’occasion d’événements de mise à l’échelle et d’activation.  Les instances préchauffées continuent d’être mises en mémoire tampon tant que la limite maximale de scale-out n’est pas atteinte.  Le nombre d’instances préchauffées par défaut est de 1, et dans la plupart des scénarios, la valeur reste à 1.  Si une application demande un long temps de préparation (comme une image de conteneur personnalisée), vous souhaiterez peut-être augmenter la capacité de cette mémoire tampon.  Une instance préchauffée ne devient active qu’après que toutes les instances actives ont été suffisamment utilisées.

Pour illustrer la façon dont les instances toujours prêtes et les instances préchauffées fonctionnent ensemble, prenons l’exemple suivant :  une application de fonction Premium compte cinq instances toujours prêtes configurées et une instance préchauffée (par défaut).  Quand l’application est inactive et qu’aucun événement n’est déclenché, l’application est provisionnée et s’exécute sur cinq instances.  À ce stade, vous ne serez pas facturé pour une instance préchauffée, car les instances toujours prêtes ne sont pas utilisées et aucune instance préchauffée n’est encore allouée.

Dès que le premier déclencheur intervient, les cinq instances toujours prêtes s’activent et une instance préchauffée est allouée.  L’application s’exécute maintenant avec six instances provisionnées : les cinq instances toujours prêtes désormais actives et la sixième mémoire tampon préchauffée et inactive.  Si le rythme des exécutions continue d’augmenter, les cinq instances actives sont finalement utilisées.  Quand la plateforme décide de procéder à une mise à l’échelle au-delà de cinq instances, elle englobe l’instance préchauffée.  Dans ce cas, les instances actives sont au nombre de six et une septième instance est instantanément provisionnée et remplit la mémoire tampon préchauffée.  Cette séquence de mise à l’échelle et de préchauffage se poursuit tant que le nombre maximal d’instances pour l’application n’est pas atteint.  Aucune instance n’est préchauffée ou activée au-delà de la valeur maximale.

Vous pouvez modifier le nombre d’instances préchauffées pour une application à partir d’Azure CLI.

```azurecli-interactive
az resource update -g <resource_group> -n <function_app_name>/config/web --set properties.preWarmedInstanceCount=<desired_prewarmed_count> --resource-type Microsoft.Web/sites
```

#### <a name="maximum-instances-for-an-app"></a>Nombre maximal d’instances pour une application

En plus du [nombre maximal d’instances du plan](#plan-and-sku-settings), vous pouvez configurer un maximum par application.  La valeur maximale d’une application peut être configurée en utilisant la [limite d’échelle de l’application](./functions-scale.md#limit-scale-out).

### <a name="private-network-connectivity"></a>Connectivité de réseau privé

Azure Functions déployée sur un plan Premium tire parti de la [nouvelle expérience d’intégration de réseau virtuel pour les applications web](../app-service/web-sites-integrate-with-vnet.md).  Une fois configurée, votre application peut communiquer avec des ressources de votre réseau virtuel ou sécurisées via des points de terminaison de service.  Des restrictions d’adresse IP sont également disponibles sur l’application pour limiter le trafic entrant.

Lors de l’attribution d’un sous-réseau à votre Function App dans un plan Premium, vous avez besoin d’un sous-réseau avec suffisamment d’adresses IP pour chaque instance potentielle. Nous imposons un bloc d’au moins 100 adresses IP disponibles.

Pour plus d’informations, consultez [Intégrer votre application de fonction à un réseau virtuel](functions-create-vnet.md).

### <a name="rapid-elastic-scale"></a>Mise à l’échelle élastique rapide

Des instances de calcul supplémentaires sont automatiquement ajoutées pour votre application en utilisant la même logique de mise à l’échelle rapide que le Plan Consommation. Les applications du même plan App Service sont mises à l’échelle indépendamment les unes des autres en fonction des besoins d’une application individuelle. Toutefois, les applications Functions dans le même plan App Service partagent des ressources de machine virtuelle pour aider à réduire les coûts, lorsque cela est possible. Le nombre d’applications associées à une machine virtuelle dépend de l’encombrement de chaque application et de la taille de la machine virtuelle.

Pour en savoir plus sur le fonctionnement de la mise à l’échelle, voir [Échelle et hébergement dans Azure Functions](./functions-scale.md#how-the-consumption-and-premium-plans-work).

### <a name="longer-run-duration"></a>Durée d’exécution plus longue

Azure Functions dans un Plan Consommation est limité à 10 minutes par exécution.  Dans le plan Premium, la durée d’exécution par défaut est de 30 minutes pour éviter toute perte de contrôle. Cependant, vous pouvez [modifier la configuration de host.json](./functions-host-json.md#functiontimeout) pour faire en sorte que la durée soit illimitée pour les applications du plan Premium (60 minutes garanties).

## <a name="plan-and-sku-settings"></a>Paramètres du plan et de la référence SKU

Pendant la création du plan, deux paramètres de taille de plan sont proposés : le nombre minimal d’instances (ou taille du plan) et la limite maximale en rafale.

Si votre application exige un nombre d’instances supérieur au nombre d’instances toujours prêtes, elle peut continuer d’effectuer un scale-out tant que le nombre d’instances n’a pas atteint la limite maximale en rafale.  Vous êtes facturé pour des instances au-delà de la taille de votre plan uniquement quand elles sont en cours d’exécution et vous sont allouées à la seconde.  Nous ferons tout notre possible pour assurer le scale-out de votre application jusqu’à sa limite maximale définie.

Vous pouvez configurer la taille et les nombres maximaux d’instances du plan via le portail Azure en sélectionnant les options **Scale-out** du plan ou une Function App déployée sur celui-ci (sous **Fonctionnalités de la plateforme** ).

Vous pouvez également augmenter la limite maximale en rafale à partir d’Azure CLI :

```azurecli-interactive
az functionapp plan update -g <resource_group> -n <premium_plan_name> --max-burst <desired_max_burst>
```

La valeur minimale pour chaque plan sera d’au moins une instance.  Le nombre minimal effectif d’instances sera automatiquement configuré en fonction du nombre d’instances toujours prêtes demandées par les applications du plan.  Par exemple, si l’application A demande cinq instances toujours prêtes et que l’application B en demande deux dans le même plan, la taille minimale du plan calculée sera de cinq.  L’application A s’exécutera sur les cinq instances et l’application B s’exécutera uniquement sur deux instances.

> [!IMPORTANT]
> Vous êtes facturé pour chaque instance allouée en lien avec le nombre minimal d’instances, que les fonctions s’exécutent ou non.

Dans la plupart des cas, ce minimum calculé automatiquement s’avère suffisant.  Cependant, si nécessaire, nous ferons de notre mieux pour assurer une mise à l’échelle au-delà du nombre minimal.  Même si cela est peu probable, il est possible qu’à un moment donné, le scale-out soit retardé du fait de l’indisponibilité d’instances supplémentaires.  En définissant un minimum supérieur au minimum calculé automatiquement, vous réservez des instances préalablement au scale-out.

Il est possible d’augmenter le minimum calculé d’un plan à partir d’Azure CLI.

```azurecli-interactive
az functionapp plan update -g <resource_group> -n <premium_plan_name> --min-instances <desired_min_instances>
```

### <a name="available-instance-skus"></a>Références SKU d’instance disponibles

Pendant la création ou la mise à l’échelle de votre plan, vous pouvez choisir entre trois tailles d’instance.  Vous serez facturé en fonction du nombre total de cœurs et de mémoire approvisionnés, pour chaque seconde où chaque instance vous est allouée.  Votre application peut automatiquement effectuer un scale-out sur plusieurs instances en fonction des besoins.

|SKU|Cœurs|Mémoire|Stockage|
|--|--|--|--|
|EP1|1|3,5 GO|250 Go|
|EP2|2|7 GO|250 Go|
|EP3|4|14 GO|250 Go|

### <a name="memory-utilization-considerations"></a>Considérations sur l’utilisation de la mémoire
L’exécution sur une machine avec davantage de mémoire ne signifie pas toujours que votre application de fonction va utiliser toute la mémoire disponible.

Par exemple, une application de fonction JavaScript est contrainte par la limite de mémoire par défaut dans Node.js. Pour augmenter cette limite de mémoire fixe, ajoutez le paramètre d’application `languageWorkers:node:arguments` avec la valeur `--max-old-space-size=<max memory in MB>`.

## <a name="region-max-scale-out"></a>Scale-out maximal par région

Vous trouverez ci-dessous les valeurs maximales de scale-out actuellement prises en charge pour un même plan dans chaque région et configuration de système d’exploitation. Pour demander une augmentation, ouvrez un ticket de support.

La disponibilité régionale complète de Functions est indiquée ici : [Azure.com](https://azure.microsoft.com/global-infrastructure/services/?products=functions)

|Région| Windows | Linux |
|--| -- | -- |
|Centre de l’Australie| 100 | Non disponible |
|Centre de l’Australie 2| 100 | Non disponible |
|Australie Est| 100 | 20 |
|Sud-Australie Est | 100 | 20 |
|Brésil Sud| 100 | 20 |
|Centre du Canada| 100 | 20 |
|USA Centre| 100 | 20 |
|Chine orientale 2| 100 | 20 |
|Chine Nord 2| 100 | 20 |
|Asie Est| 100 | 20 |
|USA Est | 100 | 20 |
|USA Est 2| 100 | 20 |
|France Centre| 100 | 20 |
|Allemagne Centre-Ouest| 100 | Non disponible |
|Japon Est| 100 | 20 |
|OuJapon Est| 100 | 20 |
|Centre de la Corée| 100 | 20 |
|Corée du Sud| Non disponible | 20 |
|Centre-Nord des États-Unis| 100 | 20 |
|Europe Nord| 100 | 20 |
|Norvège Est| 100 | 20 |
|États-Unis - partie centrale méridionale| 100 | 20 |
|Inde Sud | 100 | Non disponible |
|Asie Sud-Est| 100 | 20 |
|Suisse Nord| 100 | Non disponible |
|Suisse Ouest| 100 | Non disponible |
|Sud du Royaume-Uni| 100 | 20 |
|Ouest du Royaume-Uni| 100 | 20 |
|Gouvernement des États-Unis - Arizona| 100 | 20 |
|USGov Virginia| 100 | 20 |
|USNat East| 100 | Non disponible |
|USNat West| 100 | Non disponible |
|Europe Ouest| 100 | 20 |
|Inde Ouest| 100 | 20 |
|Centre-USA Ouest| 100 | 20 |
|USA Ouest| 100 | 20 |
|USA Ouest 2| 100 | 20 |

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Comprendre les options de mise à l’échelle et d’hébergement d’Azure Functions](functions-scale.md)
