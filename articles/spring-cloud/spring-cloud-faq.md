---
title: Questions fréquentes (FAQ) sur Azure Spring Cloud | Microsoft Docs
description: Cette page répond aux questions fréquentes à propos d’Azure Spring Cloud.
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: brendm
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 99246ecee27f7ded693e2a797f8e98480e350983
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96854527"
---
# <a name="azure-spring-cloud-faq"></a>Questions fréquentes sur Azure Spring Cloud

Cette page répond aux questions fréquentes à propos d’Azure Spring Cloud.

## <a name="general"></a>Général

### <a name="why-azure-spring-cloud"></a>Pourquoi Azure Spring Cloud ?

Azure Spring Cloud fournit une plateforme PaaS pour les développeurs Spring Cloud. Azure Spring Cloud gère votre infrastructure d’applications pour vous permettre de vous concentrer sur le code et la logique métier des applications. Les principales fonctionnalités intégrées à Azure Spring Cloud incluent Eureka, le serveur de configuration, le serveur Service Registry, le service Pivotal Build, les déploiements bleus-verts, etc. Ce service permet également aux développeurs de lier leurs applications à d’autres services Azure, tels que Azure Cosmos DB, Azure Database pour MySQL et Azure Cache pour Redis.

Azure Spring Cloud améliore l’expérience de diagnostic des applications pour les développeurs et les opérateurs en intégrant Azure Monitor, Application Insights et Log Analytics.

### <a name="how-secure-is-azure-spring-cloud"></a>Qu’en est-il de la sécurité d’Azure Spring Cloud ?

La sécurité et la confidentialité figurent parmi les principales priorités pour les clients Azure et Azure Spring Cloud. Azure garantit que seul le client a accès aux données, aux journaux ou aux configurations des applications en chiffrant de façon sécurisée toutes ces données. 

* Les instances de service dans Azure Spring Cloud sont isolées les unes des autres.
* Azure Spring Cloud offre une gestion complète des certificats et de TLS/SSL.
* Les correctifs de sécurité critiques pour les runtimes OpenJDK et Spring Cloud sont appliqués à Azure Spring Cloud dès que possible.

### <a name="in-which-regions-is-azure-spring-cloud-available"></a>Dans quelles régions Azure Spring Cloud est-il disponible ?

USA Est, USA Est 2, USA Centre, USA Centre Sud, USA Centre Nord, USA Ouest, USA Ouest 2, Europe Ouest, Europe Nord, Royaume-Uni Sud, Asie Sud-Est, Australie Est, Canada Centre et Émirats arabes unis Nord.

### <a name="is-any-customer-data-stored-outside-of-the-specified-region"></a>Les données client sont-elles stockées en dehors de la région spécifiée ?

