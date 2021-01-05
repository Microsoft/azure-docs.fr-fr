---
title: Configurer la fédération directe avec AD FS pour B2B - Azure AD
description: Découvrez comment configurer AD FS en tant que fournisseur d’identité pour la fédération directe de sorte que les invités puissent se connecter à vos applications Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 07/01/2019
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1b3d7c47ff0a2c533bf12a67958a913b22915f75
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87907222"
---
# <a name="example-direct-federation-with-active-directory-federation-services-ad-fs-preview"></a>Exemple : Fédération directe avec les services de fédération Active Directory (AD FS) (préversion)

> [!NOTE]
> La fédération directe est une fonctionnalité d’évaluation publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Cet article décrit comment configurer la [fédération directe](direct-federation.md) à l’aide des services de fédération Active Directory (AD FS) en tant que fournisseur d’identité SAML 2.0 ou WS-Fed. Pour prendre en charge la fédération directe, des attributs et revendications doivent être configurés au niveau du fournisseur d’identité. Pour illustrer comment configurer un fournisseur d’identité pour la fédération directe, nous allons utiliser les services de fédération Active Directory (AD FS) comme exemple. Nous allons montrer comment configurer AD FS en tant que fournisseur d’identité SAML et en tant que fournisseur d’identité WS-Fed.

> [!NOTE]
> Cet article décrit comment configurer AD FS pour SAML et WS-Fed à titre d’illustration. Pour les intégrations de fédération directe dans lesquelles le fournisseur d’identité est AD FS, nous recommandons d’utiliser WS-Fed en tant que protocole. 

## <a name="configure-ad-fs-for-saml-20-direct-federation"></a>Configurer AD FS pour la fédération directe SAML 2.0
Azure AD B2B peut être configuré pour la fédération avec les fournisseurs d’identité qui utilisent le protocole SAML avec certaines exigences spécifiques indiquées ci-dessous. Pour illustrer les étapes de configuration SAML, cette section montre comment configurer AD FS pour SAML 2.0. 

