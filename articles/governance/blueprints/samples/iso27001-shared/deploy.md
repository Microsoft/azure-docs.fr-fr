---
title: Déployer l’exemple de blueprint Services partagés ISO 27001
description: Étapes de déploiement pour l’exemple de blueprint Services partagés ISO 27001 et informations détaillées sur les paramètres d’artefact de blueprint.
ms.date: 11/02/2020
ms.topic: sample
ms.openlocfilehash: acce9ecf906430047fa1e687f719fd4e04daf756
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420275"
---
# <a name="deploy-the-iso-27001-shared-services-blueprint-sample"></a>Déployer l’exemple de blueprint Services partagés ISO 27001

Pour déployer l’exemple de blueprint Services partagés ISO 27001 pour Azure Blueprints, vous devez effectuer les étapes suivantes :

> [!div class="checklist"]
> - Créer un blueprint à partir de l’exemple
> - Marquer la copie de l’exemple en tant que **Publié**
> - Affecter votre copie du blueprint à un abonnement existant

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free) avant de commencer.

## <a name="create-blueprint-from-sample"></a>Créer un blueprint à partir de l’exemple

Commencez par implémenter l’exemple de blueprint. Pour cela, créez un blueprint dans votre environnement en vous servant de l’exemple comme point de départ.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Dans la page **Démarrage** à gauche, sélectionnez le bouton **Créer** sous _Créer un blueprint_.

1. Recherchez l’exemple de blueprint **ISO 27001 : Services partagés** sous _Autres exemples_ et sélectionnez **Utiliser cet exemple**.

