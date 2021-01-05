---
title: Activer la journalisation des diagnostics
description: Découvrez comment activer la journalisation de diagnostic et ajouter la fonctionnalité d’instrumentation à votre application, mais aussi comment accéder aux informations enregistrées par Azure.
ms.assetid: c9da27b2-47d4-4c33-a3cb-1819955ee43b
ms.topic: article
ms.date: 09/17/2019
ms.custom: devx-track-csharp, seodec18
ms.openlocfilehash: 875254071d0ea252508242b83102fb8ca8b44e53
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825381"
---
# <a name="enable-diagnostics-logging-for-apps-in-azure-app-service"></a>Activer la journalisation des diagnostics pour les applications dans Azure App Service
## <a name="overview"></a>Vue d’ensemble
Azure fournit des diagnostics intégrés pour aider au débogage d’une [application App Service](overview.md). Cet article vous explique comment activer la journalisation de diagnostic et ajouter la fonctionnalité d’instrumentation à votre application, et comment accéder aux informations enregistrées par Azure.

Cet article utilise le [portail Azure](https://portal.azure.com) et Azure CLI pour l’exploitation des journaux de diagnostic. Pour plus d’informations sur l’utilisation de journaux de diagnostic avec Visual Studio, consultez [Résolution des problèmes Azure dans Visual Studio](troubleshoot-dotnet-visual-studio.md).

> [!NOTE]
> En plus des instructions de journalisation figurant dans cet article, il existe une nouvelle fonctionnalité de journalisation intégrée avec Azure Monitor. Pour plus d’informations sur cette fonctionnalité, consultez la section [Envoyer des journaux à Azure Monitor (préversion)](#send-logs-to-azure-monitor-preview). 
>
>

|Type|Plateforme|Emplacement|Description|
|-|-|-|-|
| Journalisation des applications | Windows, Linux | Système de fichiers App Service et/ou objets blob de stockage Azure | Consigne les messages générés par votre code d’application. Les messages peuvent être générés par l’infrastructure web de votre choix ou directement à partir de votre code d’application à l’aide du modèle de journalisation standard de votre langage. Chaque message se voit attribuer l’une des catégories suivantes : **Critique**, **Erreur**, **Avertissement**, **Info**, **Débogage** ou **Trace**. Vous pouvez sélectionner le degré de détail de la journalisation en définissant le niveau de gravité lorsque vous activez la journalisation des applications.|
| Journalisation du serveur web| Windows | Système de fichiers App Service ou objets blob de stockage Azure| Données de requête HTTP brutes au [format de fichier journal étendu W3C](/windows/desktop/Http/w3c-logging). Chaque message de journalisation comprend des données telles que la méthode HTTP, l’URI de ressource, l’adresse IP du client, le port client, l’agent utilisateur, le code de réponse, etc. |
| Messages d’erreur détaillés| Windows | Système de fichiers App Service | Copies des pages d’erreur *.htm* qui auraient été envoyées au navigateur client. Pour des raisons de sécurité, les pages d’erreur détaillées ne doivent pas être envoyées aux clients en production, mais App Service peut enregistrer la page d’erreur chaque fois qu’une erreur d’application se produit avec le code HTTP 400 ou supérieur. La page peut contenir des informations permettant de déterminer la raison pour laquelle le serveur renvoie ce code d’erreur. |
| Suivi des demandes ayant échoué | Windows | Système de fichiers App Service | Informations de suivi détaillées des demandes qui ont échoué, y compris une trace des composants IIS utilisés pour traiter la demande et la durée dans chaque composant. Ces informations sont utiles si vous souhaitez améliorer les performances du site ou isoler une erreur HTTP spécifique. Un dossier est généré pour chaque demande ayant échoué, qui contient le fichier journal XML et la feuille de style XSL permettant d’afficher le fichier journal. |
| Journalisation du déploiement | Windows, Linux | Système de fichiers App Service | Journaux relatifs à votre publication de contenu dans une application. La journalisation du déploiement se déroule automatiquement et il n’existe aucun paramètre de configuration de la journalisation du déploiement. Elle vous aide à déterminer la raison de l’échec d’un déploiement. Par exemple, si vous utilisez un [script de déploiement personnalisé](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script), vous pouvez recourir à la journalisation de déploiement pour déterminer la cause de l’échec du script. |

> [!NOTE]
> App Service fournit un outil de diagnostic interactif dédié pour vous aider à résoudre les problèmes de votre application. Pour plus d’informations, consultez [Présentation des diagnostics Azure App Service](overview-diagnostics.md).
>
> En outre, vous pouvez utiliser d’autres services Azure pour améliorer les fonctionnalités de journalisation et de surveillance de votre application, tels qu’[Azure Monitor](../azure-monitor/app/azure-web-apps.md).
>

## <a name="enable-application-logging-windows"></a>Activer la journalisation des applications (Windows)

> [!NOTE]
> La journalisation des applications pour le stockage de blobs peut uniquement utiliser des comptes de stockage dans la même région que l’App Service.

Pour activer la journalisation des applications Windows dans le [portail Azure](https://portal.azure.com), accédez à votre application et sélectionnez **Journaux App Service**.

Sélectionnez **Activé** pour **Journal des applications (Système de fichiers)** , **Journal des applications (Blob)** ou les deux. 

L’option **Système de fichiers** est utilisée à des fins de débogage temporaire et se désactive elle-même après 12 heures. L’option **Blob** est destinée à la journalisation à long terme et a besoin d’un conteneur de stockage d’objets blob dans lequel écrire les journaux.  L’option **Blob** inclut également des informations supplémentaires dans les messages de journalisation, telles que l’ID de l’instance de machine virtuelle d’origine du message de journalisation (`InstanceId`), l’ID de thread (`Tid`) et un horodateur plus précis ([`EventTickCount`](/dotnet/api/system.datetime.ticks)).

> [!NOTE]
> Actuellement, seuls les journaux des applications .NET peuvent être écrits dans le Stockage Blob. Les journaux des applications Java, PHP, Node.js et Python peuvent être stockés uniquement dans le système de fichiers App Service (sans modifications du code pour écrire les journaux dans un stockage externe).
>
> De plus, si vous [régénérez les clés d’accès de votre compte de stockage](../storage/common/storage-account-create.md), vous devez réinitialiser la configuration de journalisation correspondante pour utiliser les clés d’accès mises à jour. Pour ce faire :
>
> 1. Sous l’onglet **Configurer**, définissez la fonctionnalité de journalisation correspondante sur **Désactivé**. Enregistrez votre paramètre.
> 2. Réactivez la journalisation de l’objet blob du compte de stockage. Enregistrez votre paramètre.
>
>

Sélectionnez le **niveau** ou le niveau de détails à consigner. Le tableau suivant présente les catégories de journaux incluses à chaque niveau :

| Level | Catégories incluses |
|-|-|
|**Désactivé** | None |
|**Error** | Erreur, Critique |
|**Avertissement** | Avertissement, Erreur, Critique|
|**Informations** | Info, Avertissement, Erreur, Critique|
|**Verbose** | Trace, Débogage, Info, Avertissement, Erreur, Critique (toutes les catégories) |

Lorsque vous avez terminé, sélectionnez **Enregistrer**.

## <a name="enable-application-logging-linuxcontainer"></a>Activer la journalisation des applications (Linux/Conteneur)

Pour activer la journalisation des applications Linux et des applications de conteneur personnalisées dans le [portail Azure](https://portal.azure.com), accédez à votre application et sélectionnez **Journaux App Service**.

Dans **Journal des applications**, sélectionnez **Système de fichiers**.

Dans **Quota (Mo)** , spécifiez le quota de disque pour les journaux des applications. Dans **Période de conservation (jours)** , définissez le nombre de jours pendant lesquels les journaux doivent être conservés.

Lorsque vous avez terminé, sélectionnez **Enregistrer**.

## <a name="enable-web-server-logging"></a>Activer la journalisation de serveur Web

Pour activer la journalisation du serveur web pour les applications Windows dans le [portail Azure](https://portal.azure.com), accédez à votre application et sélectionnez **Journaux App Service**.

Pour **Journalisation du serveur web**, sélectionnez **Stockage** pour stocker les journaux dans le stockage d’objets blob, ou **Système de fichiers** pour stocker les journaux sur le système de fichiers App service. 

Dans **Période de conservation (jours)** , définissez le nombre de jours pendant lesquels les journaux doivent être conservés.

> [!NOTE]
> Si vous [régénérez les clés d’accès de votre compte de stockage](../storage/common/storage-account-create.md), vous devez réinitialiser la configuration de journalisation correspondante pour utiliser les clés mises à jour. Pour ce faire :
>
> 1. Sous l’onglet **Configurer**, définissez la fonctionnalité de journalisation correspondante sur **Désactivé**. Enregistrez votre paramètre.
> 2. Réactivez la journalisation de l’objet blob du compte de stockage. Enregistrez votre paramètre.
>
>

Lorsque vous avez terminé, sélectionnez **Enregistrer**.

## <a name="log-detailed-errors"></a>Consigner les erreurs détaillées

Pour enregistrer la page d’erreur ou le suivi des demandes ayant échoué pour les applications Windows dans le [portail Azure](https://portal.azure.com), accédez à votre application et sélectionnez **Journaux App Service**.

Sous **Messages d’erreurs détaillés** ou **Suivi des demandes ayant échoué**, sélectionnez **Activé**, puis sélectionnez **Enregistrer**.

Les deux types de journaux sont stockés dans le système de fichiers App Service. Jusqu’à 50 erreurs (fichiers/dossiers) sont conservées. Lorsque le nombre de fichiers HTML dépasse 50, les 26 erreurs les plus anciennes sont automatiquement supprimées.

## <a name="add-log-messages-in-code"></a>Ajouter des messages de journalisation dans le code

Dans votre code d’application, vous utilisez les fonctionnalités de journalisation habituelles pour envoyer des messages de journalisation aux journaux des applications. Par exemple :

- Les applications ASP.NET peuvent utiliser la classe [System.Diagnostics.Trace](/dotnet/api/system.diagnostics.trace) pour enregistrer des informations dans le journal de diagnostic d'application. Par exemple :

    ```csharp
    System.Diagnostics.Trace.TraceError("If you're seeing this, something bad happened");
    ```

- Par défaut, ASP.NET Core utilise le fournisseur de journalisation [Microsoft.Extensions.Logging.AzureAppServices](https://www.nuget.org/packages/Microsoft.Extensions.Logging.AzureAppServices). Pour plus d’informations, consultez l’article [ASP.NET Core logging in Azure](/aspnet/core/fundamentals/logging/) (Journalisation ASP.NET Core dans Azure).

## <a name="stream-logs"></a>Diffuser les journaux d’activité en continu

Avant de diffuser des journaux en temps réel, activez le type de journal souhaité. Toute information enregistrée dans un fichier doté de l’extension .txt, .log ou .htm et stocké dans le répertoire */LogFiles* (d:/home/logfiles) est diffusée par App Service.

> [!NOTE]
> Certains types de mémoire tampon de journalisation sont écrits dans le fichier journal. Dès lors, il se peut que les événements apparaissent de manière désordonnée dans le flux. Ainsi, il est possible qu'une entrée du journal d'application qui se produit lorsqu'un utilisateur visite une page soit affichée dans le flux avant l'entrée de journal HTTP correspondante pour la demande de page.
>

### <a name="in-azure-portal"></a>Dans le portail Azure

Pour diffuser les journaux dans le [portail Azure](https://portal.azure.com), accédez à votre application et sélectionnez **Flux de journaux**. 

### <a name="in-cloud-shell"></a>Dans Cloud Shell

Pour diffuser les journaux en temps réel dans [Cloud Shell](../cloud-shell/overview.md), utilisez la commande suivante :

```azurecli-interactive
az webapp log tail --name appname --resource-group myResourceGroup
```

Pour filtrer des événements spécifiques, tels que des erreurs, utilisez le paramètre **--Filter** . Par exemple :

```azurecli-interactive
az webapp log tail --name appname --resource-group myResourceGroup --filter Error
```
Pour filtrer des types de journaux spécifiques, tels que HTTP, utilisez le paramètre **--Path** . Par exemple :

```azurecli-interactive
az webapp log tail --name appname --resource-group myResourceGroup --path http
```

### <a name="in-local-terminal"></a>Dans le terminal local

Pour diffuser les journaux dans la console locale, [installez Azure CLI](/cli/azure/install-azure-cli) et [connectez-vous à votre compte](/cli/azure/authenticate-azure-cli). Une fois connecté, suivez les [instructions pour Cloud Shell](#in-cloud-shell)

## <a name="access-log-files"></a>Accéder aux fichiers journaux

Si vous configurez l’option des objets blob de stockage Azure pour un type de journal, vous avez besoin d’un outil client qui fonctionne avec le stockage Azure. Pour plus d’informations, consultez [Outils clients du stockage Azure](../storage/common/storage-explorers.md).

Pour les journaux stockés dans le système de fichiers App Service, le moyen le plus simple consiste à télécharger le fichier ZIP dans le navigateur à l’adresse suivante :

- Applications Linux/de conteneur : `https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip`
- Applications Windows : `https://<app-name>.scm.azurewebsites.net/api/dump`

Pour les applications Linux/de conteneur, le fichier ZIP contient les journaux de sortie de la console pour l’hôte Docker et le conteneur Docker. Pour une application avec scale-out, le fichier ZIP contient un ensemble de journaux pour chaque instance. Dans le système de fichiers App Service, ces fichiers journaux sont le contenu du répertoire */home/LogFiles*.

Pour les applications Windows, le fichier ZIP contient le contenu du répertoire *D:\Home\LogFiles* dans le système de fichiers App Service. Sa structure est la suivante :

| Type de journal | Répertoire | Description |
|-|-|-|
| **Journaux d’application** |*/LogFiles/Application/* | Contient un ou plusieurs fichiers texte. Le format des messages de journalisation dépend du fournisseur de journalisation que vous utilisez. |
| **Suivi des demandes ayant échoué** | */LogFiles/W3SVC#########/* | Contient des fichiers XML et un fichier XSL. Vous pouvez afficher les fichiers XML mis en forme dans le navigateur. |
| **Journaux d’erreurs détaillés** | */LogFiles/DetailedErrors/* | Contient les fichiers d’erreur HTM. Vous pouvez afficher les fichiers HTM dans le navigateur.<br/>Un autre moyen d’afficher le suivi des demandes ayant échoué consiste à accéder à la page de votre application dans le portail. Dans le menu de gauche, sélectionnez **Diagnostiquer et résoudre les problèmes**, puis recherchez **Journaux de traçage des requêtes ayant échoué**, puis cliquez sur l’icône pour parcourir et afficher la trace que vous souhaitez. |
| **Journaux de serveur web** | */LogFiles/http/RawLogs/* | Contient les fichiers texte au [format de fichier journal étendu W3C](/windows/desktop/Http/w3c-logging). Ces informations peuvent être lues à l’aide d’un éditeur de texte ou d’un utilitaire tel que [Log Parser](https://go.microsoft.com/fwlink/?LinkId=246619).<br/>App Service ne prend pas en charge les champs `s-computername`, `s-ip` ni `cs-version`. |
| **Journaux de déploiement** | */LogFiles/Git/* et */deployments/* | Contient les journaux générés par les processus de déploiement internes, ainsi que les journaux des déploiements Git. |

## <a name="send-logs-to-azure-monitor-preview"></a>Envoyer des journaux à Azure Monitor (préversion)

Grâce à la nouvelle [intégration d’Azure Monitor](https://aka.ms/appsvcblog-azmon), vous pouvez [créer des paramètres de diagnostic (préversion)](https://azure.github.io/AppService/2019/11/01/App-Service-Integration-with-Azure-Monitor.html#create-a-diagnostic-setting) pour envoyer des journaux aux comptes de stockage, à Event Hubs et à Log Analytics. 

> [!div class="mx-imgBorder"]
> ![Paramètres de diagnostic (préversion)](media/troubleshoot-diagnostic-logs/diagnostic-settings-page.png)

### <a name="supported-log-types"></a>Types de journaux pris en charge

Le tableau suivant renseigne sur les types et la descriptions des journaux pris en charge : 

| Type de journal | Windows | Conteneur Windows | Linux | Conteneur Linux | Description |
|-|-|-|-|-|-|
| AppServiceConsoleLogs | Java SE et Tomcat | Oui | Oui | Oui | Sortie standard et erreur standard |
| AppServiceHTTPLogs | Oui | Oui | Oui | Oui | Journaux d’activité des serveurs Web |
| AppServiceEnvironmentPlatformLogs | Oui | N/A | Oui | Oui | App Service Environment : mise à l’échelle, modifications de configuration et journaux d’état|
| AppServiceAuditLogs | Oui | Oui | Oui | Oui | Activité de connexion via FTP et Kudu |
| AppServiceFileAuditLogs | Oui | Oui | À confirmer | À confirmer | Modifications de fichier apportées au contenu du site ; disponible uniquement pour le niveau Premium et supérieur |
| AppServiceAppLogs | ASP .NET | ASP .NET | Java SE & Tomcat Blessed Images <sup>1</sup> | Java SE & Tomcat Blessed Images <sup>1</sup> | Journaux d’activité d’application |
| AppServiceIPSecAuditLogs  | Oui | Oui | Oui | Oui | Demandes à partir de règles IP |
| AppServicePlatformLogs  | À confirmer | Oui | Oui | Oui | journaux des opérations du conteneur |

<sup>1</sup> Pour les applications Java SE, ajoutez « $WEBSITE_AZMON_PREVIEW_ENABLED » aux paramètres de l’application et définissez-le sur 1 ou true.

## <a name="next-steps"></a><a name="nextsteps"></a>Étapes suivantes
* [Interrogation de journaux d’activité grâce à Azure Monitor](../azure-monitor/log-query/log-query-overview.md)
* [Surveillance des applications dans Azure App Service](web-sites-monitor.md)
* [Dépannage d’une application web dans le Service d’application Microsoft Azure à l’aide de Visual Studio](troubleshoot-dotnet-visual-studio.md)
* [Analyse des journaux d’activité d’application dans HDInsight (en anglais)](https://gallery.technet.microsoft.com/scriptcenter/Analyses-Windows-Azure-web-0b27d413)
