---
title: 'Azure AD Connect : Mise à niveau à partir de DirSync | Microsoft Docs'
description: Découvrez comment mettre à niveau la synchronisation d’annuaires vers Azure AD Connect. Cet article décrit les étapes de mise à niveau à partir de DirSync vers Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: baf52da7-76a8-44c9-8e72-33245790001c
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/13/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8aa45294de4ef644c20ef66b7163706dca9759d3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996523"
---
# <a name="azure-ad-connect-upgrade-from-dirsync"></a>Azure AD Connect : Effectuer une mise à niveau à partir de DirSync
Azure AD Connect est le successeur de DirSync. Cette rubrique explique les différentes façons de procéder à une mise à niveau à partir de DirSync. Ces étapes ne fonctionnent pas pour la mise à niveau à partir d’une autre version d’Azure AD Connect ou d’Azure AD Sync.

Avant de commencer l’installation d’Azure AD Connect, veillez à [télécharger Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771) et effectuer les étapes préalables décrites dans [Azure AD Connect : matériel et prérequis](how-to-connect-install-prerequisites.md). En particulier, il est recommandé de se renseigner sur les éléments suivants, dans la mesure où ces zones sont différentes de celles de DirSync :

* version requise de .Net et de PowerShell. Les versions plus récentes doivent être sur le serveur dont DirSync a besoin.
* La configuration du serveur proxy. Si vous utilisez un serveur proxy pour accéder à internet, ce paramètre doit être configuré avant d’effectuer la mise à niveau. DirSync a toujours utilisé le serveur proxy configuré pour l’utilisateur qui effectue l’installation, alors qu’Azure AD Connect utilise les paramètres de l’ordinateur.
* Les URL doivent être ouvertes dans le serveur proxy. Pour les scénarios de base, qui sont également pris en charge par DirSync, les exigences sont les mêmes. Si vous souhaitez utiliser une des nouvelles fonctionnalités d’Azure AD Connect, certaines nouvelles URL doivent être ouvertes.

> [!NOTE]
> Une fois que vous avez activé votre nouveau serveur Azure AD Connect pour démarrer la synchronisation des modifications à Azure AD, vous ne devez pas restaurer à l’aide de DirSync ou Azure AD Sync. La rétrogradation à partir d’Azure AD Connect pour les clients hérités, notamment DirSync et Azure AD Sync n’est pas prise en charge et peut entraîner des problèmes tels que la perte de données dans Azure AD.

Si vous n’effectuez pas une mise à niveau à partir de DirSync, consultez la section Documentation connexe pour d’autres scénarios.

## <a name="upgrade-from-dirsync"></a>Effectuer une mise à niveau à partir de DirSync
Selon votre déploiement DirSync actuel, il existe différentes options pour la mise à niveau. Si la durée de mise à niveau attendue est inférieure à 3 heures, nous vous recommandons de l’effectuer sur place. Si elle est supérieure à 3 heures, nous vous recommandons d’effectuer un déploiement en parallèle sur un autre serveur. Nos estimations montrent que, si vous avez plus de 50 000 objets, la mise à niveau prendra plus de 3 heures.

