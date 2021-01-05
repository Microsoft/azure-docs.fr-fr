---
title: Exécuter des tâches de démarrage dans Azure Cloud Services | Microsoft Docs
description: Les tâches de démarrage facilitent la préparation de votre environnement de service cloud pour votre application. Cette documentation vous apprend comment fonctionnent les tâches de démarrage et comment les créer.
services: cloud-services
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 07/05/2017
ms.author: tagore
ms.openlocfilehash: f2417389de98f9998c189e7cbbbcdae77fbb8840
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020702"
---
# <a name="how-to-configure-and-run-startup-tasks-for-a-cloud-service"></a>Comment configurer et exécuter des tâches de démarrage pour un service cloud
Vous pouvez utiliser des tâches de démarrage pour exécuter des opérations avant le démarrage d’un rôle. Parmi les opérations que vous pouvez effectuer figurent l’installation d’un composant, l’enregistrement de composants COM, la définition des clés du Registre ou le démarrage d’un processus de longue durée.

> [!NOTE]
> Les tâches de démarrage ne s’appliquent pas aux rôles de machine virtuelle ; elles ne concernent que les rôles web de service cloud et de travail.
> 
> 

## <a name="how-startup-tasks-work"></a>Fonctionnement des tâches de démarrage
Les tâches de démarrage sont des actions qui sont effectuées avant le début de vos rôles, et sont définies dans le fichier [ServiceDefinition.csdef] à l’aide de l’élément [Tâche] dans l’élément [Startup]. Souvent les tâches de démarrage sont des fichiers de commandes, mais elles peuvent également être des applications console ou des fichiers de commandes qui démarrent des scripts PowerShell.

Les variables d’environnement passent des informations dans une tâche de démarrage et le stockage local peut être utilisé pour transmettre des informations hors d’une tâche de démarrage. Par exemple, une variable d’environnement peut spécifier le chemin d’accès à un programme que vous voulez installer, et des fichiers peuvent être écrits dans le stockage local qui peuvent être lus ultérieurement par vos rôles.

Votre tâche de démarrage peut enregistrer des informations et des erreurs dans un répertoire spécifié par la variable d’environnement **TEMP** . Pendant la tâche de démarrage, la variable d’environnement **TEMP** est résolue dans le répertoire *C:\\Resources\\temp\\[guid].[nom_rôle]\\RoleTemp* au moment de l’exécution sur le cloud.

Les tâches de démarrage peuvent également être exécutées plusieurs fois entre des redémarrages. Par exemple, la tâche de démarrage est exécutée chaque fois que le rôle est recyclé, et les recyclages de rôle n’incluent pas toujours un redémarrage. Les tâches de démarrage doivent être écrites de façon à pouvoir s’exécuter de façon répétée sans problèmes.

Les tâches de démarrage doivent s’arrêter avec un **errorlevel** (ou code de sortie) égal à zéro pour que le processus de démarrage soit terminé. Si une tâche de démarrage se termine par un **errorlevel** différent de zéro, le rôle ne démarre pas.

## <a name="role-startup-order"></a>Ordre de démarrage des rôles
Les informations suivantes indiquent la procédure de démarrage de rôle dans Azure :

1. L’instance est marquée comme **Starting** et ne reçoit pas de trafic.
2. Toutes les tâches de démarrage sont exécutées en fonction de leur attribut **taskType** .
   
   * Les tâches **simple** sont exécutées de façon synchrone, une par une.
   * Les tâches **background** et **foreground** sont démarrées de façon asynchrone, en parallèle de la tâche de démarrage.  
     
     > [!WARNING]
     > Il est possible qu’IIS ne soit pas configuré complètement pendant l’étape de la tâche de démarrage ; de ce fait, les données spécifiques au rôle ne sont pas forcément disponibles. Les tâches de démarrage qui ont besoin de données spécifiques au rôle doivent utiliser [Microsoft.WindowsAzure.ServiceRuntime.RoleEntryPoint.OnStart](/previous-versions/azure/reference/ee772851(v=azure.100)).
     > 
     > 
