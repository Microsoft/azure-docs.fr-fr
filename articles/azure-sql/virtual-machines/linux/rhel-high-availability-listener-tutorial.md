---
title: Configurer un écouteur de groupe de disponibilité pour SQL Server sur des machines virtuelles RHEL dans Azure - Machines virtuelles Linux | Microsoft Docs
description: Apprendre à configurer un écouteur de groupe de disponibilité dans SQL Server sur des machines virtuelles RHEL dans Azure
ms.service: virtual-machines-linux
ms.topic: tutorial
author: VanMSFT
ms.author: vanto
ms.reviewer: jroth
ms.date: 03/11/2020
ms.openlocfilehash: 83fde9e957cb2011ce585603e51d331be171bc08
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97586210"
---
# <a name="tutorial-configure-an-availability-group-listener-for-sql-server-on-rhel-virtual-machines-in-azure"></a>Tutoriel : Configurer un écouteur de groupe de disponibilité pour SQL Server sur des machines virtuelles RHEL dans Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!NOTE]
> Ce tutoriel est en **préversion publique**. 
>
> Dans ce tutoriel, nous utilisons SQL Server 2017 avec RHEL 7.6, mais il est possible d’utiliser SQL Server 2019 dans RHEL 7 ou RHEL 8 pour configurer la haute disponibilité. Les commandes permettant de configurer les ressources d’un groupe de disponibilité ont changé dans RHEL 8. Il est donc recommandé de consulter l’article [Créer une ressource de groupe de disponibilité](/sql/linux/sql-server-linux-availability-group-cluster-rhel#create-availability-group-resource) et d’étudier les ressources RHEL 8 pour obtenir des informations sur les commandes appropriées.

Ce tutoriel passe en revue les étapes à suivre pour créer un écouteur de groupe de disponibilité pour vos serveurs SQL Server sur des machines virtuelles RHEL dans Azure. Vous apprendrez à :

> [!div class="checklist"]
> - Créer un équilibreur de charge sur le portail Azure
> - Configurer le pool back-end pour l’équilibreur de charge
> - Créer une sonde pour l’équilibreur de charge
> - Configurer les règles d’équilibrage de charge
> - Créer la ressource d’équilibreur de charge dans le cluster
> - Créer l'écouteur de groupe de disponibilité
> - Tester la connexion à l’écouteur
> - Tester un basculement

## <a name="prerequisite"></a>Configuration requise

Avoir suivi le [Tutoriel : Configurer des groupes de disponibilité pour SQL Server sur des machines virtuelles RHEL dans Azure](rhel-high-availability-stonith-tutorial.md)

## <a name="create-the-load-balancer-in-the-azure-portal"></a>Créer l’équilibrage de charge dans le portail Azure

Les instructions suivantes vous guident tout au long des étapes 1 à 4 de la section [Créer et configurer l’équilibreur de charge sur le portail Azure](../windows/availability-group-load-balancer-portal-configure.md#create--configure-load-balancer) de l’article [Équilibreur de charge – Portail Azure](../windows/availability-group-load-balancer-portal-configure.md).

### <a name="create-the-load-balancer"></a>Créer l’équilibreur de charge

1. Dans le portail Azure, ouvrez le groupe de ressources contenant les machines virtuelles SQL Server. 

2. Dans le groupe de ressources, cliquez sur **Ajouter**.

3. Recherchez **équilibreur de charge** puis, dans les résultats de la recherche, sélectionnez **Équilibreur de charge**, publié par **Microsoft**.

4. Dans le panneau **Équilibrage de charge**, cliquez sur **Créer**.

5. Dans la boîte de dialogue **Créer l’équilibreur de charge**, configurez l’équilibreur de charge comme suit :

   | Paramètre | Valeur |
   | --- | --- |
   | **Nom** |Nom de l’équilibrage de charge. Par exemple, **sqlLB**. |
   | **Type** |**Interne** |
   | **Réseau virtuel** |Le réseau virtuel par défaut qui a été créé doit être nommé **VM1VNET**. |
   | **Sous-réseau** |Sélectionnez le sous-réseau contenant les instances de SQL Server. Par défaut, il s’agit de **VM1Subnet**.|
   | **Affectation d’adresses IP** |**Statique** |
   | **Adresse IP privée** |Utilisez l’adresse IP `virtualip` qui a été créée dans le cluster. |
   | **Abonnement** |Utilisez l’abonnement qui a été utilisé pour votre groupe de ressources. |
   | **Groupe de ressources** |Sélectionnez le groupe de ressources contenant les instances de SQL Server. |
   | **Lieu** |Sélectionnez l’emplacement Azure contenant les instances de SQL Server. |

### <a name="configure-the-back-end-pool"></a>Configurez le pool principal
Azure appelle *pool principal* ce pool d’adresses principal. En l’occurrence, le pool back-end est constitué des adresses des trois instances de SQL Server de votre groupe de disponibilité. 

1. Dans votre groupe de ressources, cliquez sur l’équilibreur de charge que vous avez créé. 

2. Dans **Paramètres**, cliquez sur **Pools principaux**.

3. Dans **Pools principaux**, cliquez sur **Ajouter** pour créer un pool d’adresses principal. 

4. Dans **Ajouter un pool principal** sous **Nom**, entrez un nom pour le pool principal.

5. Sous **Associé à**, sélectionnez **Machine virtuelle**. 

6. Sélectionnez chaque machine virtuelle de l’environnement, puis associez l’adresse IP appropriée à chaque sélection.

    :::image type="content" source="media/rhel-high-availability-listener-tutorial/add-backend-pool.png" alt-text="Ajouter un pool back-end":::

7. Cliquez sur **Add**. 

### <a name="create-a-probe"></a>Créer une sonde

La sonde définit la façon dont Azure identifie l’instance de SQL Server qui détient l’écouteur du groupe de disponibilité. Azure sonde le service avec l’adresse IP sur un port que vous définissez lors de la création de la sonde.

1. Dans le panneau **Paramètres** de l’équilibreur de charge, cliquez sur **Sondes d’intégrité**. 

2. Dans le panneau **Sondes d’intégrité**, cliquez sur **Ajouter**.

3. Configurez la sonde dans le panneau **Ajouter une sonde** . Utilisez les valeurs suivantes pour configurer la sonde :

   | Paramètre | Valeur |
   | --- | --- |
   | **Nom** |Nom de la sonde. Par exemple, **SQLAlwaysOnEndPointProbe**. |
   | **Protocole** |**TCP** |
   | **Port** |Vous pouvez utiliser n’importe quel port disponible. Par exemple, *59999*. |
   | **Intervalle** |*5* |
   | **Seuil de défaillance sur le plan de l’intégrité** |*2* |

4.  Cliquez sur **OK**. 

5. Connectez-vous à toutes vos machines virtuelles, puis ouvrez le port de la sonde à l’aide des commandes suivantes :

    ```bash
    sudo firewall-cmd --zone=public --add-port=59999/tcp --permanent
    sudo firewall-cmd --reload
    ```

Azure crée la sonde puis l’utilise pour identifier l’instance de SQL Server qui a l’écouteur pour le groupe de disponibilité.

### <a name="set-the-load-balancing-rules"></a>Définir les règles d’équilibrage de charge

Les règles d’équilibrage de charge déterminent comment l’équilibreur de charge route le trafic vers les instances de SQL Server. Pour cet équilibreur de charge, vous activez le retour direct du serveur, car une seule des trois instances de SQL Server est propriétaire de la ressource d’écouteur de groupe de disponibilité à un moment donné.

1. Dans le panneau **Paramètres** de l’équilibrage de charge, cliquez sur **Règles d’équilibrage de charge**. 

2. Dans le panneau **Règles d’équilibrage de charge**, cliquez sur **Ajouter**.

3. Dans le panneau **Ajouter une règle d’équilibrage de charge**, configurez la règle d’équilibrage de charge. Utilisez les paramètres suivants : 

   | Paramètre | Valeur |
   | --- | --- |
   | **Nom** |Nom représentant la règle d’équilibrage de charge. Par exemple, **SQLAlwaysOnEndPointListener**. |
   | **Protocole** |**TCP** |
   | **Port** |*1433* |
   | **Port principal** |*1433*. Cette valeur est ignorée, car cette règle utilise **Adresse IP flottante (retour direct du serveur)** . |
   | **Sonde** |Utilisez le nom de la sonde que vous avez créée pour cet équilibrage de charge. |
   | **Persistance de session** |**Aucun** |
   | **Délai d’inactivité (minutes).** |*4* |
   | **Adresse IP flottante (retour direct du serveur)** |**Activé** |

   :::image type="content" source="media/rhel-high-availability-listener-tutorial/add-load-balancing-rule.png" alt-text="Ajouter une règle d’équilibrage de charge":::

4. Cliquez sur **OK**. 
5. Azure configure la règle d’équilibrage de charge. L’équilibreur de charge est maintenant configuré pour router le trafic vers l’instance de SQL Server qui héberge l’écouteur pour le groupe de disponibilité. 

À ce stade, le groupe de ressources a un équilibreur de charge qui se connecte à tous les ordinateurs SQL Server. L’équilibreur de charge contient aussi une adresse IP pour l’écouteur de groupe de disponibilité Always On SQL Server, de sorte que n’importe quel ordinateur peut répondre aux demandes pour les groupes de disponibilité.

## <a name="create-the-load-balancer-resource-in-the-cluster"></a>Créer la ressource d’équilibreur de charge dans le cluster

1. Connectez-vous à la machine virtuelle principale. Nous devons créer la ressource pour activer le port de la sonde de l’équilibreur de charge Azure (59999 est utilisé dans notre exemple). Exécutez la commande suivante :

    ```bash
    sudo pcs resource create azure_load_balancer azure-lb port=59999
    ```

1. Créez un groupe qui contient `virtualip` et la ressource `azure_load_balancer` :

    ```bash
    sudo pcs resource group add virtualip_group azure_load_balancer virtualip
    ```

### <a name="add-constraints"></a>Ajouter des contraintes

1. Une contrainte de colocalisation doit être configurée pour faire en sorte que l’adresse IP de l’équilibreur de charge Azure et la ressource de groupe de disponibilité s’exécutent sur le même nœud. Exécutez la commande suivante :

    ```bash
    sudo pcs constraint colocation add azure_load_balancer ag_cluster-master INFINITY with-rsc-role=Master
    ```
1. Créez une contrainte de classement pour faire en sorte que la ressource de groupe de disponibilité soit active et en cours d’exécution avant l’adresse IP de l’équilibreur de charge Azure. Alors que la contrainte de colocalisation implique une contrainte de classement, elle l’applique.

    ```bash
    sudo pcs constraint order promote ag_cluster-master then start azure_load_balancer
    ```

1. Pour vérifier les contraintes, exécutez la commande suivante :

    ```bash
    sudo pcs constraint list --full
    ```

    Vous devez normalement voir la sortie suivante.

    ```output
    Location Constraints:
    Ordering Constraints:
      promote ag_cluster-master then start virtualip (kind:Mandatory) (id:order-ag_cluster-master-virtualip-mandatory)
      promote ag_cluster-master then start azure_load_balancer (kind:Mandatory) (id:order-ag_cluster-master-azure_load_balancer-mandatory)
    Colocation Constraints:
      virtualip with ag_cluster-master (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-master-INFINITY)
      azure_load_balancer with ag_cluster-master (score:INFINITY) (with-rsc-role:Master) (id:colocation-azure_load_balancer-ag_cluster-master-INFINITY)
    Ticket Constraints:
    ```

## <a name="create-the-availability-group-listener"></a>Créer l'écouteur de groupe de disponibilité

1. Sur le nœud principal, exécutez la commande suivante dans SQLCMD ou SSMS :

    - Remplacez l’adresse IP utilisée ci-dessous par l’adresse IP `virtualip`.

    ```sql
    ALTER AVAILABILITY
    GROUP [ag1] ADD LISTENER 'ag1-listener' (
            WITH IP(('10.0.0.7'    ,'255.255.255.0'))
                ,PORT = 1433
            );
    GO
    ```

1. Connectez-vous à chaque nœud de machine virtuelle. Utilisez la commande suivante pour ouvrir le fichier HOSTS et configurer la résolution de noms d’hôtes pour `ag1-listener` sur chaque machine.

    ```
    sudo vi /etc/hosts
    ```

    Dans l’éditeur **vi**, entrez `i` pour insérer du texte puis, sur une ligne vide, ajoutez l’adresse IP de `ag1-listener`. Ajoutez ensuite `ag1-listener` après un espace en regard de l’adresse IP.

    ```output
    <IP of ag1-listener> ag1-listener
    ```

    Pour quitter l’éditeur **vi**, appuyez d’abord sur la touche **Echap**, puis entrez la commande `:wq` pour écrire le fichier et quitter. Procédez de la sorte sur chaque nœud.

## <a name="test-the-listener-and-a-failover"></a>Tester l’écouteur et un basculement

### <a name="test-logging-in-to-sql-server-using-the-availability-group-listener"></a>Tester la journalisation dans SQL Server à l’aide de l’écouteur de groupe de disponibilité

1. Utilisez SQLCMD pour vous connecter au nœud principal de SQL Server en utilisant le nom de l’écouteur de groupe de disponibilité :

    - Utilisez un compte de connexion qui a été créé précédemment et remplacez `<YourPassword>` par le mot de passe correct. L’exemple ci-dessous utilise le compte de connexion `sa` qui a été créé avec SQL Server.

    ```bash
    sqlcmd -S ag1-listener -U sa -P <YourPassword>
    ```

1. Vérifiez le nom du serveur auquel vous êtes connecté. Exécutez la commande suivante dans SQLCMD :

    ```sql
    SELECT @@SERVERNAME
    ```

    Votre sortie doit présenter le nœud principal actif. Si vous n’avez jamais testé un basculement, il s’agit de `VM1`.

    Quittez la session SQL Server en tapant la commande `exit`.

### <a name="test-a-failover"></a>Tester un basculement

1. Exécutez la commande suivante pour faire basculer manuellement le réplica principal vers `<VM2>` ou vers un autre réplica. Remplacez `<VM2>` par le nom du serveur.

    ```bash
    sudo pcs resource move ag_cluster-master <VM2> --master
    ```

1. Si vous vérifiez vos contraintes, vous verrez qu’une autre contrainte a été ajoutée en raison du basculement manuel :

    ```bash
    sudo pcs constraint list --full
    ```

    Vous verrez qu’une contrainte associée à l’ID `cli-prefer-ag_cluster-master` a été ajoutée.

1. Supprimez la contrainte avec l’ID `cli-prefer-ag_cluster-master` à l’aide de la commande suivante :

    ```bash
    sudo pcs constraint remove cli-prefer-ag_cluster-master
    ```

1. Vérifiez vos ressources de cluster à l’aide de la commande `sudo pcs resource`. Vous devriez voir que l’instance principale est désormais `<VM2>`.

    > [!NOTE]
    > Cet article contient des références au terme esclave, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.


    ```output
    [<username>@<VM1> ~]$ sudo pcs resource
    Master/Slave Set: ag_cluster-master [ag_cluster]
        Masters: [ <VM2> ]
        Slaves: [ <VM1> <VM3> ]
    Resource Group: virtualip_group
        azure_load_balancer        (ocf::heartbeat:azure-lb):      Started <VM2>
        virtualip  (ocf::heartbeat:IPaddr2):       Started <VM2>
    ```

1. Utilisez SQLCMD pour vous connecter à votre réplica principal en utilisant le nom de l’écouteur :

    - Utilisez un compte de connexion qui a été créé précédemment et remplacez `<YourPassword>` par le mot de passe correct. L’exemple ci-dessous utilise le compte de connexion `sa` qui a été créé avec SQL Server.

    ```bash
    sqlcmd -S ag1-listener -U sa -P <YourPassword>
     ```

1. Vérifiez le serveur auquel vous êtes connecté. Exécutez la commande suivante dans SQLCMD :

    ```sql
    SELECT @@SERVERNAME
    ```

    Vous constatez que vous êtes maintenant connecté à la machine virtuelle vers laquelle vous avez basculé.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les équilibreurs de charge dans Azure, consultez :

> [!div class="nextstepaction"]
> [Configurer un équilibreur de charge pour un groupe de disponibilité sur SQL Server sur des machines virtuelles Azure](../windows/availability-group-load-balancer-portal-configure.md)
