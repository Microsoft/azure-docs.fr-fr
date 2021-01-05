---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: fd635d4c0563c35979f8d85c33dfbde35f05f9e6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91401027"
---
L’écouteur de groupe de disponibilité est une adresse IP et un nom réseau sur lesquels le groupe de disponibilité de SQL Server écoute. Pour créer l’écouteur de groupe de disponibilité, procédez comme suit :

1. <a name="getnet"></a>Récupérez le nom de la ressource réseau de cluster.

    a. Utilisez le protocole RDP pour vous connecter à la machine virtuelle Azure qui héberge le réplica principal. 

    b. Ouvrez le Gestionnaire du cluster de basculement.

    c. sélectionnez le nœud **Réseaux** et notez le nom de réseau du cluster. Utilisez ce nom dans la variable `$ClusterNetworkName` du script PowerShell. Dans l’image suivante, le nom réseau du cluster est **Cluster Network 1** :

   ![Nom réseau du cluster](./media/virtual-machines-ag-listener-configure/90-clusternetworkname.png)

1. <a name="addcap"></a>Ajoutez le point d’accès client.  
    Le point d’accès client est le nom réseau que les applications utilisent pour se connecter aux bases de données dans un groupe de disponibilité. Créez le point d’accès client dans le Gestionnaire du cluster de basculement .

    a. Développez le nom du cluster, puis cliquez sur **Rôles**.

    b. Dans le volet **Rôles**, cliquez avec le bouton droit sur le nom du groupe de disponibilité, puis sélectionnez **Ajouter une ressource** > **Point d’accès client**.

   ![Capture d’écran montrant l’option de menu Point d’accès client.](./media/virtual-machines-ag-listener-configure/92-addclientaccesspoint.png)

    c. Dans la zone **Nom**, créez un nom pour ce nouvel écouteur. 
   Le nom du nouvel écouteur est le nom réseau que les applications utilisent pour se connecter aux bases de données dans le groupe de disponibilité de SQL Server.

    d. Pour terminer la création de l’écouteur, cliquez sur **Suivant** deux fois, puis cliquez sur **Terminer**. Ne mettez pas l'écouteur ou la ressource en ligne à ce stade.

1. Désactivez le rôle de cluster de groupe de disponibilité. Dans le **Gestionnaire du cluster de basculement**, sous **Rôles**, cliquez avec le bouton droit sur le rôle, puis sélectionnez **Arrêter le rôle**.

1. <a name="congroup"></a>Configurez la ressource IP du groupe de disponibilité.

    a. Cliquez sur l’onglet **Ressources**, puis développez le point d’accès client que vous avez créé.  
    Le point d’accès client est hors connexion.

   ![Point d’accès client](./media/virtual-machines-ag-listener-configure/94-newclientaccesspoint.png) 

    b. Cliquez avec le bouton droit sur la ressource IP, puis cliquez sur Propriétés. Notez le nom de l’adresse IP et utilisez-le dans la variable `$IPResourceName` dans le script PowerShell.

    c. Sous **Adresse IP**, cliquez sur **Adresse IP statique**. Pour l’adresse IP, utilisez l’adresse que vous avez définie pour l’équilibreur de charge sur le portail Azure.

   ![Capture d’écran montrant où vous définissez l’adresse IP.](./media/virtual-machines-ag-listener-configure/96-ipresource.png) 

    <!-----------------------I don't see this option on server 2016
    1. Disable NetBIOS for this address and click **OK**. Repeat this step for each IP resource if your solution spans multiple Azure VNets. 
    ------------------------->

1. <a name = "dependencyGroup"></a>Créez une dépendance entre la ressource de groupe de disponibilité de SQL Server et le point d’accès client.

    a. Dans le Gestionnaire du cluster de basculement, cliquez sur **Rôles**, puis sur votre groupe de disponibilité.

    b. Dans l’onglet **Ressources**, sous **Autres ressources**, cliquez avec le bouton droit sur la ressource de groupe de disponibilité, puis cliquez sur **Propriétés**. 

    c. Dans l’onglet Dépendances, ajoutez le nom de la ressource du point d’accès client (écouteur).

   ![Capture d’écran montrant où ajouter le nom sous l’onglet Dépendances.](./media/virtual-machines-ag-listener-configure/97-propertiesdependencies.png) 

    d. Cliquez sur **OK**.

