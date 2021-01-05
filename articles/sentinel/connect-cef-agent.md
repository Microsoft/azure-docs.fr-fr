---
title: Déployer le redirecteur de journal pour connecter des données CEF à Azure Sentinel | Microsoft Docs
description: Découvrez comment déployer l’agent pour connecter des données CEF à Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/01/2020
ms.author: yelevin
ms.openlocfilehash: ead878daaab977c77b3ab36f42ccfe4d01d7bc03
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96548628"
---
# <a name="step-1-deploy-the-log-forwarder"></a>Étape 1 : Déployer le redirecteur de journal


Au cours de cette étape, vous allez désigner et configurer la machine Linux qui transférera les journaux de votre solution de sécurité vers votre espace de travail Azure Sentinel. Cet ordinateur peut être une machine physique ou virtuelle dans votre environnement local, une machine virtuelle Azure ou une machine virtuelle dans un autre cloud. À l’aide du lien fourni, vous allez exécuter un script sur l’ordinateur désigné qui effectue les tâches suivantes :

- Installe l’agent Log Analytics pour Linux (également appelé agent OMS) et le configure aux fins suivantes :
    - l’écoute des messages CEF à partir du démon Syslog Linux intégré sur le port TCP 25226
    - l’envoi sécurisé des messages via TLS à votre espace de travail Azure Sentinel, où ils sont analysés et enrichis

- Configure le démon Syslog Linux intégré (rsyslog.d/syslog-ng) aux fins suivantes :
    - l’écoute des messages Syslog de vos solutions de sécurité sur le port TCP 514
    - le transfert des seuls messages qu’il identifie comme CEF à l’agent Log Analytics sur localhost à l’aide du port TCP 25226
 
## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’autorisations élevées (sudo) sur votre machine Linux désignée.

- **Python 2.7** ou **3** doit être installé sur la machine Linux.<br>Utilisez la commande `python -version` pour vérifier.

- La machine Linux ne doit être connectée à aucun espace de travail Azure avant l’installation de l’agent Log Analytics.

- Vous pouvez avoir besoin de l’ID de l’espace de travail et de la clé primaire de l’espace de travail à un moment donné dans ce processus. Vous pouvez les trouver dans la ressource d’espace de travail, sous **Gestion des agents**.

## <a name="run-the-deployment-script"></a>Exécuter le script de déploiement
 
1. Dans le menu de navigation d’Azure Sentinel, cliquez sur **Connecteurs de données**. Dans la liste des connecteurs, cliquez sur la mosaïque **Common Event Format (CEF)** , puis sur le bouton **Ouvrir la page du connecteur** dans le coin inférieur droit. 

1. Sous **1.2 Installer le collecteur CEF sur la machine Linux**, copiez le lien fourni sous **Exécuter le script suivant pour installer et appliquer le collecteur CEF** ou à partir du texte ci-dessous (appliquer l’ID de l’espace de travail et la clé primaire au lieu des espaces de travail) :

    ```bash
    sudo wget -O https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/cef_installer.py&&sudo python cef_installer.py [WorkspaceID] [Workspace Primary Key]`
    ```

1. Pendant l’exécution du script, vérifiez que vous ne recevez pas de messages d’erreur ou d’avertissement.
    - Vous pouvez recevoir un message vous indiquant d’exécuter une commande pour corriger un problème avec le mappage du champ *Ordinateur*. Pour plus d’informations, consultez l’[explication dans le script de déploiement](#mapping-command).

1. Passez à [l’ÉTAPE 2 : Configurez votre solution de sécurité pour transférer des messages CEF](connect-cef-solution-config.md) .


> [!NOTE]
> **Utilisation du même ordinateur pour transférer à la fois des messages Syslog *et* des messages CEF**
>
> Si vous envisagez d’utiliser cette machine de transfert de journaux pour transférer des [messages Syslog](connect-syslog.md) ainsi que des messages CEF, afin d’éviter la duplication des événements dans les tables Syslog et CommonSecurityLog :
>
> 1. Sur chaque machine source qui envoie des journaux au redirecteur au format CEF, vous devez modifier le fichier config Syslog pour supprimer les fonctionnalités utilisées pour envoyer des messages CEF. De cette façon, les fonctionnalités envoyées en CEF ne sont pas aussi envoyées au format Syslog. Pour obtenir des instructions détaillées sur la procédure à suivre, consultez [Configurer Syslog sur l’agent Linux](../azure-monitor/platform/data-sources-syslog.md#configure-syslog-on-linux-agent).
>
> 1. Vous devez exécuter la commande suivante sur ces machines pour désactiver la synchronisation de l’agent avec la configuration Syslog dans Azure Sentinel. Cela permet de s’assurer que la modification de configuration que vous avez apportée à l’étape précédente n’est pas remplacée.<br>
> `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/OMS_MetaConfigHelper.py --disable'`

