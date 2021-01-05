---
title: Créer ou modifier une offre – Place de marché Azure
description: API pour créer ou mettre à jour une offre.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/29/2020
ms.openlocfilehash: dba47073a6676b17c091af048f583ca3e7712999
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87420224"
---
# <a name="create-or-modify-an-offer"></a>Créer ou modifier une offre

> [!NOTE]
> Les API de Portail Cloud Partner sont intégrées à Espace partenaires et continueront d’y fonctionner. La transition introduit de légères modifications. Passez en revue les changements répertoriés dans les [informations de référence relatives aux API de Portail Cloud Partner](./cloud-partner-portal-api-overview.md) pour vous assurer que votre code continue de fonctionner après la transition vers Espace partenaires. Les API de Portail Cloud Partner doivent uniquement être utilisées pour les produits existants intégrés avant la transition vers Espace partenaires. Les nouveaux produits doivent utiliser les API de soumission d’Espace partenaires.

Cet appel met à jour une offre spécifique dans l’espace de noms du serveur de publication ou en crée une.

  `PUT https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>?api-version=2017-10-31`

## <a name="uri-parameters"></a>Paramètres URI

|  **Nom**         |  **Description**                      |  **Type de données**  |
|  --------         |  ----------------                     |  -------------  |
| publisherId       |  Identificateur du serveur de publication, par exemple `contoso` |   String |
| offerId           |  Identificateur de l’offre                     |   String        |
| api-version       |  Version la plus récente de l’API            |   Date           |
|  |  |  |

## <a name="header"></a>En-tête

|  **Nom**        |  **Valeur**               |
|  ---------       |  ----------              | 
| Content-Type     | `application/json`       |
| Autorisation    | `Bearer YOUR_TOKEN`      |
|  |  |

## <a name="body-example"></a>Exemple de corps

L’exemple suivant illustre la création d’une offre ayant l’ID `contosovirtualmachine`.

### <a name="request"></a>Requête

``` json
  {
      "publisherId": "contoso",
      "offerTypeId": "microsoft-azure-virtualmachines",
      "id": "contosovirtualmachine",
      "offerTypeVersions": {
          "microsoft-azure-virtualmachines": 87,
          "microsoft-azure-marketplace": 39
      },
      "definition": {
          "displayText": "Contoso Virtual Machine Offer",
          "offer": {
          "microsoft-azure-marketplace.title": "Contoso App",
          "microsoft-azure-marketplace.summary": "Contoso App makes dev ops a breeze",
          "microsoft-azure-marketplace.longSummary": "Contoso App makes dev ops a breeze",
          "microsoft-azure-marketplace.description": "Contoso App makes dev ops a breeze",
          "microsoft-azure-marketplace.offerMarketingUrlIdentifier": "contosoapp",
          "microsoft-azure-marketplace.allowedSubscriptions": ["59160c40-2e25-4dcf-a2fd-6514cb08bf08"],
          "microsoft-azure-marketplace.usefulLinks": [{
              "linkTitle": "Contoso App for Azure",
              "linkUrl": "https://azuremarketplace.microsoft.com"
          }],
          "microsoft-azure-marketplace.categoryMap": [
                {
                    "categoryL1": "analytics",
                    "categoryL2-analytics": [
                        "visualization-and-reporting"
                    ]
                },
                {
                    "categoryL1": "ai-plus-machine-learning",
                    "categoryL2-ai-plus-machine-learning":[
                        "bot-services",
                        "cognitive-services",
                        "other"
                    ]
                }
            ],
          "microsoft-azure-marketplace.smallLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/6218c455-9cbc-450c-9920-f2e7a69ee132.png?sv=2014-02-14&sr=b&sig=6O8MM9dgiJ48VK0MwddkyVbprRAnBszyhVkVHGShhkI%3D&se=2019-03-28T19%3A46%3A50Z&sp=r",
          "microsoft-azure-marketplace.mediumLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/557e714b-2f31-4e12-b0cc-e48dd840edf4.png?sv=2014-02-14&sr=b&sig=NwL67NTQf9Gc9VScmZehtbHXpYmxhwZc2foy3o4xavs%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
          "microsoft-azure-marketplace.largeLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/142485da-784c-44cb-9523-d4f396446258.png?sv=2014-02-14&sr=b&sig=xaMxhwx%2FlKYfz33mJGIg8UBdVpsOwVvqhjTJ883o0iY%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
          "microsoft-azure-marketplace.wideLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/48af9013-1df7-4c94-8da8-4626e5039ce0.png?sv=2014-02-14&sr=b&sig=%2BnN7f2tprkrqb45ID6JlT01zXcy1PMTkWXtLKD6nfoE%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
          "microsoft-azure-marketplace.heroLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/c46ec74d-d214-4fb5-9082-3cee55200eba.png?sv=2014-02-14&sr=b&sig=RfDvjowFGpP4WZGAHylbF2CuXwO2NXOrwycrXEJvJI4%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
          "microsoft-azure-marketplace.screenshots": [],
          "microsoft-azure-marketplace.videos": [],
          "microsoft-azure-marketplace.leadDestination": "None",
          "microsoft-azure-marketplace.privacyURL": "https://azuremarketplace.microsoft.com",
          "microsoft-azure-marketplace.termsOfUse": "Terms of use",
          "microsoft-azure-marketplace.engineeringContactName": "Jon Doe",
          "microsoft-azure-marketplace.engineeringContactEmail": "jondoe@outlook.com",
          "microsoft-azure-marketplace.engineeringContactPhone": "555-555-5555",
          "microsoft-azure-marketplace.supportContactName": "Jon Doe",
          "microsoft-azure-marketplace.supportContactEmail": "jondoe@outlook.com",
          "microsoft-azure-marketplace.supportContactPhone": "555-555-5555",
          "microsoft-azure-marketplace.publicAzureSupportUrl": "",
          "microsoft-azure-marketplace.fairfaxSupportUrl": ""
      },
      "plans": 
      [
          {
              "planId": "contososkuidentifier",
              "microsoft-azure-virtualmachines.skuTitle": "Contoso App",
              "microsoft-azure-virtualmachines.skuSummary": "Contoso App makes dev ops a breeze.",
              "microsoft-azure-virtualmachines.skuDescription": "This is a description for the Contoso App that makes dev ops a breeze.",
              "microsoft-azure-virtualmachines.hideSKUForSolutionTemplate": false,
              "microsoft-azure-virtualmachines.cloudAvailability": ["PublicAzure"],
              "virtualMachinePricing": {
                  "isByol": true,
                  "freeTrialDurationInMonths": 0
              },
              "microsoft-azure-virtualmachines.operatingSystemFamily": "Windows",
              "microsoft-azure-virtualmachines.windowsOSType": "Other",
              "microsoft-azure-virtualmachines.operationSystem": "Contoso App",
              "microsoft-azure-virtualmachines.recommendedVMSizes": ["a0-basic", "a0-standard", "a1-basic", "a1-standard", "a2-basic", "a2-standard"],
              "microsoft-azure-virtualmachines.openPorts": [],
              "microsoft-azure-virtualmachines.vmImages": 
              {
                  "1.0.1": 
                  {
                      "osVhdUrl": "http://contosoteststorage.blob.core.windows.net/test/contosoVM.vhd?sv=2014-02-14&sr=c&sig=WlDo6Q4xwYH%2B5QEJbItPUVdgHhBcrVxPBmntZ2vU96w%3D&st=2016-06-25T18%3A30%3A00Z&se=2017-06-25T18%3A30%3A00Z&sp=rl",
                      "lunVhdDetails": []
                  }
              },
              "regions": ["AZ"]
          }
          ]
      },
      "eTag": "W/\"datetime'2017-06-07T06%3A15%3A40.4771399Z'\"",
      "version": 5
  }
```

