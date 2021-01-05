---
title: Vue d’ensemble de l’exemple de blueprint HIPAA HITRUST 9.2
description: Vue d’ensemble de l’exemple de blueprint HIPAA HITRUST 9.2. Cet exemple de blueprint aide les clients à évaluer des contrôles HIPAA HITRUST 9.2 spécifiques.
ms.date: 09/04/2020
ms.topic: sample
ms.openlocfilehash: 4df6f05019976b3de1172cc5127c27ac00fe3c44
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89493230"
---
# <a name="hipaa-hitrust-92-blueprint-sample"></a>Exemple de blueprint HIPAA HITRUST 9.2

L’exemple de blueprint HIPAA HITRUST 9.2 fournit des garde-fous de gouvernance avec [Azure Policy](../../policy/overview.md), qui vous permettent d’évaluer des contrôles HIPAA HITRUST 9.2 spécifiques. Avec ce blueprint, les clients peuvent déployer un ensemble de stratégies de base pour toute architecture déployée par Azure devant implémenter des contrôles HIPAA HITRUST 9.2.

## <a name="control-mapping"></a>Correspondance des contrôles

Le [mappage de contrôles Azure Policy](../../policy/samples/hipaa-hitrust-9-2.md) fournit des détails sur les définitions de stratégie incluses dans ce blueprint ainsi que sur la façon dont ces définitions de stratégie sont mappées aux **domaines de conformité** et aux **contrôles** dans HIPAA HITRUST 9.2. Quand des ressources sont attribuées à une architecture, elles sont évaluées par Azure Policy pour savoir si elles sont en conformité avec les stratégies affectées. Pour plus d’informations, consultez [Présentation d’Azure Policy](../../policy/overview.md).

## <a name="deploy"></a>Déployer

Pour déployer l’exemple de blueprint Azure Blueprints HIPAA HITRUST 9.2, vous devez effectuer les étapes suivantes :

> [!div class="checklist"]
> - Créer un blueprint à partir de l’exemple
> - Marquer la copie de l’exemple en tant que **Publié**
> - Affecter votre copie du blueprint à un abonnement existant

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free) avant de commencer.

### <a name="create-blueprint-from-sample"></a>Créer un blueprint à partir de l’exemple

Commencez par implémenter l’exemple de blueprint. Pour cela, créez un blueprint dans votre environnement en vous servant de l’exemple comme point de départ.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Dans la page **Démarrage**à gauche, sélectionnez le bouton **Créer** sous _Créer un blueprint_.

1. Recherchez l’exemple de blueprint **HIPAA HITRUST** sous _Autres exemples_ et sélectionnez **Utiliser cet exemple**.

1. Entrez les _Fonctions de base_ de l’exemple de blueprint :

   - **Nom du blueprint** : Donnez un nom à votre copie de l’exemple de blueprint HIPAA HITRUST 9.2.
   - **Emplacement de la définition** : utilisez les points de suspension et sélectionnez le groupe d’administration dans lequel vous souhaitez enregistrer votre copie de l’exemple.

1. Sélectionnez l’onglet _Artefacts_ dans le haut de la page ou **Suivant : Artefacts** dans le bas de la page.

1. Passez en revue la liste des artefacts qui composent l’exemple de blueprint. De nombreux artefacts ont des paramètres que nous définirons plus tard. Sélectionnez **Enregistrer comme brouillon** lorsque vous avez terminé de passer en revue l’exemple de blueprint.

### <a name="publish-the-sample-copy"></a>Publier la copie de l’exemple

Votre copie de l’exemple de blueprint est à présent créée dans votre environnement. Elle est créée en mode **Brouillon** et doit être **publiée** avant de pouvoir être attribuée et déployée. Vous pouvez personnaliser la copie de l’exemple de blueprint en fonction de votre environnement et de vos besoins, mais ces modifications peuvent l’éloigner de son alignement avec les contrôles HIPAA HITRUST 9.2.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour rechercher votre copie de l’exemple de blueprint, puis sélectionnez-la.

