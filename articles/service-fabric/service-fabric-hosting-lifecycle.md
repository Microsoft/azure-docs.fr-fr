---
title: Cycle de vie de l’activation et de la désactivation de l’hébergement d’Azure Service Fabric
description: Explique le cycle de vie d’application et de ServicePackage sur un nœud
author: tugup
ms.topic: conceptual
ms.date: 05/1/2020
ms.author: tugup
ms.openlocfilehash: f049b19703d37412d1ee1961aee6cb327efabe7c
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779598"
---
# <a name="azure-service-fabric-hosting-lifecycle"></a>Cycle de vie de l’hébergement d’Azure Service Fabric

Cet article fournit une vue d’ensemble des événements qui se produisent dans Azure Service Fabric quand une application est activée sur un nœud et différentes configurations de cluster utilisées pour contrôler le comportement.

Avant de continuer, assurez-vous de bien avoir compris les différents concepts et relations expliqués dans [Modéliser une application dans Service Fabric][a1]. 

> [!NOTE]
> Dans cet article, à moins qu’un sens différent n’ait été explicitement fourni :
>
> - Le terme *réplica* fait référence à la fois au réplica d’un service avec état et à une instance d’un service sans état.
> - *CodePackage* est traité comme étant équivalent au processus *ServiceHost*, qui inscrit un *ServiceType*, et héberge des réplicas de services de ce *ServiceType*.
>

## <a name="activation-of-applicationservicepackage"></a>Activation d’Application/ServicePackage

Le pipeline pour l’activation est le suivant :

1. Téléchargez l’ApplicationPackage. Par exemple : ApplicationManifest.xml, etc.
2. Configurez l’environnement pour Application, par exemple créez des utilisateurs, etc.
3. Démarrez le suivi d’Application pour la désactivation.
4. Téléchargez le ServicePackage. Par exemple : ServiceManifest.xml, le code, la configuration, les packages de données, etc.
5. Configurez l’environnement pour le package de service, par exemple : configurez le pare-feu, allouez des ports pour les points de terminaison, etc.
6. Démarrez le suivi de ServicePackage pour la désactivation.
7. Démarrez le point d’entrée de configuration des CodePackages et attendez la fin de l’opération.
8. Démarrez MainEntryPoint de CodePackages.

### <a name="servicetype-blocklisting"></a>Blocage de ServiceType
**ServiceTypeDisableFailureThreshold** détermine le nombre d’échecs (activation, téléchargement, échecs de CodePackage) après lequel il est prévu de bloquer le ServiceType. Le premier échec d’activation, échec de téléchargement ou incident CodePackage planifie le blocage ServiceType. La configuration de **ServiceTypeDisableGraceInterval** détermine l’intervalle de grâce après lequel ServiceType est marqué comme bloqué sur ce nœud. Pendant que tout cela se produit, l’activation, le téléchargement et le redémarrage de CodePackage sont retentés en parallèle. Une nouvelle tentative signifie, par exemple, que le redémarrage de CodePackage sera planifié après l’incident ou que Service Fabric réessayera de télécharger les packages.

Lorsque ServiceType est bloqué, vous verrez une erreur d’intégrité « "System.Hosting" a signalé une erreur pour la propriété "ServiceTypeRegistration:ServiceType". Le ServiceType a été désactivé sur le nœud. »

ServiceType est réactivé sur le nœud si l’un des événements suivants se produit :
- L’activation réussit ou atteint les nouvelles tentatives **ActivationMaxFailureCount** en cas d’échec.
- Le téléchargement réussit ou atteint les nouvelles tentatives **DeploymentMaxFailureCount** en cas d’échec.
- Un CodePackage qui a rencontré un incident démarre et inscrit correctement le ServiceType.

**ActivationMaxFailureCount** et **DeploymentMaxFailureCount** représente le nombre maximal de tentatives que Service Fabric effectuera pour activer/télécharger une application sur un nœud, après quoi Service Fabric réactivera ServiceType pour l’activation. Il s’agit d’accorder au service une autre possibilité d’activation, laquelle peut aboutir, ce qui entraîne la réparation automatique du problème. Une nouvelle opération d’activation ou de téléchargement déclenchée par le placement et l’activation d’un réplica peut déclencher à nouveau le blocage de ServiceType ou peut réussir.

