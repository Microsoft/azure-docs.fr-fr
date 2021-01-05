---
title: Configurer des groupes de disponibilité pour SQL Server sur des machines virtuelles RHEL dans Azure - Machines virtuelles Linux | Microsoft Docs
description: Découvrir comment configurer la haute disponibilité dans un environnement de cluster RHEL et comment configurer STONITH
ms.service: virtual-machines-linux
ms.topic: tutorial
author: VanMSFT
ms.author: vanto
ms.reviewer: jroth
ms.date: 06/25/2020
ms.openlocfilehash: 533f5c9e38818a8e37482cbbb3a90602366eca6f
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97587211"
---
# <a name="tutorial-configure-availability-groups-for-sql-server-on-rhel-virtual-machines-in-azure"></a>Tutoriel : Configurer des groupes de disponibilité pour SQL Server sur des machines virtuelles RHEL dans Azure 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!NOTE]
> Dans ce tutoriel, nous utilisons SQL Server 2017 avec RHEL 7.6, mais il est possible d’utiliser SQL Server 2019 dans RHEL 7 ou RHEL 8 pour configurer la haute disponibilité. Les commandes permettant de configurer les ressources du cluster Pacemaker et de groupe de disponibilité ont changé dans RHEL 8. Il est donc recommandé de consulter l’article [Créer une ressource de groupe de disponibilité](/sql/linux/sql-server-linux-availability-group-cluster-rhel#create-availability-group-resource) et d’étudier les ressources RHEL 8 pour obtenir des informations sur les commandes appropriées.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> - Créer un groupe de ressources, un groupe à haute disponibilité et des machines virtuelles Linux
> - Activer la haute disponibilité
> - Créez un cluster Pacemaker
> - Configurer un agent d’isolation en créant un appareil STONITH
> - Installer SQL Server et mssql-tools sur RHEL
> - Configurer un groupe de disponibilité SQL Server AlwaysOn
> - Configurer les ressources de groupe de disponibilité dans le cluster Pacemaker
> - Tester un basculement et l’agent d’isolation

Ce tutoriel utilise l’interface Azure CLI pour déployer des ressources dans Azure.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.30 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Si vous avez plusieurs abonnements, [sélectionnez l’abonnement](/cli/azure/manage-azure-subscriptions-azure-cli) dans lequel vous souhaitez déployer ces ressources.

Utilisez la commande suivante pour créer un groupe de ressources `<resourceGroupName>` dans une région. Remplacez `<resourceGroupName>` par le nom de votre choix. Pour ce tutoriel, nous utilisons `East US 2`. Pour plus d’informations, consultez ce [guide de démarrage rapide](../../../application-gateway/quick-create-cli.md).

```azurecli-interactive
az group create --name <resourceGroupName> --location eastus2
```

## <a name="create-an-availability-set"></a>Créer un groupe à haute disponibilité

L’étape suivante consiste à créer un groupe à haute disponibilité. Exécutez la commande suivante dans Azure Cloud Shell et remplacez `<resourceGroupName>` par le nom de votre groupe de ressources. Choisissez un nom pour `<availabilitySetName>`.

```azurecli-interactive
az vm availability-set create \
    --resource-group <resourceGroupName> \
    --name <availabilitySetName> \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 2
```

Une fois l’exécution de la commande terminée, vous devez obtenir les résultats suivants :

```output
{
  "id": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/availabilitySets/<availabilitySetName>",
  "location": "eastus2",
  "name": "<availabilitySetName>",
  "platformFaultDomainCount": 2,
  "platformUpdateDomainCount": 2,
  "proximityPlacementGroup": null,
  "resourceGroup": "<resourceGroupName>",
  "sku": {
    "capacity": null,
    "name": "Aligned",
    "tier": null
  },
  "statuses": null,
  "tags": {},
  "type": "Microsoft.Compute/availabilitySets",
  "virtualMachines": []
}
```

## <a name="create-rhel-vms-inside-the-availability-set"></a>Créer des machines virtuelles RHEL dans le groupe à haute disponibilité

> [!WARNING]
> Si vous choisissez une image RHEL avec paiement à l’utilisation (PAYG) et que vous configurez la haute disponibilité, vous devrez peut-être inscrire votre abonnement. Cela peut vous obliger à payer deux fois pour l’abonnement, car vous devrez payer à la fois pour l’abonnement RHEL Microsoft Azure de la machine virtuelle et pour l’abonnement à Red Hat. Pour plus d’informations, consultez https://access.redhat.com/solutions/2458541.
>
> Pour éviter cette « double facturation », utilisez une image de haute disponibilité RHEL lors de la création de la machine virtuelle Azure. Les images proposées en tant qu’images RHEL de haute disponibilité sont également des images avec paiement à l’utilisation pour lesquelles est préactivé un dépôt de haute disponibilité.

1. Obtenez la liste des images de machines virtuelles qui proposent RHEL avec une haute disponibilité :

    ```azurecli-interactive
    az vm image list --all --offer "RHEL-HA"
    ```

    Les résultats suivants doivent s’afficher :

    ```output
    [
      {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "7.4",
    "urn": "RedHat:RHEL-HA:7.4:7.4.2019062021",
    "version": "7.4.2019062021"
       },
       {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "7.5",
    "urn": "RedHat:RHEL-HA:7.5:7.5.2019062021",
    "version": "7.5.2019062021"
        },
        {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "7.6",
    "urn": "RedHat:RHEL-HA:7.6:7.6.2019062019",
    "version": "7.6.2019062019"
         },
         {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "8.0",
    "urn": "RedHat:RHEL-HA:8.0:8.0.2020021914",
    "version": "8.0.2020021914"
         },
         {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "8.1",
    "urn": "RedHat:RHEL-HA:8.1:8.1.2020021914",
    "version": "8.1.2020021914"
          },
          {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "80-gen2",
    "urn": "RedHat:RHEL-HA:80-gen2:8.0.2020021915",
    "version": "8.0.2020021915"
           },
           {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "81_gen2",
    "urn": "RedHat:RHEL-HA:81_gen2:8.1.2020021915",
    "version": "8.1.2020021915"
           }
    ]
    ```

    Pour ce tutoriel, nous choisissons l’image `RedHat:RHEL-HA:7.6:7.6.2019062019` pour l’exemple RHEL 7 et `RedHat:RHEL-HA:8.1:8.1.2020021914` pour l’exemple RHEL 8.
    
    Vous pouvez également choisir SQL Server 2019 préinstallé sur les images RHEL8-HA. Pour afficher la liste de ces images, exécutez la commande suivante :  
    
    ```azurecli-interactive
    az vm image list --all --offer "sql2019-rhel8"
    ```

    Les résultats suivants doivent s’afficher :

    ```output
    [
      {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "enterprise",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:enterprise:15.0.200317",
    "version": "15.0.200317"
       },
       }
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "enterprise",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:enterprise:15.0.200512",
    "version": "15.0.200512"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "sqldev",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:sqldev:15.0.200317",
    "version": "15.0.200317"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "sqldev",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:sqldev:15.0.200512",
    "version": "15.0.200512"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "standard",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:standard:15.0.200317",
    "version": "15.0.200317"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "standard",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:standard:15.0.200512",
    "version": "15.0.200512"
       }
    ]
    ```

    Si vous utilisez l’une des images ci-dessus pour créer les machines virtuelles, SQL Server 2019 est préinstallé. Ignorez la section [Installer SQL Server et mssql-tools](#install-sql-server-and-mssql-tools) comme décrit dans cet article.
    
    
    > [!IMPORTANT]
    > Pour la configuration du groupe de disponibilité, les noms de machines doivent comporter moins de 15 caractères. Le nom d’utilisateur ne peut pas contenir de caractères majuscules, et les mots de passe doivent comporter plus de 12 caractères.

1. Nous voulons créer 3 machines virtuelles dans le groupe à haute disponibilité. Remplacez ce qui suit dans la commande ci-dessous :

    - `<resourceGroupName>`
    - `<VM-basename>`
    - `<availabilitySetName>`
    - `<VM-Size>` - Par exemple, « Standard_D16_v3 »
    - `<username>`
    - `<adminPassword>`

    ```azurecli-interactive
    for i in `seq 1 3`; do
           az vm create \
             --resource-group <resourceGroupName> \
             --name <VM-basename>$i \
             --availability-set <availabilitySetName> \
             --size "<VM-Size>"  \
             --image "RedHat:RHEL-HA:7.6:7.6.2019062019" \
             --admin-username "<username>" \
             --admin-password "<adminPassword>" \
             --authentication-type all \
             --generate-ssh-keys
    done
    ```

La commande ci-dessus crée les machines virtuelles et crée un réseau virtuel par défaut pour celles-ci. Pour plus d’informations sur les différentes configurations, consultez l’article [az vm create](/cli/azure/vm).

Une fois la commande exécutée sur chaque machine virtuelle, vous devez obtenir des résultats similaires à ce qui suit :

```output
{
  "fqdns": "",
  "id": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachines/<VM1>",
  "location": "eastus2",
  "macAddress": "<Some MAC address>",
  "powerState": "VM running",
  "privateIpAddress": "<IP1>",
  "publicIpAddress": "",
  "resourceGroup": "<resourceGroupName>",
  "zones": ""
}
```

> [!IMPORTANT]
> L’image par défaut créée à l’aide de la commande ci-dessus crée un disque de système d’exploitation de 32 Go par défaut. Vous risquez de manquer d’espace avec cette installation par défaut. Vous pouvez ajouter le paramètre suivant à la commande `az vm create` ci-dessus pour créer un disque de système d’exploitation avec 128 Go, par exemple : `--os-disk-size-gb 128`.
>
> Vous pouvez ensuite [configurer le gestionnaire de volumes logiques](/previous-versions/azure/virtual-machines/linux/configure-lvm) si vous devez développer des volumes de dossiers pour votre installation.

### <a name="test-connection-to-the-created-vms"></a>Tester la connexion aux machines virtuelles créées

Connectez-vous à VM1 ou à d’autres machines virtuelles à l’aide de la commande suivante dans Azure Cloud Shell. Si vous ne parvenez pas à trouver les adresses IP de vos machines virtuelles, suivez ce [guide de démarrage rapide sur Azure Cloud Shell](../../../cloud-shell/quickstart.md#ssh-into-your-linux-vm).

```azurecli-interactive
ssh <username>@publicipaddress
```

Si la connexion a réussi, vous devez voir la sortie suivante qui représente le terminal Linux :

```output
[<username>@<VM1> ~]$
```

Tapez `exit` pour quitter la session SSH.

## <a name="enable-high-availability"></a>Activer la haute disponibilité

> [!IMPORTANT]
> Pour effectuer cette partie du tutoriel, vous devez disposer d’un abonnement RHEL et du module complémentaire Haute disponibilité. Si vous utilisez une image recommandée dans la section précédente, vous n’avez pas à inscrire un autre abonnement.
 
Connectez-vous à chaque nœud de machine virtuelle et suivez le guide ci-dessous pour activer la haute disponibilité. Pour plus d’informations, consultez [Activer l’abonnement haute disponibilité pour RHEL](/sql/linux/sql-server-linux-availability-group-cluster-rhel#enable-the-high-availability-subscription-for-rhel).

> [!TIP]
> Vous pouvez vous faciliter la tâche en ouvrant simultanément une session SSH sur chacune des machines virtuelles, car dans cet article, les mêmes commandes devront être exécutées sur toutes les machines virtuelles.
>
> Si vous copiez-collez plusieurs commandes `sudo` et êtes invité à entrer un mot de passe, seule une commande sera exécutée. Vous devez exécuter chaque commande séparément.


1. Exécutez les commandes suivantes sur chaque machine virtuelle pour ouvrir les ports du pare-feu Pacemaker :

    ```bash
    sudo firewall-cmd --permanent --add-service=high-availability
    sudo firewall-cmd --reload
    ```

1. Mettez à jour et installez les packages Pacemaker sur tous les nœuds à l’aide des commandes suivantes :

    > [!NOTE]
    > **nmap** est installé dans le cadre de ce bloc de commandes comme un outil permettant de rechercher des adresses IP disponibles dans votre réseau. L’installation de **nmap** n’est pas obligatoire, toutefois, cet outil vous sera utile plus tard dans ce tutoriel.

    ```bash
    sudo yum update -y
    sudo yum install -y pacemaker pcs fence-agents-all resource-agents fence-agents-azure-arm nmap
    sudo reboot
    ```

1. Définissez le mot de passe pour l’utilisateur par défaut qui est créé pendant l’installation des packages Pacemaker. Utilisez le même mot de passe sur tous les nœuds.

    ```bash
    sudo passwd hacluster
    ```

1. Utilisez la commande suivante pour ouvrir le fichier HOSTS et configurer la résolution des noms d’hôtes. Pour plus d’informations, consultez [Configurer le groupe de disponibilité](/sql/linux/sql-server-linux-availability-group-configure-ha#prerequisites) qui aborde la configuration du fichier HOSTS.

    ```
    sudo vi /etc/hosts
    ```

    Dans l’éditeur **vi**, entrez `i` pour insérer du texte, puis, sur une ligne vide, ajoutez l’**adresse IP privée** de la machine virtuelle correspondante. Ajoutez ensuite le nom de la machine virtuelle après un espace, à côté de l’adresse IP. Chaque ligne doit avoir une entrée distincte.

    ```output
    <IP1> <VM1>
    <IP2> <VM2>
    <IP3> <VM3>
    ```

    > [!IMPORTANT]
    > Nous vous recommandons d’utiliser votre **adresse IP privée**. L’utilisation de l’adresse IP publique dans cette configuration entraînera l’échec de l’installation. En outre, nous vous déconseillons d’exposer votre machine virtuelle à des réseaux externes.

    Pour quitter l’éditeur **vi**, appuyez d’abord sur la touche **Echap**, puis entrez la commande `:wq` pour écrire le fichier et quitter.

## <a name="create-the-pacemaker-cluster"></a>Créez le cluster Pacemaker

Dans cette section, nous allons activer et démarrer le service PCSD, puis configurer le cluster. Pour SQL Server sur Linux, les ressources du cluster ne sont pas créées automatiquement. Nous devrons activer et créer les ressources Pacemaker manuellement. Pour plus d’informations, consultez l’article sur la [configuration d’une instance de cluster de basculement pour RHEL](/sql/linux/sql-server-linux-shared-disk-cluster-red-hat-7-configure#install-and-configure-pacemaker-on-each-cluster-node).

### <a name="enable-and-start-pcsd-service-and-pacemaker"></a>Activer et démarrer le service pcsd et Pacemaker

1. Exécutez les commandes sur tous les nœuds. Ces commandes permettent aux nœuds de rejoindre le cluster après le redémarrage.

    ```bash
    sudo systemctl enable pcsd
    sudo systemctl start pcsd
    sudo systemctl enable pacemaker
    ``` 

1. Supprimez toutes les configurations de cluster existantes de tous les nœuds. Exécutez la commande suivante :

    ```bash
    sudo pcs cluster destroy 
    sudo systemctl enable pacemaker 
    ```

1. Sur le nœud principal, exécutez les commandes suivantes pour configurer le cluster.

    - Lors de l’exécution de la commande `pcs cluster auth` pour authentifier les nœuds de cluster, vous êtes invité à entrer un mot de passe. Entrez le mot de passe de l’utilisateur **hacluster** créé précédemment.

    **RHEL7**

    ```bash
    sudo pcs cluster auth <VM1> <VM2> <VM3> -u hacluster
    sudo pcs cluster setup --name az-hacluster <VM1> <VM2> <VM3> --token 30000
    sudo pcs cluster start --all
    sudo pcs cluster enable --all
    ```

    **RHEL8**

    Pour RHEL 8, vous devez authentifier les nœuds séparément. Quand vous y êtes invité, entrez manuellement le nom d’utilisateur et le mot de passe pour **hacluster**.

    ```bash
    sudo pcs host auth <node1> <node2> <node3>
    sudo pcs cluster setup <clusterName> <node1> <node2> <node3>
    sudo pcs cluster start --all
    sudo pcs cluster enable --all
    ```

1. Exécutez la commande suivante pour vérifier que tous les nœuds sont en ligne.

    ```bash
    sudo pcs status
    ```

   **RHEL 7** 
   
    Si tous les nœuds sont en ligne; vous devez voir une sortie similaire à celle-ci :

    ```output
    Cluster name: az-hacluster
     
    WARNINGS:
    No stonith devices and stonith-enabled is not false
     
    Stack: corosync
    Current DC: <VM2> (version 1.1.19-8.el7_6.5-c3c624ea3d) - partition with quorum
    Last updated: Fri Aug 23 18:27:57 2019
    Last change: Fri Aug 23 18:27:56 2019 by hacluster via crmd on <VM2>
     
    3 nodes configured
    0 resources configured
     
    Online: [ <VM1> <VM2> <VM3> ]
     
    No resources
     
     
    Daemon Status:
          corosync: active/enabled
          pacemaker: active/enabled
          pcsd: active/enabled
    ```
   
   **RHEL 8** 
   
    ```output
    Cluster name: az-hacluster
     
    WARNINGS:
    No stonith devices and stonith-enabled is not false
     
    Cluster Summary:
    * Stack: corosync
    * Current DC: <VM2> (version 1.1.19-8.el7_6.5-c3c624ea3d) - partition with quorum
    * Last updated: Fri Aug 23 18:27:57 2019
    * Last change: Fri Aug 23 18:27:56 2019 by hacluster via crmd on <VM2>
    * 3 nodes configured
    * 0 resource instances configured
     
   Node List:
    * Online: [ <VM1> <VM2> <VM3> ]
   
   Full List of Resources:
   * No resources
     
   Daemon Status:
          corosync: active/enabled
          pacemaker: active/enabled
          pcsd: active/enabled
    
    ```
    
1. Attribuez la valeur 3 aux votes attendus dans le cluster actif. Cette commande affecte uniquement le cluster actif et ne modifie pas les fichiers de configuration.

    Sur tous les nœuds, définissez les votes attendus avec la commande suivante :

    ```bash
    sudo pcs quorum expected-votes 3
    ```

## <a name="configure-the-fencing-agent"></a>Configurer l’agent d’isolation

Un appareil STONITH fournit un agent d’isolation. Les instructions ci-dessous ont été modifiées pour ce tutoriel. Pour plus d’informations, consultez [Créer un appareil STONITH](../../../virtual-machines/workloads/sap/high-availability-guide-rhel-pacemaker.md#create-stonith-device).
 
[Vérifiez la version de l’agent d’isolation Azure pour vous assurer qu’il a été mis à jour](../../../virtual-machines/workloads/sap/high-availability-guide-rhel-pacemaker.md#cluster-installation). Utilisez la commande suivante :

```bash
sudo yum info fence-agents-azure-arm
```

Vous devez voir une sortie similaire à celle de l’exemple ci-dessous.

```output
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
Installed Packages
Name        : fence-agents-azure-arm
Arch        : x86_64
Version     : 4.2.1
Release     : 11.el7_6.8
Size        : 28 k
Repo        : installed
From repo   : rhel-ha-for-rhel-7-server-eus-rhui-rpms
Summary     : Fence agent for Azure Resource Manager
URL         : https://github.com/ClusterLabs/fence-agents
License     : GPLv2+ and LGPLv2+
Description : The fence-agents-azure-arm package contains a fence agent for Azure instances.
```

### <a name="register-a-new-application-in-azure-active-directory"></a>Inscrire une application dans Azure Active Directory
 
 1. Accédez à https://portal.azure.com
 2. Ouvrez le [panneau Azure Active Directory](https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Properties). Accédez aux propriétés et notez l’ID de répertoire. Il s’agit de l’ID `tenant ID`.
 3. Cliquez sur [**Inscriptions des applications**](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade).
 4. Cliquez sur **Nouvelle inscription**.
 5. Entrez un **nom** tel que `<resourceGroupName>-app`, puis sélectionnez **Comptes dans cet annuaire d’organisation uniquement**.
 6. Sélectionnez le type d’application **Web**, entrez une URL de connexion (par exemple http://localhost) ), puis cliquez sur Ajouter. L’URL de connexion n’est pas utilisée et peut être n’importe quelle URL valide. Une fois terminé, cliquez sur **Register** (S’inscrire).
 7. Sélectionnez **Certificates and secrets** (Certificats et secrets), puis sélectionnez **New client secret** (Nouveau secret client).
 8. Entrez une description pour la nouvelle clé (secret client), sélectionnez **N’expire jamais** et cliquez sur **Ajouter**.
 9. Notez la valeur du secret. Cette valeur est utilisée comme mot de passe pour le principal de service.
10. Sélectionnez **Vue d’ensemble**. Notez l’ID de l’application. Cet identifiant est utilisé comme nom d’utilisateur (ID de connexion dans la procédure ci-dessous) du principal de service.
 
### <a name="create-a-custom-role-for-the-fence-agent"></a>Créer un rôle personnalisé pour l’agent d’isolation

Suivez le tutoriel [Créer un rôle personnalisé Azure à l’aide d’Azure CLI](../../../role-based-access-control/tutorial-custom-role-cli.md#create-a-custom-role).

Le fichier JSON doit se présenter ainsi :

- Remplacez `<username>` par un nom de votre choix. Cela permet d’éviter toute duplication lors de la création de cette définition de rôle.
- Remplacez `<subscriptionId>` par l’identifiant de votre abonnement Azure.

```json
{
  "Name": "Linux Fence Agent Role-<username>",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows to power-off and start virtual machines",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/start/action"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscriptionId>"
  ]
}
```

Pour ajouter le rôle, exécutez la commande suivante :

- Remplacez `<filename>` par le nom du fichier.
- Si vous exécutez la commande à partir d’un chemin autre que celui du dossier dans lequel est enregistré le fichier, incluez dans la commande le chemin du dossier où se trouve le fichier.

```bash
az role definition create --role-definition "<filename>.json"
```

Vous devez normalement voir la sortie suivante.

```output
{
  "assignableScopes": [
    "/subscriptions/<subscriptionId>"
  ],
  "description": "Allows to power-off and start virtual machines",
  "id": "/subscriptions/<subscriptionId>/providers/Microsoft.Authorization/roleDefinitions/<roleNameId>",
  "name": "<roleNameId>",
  "permissions": [
    {
      "actions": [
        "Microsoft.Compute/*/read",
        "Microsoft.Compute/virtualMachines/powerOff/action",
        "Microsoft.Compute/virtualMachines/start/action"
      ],
      "dataActions": [],
      "notActions": [],
      "notDataActions": []
    }
  ],
  "roleName": "Linux Fence Agent Role-<username>",
  "roleType": "CustomRole",
  "type": "Microsoft.Authorization/roleDefinitions"
}
```

### <a name="assign-the-custom-role-to-the-service-principal"></a>Affecter le rôle personnalisé au principal de service

Affectez au principal de service le rôle personnalisé `Linux Fence Agent Role-<username>` qui a été créé à l’étape précédente. N’utilisez plus le rôle Propriétaire !
 
1. Accédez à https://portal.azure.com
2. Ouvrez le panneau [Toutes les ressources](https://ms.portal.azure.com/#blade/HubsExtension/BrowseAll).
3. Sélectionnez la machine virtuelle du premier nœud de cluster.
4. Cliquez sur **Contrôle d’accès (IAM)** .
5. Cliquez sur **Ajouter une attribution de rôle**.
6. Sélectionnez le rôle `Linux Fence Agent Role-<username>` dans la liste **Rôle**.
7. Dans la liste **Sélectionner**, entrez le nom de l’application que vous avez créée ci-dessus (`<resourceGroupName>-app`).
8. Cliquez sur **Enregistrer**.
9. Répétez les étapes ci-dessus pour tous les nœuds de cluster.

### <a name="create-the-stonith-devices"></a>Créer les appareils STONITH

Exécutez les commandes suivantes sur le nœud 1 :

- Remplacez `<ApplicationID>` par la valeur d’ID obtenue lors de l’inscription de votre application.
- Remplacez `<servicePrincipalPassword>` par la valeur du secret client.
- Remplacez `<resourceGroupName>` par le groupe de ressources de l’abonnement que vous utilisez pour ce tutoriel.
- Remplacez `<tenantID>` et `<subscriptionId>` par ceux de votre abonnement Azure.

```bash
sudo pcs property set stonith-timeout=900
sudo pcs stonith create rsc_st_azure fence_azure_arm login="<ApplicationID>" passwd="<servicePrincipalPassword>" resourceGroup="<resourceGroupName>" tenantId="<tenantID>" subscriptionId="<subscriptionId>" power_timeout=240 pcmk_reboot_timeout=900
```

Étant donné que nous avons déjà ajouté une règle à notre pare-feu pour autoriser le service de haute disponibilité (`--add-service=high-availability`), il n’est pas nécessaire d’ouvrir les ports de pare-feu suivants sur tous les nœuds : 2224, 3121, 21064, 5405. Toutefois, si vous rencontrez des problèmes de connexion avec la haute disponibilité, utilisez la commande suivante pour ouvrir les ports qui sont associés à la haute disponibilité.

> [!TIP]
> Si vous le souhaitez, vous pouvez ajouter simultanément tous les ports de ce tutoriel pour gagner du temps. Les ports qui doivent être ouverts sont indiqués dans les sections ci-dessous. Si vous souhaitez ajouter tous les ports maintenant, ajoutez également les ports suivants : 1433 et 5022.

```bash
sudo firewall-cmd --zone=public --add-port=2224/tcp --add-port=3121/tcp --add-port=21064/tcp --add-port=5405/tcp --permanent
sudo firewall-cmd --reload
```

## <a name="install-sql-server-and-mssql-tools"></a>Installer SQL Server et mssql-tools

> [!NOTE]
> Si vous avez créé les machines virtuelles avec l’instance de SQL Server 2019 préinstallée sur RHEL8-HA, vous pouvez ignorer les étapes ci-dessous destinée à installer SQL Server et mssql-tools et commencer la section **Configurer un groupe de disponibilité** après avoir configuré le mot de passe d’administrateur système (AS) sur toutes les machines virtuelles en exécutant la commande `sudo /opt/mssql/bin/mssql-conf set-sa-password` sur toutes les machines virtuelles.

Utilisez la section ci-dessous pour installer SQL Server et mssql-tools sur les machines virtuelles. Vous pouvez choisir l’un des exemples ci-dessous pour installer SQL Server 2017 sur RHEL 7 ou SQL Server 2019 sur RHEL 8. Effectuez chacune de ces actions sur tous les nœuds. Pour plus d’informations, consultez [Installer SQL Server sur une machine virtuelle Red Hat](/sql/linux/quickstart-install-connect-red-hat).


### <a name="installing-sql-server-on-the-vms"></a>Installation de SQL Server sur les machines virtuelles

Exécutez les commandes suivantes pour installer SQL Server :

**RHEL 7 avec SQL Server 2017** 

```bash
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/7/mssql-server-2017.repo
sudo yum install -y mssql-server
sudo /opt/mssql/bin/mssql-conf setup
sudo yum install mssql-server-ha
```

**RHEL 8 avec SQL Server 2019** 

```bash
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-2019.repo
sudo yum install -y mssql-server
sudo /opt/mssql/bin/mssql-conf setup
sudo yum install mssql-server-ha
```
### <a name="open-firewall-port-1433-for-remote-connections"></a>Ouvrir le port de pare-feu 1433 pour les connexions distantes

Vous devez ouvrir le port 1433 sur la machine virtuelle pour vous connecter à distance. Utilisez les commandes suivantes pour ouvrir le port 1433 dans le pare-feu de chaque machine virtuelle :

```bash
sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent
sudo firewall-cmd --reload
```

### <a name="installing-sql-server-command-line-tools"></a>Installation des outils en ligne de commande SQL Server

Exécutez les commandes suivantes pour installer les outils en ligne de commande SQL Server : Pour plus d’informations, consultez [Installer les outils en ligne de commande SQL Server](/sql/linux/quickstart-install-connect-red-hat#tools).

**RHEL 7** 

```bash
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/7/prod.repo
sudo yum install -y mssql-tools unixODBC-devel
```

**RHEL 8** 

```bash
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/8/prod.repo
sudo yum install -y mssql-tools unixODBC-devel
```
 
> [!NOTE] 
> Par souci pratique, ajoutez /opt/mssql-tools/bin/ à votre variable d’environnement PATH. Vous pourrez ainsi exécuter les outils sans spécifier le chemin complet. Exécutez les commandes suivantes afin de modifier la variable PATH pour les sessions de connexion et les sessions interactives ou sans connexion :</br></br>
`echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile`</br>
`echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc`</br>
`source ~/.bashrc`


### <a name="check-the-status-of-the-sql-server"></a>Vérifier l’état de SQL Server

Une fois que vous avez terminé la configuration, vous pouvez vérifier l’état de SQL Server et vérifier qu’il est bien en cours d’exécution :

```bash
systemctl status mssql-server --no-pager
```

Vous devez normalement voir la sortie suivante.

```output
● mssql-server.service - Microsoft SQL Server Database Engine
   Loaded: loaded (/usr/lib/systemd/system/mssql-server.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2019-12-05 17:30:55 UTC; 20min ago
     Docs: https://docs.microsoft.com/en-us/sql/linux
 Main PID: 11612 (sqlservr)
   CGroup: /system.slice/mssql-server.service
           ├─11612 /opt/mssql/bin/sqlservr
           └─11640 /opt/mssql/bin/sqlservr
```

## <a name="configure-an-availability-group"></a>Configurer un groupe de disponibilité

Effectuez les étapes suivantes pour configurer un groupe de disponibilité AlwaysOn SQL Server sur vos machines virtuelles. Pour plus d’informations, consultez [Configurer des groupes de disponibilité AlwaysOn SQL Server pour la haute disponibilité sur Linux](/sql/linux/sql-server-linux-availability-group-configure-ha).

### <a name="enable-always-on-availability-groups-and-restart-mssql-server"></a>Activer les groupes de disponibilité AlwaysOn et redémarrer mssql-server

Activez les groupes de disponibilité AlwaysOn sur chaque nœud qui héberge une instance SQL Server. Ensuite, redémarrez mssql-server. Exécutez le script suivant :

```
sudo /opt/mssql/bin/mssql-conf set hadr.hadrenabled 1
sudo systemctl restart mssql-server
```

### <a name="create-a-certificate"></a>Créer un certificat

L’authentification Active Directory n’est pas prise en charge sur le point de terminaison du groupe de disponibilité. Par conséquent, nous devons utiliser un certificat pour le chiffrement du point de terminaison du groupe de disponibilité.

1. Connectez-vous à **tous les nœuds** à l’aide de SQL Server Management Studio (SSMS) ou de SQL CMD. Exécutez les commandes suivantes pour activer une session AlwaysOn_health et créer une clé principale :

    > [!IMPORTANT]
    > Si vous vous connectez à distance à votre instance SQL Server, le port 1433 doit être ouvert dans votre pare-feu. Vous devez également autoriser les connexions entrantes sur le port 1433 de votre groupe de sécurité réseau pour chaque machine virtuelle. Pour plus d’informations sur la création d’une règle de sécurité de trafic entrant, consultez [Créer une règle de sécurité](../../../virtual-network/manage-network-security-group.md#create-a-security-rule).

    - Remplacez `<Master_Key_Password>` par votre propre mot de passe.


    ```sql
    ALTER EVENT SESSION  AlwaysOn_health ON SERVER WITH (STARTUP_STATE=ON);
    GO
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<Master_Key_Password>';
    ```

 
1. Connectez-vous au réplica principal à l’aide de SSMS ou de SQL CMD. Les commandes ci-dessous créent un certificat dans `/var/opt/mssql/data/dbm_certificate.cer` et une clé privée dans `var/opt/mssql/data/dbm_certificate.pvk` sur votre réplica de SQL Server principal :

    - Remplacez `<Private_Key_Password>` par votre propre mot de passe.
    
    ```sql
    CREATE CERTIFICATE dbm_certificate WITH SUBJECT = 'dbm';
    GO
    
    BACKUP CERTIFICATE dbm_certificate
       TO FILE = '/var/opt/mssql/data/dbm_certificate.cer'
       WITH PRIVATE KEY (
               FILE = '/var/opt/mssql/data/dbm_certificate.pvk',
               ENCRYPTION BY PASSWORD = '<Private_Key_Password>'
           );
    GO
    ```

Quittez la session SQL CMD en exécutant la commande `exit`, puis revenez à votre session SSH.
 
### <a name="copy-the-certificate-to-the-secondary-replicas-and-create-the-certificates-on-the-server"></a>Copier le certificat sur les réplicas secondaires et créer les certificats sur le serveur

1. Copiez les deux fichiers qui ont été créés au même emplacement sur tous les serveurs qui hébergeront les réplicas de disponibilité.
 
    Sur le serveur principal, exécutez la commande `scp` suivante pour copier le certificat sur les serveurs cibles :

    - Remplacez `<username>` et `<VM2>` par le nom de l’utilisateur et le nom de la machine virtuelle cible que vous utilisez.
    - Exécutez cette commande pour tous les réplicas secondaires.

    > [!NOTE]
    > Vous n’êtes pas obligé d’exécuter `sudo -i`, qui vous indique l’environnement racine. Vous pouvez simplement exécuter la commande `sudo` devant chaque commande, comme nous l’avons fait précédemment dans ce tutoriel.

    ```bash
    # The below command allows you to run commands in the root environment
    sudo -i
    ```

    ```bash
    scp /var/opt/mssql/data/dbm_certificate.* <username>@<VM2>:/home/<username>
    ```

1. Sur le serveur cible, exécutez la commande suivante :

    - Remplacez `<username>` par votre ID d’utilisateur.
    - La commande `mv` permet de déplacer les fichiers ou le répertoire.
    - La commande `chown` est utilisée pour modifier le propriétaire ainsi que le groupe de fichiers, de répertoires ou de liens.
    - Exécutez ces commandes pour tous les réplicas secondaires.

    ```bash
    sudo -i
    mv /home/<username>/dbm_certificate.* /var/opt/mssql/data/
    cd /var/opt/mssql/data
    chown mssql:mssql dbm_certificate.*
    ```

1. Le script Transact-SQL suivant crée un certificat à partir de la sauvegarde que vous avez créée sur le réplica SQL Server principal. Mettez à jour le script avec des mots de passe forts. Le mot de passe de déchiffrement est le même que celui que vous avez utilisé pour créer le fichier .pvk à l’étape précédente. Pour créer le certificat, exécutez le script suivant à l’aide de SQL CMD ou de SSMS sur tous les serveurs secondaires :

    ```sql
    CREATE CERTIFICATE dbm_certificate
        FROM FILE = '/var/opt/mssql/data/dbm_certificate.cer'
        WITH PRIVATE KEY (
        FILE = '/var/opt/mssql/data/dbm_certificate.pvk',
        DECRYPTION BY PASSWORD = '<Private_Key_Password>'
                );
    GO
    ```

### <a name="create-the-database-mirroring-endpoints-on-all-replicas"></a>Créer les points de terminaison de mise en miroir de bases de données sur tous les réplicas

Exécutez le script suivant sur toutes les instances SQL Server à l’aide de SQL CMD ou de SSMS :

```sql
CREATE ENDPOINT [Hadr_endpoint]
    AS TCP (LISTENER_PORT = 5022)
    FOR DATABASE_MIRRORING (
    ROLE = ALL,
    AUTHENTICATION = CERTIFICATE dbm_certificate,
ENCRYPTION = REQUIRED ALGORITHM AES
);
GO

ALTER ENDPOINT [Hadr_endpoint] STATE = STARTED;
GO
```

### <a name="create-the-availability-group"></a>Créez le groupe de disponibilité

Connectez-vous à l’instance SQL Server qui héberge le réplica principal à l’aide de SQL CMD ou de SSMS. Exécutez la commande suivante pour créer le groupe de disponibilité :

- Remplacez `ag1` par le nom de groupe de disponibilité de votre choix.
- Remplacez les valeurs `<VM1>`, `<VM2>` et `<VM3>` par les noms des instances SQL Server qui hébergent les réplicas.

```sql
CREATE AVAILABILITY GROUP [ag1]
     WITH (DB_FAILOVER = ON, CLUSTER_TYPE = EXTERNAL)
     FOR REPLICA ON
         N'<VM1>'
          WITH (
             ENDPOINT_URL = N'tcp://<VM1>:5022',
             AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
             FAILOVER_MODE = EXTERNAL,
             SEEDING_MODE = AUTOMATIC
             ),
         N'<VM2>'
          WITH (
             ENDPOINT_URL = N'tcp://<VM2>:5022',
             AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
             FAILOVER_MODE = EXTERNAL,
             SEEDING_MODE = AUTOMATIC
             ),
         N'<VM3>'
         WITH(
            ENDPOINT_URL = N'tcp://<VM3>:5022',
            AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
            FAILOVER_MODE = EXTERNAL,
            SEEDING_MODE = AUTOMATIC
            );
GO

ALTER AVAILABILITY GROUP [ag1] GRANT CREATE ANY DATABASE;
GO
```

### <a name="create-a-sql-server-login-for-pacemaker"></a>Créer une connexion SQL Server pour Pacemaker

Sur toutes les instances SQL Server, créez un compte de connexion SQL Server pour Pacemaker. Le code Transact-SQL ci-dessous a pour effet de créer un compte de connexion.

- Remplacez `<password>` par votre propre mot de passe complexe.

```sql
USE [master]
GO

CREATE LOGIN [pacemakerLogin] with PASSWORD= N'<password>';
GO

ALTER SERVER ROLE [sysadmin] ADD MEMBER [pacemakerLogin];
GO
```

Sur toutes les instances SQL Server, enregistrez les informations d’identification utilisées pour le compte de connexion SQL Server. 

1. Créez le fichier :

    ```bash
    sudo vi /var/opt/mssql/secrets/passwd
    ```

1. Ajoutez les deux lignes suivantes au fichier :

    ```bash
    pacemakerLogin
    <password>
    ```

    Pour quitter l’éditeur **vi**, appuyez d’abord sur la touche **Echap**, puis entrez la commande `:wq` pour écrire le fichier et quitter.

1. Rendez le fichier lisible uniquement par la racine :

    ```bash
    sudo chown root:root /var/opt/mssql/secrets/passwd
    sudo chmod 400 /var/opt/mssql/secrets/passwd
    ```

### <a name="join-secondary-replicas-to-the-availability-group"></a>Joindre les réplicas secondaires au groupe de disponibilité

1. Pour joindre les réplicas secondaires au groupe de disponibilité, vous devez ouvrir le port 5022 du pare-feu pour tous les serveurs. Exécutez la commande suivante dans votre session SSH :

    ```bash
    sudo firewall-cmd --zone=public --add-port=5022/tcp --permanent
    sudo firewall-cmd --reload
    ```

1. Sur vos réplicas secondaires, exécutez les commandes suivantes pour les joindre au groupe de disponibilité :

    ```sql
    ALTER AVAILABILITY GROUP [ag1] JOIN WITH (CLUSTER_TYPE = EXTERNAL);
    GO

    ALTER AVAILABILITY GROUP [ag1] GRANT CREATE ANY DATABASE;
    GO
    ```

1. Exécutez le script Transact-SQL suivant sur le réplica principal et sur chaque réplica secondaire :

    ```sql
    GRANT ALTER, CONTROL, VIEW DEFINITION ON AVAILABILITY GROUP::ag1 TO pacemakerLogin;
    GO
    
    GRANT VIEW SERVER STATE TO pacemakerLogin;
    GO
    ```

1. Une fois les réplicas secondaires joints, vous pouvez les afficher dans l’Explorateur d’objets de SSMS en développant le nœud **Haute disponibilité Always On** :

    ![La capture d’écran présente les réplicas de disponibilité principaux et secondaires.](./media/rhel-high-availability-stonith-tutorial/availability-group-joined.png)

### <a name="add-a-database-to-the-availability-group"></a>Ajouter une base de données au groupe de disponibilité

Nous allons suivre les instructions de l’article concernant la [configuration d’un groupe de disponibilité lors de l’ajout d’une de base de données](/sql/linux/sql-server-linux-availability-group-configure-ha#add-a-database-to-the-availability-group).

Les commandes Transact-SQL suivantes sont utilisées dans cette étape. Exécutez les commandes suivantes sur le réplica principal :

```sql
CREATE DATABASE [db1]; -- creates a database named db1
GO

ALTER DATABASE [db1] SET RECOVERY FULL; -- set the database in full recovery mode
GO

BACKUP DATABASE [db1] -- backs up the database to disk
   TO DISK = N'/var/opt/mssql/data/db1.bak';
GO

ALTER AVAILABILITY GROUP [ag1] ADD DATABASE [db1]; -- adds the database db1 to the AG
GO
```

### <a name="verify-that-the-database-is-created-on-the-secondary-servers"></a>Vérifier que la base de données est créée sur les serveurs secondaires

Sur chaque réplica SQL Server secondaire, exécutez la requête suivante pour déterminer si la base de données db1 a été créée et si son état indique « SYNCHRONIZED » :

```
SELECT * FROM sys.databases WHERE name = 'db1';
GO
SELECT DB_NAME(database_id) AS 'database', synchronization_state_desc FROM sys.dm_hadr_database_replica_states;
```

Si `synchronization_state_desc` indique SYNCHRONIZED pour `db1`, cela signifie que les réplicas sont synchronisés. Les réplicas secondaires indiquent `db1` dans le réplica principal.

## <a name="create-availability-group-resources-in-the-pacemaker-cluster"></a>Créer les ressources de groupe de disponibilité dans le cluster Pacemaker

Nous allons suivre le guide permettant de [créer les ressources de groupe de disponibilité dans le cluster Pacemaker](/sql/linux/sql-server-linux-create-availability-group#create-the-availability-group-resources-in-the-pacemaker-cluster-external-only).

> [!NOTE]
> Cet article contient des références au terme esclave, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

### <a name="create-the-ag-cluster-resource"></a>Créer la ressource de cluster du groupe de disponibilité

1. Utilisez l’une des commandes suivantes en fonction de l’environnement choisi précédemment pour créer la ressource `ag_cluster` dans le groupe de disponibilité `ag1`.

    **RHEL 7**
  
    ```bash
    sudo pcs resource create ag_cluster ocf:mssql:ag ag_name=ag1 meta failure-timeout=30s master notify=true
    ```

    **RHEL 8**
  
    ```bash
    sudo pcs resource create ag_cluster ocf:mssql:ag ag_name=ag1 meta failure-timeout=30s promotable notify=true
    ```

2. Vérifiez que votre ressource est bien en ligne avant d’exécuter la commande suivante :

    ```bash
    sudo pcs resource
    ```

    Vous devez normalement voir la sortie suivante.
    
    **RHEL 7** 
    
    ```output
    [<username>@VM1 ~]$ sudo pcs resource
    Master/Slave Set: ag_cluster-master [ag_cluster]
    Masters: [ <VM1> ]
    Slaves: [ <VM2> <VM3> ]
    ```
    
    **RHEL 8** 
    
    ```output
    [<username>@VM1 ~]$ sudo pcs resource
    * Clone Set: ag_cluster-clone [ag_cluster] (promotable):
    * ag_cluster             (ocf::mssql:ag) :            Slave VMrhel3 (Monitoring) 
    * ag_cluster             (ocf::mssql:ag) :            Master VMrhel1 (Monitoring)
    * ag_cluster             (ocf::mssql:ag) :            Slave VMrhel2 (Monitoring)
    ```


### <a name="create-a-virtual-ip-resource"></a>Créer une ressource d’adresse IP virtuelle

1. Pour créer une ressource d’adresse IP virtuelle, utilisez une adresse IP statique disponible dans votre réseau. Vous pouvez en trouver une à l’aide de l’outil de commande `nmap`.

    ```bash
    nmap -sP <IPRange>
    # For example: nmap -sP 10.0.0.*
    # The above will scan for all IP addresses that are already occupied in the 10.0.0.x space.
    ```

2. Affectez la valeur false à la propriété **stonith-enabled**.

    ```bash
    sudo pcs property set stonith-enabled=false
    ```

3. Créez la ressource d’adresse IP virtuelle à l’aide de la commande suivante :

    - Remplacez la valeur `<availableIP>` ci-dessous par une adresse IP inutilisée.

    ```bash
    sudo pcs resource create virtualip ocf:heartbeat:IPaddr2 ip=<availableIP>
    ```

### <a name="add-constraints"></a>Ajouter des contraintes

1. Pour vous assurer que l’adresse IP et la ressource de disponibilité de groupe s’exécutent sur le même nœud, une contrainte de colocalisation doit être configurée. Exécutez la commande suivante :

   **RHEL 7**
  
    ```bash
    sudo pcs constraint colocation add virtualip ag_cluster-master INFINITY with-rsc-role=Master
    ```

   **RHEL 8**
   
    ```bash
     sudo pcs constraint colocation add virtualip with master ag_cluster-clone INFINITY with-rsc-role=Master
    ```
  
2. Créez une contrainte de classement pour vous assurer que la ressource de groupe de disponibilité est active et en cours d’exécution avant l’adresse IP. Alors que la contrainte de colocalisation implique une contrainte de classement, elle l’applique.

   **RHEL 7**
   
    ```bash
    sudo pcs constraint order promote ag_cluster-master then start virtualip
    ```

   **RHEL 8**
   
    ```bash
    sudo pcs constraint order promote ag_cluster-clone then start virtualip
    ```
  
3. Pour vérifier les contraintes, exécutez la commande suivante :

    ```bash
    sudo pcs constraint list --full
    ```

    Vous devez normalement voir la sortie suivante.
    
    **RHEL 7**

    ```
    Location Constraints:
    Ordering Constraints:
          promote ag_cluster-master then start virtualip (kind:Mandatory) (id:order-ag_cluster-master-virtualip-mandatory)
    Colocation Constraints:
          virtualip with ag_cluster-master (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-master-INFINITY)
    Ticket Constraints:
    ```
    
    **RHEL 8**
    
    ```output
    Location Constraints:
    Ordering Constraints:
            promote ag_cluster-clone then start virtualip (kind:Mandatory) (id:order-ag_cluster-clone-virtualip-mandatory)
    Colocation Constraints:
            virtualip with ag_cluster-clone (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-clone-INFINITY)
    Ticket Constraints:
    ```

### <a name="re-enable-stonith"></a>Réactiver stonith

Nous sommes prêts pour le test. Réactivez stonith dans le cluster en exécutant la commande suivante sur le nœud 1 :

```bash
sudo pcs property set stonith-enabled=true
```

### <a name="check-cluster-status"></a>Vérification du statut du cluster

Vous pouvez vérifier l’état de vos ressources de cluster à l’aide de la commande suivante :

```output
[<username>@VM1 ~]$ sudo pcs status
Cluster name: az-hacluster
Stack: corosync
Current DC: <VM3> (version 1.1.19-8.el7_6.5-c3c624ea3d) - partition with quorum
Last updated: Sat Dec  7 00:18:38 2019
Last change: Sat Dec  7 00:18:02 2019 by root via cibadmin on VM1

3 nodes configured
5 resources configured

Online: [ <VM1> <VM2> <VM3> ]

Full list of resources:

 Master/Slave Set: ag_cluster-master [ag_cluster]
     Masters: [ <VM2> ]
     Slaves: [ <VM1> <VM3> ]
 virtualip      (ocf::heartbeat:IPaddr2):       Started <VM2>
 rsc_st_azure   (stonith:fence_azure_arm):      Started <VM1>

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

## <a name="test-failover"></a>Test de basculement

Pour vérifier que la configuration a réussi, nous allons tester un basculement. Pour plus d’informations, consultez [Basculement du groupe de disponibilité AlwaysOn sur Linux](/sql/linux/sql-server-linux-availability-group-failover-ha).

1. Exécutez la commande suivante pour basculer manuellement le réplica principal vers `<VM2>`. Remplacez `<VM2>` par le nom du serveur.

   **RHEL 7**
   
    ```bash
    sudo pcs resource move ag_cluster-master <VM2> --master
    ```

   **RHEL 8**
   
    ```bash
    sudo pcs resource move ag_cluster-clone <VM2> --master
    ```

   Vous pouvez aussi spécifier une option supplémentaire de sorte que la contrainte temporaire créée pour déplacer la ressource vers un nœud souhaité soit désactivée automatiquement et que vous n’ayez pas à effectuer les étapes 2 et 3 ci-dessous.

   **RHEL 7**

    ```bash
    sudo pcs resource move ag_cluster-master <VM2> --master lifetime=30S
    ```

   **RHEL 8**

    ```bash
    sudo pcs resource move ag_cluster-clone <VM2> --master lifetime=30S
    ```

   Pour automatiser les étapes 2 et 3 ci-dessous, qui éliminent la contrainte temporaire de la commande de déplacement de ressource, vous pouvez aussi combiner plusieurs commandes sur une même ligne. 

   **RHEL 7**

    ```bash
    sudo pcs resource move ag_cluster-master <VM2> --master && sleep 30 && pcs resource clear ag_cluster-master
    ```

   **RHEL 8**

    ```bash
    sudo pcs resource move ag_cluster-clone <VM2> --master && sleep 30 && pcs resource clear ag_cluster-clone
    ```
    
2. Si vous vérifiez à nouveau vos contraintes, vous verrez qu’une autre contrainte a été ajoutée en raison du basculement manuel :
    
    **RHEL 7**
    
    ```output
    [<username>@VM1 ~]$ sudo pcs constraint list --full
    Location Constraints:
          Resource: ag_cluster-master
            Enabled on: VM2 (score:INFINITY) (role: Master) (id:cli-prefer-ag_cluster-master)
    Ordering Constraints:
            promote ag_cluster-master then start virtualip (kind:Mandatory) (id:order-ag_cluster-master-virtualip-mandatory)
    Colocation Constraints:
            virtualip with ag_cluster-master (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-master-INFINITY)
    Ticket Constraints:
    ```

    **RHEL 8**
    
    ```output
    [<username>@VM1 ~]$ sudo pcs constraint list --full
    Location Constraints:
          Resource: ag_cluster-master
            Enabled on: VM2 (score:INFINITY) (role: Master) (id:cli-prefer-ag_cluster-clone)
    Ordering Constraints:
            promote ag_cluster-clone then start virtualip (kind:Mandatory) (id:order-ag_cluster-clone-virtualip-mandatory)
    Colocation Constraints:
            virtualip with ag_cluster-clone (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-clone-INFINITY)
    Ticket Constraints:
    ```
    
3. Supprimez la contrainte avec l’ID `cli-prefer-ag_cluster-master` à l’aide de la commande suivante :

    **RHEL 7**
    
    ```bash
    sudo pcs constraint remove cli-prefer-ag_cluster-master
    ```

    **RHEL 8**
    
    ```bash
    sudo pcs constraint remove cli-prefer-ag_cluster-clone
    ```

1. Vérifiez vos ressources de cluster à l’aide de la commande `sudo pcs resource`. Vous devriez voir que l’instance principale est désormais `<VM2>`.

    ```output
    [<username>@<VM1> ~]$ sudo pcs resource
    Master/Slave Set: ag_cluster-master [ag_cluster]
         ag_cluster (ocf::mssql:ag):        FAILED <VM1> (Monitoring)
         Masters: [ <VM2> ]
         Slaves: [ <VM3> ]
    virtualip      (ocf::heartbeat:IPaddr2):       Started <VM2>
    [<username>@<VM1> ~]$ sudo pcs resource
    Master/Slave Set: ag_cluster-master [ag_cluster]
         Masters: [ <VM2> ]
         Slaves: [ <VM1> <VM3> ]
    virtualip      (ocf::heartbeat:IPaddr2):       Started <VM2>
    ```

## <a name="test-fencing"></a>Tester l’isolation

Vous pouvez tester STONITH en exécutant la commande suivante. Essayez d’exécuter la commande ci-dessous à partir de `<VM1>` pour `<VM3>`.

```bash
sudo pcs stonith fence <VM3> --debug
```

> [!NOTE]
> Par défaut, l’action d’isolation déconnecte puis reconnecte le nœud. Si vous souhaitez uniquement déconnecter le nœud, utilisez l’option `--off` dans la commande.

Vous devez obtenir la sortie suivante :

```output
[<username>@<VM1> ~]$ sudo pcs stonith fence <VM3> --debug
Running: stonith_admin -B <VM3>
Return Value: 0
--Debug Output Start--
--Debug Output End--
 
Node: <VM3> fenced
```
Pour plus d’informations sur le test d’un appareil d’isolation, consultez l’article [Red Hat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/s1-stonithtest-haar) suivant.

## <a name="next-steps"></a>Étapes suivantes

Pour utiliser un écouteur de groupe de disponibilité pour vos instances SQL Server, vous devez créer et configurer un équilibreur de charge.

> [!div class="nextstepaction"]
> [Tutoriel : Configurer un écouteur de groupe de disponibilité pour SQL Server sur des machines virtuelles RHEL dans Azure](rhel-high-availability-listener-tutorial.md)
