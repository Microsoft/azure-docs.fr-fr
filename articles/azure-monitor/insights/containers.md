---
title: Solution Container Monitoring dans Azure Monitor | Microsoft Docs
description: La solution Container Monitoring d’Azure Monitor vous permet d’afficher et de gérer vos hôtes de conteneur Docker et Windows dans un emplacement unique.
ms.subservice: logs
ms.topic: conceptual
author: mgoedtel
ms.author: magoedte
ms.date: 07/06/2020
ms.openlocfilehash: a02ea022bedd92e9deaa0730cc1be051a9d20c88
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93145682"
---
# <a name="container-monitoring-solution-in-azure-monitor"></a>Solution Container Monitoring dans Azure Monitor

![Symbole Containers](./media/containers/containers-symbol.png)

Cet article explique comment configurer et utiliser la solution Container Monitoring d’Azure Monitor, qui vous permet d’afficher et de gérer vos hôtes de conteneur Docker et Windows dans un emplacement unique. Docker est un système de virtualisation logicielle utilisé pour créer des conteneurs qui automatisent le déploiement de logiciels dans leur infrastructure informatique.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

La solution montre les conteneurs qui sont actuellement exécutés, l’image conteneur qu’ils exécutent et où ils s’exécutent. Vous pouvez afficher des informations d’audit détaillées montrant les commandes utilisées avec les conteneurs. Vous pouvez résoudre des problèmes de conteneurs en consultant des journaux d’activité centralisés et en y effectuant des recherches sans devoir afficher à distance les hôtes Docker ou Windows. Vous pouvez rechercher des conteneurs bruyants et consommant des ressources excessives sur un ordinateur hôte. Et vous pouvez consulter des informations centralisées sur le processeur, la mémoire, le stockage ainsi que l’utilisation et les performances du réseau. Sur les ordinateurs exécutant Windows, vous pouvez centraliser et comparer les journaux d’activité des conteneurs Windows Server, Hyper-V et Docker. La solution prend en charge les orchestrateurs de conteneur suivants :

- Docker Swarm
- DC/OS
- Kubernetes
- Service Fabric
- Red Hat OpenShift

Si vous avez des conteneurs déployés dans [Azure Service Fabric](../../service-fabric/service-fabric-overview.md), nous vous recommandons d’activer la [solution Service Fabric](../../service-fabric/service-fabric-diagnostics-oms-setup.md) et cette solution pour inclure la surveillance des événements de cluster. Avant d’activer la solution Service Fabric, consultez [Utilisation de la solution Service Fabric](../../service-fabric/service-fabric-diagnostics-event-analysis-oms.md) pour comprendre ce qu’elle fournit et comment l’utiliser.

Si vous souhaitez superviser les performances de vos charges de travail déployées dans des environnements Kubernetes hébergés sur Azure Kubernetes Service (AKS), consultez l’article [Surveillance d’Azure Kubernetes Service](./container-insights-overview.md). La solution de surveillance de conteneur ne prend pas en charge la surveillance de cette plateforme.  

Le diagramme suivant montre les relations entre différents hôtes de conteneur et agents avec Azure Monitor.

![Schéma des conteneurs](./media/containers/containers-diagram.png)

## <a name="system-requirements-and-supported-platforms"></a>Configuration requise et plateformes prises en charge

Avant de commencer, prenez connaissance des informations suivantes pour vérifier que les conditions préalables sont remplies.

### <a name="container-monitoring-solution-support-for-docker-orchestrator-and-os-platform"></a>Prise en charge de solution de supervision de conteneur pour la plateforme Docker Orchestrator et celle du système d’exploitation

Le tableau suivant présente l’orchestration de Docker et la prise en charge de la supervision du système d’exploitation pour ce qui est des journaux, des performances et de l’inventaire de conteneur avec Azure Monitor.   

|Orchestration Docker | ACS | Linux | Windows | Conteneur<br>Inventaire | Image<br>Inventaire | Nœud<br>Inventaire | Conteneur<br>Performances | Conteneur<br>Événement | Événement<br>Journal | Conteneur<br>Journal |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| Kubernetes | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; |
| Mésosphère<br>DC/OS | &#8226; | &#8226; | | &#8226; | &#8226; | &#8226; | &#8226;| &#8226; | &#8226; | &#8226; |
| Docker<br>Swarm | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | | &#8226; |
| Service<br>Structure | | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; |
| Red Hat Open<br>Shift | | &#8226; | | &#8226; | &#8226;| &#8226; | &#8226; | &#8226; | | &#8226; |
| Windows Server<br>(autonome) | | | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | | &#8226; |
| Serveur Linux<br>(autonome) | | &#8226; | | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | | &#8226; |

### <a name="docker-versions-supported-on-linux"></a>Versions de Docker prises en charge sur Linux

- Docker 1.11 à 1.13
- Docker CE et EE v17.06

### <a name="x64-linux-distributions-supported-as-container-hosts"></a>Distributions Linux x64 suivantes prises en charge en tant qu’hôtes de conteneur

- Ubuntu 14.04 LTS et 16.04 LTS
- CoreOS(stable)
- Amazon Linux 2016.09.0
- openSUSE 13.2
- openSUSE LEAP 42.2
- CentOS 7.2 et 7.3
- SLES 12
- RHEL 7.2 et 7.3
- Red Hat OpenShift Container Platform (OCP) 3.4 et 3.5
- ACS Mesosphere DC/OS 1.7.3 à 1.8.8
- ACS Kubernetes 1.4.5 à 1.6
    - Les événements Kubernetes, l’inventaire Kubernetes et les processus de conteneur sont uniquement pris en charge par l’agent Log Analytics pour Linux 1.4.1-45 et les versions ultérieures.