> [!NOTE]
> Si votre CodePackage qui n’inscrit pas un ServiceType plante, il n’a pas d’impact sur le ServiceType. Seul le CodePackage hébergeant un plantage de réplica aura un impact sur le ServiceType.
>

### <a name="codepackage-crash"></a>Plantage de CodePackage
Lorsqu’un CodePackage se plante, Service Fabric utilise une interruption pour le redémarrer. L’interruption est indépendante du fait que le package de code ait ou non inscrit un type avec le runtime Service Fabric.

La valeur de l’interruption est Min(RetryTime, **ActivationMaxRetryInterval**), et cette valeur est incrémentée de manière constante, linéaire ou exponentielle en fonction du paramètre de configuration **ActivationRetryBackoffExponentiationBase**.

- Constante : Si **ActivationRetryBackoffExponentiationBase** == 0, RetryTime = **ActivationRetryBackoffInterval**
- Linéaire :  Si **ActivationRetryBackoffExponentiationBase** == 0, RetryTime = ContinuousFailureCount * **ActivationRetryBackoffInterval**, où ContinousFailureCount est le nombre de fois où un CodePackage plante ou échoue à s’activer.
- Exponentielle : RetryTime = (**ActivationRetryBackoffInterval** en secondes) * (**ActivationRetryBackoffExponentiationBase** ^ ContinuousFailureCount)
    
Vous pouvez contrôler le comportement en modifiant les valeurs. Par exemple, si vous souhaitez effectuer plusieurs tentatives de redémarrage rapide, vous pouvez utiliser un incrément linéaire en définissant **ActivationRetryBackoffExponentiationBase** sur 0 et **ActivationRetryBackoffInterval** sur 10. Avec ces paramètres, si un CodePackage se plante, l’intervalle de début commencera après dix secondes. Si le package a continué à se planter, l’interruption passe à 20, 30, 40 secondes, et ainsi de suite, jusqu’à ce que l’activation de CodePackage ait réussi ou que le package de code ait été désactivé. 
    
La durée maximale pendant laquelle Service Fabric s’interrompt (attend) après une défaillance est régie par **ActivationMaxRetryInterval**.
    
Si votre CodePackage se plante et se rétablit, il doit rester opérationnel pendant **CodePackageContinuousExitFailureResetInterval** pour que Service Fabric le considère comme étant sain, après quoi il remplace le rapport d’intégrité Erreur précédent par OK et réinitialise ContinousFailureCount.

### <a name="codepackage-not-registering-servicetype"></a>CodePackage n’inscrivant pas ServiceType
Si un CodePackage reste actif et qu’il est censé inscrire un ServiceType pour nous mais ne le fait pas, Service Fabric génère une rapport d’intégrité avec un avertissement après **ServiceTypeRegistrationTimeout**, indiquant que ServiceType n’a pas été inscrit dans le délai imparti.

### <a name="activation-failure"></a>Échec de l’activation
Service Fabric utilise toujours une interruption linéaire (comme pour un plantage du CodePackage) quand il trouve une erreur pendant l’activation. Cela signifie que l’opération d’activation abandonnera après (0 + 10 + 20+ 30 + 40) = 100 secondes (la première nouvelle tentative étant immédiate). Après cela, il n’y a plus de nouvelle tentative d’activation.
    
La durée maximale de l’interruption peut être **ActivationMaxRetryInterval** et le nombre maximal de nouvelles tentatives peut être **ActivationMaxFailureCount**.

### <a name="download-failure"></a>Échec du téléchargement
Service Fabric utilise toujours une interruption linéaire quand il rencontre une erreur pendant le téléchargement. Cela signifie que l’opération d’activation abandonnera après (0 + 10 + 20 + 30 + 40) = 100 secondes (la première nouvelle tentative étant immédiate). Après cela, il n’y a plus de nouvelles tentatives de téléchargement. L’interruption linéaire pour le téléchargement est égale à ContinuousFailureCount**_DeploymentRetryBackoffInterval_* et la durée maximale d’interruption peut être **DeploymentMaxRetryInterval**. À l’instar des activations, l’opération de téléchargement peut faire l’objet de **ActivationMaxFailureCount** nouvelles tentatives.

