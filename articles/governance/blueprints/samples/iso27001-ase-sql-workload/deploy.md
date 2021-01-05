---
title: Déployer l’exemple de blueprint Charge de travail ASE/SQL ISO 27001
description: Étapes de déploiement de l’exemple de blueprint Charge de travail App Service Environment/SQL Database ISO 27001 et informations détaillées sur les paramètres d’artefact de blueprint.
ms.date: 11/02/2020
ms.topic: sample
ms.openlocfilehash: 2b05015c05799625bf720096e70551eae7ff4d01
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420391"
---
# <a name="deploy-the-iso-27001-app-service-environmentsql-database-workload-blueprint-sample"></a>Déployer l’exemple de blueprint Charge de travail App Service Environment/SQL Database ISO 27001

Pour déployer l’exemple de blueprint Charge de travail App Service Environment/SQL Database ISO 27001 pour Azure Blueprints, vous devez effectuer les étapes suivantes :

> [!div class="checklist"]
> - Déployer l’exemple de blueprint [Services partagés ISO 27001](../iso27001-shared/index.md)
> - Créer un blueprint à partir de l’exemple
> - Marquer la copie de l’exemple en tant que **Publié**
> - Affecter votre copie du blueprint à un abonnement existant

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free) avant de commencer.

## <a name="deploy-the-iso-27001-shared-services-blueprint-sample"></a>Déployer l’exemple de blueprint Services partagés ISO 27001

Avant de déployer cet exemple de blueprint, vous devez déployer celui intitulé [Services partagés ISO 27001](../iso27001-shared/index.md) sur l’abonnement cible. Si vous n’effectuez pas cette étape préalable, cet exemple de blueprint ne dispose pas des dépendances d’infrastructure nécessaires et échoue durant le déploiement.

> [!IMPORTANT]
> Cet exemple de blueprint doit être affecté dans le même abonnement que [Services partagés ISO 27001](../iso27001-shared/index.md).

## <a name="create-blueprint-from-sample"></a>Créer un blueprint à partir de l’exemple

Commencez par implémenter l’exemple de blueprint. Pour cela, créez un blueprint dans votre environnement en vous servant de l’exemple comme point de départ.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Dans la page **Démarrage** à gauche, sélectionnez le bouton **Créer** sous _Créer un blueprint_.

1. Recherchez l’exemple de blueprint **ISO 27001 : Charge de travail ASE/SQL** sous _Autres exemples_ et sélectionnez **Utiliser cet exemple**.

1. Entrez les _Fonctions de base_ de l’exemple de blueprint :

   - **Nom du blueprint**  : entrez le nom de votre copie de l’exemple de blueprint Charge de travail ASE/SQL ISO 27001.
   - **Emplacement de la définition**  : utilisez les points de suspension et sélectionnez le groupe d’administration dans lequel vous souhaitez enregistrer votre copie de l’exemple.

1. Sélectionnez l’onglet _Artefacts_ dans le haut de la page ou **Suivant : Artefacts** dans le bas de la page.

1. Passez en revue la liste des artefacts qui composent l’exemple de blueprint. De nombreux artefacts ont des paramètres que nous définirons plus tard. Sélectionnez **Enregistrer comme brouillon** lorsque vous avez terminé de passer en revue l’exemple de blueprint.

## <a name="publish-the-sample-copy"></a>Publier la copie de l’exemple

Votre copie de l’exemple de blueprint est à présent créée dans votre environnement. Elle est créée en mode **Brouillon** et doit être **publiée** avant de pouvoir être attribuée et déployée. Vous pouvez personnaliser la copie de l’exemple de blueprint en fonction de votre environnement et de vos besoins, mais de telles modifications peuvent l’éloigner de la norme ISO 27001.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour rechercher votre copie de l’exemple de blueprint, puis sélectionnez-la.

1. Sélectionnez **Publier le blueprint** dans le haut de la page. Dans la nouvelle page à droite, indiquez la **Version** de votre copie de l’exemple de blueprint. Cette propriété est utile si vous prévoyez d’effectuer une modification ultérieurement. Dans **Notes de changement** , indiquez par exemple « Première version publiée à partir de l’exemple de blueprint ISO 27001 ». Sélectionnez ensuite **Publier** en bas de la page.

