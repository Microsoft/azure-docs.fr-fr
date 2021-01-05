---
title: Utiliser l’API Gestion des services (Python) - Guide des fonctionnalités
description: Découvrez comment effectuer des tâches courantes de gestion des services par programme à partir de Python.
services: cloud-services
documentationcenter: python
author: tanmaygore
manager: vashan
editor: ''
ms.assetid: 61538ec0-1536-4a7e-ae89-95967fe35d73
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: python
ms.topic: article
ms.date: 05/30/2017
ms.author: tagore
ms.custom: devx-track-python
ms.openlocfilehash: ef155116904ee0d3ecab250a254010e2f7664757
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92073986"
---
# <a name="use-service-management-from-python"></a>Utiliser la gestion des services de Python
Ce guide vous explique comment effectuer des tâches courantes de gestion des services par programme à partir de Python. La classe **ServiceManagementService** du [Kit de développement logiciel (SDK) Azure pour Python](https://github.com/Azure/azure-sdk-for-python) prend en charge l’accès par programmation à une grande partie des fonctionnalités liées au management des services disponibles dans le [portail Azure][management-portal]. Vous pouvez utiliser cette fonctionnalité pour créer, mettre à jour et supprimer des services cloud, des déploiements, des services de gestion de données et des machines virtuelles. Ces fonctionnalités peuvent être utiles pour la création d'applications nécessitant un accès par programme à la gestion des services.

## <a name="what-is-service-management"></a><a name="WhatIs"> </a>Qu’est-ce que la gestion des services ?
L’API Gestion des services Azure fournit un accès par programmation aux fonctionnalités de management des services disponibles par le biais du [portail Azure][management-portal]. Vous pouvez utiliser le Kit de développement logiciel (SDK) Azure pour Python pour gérer vos services cloud et comptes de stockage.

Pour utiliser l'API de gestion des services, vous devez [créer un compte Azure](https://azure.microsoft.com/pricing/free-trial/).

## <a name="concepts"></a><a name="Concepts"> </a>Concepts
Le Kit de développement logiciel (SDK) Azure pour Python inclut l’[API Gestion des services][svc-mgmt-rest-api], qui est une API REST. Toutes les opérations de l’API sont effectuées au moyen du protocole TLS et sont mutuellement authentifiées au moyen de certificats X.509 v3. Le service de gestion est accessible à partir d’un service s’exécutant dans Azure. Il est également accessible directement via Internet à partir de toute application capable d’envoyer une demande HTTPS et de recevoir une réponse HTTPS.

## <a name="installation"></a><a name="Installation"> </a>Installation
Toutes les fonctionnalités décrites dans cet article sont disponibles dans le package `azure-servicemanagement-legacy` que vous pouvez installer à l’aide de pip. Pour plus d’informations sur l’installation (par exemple si vous ne connaissez pas Python), voir [Installation de Python et du SDK Azure](/azure/developer/python/azure-sdk-install).

## <a name="connect-to-service-management"></a><a name="Connect"> </a>Se connecter à la gestion du service
Pour vous connecter au point de terminaison de gestion du service, vous avez besoin de votre ID d’abonnement Azure et d’un certificat de gestion valide. Vous pouvez obtenir votre ID d’abonnement dans le [portail Azure][management-portal].

> [!NOTE]
> Vous pouvez désormais utiliser des certificats créés avec OpenSSL lors de l’exécution sous Windows. Vous avez besoin de Python 2.7.4 ou version ultérieure. Nous vous recommandons d’utiliser OpenSSL au lieu de .pfx parce que la prise en charge des certificats .pfx sera probablement supprimée à l’avenir.
>
>

### <a name="management-certificates-on-windowsmaclinux-openssl"></a>Certificats de gestion sur Windows/Mac/Linux (OpenSSL)
Vous pouvez utiliser [OpenSSL](https://www.openssl.org/) pour créer votre certificat de gestion. Vous devez créer deux certificats, l’un pour le serveur (un fichier `.cer`) et l’autre pour le client (fichier `.pem`). Pour créer le fichier `.pem` , exécutez :

```console
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem
```

Pour créer le certificat `.cer` , exécutez :

```console
openssl x509 -inform pem -in mycert.pem -outform der -out mycert.cer
```

Pour plus d’informations sur les certificats Azure, voir [Vue d’ensemble des certificats pour Azure Cloud Services](cloud-services-certs-create.md). Pour une description complète des paramètres OpenSSL, consultez la documentation disponible sur [https://www.openssl.org/docs/apps/openssl.html](https://www.openssl.org/docs/apps/openssl.html).

Après avoir créé ces fichiers, téléchargez le fichier `.cer` sur Azure. Dans le [portail Azure][management-portal], sous l’onglet **Paramètres**, sélectionnez **Charger**. Remarquez que vous avez enregistré le fichier `.pem`.

Après avoir obtenu votre ID d’abonnement, créez un certificat, chargez le fichier `.cer` sur Azure, puis connectez-vous au point de terminaison de gestion Azure. Connectez-vous en passant l’ID d’abonnement et le chemin d’accès au fichier `.pem` à **ServiceManagementService**.

```python
from azure import *
from azure.servicemanagement import *

subscription_id = '<your_subscription_id>'
certificate_path = '<path_to_.pem_certificate>'

sms = ServiceManagementService(subscription_id, certificate_path)
```

Dans l’exemple ci-dessus, `sms` est un objet **ServiceManagementService** . La classe **ServiceManagementService** est la classe principale utilisée pour gérer les services Azure.

### <a name="management-certificates-on-windows-makecert"></a>Certificats de gestion sur Windows (MakeCert)
Vous pouvez créer un certificat de gestion auto-signé sur votre machine en utilisant `makecert.exe`. Ouvrez une **invite de commande Visual Studio** en tant qu’**administrateur**, puis utilisez la commande suivante, en remplaçant *AzureCertificate* par le nom du certificat que vous voulez utiliser :

```console
makecert -sky exchange -r -n "CN=AzureCertificate" -pe -a sha1 -len 2048 -ss My "AzureCertificate.cer"
```

La commande crée le fichier `.cer` et l’installe dans le magasin de certificats **Personnel**. Pour plus d’informations, voir [Vue d’ensemble des certificats pour Azure Cloud Services](cloud-services-certs-create.md).

Une fois le certificat créé, chargez le fichier `.cer` sur Azure. Dans le [portail Azure][management-portal], sous l’onglet **Paramètres**, sélectionnez **Charger**.

Après avoir obtenu votre ID d’abonnement, créez un certificat, chargez le fichier `.cer` sur Azure, puis connectez-vous au point de terminaison de gestion Azure. Connectez-vous en passant l’ID d’abonnement et l’emplacement du certificat dans votre magasin de certificats **Personnel** à **ServiceManagementService** (là encore, remplacez *AzureCertificate* par le nom de votre certificat).

```python
from azure import *
from azure.servicemanagement import *

subscription_id = '<your_subscription_id>'
certificate_path = 'CURRENT_USER\\my\\AzureCertificate'

sms = ServiceManagementService(subscription_id, certificate_path)
```

Dans l’exemple ci-dessus, `sms` est un objet **ServiceManagementService** . La classe **ServiceManagementService** est la classe principale utilisée pour gérer les services Azure.

## <a name="list-available-locations"></a><a name="ListAvailableLocations"> </a>Répertorier les emplacements disponibles
Pour répertorier les emplacements disponibles pour les services d’hébergement, utilisez la méthode **list\_locations**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

result = sms.list_locations()
for location in result:
    print(location.name)
```

Quand vous créez un service cloud ou un service de stockage, vous devez fournir un emplacement valide. La méthode **list\_locations** renvoie toujours une liste à jour des emplacements disponibles actuellement. Lors de la rédaction de cet article, les emplacements disponibles étaient les suivants :

* Europe Ouest
* Europe Nord
* Asie Sud-Est
* Asie Est
* USA Centre
* Centre-Nord des États-Unis
* États-Unis - partie centrale méridionale
* USA Ouest
* USA Est
* Japon Est
* OuJapon Est
* Brésil Sud
* Australie Est
* Sud-Australie Est

## <a name="create-a-cloud-service"></a><a name="CreateCloudService"> </a>Créer un service cloud
Quand vous créez une application et l’exécutez dans Azure, l’ensemble formé par le code et la configuration dans Azure est appelé [service cloud][cloud service]. (on l’appelle également *service hébergé* dans les versions antérieures d’Azure). Vous pouvez utiliser la méthode **create\_hosted\_service** pour créer un service hébergé. Créez le service en fournissant un nom de service hébergé (qui doit être unique dans Azure), une étiquette (automatiquement codée en base64), une description et un emplacement.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

name = 'myhostedservice'
label = 'myhostedservice'
desc = 'my hosted service'
location = 'West US'

sms.create_hosted_service(name, label, desc, location)
```

Vous pouvez répertorier tous les services hébergés pour votre abonnement en utilisant la méthode **list\_hosted\_services**.

```python
result = sms.list_hosted_services()

for hosted_service in result:
    print('Service name: ' + hosted_service.service_name)
    print('Management URL: ' + hosted_service.url)
    print('Location: ' + hosted_service.hosted_service_properties.location)
    print('')
```

Si vous souhaitez obtenir des informations sur un service hébergé particulier, passez son nom à la méthode **get\_hosted\_service\_properties**.

```python
hosted_service = sms.get_hosted_service_properties('myhostedservice')

print('Service name: ' + hosted_service.service_name)
print('Management URL: ' + hosted_service.url)
print('Location: ' + hosted_service.hosted_service_properties.location)
```

Une fois un service cloud créé, déployez votre code dessus avec la méthode **create\_deployment**.

## <a name="delete-a-cloud-service"></a><a name="DeleteCloudService"> </a>Supprimer un service cloud
Vous pouvez supprimer un service cloud en passant son nom à la méthode **delete\_hosted\_service**.

```python
sms.delete_hosted_service('myhostedservice')
```

Avant de supprimer un service, vous devez supprimer tous les déploiements associés. Pour plus d’informations, voir [Supprimer un déploiement](#DeleteDeployment).

## <a name="delete-a-deployment"></a><a name="DeleteDeployment"> </a>Supprimer un déploiement
Pour supprimer un déploiement, utilisez la méthode **delete\_deployment**. L’exemple suivant montre comment supprimer un déploiement nommé `v1`:

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

sms.delete_deployment('myhostedservice', 'v1')
```

## <a name="create-a-storage-service"></a><a name="CreateStorageService"> </a>Créer un service de stockage
Un [service de stockage](../storage/common/storage-account-create.md) donne accès aux [objets blob](../storage/blobs/storage-quickstart-blobs-python.md), [tables](../cosmos-db/table-storage-how-to-use-python.md) et [files d’attente](../storage/queues/storage-python-how-to-use-queue-storage.md) Azure. Pour créer un service de stockage, vous avez besoin d’un nom pour le service (comptant entre 3 et 24 caractères minuscules, et unique dans Azure). Vous avez également besoin d’une description, d’une étiquette (comptant jusqu’à 100 caractères, automatiquement codée en base64) et d’un emplacement. L’exemple suivant indique comment créer un service de stockage en spécifiant un emplacement :

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

name = 'mystorageaccount'
label = 'mystorageaccount'
location = 'West US'
desc = 'My storage account description.'

result = sms.create_storage_account(name, desc, label, location=location)

operation_result = sms.get_operation_status(result.request_id)
print('Operation status: ' + operation_result.status)
```

Dans l’exemple ci-dessus, l’état de l’opération **create\_storage\_account** peut être extrait en passant le résultat renvoyé par **create\_storage\_account** à la méthode **get\_operation\_status**. 

Vous pouvez afficher la liste de vos comptes de stockage et leurs propriétés avec la méthode **list\_storage\_accounts**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

result = sms.list_storage_accounts()
for account in result:
    print('Service name: ' + account.service_name)
    print('Location: ' + account.storage_service_properties.location)
    print('')
```

## <a name="delete-a-storage-service"></a><a name="DeleteStorageService"> </a>Supprimer un service de stockage
Pour supprimer un service de stockage, passez son nom à la méthode **delete\_storage\_account**. La suppression d’un service de stockage supprime toutes les données qui y sont stockées (objets blob, tables et files d’attente).

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

sms.delete_storage_account('mystorageaccount')
```

## <a name="list-available-operating-systems"></a><a name="ListOperatingSystems"> </a>Répertorier les systèmes d’exploitation disponibles
Pour répertorier les systèmes d’exploitation disponibles pour les services d’hébergement, utilisez la méthode **list\_operating\_systems**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

result = sms.list_operating_systems()

for os in result:
    print('OS: ' + os.label)
    print('Family: ' + os.family_label)
    print('Active: ' + str(os.is_active))
```

Vous pouvez également utiliser la méthode **list\_operating\_system\_families**, qui regroupe les systèmes d’exploitation par famille.

```python
result = sms.list_operating_system_families()

for family in result:
    print('Family: ' + family.label)
    for os in family.operating_systems:
        if os.is_active:
            print('OS: ' + os.label)
            print('Version: ' + os.version)
    print('')
```

## <a name="create-an-operating-system-image"></a><a name="CreateVMImage"> </a>Créer une image de système d’exploitation
Pour ajouter une image de système d’exploitation au référentiel d’images, utilisez la méthode **add\_os\_image**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

name = 'mycentos'
label = 'mycentos'
os = 'Linux' # Linux or Windows
media_link = 'url_to_storage_blob_for_source_image_vhd'

result = sms.add_os_image(label, media_link, name, os)

operation_result = sms.get_operation_status(result.request_id)
print('Operation status: ' + operation_result.status)
```

Pour afficher la liste des images de systèmes d’exploitation qui sont disponibles, utilisez la méthode **list\_os\_images**. Cela inclut toutes l’ensemble des images de plateforme et d’utilisateur.

```python
result = sms.list_os_images()

for image in result:
    print('Name: ' + image.name)
    print('Label: ' + image.label)
    print('OS: ' + image.os)
    print('Category: ' + image.category)
    print('Description: ' + image.description)
    print('Location: ' + image.location)
    print('Media link: ' + image.media_link)
    print('')
```

## <a name="delete-an-operating-system-image"></a><a name="DeleteVMImage"> </a>Supprimer une image de système d’exploitation
Pour supprimer une image d’utilisateur, utilisez la méthode **delete\_os\_image**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

result = sms.delete_os_image('mycentos')

operation_result = sms.get_operation_status(result.request_id)
print('Operation status: ' + operation_result.status)
```

## <a name="create-a-virtual-machine"></a><a name="CreateVM"> </a>Créer une machine virtuelle
Pour créer une machine virtuelle, vous devez d'abord créer un [service cloud](#CreateCloudService). Ensuite, créez le déploiement de machine virtuelle en utilisant la méthode **create\_virtual\_machine\_deployment**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

name = 'myvm'
location = 'West US'

#Set the location
sms.create_hosted_service(service_name=name,
    label=name,
    location=location)

# Name of an os image as returned by list_os_images
image_name = 'OpenLogic__OpenLogic-CentOS-62-20120531-en-us-30GB.vhd'

# Destination storage account container/blob where the VM disk
# will be created
media_link = 'url_to_target_storage_blob_for_vm_hd'

# Linux VM configuration, you can use WindowsConfigurationSet
# for a Windows VM instead
linux_config = LinuxConfigurationSet('myhostname', 'myuser', 'mypassword', True)

os_hd = OSVirtualHardDisk(image_name, media_link)

sms.create_virtual_machine_deployment(service_name=name,
    deployment_name=name,
    deployment_slot='production',
    label=name,
    role_name=name,
    system_config=linux_config,
    os_virtual_hard_disk=os_hd,
    role_size='Small')
```

## <a name="delete-a-virtual-machine"></a><a name="DeleteVM"> </a>Supprimer une machine virtuelle
Pour supprimer une machine virtuelle, vous devez commencer par supprimer le déploiement en utilisant la méthode **delete\_deployment**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

sms.delete_deployment(service_name='myvm',
    deployment_name='myvm')
```

Vous pouvez ensuite supprimer le service cloud en utilisant la méthode **delete\_hosted\_service**.

```python
sms.delete_hosted_service(service_name='myvm')
```

## <a name="create-a-virtual-machine-from-a-captured-virtual-machine-image"></a>Créer une machine virtuelle à partir d’une image de machine virtuelle capturée
Pour capturer une image de machine virtuelle, commencez par appeler la méthode **capture\_vm\_image**.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

# replace the below three parameters with actual values
hosted_service_name = 'hs1'
deployment_name = 'dep1'
vm_name = 'vm1'

image_name = vm_name + 'image'
image = CaptureRoleAsVMImage    ('Specialized',
    image_name,
    image_name + 'label',
    image_name + 'description',
    'english',
    'mygroup')

result = sms.capture_vm_image(
        hosted_service_name,
        deployment_name,
        vm_name,
        image
    )
```

Pour vous assurer que vous avez bien capturé l’image, utilisez l’API **liste\_vm\_images**. Assurez-vous que votre image s’affiche dans les résultats.

```python
images = sms.list_vm_images()
```

Enfin, pour créer la machine virtuelle à l’aide de l’image capturée, utilisez la méthode **create\_virtual\_machine\_deployment** comme avant mais, cette fois, en passant vm_image_name.

```python
from azure import *
from azure.servicemanagement import *

sms = ServiceManagementService(subscription_id, certificate_path)

name = 'myvm'
location = 'West US'

#Set the location
sms.create_hosted_service(service_name=name,
    label=name,
    location=location)

sms.create_virtual_machine_deployment(service_name=name,
    deployment_name=name,
    deployment_slot='production',
    label=name,
    role_name=name,
    system_config=linux_config,
    os_virtual_hard_disk=None,
    role_size='Small',
    vm_image_name = image_name)
```

Pour en savoir plus sur la capture d’une machine virtuelle Linux dans le modèle de déploiement classique, voir [Capturer une machine virtuelle Linux](/previous-versions/azure/virtual-machines/linux/classic/capture-image-classic).

Pour en savoir plus sur la capture d’une machine virtuelle Windows dans le modèle de déploiement classique, voir [Capturer une machine virtuelle Windows](/previous-versions/azure/virtual-machines/windows/classic/capture-image-classic).

## <a name="next-steps"></a><a name="What's Next"> </a>Étapes suivantes
Vous connaissez désormais les principes de base de la gestion des services. Vous pouvez maintenant accéder à la [documentation complète de référence sur l’API du Kit de développement logiciel (SDK) Azure pour Python](https://azure-sdk-for-python.readthedocs.org/) et effectuer facilement des tâches complexes pour gérer votre application Python.

Pour plus d’informations, consultez le [Centre pour développeurs Python](https://azure.microsoft.com/develop/python/).

[What is service management?]: #WhatIs
[Concepts]: #Concepts
[Connect to service management]: #Connect
[List available locations]: #ListAvailableLocations
[Create a cloud service]: #CreateCloudService
[Delete a cloud service]: #DeleteCloudService
[Create a deployment]: #CreateDeployment
[Update a deployment]: #UpdateDeployment
[Move deployments between staging and production]: #MoveDeployments
[Delete a deployment]: #DeleteDeployment
[Create a storage service]: #CreateStorageService
[Delete a storage service]: #DeleteStorageService
[List available operating systems]: #ListOperatingSystems
[Create an operating system image]: #CreateVMImage
[Delete an operating system image]: #DeleteVMImage
[Create a virtual machine]: #CreateVM
[Delete a virtual machine]: #DeleteVM
[Next steps]: #NextSteps
[management-portal]: https://portal.azure.com/
[svc-mgmt-rest-api]: /previous-versions/azure/ee460799(v=azure.100)


[cloud service]:/azure/cloud-services/