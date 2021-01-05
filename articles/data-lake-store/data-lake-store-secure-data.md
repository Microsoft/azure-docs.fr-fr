---
title: Sécurisation des données stockées dans Azure Data Lake Storage Gen1 | Microsoft Docs
description: Apprenez à sécuriser les données dans Azure Data Lake Storage Gen1 à l’aide de groupes et de listes de contrôle d’accès
services: data-lake-store
documentationcenter: ''
author: twooley
manager: mtillman
editor: cgronlun
ms.assetid: ca35e65f-3986-4f1b-bf93-9af6066bb716
ms.service: data-lake-store
ms.devlang: na
ms.topic: how-to
ms.date: 03/26/2018
ms.author: twooley
ms.openlocfilehash: ac7666f4c4e68d24499f9c097dc9bd021d270355
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370693"
---
# <a name="securing-data-stored-in-azure-data-lake-storage-gen1"></a>Sécurisation des données stockées dans Azure Data Lake Storage Gen1
La sécurisation des données dans Azure Data Lake Storage Gen1 se fait en trois étapes.  Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) et les listes de contrôle d’accès (ACL, access control list) doivent être définis pour offrir aux utilisateurs et aux groupes de sécurité un accès total aux données.

1. Commencez par créer des groupes de sécurité dans Azure Active Directory (Azure AD). Ces groupes de sécurité sont utilisés pour implémenter le contrôle d’accès en fonction du rôle Azure (Azure RBAC) dans le portail Azure. Pour plus d’informations, consultez [Azure RBAC](../role-based-access-control/role-assignments-portal.md).
2. Affectez les groupes de sécurité Azure AD au compte Data Lake Storage Gen1. Ceci contrôle l’accès au compte Data Lake Storage Gen1 à partir du portail et les opérations de gestion à partir du portail ou des API.
3. Affectez les groupes de sécurité Azure AD comme des listes de contrôle d’accès (ACL) sur le système de fichiers Data Lake Storage Gen1.
4. En outre, vous pouvez définir une plage d’adresses IP pour les clients pouvant accéder aux données de Data Lake Storage Gen1.

Cet article explique comment utiliser le portail Azure pour effectuer les tâches ci-dessus. Pour obtenir des informations détaillées sur la manière dont Data Lake Storage Gen1 implémente la sécurité au niveau du compte et des données, consultez [Sécurité dans Azure Data Lake Storage Gen1](data-lake-store-security-overview.md). Pour des informations détaillées sur la façon dont les listes de contrôle d’accès sont implémentées dans Data Lake Storage Gen1, consultez [Vue d’ensemble du contrôle d’accès dans Data Lake Storage Gen1](data-lake-store-access-control.md).

## <a name="prerequisites"></a>Prérequis
Avant de commencer ce didacticiel, vous devez disposer des éléments suivants :

* **Un abonnement Azure** . Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Data Lake Storage Gen1** . Pour savoir comment en créer un, voir [Prise en main d’Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).

