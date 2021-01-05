---
title: Élever les privilèges d’un cloud privé
titleSuffix: Azure VMware Solution by CloudSimple
description: Décrit comment élever les privilèges sur votre cloud privé pour les fonctions administratives dans vCenter
author: sharaths-cs
ms.author: b-shsury
ms.date: 06/05/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 628824a3d303184e199452b0974c7f99d99a4933
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86998950"
---
# <a name="escalate-private-cloud-vcenter-privileges-from-the-cloudsimple-portal"></a>Élever les privilèges de vCenter pour votre cloud privé à partir du portail CloudSimple

Pour un accès administratif au vCenter de votre cloud privé, vous pouvez temporairement élever vos privilèges CloudSimple.  Avec des privilèges élevés, vous pouvez installer des solutions VMware, ajouter des sources d’identité et gérer les utilisateurs.

De nouveaux utilisateurs peuvent être créés sur le domaine d’authentification unique de vCenter et recevoir un accès à vCenter.  Lorsque vous créez de nouveaux utilisateurs, ajoutez-les aux groupes intégrés à CloudSimple pour accéder à vCenter.  Pour plus d’informations, consultez [Modèle d’autorisation de cloud privé CloudSimple de VMware vCenter](./learn-private-cloud-permissions.md).

> [!CAUTION]
> Ne modifiez pas la configuration pour les composants de gestion. Les actions effectuées au cours de l’état privilégié élevé peuvent nuire à votre système ou rendre votre système indisponible.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="escalate-privileges"></a>Élever les privilèges

1. Accédez au [portail CloudSimple](access-cloudsimple-portal.md).

2. Ouvrez la page **Ressources** et sélectionnez le cloud privé dont vous souhaitez élever les privilèges.

3. Au bas de la page de résumé, sous **Modifier les privilèges vSphere**, cliquez sur **Élever**.

    ![Modifier le privilège vSphere](media/escalate-private-cloud-privilege.png)

4. Sélectionnez le type d’utilisateur vSphere.  Seul l’utilisateur local `CloudOwner@cloudsimple.local` peut être élevé.

5. Sélectionnez l’intervalle de temps d’élévation dans la liste déroulante. Choisissez la période la plus courte qui vous permettra de terminer la tâche.

6. Cochez la case pour confirmer que vous comprenez les risques.

    ![Boîte de dialogue d’élévation des privilèges](media/escalate-private-cloud-privilege-dialog.png)

7. Cliquez sur **OK**.

8. Le processus d’élévation des privilèges peut prendre quelques minutes. Lorsque vous avez terminé, cliquez sur **OK**.

L’élévation de privilèges commence et dure jusqu'à la fin de l’intervalle sélectionné.  Connectez-vous au vCenter de votre cloud privé pour effectuer des tâches d’administration.

> [!IMPORTANT]
> Un seul utilisateur peut avoir des privilèges élevés.  Vous devez retirer les privilèges élevés de l’utilisateur avant de pouvoir élever les privilèges d’un autre utilisateur.

> [!CAUTION]
> Les nouveaux utilisateurs doivent être ajoutés uniquement à *Cloud-Owner-Group*, à *Cloud-Global-Cluster-Admin-Group*, à *Cloud-Global-Storage-Admin-Group*, à *Cloud-Global-Network-Admin-Group* ou à *Cloud-Global-VM-Admin-Group*.  Les utilisateurs ajoutés au groupe *Administrateurs* seront automatiquement supprimés.  Seuls des comptes de service doivent être ajoutés au groupe *Administrateurs*, et les comptes de service en doivent pas être utilisés pour se connecter à l’interface utilisateur web de vSphere.

## <a name="extend-privilege-escalation"></a>Étendre l’élévation de privilèges

Si vous avez besoin de plus de temps pour effectuer vos tâches, vous pouvez étendre la période d’élévation des privilèges.  Choisissez l’intervalle de temps supplémentaire qui vous permettra d’effectuer les tâches d’administration.

1. Dans **Ressources** > **Clouds privés** sur le portail CloudSimple, sélectionnez le cloud privé pour lequel vous souhaitez étendre l’élévation de privilèges.

2. Au bas de l’onglet Résumé, cliquez sur **Étendre l’élévation de privilèges**.

    ![Étendre l’élévation de privilèges](media/de-escalate-private-cloud-privilege.png)

3. Sélectionnez un intervalle de temps d’élévation dans la liste déroulante. Passez en revue la nouvelle heure de fin de l’élévation.

4. Cliquez sur **Enregistrer** pour étendre l’intervalle.

## <a name="de-escalate-privileges"></a>Annuler l’élévation des privilèges

Une fois que vos tâches administratives sont terminées, vous devez annuler l’élévation de vos privilèges.  

1. Dans **Ressources** > **Clouds privés** sur le portail CloudSimple, sélectionnez le cloud privé pour lequel vous souhaitez annuler l’élévation de privilèges.

2. Cliquez sur **Annuler l’élévation**.

3. Cliquez sur **OK**.

> [!IMPORTANT]
> Pour éviter les erreurs, déconnectez-vous de vCenter et connectez-vous à nouveau après l’annulation de l’élévation des privilèges.

## <a name="next-steps"></a>Étapes suivantes

* [Configurer les sources d’identité vCenter pour utiliser Active Directory](./set-vcenter-identity.md)
* Installer la solution de sauvegarde pour [sauvegarder des machines virtuelles de charge de travail](./backup-workloads-veeam.md)
