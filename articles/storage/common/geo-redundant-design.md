---
title: Utilisez la géo-redondance pour concevoir des applications hautement disponibles
titleSuffix: Azure Storage
description: Découvrez comment utiliser le stockage géoredondant pour concevoir une application hautement disponible suffisamment flexible pour gérer les interruptions.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 05/05/2020
ms.author: tamram
ms.reviewer: artek
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: c16f8233a2800025a8c6f601e236b86d2fd044fd
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92480681"
---
# <a name="use-geo-redundancy-to-design-highly-available-applications"></a>Utilisez la géo-redondance pour concevoir des applications hautement disponibles

L’accès à une plateforme hautement disponible et durable comme Stockage Azure pour l’hébergement des données et des applications est une caractéristique courante des infrastructures basées sur le cloud. Les développeurs d’applications cloud doivent bien réfléchir à la façon de tirer parti de cette plateforme pour optimiser ces avantages pour leurs utilisateurs. Le stockage Azure offre un stockage géoredondant pour garantir une haute disponibilité, même en cas de panne régionale. Les comptes de stockage configurés pour la réplication géoredondante sont répliqués de manière synchrone dans la région primaire, puis répliqués de manière asynchrone dans une région secondaire à des centaines de kilomètres.

Le stockage Azure offre deux options de réplication géoredondante. La seule différence entre ces deux options réside dans la manière dont les données sont répliquées dans la région primaire :

* [Stockage géoredondant interzone (GZRS)](storage-redundancy.md) : Les données sont répliquées de manière synchrone dans trois zones de disponibilité Azure dans la région primaire à l’aide du *stockage redondant interzone (ZRS)* , puis répliquées de façon asynchrone dans la région secondaire. Pour un accès en lecture aux données dans la région secondaire, activez le stockage géoredondant interzone avec accès en lecture (RA-GZRS).

    Microsoft recommande d’utiliser l’option GZRS/RA-GZRS pour les scénarios qui nécessitent une disponibilité et une durabilité maximales.

* [Stockage géo-redondant (GRS)](storage-redundancy.md) : Les données sont répliquées de manière synchrone trois fois dans la région primaire à l’aide du *stockage localement redondant (LRS)* , puis répliquées de façon asynchrone dans la région secondaire. Pour un accès en lecture aux données dans la région secondaire, activez le stockage géoredondant avec accès en lecture (RA-GRS).

Cet article explique comment concevoir votre application pour gérer une panne dans la région primaire. Si la région primaire n’est plus disponible, votre application peut s’adapter pour effectuer des opérations de lecture dans la région secondaire à la place. Avant de commencer, vérifiez que votre compte de stockage est configuré pour RA-GRS ou RA-GZRS.

## <a name="application-design-considerations-when-reading-from-the-secondary"></a>Considérations relatives à la conception d’applications lors de la lecture à partir du serveur secondaire

L’objectif de cet article est de vous montrer comment concevoir une application qui continuera à fonctionner (bien qu’avec des capacités limitées), même si un sinistre majeur affecte le centre de données principal. Vous pouvez configurer votre application pour gérer des problèmes temporaires ou à long terme via la lecture à partir de la région secondaire lorsqu’un problème empêche la lecture à partir de la région principale. Lorsque la région principale redevient disponible, votre application peut reprendre la lecture à partir de la région principale.

Gardez à l’esprit ces points clés pendant la conception de votre application pour RA-GRS ou RA-GZRS :

* Stockage Azure conserve une copie en lecture seule des données que vous stockez dans votre région primaire d’une région secondaire. Comme indiqué ci-dessus, le service de stockage détermine l’emplacement de la région secondaire.

