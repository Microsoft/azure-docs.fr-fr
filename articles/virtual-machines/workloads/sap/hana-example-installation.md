---
title: Installation de HANA sur SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Installation de HANA sur SAP HANA sur Azure (grandes instances).
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
ms.openlocfilehash: 994130ce2ecb4578a8d34f1f77b310bb558d3c7a
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967905"
---
# <a name="install-hana-on-sap-hana-on-azure-large-instances"></a>Installation de HANA sur SAP HANA sur Azure (grandes instances)

Pour installer HANA sur SAP HANA sur Azure (grandes instances), vous devez tout d’abord procéder comme suit :
- Vous devez fournir à Microsoft toutes les données pour déployer pour vous une grande instance SAP HANA.
- Vous devez recevoir la grande instance SAP HANA de Microsoft.
- Vous devez créer un réseau virtuel Azure qui est connecté à votre réseau local.
- Vous devez connecter le circuit ExpressRoute pour les grandes instances HANA au même réseau virtuel Azure.
- Vous devez installer une machine virtuelle Azure que vous utilisez comme serveur intermédiaire pour les grandes instances HANA.
- Vous devez vérifier que vous pouvez vous connecter à partir du serveur intermédiaire à l’unité de grande instance HANA et inversement.
- Vous devez vérifier que tous les packages et correctifs requis sont installés.
- Vous devez lire la documentation et les notes SAP qui traitent de l’installation de HANA sur le système d’exploitation que vous utilisez. Assurez-vous que la version de HANA que vous avez choisie est prise en charge sur la version de votre système d’exploitation.

La section suivante explique comment télécharger les packages d’installation de HANA sur la machine virtuelle faisant office de serveur intermédiaire. Dans ce cas, le système d’exploitation est Windows.

## <a name="download-the-sap-hana-installation-bits"></a>Télécharger les bits d’installation de SAP HANA
Les unités de grande instance HANA ne sont pas directement connectées à Internet. Vous ne pouvez pas télécharger directement les packages d’installation depuis SAP vers la machine virtuelle de grande instance HANA. Au lieu de cela, vous téléchargez les packages sur la machine virtuelle du serveur intermédiaire.

Vous avez besoin d’un utilisateur S ou d’un autre utilisateur SAP, qui vous permet d’accéder à la place de marché SAP.