- ACS Docker Swarm

[!INCLUDE [log-analytics-agent-note.md](../../../includes/log-analytics-agent-note.md)] 

### <a name="supported-windows-operating-system"></a>Système d’exploitation Windows pris en charge

- Windows Server 2016
- Édition Anniversaire Windows 10 (Professionnel ou Entreprise)

### <a name="docker-versions-supported-on-windows"></a>Versions de docker prises en charge sur Windows

- Docker 1.12 et 1.13
- Docker 17.03.0 et versions ultérieures

## <a name="installing-and-configuring-the-solution"></a>Installation et configuration de la solution

Utilisez les informations suivantes pour installer et configurer la solution.

1. Ajoutez la solution Container Monitoring à votre espace de travail Log Analytics à partir de la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.ContainersOMS?tab=Overview) ou en procédant de la manière décrite dans [Ajouter des solutions de supervision à partir de la Galerie Solutions](./solutions.md).

2. Installez et utilisez Docker avec un agent Log Analytics. Pour configurer votre agent, vous pouvez utiliser les méthodes suivantes, selon votre système d’exploitation et votre orchestrateur Docker.
   - Pour les hôtes autonomes :
     - Sur les systèmes d’exploitation Linux pris en charge, installez et exécutez Docker, puis installez et configurez [l’agent Log Analytics pour Linux](../learn/quick-collect-linux-computer.md).  
     - Sur CoreOS, vous ne pouvez pas exécuter l’agent Log Analytics pour Linux. À la place, vous exécutez une version en conteneur de l’agent Log Analytics pour Linux. Si vous utilisez des conteneurs dans Azure Government Cloud, consultez les sections relatives aux hôtes de conteneurs Linux, y compris CoreOS ou aux hôtes de conteneurs Linux Azure Government, y compris CoreOS.
     - Sur Windows Server 2016 et Windows 10, installez le moteur et le client Docker, puis connectez un agent afin de collecter les données et les envoyer à Azure Monitor. Consultez [Installer et configurer des hôtes de conteneur Windows](#install-and-configure-windows-container-hosts) si vous avez un environnement Windows.
   - Pour l’orchestration Docker à plusieurs hôtes :
     - Si vous disposez d’un environnement Red Hat OpenShift, consultez la section Configurer un agent Log Analytics pour Red Hat OpenShift.
     - Si vous avez un cluster Kubernetes utilisant Azure Container Service :
       - Consultez la section [Configurer un agent Log Analytics Linux pour Kubernetes](#configure-a-log-analytics-linux-agent-for-kubernetes).
       - Consultez la section [Configurer un agent Log Analytics Windows pour Kubernetes](#configure-a-log-analytics-windows-agent-for-kubernetes).
       - Consultez la section Utiliser Helm pour déployer l’agent Log Analytics sur Linux Kubernetes.
     - Si vous disposez d’un cluster DC/OS Azure Container Service, consultez l’article [Superviser un cluster DC/OS Azure Container Service avec Azure Monitor](/previous-versions/azure/container-service/dcos-swarm/container-service-monitoring-oms).
     - Si vous utilisez un environnement en mode Docker Swarm, apprenez en davantage en consultant la section Configurer un agent Log Analytics pour Docker Swarm.
     - Si vous avez un cluster Service Fabric, découvrez-en plus dans [Superviser des conteneurs avec Azure Monitor](../../service-fabric/service-fabric-diagnostics-oms-containers.md).

Examinez l’article relatif au [moteur Docker sur Windows](/virtualization/windowscontainers/manage-docker/configure-docker-daemon) pour en savoir plus sur l’installation et la configuration de vos moteurs Docker sur les ordinateurs exécutant Windows.

> [!IMPORTANT]
> Docker doit être en cours d’exécution **avant** l’installation de [l’agent Log Analytics pour Linux](../learn/quick-collect-linux-computer.md) sur vos hôtes de conteneur. Si vous avez installé l’agent avant d’installer Docker, vous devez réinstaller l’agent Log Analytics pour Linux. Pour plus d’informations sur Docker, voir le [site web Docker](https://www.docker.com).

### <a name="install-and-configure-linux-container-hosts"></a>Installer et configurer des hôtes de conteneur Linux

Après avoir installé Docker, utilisez les paramètres suivants pour votre hôte de conteneur afin de configurer l’agent en vue d’une utilisation avec Docker. Tout d’abord, vous avez besoin de l’ID et de la clé de votre espace de travail Log Analytics, qui se trouvent dans le portail Azure. Dans votre espace de travail, cliquez sur **Démarrage rapide** > **Ordinateurs** pour afficher votre **ID d’espace de travail** et votre **Clé primaire**.  Copiez-collez ces deux valeurs dans votre éditeur favori.

**Pour tous les hôtes de conteneur Linux, à l’exception de CoreOS :**

- Pour plus d’informations sur l’installation de l’agent Log Analytics pour Linux, consultez l’article [Présentation de l’agent Log Analytics](../platform/log-analytics-agent.md).

**Pour tous les hôtes de conteneur Linux, avec CoreOS :**

Démarrez le conteneur pour lequel activer la surveillance. Modifiez et utilisez l’exemple suivant :

```
sudo docker run --privileged -d -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/containers:/var/lib/docker/containers -e WSID="your workspace id" -e KEY="your key" -h=`hostname` -p 127.0.0.1:25225:25225 --name="omsagent" --restart=always microsoft/oms
```

**Pour tous les hôtes de conteneur Linux Azure Government, y compris CoreOS :**

Démarrez le conteneur pour lequel activer la surveillance. Modifiez et utilisez l’exemple suivant :

```
sudo docker run --privileged -d -v /var/run/docker.sock:/var/run/docker.sock -v /var/log:/var/log -v /var/lib/docker/containers:/var/lib/docker/containers -e WSID="your workspace id" -e KEY="your key" -e DOMAIN="opinsights.azure.us" -p 127.0.0.1:25225:25225 -p 127.0.0.1:25224:25224/udp --name="omsagent" -h=`hostname` --restart=always microsoft/oms
```

**Passage de l’utilisation d’un agent Linux installé à un agent exécuté dans un conteneur**

Si vous utilisiez précédemment l’agent installé directement et souhaitez désormais utiliser un agent qui s’exécute dans un conteneur, vous devez commencer par supprimer l’agent Log Analytics pour Linux. Pour comprendre comment désinstaller l’agent, consultez l’article [Uninstalling the Log Analytics agent for Linux](../learn/quick-collect-linux-computer.md) (Désinstallation de l’agent Log Analytics pour Linux).  

#### <a name="configure-a-log-analytics-agent-for-docker-swarm"></a>Configurer un agent Log Analytics pour Docker Swarm

Vous pouvez exécuter l’agent Log Analytics en tant que service global sur Docker Swarm. Utilisez les informations ci-après pour créer un service d’agent Log Analytics. Vous devez fournir l’ID et la clé primaire de l’espace de travail Log Analytics.

- Exécutez la commande suivante sur le nœud principal.

    ```
    sudo docker service create  --name omsagent --mode global  --mount type=bind,source=/var/run/docker.sock,destination=/var/run/docker.sock --mount type=bind,source=/var/lib/docker/containers,destination=/var/lib/docker/containers -e WSID="<WORKSPACE ID>" -e KEY="<PRIMARY KEY>" -p 25225:25225 -p 25224:25224/udp  --restart-condition=on-failure microsoft/oms
    ```

##### <a name="secure-secrets-for-docker-swarm"></a>Sécuriser les secrets pour Docker Swarm

Pour Docker Swarm, une fois que vous avez créé le secret de l’ID de l’espace de travail et de la clé primaire, utilisez les informations suivantes pour créer vos informations confidentielles.

1. Exécutez la commande suivante sur le nœud principal.

    ```
    echo "WSID" | docker secret create WSID -
    echo "KEY" | docker secret create KEY -
    ```

2. Vérifiez que les secrets ont été créés correctement.

    ```
    keiko@swarmm-master-13957614-0:/run# sudo docker secret ls
    ```

    ```
    ID                          NAME                CREATED             UPDATED
    j2fj153zxy91j8zbcitnjxjiv   WSID                43 minutes ago      43 minutes ago
    l9rh3n987g9c45zffuxdxetd9   KEY                 38 minutes ago      38 minutes ago
    ```

3. Exécutez la commande ci-après pour monter les secrets sur l’agent Log Analytics en conteneur.

    ```
    sudo docker service create  --name omsagent --mode global  --mount type=bind,source=/var/run/docker.sock,destination=/var/run/docker.sock --mount type=bind,source=/var/lib/docker/containers,destination=/var/lib/docker/containers --secret source=WSID,target=WSID --secret source=KEY,target=KEY  -p 25225:25225 -p 25224:25224/udp --restart-condition=on-failure microsoft/oms
    ```

#### <a name="configure-a-log-analytics-agent-for-red-hat-openshift"></a>Configurer un agent Log Analytics pour Red Hat OpenShift

Il existe trois façons d’ajouter l’agent Log Analytics pour Red Hat OpenShift dans le but de démarrer la collecte des données de supervision des conteneurs.

* [Installer l’agent Log Analytics pour Linux](../learn/quick-collect-linux-computer.md) directement sur chaque nœud OpenShift  
* [Activer l’extension de machine virtuelle Log Analytics](../learn/quick-collect-azurevm.md) sur chaque nœud OpenShift résidant dans Azure  
* Installer l’agent Log Analytics en tant que daemon-set OpenShift  

Dans cette section, nous allons aborder les étapes nécessaires à l’installation de l’agent Log Analytics comme un daemon-set OpenShift.  

1. Connectez-vous au nœud principal OpenShift, puis copiez le fichier yaml [ocp-omsagent.yaml](https://github.com/Microsoft/OMS-docker/blob/master/OpenShift/ocp-omsagent.yaml) de GitHub vers votre nœud principal. Ensuite, remplacez la valeur par l’ID et la clé primaire de votre espace de travail Log Analytics.
2. Exécutez les commandes suivantes pour créer un projet pour Azure Monitor et configurer le compte d’utilisateur.

    ```
    oc adm new-project omslogging --node-selector='zone=default'
    oc project omslogging  
    oc create serviceaccount omsagent  
    oc adm policy add-cluster-role-to-user cluster-reader   system:serviceaccount:omslogging:omsagent  
    oc adm policy add-scc-to-user privileged system:serviceaccount:omslogging:omsagent  
    ```

3. Pour déployer le daemon-set, exécutez la commande suivante :

    `oc create -f ocp-omsagent.yaml`

4. Pour vérifier qu’il est configuré et fonctionne correctement, tapez la commande suivante :

    `oc describe daemonset omsagent`  

    La sortie doit ressembler à ceci :

    ```
    [ocpadmin@khm-0 ~]$ oc describe ds oms  
    Name:           oms  
    Image(s):       microsoft/oms  
    Selector:       name=omsagent  
    Node-Selector:  zone=default  
    Labels:         agentVersion=1.4.0-12  
                    dockerProviderVersion=10.0.0-25  
                    name=omsagent  
    Desired Number of Nodes Scheduled: 3  
    Current Number of Nodes Scheduled: 3  
    Number of Nodes Misscheduled: 0  
    Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed  
    No events.  
    ```

Si vous souhaitez utiliser des secrets pour sécuriser l’ID et la clé primaire de votre espace de travail Log Analytics lorsque vous utilisez le fichier yaml daemon-set de l’agent Log Analytics, procédez comme suit.

1. Connectez-vous au nœud principal OpenShift, puis copiez le fichier yaml [ocp-ds-omsagent.yaml](https://github.com/Microsoft/OMS-docker/blob/master/OpenShift/ocp-ds-omsagent.yaml) et le script de génération de secrets [ocp-secretgen.sh](https://github.com/Microsoft/OMS-docker/blob/master/OpenShift/ocp-secretgen.sh) à partir de GitHub.  Ce script va générer le fichier yaml de secrets pour l’ID et la clé primaire de l’espace de travail Log Analytics afin de sécuriser vos informations secrètes.  
2. Exécutez les commandes suivantes pour créer un projet pour Azure Monitor et configurer le compte d’utilisateur. Le script de génération de secrets vous demande de fournir l’ID `<WSID>` et la clé primaire `<KEY>` de l’espace de travail Log Analytics afin de créer le fichier ocp-secret.yaml.  

    ```
    oc adm new-project omslogging --node-selector='zone=default'  
    oc project omslogging  
    oc create serviceaccount omsagent  
    oc adm policy add-cluster-role-to-user cluster-reader   system:serviceaccount:omslogging:omsagent  
    oc adm policy add-scc-to-user privileged system:serviceaccount:omslogging:omsagent  
    ```

3. Déployez le fichier de secret en exécutant la commande suivante :

    `oc create -f ocp-secret.yaml`

4. Vérifiez le déploiement en exécutant la commande suivante :

    `oc describe secret omsagent-secret`  

    La sortie doit ressembler à ceci :  

    ```
    [ocpadmin@khocp-master-0 ~]$ oc describe secret omsagent-secret  
    Name:           omsagent-secret  
    Namespace:      omslogging  
    Labels:         <none>  
    Annotations:    <none>  

    Type:   Opaque  

    Data  
    ====  
    KEY:    89 bytes  
    WSID:   37 bytes  
    ```

5. Déployez le fichier yaml daemon-set de l’agent Log Analytics en exécutant la commande suivante :

    `oc create -f ocp-ds-omsagent.yaml`  

6. Vérifiez le déploiement en exécutant la commande suivante :

    `oc describe ds oms`

    La sortie doit ressembler à ceci :

    ```
    [ocpadmin@khocp-master-0 ~]$ oc describe ds oms  
    Name:           oms  
    Image(s):       microsoft/oms  
    Selector:       name=omsagent  
    Node-Selector:  zone=default  
    Labels:         agentVersion=1.4.0-12  
                    dockerProviderVersion=10.0.0-25  
                    name=omsagent  
    Desired Number of Nodes Scheduled: 3  
    Current Number of Nodes Scheduled: 3  
    Number of Nodes Misscheduled: 0  
    Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed  
    No events.  
    ```

#### <a name="configure-a-log-analytics-linux-agent-for-kubernetes"></a>Configurer un agent Log Analytics Linux pour Kubernetes

Dans le cas de Kubernetes, vous utilisez un script pour générer le fichier yaml de secrets pour votre ID d’espace de travail et votre clé primaire afin d’installer l’agent Log Analytics pour Linux. La page [Log Analytics Docker Kubernetes GitHub](https://github.com/Microsoft/OMS-docker/tree/master/Kubernetes) (GitHub Kubernetes Log Analytics Docker) comporte des fichiers que vous pouvez utiliser avec ou sans vos informations de secret.

- Le DaemonSet par défaut de l’agent Log Analytics pour Linux ne comprend pas d’informations de secret (omsagent.yaml).
- Le fichier yaml DaemonSet de l’agent Log Analytics pour Linux utilise les informations de secret (omsagent-ds-secrets.yaml) avec des scripts de génération de secrets pour générer le fichier yaml de secrets (omsagentsecret.yaml).

Vous pouvez choisir de créer le DaemonSet de l’agent OMS avec ou sans secrets.

**Fichier yaml DaemonSet par défaut de l’agent OMS sans secrets**

- Pour le fichier yaml DaemonSet par défaut de l’agent Log Analytics, remplacez `<WSID>` et `<KEY>` par votre ID d’espace de travail et votre clé. Copiez le fichier sur votre nœud principal et exécutez la commande suivante :

    ```
    sudo kubectl create -f omsagent.yaml
    ```

**Fichier yaml DaemonSet par défaut de l’agent OMS avec secrets**

1. Pour utiliser le DaemonSet de l’agent Log Analytics à l’aide des informations de secret, commencez par créer les secrets.
    1. Copiez le fichier de modèle de secret et le script, et assurez-vous qu’ils se trouvent dans le même répertoire.
        - Script de génération de secrets - secret-gen.sh
        - Modèle de secret - secret-template.yaml
    2. Exécutez le script, comme l’exemple suivant. Le script demande l’ID et la clé primaire de l’espace de travail Log Analytics, et une fois que vous les entrez, le script crée un fichier .yaml de secrets que vous pouvez exécuter.   

        ```
        #> sudo bash ./secret-gen.sh
        ```

    3. Créez le pod de secrets en exécutant la commande suivante :
        ```
        sudo kubectl create -f omsagentsecret.yaml
        ```

    4. Pour vérifier, exécutez la commande suivante :

        ```
        keiko@ubuntu16-13db:~# sudo kubectl get secrets
        ```

        La sortie doit ressembler à :

        ```
        NAME                  TYPE                                  DATA      AGE
        default-token-gvl91   kubernetes.io/service-account-token   3         50d
        omsagent-secret       Opaque                                2         1d
        ```

        ```
        keiko@ubuntu16-13db:~# sudo kubectl describe secrets omsagent-secret
        ```

        La sortie doit ressembler à :

        ```
        Name:           omsagent-secret
        Namespace:      default
        Labels:         <none>
        Annotations:    <none>

        Type:   Opaque

        Data
        ====
        WSID:   36 bytes
        KEY:    88 bytes
        ```

    5. Créer votre DaemonsSet de l’agent OMS en exécutant ```sudo kubectl create -f omsagent-ds-secrets.yaml```

2. Vérifiez que le DaemonSet de l’agent Log Analytics s’exécute, comme ci-après :

    ```
    keiko@ubuntu16-13db:~# sudo kubectl get ds omsagent
    ```

    ```
    NAME       DESIRED   CURRENT   NODE-SELECTOR   AGE
    omsagent   3         3         <none>          1h
    ```

Dans le cas de Kubernetes, utilisez un script afin de générer le fichier yaml de secrets pour l’ID d’espace de travail et la clé primaire pour l’agent Log Analytics pour Linux. Utilisez les informations de l’exemple suivant avec le [fichier yaml de l’agent OMS](https://github.com/Microsoft/OMS-docker/blob/master/Kubernetes/omsagent.yaml) pour sécuriser vos informations secrètes.

```
keiko@ubuntu16-13db:~# sudo kubectl describe secrets omsagent-secret
Name:           omsagent-secret
Namespace:      default
Labels:         <none>
Annotations:    <none>

Type:   Opaque

Data
====
WSID:   36 bytes
KEY:    88 bytes
```

#### <a name="configure-a-log-analytics-windows-agent-for-kubernetes"></a>Configurer un agent Log Analytics Windows pour Kubernetes

Dans le cas de Windows Kubernetes, vous utilisez un script pour générer le fichier yaml de secrets pour votre ID d’espace de travail et votre clé primaire afin d’installer l’agent Log Analytics. La page [Log Analytics Docker Kubernetes GitHub](https://github.com/Microsoft/OMS-docker/tree/master/Kubernetes/windows) (GitHub Kubernetes Log Analytics Docker) comporte des fichiers que vous pouvez utiliser avec vos informations de secret.  Vous devez installer l’agent Log Analytics séparément pour les nœuds principaux et d’agent.  

1. Pour utiliser le DaemonSet de l’agent Log Analytics à l’aide des informations de secret sur le nœud principal, commencez par vous connecter et par créer les secrets.
    1. Copiez le fichier de modèle de secret et le script, et assurez-vous qu’ils se trouvent dans le même répertoire.
        - Script de génération de secrets - secret-gen.sh
        - Modèle de secret - secret-template.yaml

    2. Exécutez le script, comme l’exemple suivant. Le script demande l’ID et la clé primaire de l’espace de travail Log Analytics, et une fois que vous les entrez, le script crée un fichier .yaml de secrets que vous pouvez exécuter.

        ```
        #> sudo bash ./secret-gen.sh
        ```
    3. Créer votre DaemonsSet de l’agent OMS en exécutant ```kubectl create -f omsagentsecret.yaml```
    4. Pour vérifier, exécutez la commande suivante :

        ```
        root@ubuntu16-13db:~# kubectl get secrets
        ```

        La sortie doit ressembler à :

        ```
        NAME                  TYPE                                  DATA      AGE
        default-token-gvl91   kubernetes.io/service-account-token   3         50d
        omsagent-secret       Opaque                                2         1d
        root@ubuntu16-13db:~# kubectl describe secrets omsagent-secret
        Name:           omsagent-secret
        Namespace:      default
        Labels:         <none>
        Annotations:    <none>

        Type:   Opaque

        Data
        ====
        WSID:   36 bytes
        KEY:    88 bytes
        ```

    5. Créer votre DaemonsSet de l’agent OMS en exécutant ```kubectl create -f ws-omsagent-de-secrets.yaml```

2. Vérifiez que le DaemonSet de l’agent Log Analytics s’exécute, comme ci-après :

    ```
    root@ubuntu16-13db:~# kubectl get deployment omsagent
    NAME       DESIRED   CURRENT   NODE-SELECTOR   AGE
    omsagent   1         1         <none>          1h
    ```

3. Pour installer l’agent sur le nœud Worker qui exécute Windows, suivez les étapes décrites dans la section [Installer et configurer des hôtes de conteneur Windows](#install-and-configure-windows-container-hosts).

#### <a name="use-helm-to-deploy-log-analytics-agent-on-linux-kubernetes"></a>Utiliser Helm pour déployer l’agent Log Analytics sur Linux Kubernetes

Pour déployer l’agent Log Analytics sur votre environnement Linux Kubernetes en utilisant Helm, procédez comme suit.

1. Créer votre DaemonsSet de l’agent OMS en exécutant ```helm install --name omsagent --set omsagent.secret.wsid=<WSID>,omsagent.secret.key=<KEY> stable/msoms```
2. Les résultats ressemblent à ce qui suit :

    ```
    NAME:   omsagent
    LAST DEPLOYED: Tue Sep 19 20:37:46 2017
    NAMESPACE: default
    STATUS: DEPLOYED

    RESOURCES:
    ==> v1/Secret
    NAME            TYPE    DATA  AGE
    omsagent-msoms  Opaque  3     3s

    ==> v1beta1/DaemonSet
    NAME            DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR  AGE
    omsagent-msoms  3        3        3      3           3          <none>         3s
    ```

3. Vous pouvez vérifier l’état de l’agent OMS en exécutant : ```helm status "omsagent"```. Le résultat doit ressembler à ce qui suit :

    ```
    keiko@k8s-master-3814F33-0:~$ helm status omsagent
    LAST DEPLOYED: Tue Sep 19 20:37:46 2017
    NAMESPACE: default
    STATUS: DEPLOYED
 
    RESOURCES:
    ==> v1/Secret
    NAME            TYPE    DATA  AGE
    omsagent-msoms  Opaque  3     17m
 
    ==> v1beta1/DaemonSet
    NAME            DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR  AGE
    omsagent-msoms  3        3        3      3           3          <none>         17m
    ```
   
    Pour plus d’informations, consultez [Azure Log Analytics Container Monitoring Solution](https://aka.ms/omscontainerhelm) (Solution de surveillance de conteneur Azure Log Analytics).

### <a name="install-and-configure-windows-container-hosts"></a>Installer et configurer des hôtes de conteneur Windows

Utilisez les informations de cette section pour installer et configurer des hôtes de conteneur Windows.

#### <a name="preparation-before-installing-windows-agents"></a>Préparation préalable à l’installation des agents Windows

Avant d’installer les agents sur les ordinateurs exécutant Windows, vous devez configurer le service Docker. La configuration permet à l’agent Windows ou à l’extension de machine virtuelle Azure Monitor d’utiliser le socket Docker TCP afin d’autoriser les agents à accéder à distance au démon Docker et de capturer des données pour la supervision.

##### <a name="to-configure-the-docker-service"></a>Pour configurer le service Docker  

Exécuter les commandes PowerShell suivantes pour activer le canal TCP et le canal nommé pour Windows Server :

```
Stop-Service docker
dockerd --unregister-service
dockerd --register-service -H npipe:// -H 0.0.0.0:2375  
Start-Service docker
```

Pour plus d’informations sur la configuration du démon Docker utilisée avec les conteneurs de Windows, reportez-vous à [Moteur Docker sur Windows](/virtualization/windowscontainers/manage-docker/configure-docker-daemon).

#### <a name="install-windows-agents"></a>Installer les agents Windows

Pour activer la surveillance des conteneurs Windows et Hyper-V, installez Microsoft Monitoring Agent (MMA) sur les ordinateurs Windows qui sont des hôtes de conteneurs. Pour les ordinateurs exécutant Windows dans votre environnement local, consultez [Connecter des ordinateurs Windows à Azure Monitor](../platform/agent-windows.md). Connectez les machines virtuelles exécutées dans Azure à Azure Monitor à l’aide de l’[extension de machine virtuelle](../learn/quick-collect-azurevm.md).

Vous pouvez surveiller les conteneurs Windows en cours d’exécution sur Service Fabric. Toutefois, seules les [machines virtuelles qui s’exécutent dans Azure](../learn/quick-collect-azurevm.md) et les [ordinateurs exécutant Windows dans votre environnement local](../platform/agent-windows.md) sont actuellement pris en charge pour Service Fabric.

Vous pouvez vérifier que la solution de supervision de conteneurs est correctement configurée pour Windows. Pour vérifier que le pack d’administration a été correctement téléchargé, recherchez *ContainerManagement.xxx*. Les fichiers doivent se trouver dans le dossier C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs.

## <a name="solution-components"></a>Composants de la solution

Dans le Portail Azure, accédez à la *Galerie Solutions* et ajoutez la **solution de supervision de conteneurs**. Si vous utilisez des agents Windows, le pack d’administration suivant est installé sur chaque ordinateur où se trouve un agent lorsque vous ajoutez cette solution. Le pack d’administration ne nécessite aucune opération de configuration ou de maintenance.

- *ContainerManagement.xxx* installé dans le dossier C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs

## <a name="container-data-collection-details"></a>Détails sur la collecte de données des conteneurs

La solution de supervision de conteneurs collecte diverses métriques de performances et données de journaux à partir des hôtes de conteneur et des conteneurs utilisant les agents que vous avez activés.

Les données sont collectées toutes les trois minutes par les types d’agents suivants.

- [Agent Log Analytics pour Linux](../learn/quick-collect-linux-computer.md)
- [Agent Windows](../platform/agent-windows.md)
- [Extension de machine virtuelle Log Analytics](../learn/quick-collect-azurevm.md)

### <a name="container-records"></a>Enregistrements de conteneur

Le tableau suivant présente des exemples d’enregistrements collectés par la solution de supervision de conteneurs, ainsi que les types de données qui s’affichent dans les résultats de recherche des journaux.

| Type de données | Type de données dans Recherche de journaux | Champs |
| --- | --- | --- |
| Performances des hôtes et des conteneurs | `Perf` | Computer, ObjectName, CounterName &#40;%Processor Time, Disk Reads MB, Disk Writes MB, Memory Usage MB, Network Receive Bytes, Network Send Bytes, Processor Usage sec, Network&#41;, CounterValue,TimeGenerated, CounterPath, SourceSystem |
| Inventaire de conteneur | `ContainerInventory` | TimeGenerated, Computer, container name, ContainerHostname, Image, ImageTag, ContainerState, ExitCode, EnvironmentVar, Command, CreatedTime, StartedTime, FinishedTime, SourceSystem, ContainerID, ImageID |
| Inventaire d’image de conteneur | `ContainerImageInventory` | TimeGenerated, Computer, Image, ImageTag, ImageSize, VirtualSize, Running, Paused, Stopped, Failed, SourceSystem, ImageID, TotalContainer |
| Journal de conteneur | `ContainerLog` | TimeGenerated, Computer, image ID, container name, LogEntrySource, LogEntry, SourceSystem, ContainerID |
| Journal de service de conteneur | `ContainerServiceLog`  | TimeGenerated, Computer, TimeOfCommand, Image, Command, SourceSystem, ContainerID |
| Inventaire du nœud de conteneur | `ContainerNodeInventory_CL`| TimeGenerated, Computer, ClassName_s, DockerVersion_s, OperatingSystem_s, Volume_s, Network_s, NodeRole_s, OrchestratorType_s, InstanceID_g, SourceSystem|
| Inventaire de Kubernetes | `KubePodInventory_CL` | TimeGenerated, Computer, PodLabel_deployment_s, PodLabel_deploymentconfig_s, PodLabel_docker_registry_s, Name_s, Namespace_s, PodStatus_s, PodIp_s, PodUid_g, PodCreationTimeStamp_t, SourceSystem |
| Processus de conteneur | `ContainerProcess_CL` | TimeGenerated, Computer, Pod_s, Namespace_s, ClassName_s, InstanceID_s, Uid_s, PID_s, PPID_s, C_s, STIME_s, Tty_s, TIME_s, Cmd_s, Id_s, Name_s, SourceSystem |
| Événements Kubernetes | `KubeEvents_CL` | TimeGenerated, Computer, Name_s, ObjectKind_s, Namespace_s, Reason_s, Type_s, SourceComponent_s, SourceSystem, Message |

Les étiquettes ajoutées aux types de données *PodLabel* sont vos propres étiquettes personnalisées. Les étiquettes PodLabel ajoutées contenues dans le tableau sont des exemples. Par conséquent, `PodLabel_deployment_s`, `PodLabel_deploymentconfig_s`, `PodLabel_docker_registry_s` seront différentes dans le jeu de données de votre environnement et ressembleront à `PodLabel_yourlabel_s`.

## <a name="monitor-containers"></a>Analyser les conteneurs
Une fois la solution activée dans le portail Azure, vous voyez la vignette **Containers** qui contient des informations récapitulatives sur vos hôtes de conteneur et les conteneurs s’exécutant dans les hôtes.

![Vignette Conteneurs](./media/containers/containers-title.png)

La vignette affiche une vue d’ensemble du nombre de conteneurs présents dans l’environnement, et indique s’ils sont en échec, en cours d’exécution ou arrêtés.

### <a name="using-the-containers-dashboard"></a>Utilisation du tableau de bord Conteneurs

Cliquez sur la vignette **Conteneurs**. À partir de là, vous voyez les vues organisées par :

- **Événements de conteneur**  : affiche l’état du conteneur et les ordinateurs dont le conteneur a échoué.
- **Journaux d’activité de conteneur**  : affiche un graphique montrant les fichiers journaux de conteneur générés jusqu’à maintenant, ainsi que la liste des ordinateurs contenant le plus grand nombre de fichiers journaux.
- **Événements Kubernetes**  : affiche un graphique montrant les événements Kubernetes générés jusqu’à maintenant, ainsi que la liste des raisons pour lesquelles les pods ont généré ces événements. *Ce jeu de données est utilisé uniquement dans les environnements Linux.*
- **Inventaire des espaces de noms Kubernetes**  : indique le nombre d’espaces de noms et de pods, et montre leur hiérarchie. *Ce jeu de données est utilisé uniquement dans les environnements Linux.*
- **Inventaire des nœuds de conteneur**  : affiche le nombre de types d’orchestration utilisés sur les nœuds/hôtes de conteneur. Les nœuds/hôtes d’ordinateur sont également répertoriés avec le nombre de conteneurs. *Ce jeu de données est utilisé uniquement dans les environnements Linux.*
- **Inventaire des images de conteneur**  : affiche le nombre total des images de conteneur utilisées, ainsi que le nombre de types d’images. Le nombre d’images est également répertorié avec la balise d’image.
- **État des conteneurs**  : affiche le nombre total de nœuds de conteneur/ordinateurs hôtes comprenant des conteneurs en cours d’exécution. Les ordinateurs sont également répertoriés avec le nombre d’ordinateurs hôtes en cours d’exécution.
- **Processus de conteneur**  : affiche un graphique en courbes représentant les processus de conteneur exécutés jusqu’à maintenant. Les conteneurs peuvent également être répertoriés en exécutant la commande /process dans les conteneurs. *Ce jeu de données est utilisé uniquement dans les environnements Linux.*
- **Performances de l’UC du conteneur**  : affiche un graphique en courbes représentant l’utilisation moyenne de l’UC jusqu’à maintenant pour les nœuds/hôtes d’ordinateur. Répertorie également les nœuds/hôtes d’ordinateur en fonction de l’utilisation moyenne de l’UC.
- **Performances de la mémoire du conteneur**  : affiche un graphique en courbes représentant l’utilisation de la mémoire jusqu’à maintenant. Répertorie également l’utilisation de la mémoire par l’ordinateur en fonction du nom de l’instance.
- **Performances de l’ordinateur**  : affiche des graphiques en courbes représentant le pourcentage de performance de l’UC, le pourcentage d’utilisation de la mémoire et les mégaoctets d’espace disque libre dans le temps. Vous pouvez pointer sur une ligne du graphique pour afficher plus de détails.

Chaque zone du tableau de bord est une représentation visuelle d’une recherche exécutée sur des données collectées.

![Capture d’écran qui montre un tableau de bord permettant de consulter les données collectées. ](./media/containers/containers-dash01.png)

![Tableau de bord Conteneurs](./media/containers/containers-dash02.png)

Dans la zone **État du conteneur** , cliquez sur la zone supérieure, comme illustré ci-dessous.

![État des conteneurs](./media/containers/containers-status.png)

Log Analytics s’ouvre et affiche des informations sur l’état de vos conteneurs.

![Log Analytics pour conteneurs](./media/containers/containers-log-search.png)

À partir d’ici, vous pouvez modifier la requête de recherche de façon à trouver les informations spécifiques qui vous intéressent. Pour plus d’informations sur les requêtes de journal, consultez [Requêtes de journal dans Azure Monitor](../log-query/log-query-overview.md).

## <a name="troubleshoot-by-finding-a-failed-container"></a>Résoudre des problèmes en recherchant un conteneur en échec

Log Analytics marque un conteneur comme étant en **Échec** s’il a été fermé avec un code de sortie autre que zéro. Vous pouvez consulter un aperçu des erreurs et des échecs de l’environnement dans la zone **Conteneurs défectueux**.

### <a name="to-find-failed-containers"></a>Pour rechercher les conteneurs défectueux

1. Cliquez sur la zone **État du conteneur**.  
   ![État des conteneurs](./media/containers/containers-status.png)
2. Log Analytics s’ouvre et affiche l’état de vos conteneurs, comme ci-dessous.  
   ![État des conteneurs](./media/containers/containers-log-search.png)
3. Développez la ligne Failed, puis cliquez sur + pour ajouter ses critères à la requête. Commentez la ligne Summarize dans la requête.
   ![Capture d’écran montrant la ligne qui doit être commentée.](./media/containers/containers-state-failed-select.png)  
1. Exécutez la requête, puis développez une ligne dans les résultats pour afficher l’ID de l’image.  
   ![Capture d’écran montrant comment afficher l’ID de l’image.](./media/containers/containers-state-failed.png)  
1. Tapez ce qui suit dans la requête de journal. `ContainerImageInventory | where ImageID == <ImageID>` pour afficher des détails sur l’image, tels que sa taille ou le nombre d’images arrêtées et en échec.  
   ![Conteneurs défectueux](./media/containers/containers-failed04.png)

## <a name="query-logs-for-container-data"></a>Journaux de requête pour les données de conteneur

Lorsque vous résolvez une erreur spécifique, il peut être utile de voir l’emplacement où elle se produit dans votre environnement. Les types de journaux suivants vous aident à créer des requêtes qui retournent les informations souhaitées.

- **ContainerImageInventory**  : utilisez ce type de journal lorsque vous recherchez des informations organisées par image, et de consulter des informations sur les images, telles que leurs ID ou tailles.
- **ContainerInventory**  : utilisez ce type de journal lorsque vous recherchez des informations sur l’emplacement des conteneurs, leurs noms et les images qu’ils exécutent.
- **ContainerLog**  : utilisez ce type de journal lorsque vous recherchez des informations et entrées spécifiques du journal des erreurs.
- **ContainerNodeInventory_CL** Utilisez ce type lorsque vous souhaitez obtenir les informations sur le nœud ou l’hôte où résident les conteneurs. Il fournit la version Docker, le type d’orchestration, ainsi que des informations relatives au stockage et au réseau.
- **ContainerProcess_CL** Ce type permet de visualiser rapidement le processus actuellement exécuté dans le conteneur.
- **ContainerServiceLog**  : utilisez ce type de journal lorsque vous recherchez des informations de piste d’audit pour le démon Docker, telles que les commandes de démarrage, d’arrêter, de suppression ou d’extraction.
- **KubeEvents_CL** Ce type permet d’afficher les événements Kubernetes.
- **KubePodInventory_CL** Utilisez ce type lorsque vous voulez comprendre les informations de hiérarchie du cluster.

### <a name="to-query-logs-for-container-data"></a>Pour interroger des journaux afin de rechercher des données de conteneur

* Choisissez une image qui a échoué récemment et recherchez-la dans les journaux d’activité des erreurs. Commencez par rechercher un nom de conteneur exécutant cette image avec une recherche **ContainerInventory**. Par exemple, recherchez `ContainerInventory | where Image == "ubuntu" and ContainerState == "Failed"`  
    ![Recherche de conteneurs Ubuntu](./media/containers/search-ubuntu.png)

  Développez des lignes dans les résultats pour afficher les détails de ce conteneur.

## <a name="example-log-queries"></a>Exemples de requêtes de journal

Il est souvent utile de créer des requêtes en commençant par un exemple ou deux, puis en les modifiant afin de les adapter à votre environnement. Pour commencer, vous pouvez utiliser la zone **EXEMPLES DE REQUÊTES** tout à droite de la page de la solution pour créer des requêtes plus avancées.

![Requêtes de conteneurs](./media/containers/containers-queries.png)

## <a name="saving-log-queries"></a>Enregistrement de requêtes de journal

L’enregistrement de requêtes est une fonctionnalité standard dans Azure Monitor. En les enregistrant, vous aurez aisément accès à celles que vous avez trouvées utiles pour une utilisation ultérieure.

Après avoir créé une requête qui vous semble utile, enregistrez-la en cliquant sur **Favorites** en haut de la page Recherche de journal. Vous pourrez ainsi y accéder facilement par la suite à partir de la page **Mon tableau de bord**.

## <a name="next-steps"></a>Étapes suivantes

[Interroger des journaux](../log-query/log-query-overview.md) pour consulter des enregistrements de données de conteneur détaillés.

