---
title: Déplacez la base de données Azure AD Connect de SQL Server Express vers SQL Server. | Microsoft Docs
description: Ce document décrit comment déplacer la base de données Azure AD Connect du serveur local SQL Server Express vers un serveur distant SQL Server.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/29/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 94710e99fa7d04d757f2ad5fd7b2d3f6e01371d1
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91306340"
---
# <a name="move-azure-ad-connect-database-from-sql-server-express-to-sql-server"></a>Déplacer la base de données Azure AD Connect de SQL Server Express vers SQL Server 

Ce document décrit comment déplacer la base de données Azure AD Connect du serveur local SQL Server Express vers un serveur distant SQL Server.  Vous pouvez utiliser les procédures suivantes pour réaliser cette tâche.

## <a name="about-this-scenario"></a>À propos du scénario
Voici quelques informations sur le scénario.  Dans ce scénario, la version (1.1.819.0) d’Azure AD Connect est installée sur un contrôleur de domaine unique Windows Server 2016.  Il utilise l’édition SQL Server 2012 Express comme base de données.  La base de données sera déplacée vers un serveur SQL Server 2017.

![architecture du scénario](media/how-to-connect-install-move-db/move1.png)

## <a name="move-the-azure-ad-connect-database"></a>Déplacer la base de données Azure AD Connect
Suivez les étapes suivantes pour déplacer la base de données Azure AD Connect vers un serveur SQL distant.

1. Sur le serveur Azure AD Connect, allez à **Services** et arrêtez le service **Microsoft Azure AD**.
2. Localisez le dossier **%ProgramFiles%\Microsoft Azure AD Sync\Data** et copiez les fichiers **ADSync.mdf** et **ADSync_log.ldf** dans le serveur SQL Server distant.
3. Redémarrez le service **Microsoft Azure AD Sync** sur le serveur Azure AD Connect.
4. Pour désinstaller Azure AD Connect, allez dans le Panneau de configuration - - Programmes - Programmes et fonctionnalités.  Sélectionnez Microsoft Azure AD Connect et cliquez sur Désinstaller dans le coin supérieur.
5. Sur le serveur SQL distant, ouvrez SQL Server Management Studio.
6. Dans Bases de données, cliquez avec le bouton droit et sélectionnez Joindre.
7. À l’écran **Joindre des bases de données**, cliquez sur **Ajouter** et accédez au fichier ADSync.mdf.  Cliquez sur **OK**.
   ![Joindre une base de données](media/how-to-connect-install-move-db/move2.png)

8. Une fois la base de données jointe, retournez sur le serveur Azure AD Connect et installez Azure AD Connect.
9. Une fois l’installation du fichier MSI terminée, l’Assistant Azure AD Connect démarre le programme d’installation en mode Express. Fermez la fenêtre en cliquant sur l’icône Quitter.
   ![Screenshot that shows the "Welcome to Azure A D Connect" page with "Express Settings" in the left-side menu highlighted.](./media/how-to-connect-install-move-db/db1.png)
10. Démarrez une nouvelle invite de commandes ou session PowerShell. Accédez au dossier \<drive>\program files\Microsoft Azure AD Connect. Exécutez la commande .\AzureADConnect.exe /useexistingdatabase pour démarrer l’Assistant Azure AD Connect en mode d’installation « Utiliser une base de données existante ».
    ![PowerShell](./media/how-to-connect-install-move-db/db2.png)
11. L’écran d’accueil d’Azure AD Connect s’affiche. Après avoir accepté les termes du contrat de licence et la déclaration de confidentialité, cliquez sur **Continuer**.
    ![Screenshot that shows the "Welcome to Azure A D Connect" page](./media/how-to-connect-install-move-db/db3.png)
12. Dans l’écran **Installer les composants nécessaires**, l’option **Utiliser un SQL Server existant** est activée. Spécifiez le nom du serveur SQL qui héberge la base de données ADSync. Si l’instance du moteur SQL utilisée pour héberger la base de données ADSync n’est pas l’instance par défaut sur le serveur SQL, vous devez spécifier le nom de l’instance du moteur SQL. De plus, si l’exploration SQL n’est pas activée, vous devez également spécifier le numéro de port de l’instance du moteur SQL. Par exemple :         
    ![Screenshot that shows the "Install required components" page.](./media/how-to-connect-install-move-db/db4.png)           

13. Dans l’écran **Se connecter à Azure AD**, vous devez fournir les informations d’identification d’un administrateur général de votre annuaire Azure AD. Nous vous recommandons d’utiliser un compte du domaine onmicrosoft.com par défaut. Ce compte est uniquement utilisé pour créer un compte de service dans Azure AD et n’est plus utilisé une fois l’assistant terminé.
    ![Connexion](./media/how-to-connect-install-move-db/db5.png)
 
14. Dans l’écran **Connexion de vos annuaires**, une icône de croix rouge est affichée en regard de la forêt AD existante configurée pour la synchronisation d’annuaire. Pour synchroniser les modifications à partir d’une forêt AD locale, un compte de domaine AD DS est nécessaire. L’Assistant Azure AD Connect ne peut pas récupérer les informations d’identification du compte AD DS stockées dans la base de données ADSync, car elles sont chiffrées et peuvent uniquement être déchiffrées par le serveur Azure AD Connect précédent. Cliquez sur **Modifier les informations d’identification** pour spécifier le compte AD DS pour la forêt AD.
    ![Directories](./media/how-to-connect-install-move-db/db6.png)
 

15. Dans la boîte de dialogue contextuelle, vous pouvez (i) entrer les informations d’identification d’un administrateur d’entreprise et laisser Azure AD Connect créer le compte AD DS pour vous, ou (ii) créer vous-même le compte AD DS et fournir ses informations d’identification à Azure AD Connect. Une fois que vous avez sélectionné une option et fourni les informations d’identification nécessaires, cliquez sur **OK** pour fermer la boîte de dialogue contextuelle.
    ![Screenshot of the "A D forest account" pop-up dialog with the "Create new A D account" selected.](./media/how-to-connect-install-move-db/db7.png)
 

16. Une fois les informations d’identification fournies, la croix rouge est remplacée par une coche verte. Cliquez sur **Suivant**.
    ![Screenshot that shows the "Connect your directories" page after entering account credentials.](./media/how-to-connect-install-move-db/db8.png)
 

17. Dans l’écran **Prêt à configurer**, cliquez sur **Installer**.
    ![Bienvenue](./media/how-to-connect-install-move-db/db9.png)
 

18. Une fois l’installation terminée, le serveur Azure AD Connect est automatiquement activé pour le Mode de préproduction. Nous vous recommandons de vérifier la présence de modifications inattendues dans la configuration du serveur et les exportations en attente avant de désactiver le Mode de préproduction. 

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
- [Installer Azure AD Connect à l’aide d’une base de données ADSync existante](how-to-connect-install-existing-database.md)
- [Installer Azure AD Connect à l’aide d’autorisations administrateur déléguées SQL](how-to-connect-install-sql-delegation.md)

