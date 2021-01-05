---
title: Profiler des applications de production dans Azure avec Application Insights Profiler
description: Identifiez le chemin réactif dans le code de votre serveur web avec un profileur de faible encombrement.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 0d3074d58560df5cb5bd6bdc2c0437a4be828918
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86499390"
---
# <a name="profile-production-applications-in-azure-with-application-insights"></a>Profiler des applications de production dans Azure avec Application Insights
## <a name="enable-application-insights-profiler-for-your-application"></a>Activer Application Insights Profiler pour votre application

Azure Application Insights Profiler fournit des traces de performances pour les applications qui s’exécutent en production dans Azure. Profiler capture automatiquement les données à grande échelle sans impact négatif sur vos utilisateurs. Profiler vous aide à identifier le chemin de code « chaud » qui prend le plus de temps pendant le traitement d’une demande web particulière. 

Profiler fonctionne avec les applications .NET déployées sur les services Azure suivants. Des instructions spécifiques pour activer Profiler pour chaque type de service sont accessibles via les liens ci-dessous.

* [Azure App Service](profiler.md?toc=/azure/azure-monitor/toc.json)
* [Azure Cloud Services](profiler-cloudservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Service Fabric](profiler-servicefabric.md?toc=/azure/azure-monitor/toc.json)
* [Machines virtuelles Azure et Virtual Machine Scale Sets](profiler-vm.md?toc=/azure/azure-monitor/toc.json)
* [**PRÉVERSION** Applications web ASP.NET Core Azure Linux](profiler-aspnetcore-linux.md?toc=/azure/azure-monitor/toc.json) 

Si vous avez activé Profiler et que vous ne voyez pas les traces, consultez notre [Guide de résolution des problèmes](profiler-troubleshooting.md?toc=/azure/azure-monitor/toc.json).

## <a name="view-profiler-data"></a>Voir les données de Profiler

Pour que Profiler charge des traces, votre application doit gérer activement les demandes. Si vous effectuez une expérience, vous pouvez générer des demandes pour votre application web à l’aide des [tests de performances Application Insights](/vsts/load-test/app-service-web-app-performance-test). Si vous venez d’activer Profiler, vous pouvez exécuter un test de charge court. Pendant que le test de charge est en cours d’exécution, sélectionnez le bouton **Profiler maintenant** dans le volet [**Paramètres de Profiler**](profiler-settings.md). Une fois Profiler en cours d’exécution, il profile au hasard une fois par heure, pendant deux minutes. Si votre application traite un flux constant de demandes, Profiler charge les traces toutes les heures.

Une fois que votre application a reçu du trafic et que Profiler a eu le temps de charger les traces, vous devez voir ces traces. Ce processus peut prendre entre 5 et 10 minutes. Pour voir les traces, dans le volet **Performances**, sélectionnez **Prendre des mesures**, puis sélectionnez le bouton **Traces de Profiler**.

![Aperçu du volet de performances Application Insights - Traces du profileur][performance-blade]

Sélectionnez un exemple pour afficher, au niveau du code, une répartition du temps d’exécution de la requête.

![Explorateur de trace d’Application Insights][trace-explorer]

L’Explorateur de trace affiche les informations suivantes :

* **Afficher le chemin réactif** : ouvre le plus grand nœud feuille, ou du moins un élément qui s’en rapproche. Dans la plupart des cas, ce nœud est proche d’un goulot d’étranglement des performances.
* **Étiquette** : nom de la fonction ou de l’événement. L’arborescence affiche un mélange de code et d’événements qui se sont produits, par exemple, des événements SQL et HTTP. L’événement supérieur représente la durée globale de la requête.
* **Écoulé** : intervalle de temps entre le début de l’opération et la fin.
* **Quand** : moment auquel la fonction ou l’événement était en cours d’exécution par rapport à d’autres fonctions.

## <a name="how-to-read-performance-data"></a>Comment lire les données de performances

Le profileur de service Microsoft utilise un ensemble de méthodes d’échantillonnage et une instrumentation pour analyser les performances de votre application. Durant la collecte détaillée, le profileur de service échantillonne le pointeur d’instruction de chaque processeur de l’ordinateur à chaque milliseconde. Chaque échantillon capture l’ensemble de la pile des appels du thread en cours d’exécution. Il fournit des informations détaillées sur l’activité de ce thread, aux deux niveaux d’abstraction élevé et bas. Le profileur de service collecte également d’autres événements afin de suivre la corrélation et la causalité des activités, tels que les événements de changement de contexte, les événements TPL (Task Parallel Library) et les événements de pool de threads.

