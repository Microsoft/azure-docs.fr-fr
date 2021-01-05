---
title: Capturer une trace de navigateur pour la résolution des problèmes
description: Capturer des informations réseau à partir d’une trace de navigateur pour résoudre les problèmes liés au portail Azure.
ms.date: 05/11/2020
ms.topic: troubleshooting
ms.openlocfilehash: 504a32fb63022f97e8049c7c1ccc7721bbac31ab
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745993"
---
# <a name="capture-a-browser-trace-for-troubleshooting"></a>Capturer une trace de navigateur pour la résolution des problèmes

Si vous comptez résoudre un problème lié au portail Azure et avez besoin de contacter le support Microsoft, nous vous recommandons de commencer par capturer une trace de navigateur, ainsi que quelques informations supplémentaires. Les données collectées peuvent fournir des informations importantes sur l’état du portail au moment où le problème s’est produit. Suivez les étapes de cet article concernant les outils de développement dans le navigateur que vous utilisez : Google Chrome ou Microsoft Edge (Chromium), Microsoft Edge (EdgeHTML), Apple Safari ou Firefox.

## <a name="google-chrome-and-microsoft-edge-chromium"></a>Google Chrome et Microsoft Edge (Chromium)

Google Chrome et Microsoft Edge (Chromium) sont tous les deux basés sur le [projet open source Chromium](https://www.chromium.org/Home). Les étapes suivantes montrent comment utiliser les outils de développement, qui sont très similaires dans ces deux navigateurs. Pour plus d’informations, consultez [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools) et [Outils de développement Microsoft Edge (Chromium)](/microsoft-edge/devtools-guide-chromium).

1. Connectez-vous au [portail Azure](https://portal.azure.com). Il est important de vous connecter _avant_ de démarrer la trace. De cette façon, la trace ne contiendra pas d’informations sensibles relatives à votre connexion. 

1. Commencez l’enregistrement des actions que vous effectuez dans le portail, à l’aide de l’[Enregistreur d’actions](https://support.microsoft.com/help/22878/windows-10-record-steps).

1. Dans le portail, accédez à l’action qui a précédé le moment où le problème s’est produit.

1. Appuyez sur F12 ou sélectionnez ![Capture d’écran affichant l’icône des paramètres du navigateur.](media/capture-browser-trace/chromium-icon-settings.png) > **Autres outils** > **Outils de développement**.

1. Par défaut, le navigateur conserve uniquement les informations de trace de la page actuellement chargée. Définissez les options suivantes pour que le navigateur conserve toutes les informations de trace, même si vos étapes de reproduction nécessitent l’accès à plusieurs pages :

    1. Sélectionnez l’onglet **Network** (Réseau), puis sélectionnez **Preserve log** (Conserver le journal).

          ![Capture d’écran mettant en évidence l’option Preserve log sous l’onglet Network.](media/capture-browser-trace/chromium-network-preserve-log.png)

    1. Sélectionnez l’onglet **Console**, sélectionnez **Console settings** (Paramètres de la console), puis sélectionnez **Preserve Log** (Conserver le journal). Sélectionnez de nouveau **Console settings** (Paramètres de la console) pour fermer le volet des paramètres.

          ![Capture d’écran faisant ressortir l’option Preserve log sous l’onglet Console.](media/capture-browser-trace/chromium-console-preserve-log.png)

1. Sélectionnez l’onglet **Network** (Réseau), puis sélectionnez **Stop recording network log** (Arrêter l’enregistrement du journal réseau) et **Clear** (Effacer).

    ![Capture d’écran des options « Stop recording network log » (Arrêter l’enregistrement du journal réseau) et « Clear » (Effacer)](media/capture-browser-trace/chromium-stop-clear-session.png)

1. Sélectionnez **Record network log** (Enregistrer le journal réseau), puis reproduisez le problème dans le portail.

    ![Capture d’écran montrant comment enregistrer le journal réseau.](media/capture-browser-trace/chromium-start-session.png)

    Vous devez voir une sortie de session similaire à celle-ci :

    ![Capture d’écran montrant la sortie de session.](media/capture-browser-trace/chromium-browser-trace-results.png)

1. Une fois que vous avez reproduit le comportement inattendu du portail, sélectionnez **Stop recording network log** (Arrêter l’enregistrement du journal réseau) et **Export as HAR** (Exporter en tant que HAR), puis enregistrez le fichier.

    ![Capture d’écran montrant comment exporter HAR sous l’onglet Réseau.](media/capture-browser-trace/chromium-network-export-har.png)

1. Arrêtez l’enregistreur d’actions, puis sauvegardez l’enregistrement.

1. De retour dans le volet Outils de développement du navigateur, sélectionnez l’onglet **Console**. Cliquez avec le bouton de droite, sélectionnez **Enregistrer sous...** , puis enregistrez la sortie de la console dans un fichier texte.

    ![Capture d’écran mettant en évidence l’onglet Console et le menu Save as...](media/capture-browser-trace/chromium-console-select.png)

1. Empaquetez le fichier HAR, la sortie de la console et l’enregistrement de l’écran dans un dossier compressé (format .zip, par exemple), puis envoyez-le au support Microsoft.

## <a name="microsoft-edge-edgehtml"></a>Microsoft Edge (EdgeHTML)

Les étapes suivantes montrent comment utiliser les outils de développement de Microsoft Edge (EdgeHTML). Pour plus d’informations, consultez [Outils de développement Microsoft Edge (EdgeHTML)](/microsoft-edge/devtools-guide).

1. Connectez-vous au [portail Azure](https://portal.azure.com). Il est important de vous connecter _avant_ de démarrer la trace. De cette façon, la trace ne contiendra pas d’informations sensibles relatives à votre connexion. 

1. Commencez l’enregistrement des actions que vous effectuez dans le portail, à l’aide de l’[Enregistreur d’actions](https://support.microsoft.com/help/22878/windows-10-record-steps).

1. Dans le portail, accédez à l’action qui a précédé le moment où le problème s’est produit.

1. Appuyez sur F12 ou sélectionnez ![Capture d’écran de l’icône des paramètres du navigateur.](media/capture-browser-trace/edge-icon-settings.png) > **Autres outils** > **Outils de développement**.

1. Par défaut, le navigateur conserve uniquement les informations de trace de la page actuellement chargée. Définissez les options suivantes pour que le navigateur conserve toutes les informations de trace, même si vos étapes de reproduction nécessitent l’accès à plusieurs pages :

    1. Sélectionnez l’onglet **Network** (Réseau), puis désactivez l’option **Clear entries on navigate** (Supprimer les entrées en naviguant).

          ![Capture d’écran « Clear entries on navigate » (Supprimer les entrées en naviguant)](media/capture-browser-trace/edge-network-clear-entries.png)

    1. Sélectionnez l’onglet **Console**, puis sélectionnez **Preserve log** (Conserver le journal).

          ![Capture d’écran de l’option « Preserve Log » (Conserver le journal)](media/capture-browser-trace/edge-console-preserve-log.png)

1. Sélectionnez l’onglet **Network** (Réseau), puis sélectionnez **Stop profiling session** (Arrêter la session de profilage) et **Clear session** (Effacer la session).

    ![Capture d’écran des options « Stop profiling session » (Arrêter la session de profilage) et « Clear session » (Effacer la session)](media/capture-browser-trace/edge-stop-clear-session.png)

1. Sélectionnez **Start profiling session** (Démarrer la session de profilage), puis reproduisez le problème dans le portail.

    ![Capture d’écran de l’option « Start profiling session » (Démarrer la session de profilage)](media/capture-browser-trace/edge-start-session.png)

    Vous devez voir une sortie de session similaire à celle-ci :

    ![Capture d’écran montrant la sortie de la session de profilage.](media/capture-browser-trace/edge-browser-trace-results.png)

1. Une fois que vous avez reproduit le comportement inattendu du portail, sélectionnez **Stop profiling session** (Arrêter la session de profilage) et **Export as HAR** (Exporter en tant que HAR), puis enregistrez le fichier.

    ![Capture d’écran de l’option « Export as HAR » (Exporter en tant que HAR)](media/capture-browser-trace/edge-network-export-har.png)

1. Arrêtez l’enregistreur d’actions, puis sauvegardez l’enregistrement.

1. De retour dans le volet Outils de développement du navigateur, sélectionnez l’onglet **Console**, puis développez la fenêtre. Placez le curseur au début de la sortie de la console, puis faites glisser la souris et sélectionnez tout le contenu de la sortie. Cliquez avec le bouton droit, sélectionnez **Copy** (Copier), puis enregistrez la sortie de la console dans un fichier texte.

    ![Capture d’écran mettant en évidence l’option de menu Copy.](media/capture-browser-trace/edge-console-select.png)

1. Empaquetez le fichier HAR, la sortie de la console et l’enregistrement de l’écran dans un dossier compressé (format .zip, par exemple), puis envoyez-le au support Microsoft.

## <a name="apple-safari"></a>Apple Safari

Les étapes suivantes montrent comment utiliser les outils de développement d’Apple Safari. Pour plus d’informations, consultez [Présentation des outils de développement Safari](https://support.apple.com/guide/safari-developer/safari-developer-tools-overview-dev073038698/11.0/mac).

1. Activez les outils de développement dans Apple Safari :

    1. Sélectionnez **Safari**, puis sélectionnez **Préférences**.

        ![Capture d’écran des préférences dans Safari](media/capture-browser-trace/safari-preferences.png)

    1. Sélectionnez l’onglet **Avancé**, puis sélectionnez **Afficher le menu Développement dans la barre des menus**.

        ![Capture d’écran des préférences avancées dans Safari](media/capture-browser-trace/safari-show-develop-menu.png)

1. Connectez-vous au [portail Azure](https://portal.azure.com). Il est important de vous connecter _avant_ de démarrer la trace. De cette façon, la trace ne contiendra pas d’informations sensibles relatives à votre connexion. 

1. Commencez l’enregistrement des actions que vous effectuez dans le portail. Pour plus d’informations, consultez [Procédure d’enregistrement de l’écran de votre Mac](https://support.apple.com/HT208721).

1. Dans le portail, accédez à l’action qui a précédé le moment où le problème s’est produit.

1. Sélectionnez **Développer**, puis sélectionnez **Afficher Web Inspector**.

    ![Capture d’écran de « Afficher Web Inspector »](media/capture-browser-trace/safari-show-web-inspector.png)

1. Par défaut, le navigateur conserve uniquement les informations de trace de la page actuellement chargée. Définissez les options suivantes pour que le navigateur conserve toutes les informations de trace, même si vos étapes de reproduction nécessitent l’accès à plusieurs pages :

    1. Sélectionnez l’onglet **Network** (Réseau), puis sélectionnez **Preserve log** (Conserver le journal).

          ![Capture d’écran montrant l’option Preserve Log.](media/capture-browser-trace/safari-network-preserve-log.png)

    1. Sélectionnez l’onglet **Console**, puis sélectionnez **Preserve log** (Conserver le journal).

          ![Capture d’écran montrant l’option Preserve Log sous l’onglet Console.](media/capture-browser-trace/safari-console-preserve-log.png)

1. Sélectionnez l’onglet **Network** (Réseau), puis sélectionnez **Clear Network Items** (Effacer les éléments réseau).

    ![Capture d’écran « Clear Network Items » (Effacer les éléments réseau)](media/capture-browser-trace/safari-clear-session.png)

1. Reproduisez le problème dans le portail. Vous devez voir une sortie de session similaire à celle-ci :

    ![Capture d’écran de la sortie après avoir reproduit le problème.](media/capture-browser-trace/safari-browser-trace-results.png)

1. Une fois que vous avez reproduit le comportement inattendu du portail, sélectionnez **Export** (Exporter), puis enregistrez le fichier.

    ![Capture d’écran de l’option « Export » (Exporter)](media/capture-browser-trace/safari-network-export-har.png)

1. Arrêtez l’enregistreur d’écran et sauvegardez l’enregistrement.

1. De retour dans le volet Outils de développement du navigateur, sélectionnez l’onglet **Console**, puis développez la fenêtre. Placez le curseur au début de la sortie de la console, puis faites glisser la souris et sélectionnez tout le contenu de la sortie. Appuyez sur les touches Commande-C pour copier la sortie et l’enregistrer dans un fichier texte.

    ![Capture d’écran mettant en évidence la façon d’afficher et de copier la sortie.](media/capture-browser-trace/safari-console-select.png)

1. Empaquetez le fichier HAR, la sortie de la console et l’enregistrement de l’écran dans un dossier compressé (format .zip, par exemple), puis envoyez-le au support Microsoft.

## <a name="firefox"></a>Firefox

Les étapes suivantes montrent comment utiliser les outils de développement dans Firefox. Pour plus d’informations, consultez [Outils de développement Firefox](https://developer.mozilla.org/docs/Tools).

1. Connectez-vous au [portail Azure](https://portal.azure.com). Il est important de vous connecter _avant_ de démarrer la trace. De cette façon, la trace ne contiendra pas d’informations sensibles relatives à votre connexion. 

1. Commencez l’enregistrement des actions que vous effectuez dans le portail. Utilisez [Enregistreur d’actions](https://support.microsoft.com/help/22878/windows-10-record-steps) sur Windows ou consultez [Comment enregistrer l’écran sur votre Mac](https://support.apple.com/HT208721).

1. Dans le portail, accédez à l’action qui a précédé le moment où le problème s’est produit.

1. Appuyez sur F12 ou sélectionnez ![l’icône pour la capture d’écran des paramètres du navigateur](media/capture-browser-trace/firefox-icon-settings.png) > **Développeur web** > **Changer les outils**.

1. Par défaut, le navigateur conserve uniquement les informations de trace de la page actuellement chargée. Définissez les options suivantes pour que le navigateur conserve toutes les informations de trace, même si vos étapes de reproduction nécessitent l’accès à plusieurs pages :

    1. Sélectionnez l’onglet **Réseau**, puis sélectionnez **Conserver les journaux**.

          ![Capture d’écran mettant en évidence l’option Conserver les journaux.](media/capture-browser-trace/firefox-network-persist-logs.png)

    1. Sélectionnez l’onglet **Console**, sélectionnez **Paramètres de la console**, puis sélectionnez **Conserver les journaux**.

          ![Capture d’écran de « Conserver les journaux »](media/capture-browser-trace/firefox-console-persist-logs.png)

1. Sélectionnez l’onglet **Réseau**, puis sélectionnez **Effacer**.

    ![Capture d'écran « Effacer »](media/capture-browser-trace/firefox-clear-session.png)

1. Reproduisez le problème dans le portail. Vous devez voir une sortie de session similaire à celle-ci :

    ![Capture d’écran des résultats de la trace du navigateur](media/capture-browser-trace/firefox-browser-trace-results.png)

1. Une fois que vous avez reproduit le comportement inattendu du portail, sélectionnez **Exporter/Importer HAR**, puis **Tout enregistrer sous HAR**.

    ![Capture d’écran de l’option « Export as HAR » (Exporter en tant que HAR)](media/capture-browser-trace/firefox-network-export-har.png)

1. Arrêtez l’enregistreur d’actions sur Windows ou l’enregistrement d’écran sur Mac, puis sauvegardez l’enregistrement.

1. De retour dans le volet Outils de développement du navigateur, sélectionnez l’onglet **Console**. Cliquez avec le bouton de droite, sélectionnez **Exporter le message visible sur**, puis enregistrez la sortie de la console dans un fichier texte.

    ![Capture d’écran de la sortie de la console](media/capture-browser-trace/firefox-console-select.png)

1. Empaquetez le fichier HAR, la sortie de la console et l’enregistrement de l’écran dans un dossier compressé (format .zip, par exemple), puis envoyez-le au support Microsoft.

## <a name="next-steps"></a>Étapes suivantes

[Présentation du portail Azure](azure-portal-overview.md)