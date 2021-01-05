---
title: Collecter des métriques personnalisées pour une machine virtuelle Linux avec l’agent InfluxData Telegraf
description: Instructions sur le déploiement de l’agent InfluxData Telegraf sur une machine virtuelle Linux dans Azure, et sur la configuration de l’agent pour publier des métriques dans Azure Monitor.
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ancav
ms.subservice: metrics
ms.openlocfilehash: b80f27e490dd3b1890eab7740fb4650ba4280abb
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008988"
---
# <a name="collect-custom-metrics-for-a-linux-vm-with-the-influxdata-telegraf-agent"></a>Collecter des métriques personnalisées pour une machine virtuelle Linux avec l’agent InfluxData Telegraf

Azure Monitor vous permet de collecter des métriques personnalisées par le biais des données de télémétrie de votre application, d’un agent s’exécutant sur vos ressources Azure ou même d’un système de supervision d’interaction indirecte. Vous pouvez ensuite les envoyer directement à Azure Monitor. Cet article fournit des instructions sur le déploiement de l’agent [InfluxData](https://www.influxdata.com/) Telegraf sur une machine virtuelle Linux dans Azure, et sur la configuration de l’agent pour publier des métriques dans Azure Monitor. 

## <a name="influxdata-telegraf-agent"></a>Agent InfluxData Telegraf 

[Telegraf](https://docs.influxdata.com/telegraf/) est un agent piloté par le plug-in qui permet la collection de métriques provenant de plus de 150 sources différentes. En fonction des charges de travail qui sont exécutées sur votre machine virtuelle, vous pouvez configurer l’agent pour tirer parti des plug-ins d’entrée spécialisés afin de collecter des métriques. MySQL, NGINX et Apache en sont des exemples. À l’aide de plug-ins de sortie, l’agent peut ensuite écrire les données dans les destinations que vous choisissez. L’agent Telegraf est intégré directement à l’API REST des métriques personnalisées Azure Monitor. Il prend en charge un plug-in de sortie Azure Monitor. Avec ce plug-in, l’agent peut collecter des métriques concernant les charges de travail sur votre machine virtuelle Linux, et les envoyer en tant que métriques personnalisées à Azure Monitor. 

 ![Présentation de l’agent Telegraf](./media/collect-custom-metrics-linux-telegraf/telegraf-agent-overview.png)

> [!NOTE]  
> Les métriques personnalisées ne sont pas prises en charge dans toutes les régions. Les régions prises en charge sont répertoriées [ici](./metrics-custom-overview.md#supported-regions).

## <a name="send-custom-metrics"></a>Envoyer des métriques personnalisées 

Pour les besoins de ce tutoriel, nous allons déployer une machine virtuelle Linux exécutant le système d’exploitation Ubuntu 16.04 LTS. L’agent Telegraf est pris en charge sur la plupart des systèmes d’exploitation Linux. Les packages Debian et RPM, ainsi que les fichiers binaires décompressés de Linux, sont disponibles sur le [portail de téléchargement InfluxData](https://portal.influxdata.com/downloads). Consultez ce [guide d’installation Telegraf](https://docs.influxdata.com/telegraf/v1.8/introduction/installation/) pour obtenir des instructions et connaître les options d’installation supplémentaires. 

Connectez-vous au [portail Azure](https://portal.azure.com).

> [!NOTE]  
> Si vous souhaitez migrer des règles d’alerte classiques et utiliser une machine virtuelle Linux existante, assurez-vous que la machine virtuelle dispose d’une identité attribuée par le système **activée**.

Créez une machine virtuelle Linux : 

1. Sélectionnez l’option **Créer une ressource** dans le volet de navigation gauche. 
1. Recherchez **Machine virtuelle**.  
1. Sélectionnez **Ubuntu 16.04 LTS**, puis sélectionnez **Créer**. 
1. Donnez un nom à votre machine virtuelle comme **MyTelegrafVM**.  
1. Gardez la valeur **SSD** pour le type de disque. Ensuite, fournissez un **nom d’utilisateur**, comme **azureuser**. 
1. Pour le **Type d’authentification**, sélectionnez **Mot de passe**. Ensuite, entrez un mot de passe que vous utiliserez plus tard pour établir une connexion SSH avec cette machine virtuelle. 
1. Choisissez de **créer un groupe de ressources**. Ensuite, indiquez un nom comme **myResourceGroup**. Choisissez votre **emplacement**. Sélectionnez ensuite **OK**. 

    ![Créer une machine virtuelle Ubuntu](./media/collect-custom-metrics-linux-telegraf/create-vm.png)

1. Choisissez la taille de la machine virtuelle. Vous pouvez filtrer par **Type de calcul** ou par **Type de disque**, par exemple. 

    ![Taille de la machine virtuelle - Agent Telegraf](./media/collect-custom-metrics-linux-telegraf/vm-size.png)

1. Dans la page **Paramètres**, dans **Réseau** > **Groupe de sécurité réseau** > **Sélectionner des ports d’entrée publics**, sélectionnez **HTTP** et **SSH (22)** . Laissez les autres valeurs par défaut et sélectionnez **OK**. 

1. Sur la page de résumé, sélectionnez **Créer** pour démarrer le déploiement de la machine virtuelle. 

1. La machine virtuelle est épinglée au tableau de bord du Portail Azure. Une fois le déploiement terminé, le récapitulatif de la machine virtuelle s’ouvre automatiquement. 

1. Dans le volet de la machine virtuelle, accédez à l’onglet **Identité**. Vérifiez que votre machine virtuelle dispose d’une identité attribuée par le système définie sur **Activée**. 
 
    ![Vue de l’identité de la machine virtuelle Telegraf](./media/collect-custom-metrics-linux-telegraf/connect-to-VM.png)
 
## <a name="connect-to-the-vm"></a>Connexion à la machine virtuelle 

Créez une connexion SSH avec la machine virtuelle. Sélectionnez le bouton **Se connecter** sur la page de présentation de la machine virtuelle. 

![Page de vue d’ensemble de la machine virtuelle Telegraf](./media/collect-custom-metrics-linux-telegraf/connect-VM-button2.png)

Sur la page **Se connecter à la machine virtuelle**, conservez les options par défaut pour vous connecter par nom DNS sur le port 22. Dans **Se connecter à l’aide d’un compte local de machine virtuelle**, une commande de connexion s’affiche. Sélectionnez le bouton pour copier la commande. L’exemple suivant montre la commande de connexion SSH : 

```cmd
ssh azureuser@XXXX.XX.XXX 
```

Collez la commande de connexion SSH dans un interpréteur de commandes, par exemple, Azure Cloud Shell ou Bash sur Ubuntu dans Windows, ou utilisez le client SSH de votre choix pour établir la connexion. 

## <a name="install-and-configure-telegraf"></a>Installer et configurer Telegraf 

Pour installer le package Debian Telegraf sur la machine virtuelle, exécutez les commandes suivantes à partir de votre session SSH : 

```cmd
# download the package to the VM 
wget https://dl.influxdata.com/telegraf/releases/telegraf_1.8.0~rc1-1_amd64.deb 
# install the package 
sudo dpkg -i telegraf_1.8.0~rc1-1_amd64.deb
```
Le fichier de configuration de Telegraf définit les opérations de Telegraf. Par défaut, un exemple de fichier de configuration est installé au chemin suivant : **/etc/telegraf/telegraf.conf**. L’exemple de fichier de configuration répertorie tous les plug-ins d’entrée et de sortie possibles. Toutefois, nous allons créer un fichier de configuration personnalisé, que l’agent utilisera en exécutant les commandes suivantes : 

```cmd
# generate the new Telegraf config file in the current directory 
telegraf --input-filter cpu:mem --output-filter azure_monitor config > azm-telegraf.conf 

# replace the example config with the new generated config 
sudo cp azm-telegraf.conf /etc/telegraf/telegraf.conf 
```

> [!NOTE]  
> Le code précédent accepte uniquement deux plug-ins d’entrée : **cpu** et **mem**. Vous pouvez ajouter davantage de plug-ins d’entrée, selon la charge de travail exécutée sur votre ordinateur. Il peut s’agir, par exemple, d’une charge de travail Docker, MySQL ou NGINX. Pour obtenir la liste complète des plug-ins d’entrée, consultez la section **Configuration supplémentaire**. 

Enfin, pour que l’agent démarre avec la nouvelle configuration, nous forçons l’arrêt et le redémarrage de l’agent en exécutant les commandes suivantes : 

```cmd
# stop the telegraf agent on the VM 
sudo systemctl stop telegraf 
# start the telegraf agent on the VM to ensure it picks up the latest configuration 
sudo systemctl start telegraf 
```
Désormais, l’agent collecte les métriques de chacun des plug-ins d’entrée spécifiés, puis les envoie à Azure Monitor. 

## <a name="plot-your-telegraf-metrics-in-the-azure-portal"></a>Tracer vos métriques Telegraf dans le portail Azure 

1. Ouvrez le [portail Azure](https://portal.azure.com). 

1. Accédez au nouvel onglet **Surveiller** . Puis sélectionnez **Métriques**.  

     ![Surveiller - Métriques (préversion)](./media/collect-custom-metrics-linux-telegraf/metrics.png)

1. Sélectionnez votre machine virtuelle dans le sélecteur de ressources.

     ![Graphique de métrique](./media/collect-custom-metrics-linux-telegraf/metric-chart.png)

1. Sélectionnez l’espace de noms **Telegraf/CPU**, puis sélectionnez la métrique **usage_system**. Vous pouvez choisir de filtrer selon les dimensions de cette métrique, ou de les fractionner.  

     ![Sélectionner l’espace de noms et la métrique](./media/collect-custom-metrics-linux-telegraf/VM-resource-selector.png)

## <a name="additional-configuration"></a>Configuration supplémentaire 

La procédure pas à pas précédente explique comment configurer l’agent Telegraf pour collecter les métriques à partir de plusieurs plug-ins d’entrée de base. L’agent Telegraf prend en charge plus de 150 plug-ins d’entrée, dont certains prennent en charge des options de configuration supplémentaires. InfluxData a publié une [liste des plug-ins pris en charge](https://docs.influxdata.com/telegraf/v1.15/plugins/inputs/) et des instructions sur [comment les configurer](https://docs.influxdata.com/telegraf/v1.15/administration/configuration/).  

De plus, dans cette procédure pas à pas, vous avez utilisé un agent Telegraf pour envoyer des métriques concernant la machine virtuelle sur laquelle l’agent est déployé. L’agent Telegraf peut également servir de collecteur et de redirecteur de métriques pour d’autres ressources. Pour savoir comment configurer l’agent afin d’émettre des métriques pour les autres ressources Azure, consultez [Sortie de métriques personnalisée Azure Monitor pour Telegraf](https://github.com/influxdata/telegraf/blob/fb704500386214655e2adb53b6eb6b15f7a6c694/plugins/outputs/azure_monitor/README.md).  

## <a name="clean-up-resources"></a>Nettoyer les ressources 

Dès que vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées. Pour ce faire, sélectionnez le groupe de ressources de la machine virtuelle, puis sélectionner **Supprimer**. Ensuite, confirmez le nom du groupe de ressources à supprimer. 

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [métriques personnalisées](metrics-custom-overview.md).