### <a name="response"></a>response

``` json
 {
         "offerTypeId": "microsoft-azure-virtualmachines",
         "publisherId": "contoso",
         "status": "neverPublished",
         "id": "contosovirtualmachine",
         "version": 1,
         "definition": {
         "displayText": "Contoso Virtual Machine Offer",
         "offer": 
         {
             "microsoft-azure-marketplace-testdrive.videos": [],
             "microsoft-azure-marketplace.title": "Contoso App",
             "microsoft-azure-marketplace.summary": "Contoso App makes dev ops a breeze",
             "microsoft-azure-marketplace.longSummary": "Contoso App makes dev ops a breeze",
             "microsoft-azure-marketplace.description": "Contoso App makes dev ops a breeze",
             "microsoft-azure-marketplace.offerMarketingUrlIdentifier": "contosoapp",
             "microsoft-azure-marketplace.allowedSubscriptions": 
             [
                 "59160c40-2e25-4dcf-a2fd-6514cb08bf08"
             ],
             "microsoft-azure-marketplace.usefulLinks": 
             [
                 {
                     "linkTitle": "Contoso App for Azure",
                     "linkUrl": "https://azuremarketplace.microsoft.com"
                 }
             ],
             "microsoft-azure-marketplace.categoryMap":
             [
                 {
                    "categoryL1": "analytics",
                    "categoryL2-analytics": [
                    "visualization-and-reporting"
                    ]
                 },
                 {
                    "categoryL1": "ai-plus-machine-learning",
                    "categoryL2-ai-plus-machine-learning": [
                    "bot-services",
                    "cognitive-services",
                    "other"
                    ]
                 }
             ],
             "microsoft-azure-marketplace.smallLogo": "https://publishingstoredm.blob.core.windows.net/prodcontent/D6191_publishers_marketplace:2Dtest/testaoffer/8affcd28-60a5-4839-adf8-c560e069fd61.png?sv=2014-02-14&sr=b&sig=nGErAgn%2BDUecrX892wcmk32kh0MHgIZeJ5jcKyY%2Fuew%3D&se=2020-03-28T22%3A27%3A13Z&sp=r",
             "microsoft-azure-marketplace.mediumLogo": "https://publishingstoredm.blob.core.windows.net/prodcontent/D6191_publishers_marketplace:2Dtest/testaoffer/39550bca-1110-432c-9ea9-e12b3a2288cd.png?sv=2014-02-14&sr=b&sig=4X0hlkXYtuZOmcYq%2BsbYVZz3k5k26kngcFX6yBAJjNI%3D&se=2020-03-28T22%3A27%3A13Z&sp=r",
             "microsoft-azure-marketplace.largeLogo": "https://publishingstoredm.blob.core.windows.net/prodcontent/D6191_publishers_marketplace:2Dtest/testaoffer/ce3576e3-df12-4074-b0a3-0b8d3f329df1.png?sv=2014-02-14&sr=b&sig=mFhtCUQh%2FbFz10nlIWbqsz6jq5MBZ0M%2F5cIREE9P6V0%3D&se=2020-03-28T22%3A27%3A13Z&sp=r",
             "microsoft-azure-marketplace.wideLogo": "https://publishingstoredm.blob.core.windows.net/prodcontent/D6191_publishers_marketplace:2Dtest/testaoffer/476d6edd-12d3-4414-9def-d2970c4a9de4.png?sv=2014-02-14&sr=b&sig=pg4MDSZjAb8w8D%2FrQ9RT%2BodpynSy%2FlYOvpx0yeam2Bw%3D&se=2020-03-28T22%3A27%3A13Z&sp=r",
             "microsoft-azure-marketplace.heroLogo": "https://publishingstoredm.blob.core.windows.net/prodcontent/D6191_publishers_marketplace:2Dtest/testaoffer/46c85b7b-4438-4e0d-8218-24fb5651727a.png?sv=2014-02-14&sr=b&sig=wIsCOO5%2BDj8NsLVSwwzwTgogF71oA7Q1XjKhNB1ni5g%3D&se=2020-03-28T22%3A27%3A13Z&sp=r",
             "microsoft-azure-marketplace.screenshots": [],
             "microsoft-azure-marketplace.videos": [],
             "microsoft-azure-marketplace.leadDestination": "None",
             "microsoft-azure-marketplace.tableLeadConfiguration": {},
             "microsoft-azure-marketplace.blobLeadConfiguration": {},
             "microsoft-azure-marketplace.salesForceLeadConfiguration": {},
             "microsoft-azure-marketplace.crmLeadConfiguration": {},
             "microsoft-azure-marketplace.httpsEndpointLeadConfiguration": {},
             "microsoft-azure-marketplace.marketoLeadConfiguration": {},
             "microsoft-azure-marketplace.privacyURL": "https://azuremarketplace.microsoft.com",
             "microsoft-azure-marketplace.termsOfUse": "Terms of use",
             "microsoft-azure-marketplace.engineeringContactName": "Jon Doe",
             "microsoft-azure-marketplace.engineeringContactEmail": "jondoe@outlook.com",
             "microsoft-azure-marketplace.engineeringContactPhone": "555-555-5555",
             "microsoft-azure-marketplace.supportContactName": "Jon Doe",
             "microsoft-azure-marketplace.supportContactEmail": "jondoe@outlook.com",
             "microsoft-azure-marketplace.supportContactPhone": "555-555-5555",
             "microsoft-azure-marketplace.publicAzureSupportUrl": "",
             "microsoft-azure-marketplace.fairfaxSupportUrl": ""
         },
         "plans": 
         [
             {
                 "planId": "contososkuidentifier",
                 "microsoft-azure-virtualmachines.skuTitle": "Contoso App (Old Title)",
                 "microsoft-azure-virtualmachines.skuSummary": "Contoso App makes dev ops a breeze.",
                 "microsoft-azure-virtualmachines.skuDescription": "This is a description for the Contoso App that makes dev ops a breeze.",
                 "microsoft-azure-virtualmachines.hideSKUForSolutionTemplate": false,
                 "microsoft-azure-virtualmachines.cloudAvailability": 
                 [
                     "PublicAzure"
                 ],
                 "microsoft-azure-virtualmachines.certificationsFairfax": [],
                 "virtualMachinePricing": {
                     "isByol": true,
                     "freeTrialDurationInMonths": 0
                 },
                 "microsoft-azure-virtualmachines.operatingSystemFamily": "Windows",
                 "microsoft-azure-virtualmachines.operationSystem": "Contoso App",
                 "microsoft-azure-virtualmachines.recommendedVMSizes": 
                 [
                     "a0-basic",
                     "a0-standard",
                     "a1-basic",
                     "a1-standard",
                     "a2-basic",
                     "a2-standard"
                 ],
                 "microsoft-azure-virtualmachines.openPorts": [],
                 "microsoft-azure-virtualmachines.vmImages": 
                 {
                     "1.0.1": 
                     {
                         "osVhdUrl": "http://contosoteststorage.blob.core.windows.net/test/contosoVM.vhd?sv=2014-02-14&sr=c&sig=WlDo6Q4xwYH%2B5QEJbItPUVdgHhBcrVxPBmntZ2vU96w%3D&st=2016-06-25T18%3A30%3A00Z&se=2017-06-25T18%3A30%3A00Z&sp=rl",
                         "lunVhdDetails": []
                     }
                 },
                 "regions":
                 [
                     "AZ"
                 ]
             } 
         ]
     },
     "changedTime": "2018-03-28T22:27:13.8363879Z"
 }
```