1. <a name="listname"></a>Créez une dépendance entre la ressource du point d’accès client et l’adresse IP.

    a. Dans le Gestionnaire du cluster de basculement, cliquez sur **Rôles**, puis sur votre groupe de disponibilité. 

    b. Dans l’onglet **Ressources**, cliquez avec le bouton droit sur la ressource du point d’accès client sous **Nom du serveur**, puis cliquez sur **Propriétés**. 

   ![Capture d’écran montrant l’option de menu Propriétés pour le nom du serveur.](./media/virtual-machines-ag-listener-configure/98-dependencies.png) 

    c. Cliquez sur l'onglet **Dépendances** . Vérifiez que l’adresse IP est une dépendance. Si tel n’est pas le cas, définissez une dépendance sur l’adresse IP. Si plusieurs ressources sont répertoriées, vérifiez que les adresses IP ont des dépendances OR (et non des dépendances AND). Cliquez sur **OK**. 

   ![Ressource IP](./media/virtual-machines-ag-listener-configure/98-propertiesdependencies.png) 

    >[!TIP]
    >Vous pouvez vérifier que les dépendances sont correctement configurées. Dans le Gestionnaire du cluster de basculement, accédez à Rôles, cliquez avez le bouton droit sur le groupe de disponibilité, et cliquez sur **Autres actions**, puis sur **Afficher le rapport de dépendance**. Lorsque les dépendances sont correctement configurées, le groupe de disponibilité dépend du nom réseau, et le nom réseau dépend de l’adresse IP. 


1. <a name="setparam"></a>Définissez les paramètres de cluster dans PowerShell.

   a. Copiez le script PowerShell suivant sur l’une de vos instances SQL Server. Mettez à jour les variables de votre environnement.

   - `$ListenerILBIP` est l’adresse IP que vous avez créée sur l’équilibreur de charge Azure pour l’écouteur du groupe de disponibilité.
    
   - `$ListenerProbePort` est le port que vous avez configuré sur l’équilibreur de charge Azure pour l’écouteur du groupe de disponibilité.

   ```powershell
   $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
   $IPResourceName = "<IPResourceName>" # the IP Address resource name
   $ListenerILBIP = "<n.n.n.n>" # the IP Address of the Internal Load Balancer (ILB). This is the static IP address for the load balancer you configured in the Azure portal.
   [int]$ListenerProbePort = <nnnnn>
  
   Import-Module FailoverClusters

   Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ListenerILBIP";"ProbePort"=$ListenerProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
   ```

   b. Définissez les paramètres du cluster en exécutant le script PowerShell sur l’un des nœuds du cluster.  

   > [!NOTE]
   > Si vos instances SQL Server se trouvent dans différentes régions, vous devez exécuter le script PowerShell à deux reprises. La première fois, utilisez `$ListenerILBIP` et `$ListenerProbePort` à partir de la première région. La seconde fois, utilisez `$ListenerILBIP` et `$ListenerProbePort` à partir de la seconde région. Le nom réseau du cluster et le nom de ressource IP du cluster sont également différents pour chaque région.

1. Activez le rôle de cluster de groupe de disponibilité. Dans le **Gestionnaire du cluster de basculement**, sous **Rôles**, cliquez avec le bouton droit sur le rôle, puis sélectionnez **Démarrer le rôle**.

Si nécessaire, répétez les étapes ci-dessus afin de définir les paramètres de cluster pour l’adresse IP du cluster WSFC.

1. Obtenez le nom de l’adresse IP du cluster WSFC. Dans le **Gestionnaire du cluster de basculement**, sous **Principales ressources du cluster**, recherchez **Nom du serveur**.

1. Cliquez avec le bouton droit sur **Adresse IP**, puis sélectionnez **Propriétés**.

1. Copiez le **Nom** de l’adresse IP. Ce peut être `Cluster IP Address`. 

1. <a name="setwsfcparam"></a>Définissez les paramètres de cluster dans PowerShell.
  
   a. Copiez le script PowerShell suivant sur l’une de vos instances SQL Server. Mettez à jour les variables de votre environnement.

   - `$ClusterCoreIP` est l’adresse IP que vous avez créée sur l’équilibreur de charge Azure pour la ressource principale de cluster WSFC. Cette adresse est différente de l’adresse IP de l’écouteur du groupe de disponibilité.

   - `$ClusterProbePort` est le port que vous avez configuré sur l’équilibreur de charge Azure pour la sonde d’intégrité WSFC. Ce port est différent de la sonde configurée pour l’écouteur du groupe de disponibilité.

   ```powershell
   $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
   $IPResourceName = "<ClusterIPResourceName>" # the IP Address resource name
   $ClusterCoreIP = "<n.n.n.n>" # the IP Address of the Cluster IP resource. This is the static IP address for the load balancer you configured in the Azure portal.
   [int]$ClusterProbePort = <nnnnn> # The probe port from the WSFCEndPointprobe in the Azure portal. This port must be different from the probe port for the availability group listener probe port.
  
   Import-Module FailoverClusters
  
   Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ClusterCoreIP";"ProbePort"=$ClusterProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
   ```

   b. Définissez les paramètres du cluster en exécutant le script PowerShell sur l’un des nœuds du cluster.  

>[!WARNING]
>Le port pour la sonde d’intégrité de l’écouteur du groupe de disponibilité doit être différent du port pour la sonde d’intégrité de l’adresse IP principale du cluster. Dans ces exemples, le port de l’écouteur est 59999 et le port de la sonde d’intégrité de l’adresse IP principale du cluster est 58888. Une règle de pare-feu d’autorisation du trafic entrant doit être définie pour les deux ports.
