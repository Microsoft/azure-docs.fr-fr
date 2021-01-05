---
title: Créer et gérer une machine virtuelle Windows dans Azure à l’aide de Python
description: Apprenez à utiliser Python pour créer et gérer une machine virtuelle Windows dans Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure
ms.topic: how-to
ms.date: 06/22/2017
ms.author: cynthn
ms.custom: devx-track-python
ms.openlocfilehash: a7117d30eb1774753f21e82ad5f812a7309d8386
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483094"
---
# <a name="create-and-manage-windows-vms-in-azure-using-python"></a>Créer et gérer des machines virtuelles Windows dans Azure à l’aide de Python

Une [machine virtuelle Azure](overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) a besoin de plusieurs ressources de prise en charge Azure. Cet article décrit la création, la gestion et la suppression de ressources de machine virtuelle à l’aide de Python. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un projet Visual Studio
> * Installer des packages
> * Créer des informations d’identification
> * Créer des ressources
> * Effectuer d’autres tâches de gestion
> * Supprimer des ressources
> * Exécution de l'application

Ces étapes prennent environ 20 minutes.

## <a name="create-a-visual-studio-project"></a>Créer un projet Visual Studio

1. Si vous ne l’avez pas déjà fait, installez [Visual Studio](/visualstudio/install/install-visual-studio). Dans la page Charges de travail, sélectionnez **Développement Python**, puis cliquez sur **Installer**. Dans le résumé, vous pouvez voir que **Python 3 64 bits (3.6.0)** est automatiquement sélectionné pour vous. Si vous avez déjà installé Visual Studio, vous pouvez ajouter la charge de travail Python en utilisant le Lanceur de Visual Studio.
2. Après avoir installé et démarré Visual Studio, cliquez sur **Fichier** > **Nouveau** > **Projet**.
3. Cliquez sur **Modèles** > **Python** > **Application Python**, entrez *monProjetPython* comme nom pour le projet, sélectionnez l’emplacement du projet, puis cliquez sur **OK**.

## <a name="install-packages"></a>Installer des packages

1. Dans l’Explorateur de solutions, sous *monProjetPython*, cliquez avec le bouton droit sur **Environnements Python**, puis sélectionnez **Ajouter un environnement virtuel**.
2. Dans l’écran Ajouter un environnement virtuel, acceptez le nom par défaut *env*, assurez-vous que *Python 3.6 (64 bits)* est sélectionné pour l’interpréteur de base, puis cliquez sur **Créer**.
3. Cliquez avec le bouton droit sur l’environnement *env* que vous avez créé, cliquez sur **Installer le package Python**, entrez *azure* dans la zone de recherche, puis appuyez sur Entrée.

Dans les fenêtres de sortie, vous devriez voir que les packages Azure ont été correctement installés. 

## <a name="create-credentials"></a>Créer des informations d’identification

Avant de commencer cette étape, assurez-vous que vous disposez d’un [principal du service Active Directory](../../active-directory/develop/howto-create-service-principal-portal.md). Vous devez également enregistrer l’ID d’application, la clé d’authentification et l’ID de client dont vous aurez besoin dans une étape ultérieure.

1. Ouvrez le fichier *monProjetPython.py* qui a été créé, puis ajoutez ce code pour permettre à votre application de s’exécuter :

    ```python
    if __name__ == "__main__":
    ```

2. Pour importer le code nécessaire, ajoutez les instructions suivantes au début du fichier .py :

    ```python
    from azure.common.credentials import ServicePrincipalCredentials
    from azure.mgmt.resource import ResourceManagementClient
    from azure.mgmt.compute import ComputeManagementClient
    from azure.mgmt.network import NetworkManagementClient
    from azure.mgmt.compute.models import DiskCreateOption
    ```

3. Ensuite dans le fichier .py, ajoutez des variables après les instructions d’importation pour spécifier des valeurs communes utilisées dans le code :
   
    ```
    SUBSCRIPTION_ID = 'subscription-id'
    GROUP_NAME = 'myResourceGroup'
    LOCATION = 'westus'
    VM_NAME = 'myVM'
    ```

    Remplacez **subscription-id** par votre identificateur d’abonnement.