## <a name="assign-the-sample-copy"></a>Affecter la copie de l’exemple

Une fois que la copie de l’exemple de blueprint a été **publiée** , elle peut être affectée à un abonnement dans le groupe d’administration où elle a été enregistrée. C’est à cette étape que les paramètres sont fournis pour que chaque déploiement de la copie de l’exemple de blueprint soit unique.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour rechercher votre copie de l’exemple de blueprint, puis sélectionnez-la.

1. Sélectionnez **Affecter le blueprint** dans le haut de la page.

1. Indiquez les valeurs des paramètres pour l’affectation du blueprint :

   - Concepts de base

     - **Abonnements**  : Sélectionnez un ou plusieurs des abonnements qui font partie du groupe d’administration où vous avez enregistré votre copie de l’exemple de blueprint. Si vous sélectionnez plusieurs abonnements, une affectation est créée pour chacun d’eux à l’aide des paramètres saisis.
     - **Nom de l’affectation**  : Le nom est prérempli en fonction du nom du blueprint.
       Changez-le si nécessaire ou laissez-le tel quel.
     - **Emplacement** : Sélectionnez une région dans laquelle créer l’identité managée. Azure Blueprint utilise cette identité managée pour déployer tous les artefacts dans le blueprint affecté. Pour en savoir plus, consultez [Identités managées pour les ressources Azure](../../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Version de définition du blueprint**  : Choisissez une version **publiée** de votre copie de l’exemple de blueprint.

   - Verrouiller l'affectation

     Sélectionnez le paramètre de verrouillage de blueprint pour votre environnement. Pour plus d’informations, consultez [Verrouillage des ressources des blueprints](../../concepts/resource-locking.md).

   - Identité managée

     Conservez l’option par défaut, _Affecté(e) par le système_ , pour l’identité managée.

   - Paramètres de blueprint

     Les paramètres définis dans cette section sont utilisés par de nombreux artefacts dans la définition du blueprint à des fins de cohérence.

     - **Nom de l’organisation**  : entrez un nom court pour votre organisation. Cette propriété est principalement utilisée pour le nommage des ressources.
     - **ID d’abonnement Services partagés**  : ID d’abonnement auquel l’exemple de blueprint [Services partagés ISO 27001](../iso27001-shared/index.md) est affecté.
     - **Préfixe d’adresse de sous-réseau par défaut**  : notation CIDR du sous-réseau par défaut de réseau virtuel.
       La valeur par défaut est  _10.1.0.0/16_.
     - **Emplacement de la charge de travail**  : détermine l’emplacement dans lequel les artefacts sont déployés. Tous les services ne sont pas disponibles dans tous les emplacements. Les artefacts déployant de tels services fournissent une option de paramètre qui permet de définir l’emplacement dans lequel les artefacts sont déployés.

   - Paramètres d'artefact

     Les paramètres définis dans cette section s’appliquent à l’artefact sous lequel elle est définie. Ces paramètres sont des [paramètres dynamiques](../../concepts/parameters.md#dynamic-parameters) puisqu’ils sont définis lors de l’affectation du blueprint. Pour obtenir la liste complète des paramètres d’artefact et leur description, consultez le [tableau des paramètres d’artefact](#artifact-parameters-table).

1. Une fois tous les paramètres entrés, sélectionnez **Affecter** au bas de la page. L’affectation du blueprint est créée et le déploiement de l’artefact démarre. Le déploiement prend environ une heure. Pour vérifier l’état du déploiement, ouvrez l’affectation du blueprint.

> [!WARNING]
> Le service Azure Blueprints et les exemples de blueprint intégrés sont **gratuits**. Les ressources Azure sont [facturées par produit](https://azure.microsoft.com/pricing/). Utilisez la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour estimer le coût d’exécution des ressources déployées par cet exemple de blueprint.

## <a name="artifact-parameters-table"></a>Tableau des paramètres d’artefact

Le tableau suivant fournit la liste des paramètres d’artefact de blueprint :

|Nom de l’artefact|Type d’artefact|Nom du paramètre|Description|
|-|-|-|-|
|Groupe de ressources Log Analytics|Resource group|Nom|**Verrouillé**  : concatène le **nom de l’organisation** et `-workload-log-rg` pour rendre le groupe de ressources unique.|
|Groupe de ressources Log Analytics|Resource group|Emplacement|**Verrouillé**  : utilise le paramètre du blueprint.|
|Modèle Log Analytics|Modèle Resource Manager|Niveau de service|Définit le niveau de l’espace de travail Log Analytics. La valeur par défaut est _PerNode_.|
|Modèle Log Analytics|Modèle Resource Manager|Conservation du journal en jours|Conservation des données en jours. La valeur par défaut est  _365_.|
|Modèle Log Analytics|Modèle Resource Manager|Emplacement|Région utilisée pour créer l’espace de travail Log Analytics. La valeur par défaut est _USA Ouest 2_.|
|Groupe de ressources réseau|Resource group|Nom|**Verrouillé**  : concatène le **nom de l’organisation** et `-workload-net-rg` pour rendre le groupe de ressources unique.|
|Groupe de ressources réseau|Resource group|Emplacement|**Verrouillé**  : utilise le paramètre du blueprint.|
|Modèle Groupe de sécurité réseau|Modèle Resource Manager|Conservation du journal en jours|Conservation des données en jours. La valeur par défaut est  _365_.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|Adresse IP privée de pare-feu Azure|Configure l’adresse IP privée du [pare-feu Azure](../../../../firewall/overview.md). Doit faire partie de la notation CIDR définie dans le paramètre d’artefact _ISO 27001 : Services partagés_**Préfixe d’adresse de sous-réseau de pare-feu Azure**. La valeur par défaut est  _10.0.4.4_.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|ID d’abonnement Services partagés|Valeur utilisée pour activer le peering de réseau virtuel entre une charge de travail et les services partagés.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|Préfixe d’adresse de réseau virtuel|Notation CIDR du réseau virtuel. La valeur par défaut est  _10.1.0.0/16_.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|Préfixe d’adresse de sous-réseau par défaut|notation CIDR du sous-réseau par défaut de réseau virtuel. La valeur par défaut est  _10.1.0.0/16_.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|Adresse IP ADDS|Adresse IP de la première machine virtuelle ADDS. Cette valeur sert de DNS VNET personnalisé.|
|Groupe de ressources Key Vault|Resource group|Nom|**Verrouillé**  : concatène le **nom de l’organisation** et `-workload-kv-rg` pour rendre le groupe de ressources unique.|
|Groupe de ressources Key Vault|Resource group|Emplacement|**Verrouillé**  : utilise le paramètre du blueprint.|
|Modèle Key Vault|Modèle Resource Manager|ID d’objet AAD|Identificateur d’objet AAD du compte qui nécessite l’accès à l’instance Key Vault. N’a pas de valeur par défaut et ne peut pas être vide. Pour localiser cette valeur à partir du portail Azure, recherchez et sélectionnez « Utilisateurs » sous _Services_. Utilisez la zone _Nom_ pour filtrer le nom du compte, puis sélectionnez ce compte. Dans la page _Profil utilisateur_ , sélectionnez l’icône « Cliquez pour copier » à côté de l’ _ID d’objet_.|
|Modèle Key Vault|Modèle Resource Manager|Conservation du journal en jours|Conservation des données en jours. La valeur par défaut est  _365_.|
|Modèle Key Vault|Modèle Resource Manager|Référence SKU du coffre de clés|Spécifie la référence SKU du coffre de clés créé. La valeur par défaut est _Premium_.|
|Modèle Key Vault|Modèle Resource Manager|Nom d’utilisateur administrateur Azure SQL Server|Nom d’utilisateur utilisé pour accéder à Azure SQL Server. Doit correspondre à la valeur de propriété du **modèle Azure SQL Database**. La valeur par défaut est _sql-admin-user_.|
|Groupe de ressources Azure SQL Database|Resource group|Nom|**Verrouillé**  : concatène le **nom de l’organisation** et `-workload-azsql-rg` pour rendre le groupe de ressources unique.|
|Groupe de ressources Azure SQL Database|Resource group|Emplacement|**Verrouillé**  : utilise le paramètre du blueprint.|
|Modèle Azure SQL Database|Modèle Resource Manager|Nom d’utilisateur administrateur Azure SQL Server|Nom d’utilisateur pour Azure SQL Server. Doit correspondre à la valeur de propriété du **modèle Key Vault**. La valeur par défaut est _sql-admin-user_.|
|Modèle Azure SQL Database|Modèle Resource Manager|Mot de passe d’administrateur Azure SQL Server (ID de ressource Key Vault)|ID de ressource du coffre de clés. Utilisez « /subscription/{subscriptionId}/resourceGroups/{orgName}-workload-kv/providers/Microsoft.KeyVault/vaults/{orgName}-workload-kv » et remplacez `{subscriptionId}` par votre ID d’abonnement et `{orgName}` par le paramètre de blueprint **Nom de l’organisation**.|
|Modèle Azure SQL Database|Modèle Resource Manager|Mot de passe d’administrateur Azure SQL Server (Nom du secret Key Vault)|Nom d’utilisateur de l’administrateur SQL Server. Doit correspondre à la valeur de propriété **Modèle Key Vault** **Nom d’utilisateur administrateur Azure SQL Server**.|
|Modèle Azure SQL Database|Modèle Resource Manager|Conservation du journal en jours|Conservation des données en jours. La valeur par défaut est  _365_.|
|Modèle Azure SQL Database|Modèle Resource Manager|ID objet de l’administrateur AAD|ID objet AAD de l’utilisateur affecté en tant qu’administrateur Active Directory. N’a pas de valeur par défaut et ne peut pas être vide. Pour localiser cette valeur à partir du portail Azure, recherchez et sélectionnez « Utilisateurs » sous _Services_. Utilisez la zone _Nom_ pour filtrer le nom du compte, puis sélectionnez ce compte. Dans la page _Profil utilisateur_ , sélectionnez l’icône « Cliquez pour copier » à côté de l’ _ID d’objet_.|
|Modèle Azure SQL Database|Modèle Resource Manager|Connexion administrateur AAD|Les comptes Microsoft (comme live.com ou outlook.com) ne peuvent pas actuellement être définis en tant qu’administrateur. Seuls les utilisateurs et groupes de sécurité au sein de votre organisation peuvent être définis en tant qu’administrateur. N’a pas de valeur par défaut et ne peut pas être vide. Pour localiser cette valeur à partir du portail Azure, recherchez et sélectionnez « Utilisateurs » sous _Services_. Utilisez la zone _Nom_ pour filtrer le nom du compte, puis sélectionnez ce compte. Dans la page _Profil utilisateur_ , copiez le _Nom d’utilisateur_.|
|Groupe de ressources App Service Environment|Resource group|Nom|**Verrouillé**  : concatène le **nom de l’organisation** et `-workload-ase-rg` pour rendre le groupe de ressources unique.|
|Groupe de ressources App Service Environment|Resource group|Emplacement|**Verrouillé**  : utilise le paramètre du blueprint.|
|Modèle App Service Environment|Modèle Resource Manager|Nom de domaine|Nom de l’annuaire Active Directory créé par l’exemple. La valeur par défaut est _contoso.com_.|
|Modèle App Service Environment|Modèle Resource Manager|Emplacement ASE|Emplacement d’App Service Environment. La valeur par défaut est _USA Ouest 2_.|
|Modèle App Service Environment|Modèle Resource Manager|Conservation du journal Application Gateway en jours|Conservation des données en jours. La valeur par défaut est  _365_.|

## <a name="next-steps"></a>Étapes suivantes

Vous venez de passer en revue les étapes à suivre pour déployer l’exemple de blueprint Charge de travail App Service Environment/SQL Database ISO 27001. Consultez à présent les articles suivants pour découvrir l’architecture et le mappage des contrôles :

> [!div class="nextstepaction"]
> [Blueprint Charge de travail App Service Environment/SQL Database ISO 27001 - Vue d’ensemble](./index.md)
> [Blueprint Charge de travail App Service Environment/SQL Database ISO 27001 - Mappage des contrôles](./control-mapping.md)

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../../how-to/update-existing-assignments.md).
