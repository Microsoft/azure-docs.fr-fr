---
title: Forum Aux Questions (FAQ) concernant SQL Server sur les machines virtuelles Windows dans Azure | Microsoft Docs
description: Cet article fournit des réponses aux questions fréquemment posées sur l'exécution de SQL Server sur les machines virtuelles Azure.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.service: virtual-machines-sql
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/05/2019
ms.author: mathoma
ms.openlocfilehash: b58119ccc1551d12dfc9b09f76f6980618ba6221
ms.sourcegitcommit: dc342bef86e822358efe2d363958f6075bcfc22a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94556293"
---
# <a name="frequently-asked-questions-for-sql-server-on-azure-vms"></a>Forum aux questions concernant SQL Server sur des machines virtuelles Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [Windows](frequently-asked-questions-faq.md)
> * [Linux](../linux/frequently-asked-questions-faq.md)

Cet article fournit des réponses à certaines des questions les plus courantes sur l’exécution de [SQL Server sur les machines virtuelles Microsoft Azure](https://azure.microsoft.com/services/virtual-machines/sql-server/).

[!INCLUDE [support-disclaimer](../../../../includes/support-disclaimer.md)]

## <a name="images"></a><a id="images"></a> Images

1. **Quelles images de galerie de machines virtuelles SQL Server sont disponibles ?** 

   Azure gère les images de machines virtuelles pour toutes les versions majeures prises en charge de SQL Server sur toutes les éditions pour Windows et Linux. Pour plus d’informations, consultez la liste complète des [images de machines virtuelles Windows](sql-server-on-azure-vm-iaas-what-is-overview.md#payasyougo) et des [images de machines virtuelles Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md#create).

1. **Les images de galerie de machines virtuelles SQL Server sont-elles mises à jour?**

   Tous les deux mois, les images SQL Server dans la galerie de machines virtuelles sont mises à jour avec les dernières mises à jour Windows et Linux. Pour les images Windows, cela inclut toutes les mises à jour marquées comme importantes dans Windows Update, dont les Service Packs et mises à jour de sécurité SQL Server essentiels. Pour les images Linux, cela inclut les dernières mises à jour système. Les mises à jour cumulatives SQL Server sont gérées différemment pour Linux et Windows. Pour Linux, les mises à jour cumulatives SQL Server figurent également dans l’actualisation. Toutefois, à ce stade, les machines virtuelles Windows ne sont pas mises à jour avec les mises à jour cumulatives SQL Server ou Windows Server.

1. **Est-ce que des images de machines virtuelles SQL Server peuvent être supprimées de la galerie ?**

   Oui. Azure gère uniquement une image par version majeure et édition. Par exemple, quand un nouveau Service Pack SQL Server est publié, Azure ajoute une nouvelle image à la galerie pour ce Service Pack. L’image SQL Server pour le Service Pack précédent est immédiatement supprimée du portail Azure. Toutefois, elle est toujours disponible pour le provisionnement à partir de PowerShell pour les trois prochains mois. Après trois mois, l’image du Service Pack précédent n’est plus disponible. Cette stratégie de suppression s’applique également si une version SQL Server n’est plus prise en charge quand elle atteint la fin de son cycle de vie.


1. **Est-il possible de déployer une image plus ancienne de SQL Server qui ne se trouve pas sur le portail Azure ?**

   Oui, en utilisant PowerShell. Pour plus d’informations sur le déploiement de machines virtuelles SQL Server à l’aide de PowerShell, consultez [Guide pratique pour provisionner des machines virtuelles SQL Server à l’aide d’Azure PowerShell](create-sql-vm-powershell.md).
   
1. **Est-il possible de créer une image SQL Server généralisée sur la Place de marché Azure de ma machine virtuelle SQL Server et de l’utiliser pour déployer des machines virtuelles ?**

   Oui, mais vous devez ensuite [inscrire chaque machine virtuelle SQL Server auprès de l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md) pour gérer votre machine virtuelle SQL Server dans le portail et utiliser des fonctionnalités telles que la mise à jour corrective automatisée et les sauvegardes automatiques. Lorsque vous vous inscrivez auprès de l’extension, vous devez également spécifier le type de licence pour chaque machine virtuelle SQL Server.

1. **Comment généraliser SQL Server sur une machine virtuelle Azure et l’utiliser pour déployer de nouvelles machines virtuelles ?**

   Vous pouvez déployer une machine virtuelle Windows Server (dépourvue de SQL Server), puis utiliser le processus [SQL sysprep](/sql/database-engine/install-windows/install-sql-server-using-sysprep) pour généraliser SQL Server sur machine virtuelle Azure (Windows) avec le support d’installation SQL Server. Les clients qui disposent de la [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default?rtc=1&activetab=software-assurance-default-pivot%3aprimaryr3) peuvent obtenir leur support d’installation à partir du [Centre de gestion des licences en volume](https://www.microsoft.com/Licensing/servicecenter/default.aspx). Les clients qui ne disposent pas de la Software Assurance peuvent utiliser le support d’installation d’une image de machine virtuelle SQL Server de la Place de marché Azure correspondant à l’édition souhaitée.

   Vous pouvez également utiliser une des images SQL Server de la Place de marché Azure pour généraliser SQL Server sur des machines virtuelles Azure. Notez que vous devez supprimer la clé de Registre suivante dans l'image source avant de créer votre propre image. Si vous ne la supprimez pas, un ballonnement du dossier de démarrage de l'installation SQL Server et/ou de l'extension SQL IaaS peut survenir en état d'échec.

   Chemin de la clé de Registre :  
   `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\SysPrepExternal\Specialize`

   > [!NOTE]
   > SQL Server sur les machines virtuelles Azure, y compris les instances déployées à partir d’images généralisées personnalisées, doit être [inscrit auprès de l’extension SQL IaaS Agent](./sql-agent-extension-manually-register-single-vm.md?tabs=azure-cli%252cbash) afin de respecter les exigences de conformité et d’utiliser des fonctionnalités facultatives telles qu’une mise à jour corrective automatisée et des sauvegardes automatiques. L’extension vous permet également de [spécifier le type de licence](./licensing-model-azure-hybrid-benefit-ahb-change.md?tabs=azure-portal) pour chaque machine virtuelle SQL Server.

1. **Puis-je utiliser mon propre disque dur virtuel pour déployer une machine virtuelle SQL Server ?**

   Oui, mais vous devez ensuite [inscrire chaque machine virtuelle SQL Server auprès de l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md) pour gérer votre machine virtuelle SQL Server dans le portail et utiliser des fonctionnalités telles que la mise à jour corrective automatisée et les sauvegardes automatiques.

1. **Est-il possible de définir des configurations non affichées dans la galerie de machines virtuelles (par exemple, Windows 2008 R2 + SQL Server 2012) ?**

   Non. Pour les images de la galerie de machines virtuelles incluant SQL Server, vous devez sélectionner une des images fournies via le portail Azure ou [PowerShell](create-sql-vm-powershell.md). Toutefois, vous avez la possibilité de déployer une machine virtuelle Windows et d’y installer SQL Server. Vous devez ensuite [inscrire votre machine virtuelle SQL Server auprès de l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md) pour la gérer dans le portail Azure et utiliser des fonctionnalités telles que la mise à jour corrective automatisée et les sauvegardes automatiques. 


## <a name="creation"></a>Création

1. **Comment créer une machine virtuelle Azure avec SQL Server ?**

   La méthode la plus simple consiste à créer une machine virtuelle incluant SQL Server. Pour obtenir un didacticiel sur l’inscription à Azure et la création d’une machine virtuelle SQL Server à partir du portail, consultez [Approvisionner une machine virtuelle SQL Server dans le portail Azure](create-sql-vm-portal.md). Vous pouvez sélectionner une image de machine virtuelle qui utilise la gestion de licences SQL Server avec paiement à la seconde ou vous pouvez utiliser une image qui vous permet d’utiliser votre propre licence SQL Server. Vous avez également la possibilité d’installer manuellement SQL Server sur une machine virtuelle avec une version sous licence gratuite (Developer ou Express) ou en réutilisant une licence locale. Veillez à [inscrire votre machine virtuelle SQL Server auprès de l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md) pour la gérer dans le portail et utiliser des fonctionnalités telles que la mise à jour corrective automatisée et les sauvegardes automatiques. Si vous utilisez votre propre licence, vous devez posséder [License Mobility via Software Assurance sur Azure](https://azure.microsoft.com/pricing/license-mobility/). Pour en savoir plus, consultez l’article [Pricing guidance for SQL Server Azure VMs](pricing-guidance.md) (Tarification des machines virtuelles SQL Server Azure).

1. **Comment migrer ma base de données SQL Server locale vers le cloud ?**

   Créez tout d’abord une machine virtuelle Azure avec une instance SQL Server. Puis migrez vos bases de données locales vers cette instance. Pour les stratégies de migration de données, consultez la rubrique [Migrer une base de données SQL Server vers SQL Server dans une machine virtuelle Azure](migrate-to-vm-from-sql-server.md).

## <a name="licensing"></a>Licence

1. **Comment installer ma copie sous licence de SQL Server sur une machine virtuelle Azure ?**

   Il existe trois façons d'effectuer cette opération. Si vous êtes un client titulaire d’un Accord Entreprise (EA), vous pouvez configurer l’une des [images de machine virtuelle prenant en charge les licences](sql-server-on-azure-vm-iaas-what-is-overview.md#BYOL), également appelée BYOL (apportez votre propre licence). Si vous disposez de [Software Assurance](https://www.microsoft.com/en-us/licensing/licensing-programs/software-assurance-default), vous pouvez activer [Azure Hybrid Benefit](licensing-model-azure-hybrid-benefit-ahb-change.md) sur une image assortie d’un paiement à l’utilisation (PAYG) existante. Vous pouvez également copier le support d’installation de SQL Server dans une machine virtuelle Windows Server, puis installer SQL Server sur la machine virtuelle. Veillez à inscrire votre machine virtuelle SQL Server auprès de [l’extension](sql-agent-extension-manually-register-single-vm.md) pour pouvoir utiliser des fonctionnalités telles que la gestion du portail, la sauvegarde automatisée et la mise à jour corrective automatisée. 

1. **Puis-je modifier une machine virtuelle pour utiliser ma propre licence SQL Server si elle a été créée à partir de l’une des images de la galerie avec paiement à l’utilisation ?**

   Oui. Vous pouvez facilement basculer entre une image de la galerie de paiement à l’utilisation (PAYG) et BYOL (apportez votre propre licence) en activant [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/faq/).  Pour plus d'informations, consultez [Guide pratique pour changer le modèle de licence d'une machine virtuelle SQL Server](licensing-model-azure-hybrid-benefit-ahb-change.md). Actuellement, cette fonctionnalité n’est disponible que pour des clients de cloud public et de cloud Azure Government.

1. **Est-ce que SQL Server rencontre des temps d’arrêt si je change de modèle de licence ?**

   Non. Un [changement de modèle de licence](licensing-model-azure-hybrid-benefit-ahb-change.md) n’entraîne aucun temps d’arrêt dans SQL Server, car il prend effet immédiatement et ne nécessite pas un redémarrage de la machine virtuelle. Toutefois, pour inscrire votre machine virtuelle SQL Server auprès de l’extension SQL IaaS Agent, vous devez disposer de [l’extension SQL IaaS](sql-server-iaas-agent-extension-automate-management.md). L’installation de l’extension SQL IaaS en mode _complet_ redémarre le service SQL Server. Par conséquent, si l’extension SQL IaaS doit être installée, installez-la en mode _léger_ pour des fonctionnalités limitées ou installez-la en mode _complet_ pendant une fenêtre de maintenance. L’extension SQL IaaS installée en mode _léger_ peut être mise à niveau en mode _complet_ à tout moment, mais nécessite un redémarrage du service SQL Server. 
   
1. **Est-il possible de changer les modèles de licence sur une machine virtuelle SQL Server déployée à l’aide du modèle classique ?**

   Non. Le changement des modèles de licence n’est pas pris en charge sur une machine virtuelle classique. Vous pouvez migrer votre machine virtuelle vers le modèle Azure Resource Manager et l’inscrire auprès de l’extension SQL IaaS Agent. Une fois que la machine virtuelle est inscrite auprès de l’extension SQL IaaS Agent, les changements du modèle de licence sont disponibles sur la machine virtuelle.

1. **Puis-je utiliser le portail Azure pour gérer plusieurs instances sur la même machine virtuelle ?**

   Non. La gestion du portail est une fonctionnalité fournie par l’extension SQL IaaS Agent, sur laquelle elle s’appuie. Par conséquent, les mêmes limitations s’appliquent à l’extension. Le portail peut uniquement gérer une instance par défaut, ou une instance nommée, à condition qu’elle soit correctement configurée. Pour plus d’informations sur ces limitations, consultez [Extension de l’agent IaaS SQL Server](sql-server-iaas-agent-extension-automate-management.md). 

1. **Les abonnements CSP peuvent-ils activer Azure Hybrid Benefit ?**

   Oui, Azure Hybrid Benefit est disponible pour les abonnements CSP. Les clients CSP doivent d'abord déployer une image Paiement à l'utilisation, puis [remplacer le modèle de licence](licensing-model-azure-hybrid-benefit-ahb-change.md) par BYOL (apportez votre propre licence).
   
 
1. **Dois-je payer une licence SQL Server sur une machine virtuelle Azure si elle est utilisée uniquement pour le mode veille ou le basculement ?**

   Pour disposer d’une licence passive gratuite pour un groupe de disponibilité secondaire de secours ou une instance en cluster de basculement, vous devez respecter tous les critères suivants, comme décrit dans les [Termes des contrats de licence](https://www.microsoft.com/licensing/product-licensing/products) :

   1. Vous disposez de la [mobilité de licence](https://www.microsoft.com/licensing/licensing-programs/software-assurance-license-mobility?activetab=software-assurance-license-mobility-pivot:primaryr2) à travers la [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default?activetab=software-assurance-default-pivot%3aprimaryr3). 
   1. L’instance SQL Server passive ne fournit pas de données SQL Server aux clients ni n’exécute de charges de travail SQL Server actives. Elle est utilisée uniquement pour la synchronisation avec le serveur principal et maintient la base de données passive dans un état de secours semi-automatique. Si elle fournit des données, comme des rapports, à des clients exécutant des charges de travail SQL Server actives, ou si elle effectue n’importe quels travaux autres que ceux qui sont spécifiés dans les termes du contrat, il doit s’agir d’une instance SQL Server sous licence payante. Les activités suivantes sont autorisées sur l’instance secondaire : vérifications de cohérence de la base de données ou CheckDB, sauvegardes complètes, sauvegardes du journal des transactions et supervision des données d’utilisation des ressources. Vous pouvez également exécuter simultanément l’instance principale et l’instance de reprise d’activité correspondante pendant de courtes périodes de test de reprise d’activité tous les 90 jours. 
   1. La licence SQL Server active est couverte par Software Assurance et n’autorise qu’**une seule** instance SQL Server secondaire passive avec la même quantité de calcul que le serveur actif sous licence. 
   1. La machine virtuelle SQL Server secondaire utilise la licence [Reprise d’activité](business-continuity-high-availability-disaster-recovery-hadr-overview.md#free-dr-replica-in-azure) dans le portail Azure.
   
1. **Qu’est-ce qu’une instance passive ?**

   L’instance SQL Server passive ne fournit pas de données SQL Server aux clients ni n’exécute de charges de travail SQL Server actives. Elle est utilisée uniquement pour la synchronisation avec le serveur principal et maintient la base de données passive dans un état de secours semi-automatique. Si elle fournit des données, comme des rapports, à des clients exécutant des charges de travail SQL Server actives, ou si elle effectue tout travail autre que ceux qui sont spécifiés dans les termes du contrat, il doit s’agir d’une instance SQL Server sous licence payante. Les activités suivantes sont autorisées sur l’instance secondaire : vérifications de cohérence de la base de données ou CheckDB, sauvegardes complètes, sauvegardes du journal des transactions et supervision des données d’utilisation des ressources. Vous pouvez également exécuter simultanément l’instance principale et l’instance de reprise d’activité correspondante pendant de courtes périodes de test de reprise d’activité tous les 90 jours.
   

1. **Quels scénarios peuvent tirer parti de l'avantage Récupération d'urgence ?**

   Le [Guide de gestion des licences](https://aka.ms/sql2019licenseguide) fournit des scénarios dans lesquels l’avantage Reprise d’activité après sinistre peut être utilisé. Pour plus d’informations, reportez-vous aux Termes du contrat de licence du produit et contactez les agents chargés de la gestion des licences ou votre responsable de compte.

1. **Quels sont les abonnements qui prennent en charge l’avantage Reprise d’activité après sinistre ?**

   Les programmes complets qui offrent des droits d’abonnement équivalents à la Software Assurance en tant qu’avantage fixe prennent en charge l’avantage Reprise d’activité après sinistre. Cela comprend, notamment Open Value (OV), Open Value Subscription (OVS), Contrat Entreprise (EA), Abonnement Accord Entreprise (EAS) et Server and Cloud Enrollment (SCE). Pour plus d'informations, reportez-vous aux [Termes du contrat de licence du produit](https://www.microsoft.com/licensing/product-licensing/products) et contactez les agents chargés de la gestion des licences ou votre responsable de compte. 

   
 ## <a name="extension"></a>Extension

1. **Est-ce que l’inscription de ma machine virtuelle auprès de la nouvelle extension SQL IaaS Agent engendre des frais supplémentaires ?**

   Non. L’extension SQL IaaS Agent permet simplement de faciliter encore davantage la gestion de SQL Server sur les machines virtuelles Azure sans frais supplémentaires. 

1. **L’extension SQL IaaS Agent est-elle disponible pour tous les clients ?**
 
   Oui, tant que la machine virtuelle SQL Server a été déployée sur le cloud public à l’aide du modèle Resource Manager, et non du modèle classique. Tous les autres clients peuvent s’inscrire auprès de la nouvelle extension SQL IaaS Agent. Toutefois, seuls les clients disposant de l’avantage [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default?activetab=software-assurance-default-pivot%3aprimaryr3) peuvent utiliser leur propre licence pour activer [Azure Hybrid Benefit (AHB)](https://azure.microsoft.com/pricing/hybrid-benefit/) sur une machine virtuelle SQL Server. 

1. **Que se passe-t-il pour la ressource d’extension (_Microsoft.SqlVirtualMachine_) si la ressource de machine virtuelle est déplacée ou supprimée ?** 

   Quand la ressource Microsoft.Compute/VirtualMachine est supprimée ou déplacée, la ressource Microsoft.SqlVirtualMachine associée est alors notifiée pour répliquer l’opération de façon asynchrone.

1. **Que se passe-t-il pour la machine virtuelle si la ressource d’extension (_Microsoft.SqlVirtualMachine_) est supprimée ?**

    La ressource Microsoft.Compute/VirtualMachine n’est pas impactée quand la ressource Microsoft.SqlVirtualMachine est supprimée. Toutefois, les changements de licence rétablissent par défaut la source d’image d’origine. 

1. **Est-il possible d’inscrire des machines virtuelles SQL Server autodéployées auprès de l’extension SQL IaaS Agent ?**

    Oui. Si vous avez déployé SQL Server à partir de votre propre support et installé l’extension SQL IaaS, vous pouvez inscrire votre machine virtuelle SQL Server auprès de l’extension pour profiter des avantages de gestion fournis par l’extension SQL IaaS.    


## <a name="administration"></a>Administration

1. **Puis-je installer une seconde instance SQL Server sur la même machine virtuelle ? Puis-je modifier les fonctionnalités installées de l’instance par défaut ?**

   Oui. Le support d’installation de SQL Server se trouve dans un dossier sur le lecteur **C** . Exécutez **Setup.exe** à partir de cet emplacement pour ajouter de nouvelles instances SQL Server ou pour modifier d’autres fonctionnalités SQL Server installées sur la machine. Notez que certaines fonctionnalités comme la sauvegarde automatisée, la mise à jour corrective automatisée et Azure Key Vault Integration s’exécutent uniquement sur l’instance par défaut ou sur une instance nommée qui a été correctement configurée (voir la question 3). Les clients qui disposent d’une couverture [Software Assurance via Azure Hybrid Benefit](licensing-model-azure-hybrid-benefit-ahb-change.md) ou du modèle de licence **Paiement à l’utilisation** peuvent installer plusieurs instances SQL Server sur la machine virtuelle sans entraîner de frais de licence supplémentaires. Des instances SQL Server supplémentaires risquent de surcharger les ressources système, sauf si elles sont correctement configurées. 

1. **Quel est le nombre maximal d’instances sur une machine virtuelle ?**
   De SQL Server 2012 à SQL Server 2019, [50](/sql/sql-server/editions-and-components-of-sql-server-version-15#RDBMSSP)instances peuvent être prises en charge sur un serveur autonome. La même limite s’applique localement et dans Azure. Consultez les [bonnes pratiques](performance-guidelines-best-practices.md#multiple-instances) pour découvrir comment mieux préparer votre environnement. 

1. **Puis-je désinstaller l’instance SQL Server par défaut ?**

   Oui, mais il existe quelques considérations à prendre en compte. Premièrement, la facturation associée à SQL Server peut continuer à s’appliquer en fonction du modèle de licence de la machine virtuelle. Deuxièmement, comme indiqué dans la réponse précédente, certaines fonctionnalités reposent sur l’[extension de l’agent IaaS SQL Server](sql-server-iaas-agent-extension-automate-management.md). Si vous désinstallez l’instance par défaut sans supprimer l’extension IaaS, cette dernière continue de rechercher l’instance par défaut et peut générer des erreurs dans le journal des événements. Ces erreurs peuvent avoir deux sources : **Gestion des informations d’identification Microsoft SQL Server** et **Agent IaaS Microsoft SQL Server**. L’une des erreurs peut se présenter comme suit :

      Une erreur liée au réseau ou propre à une instance s’est produite lors de l’établissement d’une connexion à SQL Server. Le serveur est introuvable ou inaccessible.

   Si vous décidez de désinstaller l’instance par défaut, vous devez également désinstaller [l’extension de l’agent IaaS SQL Server](sql-server-iaas-agent-extension-automate-management.md). 

1. **Puis-je utiliser une instance nommée de SQL Server avec l’extension IaaS ?**
   
   Oui, si l’instance nommée est la seule instance sur SQL Server et si l’instance par défaut d’origine a été [désinstallé correctement](sql-server-iaas-agent-extension-automate-management.md#named-instance-support). En l’absence d’instance par défaut et si plusieurs instances nommées sont présentes sur une même machine virtuelle SQL Server, l’extension de l’agent IaaS SQL Server échoue à l’installation.  

1. **Puis-je supprimer SQL Server et la facturation de licence associée d’une machine virtuelle SQL Server ?**

   Oui, mais vous devrez prendre des mesures supplémentaires pour éviter d’être facturé pour votre instance de SQL Server, comme décrit dans le [Guide des prix de SQL Server sur machines virtuelles Azure](pricing-guidance.md). Si vous souhaitez supprimer complètement l’instance SQL Server, vous pouvez migrer vers une autre machine virtuelle Azure dans laquelle SQL Server n’est pas pré-installé, puis supprimer la machine virtuelle SQL Server actuelle. Si vous souhaitez conserver la machine virtuelle mais arrêter la facturation de SQL Server, procédez comme suit : 

   1. Le cas échéant, sauvegardez toutes vos données, y compris les bases de données système. 
   1. Désinstallez SQL Server complètement, y compris l’extension IaaS SQL (si elle est installée).
   1. Installez l’édition gratuite [SQL Express](https://www.microsoft.com/sql-server/sql-server-downloads).
   1. Inscrivez-vous auprès de l’extension SQL IaaS Agent en [mode léger](sql-agent-extension-manually-register-single-vm.md).
   1. (Facultatif) désactivez le service Express SQL Server en désactivant le démarrage du service. 

1. **Puis-je utiliser le portail Azure pour gérer plusieurs instances sur la même machine virtuelle ?**

   Non. La gestion du portail est fournie par l’extension SQL IaaS Agent, sur laquelle elle s’appuie. Par conséquent, les mêmes limitations s’appliquent à l’extension. Le portail peut uniquement gérer une instance par défaut, ou une instance nommée, à condition qu’elle soit correctement configurée. Pour plus d’informations, consultez [Extension de l’agent IaaS SQL Server](sql-server-iaas-agent-extension-automate-management.md). 


## <a name="updating-and-patching"></a>Mise à jour et mise à jour corrective

1. **Comment faire pour passer à une autre version/édition de SQL Server dans une machine virtuelle Azure ?**

   Les clients peuvent modifier leur version/édition de SQL Server à l’aide du support d’installation qui contient leur version ou édition souhaitée de SQL Server. Une fois que l’édition a été modifiée, utilisez le portail Azure pour modifier la propriété d’édition de la machine virtuelle pour refléter avec précision la facturation pour la machine virtuelle. Pour plus d’informations, consultez [Modifier l’édition d’une machine virtuelle SQL Server](change-sql-server-edition.md). Il n’existe aucune différence de facturation entre les différentes versions de SQL Server. Par conséquent, une fois que la version de SQL Server a été modifiée, aucune action supplémentaire n’est nécessaire.

1. **Où puis-je obtenir le support d’installation pour modifier l’édition ou la version de SQL Server ?**

   Les clients qui disposent de la [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default) peuvent obtenir leur support d’installation à partir du [Centre de gestion des licences en volume](https://www.microsoft.com/Licensing/servicecenter/default.aspx). Les clients ne disposant pas de Software Assurance peuvent utiliser le support d’installation à partir d’une image de machine virtuelle SQL Server de la Place de marché Azure correspondant à l’édition souhaitée.
   
1. **Comment les mises à jour et les service packs sont-ils appliqués sur une machine virtuelle SQL Server ?**

   Les machines virtuelles vous permettent de contrôler la machine hôte, y compris quand et comment appliquer les mises à jour. Pour le système d’exploitation, vous pouvez appliquer manuellement les mises à jour Windows, ou activer un service de planification appelé [Mise à jour corrective automatisée](automated-patching.md). La mise à jour corrective automatisée installe toutes les mises à jour importantes, y compris les mises à jour de SQL Server de cette catégorie. Les autres mises à jour SQL Server facultatives doivent être installées manuellement.

1. **Puis-je mettre à niveau mon instance de SQL Server 2008/2008 R2 après l’avoir inscrite auprès de l’extension SQL IaaS Agent ?**

   Oui, si le système d’exploitation est Windows Server 2008 R2 ou ultérieur. Vous pouvez utiliser n’importe quel support d’installation pour mettre à niveau la version et l’édition de SQL Server. Vous pouvez ensuite mettre à niveau votre [mode d’extension SQL IaaS](sql-server-iaas-agent-extension-automate-management.md#management-modes) de _aucun agent_ vers _complet_. Vous pouvez ainsi tirer profit de tous les avantages de l’extension SQL IaaS tels que la facilité de gestion du portail, les sauvegardes automatisées et la mise à jour corrective automatisée. Si la version du système d’exploitation est Windows Server 2008, seul le mode NoAgent est pris en charge. 

1. **Comment obtenir des mises à jour de sécurité étendue gratuites pour mes instances SQL Server 2008 et SQL Server 2008 R2 qui sont en fin de support ?**

   Vous pouvez vous procurer les [mises à jour de sécurité étendue gratuites](sql-server-2008-extend-end-of-support.md) en déplaçant votre serveur SQL Server en l’état vers une machine virtuelle Azure. Pour plus d’informations, consultez [Options de fin du support](/sql/sql-server/end-of-support/sql-server-end-of-life-overview). 
  
   

## <a name="general"></a>Général

1. **Les instances de cluster de basculement (FCI) SQL Server sont-elles prises en charge sur les machines virtuelles Azure ?**

   Oui. Vous pouvez installer une instance de cluster de basculement à l’aide de [partages de fichiers Premium (PFS)](failover-cluster-instance-premium-file-share-manually-configure.md) ou d’[espaces de stockage direct (S2D)](failover-cluster-instance-storage-spaces-direct-manually-configure.md) pour le sous-système de stockage. Les partages de fichiers Premium fournissent une capacité d’IOPS et de débit qui répond aux besoins de nombreuses charges de travail. Pour les charges de travail gourmandes en E/S, il peut être préférable d’utiliser des espaces de stockage direct basés sur des disques Premium managés ou des disques Ultra. Vous pouvez également utiliser les solutions de clustering ou de stockage tierces comme décrit dans [Haute disponibilité et récupération d’urgence pour SQL Server sur des machines virtuelles Azure](business-continuity-high-availability-disaster-recovery-hadr-overview.md#azure-only-high-availability-solutions).

   > [!IMPORTANT]
   > À ce stade, l’[extension de l’agent IaaS SQL Server](sql-server-iaas-agent-extension-automate-management.md)_complète_ n’est pas prise en charge pour l’ICF SQL Server sur Azure. Nous vous recommandons de désinstaller l’extension _complète_ des machines virtuelles qui participent à l’ICF, puis d’installer l’extension en mode _léger_ à la place. Cette extension prend en charge des fonctionnalités telles que la sauvegarde et la mise à jour corrective automatisées, ainsi que certaines fonctionnalités du portail pour SQL Server. Ces fonctionnalités ne s’opèrent pas pour les machines virtuelles SQL Server une fois l’agent _complet_ désinstallé.

1. **Quelle est la différence entre les machines virtuelles SQL Server et le service SQL Database ?**

   Conceptuellement, l’exécution de SQL Server sur une machine virtuelle Azure n’est pas si différente de l’exécution de SQL Server dans un centre de données distant. En revanche, la [base de données SQL](../../database/sql-database-paas-overview.md) fournit une base de données relationnelle complète en tant que service. Avec SQL Database, vous n’avez pas besoin d’accéder aux machines qui hébergent vos bases de données. Pour une comparaison complète, consultez [Choisir une option SQL Serveur cloud : Base de données SQL Azure (PaaS) ou SQL Server sur des machines virtuelles Azure (IaaS)](../../azure-sql-iaas-vs-paas-what-is-overview.md).

1. **Comment installer les outils SQL Data sur ma machine virtuelle Azure ?**

    Téléchargez et installez les outils SQL Data à partir de [Microsoft SQL Server Data Tools - Business Intelligence pour Visual Studio 2013 ](https://www.microsoft.com/download/details.aspx?id=42313).

1. **Les transactions distribuées avec MSDTC sont-elles prises en charge sur les machines virtuelles SQL Server ?**
   
    Oui. Le DTC local est pris en charge pour SQL Server 2016 SP2 et versions ultérieures. Cependant, les applications utilisant des groupes de disponibilité AlwaysOn doivent être testées, car les transactions en cours au moment d’un basculement échouent et doivent être relancées. Le DTC en cluster est disponible à partir de Windows Server 2019. 

## <a name="resources"></a>Ressources

**Machines virtuelles Windows** :

* [Vue d’ensemble de SQL Server sur une machine virtuelle Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Approvisionner SQL Server sur une machine virtuelle Windows](create-sql-vm-portal.md)
* [Migrating a Database to SQL Server on an Azure VM](migrate-to-vm-from-sql-server.md)
* [Haute disponibilité et récupération d’urgence pour SQL Server sur des machines virtuelles Azure](business-continuity-high-availability-disaster-recovery-hadr-overview.md)
* [Meilleures pratiques relatives aux performances de SQL Server sur les machines virtuelles Azure](performance-guidelines-best-practices.md)
* [Modèles d’application et stratégies de développement pour SQL Server sur des machines virtuelles Azure](application-patterns-development-strategies.md)

**Machines virtuelles Linux** :

* [Vue d’ensemble de SQL Server sur une machine virtuelle Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)
* [Approvisionner SQL Server sur une machine virtuelle Linux](../linux/sql-vm-create-portal-quickstart.md)
* [Forum Aux Questions (Linux)](../linux/frequently-asked-questions-faq.md)
* [Documentation relative à SQL Server sur Linux](/sql/linux/sql-server-linux-overview)