## <a name="create-security-groups-in-azure-active-directory"></a>Créer des groupes de sécurité dans Azure Active Directory
Pour obtenir des instructions sur la création de groupes de sécurité Azure AD et l’ajout d’utilisateurs au groupe, consultez [Gestion des groupes de sécurité dans Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

> [!NOTE] 
> Vous pouvez ajouter tant des utilisateurs que d’autres groupes à un groupe dans Azure AD à l’aide du portail Azure. Toutefois, pour ajouter un principal de service à un groupe, utilisez le [module PowerShell d’Azure AD](../active-directory/enterprise-users/groups-settings-v2-cmdlets.md).
> 
> ```powershell
> # Get the desired group and service principal and identify the correct object IDs
> Get-AzureADGroup -SearchString "<group name>"
> Get-AzureADServicePrincipal -SearchString "<SPI name>"
> 
> # Add the service principal to the group
> Add-AzureADGroupMember -ObjectId <Group object ID> -RefObjectId <SPI object ID>
> ```
 
## <a name="assign-users-or-security-groups-to-data-lake-storage-gen1-accounts"></a>Affecter les utilisateurs ou les groupes de sécurité aux comptes Data Lake Storage Gen1
Lorsque vous affectez des utilisateurs ou des groupes de sécurité aux comptes Data Lake Storage Gen1, vous contrôlez l’accès aux opérations de gestion sur le compte à l’aide du portail Azure et des API Azure Resource Manager. 

1. Ouvrez un compte Data Lake Storage Gen1. Dans le volet gauche, cliquez sur **Toutes les ressources** , puis, dans le volet Toutes les ressources, cliquez sur le nom du compte auquel vous souhaitez affecter un utilisateur ou un groupe de sécurité.

2. Dans le panneau de votre compte Data Lake Storage Gen1, cliquez sur **Contrôle d’accès (IAM)** . Le panneau par défaut répertorie les propriétaires de l’abonnement en tant que propriétaires.
   
    ![Affectation d’un groupe de sécurité à un compte Azure Data Lake Storage Gen1](./media/data-lake-store-secure-data/adl.select.user.icon1.png "Affectation d’un groupe de sécurité à un compte Azure Data Lake Storage Gen1")

3. Dans le panneau **Contrôle d’accès (IAM)** , cliquez sur **Ajouter** pour ouvrir le panneau **Ajouter des autorisations** . Dans le panneau **Ajouter des autorisations** , sélectionnez un **rôle** pour l’utilisateur ou le groupe. Recherchez le groupe de sécurité créé précédemment dans Azure Active Directory et sélectionnez-le. Si la liste d’utilisateurs et de groupes est longue, utilisez la zone de texte **Sélectionner** pour filtrer sur le nom du groupe. 
   
    ![Ajout d’un rôle à l’utilisateur](./media/data-lake-store-secure-data/adl.add.user.1.png "Ajouter un rôle à l'utilisateur")
   
    Les rôles **Propriétaire** et **Collaborateur** fournissent l’accès à différentes fonctions d’administration sur le compte Data Lake. Pour les utilisateurs qui interagiront avec les données du Data Lake, mais qui auront toujours besoin d’afficher les informations de gestion de compte, vous pouvez les ajouter au rôle **Lecteur** . L’étendue de ces rôles est limitée aux opérations de gestion associées au compte Data Lake Storage Gen1.
   
    Pour les opérations sur les données, des autorisations pour chaque système de fichiers définissent ce que les utilisateurs peuvent faire. Par conséquent, un utilisateur ayant un rôle de Lecteur peut afficher uniquement les paramètres d'administration associés au compte mais peut potentiellement lire et écrire des données en fonction des autorisations du système de fichiers qui lui sont affectées. Les autorisations de système de fichiers Data Lake Storage Gen1 sont décrites dans [Affecter un groupe de sécurité comme ACL au système de fichiers Azure Data Lake Storage Gen1](#filepermissions).

    > [!IMPORTANT]
    > Seul le rôle **Propriétaire** active automatiquement l’accès au système de fichiers. Les rôles **Contributeur** , **Lecteur** et autres requièrent des ACL pour activer n’importe quel niveau d’accès aux dossiers et aux fichiers.  Le rôle **Propriétaire** fournit des autorisations d’accès aux dossiers et aux fichiers de niveau super utilisateur qui ne peuvent pas être remplacées via des ACL. Pour plus d’informations sur la correspondance entre les stratégies Azure RBAC et l’accès aux données, consultez [Azure RBAC pour la gestion des comptes](data-lake-store-security-overview.md#azure-rbac-for-account-management).

4. Pour ajouter un utilisateur ou un groupe qui n’est pas répertorié dans le panneau **Ajouter des autorisations** , vous pouvez l’inviter en tapant son adresse e-mail dans la zone de texte **Sélectionner** , puis en le sélectionnant dans la liste.
   
    ![Ajout d’un groupe de sécurité](./media/data-lake-store-secure-data/adl.add.user.2.png "Ajouter un groupe de sécurité")
   
5. Cliquez sur **Enregistrer** . Vous devez voir le groupe de sécurité ajouté comme indiqué ci-dessous.
   
    ![Groupe de sécurité ajouté](./media/data-lake-store-secure-data/adl.add.user.3.png "Groupe de sécurité ajouté")

6. Votre groupe de sécurité ou utilisateur a désormais accès au compte Data Lake Storage Gen1. Si vous souhaitez fournir un accès à des utilisateurs en particulier, vous pouvez les ajouter au groupe de sécurité. De même, si vous souhaitez révoquer l'accès d'un utilisateur, vous pouvez le supprimer du groupe de sécurité. Vous pouvez également affecter plusieurs groupes de sécurité à un compte. 

## <a name="assign-users-or-security-groups-as-acls-to-the-data-lake-storage-gen1-file-system"></a><a name="filepermissions"></a>Affecter des utilisateurs ou des groupes de sécurité en tant qu’ACL au système de fichiers Data Lake Storage Gen1
En affectant des groupes de sécurité ou des utilisateurs au système de fichiers Data Lake Storage Gen1, vous définissez un contrôle d’accès aux données stockées dans Data Lake Storage Gen1.

1. Dans le panneau de votre compte Data Lake Storage Gen1, cliquez sur **Explorateur de données** .
   
    ![Affichage des données avec l’Explorateur de données](./media/data-lake-store-secure-data/adl.start.data.explorer.png "Affichage des données avec l’Explorateur de données")
2. Dans le panneau **Explorateur de données** , cliquez sur le dossier pour lequel vous souhaitez configurer l’ACL, puis cliquez sur **Accès** . Pour affecter des ACL à un fichier, vous devez tout d’abord cliquer sur le fichier pour en afficher un aperçu, puis cliquer sur **Accès** dans le panneau **Aperçu du fichier** .
   
    ![Définition de listes de contrôle d’accès sur le système de fichiers Data Lake Storage Gen1](./media/data-lake-store-secure-data/adl.acl.1.png "Définition de listes de contrôle d’accès sur le système de fichiers Data Lake Storage Gen1")
3. Le panneau **Accès** répertorie les propriétaires et les autorisations déjà affectés à la racine. Cliquez sur l’icône **Ajouter** pour ajouter des ACL d’accès supplémentaires.
    > [!IMPORTANT]
    > La définition d’autorisations d’accès pour un fichier unique ne donne pas nécessairement accès à ce fichier à un utilisateur ou un groupe. L’emplacement du fichier doit être accessible à l’utilisateur ou au groupe attribué. Pour obtenir plus d’informations et des exemples, consultez [Scénarios courants liés aux autorisations](data-lake-store-access-control.md#common-scenarios-related-to-permissions).
   
    ![Listing des accès standard et personnalisés](./media/data-lake-store-secure-data/adl.acl.2.png "Lister les accès standard et personnalisés")
   
   * Les sections **Propriétaires** et **Tous les autres** permettent d’attribuer un accès de type UNIX, où vous spécifiez la lecture, l’écriture et l’exécution (rwx) pour trois classes d’utilisateurs distinctes : propriétaire, groupe et autres.
   * **Autorisations attribuées** correspond aux ACL POSIX, qui vous permettent de définir des autorisations pour des utilisateurs ou des groupes nommés spécifiques ne se limitant pas au propriétaire ou au groupe du fichier. 
     
     Pour plus d'informations, consultez la page [ACL HDFS](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html#ACLs_Access_Control_Lists). Pour plus d’informations sur l’implémentation des ACL dans Data Lake Storage Gen1, consultez [Contrôle d’accès dans Data Lake Storage Gen1](data-lake-store-access-control.md).
4. Cliquez sur l’icône **Ajouter** pour ouvrir le panneau **Affecter des autorisations** . Dans ce panneau, cliquez sur **Sélectionner un utilisateur ou un groupe** , puis, dans le panneau **Sélectionner un utilisateur ou un groupe** , recherchez le groupe de sécurité créé précédemment dans Azure Active Directory. Si la liste de groupes est longue, utilisez la zone de texte en haut pour filtrer par nom de groupe. Cliquez sur le groupe que vous souhaitez ajouter, puis cliquez sur **Sélectionner** .
   
    ![Ajout d’un groupe](./media/data-lake-store-secure-data/adl.acl.3.png "Ajouter un groupe")
5. Cliquez sur **Sélectionner des autorisations** , sélectionnez les autorisations, puis indiquez si celles-ci doivent être appliquées de manière récursive et affectées en tant qu’ACL d’accès, ACL par défaut ou les deux. Cliquez sur **OK** .
   
    ![Capture d’écran du panneau Attribuer des autorisations avec l’option Sélectionner des autorisations mise en évidence, et du panneau Sélectionner des autorisations avec l’option OK mise en évidence.](./media/data-lake-store-secure-data/adl.acl.4.png "Affecter des autorisations à un groupe")
   
    Pour plus d’informations sur les autorisations dans Data Lake Storage Gen1 et sur les ACL par défaut ou d’accès, consultez [Contrôle d’accès dans Data Lake Storage Gen1](data-lake-store-access-control.md).
6. Une fois que vous avez cliqué sur **OK** dans le panneau **Sélectionner des autorisations** , le groupe qui vient d’être ajouté et les autorisations associées sont répertoriés dans le panneau **Accès** .
   
    ![Capture d’écran du panneau Accès avec l’option Engineering données mise en évidence.](./media/data-lake-store-secure-data/adl.acl.5.png "Affecter des autorisations à un groupe")
   
   > [!IMPORTANT]
   > Dans la version actuelle, vous pouvez avoir jusqu’à 28 entrées sous **Autorisations attribuées** . Pour ajouter plus de 28 utilisateurs, vous devez créer des groupes de sécurité, ajouter les utilisateurs aux groupes de sécurité et fournir à ces groupes de sécurité un accès au compte Data Lake Storage Gen1.
   > 
   > 
7. Si nécessaire, vous pouvez également modifier les autorisations d'accès après avoir ajouté le groupe. Cochez ou décochez la case de chaque type d'autorisation (lecture, écriture, exécution) selon que vous souhaitez retirer ou affecter cette autorisation au groupe de sécurité. Cliquez sur **Enregistrer** pour enregistrer les modifications, ou sur **Ignorer** pour annuler les modifications.

## <a name="set-ip-address-range-for-data-access"></a>Définir la plage d’adresses IP pour accéder aux données
Data Lake Storage Gen1 vous permet de mieux verrouiller l’accès à votre magasin de données au niveau du réseau. Vous pouvez activer le pare-feu, spécifier une adresse IP ou définir une plage d’adresses IP pour vos clients approuvés. Une fois qu’il est activé, seuls les clients qui possédant des adresses IP dans la plage définie peuvent se connecter au magasin.

![Paramètres de pare-feu et adresse IP](./media/data-lake-store-secure-data/firewall-ip-access.png "Paramètres de pare-feu et adresse IP")

## <a name="remove-security-groups-for-a-data-lake-storage-gen1-account"></a>Supprimer les groupes de sécurité d’un compte Data Lake Storage Gen1
Lorsque vous supprimez des groupes de sécurité dans les comptes Data Lake Storage Gen1, vous modifiez uniquement l’accès aux opérations de gestion effectuées sur le compte à l’aide du portail Azure et des API Azure Resource Manager.  

L’accès aux données n’est pas modifié et est toujours géré par les ACL d’accès,  exception faite des utilisateurs/groupes du rôle Propriétaires.  Les utilisateurs/groupes supprimés du rôle Propriétaires ne sont plus super utilisateurs et leur accès bascule sur les paramètres d’ACL d’accès. 

1. Dans le panneau de votre compte Data Lake Storage Gen1, cliquez sur **Contrôle d’accès (IAM)** . 
   
    ![Affectation d’un groupe de sécurité à un compte Data Lake Storage Gen1](./media/data-lake-store-secure-data/adl.select.user.icon.png "Affectation d’un groupe de sécurité à un compte Data Lake Storage Gen1")
2. Dans le panneau **Contrôle d’accès (IAM)** , cliquez sur les groupes de sécurité à supprimer. Cliquez sur **Supprimer** .
   
    ![Groupe de sécurité supprimé](./media/data-lake-store-secure-data/adl.remove.group.png "Groupe de sécurité supprimé")

## <a name="remove-security-group-acls-from-a-data-lake-storage-gen1-file-system"></a>Supprimer les ACL d’un groupe de sécurité du système de fichiers Data Lake Storage Gen1
Quand vous supprimez les ACL d’un groupe de sécurité d’un système de fichiers Data Lake Storage Gen1, vous modifiez l’accès aux données du compte Data Lake Storage Gen1.

1. Dans le panneau de votre compte Data Lake Storage Gen1, cliquez sur **Explorateur de données** .
   
    ![Création de répertoires dans un compte Data Lake Storage Gen1](./media/data-lake-store-secure-data/adl.start.data.explorer.png "Création de répertoires dans un compte Data Lake Storage Gen1")
2. Dans le panneau **Explorateur de données** , cliquez sur le dossier pour lequel vous souhaitez supprimer l’ACL, puis cliquez sur **Accès** . Pour supprimer des ACL pour un fichier, vous devez tout d’abord cliquer sur le fichier pour en afficher un aperçu, puis cliquer sur **Accès** dans le panneau **Aperçu du fichier** . 
   
    ![Définition de listes de contrôle d’accès sur le système de fichiers Data Lake Storage Gen1](./media/data-lake-store-secure-data/adl.acl.1.png "Définition de listes de contrôle d’accès sur le système de fichiers Data Lake Storage Gen1")
3. Dans le panneau **Accès** , cliquez sur le groupe de sécurité à supprimer. Dans le panneau **Détails de l’accès** , cliquez sur **Supprimer** .
   
    ![Capture d’écran du panneau Accès avec l’option Engineering données mise en évidence, et du panneau Détails de l’accès avec l’option Supprimer mise en évidence.](./media/data-lake-store-secure-data/adl.remove.acl.png "Affecter des autorisations à un groupe")

## <a name="see-also"></a>Voir aussi
* [Vue d’ensemble d’Azure Data Lake Storage Gen1](data-lake-store-overview.md)
* [Copier des données d’objets blob Stockage Azure vers Data Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md)
* [Utiliser Azure Data Lake Analytics avec Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Utiliser Azure HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)
* [Prise en main de Data Lake Storage Gen1 à l’aide de PowerShell](data-lake-store-get-started-powershell.md)
* [Prise en main de Data Lake Storage Gen1 à l’aide du kit de développement logiciel (SDK) .NET](data-lake-store-get-started-net-sdk.md)
* [Accéder aux journaux de diagnostic de Data Lake Storage Gen1](data-lake-store-diagnostic-logs.md)