3. Le processus hôte de rôle est démarré, et le site est créé dans IIS.
4. La méthode [Microsoft.WindowsAzure.ServiceRuntime.RoleEntryPoint.OnStart](/previous-versions/azure/reference/ee772851(v=azure.100)) est appelée.
5. L’instance est marquée comme **Ready** , et le trafic est acheminé vers l’instance.
6. La méthode [Microsoft.WindowsAzure.ServiceRuntime.RoleEntryPoint.Run](/previous-versions/azure/reference/ee772746(v=azure.100)) est appelée.

## <a name="example-of-a-startup-task"></a>Exemple d’une tâche de démarrage
Les tâches de démarrage sont définies dans le fichier [ServiceDefinition.csdef] , dans l’élément **Task** . L’attribut **commandLine** spécifie le nom et les paramètres du fichier de commandes de démarrage ou de la commande de la console, l’attribut **executionContext** indique le niveau de privilège de la tâche de démarrage et l’attribut **taskType** définit l’exécution de la tâche.

Dans cet exemple, une variable d’environnement **MyVersionNumber**, est créée pour la tâche de démarrage et définie sur « **1.0.0.0** ».

**ServiceDefinition.csdef**:

```xml
<Startup>
    <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple" >
        <Environment>
            <Variable name="MyVersionNumber" value="1.0.0.0" />
        </Environment>
    </Task>
</Startup>
```

Dans l’exemple suivant, le fichier de commande **Startup.cmd** écrit la ligne « The current version is 1.0.0.0 » dans le fichier StartupLog.txt dans le répertoire spécifié par la variable d’environnement TEMP. La ligne `EXIT /B 0` garantit que la tâche de démarrage s’arrête avec un **errorlevel** égal à zéro.

```cmd
ECHO The current version is %MyVersionNumber% >> "%TEMP%\StartupLog.txt" 2>&1
EXIT /B 0
```

> [!NOTE]
> Dans Visual Studio, la propriété **Copier dans le répertoire de sortie** pour votre fichier de commandes de démarrage doit être définie sur **Toujours copier** afin de vous assurer que votre fichier de commandes de démarrage est correctement déployé dans votre projet sur Azure (**approot\\bin** pour les rôles web et **approot** pour les rôles de travail).
> 
> 

## <a name="description-of-task-attributes"></a>Description des attributs de tâche
Vous trouverez ci-dessous une description des attributs de l’élément **Task** du fichier [ServiceDefinition.csdef] :

**commandLine** - spécifie la ligne de commande pour la tâche de démarrage :

