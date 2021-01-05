---
title: Référence des métadonnées de configuration Azure Blockchain Workbench
description: Vue d’ensemble des métadonnées de configuration de l’application Azure Blockchain Workbench Preview.
ms.date: 12/09/2019
ms.topic: article
ms.reviewer: brendal
ms.openlocfilehash: f0ba19bf1d7fdf05014ac199fae9392b5c3249d1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87073070"
---
# <a name="azure-blockchain-workbench-configuration-reference"></a>Référence de configuration Azure Blockchain Workbench

Les applications Azure Blockchain Workbench désignent des workflows multiniveaux définis par des métadonnées de configuration et un code de contrat intelligent. Les métadonnées de configuration définissent les workflows généraux et le modèle d’interaction de l’application blockchain. Les contrats intelligents définissent la logique métier de l’application blockchain. Workbench utilise la configuration et le code de contrat intelligent pour générer des expériences utilisateur d’application blockchain.

Les métadonnées de configuration spécifient les informations suivantes pour chaque application blockchain :

* Nom et description de l’application blockchain
* Rôles uniques pour les utilisateurs qui peuvent agir ou participer au sein de l’application blockchain
* Un ou plusieurs workflows. Chaque workflow agit comme une machine à états pour contrôler le flux de la logique métier. Les workflows peuvent être indépendants ou interagir entre eux.

Chaque workflow défini spécifie les éléments suivants :

* Nom et description du workflow
* États du workflow.  Chaque état est une étape dans le flux de contrôle de la logique métier. 
* Actions de transition à l’état suivant
* Rôles d’utilisateur autorisés à initier chaque action
* Contrats intelligents qui représentent la logique métier dans les fichiers de code

## <a name="application"></a>Application

Une application blockchain contient des métadonnées de configuration, des workflows et des rôles utilisateur qui peuvent agir ou participer au sein de l’application.

