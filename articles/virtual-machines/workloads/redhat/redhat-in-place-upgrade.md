---
title: Mise à niveau sur place des images Red Hat Enterprise Linux sur Azure
description: Découvrez comment effectuer une mise à niveau sur place des images Red Hat Enterprise 7.x vers la dernière version 8.x.
author: mathapli
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 04/16/2020
ms.author: alsin
ms.reviewer: cynthn
ms.openlocfilehash: 9b35a3cbe23af91f7f0b8aceecfb8ba3c9720461
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484411"
---
# <a name="red-hat-enterprise-linux-in-place-upgrades"></a>Mises à niveau sur place de Red Hat Enterprise Linux

Cet article donne des instructions pour effectuer une mise à niveau sur place de Red Hat Enterprise Linux (RHEL) 7 vers Red Hat Enterprise Linux 8. Elles passent par l’outil `leapp` dans Azure. Au cours de la mise à niveau sur place, le système d’exploitation RHEL 7 existant est remplacé par la version RHEL 8.

>[!Note] 
> Les offres de SQL Server sur Red Hat Enterprise Linux ne prennent pas en charge les mises à niveau sur place sur Azure.

## <a name="what-to-expect-during-the-upgrade"></a>Ce qu’il faut attendre pendant la mise à niveau
Pendant la mise à niveau, le système redémarre plusieurs fois. Le dernier redémarrage met à niveau la machine virtuelle vers la dernière version mineure de RHEL 8. 

Le processus de mise à niveau peut durer entre 20 minutes et 2 heures. La durée totale dépend de plusieurs facteurs, par exemple la taille de la machine virtuelle et le nombre de packages installés sur le système.

## <a name="preparations"></a>Préparation
Red Hat et Azure recommandent d’appliquer une mise à niveau sur place pour faire passer un système à la version majeure suivante. 

Avant de commencer la mise à niveau, gardez à l’esprit les points suivants. 

>[!Important] 
> Prenez un instantané de l’image avant de commencer la mise à niveau.

* Veillez à utiliser la dernière version de RHEL 7. Il s’agit actuellement de la version 7.9. Si vous utilisez une version verrouillée et que vous ne pouvez pas effectuer la mise à niveau vers RHEL 7.9, suivez [cette procédure pour basculer vers un référentiel non EUS (mise à jour étendue)](./redhat-rhui.md#switch-a-rhel-7x-vm-back-to-non-eus-remove-a-version-lock).

* Exécutez la commande suivante pour vérifier votre mise à niveau et voir si elle se termine correctement. La commande doit générer un fichier */var/log/leapp/leapp-report.txt*. Ce fichier explique le processus, décrit son déroulement et indique si la mise à niveau est possible.

    >[!NOTE]
    > Utilisez le compte racine pour exécuter les commandes de cet article. 

    ```bash
    leapp preupgrade --no-rhsm
    ```
* Vérifiez que la console série est fonctionnelle. Vous l’utiliserez à des fins de monitoring pendant le processus de mise à niveau.

* Activez l’accès SSH racine dans */etc/ssh/sshd_config* :
    1. Ouvrez le fichier */etc/ssh/sshd_config*.
    1. Recherchez `#PermitRootLogin yes`.
    1. Supprimez le signe dièse (`#`) pour supprimer les marques de commentaire de la chaîne.

## <a name="upgrade-steps"></a>Étapes de la mise à niveau

Suivez cette procédure avec précaution. Nous vous recommandons de tester la mise à niveau sur un ordinateur de test avant de passer aux instances de production.

1. Effectuez une mise à jour `yum` pour récupérer les derniers packages clients.
    ```bash
    yum update -y
    ```

1. Installez `leapp-client-package`.
    ```bash
    yum install leapp-rhui-azure
    ```
    
1. Sur le [portail Red Hat](https://access.redhat.com/articles/3664871), récupérez le fichier *leapp-data.tar.gz* contenant *repomap.csv* et *pes-events.json*. Extrayez le fichier *leapp-data.tar.gz*.
    1. Téléchargez le fichier *leapp-data.tar.gz*.
    1. Extrayez le contenu et supprimez le fichier. Utilisez la commande suivante :
    ```bash
    tar -xzf leapp-data12.tar.gz -C /etc/leapp/files && rm leapp-data12.tar.gz
    ```

1. Ajoutez un fichier `answers` pour `leapp`.
    ```bash
    leapp answer --section remove_pam_pkcs11_module_check.confirm=True --add
    ``` 

1. Lancez la mise à niveau.
    ```bash
    leapp upgrade --no-rhsm
    ```
1.  Une fois l’exécution de la commande `leapp upgrade` terminée, redémarrez manuellement le système pour finir le processus. Le système n’est pas disponible, car il redémarre plusieurs fois. Surveillez le processus à l’aide de la console série.

1.  Vérifiez que la mise à niveau a abouti.
    ```bash
    uname -a && cat /etc/redhat-release
    ```

1. Une fois la mise à niveau terminée, supprimez l’accès SSH racine :
    1. Ouvrez le fichier */etc/ssh/sshd_config*.
    1. Recherchez `#PermitRootLogin yes`.
    1. Ajoutez un signe dièse (`#`) pour commenter la chaîne.

1. Redémarrez le service SSHD pour appliquer les modifications.
    ```bash
    systemctl restart sshd
    ```
## <a name="common-problems"></a>Problèmes courants

Les erreurs suivantes se produisent souvent en cas d’échec du processus `leapp preupgrade` ou du processus `leapp upgrade` :

* **Erreur** : Correspondances introuvables pour les modèles de plug-in désactivés suivants.

    ```plaintext
    STDERR:
    No matches found for the following disabled plugin patterns: subscription-manager
    Warning: Packages marked by Leapp for upgrade not found in repositories metadata: gpg-pubkey
    ```

    **Solution**: Désactivez le plug-in subscription-manager. Pour cela, modifiez le fichier */etc/yum/pluginconf.d/subscription-manager.conf* et remplacez `enabled` par `enabled=0`.

    Cette erreur se produit lorsque le plug-in subscription-manager `yum` activé n’est pas utilisé pour les machines virtuelles `PAYG`.

* **Erreur** : Problèmes possibles avec l’ouverture de session à distance à l’aide de la racine.

    Cette erreur risque de se produire en cas d’échec de `leapp preupgrade` :

    ```structured-text
    ============================================================
                         UPGRADE INHIBITED
    ============================================================
    
    Upgrade has been inhibited due to the following problems:
        1. Inhibitor: Possible problems with remote login using root account
    Consult the pre-upgrade report for details and possible remediation.
    
    ============================================================
                         UPGRADE INHIBITED
    ============================================================
    ```
    **Solution**: Activez l’accès racine dans */etc/sshd_config*.

    Cette erreur se produit lorsque l’accès SSH racine n’est pas activé dans */etc/sshd_config*. Pour plus d’informations, consultez la section [Préparation](#preparations) de cet article. 


## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur les [images Red Hat dans Azure](./redhat-images.md).
* En savoir plus sur [l’infrastructure de mise à jour Red Hat](./redhat-rhui.md).
* En savoir plus sur l’[offre BYOS RHEL](./byos.md).
* Pour plus d’informations sur les processus de mise à niveau sur place de Red Hat, consultez [Mise à niveau de RHEL 7 à RHEL 8](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/upgrading_from_rhel_7_to_rhel_8/index) dans la documentation de Red Hat.
* Pour plus d’informations sur les stratégies de prise en charge par Red Hat de toutes les versions de RHEL, consultez [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata) dans la documentation de Red Hat.