1. Entrez les _Fonctions de base_ de l’exemple de blueprint :

   - **Nom du blueprint**  : Entrez le nom de votre copie de l’exemple de blueprint Services partagés ISO 27001.
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
     - **Préfixe d’adresse de sous-réseau des services partagés**  : fournissez la valeur en notation CIDR de la mise en réseau des ressources déployées ensemble.
     - **Emplacement des services partagés**  : détermine l’emplacement dans lequel les artefacts sont déployés. Tous les services ne sont pas disponibles dans tous les emplacements. Les artefacts déployant de tels services fournissent une option de paramètre qui permet de définir l’emplacement dans lequel les artefacts sont déployés.
     - **Emplacements autorisés (stratégie : initiative de blueprint pour ISO 27001**  : valeur qui indique les emplacements autorisés pour les groupes de ressources et les ressources.
     - **Espace de travail Log Analytics pour agents de machine virtuelle (stratégie : initiative de blueprint pour ISO 27001**  : spécifie l’ID de ressource d’un espace de travail. Ce paramètre utilise une fonction `concat` pour construire l’ID de ressource.

   - Paramètres d'artefact

     Les paramètres définis dans cette section s’appliquent à l’artefact sous lequel elle est définie. Ces paramètres sont des [paramètres dynamiques](../../concepts/parameters.md#dynamic-parameters) puisqu’ils sont définis lors de l’affectation du blueprint. Pour obtenir la liste complète des paramètres d’artefact et leur description, consultez le [tableau des paramètres d’artefact](#artifact-parameters-table).

1. Une fois tous les paramètres entrés, sélectionnez **Affecter** au bas de la page. L’affectation du blueprint est créée et le déploiement de l’artefact démarre. Le déploiement prend environ une heure. Pour vérifier l’état du déploiement, ouvrez l’affectation du blueprint.

> [!WARNING]
> Le service Azure Blueprints et les exemples de blueprint intégrés sont **gratuits**. Les ressources Azure sont [facturées par produit](https://azure.microsoft.com/pricing/). Utilisez la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour estimer le coût d’exécution des ressources déployées par cet exemple de blueprint.

## <a name="artifact-parameters-table"></a>Tableau des paramètres d’artefact

Le tableau suivant fournit la liste des paramètres d’artefact de blueprint :

|Nom de l’artefact|Type d’artefact|Nom du paramètre|Description|
|-|-|-|-|
|\[Préversion\] : Déployer Log Analytics Agent pour Linux VM Scale Sets (VMSS)|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Linux pris en charge à ajouter à l’étendue|(Facultatif) La valeur par défaut est _["none"]_ .|
|\[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Linux|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Linux pris en charge à ajouter à l’étendue|(Facultatif) La valeur par défaut est _["none"]_ .|
|\[Préversion\] : Déployer Log Analytics Agent pour Windows VM Scale Sets (VMSS)|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Windows pris en charge à ajouter à l’étendue|(Facultatif) La valeur par défaut est _["none"]_ .|
|\[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Windows|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Windows pris en charge à ajouter à l’étendue|(Facultatif) La valeur par défaut est _["none"]_ .|
|Types de ressources autorisés|Affectation de rôle|Types de ressources autorisés|Liste des types de ressources pouvant être déployés. Cette liste est composée de tous les types de ressources déployés dans les Services partagés.|
|Références SKU de compte de stockage autorisées|Affectation de rôle|Références SKU de stockage autorisées|Liste des références SKU de compte de stockage de journaux de diagnostic autorisées. La valeur par défaut est _["Standard_LRS"]_ .|
|Références SKU de machine virtuelle autorisées|Affectation de rôle|Liste des références SKU de machine virtuelle autorisées à être déployées. La valeur par défaut est _["Standard_DS1_v2", "Standard_DS2_v2"]_ .|
|Initiative de blueprint pour ISO 27001|Affectation de rôle|Types de ressources utilisés pour auditer les journaux de diagnostic|Liste des types de ressources utilisés pour déterminer si le paramètre de journal de diagnostic n’est pas activé. Les valeurs acceptables sont listées dans les [schémas de journaux de diagnostic Azure Monitor](../../../../azure-monitor/platform/resource-logs-schema.md#service-specific-schemas).|
|Groupe de ressources Log Analytics|Resource group|Nom|**Verrouillé**  : concatène le **nom de l’organisation** et `-sharedsvsc-log-rg` pour rendre le groupe de ressources unique.|
|Groupe de ressources Log Analytics|Resource group|Emplacement|**Verrouillé**  : utilise le paramètre du blueprint.|
|Modèle Log Analytics|Modèle Resource Manager|Niveau de service|Définit le niveau de l’espace de travail Log Analytics. La valeur par défaut est _PerNode_.|
|Modèle Log Analytics|Modèle Resource Manager|Conservation du journal en jours|Conservation des données en jours. La valeur par défaut est  _365_.|
|Modèle Log Analytics|Modèle Resource Manager|Emplacement|Région utilisée pour créer l’espace de travail Log Analytics. La valeur par défaut est _USA Ouest 2_.|
|Groupe de ressources réseau|Resource group|Nom|**Verrouillé**  : concatène le **nom de l’organisation** et `-sharedsvcs-net-rg` pour rendre le groupe de ressources unique.|
|Groupe de ressources réseau|Resource group|Emplacement|**Verrouillé**  : utilise le paramètre du blueprint.|
|Modèle Pare-feu Azure|Modèle Resource Manager|Adresse IP privée de pare-feu Azure|Configure l’adresse IP privée du [pare-feu Azure](../../../../firewall/overview.md). Cette valeur est également utilisée comme table de routage par défaut sur le sous-réseau des services partagés. Doit faire partie de la notation CIDR définie dans **Préfixe d’adresse de sous-réseau de Pare-feu Azure**. La valeur par défaut est  _10.0.4.4_.|
|Modèle Pare-feu Azure|Modèle Resource Manager|Conservation du journal en jours|Conservation des données en jours. La valeur par défaut est  _365_.|
|Modèle Groupe de sécurité réseau|Modèle Resource Manager|Conservation du journal en jours|Conservation des données en jours. La valeur par défaut est  _365_.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|Préfixe d’adresse de réseau virtuel|Notation CIDR du réseau virtuel. La valeur par défaut est  _10.0.0.0/16_.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|Activer la protection DDoS de réseau virtuel|Configure la protection DDoS pour le réseau virtuel. La valeur par défaut est _true_.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|Préfixe d’adresse de sous-réseau des services partagés|Notation CIDR du sous-réseau Services partagés. La valeur par défaut est  _10.0.0.0/24_.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|Préfixe d’adresse de sous-réseau DMZ|Notation CIDR du sous-réseau DMZ. La valeur par défaut est  _10.0.1.0/24_.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|Préfixe d’adresse de sous-réseau Application Gateway|Notation CIDR du sous-réseau de la passerelle d’application. La valeur par défaut est  _10.0.2.0/24_.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|Préfixe d’adresse de sous-réseau de passerelle de réseau virtuel|Notation CIDR du sous-réseau de la passerelle de réseau virtuel. La valeur par défaut est  _10.0.3.0/24_.|
|Modèle Réseau virtuel et table de routage|Modèle Resource Manager|Préfixe d’adresse de sous-réseau de Pare-feu Azure|Notation CIDR du sous-réseau du [pare-feu Azure](../../../../firewall/overview.md). Doit inclure le paramètre **Adresse IP privée de pare-feu Azure**.|
|Groupe de ressources Key Vault|Resource group|Nom|**Verrouillé**  : concatène le **nom de l’organisation** et `-sharedsvcs-kv-rg` pour rendre le groupe de ressources unique.|
|Groupe de ressources Key Vault|Resource group|Emplacement|**Verrouillé**  : utilise le paramètre du blueprint.|
|Modèle Key Vault|Modèle Resource Manager|Nom d’utilisateur de l’administrateur Jumpbox|Nom d’utilisateur de Jumpbox. Doit correspondre à la valeur de propriété du **modèle Jumpbox**. La valeur par défaut est _jb-admin-user_.|
|Modèle Key Vault|Modèle Resource Manager|Mot de passe ou clé SSH de l’administrateur Jumpbox|Clé ou mot de passe du compte sur Jumpbox. Doit correspondre à la valeur de propriété du **modèle Jumpbox**. N’a pas de valeur par défaut et ne peut pas être vide.|
|Modèle Key Vault|Modèle Resource Manager|Nom d’utilisateur de l’administrateur de domaine|Nom d’utilisateur utilisé pour accéder à la machine virtuelle Active Directory et joindre d’autres machines virtuelles à un domaine. Doit correspondre à la valeur de propriété **Utilisateur administrateur de domaine** dans **Modèle Active Directory Domain Services**. La valeur par défaut est _domain-admin-user_.|
|Modèle Key Vault|Modèle Resource Manager|Mot de passe d’administrateur de domaine|Mot de passe de l’utilisateur administrateur de domaine. N’a pas de valeur par défaut et ne peut pas être vide.|
|Modèle Key Vault|Modèle Resource Manager|ID d’objet AAD|Identificateur d’objet AAD du compte qui nécessite l’accès à l’instance Key Vault. N’a pas de valeur par défaut et ne peut pas être vide. Pour localiser cette valeur à partir du portail Azure, recherchez et sélectionnez « Utilisateurs » sous _Services_. Utilisez la zone _Nom_ pour filtrer le nom du compte, puis sélectionnez ce compte. Dans la page _Profil utilisateur_ , sélectionnez l’icône « Cliquez pour copier » à côté de l’ _ID d’objet_.  |
|Modèle Key Vault|Modèle Resource Manager|Conservation du journal en jours|Conservation des données en jours. La valeur par défaut est  _365_.|
|Modèle Key Vault|Modèle Resource Manager|Référence SKU du coffre de clés|Spécifie la référence SKU du coffre de clés créé. La valeur par défaut est _Premium_.|
|Groupe de ressources Jumpbox|Resource group|Nom|**Verrouillé**  : concatène le **nom de l’organisation** et `-sharedsvcs-jb-rg` pour rendre le groupe de ressources unique.|
|Groupe de ressources Jumpbox|Resource group|Emplacement|**Verrouillé**  : utilise le paramètre du blueprint.|
|Modèle Jumpbox|Modèle Resource Manager|Nom d’utilisateur de l’administrateur Jumpbox|Nom d’utilisateur utilisé pour accéder aux machines virtuelles Jumpbox. Doit correspondre à la valeur de propriété du **modèle Key Vault**. La valeur par défaut est _jb-admin-user_.|
|Modèle Jumpbox|Modèle Resource Manager|Mot de passe d’administrateur Jumpbox (ID de ressource Key Vault)|ID de ressource du coffre de clés. Utilisez « /subscriptions/{subscriptionId}/resourceGroups/{orgName}-sharedsvcs-kv-rg/providers/Microsoft.KeyVault/vaults/{orgName}-sharedsvcs-kv » et remplacez `{subscriptionId}` par votre ID d’abonnement et `{orgName}` par le paramètre de blueprint **Nom de l’organisation**.|
|Modèle Jumpbox|Modèle Resource Manager|Mot de passe d’administrateur Jumpbox (Nom du secret Key Vault)|Nom d’utilisateur de l’administrateur Jumpbox. Doit correspondre à la valeur de la propriété **Modèle Key Vault** **Nom d’utilisateur de l’administrateur Jumpbox**.|
|Modèle Jumpbox|Modèle Resource Manager|Système d’exploitation de Jumpbox|Détermine le système d’exploitation de la machine virtuelle Jumpbox. La valeur par défaut est _Windows_.|
|Groupe de ressources Active Directory Domain Services|Resource group|Nom|**Verrouillé**  : concatène le **nom de l’organisation** et `-sharedsvcs-adds-rg` pour rendre le groupe de ressources unique.|
|Groupe de ressources Active Directory Domain Services|Resource group|Emplacement|**Verrouillé**  : utilise le paramètre du blueprint.|
|Modèle Active Directory Domain Services|Modèle Resource Manager|Nom d’utilisateur de l’administrateur de domaine|Nom d’utilisateur de Jumpbox ADDS. Doit correspondre à la valeur de propriété du **modèle Key Vault**. La valeur par défaut est _adds-admin-user_.|
|Modèle Active Directory Domain Services|Modèle Resource Manager|Mot de passe d’administrateur de domaine (ID de ressource Key Vault)|ID de ressource du coffre de clés. Utilisez « /subscriptions/{subscriptionId}/resourceGroups/{orgName}-sharedsvcs-kv-rg/providers/Microsoft.KeyVault/vaults/{orgName}-sharedsvcs-kv » et remplacez `{subscriptionId}` par votre ID d’abonnement et `{orgName}` par le paramètre de blueprint **Nom de l’organisation**.|
|Modèle Active Directory Domain Services|Modèle Resource Manager|Mot de passe d’administrateur de domaine (Nom du secret Key Vault)|Nom d’utilisateur de l’administrateur de domaine. Doit correspondre à la valeur de la propriété **Modèle Key Vault** **Nom d’utilisateur de l’administrateur de domaine**.|
|Modèle Active Directory Domain Services|Modèle Resource Manager|Nom de domaine|Nom de l’annuaire Active Directory créé par l’exemple. La valeur par défaut est _contoso.com_.|
|Modèle Active Directory Domain Services|Modèle Resource Manager|Utilisateur administrateur de domaine|Nom d’utilisateur pour le compte AD d’administrateur et la jonction de périphériques au domaine AD. Doit correspondre à la valeur de propriété **Nom d’utilisateur d’administrateur Active Directory** dans **Modèle Key Vault**. La valeur par défaut est _domain-admin-user_.|
|Modèle Active Directory Domain Services|Modèle Resource Manager|Mot de passe d’administrateur de domaine|Définissez les détails de Key Vault pour stocker le mot de passe. N’a pas de valeur par défaut et ne peut pas être vide.|

## <a name="next-steps"></a>Étapes suivantes

Vous venez de passer en revue les étapes à suivre pour déployer l’exemple de blueprint Services partagés ISO 27001. Consultez à présent les articles suivants pour découvrir l’architecture et le mappage des contrôles :

> [!div class="nextstepaction"]
> [Blueprint Services partagés ISO 27001 - Vue d’ensemble](./index.md)
> [Blueprint Services partagés ISO 27001 - Mappage des contrôles](./control-mapping.md)

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../../how-to/update-existing-assignments.md).
