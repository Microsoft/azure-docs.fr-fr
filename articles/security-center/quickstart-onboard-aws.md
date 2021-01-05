---
title: Connecter votre compte AWS à Azure Security Center
description: Surveillance de vos ressources AWS à partir d’Azure Security Center
author: memildin
ms.author: memildin
ms.date: 9/22/2020
ms.topic: quickstart
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 23609613cace2f9be90ad5c88d4d51bb1c2f1a9b
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96573508"
---
#  <a name="connect-your-aws-accounts-to-azure-security-center"></a>Connecter vos comptes AWS à Azure Security Center

Les charges de travail cloud couvrant généralement plusieurs plates-formes cloud, les services de sécurité cloud se doivent d’en faire de même.

Azure Security Center protège les charges de travail dans Azure, Amazon Web Services (AWS) et Google Cloud Platform (GCP).

L’intégration de votre compte AWS dans Security Center intègre AWS Security Hub et Azure Security Center. Security Center offre ainsi une visibilité et une protection dans ces deux environnements cloud pour fournir les éléments suivants :

- Approvisionnement automatique des agents (Security Center utilise [Azure Arc](../azure-arc/servers/overview.md) pour déployer l’agent Log Analytics sur vos instances AWS)
- Gestion des stratégies
- Gestion des vulnérabilités
- Détection de point de terminaison et réponse incorporée (EDR)
- Détection des erreurs de configuration de sécurité
- Vue unique présentant les recommandations de Security Center et les découvertes d’AWS Security Hub
- Incorporation de vos ressources AWS dans les calculs de score sécurisés de Security Center
- Évaluations de conformité réglementaire de vos ressources AWS

Dans la capture d’écran ci-dessous, vous pouvez voir les comptes AWS affichés dans le tableau de bord de vue d’ensemble de Security Center.

:::image type="content" source="./media/quickstart-onboard-aws/aws-account-in-overview.png" alt-text="3 projets GCP répertoriés dans le tableau de bord de vue d’ensemble de Security Center" lightbox="./media/quickstart-onboard-gcp/gcp-account-in-overview.png":::

## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|PRÉVERSION<br>[!INCLUDE [Legalese](../../includes/security-center-preview-legal-text.md)] |
|Prix :|Nécessite [Azure Defender pour les serveurs](defender-for-servers-introduction.md)|
|Rôles et autorisations obligatoires :|**Propriétaire** ou **Contributeur** sur l’abonnement Azure approprié|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Non](./media/icons/no-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||



## <a name="connect-your-aws-account"></a>Connecter votre compte AWS

Suivez les étapes ci-dessous pour créer votre connecteur cloud AWS. 

### <a name="step-1-set-up-aws-security-hub"></a>Étape 1. Configurer AWS Security Hub :

1. Pour afficher les recommandations de sécurité pour plusieurs régions, répétez les étapes suivantes pour chaque région concernée.

    > [!IMPORTANT]
    > Si vous utilisez un compte principal AWS, répétez les trois étapes suivantes pour configurer le compte principal et tous les comptes de membres connectés dans toutes les régions concernées

    1. Activez [Configuration d’AWS](https://docs.aws.amazon.com/config/latest/developerguide/gs-console.html).
    1. Activer [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-settingup.html).
    1. Vérifier que les données sont transmises à Security Hub.

        Lorsque vous activez Security Hub pour la première fois, plusieurs heures peuvent être nécessaires pour que les données soient disponibles.

### <a name="step-2-set-up-authentication-for-security-center-in-aws"></a>Étape 2. Configurer l’authentification pour Security Center dans AWS

Il y a deux façons de permettre à Security Center de s’authentifier auprès d’AWS :

- **Créer un rôle IAM pour Security Center** : c’est la méthode la plus sécurisée et elle est recommandée
- **Utilisateur AWS pour Security Center** : une option moins sécurisée si IAM n’est pas activé

#### <a name="create-an-iam-role-for-security-center"></a>Créer un rôle IAM pour Security Center
1. Depuis votre console Amazon Web Services, sous **Security, Identity & Compliance** (Sécurité, identité et conformité), sélectionnez **IAM**.
    :::image type="content" source="./media/quickstart-onboard-aws/aws-identity-and-compliance.png" alt-text="Services AWS":::

1. Sélectionnez **Rôles** et **Créer un rôle**.
1. Sélectionnez **Another AWS account** (Autre compte AWS).
1. Entrez les informations suivantes :

    - **ID de compte** : entrez l’ID de compte Microsoft (**158177204117**) comme indiqué sur la page du connecteur AWS dans Security Center.
    - **Exiger un ID externe** : doit être sélectionné
    - **ID externe** : entrez l’ID d’abonnement comme indiqué sur la page du connecteur AWS dans Security Center 

1. Sélectionnez **Suivant**.
1. Dans la section **Attacher des stratégies d’autorisation**, sélectionnez les stratégies suivantes :

    - SecurityAudit
    - AmazonSSMAutomationRole
    - AWSSecurityHubReadOnlyAccess

1. Ajoutez des étiquettes si vous le souhaitez. L’ajout d’étiquettes à l’utilisateur n’affecte pas la connexion.
1. Sélectionnez **Suivant**.

1. Dans la liste Rôles, choisissez le rôle que vous avez créé

1. Enregistrez le Nom de ressource Amazon (ARN) pour plus tard. 

#### <a name="create-an-aws-user-for-security-center"></a>Créer un utilisateur AWS pour Security Center 
1. Ouvrez l’onglet **Utilisateurs**, puis sélectionnez **Ajouter un utilisateur**.
1. Dans l’étape **Détails**, entrez un nom d’utilisateur pour Security Center et veillez à sélectionner **Accès programmatique**  pour le type d’accès AWS. 
1. Sélectionnez **Next: Permissions** (Suivant : Autorisations).
1. Sélectionnez **Attacher directement des stratégies existantes** et appliquez les stratégies suivantes :
    - SecurityAudit
    - AmazonSSMAutomationRole
    - AWSSecurityHubReadOnlyAccess
    
1. Sélectionnez **Suivant : Balises**. Ajoutez des étiquettes si vous le souhaitez. L’ajout d’étiquettes à l’utilisateur n’affecte pas la connexion.
1. Sélectionnez **Révision**.
1. Enregistrez l’**ID de clé d’accès** généré automatiquement et le fichier CSV de la **clé d’accès secrète** pour plus tard.
1. Révisez le résumé, puis cliquez sur **Créer un utilisateur**.


### <a name="step-3-configure-the-ssm-agent"></a>Étape 3. Configurer l’agent SSM

AWS Systems Manager est requis pour l’automatisation des tâches entre vos ressources AWS. Si vos instances EC2 ne disposent pas de l’agent SSM, suivez les instructions correspondantes d’Amazon :

- [Installation et configuration de l’agent SSM sur les instances Windows](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-ssm-win.html)
- [Installation et configuration de l’agent SSM sur les instances Amazon EC2 Linux](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-ssm-agent.html)


### <a name="step-4-complete-azure-arc-prerequisites"></a>Étape 4. Satisfaire les prérequis Azure Arc
1. Vérifiez que les [fournisseurs de ressources Azure](../azure-arc/servers/agent-overview.md#register-azure-resource-providers) appropriés sont enregistrés :
    - Microsoft.HybridCompute
    - Microsoft.GuestConfiguration

1. Créez un principal de service pour une intégration à grande échelle. En tant que **propriétaire** de l’abonnement que vous souhaitez utiliser pour l’intégration, créez un principal de service pour l’intégration d’Azure Arc, comme décrit dans [Créer un principal de service pour une intégration à grande échelle](../azure-arc/servers/onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale).


### <a name="step-5-connect-aws-to-security-center"></a>Étape 5. Connecter AWS à Security Center

1. Dans le menu de Security Center, sélectionnez **Connecteurs cloud multiples**.
1. Sélectionnez **Ajouter un compte AWS**.
    :::image type="content" source="./media/quickstart-onboard-aws/add-aws-account.png" alt-text="Ajouter un compte AWS sur la page connecteurs multicloud de Security Center":::
1. Configurez les options dans l’onglet **Authentification AWS** :
    1. Entrez un **nom d’affichage** pour le connecteur.
    1. Confirmez que l’abonnement est correct. Il s’agit de l’abonnement qui inclura les recommandations relatives au connecteur et à AWS Security Hub.
    1. Selon l’option d’authentification que vous avez choisie dans l’[étape 2. Configurez l’authentification pour Security Center dans AWS](#step-2-set-up-authentication-for-security-center-in-aws) :
        - Sélectionnez **Prendre le rôle** et collez l’ARN obtenu à partir de [Créer un rôle IAM pour Security Center](#create-an-iam-role-for-security-center).
            :::image type="content" source="./media/quickstart-onboard-aws/paste-arn-in-portal.png" alt-text="Collage du fichier ARN dans le champ approprié de l’Assistant de connexion AWS dans le portail Azure":::

            OR

        - Sélectionnez **Informations d’identification** et collez la **clé d’accès** et la **clé secrète** à partir du fichier .csv que vous avez enregistré dans [Créer un utilisateur AWS pour Security Center](#create-an-aws-user-for-security-center).
1. Sélectionnez **Suivant**.
1. Configurez les options dans l’onglet **Configuration d’Azure Arc** :

    Security Center Découvre les instances EC2 dans le compte AWS connecté et utilise SSM pour les intégrer à Azure Arc. 

    > [!TIP]
    > Pour obtenir la liste des systèmes d’exploitation pris en charge, consultez [Quels sont les systèmes d’exploitation pris en charge pour mes instances EC2 ?](#what-operating-systems-for-my-ec2-instances-are-supported) dans le forum aux questions.

    1. Sélectionnez le **groupe de ressources** et la **région Azure** à laquelle les services AWS EC2 découverts seront intégrés dans l’abonnement sélectionné.
    1. Entrez l’**ID du principal du service** et la **clé secrète du client du principal du service** pour Azure Arc, comme décrit dans [Créer un principal de service pour l’intégration à l’échelle](../azure-arc/servers/onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale)
    1. Si la machine se connecte à Internet via un serveur proxy, spécifiez l’adresse IP du serveur proxy ou le nom et le numéro de port que la machine utilise pour communiquer avec le serveur proxy. Saisissez la valeur au format ```http://<proxyURL>:<proxyport>```
    1. Sélectionnez **Revoir + créer**.

        Afficher le résumé des informations

        Les sections Étiquettes répertorient toutes les étiquettes Azure qui seront automatiquement créées pour chaque EC2 intégré, avec leurs propres détails correspondants pour les reconnaître facilement dans Azure. 

        En savoir plus sur les étiquettes Azure dans [Utiliser des étiquettes pour organiser vos ressources Azure et votre hiérarchie de gestion](../azure-resource-manager/management/tag-resources.md).

### <a name="step-6-confirmation"></a>Étape 6. Confirmation

Lorsque le connecteur est correctement créé et qu’AWS Security Hub a été configuré correctement :

- Security Center analyse l’environnement pour les instances AWS EC2 en les intégrant à Azure Arc, ce qui permet d’installer l’agent Log Analytics et de fournir des recommandations en matière de protection contre les menaces et de sécurité. 
- Le service ASC analyse les nouvelles instances AWS EC2 toutes les 6 heures et les intègre en fonction de la configuration.
- La norme AWS CIS s’affiche dans le tableau de bord de conformité réglementaire de Security Center.
- Si la stratégie Security Hub est activée, des recommandations s’affichent dans le portail Security Center et le tableau de bord de conformité aux réglementations 5-10 minutes après la fin de l’intégration.
    :::image type="content" source="./media/quickstart-onboard-aws/aws-resources-in-recommendations.png" alt-text="Ressources AWS et recommandations sur la page Recommandations de Security Center":::



## <a name="monitoring-your-aws-resources"></a>Surveillance de vos ressources AWS

Comme indiqué ci-dessus, la page Recommandations de sécurité d’Azure Security Center affiche vos ressources AWS avec vos ressources Azure et GCP pour une véritable vue multicloud.

Pour afficher toutes les recommandations actives pour vos ressources par type de ressource, utilisez la page d’inventaire des ressources de Security Center et filtrez sur le type de ressource AWS qui vous intéresse :

:::image type="content" source="./media/quickstart-onboard-aws/aws-resource-types-in-inventory.png" alt-text="Filtre de type de ressource de la page d’inventaire des ressources avec les options AWS"::: 


## <a name="aws-in-security-center-faq"></a>AWS dans le FAQ de Security Center

### <a name="what-operating-systems-for-my-ec2-instances-are-supported"></a>Quels sont les systèmes d’exploitation pris en charge pour mes instances EC2 ?

Système d’exploitation pris en charge pour l’intégration automatique à Azure Arc pour les machines AWS

- Ubuntu 16.04 : l’agent SSM est préinstallé par défaut
- Ubuntu 18.04 : l’agent SSM est préinstallé par défaut
- Windows Server : l’agent SSM est préinstallé par défaut
- CentOS Linux 7 : SSM doit être installé manuellement ou intégré séparément
- SUSE Linux Enterprise Server (SLES) 15 (x64) : SSM doit être installé manuellement ou intégré séparément
- Red Hat Enterprise Linux (RHEL) 7 (x64) : SSM doit être installé manuellement ou intégré séparément


## <a name="next-steps"></a>Étapes suivantes

La connexion de votre compte AWS fait partie de l’expérience multicloud disponible dans Azure Security Center. Pour accéder à des informations connexes, consultez la page suivante :

- [Connectez vos comptes GCP à Azure Security Center](quickstart-onboard-gcp.md)