> [!NOTE]
> Avant de modifier ces paramètres, voici quelques exemples que vous devez garder à l’esprit.

* Si le CodePackage continue de se planter et s’interrompt, ServiceType est désactivé. Cependant, si la configuration d’activation est telle qu’elle a un redémarrage rapide, le CodePackage peut se trouver opérationnel à quelques reprises avant que ServiceType soit bloqué. Par exemple, supposons que votre CodePackage soit opérationnel, qu’il inscrive le ServiceType auprès de Service Fabric, puis qu’il plante. Dans ce cas, une fois que l’hébergement reçoit une inscription de type, la période **ServiceTypeDisableGraceInterval** est annulée. Et ceci peut se répéter jusqu’à ce que votre CodePackage s’interrompe à une valeur supérieure à **ServiceTypeDisableGraceInterval** : ServiceType sera bloqué sur le nœud. Cela peut prendre plus de temps que vous ne le pensez avant de voir ServiceType bloqué.

* Dans le cas d’activations, quand le système Service Fabric doit placer un réplica sur un nœud, RA (ReconfigurationAgent) demande au sous-système d’hébergement d’activer l’application et réessaye la demande d’activation toutes les 15 secondes (régie par le paramètre de configuration **RAPMessageRetryInterval**). Pour que Service Fabric sache que ServiceType a été bloqué, l’opération d’activation dans l’hébergement doit être active pendant une période plus longue que l’intervalle de nouvelle tentative et **ServiceTypeDisableGraceInterval**. Par exemple, supposons que le cluster a **ActivationMaxFailureCount** défini sur 5 et **ActivationRetryBackoffInterval** défini sur 1 seconde. Cela signifie que l’opération d’activation abandonnera après (0 + 1 + 2 + 3 + 4) = 10 secondes (rappelez-vous que la première nouvelle tentative est immédiate) et qu’après cela, l’hébergement abandonnera toute nouvelle tentative. Dans ce cas, l’opération d’activation sera terminée et il n’y aura pas de nouvelle tentative au bout de 15 secondes. Cela se produit parce que Service Fabric a épuisé toutes les nouvelles tentatives autorisées dans un délai de 15 secondes. Ainsi, chaque nouvelle tentative de ReconfigurationAgent crée une nouvelle opération d’activation dans le sous-système d’hébergement, et le modèle continue à se répéter. Par conséquent, ServiceType ne sera jamais bloqué sur le nœud. Étant donné que ServiceType ne sera pas bloqué sur le nœud, le réplica ne sera pas déplacé et essayé sur un nœud différent.
> 

## <a name="deactivation"></a>Désactivation

Quand un ServicePackage est activé sur un nœud, il est suivi pour la désactivation. 

La désactivation fonctionne de deux manières :

- Périodiquement :  À chaque **DeactivationScanInterval**, il recherche les ServicePackages qui n’ont JAMAIS hébergé de réplica et les marque comme étant des candidats à la désactivation.
- ReplicaClose : Si un réplica est fermé, le désactivateur obtient un DecrementUsageCount. Si le compteur atteint 0, cela signifie que le ServicePackage n’héberge aucun réplica et qu’il est donc candidat à la désactivation.

 En fonction du mode d’activation [exclusif/partagé][a2], les candidats à la désactivation sont planifiés après **DeactivationGraceInterval** pour SharedMode et après **ExclusiveModeDeactivationGraceInterval** pour ExclusiveMode. Si un nouveau placement de réplica se produit dans cet intervalle, la désactivation est annulée.

### <a name="periodically"></a>Périodiquement :
Nous allons aborder quelques exemples de désactivation périodique :

