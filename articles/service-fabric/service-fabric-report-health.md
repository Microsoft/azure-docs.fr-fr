---
title: Ajout de rapports d’intégrité Service Fabric personnalisés
description: Explique comment envoyer des rapports d’intégrité personnalisés à des entités d’intégrité Azure Service Fabric. Fournit des recommandations pour la conception et la mise en œuvre de rapports d’intégrité de qualité.
author: georgewallace
ms.topic: conceptual
ms.date: 2/28/2018
ms.author: gwallace
ms.custom: devx-track-csharp
ms.openlocfilehash: 6df434610a8f595ecca7f16e31f8a302373b02f9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001862"
---
# <a name="add-custom-service-fabric-health-reports"></a>Ajout de rapports d’intégrité Service Fabric personnalisés
Azure Service Fabric introduit un [modèle d’intégrité](service-fabric-health-introduction.md) conçu pour signaler des conditions de cluster et d’application défectueuses sur des entités spécifiques. Le modèle d’intégrité utilise des **rapporteurs d’intégrité** (composants système et agents de surveillance). L’objectif consiste en un diagnostic et une réparation simples et rapides. Les enregistreurs du service doivent penser en amont à l’intégrité. Toute condition pouvant avoir une incidence sur l’intégrité doit être signalée, surtout si cela peut aider à signaler des problèmes proches de la racine. Pour ce qui est du débogage et des investigations, les informations sur l’intégrité peuvent faire gagner du temps et économiser des efforts. L’utilité est particulièrement flagrante une fois que le service est en cours d’exécution, à l’échelle dans le cloud (privé ou Azure).

