---
title: Configurer une identité commune
titleSuffix: Azure Data Science Virtual Machine
description: Découvrez comment créer des comptes d’utilisateurs courants qui peuvent être utilisés sur plusieurs machines Data Science VM. Vous pouvez utiliser Azure Active Directory ou un annuaire local Active Directory pour authentifier les utilisateurs à Data Science Virtual Machine.
keywords: formation approfondie, IA, outils de science des données, machine virtuelle de science des données, analyse géospatiale, processus de science des données en équipe
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: vijetajo
ms.author: vijetaj
ms.topic: conceptual
ms.date: 05/08/2018
ms.openlocfilehash: 3dc6fb64f6e8695d84e292322293998e2f4cb0a3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009226"
---
# <a name="set-up-a-common-identity-on-a-data-science-virtual-machine"></a>Configurer une identité commune sur une Data Science Virtual Machine

Sur une machine virtuelle Microsoft Azure, notamment une Data Science Virtual Machine (DSVM), vous allez créer des comptes d’utilisateur local tout en provisionnant la machine virtuelle. Ensuite, les utilisateurs s’authentifient sur la VM à l’aide de ces informations d’identification. Si vous avez plusieurs machines virtuelles auxquelles vos utilisateurs doivent accéder, la gestion des informations d’identification peut s’avérer très lourde. Une excellente solution consiste à déployer des comptes d’utilisateur communs et à les gérer par le biais d’un fournisseur d’identité basée sur des normes. Cette approche vous permet d’utiliser un ensemble unique d’informations d’identification pour accéder à plusieurs ressources sur Azure, notamment à plusieurs DSVM.

Active Directory est un fournisseur d’identité éprouvé et pris en charge sur Azure à la fois en tant que service cloud et en tant qu’annuaire local. Vous pouvez utiliser Azure Active Directory (Azure AD) ou la version d’Active Directory locale pour authentifier les utilisateurs sur un DSVM autonome ou un cluster de DSVM dans un groupe de machines virtuelles identiques Azure. Pour cela, joignez les instances DSVM à un domaine Active Directory.

Si vous disposez déjà d’un annuaire Active Directory, vous pouvez l’utiliser en tant que fournisseur d’identité commune. Si vous n’en n’avez pas, vous pouvez exécuter une instance Active Directory managée sur Azure par le biais d’[Azure Active Directory Domain Services](../../active-directory-domain-services/index.yml)(Azure AD DS).

La documentation sur [Azure AD](../../active-directory/index.yml) fournit des [instructions de gestion](../../active-directory/hybrid/whatis-hybrid-identity.md) détaillées, notamment des procédures pour connecter Azure AD à votre annuaire local si vous en avez un.

Cet article explique comment configurer un service de domaine Active Directory entièrement managé sur Azure à l’aide d’Azure AD DS. Vous pouvez ensuite joindre vos DSVM au domaine Active Directory managé. Cette approche permet aux utilisateurs d’accéder à un pool de DSVM (et à d’autres ressources Azure) par le biais d’un compte d’utilisateur commun et d’informations d’identification communes.

## <a name="set-up-a-fully-managed-active-directory-domain-on-azure"></a>Configurer un domaine Active Directory entièrement managé dans Azure

Azure AD DS facilite la gestion des identités en fournissant un service entièrement géré dans Azure. Pour ce domaine Active Directory, vous gérez les utilisateurs et les groupes. Pour configurer un domaine Active Directory hébergé sur Azure et des comptes d’utilisateur dans votre annuaire, effectuez ces étapes :