Azure Spring Cloud est un service régional. Toutes les données client dans Azure Spring Cloud sont stockées dans plusieurs régions au sein de la même zone géographique de la région spécifiée à des fins de redondance. Pour en savoir plus sur la zone géographique et la région, consultez [Résidence des données dans Azure](https://azure.microsoft.com/global-infrastructure/data-residency/).

### <a name="what-are-the-known-limitations-of-azure-spring-cloud"></a>Quelles sont les limitations connues d’Azure Spring Cloud ?

Azure Spring Cloud présente les limitations connues suivantes :
    
* `spring.application.name` sera remplacé par le nom de l’application utilisé pour créer chaque application.
* La valeur par défaut de `server.port` est le port 1025. Si une autre valeur est appliquée, elle est remplacée. Respectez également ce paramètre et ne spécifiez pas le port du serveur dans votre code.
* Le portail Azure et les modèles Azure Resource Manager ne prennent pas en charge le chargement de packages d’application. Vous pouvez charger des packages d’application uniquement en déployant l’application via Azure CLI.

### <a name="what-pricing-tiers-are-available"></a>Quels niveaux tarifaires sont disponibles ? 
Lequel dois-je utiliser et quelles sont les limites de chaque niveau ?
* Azure Spring Cloud offre deux niveaux tarifaires : De base et Standard. Le niveau de base est conçu pour le développement/test et l’évaluation d’Azure Spring Cloud. Le niveau standard est optimisé pour exécuter le trafic de production à usage général. Consultez les [détails de la tarification d’Azure Spring Cloud](https://azure.microsoft.com/pricing/details/spring-cloud/) pour connaître les limites et la comparaison au niveau des fonctionnalités.

### <a name="how-can-i-provide-feedback-and-report-issues"></a>Comment puis-je donner un feedback et signaler des problèmes ?

Si vous rencontrez des problèmes avec Azure Spring Cloud, créez une [demande de support Azure](../azure-portal/supportability/how-to-create-azure-support-request.md). Pour soumettre une demande de fonctionnalité ou fournir des commentaires, accédez à la page de [commentaires sur Azure](https://feedback.azure.com/forums/34192--general-feedback).

## <a name="development"></a>Développement

### <a name="i-am-a-spring-cloud-developer-but-new-to-azure-what-is-the-quickest-way-for-me-to-learn-how-to-develop-an-azure-spring-cloud-application"></a>Je suis développeur Spring Cloud, mais je débute sur Azure. Quel est le moyen le plus rapide pour apprendre à développer une application Azure Spring Cloud ?

Pour connaître le moyen le plus rapide de se lancer avec Azure Spring Cloud, suivez les instructions du [guide de démarrage rapide : Lancer une application Azure Spring Cloud en utilisant le portail Azure](spring-cloud-quickstart.md).

::: zone pivot="programming-language-java"
### <a name="what-java-runtime-does-azure-spring-cloud-support"></a>Quel runtime Java est pris en charge par Azure Spring Cloud ?

Azure Spring Cloud prend en charge Java 8 et 11. Voir [Runtime Java et versions de système d’exploitation](#java-runtime-and-os-versions)

### <a name="is-spring-boot-24x-supported"></a>Spring Boot 2.4.x est-il pris en charge ?
Nous avons identifié un problème avec Spring Boot 2.4 ; nous travaillons actuellement à sa résolution avec la communauté Spring. En attendant, veuillez inclure ces deux dépendances pour activer l’authentification TLS entre vos applications et Eureka.

```xml
<dependency> 
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-client</artifactId>
    <version>1.19.4</version>
</dependency>
<dependency>
    <groupId>com.sun.jersey.contribs</groupId>
    <artifactId>jersey-apache-client4</artifactId>
    <version>1.19.4</version>
</dependency>
```

::: zone-end

### <a name="where-can-i-view-my-spring-cloud-application-logs-and-metrics"></a>Où puis-je voir les journaux et les métriques de mon application Spring Cloud ?

Vous trouverez les métriques sous l’onglet Vue d’ensemble de l’application et sous l’onglet [Azure Monitor](../azure-monitor/platform/data-platform-metrics.md#metrics-explorer).

Azure Spring Cloud prend en charge l’exportation des journaux et des métriques de votre application Spring Cloud vers Stockage Azure, EventHub et [Log Analytics](../azure-monitor/platform/data-platform-logs.md). Le nom de la table dans Log Analytics est *AppPlatformLogsforSpring*. Pour savoir comment l’activer, consultez [Services de diagnostic](diagnostic-services.md).

### <a name="does-azure-spring-cloud-support-distributed-tracing"></a>Azure Spring Cloud prend-il en charge le suivi distribué ?

Oui. Pour plus d’informations, consultez [Didacticiel : Utiliser le suivi distribué avec Azure Spring Cloud](spring-cloud-tutorial-distributed-tracing.md).

::: zone pivot="programming-language-java"
### <a name="what-resource-types-does-service-binding-support"></a>Quels sont les types de ressources pris en charge par la liaison de service ?

Trois services sont actuellement pris en charge :
* Azure Cosmos DB
* Azure Database pour MySQL
* Cache Azure pour Redis.
::: zone-end

### <a name="can-i-view-add-or-move-persistent-volumes-from-inside-my-applications"></a>Puis-je voir, ajouter ou déplacer des volumes persistants depuis mes applications ?

Oui.

### <a name="how-many-outbound-public-ip-addresses-does-an-azure-spring-cloud-instance-have"></a>Combien d’adresses IP publiques sortantes une instance Azure Spring Cloud a-t-elle ?

Le nombre d’adresses IP publiques sortantes peut varier en fonction des niveaux et d’autres facteurs. 

| Type d’instance Azure Spring Cloud | Nombre par défaut d’adresses IP publiques sortantes |
| -------------------------------- | ---------------------------------------------- |
| Instances de niveau de base             | 1                                              |
| Instances de niveau standard          | 2                                              |
| Instances d’injection de réseau virtuel         | 1                                              |


### <a name="can-i-increase-the-number-of-outbound-public-ip-addresses"></a>Puis-je augmenter le nombre d’adresses IP publiques sortantes ?

Oui, vous pouvez ouvrir un [ticket de support](https://azure.microsoft.com/support/faq/) pour demander davantage d’adresses IP publiques sortantes.

### <a name="when-i-deletemove-an-azure-spring-cloud-service-instance-will-its-extension-resources-be-deletedmoved-as-well"></a>Lorsque je supprime/déplace une instance de service Azure Spring Cloud, ses ressources d’extension seront-elles également supprimées/déplacées ?

Cela dépend des logiques des fournisseurs de ressources qui possèdent les ressources d’extension. Les ressources d’extension d’une instance de `Microsoft.AppPlatform` n’appartiennent pas au même espace de noms ; par conséquent, les comportements varient en fonction des fournisseurs de ressources. Par exemple, l’opération de suppression/déplacement n’est pas effectuée en cascade vers les ressources de **paramètres de diagnostic**. Si une nouvelle instance Azure Spring Cloud est configurée avec le même ID de ressource que celui qui a été supprimé, ou si l’instance Azure Spring Cloud précédente est déplacée, les ressources **paramètres de diagnostic** précédentes continuent de l’étendre.

Vous pouvez supprimer les paramètres de diagnostic du Cloud Spring à l’aide d’Azure CLI :

```azurecli
 az monitor diagnostic-settings delete --name $diagnosticSettingName --resource $azureSpringCloudResourceId
```

::: zone pivot="programming-language-java"
## <a name="java-runtime-and-os-versions"></a>Runtime Java et versions de système d’exploitation

### <a name="which-versions-of-java-runtime-are-supported-in-azure-spring-cloud"></a>Quelles sont les versions du runtime Java prises en charge dans Azure Spring Cloud ?

Azure Spring Cloud prend en charge les versions Java LTS avec les builds les plus récentes (actuellement juin 2020) ; Java 8 et Java 11 sont pris en charge. Consulter [Installer le JDK pour Azure et Azure Stack](/azure/developer/java/fundamentals/java-jdk-install)

### <a name="who-built-these-java-runtimes"></a>Qui a créé ces runtimes Java ?

Azul Systems. Les builds JDK Azul Zulu for Azure - Enterprise Edition sont une distribution gratuite, multiplateforme et prête pour la production d’OpenJDK pour Azure et Azure Stack pris en charge par Microsoft et Azul Systems. Elles contiennent tous les composants nécessaires pour générer et exécuter des applications Java SE.

### <a name="how-often-will-java-runtimes-get-updated"></a>À quelle fréquence les runtimes Java seront-ils mis à jour ?

Les versions JDK LTS et MTS disposent de mises à jour de sécurité et de correctifs de bogues tous les trimestres, ainsi que de mises à jour et correctifs hors bande critiques en fonction des besoins. Cette prise en charge inclut le rétroportage vers Java 7 et 8 des mises à jour de sécurité ainsi que des correctifs de bogues signalés dans les versions plus récentes de Java, comme Java 11.

### <a name="how-long-will-java-8-and-java-11-lts-versions-be-supported"></a>Combien de temps les versions Java 8 et Java 11 LTS seront-elles prises en charge ?

Consultez [Prise en charge à long terme de Java pour Azure et Azure Stack](/azure/developer/java/fundamentals/java-jdk-long-term-support).

* Java 8 LTS sera pris en charge jusqu’à décembre 2030.
* Java 11 LTS sera pris en charge jusqu’à septembre 2027.

### <a name="how-can-i-download-a-supported-java-runtime-for-local-development"></a>Comment télécharger un runtime Java pris en charge pour le développement local ?

Consultez [Installer le JDK pour Azure et Azure Stack](/azure/developer/java/fundamentals/java-jdk-install).

### <a name="what-is-the-retire-policy-for-older-java-runtimes"></a>Quelle est la stratégie de mise hors service pour les anciens runtimes Java ?

Un avis public sera envoyé 12 mois avant la mise hors service de l’ancienne version du runtime. Vous aurez 12 mois pour migrer vers une version ultérieure.

* Les administrateurs d’abonnements recevront une notification par courrier électronique lorsque nous mettrons hors service une version de Java.
* Les informations de mise hors service seront publiées dans la documentation.

### <a name="how-can-i-get-support-for-issues-at-the-java-runtime-level"></a>Comment puis-je obtenir un support pour les problèmes au niveau du runtime Java ?

Vous pouvez ouvrir un ticket de support auprès du Support Azure.  Consultez [Création d’une demande de support Azure](../azure-portal/supportability/how-to-create-azure-support-request.md).

### <a name="what-is-the-operation-system-to-run-my-apps"></a>Quel est le système d’exploitation pour exécuter mes applications ?

La version la plus récente d’Ubuntu LTS est utilisée ; actuellement [Ubuntu 20.04 LTS (Focal Fossa)](https://releases.ubuntu.com/focal/) est le système d’exploitation par défaut.

### <a name="how-often-are-os-security-patches-applied"></a>À quelle fréquence les correctifs de sécurité du système d’exploitation seront-ils appliqués ?

Les correctifs de sécurité applicables à Azure Spring Cloud sont déployés tous les mois.
Les correctifs de sécurité critiques (score CVE > = 9) applicables à Azure Spring Cloud sont déployés dès que possible.
::: zone-end

## <a name="deployment"></a>Déploiement

### <a name="does-azure-spring-cloud-support-blue-green-deployment"></a>Azure Spring Cloud prend-il en charge le déploiement bleu-vert ?
Oui. Pour plus d’informations, consultez [Configuration de votre environnement intermédiaire](spring-cloud-howto-staging-environment.md).

### <a name="can-i-access-kubernetes-to-manipulate-my-application-containers"></a>Puis-je accéder à Kubernetes pour manipuler mes conteneurs d’application ?

Non.  Azure Spring Cloud rend l’architecture sous-jacente abstraite pour le développeur, ce qui vous permet de vous concentrer sur le code et la logique métier de l’application.

### <a name="does-azure-spring-cloud-support-building-containers-from-source"></a>Azure Spring Cloud prend-il en charge la création de conteneurs à partir d’une source ?

Oui. Pour plus d’informations, consultez [Lancer votre application Spring Cloud à partir du code source](spring-cloud-quickstart.md).

### <a name="does-azure-spring-cloud-support-autoscaling-in-app-instances"></a>Azure Spring Cloud prend-il en charge la mise à l’échelle automatique dans les instances d’application ?

Oui.  Pour plus d’informations, consultez [Configuration de la mise à l’échelle automatique](spring-cloud-tutorial-setup-autoscale.md).

::: zone pivot="programming-language-java"
### <a name="what-are-the-best-practices-for-migrating-existing-spring-cloud-microservices-to-azure-spring-cloud"></a>Quelles sont les bonnes pratiques pour la migration de microservices Spring Cloud existants vers Azure Spring Cloud ?

À mesure que vous migrez des microservices Spring Cloud existants vers Azure Spring Cloud, il est judicieux d’observer les meilleures pratiques suivantes :
* toutes les dépendances de l’application doivent être résolues.
* Préparez les entrées, les variables d’environnement et les paramètres JVM de votre configuration pour pouvoir les comparer au déploiement dans Azure Spring Cloud.
* Si vous voulez utiliser la liaison de service, parcourez vos services Azure et assurez-vous d’avoir défini les autorisations d’accès appropriées.
* Nous vous recommandons de supprimer ou de désactiver les services incorporés qui peuvent entrer en conflit avec les services managés par Azure Spring Cloud, comme notre service de découverte de service, le serveur de configuration, etc.
* Nous vous recommandons d’utiliser des bibliothèques Pivotal Spring officielles et stables. Les versions non officielles, bêta ou dupliquées (forked) des bibliothèques Pivotal Spring ne bénéficient pas de prise en charge des contrats SLA.

Après la migration, supervisez les métriques de processeur/mémoire et le trafic réseau pour vérifier que les instances d’application sont mises à l’échelle de façon appropriée.
::: zone-end

::: zone pivot="programming-language-csharp"
## <a name="net-core-versions"></a>Version de .NET Core

### <a name="which-net-core-versions-are-supported"></a>Quelles sont les versions de .NET Core prises en charge ?

.NET Core 3.1 et versions ultérieures.

### <a name="how-long-will-net-core-31-be-supported"></a>Combien de temps .NET Core 3.1 sera-t-il pris en charge ?

Jusqu’au 3 décembre 2022. Voir la [stratégie de prise en charge de .NET Core](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).
::: zone-end


## <a name="troubleshooting"></a>Dépannage

### <a name="what-are-the-impacts-of-service-registry-rarely-unavailable"></a>Quels sont les impacts du registre de service rarement indisponibles ?

Dans certains scénarios rarement survenus, vous pouvez constater des erreurs telles que 
```
RetryableEurekaHttpClient: Request execution failure with status code 401; retrying on another server if available
```
à partir de vos journaux d’applications. Ce problème a été introduit par Spring Framework avec un taux très faible en raison d’une instabilité du réseau ou d’autres problèmes réseau. 

Il ne doit y avoir aucun impact sur l’expérience utilisateur, le client Eureka a à la fois une stratégie de pulsation et de nouvelle tentative pour s’en occuper. Vous pouvez le considérer comme une erreur temporaire et l’ignorer en toute sécurité.

Nous allons améliorer cette partie et éviter cette erreur des applications des utilisateurs en un peu plus tard.


## <a name="next-steps"></a>Étapes suivantes

Si vous avez d’autres questions, consultez le [Guide de résolution des problèmes d’Azure Spring Cloud](spring-cloud-troubleshoot.md).