> [!NOTE]
> Pour modifier cette offre, ajoutez un en-tête **If-Match** défini sur * à la requête ci-dessus. Utilisez le corps de la même requête que ci-dessus, mais modifiez les valeurs comme vous le souhaitez. 

### <a name="response-status-codes"></a>Codes d’état de réponse

| **Code**  |  **Description**                                                                            |
| --------  |  ---------------                                                                            |
|  200      | `OK`. La requête a été traitée avec succès et l’offre a bien été modifiée.           |
|  201      | `Created`. La requête a été traitée avec succès et l’offre a bien été modifiée.   |
|  400      | `Bad/Malformed request`. Le corps de la réponse d’erreur pourrait fournir plus d’informations.            |
|  403      | `Forbidden`. Le client n’a pas accès à l’espace de noms demandé.                     |
|  404      | `Not found`. L’entité auquel le client fait référence n’existe pas.                           |
|  412      | Le serveur ne remplit pas l’une des préconditions que le demandeur a spécifiées dans la requête. Le client doit vérifier l’ETAG envoyé avec la requête. |
|  |  |

## <a name="uploading-artifacts"></a>Chargement des artefacts

Les artefacts, tels que les images et les logos, doivent être partagées après chargement dans un emplacement accessible sur le web, puis par inclusion de chacun d’eux en tant qu’URI dans la requête PUT, comme dans l’exemple ci-dessus. Le système détecte que ces fichiers ne sont pas présents dans le stockage de la Place de marché Azure et télécharge ces fichiers dans le stockage.  Par conséquent, vous constaterez que les futures requêtes GET retourneront une URL de service de la Place de marché Azure pour ces fichiers.

