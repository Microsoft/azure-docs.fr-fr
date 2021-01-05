---
title: Se connecter aux ressources de l’espace de travail Azure Synapse Analytics Studio à partir d’un réseau restreint
description: Cet article vous apprendra à vous connecter aux ressources de votre espace de travail à partir d'un réseau restreint
author: xujxu
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 10/25/2020
ms.author: xujiang1
ms.reviewer: jrasnick
ms.openlocfilehash: 2e96cbf0c1464e27b0a384e8a813118056103b91
ms.sourcegitcommit: 192f9233ba42e3cdda2794f4307e6620adba3ff2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96296682"
---
# <a name="connect-to-workspace-resources-from-a-restricted-network"></a>Se connecter aux ressources de l’espace de travail à partir d’un réseau restreint

Supposons que vous êtes un administrateur informatique qui gère le réseau restreint de votre organisation. Vous devez activer la connexion réseau entre Azure Synapse Analytics Studio et une station de travail au sein de ce réseau restreint. Cet article vous montre comment procéder.

## <a name="prerequisites"></a>Prérequis

* **Abonnement Azure** : Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.
* **Espace de travail Azure Synapse Analytics** : Vous pouvez en créer un à partir d’Azure Synapse Analytics. Vous avez besoin du nom de l’espace de travail à l’étape 4.
* **Réseau restreint** : L’administrateur informatique gère le réseau restreint pour l’organisation, et il est autorisé à configurer la stratégie réseau. Vous avez besoin du nom du réseau virtuel et de son sous-réseau à l’étape 3.


## <a name="step-1-add-network-outbound-security-rules-to-the-restricted-network"></a>Étape 1 : Ajouter des règles de sécurité réseau sortantes au réseau restreint

Vous devez ajouter quatre règles de sécurité de trafic sortant du réseau avec quatre balises de service. 
* AzureResourceManager
* AzureFrontDoor.Frontend
* AzureActiveDirectory
* AzureMonitor (ce type de règle est facultatif, ajoutez-le uniquement lorsque vous souhaitez partager les données avec Microsoft.)

La capture d’écran suivante montre les détails de la règle de trafic sortant Azure Resource Manager.

![Capture d’écran des détails de l’étiquette de service Azure Resource Manager.](./media/how-to-connect-to-workspace-from-restricted-network/arm-servicetag.png)

Lorsque vous créez les trois autres règles, remplacez la valeur de **Destination service tag** par **AzureFrontDoor.Frontend**, **AzureActiveDirectory** ou **AzureMonitor** dans la liste.

Pour plus d’informations, consultez [Vue d’ensemble des étiquettes de service Azure](/azure/virtual-network/service-tags-overview).

## <a name="step-2-create-private-link-hubs"></a>Étape 2 : Créer des hubs de liaison privée

Créez ensuite des hubs de liaison privée à partir du portail Azure. Pour trouver ces éléments dans le portail, recherchez *Azure Synapse Analytics (hubs de liaison privée)* , puis fournissez les informations requises pour les créer. 

![Capture d’écran de la création d’un hub de liaison privée Synapse.](./media/how-to-connect-to-workspace-from-restricted-network/private-links.png)

## <a name="step-3-create-a-private-endpoint-for-your-synapse-studio"></a>Étape 3 : Créer un point de terminaison privé pour Synapse Studio

Pour accéder à Azure Synapse Analytics Studio, vous devez créer un point de terminaison privé à partir du portail Azure. Pour trouver ce point dans le portail, recherchez *Liaison privée*. Dans le **Centre de liaisons privées**, sélectionnez **Créer un point de terminaison privé**, puis fournissez les informations requises pour le créer. 

> [!Note]
> Assurez-vous que la valeur **Région** est identique à celle de votre espace de travail Azure Synapse Analytics.

![Capture d’écran montrant l’onglet des informations de la base de la page Créer un point de terminaison privé.](./media/how-to-connect-to-workspace-from-restricted-network/plink-endpoint-1.png)

Dans l’onglet **Ressource**, choisissez le hub de liaison privée que vous avez créé à l’étape 2.

![Capture d’écran montrant l’onglet Ressource de la page Créer un point de terminaison privé.](./media/how-to-connect-to-workspace-from-restricted-network/plink-endpoint-2.png)

Dans l’onglet **Configuration** : 
* Pour **Réseau virtuel**, choisissez le nom de réseau virtuel restreint.
* Pour **Sous-réseau**, choisissez le sous-réseau du réseau virtuel restreint. 
* Pour **Intégrer à une zone DNS privée**, sélectionnez **Oui**.

