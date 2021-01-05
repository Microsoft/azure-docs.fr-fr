---
title: Connecter des données Syslog à Azure Sentinel | Microsoft Docs
description: Connectez une machine ou appliance prenant en charge Syslog à Azure Sentinel en utilisant un agent sur une machine Linux entre l’appliance et Sentinel. 
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
ms.date: 07/17/2020
ms.author: yelevin
ms.openlocfilehash: 7670d00a2dd25961a51d18c50c102e0f92b30975
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88566146"
---
# <a name="collect-data-from-linux-based-sources-using-syslog"></a>Collecter des données de sources Linux à l’aide de Syslog

Vous pouvez diffuser des événements en continu à partir de machines ou d’appliances Linux prenant en charge Syslog, à l’aide de l’agent Log Analytics pour Linux (anciennement appelé agent OMS). Vous pouvez effectuer cette opération pour toute machine qui vous permet d’installer l’agent Log Analytics directement sur la machine. Le démon Syslog natif de la machine collecte les événements locaux des types spécifiés et les transfère localement à l’agent, qui les diffuse en continu dans votre espace de travail Log Analytics.

> [!NOTE]
> - Si votre appliance prend en charge **CEF (Common Event format) plutôt que Syslog**, un jeu de données plus complet est collecté et les données sont analysées au niveau de la collection. Vous devez choisir cette option et suivre les instructions fournies dans [Connecter votre solution externe à l’aide de CEF](connect-common-event-format.md).
>
> - Log Analytics prend en charge la collecte de messages envoyés par les démons **rsyslog** ou **syslog-ng**, où rsyslog est le démon par défaut. Le démon Syslog par défaut sur la version 5 de Red Hat Enterprise Linux (RHEL), CentOS et Oracle Linux (**sysklog**) ne prend pas en charge la collecte des événements Syslog. Pour collecter les données syslog avec cette version de ces distributions, le démon rsyslog doit être installé et configuré à la place de sysklog.

## <a name="how-it-works"></a>Fonctionnement

**Syslog** est un protocole de journalisation des événements commun à Linux. Lorsque l’**agent Log Analytics pour Linux** est installé sur votre machine virtuelle ou appliance, la routine d’installation configure le démon Syslog local pour qu’il transfère des messages à l’agent sur le port TCP 25224. L’agent envoie ensuite le message à votre espace de travail Log Analytics via le protocole HTTPS, où il est analysé dans une entrée du journal des événements dans la table Syslog dans **Azure Sentinel > Journaux**.

Pour en savoir plus, voir [Sources de données Syslog dans Azure Monitor](../azure-monitor/platform/data-sources-syslog.md).

## <a name="configure-syslog-collection"></a>Configurer la collecte Syslog

### <a name="configure-your-linux-machine-or-appliance"></a>Configurer votre machine ou appliance Linux

1. Dans Azure Sentinel, sélectionnez **Connecteurs de données**, puis le connecteur **Syslog**.

1. Dans le panneau **Syslog**, sélectionnez **Ouvrir la page du connecteur**.

1. Installez l’agent Linux. Sous **Choisissez l’emplacement d’installation de l’agent :**
    
    **Pour une machine virtuelle Linux Azure :**
      
    1. Sélectionnez **Installer l’agent sur une machine virtuelle Linux Azure**.
    
    1. Cliquez sur le lien **Télécharger et installer l’agent pour les machines virtuelles Linux Azure >** . 
    
    1. Dans le panneau **Machines virtuelles**, sélectionnez la machine virtuelle sur laquelle installer l’agent, puis cliquez sur **Connecter**. Répétez cette étape pour chaque machine virtuelle à laquelle vous souhaitez vous connecter.
    
    **Pour toute autre machine Linux :**

    1. Sélectionnez **Installer l’agent sur une machine Linux non-Azure**.

    1. Cliquez sur le lien **Télécharger et installer l’agent pour les machines Linux non-Azure >** . 

    1. Dans le panneau **Gestion des agents**, cliquez sur l’onglet **Serveurs Linux**, puis copiez la commande pour **Télécharger et intégrer l’agent pour Linux** et exécutez-la sur votre machine Linux. 
    
   > [!NOTE]
   > Veillez à configurer les paramètres de sécurité pour ces ordinateurs conformément à la stratégie de sécurité de votre organisation. Par exemple, vous pouvez configurer les paramètres réseau de sorte qu’ils soient conformes à la stratégie de sécurité réseau de votre organisation, puis modifier les ports et les protocoles dans le démon pour les adapter aux exigences de sécurité.

### <a name="configure-the-log-analytics-agent"></a>Configurer l’agent Log Analytics

1. En bas du panneau du connecteur Syslog, cliquez sur le lien **Ouvrir la configuration des paramètres avancés de votre espace de travail >** .