| Champ | Description | Obligatoire |
|-------|-------------|:--------:|
| ApplicationName | Nom d’application unique. Le contrat intelligent correspondant doit utiliser la même propriété **ApplicationName** pour la classe de contrat applicable.  | Oui |
| DisplayName | Nom d’affichage convivial de l’application. | Oui |
| Description | Description de l'application. | Non |
| ApplicationRoles | Collection d’[ApplicationRoles](#application-roles). Rôles utilisateur autorisés à agir ou à participer au sein de l’application.  | Oui |
| Workflows | Collection de [Workflows](#workflows). Chaque workflow agit comme une machine à états pour contrôler le flux de la logique métier. | Oui |

Pour obtenir un exemple, consultez l’[exemple de fichier de configuration](#configuration-file-example).

## <a name="workflows"></a>Workflows

La logique métier d’une application peut être modélisée comme une machine à états dans laquelle la réalisation d’une action entraîne le transfert du flux de logique métier d’un état à un autre. Un workflow est une collection de ces états et actions. Chaque workflow se compose d’un ou plusieurs contrats intelligents, qui représentent la logique métier dans les fichiers de code. Un contrat exécutable est une instance de workflow.

| Champ | Description | Obligatoire | Longueur maximale |
|-------|-------------|:--------:|-----------:|
| Nom | Nom unique du workflow. Le contrat intelligent correspondant doit utiliser la même propriété **Name** pour la classe de contrat applicable. | Oui | 50 |
| DisplayName | Nom d’affichage convivial du workflow. | Oui | 255 |
| Description | Description du workflow. | Non | 255 |
| Initiateurs | Collection d’[ApplicationRoles](#application-roles). Rôles attribués aux utilisateurs qui sont autorisés à créer des contrats dans le workflow. | Oui | |
| StartState | Nom de l’état initial du workflow. | Oui | |
| Propriétés | Collection d’[identificateurs](#identifiers). Représente des données qui peuvent être lues hors de la chaîne ou visualisées dans un outil d’expérience utilisateur. | Oui | |
| Constructeur | Définit les paramètres d’entrée pour la création d’une instance du workflow. | Oui | |
| Fonctions | Collection de [fonctions](#functions) qui peut être exécutée dans le workflow. | Oui | |
| États | Collection d’[états](#states) de workflow. | Oui | |

Pour obtenir un exemple, consultez l’[exemple de fichier de configuration](#configuration-file-example).

## <a name="type"></a>Type

Types de données pris en charge.

| Type | Description |
|-------|-------------|
| address  | Type d’adresse blockchain, par exemple *contracts* ou *users*. |
| tableau    | Tableau à un seul niveau de type integer, bool, money ou time. Les tableaux peuvent être statiques ou dynamiques. Utilisez **ElementType** pour spécifier le type de données des éléments dans le tableau. Consultez [l’exemple de configuration](#example-configuration-of-type-array). |
| bool     | Type de données Boolean. |
| contract | Adresse de type contract. |
| enum     | Ensemble énuméré de valeurs nommées. Lorsque vous utilisez le type enum, vous spécifiez également une liste de valeurs d’énumération. Chaque valeur est limitée à 255 caractères. Les valeurs peuvent contenir des lettres majuscules et minuscules (A-Z, a-z) et des chiffres (0-9). Consultez [l’exemple de configuration et d’utilisation de Solidity](#example-configuration-of-type-enum). |
| int      | Type de données Integer. |
| money    | Type de données Money. |
| state    | État du workflow. |
| string  | Type de données String. 4 000 caractères au maximum. Consultez [l’exemple de configuration](#example-configuration-of-type-string). |
| utilisateur     | Adresse de type user. |
| time     | Type de données Time. |
|`[ Application Role Name ]`| N’importe quel nom spécifié dans le rôle d’application. Limite les utilisateurs à ce type de rôle. |

### <a name="example-configuration-of-type-array"></a>Exemple de configuration de type array

```json
{
  "Name": "Quotes",
  "Description": "Market quotes",
  "DisplayName": "Quotes",
  "Type": {
    "Name": "array",
    "ElementType": {
        "Name": "int"
    }
  }
}
```

#### <a name="using-a-property-of-type-array"></a>Utilisation d’une propriété de type array

Si vous définissez une propriété de type array dans la configuration, vous devez inclure une fonction get explicite qui retourne la propriété public du type array dans Solidity. Par exemple :

```
function GetQuotes() public constant returns (int[]) {
     return Quotes;
}
```

### <a name="example-configuration-of-type-string"></a>Exemple de configuration de type chaîne

``` json
{
  "Name": "description",
  "Description": "Descriptive text",
  "DisplayName": "Description",
  "Type": {
    "Name": "string"
  }
}
```

### <a name="example-configuration-of-type-enum"></a>Exemple de configuration de type énumération

``` json
{
  "Name": "PropertyType",
  "DisplayName": "Property Type",
  "Description": "The type of the property",
  "Type": {
    "Name": "enum",
    "EnumValues": ["House", "Townhouse", "Condo", "Land"]
  }
}
```

#### <a name="using-enumeration-type-in-solidity"></a>Utilisation du type énumération dans Solidity

Lorsqu’une énumération est définie dans la configuration, vous pouvez utiliser des types d’énumération dans Solidity. Par exemple, vous pouvez définir une énumération appelée PropertyTypeEnum.

```
enum PropertyTypeEnum {House, Townhouse, Condo, Land} PropertyTypeEnum public PropertyType; 
```

Pour être valide, la liste de chaînes doit être identique dans la configuration et le contrat intelligent afin d’apparaître comme déclarations cohérentes dans Blockchain Workbench.

Exemple d’attribution :

```
PropertyType = PropertyTypeEnum.Townhouse;
```

Exemple de paramètre de fonction : 

``` 
function AssetTransfer(string description, uint256 price, PropertyTypeEnum propertyType) public
{
    InstanceOwner = msg.sender;
    AskingPrice = price;
    Description = description;
    PropertyType = propertyType;
    State = StateType.Active;
    ContractCreated();
}

```

## <a name="constructor"></a>Constructeur

Définit les paramètres d’entrée pour une instance de workflow.

| Champ | Description | Obligatoire |
|-------|-------------|:--------:|
| Paramètres | Collection d’[identificateurs](#identifiers) requise pour initialiser un contrat intelligent. | Oui |

### <a name="constructor-example"></a>Exemple de constructeur

``` json
{
  "Parameters": [
    {
      "Name": "description",
      "Description": "The description of this asset",
      "DisplayName": "Description",
      "Type": {
        "Name": "string"
      }
    },
    {
      "Name": "price",
      "Description": "The price of this asset",
      "DisplayName": "Price",
      "Type": {
        "Name": "money"
      }
    }
  ]
}
```

## <a name="functions"></a>Fonctions

Définit les fonctions qui peuvent être exécutées dans le workflow.

| Champ | Description | Obligatoire | Longueur maximale |
|-------|-------------|:--------:|-----------:|
| Nom | Nom unique de la fonction. Le contrat intelligent correspondant doit utiliser la même propriété **Name** pour la fonction applicable. | Oui | 50 |
| DisplayName | Nom d’affichage convivial de la fonction. | Oui | 255 |
| Description | Description de la fonction | Non | 255 |
| Paramètres | Collection d’[identificateurs](#identifiers) correspondant aux paramètres de la fonction. | Oui | |

### <a name="functions-example"></a>Exemples de fonctions

``` json
"Functions": [
  {
    "Name": "Modify",
    "DisplayName": "Modify",
    "Description": "Modify the description/price attributes of this asset transfer instance",
    "Parameters": [
      {
        "Name": "description",
        "Description": "The new description of the asset",
        "DisplayName": "Description",
        "Type": {
          "Name": "string"
        }
      },
      {
        "Name": "price",
        "Description": "The new price of the asset",
        "DisplayName": "Price",
        "Type": {
          "Name": "money"
        }
      }
    ]
  },
  {
    "Name": "Terminate",
    "DisplayName": "Terminate",
    "Description": "Used to cancel this particular instance of asset transfer",
    "Parameters": []
  }
]

```

## <a name="states"></a>États

Collection d’états uniques au sein d’un workflow. Chaque état capture une étape dans le workflow de la logique métier. 

| Champ | Description | Obligatoire | Longueur maximale |
|-------|-------------|:--------:|-----------:|
| Nom | Nom unique de l’état. Le contrat intelligent correspondant doit utiliser la même propriété **Name** pour l’état applicable. | Oui | 50 |
| DisplayName | Nom d’affichage convivial de l’état. | Oui | 255 |
| Description | Description de l’état. | Non | 255 |
| PercentComplete | Valeur entière qui apparaît dans l’interface utilisateur Blockchain Workbench pour afficher la progression dans le flux de contrôle de logique métier. | Oui | |
| Style | Indicateur visuel qui indique si l’état représente un état de réussite ou d’échec. Il existe deux valeurs valides : `Success` ou `Failure`. | Oui | |
| Transitions | Collection de [transitions](#transitions) disponibles de l’état actuel vers l’ensemble d’états suivant. | Non | |

### <a name="states-example"></a>Exemple d’états

``` json
"States": [
    {
      "Name": "Active",
      "DisplayName": "Active",
      "Description": "The initial state of the asset transfer workflow",
      "PercentComplete": 20,
      "Style": "Success",
      "Transitions": [
        {
          "AllowedRoles": [],
          "AllowedInstanceRoles": [ "InstanceOwner" ],
          "Description": "Cancels this instance of asset transfer",
          "Function": "Terminate",
          "NextStates": [ "Terminated" ],
          "DisplayName": "Terminate Offer"
        },
        {
          "AllowedRoles": [ "Buyer" ],
          "AllowedInstanceRoles": [],
          "Description": "Make an offer for this asset",
          "Function": "MakeOffer",
          "NextStates": [ "OfferPlaced" ],
          "DisplayName": "Make Offer"
        },
        {
          "AllowedRoles": [],
          "AllowedInstanceRoles": [ "InstanceOwner" ],
          "Description": "Modify attributes of this asset transfer instance",
          "Function": "Modify",
          "NextStates": [ "Active" ],
          "DisplayName": "Modify"
        }
      ]
    },
    {
      "Name": "Accepted",
      "DisplayName": "Accepted",
      "Description": "Asset transfer process is complete",
      "PercentComplete": 100,
      "Style": "Success",
      "Transitions": []
    },
    {
      "Name": "Terminated",
      "DisplayName": "Terminated",
      "Description": "Asset transfer has been canceled",
      "PercentComplete": 100,
      "Style": "Failure",
      "Transitions": []
    }
  ]
```

## <a name="transitions"></a>Transitions

Actions disponibles à l’état suivant. Un ou plusieurs rôles d’utilisateur peuvent exécuter une action à chaque état, où une action peut passer d’un état à un autre dans le workflow. 

| Champ | Description | Obligatoire |
|-------|-------------|:--------:|
| AllowedRoles | Liste des rôles d’application autorisés à initier la transition. Tous les utilisateurs du rôle spécifié peuvent effectuer l’action. | Non |
| AllowedInstanceRoles | Liste des rôles d’utilisateur participant ou spécifiés dans le contrat intelligent, autorisés à initier la transition. Les rôles d’instance sont définis dans **Propriétés** au sein des workflows. AllowedInstanceRoles représente un utilisateur appartenant à une instance dans un contrat intelligent. AllowedInstanceRoles vous permet d’empêcher un rôle d’utilisateur d’effectuer une action dans une instance de contrat.  Par exemple, vous souhaitez que seul l’utilisateur qui a créé le contrat (InstanceOwner) puisse le clore plutôt que tous les utilisateurs appartenant au type de rôle (Propriétaire) si vous avez spécifié le rôle dans AllowedRoles. | Non |
| DisplayName | Nom d’affichage convivial de la transition. | Oui |
| Description | Description de la transition. | Non |
| Fonction | Nom de la fonction permettant d’initier la transition. | Oui |
| NextStates | Collection d’états potentiels après une transition réussie. | Oui |

### <a name="transitions-example"></a>Exemple de transitions

``` json
"Transitions": [
  {
    "AllowedRoles": [],
    "AllowedInstanceRoles": [ "InstanceOwner" ],
    "Description": "Cancels this instance of asset transfer",
    "Function": "Terminate",
    "NextStates": [ "Terminated" ],
    "DisplayName": "Terminate Offer"
  },
  {
    "AllowedRoles": [ "Buyer" ],
    "AllowedInstanceRoles": [],
    "Description": "Make an offer for this asset",
    "Function": "MakeOffer",
    "NextStates": [ "OfferPlaced" ],
    "DisplayName": "Make Offer"
  },
  {
    "AllowedRoles": [],
    "AllowedInstanceRoles": [ "InstanceOwner" ],
    "Description": "Modify attributes of this asset transfer instance",
    "Function": "Modify",
    "NextStates": [ "Active" ],
    "DisplayName": "Modify"
  }
]

```

## <a name="application-roles"></a>Rôles d’application

Les rôles d’application définissent un ensemble de rôles qui peuvent être assignés aux utilisateurs souhaitant agir ou participer au sein de l’application. Les rôles d’application peuvent servir à limiter les actions et la participation dans l’application blockchain et les workflows correspondants. 

| Champ | Description | Obligatoire | Longueur maximale |
|-------|-------------|:--------:|-----------:|
| Nom | Nom unique du rôle d’application. Le contrat intelligent correspondant doit utiliser la même propriété **Name** pour le rôle applicable. Les noms de type de base sont réservés. Vous ne pouvez pas nommer un rôle d’application de la même manière que [Type](#type)| Oui | 50 |
| Description | Description du rôle d’application. | Non | 255 |

### <a name="application-roles-example"></a>Exemple de rôles d’application

``` json
"ApplicationRoles": [
  {
    "Name": "Appraiser",
    "Description": "User that signs off on the asset price"
  },
  {
    "Name": "Buyer",
    "Description": "User that places an offer on an asset"
  }
]
```
## <a name="identifiers"></a>Identificateurs

Les identificateurs représentent une collection d’informations utilisées pour décrire les propriétés de workflow, le constructeur et les paramètres de fonction. 

| Champ | Description | Obligatoire | Longueur maximale |
|-------|-------------|:--------:|-----------:|
| Nom | Nom unique de la propriété ou du paramètre. Le contrat intelligent correspondant doit utiliser la même propriété **Name** pour la propriété ou le paramètre applicable. | Oui | 50 |
| DisplayName | Nom d’affichage convivial pour la propriété ou le paramètre. | Oui | 255 |
| Description | Description de la propriété ou du paramètre. | Non | 255 |
| Type | [Type de données](#type) de propriété. | Oui |

### <a name="identifiers-example"></a>Exemple d’identificateurs

``` json
"Properties": [
  {
    "Name": "State",
    "DisplayName": "State",
    "Description": "Holds the state of the contract",
    "Type": {
      "Name": "state"
    }
  },
  {
    "Name": "Description",
    "DisplayName": "Description",
    "Description": "Describes the asset being sold",
    "Type": {
      "Name": "string"
    }
  }
]
```

## <a name="configuration-file-example"></a>Exemple de fichier de configuration

Le transfert de ressources est un scénario de contrat intelligent pour l’achat et la vente de ressources très précieuses, pour lequel un inspecteur et un expert sont nécessaires. Les vendeurs peuvent répertorier leurs ressources en instanciant un contrat intelligent de transfert de ressources. Les acheteurs peuvent faire des offres en effectuant une action sur le contrat intelligent, et d’autres parties peuvent effectuer des actions pour inspecter ou estimer la ressource. Quand la ressource est marquée comme inspectée et estimée, l’acheteur et le vendeur confirmeront une nouvelle fois la vente avant que le contrat ne soit défini sur l’état Terminé. À chaque étape du processus, l’ensemble des participants peuvent connaître l’état du contrat dès qu’il est mis à jour. 

Pour obtenir plus d’informations, y compris les fichiers de code, consultez [l’exemple de transfert de ressources pour Azure Blockchain Workbench](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/application-and-smart-contract-samples/asset-transfer)

Le fichier de configuration suivant est destiné à l’exemple de transfert de ressources :

``` json
{
  "ApplicationName": "AssetTransfer",
  "DisplayName": "Asset Transfer",
  "Description": "Allows transfer of assets between a buyer and a seller, with appraisal/inspection functionality",
  "ApplicationRoles": [
    {
      "Name": "Appraiser",
      "Description": "User that signs off on the asset price"
    },
    {
      "Name": "Buyer",
      "Description": "User that places an offer on an asset"
    },
    {
      "Name": "Inspector",
      "Description": "User that inspects the asset and signs off on inspection"
    },
    {
      "Name": "Owner",
      "Description": "User that signs off on the asset price"
    }
  ],
  "Workflows": [
    {
      "Name": "AssetTransfer",
      "DisplayName": "Asset Transfer",
      "Description": "Handles the business logic for the asset transfer scenario",
      "Initiators": [ "Owner" ],
      "StartState":  "Active",
      "Properties": [
        {
          "Name": "State",
          "DisplayName": "State",
          "Description": "Holds the state of the contract",
          "Type": {
            "Name": "state"
          }
        },
        {
          "Name": "Description",
          "DisplayName": "Description",
          "Description": "Describes the asset being sold",
          "Type": {
            "Name": "string"
          }
        },
        {
          "Name": "AskingPrice",
          "DisplayName": "Asking Price",
          "Description": "The asking price for the asset",
          "Type": {
            "Name": "money"
          }
        },
        {
          "Name": "OfferPrice",
          "DisplayName": "Offer Price",
          "Description": "The price being offered for the asset",
          "Type": {
            "Name": "money"
          }
        },
        {
          "Name": "InstanceAppraiser",
          "DisplayName": "Instance Appraiser",
          "Description": "The user that appraises the asset",
          "Type": {
            "Name": "Appraiser"
          }
        },
        {
          "Name": "InstanceBuyer",
          "DisplayName": "Instance Buyer",
          "Description": "The user that places an offer for this asset",
          "Type": {
            "Name": "Buyer"
          }
        },
        {
          "Name": "InstanceInspector",
          "DisplayName": "Instance Inspector",
          "Description": "The user that inspects this asset",
          "Type": {
            "Name": "Inspector"
          }
        },
        {
          "Name": "InstanceOwner",
          "DisplayName": "Instance Owner",
          "Description": "The seller of this particular asset",
          "Type": {
            "Name": "Owner"
          }
        }
      ],
      "Constructor": {
        "Parameters": [
          {
            "Name": "description",
            "Description": "The description of this asset",
            "DisplayName": "Description",
            "Type": {
              "Name": "string"
            }
          },
          {
            "Name": "price",
            "Description": "The price of this asset",
            "DisplayName": "Price",
            "Type": {
              "Name": "money"
            }
          }
        ]
      },
      "Functions": [
        {
          "Name": "Modify",
          "DisplayName": "Modify",
          "Description": "Modify the description/price attributes of this asset transfer instance",
          "Parameters": [
            {
              "Name": "description",
              "Description": "The new description of the asset",
              "DisplayName": "Description",
              "Type": {
                "Name": "string"
              }
            },
            {
              "Name": "price",
              "Description": "The new price of the asset",
              "DisplayName": "Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Terminate",
          "DisplayName": "Terminate",
          "Description": "Used to cancel this particular instance of asset transfer",
          "Parameters": []
        },
        {
          "Name": "MakeOffer",
          "DisplayName": "Make Offer",
          "Description": "Place an offer for this asset",
          "Parameters": [
            {
              "Name": "inspector",
              "Description": "Specify a user to inspect this asset",
              "DisplayName": "Inspector",
              "Type": {
                "Name": "Inspector"
              }
            },
            {
              "Name": "appraiser",
              "Description": "Specify a user to appraise this asset",
              "DisplayName": "Appraiser",
              "Type": {
                "Name": "Appraiser"
              }
            },
            {
              "Name": "offerPrice",
              "Description": "Specify your offer price for this asset",
              "DisplayName": "Offer Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Reject",
          "DisplayName": "Reject",
          "Description": "Reject the user's offer",
          "Parameters": []
        },
        {
          "Name": "AcceptOffer",
          "DisplayName": "Accept Offer",
          "Description": "Accept the user's offer",
          "Parameters": []
        },
        {
          "Name": "RescindOffer",
          "DisplayName": "Rescind Offer",
          "Description": "Rescind your placed offer",
          "Parameters": []
        },
        {
          "Name": "ModifyOffer",
          "DisplayName": "Modify Offer",
          "Description": "Modify the price of your placed offer",
          "Parameters": [
            {
              "Name": "offerPrice",
              "DisplayName": "Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Accept",
          "DisplayName": "Accept",
          "Description": "Accept the inspection/appraisal results",
          "Parameters": []
        },
        {
          "Name": "MarkInspected",
          "DisplayName": "Mark Inspected",
          "Description": "Mark the asset as inspected",
          "Parameters": []
        },
        {
          "Name": "MarkAppraised",
          "DisplayName": "Mark Appraised",
          "Description": "Mark the asset as appraised",
          "Parameters": []
        }
      ],
      "States": [
        {
          "Name": "Active",
          "DisplayName": "Active",
          "Description": "The initial state of the asset transfer workflow",
          "PercentComplete": 20,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancels this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate Offer"
            },
            {
              "AllowedRoles": [ "Buyer" ],
              "AllowedInstanceRoles": [],
              "Description": "Make an offer for this asset",
              "Function": "MakeOffer",
              "NextStates": [ "OfferPlaced" ],
              "DisplayName": "Make Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Modify attributes of this asset transfer instance",
              "Function": "Modify",
              "NextStates": [ "Active" ],
              "DisplayName": "Modify"
            }
          ]
        },
        {
          "Name": "OfferPlaced",
          "DisplayName": "Offer Placed",
          "Description": "Offer has been placed for the asset",
          "PercentComplete": 30,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Accept the proposed offer for the asset",
              "Function": "AcceptOffer",
              "NextStates": [ "PendingInspection" ],
              "DisplayName": "Accept Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you previously placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Modify the price that you specified for your offer",
              "Function": "ModifyOffer",
              "NextStates": [ "OfferPlaced" ],
              "DisplayName": "Modify Offer"
            }
          ]
        },
        {
          "Name": "PendingInspection",
          "DisplayName": "Pending Inspection",
          "Description": "Asset is pending inspection",
          "PercentComplete": 40,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceInspector" ],
              "Description": "Mark this asset as inspected",
              "Function": "MarkInspected",
              "NextStates": [ "Inspected" ],
              "DisplayName": "Mark Inspected"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceAppraiser" ],
              "Description": "Mark this asset as appraised",
              "Function": "MarkAppraised",
              "NextStates": [ "Appraised" ],
              "DisplayName": "Mark Appraised"
            }
          ]
        },
        {
          "Name": "Inspected",
          "DisplayName": "Inspected",
          "PercentComplete": 45,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceAppraiser" ],
              "Description": "Mark this asset as appraised",
              "Function": "MarkAppraised",
              "NextStates": [ "NotionalAcceptance" ],
              "DisplayName": "Mark Appraised"
            }
          ]
        },
        {
          "Name": "Appraised",
          "DisplayName": "Appraised",
          "Description": "Asset has been appraised, now awaiting inspection",
          "PercentComplete": 45,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceInspector" ],
              "Description": "Mark the asset as inspected",
              "Function": "MarkInspected",
              "NextStates": [ "NotionalAcceptance" ],
              "DisplayName": "Mark Inspected"
            }
          ]
        },
        {
          "Name": "NotionalAcceptance",
          "DisplayName": "Notional Acceptance",
          "Description": "Asset has been inspected and appraised, awaiting final sign-off from buyer and seller",
          "PercentComplete": 50,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "SellerAccepted" ],
              "DisplayName": "SellerAccept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "BuyerAccepted" ],
              "DisplayName": "BuyerAccept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            }
          ]
        },
        {
          "Name": "BuyerAccepted",
          "DisplayName": "Buyer Accepted",
          "Description": "Buyer has signed-off on inspection and appraisal",
          "PercentComplete": 75,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "SellerAccepted" ],
              "DisplayName": "Accept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            }
          ]
        },
        {
          "Name": "SellerAccepted",
          "DisplayName": "Seller Accepted",
          "Description": "Seller has signed-off on inspection and appraisal",
          "PercentComplete": 75,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "Accepted" ],
              "DisplayName": "Accept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            }
          ]
        },
        {
          "Name": "Accepted",
          "DisplayName": "Accepted",
          "Description": "Asset transfer process is complete",
          "PercentComplete": 100,
          "Style": "Success",
          "Transitions": []
        },
        {
          "Name": "Terminated",
          "DisplayName": "Terminated",
          "Description": "Asset transfer has been canceled",
          "PercentComplete": 100,
          "Style": "Failure",
          "Transitions": []
        }
      ]
    }
  ]
}
```
## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Référence de l’API REST Azure Blockchain Workbench](/rest/api/azure-blockchain-workbench)
