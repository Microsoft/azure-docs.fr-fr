---
title: Identités managées dans Azure HDInsight
description: Offre une vue d’ensemble de l’implémentation d’identités managées dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 04/15/2020
ms.openlocfilehash: 4d9a5900990ea41788ced5f25690619fbde68d33
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91854985"
---
# <a name="managed-identities-in-azure-hdinsight"></a>Identités managées dans Azure HDInsight

Une identité managée est une identité inscrite dans Azure Active Directory (Azure AD) dont les informations d’identification sont gérées par Azure. Grâce aux identités managées, il n’est pas nécessaire d’inscrire des principaux de service dans Azure AD, ni de gérer des informations d’identification, par exemple des certificats.

Les identités managées sont utilisées dans Azure HDInsight pour accéder aux services de domaine Azure AD ou aux fichiers dans Azure Data Lake Storage Gen2 si nécessaire.

Il existe deux types d’identités managées : celles affectées par l’utilisateur et celles affectées par le système. Azure HDInsight prend uniquement en charge les identités managées attribuées par l’utilisateur. HDInsight ne prend pas en charge les identités managées affectées par le système. Une identité managée attribuée par l’utilisateur est créée en tant que ressource Azure autonome, laquelle peut ensuite être attribuée à une ou plusieurs instances de service Azure. En revanche, une identité managée affectée par le système est créée dans Azure AD, après quoi elle est automatiquement activée directement sur une instance de service Azure particulière. La durée de vie de cette identité managée affectée par le système est alors liée à celle de l’instance de service sur laquelle l’identité managée est activée.

## <a name="hdinsight-managed-identity-implementation"></a>Implémentation des identités managées dans HDInsight

Dans Azure HDInsight, les identités gérées sont uniquement utilisables par le service HDInsight pour les composants internes. Il n’existe actuellement aucune méthode prise en charge permettant de générer des jetons d’accès avec les identités managées installées sur des nœuds de cluster HDInsight pour l’accès à des services externes. Dans certains services Azure comme les machines virtuelles de calcul, les identités managées sont implémentées avec un point de terminaison que vous pouvez utiliser pour acquérir des jetons d’accès. Ce point de terminaison n’est pas disponible actuellement dans les nœuds HDInsight.

Si vous avez besoin de démarrer vos applications pour éviter de renseigner des secrets/mots de passe dans les travaux d’analytique (par exemple, les travaux SCALA), vous pouvez distribuer vos propres certificats sur les nœuds de cluster à l’aide d’actions de script, puis utiliser ce certificat pour acquérir un jeton d’accès (par exemple, pour accéder à Azure Key Vault).

## <a name="create-a-managed-identity"></a>Créer une identité managée

Vous pouvez créer des identités managées au moyen de l’une de ces méthodes :

* [Azure portal](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md)
* [Azure PowerShell](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md)
* [Azure Resource Manager](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-arm.md)
* [Azure CLI](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md)

Les autres étapes à suivre pour configurer l’identité managée varient en fonction du scénario d’utilisation.

## <a name="managed-identity-scenarios-in-azure-hdinsight"></a>Scénarios d’utilisation des identités managées dans Azure HDInsight

Les identités managées sont utilisées dans Azure HDInsight dans de multiples scénarios. Consultez les documents connexes pour obtenir des instructions d’installation et de configuration détaillées :

* [Azure Data Lake Storage Gen2](hdinsight-hadoop-use-data-lake-storage-gen2-portal.md#create-a-user-assigned-managed-identity)
* [Pack Sécurité Entreprise](domain-joined/apache-domain-joined-configure-using-azure-adds.md#create-and-authorize-a-managed-identity)
* [Chiffrement de disque par clé gérée par le client](disk-encryption.md)

HDInsight renouvelle automatiquement les certificats pour les identités managées utilisées dans le cadre de ces scénarios. Il existe toutefois une limitation : lorsque plusieurs identités managées différentes sont employées pour des clusters durables, le renouvellement de certificat risque de ne pas fonctionner comme prévu pour toutes les identités managées. En raison de cette limitation, si vous envisagez d’utiliser des clusters durables (par exemple, plus de 60 jours), nous vous recommandons d’avoir recours à la même identité managée pour tous les scénarios ci-dessus. 

Si vous avez déjà créé un cluster durable avec plusieurs identités managées différentes et que vous rencontrez l’un des problèmes suivants :
 * Dans les clusters ESP, les services de cluster commencent à présenter des échecs ou des scale-up. D’autres opérations échouent sur des erreurs d’authentification.
 * Dans les clusters ESP, le certificat LDAPS AAD-DS n’est pas mis à jour automatiquement lorsqu’il est modifié. Par conséquent, la synchronisation LDAP et les scale-up commencent à présenter des échecs.
 * L’accès MSI à ADLS Gen2 commence à présenter des échecs.
 * Les clés de chiffrement ne peuvent pas être renouvelées dans le scénario CMK.
Vous devez alors attribuer les rôles et autorisations nécessaires aux scénarios ci-dessus à toutes les identités managées utilisées dans le cluster. Par exemple, si vous avez employé des identités managées différentes pour des clusters ADLS Gen2 et ESP, les rôles « Propriétaire des données d’objet blob de stockage » et « Contributeur des services de domaine HDInsight » doivent leur être affectés pour éviter ces problèmes.

## <a name="faq"></a>Questions fréquentes (FAQ)

### <a name="what-happens-if-i-delete-the-managed-identity-after-the-cluster-creation"></a>Que se passe-t-il si je supprime l’identité managée après la création du cluster ?

Votre cluster rencontre des problèmes lorsque l’identité managée est nécessaire. Il n’existe actuellement aucun moyen de mettre à jour ou de modifier une identité managée après la création du cluster. Nous vous recommandons donc de veiller à ce que l’identité managée ne soit pas supprimée pendant l’exécution du cluster. Vous pouvez également recréer le cluster et affecter une nouvelle identité managée.

## <a name="next-steps"></a>Étapes suivantes

* [Que sont les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/overview.md)
