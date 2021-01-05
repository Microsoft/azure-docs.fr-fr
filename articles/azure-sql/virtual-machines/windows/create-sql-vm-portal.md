---
title: Provisionner une machine virtuelle Windows avec le portail Azure
description: Ce guide présente les options dont vous disposez pour provisionner SQL Server sur une machine virtuelle Windows à l’aide du portail Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.assetid: 1aff691f-a40a-4de2-b6a0-def1384e086e
ms.service: virtual-machines-sql
ms.subservice: deployment
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: infrastructure-services
ms.date: 11/07/2019
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: 7914f924492264810acd929cc7d422e0cc780a89
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358790"
---
# <a name="how-to-use-the-azure-portal-to-provision-a-windows-virtual-machine-with-sql-server"></a>Comment utiliser le portail Azure pour provisionner une machine virtuelle Windows avec SQL Server

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Ce guide présente les options dont vous disposez pour provisionner SQL Server sur une machine virtuelle Windows à l’aide du portail Azure. Cet article aborde plus d’options de configuration que le [guide de démarrage rapide consacré aux machines virtuelles SQL Server](sql-vm-create-portal-quickstart.md), qui se concentre davantage sur une seule configuration. 

Utilisez ce guide pour créer votre propre machine virtuelle SQL Server. Ou bien, utilisez-le comme référence pour les options disponibles dans le portail Azure.

