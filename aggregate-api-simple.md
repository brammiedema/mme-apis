# OVERVIEW

**Submit summary information of patients matching request:**
`HTTP POST` to remote server: `<base_remote_url>/mmapi/v1/aggregate`
For example: `https://yourmatchmaker.org/mmapi/v1/aggregate`

## Aggregate Request

`HTTP POST` request to `<base_remote_url>/mmapi/v1/match`, with an `application/json` body with the following format:

### Example

```json
{
  "aggregateRequest" : [
    {
      "gene" : <gene name>|<ensembl gene ID>|<entrez gene ID>,
      "chromosome" : "1"|"2"|…|"X"|"Y",
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
#### Genes
* Is a **list of possible causes** described by:
  * `aggregateRequest`:
    * `<gene symbol>` from the [HGNC database](http://www.genenames.org/) OR
    * `<ensembl gene ID>` OR
    * `<entrez gene ID>`
  * `referenceName`: `"1"`, `"2"`, …, `"22"`, `"X"`, `"Y"`; the chromosome this variant or gene is on
  * `start`: `<number>`; the start position of the variant. (0-based)
  * `end`: `<number>`; the end position of the variant. (0-based, exclusive)
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
  "aggregateResult" : [
    {
      "institution" : <hospital name>, 
      "variants" : [
      {
        "gene" : <gene name>|<ensembl gene ID>|<entrez gene ID>,
        "referenceName" : "1"|"2"|…|"X"|"Y",
        "start" : <number>,
        "end" : <number>,
        
        "alleles"[
        {
          "allele1" : "A",
          "allele2" : "T",
          "heteroygotes" : <amount observed>,
          "homozygotesReference" : <amount observed>,
          "homozygotesAlternate" : <amount observed>
        },…
        ]
       
        "assembly" : "NCBI36"|"GRCh37.p13"|"GRCh38.p1"|…
      },…
      ]
    },
    …
  ]
}
```

#### Institution
* ***Mandatory***
* Helps match the results to the original source of the results, and allows the submitter to manage the result by institute.

#### Alleles
* ***Mandatory***
* In case of hemizygote situations leaving out the allele2 field is suggested.

#### Results
* ***Mandatory*** for inline results, but can be empty
* Is a **list of matches**, where each match has the same format as the one described above for the query


```json
{"responses":
  [
    {
      "institution" : <hospital name>, 
      "results" : […]
    },
    {
      "institution" : <hospital name>, 
      "results" : […]
    },
    …
  ]
}
```