## <a name="categories-and-industries"></a>Catégories et secteurs d’activité

Lorsque vous créez une offre, vous devez spécifier une catégorie pour votre offre sur la Place de marché. Si vous le souhaitez, pour certains types d’offre, vous pouvez également spécifier des secteurs d’activité. En fonction du type d’offre, indiquez les catégories/secteurs d’activité applicables à l’offre en utilisant des valeurs de clés spécifiques des tables suivantes.

### <a name="azure-marketplace-categories"></a>Catégories Place de marché Azure

Ces catégories et leurs clés respectives sont applicables pour les applications Azure, les machines virtuelles, les machines virtuelles principales, les conteneurs, les applications de conteneur, les modules IoT Edge et les offres SaaS. Les éléments en italique et gras (comme ***analytics***) sont des catégories, et les éléments de texte standard (comme les informations sur les données) sont les sous-catégories situées en dessous. Utilisez les valeurs de clé exactes, sans modifier l’espacement ou la casse.

| Category | Clés SaaS | Clés Azure App | Machine virtuelle, conteneurs, applications de conteneur, modules IoT Edge, clés de machines virtuelles principales |
| --- | --- | --- | --- |
| ***Analyse*** | ***analytique*** | ***analytics-azure-apps*** | ***analytics-amp*** |
| Insights sur les données | data-insights | data-insights | data-insights |
| Analytique des données | data-analytics | data-analytics | data-analytics |
| Big Data | big-data | bigData | big-data |
| Analyse prédictive | predictive-analytics | predictive-analytics | predictive-analytics |
| Analytique en temps réel/streaming | real-time-streaming-analytics | real-time-streaming-analytics | real-time-streaming-analytics |
| Autres | Autres | other-analytics | Autres |
| ***IA + Machine Learning*** | ***ArtificialIntelligence*** | ***ai-plus-machine-learning*** | ***ai-plus-machine-learning*** |
| Bot Services | bot-services | bot-services | bot-services |
| Cognitive Services | cognitive-services | cognitive-services | cognitive-services |
| ML Service | ml-service | ml-service | ml-service |
| ML automatisé | automated-ml | automated-ml | automated-ml |
| Automatisation robotisée des processus/Automatisation des processus métier | business-robotic-process-automation | business-robotic-process-automation | business-robotic-process-automation |
| Étiquetage des données | data-labelling | data-labelling | data-labelling |
| Préparation des données | data-preparation | data-preparation | data-preparation |
| Exploration des connaissances | knowledge-mining | knowledge-mining | knowledge-mining |
| Opérations ML | ml-operations | ml-operations | ml-operations |
| Autres | other-AI-plus-machine-learning | Autres | Autres |
| ***Blockchain*** | ***blockchain*** | ***blockchain*** | ***blockchain*** |
| Accélérateurs d’application | app-accelerators | app-accelerators | app-accelerators |
| Registre mononœud | single-node-ledger | single-node-ledger | single-node-ledger |
| Registre multinœud | multi-node-ledger | multi-node-ledger | multi-node-ledger |
| Outils | outils | outils | outils |
| Autres | Autres | Autres | Autres |
| ***Calcul*** | ***compute-saas*** | ***compute-azure-apps*** | ***calcul*** |
| Infrastructure d’application | appInfra | appInfrastructure | application-infrastructure |
| Systèmes d'exploitation | clientOS | clientOS | operating-systems |
| Cache | cache | cache | cache |
| Autres | other-compute | other-compute | Autres |
| ***Containers*** | ***conteneurs*** | ***conteneurs*** | ***conteneurs*** |
| Applications de conteneur | container-apps | container-apps | container-apps |
| Images conteneur | container-images | container-images | container-images |
| Prise en main des conteneurs | get-started-with-containers | get-started-with-containers | get-started-with-containers |
| Autres | Autres | Autres | Autres |
| ***Bases de données*** | ***databases-saas*** | ***database*** | ***bases de données*** |
| Bases de données NoSQL | nosql-databases | nosql-databases | nosql-databases |
| Bases de données relationnelles | relational-databases | relational-databases | relational-databases |
| Bases de données de registre/blockchain | ledger-blockchain-databases | ledger-blockchain-databases | ledger-blockchain-databases |
| Lacs de données | data-lakes | data-lakes | data-lakes |
| entrepôt de données | data-warehouse | data-warehouse | data-warehouse |
| Autres | other-databases | other-databases | Autres |
| ***Outils de développement*** | ***developer-tools-saas*** | ***developer-tools-azure-apps*** | ***developer-tools*** |
| Outils | tools-developer-tools | tools-developer-tools | tools-developer-tools |
| Scripts | Scripts | Scripts | Scripts |
| Service de développement | devService | devService | developer-service |
| Autres | other-developer-tools | other-developer-tools | Autres |
| ***DevOps*** | ***devops*** | ***devops*** | ***devops*** |
| Autres | Autres | Autres | Autres |
| ***Identité*** | ***identity*** | ***identity*** | ***identity*** |
| Gestion de l’accès | access-management | access-management | access-management |
| Autres | Autres | Autres | Autres |
| ***Intégration*** | ***intégration*** | ***intégration*** | ***intégration*** |
| Messagerie | messagerie | messagerie | messagerie |
| Autres | Autres | Autres | Autres |
| ***Internet des objets*** | ***IoT*** | ***internet-of-things-azure-apps*** | ***internet-of-things*** |
| IoT Core Services | N/A | iot-core-services | iot-core-services |
| Modules IoT Edge | N/A | iot-edge-modules | iot-edge-modules |
| Solutions IoT | iot-solutions | iot-solutions | iot-solutions |
| Analytique données et visualisation | data-analytics-and-visualization | data-analytics-and-visualization | data-analytics-and-visualization |
| Connectivité IoT | iot-connectivity | iot-connectivity | iot-connectivity |
| Autres | other-internet-of-things | other-internet-of-things | Autres |
| ***Outils informatiques et de gestion*** | ***ITandAdministration*** | ***it-and-management-tools-azure-apps*** | ***it-and-management-tools*** |
| Solutions de gestion | management-solutions | management-solutions | management-solutions |
| Applications métier | businessApplication | businessApplication | business-applications |
| Autres | other-it-management-tools | other-it-management-tools | Autres |
| ***Supervision et diagnostics*** | ***monitoring-and-diagnostics*** | ***monitoring-and-diagnostics*** | ***monitoring-and-diagnostics*** |
| Autres | Autres | Autres | Autres |
| ***Média*** | ***media*** | ***media*** | ***media*** |
| Media Services | media-services | media-services | media-services |
| Protection du contenu | content-protection | content-protection | content-protection |
| Streaming en direct et à la demande | live-and-on-demand-streaming | live-and-on-demand-streaming | live-and-on-demand-streaming |
| Autres | Autres | Autres | Autres |
| ***Migration*** | ***migration*** | ***migration*** | ***migration*** |
| Migration de données | data-migration | data-migration | data-migration |
| Autres | Autres | Autres | Autres |
| ***Réalité mixte*** | ***mixed-reality*** | ***mixed-reality*** | ***mixed-reality*** |
| Autres | Autres | Autres | Autres |
| ***Mise en réseau*** | ***réseau*** | ***réseau*** | ***réseau*** |
| Gestionnaires d’appliances | appliance-managers | appliance-managers | appliance-managers |
| Connectivité | connectivité | connectivité | connectivité |
| Pare-feux | Pare-feu | Pare-feu | Pare-feu |
| Équilibreurs de charge | load-balancers | load-balancers | load-balancers |
| Autres | Autres | Autres | Autres |
| ***Sécurité*** | ***security*** | ***security*** | ***security*** |
| Gestion des identités et des accès | identity-and-access-management | identity-and-access-management | identity-and-access-management |
| Protection contre les menaces | threat-protection | threat-protection | threat-protection |
| Information Protection | information-protection | information-protection | information-protection |
| Autres | Autres | Autres | Autres |
| ***Stockage*** | ***storage-saas*** | ***storage-azure-apps*** | ***storage*** |
| Sauvegarde et récupération | sauvegarde | sauvegarde | backup-and-recovery |
| Stockage hybride d’entreprise | enterprise-hybrid-storage | enterprise-hybrid-storage | enterprise-hybrid-storage |
| Partage de fichiers | file-sharing | file-sharing | file-sharing |
| Gestion du cycle de vie des données | data-lifecycle-management | data-lifecycle-management | data-lifecycle-management |
| Autres | other-storage | other-storage | Autres |
| ***Web*** | ***web*** | ***web*** | ***web*** |
| Blogs et CMS | blogs-and-cmss | blogs-and-cmss | blogs-and-cmss |
| Applications web de démarrage | starter-web-apps | starter-web-apps | starter-web-apps |
| Ecommerce | ecommerce | ecommerce | ecommerce |
| Frameworks Web Apps | web-apps-frameworks | web-apps-frameworks | web-apps-frameworks |
| Web Apps | web-apps | web-apps | web-apps |
| Autres | Autres | Autres | Autres |
||||

