---
title: Résolution des problèmes dans Azure Monitor Application Insights pour Java
description: Découvrez comment résoudre les problèmes liés à l’agent Java pour Azure Monitor Application Insights
ms.topic: conceptual
ms.date: 11/30/2020
ms.custom: devx-track-java
ms.openlocfilehash: 14644f76b7de53b2b6ee3f04131daaf59267a5ff
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507640"
---
# <a name="troubleshooting-guide-azure-monitor-application-insights-for-java"></a>Guide de résolution des problèmes : Azure Monitor Application Insights pour Java

Dans cet article, nous aborderons certains des problèmes courants que vous pouvez rencontrer lors de l’instrumentation d’une application Java à l’aide de l’agent Java pour Application Insights. Nous abordons également les étapes à suivre pour résoudre ces problèmes. Application Insights est une fonctionnalité du service de plateforme Azure Monitor.

## <a name="check-the-self-diagnostic-log-file"></a>Vérifier le fichier journal de diagnostic automatique

Par défaut, l’agent Java 3.0 pour Application Insights produit un fichier journal nommé `applicationinsights.log` dans le même répertoire que le fichier `applicationinsights-agent-3.0.0.jar`.

Ce fichier journal est le premier endroit où chercher des indications sur le problème rencontré.

## <a name="upgrade-from-the-application-insights-java-2x-sdk"></a>Mettre à niveau à partir du Kit de développement logiciel (SDK) Application Insights pour Java 2.x

Si vous utilisez déjà le Kit de développement logiciel (SDK) Application Insights pour Java 2.x dans votre application, vous pouvez continuer à l’utiliser. L’agent Java 3.0 le détecte. Pour plus d’informations, consultez [Mettre à niveau à partir du Kit de développement logiciel (SDK) Java 2.x](./java-standalone-upgrade-from-2x.md).

## <a name="upgrade-from-application-insights-java-30-preview"></a>Mise à niveau à partir d’Application Insights pour Java 3.0 Preview

Si vous effectuez une mise à niveau à partir de l’agent Java 3.0 Preview, examinez attentivement toutes les [options de configuration](./java-standalone-config.md). La structure JSON a été entièrement modifiée dans la version 3.0 en disponibilité générale (GA).

Ces modifications sont les suivantes :

-  Le nom du fichier config (anciennement `ApplicationInsights.json`) a été remplacé par `applicationinsights.json`.
-  Le nœud `instrumentationSettings` n’est plus présent. Tout le contenu de `instrumentationSettings` a été déplacé vers le niveau racine. 
-  Les nœuds de configuration comme `sampling`, `jmxMetrics`, `instrumentation` et `heartbeat` ont été déplacés de `preview` vers le niveau racine.

## <a name="some-logging-is-not-auto-collected"></a>Une partie de la journalisation n’est pas collectée automatiquement

La journalisation n’est capturée que si elle respecte, premièrement, le seuil configuré des infrastructures de journalisation, et deuxièmement, le seuil configuré d’Application Insights.

La meilleure façon de savoir si une instruction de journalisation particulière respecte le seuil configuré des frameworks de journalisation est de confirmer qu’elle apparaît dans le journal des applications (par exemple, fichier ou console).

Pour plus d’informations, consultez la [configuration de la journalisation collectée automatiquement](./java-standalone-config.md#auto-collected-logging).

## <a name="import-ssl-certificates"></a>Importer des certificats SSL

Si vous utilisez le magasin de clés Java par défaut, tous les certificats racine de l’autorité de certification sont déjà présents. Vous n’avez pas besoin d’importer d’autres certificats SSL.

Si vous utilisez un magasin de clés Java personnalisé, vous devrez peut-être y importer les certificats SSL de point de terminaison Application Insights.

### <a name="key-terminology"></a>Terminologie clé
Un *magasin de clés* est un référentiel qui contient des certificats, des clés publiques et des clés privées. En règle générale, les distributions Java Development Kit comprennent un exécutable permettant de les gérer : `keytool`.

L’exemple suivant est une commande simple qui permet d’importer un certificat SSL dans le magasin de clés :

`keytool -importcert -alias your_ssl_certificate -file "your downloaded SSL certificate name".cer -keystore "Your KeyStore name" -storepass "Your keystore password" -noprompt`

### <a name="steps-to-download-and-add-an-ssl-certificate"></a>Étapes pour télécharger et ajouter un certificat SSL

1.  Ouvrez votre navigateur préféré, puis accédez à l’URL `IngestionEndpoint` qui est incluse dans la chaîne de connexion utilisée pour instrumenter votre application.

    :::image type="content" source="media/java-ipa/troubleshooting/ingestion-endpoint-url.png" alt-text="Capture d’écran montrant une chaîne de connexion Application Insights.":::

2.  Sélectionnez l’icône **Afficher les informations du site** (verrou) dans le navigateur, puis sélectionnez l’option **Certificat**.

    :::image type="content" source="media/java-ipa/troubleshooting/certificate-icon-capture.png" alt-text="Capture d’écran de l’option Certificat dans les informations du site.":::

3.  Accédez à l’onglet **Détails** et sélectionnez **Copier dans le fichier**.
4.  Sélectionnez le bouton **Suivant**, sélectionnez le format **X.509 encodé en base 64 (.CER)** , puis sélectionnez à nouveau **Suivant**.

    :::image type="content" source="media/java-ipa/troubleshooting/certificate-export-wizard.png" alt-text="Capture d’écran de l’Assistant Exportation de certificat, avec un format sélectionné.":::

5.  Spécifiez le fichier dans lequel vous souhaitez enregistrer le certificat SSL. Ensuite, sélectionnez **Suivant** > **Terminer**. Le message « L’exportation a réussi »doit s’afficher.
6.  Une fois que vous disposez du certificat, il est temps de l’importer dans un magasin de clés Java. Utilisez la [commande précédente](#key-terminology) pour importer des certificats.

> [!WARNING]
> Vous devrez répéter ces étapes pour récupérer le nouveau certificat avant que le certificat actuel n’expire. Vous trouverez des informations sur l’expiration sous l’onglet **Détails** de la boîte de dialogue **Certificat**.
>
> :::image type="content" source="media/java-ipa/troubleshooting/certificate-details.png" alt-text="Capture d’écran montrant les détails du certificat SSL.":::
