---
title: Exemple de blueprint DoD Impact Level 5
description: Étapes de déploiement de l’exemple de blueprint DoD Impact Level 5 et informations détaillées sur les paramètres d’artefact de blueprint.
ms.date: 09/17/2020
ms.topic: sample
ms.openlocfilehash: deb6ffa0f886cc6a4a9bea10a38d7cd82e7df8f1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90978231"
---
# <a name="deploy-the-dod-impact-level-5-blueprint-sample"></a>Déployer l’exemple de blueprint DoD Impact Level 5

Pour déployer l’exemple de blueprint Department of Defense Impact Level 5 (DoD IL5) Azure Blueprints, vous devez effectuer les étapes suivantes :

> [!div class="checklist"]
> - Créer un blueprint à partir de l’exemple
> - Marquer la copie de l’exemple en tant que **Publié**
> - Affecter votre copie du blueprint à un abonnement existant

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free) avant de commencer.

## <a name="create-blueprint-from-sample"></a>Créer un blueprint à partir de l’exemple

Commencez par implémenter l’exemple de blueprint. Pour cela, créez un blueprint dans votre environnement en vous servant de l’exemple comme point de départ.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Dans la page **Démarrage**à gauche, sélectionnez le bouton **Créer** sous _Créer un blueprint_.

1. Recherchez l’exemple de blueprint **DoD Impact Level 5** sous _Autres exemples_, puis sélectionnez **Utiliser cet exemple**.

1. Entrez les _Fonctions de base_ de l’exemple de blueprint :

   - **Nom du blueprint** : Donnez un nom à votre copie de l’exemple de blueprint DoD Impact Level 5.
   - **Emplacement de la définition** : utilisez les points de suspension et sélectionnez le groupe d’administration dans lequel vous souhaitez enregistrer votre copie de l’exemple.

1. Sélectionnez l’onglet _Artefacts_ dans le haut de la page ou **Suivant : Artefacts** dans le bas de la page.

1. Passez en revue la liste des artefacts qui composent l’exemple de blueprint. De nombreux artefacts ont des paramètres que nous définirons plus tard. Sélectionnez **Enregistrer comme brouillon** lorsque vous avez terminé de passer en revue l’exemple de blueprint.

## <a name="publish-the-sample-copy"></a>Publier la copie de l’exemple

Votre copie de l’exemple de blueprint est à présent créée dans votre environnement. Elle est créée en mode **Brouillon** et doit être **publiée** avant de pouvoir être attribuée et déployée. Vous pouvez personnaliser la copie de l’exemple de blueprint en fonction de votre environnement et de vos besoins, mais ces modifications peuvent l’éloigner de son alignement avec les contrôles DoD Impact Level 5.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour rechercher votre copie de l’exemple de blueprint, puis sélectionnez-la.

1. Sélectionnez **Publier le blueprint** dans le haut de la page. Dans la nouvelle page à droite, indiquez la **Version** de votre copie de l’exemple de blueprint. Cette propriété est utile si vous prévoyez d’effectuer une modification ultérieurement. Dans **Notes de changement**, indiquez par exemple « Première version publiée à partir de l’exemple de blueprint DoD Impact Level 5 ». Sélectionnez ensuite **Publier** en bas de la page.

## <a name="assign-the-sample-copy"></a>Affecter la copie de l’exemple

Une fois que la copie de l’exemple de blueprint a été **publiée**, elle peut être affectée à un abonnement dans le groupe d’administration où elle a été enregistrée. C’est à cette étape que les paramètres sont fournis pour que chaque déploiement de la copie de l’exemple de blueprint soit unique.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour rechercher votre copie de l’exemple de blueprint, puis sélectionnez-la.

1. Sélectionnez **Affecter le blueprint** dans le haut de la page.

