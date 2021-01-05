---
title: 'Démarrage rapide : Obtenir des insights sur des images avec l’API REST et Go - Recherche visuelle Bing'
titleSuffix: Azure Cognitive Services
description: Découvrez comment charger une image à l’aide de l’API Recherche visuelle Bing et Go, puis récupérez des insights sur l’image.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: quickstart
ms.date: 05/22/2020
ms.author: aahi
ms.openlocfilehash: baa0c66508e63fc4363fbf72f01ed7016f2a30eb
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499099"
---
# <a name="quickstart-get-image-insights-using-the-bing-visual-search-rest-api-and-go"></a>Démarrage rapide : Obtenir des insights sur les images à l’aide de l’API REST Recherche visuelle Bing et de Go

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Utilisez ce guide de démarrage rapide pour effectuer votre premier appel à l’API Recherche visuelle Bing à l’aide du langage de programmation Go. Une requête POST charge une image sur le point de terminaison d’API. Les résultats incluent des URL et des informations descriptives à propos des images similaires à l’image chargée.

## <a name="prerequisites"></a>Prérequis

* Installez les [fichiers binaires Go](https://golang.org/dl/).
* Installez l’imprimante go-spew, qui est utilisée pour afficher les résultats. Pour installer go-spew, utilisez la commande `$ go get -u https://github.com/davecgh/go-spew`.

[!INCLUDE [cognitive-services-bing-visual-search-signup-requirements](../../../../includes/cognitive-services-bing-visual-search-signup-requirements.md)]

## <a name="project-and-libraries"></a>Projet et bibliothèques

Créez un projet Go dans votre IDE ou votre éditeur. Ensuite, importez `net/http` pour les requêtes, `ioutil` pour lire la réponse et `encoding/json` pour gérer le texte JSON des résultats. Utilisez la bibliothèque `go-spew` pour analyser les résultats JSON.

```go
package main

import (
    "bytes"
    "io"
    "fmt"
    "io/ioutil"
    "mime/multipart"
    "net/http"
    "os"
    "path/filepath"
    "encoding/json"
    "github.com/davecgh/go-spew/spew"
)

```

## <a name="struct-to-format-results"></a>Struct pour mettre en forme les résultats

La structure `BingAnswer` met en forme les données retournées dans la réponse JSON, qui est multiniveau et complexe. L’implémentation suivante couvre en partie les points essentiels :

```go
type BingAnswer struct {
    Type         string `json:"_type"`
    Instrumentation struct {
        Type string   `json:"_type"`
        PingUrlBase   string  `json:"_pingUrlBase"`
        PageLoadPingUrl  string `json:"_pageLoadPingUrl"`
    } `json:"instrumentation"`
    Tags []struct  {
        DisplayName       string    `json:"displayName"`
        Actions                 []struct {
            Type      string `json:"_type"`
            ActionType    string    `json:"actionType"`
            Data             struct {
                Value     []struct {
                    WebSearchUrl          string `json:"webSearchUrl"`
                    Name        string  `json:"name"`
                }`json:"value"`
                ImageInsightsToken string `json:"imageInsightsToken"`
                InsightsMetadata struct {
                    PagesIncludingCount int `json:"pagesIncludingCount"`
                    AvailableSizesCount  int  `json:"availableSizesCount"`
                }
                ImageId string  `json:"imageId"`
                AccentColor  string `json:"accentColor"`
            } `json:"data"`
        } `json:"actions"`
    } `json:"tags"`
    Image struct {
        WebSearchUrl   string  `json:"webSearchUrl"`
        WebSearchUrlPingSuffix  string `json:"webSearchUrlPingSuffix"`
        Name   string   `json:"name"`
        IsFamilyFriendly   bool  `json:"isFamilyFriendly"`
        ContentSize   string   `json:"contentSize"`
        EncodingFormat    string   `json:"encodingFormat"`
        HostPageDisplayUrl   string    `json:"hostPageDisplayUrl"`
        Width     int   `json:"width"`
        Height    int    `json:"height"`
        Thumbnail    struct  {
            Width   int   `json:"width"`
            Height  int   `json:"height"`
        }
        InsightsMetadata  struct {
            PagesIncludingCount   int   `json:"pagesIncludingCount"`
            AvailableSizesCount    int    `json:"availableSizesCount"`
        }
        AccentColor   string     `json:"accentColor"`
        VisualWords   string   `json:"visualWords"`
    } `json:"image"`
}

```

## <a name="main-function-and-variables"></a>Fonction et variables principales  

Le code suivant déclare la fonction principale et attribue les variables obligatoires : 

1. Vérifiez que le point de terminaison est correct et remplacez la valeur `token` par une clé d’abonnement valide à partir de votre compte Azure. 
2. Pour `batchNumber`, attribuez un GUID, qui est nécessaire pour les limites de début et de fin des données POST. 
3. Pour `fileName`, attribuez le fichier image à utiliser pour la requête POST. 
4. Pour `endpoint`, vous pouvez utiliser le point de terminaison global ci-dessous, ou le point de terminaison de [sous-domaine personnalisé](../../../cognitive-services/cognitive-services-custom-subdomains.md) affiché dans le portail Azure pour votre ressource.

```go
func main() {
    // Verify the endpoint URI and replace the token string with a valid subscription key.se
    endpoint := "https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch"
    token := "YOUR-ACCESS-KEY"
    client := &http.Client{}
    batchNumber := "d7ecc447-912f-413e-961d-a83021f1775f"
    fileName := "ElectricBike.JPG"

    body, contentType := createRequestBody(fileName, batchNumber)
    
    req, err := http.NewRequest("POST", endpoint, body)
    req.Header.Add("Content-Type", contentType)
    req.Header.Set("Ocp-Apim-Subscription-Key", token)
    
    resp, err := client.Do(req)
    if err != nil {
        panic(err)
    }
    
       defer resp.Body.Close()
    
    resbody, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        panic(err)
    }

    //fmt.Print(string(resbody))
    
    // Create a new answer.  
    ans := new(BingAnswer)
    err = json.Unmarshal(resbody, &ans)
    if err != nil {
        fmt.Println(err)
    }
    
    fmt.Print("Output of BingAnswer: \r\n\r\n")
    
    // Iterate over search results and print the result name and URL.
    for _, result := range ans.Tags {
        spew.Dump(result)
    }
}

```

## <a name="boundaries-of-post-body"></a>Limites du corps POST

Une requête POST au point de terminaison de recherche visuelle nécessite des limites de début et de fin pour délimiter les données POST. Ces fonctions ne sont pas incluses dans le bloc `main()`.

La limite de début comprend le numéro de lot, l’identificateur de type de contenu `Content-Disposition: form-data; name="image"; filename=` et le nom de fichier de l’image à charger avec POST. 

La limite de fin comprend uniquement le numéro de lot. 


```go
func BuildFormDataStart(batNum string, fileName string) string{

    startBoundary := "--batch_" + batNum + "\r\n"
    requestBoundary := startBoundary + "\r\n" + "Content-Disposition: form-data; name=\"image\"; filename=" + fileName + "\r\n\r\n"
    return requestBoundary
}

func BuildFormDataEnd(batNum string) string{

    return "\r\n" + "\r\n" + "--batch_" + batNum + "\r\n"

}

```
## <a name="add-image-bytes-to-post-body"></a>Ajouter des octets d’image au corps POST

Le code suivant crée la requête POST qui contient les données d’image :

```go
func createRequestBody(fileName string, batchNumber string) (*bytes.Buffer, string) {
    file, err := os.Open(fileName)
    if err != nil {
        panic(err)
    }
    defer file.Close()

    body := &bytes.Buffer{}
    writer := multipart.NewWriter(body)
    
    writer.SetBoundary(BuildFormDataStart(batchNumber, fileName))
    
    part, err := writer.CreateFormFile("image", filepath.Base(file.Name()))
    if err != nil {
        panic(err)
    }
    io.Copy(part, file)
    writer.WriteField("lastpart", BuildFormDataEnd(batchNumber))    
    writer.Close()
    return body, writer.FormDataContentType()
}

```

## <a name="send-the-request"></a>Envoyer la demande

Le code suivant envoie la requête et lit les résultats :

```go
resp, err := client.Do(req)
    if err != nil {
        panic(err)
    }
    
       defer resp.Body.Close()
    
    resbody, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        panic(err)
    }

```

## <a name="handle-the-response"></a>Gérer la réponse

La fonction `Unmarshall` extrait les informations du texte JSON retourné par l’API Recherche visuelle Bing. L’imprimante `go-spew` affiche les résultats.

```go
    // Create a new answer.  
    ans := new(BingAnswer)
    err = json.Unmarshal(resbody, &ans)
    if err != nil {
        fmt.Println(err)
    }
    
    fmt.Print("Output of BingAnswer: \r\n\r\n")
    
    // Iterate over search results and print the result name and URL.
    for _, result := range ans.Tags {
        spew.Dump(result)
    }

```
> [!NOTE]
> Francesco Giordano a contribué à cet exemple de code.

## <a name="results"></a>Résultats

Les résultats identifient les images similaires à l’image contenue dans le corps POST. Les champs utiles sont `WebSearchUrl` et `Name`.

```go
    Value: ([]struct { WebSearchUrl string "json:\"webSearchUrl\""; Name string "json:\"name\"" }) (len=66 cap=94) {
     (struct { WebSearchUrl string "json:\"webSearchUrl\""; Name string "json:\"name\"" }) {
      WebSearchUrl: (string) (len=129) "https://www.bing.com/images/search?view=detailv2&FORM=OIIRPO&id=B9E6621161769D578A9E4DD9FD742128DE65225A&simid=608046863828453626",
      Name: (string) (len=87) "26\"Foldable Electric Mountain Bike Bicycle Ebike W/Lithium Battery 250W 36V MY8L | eBay"
     },
     (struct { WebSearchUrl string "json:\"webSearchUrl\""; Name string "json:\"name\"" }) {
      WebSearchUrl: (string) (len=129) "https://www.bing.com/images/search?view=detailv2&FORM=OIIRPO&id=5554757348A9E88E9EEAB74217E228689E716B61&simid=607988237543409883",
      Name: (string) (len=96) "Best 25+ Electric mountain bike ideas on Pinterest | Mountain bicycle, Mtb mountain bike and ..."
     },
     (struct { WebSearchUrl string "json:\"webSearchUrl\""; Name string "json:\"name\"" }) {
      WebSearchUrl: (string) (len=129) "https://www.bing.com/images/search?view=detailv2&FORM=OIIRPO&id=B56F626A4803D829FE326842E2B97CE5B87F17F2&simid=607991699288949513",
      Name: (string) (len=100) "Electric Fat Bike Mountain Bicycle Snow Bike Cruiser Ebike Motor Lithium Battery Dual Oil Brakes ..."
     },
     (struct { WebSearchUrl string "json:\"webSearchUrl\""; Name string "json:\"name\"" }) {
      WebSearchUrl: (string) (len=129) "https://www.bing.com/images/search?view=detailv2&FORM=OIIRPO&id=CCAE400EE510DCA6F5740ABAF08A5310B4EF0698&simid=608003854048890458",
      Name: (string) (len=81) "Best Mountain Bikes Under 1500 (Outstanding Performance) | Mountain Bicycle World"
     },
     (struct { WebSearchUrl string "json:\"webSearchUrl\""; Name string "json:\"name\"" }) {
      WebSearchUrl: (string) (len=129) "https://www.bing.com/images/search?view=detailv2&FORM=OIIRPO&id=1244730E2E3F2D5DFBED9A9CFE1DBE27B09F08BC&simid=608005181199745622",
      Name: (string) (len=98) "ANCHEER Folding Electric Mountain Bike with 26\" Super Lightweight Magnesium (36V 250W) - GearScoot"
     },
     (struct { WebSearchUrl string "json:\"webSearchUrl\""; Name string "json:\"name\"" }) {
      WebSearchUrl: (string) (len=129) "https://www.bing.com/images/search?view=detailv2&FORM=OIIRPO&id=38AAD876E57BB0D502FBDD5B1CB29EB7EB8DD9E2&simid=608041654062220328",
      Name: (string) (len=97) "29 best CB Bikes Gadgets & Accessories images on Pinterest | Bicycles, Bicycling and Bike gadgets"
     },
     (struct { WebSearchUrl string "json:\"webSearchUrl\""; Name string "json:\"name\"" }) {
      WebSearchUrl: (string) (len=129) "https://www.bing.com/images/search?view=detailv2&FORM=OIIRPO&id=6892C8FD22D0E42911C99AFD8FE1FD37BD82E02C&simid=608052803759703185",
      Name: (string) (len=98) "ANCHEER Folding Electric Mountain Bike with 26\" Super Lightweight Magnesium (36V 250W) - GearScoot"
     },

```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Qu’est ce que l’API Recherche visuelle Bing ?](../overview.md)
> [Démarrage rapide de la Recherche Web Bing dans Go](../../Bing-Web-Search/quickstarts/go.md)