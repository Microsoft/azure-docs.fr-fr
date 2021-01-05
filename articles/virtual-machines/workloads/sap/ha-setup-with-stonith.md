---
title: Configuration de la haute disponibilité avec STONITH pour SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Établir une haute disponibilité pour SAP HANA sur Azure (grandes instances) dans SUSE à l’aide de STONITH
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/21/2017
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b34a7665770308b45732711f5d8328eb1d0a785f
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965066"
---
# <a name="high-availability-set-up-in-suse-using-the-stonith"></a>Configuration de la haute disponibilité dans SUSE à l’aide de STONITH
Ce document fournit les instructions détaillées étape par étape pour configurer la haute disponibilité sur le système d’exploitation SUSE à l’aide de l’appareil STONITH.

**Clause d’exclusion de responsabilité :** *Ce guide provient de tests de configuration dans l’environnement des grandes instances Microsoft HANA, dont les résultats ont été satisfaisants. Comme l’équipe de gestion des services Microsoft pour les grandes instances HANA ne prend pas en charge le système d’exploitation, vous devrez peut-être contacter SUSE pour toute résolution de problème ou clarification sur la couche système d’exploitation. L’équipe de gestion des services Microsoft configure bel et bien le périphérique STONITH et prend complètement en charge pour vous aider à résoudre les problèmes liés à ce dernier.*
## <a name="overview"></a>Vue d’ensemble
Pour configurer la haute disponibilité à l’aide du clustering SUSE, les prérequis suivants doivent être respectés.
### <a name="pre-requisites"></a>Conditions préalables
- Les grandes instances HANA sont approvisionnées.
- Le système d’exploitation est inscrit.
- Les serveurs de grandes instances HANA sont connectés au serveur SMT pour obtenir des correctifs/packages.
- Les derniers correctifs sont installés sur le système d’exploitation.
- Le serveur de temps (NTP) est configuré
- Lire et comprendre la dernière version de la documentation SUSE sur la configuration de la haute disponibilité

### <a name="setup-details"></a>Détails de la configuration
Ce guide utilise la configuration suivante :
- Système d’exploitation : SLES 12 SP1 pour SAP
- HANA - Grandes instances : 2xS192 (quatre sockets, 2 To)
- Version HANA : HANA 2.0 SP1
- Noms de serveur : sapprdhdb95 (node1) et sapprdhdb96 (node2)
- Appareil STONITH : appareil STONITH iSCSI
- Configuration NTP sur l’un des nœuds de grande instance HANA

Quand vous configurez des grandes instances HANA avec HSR, vous pouvez demander à l’équipe de gestion des services Microsoft de configurer STONITH. Si vous êtes déjà un client existant qui dispose de grandes instances HANA approvisionnées et que vous avez besoin d’installer un appareil STONITH pour vos panneaux existants, vous devez fournir les informations suivantes à l’équipe de gestion des services Microsoft dans le formulaire de demande de service (SRF). Vous pouvez demander le formulaire SRF en passant par le responsable technique de compte ou votre contact Microsoft pour l’intégration d’une grande instance HANA. Les nouveaux clients peuvent demander un appareil STONITH au moment de l’approvisionnement. Les entrées sont disponibles dans le formulaire de demande d’approvisionnement.

- Nom du serveur et adresse IP du serveur (par exemple, myhanaserver1, 10.35.0.1)
- Emplacement (par exemple, USA Est)
- Nom du client (par exemple, Microsoft)
- SID - Identificateur du système HANA (par exemple, H11)

Une fois l’appareil STONITH configuré, l’équipe de gestion des services Microsoft fournit le nom de l’appareil SBD et l’adresse IP du stockage iSCSI que vous pouvez utiliser pour configurer l’installation STONITH. 

Pour configurer la haute disponibilité de bout en bout à l’aide de STONITH, vous devez suivre les étapes suivantes :

1.  Identifier l’appareil SBD
2.  Initialiser l’appareil SBD
3.  Configurer le cluster
4.  Installer l’agent de surveillance Softdog
5.  Joindre le nœud au cluster
6.  Valider le cluster
7.  Configurer les ressources sur le cluster
8.  Tester le processus de basculement

