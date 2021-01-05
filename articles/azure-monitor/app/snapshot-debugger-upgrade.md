---
title: Mise à niveau du débogueur de capture instantanée d’Azure Application Insights
description: Guide pratique pour mettre à niveau le débogueur de capture instantanée pour les applications .NET vers la dernière version sur Azure App Services ou via des packages NuGet
ms.topic: conceptual
author: pharring
ms.author: pharring
ms.date: 03/28/2019
ms.reviewer: mbullwin
ms.openlocfilehash: 6991db6055ca8556adff67f194bac0f4aa04cb06
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77671390"
---
# <a name="upgrading-the-snapshot-debugger"></a>Mise à niveau du Débogueur de capture instantanée

Pour assurer une sécurité optimale de vos données, Microsoft abandonne les protocoles TLS 1.0 et TLS 1.1, qui se sont révélés vulnérables aux attaquants les plus déterminés. Si vous utilisez une version antérieure de l’extension de site, elle devra être mise à niveau pour continuer à fonctionner. Ce document décrit les étapes nécessaires pour mettre à niveau votre Débogueur de capture instantanée vers la dernière version. Il existe deux chemins de mise à niveau principaux selon que vous avez activé le Débogueur de capture instantanée à l’aide d’une extension de site ou que vous avez utilisé un SDK/NuGet ajouté à votre application. Ces deux chemins de mise à niveau sont décrits ci-dessous. 

## <a name="upgrading-the-site-extension"></a>Mise à niveau de l’extension de site

> [!IMPORTANT]
> Les versions antérieures d’Application Insights utilisent une extension de site privée intitulée _extension Application Insights pour Azure App Service_. Pour l’activation de l’expérience Application Insights actuelle, les paramètres de l’application sont définis de façon à optimiser une extension de site pré-installée.
> Pour éviter les conflits susceptibles d’entraîner un arrêt du fonctionnement de votre site, il est important de commencer par supprimer l’extension de site privée. Consultez l’étape 4 ci-dessous.

Si vous avez activé le Débogueur de capture instantanée à l’aide de l’extension de site, vous pouvez effectuer la mise à niveau en effectuant les étapes suivantes :

1. Connectez-vous au portail Azure.
2. Accédez à votre ressource pour laquelle Application Insights et le Débogueur de capture instantanée sont activés. Par exemple, pour une application web, accédez à la ressource App Service :

   ![Capture d’écran d’une ressource App Service individuelle nommée DiagService01](./media/snapshot-debugger-upgrade/app-service-resource.png)

3. Une fois que vous avez accédé à votre ressource, cliquez sur le panneau Extensions et attendez que la liste des extensions se remplisse :

   ![Capture d’écran des extensions App Service montrant l’extension Application Insights pour Azure App Service installée](./media/snapshot-debugger-upgrade/application-insights-site-extension-to-be-deleted.png)

4. Si une version de l’_extension Application Insights pour Azure App Service_ est installée, sélectionnez-la et cliquez sur supprimer. Cliquez sur **Oui** pour confirmer la suppression de l’extension, puis attendez la fin de l’opération avant de passer à l’étape suivante.

   ![Capture d’écran des extensions App Service montrant l’extension Application Insights pour Azure App Service avec le bouton Supprimer encadré](./media/snapshot-debugger-upgrade/application-insights-site-extension-delete.png)

5. Accédez au panneau Vue d’ensemble de votre ressource, puis cliquez sur Application Insights :

   ![Capture d’écran de trois boutons. Bouton Centrer avec le nom Application Insights sélectionné](./media/snapshot-debugger-upgrade/application-insights-button.png)

6. S’il s’agit de la première fois que vous affichez le panneau Application Insights pour cet App Service, vous serez invité à activer Application Insights. Sélectionnez **Activer Application Insights**.
 
   ![Capture d’écran de la première ouverture du panneau Application Insights avec le bouton Activer Application Insights encadré](./media/snapshot-debugger-upgrade/turn-on-application-insights.png)

7. Les paramètres actuels d’Application Insights sont affichés. Vous pouvez les conserver en l’état, sauf si vous voulez en profiter pour modifier vos paramètres. Le bouton **Appliquer** en bas du panneau n’est pas activé par défaut et vous devez activer/désactiver l’un des paramètres pour l’activer. Vous n’avez pas besoin de changer les paramètres réels. Au lieu de cela, vous pouvez changer le paramètre, puis le rétablir immédiatement. Nous vous recommandons d’activer/de désactiver le paramètre de Profiler, puis de sélectionner **Appliquer**.

   ![Capture d’écran de la page Configuration d’Application Insights App Service avec le bouton Appliquer surligné en rouge](./media/snapshot-debugger-upgrade/view-application-insights-data.png)

8. Après avoir cliqué sur **Appliquer**, vous serez invité à confirmer les changements apportés.

    > [!NOTE]
    > Le site sera redémarré dans le cadre du processus de mise à niveau.

   ![Capture d’écran de l’invite d’application de la supervision d’App Service. La zone de texte affiche le message : « Nous allons maintenant appliquer les changements de vos paramètres d’application et installer nos outils pour lier votre ressource Application Insights à l’application web. Cette opération redémarre le site. Voulez-vous continuer ? ».](./media/snapshot-debugger-upgrade/apply-monitoring-settings.png)

9. Cliquez sur **Oui** pour appliquer les modifications, puis attendez la fin du processus.

Le site a maintenant été mis à niveau et est prêt à être utilisé.

## <a name="upgrading-snapshot-debugger-using-sdknuget"></a>Mise à niveau du Débogueur de capture instantanée à l’aide d’un SDK/NuGet

Si l’application utilise une version de `Microsoft.ApplicationInsights.SnapshotCollector` antérieure à la version 1.3.1, elle devra être mise à niveau vers une [version plus récente](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) pour continuer à fonctionner.
