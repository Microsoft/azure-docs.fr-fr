---
title: Évaluer les serveurs locaux à l’aide d’un fichier CSV importé avec Azure Migrate Server Assessment
description: Explique comment découvrir des serveurs locaux pour la migration vers Azure avec un fichier CSV importé dans Azure Migrate Server Assessment
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: tutorial
ms.date: 09/14/2020
ms.openlocfilehash: dfa7ee941e2c373b02fe5fb2f2a648a60a677670
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753107"
---
# <a name="tutorial-assess-servers-using-an-imported-csv-file"></a>Tutoriel : Évaluer les serveurs à l’aide d’un fichier CSV importé

Dans le cadre de votre migration vers Azure, vous devez découvrir vos charges de travail locales et réaliser votre inventaire. 

Ce tutoriel vous montre comment évaluer des machines virtuelles locales avec l’outil Azure Migrate Server Assessment, à l’aide d’un fichier CSV importé. 

Si vous utilisez un fichier CSV, vous n’avez pas besoin de configurer l’appliance Azure Migrate pour découvrir et évaluer les serveurs. Vous pouvez contrôler les données que vous partagez dans le fichier. En outre, la plupart des données sont facultatives. Cette méthode est utile pour :

- Vous souhaitez créer une évaluation initiale rapide avant de déployer l’appliance.
- Vous ne pouvez pas déployer l’appliance Azure Migrate dans votre organisation.
- Vous ne pouvez pas partager des informations d’identification qui autorisent l’accès aux serveurs locaux.
- Les contraintes de sécurité vous empêchent de collecter et d’envoyer les données collectées par l’appliance à Azure.

> [!NOTE]
> Vous ne pouvez pas effectuer la migration des serveurs importés à l’aide d’un fichier CSV.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Configurer un compte Azure
> * Configurer un projet Azure Migrate
> * Préparer un fichier CSV
> * Importer le fichier
> * Évaluer des serveurs

> [!NOTE]
> Les tutoriels indiquent le moyen le plus rapide de tester un scénario, et ils utilisent les options par défaut lorsque cela est possible. 

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

