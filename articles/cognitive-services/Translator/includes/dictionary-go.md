---
author: laujan
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: include
ms.date: 08/06/2019
ms.author: lajanuar
---

[!INCLUDE [Prerequisites](prerequisites-go.md)]

[!INCLUDE [Set up and use environment variables](setup-env-variables.md)]

## Create a project and import required modules

Create a new Go project using your favorite IDE / editor or new folder on your desktop. Then copy this code snippet into your project/folder in a file named `dictionaryLookup.go`.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "net/url"
    "os"
)
```

## Create the main function

This sample will try to read your Translator key and endpoint from these environment variables: `TRANSLATOR_TEXT_KEY` and `TRANSLATOR_TEXT_ENDPOINT`. If you're not familiar with environment variables, you can set `key` and `endpoint` as strings and comment out the conditional statements.

Copy this code into your project:

```go
func main() {
    /*
    * Read your key from an env variable.
    * Please note: You can replace this code block with
    * var key = "YOUR_KEY" if you don't
    * want to use env variables. If so, be sure to delete the "os" import.
    */
    if "" == os.Getenv("TRANSLATOR_TEXT_KEY") {
      log.Fatal("Please set/export the environment variable TRANSLATOR_TEXT_KEY.")
    }
    key := os.Getenv("TRANSLATOR_TEXT_KEY")
    if "" == os.Getenv("TRANSLATOR_TEXT_ENDPOINT") {
      log.Fatal("Please set/export the environment variable TRANSLATOR_TEXT_ENDPOINT.")
    }
    endpoint := os.Getenv("TRANSLATOR_TEXT_ENDPOINT")
    uri := endpoint + "/dictionary/lookup?api-version=3.0"
    /*
     * This calls our breakSentence function, which we'll
     * create in the next section. It takes a single argument,
     * the key.
     */
    dictionaryLookup(key, uri)
}
```

## Create a function to get alternate translations

Let's create a function to get alternate translations. This function will take a single argument, your Translator key.

```go
func dictionaryLookup(key string, uri string) {
    /*  
     * In the next few sections, we'll add code to this
     * function to make a request and handle the response.
     */
}
```

Next, let's construct the URL. The URL is built using the `Parse()` and `Query()` methods. You'll notice that parameters are added with the `Add()` method. In this sample, we're translating from English to Spanish.

Copy this code into the `altTranslations` function.

```go
// Build the request URL. See: https://golang.org/pkg/net/url/#example_URL_Parse
u, _ := url.Parse(uri)
q := u.Query()
q.Add("from", "en")
q.Add("to", "es")
u.RawQuery = q.Encode()
```

>[!NOTE]
> For more information about endpoints, routes, and request parameters, see [Translator 3.0: Dictionary Lookup](../reference/v3-0-dictionary-lookup.md).

## Create a struct for your request body

Next, create an anonymous structure for the request body and encode it as JSON with `json.Marshal()`. Add this code to the `altTranslations` function.

```go
// Create an anonymous struct for your request body and encode it to JSON
body := []struct {
    Text string
}{
    {Text: "Pineapples"},
}
b, _ := json.Marshal(body)
```

## Build the request

Now that you've encoded the request body as JSON, you can build your POST request, and call the Translator.

```go
// Build the HTTP POST request
req, err := http.NewRequest("POST", u.String(), bytes.NewBuffer(b))
if err != nil {
    log.Fatal(err)
}
// Add required headers to the request
req.Header.Add("Ocp-Apim-Subscription-Key", key)
req.Header.Add("Content-Type", "application/json")

// Call the Translator
res, err := http.DefaultClient.Do(req)
if err != nil {
    log.Fatal(err)
}
```

If you are using a Cognitive Services multi-service subscription, you must also include the `Ocp-Apim-Subscription-Region` in your request parameters. [Learn more about authenticating with the multi-service subscription](../reference/v3-0-reference.md#authentication).

## Handle and print the response

Add this code to the `altTranslations` function to decode the JSON response, and then format and print the result.

```go
// Decode the JSON response
var result interface{}
if err := json.NewDecoder(res.Body).Decode(&result); err != nil {
    log.Fatal(err)
}
// Format and print the response to terminal
prettyJSON, _ := json.MarshalIndent(result, "", "  ")
fmt.Printf("%s\n", prettyJSON)
```

## Put it all together

That's it, you've put together a simple program that will call the Translator and return a JSON response. Now it's time to run your program:

```console
go run dictionaryLookup.go
```

If you'd like to compare your code against ours, the complete sample is available on [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Go).

## Sample response

```json
[
  {
    "displaySource": "pineapples",
    "normalizedSource": "pineapples",
    "translations": [
      {
        "backTranslations": [
          {
            "displayText": "pineapples",
            "frequencyCount": 158,
            "normalizedText": "pineapples",
            "numExamples": 5
          },
          {
            "displayText": "cones",
            "frequencyCount": 13,
            "normalizedText": "cones",
            "numExamples": 5
          },
          {
            "displayText": "piña",
            "frequencyCount": 5,
            "normalizedText": "piña",
            "numExamples": 3
          },
          {
            "displayText": "ganks",
            "frequencyCount": 3,
            "normalizedText": "ganks",
            "numExamples": 2
          }
        ],
        "confidence": 0.7016,
        "displayTarget": "piñas",
        "normalizedTarget": "piñas",
        "posTag": "NOUN",
        "prefixWord": ""
      },
      {
        "backTranslations": [
          {
            "displayText": "pineapples",
            "frequencyCount": 16,
            "normalizedText": "pineapples",
            "numExamples": 2
          }
        ],
        "confidence": 0.2984,
        "displayTarget": "ananás",
        "normalizedTarget": "ananás",
        "posTag": "NOUN",
        "prefixWord": ""
      }
    ]
  }
]
```

## Next steps

Take a look at the API reference to understand everything you can do with the Translator.

> [!div class="nextstepaction"]
> [API reference](../reference/v3-0-reference.md)