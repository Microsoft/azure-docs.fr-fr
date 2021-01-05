---
title: Configurer la connectivité des machines virtuelles Azure
titleSuffix: Azure SQL Managed Instance
description: Connectez-vous à Azure SQL Managed Instance à l'aide de SQL Server Management Studio à partir d'une machine virtuelle Azure.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein, srbozovi, bonova
ms.date: 02/18/2019
ms.openlocfilehash: a1f496d59fa626dc8750493591128f7363afa40d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91620254"
---
# <a name="quickstart-configure-an-azure-vm-to-connect-to-azure-sql-managed-instance"></a>Démarrage rapide : Configurer une machine virtuelle Azure pour qu'elle se connecte à Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Ce guide de démarrage rapide explique comment configurer une machine virtuelle Azure pour qu'elle se connecte à Azure SQL Managed Instance à l'aide de SQL Server Management Studio (SSMS). 


Pour accéder à un guide de démarrage rapide expliquant comment se connecter à partir d'un ordinateur client local via une connexion point à site, consultez [Configurer une connexion point à site](point-to-site-p2s-configure.md).

## <a name="prerequisites"></a>Prérequis

Ce guide de démarrage rapide utilise les ressources créées dans [Créer une instance gérée](instance-create-quickstart.md) comme point de départ.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

## <a name="create-a-new-subnet-vnet"></a>Créer un sous-réseau dans le réseau virtuel

Les étapes suivantes créent un sous-réseau dans le réseau virtuel SQL Managed Instance pour permettre à une machine virtuelle Azure de se connecter à l'instance gérée. Le sous-réseau SQL Managed Instance est dédié aux instances gérées. Vous ne pouvez pas créer d’autres ressources que sont notamment les machines virtuelles Azure dans ce sous-réseau.

1. Ouvrez le groupe de ressources de l'instance gérée que vous avez créée dans le guide de démarrage rapide[Créer une instance gérée](instance-create-quickstart.md). Sélectionnez le réseau virtuel de votre instance gérée.

   ![Ressources SQL Managed Instance](./media/connect-vm-instance-configure/resources.png)

2. Sélectionnez **Sous-réseaux** et **+ Sous-réseau** pour créer un sous-réseau.

   ![Sous-réseaux SQL Managed Instance](./media/connect-vm-instance-configure/subnets.png)

3. Remplissez le formulaire en utilisant les informations contenues dans ce tableau :

   | Paramètre| Valeur suggérée | Description |
   | ---------------- | ----------------- | ----------- |
   | **Nom** | Nom valide|Pour connaître les noms valides, consultez [Conventions d’affectation de noms](/azure/architecture/best-practices/resource-naming).|
   | **Plage d’adresses (bloc CIDR)** | Plage valide | La valeur par défaut convient pour ce guide de démarrage rapide.|
   | **Groupe de sécurité réseau** | None | La valeur par défaut convient pour ce guide de démarrage rapide.|
   | **Table de routage** | None | La valeur par défaut convient pour ce guide de démarrage rapide.|
   | **Points de terminaison de service** | 0 sélectionné | La valeur par défaut convient pour ce guide de démarrage rapide.|
   | **Délégation de sous-réseau** | None | La valeur par défaut convient pour ce guide de démarrage rapide.|

   ![Nouveau sous-réseau SQL Managed Instance de la machine virtuelle cliente](./media/connect-vm-instance-configure/new-subnet.png)

4. Sélectionnez **OK** pour créer ce sous-réseau supplémentaire dans le réseau virtuel SQL Managed Instance.

## <a name="create-a-vm-in-the-new-subnet"></a>Créer une machine virtuelle dans le nouveau sous-réseau 

Les étapes suivantes expliquent comment créer une machine virtuelle dans le nouveau réseau virtuel pour vous connecter à SQL Managed Instance.

## <a name="prepare-the-azure-virtual-machine"></a>Préparer la machine virtuelle Azure