- Vous pouvez ajouter jusqu’à 20 000 serveurs dans un même fichier CSV et dans un même projet Azure Migrate. 
- Les noms de systèmes d’exploitation spécifiés dans le fichier CSV doivent correspondre aux [noms pris en charge](#supported-operating-system-names).


## <a name="prepare-an-azure-user-account"></a>Préparer un compte de stockage Azure

Pour créer un projet Azure Migrate, vous avez besoin d’un compte avec :
- Des autorisations de niveau Contributeur ou Propriétaire sur un abonnement Azure
- Des autorisations permettant d’inscrire des applications Azure Active Directory

Si vous venez de créer un compte Azure gratuit, vous êtes le propriétaire de votre abonnement. Si vous n’êtes pas le propriétaire de l’abonnement, demandez à celui-ci de vous attribuer les autorisations de la façon suivante :

1. Dans le portail Azure, recherchez « abonnements », puis sous **Services**, sélectionnez **Abonnements**.

    ![Zone de recherche pour rechercher l’abonnement Azure](./media/tutorial-discover-import/search-subscription.png)

2. Dans la page **Abonnements**, sélectionnez l’abonnement dans lequel vous souhaitez créer un projet Azure Migrate. 
3. Dans l’abonnement, sélectionnez **Contrôle d’accès (IAM)**  > **Vérifier l’accès**.
4. Dans **Vérifier l’accès**, recherchez le compte d’utilisateur correspondant.
5. Dans **Ajouter une attribution de rôle**, cliquez sur **Ajouter**.

    ![Rechercher un compte d’utilisateur pour vérifier l’accès et attribuer un rôle](./media/tutorial-discover-import/azure-account-access.png)

6. Dans **Ajouter une attribution de rôle**, sélectionnez le rôle Contributeur ou Propriétaire, puis sélectionnez le compte (azmigrateuser dans notre exemple). Ensuite, cliquez sur **Enregistrer**.

    ![Ouvre la page Ajouter une attribution de rôle pour attribuer un rôle au compte.](./media/tutorial-discover-import/assign-role.png)

7. Dans le portail, recherchez des utilisateurs, puis, sous **Services**, sélectionnez **Utilisateurs**.
8. Dans **Paramètres utilisateur**, vérifiez que les utilisateurs Azure AD peuvent inscrire des applications (défini sur **Oui** par défaut).

    ![Vérifier dans les paramètres utilisateur que les utilisateurs peuvent inscrire des applications Active Directory](./media/tutorial-discover-import/register-apps.png)



## <a name="set-up-a-project"></a>Configuration d’un projet

Configurez un nouveau projet Azure Migrate si vous n’en avez pas encore un.

1. Dans le portail Azure, sélectionnez **Tous les services**, puis recherchez **Azure Migrate**.
2. Sous **Services**, sélectionnez **Azure Migrate**.
3. Dans **Vue d’ensemble**, sélectionnez **Créer un projet**.
5. Dans **Créer un projet**, sélectionnez votre abonnement Azure et votre groupe de ressources. Créez un groupe de ressources si vous n’en avez pas.
6. Dans **Détails du projet**, spécifiez le nom du projet ainsi que la zone géographique où vous souhaitez le créer. Passez en revue les zones géographiques prises en charge pour les clouds [publics](migrate-support-matrix.md#supported-geographies-public-cloud) et du [secteur public](migrate-support-matrix.md#supported-geographies-azure-government).

   ![Zones pour le nom et la région du projet](./media/tutorial-discover-import/new-project.png)

7. Sélectionnez **Create** (Créer).
8. Attendez quelques minutes, le temps nécessaire au déploiement du projet Azure Migrate.

L’outil **Azure Migrate Server Assessment** est ajouté par défaut au nouveau projet.

![Page montrant l’outil Server Assessment ajouté par défaut](./media/tutorial-discover-import/added-tool.png)

## <a name="prepare-the-csv"></a>Préparer le fichier CSV

Téléchargez le modèle CSV et ajoutez-y les informations sur le serveur.

### <a name="download-the-template"></a>Téléchargez le modèle

1. Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Assessment**, sélectionnez **Découvrir**.
2. Dans **Détecter des machines**, sélectionnez **Importer à l’aide de CSV**.
3. Sélectionnez **Télécharger** pour télécharger le modèle CSV. Vous pouvez aussi [le télécharger directement](https://go.microsoft.com/fwlink/?linkid=2109031).

    ![Télécharger le modèle CSV](./media/tutorial-discover-import/download-template.png)

### <a name="add-server-information"></a>Ajouter les informations sur le serveur

Collectez les données du serveur et ajoutez-les au fichier CSV.

- Pour collecter les données, vous pouvez les exporter à partir des outils que vous utilisez pour la gestion des serveurs locaux, par exemple VMware vSphere ou votre base de données de gestion des configurations (CMDB).
- Pour consulter les exemples de données, téléchargez notre [exemple de fichier](https://go.microsoft.com/fwlink/?linkid=2108405).

Le tableau suivant récapitule les champs du fichier à remplir :

**Nom du champ** | **Obligatoire** | **Détails**
--- | --- | ---
**Nom du serveur** | Oui | Nous vous recommandons de spécifier le nom de domaine complet (FQDN).
**Adresse IP** | Non | Adresse du serveur.
**Cœurs** | Oui | Nombre de cœurs de processeur alloués au serveur.
**Mémoire** | Oui | RAM totale, en Mo, allouée au serveur.
**Nom du système d’exploitation** | Oui | Système d’exploitation du serveur. <br/> Les noms de système d’exploitation qui correspondent aux noms ou qui contiennent les noms figurant dans [cette liste](#supported-operating-system-names) sont reconnus par l’évaluation.
**Version du SE** | Non | Version du système d’exploitation du serveur.
**Architecture du système d’exploitation** | Non | Architecture du système d’exploitation serveur <br/> Les valeurs valides sont : x64, x86, amd64, 32 bits ou 64 bits
**Nombre de disques** | Non | Inutile si les détails sur le disque individuel sont fournis.
**Taille du disque 1**  | Non | Taille maximale du disque, en Go<br/>Vous pouvez ajouter les détails d’autres disques en [ajoutant des colonnes](#add-multiple-disks) dans le modèle. Vous pouvez ajouter jusqu’à huit disques.
**Opérations de lecture sur le disque 1** | Non | Opérations de lecture sur le disque par seconde.
**Opérations d’écriture sur le disque 1** | Non | Opérations d’écriture sur le disque par seconde.
**Débit de lecture sur le disque 1** | Non | Données lues à partir du disque par seconde, en Mo par seconde.
**Débit d’écriture sur le disque 1** | Non | Données écrites sur le disque par seconde, en Mo par seconde.
**Pourcentage d’utilisation du processeur** | Non | Pourcentage d’utilisation du processeur.
**Pourcentage d’utilisation de la mémoire** | Non | Pourcentage de RAM utilisé.
**Total des opérations de lecture sur les disques** | Non | Opérations de lecture sur le disque par seconde.
**Total des opérations d’écriture sur les disques** | Non | Opérations d’écriture sur le disque par seconde.
**Total du débit de lecture sur les disques** | Non | Données lues à partir du disque, en Mo par seconde.
**Total du débit d’écriture sur les disques** | Non | Données écrites sur le disque, en Mo par seconde.
**Débit d’entrée réseau** | Non | Données reçues par le serveur, en Mo par seconde.
**Débit de sortie réseau** | Non | Données transmises par le serveur, en Mo par seconde.
**Type de microprogramme** | Non | Microprogramme du serveur. Les valeurs peuvent être « BIOS » ou « UEFI ».
**Adresse MAC**| Non | Adresse MAC du serveur.


### <a name="add-operating-systems"></a>Ajouter des systèmes d’exploitation

L’évaluation reconnaît des noms de système d’exploitation spécifiques. Tout nom que vous spécifiez doit correspondre exactement à l’une des chaînes de la [liste des noms pris en charge](#supported-operating-system-names).

### <a name="add-multiple-disks"></a>Ajouter plusieurs disques

Le modèle fournit des champs par défaut pour le premier disque. Vous pouvez ajouter des colonnes similaires pour un maximum de huit disques.

Par exemple, pour spécifier tous les champs pour un deuxième disque, ajoutez ces colonnes :

- Taille du disque 2
- Opérations de lecture sur le disque 2
- Opérations d’écriture sur le disque 2
- Débit de lecture sur le disque 2
- Débit d’écriture sur le disque 2


## <a name="import-the-server-information"></a>Importer les informations du serveur

Après avoir ajouté des informations au modèle CSV, importez le fichier CSV dans Server Assessment.

1. Dans Azure Migrate, dans **Détecter des machines**, accédez au modèle terminé.
2. Sélectionnez **Importer**.
3. L’état de l’importation est affiché.
    - Si des avertissements s’affichent dans l’état, vous pouvez soit les corriger, soit poursuivre sans les résoudre.
    - Pour améliorer la précision de l’évaluation, améliorez l’exactitude des informations sur le serveur, comme indiqué dans les avertissements.
    - Pour afficher et corriger les avertissements, sélectionnez **Télécharger le fichier .CSV des détails de l’avertissement**. Cette opération télécharge le fichier CSV avec des avertissements inclus. Passez en revue les avertissements et résolvez les problèmes si nécessaire.
    - Si des erreurs s’affichent dans l’état et que l’état de l’importation est **Échec**, vous devez corriger ces erreurs avant de pouvoir continuer l’importation :
        1. Téléchargez le fichier CSV, qui contient désormais les détails sur l’erreur.
        1. Examinez et corrigez les erreurs en fonction des besoins. 
        1. Rechargez le fichier modifié.
4. Quand l’état de l’importation est **Terminé**, cela signifie que les informations sur le serveur ont été importées. Actualisez la page si le processus d’importation ne semble pas terminé.

## <a name="update-server-information"></a>Mettre à jour les informations sur le serveur

Vous pouvez mettre à jour les informations sur un serveur en réimportant les données du serveur avec le même **nom de serveur**. Vous ne pouvez pas modifier le champ **Nom de serveur**. La suppression de serveurs n’est pas prise en charge actuellement.

## <a name="verify-servers-in-the-portal"></a>Vérifier les serveurs dans le portail

Pour vérifier que les serveurs apparaissent dans le portail Azure une fois la découverte effectuée

1. Ouvrez le tableau de bord Azure Migrate.
2. Dans la page **Azure Migrate - Serveurs** > **Azure Migrate : Server Assessment**, sélectionnez l’icône qui affiche le nombre de **Serveurs découverts**.
3. Sélectionnez l’onglet **Basé sur l’importation**.



## <a name="supported-operating-system-names"></a>Noms des systèmes d’exploitation pris en charge

Les noms de systèmes d’exploitation fournis dans le fichier CSV doivent correspondre aux noms pris en charge. Si ce n’est pas le cas, vous ne pourrez pas les évaluer. 

**A - H** | **I - R** | **S-T** | **U-Z**
--- | --- | --- | ---
Apple Mac OS X 10<br/>Asianux 3<br/>Asianux 4<br/>Asianux 5<br/>CentOS<br/>CentOS 4/5<br/>CoreOS Linux<br/>Debian GNU/Linux 4<br/>Debian GNU/Linux 5<br/>Debian GNU/Linux 6<br/>Debian GNU/Linux 7<br/>Debian GNU/Linux 8<br/>FreeBSD | IBM OS/2<br/>MS-DOS<br/>Novell NetWare 5<br/>Novell NetWare 6<br/>Oracle Linux<br/>Oracle Linux 4/5<br/>Oracle Solaris 10<br/>Oracle Solaris 11<br/>Red Hat Enterprise Linux 2<br/>Red Hat Enterprise Linux 3<br/>Red Hat Enterprise Linux 4<br/>Red Hat Enterprise Linux 5<br/>Red Hat Enterprise Linux 6<br/>Red Hat Enterprise Linux 7<br/>Red Hat Fedora | SCO OpenServer 5<br/>SCO OpenServer 6<br/>SCO UnixWare 7<br/> Serenity Systems eComStation 1<br/>Serenity Systems eComStation <br/>Sun Microsystems Solaris 8<br/>Sun Microsystems Solaris 9<br/><br/>SUSE Linux Enterprise 10<br/>SUSE Linux Enterprise 11<br/>SUSE Linux Enterprise 12<br/>SUSE Linux Enterprise 8/9<br/>SUSE Linux Enterprise 11<br/>SUSE openSUSE | Ubuntu Linux<br/>VMware ESXi 4<br/>VMware ESXi 5<br/>VMware ESXi 6<br/>Windows 10<br/>Windows 2000<br/>Windows 3<br/>Windows 7<br/>Windows 8<br/>Windows 95<br/>Windows 98<br/>Windows NT<br/>Windows Server (R) 2008<br/>Windows Server 2003<br/>Windows Server 2008<br/>Windows Server 2008 R2<br/>Windows Server 2012<br/>Windows Server 2012 R2<br/>Windows Server 2016<br/>Windows Server 2019<br/>Windows Server Threshold<br/>Windows Vista<br/>Windows Web Server 2008 R2<br/>Windows XP Professionnel

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créé un projet Azure Migrate 
> * Découvert des serveurs à l’aide d’un fichier CSV importé À présent, exécutez une évaluation pour la [migration des machines virtuelles VMware vers des machines virtuelles Azure](./tutorial-assess-vmware-azure-vm.md).