* La commande avec des paramètres de ligne de commande en option, qui débute la tâche de démarrage.
* Souvent, il s’agit du nom d’un fichier de commandes .cmd ou .bat.
* La tâche est relative au dossier AppRoot\\Bin pour le déploiement. Les variables d’environnement ne sont pas développées pour déterminer le chemin d’accès et le fichier de la tâche. Si ce développement est nécessaire, vous pouvez créer un petit script .cmd qui appelle votre tâche de démarrage.
* Il peut s’agir d’une application console ou d’un fichier de commande qui démarre un [script PowerShell](cloud-services-startup-tasks-common.md#create-a-powershell-startup-task).

**executionContext** - spécifie le niveau de privilège pour la tâche de démarrage. Le niveau de privilège peut être limité ou élevé :

* **limited**  
  : la tâche de démarrage s’exécute avec les mêmes privilèges que le rôle. Quand l’attribut **executionContext** de l’élément [Runtime] est également **limited**, les privilèges utilisateur sont utilisés.
* **elevated**  
  : la tâche de démarrage s’exécute avec des privilèges d’administrateur. Les tâches de démarrage peuvent ainsi installer des programmes, apporter des modifications à la configuration IIS, modifier le Registre et effectuer d’autres tâches d’administration, sans augmenter le niveau de privilège du rôle.  

> [!NOTE]
> Le niveau de privilège d’une tâche de démarrage n’a pas besoin d’être le même que celui du rôle.
> 
> 

**taskType** - spécifie la façon dont une tâche de démarrage est exécutée.

* **simple**  
  sont exécutées de façon synchrone, une à la fois, dans l’ordre spécifié dans le fichier [ServiceDefinition.csdef] . Quand une tâche **simple** se termine par un **errorlevel** égal à zéro, la tâche de démarrage **simple** suivante est exécutée. Quand toutes les tâches **simple** ont été exécutées, le rôle est démarré.   
  
  > [!NOTE]
  > Si la tâche **simple** se termine par un **errorlevel** différent de zéro, l’instance est bloquée. Les tâches de démarrage **simple** suivantes et le rôle ne démarrent pas.
  > 
  > 
  
    Pour vous assurer que votre fichier de commandes se termine par un **errorlevel** égal à zéro, exécutez la commande `EXIT /B 0` à la fin du processus de votre fichier de commandes.
* **background**  
  sont exécutées de façon asynchrone, en parallèle du démarrage du rôle.
* **foreground**  
  sont exécutées de façon asynchrone, en parallèle du démarrage du rôle. La principale différence entre une tâche **foreground** et une tâche **background** est que la tâche **foreground** empêche le recyclage ou l’arrêt du rôle tant qu’elle n’est pas terminée. Les tâches **background** n’ont pas cette restriction.

## <a name="environment-variables"></a>Variables d'environnement
Les variables d’environnement permettent de passer les informations à une tâche de démarrage. Par exemple, vous pouvez indiquer le chemin vers un objet blob qui contient un programme à installer ou les numéros de port que votre rôle va utiliser ou des paramètres pour contrôler les fonctionnalités de votre tâche de démarrage.

Il existe deux types de variables d’environnement pour des tâches de démarrage ; des variables d’environnement statiques et des variables d’environnement basées sur les membres de la classe [RoleEnvironment] . Elles se trouvent dans la section [Environment] du fichier [ServiceDefinition.csdef] et utilisent l’élément [Variable] et l’attribut **name**.

Les variables d’environnement statiques utilisent l’attribut **value** de l’élement [Variable] . L’exemple ci-dessus crée la variable d’environnement **MyVersionNumber** avec une valeur statique de « **1.0.0.0** ». Un autre exemple consiste à créer une variable d’environnement **StagingOrProduction** à laquelle vous pouvez manuellement attribuer les valeurs « **staging** » ou « **production** » pour exécuter des actions de démarrage différentes en fonction de la valeur de la variable d’environnement **StagingOrProduction**.

Les variables d’environnement basées sur les membres de la classe RoleEnvironment n’utilisent pas l’attribut **value** de l’élément [Variable] . À la place, l’élément [RoleInstanceValue] enfant, avec la valeur d’attribut **XPath** appropriée, est utilisé pour créer une variable d’environnement basée sur un membre spécifique de la classe [RoleEnvironment]. Les valeurs de l’attribut **XPath** pour accéder aux différentes valeurs [RoleEnvironment] se trouvent [ici](cloud-services-role-config-xpath.md).

Par exemple, pour créer une variable d’environnement qui a la valeur « **true** » quand l’instance s’exécute dans l’émulateur de calcul et la valeur « **false** » pendant une exécution dans le cloud, utilisez les éléments [Variable] et [RoleInstanceValue] :

```xml
<Startup>
    <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
        <Environment>

            <!-- Create the environment variable that informs the startup task whether it is running
                in the Compute Emulator or in the cloud. "%ComputeEmulatorRunning%"=="true" when
                running in the Compute Emulator, "%ComputeEmulatorRunning%"=="false" when running
                in the cloud. -->

            <Variable name="ComputeEmulatorRunning">
                <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
            </Variable>

        </Environment>
    </Task>
</Startup>
```

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment effectuer certaines [tâches de démarrage courantes](cloud-services-startup-tasks-common.md) avec votre service cloud.

[Créez un package](cloud-services-model-and-package.md) de votre service cloud.  

[ServiceDefinition.csdef]: cloud-services-model-and-package.md#csdef
[Tâche]: /previous-versions/azure/reference/gg557552(v=azure.100)#Task
[Startup]: /previous-versions/azure/reference/gg557552(v=azure.100)#Startup
[Runtime]: /previous-versions/azure/reference/gg557552(v=azure.100)#Runtime
[Environment]: /previous-versions/azure/reference/gg557552(v=azure.100)#Environment
[Variable]: /previous-versions/azure/reference/gg557552(v=azure.100)#Variable
[RoleInstanceValue]: /previous-versions/azure/reference/gg557552(v=azure.100)#RoleInstanceValue
[RoleEnvironment]: /previous-versions/azure/reference/ee773173(v=azure.100)