---
title: Connecter votre compte GCP à Azure Security Center
description: Surveillance de vos ressources GCP à partir d’Azure Security Center
author: memildin
ms.author: memildin
ms.date: 9/22/2020
ms.topic: quickstart
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 1095222f332e7a66f684feddf1fdab17704ca658
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96573542"
---
#  <a name="connect-your-gcp-accounts-to-azure-security-center"></a>Connectez vos comptes GCP à Azure Security Center

Les charges de travail cloud couvrant généralement plusieurs plates-formes cloud, les services de sécurité cloud se doivent d’en faire de même.

Azure Security Center protège les charges de travail dans Azure, Amazon Web Services (AWS) et Google Cloud Platform (GCP).

L’intégration de votre compte GCP dans Security Center intègre GCP Security Command et Azure Security Center. Security Center offre ainsi une visibilité et une protection dans ces deux environnements cloud pour fournir les éléments suivants :

- Détection des erreurs de configuration de sécurité
- Vue unique présentant les recommandations de Security Center et les découvertes de GCP Security Command Center
- Incorporation de vos ressources GCP dans les calculs de score sécurisés de Security Center
- Intégration des recommandations de GCP Security Command Center basées sur la norme CIS dans le tableau de bord de conformité réglementaire de Security Center

Dans la capture d’écran ci-dessous, vous pouvez voir les projets GCP affichés dans le tableau de bord de vue d’ensemble de Security Center.

:::image type="content" source="./media/quickstart-onboard-gcp/gcp-account-in-overview.png" alt-text="3 projets GCP répertoriés dans le tableau de bord de vue d’ensemble de Security Center" lightbox="./media/quickstart-onboard-gcp/gcp-account-in-overview.png":::


## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|PRÉVERSION<br>[!INCLUDE [Legalese](../../includes/security-center-preview-legal-text.md)] |
|Prix :|Nécessite [Azure Defender pour les serveurs](defender-for-servers-introduction.md)|
|Rôles et autorisations obligatoires :|**Propriétaire** ou **Contributeur** sur l’abonnement Azure approprié|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Non](./media/icons/no-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||

## <a name="connect-your-gcp-account"></a>Connecter votre compte GCP

Suivez les étapes ci-dessous pour créer votre connecteur cloud GCP. 

### <a name="step-1-set-up-gcp-security-command-center-with-security-health-analytics"></a>Étape 1. Configurer la sécurité de GCP et Command Center avec Security Health Analytics

Pour tous les projets GCP de votre organisation, vous devez également :

1. Configurer **GCP Security Command Center** à l’aide de [ces instructions de la documentation de GCP](https://cloud.google.com/security-command-center/docs/quickstart-scc-setup).
1. Activer **Security Health Analytics** à l’aide de [ces instructions de la documentation de GCP](https://cloud.google.com/security-command-center/docs/how-to-use-security-health-analytics).
1. Vérifier que les données sont transmises à Security Command Center.

Les instructions de connexion de votre environnement GCP pour la configuration de sécurité suivent les recommandations de Google pour l’utilisation des recommandations de configuration de la sécurité. L’intégration s’appuie sur Google Security Command Center et consomme des ressources supplémentaires susceptibles d’avoir un impact sur votre facturation.

Lorsque vous activez pour la première fois Security Health Center, plusieurs heures peuvent être nécessaires pour que les données soient disponibles.


### <a name="step-2-enable-gcp-security-command-center-api"></a>Étape 2. Activer l’API GCP Security Command Center

1. À partir de la **bibliothèque d’API de Google Cloud Console**, sélectionnez le projet que vous souhaitez connecter à Azure Security Center.
1. Dans la bibliothèque d’API, recherchez et sélectionnez **API Security Command Center**.
1. Sur la page de l’API, sélectionnez **ACTIVER**.

En savoir plus sur [l’API Security Command Center](https://cloud.google.com/security-command-center/docs/reference/rest/).


### <a name="step-3-create-a-dedicated-service-account-for-the-security-configuration-integration"></a>Étape 3. Créer un compte de service dédié pour l’intégration de la configuration de la sécurité

1. Dans la **console GCP**, sélectionnez le projet que vous souhaitez connecter à Security Center.
1. Dans le **menu de navigation**, sous les options **IAM & admin**, sélectionnez **Comptes de service**.
1. Sélectionnez **CRÉER LE COMPTE DE SERVICE**.
1. Entrez un nom de compte, puis sélectionnez **Créer**.
1. Spécifiez le **Rôle** comme **Visionneuse d’administration de Security Center**, puis sélectionnez **Continuer**.
1. La section **Accorder aux utilisateurs l’accès à ce compte de service** est facultative. Sélectionnez **Terminé**.
1. Copiez la **valeur E-mail** du compte de service créé, puis enregistrez-la pour une utilisation ultérieure.
1. Dans le **menu de navigation**, sous les options **IAM & admin**, sélectionnez **IAM**
    1. Basculez vers le niveau de l’organisation.
    1. Sélectionnez **AJOUTER**.
    1. Dans le champ **Nouveaux membres**, collez la **valeur E-mail** que vous avez copiée précédemment.
    1. Spécifiez le rôle **Visionneuse d’administration de Security Center**, puis sélectionnez Enregistrer.
        :::image type="content" source="./media/quickstart-onboard-gcp/iam-settings-gcp-permissions-admin-viewer.png" alt-text="Définition les autorisations GCP pertinentes":::


### <a name="step-4-create-a-private-key-for-the-dedicated-service-account"></a>Étape 4. Créer une clé privée pour le compte de service dédié
1. Passez au niveau du projet.
1. Dans le **menu de navigation**, sous les options **IAM & admin**, sélectionnez **Comptes de service**.
1. Ouvrez le compte de service dédié et sélectionnez Modifier.
1. Dans la section **Clés**, sélectionnez **Ajouter une clé**, puis **Créer une nouvelle clé**.
1. Sur l’écran Créer une clé privée, sélectionnez **JSON**, puis sélectionnez **CRÉER**.
1. Enregistrez ce fichier JSON pour une utilisation ultérieure.


### <a name="step-5-connect-gcp-to-security-center"></a>Étape 5. Connecter GCP à Security Center 
1. Dans le menu de Security Center, sélectionnez **Connecteurs cloud**.
1. Sélectionnez Ajouter un compte GCP.
1. Dans la page d’intégration, effectuez les opérations suivantes, puis sélectionnez **Suivant**.
    1. Validez l’abonnement choisi.
    1. Dans le champ **Nom d'affichage**, entrez un nom pour le connecteur.
    1. Dans le champ **ID d’organisation**, entrez l’ID de votre organisation. Si vous ne le connaissez pas, consultez [Création et gestion des organisations](https://cloud.google.com/resource-manager/docs/creating-managing-organization).
    1. Dans la zone de fichier **Clé privée**, accédez au fichier JSON que vous avez téléchargé à l’étape [l’Étape 4. Créer une clé privée pour le compte de service dédié](#step-4-create-a-private-key-for-the-dedicated-service-account).


### <a name="step-6-confirmation"></a>Étape 6. Confirmation

Lorsque le connecteur est correctement créé et que le GCP Security Command Center a été configuré correctement :

- La norme CIS GCP s’affiche dans le tableau de bord de conformité réglementaire de Security Center.
- Les recommandations de sécurité pour vos ressources GCP s’affichent dans le portail Security Center et le tableau de bord de conformité aux réglementations 5-10 minutes après la fin de l’intégration :   :::image type="content" source="./media/quickstart-onboard-gcp/gcp-resources-in-recommendations.png" alt-text="Ressources GCP et recommandations sur la page Recommandations de Security Center":::


## <a name="monitoring-your-gcp-resources"></a>Surveillance de vos ressources GCP

Comme indiqué ci-dessus, la page Recommandations de sécurité d’Azure Security Center affiche vos ressources GCP avec vos ressources Azure et AWS pour une véritable vue multi-cloud.

Pour afficher toutes les recommandations actives pour vos ressources par type de ressource, utilisez la page d’inventaire des ressources de Security Center et filtrez sur le type de ressource GCP qui vous intéresse :

:::image type="content" source="./media/quickstart-onboard-gcp/gcp-resource-types-in-inventory.png" alt-text="Filtre de type de ressource de la page d’inventaire des ressources avec les options GCP"::: 


## <a name="next-steps"></a>Étapes suivantes

La connexion de votre compte GCP fait partie de l’expérience multi-cloud disponible dans Azure Security Center. Pour accéder à des informations connexes, consultez la page suivante :

- [Connecter vos comptes AWS à Azure Security Center](quickstart-onboard-aws.md)
