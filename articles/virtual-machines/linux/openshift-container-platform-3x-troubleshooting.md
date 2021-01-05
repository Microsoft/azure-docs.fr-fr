---
title: Résoudre les problèmes de déploiement d’OpenShift Container Platform 3.11 dans Azure
description: Résolvez les problèmes de déploiement d’OpenShift Container Platform 3.11 dans Azure.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/14/2019
ms.author: haroldw
ms.custom: devx-track-ansible
ms.openlocfilehash: 9595627e9d7ca2de577aa83ebba3dd58d69e6750
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87373553"
---
# <a name="troubleshoot-openshift-container-platform-311-deployment-in-azure"></a>Résoudre les problèmes de déploiement d’OpenShift Container Platform 3.11 dans Azure

Si le cluster OpenShift ne se déploie pas correctement, le portail Azure retourne une sortie d’erreur. La sortie peut être compliquée à lire, ce qui rend difficile l’identification du problème. Examinez rapidement cette sortie pour rechercher le code de sortie 3, 4 ou 5. Ces trois codes de sortie ont les significations suivantes :

- Code de sortie 3 : le nom d’utilisateur/mot de passe ou l’ID d’organisation/clé d’activation de votre abonnement Red Hat sont incorrects
- Code de sortie 4 : votre ID de pool Red Hat est incorrect ou aucun droit n’est disponible
- Code de sortie 5 : impossible de provisionner le volume de pool dynamique Docker

Pour tous les autres codes de sortie, vous pouvez afficher les fichiers journaux correspondants en vous connectant aux hôtes via SSH.

**OpenShift Container Platform 3.11**

Établissez une connexion SSH à l’hôte de playbook Ansible. Pour le modèle ou l’offre de la Place de marché, utilisez l’hôte Bastion. À partir de l’hôte Bastion, vous pouvez établir une connexion SSH à tous les autres nœuds du cluster (master, infrastructure, CNS, calcul). Vous devez être à la racine pour afficher les fichiers journaux. Comme l’accès à la racine est désactivé pour SSH par défaut, ne l’utilisez pas pour établir une connexion aux autres nœuds.

**OKD**

Établissez une connexion SSH à l’hôte de playbook Ansible. Pour le modèle OKD (versions 3.9 et antérieures), utilisez l’hôte master-0. Pour le modèle OKD (versions 3.10 et ultérieures), utilisez l’hôte Bastion. À partir de l’hôte de playbook Ansible, vous pouvez établir une connexion SSH à tous les autres nœuds du cluster (master, infrastructure, CNS, calcul). Vous devez être à la racine (sudo su -) pour afficher les fichiers journaux. Comme l’accès à la racine est désactivé pour SSH par défaut, ne l’utilisez pas pour établir une connexion aux autres nœuds.

## <a name="log-files"></a>Fichiers journaux

Les fichiers journaux (stderr et stdout) pour les scripts de préparation d’hôte se trouvent dans `/var/lib/waagent/custom-script/download/0` sur tous les hôtes. Si une erreur s’est produite durant la préparation de l’hôte, consultez ces fichiers journaux pour identifier l’erreur.

Si les scripts de préparation se sont exécutés correctement, examinez les fichiers journaux qui se trouvent dans le répertoire `/var/lib/waagent/custom-script/download/1` sur l’hôte de playbook Ansible. Si l’erreur s’est produite pendant l’installation d’OpenShift, elle est enregistrée dans le fichier stdout. Consultez ce fichier avant de contacter le support pour demander de l’aide.

Exemple de sortie