1. Connectez-vous, puis accédez à [SAP Service Marketplace](https://support.sap.com/en/index.html) (Place de marché de service SAP). Sélectionnez **Download Software** > **Installations and Upgrade** > **By Alphabetical Index** (Télécharger le logiciel > Installations et mise à niveau > Par index alphabétique). Ensuite, sélectionnez **Under H – SAP HANA Platform Edition** > **SAP HANA Platform Edition 2.0** > **Installation** (Sous H – Édition de plateforme SAP HANA > Édition de plateforme SAP HANA 2.0 > Installation). Téléchargez les fichiers qui s’affichent dans la capture d’écran suivante.

   ![Capture d’écran des fichiers à télécharger](./media/hana-installation/image16_download_hana.PNG)

2. Dans cet exemple, nous avons téléchargé les packages d’installation SAP HANA 2.0. Sur la machine virtuelle Azure faisant office de serveur intermédiaire, développez les archives à extraction automatique dans le répertoire, comme illustré ci-dessous.

   ![Capture d’écran de l’archive à extraction automatique](./media/hana-installation/image17_extract_hana.PNG)

3. Une fois les archives extraites, copiez le répertoire qui a été créé par l’extraction (dans ce cas, 51052030). Copiez le répertoire à partir du volume /hana/shared de l’unité de grande instance HANA dans un répertoire que vous avez créé.

   > [!Important]
   > Ne copiez pas les packages d’installation dans la racine ou le LUN de démarrage, car l’espace est limité et doit être utilisé également par d’autres processus.


## <a name="install-sap-hana-on-the-hana-large-instance-unit"></a>Installer SAP HANA sur l’unité de grande instance HANA
Pour installer SAP HANA, connectez-vous en tant qu’utilisateur racine. Seule la racine dispose d’autorisations suffisantes pour installer SAP HANA. Définissez des autorisations sur le répertoire que vous avez copié dans /hana/shared.

```
chmod –R 744 <Installation bits folder>
```

Si vous souhaitez installer SAP HANA à l’aide de la configuration d’interface graphique utilisateur, le package gtk2 doit être installé sur les grandes instances HANA. Pour voir si cette configuration est installée, exécutez la commande suivante :

```
rpm –qa | grep gtk2
```

(Dans des étapes ultérieures, nous verrons la configuration de SAP HANA avec l’interface utilisateur graphique.)

Accédez au répertoire d’installation, puis naviguez dans le sous-répertoire HDB_LCM_LINUX_X86_64. 

Commencez en dehors de ce répertoire :

```
./hdblcmgui 
```
À ce stade, vous passez d’un écran à l’autre et vous fournissez les données pour l’installation. Dans cet exemple, nous installons le serveur de base de données SAP HANA et les composants du client SAP HANA. Par conséquent, notre sélection est **SAP HANA Database** (Base de données SAP HANA).

![Capture d’écran de la gestion du cycle de vie de SAP HANA, avec la base de données SAP HANA sélectionnée](./media/hana-installation/image18_hana_selection.PNG)

Dans l’écran suivant, sélectionnez **Install New System** (Installer un nouveau système).

![Capture d’écran de la gestion du cycle de vie de SAP HANA, avec l’option d’installation d’un nouveau système sélectionnée](./media/hana-installation/image19_select_new.PNG)

Ensuite, sélectionnez les composants supplémentaires que vous voulez installer.

![Capture d’écran de la gestion du cycle de vie de SAP HANA, avec la liste des composants supplémentaires](./media/hana-installation/image20_select_components.PNG)

Ici, nous choisissons le client SAP HANA et le Studio SAP HANA. Nous avons également installé une instance de montée en puissance. Puis, choisissez **Single-Host System** (Système à un seul hôte). 

![Capture d’écran de la gestion du cycle de vie de SAP HANA, avec un système à un seul hôte sélectionné](./media/hana-installation/image21_single_host.PNG)

Ensuite, renseignez certaines données.

![Capture d’écran de la gestion du cycle de vie de SAP HANA, avec les champs des propriétés système à renseigner](./media/hana-installation/image22_provide_sid.PNG)

> [!Important]
> En tant qu’ID système (SID) HANA, vous devez fournir le même SID que celui fourni à Microsoft lors de la commande du déploiement de la grande instance HANA. Le choix d’un SID différent fait échouer l’installation en raison de problèmes d’autorisation d’accès sur les différents volumes.

Pour le chemin d'accès de l’installation, utilisez le répertoire /hana/shared. À l’étape suivante, vous devez fournir les emplacements des fichiers de données et des fichiers journaux de HANA.


![Capture d’écran de la gestion du cycle de vie de SAP HANA, avec les champs de données et de journaux](./media/hana-installation/image23_provide_log.PNG)

> [!Note]
> Le SID que vous avez spécifié lorsque vous avez défini les propriétés système (deux écrans au préalable) doit correspondre au SID des points de montage. Si tel n’est pas le cas, revenez en arrière et définissez la valeur du SID sur celle dont disposent les points de montage.

À l’étape suivante, révisez le nom d’hôte et corrigez-le éventuellement. 

![Capture d’écran de la gestion du cycle de vie de SAP HANA, avec le nom d'hôte](./media/hana-installation/image24_review_host_name.PNG)

À l’étape suivante, vous devez également récupérer les données que vous avez communiquées à Microsoft lors de la commande du déploiement de la grande instance HANA. 

![Capture d’écran de la gestion du cycle de vie de SAP HANA, avec les champs administrateur système à renseigner](./media/hana-installation/image25_provide_guid.PNG)

> [!Important]
> Vous devez fournir les mêmes **ID d’administrateur système** et **ID de groupe d’utilisateurs** que ceux fournis à Microsoft lors de la commande du déploiement de l’unité. Si vous ne le faites pas, l’installation de SAP HANA sur l’unité de grande instance HANA échoue.

Les deux écrans suivants ne sont pas illustrés ici. Ils vous permettent de fournir le mot de passe pour l’utilisateur système de la base de données SAP HANA et le mot de passe pour l’utilisateur sapadm. Ce dernier est utilisé pour l’agent hôte SAP, qui est installé avec l’instance de base de données SAP HANA.

Une fois le mot de passe défini, un écran de confirmation s’affiche. Vérifiez toutes les données affichées, puis poursuivez l’installation. Vous accédez à un écran montrant la progression de l’installation, comme celui-ci :

![Capture d’écran de la gestion du cycle de vie de SAP HANA, avec les indicateurs de progression de l’installation](./media/hana-installation/image27_show_progress.PNG)

Une fois l’installation terminée, vous devriez voir un écran semblable à celui-ci :

![Capture d’écran de la gestion du cycle de vie de SAP HANA, indiquant que l’installation est terminée](./media/hana-installation/image28_install_finished.PNG)

L’instance SAP HANA doit maintenant être opérationnelle et prête à l’emploi. Vous devez être en mesure de vous y connecter à partir de SAP HANA Studio. Veillez également à rechercher et appliquer régulièrement les dernières mises à jour.


## <a name="next-steps"></a>Étapes suivantes

- [Haute disponibilité et récupération d’urgence des grandes instances SAP HANA sur Azure](hana-overview-high-availability-disaster-recovery.md)

