---
title: Présentation des métriques pour Azure Spring Cloud
description: Découvrir comment passer en revue les indicateurs de performance dans Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 015aadb791172f5033bf0198caad567c9bc03221
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95974661"
---
# <a name="understand-metrics-for-azure-spring-cloud"></a>Comprendre les indicateurs de performance pour Azure Spring Cloud

Azure Metrics Explorer est un composant du portail Microsoft Azure qui permet de tracer des graphiques, de corréler visuellement des tendances et d’examiner les pics et les creux des indicateurs de performance. Utilisez l’explorateur de métriques pour examiner l’intégrité et l’utilisation de vos ressources. 

Dans Azure Spring Cloud, il existe deux points de vue pour les métriques.
* Graphiques dans la page de présentation de chaque application
* Page des indicateurs de performance courants

 ![Graphiques d’indicateurs de performance](media/metrics/metrics-1.png)

Les graphiques dans la **Vue d’ensemble** de l’application permettent des vérifications d’état rapides pour chaque application. La page des **indicateurs de performance** courants contient tous les indicateurs de performance disponibles à des fins de référence. Vous pouvez créer vos propres graphiques dans la page des indicateurs de performance courants et les épingler au tableau de bord.

## <a name="application-overview-page"></a>Page de vue d’ensemble de l’application
Dans **Apps**, sélectionnez une application pour rechercher des graphiques dans la page de vue d’ensemble.  

 ![Gestion des indicateurs de performance d’application](media/metrics/metrics-2.png)

La page **Vue d’ensemble de l’application** de chaque application, présente un graphique des métriques qui vous permet d’effectuer une vérification rapide de l’état de votre application.  

 ![Vue d’ensemble des indicateurs de performance d’application](media/metrics/metrics-3.png)

Azure Spring Cloud fournit les cinq graphiques suivants, dont les indicateurs de performance sont mis à jour toutes les minutes :

* **Erreurs de serveur Http** : Nombre d’erreurs pour les requêtes HTTP adressées à votre application
* **Données entrantes** : Octets reçus par votre application
* **Données sortantes** : Octets envoyés par votre application
* **Requêtes** : Requêtes reçues par votre application
* **Temps de réponse moyen** : Temps de réponse Moyen de votre application

Pour le graphique, vous pouvez sélectionner un intervalle de temps d’une heure à sept jours.

## <a name="common-metrics-page"></a>Page des indicateurs de performance courants

Les **indicateurs de performance** dans le volet de navigation gauche sont liés à la page des indicateurs de performance courants.

Sélectionnez tout d’abord les indicateurs de performance à afficher :

![Sélectionner l’affichage des indicateurs de performance](media/metrics/metrics-4.png)