Les rapporteurs Service Fabric surveillent les conditions identifiées qui présentent un intérêt. Ils dressent un rapport sur ces conditions en fonction de leur vue locale. Le [magasin d’intégrité](service-fabric-health-introduction.md#health-store) agrège les données d’intégrité envoyées par tous les rapporteurs pour déterminer si les entités sont globalement intègres. Le modèle doit être riche, flexible et facile à utiliser. La qualité des rapports d’intégrité détermine la précision de la vue d’intégrité du cluster. Des faux positifs qui indiquent erronément des problèmes d’intégrité peuvent avoir une incidence négative sur les mises à niveau ou autres services utilisant des données d’intégrité. Les services de réparation et mécanismes d’alertes sont des exemples de services de ce type. Par conséquent, une réflexion est nécessaire pour générer des rapports qui capturent des conditions pertinentes de la meilleure façon possible.

Pour concevoir et implémenter des rapports d’intégrité, les agents de surveillance et les composants système doivent :

* Définir la condition qui les intéresse, la façon dont elle est surveillée et l’impact sur la fonctionnalité du cluster ou de l’application. Choisir la propriété du rapport d’intégrité et l’état d’intégrité, en fonction de ces informations.
* Déterminer l’ [entité](service-fabric-health-introduction.md#health-entities-and-hierarchy) à laquelle le rapport s’applique.
* Déterminer l’emplacement où le rapport est généré, soit à partir du service, soit à partir d’un agent de surveillance interne ou externe.
* Définir une source utilisée pour identifier le rapporteur.
* Choisir une stratégie de création de rapports, périodiquement ou selon les transitions. La méthode recommandée est la création périodique, car elle requiert un code plus simple et est moins sujette aux erreurs.
* Déterminer la durée pendant laquelle le rapport sur les conditions défectueuses doit rester dans le magasin d’intégrité et comment il doit être effacé. Grâce à ces informations, vous pouvez définir la valeur TTL du rapport et le comportement de suppression à l’expiration.

Comme mentionné auparavant, la création de rapports peut être effectuée à partir des éléments suivants :

* Le réplica de service Service Fabric surveillé.
* Les agents de surveillance internes déployés en tant que services Service Fabric (par exemple, un service Service Fabric sans état, qui surveille des conditions et émet des rapports). Les agents de surveillance peuvent être déployés sur tous les nœuds ou ils peuvent être apparentés au service surveillé.
* Les agents de surveillance internes qui s’exécutent sur les nœuds Service Fabric, mais qui ne sont *pas* implémentés en tant que services de Service Fabric.
* Les agents de surveillance externes qui sondent la ressource à partir de l’ *extérieur* du cluster Service Fabric (par exemple, un service de surveillance de type Gomez).

> [!NOTE]
> Dès le départ, le cluster comprend des rapports d’intégrité envoyés par les composants système. Pour en savoir plus, voir [Utilisation des rapports d’intégrité système pour la résolution des problèmes](service-fabric-understand-and-troubleshoot-with-system-health-reports.md). Les rapports utilisateur doivent être envoyés sur des [entités d’intégrité](service-fabric-health-introduction.md#health-entities-and-hierarchy) déjà créées par le système.
> 
> 

Lorsque la conception de la création des rapports d’intégrité est claire, l’envoi de rapports d’intégrité est simple. Vous pouvez utiliser [FabricClient](/dotnet/api/system.fabric.fabricclient) pour créer un rapport sur l’intégrité, si le cluster n’est pas [sécurisé](service-fabric-cluster-security.md) ou si le client d’infrastructure possède des privilèges d’administrateur. La génération de rapports peut se faire au moyen de l’API avec [FabricClient.HealthManager.ReportHealth](/dotnet/api/system.fabric.fabricclient.healthclient.reporthealth), via PowerShell ou REST. Des boutons de configuration regroupent les rapports afin d’améliorer les performances.

> [!NOTE]
> L’intégrité de rapport est synchrone, et représente uniquement le travail de validation côté client. Même si le rapport est accepté par le client de contrôle d’intégrité ou les objets `Partition` ou `CodePackageActivationContext`, cela ne signifie pas qu’il est appliqué dans le magasin. Il est envoyé de manière asynchrone et éventuellement regroupé avec d’autres rapports. Le traitement sur le serveur peut encore échouer, car un numéro de séquence est périmé, l’entité sur laquelle le rapport doit être appliqué a été supprimée, etc.
> 
> 

## <a name="health-client"></a>Client de contrôle d’intégrité
Les rapports d’intégrité sont envoyés au gestionnaire d’intégrité par le biais d’un client de contrôle d’intégrité, qui réside dans le client Fabric. Le gestionnaire de contrôle d’intégrité enregistre les rapports dans le magasin d’intégrité. Le client de contrôle d’intégrité peut être configuré à l’aide des paramètres suivants :

* **HealthReportSendInterval** : délai qui s’écoule entre le moment où le rapport est ajouté au client et celui où il est envoyé au gestionnaire d’intégrité. Il est utilisé pour regrouper les rapports dans un seul message, au lieu d’envoyer un message pour chaque rapport. Ce regroupement améliore les performances. Valeur par défaut : 30 secondes.
* **HealthReportRetrySendInterval** : intervalle auquel le client de contrôle d’intégrité retourne les rapports d’intégrité cumulés au gestionnaire d’intégrité. Valeur par défaut : 30 secondes, minimum : 1 seconde.
* **HealthOperationTimeout** : délai d’expiration d’un message de rapport envoyé au gestionnaire d’intégrité. Si un message expire, le client de contrôle d’intégrité réessaie de l’envoyer jusqu’à ce que le gestionnaire d’intégrité confirme que le rapport a été traité. Par défaut : deux minutes.

> [!NOTE]
> Lorsque les rapports sont traités par lot, le client Fabric doit être maintenu actif pendant au moins l’intervalle HealthReportSendInterval pour s’assurer qu’ils sont envoyés. Si le message est perdu ou si le gestionnaire d’intégrité n’est pas en mesure de d’appliquer les rapports en raison d’erreurs transitoires, le client Fabric doit être maintenu actif plus longtemps pour lui donner une chance de réessayer.
> 
> 

La mise en mémoire tampon sur le client prend en compte l’unicité des rapports. Par exemple, si un rapporteur incorrect génère 100 rapports par seconde sur la même propriété de la même entité, ces rapports sont remplacés par la dernière version. La file d’attente du client contient tout au plus un rapport de ce type. Si le traitement par lots est configuré, les rapports envoyés au gestionnaire d’intégrité sont au nombre de un par intervalle d’envoi. Il s’agit du dernier rapport ajouté, qui reflète l’état le plus récent de l’entité.
Spécifiez les paramètres de configuration lors de la création de l’élément `FabricClient`, via la transmission de [FabricClientSettings](/dotnet/api/system.fabric.fabricclientsettings) avec les valeurs souhaitées pour les entrées relatives à l’intégrité.

L’exemple suivant permet de créer un client Fabric et d’indiquer que les rapports doivent être envoyés dès qu’ils sont ajoutés. En cas de délais d’attente et d’erreurs pouvant donner lieu à une nouvelle tentative, les nouvelles tentatives ont lieu toutes les 40 secondes.

```csharp
var clientSettings = new FabricClientSettings()
{
    HealthOperationTimeout = TimeSpan.FromSeconds(120),
    HealthReportSendInterval = TimeSpan.FromSeconds(0),
    HealthReportRetrySendInterval = TimeSpan.FromSeconds(40),
};
var fabricClient = new FabricClient(clientSettings);
```

Nous recommandons de conserver le paramètre par défaut du client Fabric, qui définit `HealthReportSendInterval` sur 30 secondes. Ce paramètre garantit des performances optimales en raison du traitement par lots. Pour les rapports critiques qui doivent être envoyés dès que possible, utilisez `HealthReportSendOptions` avec Immediate `true` dans l’API [FabricClient.HealthClient.ReportHealth](/dotnet/api/system.fabric.fabricclient.healthclient.reporthealth). Les rapports immédiats ignorent l’intervalle du traitement par lots. Utilisez cet indicateur avec précaution ; nous souhaitons tirer parti du traitement par lots du client de contrôle d’intégrité chaque fois que cela est possible. L’envoi immédiat est également utile lors de la fermeture du client Fabric (par exemple, quand le processus a déterminé l’état non valide et doit s’arrêter afin d’éviter des effets secondaires). Il garantit un envoi au mieux des rapports cumulés. Lorsqu’un rapport est ajouté avec l’indicateur « Immediate », le client de contrôle d’intégrité constitue un lot de tous les rapports cumulés depuis le dernier envoi.

Il est possible de spécifier les mêmes paramètres lors de la création d’une connexion à un cluster via Powershell. L’exemple suivant démarre une connexion à un cluster local :

```powershell
PS C:\> Connect-ServiceFabricCluster -HealthOperationTimeoutInSec 120 -HealthReportSendIntervalInSec 0 -HealthReportRetrySendIntervalInSec 40
True

ConnectionEndpoint   :
FabricClientSettings : {
                       ClientFriendlyName                   : PowerShell-1944858a-4c6d-465f-89c7-9021c12ac0bb
                       PartitionLocationCacheLimit          : 100000
                       PartitionLocationCacheBucketCount    : 1024
                       ServiceChangePollInterval            : 00:02:00
                       ConnectionInitializationTimeout      : 00:00:02
                       KeepAliveInterval                    : 00:00:20
                       HealthOperationTimeout               : 00:02:00
                       HealthReportSendInterval             : 00:00:00
                       HealthReportRetrySendInterval        : 00:00:40
                       NotificationGatewayConnectionTimeout : 00:00:00
                       NotificationCacheUpdateTimeout       : 00:00:00
                       }
GatewayInformation   : {
                       NodeAddress                          : localhost:19000
                       NodeId                               : 1880ec88a3187766a6da323399721f53
                       NodeInstanceId                       : 130729063464981219
                       NodeName                             : Node.1
                       }
```

Tout comme l’API, les rapports peuvent être envoyés immédiatement à l’aide du commutateur `-Immediate`, quelle que soit la valeur de `HealthReportSendInterval`.

Pour REST, les rapports sont envoyés à la passerelle Service Fabric qui a un client Fabric interne. Par défaut, ce client est configuré pour envoyer des rapports traités par lot toutes les 30 secondes. Vous pouvez modifier l’intervalle de traitement de lots à l’aide du paramètre de configuration de cluster `HttpGatewayHealthReportSendInterval` défini sur `HttpGateway`. Comme indiqué, une meilleure solution consiste à envoyer les rapports avec `Immediate` true. 

> [!NOTE]
> Pour vous assurer que des services non autorisés ne créent pas de rapports de contrôle d’intégrité sur les entités du cluster, configurez le serveur pour qu’il accepte uniquement les demandes de clients sécurisés. La sécurité doit être activée pour l’élément `FabricClient` afin de permettre une communication avec le cluster (par exemple, avec l’authentification de certificat ou Kerberos). En savoir plus sur la [sécurité du cluster](service-fabric-cluster-security.md).
> 
> 

## <a name="report-from-within-low-privilege-services"></a>Création de rapports depuis les services à faibles privilèges
Si les services Service Fabric n’ont aucun accès administrateur au cluster, vous pouvez créer des rapports sur l’intégrité des entités à partir du contexte actuel via `Partition` ou `CodePackageActivationContext`.

* Pour les services sans état, utilisez [IStatelessServicePartition.ReportInstanceHealth](/dotnet/api/system.fabric.istatelessservicepartition.reportinstancehealth) pour créer un rapport sur l’instance de service actuelle.
* Pour les services avec état, utilisez [IStatefulServicePartition.ReportReplicaHealth](/dotnet/api/system.fabric.istatefulservicepartition.reportreplicahealth) pour créer un rapport sur le réplica en cours.
* Utilisez [IServicePartition.ReportPartitionHealth](/dotnet/api/system.fabric.iservicepartition.reportpartitionhealth) pour créer un rapport sur l’entité de partition actuelle.
* Utilisez [CodePackageActivationContext.ReportApplicationHealth](/dotnet/api/system.fabric.codepackageactivationcontext.reportapplicationhealth) pour créer un rapport sur l’application en cours.
* Utilisez [CodePackageActivationContext.ReportDeployedApplicationHealth](/dotnet/api/system.fabric.codepackageactivationcontext.reportdeployedapplicationhealth) pour créer un rapport sur l’application en cours déployée sur le nœud actuel.
* Utilisez [CodePackageActivationContext.ReportDeployedServicePackageHealth](/dotnet/api/system.fabric.codepackageactivationcontext.reportdeployedservicepackagehealth) afin de créer un rapport sur un package de services pour l’application déployée sur le nœud actuel.

> [!NOTE]
> En interne, `Partition` et `CodePackageActivationContext` contiennent un client de contrôle d’intégrité configuré avec les paramètres par défaut. Comme indiqué pour le [client de contrôle d’intégrité](service-fabric-report-health.md#health-client), les rapports sont traités par lots et envoyés sur un minuteur. Les objets doivent être maintenus actifs pour avoir une chance d’envoyer le rapport.
> 
> 

Vous pouvez spécifier `HealthReportSendOptions` lors de l’envoi de rapports par le biais des API de contrôle d’intégrité `Partition` et `CodePackageActivationContext`. Si des rapports critiques doivent être envoyés dès que possible, utilisez `HealthReportSendOptions` avec Immediate `true`. Les rapports immédiats ignorent cet intervalle de traitement par lots du client de contrôle d’intégrité interne. Comme mentionné précédemment, utilisez cet indicateur avec précaution ; nous souhaitons tirer parti de ce traitement par lots du client de contrôle d’intégrité chaque fois que cela est possible.

## <a name="design-health-reporting"></a>Conception de rapports d’intégrité
La première étape de la génération de rapports de haute qualité consiste à identifier les conditions qui peuvent avoir une incidence sur l’intégrité du service. Toute condition pouvant aider à signaler des problèmes dans le service ou le cluster dès le début ou, mieux encore, avant qu’ils se produisent, peut permettre d’économiser des milliards de dollars. Les avantages incluent moins temps d’arrêt, moins de nuits passées à investiguer et à résoudre des problèmes, et une satisfaction accrue des clients.

Une fois les conditions identifiées, les enregistreurs de surveillance doivent déterminer la meilleure façon de les surveiller pour obtenir le meilleur équilibre possible entre la surcharge et l’utilité. Prenons comme exemple un service qui effectue des calculs complexes qui utilisent des fichiers temporaires sur un partage. Un agent de surveillance peut surveiller le partage pour s’assurer que l’espace disponible est suffisant. Il peut écouter les notifications relatives aux modifications de fichiers ou de répertoires. Il peut lancer un avertissement si un seuil initial est atteint, et signaler une erreur si le partage est plein. En cas d’avertissement, un système de réparation peut démarrer le nettoyage de fichiers plus anciens sur le partage. En cas d’erreur, un système de réparation peut déplacer le réplica de service vers un autre nœud. Notez la manière dont les états de condition sont décrits en termes d’intégrité, selon qu’ils peuvent être considérés comme sains (ok) ou défectueux (avertissement ou erreur).

Une fois les détails de la surveillance définis, l’enregistreur de surveillance doit déterminer comment mettre en œuvre l’agent de surveillance. Si les conditions peuvent être déterminées à partir du service, l’agent de surveillance peut faire partie du service surveillé lui-même. Par exemple, le code de service peut vérifier l’utilisation du partage, puis générer un rapport chaque fois qu’il tente d’écrire un fichier. L’avantage de cette approche est que la création de rapports est simple. Vous devez cependant veiller à ce que des bogues de l’agent de surveillance n’affectent pas la fonctionnalité du service.

La création de rapports depuis le service surveillé n’est pas toujours une option. Il se peut qu’un agent de surveillance au sein du service ne soit pas en mesure de détecter les conditions. Il se peut qu’il ne dispose pas de la logique ou des données nécessaires pour les déterminer. La surcharge résultant de la surveillance des conditions peut être élevée. Il se peut également que les conditions ne soient pas spécifiques à un service, mais affectent des interactions entre services. Une autre option consiste à disposer d’agents de surveillance dans le cluster en tant que processus distincts. Les agents de surveillance surveillent les conditions et créent des rapports, sans affecter les services principaux. Par exemple, il est possible d’implémenter ces agents de surveillance en tant que services sans état dans la même application, déployés sur tous les nœuds ou sur les mêmes nœuds que le service.

Parfois, un agent de surveillance s’exécutant dans le cluster n’est pas non plus une option. Si la condition surveillée est la disponibilité ou la fonctionnalité du service telles que les utilisateurs les voient, il est préférable que les agents de surveillance se trouvent au même endroit que les clients de l’utilisateur. Là, ils peuvent tester les opérations de la même façon que les utilisateurs les appellent. Par exemple, vous pouvez avoir un agent de surveillance situé en dehors du cluster, qui adresse des demandes au service et qui vérifie la latence et l’exactitude du résultat. (Par exemple, pour un service de calculatrice, l’opération 2 + 2 retourne-t-elle 4 dans un délai raisonnable ?)

Une fois les détails de l’agent de surveillance finalisés, vous devez choisir un ID source qui l’identifie de façon unique. Si plusieurs agents de surveillance du même type résident dans le cluster, ils doivent générer des rapports sur des entités différentes ou, s’ils génèrent des rapports sur la même entité, ils doivent utiliser une propriété ou un ID source différent. De cette façon, leurs rapports peuvent coexister. La propriété du rapport d’intégrité doit capturer la condition surveillée. (Par exemple, dans l’exemple ci-dessus, la propriété pourrait être **ShareSize**). Si plusieurs rapports s’appliquent à une même condition, la propriété doit contenir des informations dynamiques permettant la coexistence des rapports. Par exemple, si plusieurs partages doivent être surveillés, le nom de la propriété peut être **ShareSize-sharename**.

> [!NOTE]
> Vous ne devez *pas* utiliser le magasin d’intégrité pour conserver les informations sur l’état. Seules des informations liées à l’intégrité doivent être rapportées comme telles, car ces informations ont une incidence sur l’évaluation de l’intégrité d’une entité. Le magasin d’intégrité n’a pas été conçu en tant que magasin à usage général. Il utilise une logique d’évaluation de l’intégrité pour regrouper toutes les données dans l’état d’intégrité. Le fait d’envoyer des informations non liées à l’intégrité (par exemple, de signaler un statut avec l’état d’intégrité OK) n’a pas d’incidence sur l’état d’intégrité agrégé, mais peut nuire aux performances du magasin d’intégrité.
> 
> 

La décision suivante a trait à l’entité sur laquelle générer le rapport. La plupart du temps, la condition identifie clairement l’entité. Choisissez l’entité offrant la meilleure granularité possible. Si une condition a un impact sur tous les réplicas d’une partition, le rapport doit porter sur la partition, pas le service. Il existe cependant des cas où une réflexion approfondie s’impose. Si la condition a une incidence sur une entité telle qu’un réplica, mais que vous souhaitez que la condition soit signalée pendant une durée supérieure à la durée de vie du réplica, elle doit être rapportée sur la partition. Dans le cas contraire, lorsque le réplica est supprimé, le magasin de contrôle d’intégrité nettoie tous ses rapports. Les enregistreurs de surveillance doivent tenir compte des durées de vie de l’entité et du rapport. Le moment où un rapport doit être effacé d’un magasin doit être clair (par exemple, lorsqu’une erreur signalée sur une entité ne s’applique plus).

Examinons un exemple qui réunit les points décrits ci-dessus. Examinons une application Service Fabric composée d’un service maître persistant avec état et de services subordonnés sans état déployés sur tous les nœuds (un type de service secondaire par type de tâche). Le service maître dispose d’une file d’attente de traitement contenant des commandes que les services secondaires doivent exécuter. Les services secondaires exécutent les demandes entrantes et renvoient des signaux d’accusé de réception. Une condition qui peut être surveillée est la longueur de la file d’attente de traitement du service maître. Si la longueur de la file d’attente du service maître atteint un seuil, un avertissement est lancé. Il indique que les services secondaires ne peuvent pas traiter la charge. Si la file d’attente atteint la longueur maximale et que des commandes sont abandonnées, une erreur est signalée, car le service ne peut pas récupérer. Les rapports peuvent avoir trait à la propriété **QueueStatus**. L’agent de surveillance opère à l’intérieur du service et il est envoyé régulièrement sur le réplica principal du service maître. La durée de vie est de deux minutes, et il est envoyé à intervalles réguliers de 30 secondes. Si le service principal tombe en panne, le rapport est effacé automatiquement du magasin. Si le réplica du service est actif, mais est bloqué ou rencontre d’autres problèmes, le rapport expire dans le magasin d’intégrité. Dans ce cas, l’entité est évaluée au moment de l’erreur.

L’heure d’exécution de la tâche est une autre condition qui peut être surveillée. Le service maître distribue les tâches aux services secondaires en fonction du type de tâche. Selon la conception, le service maître peut interroger les services secondaires pour évaluer l’état de la tâche. Il peut également attendre que les services secondaires renvoient des signaux d’accusé de réception quand ils ont terminé. Dans le deuxième cas, vous devez veiller à détecter les situations où des services secondaires périssent ou bien où des messages se perdent. Une option pour le service maître consiste à envoyer au même service secondaire une requête ping qui renvoie son état. Si le service maître ne reçoit aucun état, il considère qu’il s’agit d’un échec et replanifie la tâche. Ce comportement part du principe que les tâches sont idempotentes.

La condition surveillée peut être convertie en avertissement si la tâche n’est pas effectuée dans une période donnée (**t1**, par exemple 10 minutes). Si la tâche n’est pas terminée dans le temps (**t2**, par exemple 20 minutes), la condition surveillée peut être convertie en erreur. Cette création de rapport peut se faire de plusieurs façons :

* Le réplica principal du service maître génère régulièrement des rapports sur lui-même. Il peut y avoir une propriété pour toutes les tâches en attente dans la file d’attente. Si au moins une tâche prend plus de temps, le statut du rapport sur la propriété **PendingTasks** est un avertissement ou une erreur, selon le cas. Si aucune tâche n’est en attente ou si toutes les tâches ont démarré, le statut du rapport est OK. Les tâches sont persistantes. Ainsi, si le service principal s’arrête, le service principal qui lui succède peut continuer à générer des rapports correctement.
* Un autre processus de surveillance (dans le cloud ou externe) vérifie les tâches (de l’extérieur, en fonction du résultat souhaité de la tâche) pour voir si elles sont accomplies. Si elles ne respectent pas les seuils, un rapport est envoyé sur le service maître. Un rapport est également envoyé sur chaque tâche. Il inclut l’identificateur de tâche (par exemple **PendingTask+Idtâche**). Des rapports ne doivent être envoyés que sur des états défectueux. La valeur TTL est définie sur quelques minutes et les rapports à supprimer sont sélectionnés lorsqu’ils ont expiré, afin d’assurer le nettoyage.
* Le service secondaire qui exécute une tâche génère un rapport lorsque l’exécution dure plus longtemps que prévu. Il génère un rapport sur l’instance de service sur la propriété **PendingTasks**. Cela met en évidence l’instance de service qui présente des problèmes, mais ne capture pas la situation où l’instance meurt. Les rapports sont nettoyés à ce moment-là. Il peut générer un rapport sur le service secondaire. Si le service secondaire achève la tâche, l’instance secondaire efface le rapport du magasin. Cela n’a pas pour effet de capturer la situation où le message d’accusé de réception est perdu et où la tâche n’est pas achevée du point de vue du service maître.

Quelle que soit la façon dont les rapports sont générés dans les cas décrits ci-dessus, ils sont capturés dans l’intégrité de l’application lors de l’évaluation de l’intégrité.

## <a name="report-periodically-vs-on-transition"></a>Rapport régulier ou sur transition
À l’aide du modèle de génération de rapports d’intégrité, les agents de surveillance peuvent envoyer des rapports régulièrement ou en cas de transition. La méthode régulière est recommandée pour les rapports de surveillance, car le code est beaucoup plus simple et moins sujet aux erreurs. Les agents de surveillance doivent être aussi simples que possible pour éviter les bogues qui déclenchent des rapports incorrects. Des rapport d’état *défectueux* incorrects ont une incidence sur les évaluations de l’intégrité et les scénarios basés sur l’intégrité, dont les mises à niveau. Des rapports d’état *sain* incorrects masquent d’éventuels problèmes dans le cluster, ce qui n’est pas souhaitable.

Pour la création de rapports réguliers, l’agent de surveillance peut être implémenté avec une minuterie. Sur un rappel de la minuterie, l’agent de surveillance peut vérifier l’état et envoyer un rapport basé sur l’état actuel. Il est inutile de voir quel rapport a été envoyé précédemment ou d’opérer des optimisations en termes de messagerie. Le client de contrôle d’intégrité dispose de la logique de traitement par lot, qui permet d’améliorer les performances. Tant que le client de contrôle d’intégrité est actif, il réessaie en interne jusqu’à ce que le rapport soit reçu par le magasin d’intégrité ou que l’agent de surveillance crée un rapport plus récent avec une entité, une propriété et une source identiques.

La génération de rapports sur les transitions nécessite une gestion prudente de l’état. L’agent de surveillance surveille certaines conditions et génère un rapport uniquement quand les conditions changent. Cette approche présente un avantage : un nombre moins important de rapports est nécessaire. L’inconvénient est que la logique de l’agent de surveillance est complexe. L’agent de surveillance doit gérer mes conditions ou les rapports, afin qu’il soit possible de les inspecter pour déterminer les changements d’état. Lors du basculement, une attention particulière doit être apportée aux rapports qui ont été ajoutés, mais n’ont pas encore été envoyés au magasin d’intégrité. Le numéro de séquence doit toujours être croissant. Si ce n’est pas le cas, les rapports sont rejetés comme périmés. Dans les rares cas où une perte de données se produit, une synchronisation peut être nécessaire entre l’état du rapporteur et l’état du magasin d’intégrité.

La création de rapports sur les transitions est fondée pour les services de création de rapports sur eux-mêmes, via `Partition` ou `CodePackageActivationContext`. Lorsque l’objet local (réplica ou package de service déployé / application déployée) est supprimé, tous ses rapports sont également supprimés. Le nettoyage automatique assouplit la nécessité de synchronisation entre le rapporteur et le magasin d’intégrité. Si le rapport concerne la partition parente ou l’application parente, surveillez le basculement afin d’éviter les rapports obsolètes dans le magasin d’intégrité. Une logique doit être ajoutée pour maintenir l’état correct et le rapport doit être supprimé du magasin lorsqu’il est devenu inutile.

## <a name="implement-health-reporting"></a>Mise en œuvre de la création de rapports d’intégrité
Une fois que les détails de l’entité et du rapport sont clairs, l’envoi de rapports d’intégrité peut se faire via l’API, Powershell ou REST.

### <a name="api"></a>API
Pour générer un rapport via l’API, vous devez créer un rapport d’intégrité spécifique au type d’entité sur laquelle ils veulent générer un rapport. Fournissez le rapport à un client de contrôle d’intégrité. Vous pouvez également créer des informations sur l’intégrité et les transmettre aux méthodes de création de rapports correctes sur `Partition` ou `CodePackageActivationContext` pour créer un rapport sur les entités en cours.

L’exemple suivant illustre une génération de rapport périodique à partir d’un agent de surveillance dans le cluster. L’agent de surveillance vérifie si une ressource externe est accessible à partir d’un nœud. La ressource est requise par un manifeste de service au sein de l’application. Si la ressource n’est pas disponible, les autres services au sein de l’application peuvent encore fonctionner correctement. Par conséquent, le rapport est envoyé sur l’entité du package de services déployé toutes les 30 secondes.

```csharp
private static Uri ApplicationName = new Uri("fabric:/WordCount");
private static string ServiceManifestName = "WordCount.Service";
private static string NodeName = FabricRuntime.GetNodeContext().NodeName;
private static Timer ReportTimer = new Timer(new TimerCallback(SendReport), null, 30 * 1000, 30 * 1000);
private static FabricClient Client = new FabricClient(new FabricClientSettings() { HealthReportSendInterval = TimeSpan.FromSeconds(0) });

public static void SendReport(object obj)
{
    // Test whether the resource can be accessed from the node
    HealthState healthState = this.TestConnectivityToExternalResource();

    // Send report on deployed service package, as the connectivity is needed by the specific service manifest
    // and can be different on different nodes
    var deployedServicePackageHealthReport = new DeployedServicePackageHealthReport(
        ApplicationName,
        ServiceManifestName,
        NodeName,
        new HealthInformation("ExternalSourceWatcher", "Connectivity", healthState));

    // TODO: handle exception. Code omitted for snippet brevity.
    // Possible exceptions: FabricException with error codes
    // FabricHealthStaleReport (non-retryable, the report is already queued on the health client),
    // FabricHealthMaxReportsReached (retryable; user should retry with exponential delay until the report is accepted).
    Client.HealthManager.ReportHealth(deployedServicePackageHealthReport);
}
```

### <a name="powershell"></a>PowerShell
Envoyez les rapports d’intégrité avec **Send-ServiceFabric *EntityType* HealthReport**.

L’exemple suivant montre la création de rapports réguliers sur les valeurs du processeur sur un nœud. Les rapports doivent être envoyés toutes les 30 secondes, et ont une durée de vie de deux minutes. S’ils expirent, cela signifie que le rapporteur rencontre des problèmes, et le nœud est évalué au moment de l’erreur. Quand l’utilisation du processeur dépasse un seuil, le rapport a un état d’avertissement. Quand l’utilisation du processeur reste supérieure à un seuil plus longtemps que le délai configuré, une erreur est rapportée. Autrement, le rapporteur envoie un état d’intégrité OK.

```powershell
PS C:\> Send-ServiceFabricNodeHealthReport -NodeName Node.1 -HealthState Warning -SourceId PowershellWatcher -HealthProperty CPU -Description "CPU is above 80% threshold" -TimeToLiveSec 120

PS C:\> Get-ServiceFabricNodeHealth -NodeName Node.1
NodeName              : Node.1
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='PowershellWatcher', Property='CPU', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 5
                        SentAt                : 4/21/2015 8:01:17 AM
                        ReceivedAt            : 4/21/2015 8:02:12 AM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/21/2015 8:02:12 AM

                        SourceId              : PowershellWatcher
                        Property              : CPU
                        HealthState           : Warning
                        SequenceNumber        : 130741236814913394
                        SentAt                : 4/21/2015 9:01:21 PM
                        ReceivedAt            : 4/21/2015 9:01:21 PM
                        TTL                   : 00:02:00
                        Description           : CPU is above 80% threshold
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Warning = 4/21/2015 9:01:21 PM
```

L’exemple suivant rapporte un avertissement temporaire sur un réplica. Il obtient d’abord l’ID de partition, puis l’ID de réplica pour le service qui l’intéresse. Il envoie ensuite un rapport à partir de **PowershellWatcher** sur la propriété **ResourceDependency**. Le rapport n’est utile que pendant deux minutes. Ensuite, il est automatiquement supprimé du magasin.

```powershell
PS C:\> $partitionId = (Get-ServiceFabricPartition -ServiceName fabric:/WordCount/WordCount.Service).PartitionId

PS C:\> $replicaId = (Get-ServiceFabricReplica -PartitionId $partitionId | where {$_.ReplicaRole -eq "Primary"}).ReplicaId

PS C:\> Send-ServiceFabricReplicaHealthReport -PartitionId $partitionId -ReplicaId $replicaId -HealthState Warning -SourceId PowershellWatcher -HealthProperty ResourceDependency -Description "The external resource that the primary is using has been rebooted at 4/21/2015 9:01:21 PM. Expect processing delays for a few minutes." -TimeToLiveSec 120 -RemoveWhenExpired

PS C:\> Get-ServiceFabricReplicaHealth  -PartitionId $partitionId -ReplicaOrInstanceId $replicaId


PartitionId           : 8f82daff-eb68-4fd9-b631-7a37629e08c0
ReplicaId             : 130740415594605869
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='PowershellWatcher', Property='ResourceDependency', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 130740768777734943
                        SentAt                : 4/21/2015 8:01:17 AM
                        ReceivedAt            : 4/21/2015 8:02:12 AM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/21/2015 8:02:12 AM

                        SourceId              : PowershellWatcher
                        Property              : ResourceDependency
                        HealthState           : Warning
                        SequenceNumber        : 130741243777723555
                        SentAt                : 4/21/2015 9:12:57 PM
                        ReceivedAt            : 4/21/2015 9:12:57 PM
                        TTL                   : 00:02:00
                        Description           : The external resource that the primary is using has been rebooted at 4/21/2015 9:01:21 PM. Expect processing delays for a few minutes.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : ->Warning = 4/21/2015 9:12:32 PM
```

### <a name="rest"></a>REST
Envoyez des rapports d’intégrité à l’aide de REST, avec les demandes POST accédant à l’entité choisie et dont le corps présente une description du rapport d’intégrité. Par exemple, découvrez comment envoyer des [rapports REST sur l’intégrité du cluster](/rest/api/servicefabric/report-the-health-of-a-cluster) ou des [rapports REST sur l’intégrité du service](/rest/api/servicefabric/report-the-health-of-a-service). Toutes les entités sont prises en charge.

## <a name="next-steps"></a>Étapes suivantes
Grâce aux données d’intégrité, les enregistreurs de service et les administrateurs de cluster/d’application peuvent réfléchir à des façons de consommer les informations. Par exemple, ils peuvent configurer des alertes basées sur l’état d’intégrité pour intercepter des problèmes graves avant qu’ils provoquent des pannes. Les administrateurs peuvent également configurer des systèmes de réparation pour résoudre les problèmes automatiquement.

[Présentation du contrôle d’intégrité de Service Fabric](service-fabric-health-introduction.md)

[Affichage rapports d’intégrité de Service Fabric](service-fabric-view-entities-aggregated-health.md)

[Comment signaler et contrôler l’intégrité du service](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

[Utilisation des rapports d’intégrité système pour la résolution des problèmes](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)

[Surveiller et diagnostiquer les services localement](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Mise à niveau des applications Service Fabric](service-fabric-application-upgrade.md)
