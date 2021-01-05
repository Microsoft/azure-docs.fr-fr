---
title: Workflow de l'architecture de machine virtuelle Windows Azure | Microsoft Docs
description: Cet article présente les processus de workflow lorsque vous déployez un service.
services: cloud-services
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue
ms.assetid: 9f2af8dd-2012-4b36-9dd5-19bf6a67e47d
ms.service: cloud-services
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 04/08/2019
ms.author: kwill
ms.openlocfilehash: d29c98ecbbb6c9da18e6356a0e38122e253a34b6
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93026460"
---
#    <a name="workflow-of-windows-azure-classic-vm-architecture"></a>Workflow de l’architecture de machine virtuelle Windows Azure 
Cet article présente les processus de workflow qui interviennent lorsque vous déployez ou mettez à jour une ressource Azure telle qu'une machine virtuelle. 

> [!NOTE]
>Azure a deux modèles de déploiement différents pour créer et utiliser des ressources : Resource Manager et le déploiement classique. Cet article traite du modèle de déploiement classique.

Le diagramme suivant présente l’architecture des ressources Azure.

:::image type="content" source="./media/cloud-services-workflow-process/workflow.jpg" alt-text="<alt Image de flux de travail Azure>":::

## <a name="workflow-basics"></a>Concepts de workflow de base
   
**R**. RDFE/FFE correspond au chemin de communication entre l'utilisateur et la structure. RDFE (Front-End de RedDog) correspond à l’API exposée publiquement qui fait office de frontend vers le portail de gestion et l'API Gestion des services comme Visual Studio, Azure MMC, etc.  Toutes les requêtes émanant de l'utilisateur transitent par RDFE. FFE (Fabric Front-End) correspond à la couche qui translate les requêtes entre RDFE et les commandes de structure. Toutes les requêtes émanant de RDFE transitent par FFE avant d'atteindre les contrôleurs de structure.

**B**. Le contrôleur de structure est responsable de la maintenance et de la surveillance de toutes les ressources du centre de données. Il communique avec les agents hôtes de structure sur le système d'exploitation de la structure et envoie des informations telles que la version du système d’exploitation invité, le package de service, la configuration de service et l'état de service.

**C**. L’agent hôte, qui se trouve sur le système d’exploitation hôte, est chargé de configurer le système d’exploitation invité et de communiquer avec l’agent invité (WindowsAzureGuestAgent) afin de mettre à jour le rôle vers un état cible déterminé et de procéder à des vérifications de pulsation auprès de l’agent invité. Si l’agent hôte ne reçoit pas de réponse de pulsation pendant 10 minutes, il redémarre le système d’exploitation invité.

**C2**. L'agent WaAppAgent est responsable de l’installation, de la configuration et de la mise à jour de WindowsAzureGuestAgent.exe.

**D**.  L'agent WindowsAzureGuestAgent est responsable des tâches suivantes :

1. Configuration du système d’exploitation invité, notamment du pare-feu, des listes de contrôle d’accès, des ressources LocalStorage, du package des services et de leur configuration ainsi que des certificats.
2. Configuration du SID pour le compte d’utilisateur sous lequel le rôle s’exécutera.
3. Communication de l'état du rôle à la structure.
4. Démarrage et surveillance de WaHostBootstrapper pour s'assurer que le rôle est dans l’état cible.

**E**. WaHostBootstrapper est responsable de ce qui suit :

1. Lecture de la configuration de rôle et démarrage des tâches et processus appropriés pour configurer et exécuter le rôle.
2. Surveillance de tous ses processus enfants.
3. Déclenchement de l’événement StatusCheck sur le processus hôte du rôle.

**F**. IISConfigurator s’exécute si le rôle est configuré en tant que rôle web IIS complet. Il est responsable de ce qui suit :

1. Démarrage des services IIS standard
2. Configuration du module de réécriture dans la configuration web
3. Configuration du pool d’applications pour le rôle configuré dans le modèle de service
4. Configuration de la journalisation IIS pour pointer vers le dossier DiagnosticStore LocalStorage
5. Configuration des autorisations et ACL
6. Le site web se trouve dans %roleroot%:\sitesroot\0 et le pool d’applications pointe vers cet emplacement pour exécuter IIS. 

