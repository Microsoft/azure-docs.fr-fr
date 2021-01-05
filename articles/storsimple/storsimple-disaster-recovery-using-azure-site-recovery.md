---
title: Automatiser la récupération d’urgence de partages de fichiers StorSimple avec Azure Site Recovery | Microsoft Docs
description: Décrit les étapes et bonnes pratiques pour créer une solution de récupération d’urgence pour les partages de fichiers hébergés sur un stockage Microsoft Azure StorSimple.
services: storsimple
documentationcenter: NA
author: vidarmsft
manager: syadav
editor: ''
ms.assetid: 23049a2c-055e-4d0e-b8f5-af2a87ecf53f
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/13/2017
ms.author: alkohli
ms.openlocfilehash: e2d89718d953f05b3e5500db412ac8ac03bfa00b
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96301944"
---
# <a name="automated-disaster-recovery-solution-using-azure-site-recovery-for-file-shares-hosted-on-storsimple"></a>Solution de récupération d’urgence automatisée à l’aide d’Azure Site Recovery pour les partages de fichiers hébergés sur StorSimple

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="overview"></a>Vue d’ensemble
Microsoft Azure StorSimple est une solution de stockage de cloud hybride qui gère les complexités des données non structurées couramment associées aux partages de fichiers. StorSimple utilise le stockage cloud pour étendre la solution sur site et hiérarchise automatiquement les données sur le stockage local et le stockage cloud. La protection des données intégrée, qui comprend à la fois des instantanés en local et des instantanés sur le cloud, évite d’avoir à s’appuyer sur une immense infrastructure de stockage.

[Azure Site Recovery](../site-recovery/site-recovery-overview.md) est un service Azure offrant des capacités de récupération d’urgence (DR) en coordonnant la réplication, le basculement et la récupération des machines virtuelles. Azure Site Recovery prend en charge un certain nombre de technologies de réplication afin de systématiquement répliquer, protéger et basculer en toute transparence des machines virtuelles et des applications sur des clouds privés/publics ou de l’hébergeur.

Vous pouvez vous appuyer sur Azure Site Recovery, sur la réplication de machines virtuelles et sur les fonctionnalités de capture d’instantanés dans le cloud intégrées à StorSimple pour protéger l’ensemble de votre environnement de serveur de fichiers. En cas d’interruption, vous pouvez, d’un seul clic, afficher vos partages de fichiers en ligne dans Azure en seulement quelques minutes.

Ce document explique en détail comment créer une solution de récupération d’urgence pour vos partages de fichiers hébergés sur le système de stockage StorSimple, et comment effectuer des basculements planifiés/non planifiés/de test à l’aide d’un plan de récupération d’urgence accessible en un clic. En bref, il vous montre comment modifier le plan de récupération dans votre coffre Azure Site Recovery pour permettre des basculements StorSimple dans les scénarios d’urgence. Il décrit également les configurations prises en charge et les conditions requises préalables. Ce document suppose que vous soyez familiarisé avec les principes de base des architectures Azure Site Recovery et StorSimple.

## <a name="supported-azure-site-recovery-deployment-options"></a>Options de déploiement Azure Site Recovery prises en charge
Les clients peuvent déployer des serveurs de fichiers en tant que serveurs physiques ou en tant que machines virtuelles exécutés sur Hyper-V ou VMware, avant de créer des partages de fichiers à partir des volumes issus du stockage StorSimple. Azure Site Recovery peut protéger des déploiements physiques et virtuels aussi bien sur un site secondaire que sur Azure. Ce document décrit en détail une solution de récupération d’urgence qui utilise Azure comme site de récupération pour une machine virtuelle du serveur de fichiers hébergée sur Hyper-V et qui stocke des partages de fichiers sur StorSimple. Vous pouvez implémenter de la même façon d’autres scénarios impliquant un serveur de fichiers exécuté sur une machine virtuelle VMware ou sur un ordinateur physique.

## <a name="prerequisites"></a>Prérequis
Pour implémenter une solution de récupération d’urgence en un clic qui utilise Azure Site Recovery pour les partages de fichiers hébergés sur le stockage StorSimple, vous devez disposer des éléments suivants :

   - Machine virtuelle du serveur de fichiers Windows Server 2012 R2 en local hébergée sur une machine virtuelle Hyper-V ou VMware ou sur un ordinateur physique
   - Périphérique de stockage StorSimple local enregistré auprès d’Azure StorSimple Manager
   - StorSimple Cloud Appliance créé dans Azure StorSimple Manager. L’appliance peut être conservée dans un état d’arrêt.
   - Partages de fichiers hébergés sur les volumes configurés sur le périphérique de stockage StorSimple
   - [Coffre Azure Site Recovery Services](/azure/site-recovery/hyper-v-vmm-azure-tutorial) créé dans un abonnement Microsoft Azure.

