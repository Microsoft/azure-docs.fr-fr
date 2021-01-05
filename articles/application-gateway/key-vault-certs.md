---
title: Arrêt TLS avec certificats Azure Key Vault
description: Découvrez comment intégrer Azure Application Gateway avec Key Vault pour les certificats de serveur associés à des écouteurs HTTPS.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: victorh
ms.openlocfilehash: 95ca4933b97199ba6d8ac1bed7587af5d3bd559f
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94648121"
---
# <a name="tls-termination-with-key-vault-certificates"></a>Arrêt TLS avec certificats Key Vault

[Azure Key Vault](../key-vault/general/overview.md) est un magasin des secrets managé par une plateforme que vous pouvez utiliser pour protéger des secrets, des clés et des certificats TLS/SSL. Azure Application Gateway prend en charge l’intégration dans Key Vault des certificats de serveur associés à des écouteurs HTTPS. Cette prise en charge se limite à la référence SKU v2 d’Application Gateway.

L'intégration de Key Vault propose deux modèles d'arrêt TLS :

- Vous pouvez explicitement fournir des certificats TLS/SSL associés à l'écouteur. Ce modèle constitue le moyen traditionnel de transmettre des certificats TLS/SSL à Application Gateway pour un arrêt TLS.
- Vous pouvez éventuellement fournir une référence à un certificat Key Vault existant ou un secret lorsque vous créez un écouteur HTTPS.

L’intégration d’Application Gateway avec Key Vault offre de nombreux avantages, notamment :

- Une sécurité renforcée, dans la mesure où les certificats TLS/SSL ne sont pas directement gérés par l'équipe de développement d'applications. L’intégration permet à une équipe de sécurité distincte de :
  * Configurer des passerelles d’application.
  * Contrôler les cycles de vie des passerelles d’application.
  * Accorder des autorisations à certaines passerelles d’application pour accéder aux certificats stockés dans votre coffre de clés.
- Prendre en charge l’importation de certificats existants dans votre coffre de clés. Ou utiliser les API Key Vault pour créer et gérer de nouveaux certificats avec un partenaire Key Vault approuvé.
- Prise en charge du renouvellement automatique des certificats stockés dans votre coffre de clés.

À l’heure actuelle, Application Gateway prend uniquement en charge les certificats validés par logiciel. Les certificats validés par module de sécurité matériel ne sont pas pris en charge. Une fois qu'Application Gateway est configurée pour utiliser des certificats Key Vault, ses instances récupèrent le certificat dans Key Vault et l'installent localement pour l'arrêt TLS. Les instances interrogent également Key Vault à des intervalles de quatre heures pour récupérer une version renouvelée du certificat, le cas échéant. Si un certificat mis à jour est trouvé, le certificat TLS/SSL associé à l'écouteur HTTPS est automatiquement remplacé.

> [!NOTE]
> Le portail Azure prend en charge seulement les certificats KeyVault Certificate, pas les secrets. Application Gateway prend néanmoins toujours en charge le référencement des secrets KeyVault, mais seulement via des ressources autres que celles du portail, comme PowerShell, l’interface CLI, les API, les modèles ARM, etc. 

## <a name="how-integration-works"></a>Fonctionnement de l’intégration

L’intégration d’Application Gateway avec Key Vault nécessite un processus de configuration en trois étapes :

1. **Créer une identité managée attribuée par l’utilisateur**

   Vous créez ou réutilisez une identité managée existante affectée à l’utilisateur, qu’Application Gateway utilise pour récupérer les certificats de Key Vault à votre place. Pour plus d’informations, consultez [Créer, répertorier, supprimer ou affecter un rôle à une identité managée affectée par l’utilisateur à l’aide du portail Azure](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md). Cette étape crée une identité dans le tenant Azure Active Directory. L’identité est approuvée par l’abonnement utilisé pour créer l’identité.

1. **Configurer votre coffre de clés**

   Vous importez ensuite un certificat existant ou vous en créez un dans votre coffre de clés. Ce certificat sera utilisé par les applications qui s’exécutent au moyen de la passerelle d’application. Dans cette étape, vous pouvez également utiliser un secret de coffre de clés, qui est stocké dans un fichier PFX encodé en Base64 et sans mot de passe. Nous vous recommandons d’utiliser un type de certificat en raison de la fonctionnalité de renouvellement automatique disponible avec les objets de type de certificat dans le coffre de clés. Une fois que vous avez créé un certificat ou un secret, vous définissez des stratégies d’accès dans le coffre de clés pour autoriser l’identité qui disposera d’un accès *get* au secret.
   
   > [!IMPORTANT]
   > Application Gateway nécessite actuellement que Key Vault autorise l’accès à partir de tous les réseaux pour tirer parti de l’intégration. Il ne prend pas en charge l’intégration de Key Vault lorsque ce dernier est configuré pour autoriser uniquement les points de terminaison privés et certains accès aux réseaux. Nous travaillons actuellement à la prise en charge des réseaux privés et de certains autres réseaux pour permettre une intégration complète de Key Vault à Application Gateway. 

   > [!NOTE]
   > Si vous déployez la passerelle d’application à l’aide d’un modèle Resource Manager, que ce soit en utilisant Azure CLI ou PowerShell, ou via une application Azure déployée à partir du portail Azure, le certificat SSL est stocké dans le coffre de clés en tant que fichier PFX codé en Base64. Vous devez effectuer les étapes décrites dans [Utiliser Azure Key Vault pour transmettre une valeur de paramètre sécurisée pendant le déploiement](../azure-resource-manager/templates/key-vault-parameter.md). 
   >
   > Il est particulièrement important de définir `enabledForTemplateDeployment` sur `true`. Le certificat peut être sans mot de passe ou avoir un mot de passe. Dans le cas d’un certificat avec mot de passe, l’exemple suivant montre une configuration possible pour l’entrée `sslCertificates` dans `properties` de la configuration du modèle ARM pour une passerelle d’application. Les valeurs de `appGatewaySSLCertificateData` et `appGatewaySSLCertificatePassword` sont recherchées à partir du coffre de clés, comme décrit à la section [Référencement de secrets avec un ID dynamique](../azure-resource-manager/templates/key-vault-parameter.md#reference-secrets-with-dynamic-id). Suivez les références en remontant à partir de `parameters('secretName')` pour voir comment la recherche se produit. Si le certificat est sans mot de passe, omettez l’entrée `password`.
   >   
   > ```
   > "sslCertificates": [
   >     {
   >         "name": "appGwSslCertificate",
   >         "properties": {
   >             "data": "[parameters('appGatewaySSLCertificateData')]",
   >             "password": "[parameters('appGatewaySSLCertificatePassword')]"
   >         }
   >     }
   > ]
   > ```

1. **Configurer la passerelle d’application**

   Après avoir terminé les deux étapes précédentes, vous pouvez configurer ou modifier une passerelle d’application existante pour utiliser l’identité managée affectée à l’utilisateur. Pour plus d’informations, consultez [Set-AzApplicationGatewayIdentity](/powershell/module/az.network/set-azapplicationgatewayidentity).

   Vous pouvez également configurer le certificat TLS/SSL de l'écouteur HTTP pour pointer vers l'URI complet du certificat ou l'ID de secret Key Vault.

   ![Certificats Key Vault](media/key-vault-certs/ag-kv.png)

## <a name="next-steps"></a>Étapes suivantes

[Configurer l'arrêt TLS avec des certificats Key Vault à l'aide d'Azure PowerShell](configure-keyvault-ps.md)
