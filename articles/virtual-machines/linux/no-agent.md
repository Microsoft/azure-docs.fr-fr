---
title: Créer des images Linux sans agent d’approvisionnement
description: Créez des images Linux généralisées sans agent d’approvisionnement dans Azure.
author: danielsollondon
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 09/01/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: 9f0309f4e8273c2ef19ea86636de8e3aa6b6c4bc
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96435098"
---
# <a name="creating-generalized-images-without-a-provisioning-agent"></a>Création d’images généralisées sans agent d’approvisionnement

Microsoft Azure fournit des agents d’approvisionnement pour les machines virtuelles Linux sous la forme de [walinuxagent](https://github.com/Azure/WALinuxAgent) ou [cloud-init](https://github.com/canonical/cloud-init) (recommandé). Toutefois, il peut y avoir un scénario où vous ne souhaitez pas utiliser l’une de ces applications pour votre agent d’approvisionnement, par exemple :

- Votre version de distribution/Linux ne prend pas en charge cloud-init/l’agent Linux.
- Vous devez définir des propriétés de machine virtuelle spécifiques, telles que le nom d'hôte.

> [!NOTE] 
>
> Si vous n’avez pas besoin de définir des propriétés ou une forme d’approvisionnement, vous devez envisager de créer une image spécialisée.

Cet article explique comment configurer votre image de machine virtuelle pour répondre aux exigences de la plateforme Azure et définir le nom d’hôte, sans installer d’agent d’approvisionnement.

## <a name="networking-and-reporting-ready"></a>Préparer la mise en réseau et les rapports

Pour que votre machine virtuelle Linux communique avec les composants Azure, vous devez disposer d’un client DHCP pour récupérer une adresse IP d’hôte à partir du réseau virtuel, ainsi que de la résolution DNS et de la gestion de routage. La plupart des distributions sont livrées avec ces utilitaires prêts à l’emploi. Les outils qui ont été testés sur Azure par les fournisseurs de distribution Linux incluent `dhclient`, `network-manager`, `systemd-networkd` et d’autres.

> [!NOTE]
>
> Actuellement, la création d’images généralisées sans agent d’approvisionnement ne prend en charge que les machines virtuelles compatibles DHCP.

Une fois que la mise en réseau a été configurée, vous devez avoir vos rapports de prêts. Cela indique à Azure que la machine virtuelle a été configurée avec succès.

> [!IMPORTANT]
>
> Si vous ne réussissez pas à créer de rapports sur Azure, votre machine virtuelle sera redémarrée.

## <a name="demosample"></a>Démonstration/Exemple

Cette démonstration montre comment vous pouvez prendre une image existante de la Place de marché (dans ce cas, une machine virtuelle Debian Buster) et supprimer l’agent Linux (walinuxagent), mais également créer le processus le plus basique pour signaler à Azure que la machine virtuelle est « prête ».

### <a name="create-the-resource-group-and-base-vm"></a>Créez le groupe de ressources et la machine virtuelle de base :

```bash
$ az group create --location eastus --name demo1
```

Créez la machine virtuelle de base :

```bash
$ az vm create \
    --resource-group demo1 \
    --name demo1 \
    --location eastus \
    --ssh-key-value <ssh_pub_key_path> \
    --public-ip-address-dns-name demo1 \
    --image "debian:debian-10:10:latest"
```

### <a name="remove-the-image-provisioning-agent"></a>Supprimer l’image de l’agent d’approvisionnement

Une fois la machine virtuelle approvisionnée, vous pouvez l’exécuter en tant que SSH et supprimer l’agent Linux :

```bash
$ sudo apt purge -y waagent
$ sudo rm -rf /var/lib/waagent /etc/waagent.conf /var/log/waagent.log
```

### <a name="add-required-code-to-the-vm"></a>Ajouter le code requis à la machine virtuelle

Étant donné que nous avons supprimé l’agent Linux Azure, nous devons fournir un mécanisme de rapport prêt à l’intérieur de la machine virtuelle. 

#### <a name="python-script"></a>Script Python

```python
import http.client
import sys
from xml.etree import ElementTree

wireserver_ip = '168.63.129.16'
wireserver_conn = http.client.HTTPConnection(wireserver_ip)

print('Retrieving goal state from the Wireserver')
wireserver_conn.request(
    'GET',
    '/machine?comp=goalstate',
    headers={'x-ms-version': '2012-11-30'}
)

resp = wireserver_conn.getresponse()

if resp.status != 200:
    print('Unable to connect with wireserver')
    sys.exit(1)

wireserver_goalstate = resp.read().decode('utf-8')

xml_el = ElementTree.fromstring(wireserver_goalstate)

container_id = xml_el.findtext('Container/ContainerId')
instance_id = xml_el.findtext('Container/RoleInstanceList/RoleInstance/InstanceId')
print(f'ContainerId: {container_id}')
print(f'InstanceId: {instance_id}')

# Construct the XML response we need to send to Wireserver to report ready.
health = ElementTree.Element('Health')
goalstate_incarnation = ElementTree.SubElement(health, 'GoalStateIncarnation')
goalstate_incarnation.text = '1'
container = ElementTree.SubElement(health, 'Container')
container_id_el = ElementTree.SubElement(container, 'ContainerId')
container_id_el.text = container_id
role_instance_list = ElementTree.SubElement(container, 'RoleInstanceList')
role = ElementTree.SubElement(role_instance_list, 'Role')
instance_id_el = ElementTree.SubElement(role, 'InstanceId')
instance_id_el.text = instance_id
health_second = ElementTree.SubElement(role, 'Health')
state = ElementTree.SubElement(health_second, 'State')
state.text = 'Ready'

out_xml = ElementTree.tostring(
    health,
    encoding='unicode',
    method='xml'
)
print('Sending the following data to Wireserver:')
print(out_xml)

wireserver_conn.request(
    'POST',
    '/machine?comp=health',
    headers={
        'x-ms-version': '2012-11-30',
        'Content-Type': 'text/xml;charset=utf-8',
        'x-ms-agent-name': 'custom-provisioning'
    },
    body=out_xml
)

resp = wireserver_conn.getresponse()
print(f'Response: {resp.status} {resp.reason}')

wireserver_conn.close()
```

#### <a name="generic-steps-without-using-python"></a>Étapes génériques (sans utiliser Python)

Si Python n’est pas installé ou disponible sur votre machine virtuelle, vous pouvez reproduire par programmation la logique de script ci-dessus en procédant comme suit :

1. Récupérez les `ContainerId` et les `InstanceId` en analysant la réponse de WireServer : `curl -X GET -H 'x-ms-version: 2012-11-30' http://$168.63.129.16/machine?comp=goalstate`.

2. Construisez les données XML suivantes, en injectant `ContainerId` analysé et `InstanceId` à partir de l’étape ci-dessus :
   ```xml
   <Health>
     <GoalStateIncarnation>1</GoalStateIncarnation>
     <Container>
       <ContainerId>CONTAINER_ID</ContainerId>
       <RoleInstanceList>
         <Role>
           <InstanceId>INSTANCE_ID</InstanceId>
           <Health>
             <State>Ready</State>
           </Health>
         </Role>
       </RoleInstanceList>
     </Container>
   </Health>
   ```

3. Publiez ces données dans WireServer : `curl -X POST -H 'x-ms-version: 2012-11-30' -H "x-ms-agent-name: WALinuxAgent" -H "Content-Type: text/xml;charset=utf-8" -d "$REPORT_READY_XML" http://168.63.129.16/machine?comp=health`

### <a name="automating-running-the-code-at-first-boot"></a>Automatisation de l’exécution du code au premier démarrage

Cette démonstration utilise SystemD, qui est le système init le plus courant dans les distributions Linux modernes. La méthode la plus simple et la plus native pour s’assurer que ce mécanisme prêt pour les rapports s’exécute au bon moment est la création d’une unité de service système. Vous pouvez ajouter le fichier d’unité suivant à `/etc/systemd/system` (cet exemple nomme le fichier d’unité `azure-provisioning.service`) :

```
[Unit]
Description=Azure Provisioning

[Service]
Type=oneshot
ExecStart=/usr/bin/python3 /usr/local/azure-provisioning.py
ExecStart=/bin/bash -c "hostnamectl set-hostname $(curl \
    -H 'metadata: true' \
    'http://169.254.169.254/metadata/instance/compute/name?api-version=2019-06-01&format=text')"
ExecStart=/usr/bin/systemctl disable azure-provisioning.service

[Install]
WantedBy=multi-user.target
```

Ce service système fait trois choses pour l’approvisionnement de base :

1. Préparez les rapports pour Azure (pour indiquer qu’ils ont été correctement exécutés).
1. Renomme la machine virtuelle en fonction du nom de machine virtuelle fourni par l’utilisateur en extrayant ces données à partir d’[Azure Instance Metadata Service (IMDS)](./instance-metadata-service.md). **Remarque** IMDS fournit également d’autres [métadonnées d’instance](./instance-metadata-service.md#access-azure-instance-metadata-service), telles que des clés publiques SSH, pour vous permettre de définir davantage le nom d’hôte.
1. Se désactive pour s’exécuter uniquement au premier démarrage et non pas lors des redémarrages suivants.

Avec l’unité sur le système de fichiers, exécutez la commande suivante pour l’activer :

```
$ sudo systemctl enable azure-provisioning.service
```

La machine virtuelle est maintenant prête à être généralisée et une image est créée à partir de celle-ci. 

#### <a name="completing-the-preparation-of-the-image"></a>Fin de la préparation de l’image

De retour sur votre ordinateur de développement, exécutez la commande suivante pour préparer la création d’images à partir de la machine virtuelle de base :

```
$ az vm deallocate --resource-group demo1 --name demo1
$ az vm generalize --resource-group demo1 --name demo1
```

Et créez l’image à partir de cette machine virtuelle :

```
$ az image create \
    --resource-group demo1 \
    --source demo1 \
    --location eastus \
    --name demo1img
```

Nous sommes maintenant prêts à créer une nouvelle machine virtuelle (ou plusieurs machines virtuelles) à partir de l’image :

```
$ IMAGE_ID=$(az image show -g demo1 -n demo1img --query id -o tsv)
$ az vm create \
    --resource-group demo12 \
    --name demo12 \
    --location eastus \
    --ssh-key-value <ssh_pub_key_path> \
    --public-ip-address-dns-name demo12 \
    --image "$IMAGE_ID" 
    --enable-agent false
```

> [!NOTE]
>
> Il est important de définir `--enable-agent` sur `false`, car walinuxagent n’existe pas sur la machine virtuelle qui va être créée à partir de l’image.

Cette machine virtuelle doit être configurée correctement. En vous connectant à la machine virtuelle qui vient d’être configurée, vous devez être en mesure de voir la sortie du service système prêt pour le rapport :

```
$ sudo journalctl -u azure-provisioning.service
-- Logs begin at Thu 2020-06-11 20:28:45 UTC, end at Thu 2020-06-11 20:31:24 UTC. --
Jun 11 20:28:49 thstringnopa systemd[1]: Starting Azure Provisioning...
Jun 11 20:28:54 thstringnopa python3[320]: Retrieving goal state from the Wireserver
Jun 11 20:28:54 thstringnopa python3[320]: ContainerId: 7b324f53-983a-43bc-b919-1775d6077608
Jun 11 20:28:54 thstringnopa python3[320]: InstanceId: fbb84507-46cd-4f4e-bd78-a2edaa9d059b._thstringnopa2
Jun 11 20:28:54 thstringnopa python3[320]: Sending the following data to Wireserver:
Jun 11 20:28:54 thstringnopa python3[320]: <Health><GoalStateIncarnation>1</GoalStateIncarnation><Container><ContainerId>7b324f53-983a-43bc-b919-1775d6077608</ContainerId><RoleInstanceList><Role><InstanceId>fbb84507-46cd-4f4e-bd78-a2edaa9d059b._thstringnopa2</InstanceId><Health><State>Ready</State></Health></Role></RoleInstanceList></Container></Health>
Jun 11 20:28:54 thstringnopa python3[320]: Response: 200 OK
Jun 11 20:28:56 thstringnopa bash[472]:   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
Jun 11 20:28:56 thstringnopa bash[472]:                                  Dload  Upload   Total   Spent    Left  Speed
Jun 11 20:28:56 thstringnopa bash[472]: [158B blob data]
Jun 11 20:28:56 thstringnopa2 systemctl[475]: Removed /etc/systemd/system/multi-user.target.wants/azure-provisioning.service.
Jun 11 20:28:56 thstringnopa2 systemd[1]: azure-provisioning.service: Succeeded.
Jun 11 20:28:56 thstringnopa2 systemd[1]: Started Azure Provisioning.
```

## <a name="support"></a>Support

Si vous implémentez votre propre code/agent de provisionnement, vous possédez la prise en charge de ce code, le support technique de Microsoft n’examinera que les problèmes liés aux interfaces d’approvisionnement non disponibles. Nous apportons continuellement des améliorations et des modifications dans ce domaine. vous devez donc surveiller les modifications apportées à cloud-init et à l’agent Linux Azure pour l’approvisionnement des modifications de l’API.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Provisionnement Linux](provisioning.md).