**G**. Les tâches de démarrage sont définies par le modèle de rôle et démarrées par WaHostBootstrapper. Les tâches de démarrage peuvent être configurées pour s'exécuter de façon asynchrone, en arrière-plan. Le programme d’amorçage hôte démarre la tâche de démarrage, puis poursuit les autres tâches de démarrage. Les tâches de démarrage peuvent également être configurées pour s’exécuter en mode Simple (par défaut). Dans ce mode, le programme d’amorçage hôte attend que la tâche de démarrage termine l'exécution et renvoie un code de sortie de réussite (0) avant de passer à la tâche de démarrage suivante.

**H**. Ces tâches font partie du kit de développement logiciel (SDK) et sont définies en tant que plug-ins dans la définition de service du rôle (.csdef). Étendus aux tâches de démarrage, les agents **DiagnosticsAgent** et **RemoteAccessAgent** sont uniques en ce sens qu'ils définissent chacun deux tâches de démarrage, un démarrage normal et un démarrage avec le paramètre **/blockStartup**. La tâche de démarrage normal est définie en tant que tâche de démarrage en arrière-plan pour permettre son exécution en arrière-plan pendant que le rôle lui-même s'exécute. La tâche de démarrage **/blockStartup** est définie en tant que tâche de démarrage simple pour permettre à WaHostBootstrapper d'attendre sa fermeture avant de poursuivre. La tâche **/blockStartup** attend que la tâche normale finisse de s'initialiser, puis se ferme et autorise le programme d’amorçage hôte à poursuivre. Cela permet de configurer les diagnostics et l'accès au protocole RDP (Remote Desktop Protocol) avant de démarrer les processus de rôle (via la tâche /blockStartup). Cela permet également aux diagnostics et à l'accès au protocole RDP (Remote Desktop Protocol) de continuer de s'exécuter après que le programme d’amorçage de l’hôte a terminé les tâches de démarrage (via une tâche normale).

**I**. WaWorkerHost est le processus hôte standard pour les rôles de travail normaux. Ce processus hôte héberge les DLL et le code de point d'entrée du rôle, comme OnStart et Run.

**J**. WaIISHost est le processus hôte du code de point d’entrée pour les rôles web qui utilisent IIS en intégralité. Ce processus charge le premier fichier DLL trouvé qui utilise la classe **RoleEntryPoint** et exécute le code à partir de cette classe (OnStart, Run, OnStop). Tout événement **RoleEnvironment** (par exemple, StatusCheck et Changed) créé dans la classe RoleEntryPoint est déclenché dans ce processus.

**K**. W3wp est le processus de travail IIS standard utilisé si le rôle est configuré pour utiliser IIS en intégralité. Il exécute le pool d’applications configuré à partir d'IISConfigurator. Tout événement RoleEnvironment (par exemple, StatusCheck et Changed) créé est déclenché dans ce processus. Notez que les événements RoleEnvironment sont déclenchés dans les deux emplacements (WaIISHost et w3wp.exe) si vous vous inscrivez à des événements dans les deux processus.

## <a name="workflow-processes"></a>Processus de workflow

