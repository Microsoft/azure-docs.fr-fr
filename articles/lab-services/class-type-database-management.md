---
title: Configurer un laboratoire pour enseigner la gestion des bases de données pour les bases de données relationnelles | Microsoft Docs
description: Découvrez comment configurer un laboratoire pour enseigner la gestion des bases de données relationnelles.
author: emaher
ms.topic: article
ms.date: 06/26/2020
ms.author: enewman
ms.openlocfilehash: 32079d6dc31cf3fcad66976737630472129b4271
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462426"
---
# <a name="set-up-a-lab-to-teach-database-management-for-relational-databases"></a>Configurer un laboratoire pour enseigner la gestion des bases de données pour les bases de données relationnelles

Cet article explique comment configurer un laboratoire pour une classe de gestion de bases de données de base dans Azure Lab Services. Les concepts des bases de données constituent l’un des cours d’introduction abordés dans la plupart des départements informatiques à l’université. Le langage SQL est une norme internationale. SQL est le langage standard pour la gestion des bases de données relationnelles, notamment l’ajout, la récupération et la gestion du contenu dans une base de données.  Il est surtout indiqué pour sa capacité de traitement rapide, sa fiabilité, sa facilité et sa flexibilité d’utilisation.

Dans cet article, nous allons montrer comment configurer un modèle d'ordinateur virtuel dans un laboratoire avec un serveur de base de données MySQL et un serveur SQL Server 2019.  [MySQL](https://www.mysql.com/) est un système de gestion de base de données relationnelle (SGBDR) open source disponible gratuitement.  [SQL Server 2019](https://www.microsoft.com/sql-server/sql-server-2019) est la dernière version du SGBDR de Microsoft.

## <a name="lab-configuration"></a>Configuration du laboratoire

Pour configurer ce labo, vous avez besoin d’un abonnement Azure et d’un compte Lab pour commencer. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer. Une fois que vous disposez d’un abonnement Azure, vous pouvez créer un compte lab dans Azure Lab Services. Pour plus d’informations sur la création d’un compte de laboratoire, consultez notre [Tutoriel pour configurer un compte lab](tutorial-setup-lab-account.md).  Vous pouvez également utiliser un compte Lab existant.

### <a name="lab-account-settings"></a>Paramètres du compte Lab

Activez les paramètres décrits dans le tableau ci-dessous pour le compte lab. Pour plus d’informations sur l’activation des images de la Place de marché, consultez [Spécifier les images de la Place de marché accessibles aux créateurs de labo](./specify-marketplace-images.md).

| Paramètres du compte lab | Instructions |
| ------------------- | ------------ |
|Image de la Place de marché| Activez l’image « SQL Server 2019 Standard sur Windows Server 2019 » pour l’utiliser dans votre compte lab.|

### <a name="lab-settings"></a>Paramètres du labo

Utilisez les paramètres du tableau ci-dessous lors de la configuration d’un laboratoire de classe.  Pour plus d’informations sur la création d’un laboratoire de classe, consultez le [didacticiel Configurer un laboratoire de salle de classe](tutorial-setup-classroom-lab.md).

| Paramètres du labo | Valeur/instructions |
| ------------ | ------------------ |
|Taille de la machine virtuelle| Moyenne. Cette taille est idéale pour les bases de données relationnelles, le caching en mémoire et l’analyse.|
|Image de machine virtuelle| SQL Server 2019 Standard sur Windows Server 2019|

## <a name="template-machine-configuration"></a>Configuration du modèle de machine

Pour installer MySQL sur Windows Server 2019, vous pouvez suivre les étapes indiquées dans [Installer et exécuter le serveur de communauté MySQL sur une machine virtuelle](/previous-versions/azure/virtual-machines/windows/classic/mysql-2008r2?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json#install-and-run-mysql-community-server-on-the-virtual-machine).

SQL Server 2019 est préinstallé dans l’image de machine virtuelle que nous avons choisie lors de la création du laboratoire.

## <a name="cost-estimate"></a>Estimation du coût

Nous allons aborder une estimation de coût possible pour cette classe.  Nous allons utiliser une classe de 25 élèves.  Nous prévoyons 20 heures de temps de classe.  En outre, chaque étudiant obtient un quota de 10 heures pour les devoirs ou travaux en dehors des heures de cours planifiées.  La taille de machine virtuelle que nous avons choisie était moyenne, soit 42 unités Lab.

Voici un exemple d’estimation de coût possible pour cette classe :

25 élèves \* (20 heures planifiées + 10 heures de quota) \* 0,42 USD par heure = 315,00 USD

Pour plus d’informations sur les tarifs, consultez [Tarification Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

## <a name="conclusion"></a>Conclusion

Cet article vous a présenté les étapes nécessaires à la création d’un laboratoire pour les concepts de base de gestion de base de données à l’aide de MySQL et de SQL Server. Vous pouvez utiliser une configuration similaire pour d’autres classes de base de données.

## <a name="next-steps"></a>Étapes suivantes

Les étapes suivantes sont communes à la configuration de n’importe quel labo.

- [Créer et gérer un modèle](how-to-create-manage-template.md)
- [Ajout d'utilisateurs](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Définir un quota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Définir un calendrier](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Envoyer par mail des liens d’inscription aux étudiants](how-to-configure-student-usage.md#send-invitations-to-users)