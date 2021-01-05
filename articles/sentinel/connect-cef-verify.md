---
title: Valider la connectivité à Azure Sentinel | Microsoft Docs
description: Validez la connectivité de votre solution de sécurité pour vous assurer que les messages CEF sont transférés vers Azure Sentinel.
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
ms.openlocfilehash: 974418a1b3c1e7fe93b2f6839c16169e5bd5abc5
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94696997"
---
# <a name="step-3-validate-connectivity"></a>ÉTAPE 3 : Valider la connectivité

Une fois que vous avez déployé votre redirecteur de journal (à l’étape 1) et configuré votre solution de sécurité pour lui envoyer des messages CEF (à l’étape 2), suivez ces instructions pour vérifier la connectivité entre votre solution de sécurité et Azure Sentinel. 

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’autorisations élevées (sudo) sur votre machine de transfert de journaux.

- **Python 2.7** ou **3** doit être installé sur votre machine de transfert de journaux.<br>
Utilisez la commande `python –version` pour vérifier.

- Vous pouvez avoir besoin de l’ID et de la clé primaire de l’espace de travail à un moment donné dans ce processus. Vous pouvez les trouver dans la ressource d’espace de travail, sous **Gestion des agents**.

## <a name="how-to-validate-connectivity"></a>Comment valider la connectivité

1. Dans le menu de navigation d’Azure Sentinel, ouvrez **Journaux**. Exécutez une requête à l’aide du schéma **CommonSecurityLog** pour voir si vous recevez les journaux de votre solution de sécurité.<br>
Notez qu’environ 20 minutes peuvent être nécessaires avant que vos journaux commencent à apparaître dans **Log Analytics**. 

1. Si vous ne voyez aucun résultat lors de la requête, vérifiez que des événements sont générés à partir de votre solution de sécurité, ou essayez d’en générer certains, et vérifiez qu’ils sont transférés vers la machine de redirection de syslog que vous avez désignée. 

