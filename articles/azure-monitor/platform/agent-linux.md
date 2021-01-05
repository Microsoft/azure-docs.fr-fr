---
title: Installer l’agent Log Analytics sur des ordinateurs Linux
description: Cet article décrit la connexion d’ordinateurs Linux hébergés dans d’autres clouds ou localement à Azure Monitor avec l’agent Log Analytics pour Linux.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/21/2020
ms.openlocfilehash: e1dbf5e20aa206189397cab26e9b867f4942e1d5
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94886836"
---
# <a name="install-log-analytics-agent-on-linux-computers"></a>Installer l’agent Log Analytics sur des ordinateurs Linux
Cet article fournit des détails sur l’installation de l’agent Log Analytics sur des ordinateurs Linux à l’aide des méthodes suivantes :

* [Installez l’agent pour Linux à l’aide d’un script wrapper](#install-the-agent-using-wrapper-script) hébergé sur GitHub. Il s’agit de la méthode recommandée pour installer et mettre à niveau l'agent lorsque l'ordinateur dispose d’une connexion Internet, directement ou via un serveur proxy.
* [Téléchargez et installez manuellement](#install-the-agent-manually) l’agent. Ceci est nécessaire lorsque l’ordinateur Linux n'a pas accès à Internet et communiquera avec Azure Monitor ou Azure Automation via la [passerelle Log Analytics](gateway.md). 

>[!IMPORTANT]
> Les méthodes d’installation décrites dans cet article sont généralement utilisées pour les machines virtuelles locales ou dans d’autres clouds. Pour d’autres options efficaces que vous pouvez utiliser pour des machines virtuelles Azure, consultez [Options d’installation](log-analytics-agent.md#installation-options).



## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

Pour obtenir la liste des distributions Linux que l’agent Log Analytics prend en charge, consultez [Vue d’ensemble des agents Azure Monitor](agents-overview.md#supported-operating-systems).

>[!NOTE]
>OpenSSL 1.1.0 est uniquement pris en charge sur les plateformes x86_x64 (64 bits), tandis qu’OpenSSL antérieur à 1.x n’est pris en charge sur aucune plateforme.

>[!NOTE]
>L'exécution de l'agent Log Analytics pour Linux n'est pas prise en charge dans les conteneurs. Si vous devez surveiller des conteneurs, utilisez la [solution Container Monitoring](../insights/containers.md) pour les hôtes Docker ou [Azure Monitor pour conteneurs](../insights/container-insights-overview.md) pour Kubernetes.

À partir des versions publiées après août 2018, nous apportons les changements suivants à notre modèle de prise en charge :  

* Seules les versions serveur sont prises en charge, pas les versions client.  
* Nous concentrons la prise en charge sur les [distributions approuvées Azure Linux](../../virtual-machines/linux/endorsed-distros.md). Notez qu’il peut y avoir un certain délai entre la sortie d’une nouvelle distribution ou version approuvée par Azure Linux, et sa prise en charge pour l’agent Log Analytics Linux.
* Toutes les versions mineures de chaque version majeure listée sont prises en charge.
* Les versions qui ont dépassé la date de fin de support de leur fabricant ne sont pas prises en charge.
* Seules les images de machine virtuelle sont prises en charge ; les conteneurs, même ceux dérivés des images des éditeurs distributeurs officiels, ne sont pas pris en charge.
* Les nouvelles versions d’AMI ne sont pas prises en charge.  
* Seules les versions qui exécutent SSL 1.x par défaut sont prises en charge.

>[!NOTE]
>Si vous utilisez une distribution ou une version qui n’est pas prise en charge actuellement et qui ne s’aligne pas sur notre modèle de prise en charge, nous vous recommandons de dupliquer ce dépôt, en sachant que le support de Microsoft ne fournira aucune assistance sur les versions d’agent dupliqué.

### <a name="python-requirement"></a>Exigence relative à Python

À partir de la version de l’agent 1.13.27, l’agent Linux prend en charge à la fois Python 2 et 3. Nous vous recommandons toujours d’utiliser l’agent le plus récent. 

Si vous utilisez une version antérieure de l’agent, vous devez faire en sorte que la machine virtuelle utilise Python 2 par défaut. Si votre machine virtuelle utilise une distribution qui n’inclut pas Python 2 par défaut, vous devez l’installer. Les exemples de commandes suivants installent Python 2 sur différentes distributions.

 - Red Hat, CentOS, Oracle : `yum install -y python2`
 - Ubuntu, Debian : `apt-get install -y python2`
 - SUSE : `zypper install -y python2`

L’exécutable python2 doit avoir un alias pour *python*. Voici une méthode que vous pouvez utiliser pour définir cet alias :

1. Exécutez la commande suivante pour supprimer tous les alias existants.
 
    ```
    sudo update-alternatives --remove-all python
    ```

2. Pour créer l’alias, exécutez la commande suivante.

    ```
    sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
    ```

## <a name="supported-linux-hardening"></a>Renforcement de la sécurité Linux pris en charge
L’agent OMS prend en charge une personnalisation limitée de Linux. 

Les éléments suivants sont actuellement pris en charge : 
- FIP

Les éléments suivants sont prévus, mais ne sont pas encore pris en charge :
- CIS
- SELINUX

Les autres méthodes de renforcement de la sécurité et de personnalisation ne sont pas prises en charge ni prévues pour l’agent OMS.  

## <a name="agent-prerequisites"></a>Conditions préalables associées à l’agent

Le tableau suivant met en évidence les packages requis pour les [distributions Linux prises en charge](#supported-operating-systems) sur lesquelles l’agent est installé.

|Package requis |Description |Version minimale |
|-----------------|------------|----------------|
|Glibc |    Bibliothèque C de GNU | 2.5-12 
|Openssl    | Bibliothèques OpenSSL | 1.0.x ou 1.1.x |
|Curl | Client web cURL | 7.15.5 |
|Python | | 2.6+ ou 3.3+
|Python-ctypes | | 
|PAM | Modules d’authentification enfichable | | 

>[!NOTE]
>rsyslog ou syslog-ng est requis pour collecter les messages syslog. Le démon syslog par défaut sur la version 5 de Red Hat Enterprise Linux, CentOS et Oracle Linux (sysklog) ne prend pas en charge la collecte des événements syslog. Pour collecter les données syslog avec cette version de ces distributions, le démon rsyslog doit être installé et configuré à la place de sysklog.

## <a name="network-requirements"></a>Configuration requise pour le réseau
Pour connaître la configuration réseau requise pour l’agent Linux, consultez [Présentation de l’agent Log Analytics](log-analytics-agent.md#network-requirements).

## <a name="agent-install-package"></a>Package d’installation de l’agent

L'agent Log Analytics pour Linux comprend plusieurs packages. Le fichier de la version contient les packages suivants, disponibles en exécutant l’application shell avec le paramètre `--extract` :

**Package** | **Version** | **Description**
----------- | ----------- | --------------
omsagent | 1.13.9 | Agent Log Analytics pour Linux
omsconfig | 1.1.1 | Agent de configuration pour l'agent Log Analytics
omi | 1.6.4 | Open Management Infrastructure (OMI) -- Serveur CIM léger. *Notez qu'OMI a besoin d'un accès root pour exécuter une tâche cron nécessaire au fonctionnement du service*
scx | 1.6.4 | Fournisseurs CIM OMI pour les mesures de performances des systèmes d’exploitation
apache-cimprov | 1.0.1 | Fournisseur de surveillance des performances d’Apache HTTP Server pour OMI. Installé uniquement si Apache HTTP Server est détecté.
mysql-cimprov | 1.0.1 | Fournisseur de surveillance des performances de MySQL Server pour OMI. Installé uniquement si MySQL/MariaDB Server est détecté.
docker-cimprov | 1.0.0 | Fournisseur de Docker pour OMI. Installé uniquement si Docker est détecté.

### <a name="agent-installation-details"></a>Détails de l’installation de l’agent

Après avoir installé les packages de l’agent Log Analytics pour Linux, les modifications de configuration système suivantes sont appliquées. Ces artefacts sont supprimés lorsque le package omsagent est désinstallé.

* Un utilisateur sans privilège nommé `omsagent` est créé. Le démon est exécuté avec ces informations d’identification. 
* Un fichier sudoers *include* est créé dans `/etc/sudoers.d/omsagent`. Cela autorise `omsagent` à redémarrer les démons syslog et omsagent. Si les directives *include* sudo ne sont pas prises en charge dans la version installée de sudo, ces entrées sont inscrites dans `/etc/sudoers`.
* La configuration de syslog est modifiée pour transférer un sous-ensemble d’événements à l’agent. Pour plus d'informations, voir [Configurer la collecte de données Syslog](data-sources-syslog.md).

Sur un ordinateur Linux supervisé, l'agent apparaît comme `omsagent`. `omsconfig` est l’agent de configuration de l’agent Log Analytics pour Linux qui cherche la nouvelle configuration côté portail toutes les 5 minutes. La nouvelle configuration mise à jour est appliquée aux fichiers de configuration de l’agent, situés dans `/etc/opt/microsoft/omsagent/conf/omsagent.conf`.

## <a name="install-the-agent-using-wrapper-script"></a>Installer l’agent en utilisant le script du wrapper

Les étapes suivantes configurent l'agent pour Log Analytics dans Azure et dans Azure Government Cloud en utilisant le script du wrapper pour les ordinateurs Linux qui peuvent communiquer directement ou via un serveur proxy pour télécharger puis installer l'agent hébergé sur GitHub.  

Si votre ordinateur Linux a besoin d’un serveur proxy pour communiquer avec Log Analytics, cette configuration peut être spécifiée sur la ligne de commande en incluant `-p [protocol://][user:password@]proxyhost[:port]`. La propriété *protocol* accepte `http` ou `https`, et la propriété *proxyhost* accepte un nom de domaine complet ou l’adresse IP du serveur proxy. 

Par exemple : `https://proxy01.contoso.com:30443`

Si l'authentification est requise dans l'un ou l'autre cas, vous devez spécifier le nom d'utilisateur et le mot de passe. Par exemple : `https://user01:password@proxy01.contoso.com:30443`

1. Pour configurer l’ordinateur Linux en vue d’une connexion à un espace de travail Log Analytics, exécutez la commande suivante en fournissant l’ID d’espace de travail et la clé primaire. La commande suivante télécharge l’agent, valide sa somme de contrôle et l’installe.
    
    ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY>
    ```

    La commande suivante contient le paramètre `-p` proxy et un exemple de syntaxe quand l’authentification est requise par votre serveur proxy :

   ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -p [protocol://]<proxy user>:<proxy password>@<proxyhost>[:port] -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY>
    ```

2. Pour configurer l’ordinateur Linux en vue d’une connexion à un espace de travail Log Analytics dans le cloud Azure Government, exécutez la commande suivante en fournissant l’ID d’espace de travail et la clé primaire copiés précédemment. La commande suivante télécharge l’agent, valide sa somme de contrôle et l’installe. 

    ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY> -d opinsights.azure.us
    ``` 

    La commande suivante contient le paramètre `-p` proxy et un exemple de syntaxe quand l’authentification est requise par votre serveur proxy :

   ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -p [protocol://]<proxy user>:<proxy password>@<proxyhost>[:port] -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY> -d opinsights.azure.us
    ```
2. Redémarrez l’agent en exécutant la commande suivante : 

    ```
    sudo /opt/microsoft/omsagent/bin/service_control restart [<workspace id>]
    ``` 


## <a name="install-the-agent-manually"></a>Installation de l’agent manuellement

L’agent Log Analytics pour Linux est fourni dans un groupe de scripts shell à extraction automatique et pouvant être installé. Ce groupe contient des packages Debian et RPM pour chacun des composants de l’agent et peut être installé directement ou être extrait pour récupérer les packages individuels. Un groupe est fourni pour les architectures x64 et un autre l’est pour les architectures x86. 

> [!NOTE]
> Pour les machines virtuelles Azure, nous vous recommandons d'installer l'agent sur ces machines en utilisant l'[extension de machine virtuelle Azure Log Analytics](../../virtual-machines/extensions/oms-linux.md) pour Linux. 

1. [Téléchargez](https://github.com/microsoft/OMS-Agent-for-Linux#azure-install-guide) et transférez le bundle approprié (x64 ou x86) sur votre machine virtuelle ou ordinateur physique Linux à l’aide de scp/sftp.

2. Installez le groupe à l’aide de l’argument `--install`. Pour intégrer un espace de travail Log Analytics pendant l'installation, spécifiez les paramètres `-w <WorkspaceID>` et `-s <workspaceKey>` copiés précédemment.

    >[!NOTE]
    >Vous devez utiliser l'argument `--upgrade` si des packages dépendants tels que omi, scx, omsconfig ou leurs anciennes versions sont installés, comme ce serait le cas si l'agent System Center Operations Manager pour Linux est déjà installé. 

    ```
    sudo sh ./omsagent-*.universal.x64.sh --install -w <workspace id> -s <shared key>
    ```

3. Pour configurer l'agent Linux afin de l’installer et de le connecter à un espace de travail Log Analytics via une passerelle Log Analytics, exécutez la commande suivante en spécifiant le proxy, l'ID de l'espace de travail et les paramètres clés de l'espace de travail. Cette configuration peut être spécifiée sur la ligne de commande en incluant `-p [protocol://][user:password@]proxyhost[:port]`. La propriété *proxyhost* accepte un nom de domaine complet ou l’adresse IP du serveur de passerelle Log Analytics.  

    ```
    sudo sh ./omsagent-*.universal.x64.sh --upgrade -p https://<proxy address>:<proxy port> -w <workspace id> -s <shared key>
    ```

    Si une authentification est requise, vous devez spécifier le nom d'utilisateur et le mot de passe. Par exemple : 
    
    ```
    sudo sh ./omsagent-*.universal.x64.sh --upgrade -p https://<proxy user>:<proxy password>@<proxy address>:<proxy port> -w <workspace id> -s <shared key>
    ```

4. Pour configurer l’ordinateur Linux en vue d’une connexion à un espace de travail Log Analytics dans le cloud Azure Government, exécutez la commande suivante en fournissant l’ID d’espace de travail et la clé primaire copiés précédemment.

    ```
    sudo sh ./omsagent-*.universal.x64.sh --upgrade -w <workspace id> -s <shared key> -d opinsights.azure.us
    ```

Si vous souhaitez installer les packages d'agents et les configurer pour qu’ils génèrent ultérieurement des rapports sur un espace de travail Log Analytics spécifique, exécutez la commande suivante :

```
sudo sh ./omsagent-*.universal.x64.sh --upgrade
```

Si vous souhaitez extraire les packages de l’agent du bundle sans installer l’agent, exécutez la commande suivante :

```
sudo sh ./omsagent-*.universal.x64.sh --extract
```

## <a name="upgrade-from-a-previous-release"></a>Mettre à niveau à partir d’une version antérieure

La mise à niveau à partir d'une version antérieure, à compter de la version 1.0.0-47, est prise en charge dans chaque version. Effectuez l’installation avec le paramètre `--upgrade` pour mettre à niveau tous les composants de l’agent vers la dernière version.

## <a name="cache-information"></a>Informations sur le cache
Les données de l’agent Log Analytics pour Linux sont mises en cache sur l’ordinateur local dans *%STATE_DIR_WS%/out_oms_common*.buffer* avant leur envoi à Azure Monitor. Les données de journal personnalisées sont mises en mémoire tampon dans *% STATE_DIR_WS%/out_oms_blob*.buffer*. Le chemin d’accès peut être différent pour certains [types de données et solutions](https://github.com/microsoft/OMS-Agent-for-Linux/search?utf8=%E2%9C%93&q=+buffer_path&type=).

L’agent tente de charger toutes les 20 secondes. En cas d’échec, il met un temps de plus en plus long pour réussir : 30 secondes avant la deuxième tentative, 60 secondes avant la troisième, puis 120 secondes, et ainsi de suite jusqu’à un maximum de 16 minutes entre les nouvelles tentatives avant que la reconnexion aboutisse. L’agent effectue jusqu’à six nouvelles tentatives pour un bloc de données particulier avant de l’ignorer et de passer au suivant. Cela se poursuit jusqu’à ce que l’agent puisse à nouveau charger correctement. Cela signifie que les données peuvent être mises en mémoire tampon pendant environ 30 minutes au maximum avant d’être ignorées.

La taille de cache par défaut est de 10 Mo, mais elle peut être modifiée dans le [fichier omsagent.conf](https://github.com/microsoft/OMS-Agent-for-Linux/blob/e2239a0714ae5ab5feddcc48aa7a4c4f971417d4/installer/conf/omsagent.conf).


## <a name="next-steps"></a>Étapes suivantes

- Consultez [Gestion et maintenance de l’agent Log Analytics pour Windows et Linux](agent-manage.md) pour en savoir plus sur la manière de reconfigurer, mettre à niveau ou supprimer l’agent de la machine virtuelle.

- Consultez [Résolution des problèmes de l’agent Linux](agent-linux-troubleshoot.md) si vous rencontrez des problèmes lors de l’installation ou de gestion de l’agent.