1. Un utilisateur formule une requête, par exemple télécharger des fichiers « .cspkg » et « .cscfg », demander à une ressource de s’arrêter ou apporter une modification à la configuration, etc. Cela peut se faire via le portail Azure ou un outil utilisant l’API Gestion des services, tel que la fonctionnalité de publication Visual Studio. Cette requête est transmise à RDFE qui effectue toutes les tâches relatives à l’abonnement, puis transmet la requête à FFE. La suite de ces étapes de workflow consiste à déployer un nouveau package et à le démarrer.
2. FFE recherche le pool de machines qui convient (en fonction de l'entrée du client, comme groupe d'affinités, et de l'entrée de la structure, comme disponibilité des machines) et communique avec le contrôleur de structure principal dans ce pool d’ordinateurs.
3. Le contrôleur de structure détecte un hôte doté de cœurs d'unité centrale disponibles (ou prépare un nouvel hôte). Le package de service et la configuration sont copiés dans l'hôte, et le contrôleur de structure communique avec l’agent hôte sur le système d’exploitation hôte pour déployer le package (configuration des DIP, des ports, du système d'exploitation invité, etc.).
4. L’agent hôte démarre le système d’exploitation invité et communique avec l’agent invité (WindowsAzureGuestAgent). L’hôte envoie des pulsations à l’invité pour s'assurer que le rôle fonctionne dans l'état cible.
5. WindowsAzureGuestAgent configure le système d’exploitation invité (pare-feu, ACL, LocalStorage, etc.), copie un nouveau fichier de configuration XML vers C:\Config, puis démarre le processus WaHostBootstrapper.
6. Pour les rôles web IIS complets, WaHostBootstrapper démarre IISConfigurator et lui indique de supprimer tout pool d'applications existant pour le rôle web dans IIS.
7. WaHostBootstrapper lit les tâches **Startup** depuis E:\RoleModel.xml et commence à exécuter les tâches de démarrage. WaHostBootstrapper attend la fin de toutes les tâches de démarrage simple et renvoie un message de « réussite ».
8. Pour les rôles web IIS complets, WaHostBootstrapper indique à IISConfigurator de configurer le pool d’applications IIS et pointe le site vers `E:\Sitesroot\<index>`, où `<index>` correspond à un index basé sur 0 dans le nombre de `<Sites>` éléments définis pour le service.
9. WaHostBootstrapper démarre le processus hôte en fonction du type de rôle :
    1. **Rôle de travail**  : WaWorkerHost.exe est démarré. WaHostBootstrapper exécute la méthode OnStart(). Une fois les résultats retournés, WaHostBootstrapper commence à exécuter la méthode Run(), puis marque simultanément le rôle comme étant Prêt et le place dans la rotation de l’équilibreur de charge (si des InputEndpoints sont définis). WaHostBootsrapper entre ensuite dans une boucle de vérification d'état du rôle.
    2. **Rôle web IIS complet**  : aIISHost est démarré. WaHostBootstrapper exécute la méthode OnStart(). Ensuite, il commence à exécuter la méthode Run(), puis marque simultanément le rôle comme Prêt et le place dans la rotation d’équilibrage de charge. WaHostBootsrapper entre ensuite dans une boucle de vérification d'état du rôle.
10. Les requêtes web entrantes vers un rôle web IIS complet déclenchent IIS pour démarrer le processus W3WP et répondre à la requête, comme dans un environnement IIS local.

## <a name="log-file-locations"></a>Emplacements des fichiers journaux

**WindowsAzureGuestAgent**

- C:\Logs\AppAgentRuntime.Log.  
Ce journal contient les modifications apportées au service, y compris les démarrages, arrêts et nouvelles configurations. Si aucune modification n'est apportée au service, attendez-vous à constater d'importants écarts de temps dans ce fichier journal.
- C:\Logs\WaAppAgent.Log.  
Ce journal contient les mises à jour d'état ainsi que les notifications de pulsation, et est mis à jour toutes les 2 à 3 secondes.  Ce journal contient une vue d'historique d'état de l’instance et vous indique quand l’instance n’était pas dans l’état Prêt.
 
**WaHostBootstrapper**

`C:\Resources\Directory\<deploymentID>.<role>.DiagnosticStore\WaHostBootstrapper.log`
 
**WaIISHost**

`C:\Resources\Directory\<deploymentID>.<role>\WaIISHost.log`
 
**IISConfigurator**

`C:\Resources\Directory\<deploymentID>.<role>\IISConfigurator.log`
 
**Journaux d’activité IIS**

`C:\Resources\Directory\<guid>.<role>.DiagnosticStore\LogFiles\W3SVC1`
 
**Journaux d’événements Windows**

`D:\Windows\System32\Winevt\Logs`