```json
TASK [openshift_storage_glusterfs : Load heketi topology] **********************
fatal: [mycluster-master-0]: FAILED! => {"changed": true, "cmd": ["oc", "--config=/tmp/openshift-glusterfs-ansible-IbhnUM/admin.kubeconfig", "rsh", "--namespace=glusterfs", "deploy-heketi-storage-1-d9xl5", "heketi-cli", "-s", "http://localhost:8080", "--user", "admin", "--secret", "VuoJURT0/96E42Vv8+XHfsFpSS8R20rH1OiMs3OqARQ=", "topology", "load", "--json=/tmp/openshift-glusterfs-ansible-IbhnUM/topology.json", "2>&1"], "delta": "0:00:21.477831", "end": "2018-05-20 02:49:11.912899", "failed": true, "failed_when_result": true, "rc": 0, "start": "2018-05-20 02:48:50.435068", "stderr": "", "stderr_lines": [], "stdout": "Creating cluster ... ID: 794b285745b1c5d7089e1c5729ec7cd2\n\tAllowing file volumes on cluster.\n\tAllowing block volumes on cluster.\n\tCreating node mycluster-cns-0 ... ID: 45f1a3bfc20a4196e59ebb567e0e02b4\n\t\tAdding device /dev/sdd ... OK\n\t\tAdding device /dev/sde ... OK\n\t\tAdding device /dev/sdf ... OK\n\tCreating node mycluster-cns-1 ... ID: 596f80d7bbd78a1ea548930f23135131\n\t\tAdding device /dev/sdc ... Unable to add device: Unable to execute command on glusterfs-storage-4zc42:   Device /dev/sdc excluded by a filter.\n\t\tAdding device /dev/sde ... OK\n\t\tAdding device /dev/sdd ... OK\n\tCreating node mycluster-cns-2 ... ID: 42c0170aa2799559747622acceba2e3f\n\t\tAdding device /dev/sde ... OK\n\t\tAdding device /dev/sdf ... OK\n\t\tAdding device /dev/sdd ... OK", "stdout_lines": ["Creating cluster ... ID: 794b285745b1c5d7089e1c5729ec7cd2", "\tAllowing file volumes on cluster.", "\tAllowing block volumes on cluster.", "\tCreating node mycluster-cns-0 ... ID: 45f1a3bfc20a4196e59ebb567e0e02b4", "\t\tAdding device /dev/sdd ... OK", "\t\tAdding device /dev/sde ... OK", "\t\tAdding device /dev/sdf ... OK", "\tCreating node mycluster-cns-1 ... ID: 596f80d7bbd78a1ea548930f23135131", "\t\tAdding device /dev/sdc ... Unable to add device: Unable to execute command on glusterfs-storage-4zc42:   Device /dev/sdc excluded by a filter.", "\t\tAdding device /dev/sde ... OK", "\t\tAdding device /dev/sdd ... OK", "\tCreating node mycluster-cns-2 ... ID: 42c0170aa2799559747622acceba2e3f", "\t\tAdding device /dev/sde ... OK", "\t\tAdding device /dev/sdf ... OK", "\t\tAdding device /dev/sdd ... OK"]}

PLAY RECAP *********************************************************************
mycluster-cns-0       : ok=146  changed=57   unreachable=0    failed=0   
mycluster-cns-1       : ok=146  changed=57   unreachable=0    failed=0   
mycluster-cns-2       : ok=146  changed=57   unreachable=0    failed=0   
mycluster-infra-0     : ok=143  changed=55   unreachable=0    failed=0   
mycluster-infra-1     : ok=143  changed=55   unreachable=0    failed=0   
mycluster-infra-2     : ok=143  changed=55   unreachable=0    failed=0   
mycluster-master-0    : ok=502  changed=198  unreachable=0    failed=1   
mycluster-master-1    : ok=348  changed=140  unreachable=0    failed=0   
mycluster-master-2    : ok=348  changed=140  unreachable=0    failed=0   
mycluster-node-0      : ok=143  changed=55   unreachable=0    failed=0   
mycluster-node-1      : ok=143  changed=55   unreachable=0    failed=0   
localhost                  : ok=13   changed=0    unreachable=0    failed=0   

INSTALLER STATUS ***************************************************************
Initialization             : Complete (0:00:39)
Health Check               : Complete (0:00:24)
etcd Install               : Complete (0:01:24)
Master Install             : Complete (0:14:59)
Master Additional Install  : Complete (0:01:10)
Node Install               : Complete (0:10:58)
GlusterFS Install          : In Progress (0:03:33)
    This phase can be restarted by running: playbooks/openshift-glusterfs/config.yml

Failure summary:

  1. Hosts:    mycluster-master-0
     Play:     Configure GlusterFS
     Task:     Load heketi topology
     Message:  Failed without returning a message.
```

Les erreurs les plus courantes lors de l’installation sont les suivantes :

1. La clé privée a une phrase secrète
2. Le secret de coffre de clés avec la clé privée n’a pas été créé correctement
3. Les informations d’identification du principal de service n’ont pas été entrées correctement
4. Le principal de service n’a pas l’accès Contributeur au groupe de ressources

### <a name="private-key-has-a-passphrase"></a>La clé privée a une phrase secrète

Une erreur s’affiche indiquant que l’autorisation a été refusée pour ssh. Établissez une connexion ssh à l’hôte de playbook Ansible pour vérifier si la clé privée comporte une phrase secrète.

### <a name="key-vault-secret-with-private-key-wasnt-created-correctly"></a>Le secret de coffre de clés avec la clé privée n’a pas été créé correctement

La clé privée est copiée dans l’hôte de playbook Ansible - ~/.ssh/id_rsa. Vérifiez que ce fichier est correct. Testez-le en ouvrant une session SSH sur l’un des nœuds du cluster à partir de l’hôte de playbook Ansible.

### <a name="service-principal-credentials-were-entered-incorrectly"></a>Les informations d’identification du principal de service n’ont pas été entrées correctement

Les informations fournies dans le modèle ou l’offre de la Place de marché ne sont pas correctes. Vérifiez que vous utilisez l’appId (clientId) et le mot de passe (clientSecret) corrects pour le principal de service. Pour cela, exécutez la commande Azure CLI suivante.

```azurecli
az login --service-principal -u <client id> -p <client secret> -t <tenant id>
```

### <a name="service-principal-doesnt-have-contributor-access-to-the-resource-group"></a>Le principal de service n’a pas l’accès Contributeur au groupe de ressources

Si le fournisseur de cloud Azure est activé, le principal de service utilisé doit avoir l’accès Contributeur au groupe de ressources. Pour cela, exécutez la commande Azure CLI suivante.

```azurecli
az group update -g <openshift resource group> --set tags.sptest=test
```

## <a name="additional-tools"></a>Outils supplémentaires

Pour obtenir plus d’informations sur certaines erreurs, vous pouvez également utiliser les commandes suivantes :

1. systemctl status \<service>
2. journalctl -xe
