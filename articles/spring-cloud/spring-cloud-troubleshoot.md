---
title: Guide de dépannage pour Azure Spring Cloud | Microsoft Docs
description: Guide de dépannage pour Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: troubleshooting
ms.date: 09/08/2020
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 09415c47432f71310b10c86390c10e55f1ccc4b2
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498606"
---
# <a name="troubleshoot-common-azure-spring-cloud-issues"></a>Résoudre les problèmes courants liés à Azure Spring Cloud

Cet article fournit des instructions pour résoudre les problèmes de développement relatifs à Azure Spring Cloud. Pour plus d’informations, consultez [FAQ sur Azure Spring Cloud](spring-cloud-faq.md).

## <a name="availability-performance-and-application-issues"></a>Problèmes liés à la disponibilité, aux performances et aux applications

### <a name="my-application-cant-start-for-example-the-endpoint-cant-be-connected-or-it-returns-a-502-after-a-few-retries"></a>Mon application ne démarre pas (par exemple, le point de terminaison ne peut pas être connecté ou retourne une erreur 502 après quelques tentatives)

Exportez les journaux dans Azure Log Analytics. La table des journaux des applications Spring est nommée *AppPlatformLogsforSpring*. Pour en savoir plus, consultez [Analyser les journaux et les métriques avec les paramètres de diagnostic](diagnostic-services.md).

Le message d’erreur suivant peut s’afficher dans vos journaux :

> « org.springframework.context.ApplicationContextException : Impossible de démarrer le serveur web »

Le message indique l’un des deux problèmes probables : 
* L’un des beans ou l’une de ses dépendances est manquante.
* L’une des propriétés du bean est manquante ou non valide. Dans ce cas, « java.lang.IllegalArgumentException » s’affichera probablement.

Les liaisons de service peuvent également provoquer des échecs de démarrage de l’application. Pour interroger les journaux, utilisez les mots clés associés aux services liés. Par exemple, supposons que votre application ait une liaison à une instance MySQL qui est définie avec l’heure système locale. Si l’application ne parvient pas à démarrer, le message d’erreur suivant peut s’afficher dans le journal :

> « java.sql.SQLException : La valeur de fuseau horaire du serveur “Temps universel coordonné” n’est pas reconnue ou représente plus d’un fuseau horaire. »

Pour résoudre cette erreur, accédez à la section `server parameters` de votre instance MySQL et, pour la valeur `time_zone`, remplacez *SYSTEM* par *+0:00*.


### <a name="my-application-crashes-or-throws-an-unexpected-error"></a>Mon application se bloque ou déclenche une erreur inattendue

Quand vous déboguez des incidents d’application, commencez par vérifier l’état d’exécution et l’état de détection de l’application. Pour ce faire, accédez à _Gestion des applications_ dans le Portail Azure pour vérifier que les états de toutes les applications sont _En cours d’exécution_ et _En service_.

