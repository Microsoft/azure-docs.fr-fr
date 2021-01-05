---
title: Gérer le regroupement d’inventaire Azure Automation à partir de machines virtuelles | Microsoft Docs
description: Cet article explique comment gérer le regroupement d'inventaire à partir de machines virtuelles.
services: automation
ms.subservice: change-inventory-management
keywords: inventaire, automatisation, suivi des modifications
ms.date: 10/14/2020
ms.topic: conceptual
ms.openlocfilehash: 2acf22ba76acdfa6152ef8966f4000aa325a9e91
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92209030"
---
# <a name="manage-inventory-collection-from-vms"></a>Gérer une collecte de données d’inventaire à partir de machines virtuelles

Vous pouvez activer le suivi d’inventaire pour une machine virtuelle Azure à partir de la page des ressources de celle-ci. Vous pouvez collecter et afficher les informations d’inventaire suivantes sur vos ordinateurs :

- mises à jour Windows, applications Windows, services, fichiers et clés de Registre ;
- logiciels (packages), démons et fichiers Linux.

La fonctionnalité Suivi des modifications et inventaire d’Azure Automation fournit une interface utilisateur basée sur le navigateur pour configurer le regroupement d'inventaire.

## <a name="before-you-begin"></a>Avant de commencer

Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit](https://azure.microsoft.com/free/).

Cet article suppose que vous disposez d’une machine virtuelle pour laquelle activer la fonctionnalité Suivi des modifications et inventaire. Si vous n’avez pas de machine virtuelle Azure, vous pouvez [Créer une machine virtuelle](../../virtual-machines/windows/quick-create-portal.md).

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

## <a name="enable-inventory-collection-from-the-vm-resource-page"></a>Activer le regroupement d'inventaire à partir de la page de ressources de la machine virtuelle

1. Dans le volet de gauche du portail Azure, sélectionnez **Machines virtuelles**.
2. Dans la liste des machines virtuelles, sélectionnez une machine.
3. Dans le menu **Ressource**, sous **Opérations**, sélectionnez **Inventaire**.
4. Sélectionnez un espace de travail Log Analytics pour stocker vos journaux de données.
    Si aucun espace de travail n’est disponible pour cette région, vous êtes invité à créer un compte Automation et un espace de travail par défaut.
5. Pour commencer l’activation de votre ordinateur, sélectionnez **Activer**.

   ![Afficher les options d’intégration](./media/manage-inventory-vms/inventory-onboarding-options.png)

    Une barre d’état vous informe que la fonctionnalité Suivi des modifications et inventaire est en cours d’activation. Ce processus peut prendre jusqu’à 15 minutes. Pendant ce temps, vous pouvez fermer la fenêtre ou la laisser ouverte afin qu’elle vous tienne informé de l’activation de la fonctionnalité. Vous pouvez surveiller l’état du déploiement dans le volet des notifications.

   ![Afficher l’inventaire](./media/manage-inventory-vms/inventory-onboarded.png)

Une fois le déploiement terminé, la barre d’état disparaît. Le système est toujours en cours de collection des données d’inventaire ; ces données peuvent ne pas être encore visibles. Une collecte complète des données peut prendre 24 heures.

## <a name="configure-your-inventory-settings"></a>Configurer les paramètres d’inventaire

Par défaut, les logiciels, les services Windows et les démons Linux sont configurés pour la collection. Pour collecter l’inventaire des fichiers et registres Windows, configurez les paramètres de collection d’inventaire.

1. En haut de la page Inventaire, cliquez sur **Modifier les paramètres**.
2. Pour ajouter un nouveau paramètre de collection, accédez à la catégorie de paramètres concernée en sélectionnant l’onglet **Registre Windows**, **Fichiers Windows** ou **Fichiers Linux**.
3. Sélectionnez la catégorie appropriée et cliquez sur **Ajouter** en haut de la page.

Les sections suivantes fournissent des informations sur chaque propriété qui peut être configurée pour les différentes catégories.

### <a name="windows-registry"></a>Registre Windows

|Propriété  |Description  |
|---------|---------|
|activé     | Détermine si le paramètre est appliqué.        |
|Item Name     | Nom convivial du fichier à suivre.        |
|Groupe     | Nom de groupe pour le regroupement logique des fichiers.        |
|Clé de Registre Windows   | Chemin d’accès pour rechercher le fichier. Exemple : « HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders\Common Startup »      |

### <a name="windows-files"></a>Fichiers Windows

|Propriété  |Description  |
|---------|---------|
|activé     | True si le paramètre est appliqué, False sinon.        |
|Item Name     | Nom convivial du fichier à suivre.        |
|Groupe     | Nom de groupe pour le regroupement logique des fichiers.       |
|Entrer le chemin     | Chemin dans lequel rechercher le fichier. Exemple : **C:\temp\monfichier.txt**.

### <a name="linux-files"></a>Fichiers Linux

|Propriété  |Description  |
|---------|---------|
|activé     | True si le paramètre est appliqué, False sinon.        |
|Item Name     | Nom convivial du fichier à suivre.        |
|Groupe     | Nom de groupe pour le regroupement logique des fichiers.        |
|Entrer le chemin     | Chemin dans lequel rechercher le fichier. Exemple : **/etc/*.conf**.       |
|Type de chemin     | Type d’élément à suivre. Valeurs possibles : Fichier et Répertoire.        |
|Récursivité     | True si la récursivité est utilisée pour rechercher l’élément à suivre, False sinon.        |
|Utiliser sudo     | True si sudo est utilisé pour rechercher l’élément, False sinon.         |
|Liens     | Valeur indiquant le traitement des liens symboliques lors du parcours des répertoires. Les valeurs possibles sont les suivantes : <br> Ignorer : ignore les liens symboliques et n’inclut pas les fichiers/répertoires référencés.<br>Suivre : suit les liens symboliques pendant la récursivité et inclut également les fichiers/répertoires référencés.<br>Gérer : suit les liens symboliques et autorise la modification du traitement du contenu retourné.      |

## <a name="manage-machine-groups"></a>Gérer des groupes de machines

L’inventaire vous permet de créer et de visualiser des groupes de machines dans les journaux Azure Monitor. Les groupes de machines sont des collections de machines définies par une requête dans les journaux Azure Monitor.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

Pour visualiser vos groupes de machines, sélectionnez l’onglet **Groupes de machines** sur la page Inventaire.

![Affichage des groupes de machines sur la page d’inventaire](./media/manage-inventory-vms/inventory-machine-groups.png)

La sélection d’un groupe de machines dans la liste affiche la page Groupes de machines. Cette page affiche des détails sur le groupe de machines. Ces détails comprennent la requête de journal Azure Monitor utilisée pour définir le groupe. La partie inférieure de la page présente une liste paginée des machines qui font partie de ce groupe.

![Affichage de la page d’un groupe de machines](./media/manage-inventory-vms/machine-group-page.png)

Cliquez sur **+ Cloner** pour cloner le groupe de machines. Vous devez donner au groupe un nouveau nom, ainsi qu’un alias. Vous pouvez modifier la définition à ce stade. Après avoir modifié la requête, cliquez sur **Valider la requête** pour afficher un aperçu des machines sélectionnées. Lorsque le groupe de machines vous convient, cliquez sur **Créer** pour le créer.

Si vous souhaitez créer un autre groupe de machines, cliquez sur **+ Créer un groupe de machines**. Ce bouton ouvre la page **Créer un groupe de machines** permettant de définir le nouveau groupe. Cliquez sur **Créer** pour créer le groupe.

![Création d’un groupe de machines](./media/manage-inventory-vms/create-new-group.png)

## <a name="disconnect-your-vm-from-management"></a>Déconnecter votre machine virtuelle de la gestion

Pour supprimer votre machine virtuelle de la gestion de la fonctionnalité Suivi des modifications et inventaire :

1. Dans le volet gauche du portail Azure, sélectionnez **Log Analytics**, puis sélectionnez l’espace de travail que vous avez utilisé lors de l’activation de votre machine virtuelle pour la fonctionnalité Suivi des modifications et inventaire.
2. Dans la page **Log Analytics**, ouvrez le menu **Ressource**.
3. Sélectionnez **Machines virtuelles** sous **Sources de données de l’espace de travail**.
4. Dans la liste, sélectionnez la machine virtuelle à déconnecter. Une coche verte s’affiche en regard du texte indiquant **Cet espace de travail** dans la colonne **Connexion OMS**, sur la machine.

   >[!NOTE]
   >Pour faire référence à Operations Management Suite (OMS), nous parlons maintenant de journaux Azure Monitor.

5. En haut de la page suivante, cliquez sur **Déconnecter**.
6. Dans la fenêtre de confirmation, cliquez sur **Oui** pour déconnecter la machine de la gestion.

>[!NOTE]
>Les machines s’affichent toujours même après avoir été désinscrites, car nous faisons état de toutes les machines inventoriées au cours des dernières 24 heures. Une fois la machine déconnectée, vous devez attendre 24 heures avant qu’elle ne soit plus listée.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’utilisation de la fonctionnalité, consultez [Gérer Suivi des modifications et inventaire](manage-change-tracking.md).
* Pour en savoir plus sur le suivi des modifications logicielles, consultez [Suivre les modifications logicielles dans votre environnement avec la fonctionnalité Suivi des modifications](overview.md).
* Pour résoudre des problèmes généraux liés à la fonctionnalité, consultez [Résoudre les problèmes rencontrés avec la fonctionnalité Suivi des modifications et inventaire](../troubleshoot/change-tracking.md).