## <a name="1---identify-the-sbd-device"></a>1.   Identifier l’appareil SBD
Cette section décrit comment déterminer l’appareil SBD adapté à votre configuration une fois que l’équipe de gestion des services Microsoft a configuré l’appareil STONITH. **Cette section s’applique uniquement au client existant**. Si vous êtes nouveau client, l’équipe de gestion des services Microsoft vous fournit le nom de l’appareil SDB et vous pouvez ignorer cette section.

1.1 Modifiez */etc/iscsi/initiatorname.isci* pour 
``` 
iqn.1996-04.de.suse:01:<Tenant><Location><SID><NodeNumber> 
```

L’équipe de gestion des services Microsoft fournit cette chaîne. Modifiez le fichier sur **les deux** nœuds, toutefois, le numéro de nœud est différent sur chaque nœud.

![Capture d'écran représentant un fichier initiatorname avec les valeurs InitiatorName d'un nœud.](media/HowToHLI/HASetupWithStonith/initiatorname.png)

1.2 Modifiez */etc/iscsi/iscsid.conf* : définissez *node.session.timeo.replacement_timeout=5* et *node.startup = automatic*. Modifier le fichier sur **les deux** nœuds.

1.3 Exécutez la commande de découverte. Vous voyez quatre sessions. Exécutez-la sur les deux nœuds.

```
iscsiadm -m discovery -t st -p <IP address provided by Service Management>:3260
```

