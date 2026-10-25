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

```javascript
import { 
  Translator, OntologyDictionary, parse, langTag, LanguagePreferences,
} from "https://cdn.jsdelivr.net/gh/doga/rdf-to-text@1.0.0/mod.mjs";

await run();

async function run(){
  const
  dataset = await parse(`
    PREFIX foaf: <http://xmlns.com/foaf/0.1/>
    PREFIX qrmm: <https://vocab.qworum.net/qrmm.ttl#>
    PREFIX : <http://site.example/>

    :according_to qrmm:prose 'is according to'@en.

    :jonka
      a foaf:Person ;
      foaf:name "Jon Ka".

    :jimdo
      a foaf:Person ;
      foaf:name "Jim Do".

    << :jimdo foaf:name "Jim Do" >> :according_to :jonka .
  `),
  dictionaries = [
    await buildFoafDictionary(),
    await buildRdfDictionary()
  ],
  langPrefs = new LanguagePreferences([
    langTag`en`, langTag`fr`, langTag`de`, langTag`tr`
  ]),
  translator  = new Translator(langPrefs),
  translation = translator.translate(dataset, dictionaries);

  console.info(translation.filter(str => str !== null).join('\n'));
}

async function buildRdfDictionary() {
  const 
  dataset = await parse(`
    # https://vocab.qworum.net/prose-annotations/rdf-and-rdfs-annotations.ttl
    PREFIX qrmm: <https://vocab.qworum.net/qrmm.ttl#>
    PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
    PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

    rdf:type qrmm:prose 'is'@en .
    rdfs:label qrmm:prose 'is'@en .
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

    foaf:Person qrmm:prose 'a person'@en .
    foaf:name qrmm:prose
      'is named'@en,
      'est nommé'@fr,
      'hat den Namen'@de.
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
Jim Do is named Jim Do; this is according to Jon Ka.
```

∎
