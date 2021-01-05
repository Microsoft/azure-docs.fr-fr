---
title: Transformations du provisionnement cloud Azure AD Connect
description: Cet article explique comment utiliser les transformations pour modifier les mappages d’attributs par défaut.
author: billmath
ms.author: billmath
manager: davba
ms.date: 12/02/2019
ms.topic: how-to
ms.prod: windows-server-threshold
ms.technology: identity-adfs
ms.openlocfilehash: a15a25ce4fbe1a43c2a5065bf88f5c40c5865721
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85360688"
---
# <a name="transformations"></a>Transformations

Une transformation vous permet de modifier le comportement par défaut de la synchronisation d’un attribut avec Azure Active Directory (Azure AD) à l’aide du provisionnement cloud.

Pour effectuer cette tâche, vous devez modifier le schéma, puis le soumettre de nouveau par le biais d’une requête web.

Pour plus d’informations sur les attributs de provisionnement cloud, consultez [Comprendre le schéma Azure AD](concept-attributes.md).


## <a name="retrieve-the-schema"></a>Récupérer la sortie
Pour récupérer le schéma, effectuez les étapes décrites dans [Afficher le schéma](concept-attributes.md#view-the-schema). 

## <a name="custom-attribute-mapping"></a>Mappage des attributs personnalisés
Pour ajouter un mappage d’attributs personnalisés, effectuez les étapes suivantes.

1. Copiez le schéma dans un éditeur de texte ou de code, tel que [Visual Studio Code](https://code.visualstudio.com/).
1. Recherchez l’objet que vous souhaitez mettre à jour dans le schéma.

   ![Objet dans le schéma](media/how-to-transformation/transform1.png)</br>
1. Recherchez le code pour `ExtensionAttribute3` sous l’objet utilisateur.

    ```
                            {
                                "defaultValue": null,
                                "exportMissingReferences": false,
                                "flowBehavior": "FlowWhenChanged",
                                "flowType": "Always",
                                "matchingPriority": 0,
                                "targetAttributeName": "ExtensionAttribute3",
                                "source": {
                                    "expression": "Trim([extensionAttribute3])",
                                    "name": "Trim",
                                    "type": "Function",
                                    "parameters": [
                                        {
                                            "key": "source",
                                            "value": {
                                                "expression": "[extensionAttribute3]",
                                                "name": "extensionAttribute3",
                                                "type": "Attribute",
                                                "parameters": []
                                            }
                                        }
                                    ]
                                }
                            },
    ```
1. Modifiez le code afin que l’attribut « société » soit mappé à `ExtensionAttribute3`.

   ```
                                    {
                                        "defaultValue": null,
                                        "exportMissingReferences": false,
                                        "flowBehavior": "FlowWhenChanged",
                                        "flowType": "Always",
                                        "matchingPriority": 0,
                                        "targetAttributeName": "ExtensionAttribute3",
                                        "source": {
                                            "expression": "Trim([company])",
                                            "name": "Trim",
                                            "type": "Function",
                                            "parameters": [
                                                {
                                                    "key": "source",
                                                    "value": {
                                                        "expression": "[company]",
                                                        "name": "company",
                                                        "type": "Attribute",
                                                        "parameters": []
                                                    }
                                                }
                                            ]
                                        }
                                    },
   ```
 1. Recopiez le schéma dans l’Explorateur Graph, remplacez le **Type de requête** par **PUT** et sélectionnez **Exécuter la requête**.

    ![Exécuter la requête](media/how-to-transformation/transform2.png)

 1. À présent, dans le portail Azure, accédez à la configuration du provisionnement cloud et sélectionnez **Redémarrer le provisionnement**.

    ![Redémarrer le provisionnement](media/how-to-transformation/transform3.png)

 1. Après quelques instants, vérifiez que les attributs sont remplis en exécutant la requête suivante dans l’Afficheur Graph : `https://graph.microsoft.com/beta/users/{Azure AD user UPN}`.
 1. Vous devez maintenant voir la valeur.

    ![La valeur s’affiche.](media/how-to-transformation/transform4.png)

## <a name="custom-attribute-mapping-with-function"></a>Mappage d’attributs personnalisés avec fonction
Pour un mappage plus poussé, vous pouvez utiliser des fonctions permettant de manipuler les données et de créer des valeurs pour que les attributs soient adaptés aux besoins de votre organisation.

Pour effectuer cette tâche, effectuez les étapes précédentes, puis modifiez la fonction qui est utilisée pour construire la valeur finale.

Pour obtenir des informations sur la syntaxe, et des exemples d’expressions, consultez [Écriture d’expressions pour les mappages d’attributs dans Azure Active Directory](reference-expressions.md).


## <a name="next-steps"></a>Étapes suivantes 

- [Présentation du provisionnement](what-is-provisioning.md)
- [Présentation du provisionnement cloud Azure AD Connect](what-is-cloud-provisioning.md)