1. Indiquez les valeurs des paramètres pour l’affectation du blueprint :

   - Concepts de base

     - **Abonnements** : Sélectionnez un ou plusieurs des abonnements qui font partie du groupe d’administration où vous avez enregistré votre copie de l’exemple de blueprint. Si vous sélectionnez plusieurs abonnements, une affectation est créée pour chacun d’eux à l’aide des paramètres saisis.
     - **Nom de l’affectation** : Le nom est prérempli en fonction du nom du blueprint.
       Changez-le si nécessaire ou laissez-le tel quel.
     - **Emplacement** : Sélectionnez une région dans laquelle créer l’identité managée. Azure Blueprint utilise cette identité managée pour déployer tous les artefacts dans le blueprint affecté. Pour en savoir plus, consultez [Identités managées pour les ressources Azure](../../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Version de définition du blueprint** : Choisissez une version **publiée** de votre copie de l’exemple de blueprint.

   - Verrouiller l'affectation

     Sélectionnez le paramètre de verrouillage de blueprint pour votre environnement. Pour plus d’informations, consultez [Verrouillage des ressources des blueprints](../../concepts/resource-locking.md).

   - Identité managée

     Conservez l’option par défaut, _Affecté(e) par le système_, pour l’identité managée.

   - Paramètres d'artefact

     Les paramètres définis dans cette section s’appliquent à l’artefact sous lequel elle est définie. Ces paramètres sont des [paramètres dynamiques](../../concepts/parameters.md#dynamic-parameters) puisqu’ils sont définis lors de l’affectation du blueprint. Pour obtenir la liste complète des paramètres d’artefact et leur description, consultez le [tableau des paramètres d’artefact](#artifact-parameters-table).

1. Une fois tous les paramètres entrés, sélectionnez **Affecter** au bas de la page. L’affectation du blueprint est créée et le déploiement de l’artefact démarre. Le déploiement prend environ une heure. Pour vérifier l’état du déploiement, ouvrez l’affectation du blueprint.

> [!WARNING]
> Le service Azure Blueprints et les exemples de blueprint intégrés sont **gratuits**. Les ressources Azure sont [facturées par produit](https://azure.microsoft.com/pricing/). Utilisez la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour estimer le coût d’exécution des ressources déployées par cet exemple de blueprint.

## <a name="artifact-parameters-table"></a>Tableau des paramètres d’artefact

Le tableau suivant fournit la liste des paramètres d’artefact de blueprint :

|Nom de l’artefact|Type d’artefact|Nom du paramètre|Description|
|-|-|-|-|
|DoD Impact Level 5|Affectation de rôle|Liste des utilisateurs à inclure dans le groupe Administrateurs de machine virtuelle Windows|Liste séparée par des points-virgules des utilisateurs qui doivent être inclus dans le groupe Administrateurs local. Ex. : Administrator; myUser1; myUser2|
|DoD Impact Level 5|Affectation de rôle|Liste des utilisateurs exclus du groupe Administrateurs de machine virtuelle Windows|Liste séparée par des points-virgules des utilisateurs qui doivent être exclus du groupe Administrateurs local. Ex. : Administrator; myUser1; myUser2|
|DoD Impact Level 5|Affectation de rôle|Liste des types de ressources devant avoir les journaux de diagnostic activés||
|DoD Impact Level 5|Affectation de rôle|ID d’espace de travail Log Analytics pour les rapports d’agent de machine virtuelle|ID (GUID) de l’espace de travail Log Analytics où les agents de machine virtuelle doivent envoyer leur rapport|
|DoD Impact Level 5|Affectation de rôle|Liste des régions où Network Watcher doit être activé|Pour voir la liste complète des régions, utilisez Get-AzLocation|
|DoD Impact Level 5|Affectation de rôle|Version TLS minimale pour les serveurs web Windows|Version de protocole TLS minimale qui doit être activée sur les serveurs web Windows|
|DoD Impact Level 5|Affectation de rôle|Dernière version de PHP|Dernière version de PHP prise en charge pour App Services|
|DoD Impact Level 5|Affectation de rôle|Dernière version de Java|Dernière version de Java prise en charge pour App Services|
|DoD Impact Level 5|Affectation de rôle|Dernière version de Python sur Windows|Dernière version de Python prise en charge pour App Services|
|DoD Impact Level 5|Affectation de rôle|Dernière version de Python sur Linux|Dernière version de Python prise en charge pour App Services|
|DoD Impact Level 5|Affectation de rôle|Facultatif : Liste des images de machine virtuelle Windows prenant en charge l’agent Log Analytics à ajouter à l’étendue d’audit|Liste des images séparées par des points-virgules. Ex. : /subscriptions/<subscriptionId>/resourceGroups/YourResourceGroup/providers/Microsoft.Compute/images/ContosoStdImage|
|DoD Impact Level 5|Affectation de rôle|Facultatif : Liste des images de machine virtuelle Linux prenant en charge l’agent Log Analytics à ajouter à l’étendue d’audit|Liste des images séparées par des points-virgules. Ex. : /subscriptions/<subscriptionId>/resourceGroups/YourResourceGroup/providers/Microsoft.Compute/images/ContosoStdImage|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Plusieurs propriétaires doivent être affectés à votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Le chiffrement de disque doit être appliqué sur les machines virtuelles|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : La notification par e-mail au propriétaire de l’abonnement pour les alertes à gravité élevée doit être activée|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Le débogage à distance doit être désactivé pour les applications de fonction|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version du .NET Framework est la plus récente, si elle est utilisée dans le cadre de l’application de fonction|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : La technologie Transparent Data Encryption doit être activée sur les bases de données SQL|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L’évaluation des vulnérabilités doit être activée sur vos instances managées SQL|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de PHP est la plus récente, si elle est utilisée dans le cadre de l’application API|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Un administrateur Azure Active Directory doit être approvisionné pour les serveurs SQL|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Seules les connexions sécurisées à votre cache Redis doivent être activées|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : La solution de protection des points de terminaison doit être installée sur les groupes de machines virtuelles identiques|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Auditer l'accès réseau non restreint aux comptes de stockage|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les paramètres Advanced Data Security pour l’instance gérée SQL doivent inclure une adresse e-mail pour la réception des alertes de sécurité.|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : La sécurisation du transfert vers des comptes de stockage doit être activée|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les ces contrôles d’application adaptatifs doit être activés sur les machines virtuelles|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : La sauvegarde géoredondante doit être activée pour Azure Database pour PostgreSQL|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de Java est la plus récente, si elle est utilisée dans le cadre de l’application web|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Trois propriétaires au plus doivent être désignés pour votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Une adresse e-mail de contact de sécurité doit être fournie pour votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : CORS ne doit pas autoriser toutes les ressources à accéder à vos applications web|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les comptes externes disposant d’autorisations d’écriture doivent être supprimés de votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les comptes externes disposant d’autorisations de lecture doivent être supprimés de votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les comptes déconseillés doivent être supprimés de votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Function App ne doit pas être accessible via HTTPS|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de Python est la plus récente, si elle est utilisée dans le cadre de l’application web|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de Python est la plus récente, si elle est utilisée dans le cadre de l’application de fonction|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de PHP est la plus récente, si elle est utilisée dans le cadre de l’application web|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de Python est la plus récente, si elle est utilisée dans le cadre de l’application API|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : La sauvegarde géoredondante doit être activée pour Azure Database pour MySQL|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version du .NET Framework est la plus récente, si elle est utilisée dans le cadre de l’application web|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Des mises à jour système doivent être installées sur vos machines|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de Java est la plus récente, si elle est utilisée dans le cadre de l’application API|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de HTTP est la plus récente, si elle est utilisée pour exécuter l’application web|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : La dernière version de TLS doit être utilisée dans votre application API|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations d’écriture sur votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les paramètres Advanced Data Security pour le serveur SQL doivent inclure une adresse e-mail pour la réception des alertes de sécurité.|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de HTTP est la plus récente, si elle est utilisée pour exécuter l’application API|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L’extension Microsoft IaaSAntimalware doit être déployée sur les serveurs Windows|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de Java est la plus récente, si elle est utilisée dans le cadre de l’application de fonction|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L'accès via un point de terminaison accessible sur Internet doit être limité|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Le niveau tarifaire standard Security Center doit être sélectionné|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Auditer l’utilisation de règles personnalisées RBAC|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L'application web ne doit pas être accessible via HTTPS|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L’audit sur SQL Server doit être activé|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L’agent Log Analytics doit être installé sur les machines virtuelles|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : DDoS Protection Standard doit être activé|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de PHP est la plus récente, si elle est utilisée dans le cadre de l’application de fonction|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Advanced Data Security doit être activé sur vos serveurs SQL|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Advanced Data Security doit être activée sur vos instances managées SQL.|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les notifications par e-mail aux administrateurs et propriétaires d’abonnements doivent être activées dans les paramètres Advanced Data Security de l’instance managée SQL|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Superviser les agents Endpoint Protection manquants dans Azure Security Center|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Le contrôle d’accès réseau juste-à-temps doit être appliqué sur les machines virtuelles|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Un numéro de téléphone de contact de sécurité doit être fourni pour votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les clusters Service Fabric ne doivent utiliser Azure Active Directory que pour l’authentification client|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L'application API doit uniquement être accessible via HTTPS|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les types Advanced Threat Protection doivent être définis sur « Tous » dans les paramètres Advanced Data Security de l’instance gérée SQL.|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Le stockage géoredondant doit être activé pour les comptes de stockage|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version du .NET Framework est la plus récente, si elle est utilisée dans le cadre de l’application API|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les mises à jour système doivent être installées sur les groupes de machines virtuelles identiques|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les notifications par e-mail aux administrateurs et propriétaires d’abonnements doivent être activées dans les paramètres Advanced Data Security SQL Server|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Le débogage à distance doit être désactivé pour les applications web|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : La sauvegarde géoredondante à long terme doit être activée pour les bases de données Azure SQL|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les vulnérabilités de la configuration de sécurité sur vos machines doivent être corrigées|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Vérifier que la version de HTTP est la plus récente, si elle est utilisée pour exécuter l’application de fonction|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L'authentification multifacteur doit être activée sur les comptes disposant d’autorisations de lecture de votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les types Advanced Threat Protection doivent être définis sur « Tous » dans les paramètres Advanced Data Security du serveur SQL.|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les vulnérabilités dans les configurations de la sécurité des conteneurs doivent être corrigées|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Le débogage à distance doit être désactivé pour les applications API|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L’évaluation des vulnérabilités doit être activée sur vos serveurs SQL|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : L’agent Log Analytics doit être installé sur les groupes identiques de machines virtuelles|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : La dernière version de TLS doit être utilisée dans votre application web|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : La dernière version de TLS doit être utilisée dans votre application de fonction|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : [Préversion] : Kubernetes Services doit être mis à niveau vers une version non vulnérable de Kubernetes|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|
|DoD Impact Level 5|Affectation de rôle|Effet de la stratégie : Les vulnérabilités de vos bases de données SQL doivent être éliminées|Effet Azure Policy sur cette stratégie. Pour plus d’informations sur les effets, visitez https://aka.ms/policyeffects|

## <a name="next-steps"></a>Étapes suivantes

Vous venez de passer en revue la procédure de déploiement de l’exemple de blueprint DoD Impact Level 5. Consultez à présent les articles suivants pour en savoir plus sur le blueprint et le mappage des contrôles :

> [!div class="nextstepaction"]
> [Blueprint DoD Impact Level 5 - Vue d’ensemble](./index.md)
> [Blueprint DoD Impact Level 5 - Mappage des contrôles](./control-mapping.md)

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../../how-to/update-existing-assignments.md).