![Capture d'écran représentant une fenêtre de console avec les résultats de la commande de découverte isciadm.](media/HowToHLI/HASetupWithStonith/iSCSIadmDiscovery.png)

1.4 Exécutez la commande pour vous connecter au périphérique iSCSI. Vous voyez quatre sessions. Exécutez-la sur **les deux** nœuds.

```
iscsiadm -m node -l
```
![Capture d'écran représentant une fenêtre de console avec les résultats de la commande de nœud iscsiadm.](media/HowToHLI/HASetupWithStonith/iSCSIadmLogin.png)

1.5 Exécutez le script de relance d’analyse : *rescan-scsi-bus.sh*.  Ce script affiche les nouveaux disques créés pour vous.  Exécutez-la sur les deux nœuds. Vous devez voir un numéro d’unité logique supérieur à zéro (par exemple : 1, 2 etc.)

```
rescan-scsi-bus.sh
```
![Capture d'écran représentant une fenêtre de console avec les résultats du script.](media/HowToHLI/HASetupWithStonith/rescanscsibus.png)

1.6 Pour obtenir le nom de l’appareil, exécutez la commande *fdisk –l*. Exécutez-la sur les deux nœuds. Sélectionnez le périphérique avec une taille de **178 MiB**.

```
  fdisk –l
```

![Capture d'écran représentant une fenêtre de console avec les résultats de la commande fdisk.](media/HowToHLI/HASetupWithStonith/fdisk-l.png)

## <a name="2---initialize-the-sbd-device"></a>2.   Initialiser l’appareil SBD

2.1 Initialisez l’appareil SBD sur **les deux** nœuds.

```
sbd -d <SBD Device Name> create
```
![Capture d'écran représentant une fenêtre de console avec le résultat de la commande sbd create.](media/HowToHLI/HASetupWithStonith/sbdcreate.png)

2.2 Vérifiez ce qui a été écrit sur l’appareil. Faites-le sur **les deux** nœuds.

```
sbd -d <SBD Device Name> dump
```

## <a name="3---configuring-the-cluster"></a>3.   Configurer le cluster
Cette section décrit les étapes de configuration du cluster à haute disponibilité SUSE.
### <a name="31-package-installation"></a>3.1 Installation du package
3.1.1 Vérifiez que les modèles ha_sles et SAPHanaSR-doc sont installés. S’ils ne sont pas installés, installez-les. Installez-les sur **les deux** nœuds.
```
zypper in -t pattern ha_sles
zypper in SAPHanaSR SAPHanaSR-doc
```
![Capture d'écran représentant une fenêtre de console avec le résultat de la commande pattern.](media/HowToHLI/HASetupWithStonith/zypperpatternha_sles.png)
![Capture d'écran représentant une fenêtre de console avec le résultat de la commande SAPHanaSR-doc.](media/HowToHLI/HASetupWithStonith/zypperpatternSAPHANASR-doc.png)

### <a name="32-setting-up-the-cluster"></a>3.2 Configuration du cluster
3.2.1 Vous pouvez utiliser la commande *ha-cluster-init* ou l’Assistant yast2 pour configurer le cluster. Dans ce cas, l’Assistant yast2 est utilisé. Vous effectuez cette étape **uniquement sur le nœud principal**.

Suivez yast2 > Haute disponibilité > Cluster ![Capture d'écran représentant le Centre de contrôle YaST dans lequel Haute disponibilité et Cluster sont sélectionnés.](media/HowToHLI/HASetupWithStonith/yast-control-center.png)
![Capture d'écran représentant une boîte de dialogue qui contient les options Installer et Annuler.](media/HowToHLI/HASetupWithStonith/yast-hawk-install.png)

Cliquez sur **Annuler** puisque le package halk2 est déjà installé.

![Capture d'écran représentant un message relatif à votre option d'annulation.](media/HowToHLI/HASetupWithStonith/yast-hawk-continue.png)

Cliquez sur **Continuer**.

Valeur attendue = Nombre de nœuds déployés (dans ce cas 2) ![ Capture d'écran représentant Sécurité du cluster et la case à cocher Activer l'authentification de la sécurité.](media/HowToHLI/HASetupWithStonith/yast-Cluster-Security.png)
Cliquez sur **Suivant**
![Capture d'écran représentant la fenêtre Configurer le cluster ainsi que les listes Hôte de synchronisation et Fichier de synchronisation.](media/HowToHLI/HASetupWithStonith/yast-cluster-configure-csync2.png)
Ajoutez des noms de nœuds, puis cliquez sur « Ajouter les fichiers suggérés ».

Cliquez sur « Activer csync2 ».

Cliquez sur « Générer des clés prépartagées ». Le menu contextuel ci-dessous s’affiche.

![Capture d'écran représentant un message qui indique que votre clé a été générée.](media/HowToHLI/HASetupWithStonith/yast-key-file.png)

Cliquez sur **OK**

L’authentification est effectuée à l’aide des adresses IP et des clés prépartagées dans Csync2. Le fichier de clé est généré avec csync2 -k /etc/csync2/key_hagroup. Vous devez copier manuellement le fichier key_hagroup sur tous les membres du cluster après sa création. **Veillez à copier le fichier à partir du nœud 1 vers le nœud 2**.

![Capture d'écran représentant la boîte de dialogue Configurer le cluster avec les options nécessaires pour copier la clé vers tous les membres du cluster.](media/HowToHLI/HASetupWithStonith/yast-cluster-conntrackd.png)

Cliquez sur **Suivant**
![Capture d'écran représentant la fenêtre Service de cluster.](media/HowToHLI/HASetupWithStonith/yast-cluster-service.png)

Dans l’option par défaut, le démarrage était désactivé. Activez-le pour Pacemaker se lance au démarrage. Vous pouvez effectuer votre choix en fonction de vos exigences d’installation.
Cliquez sur **Suivant** pour terminer la configuration du cluster.

## <a name="4---setting-up-the-softdog-watchdog"></a>4.   Installer l’agent de surveillance Softdog
Cette section décrit la configuration de l’agent de surveillance (softdog).

4.1 Ajoutez la ligne suivante à */etc/init.d/boot.local* sur **les deux** nœuds.
```
modprobe softdog
```
![Capture d'écran représentant un fichier de démarrage dans lequel la ligne softdog a été ajoutée.](media/HowToHLI/HASetupWithStonith/modprobe-softdog.png)

4.2 Mettez à jour le fichier */etc/sysconfig/sbd* sur **les deux** nœuds, comme indiqué ci-dessous :
```
SBD_DEVICE="<SBD Device Name>"
```
![Capture d'écran représentant le fichier sbd dans lequel la valeur S B D_DEVICE a été ajoutée.](media/HowToHLI/HASetupWithStonith/sbd-device.png)

4.3 Chargez le module du noyau sur **les deux** nœuds en exécutant la commande suivante.
```
modprobe softdog
```
![Capture d'écran représentant une fenêtre de console partielle avec la commande modprobe softdog.](media/HowToHLI/HASetupWithStonith/modprobe-softdog-command.png)

4.4 Vérifiez que softdog est en cours d’exécution comme ci-dessous sur **les deux** nœuds :
```
lsmod | grep dog
```
![Capture d'écran représentant une fenêtre de console partielle avec le résultat de l'exécution de la commande l s mod.](media/HowToHLI/HASetupWithStonith/lsmod-grep-dog.png)

4.5 Démarrez l’appareil SBD sur **les deux** nœuds.
```
/usr/share/sbd/sbd.sh start
```
![Capture d'écran représentant une fenêtre de console partielle avec la commande start.](media/HowToHLI/HASetupWithStonith/sbd-sh-start.png)

4.6 Testez le démon SBD sur **les deux** nœuds. Vous voyez deux entrées une fois que vous le configurez sur **les deux** nœuds.
```
sbd -d <SBD Device Name> list
```
![Capture d'écran représentant fenêtre de console partielle dans laquelle deux entrées sont affichées.](media/HowToHLI/HASetupWithStonith/sbd-list.png)

4.7 Envoyez un message de test à l’**un** de vos nœuds.
```
sbd  -d <SBD Device Name> message <node2> <message>
```
![Capture d'écran représentant fenêtre de console partielle dans laquelle deux entrées sont affichées.](media/HowToHLI/HASetupWithStonith/sbd-list.png)

4.8 Sur le **second** nœud (node2), vous pouvez vérifier l’état du message.
```
sbd  -d <SBD Device Name> list
```
![Capture d'écran représentant une fenêtre de console partielle dans laquelle l'un des membres affiche une valeur de test pour l'autre membre.](media/HowToHLI/HASetupWithStonith/sbd-list-message.png)

4.9 Pour adopter la configuration sbd, mettez à jour le fichier */etc/sysconfig/sbd* comme suit. Modifiez le fichier sur **les deux** nœuds
```
SBD_DEVICE=" <SBD Device Name>" 
SBD_WATCHDOG="yes" 
SBD_PACEMAKER="yes" 
SBD_STARTMODE="clean" 
SBD_OPTS=""
```
4.10 Démarrez le service Pacemaker sur le **nœud principal** (node1).
```
systemctl start pacemaker
```
![Capture d'écran représentant une fenêtre de console qui affiche l'état après le démarrage du service Pacemaker.](media/HowToHLI/HASetupWithStonith/start-pacemaker.png)

Si le service Pacemaker *échoue*, reportez-vous au *scénario 5 : échec du service Pacemaker*

## <a name="5---joining-the-cluster"></a>5.   Joindre le cluster
Cette section explique comment joindre le nœud au cluster.

### <a name="51-add-the-node"></a>5.1 Ajouter le nœud
Exécutez la commande suivante sur **node2** pour permettre à node2 de joindre le cluster.
```
ha-cluster-join
```
Si vous recevez une *erreur* au cours de la jonction du cluster, reportez-vous au *scénario 6 : node2 ne parvient pas à joindre le cluster*.

## <a name="6---validating-the-cluster"></a>6.   Validation du cluster

### <a name="61-start-the-cluster-service"></a>6.1 Démarrer le service de cluster
Vérifiez et éventuellement démarrez le cluster pour la première fois sur **les deux** nœuds.
```
systemctl status pacemaker
systemctl start pacemaker
```
![Capture d'écran représentant une fenêtre de console avec l'état du service Pacemaker.](media/HowToHLI/HASetupWithStonith/systemctl-status-pacemaker.png)
### <a name="62-monitor-the-status"></a>6.2 Surveiller l’état
Exécutez la commande *crm_mon* pour vérifier que **les deux** nœuds sont en ligne. Vous pouvez l’exécuter sur **l’un ou l’autre des nœuds** du cluster.
```
crm_mon
```
![Capture d'écran représentant une fenêtre de console avec les résultats de c r m_mon.](media/HowToHLI/HASetupWithStonith/crm-mon.png)
Vous pouvez également vous connecter à hawk pour vérifier l'état du cluster *https://\<node IP>:7630*. L’utilisateur par défaut est hacluster et le mot de passe est linux. Si nécessaire, vous pouvez modifier le mot de passe à l’aide de la commande *passwd*.

## <a name="7-configure-cluster-properties-and-resources"></a>7. Configurer les propriétés et ressources du cluster 
Cette section décrit les étapes permettant de configurer les ressources du cluster.
Dans cet exemple, configurez la ressource suivante, le reste peut être configuré (si nécessaire) en se reportant au guide de la haute disponibilité SUSE. Effectuez la configuration sur l’**un des nœuds** uniquement. Effectuez-la sur le nœud principal.

- Bootstrap du cluster
- Appareil STONITH
- Adresse IP virtuelle


### <a name="71-cluster-bootstrap-and-more"></a>7.1 Bootstrap du cluster et autres
Ajoutez le bootstrap du cluster. Créez le fichier et ajoutez le texte comme indiqué ci-après :
```
sapprdhdb95:~ # vi crm-bs.txt
# enter the following to crm-bs.txt
property $id="cib-bootstrap-options" \
no-quorum-policy="ignore" \
stonith-enabled="true" \
stonith-action="reboot" \
stonith-timeout="150s"
rsc_defaults $id="rsc-options" \
resource-stickiness="1000" \
migration-threshold="5000"
op_defaults $id="op-options" \
timeout="600"
```
Ajoutez la configuration au cluster.
```
crm configure load update crm-bs.txt
```
![Capture d'écran représentant une fenêtre de console partielle dans laquelle la commande c r m est exécutée.](media/HowToHLI/HASetupWithStonith/crm-configure-crmbs.png)

### <a name="72-stonith-device"></a>7.2 Appareil STONITH
Ajoutez une ressource STONITH. Créez le fichier et ajoutez le texte comme indiqué ci-après.
```
# vi crm-sbd.txt
# enter the following to crm-sbd.txt
primitive stonith-sbd stonith:external/sbd \
params pcmk_delay_max="15"
```
Ajoutez la configuration au cluster.
```
crm configure load update crm-sbd.txt
```

### <a name="73-the-virtual-ip-address"></a>7.3 Adresse IP virtuelle
Ajoutez l’adresse IP virtuelle de la ressource. Créez le fichier et ajoutez le texte comme indiqué ci-dessous.
```
# vi crm-vip.txt
primitive rsc_ip_HA1_HDB10 ocf:heartbeat:IPaddr2 \
operations $id="rsc_ip_HA1_HDB10-operations" \
op monitor interval="10s" timeout="20s" \
params ip="10.35.0.197"
```
Ajoutez la configuration au cluster.
```
crm configure load update crm-vip.txt
```

### <a name="74-validate-the-resources"></a>7.4 Valider les ressources

Quand vous exécutez la commande *crm_mon*, vous pouvez voir les deux ressources ici.
![Capture d'écran représentant une fenêtre de console avec deux ressources.](media/HowToHLI/HASetupWithStonith/crm_mon_command.png)

De plus, vous pouvez voir l’état à l’adresse *https://\<node IP address>:7630/cib/live/state*

![Capture d'écran affichant l'état des deux ressources.](media/HowToHLI/HASetupWithStonith/hawlk-status-page.png)

## <a name="8-testing-the-failover-process"></a>8. Tester le processus de basculement
Pour tester le processus de basculement, arrêtez le service Pacemaker sur node1 et le basculement des ressources sur node2.
```
Service pacemaker stop
```
Maintenant, arrêtez le service Pacemaker sur **node2** et le basculement des ressources sur **node1**.

**Avant le basculement**  
![Capture d'écran affichant l'état des deux ressources avant le basculement.](media/HowToHLI/HASetupWithStonith/Before-failover.png)  

**Après le basculement**  
![Capture d'écran affichant l'état des deux ressources après le basculement.](media/HowToHLI/HASetupWithStonith/after-failover.png)  
![Capture d'écran représentant une fenêtre de console qui affiche l'état des ressources après le basculement.](media/HowToHLI/HASetupWithStonith/crm-mon-after-failover.png)  


## <a name="9-troubleshooting"></a>9. Dépannage
Cette section décrit les peu nombreux scénarios d’échec, que vous pouvez rencontrer pendant l’installation. Vous n’êtes pas nécessairement confronté à ces problèmes.

### <a name="scenario-1-cluster-node-not-online"></a>Scénario 1 : le nœud de cluster n’est pas en ligne
Si l’un des nœuds n’apparaît pas en ligne dans le gestionnaire du cluster, vous pouvez essayer ce qui suit pour le mettre en ligne.

Démarrez le service iSCSI.
```
service iscsid start
```

Et maintenant, vous devez être en mesure de vous connecter à ce nœud iSCSI.
```
iscsiadm -m node -l
```
Le résultat attendu ressemble au suivant.
```
sapprdhdb45:~ # iscsiadm -m node -l
Logging in to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.11,3260] (multiple)
Logging in to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.12,3260] (multiple)
Logging in to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.22,3260] (multiple)
Logging in to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.21,3260] (multiple)
Login to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.11,3260] successful.
Login to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.12,3260] successful.
Login to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.22,3260] successful.
Login to [iface: default, target: iqn.1992-08.com.netapp:hanadc11:1:t020, portal: 10.250.22.21,3260] successful.
```
### <a name="scenario-2-yast2-does-not-show-graphical-view"></a>Scénario 2 : yast2 ne montre pas la vue graphique
L’écran graphique de yast2 est utilisé pour configurer le cluster à haute disponibilité dans ce document. Si yast2 ne s’ouvre pas avec la fenêtre graphique indiquée et lance une erreur Qt, suivez les étapes ci-dessous. Si yast2 s’ouvre avec la fenêtre graphique, vous pouvez ignorer ces étapes.

**Error**

![Capture d'écran représentant une fenêtre de console partielle qui affiche un message d'erreur.](media/HowToHLI/HASetupWithStonith/yast2-qt-gui-error.png)

**Résultat attendu**

![Capture d'écran représentant le Centre de contrôle YaST dans lequel Haute disponibilité et Cluster sont en surbrillance.](media/HowToHLI/HASetupWithStonith/yast-control-center.png)

Si yast2 ne s’ouvre pas avec la vue graphique, suivez les étapes ci-après.

Installez les packages requis. Vous devez être connecté en tant qu’utilisateur « racine » et avoir configuré SMT pour télécharger/installer les packages.

Pour installer les packages, utilisez yast > Logiciels > Gestion des logiciels > Dépendances > option « Installer les packages recommandés... ». La capture d’écran suivante illustre les écrans attendus.
>[!NOTE]
>Vous devez effectuer les étapes sur les deux nœuds, afin de pouvoir accéder à la vue graphique yast2 à partir des deux nœuds.

![Capture d'écran représentant une fenêtre de console qui affiche le Centre de contrôle YaST.](media/HowToHLI/HASetupWithStonith/yast-sofwaremanagement.png)

Sous Dépendances, sélectionnez « Installer les packages recommandés » ![Capture d'écran représentant une fenêtre de console dans laquelle l'option Installer les packages recommandés est sélectionnée.](media/HowToHLI/HASetupWithStonith/yast-dependencies.png)

Passez en revue les modifications et appuyez sur OK.

![yast](media/HowToHLI/HASetupWithStonith/yast-automatic-changes.png)

L'installation du package se poursuit. ![Capture d'écran représentant une fenêtre de console qui affiche la progression de l'installation.](media/HowToHLI/HASetupWithStonith/yast-performing-installation.png)

Cliquez sur Suivant.

![Capture d'écran représentant une fenêtre de console qui affiche un message de réussite.](media/HowToHLI/HASetupWithStonith/yast-installation-report.png)

Cliquez sur Terminer.

Vous devez également installer les packages libqt4 et libyui-qt.
```
zypper -n install libqt4
```
![Capture d'écran représentant une fenêtre de console lors de l'installation du package libqt4.](media/HowToHLI/HASetupWithStonith/zypper-install-libqt4.png)
```
zypper -n install libyui-qt
```
![Capture d'écran représentant une fenêtre de console lors de l'installation du package libyui-qt.](media/HowToHLI/HASetupWithStonith/zypper-install-ligyui.png)
![Capture d'écran représentant une fenêtre de console lors de l'installation du package libyui-qt (suite).](media/HowToHLI/HASetupWithStonith/zypper-install-ligyui_part2.png)
Yast2 devrait maintenant être en mesure d'ouvrir la vue graphique comme indiqué ici.
![Capture d'écran représentant le Centre de contrôle YaST dans lequel Logiciel et Mise à jour en ligne sont sélectionnés. ](media/HowToHLI/HASetupWithStonith/yast2-control-center.png)

### <a name="scenario-3-yast2-does-not-high-availability-option"></a>Scénario 3 : yast2 ne comporte pas l’option de haute disponibilité
Pour que l’option de haute disponibilité soit visible dans le centre de contrôle yast2, vous devez installer les packages supplémentaires.

Utilisez Yast2 > Logiciels > Gestion des logiciels > Sélectionner les modèles suivants.

- Base de serveur SAP HANA
- Compilateur et outils C/C++
- Haute disponibilité
- Base de serveur d’applications SAP

L’écran suivant montre les étapes permettant d’installer les modèles.

Utilisez yast2 > Logiciels > Gestion des logiciels.

![Capture d'écran représentant le Centre de contrôle YaST dans lequel Logiciel et Mise à jour en ligne sont sélectionnés pour entamer l'installation.](media/HowToHLI/HASetupWithStonith/yast2-control-center.png)

Sélectionnez les modèles.

![Capture d'écran représentant la sélection du premier modèle dans l'élément Compilateur et outils C / C ++.](media/HowToHLI/HASetupWithStonith/yast-pattern1.png)
![Capture d'écran représentant la sélection du deuxième modèle dans l'élément Compilateur et outils C / C ++](media/HowToHLI/HASetupWithStonith/yast-pattern2.png)

Cliquez sur **Accepter**.

![Capture d'écran représentant la boîte de dialogue Packages modifiés qui affiche les packages modifiés pour résoudre les dépendances.](media/HowToHLI/HASetupWithStonith/yast-changed-packages.png)

Cliquez sur **Continuer**.

![Capture d'écran représentant la page d'état Installation en cours.](media/HowToHLI/HASetupWithStonith/yast2-performing-installation.png)

Cliquez sur **Suivant** lorsque l’installation est terminée

![Capture d'écran représentant le rapport d'installation.](media/HowToHLI/HASetupWithStonith/yast2-installation-report.png)

### <a name="scenario-4-hana-installation-fails-with-gcc-assemblies-error"></a>Scénario 4 : échec de l’installation de HANA avec une erreur des assemblys gcc
L’installation de HANA échoue avec l’erreur suivante.

![Capture d'écran représentant un message d'erreur qui indique que le système d'exploitation n'est pas prêt pour les assemblys c c 5.](media/HowToHLI/HASetupWithStonith/Hana-installation-error.png)

Pour résoudre le problème, vous devez installer des bibliothèques (libgcc_sl et libstdc++6) comme indiqué ci-après.

![Capture d'écran représentant une fenêtre de console lors de l'installation des bibliothèques requises.](media/HowToHLI/HASetupWithStonith/zypper-install-lib.png)

### <a name="scenario-5-pacemaker-service-fails"></a>Scénario 5 : échec du service Pacemaker

Le problème suivant s’est produit au démarrage du service Pacemaker.

```
sapprdhdb95:/ # systemctl start pacemaker
A dependency job for pacemaker.service failed. See 'journalctl -xn' for details.
```
```
sapprdhdb95:/ # journalctl -xn
-- Logs begin at Thu 2017-09-28 09:28:14 EDT, end at Thu 2017-09-28 21:48:27 EDT. --
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [SERV  ] Service engine unloaded: corosync configuration map
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [QB    ] withdrawing server sockets
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [SERV  ] Service engine unloaded: corosync configuration ser
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [QB    ] withdrawing server sockets
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [SERV  ] Service engine unloaded: corosync cluster closed pr
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [QB    ] withdrawing server sockets
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [SERV  ] Service engine unloaded: corosync cluster quorum se
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [SERV  ] Service engine unloaded: corosync profile loading s
Sep 28 21:48:27 sapprdhdb95 corosync[68812]: [MAIN  ] Corosync Cluster Engine exiting normally
Sep 28 21:48:27 sapprdhdb95 systemd[1]: Dependency failed for Pacemaker High Availability Cluster Manager
-- Subject: Unit pacemaker.service has failed
-- Defined-By: systemd
-- Support: https://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- Unit pacemaker.service has failed.
--
-- The result is dependency.
```
```
sapprdhdb95:/ # tail -f /var/log/messages
2017-09-28T18:44:29.675814-04:00 sapprdhdb95 corosync[57600]:   [QB    ] withdrawing server sockets
2017-09-28T18:44:29.676023-04:00 sapprdhdb95 corosync[57600]:   [SERV  ] Service engine unloaded: corosync cluster closed process group service v1.01
2017-09-28T18:44:29.725885-04:00 sapprdhdb95 corosync[57600]:   [QB    ] withdrawing server sockets
2017-09-28T18:44:29.726069-04:00 sapprdhdb95 corosync[57600]:   [SERV  ] Service engine unloaded: corosync cluster quorum service v0.1
2017-09-28T18:44:29.726164-04:00 sapprdhdb95 corosync[57600]:   [SERV  ] Service engine unloaded: corosync profile loading service
2017-09-28T18:44:29.776349-04:00 sapprdhdb95 corosync[57600]:   [MAIN  ] Corosync Cluster Engine exiting normally
2017-09-28T18:44:29.778177-04:00 sapprdhdb95 systemd[1]: Dependency failed for Pacemaker High Availability Cluster Manager.
2017-09-28T18:44:40.141030-04:00 sapprdhdb95 systemd[1]: [/usr/lib/systemd/system/fstrim.timer:8] Unknown lvalue 'Persistent' in section 'Timer'
2017-09-28T18:45:01.275038-04:00 sapprdhdb95 cron[57995]: pam_unix(crond:session): session opened for user root by (uid=0)
2017-09-28T18:45:01.308066-04:00 sapprdhdb95 CRON[57995]: pam_unix(crond:session): session closed for user root
```

Pour le corriger, supprimez la ligne suivante du fichier */usr/lib/systemd/system/fstrim.timer*

```
Persistent=true
```

![Capture d'écran représentant le fichier f s trim avec la valeur Persistent = true à supprimer.](media/HowToHLI/HASetupWithStonith/Persistent.png)

### <a name="scenario-6-node-2-unable-to-join-the-cluster"></a>Scénario 6 : node2 ne parvient pas à joindre le cluster

Lors de la jonction de node2 au cluster à l’aide de la commande *ha-cluster-join*, l’erreur suivante s’est produite.

```
ERROR: Can’t retrieve SSH keys from <Primary Node>
```

![Capture d'écran représentant une fenêtre de console qui affiche le message d'erreur Impossible de récupérer les clés S S H à partir d'une adresse IP.](media/HowToHLI/HASetupWithStonith/ha-cluster-join-error.png)

Pour résoudre le problème, exécutez la commande suivante sur les deux nœuds.

```
ssh-keygen -q -f /root/.ssh/id_rsa -C 'Cluster Internal' -N ''
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
```

![Capture d'écran représentant une fenêtre de console partielle dans laquelle la commande est exécutée sur le premier nœud.](media/HowToHLI/HASetupWithStonith/ssh-keygen-node1.PNG)

![Capture d'écran représentant une fenêtre de console partielle dans laquelle la commande est exécutée sur le deuxième nœud.](media/HowToHLI/HASetupWithStonith/ssh-keygen-node2.PNG)

Après la correction précédente, node2 doit être ajouté au cluster.

![Capture d'écran représentant une fenêtre de console avec une commande ha-cluster-join réussie.](media/HowToHLI/HASetupWithStonith/ha-cluster-join-fix.png)

## <a name="10-general-documentation"></a>10. Documentation générale
Vous trouverez plus d’informations sur la configuration de la haute disponibilité SUSE dans les articles suivants : 

- [SAP HANA SR Performance Optimized Scenario](https://www.suse.com/docrep/documents/ir8w88iwu7/suse_linux_enterprise_server_for_sap_applications_12_sp1.pdf ) (Scénario d’optimisation des performances de réplication système de SAP HANA)
- [Délimitation selon le stockage](https://www.suse.com/documentation/sle_ha/book_sleha/data/sec_ha_storage_protect_fencing.html)
- [Blog - Using Pacemaker Cluster for SAP HANA- Part 1](https://blogs.sap.com/2017/11/19/be-prepared-for-using-pacemaker-cluster-for-sap-hana-part-1-basics/)
- [Blog - Using Pacemaker Cluster for SAP HANA- Part 2](https://blogs.sap.com/2017/11/19/be-prepared-for-using-pacemaker-cluster-for-sap-hana-part-2-failure-of-both-nodes/)
