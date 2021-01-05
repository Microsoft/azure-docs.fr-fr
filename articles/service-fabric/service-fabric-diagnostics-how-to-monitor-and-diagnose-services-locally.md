---
title: Déboguer des applications Azure Service Fabric dans Windows
description: Découvrez comment analyser et diagnostiquer vos services écrits à l’aide de Microsoft Azure Service Fabric sur un ordinateur de développement local.
author: srrengar
ms.topic: conceptual
ms.date: 02/25/2019
ms.author: srrengar
ms.openlocfilehash: 58319b47c78a85b4f06c2c834db20f6c42cc1939
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86247419"
---
# <a name="monitor-and-diagnose-services-in-a-local-machine-development-setup"></a>Surveillance et diagnostic des services dans une configuration de développement d’ordinateur local
> [!div class="op_single_selector"]
> * [Windows](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)
> * [Linux](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux.md)
> 
> 

L’analyse, la détection, le diagnostic et la résolution des problèmes permettent aux services de fonctionner avec une interruption minimale de l’expérience utilisateur. Bien que l’analyse et le diagnostic soient essentiels dans un environnement de production réel déployé, leur efficacité dépend de l’adoption d’un modèle semblable pendant le développement des services pour garantir leur fonctionnement lors du passage à une configuration réelle. Service Fabric facilite pour les développeurs de service l’implémentation de diagnostics qui peuvent fonctionner parfaitement aussi bien sur une configuration de développement d’ordinateur local unique que sur une configuration réelle de cluster de production.

## <a name="event-tracing-for-windows"></a>Suivi d’événements pour Windows
[Suivi d'événements pour Windows](/windows/win32/etw/event-tracing-portal) (ETW) est la technologie recommandée pour le suivi des messages dans Service Fabric. Voici quelques avantages liés à l’utilisation d’ETW :

* **ETW est rapide.** Il a été créé en tant que technologie de suivi avec un impact minimal sur le temps d’exécution du code.
* **Le suivi ETW fonctionne parfaitement dans des environnements de développement local ainsi que dans les configurations de cluster réel.** Cela signifie que vous n’êtes pas obligé de réécrire votre code de suivi quand vous êtes prêt à déployer votre code dans un cluster réel.
* **Le code système de Service Fabric utilise également ETW pour le suivi interne.** Vous pouvez ainsi afficher les traces de votre application entrelacées avec celles du système Service Fabric. Vous pouvez également comprendre plus facilement les séquences et les relations entre votre code d’application et les événements du système sous-jacent.
* **Les outils Visual Studio de Service Fabric prennent en charge l'affichage des événements ETW.** Les événements ETW apparaissent dans l’affichage des événements de diagnostic de Visual Studio une fois que Visual Studio est correctement configuré avec Service Fabric. 

## <a name="view-service-fabric-system-events-in-visual-studio"></a>Afficher les événements système de Service Fabric dans Visual Studio
Service Fabric émet des événements ETW pour aider les développeurs d’applications à comprendre ce qui se passe dans la plateforme. Si ce n’est pas déjà fait, continuez en suivant les étapes décrites dans [Création de votre première application dans Visual Studio](service-fabric-tutorial-create-dotnet-app.md). Ces informations vous permettront de créer une application opérationnelle tout en observant les messages de trace dans la visionneuse d’événements de diagnostic.

1. Si la fenêtre des événements de diagnostics ne s’affiche pas automatiquement, accédez à l’onglet **Affichage** dans Visual Studio, puis choisissez **Autres fenêtres** et **Visionneuse d’événements de diagnostics**.
2. Chaque événement comporte des informations de métadonnées standard qui vous indiquent le nœud, l’application et le service d’où il provient. Vous pouvez également filtrer la liste des événements à l’aide de la zone **Filtrer les événements** située en haut de la fenêtre Événements. Par exemple, vous pouvez filtrer sur **Nom du nœud** ou **Nom du service**. Lorsque vous examinez les détails d’un événement, vous pouvez également mettre en pause à l’aide du bouton **Pause** en haut de la fenêtre Événements et reprendre plus tard sans perte d’événements.
   
   ![Observateur d'événements de diagnostic de Visual Studio](./media/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/DiagEventsExamples2.png)

## <a name="add-your-own-custom-traces-to-the-application-code"></a>Ajouter vos propres traces personnalisées au code d'application
Les modèles de projet Visual Studio de Service Fabric contiennent des exemples de code. Le code indique comment ajouter les traces ETW de code d’application personnalisé qui s’affichent dans la visionneuse ETW de Visual Studio avec les traces système de Service Fabric. L’avantage de cette méthode est que les métadonnées sont automatiquement ajoutées aux traces et la visionneuse d’événements de diagnostics de Visual Studio est déjà configurée pour les afficher.

Pour les projets créés à partir des **modèles de service** (sans état ou avec état), cherchez l’implémentation `RunAsync` :

1. L’appel de `ServiceEventSource.Current.ServiceMessage` in the `RunAsync` est un exemple de suivi ETW personnalisé à partir du code d’application.
2. Dans le fichier **ServiceEventSource.cs**, vous trouverez une surcharge pour la méthode `ServiceEventSource.ServiceMessage` devant être utilisée pour les événements de haute fréquence pour une question de performances.

Pour les projets créés à partir des **modèles d'acteur** (sans état ou avec état) :

1. Ouvrez le fichier **« Nom_projet ».cs** où *Nom_projet* est le nom que vous avez choisi pour votre projet Visual Studio.  
2. Recherchez le code `ActorEventSource.Current.ActorMessage(this, "Doing Work");` dans la méthode *DoWorkAsync* .  Il s'agit d'un exemple de suivi écrit ETW personnalisé à partir du code d'application.  
3. Dans le fichier **ActorEventSource.cs**, vous trouverez une surcharge pour la méthode `ActorEventSource.ActorMessage` devant être utilisée pour les événements de haute fréquence pour une question de performances.

Après avoir ajouté le suivi ETW personnalisé à votre code de service, vous pouvez générer, déployer et réexécuter l’application pour afficher vos événements dans la visionneuse d’événements de diagnostics. Si vous déboguez l’application en appuyant sur **F5**, la visionneuse d’événements de diagnostics s’ouvre automatiquement.

## <a name="next-steps"></a>Étapes suivantes
Le code de traçage que vous avez ajouté à votre application ci-dessus pour le diagnostic local fonctionnera avec les outils que vous pouvez utiliser pour afficher ces événements lors de l’exécution de votre application sur un cluster Azure. Consultez ces articles qui traitent des différentes options pour les outils et décrivent comment vous pouvez les configurer.

* [Collecte des journaux avec Diagnostics Azure](./service-fabric-diagnostics-event-aggregation-wad.md)
* [Agrégation et collection d’événements avec EventFlow](service-fabric-diagnostics-event-aggregation-eventflow.md)
