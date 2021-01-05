---
title: Configurer un conteneur personnalisé
description: Découvrez comment configurer un conteneur personnalisé dans Azure App Service. Cet article présente les tâches de configuration les plus courantes.
ms.topic: article
ms.date: 09/22/2020
zone_pivot_groups: app-service-containers-windows-linux
ms.openlocfilehash: a7582bbb866a63820abbd959e06628eda5d57e29
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007634"
---
# <a name="configure-a-custom-container-for-azure-app-service"></a>Configurer un conteneur personnalisé pour Azure App Service

Cet article montre comment configurer un conteneur personnalisé à exécuter sur Azure App Service.

::: zone pivot="container-windows"

Ce guide fournit les principaux concepts et instructions pour la conteneurisation d’applications Windows dans App Service. Si vous n’avez jamais utilisé Azure App Service, commencez par suivre le [démarrage rapide](quickstart-custom-container.md) et le [tutoriel](tutorial-custom-container.md) sur le conteneur personnalisé.

::: zone-end

::: zone pivot="container-linux"

Ce guide fournit les principaux concepts et des instructions pour la mise en conteneur d’applications Linux dans App Service. Si vous n’avez jamais utilisé Azure App Service, commencez par suivre le [démarrage rapide](quickstart-custom-container.md) et le [tutoriel](tutorial-custom-container.md) sur le conteneur personnalisé. Il existe également un [démarrage rapide](quickstart-multi-container.md) et un [tutoriel](tutorial-multi-container-app.md) pour les applications à conteneurs multiples.

::: zone-end

::: zone pivot="container-windows"

## <a name="supported-parent-images"></a>Images parentes prises en charge