### <a name="microsoft-appsource-categories"></a>Catégories Microsoft AppSource

Ces catégories et leurs clés respectives s’appliquent aux types d’offres SaaS, applications PowerBI, Dynamics 365 Business Central, Dynamics 365 for Customer Engagement et Dynamics 365 for Operation. Les éléments en italique et gras (comme ***analytics***) sont des catégories, et les éléments de texte standard (comme advanced-analytics) sont les sous-catégories situées en dessous. Utilisez les valeurs de clé exactes, sans modifier l’espacement ou la casse.

| Category | Clés SaaS | Clés Dynamics 365 Business Central, Dynamics 365 for Customer Engagement, Dynamics 365 for Operation | Clés d’application PowerBI |
| --- | --- | --- | --- |
| ***Analyse*** | ***analytique*** | ***Analyse*** | ***Analyse*** |
| Analytique avancée | advanced-analytics | advanced-analytics | advanced-analytics |
| Visualisation & rapport | visualization-reporting | visualization-reporting | visualization-reporting |
| Autres | Autres | other-analytics | other-analytics |
| ***IA + Machine Learning*** | ***ArtificialIntelligence*** | ***ai-plus-machine-learning-dynamics*** | ***ai-plus-machine-learning-appsource*** |
| IA pour entreprises | ai-for-business | ai-for-business | ai-for-business |
| Applications bot | bot-apps | bot-apps | bot-apps |
| Autres | other-ai-plus-machine-learning | other-ai-plus-machine-learning | other-ai-plus-machine-learning |
| ***Collaboration*** | ***Collaboration*** | ***Collaboration*** | ***collaboration*** |
| Contact et personnes | contact-people | contact-people | contact-and-people |
| Gestion des réunions | meeting-management | meeting-management | meeting-management |
| Conception et gestion de site | site-design-management | site-design-management | site-design-and-management |
| Gestion de tâche et de projet | task-project-management | task-project-management | task-and-project-management |
| Audioconférence et vidéoconférence | voice-video-conferencing | voice-video-conferencing | voice-and-video-conferencing |
| Autres | other-collaboration | other-collaboration | Autres |
| ***Conformité et informations légales*** | ***conformité*** | ***conformité*** | ***compliance-and-legal*** |
| Fiscalité et audit | tax-audit | tax-audit | tax-and-audit |
| Informations juridiques | Informations juridiques | Informations juridiques | legal |
| Données, gouvernance et confidentialité | data-governance-privacy | data-governance-privacy | data-governance-and-privacy |
| Intégrité et sécurité | health-safety | health-safety | health-and-safety |
| Autres | other-compliance-legal | other-compliance-legal | Autres |
| ***Service clients*** | ***CustomerService*** | ***CustomerService*** | ***customer-service*** |
| Centre d’appel | contact-center | contact-center | contact-center |
| Service personnalisé | face-to-face-service | face-to-face-service | face-to-face-service |
| Services administratifs et du personnel | back-office-employee-service | back-office-employee-service | back-office-and-employee-service |
| Gestion des dossiers et des connaissances | knowledge-case-management | knowledge-case-management | knowledge-and-case-management |
| Médias sociaux et engagement omnicanal | social-media-omnichannel-engagement | social-media-omnichannel-engagement | social-media-and-omnichannel-engagement |
| Autres | other-customer-service | other-customer-service | Autres |
| ***Finances*** | ***Finances*** | ***Finances*** | ***finance*** |
| Comptabilité | comptabilité | comptabilité | comptabilité |
| Gestion des ressources | asset-management | asset-management | asset-management |
| Analytique, regroupement et rapport | analytics-consolidation-reporting | analytics-consolidation-reporting | analytics-consolidation-and-reporting |
| Crédit et recouvrements | credit-collections | credit-collections | credit-and-collections |
| Gestion de la conformité et des risques | compliance-risk-management | compliance-risk-management | compliance-and-risk-management |
| Autres | other-finance | other-finance | Autres |
| ***Ressources humaines*** | ***HumanResources*** | ***HumanResources*** | ***human-resources*** |
| Acquisition de compétences | talent-acquisition | talent-acquisition | talent-acquisition |
| Gestion des compétences | talent-management | talent-management | talent-management |
| Opérations RH | hr-operations | hr-operations | hr-operations |
| Planification et analytique des effectifs | workforce-planning-analytics | workforce-planning-analytics | workforce-planning-and-analytics |
| Autres | other-human-resources | other-human-resources | Autres |
| ***Internet des objets*** | ***IoT*** | ***internet-of-things-dynamics*** | ***internet-of-things-appsource*** |
| Gestion des ressources et exploitation | asset-management-operations | asset-management-operations | asset-management-and-operations |
| Produits connectés | connected-products | connected-products | connected-products |
| Chaîne d’approvisionnement intelligente | intelligent-supply-chain | intelligent-supply-chain | intelligent-supply-chain |
| Maintenance prédictive | predictive-maintenance | predictive-maintenance | predictive-maintenance |
| Surveillance à distance | remote-monitoring | remote-monitoring | remote-monitoring |
| Sécurité | safety-security | safety-security | safety-and-security |
| Infrastructure et ressources intelligentes | smart-infrastructure-resources | smart-infrastructure-resources | smart-infrastructure-and-resources |
| Véhicules et mobilité | vehicles-mobility | vehicles-mobility | vehicles-and-mobility |
| Autres | other-internet-of-things | other-internet-of-things | Autres |
| ***Outils informatiques et de gestion*** | ***ITandAdministration*** | ***ITandAdministration*** | ***it-and-management-tools*** |
| Solutions de gestion | management-solutions | management-solutions | management-solutions |
| Applications métier | businessApplication | businessApplication | business-applications |
| Autres | other-it-management-tools | other-it-management-tools | Autres |
| ***Marketing*** | ***Marketing*** | ***Marketing*** | ***marketing*** |
| Publication | publication | publication | publication |
| Analytique | analytics-marketing | analytics-marketing | analytics-marketing |
| Gestion et automatisation de campagnes | campaign-management-automation | campaign-management-automation | campaign-management-and-automation |
| Marketing par e-mail | email-marketing | email-marketing | email-marketing |
| L2 - Gestion des événements et des ressources | events-resource-management | events-resource-management | events-and-resource-management |
| Recherche et analyse | research-analytics | research-analytics | research-and-analysis |
| Médias sociaux | social-media | social-media | social-media |
| Autres | other-marketing | other-marketing | Autres |
| ***Opérations et chaîne logistique*** | ***OperationsSupplyChain*** | ***OperationsSupplyChain*** | ***operations-and-supply-chain*** |
| Gestion des ressources et de la production | asset-production-management | asset-production-management | asset-and-production-management |
| Prédiction de la demande | demand-forecasting | demand-forecasting | demand-forecasting |
| Gestion des informations et connectivité | information-management-connectivity | information-management-connectivity | information-management-and-connectivity |
| Planification, achat et rapport | planning-purchasing-reporting | planning-purchasing-reporting | planning-purchasing-and-reporting |
| Management des services et de la qualité | quality-service-management | quality-service-management | quality-and-service-management |
| Gestion des ventes et des commandes | sales-order-management | sales-order-management | sales-and-order-management |
| Gestion du transport et de l’entrepôt | transportation-warehouse-management | transportation-warehouse-management | transportation-and-warehouse-management |
| Autres | other-operations-supply-chain | other-operations-supply-chain | Autres |
| ***Productivité*** | ***Productivité*** | ***Productivité*** | ***productivité*** |
| Création et gestion de contenu | content-creation-management | content-creation-management | content-creation-and-management |
| Langue et traduction | language-translation | language-translation | language-and-translation |
| Gestion de documents | document-management | document-management | document-management |
| Gestion des e-mails | email-management | email-management | email-management |
| Recherche et référence | search-reference | search-reference | search-and-reference |
| Autres | other-productivity | other-productivity | Autres |
| Gamification | Gamification | Gamification | Gamification |
| ***Ventes*** | ***Ventes*** | ***Ventes*** | ***Ventes*** |
| Télévente | télévente | télévente | télévente |
| Configuration, prix, devis | configure-price-quote | configure-price-quote | configure-price-quote |
| Gestion des contrats | contract-management | contract-management | contract-management |
| CRM | crm | crm | crm |
| E-commerce | e-commerce | e-commerce | e-commerce |
| Enrichissement des données métier | business-data-enrichment | business-data-enrichment | business-data-enrichment |
| Aide à la vente | sales-enablement | sales-enablement | sales-enablement |
| Autres | other-sales | other-sales | other-sales |
| ***Géolocalisation*** | ***géolocalisation*** | ***géolocalisation*** | ***géolocalisation*** |
| Cartes | cartes | cartes | cartes |
| Actualités et météo | news-and-weather | news-and-weather | news-and-weather |
| Autres | other-geolocation | other-geolocation | other-geolocation |
||||

