---
title: Connexion à distance à un nœud de cluster Azure Service Fabric
description: Découvrez comment vous connecter à distance à une instance de groupe identique (un nœud de cluster Service Fabric).
ms.topic: conceptual
ms.date: 03/23/2018
ms.openlocfilehash: 98d573af4fc2026134e75d4caf24a09e57e52c87
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012542"
---
# <a name="remote-connect-to-a-virtual-machine-scale-set-instance-or-a-cluster-node"></a>Connexion distante à une instance de groupe de machines virtuelles identiques ou à un nœud de cluster
Dans un cluster Service Fabric s’exécutant dans Azure, chaque type de nœud de cluster que vous définissez [définit une échelle mise à l’échelle séparée des machines virtuelles](service-fabric-cluster-nodetypes.md).  Vous pouvez vous connecter à distance à des instances de groupes identiques (nœuds de cluster) spécifiques.  Contrairement aux machines virtuelles à une seule instance, les instances de groupe identique ne possèdent pas leurs propres adresses IP virtuelles. Cela peut poser des problèmes quand il s’agit de rechercher une adresse IP et un port permettant de se connecter à distance à une instance spécifique.

Pour rechercher une adresse IP et un port permettant de se connecter à distance à une instance spécifique, effectuez les étapes suivantes.

1. Obtenez les règles NAT de trafic entrant pour le protocole RDP (Remote Desktop Protocol).

    En général, chaque type de nœud défini dans votre cluster a sa propre adresse IP virtuelle et un équilibreur de charge dédié. Par défaut, l’équilibreur de charge pour un type de nœud est nommé avec le format suivant : *LB-{nom de cluster}-{type de nœud}* ; par exemple, *LB-mycluster-FrontEnd*. 
    
    Dans la page de votre équilibreur de charge dans le portail Azure, sélectionnez **Paramètres** > **Règles NAT de trafic entrant** : 

    ![Capture d’écran de la page d’un équilibreur de charge dans le portail Azure. Dans le menu de gauche, sous PARAMÈTRES, l’option Règles NAT de trafic entrant est sélectionnée.](./media/service-fabric-cluster-remote-connect-to-azure-cluster-node/lb-window.png)

    La capture d’écran suivante montre les règles NAT de trafic entrant pour un type de nœud nommé FrontEnd : 

    ![Capture d’écran montrant les règles NAT de trafic entrant pour un équilibreur de charge. Le nom, la version du protocole Internet, la destination, la cible et le service sont répertoriés pour chaque règle.](./media/service-fabric-cluster-remote-connect-to-azure-cluster-node/nat-rules.png)

    Pour chaque nœud, l’adresse IP s’affiche dans la colonne **DESTINATION**, la colonne **TARGET** donne l’instance de groupe identique et la colonne **SERVICE** fournit le numéro de port. Pour une connexion à distance, les ports sont alloués à chaque nœud dans l’ordre croissant, en commençant par le port 3389.

    Les règles NAT de trafic entrant sont également disponibles dans la section `Microsoft.Network/loadBalancers` du modèle Resource Manager de votre cluster.
    
2. Pour vérifier le mappage du port d’entrée au port cible d’un nœud, vous pouvez cliquer sur sa règle et examiner la valeur **Port cible**. La capture d’écran suivante montre la règle NAT de trafic entrant du nœud **FrontEnd (Instance 1)** à l’étape précédente. Notez que, même si le numéro de port (entrant) est 3390, le port cible est mappé au port 3389, qui est le port du service RDP sur la cible.  

    ![Mappage du port cible](./media/service-fabric-cluster-remote-connect-to-azure-cluster-node/port-mapping.png)

    Par défaut, pour les clusters Windows, le port cible est le port 3389, qui est mappé au service RDP sur le nœud cible. Pour les clusters Linux, le port cible est le port 22, qui est mappé au service SSH (Secure Shell).

3. Connectez-vous à distance au nœud (instance de groupe identique) spécifique. Vous pouvez utiliser le nom d’utilisateur et le mot de passe que vous avez définis lors de la création du cluster ou de toutes autres informations d’identification que vous avez configurées. 

    La capture d’écran suivante illustre l’utilisation de la connexion Bureau à distance pour se connecter au nœud **FrontEnd (Instance 1)** dans un cluster Windows :
    
    ![Connexion Bureau à distance](./media/service-fabric-cluster-remote-connect-to-azure-cluster-node/rdp-connect.png)

    Sur les nœuds Linux, vous pouvez vous connecter avec SSH (par souci de concision, l’exemple suivant réutilise la même adresse IP et le même port) :

    ``` bash
    ssh SomeUser@40.117.156.199 -p 3390
    ```


Pour les prochaines étapes, lisez les articles suivants :
* Consultez [Vue d’ensemble de la fonction « Déployer n’importe où » et comparaison avec les clusters gérés par Azure](service-fabric-deploy-anywhere.md).
* Découvrez plus en détail la [sécurité des clusters](service-fabric-cluster-security.md).
* [Mettre à jour les valeurs de plages de port RDP](./scripts/service-fabric-powershell-change-rdp-port-range.md) sur des machines virtuelles de cluster après le déploiement
* [Modifier le nom d’utilisateur et le mot de passe administrateur](./scripts/service-fabric-powershell-change-rdp-user-and-pw.md) pour les machines virtuelles de cluster

