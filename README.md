![header](https://raw.githubusercontent.com/doga/doga/1eb1b7ea250c2b68b69ff87304a27937b14e940d/RDF-to-text-project.svg)

# RDF to text

Translates RDF into human languages. Supported file formats:

- Turtle
- TriG
- N-Triples
- N-Quads

## Available ontology translations

[See the list here](https://vocab.qworum.net). [Contribute prose annotations](https://github.com/doga/qworum-vocab-website).

## How to use

The translation produced for a dataset depends on:

1. the dictionaries used for translating ontologies to human languages, and
2. the user's language preferences.

Note that the translated dataset can also contain translations.

Here are the predicates that define a translation for an IRI or blank node:

- For ontologies: `qrmm:prose`, with `rdfs:label` as the lower-priority fallback. Blank nodes are not translated.
- In the translated datasets: `qrmm:prose`, `foaf:name`, `schema:name`, `schema:givenName`, with `rdfs:label` as the lower-priority fallback.

```javascript
import { 
  Translator, OntologyDictionary, parse, langTag, LanguagePreferences,
} from "https://cdn.jsdelivr.net/gh/doga/rdf-to-text@1.0.3/mod.mjs";

await run();

async function run(){
  const
  dataset = await parse(`
    PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
    PREFIX schema: <https://schema.org/>
    PREFIX qrmm:   <https://vocab.qworum.net/qrmm.ttl#>
    PREFIX qrmd:   <https://vocab.qworum.net/qrmd.ttl#>
    PREFIX :       <http://site.example/>
    :jonka
      a foaf:Person ;
      foaf:name "Jon Ka".
    :jimdo
      a foaf:Person ;
      foaf:name "Jim Do".
    << :jimdo foaf:weblog "https://weblog.jimdo.me"^^qrmd:anyIRI >> :acc :jonka .
    :acc qrmm:prose 'is according to'@en.
  `),
  dictionaries = [
    await buildFoafDictionary(),
    await buildRdfDictionary()
  ],
  langPrefs = new LanguagePreferences([
    langTag`en`, langTag`fr-CA`, langTag`de`, langTag`tr-TR`
  ]),

  //▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼
  translator  = new Translator(langPrefs),
  translation = translator.translate(dataset, dictionaries);
  //▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲

  console.info(translation.filter(str => str !== null).join('\n'));
}

async function buildRdfDictionary() {
  const 
  dataset = await parse(`
    PREFIX qrmm: <https://vocab.qworum.net/qrmm.ttl#>
    PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
    PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
    rdf:type qrmm:prose 'is'@en.
    rdfs:label qrmm:prose 'is'@en.
  `),
  dictionary = new OntologyDictionary();
  dictionary.addDataset(dataset);
  return dictionary;
}

async function buildFoafDictionary() {
  const 
  dataset = await parse(`
    PREFIX qrmm: <https://vocab.qworum.net/qrmm.ttl#>
    PREFIX foaf: <http://xmlns.com/foaf/0.1/>
    foaf:Person qrmm:prose 'a person'@en.
    foaf:name qrmm:prose 'is named'@en, 'est nommé'@fr, 'hat den Namen'@de.
    foaf:weblog qrmm:prose 'has the weblog'@en.
  `),
  dictionary = new OntologyDictionary();
  dictionary.addDataset(dataset);
  return dictionary;
}
```

The output:

```text
Jon Ka is a person.
Jim Do is a person.
Jim Do has the weblog https://weblog.jimdo.me; this is according to Jon Ka.
```

∎