En outre, si Azure est votre site de récupération d’urgence, exécutez l’ [outil d’évaluation de la disponibilité des machines virtuelles Azure](https://azure.microsoft.com/downloads/vm-readiness-assessment/) sur des machines virtuelles afin de vérifier qu’elles sont compatibles avec les machines virtuelles Azure et Azure Site Recovery Services.

Pour éviter des problèmes de latence (qui peuvent entraîner des coûts supplémentaires), veillez à créer votre appliance cloud StorSimple, votre compte Automation et votre compte de stockage dans la même région.

## <a name="enable-dr-for-storsimple-file-shares"></a>Activer la récupération d’urgence pour les partages de fichiers StorSimple
Chaque composant de l’environnement local doit être protégé pour permettre la prise en charge complète des tâches de restauration et de réplication. Cette section explique comment :
    
   - configurer la réplication Active Directory et DNS (facultatif) ;
   - utiliser Azure Site Recovery pour activer la protection de la machine virtuelle du serveur de fichiers ;
   - activer la protection des volumes StorSimple ;
   - configurer le réseau.

### <a name="set-up-active-directory-and-dns-replication-optional"></a>configurer la réplication Active Directory et DNS (facultatif) ;
Si vous voulez protéger les ordinateurs qui exécutent Active Directory et DNS afin qu’ils soient disponibles sur le site de récupération d’urgence, vous devez les protéger explicitement (afin que les serveurs de fichiers soient accessibles après un basculement avec authentification). En fonction de la complexité de l’environnement local du client, deux options vous sont recommandées.

#### <a name="option-1"></a>Option 1 :
Si le client dispose d’un nombre d’applications limité et d’un seul contrôleur de domaine pour l’ensemble de son site local, et qu’il souhaite basculer l’intégralité du site, nous recommandons l’utilisation de la réplication Azure Site Recovery pour répliquer l’ordinateur contrôleur de domaine sur un site secondaire (applicable à la fois pour les scénarios de site à site et de site à Azure).

#### <a name="option-2"></a>Option 2 :
Si le client possède un grand nombre d’applications, qu’il exécute une forêt Active Directory et qu’il souhaite basculer quelques applications à la fois, nous recommandons de configurer un contrôleur de domaine supplémentaire sur le site de récupération d’urgence (sur un site secondaire ou dans Azure).

Pour plus d’informations sur la mise à disposition d’un contrôleur de domaine sur le site de récupération d’urgence, consultez [Solution de récupération d’urgence pour Active Directory et DNS avec Azure Site Recovery](../site-recovery/site-recovery-active-directory.md). Dans la suite de ce document, nous supposons qu’un contrôleur de domaine est disponible sur le site de récupération d’urgence.

### <a name="use-azure-site-recovery-to-enable-protection-of-the-file-server-vm"></a>utiliser Azure Site Recovery pour activer la protection de la machine virtuelle du serveur de fichiers ;
Cette étape suppose de préparer l’environnement de serveur de fichiers local, de créer et préparer un coffre Azure Site Recovery, et d’activer la protection des fichiers de la machine virtuelle.

#### <a name="to-prepare-the-on-premises-file-server-environment"></a>Pour préparer l’environnement de serveur de fichiers local
1. Définissez l’option **Contrôle de compte d’utilisateur** sur le paramètre **Never Notify** (Ne jamais m’avertir). Vous devez effectuer ce paramétrage afin de pouvoir utiliser les scripts Azure Automation pour connecter les cibles iSCSI après un basculement par Azure Site Recovery.
   
   1. Appuyez sur la touche Windows + Q et lancez une recherche sur **UAC**.  
   1. Sélectionnez **Modifier les paramètres du contrôle de compte d’utilisateur**.  
   1. Faites glisser la barre vers le bas vers l’option **Never Notify**(Ne jamais m’avertir).  
   1. Cliquez sur **OK**, puis sélectionnez **Oui** à l’invite.  
   
      ![Paramètres du contrôle de compte d’utilisateur](./media/storsimple-disaster-recovery-using-azure-site-recovery/image1.png) 

1. Installez l’agent de machine virtuelle sur chacune des machines virtuelles du serveur de fichiers. Cela vous permettra d’exécuter les scripts Azure Automation sur les machines virtuelles qui ont fait l’objet d’un basculement.
   
   1. [Téléchargez l’agent](https://aka.ms/vmagentwin) dans le répertoire `C:\\Users\\<username>\\Downloads`.
   1. Ouvrez Windows PowerShell en mode administrateur (Exécuter en tant qu’administrateur) et entrez la commande suivante pour accéder à l’emplacement de téléchargement :  
         `cd C:\\Users\\<username>\\Downloads\\WindowsAzureVmAgent.2.6.1198.718.rd\_art\_stable.150415-1739.fre.msi`
         
         > [!NOTE]
         > Le nom de fichier peut varier en fonction de la version.
      
1. Cliquez sur **Suivant**.
1. Acceptez les **termes du contrat**, puis cliquez sur **Suivant**.
1. Cliquez sur **Terminer**.
1. Créez des partages de fichiers à l’aide de volumes issus du stockage StorSimple. Pour plus d’informations, consultez la page [Utiliser le service StorSimple Manager pour gérer les volumes](./index.yml).
   
   1. Sur vos machines virtuelles en local, appuyez sur la touche Windows + Q et recherchez **iSCSI**.
   1. Sélectionnez **Initiateur iSCSI**.
   1. Cliquez sur l’onglet **Configuration** et copiez le nom de l’initiateur.
   1. Connectez-vous au [portail Azure](https://portal.azure.com/).
   1. Cliquez sur l’onglet **StorSimple** , puis sélectionnez le service StorSimple Manager qui contient le périphérique physique.
   1. Créez un ou plusieurs conteneurs de volumes, puis créez un ou plusieurs volumes (ces volumes sont réservés aux partages de fichiers résidant sur les machines virtuelles du serveur de fichiers). Copiez le nom de l’initiateur et attribuez un nom approprié aux enregistrements de contrôle d’accès lorsque vous créez des volumes.
   1. Cliquez sur l’onglet **Configurer** et notez l’adresse IP de l’appareil.
   1. Sur vos machines virtuelles en local, accédez de nouveau à l’ **initiateur iSCSI** et entrez l’adresse IP dans la section Connexion rapide. Cliquez sur **Connexion rapide** (l’appareil doit maintenant être connecté).
   1. Ouvrez le portail Azure et cliquez sur l’onglet **Volumes et périphériques**. Cliquez sur **Configuration automatique**. Le volume que vous avez créé doit normalement apparaître.
   1. Dans le portail, cliquez sur l’onglet **Périphériques**, puis sélectionnez **Create a New Virtual Device.** (Créer un périphérique virtuel) (Créer un périphérique virtuel) (celui-ci sera utilisé si un basculement se produit). Ce nouveau périphérique virtuel peut être conservé à l’état hors connexion afin d’éviter des coûts supplémentaires. Pour mettre le périphérique virtuel hors connexion, accédez à la section **Machines virtuelles** du portail, puis arrêtez simplement le périphérique.
   1. Revenez à vos machines virtuelles locales et ouvrez Gestion des disques (appuyez sur la touche Windows + X et sélectionnez **Gestion des disques**).
   1. Vous remarquerez la présence de quelques disques supplémentaires (en fonction du nombre de volumes que vous avez créés). Cliquez avec le bouton droit sur le premier disque, sélectionnez **Initialiser le disque**, puis cliquez sur **OK**. Cliquez avec le bouton droit sur la section **Non alloué**, sélectionnez **Nouveau volume simple**, attribuez-lui une lettre de lecteur, puis terminez l’Assistant.
   1. Répétez l’étape l pour tous les disques. Tous les disques doivent maintenir figurer dans le répertoire **Ce PC** de l’Explorateur Windows.
   1. Utilisez le rôle Services de fichiers et de stockage pour créer des partages de fichiers sur ces volumes.

#### <a name="to-create-and-prepare-an-azure-site-recovery-vault"></a>Pour créer et préparer un coffre Azure Site Recovery
Reportez-vous à la [documentation d’Azure Site Recovery](/azure/site-recovery/) pour vous familiariser avec Azure Site Recovery avant de protéger la machine virtuelle du serveur de fichiers.

#### <a name="to-enable-protection"></a>Pour activer la protection
1. Déconnectez la ou les cibles iSCSI des machines virtuelles en local que vous souhaitez protéger dans Azure Site Recovery :
   
   1. Appuyez sur la touche Windows + Q et lancez une recherche sur **iSCSI**.
   1. Sélectionnez **Configurer l’initiateur iSCSI**.
   1. Déconnectez l’appareil StorSimple que vous avez connecté auparavant. Vous pouvez également désactiver le serveur de fichiers pendant quelques minutes au moment de l’activation de la protection.
      
   > [!NOTE]
   > Les partages de fichiers seront alors temporairement indisponibles.
   
1. [Activez la protection des machines virtuelles](/azure/site-recovery/hyper-v-azure-tutorial) du serveur de fichiers à partir du portail Azure Site Recovery.
1. Vous pouvez reconnecter la cible dès le lancement de la synchronisation initiale. Accédez à l’initiateur iSCSI, sélectionnez l’appareil StorSimple, puis cliquez sur **Connecter**.
1. Lorsque la synchronisation est terminée et que la machine virtuelle passe à l’état **Protégé**, sélectionnez la machine virtuelle, puis cliquez sur l’onglet **Configurer** et mettez à jour le réseau de la machine virtuelle en conséquence (il s’agit du réseau auquel sont rattachées les machines virtuelles ayant fait l’objet du basculement). Si le réseau ne s’affiche pas, cela signifie que la synchronisation est en cours.

### <a name="enable-protection-of-storsimple-volumes"></a>activer la protection des volumes StorSimple ;
Si vous n’avez pas sélectionné l’option **Autoriser une sauvegarde par défaut pour ce volume** pour les volumes StorSimple, accédez à **Stratégies de sauvegarde** dans le service StorSimple Manager et créez une stratégie de sauvegarde appropriée pour l’ensemble des volumes. Nous vous recommandons de faire coïncider la fréquence des sauvegardes avec l’objectif de point de récupération (RPO) que vous souhaitez obtenir pour l’application.

### <a name="configure-the-network"></a>configurer le réseau.
Pour la machine virtuelle du serveur de fichiers, configurez les paramètres réseau dans Azure Site Recovery afin que les réseaux de machines virtuelles soient attachés au réseau de récupération d’urgence approprié au terme du processus de basculement.

Vous pouvez sélectionner la machine virtuelle sous l’onglet **Éléments répliqués** afin de configurer les paramètres réseau comme indiqué dans l’image ci-dessous.

![Calcul et réseau](./media/storsimple-disaster-recovery-using-azure-site-recovery/image2.png)

## <a name="create-a-recovery-plan"></a>Créer un plan de récupération
Vous pouvez créer un plan de récupération dans ASR pour automatiser le processus de basculement des partages de fichiers. En cas d’interruption, vous pouvez, d’un seul clic, faire apparaître les partages de fichiers en quelques minutes. Pour activer cette automatisation, vous devez disposer d’un compte Azure Automation.

#### <a name="to-create-an-automation-account"></a>Pour créer un compte Automation
1. Accédez au Portail Azure &gt; section **Automation**.
1. Cliquez sur le bouton **+ Ajouter** pour ouvrir le panneau ci-dessous.
   
   ![Ajouter un compte Automation](./media/storsimple-disaster-recovery-using-azure-site-recovery/image11.png)
   
   - Nom : entrez le nom d’un nouveau compte Automation
   - Abonnement : choisissez l’abonnement
   - Groupe de ressources : sélectionnez un groupe de ressources existant ou créez-en un
   - Emplacement : utilisez la même région géographique que celle où ont été créés StorSimple Cloud Appliance et les comptes de stockage.
   - Créer un compte d’identification Azure : sélectionnez l’option **Oui**.
   
1. Accédez au compte Automation, puis cliquez sur **Runbooks** &gt; **Parcourir la galerie** pour importer tous les runbooks nécessaires dans le compte Automation.
1. Ajoutez les Runbooks suivants en recherchant l’onglet **Récupération d’urgence** dans la galerie :
   
   - Clean up of StorSimple volumes after Test Failover (TFO) (Nettoyer les volumes StorSimple après le test de basculement (TFO))
   - Fail over StorSimple volume containers (Basculer des conteneurs de volumes StorSimple)
   - Mount volumes on StorSimple device after failover (Monter les volumes sur l’appareil StorSimple après le basculement)
   - Uninstall custom script extension in Azure VM (Désinstaller l’extension de script personnalisée dans la machine virtuelle Azure)
   - Start StorSimple Virtual Appliance (Démarrer l’appliance virtuelle StorSimple)
   
      ![Parcourir la galerie](./media/storsimple-disaster-recovery-using-azure-site-recovery/image3.png)
   
1. Publiez tous les scripts en sélectionnant le runbook dans le compte Automation et cliquez sur **Modifier** &gt; **Publier**, puis sur **Oui** dans le message de vérification. Après cette étape, l’onglet **Runbooks** s’affiche comme suit :
   
   ![Runbooks](./media/storsimple-disaster-recovery-using-azure-site-recovery/image4.png)
   
1. Dans le compte Automation, cliquez sur **Variable** &gt; **Ajouter une variable**, puis ajoutez les variables suivantes. Vous pouvez choisir de chiffrer ces ressources. Ces variables sont spécifiques au plan de récupération. Si votre plan de récupération (celui que vous allez créer à l’étape suivante) porte le nom TestPlan, vos variables doivent être TestPlan-StorSimRegKey, TestPlan-AzureSubscriptionName, et ainsi de suite.

   - **BaseUrl** : URL Resource Manager pour le cloud Azure. Utilisez l’applet de commande **Get-AzEnvironment | Nom objet sélectionné, ResourceManagerUrl**.
   - _RecoveryPlanName_ **-ResourceGroupName** : Groupe Resource Manager qui a la ressource StorSimple.
   - _RecoveryPlanName_ **-ManagerName** : Ressource StorSimple qui a l’appareil StorSimple.
   - _RecoveryPlanName_ **-DeviceName** : Appareil StorSimple qui doit être basculé.
   - _RecoveryPlanName_ **-DeviceIpAddress** : Adresse IP de l’appareil (qui se trouve dans l’onglet **Appareils** de la section StorSimple Device Manager &gt; **Paramètres** &gt; **Réseau** &gt; groupe **Paramètres DNS**).
   - _RecoveryPlanName_ **-VolumeContainers** : Chaîne séparée par des virgules des conteneurs de volumes présents sur l’appareil qui doit faire l’objet d’un basculement ; par exemple, volcon1, volcon2, volcon3.
   - _RecoveryPlanName_ **-TargetDeviceName** : Appliance cloud StorSimple sur laquelle les conteneurs doivent être basculés.
   - _RecoveryPlanName_ **-TargetDeviceIpAddress** : Adresse IP de l’appareil cible (qui se trouve dans la section **Machine virtuelle** &gt; groupe **Paramètres** &gt; onglet **Réseaux**).
   - _RecoveryPlanName_ **-StorageAccountName** : Nom du compte de stockage dans lequel sera stocké le script (qui doit s’exécuter sur la machine virtuelle basculée). Il peut s’agir de n’importe quel compte de stockage disposant d’un minimum d’espace pour stocker temporairement le script.
   - _RecoveryPlanName_ **-StorageAccountKey** : Clé d’accès du compte de stockage ci-dessus.
   - _RecoveryPlanName_ **-VMGUIDS** : Lors de la protection d’une machine virtuelle, Azure Site Recovery affecte à chaque machine virtuelle un ID unique qui fournit des détails sur la machine virtuelle basculée. Pour obtenir le VMGUID, sélectionnez l’onglet **Recovery Services**, puis cliquez sur **Élément protégé** &gt; **Groupes de protection** &gt; **Machines** &gt; **Propriétés**. Si vous disposez de plusieurs machines virtuelles, ajoutez les GUID sous forme de chaîne séparée par des virgules.

     Par exemple, si le nom du plan de récupération est fileServerpredayRP, votre onglet **Variables**, **Connexions** et **Certificats** doit se présenter comme suit une fois que vous avez ajouté toutes les ressources.

      ![Éléments multimédias](./media/storsimple-disaster-recovery-using-azure-site-recovery/image5.png)

1. Chargez le module Runbook de la série StorSimple 8000 dans votre compte Automation. Procédez comme suit pour ajouter un module :
   
   1. Ouvrez PowerShell, créez un dossier et accédez à ce nouveau dossier.
      
      ```
            mkdir C:\scripts\StorSimpleSDKTools
            cd C:\scripts\StorSimpleSDKTools
      ```
   1. Téléchargez l’interface CLI NuGet sous le même dossier qu’a l’étape 1.
      Différentes versions de nuget.exe sont disponibles sur les [téléchargements NuGet](https://www.nuget.org/downloads). Chaque lien de téléchargement pointe directement vers un fichier .exe : veillez donc à cliquer avec le bouton droit et à enregistrer le fichier sur votre ordinateur, au lieu de l’exécuter à partir du navigateur.
      
      ```
            wget https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -Out C:\scripts\StorSimpleSDKTools\nuget.exe
      ```
      
   1. Télécharger le SDK dépendant
      
      ```
            C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Azure.Management.Storsimple8000series
            C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.28.3
            C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Rest.ClientRuntime.Azure.Authentication -Version 2.2.9-preview
      ```
      
   1. Créez un module Runbook Azure Automation pour la gestion des appareils StorSimple série 8000. Utilisez les commandes ci-dessous pour créer un fichier zip de module Automation.
         
      ```powershell
            # set path variables
            $downloadDir = "C:\scripts\StorSimpleSDKTools"
            $moduleDir = "$downloadDir\AutomationModule\Microsoft.Azure.Management.StorSimple8000Series"

            #don't change the folder name "Microsoft.Azure.Management.StorSimple8000Series"
            mkdir "$moduleDir"

            copy "$downloadDir\Microsoft.IdentityModel.Clients.ActiveDirectory.2.28.3\lib\net45\Microsoft.IdentityModel.Clients.ActiveDirectory*.dll" $moduleDir
            copy "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.3.3.7\lib\net452\Microsoft.Rest.ClientRuntime.Azure*.dll" $moduleDir
            copy "$downloadDir\Microsoft.Rest.ClientRuntime.2.3.8\lib\net452\Microsoft.Rest.ClientRuntime*.dll" $moduleDir
            copy "$downloadDir\Newtonsoft.Json.6.0.8\lib\net45\Newtonsoft.Json*.dll" $moduleDir
            copy "$downloadDir\Microsoft.Rest.ClientRuntime.Azure.Authentication.2.2.9-preview\lib\net45\Microsoft.Rest.ClientRuntime.Azure.Authentication*.dll" $moduleDir
            copy "$downloadDir\Microsoft.Azure.Management.Storsimple8000series.1.0.0\lib\net452\Microsoft.Azure.Management.Storsimple8000series*.dll" $moduleDir

            #Don't change the name of the Archive
            compress-Archive -Path "$moduleDir" -DestinationPath Microsoft.Azure.Management.StorSimple8000Series.zip
      ```
         
   1. Importez le fichier zip de module Azure Automation (Microsoft.Azure.Management.StorSimple8000Series.zip) créé à l’étape ci-dessus. Ceci peut être fait en sélectionnant le compte Automation, en cliquant sur **Modules** sous RESSOURCES PARTAGÉES, puis en cliquant sur **Ajouter un module**.
   
   Après avoir importé le module de la série StorSimple 8000, l’onglet **Modules** doit apparaître comme ceci :
   
      ![Modules](./media/storsimple-disaster-recovery-using-azure-site-recovery/image12.png)

1. Accédez à la section **Services de récupération** et sélectionnez le coffre Azure Site Recovery que vous avez créé précédemment.
1. Sélectionnez l’option **Plans de récupération (Site Recovery)** à partir du groupe **Gérer** et créez un plan de récupération comme suit :
   
   - Cliquez sur le bouton **+ Plan de récupération**, qui affiche le panneau ci-dessous.
      
      ![Créer un plan de récupération](./media/storsimple-disaster-recovery-using-azure-site-recovery/image6.png)
      
   - Entrez un nom de plan de récupération, choisissez les valeurs du modèle Source, Cible et Déploiement.
   
   - Sélectionnez les machines virtuelles du groupe de protection que vous souhaitez inclure dans le plan de récupération et cliquez sur le bouton **OK**.
   
   - Sélectionnez le plan de récupération que vous avez créé précédemment, cliquez sur le bouton **Personnaliser** pour ouvrir la vue de personnalisation du plan de récupération.
   
   - Cliquez avec le bouton droit sur **Arrêt de tous les groupes** et cliquez sur **Add pre action (Ajouter action antérieure)** .
   
   - Ouvrez le panneau Insérer une action, entrez un nom, sélectionnez l’option **Primary side (Côté principal)** dans l’option Where to run (Où exécuter), sélectionnez le compte Automation (dans lequel vous avez ajouté les Runbooks), puis sélectionnez le Runbook **Failover-StorSimple-Volume-Containers**.
   
   - Cliquez avec le bouton droit sur **Groupe 1 : Démarrer**, cliquez sur l’option **Add protected items (Ajouter des éléments protégés)** , puis sélectionnez les machines virtuelles qui doivent être protégées dans le plan de récupération et cliquez sur le bouton **Ok**. Facultatif, s’il s’agit déjà des machines virtuelles sélectionnées.
   
   - Cliquez avec le bouton droit sur **Groupe 1 : Démarrer** et cliquez sur l’option **Post action (Action postérieure)** , puis ajoutez tous les scripts suivants :  
      
      - Start-StorSimple-Virtual-Appliance runbook  
      - Fail over-StorSimple-volume-containers runbook  
      - Mount-volumes-after-failover runbook  
      - Uninstall-custom-script-extension runbook  
        
   - Ajoutez une action manuelle après les 4 scripts ci-dessus dans la même section **Groupe 1 : étapes suivantes**. Cette action marque l’étape à laquelle vous pouvez vérifier que tout fonctionne correctement. Cette action doit être ajoutée uniquement dans le cadre du test de basculement (vous ne devez donc cocher que la case **Test de basculement**).
    
   - Après l’action manuelle, ajoutez le script **Nettoyage** à l’aide de la même procédure que celle utilisée pour les autres Runbooks. **Enregistrez** le plan de récupération.
    
   > [!NOTE]
   > Lorsque vous exécutez un test de basculement, vous devez vérifier tous les éléments à l’étape de l’action manuelle, car les volumes StorSimple qui avaient été clonés sur le périphérique cible seront supprimés dans le cadre du nettoyage après l’exécution de l’action manuelle.
       
      ![Plan de récupération](./media/storsimple-disaster-recovery-using-azure-site-recovery/image7.png)

## <a name="perform-a-test-failover"></a>Exécution d’un test de basculement
Reportez-vous au guide d’accompagnement [Solution de récupération d’urgence d’Active Directory](../site-recovery/site-recovery-active-directory.md) pour obtenir des informations propres à Active Directory pendant le test de basculement. L’installation en local n’est pas perturbée au cours du test de basculement. Les volumes StorSimple qui étaient attachées à la machine virtuelle locale sont clonés sur l’appliance cloud StorSimple dans Azure. Une machine virtuelle de test s’affiche dans Azure et les volumes clonés sont attachés à la machine virtuelle.

#### <a name="to-perform-the-test-failover"></a>Pour effectuer le test de basculement
1. Dans le portail Azure, sélectionnez votre coffre Site Recovery.
1. Cliquez sur le plan de récupération que vous avez créé pour la machine virtuelle du serveur de fichiers.
1. Cliquez sur **Test de basculement**.
1. Sélectionnez le réseau virtuel Azure auquel les machines virtuelles Azure seront connectées après le basculement.
   
   ![Démarrer le basculement](./media/storsimple-disaster-recovery-using-azure-site-recovery/image8.png)
   
1. Cliquez sur **OK** pour commencer le basculement. Si vous souhaitez suivre la progression, cliquez sur la machine virtuelle pour ouvrir ses propriétés, ou sur le travail **Test de basculement** dans le nom du coffre &gt; **Travaux** &gt; **Travaux Site Recovery**.
1. Une fois le basculement terminé, vous devriez également voir la machine Azure de réplication apparaître sur le Portail Azure &gt; **Machines virtuelles**. Vous pouvez effectuer vos validations.
1. Une fois les validations terminées, cliquez sur **Validations Complete**(Validations terminées). Cette opération supprime les volumes StorSimple et arrête le StorSimple Cloud Appliance.
1. Une fois que vous avez terminé, cliquez sur **Nettoyer le test de basculement de nettoyage** sur le plan de récupération. Cliquez sur Notes pour consigner et enregistrer d’éventuelles observations associées au test de basculement. Cette opération supprimera les machines virtuelles qui ont été créées au cours du test de basculement.

## <a name="perform-a-planned-failover"></a>Exécuter un basculement planifié
   Lors d’un basculement planifié, la machine virtuelle locale du serveur de fichiers s’arrête normalement et un instantané des volumes de l’appareil StorSimple est sauvegardé dans le cloud. Les volumes StorSimple sont basculés vers l’appareil virtuel, une machine virtuelle de réplication s’affiche sur Azure et les volumes sont associés à la machine virtuelle.

#### <a name="to-perform-a-planned-failover"></a>Pour exécuter un basculement planifié
1. Sur le Portail Azure, sélectionnez le coffre **Recovery Services** &gt; **Plans de récupération (Site Recovery)** &gt; **nom_planrécupération** créé pour la machine virtuelle du serveur de fichiers.
1. Dans le panneau Plan de récupération, cliquez sur **Plus** &gt; **Basculement planifié**.

   ![Capture d’écran mettant en évidence les options Basculement planifié et Basculement.](./media/storsimple-disaster-recovery-using-azure-site-recovery/image9.png)
1. Sur le panneau **Confirmer le basculement planifié**, choisissez les emplacements source et cible, sélectionnez la réseau cible, puis cliquez sur l’icône de coche ✓ pour démarrer le processus de basculement.
1. Une fois créés, les ordinateurs virtuels de réplication sont en attente de validation. Cliquez sur **Valider** pour valider le basculement.
1. Une fois la réplication terminée, les machines virtuelles démarrent à l’emplacement secondaire.

## <a name="perform-a-failover"></a>Effectuer un basculement
Lors d’un basculement non planifié, les volumes StorSimple sont basculés vers l’appareil virtuel, une machine virtuelle de réplication s’affiche sur Azure et les volumes sont associés à la machine virtuelle.

#### <a name="to-perform-a-failover"></a>Pour effectuer un basculement
1. Sur le Portail Azure, sélectionnez le coffre **Recovery Services** &gt; **Plans de récupération (Site Recovery)** &gt; **nom_planrécupération** créé pour la machine virtuelle du serveur de fichiers.
1. Dans le panneau Plan de récupération, cliquez sur **Plus** &gt; **Basculement**.
1. Dans le panneau **Confirmer le basculement**, choisissez les emplacements source et cible.
1. Sélectionnez **Arrêter les machines virtuelles et synchroniser les dernières données** pour indiquer que le logiciel Site Recovery doit essayer d’arrêter la machine virtuelle protégée et de synchroniser les données, afin que la dernière version de ces dernières fasse l’objet d’un basculement.
1. Après le basculement, les machines virtuelles présentent un état de validation en attente. Cliquez sur **Valider** pour valider le basculement.


## <a name="perform-a-failback"></a>Effectuer une restauration automatique
Pendant une restauration automatique, les conteneurs de volumes StorSimple basculent vers l’appareil physique après l’exécution d’une sauvegarde.

#### <a name="to-perform-a-failback"></a>Pour effectuer une restauration automatique
1. Sur le Portail Azure, sélectionnez le coffre **Recovery Services** &gt; **Plans de récupération (Site Recovery)** &gt; **nom_planrécupération** créé pour la machine virtuelle du serveur de fichiers.
1. Dans le panneau Plan de récupération, cliquez sur **Plus** &gt; **Basculement planifié**.
1. Choisissez les emplacements source et cible et sélectionnez les options de synchronisation de données et de création de machines virtuelles appropriées.
1. Cliquez sur le bouton **OK** pour démarrer le processus de basculement.
   
   ![Démarrer la restauration automatique](./media/storsimple-disaster-recovery-using-azure-site-recovery/image10.png)

## <a name="best-practices"></a>Bonnes pratiques
### <a name="capacity-planning-and-readiness-assessment"></a>Planification de la capacité et évaluation de la préparation
#### <a name="hyper-v-site"></a>Site Hyper-V
Utilisez l’ [outil de planification de capacité utilisateur](https://www.microsoft.com/download/details.aspx?id=39057) pour concevoir l’infrastructure de serveur, de stockage et de réseau adaptée à votre environnement de réplication Hyper-V.

#### <a name="azure"></a>Azure
Vous pouvez exécuter l’ [outil d’évaluation de la disponibilité des machines virtuelles Azure](https://azure.microsoft.com/downloads/vm-readiness-assessment/) sur des machines virtuelles, afin de vérifier qu’elles sont compatibles avec les machines virtuelles Azure et Azure Site Recovery Services. Cet outil vérifie les configurations de machines virtuelles et vous avertit lorsque des configurations ne sont pas compatibles avec Azure. Par exemple, il émet un avertissement dans le cas d’un lecteur C: supérieur à 127 Go.

La planification de la capacité comporte au moins deux processus importants :

   - Mappage des machines virtuelles Hyper-V locales sur les tailles de machine virtuelle Azure (par exemple, A6, A7, A8 et A9).
   - Détermination de la bande passante Internet requise.

## <a name="limitations"></a>Limites
- Il n’est actuellement possible de basculer qu’un seul périphérique StorSimple (sur une seule appliance cloud StorSimple). Le scénario d’un serveur de fichiers qui s’étend sur plusieurs périphériques StorSimple n’est pas encore pris en charge.
- Si vous obtenez une erreur lors de l’activation de la protection d’une machine virtuelle, assurez-vous que vous avez déconnecté les cibles iSCSI.
- Tous les conteneurs de volumes qui ont été regroupés en raison des stratégies de sauvegarde communes aux différents conteneurs de volumes basculeront ensemble.
- Tous les volumes des conteneurs de volumes que vous avez choisis seront basculés.
- Les volumes qui peuvent cumuler une capacité supérieure à 64 To ne peuvent pas être basculés car la capacité maximale d’une appliance cloud StorSimple est de 64 To.
- Si le processus de basculement planifié ou non planifié échoue et que les machines virtuelles sont créées dans Azure, évitez de nettoyer les machines virtuelles. Effectuez plutôt une restauration automatique. Si vous supprimez les machines virtuelles, vous ne pourrez plus réactiver les machines virtuelles locales.
- Après un basculement, si vous n’êtes pas en mesure de voir les volumes, accédez aux machines virtuelles, ouvrez Gestion des disques, réanalysez les disques, puis mettez-les en ligne.
- Dans certains cas, les lettres de lecteur du site de récupération d’urgence peuvent être différentes de celles disponibles en local. Si cela se produit, vous devez corriger le problème manuellement après le basculement.
- Délai d’attente de la tâche de basculement : Le script StorSimple expire si le temps de basculement des conteneurs de volumes dépasse la limite par script d’Azure Site Recovery (actuellement établie à 120 minutes).
- Délai d’attente de la tâche de sauvegarde : Le script StorSimple expire si le temps de sauvegarde des volumes dépasse la limite par script d’Azure Site Recovery (actuellement établie à 120 minutes).
   
  > [!IMPORTANT]
  > Exécutez manuellement la sauvegarde à partir du portail Azure et réexécutez le plan de récupération.
   
- Délai d’attente de la tâche de clonage : Le script StorSimple expire si le temps de clonage des volumes dépasse la limite par script d’Azure Site Recovery (actuellement établie à 120 minutes).
- Erreur de synchronisation de temps : Les scripts StorSimple génèrent une erreur indiquant que les sauvegardes ont échoué, même si la sauvegarde a réussi dans le portail. Ce problème peut être lié à un défaut de synchronisation de l’heure de l’appliance StorSimple avec l’heure actuelle du fuseau horaire.
   
  > [!IMPORTANT]
  > Synchronisez l’heure de l’appliance avec l’heure actuelle du fuseau horaire.
   
- Erreur de basculement de l’appliance : Le script StorSimple peut échouer si l’appliance est en cours de basculement pendant l’exécution du plan de récupération.
   
  > [!IMPORTANT]
  > Exécutez à nouveau le plan de récupération une fois le basculement de l’appliance terminé.


## <a name="summary"></a>Résumé
Utilisez Azure Site Recovery pour créer un plan de récupération d’urgence automatisée complet pour une machine virtuelle du serveur de fichiers comportant des partages de fichiers sur le stockage StorSimple. Vous pouvez lancer le basculement en quelques secondes, où que vous soyez, et bénéficier d’une application opérationnelle en quelques minutes.