* La copie en lecture seule est [cohérente](https://en.wikipedia.org/wiki/Eventual_consistency) avec les données de la région primaire.

* Pour les blobs, tables et files d’attente, vous pouvez interroger la région secondaire pour obtenir la *dernière heure de synchronisation* . Cette valeur vous indique à quel moment la dernière réplication de la région primaire sur la région secondaire s’est produite. (Cette fonctionnalité n’est pas prise en charge pour Azure Files, qui n’a pas la redondance RA-GRS pour l’instant.)

* Vous pouvez utiliser la bibliothèque cliente de stockage pour lire et écrire les données de la région primaire ou secondaire. Vous pouvez également rediriger les demandes de lecture automatiquement vers la région secondaire si une demande de lecture adressée à la région primaire arrive à expiration.

* Si la région primaire devient indisponible, vous pouvez initier un basculement de compte. Lorsque vous basculez vers la région secondaire, les entrées DNS pointant vers la région primaire sont modifiées pour pointer vers la région secondaire. Au terme du basculement, l'accès en écriture est restauré pour les comptes GRS et RA-GRS. Pour plus d’informations, consultez [Récupération d’urgence et basculement de compte de stockage](storage-disaster-recovery-guidance.md).

### <a name="using-eventually-consistent-data"></a>Utilisation de données cohérentes

La solution proposée part du principe qu’il est acceptable de retourner à l’application appelante des données potentiellement périmées. Comme les données de la région secondaire finissent par être cohérentes, il est possible que la région principale devenir inaccessible avant qu’une mise à jour dans la région secondaire ait terminé la réplication.

Par exemple, votre client soumet une mise à jour avec succès, mais la région principale échoue avant la propagation de cette mise à jour à la région secondaire. Lorsque le client demande à relire les données, il reçoit les données périmées de la région secondaire au lieu des données mises à jour. Lorsque vous concevez votre application, vous devez décider si cela est acceptable et, si tel est le cas, la façon dont vous enverrez un message au client. 

Plus loin dans cet article, nous vous montrerons comment vérifier la dernière heure de synchronisation des données secondaires pour savoir si la région secondaire est à jour.

### <a name="handling-services-separately-or-all-together"></a>Gestion des services ensemble ou séparément

Même si cette situation est peu probable, il est possible qu’un service devienne indisponible, alors que tous les autres restent entièrement fonctionnels. Vous pouvez gérer les nouvelles tentatives et le mode lecture seule pour chaque service séparément (blobs, files d’attente, tables) ou gérer les nouvelles tentatives de façon générique pour l’ensemble des services de stockage.

Par exemple, si vous utilisez des files d’attente et des blobs dans votre application, vous pouvez décider d’insérer un code distinct pour gérer les erreurs renouvelables pour chacun d’eux. Par la suite, si vous recevez une nouvelle tentative du service BLOB mais que le service de File d’attente est toujours fonctionnel, seule la partie de votre application qui gère les blobs sera affectée. Si vous décidez de gérer toutes les nouvelles tentatives de service de stockage de manière générique et qu’un appel au service BLOB retourne une erreur renouvelable, les demandes au service BLOB et au service de File d’attente seront affectées.

Cela dépend de la complexité de votre application. Vous pouvez décider de ne pas gérer les échecs par service, mais plutôt de rediriger les demandes de lecture pour tous les services de stockage vers la région secondaire et d’exécuter l’application en mode lecture seule lorsque vous détectez un problème affectant tout service de stockage dans la région primaire.

### <a name="other-considerations"></a>Autres considérations

Voici les autres considérations dont nous parlerons dans le reste de cet article.

* Gestion des nouvelles tentatives de demandes de lecture à l’aide du modèle Disjoncteur

* Données cohérentes et dernière heure de synchronisation

* Test

## <a name="running-your-application-in-read-only-mode"></a>Exécution de votre application en mode lecture seule

Pour préparer efficacement une interruption dans la région primaire, vous devez être en mesure de gérer les demandes de lecture et de mise à jour (dans le cas présent, les insertions, mises à jour et suppressions) ayant échoué. En cas d’échec de la région primaire, les demandes de lecture peuvent être redirigées vers la région secondaire. Mais les demandes de mise à jour ne peuvent pas être redirigées au centre de données secondaire car celui-ci est en lecture seule. C’est la raison pour laquelle vous devez configurer votre application pour s’exécuter en mode lecture seule.

Par exemple, vous pouvez définir un indicateur qui est vérifié avant l’envoi de toute demande de mise à jour à Stockage Azure. Lorsque l’une des demandes de mise à jour aboutit, vous pouvez l’ignorer et retourner une réponse appropriée au client. Vous pouvez même décider de désactiver certaines fonctionnalités simultanément, et ce, jusqu’à ce que le problème soit résolu, et informer les utilisateurs que ces fonctionnalités sont temporairement indisponibles.

Si vous décidez de gérer les erreurs pour chaque service séparément, vous devez également gérer la possibilité d’exécuter votre application en mode lecture seule par service. Par exemple, vous pouvez définir des indicateurs en lecture seule pour chaque service qui peut être activé et désactivé. Vous pouvez ensuite gérer l’indicateur aux emplacements appropriés dans votre code.

La possibilité d’exécuter votre application en mode lecture seule offre un autre avantage : vous avez la possibilité d’assurer des fonctionnalités limitées pendant une mise à niveau majeure de l’application. Vous pouvez déclencher l’exécution de votre application en mode lecture seule et la faire pointer vers le centre de données secondaire. Vous avez ainsi la certitude que personne n’accède aux données dans la région primaire pendant vous effectuez des mises à niveau.

## <a name="handling-updates-when-running-in-read-only-mode"></a>Gestion des mises à jour lors d’une exécution en mode lecture seule

Il existe de nombreuses façons de gérer les demandes de mise à jour lors d’une exécution en mode lecture seule. Nous n’abordons pas ce point de façon complète, mais vous pouvez généralement prendre quelques modèles en considération.

1. Vous pouvez répondre à votre utilisateur et lui indiquer que les mises à jour ne sont actuellement pas autorisées. Par exemple, un système de gestion des contacts pourrait permettre aux clients d’accéder aux informations de contact sans toutefois autoriser les mises à jour.

2. Vous pouvez empiler vos mises à jour dans une autre région. Dans ce cas, vous écrirez vos demandes de mise à jour en attente dans la file d’attente d’une autre région et disposerez d’un moyen de traiter ces demandes une fois le centre de données principal à nouveau en ligne. Dans ce scénario, vous devez informer le client que la mise à jour demandée a été mise en file d’attente pour un traitement ultérieur.

3. Vous pouvez écrire vos mises à jour dans un compte de stockage d’une autre région. Puis, lorsque le centre de données principal est de nouveau en ligne, vous disposez d’un moyen de fusionner ces mises à jour dans les données primaires, selon la structure des données. Par exemple, si vous créez des fichiers distincts, dont le nom contient l’horodatage, vous pouvez recopier ces fichiers dans la région primaire. Cela fonctionne pour certaines charges de travail, notamment la journalisation et les données IoT.

## <a name="handling-retries"></a>Gestion des nouvelles tentatives

La bibliothèque cliente de stockage Azure vous permet de déterminer les erreurs qui peuvent être renouvelées. Par exemple, une erreur 404 (ressource introuvable) ne fera pas l’objet d’une nouvelle tentative, car elle serait peu susceptible d’aboutir. En revanche, une erreur 500 fera l’objet d’une nouvelle tentative, car il s’agit d’une erreur de serveur : le problème peut donc être simplement temporaire. Pour plus d’informations, consultez le [code open source de la classe ExponentialRetry](https://github.com/Azure/azure-storage-net/blob/87b84b3d5ee884c7adc10e494e2c7060956515d0/Lib/Common/RetryPolicies/ExponentialRetry.cs) dans la bibliothèque cliente de stockage .NET. (Recherchez la méthode ShouldRetry.)

### <a name="read-requests"></a>Demandes de lecture

En cas de problème avec le stockage principal, les demandes de lecture peuvent être redirigées vers le stockage secondaire. Comme indiqué plus haut dans la section [Utilisation de données cohérentes](#using-eventually-consistent-data), la lecture de données périmées par votre application doit être acceptable. Si vous utilisez la bibliothèque cliente de stockage pour accéder aux données à partir de la région secondaire, vous pouvez spécifier le comportement de nouvelle tentative d’une demande de lecture en affectant à la propriété **LocationMode** l’une des valeurs suivantes :

* **PrimaryOnly** (valeur par défaut)

* **PrimaryThenSecondary**

* **SecondaryOnly**

* **SecondaryThenPrimary**

Lorsque vous affectez à la propriété **LocationMode** la valeur **PrimaryThenSecondary** , si la demande de lecture initiale au point de terminaison principal échoue avec une erreur renouvelable, le client effectue automatiquement une autre demande de lecture au point de terminaison secondaire. Si l’erreur est liée au délai d’attente du serveur, le client devra attendre l’expiration du délai avant la réception d’une erreur renouvelable du service.

Lorsque vous décidez de la façon de répondre à une erreur renouvelable, deux scénarios principaux doivent être envisagés :

* Il s’agit d’un problème isolé et les demandes ultérieures au point de terminaison principal ne retourneront pas d’erreur renouvelable. Ceci peut se produire, par exemple, en cas d’erreur réseau temporaire.

    Dans ce scénario, l’affectation à la propriété **LocationMode** de la valeur **PrimaryThenSecondary** n’entraîne pas de perte de performances significative, dans la mesure où cela ne se produit que rarement.

* Le problème concerne au moins un des services de stockage de la région primaire, et toutes les demandes ultérieures à l’attention de ce service dans la région primaire sont susceptibles de retourner des erreurs renouvelables sur une certaine période. Cela peut se produire, par exemple, lorsque la région primaire est totalement inaccessible.

    Dans ce scénario, une perte de performances est observée, dans la mesure où toutes vos demandes de lecture essaieront le point de terminaison principal en premier, attendront l’expiration du délai, puis basculeront vers le point de terminaison secondaire.

Dans le cadre de ces scénarios, vous devez savoir qu’un problème affecte actuellement le point de terminaison principal et envoyer toutes les demandes de lecture directement au point de terminaison secondaire en affectant à la propriété **LocationMode** la valeur **SecondaryOnly** . À ce stade, vous devez également définir l’exécution de l’application en mode lecture seule. Cette approche est appelée le modèle Disjoncteur. Voir [Circuit Breaker Pattern](/azure/architecture/patterns/circuit-breaker) (modèle Disjoncteur).

### <a name="update-requests"></a>Demandes de mise à jour

Le modèle Disjoncteur peut également être appliqué aux demandes de mise à jour. Toutefois, les demandes de mise à jour ne peuvent pas être redirigées vers un stockage secondaire, qui est en lecture seule. Pour ces demandes, vous devez laisser la propriété **LocationMode** définie sur la valeur **PrimaryOnly** (valeur par défaut). Pour gérer ces erreurs, vous pouvez appliquer une mesure à ces demandes (10 échecs consécutifs, par exemple) et faire basculer l’application en mode lecture seule lorsque le seuil est atteint. Vous pouvez utiliser les méthodes décrites ci-après, dans la section Modèle Disjoncteur, pour rétablir le mode de mise à jour.

## <a name="circuit-breaker-pattern"></a>Modèle Disjoncteur

L’utilisation du modèle Disjoncteur dans votre application peut l’empêcher de réessayer une opération qui échouera probablement de nouveau. Il permet à l’application de poursuivre son exécution plutôt que de perdre du temps en multipliant les tentatives. Il détecte également le moment auquel l’erreur est corrigée et auquel l’application peut essayer l’opération de nouveau.

### <a name="how-to-implement-the-circuit-breaker-pattern"></a>Comment implémenter le modèle Disjoncteur

Pour savoir si un problème affecte actuellement un point de terminaison principal, vous pouvez contrôler la fréquence à laquelle le client rencontre des erreurs renouvelables. Chaque cas étant différent, vous devez décider du seuil à utiliser pour déterminer quand basculer vers le point de terminaison secondaire et exécuter l’application en mode lecture seule. Par exemple, vous pouvez décider d’effectuer le basculement au bout de 10 échecs consécutifs. Vous pouvez également effectuer le basculement si 90 % des demandes échouent sur une période de 2 minutes.

Dans le cadre du premier scénario, vous pouvez simplement tenir le compte des échecs et, en cas de réussite avant que la valeur maximale soit atteinte, remettre le compte à zéro. Dans le cadre du deuxième scénario, vous pouvez, par exemple, utiliser l’objet MemoryCache (dans .NET). Pour chaque demande, ajoutez un CacheItem au cache, définissez la valeur sur 1 (réussite) ou 0 (échec) et définissez le délai d’expiration à 2 minutes à partir de maintenant (ou sur une autre valeur selon vos contraintes de temps). Quand le délai d’expiration d’une entrée est atteint, l’entrée est automatiquement supprimée. Cela vous donne une fenêtre cumulative de 2 minutes. À chaque demande effectuée au service de stockage, vous utilisez d’abord une requête LINQ sur l’objet MemoryCache pour calculer le pourcentage de réussite en additionnant les valeurs et en divisant le résultat par la valeur du compte. Lorsque le pourcentage de réussite est inférieur à un certain seuil (par exemple, 10 %), affectez à la propriété **LocationMode** la valeur **SecondaryOnly** pour les demandes de lecture et faites basculer l’application en mode lecture seule avant de continuer.

Le seuil d’erreurs, qui permet de déterminer le moment auquel effectuer le basculement, peut varier d’un service à l’autre dans votre application. Vous devez donc envisager de rendre ces paramètres configurables. C’est également le moment où vous décidez de gérer les erreurs renouvelables pour chaque service séparément ou comme un ensemble, comme indiqué précédemment.

Vous devez également réfléchir à la façon de gérer plusieurs instances d’une application et à ce que vous souhaitez faire lorsque vous détectez des erreurs renouvelables dans chaque instance. Par exemple, vous pouvez avoir 20 machines virtuelles en cours d’exécution, sur lesquelles la même application est chargée. Gérez-vous chaque instance séparément ? Si une instance commence à présenter des problèmes, souhaitez-vous limiter la réponse à cette instance ou essayer de faire en sorte que toutes les instances répondent de la même façon ? Gérer les instances séparément est beaucoup plus simple que d’essayer de coordonner la réponse entre elles. Cependant, la façon de procéder varie selon l’architecture de votre application.

### <a name="options-for-monitoring-the-error-frequency"></a>Options disponibles pour la surveillance de la fréquence d’erreur

Vous disposez de trois options principales pour la surveillance de la fréquence des nouvelles tentatives dans la région primaire, en vue de déterminer à quel moment basculer vers la région secondaire et faire passer l’application en mode lecture seule.

* Ajoutez un gestionnaire pour l’événement [**Retrying**](/dotnet/api/microsoft.azure.cosmos.table.operationcontext.retrying) sur l’objet [**OperationContext**](/java/api/com.microsoft.applicationinsights.extensibility.context.operationcontext) transmis à vos demandes de stockage. Il s’agit de la méthode présentée dans cet article et utilisée dans l’exemple qui l’accompagne. Ces événements se déclenchent à chaque fois que le client tente une nouvelle demande, ce qui vous permet de suivre la fréquence à laquelle le client rencontre des erreurs renouvelables sur un point de terminaison principal.

    ```csharp
    operationContext.Retrying += (sender, arguments) =>
    {
        // Retrying in the primary region
        if (arguments.Request.Host == primaryhostname)
            ...
    };
    ```

* Dans la méthode [**Evaluate**](/dotnet/api/microsoft.azure.cosmos.table.iextendedretrypolicy.evaluate) d’une stratégie de nouvelle tentative personnalisée, vous pouvez exécuter du code personnalisé chaque fois qu’une nouvelle tentative est effectuée. Le moment où une nouvelle tentative est effectuée est enregistré. En outre, cela vous donne également la possibilité de modifier le comportement de nouvelle tentative.

    ```csharp
    public RetryInfo Evaluate(RetryContext retryContext,
    OperationContext operationContext)
    {
        var statusCode = retryContext.LastRequestResult.HttpStatusCode;
        if (retryContext.CurrentRetryCount >= this.maximumAttempts
            || ((statusCode >= 300 && statusCode < 500 && statusCode != 408)
            || statusCode == 501 // Not Implemented
            || statusCode == 505 // Version Not Supported
            ))
        {
            // Do not retry
            return null;
        }

        // Monitor retries in the primary location
        ...

        // Determine RetryInterval and TargetLocation
        RetryInfo info =
            CreateRetryInfo(retryContext.CurrentRetryCount);

        return info;
    }
    ```

* La troisième approche consiste à implémenter un composant de contrôle personnalisé dans votre application, qui effectue des tests ping en permanence sur votre point de terminaison de stockage principal avec des demandes de lecture factices (par exemple, la lecture d’un blob de petite taille) pour en déterminer l’état d’intégrité. Dans ce cas, la quantité de ressources sollicitées est raisonnable. Lorsque le système détecte qu’un problème atteint votre seuil, vous effectuez alors le basculement vers **SecondaryOnly** et le mode lecture seule.

À un moment donné, vous souhaiterez rétablir l’utilisation du point de terminaison principal et autoriser les mises à jour. Si vous utilisez l’une des deux premières méthodes indiquées plus haut, vous pouvez simplement rebasculer vers le point de terminaison principal et activer le mode de mise à jour après une période sélectionnée arbitrairement ou une fois effectué un certain nombre d’opérations. Vous pouvez ensuite laisser place de nouveau à la logique de nouvelle tentative. Si le problème a été résolu, le point de terminaison principal reste utilisé et les mises à jour restent autorisées. Si le problème persiste, un basculement vers le point de terminaison secondaire et le mode lecture seul est de nouveau effectué dès lors que les critères que vous avez définis ne sont pas respectés.

Dans le cadre du troisième scénario, lorsque les tests ping effectués sur le point de terminaison de stockage principal aboutissent de nouveau, vous pouvez déclencher le nouveau basculement vers **PrimaryOnly** et continuer à autoriser les mises à jour.

## <a name="handling-eventually-consistent-data"></a>Gestion des données cohérentes

Le stockage géoredondant réplique des transactions de la région primaire vers la région secondaire. Ce processus de réplication garantit que les données de la région secondaire sont *cohérentes* . Cela signifie que toutes les transactions de la région primaire apparaîtront dans la région secondaire. Cependant, cela peut prendre un certain temps, et rien ne garantit que les transactions arrivent dans la région secondaire dans l’ordre dans lequel elles ont été initialement appliquées dans la région primaire. Si vos transactions arrivent dans la région secondaire dans le désordre, vous *pouvez* considérer que vos données dans cette région resteront dans un état incohérent jusqu’à ce que le service rattrape son retard.

Le tableau suivant illustre ce qui peut se produire lorsque vous mettez à jour les informations d’un employé pour qu’il devienne un membre du rôle *Administrateurs* . Cet exemple implique que vous mettiez à jour l’entité **d’employé** et une entité de **rôle administrateur** avec le nombre total d’administrateurs. Notez la façon dont les mises à jour sont appliquées dans le désordre dans la région secondaire.

| **Time** | **Transaction**                                            | **Réplication**                       | **Dernière heure de synchronisation** | **Résultat** |
|----------|------------------------------------------------------------|---------------------------------------|--------------------|------------| 
| T0       | Transaction A : <br> Insérer l’entité d’employé <br> dans la région primaire |                                   |                    | Transaction A insérée dans la région primaire,<br> pas encore répliquée. |
| T1       |                                                            | Transaction A <br> répliquée sur<br> la région secondaire | T1 | Transaction A répliquée sur la région secondaire. <br>Dernière heure de synchronisation mise à jour.    |
| T2       | Transaction B :<br>Update<br> l’entité d’employé<br> dans la région primaire  |                                | T1                 | Transaction B écrite dans la région primaire,<br> pas encore répliquée.  |
| T3       | Transaction C :<br> Update <br>administrator<br>entité de rôle dans<br>primary |                    | T1                 | Transaction C écrite dans la région primaire,<br> pas encore répliquée.  |
| *T4*     |                                                       | Transaction C <br>répliquée sur<br> la région secondaire | T1         | Transaction C répliquée sur la région secondaire.<br>LastSyncTime pas encore mis à jour car <br>la transaction B n’a pas encore été répliquée.|
| *T5*     | Lire les entités <br>de la région secondaire                           |                                  | T1                 | Vous obtenez la valeur périmée pour l’entité d’employé <br> car la transaction B n’a <br> pas encore été répliquée. Vous obtenez la nouvelle valeur pour<br> l’entité de rôle d’administrateur car C a<br> été répliquée. La dernière heure de synchronisation n’a pas encore<br> été mise à jour car la transaction B<br> n’a pas été répliquée. Vous savez que<br>l’entité de rôle d’administrateur est cohérente <br>car l’heure/la date de l’entité sont postérieures à <br>la dernière heure de synchronisation. |
| *T6*     |                                                      | Transaction B<br> répliquée sur<br> la région secondaire | T6                 | *T6* – Toutes les transactions jusqu’à C ont <br>été répliquées. La dernière heure de synchronisation<br> est mise à jour. |

Dans cet exemple, supposons que le client bascule vers la lecture à partir de la région secondaire à l’instant T5. À ce stade, il peut lire correctement l’entité de **rôle administrateur** . Cependant, l’entité contient une valeur pour le nombre d’administrateurs qui n’est pas cohérente avec le nombre d’entités **d’employé** marquées comme administrateurs dans la région secondaire. Votre client pourrait simplement afficher cette valeur au risque que les informations soient incohérentes. Il pourrait également tenter de déterminer que le **rôle administrateur** est dans un état potentiellement incohérent dans la mesure où les mises à jour ont été effectuées dans le désordre et en informer l’utilisateur.

Pour déterminer que ses données sont potentiellement incohérentes, le client peut utiliser la valeur de la *dernière heure de synchronisation* , que vous pouvez obtenir à tout moment en interrogeant un service de stockage. Elle vous indique la dernière heure à laquelle les données de la région secondaire étaient cohérentes et à laquelle le service avait appliqué toutes les transactions. Dans l’exemple ci-dessus, une fois que le service insère l’entité **d’employé** dans la région secondaire, la dernière heure de synchronisation est définie sur *T1* . Elle reste définie sur *T1* jusqu’à ce que le service mette à jour l’entité **d’employé** dans la région secondaire, puis est définie sur *T6* . Si le client récupère la dernière heure de synchronisation lors de la lecture de l’entité à l’instant *T5* , il peut la comparer avec l’horodatage de l’entité. Si l’horodatage de l’entité est postérieur à la dernière heure de synchronisation, l’entité est dans un état potentiellement incohérent, et vous pouvez alors effectuer toute action appropriée pour votre application. L’utilisation de ce champ requiert que vous sachiez à quel moment a été effectuée la dernière mise à jour de la région primaire.

Pour savoir comment vérifier l’heure de la dernière synchronisation, consultez [Vérifier la propriété Heure de la dernière synchronisation pour un compte de stockage](last-sync-time-get.md).

## <a name="testing"></a>Test

Il est important de tester si votre application se comporte comme prévu lorsqu’elle rencontre des erreurs renouvelables. Par exemple, vous devez tester si l’application bascule vers la région secondaire et le mode lecture seule lorsqu’elle détecte un problème et bascule de nouveau lorsque la région primaire est à nouveau disponible. Pour ce faire, il vous faut un moyen de simuler les erreurs renouvelables et de contrôler la fréquence à laquelle elles se produisent.

Vous pouvez utiliser [Fiddler](https://www.telerik.com/fiddler) pour intercepter et modifier les réponses HTTP dans un script. Ce script peut identifier les réponses provenant de votre point de terminaison principal et remplacer le code d’état HTTP par un code que la bibliothèque cliente de stockage reconnaît comme une erreur renouvelable. Cet extrait de code offre un exemple simple de script Fiddler, qui intercepte les réponses aux demandes de lecture portant sur la table **employeedata** pour retourner un état 502 :

```java
static function OnBeforeResponse(oSession: Session) {
    ...
    if ((oSession.hostname == "\[yourstorageaccount\].table.core.windows.net")
      && (oSession.PathAndQuery.StartsWith("/employeedata?$filter"))) {
        oSession.responseCode = 502;
    }
}
```

Vous pouvez étendre cet exemple pour intercepter un plus grand nombre de demandes et modifier le **responseCode** uniquement sur certaines d’entre elles pour mieux simuler un scénario réel. Pour plus d’informations sur la personnalisation des scripts Fiddler, consultez [Modifying a Request or Response](https://docs.telerik.com/fiddler/KnowledgeBase/FiddlerScript/ModifyRequestOrResponse) (Modification d’une demande ou d’une réponse) dans la documentation Fiddler.

Si vous avez rendu configurables les seuils de basculement de votre application en mode lecture seule, il sera plus facile de tester le comportement avec des volumes de transaction hors production.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir un exemple complet montrant comment effectuer les basculements entre les points de terminaison principaux et secondaires, consultez [Azure Samples – Using the Circuit Breaker Pattern with RA-GRS storage](https://github.com/Azure-Samples/storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs) (Exemples Azure – Utilisation du modèle Disjoncteur avec le stockage RA-GRS).