4. Pour créer les informations d’identification Active Directory dont vous avez besoin pour effectuer des demandes, ajoutez la fonction suivant après les variables dans le fichier .py :

    ```python
    def get_credentials():
        credentials = ServicePrincipalCredentials(
            client_id = 'application-id',
            secret = 'authentication-key',
            tenant = 'tenant-id'
        )

        return credentials
    ```

    Remplacez **application-id**, **authentication-key** et **tenant-id** par les valeurs que vous avez collectées précédemment lors de la création de votre principal du service Azure Active Directory.

5. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :

    ```python
    credentials = get_credentials()
    ```

## <a name="create-resources"></a>Créer des ressources
 
### <a name="initialize-management-clients"></a>Initialiser des clients de gestion

Des clients de gestion sont nécessaires pour créer et gérer des ressources à l’aide du Kit de développement logiciel (SDK) Python dans Azure. Pour créer les clients de gestion, ajoutez le code suivant sous l’instruction **si** à la fin du fichier .py :

```python
resource_group_client = ResourceManagementClient(
    credentials,
    SUBSCRIPTION_ID
)
network_client = NetworkManagementClient(
    credentials,
    SUBSCRIPTION_ID
)
compute_client = ComputeManagementClient(
    credentials,
    SUBSCRIPTION_ID
)
```

### <a name="create-the-vm-and-supporting-resources"></a>Créer la machine virtuelle et les ressources de prise en charge

Toutes les ressources doivent être contenues dans un [groupe de ressources](../../azure-resource-manager/management/overview.md).

1. Pour créer un groupe de ressources, ajoutez la fonction suivante après les variables dans le fichier .py :

    ```python
    def create_resource_group(resource_group_client):
        resource_group_params = { 'location':LOCATION }
        resource_group_result = resource_group_client.resource_groups.create_or_update(
            GROUP_NAME, 
            resource_group_params
        )
    ```

2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :

    ```python
    create_resource_group(resource_group_client)
    input('Resource group created. Press enter to continue...')
    ```

Les [groupes à haute disponibilité](tutorial-availability-sets.md) facilitent la maintenance des machines virtuelles utilisées par votre application.

1. Pour créer un groupe à haute disponibilité, ajoutez la fonction suivante après les variables dans le fichier .py :
   
    ```python
    def create_availability_set(compute_client):
        avset_params = {
            'location': LOCATION,
            'sku': { 'name': 'Aligned' },
            'platform_fault_domain_count': 3
        }
        availability_set_result = compute_client.availability_sets.create_or_update(
            GROUP_NAME,
            'myAVSet',
            avset_params
        )
    ```

2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :

    ```python
    create_availability_set(compute_client)
    print("------------------------------------------------------")
    input('Availability set created. Press enter to continue...')
    ```

Une [adresse IP publique](../../virtual-network/public-ip-addresses.md) est nécessaire pour communiquer avec la machine virtuelle.

1. Pour créer une adresse IP publique pour la machine virtuelle, ajoutez la fonction suivante après les variables dans le fichier .py :

    ```python
    def create_public_ip_address(network_client):
        public_ip_addess_params = {
            'location': LOCATION,
            'public_ip_allocation_method': 'Dynamic'
        }
        creation_result = network_client.public_ip_addresses.create_or_update(
            GROUP_NAME,
            'myIPAddress',
            public_ip_addess_params
        )

        return creation_result.result()
    ```

