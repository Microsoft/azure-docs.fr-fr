---
title: Déploiement de SAP S/4HANA ou BW/4HANA sur une machine virtuelle Azure | Microsoft Docs
description: Déploiement de SAP S/4HANA ou BW/4HANA sur une machine virtuelle Azure
author: hermanndms
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 09/15/2016
ms.author: hermannd
ms.reviewer: cynthn
ms.openlocfilehash: 6492e770479042a5a1b4da6f61917832adad4873
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021416"
---
# <a name="deploy-sap-s4hana-or-bw4hana-on-azure"></a>Déploiement de SAP S/4HANA ou BW/4HANA sur Azure
Cet article explique comment déployer S/4HANA sur Azure en utilisant SAP Cloud Appliance Library (SAP CAL) 3.0. Procédez de la même façon pour déployer d’autres solutions SAP HANA, comme BW/4HANA.

> [!NOTE]
> Pour plus d’informations sur SAP CAL, accédez au site web [SAP Cloud Appliance Library](https://cal.sap.com/). Il existe également un blog SAP sur [SAP Cloud Appliance Library 3.0](https://scn.sap.com/community/cloud-appliance-library/blog/2016/05/27/sap-cloud-appliance-library-30-came-with-a-new-user-experience).
> 
> [!NOTE]
> Depuis le 29 mai 2017, vous pouvez utiliser le modèle de déploiement Azure Resource Manager en plus du modèle de déploiement Classic (moins recommandé) pour déployer SAP CAL. Nous vous recommandons d’utiliser le nouveau modèle de déploiement Resource Manager et d’ignorer le modèle de déploiement Classic.

## <a name="step-by-step-process-to-deploy-the-solution"></a>Procédure pas à pas de déploiement de la solution

La séquence de captures d’écran suivante montre comment déployer S/4HANA sur Azure à l’aide de SAP CAL. Le processus fonctionne de la même façon pour les autres solutions, comme BW/4HANA.

La page **Solutions** montre certaines solutions SAP CAL HANA disponibles sur Azure. **SAP S/4HANA 1610 FPS01, Fully-Activated Appliance** figure sur la ligne centrale :

![Solutions SAP CAL](./media/cal-s4h/s4h-pic-1c.png)

### <a name="create-an-account-in-the-sap-cal"></a>Création d’un compte dans SAP CAL
1. Lorsque vous vous connectez à SAP CAL pour la première fois, utilisez votre identifiant SAP S-User ou un autre identifiant utilisateur enregistré auprès de SAP. Ensuite, définissez un compte SAP CAL utilisé par SAP CAL pour déployer des appliances sur Azure. Dans la définition du compte, procédez comme suit :

    a. Sélectionnez le modèle de déploiement sur Azure (Resource Manager ou Classic).

    b. Entrez votre abonnement Azure. Un compte SAP CAL ne peut être assigné qu’à un seul abonnement. Si vous avez besoin de plusieurs abonnements, vous devez créer un autre compte SAP CAL.

    c. Accordez à SAP CAL l’autorisation d’effectuer des déploiements dans votre abonnement Azure.

   > [!NOTE]
   >  Les étapes suivantes montrent comment créer un compte SAP CAL pour les déploiements Resource Manager. Si vous disposez déjà d’un compte SAP CAL lié au modèle de déploiement Classic, vous *devez* suivre ces étapes pour créer un nouveau compte SAP CAL. Le nouveau compte SAP CAL doit être déployé dans le modèle Resource Manager.

1. Créez un compte SAP CAL. La page **Accounts** (Comptes) affiche trois possibilités pour Azure : 

    a. **Microsoft Azure (Classic)** est le modèle de déploiement classique et n’est plus recommandé.

    b. **Microsoft Azure** est le nouveau modèle de déploiement Resource Manager.

    c. **Windows Azure operated by 21Vianet** (Windows Azure géré par 21Vianet) est une option pour la Chine, qui utilise le modèle de déploiement Classic.

    Pour effectuer un déploiement dans le modèle Resource Manager, sélectionnez **Microsoft Azure**.

    ![Détails du compte SAP CAL](./media/cal-s4h/s4h-pic-2a.png)

1. Dans le champ **Subscription ID** (ID d’abonnement), entrez l’ID d’abonnement Azure indiqué dans le Portail Azure.

   ![Comptes SAP CAL](./media/cal-s4h/s4h-pic3c.png)

1. Pour autoriser le déploiement de SAP CAL dans l’abonnement Azure que vous avez défini, cliquez sur **Authorize** (Autoriser). La page suivante s’affiche dans l’onglet du navigateur :

   ![Connexion aux services cloud d’Internet Explorer](./media/cal-s4h/s4h-pic4c.png)

1. Si plusieurs utilisateurs sont répertoriés, choisissez le compte Microsoft lié en tant que coadministrateur de l’abonnement Azure que vous avez sélectionné. La page suivante s’affiche dans l’onglet du navigateur :

   ![Confirmation des services cloud d’Internet Explorer](./media/cal-s4h/s4h-pic5a.png)

1. Cliquez sur **Accepter**. Si l’autorisation est accordée, la définition du compte SAP CAL s’affiche à nouveau. Après une courte période, un message confirme que le processus d’autorisation a réussi.

1. Pour assigner le compte SAP CAL nouvellement créé à votre utilisateur, entrez votre identifiant utilisateur dans la zone de texte **User ID** (Identifiant utilisateur) à droite, puis cliquez sur **Add** (Ajouter).

   ![Association du compte à l’utilisateur](./media/cal-s4h/s4h-pic8a.png)

1. Pour associer votre compte à l’utilisateur que vous utilisez pour vous connecter à SAP CAL, cliquez sur **Review** (Revoir). 
 
1. Pour créer l’association entre votre utilisateur et le compte SAP CAL nouvellement créé, cliquez sur **Create** (Créer).

   ![Association d’un utilisateur au compte SAP CAL](./media/cal-s4h/s4h-pic9b.png)

Vous avez correctement créé un compte SAP CAL capable d’effectuer les tâches suivantes :

- Utiliser le modèle de déploiement Resource Manager.
- Déployer des systèmes SAP dans votre abonnement Azure.

Vous pouvez à présent commencer à déployer S/4HANA dans votre abonnement utilisateur dans Azure.

> [!NOTE]
> Avant de poursuivre, déterminez si vous êtes lié à des quotas de processeurs virtuels pour les machines virtuelles Azure série H. Pour le moment, SAP CAL utilise des machines virtuelles Azure série H pour déployer certaines solutions SAP HANA. Il se peut que votre abonnement Azure ne soit pas associé à des quotas de processeurs virtuels pour la série H. Dans ce cas, vous devrez peut-être contacter le support Azure pour obtenir un quota d’au moins 16 processeurs virtuels de série H.
> 
> [!NOTE]
> Lorsque vous déployez une solution sur Azure dans SAP CAL, vous constaterez peut-être que vous ne pouvez choisir qu’une seule région Azure. Pour effectuer un déploiement dans des régions Azure autres que celle suggérée par SAP CAL, vous devez acheter un abonnement CAL auprès de SAP. Vous devrez peut-être également ouvrir un message avec SAP pour que votre compte CAL puisse déployer dans des régions Azure autres que celles suggérées initialement.

### <a name="deploy-a-solution"></a>Déployer une solution

Nous allons à présent déployer une solution à partir de la page **Solutions** de SAP CAL. SAP CAL offre deux séquences de déploiement :

- Une séquence de base qui utilise une page pour définir le système à déployer
- Une séquence avancée qui vous offre différents choix en termes de tailles de machine virtuelle 

Nous présentons ici la séquence de déploiement de base.

1. Sur la page **Account Details** (Détails du compte), effectuez les tâches suivantes :

    a. Sélectionnez un compte SAP CAL. (Utilisez un compte associé au déploiement avec le modèle de déploiement Resource Manager.)

    b. Entrez le nom de l’instance dans le champ **Name** (Nom).

    c. Sélectionnez une région Azure dans le champ **Region** (Région). SAP CAL suggère une région. Si vous avez besoin d’une autre région Azure et que vous ne possédez pas d’abonnement SAP CAL, vous devez vous procurer un abonnement CAL auprès de SAP.

    d. Entrez un mot de passe principal pour la solution dans le champ **Password** (Mot de passe). Celui-ci doit comprendre huit ou neuf caractères. Le mot de passe est utilisé pour les administrateurs des différents composants.

   ![Mode de base SAP CAL : créer une instance](./media/cal-s4h/s4h-pic10a.png)

1. Cliquez sur **Create** (Créer), puis, dans la boîte de message qui s’affiche, cliquez sur **OK**.

   ![Tailles de machine virtuelle prises en charge par SAP CAL](./media/cal-s4h/s4h-pic10b.png)

1. Dans la boîte de dialogue **Private Key** (Clé privée), cliquez sur **Store** (Stocker) pour stocker la clé privée dans SAP CAL. Pour protéger la clé privée par mot de passe, cliquez sur **Download** (Télécharger). 

   ![Clé privée SAP CAL](./media/cal-s4h/s4h-pic10c.png)

1. Lisez le **message d’avertissement** SAP CAL, puis cliquez sur **OK**.

   ![Avertissement SAP CAL](./media/cal-s4h/s4h-pic10d.png)

    Le déploiement s’effectue. Après un délai variable en fonction de la taille et de la complexité de la solution (SAP CAL en donne une estimation), la solution apparaît comme active et prête à être utilisée.

1. Pour trouver les machines virtuelles collectées avec les autres ressources associées dans un groupe de ressources, accédez au Portail Azure : 

   ![Objets SAP CAL déployés dans le nouveau portail](./media/cal-s4h/sapcaldeplyment_portalview.png)

1. Sur le portail SAP CAL, l’état affiché est **Active**. Pour vous connecter à la solution, cliquez sur **Connect** (Connexion). Plusieurs options de connexion aux différents composants sont déployées au sein de cette solution.

   ![Instances SAP CAL](./media/cal-s4h/active_solution.png)

1. Pour pouvoir utiliser l’une des options de connexion aux systèmes déployés, cliquez sur **Getting Started Guide** (Guide de prise en main). 

   ![Connexion à l’instance](./media/cal-s4h/connect_to_solution.png)

    La documentation désigne les utilisateurs pour chacune des méthodes de connectivité. Le mot de passe principal défini au début du processus de déploiement définit les mots de passe de ces utilisateurs. La documentation répertorie d’autres utilisateurs plus fonctionnels avec leurs mots de passe. Vous pouvez les utiliser pour vous connecter au système déployé. 

    Par exemple, si vous utilisez l’interface graphique utilisateur SAP pré-installée sur l’ordinateur Bureau à distance Windows, le système S/4 peut ressembler à ceci :

   ![SM50 dans l’interface graphique utilisateur SAP pré-installée](./media/cal-s4h/gui_sm50.png)

    Si vous utilisez DBACockpit, l’instance peut ressembler à ceci :

   ![SM50 dans l’interface graphique utilisateur SAP DBACockpit](./media/cal-s4h/dbacockpit.png)

En quelques heures, une appliance SAP S/4 intègre est déployée dans Azure.

Si vous avez acheté un abonnement SAP CAL, SAP prend entièrement en charge les déploiements par le biais de SAP CAL sur Azure. La file d’attente de prise en charge est BC-VCM-CAL.