| Scénario |
| --- |
| [Mise à niveau sur place](#in-place-upgrade) |
| [Déploiement en parallèle](#parallel-deployment) |

> [!NOTE]
> Lorsque vous envisagez de mettre à niveau à partir de DirSync vers Azure AD Connect, ne désinstallez pas DirSync vous-même avant la mise à niveau. Azure AD Connect lit et migre la configuration de DirSync et procède à la désinstallation après l'inspection du serveur.

**Mise à niveau sur place**  
L’Assistant affiche la durée estimée de l’opération. Cette estimation est basée sur l’hypothèse qu’il faut 3 heures pour mettre à niveau une base de données contenant 50 000 objets (utilisateurs, contacts et groupes). Azure AD Connect recommande une mise à niveau sur place si votre base de données contient moins de 50 000 objets. Si vous décidez de continuer, vos paramètres actuels sont appliqués automatiquement au cours de la mise à niveau et votre serveur rétablit automatiquement la synchronisation active.

Si vous souhaitez effectuer une migration de la configuration et un déploiement en parallèle, vous pouvez ne pas effectuer la mise à niveau sur place. Vous pouvez, par exemple, prendre l'opportunité d'actualiser le matériel et le système d'exploitation. Pour plus d’informations, consultez la section [Déploiement en parallèle](#parallel-deployment).

**Déploiement en parallèle**  
Un déploiement en parallèle est recommandé si vous avez plus de 50 000 objets. Ce déploiement permet d’éviter les retards opérationnels rencontrés par vos utilisateurs. L’installation d’Azure AD Connect tente d’estimer le temps d’arrêt associé à la mise à niveau, mais si vous avez déjà mis à niveau DirSync, votre propre expérience est sans doute plus fiable.

### <a name="supported-dirsync-configurations-to-be-upgraded"></a>Configurations de DirSync prises en charge à mettre à niveau
Les modifications de configuration suivantes sont prises en charge avec la mise à niveau de DirSync :

* Filtrage domaine et unité organisationnelle
* Autre ID (UPN)
* Synchronisation de mot de passe et paramètres hybrides d'Exchange
* Vos paramètres domaine/forêt et Azure AD
* Filtrage basé sur les attributs de l'utilisateur

La modification suivante ne peut pas être mise à niveau. Si vous avez cette configuration, la mise à niveau est bloquée :

* Modifications de DirSync non prises en charge, par exemple : attributs supprimés et utilisation d’une DLL d’extension personnalisée

![Blocage de la mise à niveau](./media/how-to-dirsync-upgrade-get-started/analysisblocked.png)

Dans ces cas, il est recommandé d’installer un nouveau serveur Azure AD Connect en [mode de préproduction](how-to-connect-sync-staging-server.md) et de vérifier l’ancienne configuration de DirSync et la nouvelle configuration d’Azure AD Connect. Appliquez de nouveau les éventuelles modifications à l’aide de la configuration personnalisée, comme décrit dans [Azure AD Connect Sync : configuration personnalisée](how-to-connect-sync-whatis.md).

Les mots de passe utilisés par DirSync pour les comptes de service ne peuvent pas être récupérés et ne sont pas migrés. Ces mots de passe sont réinitialisés lors de la mise à niveau.

### <a name="high-level-steps-for-upgrading-from-dirsync-to-azure-ad-connect"></a>Étapes générales pour la mise à niveau à partir de DirSync vers Azure AD Connect
1. Bienvenue dans Azure AD Connect
2. Analyse de la configuration DirSync actuelle
3. Collecte du mot de passe administrateur global Azure AD
4. Collecte des informations d’identification pour le compte d’administrateur entreprise (utilisé uniquement lors de l’installation d’Azure AD Connect)
5. Installation d'Azure AD Connect
   * Désinstaller DirSync (ou le désactiver temporairement)
   * Installer Azure AD Connect
   * Synchronisation, si nécessaire

Des étapes supplémentaires sont nécessaires dans les cas suivants :

* Vous utilisez actuellement un serveur SQL complet, local ou distant
* Vous avez plus de 50 000 objets à synchroniser

## <a name="in-place-upgrade"></a>Mise à niveau sur place
1. Lancez le programme d'installation d'Azure AD Connect (MSI).
2. Lisez et acceptez les termes du contrat de licence et la déclaration de confidentialité.  
   ![Bienvenue dans Azure AD](./media/how-to-dirsync-upgrade-get-started/Welcome.png)
3. Cliquez sur le bouton Suivant pour analyser votre installation DirSync existante.  
   ![Analyse de l’installation existante de la synchronisation d’annuaires](./media/how-to-dirsync-upgrade-get-started/Analyze.png)
4. Une fois l’analyse terminée, nous proposons des recommandations sur la marche à suivre.  
   * Si vous utilisez SQL Server Express et que vous avez moins de 50 000 objets, l'écran suivant s'affiche :  
     ![Analyse terminée, mise à niveau à partir de la synchronisation d’annuaires prête](./media/how-to-dirsync-upgrade-get-started/AnalysisReady.png)
   * Si vous utilisez une instance SQL Server complète pour DirSync, vous voyez cette page à la place :  
     ![screenshot that shows the existing SQL database server being used.](./media/how-to-dirsync-upgrade-get-started/AnalysisReadyFullSQL.png)  
     Les informations concernant le serveur de base de données SQL Server utilisé par DirSync s’affichent. Apportez des modifications, si nécessaire. Cliquez sur **Suivant** pour continuer l'installation.
   * Si vous avez plus de 50 000 objets, vous voyez cet écran à la place :  
     ![Screenshot that shows the screen you see when you have more than 50,000 objects to upgrade.](./media/how-to-dirsync-upgrade-get-started/AnalysisRecommendParallel.png)  
     Pour procéder à une mise à niveau sur place, cochez la case en regard du message : **Continuez à mettre à niveau DirSync sur cet ordinateur.**
     Pour effectuer un [parallel deployment](#parallel-deployment) , exportez les paramètres de configuration de DirSync et copiez-les sur le nouveau serveur.
5. Indiquez le mot de passe du compte que vous utilisez actuellement pour vous connecter à Azure AD. Il doit s’agir du compte actuellement utilisé par DirSync.  
   ![Entrez vos informations d’identification Azure AD.](./media/how-to-dirsync-upgrade-get-started/ConnectToAzureAD.png)  
   Si vous recevez une erreur et que vous avez des problèmes de connectivité, consultez [Résoudre les problèmes de connectivité liés à Azure AD Connect](tshoot-connect-connectivity.md).
6. Indiquez un compte d'administrateur d'entreprise pour Active Directory.  
   ![Entrez vos informations d'identification ADDS.](./media/how-to-dirsync-upgrade-get-started/ConnectToADDS.png)
7. Vous êtes prêt à passer à la configuration. Lorsque vous cliquez sur **Mettre à niveau**, DirSync est désinstallé, puis Azure AD Connect est configuré et lance la synchronisation.  
   ![Prêt à configurer](./media/how-to-dirsync-upgrade-get-started/ReadyToConfigure.png)
8. Une fois l’installation terminée, déconnectez-vous puis reconnectez-vous à Windows avant d’utiliser le gestionnaire Synchronization Service Manager, l’éditeur de règles de synchronisation ou de tenter d’apporter d’autres modifications à la configuration.

## <a name="parallel-deployment"></a>Déploiement en parallèle
### <a name="export-the-dirsync-configuration"></a>Exportation de la configuration de DirSync
**Déploiement en parallèle avec plus de 50 000 objets**

Si vous avez plus de 50 000 objets, l’installation d’Azure AD Connect recommande un déploiement en parallèle.

Un écran semblable à celui-ci s’affiche :  
![Analyse terminée](./media/how-to-dirsync-upgrade-get-started/AnalysisRecommendParallel.png)

Pour effectuer un déploiement en parallèle, vous devez effectuer les opérations suivantes :

* Cliquez sur le bouton **Paramètres d'exportation** . Lorsque vous installez Azure AD Connect sur un autre serveur, ces paramètres sont migrés de votre installation DirSync vers votre nouvelle installation Azure AD Connect.

Une fois vos paramètres exportés, vous pouvez quitter l’Assistant Azure AD Connect sur le serveur de synchronisation d’annuaires. Passez à l’étape suivante pour installer Azure AD Connect sur un serveur distinct.

**Déploiement en parallèle avec moins de 50 000 objets**

Si vous avez moins de 50 000 objets mais que vous souhaitez effectuer un déploiement en parallèle, procédez comme suit :

1. Lancez le programme d’installation d’Azure AD Connect (MSI).
2. Lorsque l'écran **Bienvenue à Azure AD Connect** s'affiche, quittez l'Assistant d'installation en cliquant sur le X en haut à droite de la fenêtre.
3. Ouvrez une invite de commandes.
4. À partir de l’emplacement d’installation d’Azure AD Connect (par défaut : C:\Program Files\Microsoft Azure Active Directory Connect), exécutez la commande suivante : `AzureADConnect.exe /ForceExport`.
5. Cliquez sur le bouton **Paramètres d'exportation** . Lorsque vous installez Azure AD Connect sur un autre serveur, ces paramètres sont migrés de votre installation DirSync vers votre nouvelle installation Azure AD Connect.

![Capture d’écran montrant l’option Paramètres d’exportation pour la migration de vos paramètres vers la nouvelle installation d’Azure AD Connect.](./media/how-to-dirsync-upgrade-get-started/forceexport.png)

Une fois vos paramètres exportés, vous pouvez quitter l’Assistant Azure AD Connect sur le serveur de synchronisation d’annuaires. Passez à l’étape suivante pour installer Azure AD Connect sur un serveur distinct.

### <a name="install-azure-ad-connect-on-separate-server"></a>Installation d'Azure AD Connect sur un serveur distinct
Lorsque vous installez Azure AD Connect sur un nouveau serveur, le système présuppose que vous voulez effectuer une nouvelle installation d’Azure AD Connect. Comme vous souhaitez utiliser la configuration de DirSync, quelques étapes supplémentaires sont nécessaires :

1. Lancez le programme d’installation d’Azure AD Connect (MSI).
2. Lorsque l'écran **Bienvenue à Azure AD Connect** s'affiche, quittez l'Assistant d'installation en cliquant sur le X en haut à droite de la fenêtre.
3. Ouvrez une invite de commandes.
4. À partir de l’emplacement d’installation d’Azure AD Connect (par défaut : C:\Program Files\Microsoft Azure Active Directory Connect), exécutez la commande suivante : `AzureADConnect.exe /migrate`.
   L’Assistant d’installation d’Azure AD Connect démarre et affiche l’écran suivant :  
   ![Screenshot that shows where to import the settings file when you upgrade.](./media/how-to-dirsync-upgrade-get-started/ImportSettings.png)
5. Sélectionnez le fichier de paramètres exportés à partir de votre installation de la synchronisation d’annuaires.
6. Configurez les options avancées :
   * Un emplacement d'installation personnalisé pour Azure AD Connect.
   * Une instance de SQL Server (par défaut, Azure AD Connect installe SQL Server 2012 Express). N'utilisez pas la même instance de base de données que celle de votre serveur DirSync.
   * Un compte de service utilisé pour la connexion à SQL Server (si votre base de données SQL Server est distante, ce compte doit être un compte de service du domaine).
     Ces options s'affichent dans cet écran :  
     ![Screenshot that shows the advance configuration options for upgrading from DirSync.](./media/how-to-dirsync-upgrade-get-started/advancedsettings.png)
7. Cliquez sur **Suivant**.
8. Sur la page **Prêt pour la configuration**, laissez la case **Démarrer le processus de synchronisation dès que la configuration est terminée** cochée. Le serveur étant maintenant en [mode de préproduction](how-to-connect-sync-staging-server.md) , les modifications ne sont pas exportées vers Azure AD.
9. Cliquez sur **Installer**.
10. Une fois l’installation terminée, déconnectez-vous puis reconnectez-vous à Windows avant d’utiliser le gestionnaire Synchronization Service Manager, l’éditeur de règles de synchronisation ou de tenter d’apporter d’autres modifications à la configuration.

> [!NOTE]
> La synchronisation entre Windows Server Active Directory et Azure Active Directory commence, mais aucune modification n’est exportée vers Azure AD. Les modifications ne peuvent être exportées activement que par un seul outil de synchronisation à la fois. Cet état est appelé [mode de préproduction](how-to-connect-sync-staging-server.md).

### <a name="verify-that-azure-ad-connect-is-ready-to-begin-synchronization"></a>Vérifiez qu'Azure AD Connect est prêt à commencer la synchronisation
Pour vérifier qu’Azure AD Connect est prêt à remplacer DirSync, vous devrez ouvrir le **Synchronization Service Manager** dans le groupe **Azure AD Connect** à l’aide du menu Démarrer.

Dans l’application, cliquez sur l’onglet **Operations** . Dans cet onglet, vérifiez que les opérations suivantes ont abouti :

* Importation sur le connecteur Active Directory
* Importation sur le connecteur Azure Active Directory
* Synchronisation complète sur le connecteur Active Directory
* Synchronisation complète sur le connecteur Azure Active Directory

![Importation et synchronisation terminées](./media/how-to-dirsync-upgrade-get-started/importsynccompleted.png)

Examinez le résultat de ces opérations et vérifiez qu'il n'y a aucune erreur.

Si vous souhaitez afficher et examiner les modifications qui sont sur le point d’être exportées vers Azure AD, lisez comment vérifier la configuration en [mode de préproduction](how-to-connect-sync-staging-server.md). Apportez les modifications de la configuration requises jusqu'à ce que tout soit correct.

Vous êtes prêt à passer de DirSync à Azure AD lorsque vous avez effectué ces étapes et que vous êtes satisfait du résultat.

### <a name="uninstall-dirsync-old-server"></a>Désinstallation de DirSync (ancien serveur)
* Dans **Programmes et fonctionnalités**, recherchez **Outil de synchronisation Windows Azure Active Directory**
* Désinstallez **Outil de synchronisation Windows Azure Active Directory**
* La désinstallation peut prendre jusqu’à 15 minutes.

Si vous préférez désinstaller DirSync ultérieurement, vous pouvez également arrêter le serveur ou désactiver le service. Si une erreur survient, cette méthode vous permet de le réactiver. Toutefois, comme il est peu probable que l’étape suivante échoue, cette opération ne devrait pas être nécessaire.

Une fois DirSync désinstallé ou désactivé, aucun serveur actif n’effectue d’exportations vers Azure AD. L’étape suivante consistant à activer Azure AD Connect, doit aboutir pour que les modifications dans votre Active Directory local restent synchronisées avec Azure AD.

### <a name="enable-azure-ad-connect-new-server"></a>Activation d'Azure AD Connect (nouveau serveur)
Après l’installation, lorsque vous rouvrez Azure AD Connect, vous aurez la possibilité d’apporter des modifications supplémentaires à la configuration. Démarrez **Azure AD Connect** depuis le menu Démarrer ou à partir du raccourci sur le bureau. Assurez-vous que vous n'essayez pas d'exécuter à nouveau le programme MSI d'installation.

Les éléments suivants doivent s’afficher :  
![Tâches supplémentaires](./media/how-to-dirsync-upgrade-get-started/AdditionalTasks.png)

* Sélectionnez **Configuration du mode de préproduction**.
* Désactivez le mode de préproduction en désactivant la case à cocher **Mode de préproduction activé** .

![Capture d’écran montrant l’option permettant d’activer le mode de processus de site.](./media/how-to-dirsync-upgrade-get-started/configurestaging.png)

* Cliquez sur le bouton **Suivant** .
* Dans la page Confirmation, cliquez sur le bouton **Installer** .

Azure AD Connect est maintenant votre serveur actif et vous ne devez pas revenir vers votre serveur DirSync existant.

## <a name="next-steps"></a>Étapes suivantes
Azure AD Connect étant installé, vous pouvez passer à [Vérification de l’installation et affectation des licences](how-to-connect-post-installation.md).

Pour en savoir plus sur ces nouvelles fonctionnalités, activées lors de l’installation, consultez les pages : [Mise à niveau automatique](how-to-connect-install-automatic-upgrade.md), [Prévention des suppressions accidentelles](how-to-connect-sync-feature-prevent-accidental-deletes.md) et [Azure AD Connect Health](how-to-connect-health-sync.md).

Pour en savoir plus sur ces sujets courants, consultez l’article [Planificateur Azure AD Connect Sync](how-to-connect-sync-feature-scheduler.md).

En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