![Capture d’écran de l’onglet Configuration de la fenêtre Créer un point de terminaison privé.](./media/how-to-connect-to-workspace-from-restricted-network/plink-endpoint-3.png)

Une fois le point d'extrémité du lien privé créé, vous pouvez accéder à la page de connexion de l'outil web Azure Synapse Analytics Studio. Toutefois, vous ne pouvez pas encore accéder aux ressources dans votre espace de travail. Pour cela, vous devez effectuer l’étape suivante.

## <a name="step-4-create-private-endpoints-for-your-workspace-resource"></a>Étape 4 : Créer des points de terminaison privé pour votre ressource d’espace de travail

Pour accéder aux ressources dans votre ressource d’espace de travail Azure Synapse Analytics Studio, vous devez créer les éléments suivants :

- Au moins un point de terminaison de liaison privée avec un type **Dev** défini sur **Target sub-resource**.
- Deux autres points de terminaison de liaison privée facultatifs avec des types **Sql** ou **SqlOnDemand**, en fonction des ressources de l’espace de travail auxquelles vous souhaitez accéder.

Leur création est similaire à la celle du point de terminaison à l’étape précédente.  

Dans l’onglet **Ressource** :

* pour **Type de ressource**, sélectionnez **Microsoft.Synapse/workspaces**.
* Pour **Ressource**, sélectionnez le nom de l’espace de travail que vous avez créé précédemment.
* Pour **Sous-ressource cible**, sélectionnez le type de point de terminaison :
  * **Sql** est destiné à l’exécution de la requête SQL dans le pool SQL.
  * **SqlOnDemand** est destiné à l’exécution de requêtes SQL intégrées.
  * **Dev** est destiné à l’accès aux autres ressources dans les espaces de travail Azure Synapse Analytics Studio. Vous devez créer au moins un point de terminaison de la liaison privée de ce type.

![Capture d’écran de l’espace de travail montrant l’onglet Ressource de la page Créer un point de terminaison privé.](./media/how-to-connect-to-workspace-from-restricted-network/plinks-endpoint-ws-1.png)


## <a name="step-5-create-private-endpoints-for-workspace-linked-storage"></a>Étape 5 : Créer des points de terminaison privé pour le stockage lié de l’espace de travail

Pour accéder au stockage lié à l’explorateur de stockage dans l’espace de travail Azure Synapse Analytics Studio, vous devez créer un point de terminaison privé. Les étapes de ce processus sont similaires à celles de l’étape 3. 

Dans l’onglet **Ressource** :
* Pour **Type de ressource**, sélectionnez **Microsoft.Synapse/storageAccounts**.
* Pour **Ressource**, sélectionnez le nom du compte de stockage que vous avez créé précédemment.
* Pour **Sous-ressource cible**, sélectionnez le type de point de terminaison :
  * **blob** est destiné à Stockage Blob Azure.
  * **dfs** pour Azure Data Lake Storage Gen2.

![Capture d’écran montrant le stockage dans l’onglet Ressource de la page Créer un point de terminaison privé.](./media/how-to-connect-to-workspace-from-restricted-network/plink-endpoint-storage.png)

Vous pouvez maintenant accéder à la ressource de stockage liée. Au sein de votre réseau virtuel, dans votre espace de travail Azure Synapse Analytics Studio, vous pouvez utiliser l’explorateur de stockage pour accéder à la ressource de stockage liée.

Vous pouvez activer un réseau virtuel managé pour votre espace de travail, comme illustré dans cette capture d’écran :

![Capture d’écran de la création d’un espace de travail Synapse, avec l’option Activer le réseau virtuel managé en surbrillance.](./media/how-to-connect-to-workspace-from-restricted-network/ws-network-config.png)

Si vous souhaitez que votre notebook accède aux ressources de stockage liées sous un compte de stockage, ajoutez des points de terminaison privés managés sous Azure Synapse Analytics Studio. Le nom du compte de stockage doit être celui auquel votre notebook doit accéder. Pour plus d’informations, consultez [Créer un point de terminaison privé managé pour votre source de données](./how-to-create-managed-private-endpoints.md).

Une fois ce point de terminaison créé, l’état d’approbation affiche l’état **En attente**. Demandez l’approbation du propriétaire de ce compte de stockage, dans l’onglet **Connexions des points de terminaison privés** de ce compte de stockage dans le portail Azure. Une fois approuvé, votre notebook peut accéder aux ressources de stockage liées sous ce compte de stockage.

