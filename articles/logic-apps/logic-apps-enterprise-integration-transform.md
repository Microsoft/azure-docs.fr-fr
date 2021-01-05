---
title: Transformer des données XML d’un format vers un autre
description: Créer des transformations ou des mappages qui convertissent des données XML entre les différents formats dans Azure Logic Apps à l’aide d’Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 07/08/2016
ms.openlocfilehash: 038c1d4c0f0b5ffd7b9aabea2de32e3a44e3b221
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97654130"
---
# <a name="create-maps-that-transform-xml-between-formats-in-azure-logic-apps-with-enterprise-integration-pack"></a>Créer des mappages qui convertissent des données XML entre les différents formats dans Azure Logic Apps à l’aide d’Enterprise Integration Pack

Le connecteur Enterprise Integration Transform convertit les données d’un format vers un autre format. Vous pouvez, par exemple, avoir un message entrant qui contient la date du jour au format AnnéeMoisJour. Vous pouvez utiliser une transformation pour remettre en forme la date au format MoisJourAnnée.

## <a name="what-does-a-transform-do"></a>Que fait une transformation ?
Une transformation, également appelée mappage, est composée d’un schéma XML source (l’entrée) et d’un schéma XML cible (la sortie). Vous pouvez utiliser différentes fonctions intégrées pour aider à manipuler ou à contrôler les données, y compris les manipulations de chaînes, les affectations conditionnelles, les expressions arithmétiques, les formateurs d'heure et de date et même les constructions de bouclage.