Pour configurer la fédération directe, les attributs suivants doivent être reçus dans la réponse SAML 2.0 à partir du fournisseur d’identité. Ces attributs peuvent être configurés en liant le fichier XML du service d’émission de jeton de sécurité en ligne ou en les entrant manuellement. L’étape 12 de [Créer une instance de test AD FS](https://medium.com/in-the-weeds/create-a-test-active-directory-federation-services-3-0-instance-on-an-azure-virtual-machine-9071d978e8ed) décrit comment rechercher les points de terminaison AD FS ou comment générer votre URL de métadonnées, par exemple `https://fs.iga.azure-test.net/federationmetadata/2007-06/federationmetadata.xml`. 

|Attribut  |Valeur  |
|---------|---------|
|AssertionConsumerService     |`https://login.microsoftonline.com/login.srf`         |
|Public visé     |`urn:federation:MicrosoftOnline`         |
|Émetteur     |L’URI de l’émetteur du fournisseur d’identité partenaire, par exemple `http://www.example.com/exk10l6w90DHM0yi...`         |

Les revendications suivantes doivent être configurées dans le jeton SAML 2.0 émis par le fournisseur d’identité :


|Attribut  |Valeur  |
|---------|---------|
|Format NameID     |`urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`         |
|emailaddress     |`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`         |


La section suivante illustre comment configurer les attributs et revendications requis à l’aide d’AD FS comme exemple de fournisseur d’identité SAML 2.0.

### <a name="before-you-begin"></a>Avant de commencer

Un serveur AD FS doit déjà être configuré et opérationnel avant de commencer cette procédure. Pour obtenir de l’aide sur la configuration d’un serveur AD FS, consultez [Créer une instance de test AD FS 3.0 sur une machine virtuelle Azure](https://medium.com/in-the-weeds/create-a-test-active-directory-federation-services-3-0-instance-on-an-azure-virtual-machine-9071d978e8ed).

### <a name="add-the-claim-description"></a>Ajouter la description de revendication

1. Sur votre serveur AD FS, sélectionnez **Outils** > **Gestion AD FS**.
2. Dans le volet de navigation, sélectionnez **Service** > **Descriptions des revendications**.
3. Sous **Actions**, sélectionnez **Ajouter une description de revendication**.
4. Dans la fenêtre **Ajouter une description de revendication**, spécifiez les valeurs suivantes :

   - **Nom d’affichage** : Identificateur persistant
   - **Identificateur de revendication** : `urn:oasis:names:tc:SAML:2.0:nameid-format:persistent` 
   - Cochez la case **Publier cette description de revendication dans les métadonnées de fédération en tant que type de revendication pouvant être accepté par ce service FS (Federation Service)** .
   - Cochez la case **Publier cette description de revendication dans les métadonnées de fédération en tant que type de revendication pouvant être envoyé par ce service FS (Federation Service)** .

5. Cliquez sur **OK**.

### <a name="add-the-relying-party-trust-and-claim-rules"></a>Ajouter l’approbation de partie de confiance et de règles de revendication

1. Sur le serveur AD FS, accédez à **Outils** > **Gestion AD FS**.
2. Dans le volet de navigation, sélectionnez **Relations d’approbation** > **Approbations de partie de confiance**.
3. Sous **Actions**, sélectionnez **Ajouter une approbation de partie de confiance**. 
4. Dans l’Assistant d’approbation de partie de confiance, pour **Sélectionner une source de données**, utilisez l’option **Importer les données, publiées en ligne ou sur un réseau local, concernant la partie de confiance**. Spécifiez cette URL des métadonnées de fédération https://nexus.microsoftonline-p.com/federationmetadata/saml20/federationmetadata.xml. Laissez les autres sélections par défaut. Sélectionnez **Fermer**.
5. L’Assistant **Modifier les règles de revendication** s’ouvre.
6. Dans l’Assistant **Modifier les règles de revendication**, sélectionnez **Ajouter une règle**. Dans **Choisir le type de règle**, sélectionnez **Envoyer les attributs LDAP en tant que revendications**. Sélectionnez **Suivant**.
7. Dans **Configurer la règle de revendication**, spécifiez les valeurs suivantes : 

   - **Nom de la règle de revendication** : Règle de revendication d’e-mail 
   - **Magasin d’attributs** : Active Directory 
   - **Attribut LDAP** : Adresses e-mail 
   - **Type de revendication sortante** : Adresse e-mail

8. Sélectionnez **Terminer**.
9. La fenêtre **Modifier les règles de revendication** affiche la nouvelle règle. Cliquez sur **Appliquer**. 
10. Cliquez sur **OK**.  

### <a name="create-an-email-transform-rule"></a>Créer une règle de transformation d’e-mail
1. Accédez à **Modifier les règles de revendication** et cliquez sur **Ajouter une règle**. Dans **Choisir le type de règle**, sélectionnez **Transformer une revendication entrante** et cliquez sur **Suivant**. 
2. Dans **Configurer la règle de revendication**, spécifiez les valeurs suivantes : 

   - **Nom de la règle de revendication** : Règle de transformation d’e-mail 
   - **Type de revendication entrante** : Adresse e-mail 
   - **Type de revendication sortante** : ID de nom 
   - **Format d’ID de nom sortant** : Identificateur persistant 
   - Sélectionnez **Transférer toutes les valeurs de revendication**.

3. Cliquez sur **Terminer**. 
4. La fenêtre **Modifier les règles de revendication** affiche les nouvelles règles. Cliquez sur **Appliquer**. 
5. Cliquez sur **OK**. Le serveur AD FS est désormais configuré pour la fédération directe à l’aide du protocole SAML 2.0.

## <a name="configure-ad-fs-for-ws-fed-direct-federation"></a>Configurer AD FS pour la fédération directe WS-Fed 
Azure AD B2B peut être configuré pour la fédération avec les fournisseurs d’identité qui utilisent le protocole WS-Fed avec les exigences spécifiques indiquées ci-dessous. Actuellement, les deux fournisseurs WS-Fed testés pour assurer la compatibilité avec Azure AD incluent AD FS et Shibboleth. Nous allons utiliser ici les services de fédération Active Directory (AD FS) comme exemple de fournisseur d’identité WS-Fed. Pour plus d’informations sur l’approbation de partie de confiance entre un fournisseur compatible WS-Fed et Azure AD, téléchargez les documents relatifs à la compatibilité du fournisseur d’identité Azure AD.

Pour configurer la fédération directe, les attributs suivants doivent être reçus dans le message WS-Fed à partir du fournisseur d’identité. Ces attributs peuvent être configurés en liant le fichier XML du service d’émission de jeton de sécurité en ligne ou en les entrant manuellement. L’étape 12 de [Créer une instance de test AD FS](https://medium.com/in-the-weeds/create-a-test-active-directory-federation-services-3-0-instance-on-an-azure-virtual-machine-9071d978e8ed) décrit comment rechercher les points de terminaison AD FS ou comment générer votre URL de métadonnées, par exemple `https://fs.iga.azure-test.net/federationmetadata/2007-06/federationmetadata.xml`.
 
|Attribut  |Valeur  |
|---------|---------|
|PassiveRequestorEndpoint     |`https://login.microsoftonline.com/login.srf`         |
|Public visé     |`urn:federation:MicrosoftOnline`         |
|Émetteur     |L’URI de l’émetteur du fournisseur d’identité partenaire, par exemple `http://www.example.com/exk10l6w90DHM0yi...`         |

Revendications requises pour le jeton WS-Fed émis par le fournisseur d’identité :

|Attribut  |Valeur  |
|---------|---------|
|ImmutableID     |`http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID`         |
|emailaddress     |`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`         |

La section suivante illustre comment configurer les attributs et revendications requis à l’aide d’AD FS comme exemple de fournisseur d’identité WS-Fed.

### <a name="before-you-begin"></a>Avant de commencer
Un serveur AD FS doit déjà être configuré et opérationnel avant de commencer cette procédure. Pour obtenir de l’aide sur la configuration d’un serveur AD FS, consultez [Créer une instance de test AD FS 3.0 sur une machine virtuelle Azure](https://medium.com/in-the-weeds/create-a-test-active-directory-federation-services-3-0-instance-on-an-azure-virtual-machine-9071d978e8ed).


### <a name="add-the-relying-party-trust-and-claim-rules"></a>Ajouter l’approbation de partie de confiance et de règles de revendication 
1. Sur le serveur AD FS, accédez à **Outils** > **Gestion AD FS**. 
1. Dans le volet de navigation, sélectionnez **Relations d’approbation** > **Approbations de partie de confiance**. 
1. Sous **Actions**, sélectionnez **Ajouter une approbation de partie de confiance**.  
1. Dans l’Assistant d’approbation de partie de confiance, pour **Sélectionner une source de données**, utilisez l’option **Importer les données, publiées en ligne ou sur un réseau local, concernant la partie de confiance**. Spécifiez cette URL des métadonnées de fédération : `https://nexus.microsoftonline-p.com/federationmetadata/2007-06/federationmetadata.xml`.  Laissez les autres sélections par défaut. Sélectionnez **Fermer**.
1. L’Assistant **Modifier les règles de revendication** s’ouvre. 
1. Dans l’Assistant **Modifier les règles de revendication**, sélectionnez **Ajouter une règle**. Dans **Choisir le type de règle**, sélectionnez **Envoyer les revendications en utilisant une règle personnalisée**. Sélectionnez *Suivant*. 
1. Dans **Configurer la règle de revendication**, spécifiez les valeurs suivantes :

   - **Nom de la règle de revendication** : ID non modifiable du problème  
   - **Règle personnalisée** : `c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"] => issue(store = "Active Directory", types = ("http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID"), query = "samAccountName={0};objectGUID;{1}", param = regexreplace(c.Value, "(?<domain>[^\\]+)\\(?<user>.+)", "${user}"), param = c.Value);`

1. Sélectionnez **Terminer**. 
1. La fenêtre **Modifier les règles de revendication** affiche la nouvelle règle. Cliquez sur **Appliquer**.  
1. Dans le même Assistant **Modifier les règles de revendication**, sélectionnez **Ajouter une règle**. Dans **Choisir le type de règle**, sélectionnez **Envoyer les attributs LDAP en tant que revendications**. Sélectionnez **Suivant**.
1. Dans **Configurer la règle de revendication**, spécifiez les valeurs suivantes : 

   - **Nom de la règle de revendication** : Règle de revendication d’e-mail  
   - **Magasin d’attributs** : Active Directory  
   - **Attribut LDAP** : Adresses e-mail  
   - **Type de revendication sortante** : Adresse e-mail 

1.  Sélectionnez **Terminer**. 
1.  La fenêtre **Modifier les règles de revendication** affiche la nouvelle règle. Cliquez sur **Appliquer**.  
1.  Cliquez sur **OK**. Le serveur AD FS est désormais configuré pour la fédération directe à l’aide de WS-Fed.

## <a name="next-steps"></a>Étapes suivantes
Vous allez ensuite [configurer la fédération directe dans Azure AD](direct-federation.md#step-2-configure-direct-federation-in-azure-ad) dans le portail Azure AD ou à l’aide de PowerShell. 
