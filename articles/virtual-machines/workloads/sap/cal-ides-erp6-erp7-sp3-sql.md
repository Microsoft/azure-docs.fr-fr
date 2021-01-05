---
title: Déploiement de SAP IDES EHP7 SP3 pour SAP ERP 6.0 sur Azure | Microsoft Docs
description: Déploiement de SAP IDES EHP7 SP3 pour SAP ERP 6.0 sur Azure
author: hermanndms
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 09/16/2016
ms.author: hermannd
ms.reviewer: cynthn
ms.openlocfilehash: dc0c1744c2943d33d516de4ed9d28893541773ad
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94957875"
---
# <a name="deploy-sap-ides-ehp7-sp3-for-sap-erp-60-on-azure"></a>Déploiement de SAP IDES EHP7 SP3 pour SAP ERP 6.0 sur Azure
Cet article explique comment déployer un système SAP IDES exécuté avec SQL Server et le système d’exploitation Windows sur Azure via SAP Cloud Appliance Library (SAP CAL) 3.0. Les captures d’écran montrent la procédure étape par étape. Pour déployer une autre solution, suivez la même procédure.

Pour démarrer avec SAP CAL, accédez au site web [SAP Cloud Appliance Library](https://cal.sap.com/). Il existe également un blog SAP sur le nouveau [SAP Cloud Appliance Library 3.0](https://scn.sap.com/community/cloud-appliance-library/blog/2016/05/27/sap-cloud-appliance-library-30-came-with-a-new-user-experience). 

> [!NOTE]
> Depuis le 29 mai 2017, vous pouvez utiliser le modèle de déploiement Azure Resource Manager en plus du modèle de déploiement Classic (moins recommandé) pour déployer SAP CAL. Nous vous recommandons d’utiliser le nouveau modèle de déploiement Resource Manager et d’ignorer le modèle de déploiement Classic.

Si vous avez déjà créé un compte SAP CAL qui utilise le modèle classique, *vous devez créer un autre compte SAP CAL*. Ce compte doit être destiné exclusivement au déploiement dans Azure à l’aide du modèle Resource Manager.

Après vous être connecté à SAP CAL, la première page vous conduit généralement à la page **Solutions**. Les solutions proposées sur SAP CAL sont toujours plus nombreuses, si bien que vous devrez peut-être en faire défiler plusieurs avant de trouver celle souhaitée. La solution SAP IDES basée sur Windows mise en surbrillance disponible exclusivement sur Azure montre le processus de déploiement :

![Solutions SAP CAL](./media/cal-ides-erp6-ehp7-sp3-sql/ides-pic1.jpg)

### <a name="create-an-account-in-the-sap-cal"></a>Création d’un compte dans SAP CAL
1. Lorsque vous vous connectez à SAP CAL pour la première fois, utilisez votre identifiant SAP S-User ou un autre identifiant utilisateur enregistré auprès de SAP. Ensuite, définissez un compte SAP CAL utilisé par SAP CAL pour déployer des appliances sur Azure. Dans la définition du compte, procédez comme suit :

    a. Sélectionnez le modèle de déploiement sur Azure (Resource Manager ou Classic).

    b. Entrez votre abonnement Azure. Un compte SAP CAL ne peut être assigné qu’à un seul abonnement. Si vous avez besoin de plusieurs abonnements, vous devez créer un autre compte SAP CAL.
    
    c. Accordez à SAP CAL l’autorisation d’effectuer des déploiements dans votre abonnement Azure.

   > [!NOTE]
   >  Les étapes suivantes montrent comment créer un compte SAP CAL pour les déploiements Resource Manager. Si vous disposez déjà d’un compte SAP CAL lié au modèle de déploiement Classic, vous *devez* suivre ces étapes pour créer un nouveau compte SAP CAL. Le nouveau compte SAP CAL doit être déployé dans le modèle Resource Manager.

1. Pour créer un nouveau compte SAP CAL, vous disposez de deux choix dans la page **Compte** pour Azure : 

    a. **Microsoft Azure (Classic)** est le modèle de déploiement classique et n’est plus recommandé.

    b. **Microsoft Azure** est le nouveau modèle de déploiement Resource Manager.

    ![Capture d’écran montrant les comptes SAP CAL avec Microsoft Azure en évidence.](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic-2a.PNG)

    Pour effectuer un déploiement dans le modèle Resource Manager, sélectionnez **Microsoft Azure**.

    ![Capture d’écran montrant les comptes SAP CAL.](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic3c.PNG)

1. Dans le champ **Subscription ID** (ID d’abonnement), entrez l’ID d’abonnement Azure indiqué dans le Portail Azure. 

    ![ID d’abonnement SAP CAL](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic3c.PNG)

1. Pour autoriser le déploiement de SAP CAL dans l’abonnement Azure que vous avez défini, cliquez sur **Authorize** (Autoriser). La page suivante s’affiche dans l’onglet du navigateur :

    ![Connexion aux services cloud d’Internet Explorer](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic4c.PNG)

1. Si plusieurs utilisateurs sont répertoriés, choisissez le compte Microsoft lié en tant que coadministrateur de l’abonnement Azure que vous avez sélectionné. La page suivante s’affiche dans l’onglet du navigateur :

    ![Confirmation des services cloud d’Internet Explorer](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic5a.PNG)

1. Cliquez sur **Accepter**. Si l’autorisation est accordée, la définition du compte SAP CAL s’affiche à nouveau. Après une courte période, un message confirme que le processus d’autorisation a réussi.

1. Pour assigner le compte SAP CAL nouvellement créé à votre utilisateur, entrez votre identifiant utilisateur dans la zone de texte **User ID** (Identifiant utilisateur) à droite, puis cliquez sur **Add** (Ajouter). 

    ![Association du compte à l’utilisateur](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic8a.PNG)

1. Pour associer votre compte à l’utilisateur que vous utilisez pour vous connecter à SAP CAL, cliquez sur **Review** (Revoir). 

1. Pour créer l’association entre votre utilisateur et le compte SAP CAL nouvellement créé, cliquez sur **Create** (Créer).

    ![Association d’un utilisateur au compte](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic9b.PNG)

Vous avez correctement créé un compte SAP CAL capable d’effectuer les tâches suivantes :

- Utiliser le modèle de déploiement Resource Manager.
- Déployer des systèmes SAP dans votre abonnement Azure.

> [!NOTE]
> Avant de pouvoir déployer la solution SAP IDES basée sur Windows et SQL Server, vous devrez peut-être souscrire à un abonnement SAP CAL. Sinon, la solution risque d’apparaître comme **Verrouillée** dans la page de vue d’ensemble.

### <a name="deploy-a-solution"></a>Déployer une solution
1. Après avoir configuré un compte SAP CAL, sélectionnez la solution **Solution SAP IDES sur Windows et SQL Server**. Cliquez sur **Créer une instance** et vérifiez les conditions d’utilisation. 

1. Sur la page **Basic Mode: Create Instance** (Mode de base : Créer une instance), procédez comme suit :

    a. Entrez le nom de l’instance dans le champ **Name** (Nom).

    b. Sélectionnez une région Azure dans le champ **Region** (Région). Vous aurez peut-être besoin d’un abonnement SAP CAL pour avoir le choix entre plusieurs régions Azure.

    c.  Entrez le **Mot de passe** maître pour la solution, comme indiqué :

    ![Mode de base SAP CAL : créer une instance](./media/cal-ides-erp6-ehp7-sp3-sql/ides-pic10a.png)

1. Cliquez sur **Créer**. Après un délai variable en fonction de la taille et de la complexité de la solution (SAP CAL en donne une estimation), la solution apparaît comme active et prête à être utilisée : 

    ![Instances SAP CAL](./media/cal-ides-erp6-ehp7-sp3-sql/ides-pic12a.png)

1. Pour trouver le groupe de ressources et tous ses objets créés par SAP CAL, accédez au portail Azure. Les machines virtuelles sont repérables car elles commencent par le nom d’instance qui a été spécifié dans SAP CAL.

    ![Objets du groupe de ressources](./media/cal-ides-erp6-ehp7-sp3-sql/ides_resource_group.PNG)

1. Dans le portail SAP CAL, accédez aux instances déployées, puis cliquez sur **Connect** (Connexion). La fenêtre indépendante suivante apparaît : 

    ![Connexion à l’instance](./media/cal-ides-erp6-ehp7-sp3-sql/ides-pic14a.PNG)

1. Pour pouvoir utiliser l’une des options de connexion aux systèmes déployés, cliquez sur **Getting Started Guide** (Guide de prise en main). La documentation désigne les utilisateurs pour chacune des méthodes de connectivité. Le mot de passe principal défini au début du processus de déploiement définit les mots de passe de ces utilisateurs. La documentation répertorie d’autres utilisateurs plus fonctionnels avec leurs mots de passe. Vous pouvez les utiliser pour vous connecter au système déployé.

    ![Documentation de présentation de SAP](./media/cal-ides-erp6-ehp7-sp3-sql/ides-pic15.jpg)

En quelques heures, un système SAP IDES sain est déployé dans Azure.

Si vous avez acheté un abonnement SAP CAL, SAP prend entièrement en charge les déploiements par le biais de SAP CAL sur Azure. La file d’attente de prise en charge est BC-VCM-CAL.