> [!NOTE]
> **Modification de la source du champ TimeGenerated**
>
> - Par défaut, l’agent Log Analytics remplit le champ *TimeGenerated* du schéma en indiquant l’heure à laquelle il a reçu l’événement du démon Syslog. Par conséquent, l’heure à laquelle l’événement a été généré sur le système source n’est pas enregistrée dans Azure Sentinel.
>
> - Vous pouvez toutefois exécuter la commande suivante, qui permet de télécharger et d’exécuter le script `TimeGenerated.py`. Ce script configure l’agent Log Analytics de sorte qu’il remplisse le champ *TimeGenerated* en indiquant l’heure d’origine de l’événement sur son système source, et non l’heure à laquelle l’agent l’a reçu.
>
>    ```bash
>    wget -O TimeGenerated.py https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/TimeGenerated.py && python TimeGenerated.py {ws_id}
>    ```

## <a name="deployment-script-explained"></a>Explication du script de déploiement

Voici une description commande par commande des actions du script de déploiement.

Choisissez un démon Syslog pour afficher la description appropriée.

# <a name="rsyslog-daemon"></a>[démon rsyslog](#tab/rsyslog)

1. **Téléchargement et installation de l’agent Log Analytics :**

    - Télécharge le script d’installation pour l’agent Linux Log Analytics (OMS).

        ```bash
        wget -O https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/
            onboard_agent.sh
        ```

    - Installe l’agent Log Analytics/
    
        ```bash
        sh onboard_agent.sh -w [workspaceID] -s [Primary Key] -d opinsights.azure.com
        ```

1. **Paramétrage de la configuration de l’agent Log Analytics pour écouter sur le port 25226 et transférer les messages CEF vers Azure Sentinel :**

    - Télécharge la configuration à partir du dépôt GitHub de l’agent Log Analytics.

        ```bash
        wget -O /etc/opt/microsoft/omsagent/[workspaceID]/conf/omsagent.d/security_events.conf
            https://raw.githubusercontent.com/microsoft/OMS-Agent-for-Linux/master/installer/conf/
            omsagent.d/security_events.conf
        ```

1. **Configuration du démon Syslog :**

    - Ouvre le port 514 pour la communication TCP à l’aide du fichier config Syslog `/etc/rsyslog.conf`.

    - Configure le démon pour transférer les messages CEF à l’agent Log Analytics sur le port TCP 25226, en insérant un fichier config spécial `security-config-omsagent.conf` dans le répertoire Syslog `/etc/rsyslog.d/` du démon.

        Contenu du fichier `security-config-omsagent.conf` :

        ```bash
        if $rawmsg contains "CEF:" or $rawmsg contains "ASA-" then @@127.0.0.1:25226 
        ```

1. **Redémarrage du démon Syslog et de l’agent Log Analytics :**

    - Redémarre le démon rsyslog.
    
        ```bash
        service rsyslog restart
        ```

    - Redémarre l’agent Log Analytics.

        ```bash
        /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```

