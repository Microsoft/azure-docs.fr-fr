---
title: Tutoriel d’application web – Tester la connexion à l’API Azure pour FHIR
description: Ces tutoriels décrivent en détail un exemple de déploiement d’application web simple. Cette section du tutoriel explique comment tester la connexion au serveur FHIR avec Postman
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 01/03/2020
ms.openlocfilehash: 1c64468a2e420734ca51a5b9308bb52e13712c51
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023310"
---
# <a name="testing-the-fhir-api"></a>Test de l’API FHIR
Au cours des deux étapes précédentes, vous avez déployé l’API Azure pour FHIR et inscrit votre application cliente. Vous êtes maintenant prêt à vérifier que votre API Azure pour FHIR est configurée avec votre application cliente. 

## <a name="retrieve-capability-statement"></a>Récupérer l’énoncé de capacité
Tout d’abord, nous allons obtenir l’énoncé de capacité pour votre API Azure pour FHIR. 
1. Ouvrez Postman.
1. Récupérez l’énoncé de capacité en exécutant la commande GET https://\<FHIR-SERVER-NAME>.azurehealthcareapis.com/metadata. Dans l’image ci-dessous, le nom du serveur FHIR est **fhirserver**.

![Énoncé de capacité](media/tutorial-web-app/postman-capability-statement.png)

Ensuite, nous allons essayer de récupérer un patient. Pour récupérer un patient, entrez la commande GET https://\<FHIR-SERVER-NAME>.azurehealthcareapis.com/Patient. Vous obtenez alors une erreur 401 Non autorisé. Cette erreur est due au fait que vous n’avez pas prouvé que vous étiez habilité à accéder aux données des patients.

## <a name="get-patient-from-fhir-server"></a>Obtenir un patient auprès du serveur FHIR
![Patient en échec](media/tutorial-web-app/postman-patient-authorization-failed.png)

Pour obtenir un accès, vous avez besoin d’un jeton d’accès.
1. Dans Postman, sélectionnez **Authorization** et définissez le paramètre Type sur **OAuth 2.0**.
1. Sélectionnez **Get New Access Token** (Obtenir un nouveau jeton d’accès).
1. Complétez les champs et sélectionnez **Request Token** (Demander un jeton). Les valeurs de chaque champ sont indiquées ci-dessous pour les besoins de ce tutoriel.

|Champ                |Valeur                                                               |
|---------------------|--------------------------------------------------------------------|
|Token Name (Nom du jeton)           |Nom de votre jeton.                                               |
|Grant Type (Type d’autorisation)           |Code d’autorisation                                                  |
|Callback URL (URL de rappel)         |https://www.getpostman.com/oauth2/callback                          |
|Auth URL (URL d’autorisation)             |https://login.microsoftonline.com/\<AZURE-AD-TENANT-ID> /oauth2/?resource=https://\<FHIR-SERVER-NAME>.azurehealthcareapis.com|
|Access Token URL (URL du jeton d’accès)     |https://login.microsoftonline.com/\<AZURE-AD-TENANT-ID> /oauth2/token|
|Client ID (ID client)            |ID client que vous avez copié aux étapes précédentes             |
|Client Secret (Clé secrète client)        |\<BLANK>                                                            |
|Scope (Étendue)                |\<BLANK>                                                            |
|State (État)                |1234                                                                |
|Client Authentication|Envoyer les informations d’identification du client dans le corps                                     |

4. Connectez-vous avec vos informations d’identification, puis sélectionnez **Accept** (Accepter).
1. Accédez au bas des résultats, puis sélectionnez **Use Token** (Utiliser le jeton).
1. Sélectionnez à nouveau **Send** (Envoyer) en haut pour obtenir cette fois le résultat ![Success Patient](media/tutorial-web-app/postman-patient-authorization-success.png) (Succès patient).

## <a name="post-patient-into-fhir-server"></a>Transmettre le patient au serveur FHIR
Maintenant que vous avez un accès, vous pouvez créer un nouveau patient. Voici un exemple de patient simple que vous pouvez ajouter dans votre serveur FHIR. Entrez le code ci-dessous dans la section **Body** (Corps) de Postman.

``` json
    {
        "resourceType": "Patient",
        "active": true,
        "name": [
            {
                "use": "official",
                "family": "Kirk",
                "given": [
                    "James",
                    "Tiberious"
                ]
            },
            {
                "use": "usual",
                "given": [
                    "Jim"
                ]
            }
        ],
        "gender": "male",
        "birthDate": "1960-12-25"
    }
```
Cette commande POST a pour effet de créer un patient dans votre serveur FHIR sous le nom de James Tiberious Kirk.
![Transmission du patient](media/tutorial-web-app/postman-post-patient.png)

Si vous effectuez à nouveau l’étape GET ci-dessus pour récupérer un patient, le nom de James Tiberious Kirk figure alors dans la sortie.

## <a name="troubleshooting-access-issues"></a>Résolution des problèmes d’accès
Si vous avez rencontré des problèmes à l’une de ces étapes, parcourez les documents sur Azure Active Directory et l’API Azure pour FHIR que nous avons réunis. 

* [Azure AD et API Azure pour FHIR](azure-ad-hcapi.md) – Ce document décrit certains principes de base d’Azure Active Directory ainsi que son interaction avec l’API Azure pour FHIR.
* [Validation des jetons d’accès](azure-ad-hcapi-token-validation.md) – Ce guide pratique fournit des détails plus spécifiques sur la validation des jetons d’accès et sur la façon de résoudre les problèmes d’accès.

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous savez comment vous connecter à votre application cliente, vous êtes prêt à écrire votre application web.

>[!div class="nextstepaction"]
>[Écrire une application web](tutorial-web-app-write-web-app.md)