1. Sélectionnez **Publier le blueprint** dans le haut de la page. Dans la nouvelle page à droite, indiquez la **Version** de votre copie de l’exemple de blueprint. Cette propriété est utile si vous prévoyez d’effectuer une modification ultérieurement. Fournissez des **Notes de changement**, en indiquant par exemple « Première version publiée à partir de l’exemple de blueprint HIPAA HITRUST 9.2 ». Sélectionnez ensuite **Publier** en bas de la page.

### <a name="assign-the-sample-copy"></a>Affecter la copie de l’exemple

Une fois que la copie de l’exemple de blueprint a été **publiée**, elle peut être affectée à un abonnement dans le groupe d’administration où elle a été enregistrée. C’est à cette étape que les paramètres sont fournis pour que chaque déploiement de la copie de l’exemple de blueprint soit unique.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour rechercher votre copie de l’exemple de blueprint, puis sélectionnez-la.

1. Sélectionnez **Affecter le blueprint** dans le haut de la page.

1. Indiquez les valeurs des paramètres pour l’affectation du blueprint :

   - Concepts de base

     - **Abonnements** : Sélectionnez un ou plusieurs des abonnements qui font partie du groupe d’administration où vous avez enregistré votre copie de l’exemple de blueprint. Si vous sélectionnez plusieurs abonnements, une affectation est créée pour chacun d’eux à l’aide des paramètres saisis.
     - **Nom de l’affectation** : Le nom est prérempli en fonction du nom du blueprint.
       Changez-le si nécessaire ou laissez-le tel quel.
     - **Emplacement** : Sélectionnez une région dans laquelle créer l’identité managée. Azure Blueprint utilise cette identité managée pour déployer tous les artefacts dans le blueprint affecté. Pour en savoir plus, consultez [Identités managées pour les ressources Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Version de définition du blueprint** : Choisissez une version **publiée** de votre copie de l’exemple de blueprint.

   - Verrouiller l'affectation

     Sélectionnez le paramètre de verrouillage de blueprint pour votre environnement. Pour plus d’informations, consultez [Verrouillage des ressources des blueprints](../concepts/resource-locking.md).

   - Identité managée

     Conservez l’option par défaut, _Affecté(e) par le système_, pour l’identité managée.

   - Paramètres d'artefact

     Les paramètres définis dans cette section s’appliquent à l’artefact sous lequel elle est définie. Ces paramètres sont des [paramètres dynamiques](../concepts/parameters.md#dynamic-parameters) puisqu’ils sont définis lors de l’affectation du blueprint. Pour obtenir la liste complète des paramètres d’artefact et leur description, consultez le [tableau des paramètres d’artefact](#artifact-parameters-table).

1. Une fois tous les paramètres entrés, sélectionnez **Affecter** au bas de la page. L’affectation du blueprint est créée et le déploiement de l’artefact démarre. Le déploiement prend environ une heure. Pour vérifier l’état du déploiement, ouvrez l’affectation du blueprint.

> [!WARNING]
> Le service Azure Blueprints et les exemples de blueprint intégrés sont **gratuits**. Les ressources Azure sont [facturées par produit](https://azure.microsoft.com/pricing/). Utilisez la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour estimer le coût d’exécution des ressources déployées par cet exemple de blueprint.

### <a name="artifact-parameters-table"></a>Tableau des paramètres d’artefact

Le tableau suivant fournit la liste des paramètres d’artefact de blueprint :

|Nom de l’artefact |Nom du paramètre |Description |
|---|---|---|
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |L'accès via un point de terminaison accessible sur Internet doit être limité |Activer ou désactiver la supervision des règles NSG entrantes excessivement permissives |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Comptes : état de compte d’invité |Spécifie si le compte Invité local est désactivé. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Les ces contrôles d’application adaptatifs doit être activés sur les machines virtuelles |Activer ou désactiver la supervision de la liste verte d’applications dans Azure Security Center |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Autoriser les connexions simultanées à Internet ou à un domaine Windows |Spécifiez s’il est nécessaire d’empêcher les ordinateurs de se connecter en même temps à un réseau basé sur un domaine et à un réseau non basé sur un domaine. La valeur 0 autorise les connexions simultanées, et la valeur 1 les bloque. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |L’application API doit être accessible uniquement via HTTPS V2 |Activer ou désactiver la supervision de l’utilisation du protocole HTTPS dans une application API V2 |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Noms des applications (prend en charge les caractères génériques) |Liste séparée par des points-virgules des noms des applications à installer. Par exemple : Par exemple, « Microsoft SQL Server 2014 (64 bits);Microsoft Visual Studio Code » ou « Microsoft SQL Server 2014* » (qui correspond à toutes les applications commençant par « Microsoft SQL Server 2014 ») |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Auditer la fin du processus |Spécifie si les événements d’audit sont générés quand un processus s’est arrêté. Recommandé pour la supervision de l’arrêt des processus critiques. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Auditer l'accès réseau non restreint aux comptes de stockage |Activer ou désactiver la supervision de l’accès réseau au compte de stockage |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Audit : arrêter immédiatement le système s’il n’est pas possible de se connecter aux audits de sécurité |Effectue un audit pour vérifier si le système s’arrête quand il ne peut pas journaliser les événements de sécurité. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Empreinte numérique de certificat |Liste de valeurs séparées par des points-virgules correspondant aux empreintes numériques de certificats qui doivent figurer sous le magasin de certificats racine approuvés (Cert:\LocalMachine\Root). Par exemple, THUMBPRINT1;THUMBPRINT2;THUMBPRINT3 |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Les journaux de diagnostic doivent être activés dans les comptes Batch |Activer ou désactiver la supervision des journaux de diagnostic dans les comptes Batch |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Les journaux de diagnostic dans Event Hub doivent être activés. |Activer ou désactiver la supervision des journaux de diagnostic dans les comptes Event Hub |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Les journaux de diagnostic dans les services Search doivent être activés. |Activer ou désactiver la supervision des journaux de diagnostic dans le service Recherche Azure |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Les journaux de diagnostic dans les groupes identiques de machines virtuelles doivent être activés |Activer ou désactiver la supervision des journaux de diagnostic dans Service Fabric |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Le chiffrement de disque doit être appliqué sur les machines virtuelles |Activer ou désactiver la supervision du chiffrement des disques de machine virtuelle |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Activer les ouvertures de session invité non sécurisées |Spécifie si le client SMB va autoriser les ouvertures de session invitées non sécurisées à un serveur SMB. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Le contrôle d’accès réseau juste-à-temps doit être appliqué sur les machines virtuelles |Activer ou désactiver la supervision de l’accès juste-à-temps au réseau |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Les ports de gestion doivent être fermés sur vos machines virtuelles |Activer ou désactiver la supervision des ports de gestion ouverts sur les machines virtuelles |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations d’écriture sur votre abonnement |Activer ou désactiver la supervision de MFA pour les comptes disposant d’autorisations d’écriture dans l’abonnement |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement |Activer ou désactiver la supervision de MFA pour les comptes disposant d’autorisations de propriétaire dans l’abonnement |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Accès réseau : les chemins de Registre accessibles à distance |Spécifie les chemins de Registre accessibles sur le réseau, quels que soient les utilisateurs ou groupes listés dans la liste ACL (liste de contrôle d’accès) de la clé de Registre `winreg`. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Accès réseau : chemins et sous-chemins de Registre accessibles à distance |Spécifie les chemins et sous-chemins de Registre accessibles sur le réseau, quels que soient les utilisateurs ou groupes listés dans la liste ACL de la clé de Registre `winreg`. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Accès réseau : les partages qui sont accessibles de manière anonyme |Spécifie les partages réseau accessibles aux utilisateurs anonymes. La configuration par défaut de ce paramètre de stratégie a peu d’effet, car tous les utilisateurs doivent être authentifiés avant de pouvoir accéder aux ressources partagées sur le serveur. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Console de récupération : autoriser la copie de disquettes et l’accès à tous les lecteurs et dossiers |Spécifie si la commande SET de la console de récupération doit être disponible, ce qui permet de définir les variables d’environnement de la console de récupération. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Le débogage à distance doit être désactivé pour l’application API |Activer ou désactiver la supervision du débogage à distance pour une application API |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Le débogage à distance doit être désactivé pour l'application web |Activer ou désactiver la supervision du débogage à distance pour une application web |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Conservation obligatoire (en jours) des journaux dans les comptes Batch |Période de conservation obligatoire des journaux de diagnostic en jours |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Conservation obligatoire (en jours) des journaux dans le service Recherche Azure |Période de conservation obligatoire des journaux de diagnostic en jours |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Conservation obligatoire (en jours) des journaux dans les comptes Event Hub |Période de conservation obligatoire des journaux de diagnostic en jours |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Nom du groupe de ressources du compte de stockage (doit exister) permettant de déployer les paramètres de diagnostic pour les groupes de sécurité réseau |Groupe de ressources dans lequel le compte de stockage sera créé. Ce groupe de ressources doit déjà exister. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Le contrôle d’accès en fonction du rôle (RBAC) doit être utilisé sans Kubernetes Services. |Activer ou désactiver la supervision des services Kubernetes sans RBAC activé |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Le protecteur TDE de l’instance gérée SQL doit être chiffré avec votre propre clé. |Activez ou désactivez la supervision du chiffrement TDE (Transparent Data Encryption) avec prise en charge de votre propre clé. Le chiffrement TDE (Transparent Data Encryption) avec prise en charge de votre propre clé offre les avantages suivants : transparence et contrôle améliorés sur le protecteur TDE, sécurité durcie avec un service externe HSM et promotion de la séparation des tâches. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Le protecteur TDE du serveur SQL doit être chiffré avec votre propre clé. |Activez ou désactivez la supervision du chiffrement TDE (Transparent Data Encryption) avec prise en charge de votre propre clé. Le chiffrement TDE (Transparent Data Encryption) avec prise en charge de votre propre clé offre les avantages suivants : transparence et contrôle améliorés sur le protecteur TDE, sécurité durcie avec un service externe HSM et promotion de la séparation des tâches. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Préfixe du compte de stockage régional permettant de déployer les paramètres de diagnostic pour les groupes de sécurité réseau |Ce préfixe peut être combiné à l’emplacement du groupe de sécurité réseau pour former le nom de compte de stockage créé. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Les mises à jour système doivent être installées sur les groupes de machines virtuelles identiques |Activer ou désactiver les rapports des mises à jour système des groupes de machines virtuelles identiques |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Les mises à jour système doivent être installées sur les groupes de machines virtuelles identiques |Activer ou désactiver les rapports des mises à jour système des groupes de machines virtuelles identiques |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Désactiver la résolution de noms multidiffusion |Spécifie si LLMNR, un protocole de résolution de noms secondaire qui transmet en multidiffusion via un lien de sous-réseau local sur un seul sous-réseau, est activé. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Les machines virtuelles doivent être migrées vers de nouvelles ressources Azure Resource Manager |Activer ou désactiver la supervision des machines virtuelles de calcul classiques |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées |Activer ou désactiver la supervision des vulnérabilités de système d’exploitation des groupes de machines virtuelles identiques |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités |Activer ou désactiver la détection des vulnérabilités de machine virtuelle par une solution d’évaluation des vulnérabilités |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |L’évaluation des vulnérabilités doit être activée sur vos instances managées SQL |Auditez les instances managées SQL qui n’ont pas d’analyses récurrentes d’évaluation des vulnérabilités activées. L’évaluation des vulnérabilités peut découvrir et suivre les potentielles vulnérabilités de base de données, et vous aider à y remédier. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (domaine) : appliquer les règles de pare-feu locales |Spécifie si les administrateurs locaux sont autorisés à créer des règles de pare-feu locales qui s’appliquent avec les règles de pare-feu configurées par la stratégie de groupe pour le profil de domaine. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (domaine) : comportement pour les connexions sortantes |Spécifie le comportement des connexions sortantes du profil de domaine qui ne correspondent pas à une règle de pare-feu de trafic sortant. La valeur par défaut 0 signifie que les connexions sont autorisées, et la valeur 1 signifie qu’elles sont bloquées. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (domaine) : comportement pour les connexions sortantes |Spécifie le comportement des connexions sortantes du profil de domaine qui ne correspondent pas à une règle de pare-feu de trafic sortant. La valeur par défaut 0 signifie que les connexions sont autorisées, et la valeur 1 signifie qu’elles sont bloquées. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (domaine) : afficher les notifications |Spécifie si le Pare-feu Windows avec fonctions avancées de sécurité notifie l’utilisateur quand un programme ne peut pas recevoir de connexions entrantes pour le profil de domaine. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (domaine) : utiliser les paramètres du profil |Spécifie si le Pare-feu Windows avec fonctions avancées de sécurité utilise les paramètres du profil de domaine pour filtrer le trafic réseau. Si vous sélectionnez Désactivé, le Pare-feu Windows avec fonctions avancées de sécurité n’utilise aucune des règles de pare-feu ou de sécurité de connexion pour ce profil. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (privé) : appliquer les règles de sécurité de connexion locale |Spécifie si les administrateurs locaux sont autorisés à créer des règles de sécurité de connexion qui s’appliquent avec les règles de sécurité de connexion configurées par la stratégie de groupe pour le profil privé. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (privé) : appliquer les règles de pare-feu locales |Spécifie si les administrateurs locaux sont autorisés à créer des règles de pare-feu locales qui s’appliquent avec les règles de pare-feu configurées par la stratégie de groupe pour le profil privé. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (privé) : comportement pour les connexions sortantes |Spécifie le comportement des connexions sortantes du profil privé qui ne correspondent pas à une règle de pare-feu de trafic sortant. La valeur par défaut 0 signifie que les connexions sont autorisées, et la valeur 1 signifie qu’elles sont bloquées. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (privé) : afficher les notifications |Spécifie si le Pare-feu Windows avec fonctions avancées de sécurité notifie l’utilisateur quand un programme ne peut pas recevoir de connexions entrantes pour le profil privé. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (privé) : utiliser les paramètres du profil |Spécifie si le Pare-feu Windows avec fonctions avancées de sécurité utilise les paramètres du profil privé pour filtrer le trafic réseau. Si vous sélectionnez Désactivé, le Pare-feu Windows avec fonctions avancées de sécurité n’utilise aucune des règles de pare-feu ou de sécurité de connexion pour ce profil. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (public) : appliquer les règles de sécurité de connexion locale |Spécifie si les administrateurs locaux sont autorisés à créer des règles de sécurité de connexion qui s’appliquent avec les règles de sécurité de connexion configurées par la stratégie de groupe pour le profil public. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (public) : appliquer les règles de pare-feu locales |Spécifie si les administrateurs locaux sont autorisés à créer des règles de pare-feu locales qui s’appliquent avec les règles de pare-feu configurées par la stratégie de groupe pour le profil public. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (public) : comportement pour les connexions sortantes |Spécifie le comportement des connexions sortantes du profil public qui ne correspondent pas à une règle de pare-feu de trafic sortant. La valeur par défaut 0 signifie que les connexions sont autorisées, et la valeur 1 signifie qu’elles sont bloquées. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (public) : afficher les notifications |Spécifie si le Pare-feu Windows avec fonctions avancées de sécurité notifie l’utilisateur quand un programme ne peut pas recevoir de connexions entrantes pour le profil public. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows (public) : utiliser les paramètres du profil |Spécifie si le Pare-feu Windows avec fonctions avancées de sécurité utilise les paramètres du profil public pour filtrer le trafic réseau. Si vous sélectionnez Désactivé, le Pare-feu Windows avec fonctions avancées de sécurité n’utilise aucune des règles de pare-feu ou de sécurité de connexion pour ce profil. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows : Domaine : autoriser la réponse monodiffusion |Spécifie si le Pare-feu Windows avec fonctions avancées de sécurité permet à l’ordinateur local de recevoir des réponses de type monodiffusion à ses messages de type multidiffusion ou de type diffusion sortants pour le profil de domaine. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows : privé : autoriser la réponse monodiffusion |Spécifie si le Pare-feu Windows avec fonctions avancées de sécurité permet à l’ordinateur local de recevoir des réponses de type monodiffusion à ses messages de type multidiffusion ou de type diffusion sortants pour le profil privé. |
|Auditer les contrôles HITRUST/HIPAA et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit |Pare-feu Windows : public : autoriser la réponse monodiffusion |Spécifie si le Pare-feu Windows avec fonctions avancées de sécurité permet à l’ordinateur local de recevoir des réponses de type monodiffusion à ses messages de type multidiffusion ou de type diffusion sortants pour le profil public. |

## <a name="next-steps"></a>Étapes suivantes

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../how-to/update-existing-assignments.md).