## <a name="how-to-create-a-transform"></a>Création d’une transformation
Vous pouvez créer une transformation/un mappage à l’aide du [Kit de développement logiciel (SDK) Enterprise Integration](https://aka.ms/vsmapsandschemas)de Visual Studio. Lorsque vous avez terminé de créer et de tester la transformation, vous la téléchargez dans votre compte d’intégration. 

## <a name="how-to-use-a-transform"></a>Utilisation d’une transformation
Après avoir chargé la transformation / le mappage dans votre compte d’intégration, vous pouvez l’utiliser pour créer une application logique. L’application logique exécute vos transformations à chaque fois qu’elle est déclenchée (et qu’il existe du contenu d’entrée qui doit être transformé).

**Voici les étapes à suivre pour utiliser une transformation**:

### <a name="prerequisites"></a>Prérequis

* Créer un compte d’intégration et y ajouter un mappage  

Maintenant que vous avez exécuté la configuration requise, il est temps de créer votre application logique :  

1. Créez une application logique et [liez-la à votre compte d’intégration](./logic-apps-enterprise-integration-create-integration-account.md "Découvrez comment lier un compte d’intégration à une application logique") qui contient le mappage.
2. Ajouter un déclenchement de **Requête** à votre application logique  
   ![Screenshot of the "Show Microsoft managed APIs" dropdown with the Request trigger selected. The dropdown is in a Logic app created using the Visual Studio Enterprise Integration SDK.](./media/logic-apps-enterprise-integration-transforms/transform-1.png)    
3. Ajoutez l’action **Transformer XML** en sélectionnant d’abord **Ajouter une action**   
   ![Screenshot showing the "Add an action" button selected in the Request trigger screen.](./media/logic-apps-enterprise-integration-transforms/transform-2.png)   
4. Entrez le mot *transform* dans la zone de recherche afin de filtrer toutes les actions et d’obtenir celle que vous souhaitez utiliser  
   ![Screenshot showing how to search for the Transform XML action in the "Show Microsoft managed APIs" dropdown so that it can be added to the Request trigger.](./media/logic-apps-enterprise-integration-transforms/transform-3.png)  
5. Sélectionnez l’action **Transformer XML**   
6. Ajoutez le **CONTENU** XML à transformer. Vous pouvez utiliser toute donnée XML que vous recevez dans la requête HTTP en tant que **CONTENU**. Dans cet exemple, sélectionnez le corps de la demande HTTP qui a déclenché l’application logique.

   > [!NOTE]
   > Vérifiez que le contenu de **Transformer XML** est au format XML. Si le contenu n’est pas au format XML ou s’il est codé en base 64, vous devez spécifier une expression qui traite le contenu. Vous pouvez par exemple utiliser des [fonctions](logic-apps-workflow-definition-language.md#functions) comme ```@base64ToBinary``` pour le décodage du contenu ou ```@xml``` pour le traitement du contenu au format XML.
 

7. Sélectionnez le nom du **MAPPAGE** que vous souhaitez utiliser pour effectuer la transformation. Le mappage doit déjà exister dans votre compte d’intégration. Lors d’une étape précédente, vous avez déjà attribué à votre application logique l’accès à votre compte d’intégration qui contient le mappage.      
   ![Screenshot showing the Content and Map fields in the Transform XML screen for the Request trigger.](./media/logic-apps-enterprise-integration-transforms/transform-4.png) 
8. Enregistrez votre travail  
    ![Capture d’écran montrant le bouton Enregistrer dans le concepteur Logic Apps.](./media/logic-apps-enterprise-integration-transforms/transform-5.png) 

À ce stade, vous avez terminé de configurer votre mappage. Dans une application réelle, vous souhaiterez peut-être stocker les données transformées dans une application métier, comme SalesForce. Vous pouvez facilement ajouter une action pour envoyer la sortie de la transformation à SalesForce. 

Vous pouvez maintenant tester votre transformation en effectuant une demande au point de terminaison HTTP.  


## <a name="features-and-use-cases"></a>Fonctionnalités et cas d’usage
* La transformation créée dans un mappage peut être simple, par exemple la copie d'un nom et de l'adresse d'un document vers un autre. Vous pouvez aussi créer des transformations plus complexes à l'aide des opérations de mappage prêtes à l'emploi.  
* Plusieurs fonctions ou opérations de mappage sont disponibles, y compris des chaînes, des fonctions de date et d'heure, et ainsi de suite.  
* Vous pouvez effectuer une copie de données directe entre les schémas. Dans le Mappeur inclus dans le Kit de développement logiciel (SDK), il suffit de dessiner une ligne qui relie les éléments dans le schéma source à leurs équivalents dans le schéma de destination.  
* Lors de la création d’un mappage, sa représentation graphique est affichée, notamment toutes les relations et les liens que vous créez.
* Utilisez la fonctionnalité Tester le mappage pour ajouter un exemple de message XML. Avec un simple clic, vous pouvez tester le mappage que vous avez créé et afficher la sortie générée.  
* Téléchargement de mappages existants  
* Comprend la prise en charge du format XML.

## <a name="advanced-features"></a>Fonctionnalités avancées

### <a name="reference-assembly-or-custom-code-from-maps"></a>Assembly de référence ou code personnalisé à partir de mappages 
L’action de transformation prend également en charge les mappages et les transformations comportant une référence à un assembly externe. Cette fonctionnalité autorise les appels de code .NET personnalisé effectués directement à partir de mappages XSLT. Voici les prérequis à respecter pour utiliser des assemblys dans des mappages.

* Le mappage et l’assembly auquel il fait référence doivent être [chargés sur le compte d’intégration](./logic-apps-enterprise-integration-maps.md). 

  > [!NOTE]
  > Le mappage et l’assembly doivent être chargés dans un ordre précis. Vous devrez charger l’assembly avant de charger le mappage qui y fait référence.

* Le mappage doit également comporter ces attributs et une section CDATA contenant l’appel au code de l’assembly :

    * **name** est le nom d’assembly personnalisé.
    * **namespace** est l’espace de noms de l’assembly qui comprend le code personnalisé.

  Cet exemple montre un mappage qui fait référence à un assembly nommé « XslUtilitiesLib » et appelle la méthode `circumreference` à partir de celui-ci.

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" xmlns:user="urn:my-scripts">
  <msxsl:script language="C#" implements-prefix="user">
    <msxsl:assembly name="XsltHelperLib"/>
    <msxsl:using namespace="XsltHelpers"/>
    <![CDATA[public double circumference(int radius){ XsltHelper helper = new XsltHelper(); return helper.circumference(radius); }]]>
  </msxsl:script>
  <xsl:template match="data">
   <circles>
    <xsl:for-each select="circle">
      <circle>
        <xsl:copy-of select="node()"/>
          <circumference>
            <xsl:value-of select="user:circumference(radius)"/>
          </circumference>
      </circle>
    </xsl:for-each>
   </circles>
  </xsl:template>
    </xsl:stylesheet>
  ```


### <a name="byte-order-mark"></a>Marque d’ordre d’octet
Par défaut, la réponse de la transformation commence par la marque d’ordre d’octet. Cette fonctionnalité n’est disponible que dans l’éditeur en mode Code. Pour désactiver cette fonctionnalité, spécifiez `disableByteOrderMark` pour la propriété `transformOptions` :

```json
"Transform_XML": {
    "inputs": {
        "content": "@{triggerBody()}",
        "integrationAccount": {
            "map": {
                "name": "TestMap"
            }
        },
        "transformOptions": "disableByteOrderMark"
    },
    "runAfter": {},
    "type": "Xslt"
}
```





## <a name="learn-more"></a>En savoir plus
* [En savoir plus sur Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md "En savoir plus sur Enterprise Integration Pack")  
* [En savoir plus sur les mappages](../logic-apps/logic-apps-enterprise-integration-maps.md "En savoir plus sur les mappages d’intégration d’entreprise")  