1. Dans le portail Azure, ajoutez l’utilisateur à Active Directory : 

   1. Connectez-vous au [centre d’administration Azure Active Directory](https://aad.portal.azure.com) en utilisant un compte d’administrateur général pour l’annuaire.
    
   1. Sélectionnez **Azure Active Directory**, puis **Utilisateurs et groupes**.
    
   1. Dans **Utilisateurs et groupes**, sélectionnez **Tous les utilisateurs**, puis **Nouvel utilisateur**.
   
        Le volet **Utilisateur** s’ouvre :
      
        ![Volet « Utilisateur »](./media/add-user.png)
    
   1. Entrez les détails de l’utilisateur, dont son **nom** et son **nom d’utilisateur**. La partie du nom de domaine du nom d’utilisateur doit être le nom de domaine initial par défaut, « [nom de domaine].onmicrosoft.com », ou un [nom de domaine personnalisé](../../active-directory/fundamentals/add-custom-domain.md) vérifié, non fédéré, comme « contoso.com ».
    
   1. Copiez ou notez d’une autre façon le mot de passe généré de sorte à pouvoir le fournir à l’utilisateur une fois ce processus terminé.
    
   1. Si vous le souhaitez, vous pouvez ouvrir et renseigner les informations dans **Profil**, **Groupes** ou **Rôle d’annuaire** pour l’utilisateur. 
    
   1. Sous **Utilisateur**, sélectionnez **Créer**.
    
   1. Distribuez de manière sécurisée le mot de passe généré au nouvel utilisateur afin qu’il puisse se connecter.

1. Créez une instance Azure AD DS. Suivez les instructions données dans [Activer Azure Active Directory Domain Services à l’aide du portail Azure](../../active-directory-domain-services/tutorial-create-instance.md) (section « Créer une instance et configurer les paramètres de base »). Il est important de mettre à jour les mots de passe existants de l’utilisateur dans Active Directory afin que le mot de passe dans Azure AD DS soit synchronisé. Il est également important d’ajouter DNS à Azure AD DS, comme décrit sous « Renseigner les champs de la fenêtre Général du portail Azure pour créer une instance Azure AD DS » dans cette section.

1. Créez un sous-réseau DSVM distinct dans le réseau virtuel créé dans la section « Créer et configurer le réseau virtuel » de l’étape précédente.
1. Créez une ou plusieurs instances de DSVM dans le sous-réseau DSVM.
1. Suivez les [instructions](../../active-directory-domain-services/join-ubuntu-linux-vm.md) pour ajouter la DSVM à Active Directory. 
1. Montez un partage Azure Files pour héberger votre répertoire de base ou de notebooks afin que votre espace de travail puisse être monté sur n’importe quelle machine. (Si vous avez besoin d’autorisations limitées au niveau des fichiers, vous avez besoin d’un système NFS (Network File System) s’exécutant sur une ou plusieurs machines virtuelles.)

   1. [Créez un partage Azure Files](../../storage/files/storage-how-to-create-file-share.md).
    
   2.  Montez ce partage sur la DSVM Linux. Quand vous sélectionnez **Connexion** dans le partage Azure Files de votre compte de stockage dans le portail Azure, la commande à exécuter dans un interpréteur de commandes bash s’affiche sur la DSVM Linux. La commande se présente comme suit :
   
   ```
   sudo mount -t cifs //[STORAGEACCT].file.core.windows.net/workspace [Your mount point] -o vers=3.0,username=[STORAGEACCT],password=[Access Key or SAS],dir_mode=0777,file_mode=0777,sec=ntlmssp
   ```
1. Par exemple, supposons que vous avez monté votre partage Azure Files dans /data/workspace. À présent, créez des répertoires pour chacun de vos utilisateurs dans le partage : /data/workspace/user1, /data/workspace/user2, et ainsi de suite. Créez un répertoire `notebooks` dans l’espace de travail de chaque utilisateur. 
1. Créez des liens symboliques pour `notebooks` dans `$HOME/userx/notebooks/remote`.   

Les utilisateurs de votre instance Active Directory sont maintenant hébergés dans Azure. En utilisant les informations d’identification Active Directory, les utilisateurs peuvent se connecter à n’importe quelle DSVM (SSH ou JupyterHub) jointe à Azure AD DS. Étant donné que l’espace de travail utilisateur se trouve dans le partage Azure Files, les utilisateurs ont accès à leurs blocs-notes et aux autres travaux à partir de n’importe quelle DSVM lorsqu’ils utilisent JupyterHub.

Pour la mise à l’échelle automatique, vous pouvez utiliser le groupe de machines virtuelles identiques pour créer un pool de VM qui sont toutes jointes au domaine de cette manière et avec le disque partagé monté. Les utilisateurs peuvent se connecter à n’importe quelle machine disponible dans le groupe de machines virtuelles identiques et avoir accès au disque partagé sur lequel leurs notebooks sont enregistrés. 

## <a name="next-steps"></a>Étapes suivantes

* [Stocker en toute sécurité les informations d’identification pour accéder aux ressources cloud](dsvm-secure-access-keys.md)