1. **Vérification du mappage du champ *Ordinateur* :**

    - Vérifie que le champ *Ordinateur* de la source syslog est correctement mappé dans l’agent Log Analytics en utilisant la commande suivante : 

        ```bash
        grep -i "'Host' => record\['host'\]"  /opt/microsoft/omsagent/plugin/filter_syslog_security.rb
        ```
    - <a name="mapping-command"></a>En cas de problème avec le mappage, le script génère un message d’erreur qui vous indique d’**exécuter manuellement la commande suivante** (en appliquant l’ID de l’espace de travail au lieu de l’espace réservé). La commande va garantir le mappage correct et redémarrer l’agent.
    
        ```bash
        sed -i -e "/'Severity' => tags\[tags.size - 1\]/ a \ \t 'Host' => record['host']" -e "s/'Severity' => tags\[tags.size - 1\]/&,/" /opt/microsoft/omsagent/plugin/filter_syslog_security.rb && sudo /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```

# <a name="syslog-ng-daemon"></a>[démon syslog-ng](#tab/syslogng)

1. **Téléchargement et installation de l’agent Log Analytics :**

    - Télécharge le script d’installation pour l’agent Linux Log Analytics (OMS).

        ```bash
        wget -O https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/
            onboard_agent.sh
        ```

    - Installe l’agent Log Analytics/
    
        ```bash
        sh onboard_agent.sh -w [workspaceID] -s [Primary Key] -d opinsights.azure.com
        ```

1. **Paramétrage de la configuration de l’agent Log Analytics pour écouter sur le port 25226 et transférer les messages CEF vers Azure Sentinel :**

    - Télécharge la configuration à partir du dépôt GitHub de l’agent Log Analytics.

        ```bash
        wget -O /etc/opt/microsoft/omsagent/[workspaceID]/conf/omsagent.d/security_events.conf
            https://raw.githubusercontent.com/microsoft/OMS-Agent-for-Linux/master/installer/conf/
            omsagent.d/security_events.conf
        ```

1. **Configuration du démon Syslog :**

    - Ouvre le port 514 pour la communication TCP à l’aide du fichier config Syslog `/etc/syslog-ng/syslog-ng.conf`.

    - Configure le démon pour transférer les messages CEF à l’agent Log Analytics sur le port TCP 25226, en insérant un fichier config spécial `security-config-omsagent.conf` dans le répertoire Syslog `/etc/syslog-ng/conf.d/` du démon.

        Contenu du fichier `security-config-omsagent.conf` :

        ```bash
        filter f_oms_filter {match(\"CEF\|ASA\" ) ;};
        destination oms_destination {tcp(\"127.0.0.1\" port("25226"));};
        log {source(s_src);filter(f_oms_filter);destination(oms_destination);};
        ```

1. **Redémarrage du démon Syslog et de l’agent Log Analytics :**

    - Redémarre le démon syslog-ng.
    
        ```bash
        service syslog-ng restart
        ```

    - Redémarre l’agent Log Analytics.

        ```bash
        /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```

1. **Vérification du mappage du champ *Ordinateur* :**

    - Vérifie que le champ *Ordinateur* de la source syslog est correctement mappé dans l’agent Log Analytics en utilisant la commande suivante : 

        ```bash
        grep -i "'Host' => record\['host'\]"  /opt/microsoft/omsagent/plugin/filter_syslog_security.rb
        ```
    - <a name="mapping-command"></a>En cas de problème avec le mappage, le script génère un message d’erreur qui vous indique d’**exécuter manuellement la commande suivante** (en appliquant l’ID de l’espace de travail au lieu de l’espace réservé). La commande va garantir le mappage correct et redémarrer l’agent.
    
        ```bash
        sed -i -e "/'Severity' => tags\[tags.size - 1\]/ a \ \t 'Host' => record['host']" -e "s/'Severity' => tags\[tags.size - 1\]/&,/" /opt/microsoft/omsagent/plugin/filter_syslog_security.rb && sudo /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```
---

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à déployer l’agent Log Analytics pour connecter des appliances CEF à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](./tutorial-detect-threats-built-in.md).