La pile des appels présentée dans l’affichage chronologique est le résultat de l’échantillonnage et de l’instrumentation. Comme chaque échantillon capture l’ensemble de la pile des appels du thread, il inclut le code de Microsoft .NET Framework et des autres frameworks que vous référencez.

### <a name="object-allocation-clrjit_new-or-clrjit_newarr1"></a><a id="jitnewobj"></a>Allocation d’objets (clr!JIT\_Nouveau ou clr!JIT\_Newarr1)

**clr!JIT\_New** et **clr!JIT\_Newarr1** sont des fonctions d’assistance dans .NET Framework qui allouent la mémoire à partir d’un tas managé. **clr!JIT\_Nouveau** est appelé lorsqu’un objet est alloué. **clr!JIT\_Newarr1** est appelé lorsqu’un tableau d’objet est alloué. Ces deux fonctions sont généralement rapides et s’exécutent en relativement peu de temps. Si **clr!JIT\_New** ou **clr!JIT\_Newarr1** prennent un certain temps dans votre chronologie, le code alloue peut-être de nombreux objets et consomme une quantité importante de mémoire.

### <a name="loading-code-clrtheprestub"></a><a id="theprestub"></a>Code de chargement (clr!ThePreStub)

**clr!ThePreStub** est une fonction d’assistance dans .NET Framework qui prépare le code à exécuter pour la première fois. Cette exécution implique généralement au moins une compilation JIT (juste-à-temps). Pour chaque méthode C#, **clr!ThePreStub** doit être appelé au maximum une fois pendant un processus.

Si **clr!ThePreStub** prend beaucoup de temps pour traiter une demande, la demande est la première à exécuter cette méthode. Le runtime du .NET Framework a besoin de temps pour charger la première méthode. Vous pouvez envisager d’utiliser un processus de mise en route qui exécute cette partie du code avant que vos utilisateurs y accèdent, ou encore envisager d’exécuter Native Image Generator (ngen.exe) sur vos assemblys.

### <a name="lock-contention-clrjitutil_moncontention-or-clrjitutil_monenterworker"></a><a id="lockcontention"></a>Contention de verrouillage (clr!JITutil\_MonContention or clr!JITutil\_MonEnterWorker)

**clr!JITutil\_MonContention** ou **clr!JITutil\_MonEnterWorker** indique que le thread actuel est en attente d’ouverture d’un verrou. Ce texte est souvent affiché quand vous exécutez une instruction **LOCK** C#, que vous appelez la méthode **Monitor.Enter** ou une méthode avec l’attribut **MethodImplOptions.Synchronized**. La contention de verrouillage se produit généralement quand le thread _A_ acquiert un verrou et que le thread _B_ tente d’acquérir le même verrou avant qu’il ne soit libéré par le thread _A_.

### <a name="loading-code-cold"></a><a id="ngencold"></a>Code de chargement ([COLD])

Si le nom de la méthode contient **[COLD]** , par exemple, **mscorlib.ni![COLD]System.Reflection.CustomAttribute.IsDefined**, le runtime du .NET Framework exécute pour la première fois du code qui n’utilise pas d’[optimisation guidée par profil](/cpp/build/profile-guided-optimizations). Pour chaque méthode, il doit s’afficher au maximum une fois pendant le processus.

Si le chargement du code prend beaucoup de temps pour une demande, la demande est la première à exécuter la partie non optimisée de la méthode. Vous pouvez envisager d’utiliser un processus de mise en route qui exécute cette partie du code avant que vos utilisateurs y accèdent.

### <a name="send-http-request"></a><a id="httpclientsend"></a>Envoyer une requête HTTP

Les méthodes comme **HttpClient.Send** indiquent que le code attend la fin d’une requête HTTP.

### <a name="database-operation"></a><a id="sqlcommand"></a>Opération de base de données

Une méthode comme **SqlCommand.Execute** indique que le code attend la fin d’une opération de base de données.

### <a name="waiting-await_time"></a><a id="await"></a>En attente (AWAIT\_TIME)