1. Exécutez le script suivant sur le redirecteur de journal (en appliquant l’ID d’espace de travail à la place de l’espace réservé) pour vérifier la connectivité entre votre solution de sécurité, le redirecteur de journal et Azure Sentinel. Ce script vérifie que le démon écoute les ports appropriés, que le transfert est correctement configuré, et s’assure que rien ne bloque la communication entre le démon et l’agent Log Analytics. Il envoie également des messages fictifs « TestCommonEventFormat » pour vérifier la connectivité de bout en bout. <br>

    ```bash
    sudo wget -O https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/cef_troubleshoot.py&&sudo python cef_troubleshoot.py [WorkspaceID]` 
    ```

   - Vous verrez peut-être s’afficher un message vous demandant d’exécuter une commande pour corriger un problème de **mappage du champ *Ordinateur***. Pour plus d’informations, consultez l’[explication dans le script de validation](#mapping-command).

    - Vous verrez peut-être s’afficher un message vous demandant d’exécuter une commande pour corriger un problème d’**analyse de journaux de pare-feu Cisco ASA**. Pour plus d’informations, consultez l’[explication dans le script de validation](#parsing-command).

## <a name="validation-script-explained"></a>Explication du script de validation

Le script de validation effectue les vérifications suivantes :

# <a name="rsyslog-daemon"></a>[rsyslog daemon](#tab/rsyslog)

1. Vérifie que le fichier<br>
    `/etc/opt/microsoft/omsagent/[WorkspaceID]/conf/omsagent.d/security_events.conf`<br>
    existe et est valide.

1. Vérifie que le fichier contient le texte suivant :

    ```bash
    <source>
        type syslog
        port 25226
        bind 127.0.0.1
        protocol_type tcp
        tag oms.security
        format /(?<time>(?:\w+ +){2,3}(?:\d+:){2}\d+|\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}.[\w\-\:\+]{3,12}):?\s*(?:(?<host>[^: ]+) ?:?)?\s*(?<ident>.*CEF.+?(?=0\|)|%ASA[0-9\-]{8,10})\s*:?(?<message>0\|.*|.*)/
        <parse>
            message_format auto
        </parse>
    </source>

    <filter oms.security.**>
        type filter_syslog_security
    </filter>
    ```

1. Vérifie que l’analyse des événements de pare-feu Cisco ASA est configurée comme prévu à l’aide de la commande suivante : 

    ```bash
    grep -i "return ident if ident.include?('%ASA')" /opt/microsoft/omsagent/plugin/security_lib.rb
    ```

    - <a name="parsing-command"></a>En cas de problème avec l’analyse, le script génère un message d’erreur qui vous indique d’**exécuter manuellement la commande suivante** (en appliquant l’ID de l’espace de travail au lieu de l’espace réservé). La commande va garantir l’analyse correcte et redémarrer l’agent.
    
        ```bash
        # Cisco ASA parsing fix
        sed -i "s|return '%ASA' if ident.include?('%ASA')|return ident if ident.include?('%ASA')|g" /opt/microsoft/omsagent/plugin/security_lib.rb && sudo /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```

1. Vérifie que le champ *Ordinateur* de la source syslog est correctement mappé dans l’agent Log Analytics en utilisant la commande suivante : 

    ```bash
    grep -i "'Host' => record\['host'\]"  /opt/microsoft/omsagent/plugin/filter_syslog_security.rb
    ```

    - <a name="mapping-command"></a>En cas de problème avec le mappage, le script génère un message d’erreur qui vous indique d’**exécuter manuellement la commande suivante** (en appliquant l’ID d’espace de travail à la place de l’espace réservé). La commande va garantir le mappage correct et redémarrer l’agent.

        ```bash
        # Computer field mapping fix
        sed -i -e "/'Severity' => tags\[tags.size - 1\]/ a \ \t 'Host' => record['host']" -e "s/'Severity' => tags\[tags.size - 1\]/&,/" /opt/microsoft/omsagent/plugin/filter_syslog_security.rb && sudo /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```

1. Vérifie s’il existe des améliorations de sécurité sur la machine qui peuvent bloquer le trafic réseau (par exemple un pare-feu hôte).

1. Vérifie que le démon syslog (rsyslog) est correctement configuré pour envoyer les messages (qu’il identifie en tant que CEF) à l’agent Log Analytics sur le port TCP 25226 :

    - Fichier de configuration : `/etc/rsyslog.d/security-config-omsagent.conf`

        ```bash
        if $rawmsg contains "CEF:" or $rawmsg contains "ASA-" then @@127.0.0.1:25226 
        ```

1. Redémarre le démon syslog et l’agent Log Analytics :

    ```bash
    service rsyslog restart

    /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
    ```

1. Vérifie que les connexions nécessaires sont établies : TCP 514 pour la réception de données, TCP 25226 pour la communication interne entre le démon syslog et l’agent Log Analytics :

    ```bash
    netstat -an | grep 514

    netstat -an | grep 25226
    ```

1. Vérifie que le démon syslog reçoit des données sur le port 514 et que l’agent reçoit les données sur le port 25226 :

    ```bash
    sudo tcpdump -A -ni any port 514 -vv

    sudo tcpdump -A -ni any port 25226 -vv
    ```

1. Envoie des données fictives au port 514 sur localhost. Ces données doivent être observables dans l’espace de travail Azure Sentinel en exécutant la requête suivante :

    ```kusto
    CommonSecurityLog
    | where DeviceProduct == "MOCK"
    ```

# <a name="syslog-ng-daemon"></a>[syslog-ng daemon](#tab/syslogng)

1. Vérifie que le fichier<br>
    `/etc/opt/microsoft/omsagent/[WorkspaceID]/conf/omsagent.d/security_events.conf`<br>
    existe et est valide.

1. Vérifie que le fichier contient le texte suivant :

    ```bash
    <source>
        type syslog
        port 25226
        bind 127.0.0.1
        protocol_type tcp
        tag oms.security
        format /(?<time>(?:\w+ +){2,3}(?:\d+:){2}\d+|\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}.[\w\-\:\+]{3,12}):?\s*(?:(?<host>[^: ]+) ?:?)?\s*(?<ident>.*CEF.+?(?=0\|)|%ASA[0-9\-]{8,10})\s*:?(?<message>0\|.*|.*)/
        <parse>
            message_format auto
        </parse>
    </source>

    <filter oms.security.**>
        type filter_syslog_security
    </filter>
    ```

1. Vérifie que l’analyse des événements de pare-feu Cisco ASA est configurée comme prévu à l’aide de la commande suivante : 

    ```bash
    grep -i "return ident if ident.include?('%ASA')" /opt/microsoft/omsagent/plugin/security_lib.rb
    ```

    - <a name="parsing-command"></a>En cas de problème avec l’analyse, le script génère un message d’erreur qui vous indique d’**exécuter manuellement la commande suivante** (en appliquant l’ID de l’espace de travail au lieu de l’espace réservé). La commande va garantir l’analyse correcte et redémarrer l’agent.
    
        ```bash
        # Cisco ASA parsing fix
        sed -i "s|return '%ASA' if ident.include?('%ASA')|return ident if ident.include?('%ASA')|g" /opt/microsoft/omsagent/plugin/security_lib.rb && sudo /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```

1. Vérifie que le champ *Ordinateur* de la source syslog est correctement mappé dans l’agent Log Analytics en utilisant la commande suivante : 

    ```bash
    grep -i "'Host' => record\['host'\]"  /opt/microsoft/omsagent/plugin/filter_syslog_security.rb
    ```

    - <a name="mapping-command"></a>En cas de problème avec le mappage, le script génère un message d’erreur qui vous indique d’**exécuter manuellement la commande suivante** (en appliquant l’ID d’espace de travail à la place de l’espace réservé). La commande va garantir le mappage correct et redémarrer l’agent.

        ```bash
        # Computer field mapping fix
        sed -i -e "/'Severity' => tags\[tags.size - 1\]/ a \ \t 'Host' => record['host']" -e "s/'Severity' => tags\[tags.size - 1\]/&,/" /opt/microsoft/omsagent/plugin/filter_syslog_security.rb && sudo /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```

1. Vérifie s’il existe des améliorations de sécurité sur la machine qui peuvent bloquer le trafic réseau (par exemple un pare-feu hôte).

1. Vérifie que le démon syslog (syslog-ng) est correctement configuré pour envoyer les messages qu’il identifie en tant que CEF (à l’aide d’une expression régulière) à l’agent Log Analytics sur le port TCP 25226 :

    - Fichier de configuration : `/etc/syslog-ng/conf.d/security-config-omsagent.conf`

        ```bash
        filter f_oms_filter {match(\"CEF\|ASA\" ) ;};
        destination oms_destination {tcp(\"127.0.0.1\" port("25226"));};
        log {source(s_src);filter(f_oms_filter);destination(oms_destination);};
        ```

1. Redémarre le démon syslog et l’agent Log Analytics :

    ```bash
    service syslog-ng restart

    /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
    ```

1. Vérifie que les connexions nécessaires sont établies : TCP 514 pour la réception de données, TCP 25226 pour la communication interne entre le démon syslog et l’agent Log Analytics :

    ```bash
    netstat -an | grep 514

    netstat -an | grep 25226
    ```

1. Vérifie que le démon syslog reçoit des données sur le port 514 et que l’agent reçoit les données sur le port 25226 :

    ```bash
    sudo tcpdump -A -ni any port 514 -vv

    sudo tcpdump -A -ni any port 25226 -vv
    ```

1. Envoie des données fictives au port 514 sur localhost. Ces données doivent être observables dans l’espace de travail Azure Sentinel en exécutant la requête suivante :

    ```kusto
    CommonSecurityLog
    | where DeviceProduct == "MOCK"
    ```
---

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter les appliances CEF à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](./tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.
