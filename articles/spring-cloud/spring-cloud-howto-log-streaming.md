---
title: Envoyer en streaming les journaux d’application Azure Spring Cloud en temps réel
description: Comment utiliser la diffusion en continu des journaux pour afficher instantanément les journaux des applications
author: MikeDodaro
ms.author: barbkess
ms.service: spring-cloud
ms.topic: how-to
ms.date: 01/14/2019
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: fcfddce568be6c641a5bf5be70c2cd0ad368095f
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843602"
---
# <a name="stream-azure-spring-cloud-app-logs-in-real-time"></a>Envoyer en streaming les journaux d’application Azure Spring Cloud en temps réel

**Cet article s’applique à :** ✔️ Java ✔️ C#

Azure Spring Cloud active la diffusion en continu de journaux dans Azure CLI pour obtenir des journaux de console d’application en temps réel pour la résolution des problèmes. Vous pouvez également [analyser les journaux et les mesures à l’aide des paramètres de diagnostic](./diagnostic-services.md).

## <a name="prerequisites"></a>Conditions préalables requises

* Installez [extension Azure CLI](/cli/azure/install-azure-cli) pour Spring Cloud, version minimale 0.2.0.
* Une instance d’**Azure Spring Cloud** avec une application en cours d’exécution, par exemple [application Spring Cloud](./spring-cloud-quickstart.md).

> [!NOTE]
>  L’extension de l’interface CLI ASC est mise à jour de la version 0.2.0 vers la version 0.2.1. Cette modification affecte la syntaxe de la commande pour le streaming des journaux : `az spring-cloud app log tail`, qui est remplacée par : `az spring-cloud app logs`. La commande `az spring-cloud app log tail` sera dépréciée dans une version ultérieure. Si vous utilisez la version 0.2.0, vous pouvez effectuer la mise à niveau vers la version 0.2.1. Tout d’abord, supprimez l’ancienne version à l’aide de la commande `az extension remove -n spring-cloud`.  Ensuite, installez la version 0.2.1 à l’aide de la commande `az extension add -n spring-cloud`.

## <a name="use-cli-to-tail-logs"></a>Utiliser l’interface CLI pour les dernières lignes des journaux

Pour éviter de spécifier à plusieurs reprises le nom de votre groupe de ressources et de votre instance de service, définissez le nom du groupe de ressources et le nom du cluster par défaut.
```
az configure --defaults group=<service group name>
az configure --defaults spring-cloud=<service instance name>
```
Dans les exemples suivants, le groupe de ressources et le nom du service sont omis dans les commandes.

### <a name="tail-log-for-app-with-single-instance"></a>Dernières lignes du journal pour l’application avec une seule instance
Si une application nommée « auth-service » ne dispose que d’une seule instance, vous pouvez afficher le journal de l’instance de l’application à l’aide de la commande suivante :
```
az spring-cloud app logs -n auth-service
```
Des journaux seront retournés :
```
...
2020-01-15 01:54:40.481  INFO [auth-service,,,] 1 --- [main] o.apache.catalina.core.StandardService  : Starting service [Tomcat]
2020-01-15 01:54:40.482  INFO [auth-service,,,] 1 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.22]
2020-01-15 01:54:40.760  INFO [auth-service,,,] 1 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/uaa]  : Initializing Spring embedded WebApplicationContext
2020-01-15 01:54:40.760  INFO [auth-service,,,] 1 --- [main] o.s.web.context.ContextLoader  : Root WebApplicationContext: initialization completed in 7203 ms

...
```

### <a name="tail-log-for-app-with-multiple-instances"></a>Dernières lignes du journal pour l’application avec une seule instance
Si plusieurs instances existent pour l’application nommée `auth-service`, vous pouvez afficher le journal des instances à l’aide de l’option `-i/--instance`. 

Tout d’abord, vous pouvez récupérer les noms des instances de l’application à l’aide de la commande suivante.

```
az spring-cloud app show -n auth-service --query properties.activeDeployment.properties.instances -o table
```
Avec les résultats :

```
Name                                         Status    DiscoveryStatus
-------------------------------------------  --------  -----------------
auth-service-default-12-75cc4577fc-pw7hb  Running   UP
auth-service-default-12-75cc4577fc-8nt4m  Running   UP
auth-service-default-12-75cc4577fc-n25mh  Running   UP
``` 
Ensuite, vous pouvez diffuser en continu des journaux d’une instance d’application avec l’option `-i/--instance` :

```
az spring-cloud app logs -n auth-service -i auth-service-default-12-75cc4577fc-pw7hb
```

Vous pouvez également obtenir des informations sur les instances d’application à partir du portail Azure.  Après avoir sélectionné **Applications** dans le volet de navigation gauche de votre service Azure Spring Cloud, sélectionnez **instances de l’application**.

### <a name="continuously-stream-new-logs"></a>Diffuser en continu de nouveaux journaux
Par défaut, `az spring-cloud ap log tail` imprime uniquement les journaux existants diffusés en continu à la console d’application, puis se ferme. Si vous souhaitez diffuser de nouveaux journaux, ajoutez la lettre  f (--Follow) :  

```
az spring-cloud app logs -n auth-service -f
``` 
Pour vérifier toutes les options de journalisation prises en charge :
``` 
az spring-cloud app logs -h 
```

## <a name="next-steps"></a>Étapes suivantes
* [Démarrage rapide : Supervision des applications Azure Spring Cloud avec les journaux, les métriques et le suivi](spring-cloud-quickstart-logs-metrics-tracing.md)
* [Analyser les journaux et les indicateurs de performance avec les paramètres de diagnostic](./diagnostic-services.md)