Tout est défini à présent. Vous pouvez accéder à votre ressource de l’espace de travail Azure Synapse Analytics Studio.

## <a name="appendix-dns-registration-for-private-endpoint"></a>Annexe : Inscription DNS pour un point de terminaison privé

Si l’option « Intégrer à une zone DNS privée » n’est pas activée lors de la création du point de terminaison privé comme dans la capture d’écran ci-dessous, vous devez créer la « **zone DNS privée** » pour chacun de vos points de terminaison privés.
![Capture d’écran de la création d’une zone DNS 1 privée Synapse.](./media/how-to-connect-to-workspace-from-restricted-network/pdns-zone-1.png)

Pour rechercher la **zone DNS privée** dans le portail, recherchez *Zone DNS privée*. Dans la **zone DNS privée**, fournissez les informations requises ci-dessous pour créer la zone.

* Pour **Nom**, entrez le nom dédié de la zone DNS privée pour un point de terminaison privé spécifique, comme indiqué ci-dessous :
  * **`privatelink.azuresynapse.net`** s’applique au point de terminaison privé pour l’accès à la passerelle Azure Synapse Analytics Studio. Consultez ce type de création de point de terminaison privé à l’étape 3.
  * **`privatelink.sql.azuresynapse.net`** s’applique à ce type de point de terminaison privé pour l’exécution d’une requête SQL dans le pool SQL et le pool intégré. Consultez la création d’un point de terminaison à l’étape 4.
  * **`privatelink.dev.azuresynapse.net`** s’applique à ce type de point de terminaison privé pour l’accès aux autres ressources dans les espaces de travail Azure Synapse Analytics Studio. Consultez ce type de création de point de terminaison privé à l’étape 4.
  * **`privatelink.dfs.core.windows.net`** s’applique au point de terminaison privé pour l’accès à l’espace de travail lié Azure Data Lake Storage Gen2. Consultez ce type de création de point de terminaison privé à l’étape 5.
  * **`privatelink.blob.core.windows.net`** s’applique au point de terminaison privé pour l’accès à l’espace de travail lié Azure Blob Storage. Consultez ce type de création de point de terminaison privé à l’étape 5.

![Capture d’écran de la création d’une zone DNS 2 privée Synapse.](./media/how-to-connect-to-workspace-from-restricted-network/pdns-zone-2.png)

Une fois la **zone DNS privée** créée, entrez la zone DNS privée créée, puis sélectionnez les **liens de réseau virtuel** pour ajouter le lien à votre réseau virtuel. 

![Capture d’écran de la création d’une zone DNS 3 privée Synapse.](./media/how-to-connect-to-workspace-from-restricted-network/pdns-zone-3.png)

Remplissez les champs obligatoires, comme indiqué ci-dessous :
* Pour **Nom du lien**, entrez le nom du lien.
* Pour **Réseau virtuel**, sélectionnez votre réseau virtuel.

![Capture d’écran de la création d’une zone DNS 4 privée Synapse.](./media/how-to-connect-to-workspace-from-restricted-network/pdns-zone-4.png)

Une fois le lien de réseau virtuel ajouté, vous devez ajouter le jeu d’enregistrements DNS dans la **zone DNS privée** que vous avez créée précédemment.

* Pour **Nom**, entrez les chaînes de noms dédiées pour différents points de terminaison privés : 
  * **web** s’applique au point de terminaison privé pour l’accès à la passerelle Azure Synapse Analytics Studio.
  * « ***YourWorkSpaceName**_ » s’applique au point de terminaison privé pour l’exécution d’une requête SQL dans le pool SQL, ainsi qu’au point de terminaison privé pour l’accès aux autres ressources dans les espaces de travail Azure Synapse Analytics Studio. _ «***  YourWorkSpaceName*-ondemand** » s’applique au point de terminaison privé pour l’exécution d’une requête SQL dans le pool intégré.
* Pour **Type**, sélectionnez le type d’enregistrement DNS **A** uniquement. 
* Pour **Adresse IP**, entrez l’adresse IP correspondante de chaque point de terminaison privé. Vous pouvez récupérer l’adresse IP dans **Interface réseau** à partir de la vue d’ensemble de votre point de terminaison privé.

![Capture d’écran de la création d’une zone DNS 5 privée Synapse.](./media/how-to-connect-to-workspace-from-restricted-network/pdns-zone-5.png)


## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur le [Réseau virtuel d’espace de travail managé](./synapse-workspace-managed-vnet.md).

En savoir plus sur les [Points de terminaison privés managés](./synapse-workspace-managed-private-endpoints.md).
