---
title: Tutoriel de configuration de Saviynt avec Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Tutoriel pour configurer Azure Active Directory B2C avec Saviynt pour l’intégration interapplication afin de simplifier la modernisation informatique et d’améliorer la sécurité, la gouvernance et la conformité. 
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 09/16/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 99741a412ae02a2d30293e1aa254245f5ca7b662
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96171430"
---
# <a name="tutorial-for-configuring-saviynt-with-azure-active-directory-b2c"></a>Tutoriel de configuration de Saviynt avec Azure Active Directory B2C

Dans cet exemple de tutoriel, nous fournissons des conseils sur la façon d’intégrer Azure Active Directory (AD) B2C à [Saviynt](https://saviynt.com/integrations/azure-ad/for-b2c/). La plateforme Security Manager de Saviynt fournit la visibilité, la sécurité et la gouvernance dont les entreprises d’aujourd’hui ont besoin dans une seule plateforme unifiée. Saviynt intègre le risque et la gouvernance des applications, la gestion de l’infrastructure, la gestion des comptes privilégiés et l’analyse des risques des clients.

Dans cet exemple de tutoriel, vous allez configurer Saviynt pour fournir une administration déléguée affinée basée sur le contrôle d’accès pour les utilisateurs d’Azure AD B2C. Saviynt effectue les vérifications suivantes pour déterminer si un utilisateur est autorisé à gérer les utilisateurs d’Azure AD B2C.

- Sécurité au niveau des fonctionnalités pour déterminer si un utilisateur peut effectuer une opération spécifique. Par exemple, Créer un utilisateur, Mettre à jour un utilisateur, Réinitialiser le mot de passe de l’utilisateur, etc.

- Sécurité au niveau des champs pour déterminer si un utilisateur peut lire/écrire un attribut spécifique d’un autre utilisateur pendant les opérations de gestion des utilisateurs. Par exemple, l’agent du support technique peut uniquement mettre à jour le numéro de téléphone, et tous les autres attributs sont en lecture seule.

- Sécurité au niveau des données pour déterminer si un utilisateur peut effectuer une opération donnée sur un utilisateur spécifique. Par exemple, l’administrateur du support technique pour la région Royaume-Uni peut uniquement gérer les utilisateurs du Royaume-Uni.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous avez :

- Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

- Un [locataire Azure AD B2C](./tutorial-create-tenant.md). Le locataire est lié à votre abonnement Azure.

- Un [abonnement](https://saviynt.com/contact-us/) Saviynt.

## <a name="scenario-description"></a>Description du scénario

L’intégration de Saviynt inclut les composants suivants :

- [Azure AD B2C](https://azure.microsoft.com/services/active-directory/external-identities/b2c/) : identité business-to-customer en tant que service qui permet le contrôle personnalisé de la manière dont vos clients s’inscrivent, se connectent et gèrent leurs profils.

- [Saviynt](https://saviynt.com/integrations/azure-ad/for-b2c/) : plateforme de gouvernance des identités qui fournit une administration déléguée fine pour la gestion du cycle de vie des utilisateurs et la gouvernance des accès des utilisateurs Azure AD B2C.  

- [API Microsoft Graph](/graph/use-the-api) : cette API fournit les interfaces permettant à Saviynt de gérer les utilisateurs Azure AD B2C et leur accès dans Azure AD B2C.

Le diagramme d’architecture suivant illustre l’implémentation.

![Image montrant le diagramme de l’architecture Saviynt](./media/partner-saviynt/saviynt-architecture-diagram.png)

|Étape | Description |
|:-----| :-----------|
| 1. | Un administrateur délégué lance une opération de gestion des utilisateurs Azure AD B2C via Saviynt.
| 2. | Saviynt vérifie avec son moteur d’autorisation si l’administrateur délégué peut effectuer l’opération spécifique.
| 3. | Le moteur d’autorisation de Saviynt envoie une réponse de réussite ou d’échec de l’autorisation.
| 4. | Saviynt permet à l’administrateur délégué d’effectuer l’opération requise.
| 5. | Saviynt appelle l’API Microsoft Graph ainsi que les attributs utilisateur pour gérer l’utilisateur dans Azure AD B2C.
| 6. | L’API Microsoft Graph va à son tour créer/mettre à jour/supprimer l’utilisateur dans Azure AD B2C.
| 7. | Azure AD B2C enverra une réponse de réussite ou d’échec.
| 8. | L’API Microsoft Graph renverra ensuite la réponse à Saviynt.

## <a name="onboard-with-saviynt"></a>Intégration à Saviynt

1. Pour créer un compte Saviynt, contactez [Saviynt](https://saviynt.com/contact-us/).

2. Créez des stratégies d’administration déléguée et assignez des utilisateurs en tant qu’[administrateurs délégués](../active-directory/roles/concept-delegation.md) avec différents rôles.

## <a name="configure-azure-ad-b2c-with-saviynt"></a>Configurer Azure AD B2C avec Saviynt

### <a name="create-an-azure-ad-application-for-saviynt"></a>Créer une application Azure AD pour Saviynt

1. Connectez-vous au [portail Azure](https://portal.azure.com/#home).

2. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.

3. Dans le portail Azure, recherchez et sélectionnez **Azure AD B2C**.

4. Sélectionnez **Inscription d’applications** > **Nouvelle inscription**.

5. Entrez un Nom pour l’application. Par exemple, Saviynt, puis sélectionnez **Créer**.

6. Accédez à **Autorisations de l’API** et sélectionnez **+Ajouter une autorisation**.

7. La page Demander des autorisations d’API apparaît. Sélectionnez l’onglet **API Microsoft** et sélectionnez **Microsoft Graph** comme API Microsoft couramment utilisées.

8. Accédez à la page suivante, puis sélectionnez **Permissions d’application**.

9. Sélectionnez **Répertoire**, puis cochez les cases **Directory.Read.All** et **Directory.ReadWrite.All**.

10. Sélectionnez **Ajouter des autorisations**. Passez en revue les autorisations ajoutées.

11. Sélectionnez **Accorder le consentement de l’administrateur pour le répertoire par défaut** > **Enregistrer**.

12. Accédez à **Certificats et secrets** et sélectionnez **+Ajouter une clé secrète client**. Entrez la description de la clé secrète client, sélectionnez l’option d’expiration, puis sélectionnez **Ajouter**.

13. La clé secrète est générée et affichée dans la section Clé secrète client.

    >[!NOTE]
    > Vous aurez besoin de la clé secrète client plus tard.

14. Accédez à **Vue d’ensemble** et récupérez l’**ID client** et l’**ID de locataire**.

15. L’ID de locataire, l’ID client et la clé secrète client seront nécessaires pour terminer l’installation dans Saviynt.

### <a name="enable-saviynt-to-delete-users"></a>Permettre à Saviynt de supprimer des utilisateurs

Les étapes ci-dessous expliquent comment permettre à Saviynt d’effectuer des opérations de suppression d’utilisateur dans Azure AD B2C.

>[!NOTE]
>[Évaluez le risque avant d’accorder aux rôles d’administrateur l’accès à un principal de service.](../active-directory/develop/app-objects-and-service-principals.md)

1. Installez la version la plus récente du module MSOnline PowerShell sur une station de travail ou un serveur Windows.

2. Connectez-vous au module AzureAD PowerShell et exécutez les commandes suivantes :

```powershell
Connect-msolservice #Enter Admin credentials of the Azure portal
$webApp = Get-MsolServicePrincipal –AppPrincipalId “<ClientId of Azure AD Application>”
Add-MsolRoleMember -RoleName "Company Administrator" -RoleMemberType ServicePrincipal -RoleMemberObjectId $webApp.ObjectId
```

## <a name="test-the-solution"></a>Test de la solution

Accédez à votre locataire d’application Saviynt et testez la gestion du cycle de vie des utilisateurs et la gouvernance des accès.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans Azure AD B2C](./custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](./custom-policy-get-started.md?tabs=applications)

- [Créer une application API web](./add-web-api-application.md)