Dans la mesure où SQL Managed Instance est placé dans votre réseau virtuel privé, vous devez créer une machine virtuelle Azure et y installer un outil client SQL comme SQL Server Management Studio ou Azure Data Studio. Cet outil vous permet de vous connecter à SQL Managed Instance et d'exécuter des requêtes. Ce guide de démarrage rapide utilise SQL Server Management Studio.

Pour créer une machine virtuelle cliente avec tous les outils nécessaires, le plus simple consiste à utiliser les modèles Azure Resource Manager.

1. Vérifiez que vous êtes connecté au portail Azure dans un autre onglet de navigateur. Cliquez ensuite sur le bouton suivant pour créer une machine virtuelle cliente et installer SQL Server Management Studio :

   [![Image showing a button labeled "Deploy to Azure".](https://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjovanpop-msft%2Fazure-quickstart-templates%2Fsql-win-vm-w-tools%2F201-vm-win-vnet-sql-tools%2Fazuredeploy.json)

2. Remplissez le formulaire en utilisant les informations contenues dans le tableau suivant :

   | Paramètre| Valeur suggérée | Description |
   | ---------------- | ----------------- | ----------- |
   | **Abonnement** | Un abonnement valide | Doit être un abonnement dans lequel vous êtes autorisé à créer des ressources. |
   | **Groupe de ressources** |Groupe de ressources que vous avez spécifié dans le guide de démarrage rapide [Créer SQL Managed Instance](instance-create-quickstart.md).|Ce groupe de ressources doit être celui dans lequel se trouve le réseau virtuel.|
   | **Lieu** | Emplacement du groupe de ressources | Cette valeur est renseignée en fonction du groupe de ressources sélectionné. |
   | **Nom de la machine virtuelle**  | Nom valide | Pour connaître les noms valides, consultez [Conventions d’affectation de noms](/azure/architecture/best-practices/resource-naming).|
   |**Nom de l’utilisateur administrateur**|Tout nom d’utilisateur valide|Pour connaître les noms valides, consultez [Conventions d’affectation de noms](/azure/architecture/best-practices/resource-naming). N’utilisez pas « serveradmin », car il s’agit d’un rôle réservé au niveau du serveur.<br>Vous utilisez ce nom d’utilisateur chaque fois que vous vous [connectez à la machine virtuelle](#connect-to-the-virtual-machine).|
   |**Mot de passe**|Mot de passe valide|Le mot de passe doit contenir au moins 12 caractères et satisfaire aux [exigences de complexité définies](../../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).<br>Vous utilisez ce mot de passe chaque fois que vous vous [connectez à la machine virtuelle](#connect-to-the-virtual-machine).|
   | **Taille de la machine virtuelle** | Toute taille valide | La valeur par défaut **Standard_B2s** de ce modèle est suffisante pour ce guide de démarrage rapide. |
   | **Lieu**|[resourceGroup().location].| Ne changez pas cette valeur. |
   | **Nom du réseau virtuel**|Réseau virtuel dans lequel vous avez créé l'instance gérée.|
   | **Nom du sous-réseau**|Nom du sous-réseau que vous avez créé dans la procédure précédente| Ne choisissez pas le sous-réseau dans lequel vous avez créé l'instance gérée.|
   | **Emplacement des artefacts** | [deployment().properties.templateLink.uri] | Ne changez pas cette valeur. |
   | **Jeton SAP de l’emplacement des artefacts** | Laisser vide | Ne changez pas cette valeur. |

   ![créer une machine virtuelle client](./media/connect-vm-instance-configure/create-client-sql-vm.png)

   Si vous avez utilisé le nom de réseau virtuel suggéré et le sous-réseau par défaut dans [Créer votre instance de SQL Managed Instance](instance-create-quickstart.md), vous n'avez pas besoin de modifier les deux derniers paramètres. Sinon, vous devez remplacer ces valeurs par les valeurs que vous avez entrées lorsque vous avez configuré l’environnement réseau.

3. Cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
4. Sélectionnez **Acheter** pour déployer la machine virtuelle Azure dans votre réseau.
5. Sélectionnez l’icône **Notifications** pour afficher l’état du déploiement.

> [!IMPORTANT]
> Attendez environ 15 minutes après la création de la machine virtuelle afin de donner le temps aux scripts de post-création d'installer SQL Server Management Studio.

## <a name="connect-to-the-virtual-machine"></a>Connectez-vous à la machine virtuelle.

Les étapes suivantes expliquent comment vous connecter à votre machine virtuelle nouvellement créée à l'aide d'une connexion Bureau à distance.

1. Une fois le déploiement terminé, accédez à la ressource de machine virtuelle.

    ![Capture d’écran montrant le Portail Azure avec la page Vue d’ensemble d’une machine virtuelle sélectionnée et Connexion mis en surbrillance.](./media/connect-vm-instance-configure/vm.png)  

2. Sélectionnez **Connecter**.

   Un formulaire de fichier .rdp (Remote Desktop Protocol) s’affiche avec l’adresse IP publique et le numéro de port de la machine virtuelle.

   ![Formulaire RDP](./media/connect-vm-instance-configure/rdp.png)  

3. Sélectionnez **Télécharger le fichier RDP**.

   > [!NOTE]
   > Vous pouvez également utiliser le protocole SSH pour vous connecter à votre machine virtuelle.

4. Fermez le formulaire **Se connecter à la machine virtuelle**.
5. Pour vous connecter à votre machine virtuelle, ouvrez le fichier RDP téléchargé.
6. Lorsque vous y êtes invité, sélectionnez **Connexion**. Sur un Mac, vous devez disposer d'un client RDP semblable à ce [Client Bureau à distance](https://apps.apple.com/app/microsoft-remote-desktop-10/id1295203466?mt=12) disponible sur le Mac App Store.

7. Entrez le nom d'utilisateur et le mot de passe que vous avez spécifiés lors de la création de la machine virtuelle, puis choisissez **OK**.

8. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Choisissez **Oui** ou **Continuer** pour établir la connexion.

Vous êtes connecté à votre machine virtuelle dans le tableau de bord Gestionnaire de serveur.

## <a name="connect-to-sql-managed-instance"></a>Se connecter à SQL Managed Instance 

1. Dans la machine virtuelle, ouvrez SQL Server Management Studio.

   L'ouverture de SSMS est un peu plus longue à l'occasion de son premier démarrage, car il doit être configuré.
2. Dans la boîte de dialogue **Se connecter au serveur**, entrez le **nom d'hôte** complet de votre instance gérée dans la zone **Nom du serveur**. Sélectionnez **Authentification SQL Server**, fournissez votre nom d’utilisateur et votre mot de passe, puis sélectionnez **Connecter**.

    ![Connecter SSMS](./media/connect-vm-instance-configure/ssms-connect.png)  

Après vous être connecté, vous pouvez voir votre système et vos bases de données utilisateur dans le nœud Bases de données, ainsi que divers objets dans les nœuds Sécurité, Objets serveur, Réplication, Gestion, SQL Server Agent et XEvent Profiler.

## <a name="next-steps"></a>Étapes suivantes

- Pour un guide de démarrage rapide montrant comment se connecter à partir d’un ordinateur client local à l’aide d’une connexion point à site, consultez [Configurer une connexion point à site](point-to-site-p2s-configure.md).
- Pour obtenir une vue d'ensemble des options de connexion des applications, consultez [Connecter vos applications à SQL Managed Instance](connect-application-instance.md).
- Pour restaurer une base de données SQL Server locale existante dans une instance gérée, vous pouvez utiliser [Azure Database Migration Service pour la migration](../../dms/tutorial-sql-server-to-managed-instance.md) ou la [commande T-SQL RESTORE](restore-sample-database-quickstart.md) afin d'effectuer la restauration à partir d'un fichier de sauvegarde de base de données.
