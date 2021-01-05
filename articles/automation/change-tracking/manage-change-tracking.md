---
title: Gérer Suivi des modifications et inventaire dans Azure Automation
description: Cet article explique comment utiliser le Change Tracking and Inventory pour suivre les modifications apportées aux logiciels et aux services Microsoft dans votre environnement.
services: automation
ms.subservice: change-inventory-management
ms.date: 12/10/2020
ms.topic: conceptual
ms.openlocfilehash: 636dbf95567f761aee19bd567b0835173ce36ccc
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97093619"
---
# <a name="manage-change-tracking-and-inventory"></a>Gérer Change Tracking et Inventory

Quand vous ajoutez un fichier ou une clé de Registre à suivre, Azure Automation l’active pour [Change Tracking et Inventory](overview.md). Cet article explique comment configurer le suivi, passer en revue les résultats du suivi et gérer les alertes en cas de détection de modifications.

Avant d’appliquer les procédures décrites dans cet article, vérifiez que vous avez activé Change Tracking et Inventory sur vos machines virtuelles à l’aide de l’une des techniques suivantes :

* [Activer Change Tracking et Inventory à partir d’un compte Automation](enable-from-automation-account.md)
* [Activer Change Tracking et Inventory en parcourant le portail Azure](enable-from-portal.md)
* [Activer Change Tracking et Inventory à partir d’un runbook](enable-from-runbook.md)
* [Activer Change Tracking et Inventory à partir d’une machine virtuelle Azure](enable-from-vm.md)

## <a name="limit-the-scope-for-the-deployment"></a><a name="scope-configuration"></a>Limiter l’étendue du déploiement

La fonctionnalité Change Tracking et Inventory utilise une configuration d’étendue dans l’espace de travail pour cibler les ordinateurs devant recevoir des modifications. Pour plus d’informations, consultez [Limiter l’étendue du déploiement de Change Tracking et Inventory](manage-scope-configurations.md).

## <a name="track-files"></a>Suivre des fichiers

Vous pouvez utiliser Change Tracking et Inventory pour suivre les modifications apportées aux fichiers et aux dossiers/répertoires. Cette section vous explique comment configurer le suivi des fichiers sur Windows et Linux.

### <a name="configure-file-tracking-on-windows"></a>Configurer le suivi des fichiers sur Windows

Utilisez les étapes suivantes pour configurer le suivi des fichiers sur des ordinateurs Windows :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Dans le portail Azure, sélectionnez **Tous les services**. Dans la liste de ressources, saisissez **Automation**. Au fur et à mesure de la saisie, la liste filtre les suggestions. Sélectionnez **Comptes Automation**.

3. Dans votre liste de comptes Automation, sélectionnez le compte que vous avez choisi quand vous avez activé Suivi des modifications et inventaire.

4. Dans votre compte Automation, sélectionnez **Change Tracking** sous **Gestion de la configuration**.

5. Sélectionnez **Modifier les paramètres** (symbole de l’engrenage).

6. Dans la page Configuration de l’espace de travail, sélectionnez **Fichiers Windows**, puis cliquez sur **+ Ajouter** pour ajouter un nouveau fichier à suivre.

