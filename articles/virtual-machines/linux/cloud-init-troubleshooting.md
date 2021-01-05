---
title: Résoudre les problèmes avec cloud-init
description: Résoudre les problèmes de configuration d’une machine virtuelle Azure à l’aide de cloud-init.
author: danielsollondon
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: troubleshooting
ms.date: 07/06/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: 6412036e3f16e2efb3bbf6669f6a31e9dc6e3584
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89434637"
---
# <a name="troubleshooting-vm-provisioning-with-cloud-init"></a>Résoudre les problèmes d’approvisionnement des machines virtuelles avec cloud-init

Si vous avez créé des images personnalisées généralisées à l’aide de cloud-init pour effectuer l’approvisionnement, mais que la machine virtuelle n’a pas été créée correctement, vous devez dépanner vos images personnalisées.

Voici quelques exemples de problèmes d’approvisionnement :
- La machine virtuelle est bloquée lors de la « création » pendant 40 minutes, et la création de la machine virtuelle est marquée comme ayant échoué
- `CustomData` n’est pas traité
- Échec du montage du disque éphémère
- Les utilisateurs ne sont pas créés, ou il y a des problèmes d’accès utilisateur
- La mise en réseau n’est pas configurée correctement
- Échecs de fichier ou de partition d’échange

Cet article vous explique les procédures pour résoudre des problèmes avec cloud-init. Pour plus d’informations, consultez [Formation approfondie sur cloud-init](./cloud-init-deep-dive.md).

## <a name="step-1-test-the-deployment-without-customdata"></a>Étape 1 : Tester le déploiement sans `customData`

Cloud-init peut accepter `customData`, qui lui est transmis, lors de la création de la machine virtuelle. Tout d’abord, vous devez vous assurer que cela ne pose aucun problème avec les déploiements. Essayez d’approvisionner la machine virtuelle sans intégrer de configuration. Si vous ne parvenez pas à approvisionner la machine virtuelle, poursuivez avec les étapes ci-dessous, si vous trouvez que la configuration que vous intégrez n’est pas appliquée, passez [l’étape 4](). 

## <a name="step-2-review-image-requirements"></a>Étape 2 : Vérifier les exigences des images
La cause principale de l’échec de l’approvisionnement de la machine virtuelle est que l’image du système d’exploitation ne satisfait pas les conditions préalables à l’exécution sur Azure. Assurez-vous que vos images sont correctement préparées avant d’essayer de les configurer dans Azure. 


Les articles ci-après expliquent comment préparer les diverses distributions Linux prises en charge sur Azure :

- [Distributions CentOS](create-upload-centos.md)
- [Debian Linux](debian-create-upload-vhd.md)
- [Flatcar Container Linux](flatcar-create-upload-vhd.md)
- [Oracle Linux](oracle-create-upload-vhd.md)
- [Red Hat Enterprise Linux](redhat-create-upload-vhd.md)
- [SLES et openSUSE](suse-create-upload-vhd.md)
- [Ubuntu](create-upload-ubuntu.md)
- [Autres : Distributions non approuvées](create-upload-generic.md)

Pour les [images Azure cloud-init prises en charge](./using-cloud-init.md), les distributions Linux disposent déjà de tous les packages et configurations nécessaires pour approvisionner correctement l’image dans Azure. Si vous constatez que la création de votre machine virtuelle échoue à partir de votre propre image, essayez une image de la Place de marché Azure prise en charge qui est déjà configurée pour cloud-init, avec votre `customData` facultatif. Si le `customData` fonctionne correctement avec une image de la Place de marché Azure, il y a probablement un problème avec votre image organisée.

## <a name="step-3-collect--review-vm-logs"></a>Étape 3 : Collecter et consulter les journaux des machines virtuelles

En cas d’échec de l’approvisionnement de la machine virtuelle, Azure affiche l’état « création », pendant 20 minutes, puis redémarrer la machine virtuelle. Patientez ensuite 20 minutes avant de marquer finalement le déploiement de la machine virtuelle comme ayant échoué (erreur `OSProvisioningTimedOut`).

Pendant que la machine virtuelle est en cours d’exécution, vous aurez besoin des journaux de la machine virtuelle pour comprendre la raison de l’échec de l’approvisionnement.  Pour comprendre la raison de l’échec de l’approvisionnement d’une machine virtuelle, n’arrêtez pas la machine virtuelle. Laissez la machine virtuelle en cours d’exécution. Vous devez conserver l’état d’exécution de la machine virtuelle qui a échoué afin de collecter les journaux. Pour collecter les journaux, utilisez l’une des méthodes suivantes :

- [Console série](../troubleshooting/serial-console-grub-single-user-mode.md)

