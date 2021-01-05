---
title: La taille par défaut du dossier TEMP est trop petite pour un rôle | Microsoft Docs
description: Un rôle de service cloud offre un espace limité pour le dossier TEMP. Cet article fournit quelques suggestions pour éviter de manquer d'espace.
services: cloud-services
documentationcenter: ''
author: simonxjx
manager: dcscontentpm
editor: ''
tags: top-support-issue
ms.assetid: 9f2af8dd-2012-4b36-9dd5-19bf6a67e47d
ms.service: cloud-services
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 06/15/2018
ms.author: v-six
ms.openlocfilehash: d18e174eac30bbcf87010531821a4017efb088ba
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92075108"
---
# <a name="default-temp-folder-size-is-too-small-on-a-cloud-service-webworker-role"></a>La taille par défaut du dossier TEMP est trop petite pour un rôle web/de travail de service cloud
Le répertoire temporaire par défaut d'un rôle web ou de travail de service cloud a une taille maximale de 100 Mo, et risque d’être saturé à un moment donné. Cet article décrit comment éviter de manquer d'espace sur le répertoire temporaire.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="why-do-i-run-out-of-space"></a>Pourquoi l’espace devient-il insuffisant ?
Les variables d’environnement Windows standard TEMP et TMP sont accessibles au code exécuté dans votre application. TEMP et TMP pointent toutes les deux vers un répertoire unique d’une taille maximale de 100 Mo. Les données stockées dans ce répertoire ne sont pas conservées tout au long du cycle de vie du service cloud ; si les instances de rôle d’un service cloud sont recyclées, le répertoire est nettoyé.

## <a name="suggestion-to-fix-the-problem"></a>Suggestion pour corriger le problème
Implémentez l'une des alternatives suivantes :

* Configurez une ressource de stockage local et accédez-y directement au lieu d’utiliser TEMP ou TMP. Pour accéder à une ressource de stockage local à partir du code exécuté dans votre application, appelez la méthode [RoleEnvironment.GetLocalResource](/previous-versions/azure/reference/ee772845(v=azure.100)) .
* Configurez une ressource de stockage local et pointez les répertoires TEMP et TMP vers le chemin d'accès de la ressource de stockage local. Cette modification doit être effectuée dans la méthode [RoleEntryPoint.OnStart](/previous-versions/azure/reference/ee772851(v=azure.100)) .

L'exemple de code suivant montre comment modifier les répertoires cible TEMP et TMP dans la méthode OnStart :

```csharp
using System;
using Microsoft.WindowsAzure.ServiceRuntime;

namespace WorkerRole1
{
    public class WorkerRole : RoleEntryPoint
    {
        public override bool OnStart()
        {
            // The local resource declaration must have been added to the
            // service definition file for the role named WorkerRole1:
            //
            // <LocalResources>
            //    <LocalStorage name="CustomTempLocalStore"
            //                  cleanOnRoleRecycle="false"
            //                  sizeInMB="1024" />
            // </LocalResources>

            string customTempLocalResourcePath =
            RoleEnvironment.GetLocalResource("CustomTempLocalStore").RootPath;
            Environment.SetEnvironmentVariable("TMP", customTempLocalResourcePath);
            Environment.SetEnvironmentVariable("TEMP", customTempLocalResourcePath);

            // The rest of your startup code goes here…

            return base.OnStart();
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes
Lisez un blog expliquant [comment augmenter la taille du dossier temporaire ASP.NET du rôle web Azure](/archive/blogs/kwill/how-to-increase-the-size-of-the-windows-azure-web-role-asp-net-temporary-folder).

Affichez plus d’ [articles de résolution des problèmes](/visualstudio/azure/vs-azure-tools-debugging-cloud-services-overview) liés aux services cloud.

Pour découvrir comment résoudre les problèmes liés aux rôles de service cloud à l’aide des données de diagnostic informatiques PaaS Azure, consultez la [série de blogs de Kevin Williamson](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data).