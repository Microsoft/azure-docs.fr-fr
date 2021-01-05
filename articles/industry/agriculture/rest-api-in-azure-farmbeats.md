---
title: API Azure FarmBeats
description: Découvrez les API Azure FarmBeats offrent aux entreprises agricoles une interface RESTful standardisée avec des réponses JSON.
author: sunasing
ms.topic: article
ms.date: 11/04/2019
ms.author: sunasing
ms.openlocfilehash: 292374bcce90da042dc6bd672feaad3750759aa9
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92674504"
---
# <a name="azure-farmbeats-apis"></a>API Azure FarmBeats

Cet article décrit les API Azure FarmBeats. Les API Azure FarmBeats offrent aux entreprises agricoles une interface RESTful standardisée avec des réponses JSON pour vous aider à tirer parti des fonctionnalités Azure FarmBeats, telles que :

- API pour obtenir des données de capteurs, caméras, drones, météo et satellites, ainsi que des données organisées concernant le sol.
- Normalisation et contextualisation des données entre les fournisseurs de données courants.
- Fonctionnalités d’accès et de requête schématisées sur toutes les données ingérées.
- Génération automatique de métadonnées qui peuvent être interrogées, en fonction des fonctionnalités agronomiques.
- Agrégats de séquence temporelle générés automatiquement pour la génération rapide de modèles.
- Moteur Azure Data Factory intégré pour créer facilement des pipelines personnalisés de traitement de données.

## <a name="application-development"></a>Développement d’applications

