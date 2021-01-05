---
title: Configurer un laboratoire SolidWorks pour l’ingénierie avec Azure Lab Services | Microsoft Docs
description: Découvrez comment configurer un laboratoire pour les cours d’ingénierie à l’aide de SOLIDWORKS.
author: nicolela
ms.topic: article
ms.date: 06/26/2020
ms.author: nicolela
ms.openlocfilehash: dc6fdadbdfdbdd1d32f640e356a67841187a83c9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999261"
---
# <a name="set-up-a-lab-for-engineering-classes-using-solidworks"></a>Configurer un laboratoire pour les cours d’ingénierie à l’aide de SOLIDWORKS

[SOLIDWORKS](https://www.solidworks.com/) fournit un environnement de conception assistée par ordinateur (CAO) 3D pour la modélisation d’objets solides et est utilisé dans différents domaines d’ingénierie.  Avec SOLIDWORKS, les ingénieurs peuvent facilement créer, visualiser, simuler et documenter leurs conceptions.

La licence réseau de SOLIDWORKS est couramment utilisée par les universités.   Avec cette option, les utilisateurs partagent un pool de licences qui sont gérées par un serveur de licences.  Ce type de licence est parfois appelé « licence flottante », car vous n’avez besoin que du nombre de licences correspondant au nombre d’utilisateurs simultanés.  Quand un utilisateur a fini d’utiliser SOLIDWORKS, sa licence revient dans le pool de licences géré de manière centralisée afin d’être réutilisée par un autre utilisateur.

Dans cet article, nous allons vous montrer comment configurer une classe qui utilise SOLIDWORKS 2019 et la licence réseau.

## <a name="license-server"></a>Serveur de licences

Le gestionnaire de licences réseau SOLIDWORKS nécessite l’installation et l’activation du gestionnaire de licences SolidNetWork sur votre serveur de licences.  Ce serveur de licences se trouve généralement sur votre réseau local ou sur un réseau privé au sein d’Azure.  Pour plus d’informations sur la configuration du gestionnaire de licences SolidNetWork sur votre serveur, consultez [Installer et activer un gestionnaire de licences](https://help.solidworks.com/2019/English/Installation/install_guide/t_installing_snl_lic_mgr.htm) dans le Guide d’installation de SOLIDWORKS.  Quand vous configurez cette option, souvenez-vous du **numéro de port** et du [**numéro de série**](https://help.solidworks.com/2019/english/installation/install_guide/r_hid_state_serial_number.htm) utilisés, car ils seront nécessaires pour les étapes ultérieures.

Une fois que votre serveur de licences est configuré, vous devez procéder au peering du [réseau virtuel (VNet)](./how-to-connect-peer-virtual-network.md) à votre [compte de labo](./tutorial-setup-lab-account.md).  Le peering de réseau doit être effectué avant de créer le laboratoire afin que les machines virtuelles du laboratoire puissent accéder au serveur de licences et inversement.

> [!NOTE]
> Vous devez vérifier que les ports appropriés sont ouverts sur vos pare-feu pour permettre la communication entre les machines virtuelles du labo et le serveur de licences.  Par exemple, consultez les instructions de la section [Modifier les ports d’ordinateur du gestionnaire de licences pour le pare-feu Windows](http://help.solidworks.com/2019/english/installation/install_guide/t_mod_ports_on_lic_mgr_for_firewall.htm), qui indiquent comment ajouter des règles de trafic entrant et sortant au pare-feu du serveur de licences.  Vous devrez peut-être également ouvrir des ports sur les machines virtuelles du labo.  Suivez les étapes de l’article sur les [paramètres du pare-feu pour les laboratoires](./how-to-configure-firewall-settings.md) pour plus d’informations, notamment sur l’obtention de l’adresse IP publique du laboratoire.

## <a name="lab-configuration"></a>Configuration du laboratoire

Pour configurer ce labo, vous avez besoin d’un abonnement Azure et d’un compte Lab pour commencer. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer. Une fois que vous disposez d’un abonnement Azure, vous pouvez créer un compte lab dans Azure Lab Services. Pour plus d’informations sur la création d’un compte de laboratoire, consultez notre tutoriel sur [la configuration d’un compte de labo](./tutorial-setup-lab-account.md). Vous pouvez également utiliser un compte Lab existant.

### <a name="lab-account-settings"></a>Paramètres du compte Lab

Activez les paramètres décrits dans le tableau ci-dessous pour le compte lab. Pour plus d’informations sur l’activation des images de la Place de marché, consultez l’article [Spécifier les images de la Place de marché accessibles aux créateurs de labo](./specify-marketplace-images.md).

| Paramètres du compte lab | Instructions |
| ------------------- | ------------ |
|Image de la Place de marché| Activez l’image Windows 10 Pro pour l’utiliser dans votre compte lab.|

> [!NOTE]
> En plus de Windows 10, SOLIDWORKS prend aussi en charge d’autres versions de Windows.  Consultez [Configuration système requise pour SOLIDWORKS](https://www.solidworks.com/sw/support/SystemRequirements.html) pour plus d’informations.

### <a name="lab-settings"></a>Paramètres du labo

Utilisez les paramètres du tableau ci-dessous lors de la configuration d’un laboratoire de classe. Pour plus d’informations sur la création d’un laboratoire de classe, consultez le tutoriel Configurer un laboratoire de salle de classe.

| Paramètres du labo | Valeur/instructions |
| ------------ | ------------------ |
|Taille de la machine virtuelle| **GPU de petite taille (visualisation)** .  Cette machine virtuelle est optimisée pour la visualisation à distance, la diffusion en continu, les jeux et l’encodage avec des infrastructures comme OpenGL ou DirectX.|  
|Image de machine virtuelle| Windows 10 Pro|

> [!NOTE]
> La machine virtuelle de **GPU de petite taille (visualisation)** est configurée pour permettre une expérience graphique très performante.  Pour plus d’informations sur cette taille de machine virtuelle, consultez l’article sur [la configuration d’un laboratoire avec des GPU](./how-to-setup-lab-gpu.md).

> [!WARNING]
> N’oubliez pas [d’interconnecter le réseau virtuel](https://www.mathworks.com/support/requirements/matlab-system-requirements.html) du compte de laboratoire au réseau virtuel du serveur de licences **avant** de créer le laboratoire.

## <a name="template-virtual-machine-configuration"></a>Configuration de la machine virtuelle de modèle

Les étapes de cette section montrent comment configurer votre machine virtuelle de modèle en téléchargeant les fichiers d’installation de SOLIDWORKS et en installant le logiciel client :

1. Démarrez la machine virtuelle de modèle et connectez-vous à l’ordinateur à l’aide du protocole RDP.

1. Téléchargez les fichiers d’installation du logiciel client SOLIDWORKS. Vous disposez de deux options pour le téléchargement :
   - Téléchargez à partir du [portail client SOLIDWORKS](https://login.solidworks.com/nidp/idff/sso?id=cpenglish&sid=1&option=credential&sid=1&target=https%3A%2F%2Fcustomerportal.solidworks.com%2F).
   - Télécharger à partir d’un répertoire sur un serveur.  Si vous avez utilisé cette option, vous devez vous assurer que le serveur est accessible à partir de la machine virtuelle de modèle.  Par exemple, ce serveur peut se trouver dans le même réseau virtuel que celui qui est en peering avec votre compte de labo.
  
    Pour plus d’informations, consultez [Installation sur des ordinateurs individuels dans le SOLIDWORKS](http://help.solidworks.com/2019/english/Installation/install_guide/c_installing_on_individual_computers.htm?id=fc149e8a968a422a89e2a943265758d3#Pg0) dans le guide d’installation de SolidWorks.

1. Une fois les fichiers d’installation téléchargés, installez le logiciel client à l’aide du gestionnaire d’installation de SOLIDWORKS. Pour plus d’informations, consultez [Insaller une licence client](http://help.solidworks.com/2019/english/installation/install_guide/t_installing_snl_license_client.htm) dans le guide d’installation SOLIDWORKS.

    > [!NOTE]
    > Dans la boîte de dialogue **Ajouter un serveur**, vous êtes invité à indiquer le **numéro de port** utilisé pour votre serveur de licences, ainsi que le nom ou l’adresse IP du serveur de licences.

## <a name="cost"></a>Coût

Nous allons aborder une estimation de coût possible pour cette classe. Cette estimation n’inclut pas le coût d’utilisation du serveur de licences. Nous allons utiliser une classe de 25 élèves. Nous prévoyons 20 heures de temps de classe. En outre, chaque étudiant obtient un quota de 10 heures pour les devoirs ou travaux en dehors des heures de cours planifiées. La taille de machine virtuelle que nous avons choisie était avec un **GPU de petite taille (visualisation)** , soit 160 unités Lab.

25 étudiants *\* (20 heures planifiées + 10 heures de quota) *\* 160 unités Lab * 0,01 USD par heure = 1200,00 USD

>[!IMPORTANT]
> L’estimation du coût est fournie à titre d’exemple uniquement.  Pour en savoir plus sur les tarifs actuels, consultez [Tarification Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).  

## <a name="next-steps"></a>Étapes suivantes

Les étapes suivantes sont communes à la configuration de n’importe quel labo.

- [Créer et gérer un modèle](how-to-create-manage-template.md)
- [Ajout d'utilisateurs](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Définir un quota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Définir un calendrier](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Envoyer par mail des liens d’inscription aux étudiants](how-to-configure-student-usage.md#send-invitations-to-users)