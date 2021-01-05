---
title: Installer l’agent de provisionnement cloud Azure AD Connect
description: Cet article explique comment installer l’agent d’approvisionnement Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/16/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4c4ba11dabc8e805f3604afc2fc4388840ac67cf
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348532"
---
# <a name="install-the-azure-ad-connect-cloud-provisioning-agent"></a>Installer l’agent de provisionnement cloud Azure AD Connect
Ce document vous guide tout au long du processus d’installation de l’agent d’approvisionnement Azure Active Directory (Azure AD) Connecte, et explique comment le configurer au départ dans le portail Azure.

>[!IMPORTANT]
>Les instructions d’installation suivantes supposent que tous les [prérequis](how-to-prerequisites.md) ont été réunis.

L’installation et la configuration du provisionnement Azure AD Connect sont accomplies dans les étapes suivantes :

- [Comptes de service administrés de groupe](#group-managed-service-accounts) 
- [Installer l’agent](#install-the-agent)
- [Vérifier l’installation de l’agent](#verify-agent-installation)


## <a name="group-managed-service-accounts"></a>Group Managed Service Accounts
Un compte de service managé de groupe est un compte de domaine managé qui fournit la gestion automatique des mots de passe, la gestion simplifiée du nom de principal du service (SPN), la possibilité de déléguer la gestion à d’autres administrateurs et cette fonctionnalité s’étend sur plusieurs serveurs.  Azure AD Connect Cloud Sync prend en charge et recommande l’utilisation d’un compte de service géré de groupe pour l’exécution de l’agent.  Pour plus d’informations sur un gMSA, consultez [Comptes de service managés de groupe](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) 


### <a name="upgrading-an-existing-agent-to-use-the-gmsa-account"></a>Mise à niveau d’un agent existant pour utiliser le compte gMSA
Pour mettre à niveau un agent existant afin d’utiliser le compte gMSA créé pendant l’installation, il vous suffit de mettre à jour le service de l’agent vers la dernière version en exécutant AADConnectProvisioningAgent.msi.  Cette opération mettra à niveau le service vers la version la plus récente.  À présent, exécutez à nouveau l’assistant d’installation et fournissez les informations d’identification pour créer le compte lorsque vous y êtes invité.



## <a name="install-the-agent"></a>Installer l’agent
Pour installer l’agent, procédez comme suit.

 1. Connectez-vous au serveur que vous allez utiliser à l’aide des autorisations d’administrateur d’entreprise.
 2. Connectez-vous au portail Azure, puis accédez à **Azure Active Directory**.
 3. Dans le menu de gauche, sélectionnez **Azure AD Connect**.
 4. Sélectionnez **Gérer le provisionnement (préversion)**  > **Passer en revue tous les agents**.
 5. Téléchargez l’agent de provisionnement Azure AD Connect à partir du portail Azure.
   ![Télécharger un agent local](media/how-to-install/install-9.png)</br>
 6. Exécutez le programme d’installation d’approvisionnement AADConnectProvisioningAgent.msi.
 7. Dans l’écran **Package Agent d’approvisionnement Microsoft Azure AD Connect**, acceptez les termes du contrat de licence et sélectionnez **Installer**.
   ![Écran Package Agent d’approvisionnement Microsoft Azure AD Connect](media/how-to-install/install-1.png)</br>
 8. Une fois cette opération terminée, l’Assistant Configuration démarre. Connectez-vous avec votre compte d’administrateur général Azure AD.
 9. Dans l’écran **Configurer le compte de service**, sélectionnez **create gMSA** (créer un gMSA) ou **Use custom gMSA** (utiliser un gMSA personnalisé).  Si vous autorisez l’agent à créer le compte, il se nommera provAgentgMSA$. Si vous choisissez **Use custom gMSA** (utiliser un gMSA personnalisé). vous serez invité à spécifier ce compte.
 10. Entrez les informations d’identification de l’administrateur du domaine pour créer le compte de service géré de groupe qui sera utilisé pour exécuter le service Agent. Cliquez sur **Suivant**.  
   ![Créer gMSA](media/how-to-install/install-12.png)</br>
 11. Dans l’écran **Connecter Active Directory**, sélectionnez **Ajouter un répertoire**. Ensuite, connectez-vous avec votre compte Administrateur Active Directory. Cette opération ajoute votre répertoire local. 
 12. Vous pouvez également gérer la préférence des contrôleurs de domaine que l’agent utilisera en choisissant **Sélectionner la priorité du contrôleur de domaine** et en classant la liste des contrôleurs de domaine.   Cliquez sur **OK**.
  ![Classer les contrôleurs de domaine](media/how-to-install/install-2a.png)</br>
 13. Sélectionnez **Suivant**.
  ![Écran Connecter Active Directory](media/how-to-install/install-3a.png)</br>
 14.  Dans l’écran **Installation de l’agent**, confirmez les paramètres et le compte qui sera créé, puis cliquez sur **Confirmer**.
  ![Confirmer les paramètres](media/how-to-install/install-11.png)</br>
 15. Une fois cette opération terminée, vous devriez voir le message **Your agent installation is complete** (l’installation de votre agent est terminée). Sélectionnez **Quitter**.
  ![Écran Configuration terminée](media/how-to-install/install-4a.png)</br>
1. Si vous l’écran initial **Package Agent d’approvisionnement Microsoft Azure AD Connect** est toujours affiché, sélectionnez **Fermer**.

## <a name="verify-agent-installation"></a>Vérifier l’installation de l’agent
La vérification de l’agent se produit dans le portail Azure et sur le serveur local qui exécute l’agent.

### <a name="azure-portal-agent-verification"></a>Vérification de l’agent dans le portail Azure
Pour vérifier que l’agent est visible par Azure, procédez comme suit.

1. Connectez-vous au portail Azure.
1. À gauche, sélectionnez **Azure Active Directory** > **Azure AD Connect** Au centre, sélectionnez **Gérer le provisionnement (préversion)** .

   ![Portail Azure](media/how-to-install/install-6.png)</br>

1.  Dans l’écran **Provisionnement Azure AD (préversion)** , sélectionnez **Passer en revue tous les agents**.

    ![Option Passer en revue tous les agents](media/how-to-install/install-7.png)</br>
 
1. L’écran **Agents de provisionnement locaux** affiche les agents que vous avez installés. Vérifiez que l’agent en question est présent et qu’il est marqué comme étant *actif*.

   ![Écran Agents de provisionnement locaux](media/how-to-install/verify-1.png)</br>



### <a name="on-the-local-server"></a>Sur le serveur local
Pour vérifier que l’agent est en cours d’exécution, procédez comme suit.

1.  Connectez-vous au serveur avec un compte Administrateur.
1.  Ouvrez **Services** en y accédant ou en accédant à **Démarrer** > **Exécuter** > **Services.msc**.
1.  Sous **Services**, assurez-vous que le **Programme de mise à jour de l’agent Microsoft Azure AD Connect** et l’**agent d’approvisionnement Microsoft Azure AD Connect** sont présents, et que leur état est *En cours d’exécution*.

    ![Écran Services](media/how-to-install/troubleshoot-1.png)

>[!IMPORTANT]
>L’agent a été installé mais il faut le configurer et l’activer pour qu’il commence à synchroniser les utilisateurs. Pour configurer un nouvel agent, consultez [Créer une configuration pour le provisionnement cloud Azure AD Connect](how-to-configure.md).




## <a name="next-steps"></a>Étapes suivantes 

- [Présentation du provisionnement](what-is-provisioning.md)
- [Présentation du provisionnement cloud Azure AD Connect](what-is-cloud-provisioning.md)
