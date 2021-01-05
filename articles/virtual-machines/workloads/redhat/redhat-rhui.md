---
title: Infrastructure de mise à jour Red Hat | Microsoft Docs
description: Découvrez l’infrastructure de mise à jour Red Hat pour les instances Red Hat Enterprise Linux à la demande dans Microsoft Azure
author: asinn826
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 02/10/2020
ms.author: alsin
ms.reviewer: cynthn
ms.openlocfilehash: d4af869a3bf4ba7f454ae8e5c9c9f4eb81f5939f
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94957450"
---
# <a name="red-hat-update-infrastructure-for-on-demand-red-hat-enterprise-linux-vms-in-azure"></a>Infrastructure de mise à jour Red Hat pour machines virtuelles Red Hat Enterprise Linux à la demande dans Azure
 [Infrastructure de mise à jour Red Hat](https://access.redhat.com/products/red-hat-update-infrastructure) (RHUI) permet aux fournisseurs de cloud, par exemple Azure, de mettre en miroir le contenu du référentiel hébergé par Red Hat, de créer des référentiels personnalisés avec du contenu spécifique à Azure et de rendre ces référentiels accessibles aux machines virtuelles des utilisateurs finaux.

Les images Red Hat Enterprise Linux (RHEL) Pay-As-You-Go (PAYG, paiement à l’utilisation) sont préconfigurées pour accéder à Azure RHUI. Aucune configuration supplémentaire n’est nécessaire. Pour obtenir les dernières mises à jour, exécutez `sudo yum update` lorsque votre instance RHEL est prête. Ce service est inclus dans le cadre des frais de logiciel RHEL PAYG.

Des informations supplémentaires sur les images RHEL dans Azure, y compris les stratégies de publication et de conservation, sont disponibles [ici](./redhat-images.md).

Vous trouverez des informations sur les stratégies de prise en charge de Red Hat pour toutes les versions de RHEL sur la page [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).

> [!IMPORTANT]
> RHUI est destiné uniquement aux images de paiement à l’utilisation (PAYG). Pour les images personnalisées et golden, également appelées BYOS (Bring-Your-Own-Subscription), le système doit être attaché à RHSM ou Satellite pour recevoir les mises à jour. Consultez l’[article Red Hat](https://access.redhat.com/solutions/253273) pour plus de détails.


## <a name="important-information-about-azure-rhui"></a>Informations importantes concernant Azure RHUI

* Azure RHUI est l’infrastructure de mise à jour qui prend en charge toutes les machines virtuelles RHEL PAYG créées dans Azure. Cela ne vous empêche pas d’inscrire vos machines virtuelles PAYG RHEL avec le Gestionnaire d’abonnements ou Satellite, ou via toute autre source de mise à jour. Cependant, en procédant ainsi avec une machine virtuelle PAYG, vous créerez une double facturation indirecte. Consultez le point suivant pour en savoir plus.
* L’accès à l’infrastructure RHUI hébergée sur Azure est inclus dans le prix de l’image RHEL PAYG. Si vous annulez l’inscription à l’infrastructure RHUI hébergée sur Azure d’une machine virtuelle RHEL avec paiement à l’utilisation, cela ne convertit pas la machine virtuelle en une machine virtuelle de type BYOL (apportez votre propre licence). Si vous inscrivez la même machine virtuelle avec une autre source de mises à jour, vous pouvez être confronté à des frais doubles _indirects_. Vous êtes facturés une première fois pour les frais du logiciel RHEL de Azure. Vous êtes facturé une deuxième fois pour les abonnements Red Hat achetés précédemment. Si vous devez systématiquement utiliser une infrastructure de mise à jour autre que l’infrastructure RHUI hébergée sur Azure, envisagez l’inscription afin de pouvoir utiliser les [images BYOS RHEL](./byos.md).

* Les images RHEL SAP PAYG dans Azure (RHEL for SAP HANA, RHEL for SAP Business Applications) sont connectées aux canaux RHUI dédiés qui restent sur la version mineure RHEL spécifique nécessaire pour la certification SAP.

* L’accès à l’infrastructure RHUI hébergée sur Azure est limité aux machines virtuelles figurant dans les [plages IP du centre de données Azure](https://www.microsoft.com/download/details.aspx?id=41653). Si vous redirigez à l’aide d’un proxy tout le trafic de la machine virtuelle via une infrastructure réseau locale, vous devrez peut-être configurer des itinéraires définis par l’utilisateur pour permettre aux machines virtuelles RHEL PAYG d’accéder à Azure RHUI. Si c’est le cas, les routes définies par l’utilisateur doivent être ajoutées pour _toutes_ les adresses IP RHUI.


## <a name="image-update-behavior"></a>Comportement de mise à jour des images

Depuis le mois d’avril 2019, Azure fournit des images RHEL qui sont connectées à des dépôts EUS (Extended Update Support) par défaut et des images RHEL qui sont connectées aux dépôts standard (non EUS) par défaut. Vous trouverez plus de détails sur les EUS RHEL dans la [documentation sur le cycle de vie des versions](https://access.redhat.com/support/policy/updates/errata) et dans la [documentation sur EUS](https://access.redhat.com/articles/rhel-eus) de Red Hat. Le comportement par défaut de `sudo yum update` varie en fonction de l’image RHEL à partir de laquelle vous avez effectué le provisionnement, car les différentes images sont connectées à des dépôts différents.

Pour obtenir la liste complète des images, exécutez `az vm image list --publisher redhat --all` avec Azure CLI.

### <a name="images-connected-to-non-eus-repositories"></a>Images connectées à des dépôts non EUS

Si vous provisionnez une machine virtuelle à partir d’une image RHEL qui est connectée à des dépôts non EUS, votre version est mise à niveau vers la dernière version mineure de RHEL quand vous exécutez `sudo yum update`. Par exemple, si vous approvisionnez une machine virtuelle à partir d'une image PAYG RHEL 7.4 et que vous exécutez `sudo yum update`, vous obtenez une machine virtuelle RHEL 7.8 (version mineure la plus récente de la famille RHEL7).

Les images connectées à des dépôts non EUS ne comportent pas de numéro de version mineure dans la référence SKU. La référence SKU est le troisième élément de l’URN (nom complet de l’image). Par exemple, toutes les images suivantes sont fournies en étant déjà connectées à des dépôts non EUS :

```text
RedHat:RHEL:7-LVM:7.4.2018010506
RedHat:RHEL:7-LVM:7.5.2018081518
RedHat:RHEL:7-LVM:7.6.2019062414
RedHat:RHEL:7-RAW:7.4.2018010506
RedHat:RHEL:7-RAW:7.5.2018081518
RedHat:RHEL:7-RAW:7.6.2019062120
```

Notez que les références SKU sont de la version 7-LVM ou 7-RAW. La version mineure est indiquée dans la version (quatrième élément de l’URN) de ces images.

### <a name="images-connected-to-eus-repositories"></a>Images connectées à des dépôts EUS

Si vous provisionnez une machine virtuelle à partir d’une image RHEL qui est connectée à des dépôts EUS, votre version n’est pas mise à niveau vers la dernière version mineure de RHEL quand vous exécutez `sudo yum update`. Cela est dû au fait que la version des images connectées à des dépôts EUS est également verrouillée sur leur version mineure spécifique.

Les images connectées à des dépôts EUS comportent un numéro de version mineure dans la référence SKU. Par exemple, toutes les images suivantes sont fournies en étant déjà connectées à des dépôts EUS :

```text
RedHat:RHEL:7.4:7.4.2019062107
RedHat:RHEL:7.5:7.5.2019062018
RedHat:RHEL:7.6:7.6.2019062116
```

## <a name="rhel-eus-and-version-locking-rhel-vms"></a>EUS RHEL et machines virtuelles RHEL avec verrouillage de version

Les référentiels EUS (Extended Update Support) sont proposés aux clients qui souhaitent verrouiller leurs machines virtuelles RHEL sur une certaine version mineure RHEL après provisionnement de la machine virtuelle. Vous pouvez verrouiller la version de votre machine virtuelle RHEL sur une version mineure spécifique en mettant à jour les dépôts pour qu’ils pointent vers les dépôts Extended Update Support. Vous pouvez également annuler l’opération de verrouillage de version EUS.

>[!NOTE]
> La technologie EUS n’est pas prise en charge par RHEL Extras. Cela signifie que vous ne pourrez pas, sur EUS, installer un package généralement disponible à partir du canal RHEL Extras. Le cycle de vie des produits Red Hat Extras est détaillé [ici](https://access.redhat.com/support/policy/updates/extras/).

Au moment de la rédaction de ce document, la prise en charge d’EUS est terminée pour RHEL 7.4 et les versions antérieures. Pour en savoir plus, consultez la section « Maintenance prolongée de Red Hat Enterprise Linux » de la [documentation Red Hat](https://access.redhat.com/support/policy/updates/errata/#Long_Support).
* La prise en charge de RHEL 7.4 EUS se terminera le 31 août 2019.
* La prise en charge de RHEL 7.5 EUS se terminera le 30 avril 2020.
* La prise en charge de RHEL 7.6 EUS se terminera le 31 mai 2021.
* La prise en charge de RHEL 7.7 EUS se terminera le 30 août 2021.

### <a name="switch-a-rhel-vm-7x-to-eus-version-lock-to-a-specific-minor-version"></a>Faire passer une machine virtuelle RHEL 7.x à EUS (verrouillage sur une version mineure spécifique)
Utilisez les instructions suivantes pour verrouiller une machine virtuelle RHEL 7.x sur une version mineure spécifique (à exécuter comme utilisateur root) :

>[!NOTE]
> Ceci s’applique uniquement aux versions RHEL 7.x pour laquelle EUS est disponible. Au moment de la rédaction de cet article, cela inclut RHEL 7.2-7.7. Pour plus de détails, voir la page [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).

1. Désactivez les dépôts non-EUS :
    ```bash
    yum --disablerepo='*' remove 'rhui-azure-rhel7'
    ```

1. Ajoutez les dépôts EUS :
    ```bash
    yum --config='https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel7-eus.config' install 'rhui-azure-rhel7-eus'
    ```

1. Verrouillez la variable `releasever` (en exécutant les opérations en tant que racine) :
    ```bash
    echo $(. /etc/os-release && echo $VERSION_ID) > /etc/yum/vars/releasever
    ```

    >[!NOTE]
    > L’instruction ci-dessus verrouille la version mineure de RHEL sur la version mineure actuelle. Entrez une version mineure spécifique si vous souhaitez mettre à niveau et verrouiller une version mineure ultérieure qui n’est pas la dernière version. Par exemple, `echo 7.5 > /etc/yum/vars/releasever` verrouille votre version de RHEL sur RHEL 7.5.

1. Mettre à jour votre machine virtuelle RHEL
    ```bash
    sudo yum update
    ```

### <a name="switch-a-rhel-vm-8x-to-eus-version-lock-to-a-specific-minor-version"></a>Faire passer une machine virtuelle RHEL 8.x à EUS (verrouillage sur une version mineure spécifique)
Utilisez les instructions suivantes pour verrouiller une machine virtuelle RHEL 8.x sur une version mineure spécifique (à exécuter comme utilisateur root) :

>[!NOTE]
> Ceci s’applique uniquement aux versions RHEL 8.x pour laquelle EUS est disponible. Au moment de la rédaction de cet article, ceci inclut RHEL 8.1 - 8.2. Pour plus de détails, voir la page [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).

1. Désactivez les dépôts non-EUS :
    ```bash
    yum --disablerepo='*' remove 'rhui-azure-rhel8'
    ```

1. Procurez-vous le fichier de configuration de dépôt EUS :
    ```bash
    wget https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel8-eus.config
    ```

1. Ajoutez les dépôts EUS :
    ```bash
    yum --config=rhui-microsoft-azure-rhel8-eus.config install rhui-azure-rhel8-eus
    ```

1. Verrouillez la variable `releasever` (en exécutant les opérations en tant que racine) :
    ```bash
    echo $(. /etc/os-release && echo $VERSION_ID) > /etc/yum/vars/releasever
    ```

    >[!NOTE]
    > L’instruction ci-dessus verrouille la version mineure de RHEL sur la version mineure actuelle. Entrez une version mineure spécifique si vous souhaitez mettre à niveau et verrouiller une version mineure ultérieure qui n’est pas la dernière version. Par exemple, `echo 8.1 > /etc/yum/vars/releasever` verrouille votre version RHEL sur RHEL 8.1.

    >[!NOTE]
    > S’il existe des problèmes d’autorisation pour accéder au releasever, vous pouvez éditer le fichier en utilisant « nano /etc/yum/vars/releaseve », ajouter les détails de la version de l’image et enregistrer (« Ctrl+O », puis appuyer sur Entrée, puis sur « Ctrl+X »).  

1. Mettre à jour votre machine virtuelle RHEL
    ```bash
    sudo yum update
    ```


### <a name="switch-a-rhel-7x-vm-back-to-non-eus-remove-a-version-lock"></a>Rétablir une machine virtuelle RHEL 7.x à une version non-EUS (supprimer un verrouillage de version)
Exécutez l’opération suivante, en tant que racine :
1. Supprimez le fichier `releasever` :
    ```bash
    rm /etc/yum/vars/releasever
     ```

1. Désactivez le référentiel EUS :
    ```bash
    yum --disablerepo='*' remove 'rhui-azure-rhel7-eus'
   ```

1. Configurer la machine virtuelle RHEL
    ```bash
    yum --config='https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel7.config' install 'rhui-azure-rhel7'
    ```

1. Mettre à jour votre machine virtuelle RHEL
    ```bash
    sudo yum update
    ```

### <a name="switch-a-rhel-8x-vm-back-to-non-eus-remove-a-version-lock"></a>Rétablir une machine virtuelle RHEL 8.x à une version non-EUS (supprimer un verrouillage de version)
Exécutez l’opération suivante, en tant que racine :
1. Supprimez le fichier `releasever` :
    ```bash
    rm /etc/yum/vars/releasever
     ```

1. Désactivez le référentiel EUS :
    ```bash
    yum --disablerepo='*' remove 'rhui-azure-rhel8-eus'
   ```

1. Procurez-vous le fichier de configuration de dépôt standard :
    ```bash
    wget https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel8.config
    ```

1. Ajoutez les dépôts EUS :
    ```bash
    yum --config=rhui-microsoft-azure-rhel8.config install rhui-azure-rhel8
    ```
    
1. Mettre à jour votre machine virtuelle RHEL
    ```bash
    sudo yum update
    ```

## <a name="the-ips-for-the-rhui-content-delivery-servers"></a>Adresses IP des serveurs de distribution de contenu RHUI

L’infrastructure RHUI est disponible dans toutes les régions où les images RHEL à la demande sont disponibles. Cela inclut actuellement toutes les régions publiques répertoriées sur la page [Tableau de bord Statut Azure](https://azure.microsoft.com/status/) et les régions Microsoft Azure du gouvernement des États-Unis et d’Allemagne.

Si vous utilisez une configuration du réseau pour restreindre davantage l’accès à partir de machines virtuelles RHEL PAYG, assurez-vous que les adresses IP suivantes sont autorisées pour que `yum update` fonctionne en fonction de votre environnement :


```
# Azure Global
13.91.47.76
40.85.190.91
52.187.75.218
52.174.163.213
52.237.203.198

# Azure US Government
13.72.186.193
13.72.14.155
52.244.249.194

# Azure Germany
51.5.243.77
51.4.228.145
```
>[!NOTE]
>Les nouvelles images Azure US Government, à compter de janvier 2020, utiliseront une adresse IP publique mentionnée dans l’en-tête Azure Global ci-dessus.

>[!NOTE]
>Notez également qu’Azure Allemagne est déconseillé en faveur des régions publiques d’Allemagne. Pour les clients Azure Allemagne, il est recommandé de commencer par pointer sur les RHUI publics à l’aide de [ces étapes](#manual-update-procedure-to-use-the-azure-rhui-servers).

## <a name="azure-rhui-infrastructure"></a>Infrastructure Azure RHUI


### <a name="update-expired-rhui-client-certificate-on-a-vm"></a>Mettre à jour le certificat de client RHUI arrivé à expiration sur une machine virtuelle

Si vous utilisez une ancienne image de machine virtuelle RHEL, par exemple, RHEL 7.4 (URN d’image : `RedHat:RHEL:7.4:7.4.2018010506`), vous rencontrerez des problèmes de connectivité à RHUI en raison d’un certificat de client TLS/SSL arrivé à expiration. L’erreur que vous voyez peut ressembler à _« L’homologue SSL a rejeté le certificat car il a expiré »_ ou _« Erreur : Impossible de récupérer les métadonnées du référentiel (repomd.xml) pour le référentiel :... Vérifiez son chemin d’accès et essayez à nouveau »_ . Pour résoudre ce problème, mettez à jour le package client RHUI sur la machine virtuelle à l’aide de la commande suivante :

```bash
sudo yum update -y --disablerepo='*' --enablerepo='*microsoft*'
```

Sinon, l’exécution de `sudo yum update` met aussi à jour le package de certificat client (en fonction de la version de votre RHEL) en dépit des erreurs « certificat SSL expiré » que vous voyez pour d’autres dépôts. Si la mise à jour fonctionne, une connectivité normale à d’autres dépôts RHUI doit être restaurée, vous pouvez donc exécuter `sudo yum update`.

Si vous rencontrez une erreur 404 lorsque vous exécutez `yum update`, essayez la procédure suivante pour actualiser votre cache yum :
```bash
sudo yum clean all;
sudo yum makecache
```

### <a name="troubleshoot-connection-problems-to-azure-rhui"></a>Résoudre des problèmes de connexion à Azure RHUI
Si vous rencontrez des problèmes de connexion à RHUI Azure à partir de votre machine virtuelle Azure RHEL PAYG, procédez comme suit :

1. Examinez la configuration de machine virtuelle pour le point de terminaison RHUI Azure :

    1. Vérifiez si le fichier `/etc/yum.repos.d/rh-cloud.repo` contient la référence à `rhui-[1-3].microsoft.com` dans la `baseurl` de la section `[rhui-microsoft-azure-rhel*]` du fichier. Si c’est le cas, vous utilisez la nouvelle version de RHUI Azure.

    1. Si elle pointe vers un emplacement avec le modèle suivant; `mirrorlist.*cds[1-4].cloudapp.net`, la mise à jour de la configuration est requise. Vous utilisez une ancienne capture instantanée de la machine virtuelle, et vous devez la mettre à jour pour pointer vers la nouvelle RHUI Azure.

1. L’accès à la RHUI hébergée par Azure est limité aux machines virtuelles dans les [plages d’adresses IP des centres de données Azure](https://www.microsoft.com/download/details.aspx?id=41653).

1. Si vous utilisez la nouvelle configuration et que vous ne parvenez toujours pas à vous connecter à RHUI Azure après avoir vérifié que la machine virtuelle se connecte à l’aide de la plage d’adresses IP Azure, ouvrez une demande de support avec Microsoft ou Red Hat.

### <a name="infrastructure-update"></a>Mise à jour de l’infrastructure

En septembre 2016, nous avons déployé une mise à jour de RHUI Azure. En avril 2017, nous avons mis fin à l’ancien RHUI Azure. Si vous avez utilisé les images RHEL PAYG (ou leurs captures instantanées) à partir de septembre 2016, vous vous connectez automatiquement à la nouvelle RHUI Azure. Si toutefois vous avez des captures instantanées plus anciennes sur vos machines virtuelles, vous devez mettre à jour leur configuration manuellement pour accéder à la RHUI Azure, comme décrit dans la section suivante.

Les nouveaux serveurs RHUI Azure sont déployés avec [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/). Dans Traffic Manager, un simple point de terminaison (rhui-1.microsoft.com) peut être utilisé par n’importe quelle machine virtuelle, quelle que soit la région.

### <a name="manual-update-procedure-to-use-the-azure-rhui-servers"></a>Procédure de mise à jour manuelle pour utiliser les serveurs RHUI Azure
Cette procédure est fournie uniquement à titre de référence. Les images RHEL PAYG possèdent déjà la configuration correcte pour se connecter à la RHUI Azure. Pour mettre à jour manuellement la configuration afin d’utiliser les serveurs RHUI Azure, procédez comme suit :

- Pour RHEL 6 :
  ```bash
  yum --config='https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel6.config' install 'rhui-azure-rhel6'
  ```

- Pour RHEL 7 :
  ```bash
  yum --config='https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel7.config' install 'rhui-azure-rhel7'
  ```

- Pour RHEL 8 :
    1. Création d’un fichier config :
        ```bash
        vi rhel8.config
        ```
    1. Ajoutez le contenu suivant au fichier config :
        ```bash
        [rhui-microsoft-azure-rhel8]
        name=Microsoft Azure RPMs for Red Hat Enterprise Linux 8
        baseurl=https://rhui-1.microsoft.com/pulp/repos/microsoft-azure-rhel8 https://rhui-2.microsoft.com/pulp/repos/microsoft-azure-rhel8 https://rhui-3.microsoft.com/pulp/repos/microsoft-azure-rhel8
        enabled=1
        gpgcheck=1
        gpgkey=https://rhelimage.blob.core.windows.net/repositories/RPM-GPG-KEY-microsoft-azure-release sslverify=1
        ```
    1. Enregistrez le fichier et exécutez la commande suivante :
        ```bash
        dnf --config rhel8.config install 'rhui-azure-rhel8'
        ```
    1. Mettre à jour votre machine virtuelle
        ```bash
        sudo dnf update
        ```


## <a name="next-steps"></a>Étapes suivantes
* Pour créer une machine virtuelle Red Hat Enterprise Linux à partir d’une image PAYG dans le service Marketplace Azure et utiliser l’infrastructure RHUI hébergée sur Azure, voir [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/RedHat.RHEL_6).
* Pour en savoir plus sur les images Red Hat dans Azure, accédez à la [page de documentation](./redhat-images.md).
* Vous trouverez des informations sur les stratégies de prise en charge de Red Hat pour toutes les versions de RHEL sur la page [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).