2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :

    ```python
    creation_result = create_public_ip_address(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

Un machine virtuelle doit figurer dans un sous-réseau d’un [réseau virtuel](../../virtual-network/virtual-networks-overview.md).

1. Pour créer un réseau virtuel, ajoutez la fonction suivante après les variables dans le fichier .py :

    ```python
    def create_vnet(network_client):
        vnet_params = {
            'location': LOCATION,
            'address_space': {
                'address_prefixes': ['10.0.0.0/16']
            }
        }
        creation_result = network_client.virtual_networks.create_or_update(
            GROUP_NAME,
            'myVNet',
            vnet_params
        )
        return creation_result.result()
    ```

2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :
   
    ```python
    creation_result = create_vnet(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

3. Pour ajouter un sous-réseau au réseau virtuel, ajoutez la fonction suivante après les variables dans le fichier .py :
    
    ```python
    def create_subnet(network_client):
        subnet_params = {
            'address_prefix': '10.0.0.0/24'
        }
        creation_result = network_client.subnets.create_or_update(
            GROUP_NAME,
            'myVNet',
            'mySubnet',
            subnet_params
        )

        return creation_result.result()
    ```
        
4. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :
   
    ```python
    creation_result = create_subnet(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

Une machine virtuelle a besoin d’une interface réseau pour communiquer sur le réseau virtuel.

1. Pour créer une interface réseau, ajoutez la fonction suivante après les variables dans le fichier .py :

    ```python
    def create_nic(network_client):
        subnet_info = network_client.subnets.get(
            GROUP_NAME, 
            'myVNet', 
            'mySubnet'
        )
        publicIPAddress = network_client.public_ip_addresses.get(
            GROUP_NAME,
            'myIPAddress'
        )
        nic_params = {
            'location': LOCATION,
            'ip_configurations': [{
                'name': 'myIPConfig',
                'public_ip_address': publicIPAddress,
                'subnet': {
                    'id': subnet_info.id
                }
            }]
        }
        creation_result = network_client.network_interfaces.create_or_update(
            GROUP_NAME,
            'myNic',
            nic_params
        )

        return creation_result.result()
    ```

2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :

    ```python
    creation_result = create_nic(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

Maintenant que vous avez créé l’ensemble des ressources de prise en charge, vous pouvez créer une machine virtuelle.

1. Pour créer la machine virtuelle, ajoutez la fonction suivante après les variables dans le fichier .py :
   
    ```python
    def create_vm(network_client, compute_client):  
        nic = network_client.network_interfaces.get(
            GROUP_NAME, 
            'myNic'
        )
        avset = compute_client.availability_sets.get(
            GROUP_NAME,
            'myAVSet'
        )
        vm_parameters = {
            'location': LOCATION,
            'os_profile': {
                'computer_name': VM_NAME,
                'admin_username': 'azureuser',
                'admin_password': 'Azure12345678'
            },
            'hardware_profile': {
                'vm_size': 'Standard_DS1'
            },
            'storage_profile': {
                'image_reference': {
                    'publisher': 'MicrosoftWindowsServer',
                    'offer': 'WindowsServer',
                    'sku': '2012-R2-Datacenter',
                    'version': 'latest'
                }
            },
            'network_profile': {
                'network_interfaces': [{
                    'id': nic.id
                }]
            },
            'availability_set': {
                'id': avset.id
            }
        }
        creation_result = compute_client.virtual_machines.create_or_update(
            GROUP_NAME, 
            VM_NAME, 
            vm_parameters
        )
    
        return creation_result.result()
    ```

    > [!NOTE]
    > Ce didacticiel crée une machine virtuelle exécutant une version du système d’exploitation Windows Server. Pour en savoir plus sur la sélection d’autres images, consultez [Parcourir et sélectionner des images de machine virtuelle Azure avec Windows PowerShell et l’interface CLI Azure](../linux/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
    > 
    > 

2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :

    ```python
    creation_result = create_vm(network_client, compute_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

## <a name="perform-management-tasks"></a>Effectuer d’autres tâches de gestion

Pendant le cycle de vie d’une machine virtuelle, vous souhaiterez exécuter des tâches de gestion telles que le démarrage, l’arrêt ou la suppression d’une machine virtuelle. En outre, vous souhaiterez peut-être créer du code pour automatiser les tâches répétitives ou complexes.

### <a name="get-information-about-the-vm"></a>Obtenir des informations sur la machine virtuelle

1. Pour obtenir plus d’informations sur la machine virtuelle, ajoutez la fonction suivante après les variables dans le fichier .py :

    ```python
    def get_vm(compute_client):
        vm = compute_client.virtual_machines.get(GROUP_NAME, VM_NAME, expand='instanceView')
        print("hardwareProfile")
        print("   vmSize: ", vm.hardware_profile.vm_size)
        print("\nstorageProfile")
        print("  imageReference")
        print("    publisher: ", vm.storage_profile.image_reference.publisher)
        print("    offer: ", vm.storage_profile.image_reference.offer)
        print("    sku: ", vm.storage_profile.image_reference.sku)
        print("    version: ", vm.storage_profile.image_reference.version)
        print("  osDisk")
        print("    osType: ", vm.storage_profile.os_disk.os_type.value)
        print("    name: ", vm.storage_profile.os_disk.name)
        print("    createOption: ", vm.storage_profile.os_disk.create_option.value)
        print("    caching: ", vm.storage_profile.os_disk.caching.value)
        print("\nosProfile")
        print("  computerName: ", vm.os_profile.computer_name)
        print("  adminUsername: ", vm.os_profile.admin_username)
        print("  provisionVMAgent: {0}".format(vm.os_profile.windows_configuration.provision_vm_agent))
        print("  enableAutomaticUpdates: {0}".format(vm.os_profile.windows_configuration.enable_automatic_updates))
        print("\nnetworkProfile")
        for nic in vm.network_profile.network_interfaces:
            print("  networkInterface id: ", nic.id)
        print("\nvmAgent")
        print("  vmAgentVersion", vm.instance_view.vm_agent.vm_agent_version)
        print("    statuses")
        for stat in vm_result.instance_view.vm_agent.statuses:
            print("    code: ", stat.code)
            print("    displayStatus: ", stat.display_status)
            print("    message: ", stat.message)
            print("    time: ", stat.time)
        print("\ndisks");
        for disk in vm.instance_view.disks:
            print("  name: ", disk.name)
            print("  statuses")
            for stat in disk.statuses:
                print("    code: ", stat.code)
                print("    displayStatus: ", stat.display_status)
                print("    time: ", stat.time)
        print("\nVM general status")
        print("  provisioningStatus: ", vm.provisioning_state)
        print("  id: ", vm.id)
        print("  name: ", vm.name)
        print("  type: ", vm.type)
        print("  location: ", vm.location)
        print("\nVM instance status")
        for stat in vm.instance_view.statuses:
            print("  code: ", stat.code)
            print("  displayStatus: ", stat.display_status)
    ```
2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :

    ```python
    get_vm(compute_client)
    print("------------------------------------------------------")
    input('Press enter to continue...')
    ```

### <a name="stop-the-vm"></a>Arrêtez la machine virtuelle.

Vous pouvez arrêter une machine virtuelle et conserver tous ses paramètres, mais continuer à être facturé, ou arrêter une machine virtuelle et la libérer. Lorsqu’une machine virtuelle est libérée, toutes les ressources qui lui sont associées sont également libérées et la facturation de la machine virtuelle prend fin.

1. Pour arrêter la machine virtuelle sans la désallouer, ajoutez la fonction suivante après les variables dans le fichier .py :

    ```python
    def stop_vm(compute_client):
        compute_client.virtual_machines.power_off(GROUP_NAME, VM_NAME)
    ```

    Si vous souhaitez désallouer la machine virtuelle, modifiez l’appel power_off à ce code :

    ```python
    compute_client.virtual_machines.deallocate(GROUP_NAME, VM_NAME)
    ```

2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :

    ```python
    stop_vm(compute_client)
    input('Press enter to continue...')
    ```

### <a name="start-the-vm"></a>Démarrer la machine virtuelle

1. Pour démarrer la machine virtuelle, ajoutez la fonction suivante après les variables dans le fichier .py :

    ```python
    def start_vm(compute_client):
        compute_client.virtual_machines.start(GROUP_NAME, VM_NAME)
    ```

2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :

    ```python
    start_vm(compute_client)
    input('Press enter to continue...')
    ```

### <a name="resize-the-vm"></a>Redimensionner la machine virtuelle

De nombreux aspects du déploiement doivent être pris en considération lors du choix d’une taille pour votre machine virtuelle. Pour plus d’informations, voir [Tailles des machines virtuelles](../sizes.md).

1. Pour modifier la taille de la machine virtuelle, ajoutez la fonction suivante après les variables dans le fichier .py :

    ```python
    def update_vm(compute_client):
        vm = compute_client.virtual_machines.get(GROUP_NAME, VM_NAME)
        vm.hardware_profile.vm_size = 'Standard_DS3'
        update_result = compute_client.virtual_machines.create_or_update(
            GROUP_NAME, 
            VM_NAME, 
            vm
        )

    return update_result.result()
    ```

2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :

    ```python
    update_result = update_vm(compute_client)
    print("------------------------------------------------------")
    print(update_result)
    input('Press enter to continue...')
    ```

### <a name="add-a-data-disk-to-the-vm"></a>Ajouter un disque de données à la machine virtuelle

Des machines virtuelles peuvent disposer d’un ou plusieurs [disques de données](../managed-disks-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) stockés en tant que disques durs virtuels.

1. Pour ajouter un disque de données à la machine virtuelle, ajoutez la fonction suivante après les variables dans le fichier .py : 

    ```python
    def add_datadisk(compute_client):
        disk_creation = compute_client.disks.create_or_update(
            GROUP_NAME,
            'myDataDisk1',
            {
                'location': LOCATION,
                'disk_size_gb': 1,
                'creation_data': {
                    'create_option': DiskCreateOption.empty
                }
            }
        )
        data_disk = disk_creation.result()
        vm = compute_client.virtual_machines.get(GROUP_NAME, VM_NAME)
        add_result = vm.storage_profile.data_disks.append({
            'lun': 1,
            'name': 'myDataDisk1',
            'create_option': DiskCreateOption.attach,
            'managed_disk': {
                'id': data_disk.id
            }
        })
        add_result = compute_client.virtual_machines.create_or_update(
            GROUP_NAME,
            VM_NAME,
            vm)

        return add_result.result()
    ```

2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :

    ```python
    add_result = add_datadisk(compute_client)
    print("------------------------------------------------------")
    print(add_result)
    input('Press enter to continue...')
    ```

## <a name="delete-resources"></a>Supprimer des ressources

Étant donné que les ressources utilisées dans Microsoft Azure vous sont facturées, il est toujours conseillé de supprimer les ressources qui ne sont plus nécessaires. Si vous souhaitez supprimer les machines virtuelles et l’ensemble des ressources de prise en charge, il vous suffit de supprimer le groupe de ressources.

1. Pour supprimer le groupe de ressources et toutes les ressources, ajoutez la fonction suivante après les variables dans le fichier .py :
   
    ```python
    def delete_resources(resource_group_client):
        resource_group_client.resource_groups.delete(GROUP_NAME)
    ```

2. Pour appeler la fonction que vous avez ajoutée précédemment, ajoutez le code suivant sous l’instruction **if** à la fin du fichier .py :
   
    ```python
    delete_resources(resource_group_client)
    ```

3. Enregistrez *monProjetPython.py*.

## <a name="run-the-application"></a>Exécution de l'application

1. Pour exécuter l’application console, cliquez sur **Démarrer** dans Visual Studio.

2. Appuyez sur **Entrée** après que l’état de chaque ressource a été retourné. Dans les informations d’état, vous devriez voir l’état d’approvisionnement **Réussi**. Une fois la machine virtuelle créée, vous pouvez supprimer toutes les ressources que vous avez créées. Avant d’appuyer sur **Entrée** pour démarrer la suppression des ressources, prenez quelques minutes pour vérifier leur création dans le portail Azure. Si le portail Azure est ouvert, il se peut que vous deviez actualiser le panneau pour afficher de nouvelles ressources.  

    L’exécution complète de cette application console devrait durer cinq minutes environ. Quelques minutes peuvent s’écouler après la fin de l’exécution de l’application avant que toutes les ressources et le groupe de ressources soient supprimés.


## <a name="next-steps"></a>Étapes suivantes

- Si vous rencontrez des problèmes de déploiement, consultez [Résolution des problèmes liés aux déploiements de groupes de ressources avec le portail Azure](../../azure-resource-manager/templates/deployment-history.md).
- En savoir plus sur la [Bibliothèque Python Azure](/python/api/overview/azure/?view=azure-python)