### <a name="microsoft-appsource-industries"></a>Secteurs d’activité Microsoft AppSource

Ces secteurs d’activité et leurs clés respectives s’appliquent aux types d’offres SaaS, applications PowerBI, Dynamics 365 Business Central, Dynamics 365 for Customer Engagement et Dynamics 365 for Operation. Les éléments en italique et gras (comme ***Automobile***) sont des catégories, et les éléments de texte standard (comme AutomotiveL2) sont les sous-catégories situées en dessous. Utilisez les valeurs de clé exactes, sans modifier l’espacement ou la casse.

| Secteur d’activité | Clés SaaS, Dynamics 365 Business Central, Dynamics 365 for Customer Engagement, Dynamics 365 for Operations | Clés des applications PowerBI |
| --- | --- | --- |
| ***Automobile*** | ***Automobile*** | ***automotive*** |
| Industrie automobile | AutomotiveL2 | AutomotiveL2 |
| ***Agriculture*** | ***Agriculture*** | ***agriculture*** |
| Autre - Non segmenté | Agriculture\_OtherUnsegmented | other-unsegmented |
| ***Distribution*** | ***Distribution*** | ***distribution*** |
| Vente en gros | Vente en gros | vente en gros |
| Expédition de paquets et colis | ParcelAndPackageShipping | parcel-and-package-shipping |
| ***Education*** | ***Education*** | ***education*** |
| Enseignement supérieur | HigherEducation | higher-education |
| Enseignement primaire et secondaire/K-12 | PrimaryAndSecondaryEducationK12 | primary-and-secondary-education |
| Bibliothèques et musées | LibrariesAndMuseums | libraries-and-museums |
| ***Services financiers*** | ***FinancialServices*** | ***financial-services*** |
| Banques et marchés de capitaux | BankingAndCapitalMarkets | banking-and-capital-markets |
| Assurance | Assurance | assurance |
| ***Secteur public*** | ***Secteur public*** | ***secteur public*** |
| Défense et intelligence | DefenseAndIntelligence | defense-and-intelligence |
| Sécurité publique et justice | PublicSafetyAndJustice | public-safety-and-justice |
| Gouvernement civil | CivilianGovernment | civilian-government |
| ***Santé*** | ***HealthCareandLifeSciences*** | ***healthcare*** |
| Organisme de sécurité sociale | HealthPayor | health-payor |
| Professionnel de santé | HealthProvider | health-provider |
| Produits pharmaceutiques | Produits pharmaceutiques | produits pharmaceutiques |
| ***Industrie et ressources*** | ***Fabrication*** | ***manufacturing-and-resources*** |
| Produits chimiques et agrochimiques | ChemicalAndAgrochemical | chemical-and-agrochemical |
| Fabrication discrète | DiscreteManufacturing | discrete-manufacturing |
| Énergie | Énergie | énergie |
| ***Commerce et produits de consommation*** | ***RetailandConsumerGoods*** | ***retail-and-consumer-goods*** |
| Produits de consommation | ConsumerGoods | consumer-goods |
| Détaillants | Détaillants | détaillants |
| ***Médias et communications*** | ***MediaAndCommunications*** | ***media-and-communications*** |
| Multimédia et divertissement | MediaandEntertainment | media-and-entertainment |
| Télécommunications | Télécommunications | télécommunications |
| ***Services professionnels*** | ***ProfessionalServices*** | ***professional-services*** |
| Informations juridiques | Informations juridiques | legal |
| Services professionnels partenaires | PartnerProfessionalServices | partner-professional-services |
| ***Architecture et construction*** | ***ArchitectureAndConstruction*** | ***architecture-and-construction*** |
| Autre - Non segmenté | ArchitectureAndConstruction\_OtherUnsegmented | other-unsegmented |
| ***Tourisme et voyage*** | ***HospitalityandTravel*** | ***hospitality-and-travel*** |
|    Hôtels et loisirs | HotelsAndLeisure | hotels-and-leisure |
| Transport et voyages | TravelAndTransportation | travel-and-transportation |
| Restaurants et services traiteur | RestaurantsAndFoodServices | restaurants-and-food-services |
| ***Autres secteurs publics*** | ***OtherPublicSectorIndustries*** | ***other-public-sector-industries*** |
| Foresterie et pêche | ForestryAndFishing | forestry-and-fishing |
| Organisations à but non lucratif | Organisations à but non lucratif | organisations à but non lucratif |
| ***Immobilier*** | ***RealEstate*** | ***real-estate*** |
| Autre - Non segmenté | RealEstate\_OtherUnsegmented | other-unsegmented |
|||
