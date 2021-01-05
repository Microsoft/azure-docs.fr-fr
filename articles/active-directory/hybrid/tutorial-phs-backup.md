---
title: 'Tutoriel :  Configurer PHS comme sauvegarde pour AD FS dans Azure AD Connect | Microsoft Docs'
description: Explique comment activer la synchronisation de hachage de mot de passe en tant que sauvegarde et pour AD FS.
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 04/25/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3d9f59906c566d80344891c0796a85b0a4972e68
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91313091"
---
# <a name="tutorial--setting-up-phs-as-backup-for-ad-fs-in-azure-ad-connect"></a>Tutoriel :  Configurer PHS en tant que sauvegarde pour AD FS dans Azure AD Connect

Ce tutoriel explique comment configurer la synchronisation de hachage de mot de passe en tant que sauvegarde et basculement pour AD FS.  Ce document explique également comment activer la synchronisation de hachage de mot de passe en tant que méthode d’authentification principale, si AD FS a échoué ou n'est plus disponible.

>[!NOTE] 
>Bien que ces étapes soient généralement effectuées en cas d’urgence ou de panne, il est recommandé de les tester et de vérifier vos procédures avant qu’une panne ne se produise.

>[!NOTE]
>Si vous n’avez pas accès au serveur Azure AD Connect ou si celui-ci n’a pas accès à Internet, vous pouvez contacter le [support Microsoft](https://support.microsoft.com/en-us/contactus/) pour qu’il vous aide à apporter des changements à la partie Azure AD.

## <a name="prerequisites"></a>Prérequis
Ce tutoriel s'appuie sur le tutoriel [ : Fédérer un environnement de forêt AD unique dans le cloud](tutorial-federation.md), un prérequis avant de suivre le présent tutoriel.  Si vous n’avez pas terminé ce tutoriel, faites-le avant de suivre la procédure décrite dans ce document.

>[!IMPORTANT]
>Avant de passer à PHS, vous devez créer une sauvegarde de votre environnement AD FS.  Pour cela, vous pouvez utiliser l’[outil de restauration rapide AD FS](/windows-server/identity/ad-fs/operations/ad-fs-rapid-restore-tool#how-to-use-the-tool).

## <a name="enable-phs-in-azure-ad-connect"></a>Activer PHS dans Azure AD Connect
La première étape, maintenant que nous disposons d'un environnement Azure AD Connect utilisant la fédération, consiste à activer la synchronisation de hachage de mot de passe et à autoriser Azure AD Connect à synchroniser les hachages.

Effectuez les actions suivantes :

1.  Double-cliquez sur l’icône Azure AD Connect créée sur le bureau.
2.  Cliquez sur **Configurer**.
3.  Sur la page Tâches supplémentaires, sélectionnez **Personnalisation des options de synchronisation**, puis cliquez sur **Suivant**.
4.  Entrez le nom d’utilisateur et le mot de passe de votre administrateur général.  Ce compte a été créé [ici](tutorial-federation.md#create-a-global-administrator-in-azure-ad) dans le précédent didacticiel.
5.  Dans l’écran **Connecter vos répertoires**, cliquez sur **Suivant**.
6.  Dans l'écran **Filtrage domaine ou unité organisationnelle**, cliquez sur **Suivant**.
7.  Dans l’écran **Fonctionnalités facultatives**, cochez **Synchronisation de hachage de mot de passe**, puis cliquez sur **Suivant**.
![Select](media/tutorial-phs-backup/backup1.png)</br>
8.  Dans l’écran **Prêt à configurer**, cliquez sur **Configurer**.
9.  Une fois la configuration terminée, cliquez sur **Quitter**.
10. Et voilà !  Vous avez terminé.  La synchronisation de hachage de mot de passe intervient et peut être utilisée en tant que sauvegarde si AD FS n’est plus disponible.

## <a name="switch-to-password-hash-synchronization"></a>Activer la synchronisation de hachage de mot de passe
Nous allons maintenant vous expliquer comment basculer vers la synchronisation de hachage de mot de passe. Avant de commencer, réfléchissez aux conditions dans lesquelles vous devez effectuer le basculement. N’effectuez pas le basculement pour des raisons temporaires, par exemple en cas de panne du réseau, de problème mineur lié à AD FS ou de problème qui affecte un sous-ensemble de vos utilisateurs. Si vous décidez d’effectuer le basculement car la résolution du problème prendrait trop de temps, effectuez les étapes suivantes :

> [!IMPORTANT]
> Sachez que la synchronisation des hachages de mot de passe avec Azure AD prend du temps.  Autrement dit, vous devrez peut-être attendre la fin de l’opération de synchronisation qui peut durer 3 heures avant de pouvoir vous authentifier à l’aide de hachages de mot de passe.

1. Double-cliquez sur l’icône Azure AD Connect créée sur le bureau.
2.  Cliquez sur **Configurer**.
3.  Sélectionnez **Modifier la connexion utilisateur**, puis cliquez sur **Suivant**.
![Modification](media/tutorial-phs-backup/backup2.png)</br>
4.  Entrez le nom d’utilisateur et le mot de passe de votre administrateur général.  Ce compte a été créé [ici](tutorial-federation.md#create-a-global-administrator-in-azure-ad) dans le précédent didacticiel.
5.  Dans l'écran **Connexion de l’utilisateur**, sélectionnez **Synchronisation de hachage de mot de passe** et cochez la case **Ne pas convertir les comptes d'utilisateur**.  
6.  Vérifiez que l'option par défaut **Activer l’authentification unique** est bien sélectionnée, puis cliquez **Suivant**.
7.  Dans l'écran **Activer l'authentification unique**, cliquez sur **Suivant**.
8.  Dans l’écran **Prêt à configurer**, cliquez sur **Configurer**.
9.  Une fois la configuration terminée, cliquez sur **Quitter**.
10. Les utilisateurs peuvent désormais utiliser leurs mots de passe pour se connecter à Azure et aux services Azure.

## <a name="test-signing-in-with-one-of-our-users"></a>Tester la connexion avec un des utilisateurs

1. Accédez à [https://myapps.microsoft.com](https://myapps.microsoft.com).
2. Connectez-vous avec un compte d’utilisateur créé dans le nouveau locataire.  Vous devez vous connecter en utilisant le format suivant : (user@domain.onmicrosoft.com). Saisissez le même mot de passe que celui utilisé par l’utilisateur pour se connecter en local.</br>
   ![Capture d’écran montrant un message indiquant que le test de connexion a réussi. ](media/tutorial-password-hash-sync/verify1.png)</br>

## <a name="switch-back-to-federation"></a>Revenir à la fédération
Nous allons maintenant vous montrer comment revenir à la fédération.  Pour cela, effectuez les étapes suivantes :

1.  Double-cliquez sur l’icône Azure AD Connect créée sur le bureau.
2.  Cliquez sur **Configurer**.
3.  Sélectionnez **Modifier la connexion utilisateur**, puis cliquez sur **Suivant**.
4.  Entrez le nom d’utilisateur et le mot de passe de votre administrateur général.  Ce compte a été créé [ici](tutorial-federation.md#create-a-global-administrator-in-azure-ad) dans le précédent tutoriel.
5.  Sur l’écran **Connexion utilisateur**, sélectionnez **Fédération avec AD FS** et cliquez sur **Suivant**.  
6. Sur la page Informations d’identification de l’administrateur de domaine, saisissez le nom d’utilisateur et le mot de passe contoso\Administrator, et cliquez sur **Suivant**.
7. Sur l’écran de la batterie de serveurs AD FS, cliquez sur **Suivant**.
8. Sur l’écran **Domaine Azure AD**, sélectionnez le domaine dans la liste déroulante, puis cliquez sur **Suivant**.
9. Dans l’écran **Prêt à configurer**, cliquez sur **Configurer**.
10. Une fois la configuration terminée, cliquez sur **Suivant**.
![Configurer](media/tutorial-phs-backup/backup4.png)</br>
11. Sur l’écran **Vérifier la connectivité de fédération** , cliquez sur **Vérifier**.  Vous devrez peut-être configurer des enregistrements DNS (ajouter des enregistrements A et AAAA) pour que cette opération aboutisse.
![Capture d’écran montrant l’écran Vérifier la connectivité de fédération et le bouton Vérifier.](media/tutorial-phs-backup/backup5.png)</br>
12. Cliquez sur **Quitter**.

## <a name="reset-the-ad-fs-and-azure-trust"></a>Réinitialiser la confiance entre AD FS et Azure
Nous devons maintenant réinitialiser la confiance entre AD FS et Azure.

1.  Double-cliquez sur l’icône Azure AD Connect créée sur le bureau.
2.  Cliquez sur **Configurer**.
3.  Sélectionnez **Gérer la fédération** et cliquez sur **Suivant**.
4.  Sélectionnez **Réinitialiser la confiance Azure AD** et cliquez sur **Suivant**.
![Réinitialiser](media/tutorial-phs-backup/backup6.png)</br>
5.  Sur l’écran **Connexion à Azure AD**, entrez le nom d’utilisateur et le mot de passe de votre administrateur général.
6.  Dans l’écran **Se connecter à AD FS**, entrez le nom d’utilisateur contoso\Administrator et le mot de passe, puis cliquez sur **Suivant**.
7.  Sur l’écran **Certificats**, cliquez sur **Suivant**.

## <a name="test-signing-in-with-a-user"></a>Test de connexion avec un utilisateur

1.  Accédez à [https://myapps.microsoft.com](https://myapps.microsoft.com).
2. Connectez-vous avec un compte d’utilisateur créé dans le nouveau locataire.  Vous devez vous connecter en utilisant le format suivant : (user@domain.onmicrosoft.com). Saisissez le même mot de passe que celui utilisé par l’utilisateur pour se connecter en local.
![Vérifier](media/tutorial-password-hash-sync/verify1.png)

Vous avez maintenant configuré un environnement d’identité hybride que vous pouvez utiliser à des fins de test et pour vous familiariser avec les fonctionnalités d’Azure.

## <a name="next-steps"></a>Étapes suivantes


- [Matériel et conditions préalables](how-to-connect-install-prerequisites.md) 
- [Paramètres Express](how-to-connect-install-express.md)
- [Synchronisation de hachage de mot de passe](how-to-connect-password-hash-synchronization.md)