> [!TIP]
> Si vous avez des questions sur les machines virtuelles SQL Server, consultez le [Forum aux Questions](frequently-asked-questions-faq.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="sql-server-virtual-machine-gallery-images"></a><a id="select"></a> Images de la galerie des machines virtuelles SQL Server

Lorsque vous créez une machine virtuelle SQL Server, vous pouvez sélectionner une des images préconfigurées à partir de la galerie des machines virtuelles. Les étapes suivantes montrent comment sélectionner une des images SQL Server 2017.

1. Dans le menu de gauche du portail Azure, sélectionnez **Azure SQL**. Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis tapez *Azure SQL* dans la zone de recherche. 

   Vous pouvez également sélectionner l’étoile en regard d’**Azure SQL** pour l’enregistrer en tant que favori et l’ajouter en tant qu’élément dans le volet de navigation de gauche. 

1. Sélectionnez **+Ajouter** pour ouvrir la page **Sélectionner l’option de déploiement SQL**. Vous pouvez afficher des informations supplémentaires en sélectionnant **Afficher les détails**. 
1. Entrez *2017* dans la zone de recherche d’image SQL Server sur la vignette **machines virtuelles SQL**, puis sélectionnez **Licence SQL Server gratuite : SQL Server 2017 Developer sur Windows Server 2016** dans la liste déroulante. 

   ![Sélectionner une image de machine virtuelle SQL](./media/create-sql-vm-portal/select-sql-vm-image-portal.png)

   > [!TIP]
   > L’édition Developer est utilisée dans cet article, car il s’agit d’une version complète et gratuite de SQL Server pour les tests de développement. Vous payez uniquement pour le coût d’exécution de la machine virtuelle. Vous êtes libre, toutefois, de choisir les images à utiliser dans cette procédure pas à pas. Pour obtenir une description des images disponibles, consultez la [Vue d’ensemble des machines virtuelles Windows SQL Server](sql-server-on-azure-vm-iaas-what-is-overview.md#payasyougo).

   > [!TIP]
   > Les coûts de licence pour SQL Server sont intégrés dans le tarif à la seconde de la machine virtuelle que vous créez, et varie selon l’édition et les cœurs. Toutefois, l’édition SQL Server Développeur est gratuite pour le développement et le test, pas pour la production. En outre, SQL Express est gratuit pour les charges de travail légères (moins de 1 Go de mémoire, moins de 10 Go de stockage). Vous pouvez aussi utiliser votre solution BYOL (apportez votre propre licence) et payer seulement pour la machine virtuelle. Ces noms d’images sont préfixés avec {BYOL}. 
   >
   > Pour en savoir plus sur ces options, consultez l’article [Pricing guidance for SQL Server Azure VMs](pricing-guidance.md) (Tarification des machines virtuelles SQL Server Azure).


1. Sélectionnez **Create** (Créer).


## <a name="1-configure-basic-settings"></a>1. Configurer les paramètres de base

Sous l’onglet **De base**, fournissez les informations suivantes :

* Sous **Détails du projet**, vérifiez que l’abonnement approprié est sélectionné. 
* Dans la section **Groupe de ressources**, sélectionnez un groupe de ressources existant dans la liste déroulante ou choisissez **Créer** pour en créer un. Un groupe de ressources est une collection de ressources liées dans Azure (machines virtuelles, comptes de stockage, réseaux virtuels, etc.). 

  ![Abonnement](./media/create-sql-vm-portal/basics-project-details.png)

  > [!NOTE]
  > L’utilisation d’un nouveau groupe de ressources est utile si vous testez ou découvrez les déploiements SQL Server dans Azure. Une fois que vous avez terminé votre test, supprimez le groupe de ressources pour supprimer automatiquement la machine virtuelle et toutes les ressources associées à ce groupe de ressources. Pour plus d’informations sur les groupes de ressources, consultez [Vue d’ensemble d’Azure Resource Manager](../../../active-directory-b2c/overview.md).


* Sous **Détails de l’instance** :

    1. Entrez un **nom** unique pour la machine virtuelle.  
    1. Choisissez un emplacement pour votre **Région**. 
    1. Pour les besoins de ce guide, laissez **Options de disponibilité** défini sur _Aucune redondance d’infrastructure requise_. Pour plus d’informations sur les options de disponibilité, voir [Disponibilité](../../../virtual-machines/availability.md). 
    1. Dans la liste **Image**, sélectionnez _Licence SQL Server gratuite : SQL Server 2017 Developer sous Windows Server 2016_.  
    1. Choisissez **Modifier la taille** pour la **taille** de la machine virtuelle, puis sélectionnez l’offre **A2 de base**. Nettoyez vos ressources lorsque vous n’en avez plus besoin afin d’éviter des frais imprévus. Pour les charges de travail de production, consultez les tailles de machine et la configuration recommandées dans l’article [Meilleures pratiques relatives aux performances de SQL Server dans les machines virtuelles Azure](performance-guidelines-best-practices.md).

    ![Détails de l’instance](./media/create-sql-vm-portal/basics-instance-details.png)

> [!IMPORTANT]
> Le coût mensuel estimé affiché sur la fenêtre **Choisir une taille** n’inclut pas les coûts de licence SQL Server. Cette estimation correspond uniquement au coût de la machine virtuelle. Pour les éditions Express et Developer de SQL Server, cette estimation représente le coût total estimé. Pour les autres éditions, consultez la [page de tarification des machines virtuelles Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) et sélectionnez votre édition de SQL Server. Consultez également [Tarification des machines virtuelles SQL Server Azure](pricing-guidance.md) et [Tailles des machines virtuelles](../../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

* Sous **Compte Administrateur**, spécifiez un nom d’utilisateur et un mot de passe. Le mot de passe doit contenir au moins 12 caractères et satisfaire aux [exigences de complexité définies](../../../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).

   ![Compte d’administrateur](./media/create-sql-vm-portal/basics-administrator-account.png)

* Sous **Règles des ports d’entrée**, choisissez **Autoriser les ports sélectionnés**, puis sélectionnez **RDP (3389)** dans la liste déroulante. 

   ![Règles des ports d’entrée](./media/create-sql-vm-portal/basics-inbound-port-rules.png)


## <a name="2-configure-optional-features"></a>2. Configurer des fonctionnalités facultatives

### <a name="disks"></a>Disques

Sous l’onglet **Disques**, configurez les options de vos disques. 

* Sous **Type de disque du système d’exploitation**, sélectionnez dans la liste déroulante le type de disque que vous voulez pour votre système d’exploitation. Premium est recommandé pour les systèmes en production, mais n’est pas disponible pour une machine virtuelle De base. Pour utiliser un disque SSD Premium, changez la taille de la machine virtuelle. 
* Sous **Avancé**, sélectionnez **Oui** sous **Utiliser des disques managés**.

   > [!NOTE]
   > Microsoft recommande la fonctionnalité Disques managés pour SQL Server. La fonctionnalité Disques managés gère le stockage en arrière-plan. En outre, lorsque les machines virtuelles avec la fonctionnalité Disques managés sont dans le même groupe à haute disponibilité, Azure distribue les ressources de stockage pour fournir une redondance appropriée. Pour plus d’informations, consultez [Vue d’ensemble des disques managés Azure](../../../virtual-machines/managed-disks-overview.md). Pour plus de détails sur les disques managés dans un groupe à haute disponibilité, consultez l’article [Utilisation de disques managés pour les machines virtuelles dans le groupe à haute disponibilité](../../../virtual-machines/manage-availability.md).

![Paramètres des disques des machines virtuelles SQL](./media/create-sql-vm-portal/azure-sqlvm-disks.png)
  
  
### <a name="networking"></a>Mise en réseau

Sous l’onglet **Réseau**, configurez vos options pour le réseau. 

* Créez un **réseau virtuel** ou utilisez un réseau virtuel existant pour votre machine virtuelle SQL Server. Désignez aussi un **sous-réseau**. 

* Sous **Groupe de sécurité réseau de la carte réseau**, sélectionnez un groupe de sécurité de base ou le groupe de sécurité Avancé. Le fait de choisir l’option de base vous permet de sélectionner des ports d’entrée pour la machine virtuelle SQL Server qui sont les mêmes valeurs que celles configurées sous l’onglet **De base**. Sélectionner l’option Avancé vous permet de choisir un groupe de sécurité réseau existant ou d’en créer un. 

* Vous pouvez changer d’autres paramètres réseau ou conserver les valeurs par défaut.

![Paramètres réseau des machines virtuelles SQL](./media/create-sql-vm-portal/azure-sqlvm-networking.png)

#### <a name="monitoring"></a>Surveillance

Sous l’onglet **Supervision**, configurez la supervision et l’arrêt automatique. 

* Par défaut, Azure active **Diagnostics de démarrage** avec le compte de stockage désigné pour la machine virtuelle. Sous cet onglet, vous pouvez changer ces paramètres et activer **Diagnostic du système d’exploitation invité**. 
* Vous pouvez également activer sous cet onglet **Identité managée affectée par le système** et **Arrêt automatique**. 

![Paramètres d’administration des machines virtuelles SQL](./media/create-sql-vm-portal/azure-sqlvm-management.png)


## <a name="3-configure-sql-server-settings"></a>3. Configurer les paramètres de SQL Server

Sous l’onglet **Paramètres SQL Server**, configurez des paramètres spécifiques et des optimisations pour SQL Server. Vous pouvez configurer les paramètres suivants pour SQL Server :

- [Connectivité](#connectivity)
- [Authentification](#authentication)
- [Intégration d’Azure Key Vault](#azure-key-vault-integration)
- [Configuration du stockage](#storage-configuration)
- [Mise à jour corrective automatisée](#automated-patching)
- [Sauvegarde automatisée](#automated-backup)
- [Machine Learning Services](#machine-learning-services)


### <a name="connectivity"></a>Connectivité

Sous **SQL connectivity**(Connectivité SQL), spécifiez le type d’accès à l’instance SQL Server souhaité sur cette machine virtuelle. Pour les besoins de cette procédure pas à pas, sélectionnez **Public (Internet)** afin d’autoriser les connexions à SQL Server depuis des machines ou des services sur Internet. Avec cette option sélectionnée, Azure configure automatiquement le pare-feu et le groupe de sécurité réseau pour autoriser le trafic sur le port sélectionné.

> [!TIP]
> Par défaut, SQL Server écoute un port bien connu, le port **1433**. Pour accroître la sécurité, remplacez dans la boîte de dialogue précédente le port par défaut par un autre port, tel que 1401. Si vous changez le port, vous devez vous connecter par le biais de ce port à partir de tous les outils clients, tels que SQL Server Management Studio (SSMS).

![Sécurité des machines virtuelles SQL](./media/create-sql-vm-portal/azure-sqlvm-security.png)

Pour vous connecter à SQL Server via Internet, vous devez également activer l’authentification SQL Server décrite à la section suivante.

Si vous préférez ne pas activer les connexions au moteur de base de données via Internet, choisissez l’une des options suivantes :

* **Local (sur la machine virtuelle uniquement)** pour autoriser les connexions à SQL Server uniquement depuis la machine virtuelle.
* **Privé (dans un réseau virtuel)** pour autoriser les connexions à SQL Server depuis des machines ou des services résidant dans le même réseau virtuel.

En règle générale, améliorez la sécurité en choisissant la connectivité autorisée par votre scénario. Mais toutes les options sont sécurisables via les règles du groupe de sécurité réseau et l’authentification Windows/SQL. Vous pouvez modifier le groupe de sécurité réseau après la création de la machine virtuelle. Pour plus d’informations, consultez [Considérations relatives à la sécurité de SQL Server sur les machines virtuelles Azure](security-considerations-best-practices.md).

### <a name="authentication"></a>Authentification

Si vous exigez l’authentification SQL Server, sélectionnez **Activer** sous **Authentification SQL**, sous l’onglet **Paramètres SQL Server**.

![l’authentification SQL Server](./media/create-sql-vm-portal/azure-sqlvm-authentication.png)

> [!NOTE]
> Si vous envisagez d’accéder à SQL Server via Internet (l’option Connectivité publique), vous devez activer l’authentification SQL ici. L’accès public à SQL Server nécessite l’authentification SQL.

Si vous activez l’authentification SQL Server, spécifiez un **nom de connexion** et un **mot de passe**. Ce nom de connexion est configuré en tant que connexion d’authentification SQL Server et membre du rôle serveur fixe **sysadmin**. Pour plus d’informations sur les modes d’authentification, voir [Choisir un mode d’authentification](/sql/relational-databases/security/choose-an-authentication-mode).

Si vous préférez ne pas activer l’authentification SQL Server, vous pouvez utiliser le compte d’administrateur local sur la machine virtuelle pour vous connecter à l’instance SQL Server.

### <a name="azure-key-vault-integration"></a>Intégration du coffre de clés Azure

Pour stocker les secrets de sécurité dans Azure pour le chiffrement, sélectionnez **Paramètres SQL Server** et faites défiler jusqu’à **Intégration d’Azure Key Vault**. Sélectionnez **Activer** et renseignez les informations demandées. 

![Intégration du coffre de clés Azure](./media/create-sql-vm-portal/azure-sqlvm-akv.png)

Le tableau suivant liste les paramètres requis pour configurer l’intégration du coffre de clés Azure (Azure Key Vault).

| PARAMÈTRE | Description | EXEMPLE |
| --- | --- | --- |
| **URL du coffre de clés** |L’emplacement du coffre de clés. |`https://contosokeyvault.vault.azure.net/` |
| **Nom du principal** |Nom du principal du service Azure Active Directory Également appelé ID client. |`fde2b411-33d5-4e11-af04eb07b669ccf2` |
| **Secret du principal** |Secret du principal du service Azure Active Directory Également appelé Secret client. |`9VTJSQwzlFepD8XODnzy8n2V01Jd8dAjwm/azF1XDKM=` |
| **Nom des informations d’identification** |**Nom des informations d'identification** : L’intégration du coffre de clés Azure crée des informations d’identification dans SQL Server et permet à la machine virtuelle d’accéder au coffre de clés. Choisissez un nom pour cette identification. |`mycred1` |

Pour plus d’informations, consultez [Configurer l’intégration d’Azure Key Vault pour SQL Server sur des machines virtuelles Azure](azure-key-vault-integration-configure.md).

### <a name="storage-configuration"></a>Configuration du stockage

Dans l’onglet **Paramètres SQL Server**, sous **Configuration du stockage**, sélectionnez **Modifier la configuration** pour ouvrir la page Configuration du stockage à performances optimisées et spécifier les exigences de stockage.

![Capture d’écran faisant ressortir l’endroit où vous pouvez modifier la configuration du stockage.](./media/create-sql-vm-portal/sql-vm-storage-configuration-provisioning.png)

Sous **Stockage optimisé pour**, sélectionnez l’une des options suivantes :

* **Général** est le paramètre par défaut ; il prend en charge la plupart des charges de travail.
* **Traitement transactionnel** optimise le stockage pour les charges de travail OLTP de base de données traditionnelles.
* **entreposage de données** optimise le stockage pour les charges de travail d’analyse et de création de rapports.

![Configuration du stockage des machines virtuelles SQL](./media/create-sql-vm-portal/sql-vm-storage-configuration.png)

Vous pouvez conserver les valeurs par défaut ou modifier manuellement la topologie du stockage en fonction de vos besoins en E/S par seconde. Pour plus d'informations, consultez [Configuration du stockage](storage-configuration.md). 

### <a name="sql-server-license"></a>Licence SQL Server

Si vous êtes un client Software Assurance, vous pouvez utiliser [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) pour amener votre propre licence SQL Server et économiser sur les ressources. 

![Licence pour les machines virtuelles SQL](./media/create-sql-vm-portal/azure-sqlvm-license.png)

### <a name="automated-patching"></a>Mise à jour corrective automatisée

**Automated patching** est activée par défaut. La mise à jour corrective automatisée permet à Azure de corriger automatiquement SQL Server et le système d’exploitation. Spécifiez un jour de la semaine, une heure et une durée pour la fenêtre de maintenance. Azure effectue la mise à jour corrective dans cette fenêtre de maintenance. La planification de la fenêtre de maintenance utilise les paramètres régionaux de la machine virtuelle. Si vous ne souhaitez pas qu’Azure corrige automatiquement SQL Server et le système d’exploitation, sélectionnez **Désactiver**.  

![Mise à jour corrective automatique pour les machines virtuelles SQL](./media/create-sql-vm-portal/azure-sqlvm-automated-patching.png)

Pour plus d’informations, consultez [Mise à jour corrective automatisée pour SQL Server dans les machines virtuelles Azure](automated-patching.md).

### <a name="automated-backup"></a>Sauvegarde automatisée

Activez les sauvegardes automatiques de base de données pour toutes les bases de données sous **Sauvegarde automatisée**. La sauvegarde automatisée est désactivée par défaut.

Lorsque vous activez la sauvegarde SQL automatisée, vous pouvez configurer les paramètres suivants :

* Période de rétention (jours) pour les sauvegardes
* Compte de stockage à utiliser pour les sauvegardes
* Option de chiffrement et mot de passe pour les sauvegardes
* Sauvegarde des bases de données système
* Configuration de la planification de sauvegarde

Pour chiffrer la sauvegarde, sélectionnez **Activer**. Spécifiez ensuite le **mot de passe**. Azure crée un certificat pour chiffrer les sauvegardes et utilise le mot de passe pour protéger ce certificat. Par défaut, la planification est définie automatiquement, mais vous pouvez créer une planification manuelle en sélectionnant **Manuel**. 

![Sauvegardes automatisées des machines virtuelles SQL](./media/create-sql-vm-portal/automated-backup.png)

Pour plus d’informations, voir [Sauvegarde automatisée pour SQL Server dans les machines virtuelles Azure](automated-backup-sql-2014.md).


### <a name="machine-learning-services"></a>Machine Learning Services

Vous avez la possibilité d’activer [Services de Machine Learning](/sql/advanced-analytics/). Cette option vous permet d’utiliser le machine learning avec Python et R dans SQL Server 2017. Sélectionnez **Activer** dans la fenêtre **Paramètres SQL Server**.


## <a name="4-review--create"></a>4. Vérifier + créer

Sous l’onglet **Vérifier + créer** :
1. Passez en revue le résumé.
1. Sélectionnez **Créer** pour créer le server SQL Server, le groupe de ressources et les ressources spécifiées pour cette machine virtuelle.

Vous pouvez surveiller le déploiement à partir du portail Azure. Le bouton **Notifications** en haut de l’écran affiche l’état de base du déploiement.

> [!NOTE]
> Exemple de temps nécessaire à Azure pour déployer une machine virtuelle SQL Server : le déploiement d’une machine virtuelle SQL Server de test provisionnée dans la région USA Est avec des paramètres par défaut prend environ 12 minutes. Ces durées peuvent varier en fonction de votre région et des paramètres sélectionnés.

## <a name="open-the-vm-with-remote-desktop"></a><a id="remotedesktop"></a> Ouvrir la machine virtuelle à l’aide du Bureau à distance

Pour vous connecter à la machine virtuelle SQL Server à l’aide du protocole RDP (Remote Desktop Protocol), procédez comme suit :

[!INCLUDE [Connect to SQL Server VM with remote desktop](../../../../includes/virtual-machines-sql-server-remote-desktop-connect.md)]

Une fois que vous vous connectez à la machine virtuelle SQL Server, vous pouvez lancer SQL Server Management Studio et vous connecter avec l’authentification Windows à l’aide de vos informations d’identification d’administrateur local. Si vous avez activé l’authentification SQL Server, vous pouvez également vous connecter avec l’authentification SQL à l’aide de la connexion SQL et du mot de passe configuré lors de l’approvisionnement.

L’accès à la machine permet de modifier directement les paramètres SQL Server et ceux de la machine en fonction de vos besoins. Par exemple, vous pourriez configurer les paramètres du pare-feu ou modifier les paramètres de configuration SQL Server.

## <a name="connect-to-sql-server-remotely"></a><a id="connect"></a> Se connecter à SQL Server à distance

Dans cette procédure pas à pas, vous avez sélectionné l’accès **Public** pour la machine virtuelle et **l’authentification SQL Server**. Ces paramètres ont configuré automatiquement la machine virtuelle pour autoriser les connexions SQL Server à partir de n’importe quel client sur internet (en supposant qu’il dispose de la connexion SQL qui convient).

> [!NOTE]
> Si vous n’avez pas sélectionné Public lors de l’approvisionnement, vous pouvez modifier vos paramètres de connectivité SQL via le portail après l’approvisionnement. Pour plus d’informations, consultez la page [Modifier vos paramètres de connectivité SQL](ways-to-connect-to-sql.md#change).

Les sections suivantes montrent comment se connecter via Internet à votre instance de machine virtuelle SQL Server.

[!INCLUDE [Connect to SQL Server in a VM Resource Manager](../../../../includes/virtual-machines-sql-server-connection-steps-resource-manager.md)]

  > [!NOTE]
  > Cet exemple utilise le port courant 1433. Cependant, cette valeur doit être changée si un autre port (comme 1401) a été spécifié lors du déploiement de la machine virtuelle SQL Server. 


## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation de SQL Server dans Azure, consultez [Présentation de SQL Server sur les machines virtuelles](sql-server-on-azure-vm-iaas-what-is-overview.md) Azure et le [Forum Aux Questions](frequently-asked-questions-faq.md) associé.