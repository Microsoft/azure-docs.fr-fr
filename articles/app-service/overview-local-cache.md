---
title: Cache local
description: Découvrez comment le cache local fonctionne dans Azure App Service, et comment activer, redimensionner et interroger l’état du cache local de votre application.
tags: optional
ms.assetid: e34d405e-c5d4-46ad-9b26-2a1eda86ce80
ms.topic: article
ms.date: 03/04/2016
ms.custom: seodec18
ms.openlocfilehash: 81782f63199a9fe8f43f56aeefcd1c68951d57a4
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96852250"
---
# <a name="azure-app-service-local-cache-overview"></a>Présentation du cache local d’Azure App Service

> [!NOTE]
> Le cache local n’est pas pris en charge dans les applications de fonction ni dans les applications App Service en conteneur, telles que dans [Conteneurs Windows](quickstart-custom-container.md?pivots=container-windows) ou sur [App Service sur Linux](overview.md#app-service-on-linux).


Le contenu Azure App Service est stocké dans le Stockage Azure et est exposé de manière durable en tant que partage de contenu. Destinée à fonctionner avec de nombreuses applications, cette conception présente les caractéristiques suivantes :  

* Le contenu est partagé entre plusieurs instances de machine virtuelle de l’application.
* Le contenu est durable et peut être modifié en exécutant des applications.
* Les fichiers journaux et les fichiers de données de diagnostic sont disponibles sous le même dossier de contenu partagé.
* La publication d’un nouveau contenu met directement à jour le dossier de contenu, que vous pouvez consulter tout de suite via le site web SCM et l’application en cours d’exécution (pour obtenir le contenu le plus récent, certaines technologies, comme ASP.NET, lancent généralement un redémarrage de l’application quand des changements sont apportés aux fichiers).

Tandis que de nombreuses applications utilisent une seule ou la totalité de ces fonctionnalités, certaines autres ont uniquement besoin d’un magasin de contenu en lecture seule très performant à partir duquel elles peuvent s’exécuter avec une haute disponibilité. Ces applications peuvent tirer profit d’une instance de machine virtuelle sur un cache local spécifique.

La fonctionnalité de cache local d’Azure App Service fournit une vue de rôle web de votre contenu. Ce contenu est un cache d’écriture avec rejet de votre contenu de stockage qui est créé de façon asynchrone au démarrage du site. Quand le cache est prêt, le site est basculé pour s’exécuter sur le contenu mis en cache. Les applications qui s’exécutent sur le cache local bénéficient des avantages suivants :

* Elles sont protégées contre les latences qui se produisent quand elles accèdent au contenu sur Azure Storage.
* Elles ne sont pas affectées par les mises à niveau planifiées ou les temps d’arrêt imprévus, ni par d’autres interruptions éventuelles d’Azure Storage sur les serveurs qui fournissent le partage de contenu.
* Elles ne redémarrent pas systématiquement après des modifications du partage de stockage.

## <a name="how-the-local-cache-changes-the-behavior-of-app-service"></a>Comment le cache local change le comportement d’App Service
* _D:\home_ pointe vers le cache local, qui est créé sur l’instance de machine virtuelle au démarrage de l’application. _D:\local_ continue de pointer vers le stockage propre à la machine virtuelle temporaire.
* Le cache local contient une copie unique des dossiers _/site_ et _/siteextensions_ du magasin de contenu partagé dans _D:\home\site_ et _D:\home\siteextensions_, respectivement. Les fichiers sont copiés dans le cache local, au démarrage de l’application. La taille des deux dossiers pour chaque application est limitée à 1 Go par défaut, mais vous pouvez l’augmenter à 2 Go. Notez que le temps de chargement du cache s’allonge proportionnellement à l’augmentation de la taille du cache. Si vous avez augmenté la limite du cache local à 2 Go et que les fichiers copiés dépassent la taille maximale de 2 Go, App Service ignore silencieusement le cache local et lit à partir du partage de fichiers distant. Si aucune limite n’est définie ou si la limite est définie sur une valeur inférieure à 2 Go et que les fichiers copiés dépassent la limite, le déploiement ou l’échange peut échouer avec une erreur.
* Le cache local est en lecture-écriture. Toutefois, toute modification est ignorée quand l’application change de machine virtuelle ou est redémarrée. N’utilisez pas le cache local pour des applications qui stockent des données stratégiques dans le magasin de contenu.
* _D:\home\LogFiles_ et _D:\home\Data_ contiennent des fichiers journaux et des données d’application. Les deux sous-dossiers sont stockés localement sur l’instance de machine virtuelle et sont copiés régulièrement dans le magasin de contenu partagé. Les applications peuvent conserver des fichiers journaux et des données en les écrivant dans ces dossiers. Toutefois, la copie dans le magasin de contenu partagé est une technique de « meilleur effort », vous n’êtes donc pas à l’abri d’une perte des fichiers journaux et des données en cas d’incident soudain sur une instance de machine virtuelle.
* Le [streaming des journaux](troubleshoot-diagnostic-logs.md#stream-logs) est affecté par la copie de « meilleur effort ». Vous pouvez observer jusqu’à une minute de délai dans les journaux d’activité diffusés en continu.
* Dans le magasin de contenu partagé, la structure des dossiers _LogFiles_ et _Data_ change pour les applications qui utilisent le cache local. Ces dossiers contiennent maintenant des sous-dossiers dont le nom est formé d’un « identificateur unique » et d’un horodatage. Chaque sous-dossier correspond à une instance de machine virtuelle sur laquelle l’application est en cours d’exécution ou s’est exécutée.
* Les autres dossiers de _D:\home_ restent dans le cache local et ne sont pas copiés dans le magasin de contenu partagé.
* Le déploiement d’applications par n’importe quelle méthode prise en charge publie directement dans le magasin de contenu partagé durable. Pour actualiser les dossiers _D:\home\site_ et _D:\home\siteextensions_ dans le cache local, l’application doit être redémarrée. vous pouvez rendre le cycle de vie transparent. Pour plus d’informations, consultez la suite de cet article.
* L’affichage de contenu par défaut du site SCM continue à être celui du magasin de contenu partagé.

## <a name="enable-local-cache-in-app-service"></a>Activer le cache local dans App Service 

> [!NOTE]
> Le cache local n’est pas pris en charge pour les niveaux **F1** et **D1**. 

Configurez le cache local à l’aide d’une combinaison de paramètres d’application réservés. Pour configurer ces paramètres d’application, vous pouvez utiliser les méthodes suivantes :

* [Azure portal](#Configure-Local-Cache-Portal)
* [Azure Resource Manager](#Configure-Local-Cache-ARM)

### <a name="configure-local-cache-by-using-the-azure-portal"></a>Configurer le cache local à l’aide du portail Azure
<a name="Configure-Local-Cache-Portal"></a>

Activez le cache local pour chaque application web en utilisant ce paramètre d’application : `WEBSITE_LOCAL_CACHE_OPTION` = `Always`  

![Paramètres d’application du Portail Azure : cache local](media/app-service-local-cache-overview/app-service-local-cache-configure-portal.png)

### <a name="configure-local-cache-by-using-azure-resource-manager"></a>Configurer le cache local à l’aide d’Azure Resource Manager
<a name="Configure-Local-Cache-ARM"></a>

```json

...

{
    "apiVersion": "2015-08-01",
    "type": "config",
    "name": "appsettings",
    "dependsOn": [
        "[resourceId('Microsoft.Web/sites/', variables('siteName'))]"
    ],

    "properties": {
        "WEBSITE_LOCAL_CACHE_OPTION": "Always",
        "WEBSITE_LOCAL_CACHE_SIZEINMB": "1000"
    }
}

...
```

## <a name="change-the-size-setting-in-local-cache"></a>Modifier le paramètre de taille dans le cache local
Par défaut, la taille du cache local est de **1 Go**. Elle inclut les dossiers /site et /siteextensions qui sont copiés à partir du magasin de contenu, ainsi que tous les dossiers de journaux d’activité et de données créés localement. Pour augmenter cette limite, utilisez le paramètre d’application `WEBSITE_LOCAL_CACHE_SIZEINMB`. Vous pouvez augmenter la taille jusqu’à **2 Go** (2000 Mo) par application. Notez que le temps de chargement du cache s’allonge proportionnellement à l’augmentation de sa taille.

## <a name="best-practices-for-using-app-service-local-cache"></a>Bonnes pratiques pour utiliser le cache local d’App Service
Nous vous recommandons d’utiliser le cache local conjointement avec la fonctionnalité [Environnements de préproduction](../app-service/deploy-staging-slots.md) .

* Ajoutez le paramètre d’application *associé*`WEBSITE_LOCAL_CACHE_OPTION` avec la valeur `Always` à votre emplacement de **production**. Si vous utilisez le paramètre d’application `WEBSITE_LOCAL_CACHE_SIZEINMB`, ajoutez-le également comme paramètre associé à votre emplacement de production.
* Créez un emplacement de **préproduction** pour la publication. En règle générale, si vous utilisez le cache local pour l’emplacement de production, vous n’avez pas à définir l’emplacement de préproduction pour utiliser le cache local en vue d’implémenter un cycle de vie build-déploiement-test transparent.
* Testez votre site par rapport à votre emplacement de préproduction.  
* Quand vous êtes prêt, lancez une [opération d’échange](../app-service/deploy-staging-slots.md#Swap) entre vos emplacements de préproduction et de production.  
* Les paramètres associés incluent un nom et sont rattachés à un emplacement. Ainsi, quand l’emplacement de préproduction est échangé avec l’emplacement de production, il hérite des paramètres d’application du cache local. L’emplacement de production qui vient d’être échangé s’exécute sur le cache local après quelques minutes. Il est ensuite initialisé dans le cadre de l’initialisation des emplacements après l’échange. Une fois l’échange des emplacements terminé, votre emplacement de production s’exécute sur le cache local.

## <a name="frequently-asked-questions-faq"></a>Forum Aux Questions (FAQ)

### <a name="how-can-i-tell-if-local-cache-applies-to-my-app"></a>Comment savoir si mon application peut bénéficier de la fonctionnalité de cache local ?
Utilisez la fonctionnalité de cache local si votre application a besoin d’un magasin de contenu fiable et très performant, si elle n’utilise pas le magasin de contenu pour écrire des données stratégiques au moment de l’exécution et si elle a une taille totale inférieure à 2 Go. Vous pouvez obtenir la taille totale de vos dossiers /site et /siteextensions en utilisant l’extension de site Utilisation du disque d’Azure Web Apps.

### <a name="how-can-i-tell-if-my-site-has-switched-to-using-local-cache"></a>Comment savoir si mon site a basculé pour utiliser le cache local ?
Si vous utilisez la fonctionnalité de cache local avec des environnements de préproduction, l’opération d’échange prend fin seulement après l’initialisation du cache local. Pour vérifier si votre site s’exécute sur le cache local, examinez la variable d’environnement de processus de travail `WEBSITE_LOCALCACHE_READY`. Suivez les instructions fournies dans la page de la [variable d’environnement de processus de travail](https://github.com/projectkudu/kudu/wiki/Process-Threads-list-and-minidump-gcdump-diagsession#process-environment-variable) pour accéder à cette variable sur plusieurs instances.  

### <a name="i-just-published-new-changes-but-my-app-does-not-seem-to-have-them-why"></a>Je viens de publier de nouvelles modifications, mais mon application ne semble pas les avoir intégrées. Pourquoi ?
Si votre application utilise le cache local, vous devez redémarrer votre site pour voir les dernières modifications. Si vous ne voulez pas publier les modifications sur un site de production, consultez les options d’emplacement décrites dans la section sur les bonnes pratiques, plus haut dans cet article.

> [!NOTE]
> L’option de déploiement [exécuter à partir du package](deploy-run-package.md) n’est pas compatible avec un cache local.

### <a name="where-are-my-logs"></a>Où sont mes journaux d’activité ?
Avec le cache local, vos dossiers de données et de journaux d’activité se présentent un peu différemment. Toutefois, la structure de vos sous-dossiers reste la même, excepté que les sous-dossiers se trouvent sous un sous-dossier dont le nom est formé d’un identificateur de machine virtuelle unique et d’un horodatage.

### <a name="i-have-local-cache-enabled-but-my--app-still-gets-restarted-why-is-that-i-thought-local-cache-helped-with-frequent-app-restarts"></a>J’ai activé le cache local, mais mon application redémarre systématiquement. Pourquoi ? Je pensais que le cache local évitait les redémarrages d’application fréquents.
En effet, le cache local contribue à limiter les redémarrages d’application liés au stockage. Toutefois, des redémarrages de votre application peuvent toujours être nécessaires pendant les mises à niveau planifiées de l’infrastructure de la machine virtuelle. Quand le cache local est activé, les redémarrages d’application globaux sont normalement moins nombreux.

### <a name="does-local-cache-exclude-any-directories-from-being-copied-to-the-faster-local-drive"></a>Le cache local exclut-il des répertoires de la copie vers le disque local plus rapide ?
Durant l’étape de copie du contenu du stockage, tous les dossiers étant des référentiels nommés sont exclus. Cela est utile pour les scénarios où le contenu de votre site peut contenir un dépôt de contrôle de code source qui n’est pas nécessaire dans une utilisation quotidienne de l’application. 

### <a name="how-to-flush-the-local-cache-logs-after-a-site-management-operation"></a>Comment vider les journaux du cache local après une opération de gestion de site ?
Pour vider les journaux du cache local, arrêtez et redémarrez l’application. Cette action efface l’ancien cache. 
