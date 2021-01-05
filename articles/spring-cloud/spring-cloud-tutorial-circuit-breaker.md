---
title: 'Tutoriel : Utiliser le tableau de bord Circuit Breaker avec Azure Spring Cloud'
description: Découvrez comment utiliser le tableau de bord Circuit Breaker avec Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 04/06/2020
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 53884c2f6d9f2e8cbb5676e9ac10e8fb15ed919e
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024277"
---
# <a name="use-circuit-breaker-dashboard-with-azure-spring-cloud"></a>Utiliser le tableau de bord Circuit Breaker avec Azure Spring Cloud

**Cet article s’applique à :** ✔️ Java

Spring [Cloud Netflix Turbine](https://github.com/Netflix/Turbine) est couramment utilisé pour regrouper plusieurs flux de métriques [Hystrix](https://github.com/Netflix/Hystrix), en vue de leur supervision dans une vue unique à l’aide du tableau de bord Hystrix. Ce tutoriel montre comment les utiliser sur Azure Spring Cloud.
> [!NOTE]
> Netflix Hystrix est couramment utilisé dans de nombreuses applications Spring Cloud, mais il ne fait plus l’objet d’un développement actif. Si vous développez un nouveau projet, utilisez à la place des implémentations Spring Cloud Circuit Breaker comme [resilience4j](https://github.com/resilience4j/resilience4j). À la différence de Turbine présenté dans ce tutoriel, le nouveau framework Spring Cloud Circuit Breaker unifie toutes les implémentations de son pipeline de données de métriques dans Micrometer. Comme nous travaillons toujours à la prise en charge de Micrometer dans Azure Spring Cloud, il n’est pas abordé dans ce tutoriel.

## <a name="prepare-your-sample-applications"></a>Préparer vos exemples d’applications
L’exemple est dupliqué (fork) à partir de ce [dépôt](https://github.com/StackAbuse/spring-cloud/tree/master/spring-turbine).

Clonez l’exemple de dépôt dans votre environnement de développement :
```
git clone https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples.git
cd Azure-Spring-Cloud-Samples/hystrix-turbine-sample
```

Générez les 3 applications qui seront utilisées dans ce tutoriel :
* user-service : service REST simple qui a un seul point de terminaison de /personalized/{id}
* recommendation-service : service REST simple qui a un seul point de terminaison de /recommendations, qui est appelé par user-service.
* hystrix-turbine : service de tableau de bord Hystrix permettant d’afficher les flux Hystrix et service Turbine regroupant les flux de métriques Hystrix provenant d’autres services.
```
mvn clean package -D skipTests -f user-service/pom.xml
mvn clean package -D skipTests -f recommendation-service/pom.xml
mvn clean package -D skipTests -f hystrix-turbine/pom.xml
```
## <a name="provision-your-azure-spring-cloud-instance"></a>Provisionner votre instance Azure Spring Cloud
Suivez la procédure [Provisionner une instance de service sur Azure CLI](./spring-cloud-quickstart.md#provision-an-instance-of-azure-spring-cloud).

## <a name="deploy-your-applications-to-azure-spring-cloud"></a>Déployer vos applications sur Azure Spring Cloud
Ces applications n’utilisant pas de **serveur de configuration**, il n’est pas nécessaire de configurer de **serveur de configuration** pour Azure Spring Cloud.  Procédez à la création et au déploiement comme suit :
```azurecli
az spring-cloud app create -n user-service --is-public
az spring-cloud app create -n recommendation-service
az spring-cloud app create -n hystrix-turbine --is-public

az spring-cloud app deploy -n user-service --jar-path user-service/target/user-service.jar
az spring-cloud app deploy -n recommendation-service --jar-path recommendation-service/target/recommendation-service.jar
az spring-cloud app deploy -n hystrix-turbine --jar-path hystrix-turbine/target/hystrix-turbine.jar
```
## <a name="verify-your-apps"></a>Vérifier vos applications
Une fois que toutes les applications sont en cours d’exécution et découvrables, accédez à `user-service` en suivant le chemin https://<username>-user-service.azuremicroservices.io/personalized/1 à partir de votre navigateur. Si le service user-service peut accéder à `recommendation-service`, vous devez recevoir la sortie suivante. Actualisez la page web plusieurs fois si nécessaire.
```json
[{"name":"Product1","description":"Description1","detailsLink":"link1"},{"name":"Product2","description":"Description2","detailsLink":"link3"},{"name":"Product3","description":"Description3","detailsLink":"link3"}]
```
## <a name="access-your-hystrix-dashboard-and-metrics-stream"></a>Accéder à votre tableau de bord Hystrix et au flux de métriques
Effectuez une vérification à l’aide de points de terminaison publics ou de points de terminaison de test privés.

### <a name="using-public-endpoints"></a>Utilisation de points de terminaison publics
Accédez à hystrix-turbine en suivant le chemin `https://<SERVICE-NAME>-hystrix-turbine.azuremicroservices.io/hystrix` à partir de votre navigateur.  L’illustration suivante montre le tableau de bord Hystrix en cours d’exécution dans cette application.

![Tableau de bord Hystrix](media/spring-cloud-circuit-breaker/hystrix-dashboard.png)

Copiez l’URL du flux Turbine `https://<SERVICE-NAME>-hystrix-turbine.azuremicroservices.io/turbine.stream?cluster=default` dans la zone de texte, puis cliquez sur **Monitor Stream** (Superviser le flux).  Le tableau de bord s’affiche. Si rien ne s’affiche dans la visionneuse, appuyez sur les points de terminaison `user-service` pour générer des flux.

![Flux Hystrix](media/spring-cloud-circuit-breaker/hystrix-stream.png) Vous pouvez maintenant expérimenter le tableau Circuit Breaker.
> [!NOTE] 
> En production, le tableau de bord Hystrix et le flux de métriques ne doivent pas être exposés à Internet.

### <a name="using-private-test-endpoints"></a>Utilisation de points de terminaison de test privés
Les flux de métriques Hystrix sont également accessibles à partir de `test-endpoint`. S’agissant d’un service back-end, nous n’avons pas affecté de point de terminaison public pour `recommendation-service`, mais nous pouvons afficher ses métriques avec test-endpoint à l’adresse `https://primary:<KEY>@<SERVICE-NAME>.test.azuremicroservices.io/recommendation-service/default/actuator/hystrix.stream`

![Flux de points de terminaison de test Hystrix](media/spring-cloud-circuit-breaker/hystrix-test-endpoint-stream.png)

En tant qu’application web, le tableau de bord Hystrix doit fonctionner sur `test-endpoint`. Deux raisons peuvent expliquer son dysfonctionnement éventuel : l’utilisation de `test-endpoint` a changé l’URL de base de `/ to /<APP-NAME>/<DEPLOYMENT-NAME>`, ou l’application web utilise le chemin d’accès absolu de la ressource statique. Pour qu’il fonctionne sur `test-endpoint`, vous devrez peut-être modifier manuellement l’URL de <base> dans les fichiers front-end.

## <a name="next-steps"></a>Étapes suivantes
* [Provisionner une instance de service sur Azure CLI](./spring-cloud-quickstart.md#provision-an-instance-of-azure-spring-cloud)
* [Préparer une application Spring Java pour le déploiement dans Azure Spring Cloud](./spring-cloud-tutorial-prepare-app-deployment.md)