Exemple 1 : Supposons que le désactivateur effectue une recherche à un instant T (**DeactivationScanInterval**). Sa recherche suivante se fera à l’instant 2T. Supposons qu’une activation de ServicePackage se soit produite en T+1. Ce ServicePackage n’héberge pas de réplica et doit donc être désactivé. Pour que le ServicePackage soit un candidat à la désactivation, il doit être dans l’état sans réplica pour une durée d’au moins T. Cela signifie qu’il sera éligible pour la désactivation à 2T+1. Ainsi, la recherche en 2T ne trouve pas ce ServicePackage comme étant candidat pour la désactivation. Le cycle de désactivation suivant en 3T va planifier ce ServicePackage pour la désactivation, car à ce moment, il a été dans l’état sans réplica pour le moment T.  

Exemple 2 : Supposons qu’un ServicePackage soit activé à l’instant T-1 et que le désactivateur effectue une recherche à l’instant T. ServicePackage n’héberge pas de réplica. Ensuite, à la recherche suivante à l’instant 2T, ce ServicePackage sera trouvé comme candidat à la désactivation et sera donc planifié pour la désactivation.  

Exemple 3 : Supposons qu’un ServicePackage soit activé à l’instant T-1 et que le désactivateur effectue une recherche à l’instant T. ServicePackage n’héberge pas encore de réplica. Maintenant, à T+1, un réplica est placé, c’est-à-dire que l’hébergement obtient un IncrementUsageCount, ce qui signifie qu’un réplica est créé. En 2T, ce ServicePackage ne sera pas planifié pour la désactivation. Étant donné qu’il contient un réplica, la désactivation sera déplacée vers la logique ReplicaClose expliquée ci-dessous.

Exemple 4 : Supposons que votre ServicePackage soit de grande taille, par exemple 10 Go, et que le téléchargement sur le nœud peut donc durer un certain temps. Une fois qu’une application est activée, le désactivateur effectue le suivi de son cycle de vie. Maintenant, si la configuration de **DeactivationScanInterval** est définie sur une valeur faible, vous pouvez rencontrer des problèmes dans la mesure où votre ServicePackage n’a pas suffisamment de temps pour s’activer sur le nœud, car tout le temps a été passé en téléchargement. Pour résoudre le problème, vous pouvez [prétélécharger le ServicePackage sur le nœud][p1] ou augmenter **DeactivationScanInterval**. 

> [!NOTE]
> Un ServicePackage peut être désactivé n’importe quand entre (**DeactivationScanInterval** à 2***_DeactivationScanInterval_*) + **DeactivationGraceInterval**/** ExclusiveModeDeactivationGraceInterval**. 
>

### <a name="replica-close"></a>Fermeture de réplica :
La désactivation tient le compte du nombre de réplicas qu’un ServicePackage contient. Si un ServicePackage contient un réplica et que le réplica est fermé/arrêté, l’hébergement reçoit un DecrementUsageCount. Quand un réplica est ouvert, l’hébergement reçoit un IncrementUsageCount. La décrémentation signifie que le ServicePackage héberge maintenant un réplica de moins et, quand le nombre tombe à 0, le ServicePackage est planifié pour la désactivation, et l’intervalle de temps après lequel il sera désactivé est **DeactivationGraceInterval**/**ExclusiveModeDeactivationGraceInterval**. 

