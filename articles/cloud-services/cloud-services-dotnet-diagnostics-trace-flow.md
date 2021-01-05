---
title: Assurer le suivi du flux dans une application Cloud Services avec Diagnostics Azure
titleSuffix: Azure Cloud Services
description: Ajouter des messages de suivi à une application Azure pour aider au débogage, à la mesure des performances, à la surveillance, à l’analyse du trafic et bien plus encore.
services: cloud-services
documentationcenter: .net
author: tgore03
ms.service: cloud-services
ms.devlang: dotnet
ms.custom: devx-track-dotnet
ms.topic: article
ms.date: 02/20/2016
ms.author: tagore
ms.openlocfilehash: 8270766413729454181c461d469d49e418a1aa67
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88932302"
---
# <a name="trace-the-flow-of-a-cloud-services-application-with-azure-diagnostics"></a>Assurer le suivi du flux dans une application Cloud Services avec Diagnostics Azure
Le suivi est un moyen de surveiller l’exécution de votre application pendant son exécution . Vous pouvez utiliser les classes [System.Diagnostics.Trace](/dotnet/api/system.diagnostics.trace), [System.Diagnostics.Debug](/dotnet/api/system.diagnostics.debug) et [System.Diagnostics.TraceSource](/dotnet/api/system.diagnostics.tracesource) pour enregistrer des informations relatives aux erreurs et à l’exécution des applications dans des journaux d’activité, des fichiers texte ou d’autres périphériques pour une analyse ultérieure. Pour plus d’informations sur le suivi, consultez [Applications de suivi et instrumentation](/dotnet/framework/debug-trace-profile/tracing-and-instrumenting-applications).

## <a name="use-trace-statements-and-trace-switches"></a>Utilisez les instructions et commutateurs de suivi
Mettez en œuvre le suivi dans votre application Cloud Services en ajoutant [DiagnosticMonitorTraceListener](/previous-versions/azure/reference/ee758610(v=azure.100)) à la configuration d’application et en lançant des appels vers System.Diagnostics.Trace ou vers System.Diagnostics.Debug dans votre code d’application. Utilisez le fichier de configuration *app.config* pour les rôles de travail et le fichier *web.config* pour les rôles web. Lorsque vous créez un nouveau service hébergé à l’aide d’un modèle Visual Studio, Diagnostics Azure est automatiquement ajouté au projet et DiagnosticMonitorTraceListener est ajouté au fichier de configuration approprié pour les rôles que vous ajoutez.

Pour plus d’informations sur le placement des instructions de suivi, consultez [Comment : Ajoutez des instructions de traçage au code d’application](/dotnet/framework/debug-trace-profile/how-to-add-trace-statements-to-application-code).

En plaçant des [Commutateurs de suivi](/dotnet/framework/debug-trace-profile/trace-switches) dans votre code, vous pouvez contrôler le traçage et son importance. Vous pouvez ainsi surveiller l’état de votre application dans un environnement de production, ce qui est particulièrement important dans une application qui utilise plusieurs composants s’exécutant sur plusieurs ordinateurs. Pour plus d’informations, consultez [Procédure : Configurer des commutateurs de Trace](/dotnet/framework/debug-trace-profile/how-to-create-initialize-and-configure-trace-switches).

## <a name="configure-the-trace-listener-in-an-azure-application"></a>Configurer l’écouteur de suivi dans une application Azure
Trace, Debug et TraceSource nécessitent que vous définissiez des « écouteurs » pour recueillir et enregistrer les messages qui sont envoyés. Les écouteurs recueillent, stockent et acheminent les messages de suivi. Ils orientent la sortie de suivi vers une cible appropriée, par exemple un journal, une fenêtre ou un fichier texte. Diagnostics Azure utilise la classe [DiagnosticMonitorTraceListener](/previous-versions/azure/reference/ee758610(v=azure.100)) .

Avant d’exécuter la procédure suivante, vous devez initialiser le moniteur de diagnostic Azure. Pour ce faire, voir [Activation des diagnostics dans Microsoft Azure](cloud-services-dotnet-diagnostics.md).

Notez que si vous utilisez les modèles fournis par Visual Studio, la configuration de l’écouteur est automatiquement ajoutée pour vous.

### <a name="add-a-trace-listener"></a>Ajouter un écouteur de suivi
1. Ouvrez le fichier web.config ou app.config correspondant à votre rôle.
2. Ajoutez le code suivant au fichier. Modifiez l’attribut de version pour utiliser le numéro de version de l’assembly que vous référencez. La version d’assembly ne change pas nécessairement avec chaque version du Kit de développement logiciel (SDK) Azure sauf s’il existe des mises à jour.
   
    ```
    <system.diagnostics>
        <trace>
            <listeners>
                <add type="Microsoft.WindowsAzure.Diagnostics.DiagnosticMonitorTraceListener,
                  Microsoft.WindowsAzure.Diagnostics,
                  Version=2.8.0.0,
                  Culture=neutral,
                  PublicKeyToken=31bf3856ad364e35"
                  name="AzureDiagnostics">
                    <filter type="" />
                </add>
            </listeners>
        </trace>
    </system.diagnostics>
    ```
   > [!IMPORTANT]
   > Assurez-vous de disposer d’une référence de projet à l’assembly Microsoft.WindowsAzure.Diagnostics. Mettre à jour le numéro de version dans le document xml ci-dessus pour correspondre à la version de l’assembly Microsoft.WindowsAzure.Diagnostics référencé.
   > 
   > 
3. Enregistrez le fichier de configuration.

Pour plus d’informations sur les écouteurs, consultez [Suivi des écouteurs](/dotnet/framework/debug-trace-profile/trace-listeners).

Une fois les opérations destinées à ajouter l’écouteur terminées, vous pouvez ajouter des instructions de suivi à votre code.

### <a name="to-add-trace-statement-to-your-code"></a>Pour ajouter des instructions de suivi à votre code
1. Ouvrez un fichier source pour votre application. Par exemple, le fichier \<RoleName>.cs pour le rôle de travail ou le rôle web.
2. Ajoutez la directive d’utilisation suivante si elle n’a pas encore été ajoutée :
    ```
        using System.Diagnostics;
    ```
3. Ajoutez les instructions de suivi à l’endroit où vous souhaitez capturer des informations sur l’état de votre application. Vous pouvez utiliser différentes méthodes pour mettre en forme la sortie de l’instruction de suivi. Pour plus d’informations, consultez [Procédure : Ajoutez des instructions de traçage au code d’application](/dotnet/framework/debug-trace-profile/how-to-add-trace-statements-to-application-code).
4. Enregistrez le fichier source.