Pour votre image Windows personnalisée, vous devez choisir l’[image parente (image de base)](https://docs.docker.com/develop/develop-images/baseimages/) appropriée pour l’infrastructure souhaitée :

- Pour déployer des applications .NET Framework, utilisez une image parente basée sur la version Windows Server Core [Long-Term Servicing Channel (LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc). 
- Pour déployer des applications .NET Core, utilisez une image parente basée sur la version Windows Server Nano [Semi-Annual Servicing Channel (SAC)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel). 

Le téléchargement d’une image parente lors du démarrage de l’application peut prendre un certain temps. Toutefois, vous pouvez réduire le temps de démarrage en utilisant l’une des images parentes suivantes déjà mises en cache dans Azure App Service :

- [mcr.microsoft.com/windows/servercore](https://hub.docker.com/_/microsoft-windows-servercore):2004
- [mcr.microsoft.com/windows/servercore](https://hub.docker.com/_/microsoft-windows-servercore):ltsc2019
- [mcr.microsoft.com/dotnet/framework/aspnet](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/):4.8-windowsservercore-2004
- [mcr.microsoft.com/dotnet/framework/aspnet](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/):4.8-windowsservercore-ltsc2019
- [mcr.microsoft.com/dotnet/core/runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/):3.1-nanoserver-2004
- [mcr.microsoft.com/dotnet/core/runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/):3.1-nanoserver-1909
- [mcr.microsoft.com/dotnet/core/runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/):3.1-nanoserver-1903
- [mcr.microsoft.com/dotnet/core/runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/):3.1-nanoserver-1809
- [mcr.microsoft.com/dotnet/core/aspnet](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/):3.1-nanoserver-2004
- [mcr.microsoft.com/dotnet/core/aspnet](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/):3.1-nanoserver-1909
- [mcr.microsoft.com/dotnet/core/aspnet](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/):3.1-nanoserver-1903
- [mcr.microsoft.com/dotnet/core/aspnet](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/):3.1-nanoserver-1809

::: zone-end

## <a name="change-the-docker-image-of-a-custom-container"></a>Modifier l’image Docker d’un conteneur personnalisé

Pour modifier une application de conteneur personnalisée existante de l’image Docker actuelle par une nouvelle image, utilisez la commande suivante :

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <docker-hub-repo>/<image>
```

## <a name="use-an-image-from-a-private-registry"></a>Utiliser une image à partir d’un registre privé

Pour utiliser une image à partir d’un registre privé, comme Azure Container Registry, exécutez la commande suivante :

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <image-name> --docker-registry-server-url <private-repo-url> --docker-registry-server-user <username> --docker-registry-server-password <password>
```

Pour *\<username>* et *\<password>* , fournissez les informations d’identification de connexion pour votre compte de registre privé.

## <a name="i-dont-see-the-updated-container"></a>Je ne vois pas le conteneur mis à jour

Si vous modifiez vos paramètres de conteneur Docker pour qu’ils pointent vers un nouveau conteneur, l’application peut prendre quelques minutes avant de servir les requêtes HTTP du nouveau conteneur. Pendant l’extraction et le démarrage du nouveau conteneur, App Service continue de servir les requêtes de l’ancien conteneur. Ce n’est qu’une fois le nouveau conteneur démarré et prêt à recevoir des requêtes qu’App Service commence à lui envoyer des requêtes.

## <a name="how-container-images-are-stored"></a>Stockage d’images de conteneur

La première fois que vous exécutez une image Docker personnalisée dans App Service, App Service effectue une opération `docker pull` et extrait toutes les couches d’image. Ces couches sont stockées sur disque, comme si vous utilisiez Docker localement. À chaque redémarrage de l’application, App Service effectue une opération `docker pull`, mais extrait uniquement les couches qui ont changé. Si aucune modification n’a été apportée, App Service utilise les couches existantes sur le disque local.

Si l’application modifie des instances de calcul pour une raison quelconque, telle que la mise à l’échelle des niveaux tarifaires, App Service doit de nouveau extraire toutes les couches. Il en va de même si vous effectuez un scale-out pour ajouter des instances. Il existe également de rares cas où les instances d’application peuvent changer sans opération de mise à l’échelle.

## <a name="configure-port-number"></a>Configurer le numéro de port

Par défaut, App Service suppose que votre conteneur personnalisé écoute sur le port 80. Si votre conteneur écoute un port différent, définissez le paramètre d’application `WEBSITES_PORT` dans votre application App Service. Vous pouvez le définir via le [Cloud Shell](https://shell.azure.com). Dans Bash :

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITES_PORT=8000
```

Dans PowerShell :

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITES_PORT"="8000"}
```

App Service autorise actuellement votre conteneur à exposer un seul port pour les requêtes HTTP. 

## <a name="configure-environment-variables"></a>Configuration des variables d’environnement

Votre conteneur personnalisé peut utiliser des variables d’environnement qui doivent être fournies en externe. Vous pouvez les transmettre via le [Cloud Shell](https://shell.azure.com). Dans Bash :

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings DB_HOST="myownserver.mysql.database.azure.com"
```

Dans PowerShell :

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"DB_HOST"="myownserver.mysql.database.azure.com"}
```

Lorsque votre application s’exécute, les paramètres d’application App Service sont automatiquement injectés dans le processus en tant que variables d’environnement. 

::: zone pivot="container-windows"
Pour des conteneurs basés sur IIS ou .NET Framework (version 4.0 ou ultérieure), ils sont automatiquement injectés par App Service dans `System.ConfigurationManager` en tant que paramètres d’application .NET et chaînes de connexion. Pour toute autre langage ou infrastructure, ils sont fournis en tant que variables d’environnement pour le processus, avec l’un des préfixes correspondants suivants :

- `APPSETTING_`
- `SQLCONTR_`
- `MYSQLCONTR_`
- `SQLAZURECOSTR_`
- `POSTGRESQLCONTR_`
- `CUSTOMCONNSTR_`

::: zone-end

::: zone pivot="container-linux"

Cette méthode fonctionne à la fois pour les applications à conteneur unique et à plusieurs conteneurs, où les variables d’environnement sont spécifiées dans le fichier *docker-compose.yml*.

::: zone-end

## <a name="use-persistent-shared-storage"></a>Utiliser le stockage partagé persistant

::: zone pivot="container-windows"

Vous pouvez utiliser le répertoire *C:\home* dans le système de fichiers de votre application pour conserver les fichiers au fil des redémarrages, et les partager entre des instances. Le répertoire `C:\home` dans votre application est fourni pour permettre à votre application de conteneur d’accéder au stockage persistant.

Lorsque le stockage persistant est désactivé, les écritures dans le répertoire `C:\home` ne sont pas conservées. Les [journaux d’hôte de conteneur Docker](#access-diagnostic-logs) sont enregistrés dans un stockage partagé persistant par défaut, qui n’est pas attaché au conteneur. Lorsque le stockage persistant est activé, toutes les écritures dans le répertoire `C:\home` sont conservées et accessibles par toutes les instances d’une application ayant fait l’objet d’une montée en charge parallèle, et les journaux sont accessibles dans `C:\home\LogFiles`.

::: zone-end

::: zone pivot="container-linux"

Vous pouvez utiliser le répertoire */home* dans le système de fichiers de votre application pour rendre persistants les fichiers entre les redémarrages et les partager entre plusieurs instances. Le répertoire `/home` dans votre application est fourni pour permettre à votre application de conteneur d’accéder au stockage persistant.

Lorsque le stockage persistant est désactivé, les écritures dans le répertoire `/home` ne sont pas rendues persistantes entre les redémarrages d’applications ou entre plusieurs instances. La seule exception concerne le répertoire `/home/LogFiles`, qui est utilisé pour stocker les journaux Docker et de conteneurs. Lorsque le stockage persistant est activé, toutes les écritures dans le répertoire `/home` sont rendues persistantes et sont accessibles par toutes les instances d’une application avec scale-out.

::: zone-end

Par défaut, le stockage persistant est désactivé, et le paramètre n’est pas exposé dans les paramètres d’application. Pour l’activer, définissez le paramètre d’application `WEBSITES_ENABLE_APP_SERVICE_STORAGE` via le [Cloud Shell](https://shell.azure.com). Dans Bash :

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=true
```

Dans PowerShell :

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITES_ENABLE_APP_SERVICE_STORAGE"=true}
```

> [!NOTE]
> Vous pouvez également [configurer votre propre stockage persistant](configure-connect-to-azure-storage.md).

## <a name="detect-https-session"></a>Détecter une session HTTPS

App Service termine les requêtes TLS/SSL au niveau des serveurs frontaux. Cela signifie que les requêtes TLS/SSL n’atteignent jamais votre application. Vous n’avez pas besoin et devez vous abstenir d’implémenter la prise en charge des protocoles TLS/SSL dans votre application. 

Les serveurs frontaux se trouvent dans des centres de données Azure. Si vous utilisez les protocoles TLS/SSL avec votre application, votre trafic sur Internet sera toujours chiffré en toute sécurité.

::: zone pivot="container-windows"

## <a name="customize-aspnet-machine-key-injection"></a>Personnaliser l’injection de clé machine ASP.NET

 Pendant le démarrage du conteneur, des clés générées automatiquement sont injectées dans le conteneur en tant que clés machine pour les routines de chiffrement ASP.NET. Vous pouvez [rechercher ces clés dans votre conteneur](#connect-to-the-container) en recherchant les variables d’environnement suivantes : `MACHINEKEY_Decryption`, `MACHINEKEY_DecryptionKey`, `MACHINEKEY_ValidationKey`, `MACHINEKEY_Validation`. 

Les nouvelles clés générées à chaque redémarrage peuvent réinitialiser l’authentification par formulaires ASP.NET et l’état d’affichage, si votre application en dépend. Pour empêcher la régénération automatique des clés, [définissez-les manuellement en tant que paramètres d’application App Service](#configure-environment-variables). 

## <a name="connect-to-the-container"></a>Connexion au conteneur

Vous pouvez vous connecter directement à votre conteneur Windows pour les tâches de diagnostic en accédant à `https://<app-name>.scm.azurewebsites.net/DebugConsole`. Fonctionnement de l’opération :

- La console de débogage vous permet d’exécuter des commandes interactives, telles que le démarrage de sessions PowerShell, l’inspection de clés de Registre et la navigation dans l’ensemble du système de fichiers du conteneur.
- Elle fonctionne séparément du navigateur graphique, au-dessus de celui-ci, et n’affiche que les fichiers de votre [stockage partagé](#use-persistent-shared-storage).
- Dans une application ayant fait l’objet d’une montée en charge parallèle, la console de débogage est connectée à l’une des instances de conteneur. Vous pouvez sélectionner une autre instance dans la liste déroulante **Instance** dans le menu supérieur.
- Les modifications apportées au conteneur à partir de la console ne sont *pas* conservée après un redémarrage de votre application (à l’exception des modifications apportées dans le stockage partagé), car elles ne font pas partie de l’image Docker. Pour conserver vos modifications, telles que des paramètres du Registre et l’installation de logiciels, intégrez-les dans le fichier Dockerfile.

## <a name="access-diagnostic-logs"></a>Accéder aux journaux de diagnostic

App Service journalise les actions de l’hôte Docker, ainsi que les activités en provenance du conteneur. Les journaux de l’hôte Docker (journaux de plateforme) sont livrés par défaut, mais les journaux d’application ou de serveur web en provenance du conteneur doivent être activés manuellement. Pour plus d’informations, consultez [Activer la journalisation des applications](troubleshoot-diagnostic-logs.md#enable-application-logging-linuxcontainer) et [Activer la journalisation du serveur web](troubleshoot-diagnostic-logs.md#enable-web-server-logging). 

Il existe plusieurs moyens d’accéder aux journaux du Docker :

- [Dans le portail Azure](#in-azure-portal)
- [À partir de la console Kudu](#from-the-kudu-console)
- [Avec l’API Kudu](#with-the-kudu-api)
- [Envoyer des journaux à Azure Monitor](troubleshoot-diagnostic-logs.md#send-logs-to-azure-monitor-preview)

### <a name="in-azure-portal"></a>Dans le portail Azure

Les journaux du Docker sont affichés sur le portail, dans la page **Paramètres du conteneur** de votre application. Les journaux sont tronqués, mais vous pouvez les télécharger tous en cliquant sur **Télécharger**. 

### <a name="from-the-kudu-console"></a>À partir de la console Kudu

Accédez à `https://<app-name>.scm.azurewebsites.net/DebugConsole`, puis cliquez sur le dossier **LogFiles** pour afficher les fichiers journaux individuels. Pour télécharger l’intégralité du répertoire **LogFiles**, cliquez sur l’icône **Télécharger** à gauche du nom du répertoire. Vous pouvez également accéder à ce dossier à l’aide d’un client FTP.

Dans le terminal de la console, vous ne pouvez pas accéder au dossier `C:\home\LogFiles` par défaut, car le stockage partagé persistant n’est pas activé. Pour activer ce comportement dans le terminal de la console, vous devez [activer le stockage partagé persistant](#use-persistent-shared-storage).

Si vous tentez de télécharger le journal de Docker actuellement utilisé à l’aide d’un client FTP, vous risquez d’obtenir une erreur en raison d’un verrou de fichier.

### <a name="with-the-kudu-api"></a>Avec l’API Kudu

Accédez directement à `https://<app-name>.scm.azurewebsites.net/api/logs/docker` pour afficher les métadonnées des journaux de Docker. Il se peut que plusieurs fichiers journaux soient répertoriés. La propriété `href` vous permet de télécharger le fichier journal directement. 

Pour télécharger tous les journaux dans un fichier ZIP, accédez à `https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip`.

## <a name="customize-container-memory"></a>Personnaliser la mémoire du conteneur

Par défaut, tous les conteneurs Windows déployés dans Azure App Service sont limités à 1 Go de RAM. Vous pouvez modifier cette valeur en fournissant le paramètre d’application `WEBSITE_MEMORY_LIMIT_MB` à l’aide du [Cloud Shell](https://shell.azure.com). Dans Bash :

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITE_MEMORY_LIMIT_MB=2000
```

Dans PowerShell :

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITE_MEMORY_LIMIT_MB"=2000}
```

La valeur est définie en Mo et doit être inférieure ou égale à la mémoire physique totale de l’hôte. Par exemple, dans un plan de App Service avec 8 Go de RAM, le total cumulé de `WEBSITE_MEMORY_LIMIT_MB` pour toutes les applications ne doit pas dépasser 8 Go. Pour plus d’informations sur la quantité de mémoire disponible pour chaque niveau tarifaire, consultez [Tarification d’App Service](https://azure.microsoft.com/pricing/details/app-service/windows/), dans la section **Plan de conteneur Premium (Windows)** .

## <a name="customize-the-number-of-compute-cores"></a>Personnaliser le nombre de cœurs de calcul

Par défaut, un conteneur Windows s’exécute avec tous les cœurs disponibles pour le niveau tarifaire choisi. Vous pouvez par exemple réduire le nombre de cœurs qu’utilise votre emplacement de préproduction. Pour réduire le nombre de cœurs qu’utilise un conteneur, définissez le paramètre d’application `WEBSITE_CPU_CORES_LIMIT` sur le nombre préféré de cœurs. Vous pouvez le définir via le [Cloud Shell](https://shell.azure.com). Dans Bash :

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --slot staging --settings WEBSITE_CPU_CORES_LIMIT=1
```

Dans PowerShell :

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITE_CPU_CORES_LIMIT"=1}
```

> [!NOTE]
> La mise à jour du paramètre d’application déclenche un redémarrage automatique qui entraîne un temps d’arrêt minimal. Pour une application de production, envisagez de la permuter dans un emplacement de préproduction, modifiez le paramètre d’application dans l’emplacement de préproduction, puis re-permutez-la en production.

Vérifiez votre nombre ajusté en accédant à la console Kudu (`https://<app-name>.scm.azurewebsites.net`) et en tapant les commandes suivantes à l’aide de PowerShell. Chaque commande génère un nombre.

```PowerShell
Get-ComputerInfo | ft CsNumberOfLogicalProcessors # Total number of enabled logical processors. Disabled processors are excluded.
Get-ComputerInfo | ft CsNumberOfProcessors # Number of physical processors.
```

Les processeurs peuvent être de type multicœur ou hyperthreading. Pour plus d’informations sur le nombre de cœurs disponibles pour chaque niveau tarifaire, consultez [Tarification d’App Service](https://azure.microsoft.com/pricing/details/app-service/windows/), dans la section **Plan de conteneur Premium (Windows)** .

## <a name="customize-health-ping-behavior"></a>Personnaliser le comportement de test Ping d’intégrité

App Service considère que le démarrage d’un conteneur est correct quand le conteneur démarre et répond à un test Ping HTTP. La requête Ping d’intégrité contient l’en-tête `User-Agent= "App Service Hyper-V Container Availability Check"`. Si le conteneur démarre mais ne répond pas à un test Ping après un certain temps, App Service journalise un événement dans le journal de Docker, indiquant que le conteneur n’a pas démarré. 

Si votre application est gourmande en ressources, il se peut que le conteneur ne réponde à temps pas à la requête Ping HTTP. Pour contrôler les actions en cas d’échec des tests Ping HTTP, définissez le paramètre d’application `CONTAINER_AVAILABILITY_CHECK_MODE`. Vous pouvez le définir via le [Cloud Shell](https://shell.azure.com). Dans Bash :

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings CONTAINER_AVAILABILITY_CHECK_MODE="ReportOnly"
```

Dans PowerShell :

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"CONTAINER_AVAILABILITY_CHECK_MODE"="ReportOnly"}
```

Le tableau suivant présente les valeurs possibles :

| Valeur | Descriptions |
| - | - |
| **Repair** | Redémarrer le conteneur après trois vérifications de disponibilité consécutives |
| **ReportOnly** | Valeur par défaut. Ne pas redémarrer le conteneur mais signaler dans les journaux de Docker du conteneur après trois vérifications de disponibilité consécutives. |
| **Désactivé** | Ne pas vérifier la disponibilité. |

## <a name="support-for-group-managed-service-accounts"></a>Prise en charge des comptes de service géré par un groupe

Les comptes de service géré de groupe (GMSA) ne sont actuellement pas pris en charge dans les conteneurs Windows dans App Service.

::: zone-end

::: zone pivot="container-linux"

## <a name="enable-ssh"></a>Activation de SSH

SSH permet d’établir une communication sécurisée entre un conteneur et un client. Pour qu’un conteneur personnalisé prenne en charge SSH, vous devez l’ajouter dans le fichier Dockerfile lui-même.

> [!TIP]
> Tous les conteneurs Linux intégrés ont ajouté les instructions SSH dans leurs dépôts d’images. Vous pouvez parcourir les instructions suivantes avec le [dépôt Node.js 10.14](https://github.com/Azure-App-Service/node/blob/master/10.14) afin de déterminer comment il y est activé.

- Utilisez l’instruction [Run](https://docs.docker.com/engine/reference/builder/#run) pour installer le serveur SSH et définir le mot de passe du compte racine sur `"Docker!"`. Par exemple, pour une image basée sur [Alpine Linux](https://hub.docker.com/_/alpine), vous avez besoin des commandes suivantes :

    ```Dockerfile
    RUN apk add openssh \
         && echo "root:Docker!" | chpasswd 
    ```

    Cette configuration n’autorise pas les connexions externes avec le conteneur. SSH est disponible uniquement via `https://<app-name>.scm.azurewebsites.net` et authentifié avec les informations d’identification de publication.

- Ajoutez [ce fichier sshd_config](https://github.com/Azure-App-Service/node/blob/master/10.14/sshd_config) à votre dépôt d’images et utilisez l’instruction [Copy](https://docs.docker.com/engine/reference/builder/#copy) pour copier le fichier dans le répertoire */etc/ssh/*. Pour plus d’informations sur les fichiers *sshd_config*, consultez [la documentation OpenBSD](https://man.openbsd.org/sshd_config).

    ```Dockerfile
    COPY sshd_config /etc/ssh/
    ```

    > [!NOTE]
    > Le fichier *sshd_config* doit inclure les éléments suivants :
    > - `Ciphers` doit inclure au moins un élément dans cette liste : `aes128-cbc,3des-cbc,aes256-cbc`.
    > - `MACs` doit inclure au moins un élément dans cette liste : `hmac-sha1,hmac-sha1-96`.

- Utilisez l’instruction [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose) pour ouvrir le port 2222 dans le conteneur. Bien que le mot de passe racine soit connu, le port 2222 n’est pas accessible à partir d’Internet. Il est accessible uniquement par les conteneurs au sein du pont réseau d’un réseau privé virtuel.

    ```Dockerfile
    EXPOSE 80 2222
    ```

- Dans le script de démarrage pour votre conteneur, démarrez le serveur SSH.

    ```bash
    /usr/sbin/sshd
    ```

    Pour obtenir un exemple, voyez comment le [conteneur Node.js 10.14 par défaut](https://github.com/Azure-App-Service/node/blob/master/10.14/startup/init_container.sh) démarre le serveur SSH.

## <a name="access-diagnostic-logs"></a>Accéder aux journaux de diagnostic

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

## <a name="configure-multi-container-apps"></a>Configurer des applications à plusieurs conteneurs

- [Utiliser le stockage persistant dans Docker Compose](#use-persistent-storage-in-docker-compose)
- [Limitations de la version préliminaire](#preview-limitations)
- [Options Docker Compose](#docker-compose-options)

### <a name="use-persistent-storage-in-docker-compose"></a>Utiliser le stockage persistant dans Docker Compose

Les applications multiconteneurs telles que WordPress nécessitent un stockage persistant pour fonctionner correctement. Pour l’activer, votre configuration Docker Compose doit pointer vers un emplacement de stockage *à l’extérieur* de votre conteneur. Les emplacements de stockage à l’intérieur de votre conteneur ne rendent pas les modifications persistantes au-delà du redémarrage de l’application.

Activez le stockage persistant en définissant le paramètre d’application `WEBSITES_ENABLE_APP_SERVICE_STORAGE`, à l’aide de la commande [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set) dans le [Cloud Shell](https://shell.azure.com).

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=TRUE
```

Dans votre fichier *docker-compose.yml*, mappez l’option `volumes` sur `${WEBAPP_STORAGE_HOME}`. 

`WEBAPP_STORAGE_HOME` est une variable d’environnement d’App Service mappée sur le stockage persistant de votre application. Par exemple :

```yaml
wordpress:
  image: wordpress:latest
  volumes:
  - ${WEBAPP_STORAGE_HOME}/site/wwwroot:/var/www/html
  - ${WEBAPP_STORAGE_HOME}/phpmyadmin:/var/www/phpmyadmin
  - ${WEBAPP_STORAGE_HOME}/LogFiles:/var/log
```

### <a name="preview-limitations"></a>Limitations de la version préliminaire

La fonctionnalité multiconteneurs est actuellement en préversion. Les fonctionnalités suivantes de la plateforme App Service ne sont pas prises en charge :

- Authentification / Autorisation
- Identités managées
- CORS

### <a name="docker-compose-options"></a>Options Docker Compose

Les listes suivantes présentent des options de configuration Docker Compose prises en charge et non prises en charge :

#### <a name="supported-options"></a>Options prises en charge

- command
- entrypoint
- Environnement
- image
- ports
- restart
- services
- volumes

#### <a name="unsupported-options"></a>Options non prises en charge

- build (non autorisée)
- depends_on (ignorée)
- networks (ignorée)
- secrets (ignorée)
- ports autres que 80 et 8080 (ignorées)

> [!NOTE]
> Les autres options qui ne sont pas explicitement appelées sont ignorées dans la préversion publique.

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

::: zone-end

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Migrer des logiciels personnalisés vers Azure App Service à l’aide d’un conteneur personnalisé](tutorial-custom-container.md)

::: zone pivot="container-linux"

> [!div class="nextstepaction"]
> [Tutoriel : Application WordPress avec plusieurs conteneurs](tutorial-multi-container-app.md)

::: zone-end

Ou bien, consultez les ressources supplémentaires :

[Charger le certificat dans les conteneurs Windows/Linux](configure-ssl-certificate-in-code.md#load-certificate-in-linuxwindows-containers)