Les API FarmBeats incluent une documentation technique Swagger. Pour plus d’informations sur toutes les API et leurs demandes ou réponses correspondantes, consultez [Swagger](https://aka.ms/FarmBeatsDatahubSwagger).

Le tableau suivant récapitule tous les objets et toutes les ressources dans FarmBeats Datahub :

| Objets et ressources | Description
--- | ---|
Farm | Farm correspond à un emplacement physique intéressant au sein du système FarmBeats. Chaque exploitation agricole a un nom Farm et un ID d’exploitation agricole unique. |
Appareil  | Device correspond à un appareil physique présent dans l’exploitation agricole. Chaque appareil a une identité d’appareil unique. Un appareil est généralement approvisionné dans une exploitation agricole avec un ID d’exploitation agricole.
DeviceModel  | DeviceModel correspond aux métadonnées de l’appareil, telles que le fabricant et le type d’appareil (passerelle ou nœud).
Capteur  | Sensor correspond à un capteur physique qui enregistre des valeurs. Un capteur est généralement connecté à un appareil avec une identité d’appareil.
SensorModel  | SensorModel correspond aux métadonnées du capteur, telles que le fabricant, le type de capteur (analogique ou numérique) et la mesure effectuée par le capteur (température ambiante, pression, etc.).
Télémétrie  | Telemetry offre la possibilité de lire des messages de télémétrie pour un capteur et un intervalle de temps particuliers.
Travail  | Job correspond à n’importe quel workflow des activités exécutées dans le système FarmBeats pour obtenir la sortie souhaitée. Chaque travail est associé à un ID et type de travail.
JobType  | JobType correspond à différents types de travail pris en charge par le système. Les types de travaux définis par le système ou par l’utilisateur sont inclus.
ExtendedType  | ExtendedType correspond à la liste des types définis par le système ou l’utilisateur figurant dans le système. ExtendedType permet de configurer un nouveau type de capteur, de scène ou de fichier de scène dans le système FarmBeats.
Partenaire  | Partner correspond au partenaire d’intégration de capteur et d’imagerie pour FarmBeats.
Scene  | Scene correspond à toute sortie générée dans le contexte d’une exploitation agricole. Chaque scène est associée à un ID de scène, à une source de scène, à un type de scène et à un ID d’exploitation agricole. Chaque ID de scène peut être associé à plusieurs fichiers de scène.
SceneFile |SceneFile correspond à tous les fichiers qui sont générés pour une seule scène. Un ID de scène unique peut être associé à plusieurs ID SceneFile.
Règle  |Rule correspond à une condition pour que les données liées à l’exploitation agricole déclenchent une alerte. Chaque règle figure dans le contexte des données d’une exploitation agricole.
Alerte  | Alert correspond à une notification, qui est générée lorsqu’une condition de règle est remplie. Chaque alerte figure dans le contexte d’une règle.
RoleDefinition  | RoleDefinition définit des actions autorisées et interdites pour un rôle.
RoleAssignment  |RoleAssignment correspond à l’assignation d’un rôle à un utilisateur ou à un principal de service.

### <a name="data-format"></a>Format de données

JSON est un format de données courant, indépendant du langage, qui fournit une représentation textuelle simple de structures de données arbitraires. Pour plus d’informations, rendez-vous sur le [site web de JSON](https://www.json.org/).

## <a name="authentication-and-authorization"></a>Authentification et autorisation

Les requêtes HTTP envoyées à l’API REST sont protégées par Azure Active Directory (Azure AD).
Pour effectuer une requête authentifiée auprès des API REST, le code client requiert une authentification à l’aide d’informations d’identification valides avant de pouvoir appeler l’API. L’authentification est coordonnée entre les différents acteurs par Azure AD. Elle fournit à votre client un jeton d’accès comme preuve de l’authentification. Le jeton est alors envoyé dans l’en-tête d’autorisation HTTP des requêtes de l’API REST. Pour en savoir plus sur l’authentification Azure AD, consultez [Azure Active Directory](https://portal.azure.com) pour les développeurs.

Le jeton d’accès doit être envoyé dans les requête d’API suivantes, dans la section de l’en-tête, comme suit :

```http
headers = {"Authorization": "Bearer " + **access_token**}
```

### <a name="http-request-headers"></a>En-têtes de requête HTTP

Voici les en-têtes de requête les plus courants qui doivent être spécifiés lors d’un appel d’API à Azure FarmBeats Datahub.


**En-tête** | **Description et exemple**
--- | ---
Content-Type  | Format de la demande (Content-Type: application/<format>). Pour les API Azure FarmBeats Datahub, le format est JSON. Content-Type: application/json
Autorisation  | Spécifie le jeton d’accès requis pour effectuer un appel d’API. Autorisation : Porteur <jeton-accès>
Acceptation | Format de la réponse. Pour les API Azure FarmBeats Datahub, le format est JSON. Accept: application/json

### <a name="api-requests"></a>Requêtes d’API

Pour effectuer une requête d’API REST, vous devez combiner la méthode HTTP (GET, POST, PUT ou DELETE), l’URL du service d’API, l’URI de la ressource à interroger, à laquelle envoyer les données, à mettre à jour ou à supprimer, puis ajouter un ou plusieurs en-têtes de requête HTTP.

L’URL du service d’API est votre URL Datahub, par exemple, https://\<yourdatahub-website-name>.azurewebsites.net.

Si vous le souhaitez, vous pouvez inclure des paramètres de requête à des appels GET pour filtrer les données dans les réponses, en limiter la taille et les trier.

L’exemple de requête suivant est utilisé pour obtenir la liste des appareils :

```bash
curl -X GET "https://microsoft-farmbeats.azurewebsites.net/Device" -H "Content-Type: application/json" -H "Authorization: Bearer <Access-Token>”
```

La plupart des appels GET, POST et PUT nécessitent un corps de demande JSON.

L’exemple de requête suivant crée un appareil. Cette requête comporte du code JSON d’entrée avec le corps de la requête.

```bash
curl -X POST "https://microsoft-farmbeats.azurewebsites.net/Device" -H  "accept: application/json" -H  "Content-Type: application/json" -H "Authorization: Bearer <Access-Token>" -d "{  \"deviceModelId\": \"ID123\",  \"hardwareId\": \"MHDN123\",  \"reportingInterval\": 900,  \"name\": \"Device123\",  \"description\": \"Test Device 123\",}"
```

### <a name="query-parameters"></a>Paramètres de requête

Pour les appels GET REST, vous pouvez filtrer les données, en limiter la taille et les trier dans une réponse d’API en incluant un ou plusieurs paramètres de requête dans l’URI de requête. Pour les paramètres de requête, consultez la documentation de l’API et la section sur les appels GET.
Par exemple, lors de l’interrogation de la liste des appareils (appel GET sur /Device), les paramètres de requête suivants peuvent être spécifiés :

![Liste des appareils](./media/references-for-azure-farmbeats/query-parameters-device-1.png)

### <a name="error-handling"></a>Gestion des erreurs

Les API Azure FarmBeats Datahub retournent les erreurs HTTP standard. Les codes d’erreur les plus courants sont les suivants :

 |Code d'erreur             | Description |
 |---                    | --- |
 |200                    | Succès |
 |201                    | Réussite de la création (Post) |
 |400                    | Demande incorrecte. Il existe une erreur dans la requête. |
 |401                    | Non autorisé. L’appelant de l’API n’est pas autorisé à accéder à la ressource. |
 |404                    | Ressource introuvable |
 |5XX                    | Erreur interne du serveur. Les codes d’erreur commençant par 5XX signifient qu’il y a une erreur sur le serveur. Pour plus d’informations, reportez-vous aux journaux du serveur et à la section suivante. |


En plus des erreurs HTTP standard, les API Azure FarmBeats Datahub retournent également des erreurs internes dans le format suivant :

```json
    {
      "message": "<More information on the error>",
      "status": "<error code>”,
      "code": "<InternalErrorCode>",
      "moreInfo": "<Details of the error>"
    }
```

Dans cet exemple, lorsqu’une exploitation agricole a été créée, le champ obligatoire « Name » n’a pas été spécifié dans la charge utile d’entrée. Le message d’erreur obtenu est le suivant :

 ```json    
    {
      "message": "Model validation failed",
      "status": 400,
      "code": "ModelValidationFailed",
      "moreInfo": "[\"The Name field is required.\"]"
    }
  ```

## <a name="add-users-or-app-registrations-to-azure-active-directory"></a>Ajouter des utilisateurs ou des inscriptions d’applications à Azure Active Directory

Les API Azure FarmBeats sont accessibles à un utilisateur ou à une inscription d’application dans Azure Active Directory. Pour créer une inscription d’application dans Azure Active Directory, procédez comme suit :

1. Accédez au [Portail Azure](https://portal.azure.com) et sélectionnez **Azure Active Directory** > **Inscriptions d’applications** > **Nouvelle inscription** . Vous pouvez également utiliser un compte existant.
2. Pour un nouveau compte, procédez comme suit :

    - Entrez un nom.
    - Sélectionnez **Comptes dans ce répertoire d’organisation uniquement (Locataire unique)** .
    - Utilisez les valeurs par défaut dans les autres champs.
    - Sélectionnez **Inscription** .

3. Dans le volet **Vue d’ensemble** de l’inscription des applications nouvelles et existantes, procédez comme suit :

    - Capturez l’ **ID client** et l’ **ID de locataire** .
    - Accédez à **Certificats et secrets** pour générer une nouvelle clé secrète client et capturer la **Client-Secret** .
    - Revenez à **Vue d’ensemble** , puis cliquez sur le lien à côté de **Gérer l’application dans le répertoire local** .
    - Accédez à **Propriétés** pour capturer l’ **ID d’objet** .

4. Accédez à votre Datahub Swagger (https://<yourdatahub>.azurewebsites.net/swagger/index.html) et procédez comme suit :
    - Accédez à l’ **API RoleAssignment** .
    - Effectuez un appel POST afin de créer un objet **RoleAssignment** pour l’ **ID d’objet** que vous venez de créer.
 
```json
{
  "roleDefinitionId": "a400a00b-f67c-42b7-ba9a-f73d8c67e433",
  "objectId": "objectId from step 3 above",
  "objectIdType": "ServicePrincipalId",
  "tenantId": "tenant id of your Azure subscription"
}
```

  > [!NOTE]
  > Pour plus d’informations sur l’ajout d’utilisateurs et l’inscription Active Directory, consultez [Azure Active Directory](../../active-directory/develop/howto-create-service-principal-portal.md).

Lorsque vous avez terminé les étapes précédentes, l’inscription de votre application (client) peut appeler les API Azure FarmBeats à l’aide d’un jeton d’accès via l’authentification du porteur.

Utilisez le jeton d’accès pour l’envoyer dans les requête d’API suivantes dans la section d’en-tête comme suit :

```http
headers = {"Authorization": "Bearer " + **access_token**, "Content-Type" : "application/json" }
```