---
title: Comment configurer un serveur SMT pour SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Comment configurer un serveur SMT pour SAP HANA sur Azure (grandes instances).
services: virtual-machines-linux
documentationcenter: ''
author: hermanndms
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: hermannd
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 88b1cd2a5bf33d6401f0d2e15237400e27c72e54
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967650"
---
# <a name="set-up-smt-server-for-suse-linux"></a>Configuration de serveur SMT pour SUSE Linux
Les grandes instances SAP HANA n’ont pas de connexion directe à Internet. Le processus incluant l’inscription d’une telle unité auprès du fournisseur du système d’exploitation ainsi que le téléchargement et l’application des mises à jour n’est pas un simple. Avec SUSE Linux, une solution peut être de configurer un serveur SMT sur une machine virtuelle Azure. Hébergez la machine virtuelle dans un réseau virtuel Azure connecté à la grande instance HANA. Avec un tel serveur SMT, l’unité de grande instance HANA peut inscrire et télécharger des mises à jour. 

Pour plus d’informations sur SUSE, reportez-vous à son [outil de gestion des abonnements pour SLES 12 SP2](https://www.suse.com/documentation/sles-12/pdfdoc/book_smt/book_smt.pdf). 

Voici les conditions requises pour installer un serveur SMT accomplissant la tâche pour les grandes Instances HANA :

- un réseau virtuel Azure connecté au circuit ExpressRoute de la grande instance HANA.
- un compte SUSE associé à une organisation, L’organisation doit disposer d’un abonnement SUSE valide.

## <a name="install-smt-server-on-an-azure-virtual-machine"></a>Installation du serveur SMT sur une machine virtuelle Azure

Connectez-vous tout d’abord au [SUSE Customer Center](https://scc.suse.com/).

Accédez à **Organization** > **Organization Credentials**. Dans cette section, vous devez rechercher les informations d’identification nécessaires pour configurer le serveur SMT.

Installez ensuite une machine virtuelle SUSE Linux dans le réseau virtuel Azure. Pour déployer la machine virtuelle, prenez une image de la galerie SLES 12 SP2 d’Azure (sélectionnez l’image BYOS SUSE). Dans le processus de déploiement, ne définissez pas de nom DNS et n’utilisez pas d’adresses IP statiques.

![Capture d’écran du déploiement d’une machine virtuelle pour un serveur SMT](./media/hana-installation/image3_vm_deployment.png)

La machine virtuelle déployée est plus petite et a obtenu l’adresse IP interne du réseau virtuel Azure : 10.34.1.4. Le nom de la machine virtuelle est *smtserver*. Après l’installation, la connectivité aux unités de grande instance HANA est vérifiée. Selon la façon dont vous avez organisé la résolution de noms, il se peut que vous deviez configurer la résolution des unités de grande instance HANA dans le fichier etc/hosts de la machine virtuelle Azure. 

Ajoutez un disque à la machine virtuelle. Vous utilisez ce disque pour stocker les mises à jour, et le disque de démarrage lui-même pourrait être trop petit. Ici, le disque a été monté sur /srv/www/htdocs, comme indiqué dans la capture d’écran suivante. Un disque de 100 Go devrait suffire.

![Capture d’écran montrant le disque ajouté dans la fenêtre PuTTy.](./media/hana-installation/image4_additional_disk_on_smtserver.PNG)

Connectez-vous aux unités de grande instance HANA, conservez le fichier /etc/hosts et vérifiez si vous pouvez atteindre la machine virtuelle Azure supposée exécuter le serveur SMT sur le réseau.

Après cette vérification, connectez-vous à la machine virtuelle Azure qui doit exécuter le serveur SMT. Si vous utilisez PuTTY pour vous connecter à la machine virtuelle, exécutez cette séquence de commandes dans la fenêtre Bash :

```
cd ~
echo "export NCURSES_NO_UTF8_ACS=1" >> .bashrc
```

Redémarrez votre interpréteur de commandes Bash pour activer les paramètres. Ensuite, démarrez YaST.

Connectez votre machine virtuelle (smtserver) au site SUSE.

```
smtserver:~ # SUSEConnect -r <registration code> -e s<email address> --url https://scc.suse.com
Registered SLES_SAP 12.2 x86_64
To server: https://scc.suse.com
Using E-Mail: email address
Successfully registered system.
```

Une fois la machine virtuelle connectée au site SUSE, installez les packages smt. Saisissez la commande PuTTY suivante pour installer les packages smt.

```
smtserver:~ # zypper in smt
Refreshing service 'SUSE_Linux_Enterprise_Server_for_SAP_Applications_12_SP2_x86_64'.
Loading repository data...
Reading installed packages...
Resolving package dependencies...
```


Vous pouvez également utiliser l’outil YAST pour installer les packages smt. Dans YaST, accédez à **Software Maintenance** (Maintenance logicielle), puis recherchez smt. Sélectionnez **smt**, qui bascule automatiquement vers yast2-smt.

![Capture d’écran de SMT dans YAST](./media/hana-installation/image5_smt_in_yast.PNG)


Acceptez la sélection pour l’installation sur smtserver. Une fois l’installation terminée, accédez à la configuration du serveur SMT. Entrez les informations d'identification de votre organisation que vous avez récupérées précédemment à partir du SUSE Customer Center. Entrez également le nom d’hôte de votre machine virtuelle Azure en tant qu’URL du serveur SMT. Cette démonstration utilise https:\//smtserver.

![Capture d’écran d’une configuration de serveur SMT](./media/hana-installation/image6_configuration_of_smtserver1.png)

Vérifiez maintenant si la connexion au SUSE Customer Center fonctionne. Comme le montre la capture d’écran suivante, dans le cas de la démonstration, cela n’a pas fonctionné.

![Capture d’écran d’un test de connexion au SUSE Customer Center](./media/hana-installation/image7_test_connect.png)

Une fois le programme d’installation de SMT lancé, fournissez un mot de passe de base de données. Comme il s’agit d’une nouvelle installation, vous devez définir ce mot de passe, comme indiqué dans la capture d’écran suivante.

![Capture d’écran de la définition du mot de passe pour la base de données](./media/hana-installation/image8_define_db_passwd.PNG)

L’étape suivante consiste à créer un certificat.

![Capture d’écran de la création d’un certificat pour le serveur SMT](./media/hana-installation/image9_certificate_creation.PNG)

À la fin de la configuration, la vérification de la synchronisation peut prendre quelques minutes. Une fois l’installation et la configuration du serveur SMT terminées, vous devez rechercher le référentiel de répertoire sous le point de montage /srv/www/htdocs/. Le référentiel contient également certains sous-répertoires. 

Redémarrez le serveur SMT et ses services associés avec ces commandes.

```
rcsmt restart
systemctl restart smt.service
systemctl restart apache2
```

## <a name="download-packages-onto-smt-server"></a>Télécharger des packages sur le serveur SMT

Une fois que tous les services ont redémarré, sélectionnez les packages appropriés dans SMT Management (Gestion de SMT) à l’aide de YaST. La sélection du package dépend de l’image du système d’exploitation du serveur de la grande instance HANA. La sélection du package ne dépend pas de la version SLES ou de la machine virtuelle exécutant le serveur SMT. La capture d’écran suivante présente un exemple de l’écran de sélection.

![Capture d’écran de sélection de packages](./media/hana-installation/image10_select_packages.PNG)

Lancez ensuite la copie initiale des packages sélectionnés sur le serveur SMT que vous configurez. Cette copie est déclenchée dans l’interpréteur de commandes à l’aide de la commande smt-mirror.

![Capture d’écran de téléchargement des packages sur le serveur SMT](./media/hana-installation/image11_download_packages.PNG)

Les packages doivent être copiés dans les répertoires créés sous le montage point /srv/www/htdocs. Ce processus peut prendre une ou plusieurs heures selon le nombre de packages que vous sélectionnez. Lorsque ce processus s’achève, passez à la configuration du client SMT. 

## <a name="set-up-the-smt-client-on-hana-large-instance-units"></a>Configurer le client SMT sur des unités de grande instance HANA

Dans ce cas, les clients sont les unités de grande instance HANA. Le programme d’installation du serveur SMT a copié le script clientSetup4SMT.sh sur la machine virtuelle Azure. Copiez ce script sur l’unité de grande instance HANA que vous souhaitez connecter à votre serveur SMT. Démarrez le script avec l’option -h, puis donnez-lui en guise de paramètre le nom de votre serveur SMT. Dans cet exemple, le nom est *smtserver*.

![Capture d’écran de configuration du client SMT](./media/hana-installation/image12_configure_client.PNG)

Il peut y avoir un scénario où le chargement du certificat à partir du serveur par le client réussit, mais où l’inscription échoue, comme le montre la capture d’écran suivante.

![Capture d’écran d’une inscription de client ayant échoué](./media/hana-installation/image13_registration_failed.PNG)

Si l’inscription échoue, lisez le [document de support SUSE](https://www.suse.com/de-de/support/kb/doc/?id=7006024), puis effectuez les étapes qu’il décrit.

> [!IMPORTANT] 
> Pour le nom du serveur, fournissez le nom de la machine virtuelle, en l’occurrence *smtserver*, sans le nom de domaine complet. 

Après avoir effectué ces étapes, exécutez la commande suivante sur l’unité de grande instance HANA :

```
SUSEConnect –cleanup
```

> [!Note] 
> Attendez quelques minutes après cette étape. Si vous exécutez clientSetup4SMT.sh immédiatement, une erreur peut s’afficher.

Si vous rencontrez un problème que vous devez résoudre en suivant les étapes de l’article sur SUSE, redémarrez clientSetup4SMT.sh sur l’unité de grande instance HANA. L’opération devrait à présent aboutir correctement.

![Capture d’écran d’une inscription de client réussie](./media/hana-installation/image14_finish_client_config.PNG)

Vous avez configuré le client SMT de l’unité de grande instance HANA pour qu’il se connecte au serveur SMT que vous avez installé sur la machine virtuelle Azure. Vous pouvez à présent utiliser « zypper up » ou « zypper in » pour installer des mises à jour du système d’exploitation sur des grandes instances HANA, ou installer des packages supplémentaires. Vous pouvez uniquement obtenir des mises à jour que vous avez téléchargées auparavant sur le serveur SMT.

## <a name="next-steps"></a>Étapes suivantes
- [Installation de HANA sur HLI](hana-example-installation.md).