**AWAIT\_TIME** indique que le code attend la fin de l’exécution d’une autre tâche. Ce délai se produit généralement avec l’instruction C# **AWAIT**. Lorsque le code exécute une instruction C# **AWAIT**, le thread se déroule et redonne le contrôle au pool de threads, et aucun thread n’est bloqué en attendant la fin de l’exécution de l’instruction **AWAIT**. Toutefois, le thread qui a exécuté l’instruction **AWAIT** est logiquement « bloqué » en attendant la fin de l’opération. L’instruction **AWAIT\_TIME** indique le temps de blocage en attendant l’exécution de la tâche.

### <a name="blocked-time"></a><a id="block"></a>Temps de blocage

**BLOCKED_TIME** indique que le code attend qu’une autre ressource soit disponible. Il peut, par exemple, attendre un objet de synchronisation, la disponibilité d’un thread ou la fin d’une requête.

### <a name="unmanaged-async"></a>Asynchrone non managé

.NET Framework émet des événements ETW et transmet les ID d’activité entre les threads pour permettre le suivi des appels asynchrones. Le code non managé (code natif) et les styles de code asynchrone plus anciens ne détectent ni ces événements ni les ID d'activité et dès lors, le profileur n'est pas en mesure de déterminer le thread ou les fonctions en cours d'exécution sur ce thread. Cela est indiqué par « Asynchrone non managé » dans la pile des appels. En téléchargeant le fichier ETW, vous devriez pouvoir utiliser [PerfView](https://github.com/Microsoft/perfview/blob/master/documentation/Downloading.md) pour obtenir plus d’informations sur ce qui se passe.

### <a name="cpu-time"></a><a id="cpu"></a>Temps processeur

Le processeur est occupé à exécuter les instructions.

### <a name="disk-time"></a><a id="disk"></a>Temps du disque

L’application effectue les opérations de disque.

### <a name="network-time"></a><a id="network"></a>Temps réseau

L’application effectue les opérations réseau.

### <a name="when-column"></a><a id="when"></a>Colonne « When » (Quand)

La colonne **When** permet de voir comment les exemples INCLUSIFS collectés pour un nœud varient au fil du temps. La plage totale de la requête est divisée en 32 périodes. Les exemples inclusifs de ce nœud sont regroupés dans ces 32 périodes. Chaque période est représentée sous forme de barre. La hauteur de la barre représente une valeur à l’échelle. Pour les nœuds marqués **CPU_TIME** ou **BLOCKED_TIME**, ou quand il existe une relation évidente de consommation d’une ressource (par exemple, processeur, disque, thread), la barre représente la consommation d’une des ressources au cours de cette période. Pour ces mesures, il est possible d’obtenir une valeur supérieure à 100 % en consommant plusieurs ressources. Par exemple, si vous utilisez en moyenne deux processeurs sur un intervalle donné, vous obtenez 200 %.

## <a name="limitations"></a>Limites

Par défaut, la période de conservation des données est de 5 jours. La quantité maximale de données ingérées par jour est de 10 Go.

L’utilisation du service Profiler est gratuite. Pour l’utiliser, votre application web doit être hébergée au moins au niveau De base de la fonctionnalité Web Apps d’Azure App Service.

## <a name="overhead-and-sampling-algorithm"></a>Surcharge et algorithme d’échantillonnage

Profiler s’exécute aléatoirement pendant 2 minutes toutes les heures sur chaque machine virtuelle qui héberge l’application sur laquelle Profiler est activé pour capturer des traces. Quand Profiler est en cours d’exécution, il ajoute entre 5 et 15 % de surcharge de processeur sur le serveur.

## <a name="next-steps"></a>Étapes suivantes
Activer Application Insights Profiler pour votre application Azure. Voir aussi :
* [Services d’application](profiler.md?toc=/azure/azure-monitor/toc.json)
* [Azure Cloud Services](profiler-cloudservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Service Fabric](profiler-servicefabric.md?toc=/azure/azure-monitor/toc.json)
* [Machines virtuelles Azure et Virtual Machine Scale Sets](profiler-vm.md?toc=/azure/azure-monitor/toc.json)


[performance-blade]: ./media/profiler-overview/performance-blade-v2-examples.png
[trace-explorer]: ./media/profiler-overview/trace-explorer.png