* Si l’état est _En cours d’exécution_, mais que l’état de détection n’est pas _En service_, accédez à la section [« Mon application ne peut pas être inscrite »](#my-application-cant-be-registered).

* Si l’état de détection est _En service_, accédez à Métriques pour vérifier l’intégrité de l’application. Inspectez les métriques suivantes :


  - `TomcatErrorCount` (_tomcat.global.error_) : Toutes les exceptions d’application Spring sont comptées ici. Si ce nombre est important, accédez à Azure Log Analytics pour inspecter vos journaux d’applications.

  - `AppMemoryMax` (_jvm.memory.max_) : Quantité maximale de mémoire disponible pour l’application. La quantité peut être indéfinie, ou elle peut changer au fil du temps si elle est définie. Si elle est définie, la quantité de mémoire utilisée et allouée est toujours inférieure ou égale au maximum. Toutefois, une répartition de mémoire peut échouer avec un message `OutOfMemoryError` si la répartition tente d’augmenter la mémoire utilisée, de telle sorte que *utilisée > allouée*, même si *utilisée <= max* est toujours vrai. Dans ce cas, essayez d’augmenter la taille maximale du segment de mémoire à l’aide du paramètre `-Xmx`.

  - `AppMemoryUsed` (_jvm.memory.used_) : Quantité de mémoire en octets actuellement utilisée par l’application. Pour une application Java à charge normale, cette série de métriques se présente sous la forme d’un modèle *en dents de scie*, dans lequel l’utilisation de la mémoire augmente et diminue régulièrement par petits incréments et chute soudainement, puis ce modèle se répète. Cette série de métriques est due au nettoyage de la mémoire (garbage collection) à l’intérieur de la machine virtuelle Java, où les actions de collection représentent les points bas dans le modèle en dents de scie.
    
    Cette métrique est importante pour aider à identifier les problèmes de mémoire, tels que :
    * Une explosion de mémoire au tout début.
    * La répartition de mémoire en surtension pour un chemin d’accès logique spécifique.
    * Fuites de mémoire progressives.
  Pour plus d’informations, consultez [Métriques](spring-cloud-concept-metrics.md).
  
* Si l’application ne démarre pas, vérifiez qu’elle possède des paramètres jvm valides. Si la mémoire jvm est trop élevée, le message d’erreur suivant peut s’afficher dans vos journaux :

  >« required memory 2728741K is greater than 2000M available for allocation »



Pour en savoir plus sur Azure Log Analytics, consultez [Prise en main de Log Analytics dans Azure Monitor](../azure-monitor/log-query/log-analytics-tutorial.md).

### <a name="my-application-experiences-high-cpu-usage-or-high-memory-usage"></a>Mon application connaît une utilisation élevée du processeur ou une utilisation élevée de la mémoire

Si votre application connaît une utilisation élevée de l’UC ou de la mémoire, l’une des deux propositions suivantes est vraie :
* Toutes les instances d’application connaissent une utilisation élevée de l’UC ou de la mémoire.
* Certaines instances d’application connaissent une utilisation élevée de l’UC ou de la mémoire.

Pour déterminer quelle situation s’applique, procédez comme suit :

1. Accédez à **Métriques**, puis sélectionnez **Pourcentage d’utilisation de l’UC de service** ou **Mémoire de service utilisée**.
2. Ajoutez un filtre **App=** pour spécifier l’application que vous souhaitez analyser.
3. Fractionnez les métriques par **Instance**.

Si *toutes les instances* connaissent une utilisation élevée de l’UC ou de la mémoire, vous devez effectuer un scale-out de l’application ou un scale-up de l’utilisation de l’UC ou de la mémoire. Pour plus d’informations, consultez [Didacticiel : Mettre à l’échelle une application dans Azure Spring Cloud](spring-cloud-tutorial-scale-manual.md).

Si *certaines instances* présentent une utilisation élevée de l’UC ou de la mémoire, vérifiez l’état de l’instance et son état de détection.

Pour plus d’informations, consultez [Métriques pour Azure Spring Cloud](spring-cloud-concept-metrics.md).

Si toutes les instances sont opérationnelles, accédez à Azure Log Analytics pour interroger vos journaux d’application et passez en revue votre logique de code. Cela vous permettra de déterminer si l’une d’entre elles peut perturber le partitionnement de l’échelle. Pour plus d’informations, consultez [Analyser les journaux et les métriques avec les paramètres de diagnostic](diagnostic-services.md).

Pour en savoir plus sur Azure Log Analytics, consultez [Prise en main de Log Analytics dans Azure Monitor](../azure-monitor/log-query/log-analytics-tutorial.md). Interrogez les journaux à l’aide du [langage de requête Kusto](/azure/kusto/query/).

### <a name="checklist-for-deploying-your-spring-application-to-azure-spring-cloud"></a>Check-list pour le déploiement de votre application Spring sur Azure Spring Cloud

Avant d’intégrer votre application, assurez-vous qu’elle répond aux critères suivants :

* L’application peut être exécutée localement avec la version de runtime Java spécifiée.
* La configuration de l’environnement (Processeur/RAM/instances) répond à la configuration minimale définie par le fournisseur d’applications.
* Les éléments de configuration ont les valeurs attendues. Pour plus d’informations, consultez [Serveur de configuration](spring-cloud-tutorial-config-server.md).
* Les variables d’environnement ont les valeurs attendues.
* Les paramètres JVM ont les valeurs attendues.
* Nous vous recommandons de désactiver ou supprimer les services _Config Server_ et _Spring Service Registry_ incorporés du package d’application.
* Si des ressources Azure doivent être liées via la _liaison de service_, assurez-vous que les ressources cibles sont en cours d’exécution.

## <a name="configuration-and-management"></a>Configuration et gestion

### <a name="i-encountered-a-problem-with-creating-an-azure-spring-cloud-service-instance"></a>J’ai rencontré un problème lors de la création d’une instance du service Azure Spring Cloud

Quand vous configurez une instance du service Azure Spring Cloud par le biais du Portail Azure, Azure Spring Cloud effectue la validation pour vous.

Toutefois, si vous tentez de configurer l’instance de service Azure Spring Cloud par le biais d’[Azure CLI](/cli/azure/get-started-with-azure-cli) ou du [modèle Azure Resource Manager](../azure-resource-manager/index.yml), vérifiez les points suivants :

* L’abonnement est actif.
* L’emplacement est [pris en charge](spring-cloud-faq.md) par Azure Spring Cloud.
* Le groupe de ressources pour l’instance est créé.
* Le nom de la ressource est conforme à la règle de nommage. Il ne doit contenir que des lettres minuscules, des chiffres et des traits d’union. Le premier caractère doit être une lettre. Le dernier caractère doit être une lettre ou un chiffre. La valeur doit comprendre entre 2 et 32 caractères.

Si vous souhaitez configurer l’instance de service Azure Spring Cloud à l’aide du modèle Resource Manager, consultez d’abord [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md).

Le nom de l’instance de service Azure Spring Cloud est utilisé pour demander un nom de sous-domaine sous `azureapps.io`, de sorte que la configuration échoue si le nom est en conflit avec un nom existant. Vous pouvez trouver plus de détails dans les journaux d’activité.

### <a name="i-cant-deploy-a-net-core-app"></a>Je ne parviens pas à déployer une application .NET Core

Vous ne pouvez pas télécharger un fichier *.zip* pour une application .NET Core Steeltoe à l’aide du portail Azure ou du modèle Resource Manager.

Quand vous déployez votre package d’application à l’aide d’[Azure CLI](/cli/azure/get-started-with-azure-cli), ce dernier interroge régulièrement l’avancement du déploiement, puis, à la fin, affiche son résultat.

Vérifiez que votre application est empaquetée dans le bon format *.zip*. Si elle n’est pas empaquetée correctement, le processus cessera de répondre ou un message d’erreur s’affichera.

### <a name="i-cant-deploy-a-jar-package"></a>Je ne peux pas déployer un package JAR

Vous ne pouvez pas télécharger le package source ou celui du fichier d’archive Java (JAR) à l’aide du Portail Azure ou du modèle Resource Manager.

Quand vous déployez votre package d’application à l’aide d’[Azure CLI](/cli/azure/get-started-with-azure-cli), ce dernier interroge régulièrement l’avancement du déploiement, puis, à la fin, affiche son résultat.

Si l’interrogation est interrompue, vous pouvez toujours utiliser la commande suivante pour extraire les journaux de déploiement :

`az spring-cloud app show-deploy-log -n <app-name>`

Vérifiez que votre application est empaquetée dans le bon [format JAR exécutable](https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html). Si elle n’est pas empaquetée correctement, un message d’erreur semblable au suivant s’affiche :

> « Erreur : Jarfile /Jar/38bc8ea1-a6bb-4736-8e93-e8f3b52c8714 non valide ou corrompu »

### <a name="i-cant-deploy-a-source-package"></a>Je ne peux pas déployer un package source

Vous ne pouvez pas télécharger le package source ou JAR à l’aide du Portail Azure ou du modèle Resource Manager.

Quand vous déployez votre package d’application à l’aide d’[Azure CLI](/cli/azure/get-started-with-azure-cli), ce dernier interroge régulièrement l’avancement du déploiement, puis, à la fin, affiche son résultat.

Si l’interrogation est interrompue, vous pouvez toujours utiliser la commande suivante pour extraire les journaux de compilation et de déploiement :

`az spring-cloud app show-deploy-log -n <app-name>`

Toutefois, notez qu’une instance de service Azure Spring Cloud ne peut déclencher qu’un seul travail de génération pour un package source à la fois. Pour plus d’informations, consultez [Déployer une application](spring-cloud-quickstart.md) et [Configurer un environnement intermédiaire dans Azure Spring Cloud](spring-cloud-howto-staging-environment.md).

### <a name="my-application-cant-be-registered"></a>Impossible d’inscrire mon application

Dans la plupart des cas, cette situation se produit lorsque les paramètres *Dépendances nécessaires* et *Détection de service* ne sont pas correctement configurés dans votre fichier Modèle objet du projet (POM). Une fois configuré, le point de terminaison de serveur Service Registry intégré est injecté en tant que variable d’environnement avec votre application. Les applications s’inscrivent alors automatiquement auprès du serveur Service Registry et découvrent d’autres microservices dépendants.

Attendez au moins deux minutes avant qu’une instance nouvellement inscrite ne commence à recevoir du trafic.

Si vous migrez une solution existante basée sur Spring Cloud vers Azure, vérifiez que vos instances _Service Registry_ et _Config Server_ ad hoc sont supprimées (ou désactivées) pour éviter tout conflit avec les instances gérées fournies par Azure Spring Cloud.

Vous pouvez également vérifier les journaux du client _Service Registry_ dans Azure Log Analytics. Pour plus d’informations, consultez [Analyser les journaux et les métriques avec les paramètres de diagnostic](diagnostic-services.md)

Pour en savoir plus sur Azure Log Analytics, consultez [Prise en main de Log Analytics dans Azure Monitor](../azure-monitor/log-query/log-analytics-tutorial.md). Interrogez les journaux à l’aide du [langage de requête Kusto](/azure/kusto/query/).

### <a name="i-want-to-inspect-my-applications-environment-variables"></a>Je souhaite inspecter les variables d’environnement de mon application

Les variables d’environnement informent l’infrastructure Azure Spring Cloud et permettent de vérifier qu’Azure comprend où et comment configurer les services qui composent votre application. La vérification de l’exactitude des variables d’environnement est une première étape nécessaire à la résolution des problèmes potentiels.  Vous pouvez utiliser le point de terminaison Spring Boot Actuator pour passer en revue vos variables d’environnement.  

> [!WARNING]
> Cette procédure expose vos variables d’environnement à l’aide de votre point de terminaison de test.  N’allez pas plus loin si votre point de terminaison de test est public, ou si vous avez affecté un nom de domaine à votre application.

1. Atteindre `https://<your application test endpoint>/actuator/health`.  
    - Une réponse similaire à `{"status":"UP"}` indique que le point de terminaison a été activé.
    - Si la réponse est négative, incluez la dépendance suivante dans votre fichier *POM.xml* :

        ```xml
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-actuator</artifactId>
            </dependency>
        ```

1. Une fois le point de terminaison Spring Boot Actuator activé, accédez au Portail Azure et recherchez la page de configuration de votre application.  Ajoutez une variable d’environnement avec le nom `MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE` et la valeur `*`. 

1. Redémarrez votre application.

1. Accédez à `https://<your application test endpoint>/actuator/env` et examinez la réponse.  Il doit se présenter comme suit :

    ```json
    {
        "activeProfiles": [],
        "propertySources": {,
            "name": "server.ports",
            "properties": {
                "local.server.port": {
                    "value": 1025
                }
            }
        }
    }
    ```

Recherchez le nœud enfant nommé `systemEnvironment`.  Ce nœud contient les variables d’environnement de votre application.

> [!IMPORTANT]
> N’oubliez pas d’inverser l’exposition de vos variables d’environnement avant de rendre votre application publique.  Accédez au Portail Azure, recherchez la page de configuration de votre application, puis supprimez la variable d’environnement suivante : `MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE`.

### <a name="i-cant-find-metrics-or-logs-for-my-application"></a>Impossible de trouver des métriques ou des journaux pour mon application

Accédez à **Gestion des applications** pour vous assurer que les états de l’application sont _En cours d’exécution_ et _En service_.

Vérifiez que _JMX_ est activé dans votre package d’application. Cette fonctionnalité peut être activée à l’aide de la propriété de configuration `spring.jmx.enabled=true`.  

Vérifiez si la dépendance `spring-boot-actuator` est activée dans votre package d’application et démarre correctement.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Si vos journaux d’applications peuvent être archivés dans un compte de stockage, mais ne sont pas envoyés à Azure Log Analytics, vérifiez si vous avez [correctement configuré votre espace de travail](../azure-monitor/learn/quick-create-workspace.md). Si vous utilisez un niveau gratuit d’Azure Log Analytics, notez que [le niveau gratuit ne fournit pas de contrat de niveau de service (SLA)](https://azure.microsoft.com/support/legal/sla/log-analytics/v1_3/).

## <a name="next-steps"></a>Étapes suivantes

* [Guide pratique pour diagnostiquer et résoudre les problèmes dans Azure Spring Cloud par soi-même](spring-cloud-howto-self-diagnose-solve.md)