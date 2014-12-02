# OVERVIEW

**Submit patient matching request:**
`HTTP POST` to remote server: `<base_remote_url>/mmapi/v1/aggregate`
For example: `https://yourmatchmaker.org/mmapi/v1/match`

## aggregate Request

`HTTP POST` request to `<base_remote_url>/mmapi/v1/match`, with an `application/json` body with the following format:

### Example

```json
{
  "id" : <identifier>,

  "submitter" : {
     "name" : "First Last",
     "email" : <email address>,
     "institution" : "Some Hospital"
  },

  "genes" : [
    {
      "gene" : <gene name>|<ensembl gene ID>|<entrez gene ID>,
      "referenceName" : "1"|"2"|…|"X"|"Y",
      "start" : <number>,
      "end" : <number>,
      "referenceBases" : "A"|"ACG"|…,
      "alternateBases" : "A"|"ACG"|…,
      "assembly" : "NCBI36"|"GRCh37.p13"|"GRCh38.p1"|…
    },
    …
  ]
}
```

#### ID
* ***Mandatory***
* The internal identifier (obfuscated or not) that can be used by the originating system to reference the patient data.
* Transparent string, limited to 255 characters in utf-8.

#### Submitter
* ***Mandatory*** if an email response is expected, *Optional* otherwise
* Consists of contact information of the person that submitted the search:
  * `email`: the email address where matches can be sent (***mandatory***); the values must conform to the [RFC 2822 address specification](http://tools.ietf.org/html/rfc2822#section-3.4) mailbox format (no group)
  * `name`: the first and last name (*optional*)
  * `institution`: human-readable institution name (*optional*)
* **The contact information is for transmitting match results only, and may not be collected and/or used for any other purposes**

#### Genes
* Is a **list of possible causes** described by:
  * `gene`:
    * `<gene symbol>` from the [HGNC database](http://www.genenames.org/) OR
    * `<ensembl gene ID>` OR
    * `<entrez gene ID>`
  * `referenceName`: `"1"`, `"2"`, …, `"22"`, `"X"`, `"Y"`; the chromosome this variant or gene is on
  * `start`: `<number>`; the start position of the variant. (0-based)
  * `end`: `<number>`; the end position of the variant. (0-based, exclusive)
      * **NOTE:** The location (`referenceName`, `start`, `end`) is *optional*
  * `referenceBases`: `"A"`|`"ACG"`|…, VCF-style reference of at least one base (*optional*)
  * `alternateBases`: `"A"`|`"ACG"`|…, VCF-style alternate allele of at least one base (*optional*)
  * `assembly`: reference assembly identifier, including patch number if relevant, of the form: `<assembly>[.<patch>]` (***mandatory***)
    * example valid values: `"NCBI36"`, `"GRCh37"`, `"GRCh37.p13"`, `"GRCh38"`, `"GRCh38.p1"`
    * If the patch is not provided, the assembly is assumed to represent the initial (unpatched) release of that assembly.
* This should list either *candidate genes*, using the `gene` field with optionally other more specific fields, or precise *genomic variants*, specifying the assembly, the location (`referenceName`, `start`, `end`), and the reference and alternate bases

## Search Results Response
An asynchronous `application/json` `HTTP POST` request to `<base_origin_url>/mmapi/v1/matchResults`, this wil result in a response holding the resulting aggregate in json format.

The response to the search request looks like:

### Example

```json
{
  "queryID" : <identifier>,
  "responseType" : "inline",
  "results" : [
    {
      "genotyes" : [
        { "1/0" : <number>,
          "1/1" : <number>,
          "0/0" : <number>,
        },…
      ]
      "genes" : […]
    },
    …
  ]
}
```

#### Query identifier
* ***Mandatory***
* Helps match the results to the original query for asynchronous results, and allows the submitter to manage the search submission
* This does not have to be the same as the id sent in the request since it represents how the remote host stores queries
* Transparent string, limited to 255 characters in utf-8.

#### Results
* ***Mandatory*** for inline results, but can be empty
* Is a **list of matches**, where each match has the same format as the one described above for the query


```json
{"responses":
  [
    {
      "queryID" : <identifier>,
      "results" : […]
    },
    {
      "queryID" : <identifier>,
      "results" : […]
    },
    …
  ]
}
```
