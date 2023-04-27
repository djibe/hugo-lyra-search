# hugo-lyra-search

A [Lyra search](https://docs.lyrasearch.io/) implementation for [Hugo](https://gohugo.io/).

**Warning.** This is Work In Progress

## 1. Create the database layout

We use Hugo's fast file generation to generate our index in JSON format.

In the `layouts` folder, create a `index.json` file.

```html
[ {{- $i := 0 -}}
  {{- range where .Site.RegularPages "Section" "ne" "" -}}
     {{- if not .Params.noSearch -}}
        {{- if gt $i 0 }},{{ end -}}
        {"title":{{ .Title | jsonify }},"body": {{ .Plain | htmlUnescape | chomp | jsonify }},"uri":"{{ .RelPermalink }}","meta":{"date": "{{ .Date.Format "2006-01-02" }}","synonyms": {{ .Params.Synonyms | jsonify }},"specialites": {{ .Params.specialites | jsonify }},"annees": "{{ .Params.annees }}","sources": {{ .Params.sources | jsonify }},"tags": [{{- $t := 0 }}{{- range .Param "tags" -}}{{ if gt $t 0 }},{{ end }}{{ . | jsonify }}{{ $t = add $t 1 }}{{ end -}}]}}
        {{- $i = add $i 1 -}}
     {{- end -}}
  {{- end -}} ]
```

Now tell Hugo to generate the `index.json` from this template.

In you `config.toml`, add a `[outputs]` section (or edit it like this) to generate `index.html` (homepage), `index.xml` (RSS feed) and our `index.json` (search index).

```toml
[outputs]
  home = ["HTML", "RSS", "JSON"]
```

## 2. Import Lyra in your footer

Import *Lyra* before your `body` closing tag.

```html
<script type="module" async crossorigin>
  import { create, search, insert, insertBatch } from "https://cdn.jsdelivr.net/npm/@lyrasearch/lyra@0.4.12/dist/index.js";
  import { stemmer } from "https://cdn.jsdelivr.net/npm/@lyrasearch/lyra@0.4.12/dist/stemmer/fr.min.js"; // For internationalization only

  async function fetchJson(url) {
    const response = await fetch(url);

    if (response.status == 200) {
      const json = await response.json();
      return json;
    }

    throw new Error(response.status);
  }

  const db = await create({
    schema: {
      title: "string",
      body: "string",
      uri: "string",
      meta: {
        date: "string",
        tags: "string"
      }
    },
    defaultLanguage: "french",
    components: {
      tokenizer: {
        stemmingFn: stemmer,
      }
    }
  });

  const searchIndex = await fetchJson('/index.json').catch(alert);;
  const newData = await insertBatch(db, searchIndex);
  const searchResult = await search(db, {
    term: "2022",
    properties: "*"
  });
  // Print Lyra result object
  let searchList = "";
  if (searchResult.count > 0){
    [...searchResult.hits].forEach(item => {
      searchList += `Title: ${item.document.title}\nAuthor: ${item.document.uri}`
    });
  } else { console.log('no results') }
</script>
```
