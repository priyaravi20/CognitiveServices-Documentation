<!--
NavPath: Linguistic Analysis API
LinkLabel: Analyze Method
Url: Linguistic-Analysis-API/documentation/AnalyzeMethod
Weight: 80
-->

# Analyze Method

The **analyze** REST API is used to analyze a given natural language input.
That might involve just finding the [sentences and tokens](Sentences-and-Tokens.md) within that input, finding the [part-of-speech tags](POS-tagging.md), or finding the [constitutency tree](Constituency-Parsing.md).
You can specify which results you want by picking the relevant analyzers.
To list all available analyzers, look at the **[analyzers](AnalyzersMethod.md)**.

Note that you need to specify the language of the input string.

**REST endpoint:**
```
https://api.projectoxford.ai/linguistic-analysis/v1.0/analyze?
```
<br>

## Request parameters

Name | Type | Required | Description
-----|-------|----------|------------
**language**    | string | Yes | The two letter ISO language code to be used for analysis. For instance, English is "en".
**analyzerIds** | list of strings | Yes | List of GUIDs of analyzers to apply. See the Analyzers documentation for more information.
**text**        | string | Yes | Raw input to be analyzed. This might be a short string such as a word or phrase, a full sentence, or a full paragraph or discourse.

<br>
## Response (JSON)
An array of analysis outputs, one for each attribute specified in the request.

The results look as follows:

Name | Type | Description
-----|------|--------------
analyzerId | string | GUID of the analyzer specified
result | object | analyzer result

Note that the type of the result depends on the input analyzer type.

### Tokens Response (JSON)

Name | Type | Description
-----|------|-------------
result | list of sentence objects | sentence boundaries identified within the text |
result[x].Offset | int | starting character offset of each sentence |
result[x].Len | int | length in characters of each sentence |
result[x].Tokens | list of token objects | token boundaries identified within the sentence |
result[x].Tokens[y].Offset | int | starting character offset of the token |
result[x].Tokens[y].Len | int | length in characters of the token |
result[x].Tokens[y].RawToken | string | the characters inside that token, before normalization |
result[x].Tokens[y].NormalizedToken | string | a normalized form of the character, safe for use in a [parse tree](Constituency-Parsing.md); for instance, an open parenthesis character '(' becomes '-LRB-' |

Example input: `This is a test. Hello.'
Example JSON response:
```json
[
  {
    "Len": 15,
    "Offset": 0,
    "Tokens": [
      {
        "Len": 4,
        "NormalizedToken": "This",
        "Offset": 0,
        "RawToken": "This"
      },
      {
        "Len": 2,
        "NormalizedToken": "is",
        "Offset": 5,
        "RawToken": "is"
      },
      {
        "Len": 1,
        "NormalizedToken": "a",
        "Offset": 8,
        "RawToken": "a"
      },
      {
        "Len": 4,
        "NormalizedToken": "test",
        "Offset": 10,
        "RawToken": "test"
      },
      {
        "Len": 1,
        "NormalizedToken": ".",
        "Offset": 14,
        "RawToken": "."
      }
    ]
  },
  {
    "Len": 6,
    "Offset": 16,
    "Tokens": [
      {
        "Len": 5,
        "NormalizedToken": "Hello",
        "Offset": 16,
        "RawToken": "Hello"
      },
      {
        "Len": 1,
        "NormalizedToken": ".",
        "Offset": 21,
        "RawToken": "."
      }
    ]
  }
]
```


### POS Tags Response (JSON)

The result is list of lists of strings.
For each sentence, there is a list of POS tags, one POS tag for each token.
To find the token corresponding to each POS tag, you'll want to ask for a tokenization object as well.

### Constituency Tree Response (JSON)

The result is a list of strings, one parse tree for each sentence found in the input.
The parse trees are represented in a parenthesized form.

<br>

## Example

`POST /analyze`

Request Body: JSON payload
```json
{
  "language": "en",
  "analyzerIds": [
    "4D4F03C1-2B1B-43E7-8E16-E5F2FF66BF93",
    "1F115AB3-A64D-41C6-9D2F-87519AAF2712" ],
  "text": "Hi, Tom! How are you today?" 
}
```

Response: JSON
```json
[
  {
    "analyzerId": "4D4F03C1-2B1B-43E7-8E16-E5F2FF66BF93", 
    "result": [ ["NNP",",","NNP","."], ["WRB","VBP","PRP","NN","."] ]
  },
  {
    "analyzerId": "1F115AB3-A64D-41C6-9D2F-87519AAF2712", 
    "result":["(TOP (S (NNP Hi) (, ,) (NNP Tom) (. !)))","(TOP (SBARQ (WHADVP (WRB How)) (SQ (VP (VBP are)) (NP (PRP you)) (NN today) (. ?))))"]
  }
]
```