Vous trouverez les détails de toutes les options de métriques dans la [section](#user-metrics-options) ci-dessous.

Sélectionnez ensuite le type d’agrégation de chaque indicateur de performance :

![Agrégation d’indicateurs de performance](media/metrics/metrics-5.png)

Le type d'agrégation indique comment agréger des points d’indicateurs de performance dans le graphique en fonction du temps. Un point de métrique brut est créé chaque minute, et le type de pré-agrégation au cours d’une minute est prédéfini par le type de métrique.
* Somme : Additionne toutes les valeurs comme résultat cible.
* Moyenne : Utilise la valeur moyenne de la période comme résultat cible.
* Max/Min : Utilise la valeur maximale/minimale de la période comme résultat cible.

L’intervalle de temps peut également être ajusté des 30 dernières minutes aux 30 derniers jours, ou un intervalle de temps personnalisé.

![Modification d’indicateur de performance](media/metrics/metrics-6.png)

Par défaut, la vue regroupe tous les indicateurs de performance de l’application du service Azure Spring Cloud. Les indicateurs de performance d’une application ou d’une instance peuvent être filtrés dans l’affichage.  Cliquez sur **Ajouter un filtre**, définissez la propriété sur **Application**, puis sélectionnez l’application cible que vous voulez superviser dans la zone de texte **Valeurs**. 

Vous pouvez utiliser deux types de filtres (propriétés) :
* Application : filtrer par nom d’application
* Instance : filtrer par instance d’application

![Filtres d’indicateurs de performance](media/metrics/metrics-7.png)

Vous pouvez également utiliser l’option **Appliquer la division**, qui permet de créer plusieurs lignes pour une application :

![Division d’indicateurs de performance](media/metrics/metrics-8.png)

>[!TIP]
> Vous pouvez créer vos propres graphiques dans la page des métriques et les épingler à votre **tableau de bord**. Commencez par nommer votre graphique.  Ensuite, sélectionnez **Épingler au tableau de bord dans le coin supérieur droit**. Vous pouvez maintenant vérifier votre application sur votre **tableau de bord** du portail.

## <a name="user-metrics-options"></a>Options d’indicateur de performance de l’utilisateur

Les tableaux suivants affichent les indicateurs de performance disponibles et leurs détails.

### <a name="error"></a>Error
>[!div class="mx-tdCol2BreakAll"]
>| Nom | Nom de l’indicateur de performance Spring Actuator | Unité | Détails |
>|----|----|----|------------|
>| tomcat.global.error | tomcat.global.error | Count | Nombre d’erreurs qui se sont produites dans les requêtes traitées |

### <a name="performance"></a>Performances
>[!div class="mx-tdCol2BreakAll"]
>| Nom | Nom de l’indicateur de performance Spring Actuator | Unité | Détails |
>|----|----|----|------------|
>| system.cpu.usage | system.cpu.usage | Pourcentage | Utilisation récente de l’UC pour l’ensemble du système. Cette valeur est double dans l’intervalle [0.0,1.0]. La valeur 0.0 indique que toutes les UC étaient inactives pendant la dernière période observée, alors que la valeur 1.0 indique que toutes les UC ont été actives 100 % du temps pendant la dernière période observée.|
>| process.cpu.usage | Pourcentage d’utilisation du processeur d’application | Pourcentage | Utilisation récente de l’UC pour le processus Machine virtuelle Java. Cette valeur est double dans l’intervalle [0.0,1.0]. La valeur 0.0 indique qu’aucune des UC n’exécutait de threads du processus JVM pendant la dernière période observée, alors que la valeur 1.0 indique que toutes les UC ont exécuté des threads du processus JVM 100 % du temps pendant la dernière période observée. Les threads de JVM incluent les threads d’application, ainsi que les threads internes JVM.|
>| jvm.memory.committed | jvm.memory.committed | Octets | Représente la quantité de mémoire dont la disponibilité est garantie pour une utilisation par la machine virtuelle Java. La machine virtuelle Java peut libérer de la mémoire sur le système et l’allocation peut être inférieure à la quantité initialement. validée sera toujours supérieure ou égale à la quantité utilisée. |
>| jvm.memory.used | jvm.memory.used | Octets | Représente la quantité de mémoire actuellement utilisée, en octets. |
>| jvm.memory.max | jvm.memory.max | Octets | Représente la quantité maximale de mémoire utilisable pour la gestion de la mémoire. La quantité de mémoire utilisée et validée sera toujours inférieure ou égale à Max si la quantité maximale est définie. Une allocation de mémoire peut échouer si elle tente d’augmenter la mémoire utilisée de telle sorte que used > committed même si used <= max serait toujours vrai (par exemple, lorsque la mémoire virtuelle du système est insuffisante). |
>| jvm.gc.max.data.size | jvm.gc.max.data.size | Octets | Pic d’utilisation de la mémoire du pool de mémoire d’ancienne génération depuis le démarrage de la machine virtuelle Java. |
>| jvm.gc.live.data.size | jvm.gc.live.data.size | Octets | Taille du pool de mémoire d’ancienne génération après un GC complet. |
>| jvm.gc.memory.promoted | jvm.gc.memory.promoted | Octets | Nombre d’augmentations positives de la taille du pool de mémoire d’ancienne génération avant l’application du GC jusqu’au terme de cette application. |
>| jvm.gc.memory.allocated | jvm.gc.memory.allocated | Octets | Incrémenté pour une augmentation de la taille du pool de mémoire de nouvelle génération après un GC avant le suivant. |
>| jvm.gc.pause.total.count | jvm.gc.pause (nombre total) | Count | Nombre total de GC après le démarrage de cette JMV, y compris les GC de nouvelle et d’ancienne génération. |
>| jvm.gc.pause.total.count | jvm.gc.pause (durée totale) | Millisecondes | Durée totale du GC utilisée après le démarrage de cette JMV, y compris les GC de nouvelle et d’ancienne génération. |

### <a name="performance-net"></a>Performance (.NET)

>[!div class="mx-tdCol2BreakAll"]
>| Nom | Nom de l’indicateur de performance Spring Actuator | Unité | Détails |
>|------|-----------------------------|------|---------|
>| Utilisation de l’UC       | cpu-usage      | Millisecondes | Temps d’utilisation du processeur par le processus. |
>| Plage de travail     | working-set    | Mo    | Quantité de plage de travail utilisée par le processus. |
>| Taille des segments de mémoire du récupérateur de mémoire    | gc-heap-size   | Mo    | Taille totale des segments de mémoire rapportée par le récupérateur de mémoire. |
>| Nombre de nettoyages de mémoire Gén 0  | gen-0-gc-count | Count        | Nombre de nettoyages de mémoire de Génération 0 par seconde. |
>| Nombre de nettoyages de mémoire Gén 1  | gen-1-gc-count | Count        | Nombre de nettoyages de mémoire de Génération 1 par seconde. |
>| Nombre de nettoyages de mémoire Gén 2  | gen-2-gc-count | Count        | Nombre de nettoyages de mémoire de Génération 2 par seconde. |
>| Taille du tas de la génération 0 | gen-0-size     | Octets        | Taille des segments de mémoire de Génération 0. |
>| Taille du tas de la génération 1 | gen-1-size     | Octets        | Taille des segments de mémoire de Génération 1. |
>| Taille du tas de la génération 2 | gen-2-size     | Octets        | Taille des segments de mémoire de Génération 2. |
>| Taille du tas des objets volumineux   | loh-size       | Octets        | Taille du tas des objets volumineux. |
>| Taux d’allocation | alloc-rate     | Octets        | Nombre d’octets alloués par seconde. |
>| Nombre d’assemblys  | assembly-count | Count        | Nombre d’assemblys chargés. |
>| Nombre d’exceptions | exception-count | Count       | Nombre d’exceptions par seconde. |
>| Nombre de threads du pool de threads      | threadpool-thread-count              | Count | Nombre de threads dans un pool. |
>| Nombre de contentions de verrouillage du moniteur | monitor-lock-contention-count        | Count | Nombre de fois par seconde où il y a eu contention lors d’une tentative de suppression de verrou d’un moniteur. |
>| Longueur de file d’attente du pool de threads      | threadpool-queue-length              | Count | Longueur de la file d’attente des éléments de travail du pool de threads. |
>| Nombre d’éléments terminés dans le pool de threads | threadpool-completed-items-count | Count | Nombre d’éléments de travail terminés dans le pool de threads. |
>| Nombre de minuteurs actifs               | active-timer-count               | Count | Nombre de minuteurs actuellement actifs. Un minuteur actif est un minuteur défini pour sonner à un moment donné et qui n’a pas encore été annulé. |

Pour plus d’informations, consultez [compteurs dotnet](/dotnet/core/diagnostics/dotnet-counters).

### <a name="request"></a>Requête
>[!div class="mx-tdCol2BreakAll"]
>| Nom | Nom de l’indicateur de performance Spring Actuator | Unité | Détails |
>|----|----|----|------------|
>| tomcat.global.sent | tomcat.global.sent | Octets | Quantité de données envoyées par le serveur web Tomcat |
>| tomcat.global.received | tomcat.global.received | Octets | Quantité de données reçues par le serveur web Tomcat |
>| tomcat.global.request.total.count | tomcat.global.request (nombre total) | Count | Nombre total de requêtes traitées par le serveur web Tomcat |
>| tomcat.global.request.max | tomcat.global.request.max | Millisecondes | Durée maximale de traitement d’une requête par le serveur web Tomcat |

### <a name="request-net"></a>Requête (.NET)

>[!div class="mx-tdCol2BreakAll"]
>| Nom | Nom de l’indicateur de performance Spring Actuator | Unité | Détails |
>|------|-----------------------------|------|---------|
>| Demandes par seconde | requests-per-second | Count | Le taux de demandes. |
>| Nombre total de requêtes | total-requests | Count | Nombre total de requêtes. |
>| Requêtes en cours | current-requests | Count | Nombre de requêtes en cours. |
>| Demandes ayant échoué | failed-requests | Count | Nombre d’échecs de requêtes. |

Pour plus d’informations, consultez [compteurs dotnet](/dotnet/core/diagnostics/dotnet-counters).

### <a name="session"></a>session
>[!div class="mx-tdCol2BreakAll"]
>| Nom | Nom de l’indicateur de performance Spring Actuator | Unité | Détails |
>|----|----|----|------------|
>| tomcat.sessions.active.max | tomcat.sessions.active.max | Count | Nombre maximal de sessions actives simultanément |
>| tomcat.sessions.alive.max | tomcat.sessions.alive.max | Millisecondes | Durée maximale (en secondes) pendant laquelle une session ayant expiré a été active |
>| tomcat.sessions.created | tomcat.sessions.created | Count | Nombre de sessions créées |
>| tomcat.sessions.expired | tomcat.sessions.expired | Count | Nombre de sessions qui ont expiré |
>| tomcat.sessions.rejected | tomcat.sessions.rejected | Count | Nombre de sessions qui n’ont pas été créées parce que le nombre maximal de sessions actives a été atteint. |
>| tomcat.sessions.active.current | tomcat.sessions.active.current | Count | Nombre de sessions actives Tomcat |

## <a name="see-also"></a>Voir aussi

* [Démarrage rapide : Supervision des applications Azure Spring Cloud avec les journaux, les métriques et le suivi](spring-cloud-quickstart-logs-metrics-tracing.md)

* [Bien démarrer avec Azure Metrics Explorer](../azure-monitor/platform/metrics-getting-started.md)

* [Analyser les journaux et les indicateurs de performance avec les paramètres de diagnostic](./diagnostic-services.md)

## <a name="next-steps"></a>Étapes suivantes

* [Tutoriel : Superviser les ressources Spring Cloud avec des alertes et des groupes d’actions](./spring-cloud-tutorial-alerts-action-groups.md)

* [Quotas et plans de service pour Azure Spring Cloud](./spring-cloud-quotas.md)