7. Dans le volet Ajouter le fichier Windows pour le suivi des modifications, entrez les informations du fichier ou du dossier à suivre, puis cliquez sur **Enregistrer**. Le tableau suivant définit les propriétés que vous pouvez utiliser pour les informations.

    |Propriété  |Description  |
    |---------|---------|
    |activé     | True si le paramètre est appliqué, False sinon.        |
    |Item Name     | Nom convivial du fichier à suivre.        |
    |Groupe     | Nom de groupe pour le regroupement logique des fichiers.        |
    |Entrer le chemin     | Chemin où rechercher le fichier, par exemple **c:\temp\\\*.txt**. Vous pouvez également utiliser des variables d’environnement telles que `%winDir%\System32\\\*.*`.       |
    |Type de chemin     | Type du chemin. Les valeurs possibles sont Fichier et Dossier.        |    
    |Récursivité     | True si la récursivité est utilisée pour rechercher l’élément à suivre, False sinon.        |    
    |Charger le contenu du fichier | True pour charger le contenu du fichier d'après les modifications suivies, et False dans le cas contraire.|

    Si vous envisagez de configurer l’analyse des fichiers et des dossiers à l’aide de caractères génériques, tenez compte des points suivants :

    - Les caractères génériques sont requis pour effectuer le suivi de plusieurs fichiers.
    - Les caractères génériques ne peuvent être utilisés que dans le dernier segment d’un chemin d’accès, comme *C:\dossier\fichier* ou */etc/* .conf*
    - Si le chemin d’accès d’une variable d’environnement n’est pas valide, la validation réussit, mais le chemin d’accès échoue lors de l’exécution de l’inventaire.
    - Lors de la définition du chemin d’accès, évitez les chemins d’accès généraux comme *c:* .** sinon un trop grand nombre de dossiers sont parcourus.

8. Veillez à affecter la valeur True à **Charger le contenu du fichier**. Ce paramètre active le suivi de contenu de fichier pour le chemin de fichier indiqué.

### <a name="configure-file-tracking-on-linux"></a>Configurer le suivi des fichiers sur Linux

Utilisez les étapes ci-dessous pour configurer le suivi des fichiers sur des ordinateurs Linux :

1. Sélectionnez **Modifier les paramètres** (symbole de l’engrenage).

2. Dans la page Configuration de l’espace de travail, sélectionnez **Fichiers Linux**, puis sélectionnez **+ ajouter** pour ajouter un nouveau fichier à suivre.

3. Sur la page **Ajouter un fichier Linux pour le Suivi des modifications** , entrez les informations du fichier ou du répertoire à suivre, puis sélectionnez **Enregistrer**. Le tableau suivant définit les propriétés que vous pouvez utiliser pour les informations.

    |Propriété  |Description  |
    |---------|---------|
    |activé     | True si le paramètre est appliqué, False sinon.        |
    |Item Name     | Nom convivial du fichier à suivre.        |
    |Groupe     | Nom de groupe pour le regroupement logique des fichiers.        |
    |Entrer le chemin     | Chemin dans lequel rechercher le fichier. Exemple : **/etc/*.conf**.       |
    |Type de chemin     | Type du chemin. Les valeurs possibles sont Fichier et Répertoire.        |
    |Récursivité     | True si la récursivité est utilisée pour rechercher l’élément à suivre, False sinon.        |
    |Utiliser sudo     | True pour utiliser sudo afin de rechercher l’élément, False sinon.         |
    |Liens     | Paramètre qui détermine comment traiter les liens symboliques quand vous parcourez des répertoires. Les valeurs possibles sont les suivantes :<br> Ignorer : ignore les liens symboliques et n’inclut pas les fichiers/répertoires référencés.<br>Suivre : suit les liens symboliques pendant la récursivité et inclut également les fichiers/répertoires référencés.<br>Gérer : suit les liens symboliques et autorise la modification du contenu retourné.<br>**Remarque :** L’option Gérer n’est pas recommandée, car elle ne prend pas en charge la récupération de contenu de fichier.    |
    |Charger le contenu du fichier | True pour charger le contenu du fichier d'après les modifications suivies, et False dans le cas contraire. |

4. Veillez à spécifier **True** pour **Charger le contenu du fichier**. Ce paramètre active le suivi de contenu de fichier pour le chemin de fichier indiqué.

   ![Ajouter le fichier Linux](./media/manage-change-tracking/add-linux-file.png)

## <a name="track-file-contents"></a>Effectuer le suivi du contenu du fichier

Le suivi de contenu de fichier vous permet de voir le contenu d’un fichier avant et après un changement suivi. La fonctionnalité enregistre le contenu du fichier dans un [compte de stockage](../../storage/common/storage-account-overview.md) chaque fois qu’un changement a lieu. Voici quelques règles à suivre pour le suivi du contenu de fichier :

* Un compte de stockage standard utilisant le modèle de déploiement Resource Manager est nécessaire pour stocker le contenu du fichier.
* Par défaut, les comptes de stockage acceptent les connexions des clients sur n’importe quel réseau. Si vous avez sécurisé votre compte de stockage pour autoriser uniquement certains types de trafic, vous devez modifier vos règles de configuration pour autoriser votre compte Automation à s’y connecter. Consultez [Configurer des pare-feu Stockage Azure et des réseaux virtuels](../../storage/common/storage-network-security.md).
* N’utilisez pas de comptes de stockage de modèles de déploiement Premium et Classic. Voir [À propos des comptes Stockage Azure](../../storage/common/storage-account-create.md).
* Vous pouvez connecter le compte de stockage à un seul compte Automation.
* Change Tracking et Inventory doit être activé dans votre compte Automation.

### <a name="enable-tracking-for-file-content-changes"></a>Activer le suivi des modifications de contenu de fichier

Effectuez les étapes suivantes pour activer le suivi des modifications apportées au contenu des fichiers :

1. Sélectionnez **Modifier les paramètres** (symbole de l’engrenage).

2. Sélectionnez **Contenu du fichier** puis sélectionnez **Lien**. Cette sélection ouvre la page **Ajouter l’emplacement du contenu pour le Suivi des modifications** .

   ![Ajouter l’emplacement du contenu](./media/manage-change-tracking/enable.png)

3. Sélectionnez l’abonnement et le compte de stockage à utiliser pour stocker le contenu du fichier.

5. Si vous voulez activer le suivi de contenu de fichier pour tous les fichiers suivis existants, sélectionnez **Activé** pour **Charger le contenu de fichier pour tous les paramètres**. Vous pourrez modifier ce paramètre pour chaque chemin de fichier ultérieurement.

   ![Définir le compte de stockage](./media/manage-change-tracking/storage-account.png)

6. Suivi des modifications et inventaire affiche le compte de stockage et les URI de signature d’accès partagé (SAS) quand il active le suivi des modifications de contenu de fichier. Les signatures expirent au bout de 365 jours, et vous pouvez les recréer en sélectionnant **Recréer**.

   ![Répertorier les clés de compte](./media/manage-change-tracking/account-keys.png)

### <a name="view-the-contents-of-a-tracked-file"></a>Afficher le contenu d’un fichier suivi

Une fois que Suivi des modifications et inventaire a détecté une modification apportée à un fichier suivi, vous pouvez afficher le contenu du fichier dans le volet Modifier les détails.  

![Lister les modifications](./media/manage-change-tracking/change-list.png)

1. Sur la page **Suivi des modifications** à partir de votre compte Automation, choisissez un fichier dans la liste des modifications, puis sélectionnez **Afficher les modifications de contenu de fichier** pour afficher le contenu du fichier. Le volet Modifier les détails présente les informations de fichier standard, avant et après, pour chaque propriété.

   ![Détails des modifications](./media/manage-change-tracking/change-details.png)

2. Le contenu du fichier est présenté dans une vue côte à côte. Vous pouvez sélectionner **Inline** pour avoir une vue inline des modifications.

## <a name="track-registry-keys"></a>Effectuer le suivi de clés de Registre

Utilisez les étapes suivantes pour configurer le suivi des clés de Registre sur des ordinateurs Windows :

1. Sur la page **Suivi des modifications** à partir de votre compte Automation, sélectionnez **Modifier les paramètres** (symbole de l’engrenage).

2. Dans la page Configuration de l’espace de travail, sélectionnez **Registre Windows**.

3. Sélectionnez **+ ajouter**  pour ajouter une nouvelle clé de Registre à suivre.

4. Sur la page **Ajouter le Registre Windows pour le Suivi des modifications** , entrez les informations relatives à la clé à suivre, puis sélectionnez **Enregistrer**. Le tableau suivant définit les propriétés que vous pouvez utiliser pour les informations. Lorsque vous spécifiez un chemin d’accès au registre, il doit s’agir de la clé et non d’une valeur.

    |Property  |Description  |
    |---------|---------|
    |activé     | True si un paramètre est appliqué, False sinon.        |
    |Item Name     | Nom convivial de la clé de Registre à suivre.        |
    |Groupe     | Nom de groupe pour le regroupement logique des clés de Registre.        |
    |Clé de Registre Windows   | Nom de clé avec chemin, par exemple `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders\Common Startup`.      |

## <a name="search-logs-for-change-records"></a>Rechercher des enregistrements de modifications dans les journaux

En matière d’enregistrements de modifications, vous pouvez effectuer différentes recherches dans les journaux Azure Monitor. Avec la page Change Tracking ouverte, cliquez sur **Log Analytics** pour ouvrir la page Journaux. Le tableau suivant fournit des exemples de recherche de modifications dans les journaux d’enregistrements.

|Requête  |Description  |
|---------|---------|
|`ConfigurationData`<br>&#124; `where ConfigDataType == "WindowsServices" and SvcStartupType == "Auto"`<br>&#124; `where SvcState == "Stopped"`<br>&#124; `summarize arg_max(TimeGenerated, *) by SoftwareName, Computer`         | Affiche les enregistrements d’inventaire les plus récents des services Microsoft qui ont été définis sur Auto, mais qui ont été signalés comme étant arrêtés. Les résultats se limitent à l’enregistrement le plus récent pour le nom de logiciel et l’ordinateur spécifiés.    |
|`ConfigurationChange`<br>&#124; `where ConfigChangeType == "Software" and ChangeCategory == "Removed"`<br>&#124; `order by TimeGenerated desc`|Affiche les enregistrements de modifications des logiciels supprimés.|

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les configurations d’étendue, consultez [Limiter l’étendue du déploiement de Change Tracking et Inventory](manage-scope-configurations.md).
* Si vous avez besoin de rechercher des journaux stockés dans des Journaux de Azure Monitor, consultez la [Recherche dans les journaux de Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).
* Si vous avez terminé avec les déploiements, consultez [Supprimer le Suivi des modifications et inventaire](remove-feature.md).
* Pour supprimer vos machines virtuelles de Change Tracking et Inventory, consultez [Supprimer des machines virtuelles de Change Tracking et Inventory](remove-vms-from-change-tracking.md).
* Pour détecter un problème lié aux modifications apportées à une machine virtuelle Azure, consultez [Résoudre les problèmes liés à Change Tracking and Inventory](../troubleshoot/change-tracking.md).