1. Dans le panneau **Paramètres avancés**, sélectionnez **Données** > **Syslog**. Ajoutez ensuite les installations du connecteur à collecter.
    
    - Ajoutez les installations que votre appliance Syslog inclut dans ses en-têtes de journal. 
    
    - Si vous souhaitez utiliser la détection de connexion SSH anormale avec les données que vous collectez, ajoutez **auth** et **authpriv**. Pour plus de détails, voir la [section suivante](#configure-the-syslog-connector-for-anomalous-ssh-login-detection).

1. Après avoir ajouté toutes les installations à surveiller et ajusté les options de gravité pour chacune d’elles, activez la case à cocher **Appliquer la configuration ci-dessous à mes machines**.

1. Sélectionnez **Enregistrer**. 

1. Sur votre machine virtuelle ou appliance, assurez-vous d’envoyer les fonctionnalités que vous avez spécifiées.

1. Pour interroger les données du journal Syslog dans **Journaux**, entrez `Syslog` dans la fenêtre de requête.

1. Vous pouvez utiliser les paramètres de requête décrits dans [Utilisation de fonctions dans les requêtes de journal Azure Monitor](../azure-monitor/log-query/functions.md) pour analyser vos messages Syslog. Vous pouvez ensuite enregistrer la requête sous la forme d’une nouvelle fonction de Log Analytics et l’utiliser comme nouveau type de données.

> [!NOTE]
> **Utilisation du même ordinateur pour transférer à la fois des messages Syslog *et* des messages CEF**
>
>
> Vous pouvez utiliser votre [machine de transfert de journaux CEF](connect-cef-agent.md) existant pour collecter et transférer des journaux à partir de sources Syslog ordinaires également. Toutefois, vous devez effectuer les étapes suivantes pour éviter d’envoyer des événements dans les deux formats à Azure Sentinel, car cela entraînera une duplication des événements.
>
>    Si vous avez déjà configuré [la collecte de données à partir de vos sources CEF](connect-common-event-format.md) et que vous avez configuré l’agent Log Analytics comme indiqué ci-dessus :
>
> 1. Sur chaque machine qui envoie des journaux au format CEF, vous devez modifier le fichier config Syslog pour supprimer les fonctionnalités utilisées pour envoyer des messages CEF. De cette façon, les fonctionnalités envoyées en CEF ne sont pas aussi envoyées au format Syslog. Pour obtenir des instructions détaillées sur la procédure à suivre, consultez [Configurer Syslog sur l’agent Linux](../azure-monitor/platform/data-sources-syslog.md#configure-syslog-on-linux-agent).
>
> 1. Vous devez exécuter la commande suivante sur ces machines pour désactiver la synchronisation de l’agent avec la configuration Syslog dans Azure Sentinel. Cela permet de s’assurer que la modification de configuration que vous avez apportée à l’étape précédente n’est pas remplacée.<br>
> `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/OMS_MetaConfigHelper.py --disable'`


### <a name="configure-the-syslog-connector-for-anomalous-ssh-login-detection"></a>Configurer le connecteur Syslog pour la détection de connexion SSH anormale

> [!IMPORTANT]
> La détection de connexion SSH anormale est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Sentinel peut appliquer du Machine Learning aux données Syslog pour identifier une activité de connexion SSH (Secure Shell) anormale. Il s’agit entre autres des scénarios suivants :

- Voyage impossible : lorsque deux événements de connexion réussis se produisent à partir de deux emplacements où il est impossible de se trouver dans l’intervalle de temps des deux événements de connexion.
- Emplacement inattendu : l’emplacement à partir duquel un événement de connexion réussi s’est produit est suspect. Par exemple, l’emplacement n’a pas été observé récemment.
 
Cette détection nécessite une configuration spécifique du connecteur de données Syslog : 

1. Pour l’étape 5 de la procédure précédente, assurez- vous que **auth** et **authpriv** sont sélectionnés en tant qu’installations à surveiller. Conservez les paramètres par défaut pour les options de gravité, afin qu’elles soient toutes sélectionnées. Par exemple :
    
    > [!div class="mx-imgBorder"]
    > ![Installations requises pour la détection de connexion SSH anormale](./media/connect-syslog/facilities-ssh-detection.png)

2. Laissez suffisamment de temps pour la collecte des informations Syslog. Ensuite, accédez à **Azure Sentinel - Logs**, puis copiez et collez la requête suivante :
    
    ```console
    Syslog |  where Facility in ("authpriv","auth")| extend c = extract( "Accepted\\s(publickey|password|keyboard-interactive/pam)\\sfor ([^\\s]+)",1,SyslogMessage)| where isnotempty(c) | count 
    ```
    
    Modifiez l’**Intervalle de temps** si nécessaire, puis sélectionnez **Exécuter**.
    
    Si le résultat obtenu est égal à zéro, vérifiez la configuration du connecteur et que les ordinateurs analysés ont une activité de connexion réussie pour la période que vous avez spécifiée pour votre requête.
    
    Si le résultat est supérieur à zéro, vos données Syslog sont adaptées à la détection de connexion SSH anormale. Vous activez cette détection à partir de **Analytique** >  **Modèles de règle** > **Détection de connexion SSH anormale (préversion)** .

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter les appliances locales Syslog à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.