Par exemple, supposons qu’une décrémentation se produise à l’instant T et que ServicePackage soit planifié pour la désactivation en 2T+X(**DeactivationGraceInterval**/**ExclusiveModeDeactivationGraceInterval**). Si pendant cette période, l’hébergement reçoit un IncrementUsage signifiant qu’un réplica est créé, la désactivation est annulée.

> [!NOTE]
> Que signifient ces paramètres de configuration ?
**DeactivationGraceInterval**/**ExclusiveModeDeactivationGraceInterval** : Le temps accordé à un ServicePackage pour héberger à nouveau un autre réplica quand il n’héberge plus aucun réplica. 
**DeactivationScanInterval** : Le temps minimal accordé à ServicePackage pour héberger un réplica s’il n’a JAMAIS hébergé de réplica, c’est-à-dire s’il n’est pas utilisé.
>

### <a name="ctrlc"></a>Ctrl+C
Une fois qu’un ServicePackage dépasse **DeactivationGraceInterval**/**ExclusiveModeDeactivationGraceInterval** et qu’il n’héberge toujours pas de réplica, la désactivation n’est pas annulable. Les CodePackage ont un gestionnaire Ctrl+C, ce qui signifie que le pipeline de désactivation doit en tenir compte pour arrêter le processus. Pendant ce temps, si un nouveau réplica pour le même ServicePackage tente d’être placé, il échoue, car nous ne pouvons pas passer de la désactivation à l’activation.

## <a name="cluster-configs"></a>Configurations de cluster

Les configurations avec des valeurs par défaut impactent l’activation/désactivation.

### <a name="servicetype"></a>ServiceType
**ServiceTypeDisableFailureThreshold** : la valeur par défaut est 1. Le seuil pour le nombre d’échecs après lequel FM (FailoverManager) reçoit une notification lui indiquant de désactiver le type de service sur ce nœud et d’essayer un autre nœud pour le placement.
**ServiceTypeDisableGraceInterval** : La valeur par défaut est de 30 secondes. C’est l’intervalle de temps après lequel le type de service peut être désactivé.
**ServiceTypeRegistrationTimeout** : La valeur par défaut est de 300 secondes. Il s’agit du délai d’expiration pour l’inscription de ServiceType auprès de Service Fabric.

### <a name="activation"></a>Activation
**ActivationRetryBackoffInterval** : La valeur par défaut est de 10 secondes. Il s’agit de l’intervalle d’interruption sur chaque échec d’activation.
**ActivationMaxFailureCount** : La valeur par défaut est de 20. Nombre maximal de nouvelles tentatives d’activation en échec par le système avant abandon. 
**ActivationRetryBackoffExponentiationBase** : La valeur par défaut est 1,5.
**ActivationMaxRetryInterval** : La valeur par défaut est de 3 600 secondes. Il s’agit de la durée d’interruption maximale pour l’activation en cas d’échec.
**CodePackageContinuousExitFailureResetInterval** : La valeur par défaut 300 secondes. Il s’agit du délai d’expiration pour réinitialiser le nombre d’échecs de sortie continue pour CodePackage.

### <a name="download"></a>Téléchargement
**DeploymentRetryBackoffInterval** : Valeur par défaut : 10. Intervalle de temporisation pour l’échec du déploiement.
**DeploymentMaxRetryInterval** : La valeur par défaut est de 3600 secondes. Il s’agit de la durée d’interruption maximale pour le déploiement en cas d’échec.
**DeploymentMaxFailureCount** : La valeur par défaut est de 20. Le déploiement de l’application sera retenté DeploymentMaxFailureCount fois avant l’échec du déploiement de cette application sur le nœud.

### <a name="deactivation"></a>Désactivation
**DeactivationScanInterval** : La valeur par défaut est de 600 secondes. Il s’agit du temps minimal accordé à ServicePackage pour héberger un réplica s’il n’a jamais hébergé de réplica, c’est-à-dire s’il n’est pas utilisé.
**DeactivationGraceInterval** : La valeur par défaut est de 60 secondes. Il s’agit du temps accordé à un ServicePackage pour héberger à nouveau un autre réplica quand il n’héberge plus aucun réplica dans le cas du modèle de processus **Partagé**.
**ExclusiveModeDeactivationGraceInterval** : La valeur par défaut est de 1 seconde. Il s’agit du temps accordé à un ServicePackage pour héberger à nouveau un autre réplica quand il n’héberge plus aucun réplica dans le cas du modèle de processus **Exclusif**.

## <a name="next-steps"></a>Étapes suivantes
[Empaquetez une application][a3] et préparez-la pour le déploiement.

[Déployer et supprimer des applications][a4]. Cet article explique comment utiliser PowerShell pour gérer des instances d’application.

<!--Link references--In actual articles, you only need a single period before the slash-->
[a1]: service-fabric-application-model.md
[a2]: service-fabric-hosting-model.md
[a3]: service-fabric-package-apps.md
[a4]: service-fabric-deploy-remove-applications.md

[p1]: /powershell/module/servicefabric/copy-servicefabricservicepackagetonode