- [Activez les diagnostics de démarrage](./tutorial-monitor.md#enable-boot-diagnostics) avant de créer la machine virtuelle, puis les [afficher](./tutorial-monitor.md#view-boot-diagnostics) au cours du démarrage.

- [Exécutez la commande AZ VM Repair](../troubleshooting/repair-linux-vm-using-azure-virtual-machine-repair-commands.md) pour attacher et monter le disque du système d’exploitation, ce qui vous permet de collecter ces journaux :
```bash
/var/log/cloud-init*
/var/log/waagent*
/var/log/syslog*
/var/log/rsyslog*
/var/log/messages*
/var/log/kern*
/var/log/dmesg*
/var/log/boot*
```
Pour commencer le dépannage initial, commencez par les journaux cloud-init et cherchez où la défaillance s’est produite, puis utilisez les autres journaux pour approfondir vos recherches et fournir des informations supplémentaires. 
* /var/log/cloud-init.log
* /var/log/cloud-init-output.log
* Journaux de démarrage/série

Dans tous les journaux, commencez par rechercher « Failed », « WARNING », « WARN », « err », « error », « ERROR ». La définition de la configuration pour ignorer les recherches respectant la casse est recommandée. 

> [!TIP]
> Si vous dépannez une image personnalisée, vous devez envisager d’ajouter un utilisateur au cours de l’image. Si l’approvisionnement ne parvient pas à définir l’utilisateur administrateur, vous pouvez toujours vous connecter au système d’exploitation.

## <a name="analyzing-the-logs"></a>Analyse des journaux

Voici plus d’informations sur les éléments à rechercher dans chaque journal cloud-init.

### <a name="varlogcloud-initlog"></a>/var/log/cloud-init.log

Par défaut, tous les événements cloud-init avec une priorité de débogage ou supérieure sont écrits dans `/var/log/cloud-init.log`. Cela fournit des journaux détaillés de chaque événement produit pendant l’initialisation de cloud-init. 

Par exemple :

```console
2019-10-10 04:51:25,321 - util.py[DEBUG]: Failed mount of '/dev/sr0' as 'auto': Unexpected error while running command.
Command: ['mount', '-o', 'ro,sync', '-t', 'auto', u'/dev/sr0', '/run/cloud-init/tmp/tmpLIrklc']
Exit code: 32
Reason: -
Stdout:
Stderr: mount: unknown filesystem type 'udf'
2020-01-31 00:21:53,352 - DataSourceAzure.py[WARNING]: /dev/sr0 was not mountable
```


Une fois que vous avez trouvé une erreur ou un avertissement, remontez dans le journal cloud-init pour comprendre ce que cloud-init a tenté avant d’atteindre l’erreur ou l’avertissement. Dans de nombreux cas, cloud-init a exécuté des commandes de système d’exploitation ou effectué des opérations d’approvisionnement avant l’erreur, ce qui peut fournir des informations sur la raison pour laquelle des erreurs se sont produites dans les journaux. L’exemple suivant montre que cloud-init a tenté de monter un appareil juste avant d’avoir rencontré une erreur.

```output
2019-10-10 04:51:24,010 - util.py[DEBUG]: Running command ['mount', '-o', 'ro,sync', '-t', 'auto', u'/dev/sr0', '/run/cloud-init/tmp/tmpXXXXX'] with allowed return codes [0] (shell=False, capture=True)
```

Si vous avez accès à la [Console série ](../troubleshooting/serial-console-grub-single-user-mode.md), vous pouvez réexécutez la commande que cloud-init essayait d’exécuter.

La journalisation de `/var/log/cloud-init.log` peut également être reconfigurée dans /etc/cloud/cloud.cfg.d/05_logging.cfg. Pour plus d’informations sur la journalisation de cloud-init, consultez la [documentation cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/logging.html). 

### <a name="varlogcloud-init-outputlog"></a>/var/log/cloud-init-output.log

Vous pouvez récupérer des informations à partir de `stdout` et `stderr` au cours des différentes [étapes de cloud-init](cloud-init-deep-dive.md). Cela implique généralement les informations de table de routage, de mise en réseau, de vérification de clé d’hôte SSH, de `stdout` et de `stderr` pour chaque étape de cloud-init, ainsi que leur horodatage. Si vous le souhaitez, la journalisation de `stderr` et `stdout` peut être reconfigurée à partir de `/etc/cloud/cloud.cfg.d/05_logging.cfg`.

### <a name="serialboot-logs"></a>Journaux de démarrage/série 

Cloud-init a plusieurs dépendances, celles-ci sont documentées dans les conditions préalables requises pour les images sur Azure, telles que la mise en réseau, le stockage, la possibilité de monter un fichier ISO et le montage et le formatage du disque temporaire. L’une d’entre elles peut lever des erreurs et provoquer l’échec de cloud-init. Par exemple, si la machine virtuelle ne peut pas obtenir un bail DHCP, cloud-init échoue.

Si vous ne pouvez toujours pas isoler la raison de l’échec de l’approvisionnement de cloud-init, vous devez comprendre les étapes de Cloud-init et le moment où les modules s’exécutent. Pour plus d’informations, consultez [Exploration approfondie de cloud-init](cloud-init-deep-dive.md).


## <a name="step-4-investigate-why-the-configuration-isnt-being-applied"></a>Étape 4 : Rechercher la raison pour laquelle la configuration n’est pas appliquée
Toutes les défaillances dans cloud-init entraînent un échec d’approvisionnement irrécupérable. Par exemple, si vous utilisez le module `runcmd` dans une configuration cloud-init, un code de sortie différent de zéro de la commande exécutée entraîne l’échec de l’approvisionnement de la machine virtuelle. Cela est dû au fait qu’il s’exécute après la fonctionnalité de configuration principale qui se produit au cours des trois premières phases de cloud-init. Pour résoudre les problèmes d’application des configurations, passez en revue les journaux à l’étape 3 et les modules cloud-init manuellement. Par exemple :

- `runcmd` : les scripts s’exécutent-ils sans erreur ? Exécutez la configuration manuellement à partir du terminal pour vous assurer qu’ils s’exécutent comme prévu.
- Installation de packages : la machine virtuelle a-t-elle accès aux référentiels de packages ?
- Vous devez également vérifier la configuration des données de `customData` qui a été fournie à la machine virtuelle, qui se trouve dans `/var/lib/cloud/instances/<unique-instance-identifier>/user-data.txt`.


## <a name="next-steps"></a>Étapes suivantes

Si vous ne pouvez toujours pas isoler la raison pour laquelle cloud-init n’a pas exécuté la configuration, vous devez examiner de plus près ce qui se passe dans chaque étape cloud-init, et quand les modules s’exécutent. Pour plus d’informations, consultez [Exploration approfondie de cloud-init](./cloud-init-deep-dive.md). 
