# Hugo Lyra search

A [Lyra search](https://docs.lyrasearch.io/) implementation for [Hugo](https://gohugo.io/).

## 1. Create the database layout

We use Hugo's fast file generation to generate our index in JSON format.

In the `layouts` folder, create an `index.json` file.

```html
[ {{- $i := 0 -}}
  {{- range where .Site.RegularPages "Section" "ne" "" -}}
     {{- if not .Params.noSearch -}}
        {{- if gt $i 0 }},{{ end -}}
        {"title":{{ .Title | jsonify }},"body": {{ .Plain | htmlUnescape | chomp | jsonify }},"url":"{{ .RelPermalink }}","meta":{"date": "{{ .Date.Format "2006-01-02" }}","synonyms": {{ .Params.Synonyms | jsonify }},"specialites": {{ .Params.specialites | jsonify }},"annees": "{{ .Params.annees }}","sources": {{ .Params.sources | jsonify }},"tags": [{{- $t := 0 }}{{- range .Param "tags" -}}{{ if gt $t 0 }},{{ end }}{{ . | jsonify }}{{ $t = add $t 1 }}{{ end -}}]}}
        {{- $i = add $i 1 -}}
     {{- end -}}
  {{- end -}} ]
```

Now tell Hugo to generate the `index.json` from this template.

In you `config.toml`, add a `[outputs]` section (or edit it like this) to generate `index.html` (homepage) and our `index.json` (search index).

```toml
[outputs]
  home = ["HTML", "JSON"]
```

## 2. Import Lyra in your footer

Import *Lyra* before your `body` closing tag.

```html
<script type="module" async crossorigin>
  import {create, search, insertBatch} from 'https://cdn.jsdelivr.net/npm/@lyrasearch/lyra@0.4.12/dist/index.js';
  import {stemmer} from 'https://cdn.jsdelivr.net/npm/@lyrasearch/lyra@0.4.12/dist/stemmer/fr.min.js';
</script>
```

Now add this code within the module:

```js
  const indexResponse = await fetch('/index.json')
  const index = await indexResponse.json();

  const searchEngine = await create({
    schema: {
      title: 'string',
      content: 'string',
      url: 'string'
    },
    defaultLanguage: 'french',
    components: {
      tokenizer: {
        stemmingFn: stemmer,
      }
    }
  });
  await insertBatch(searchEngine, index);

  const searchResults = document.getElementById('search-results');

  const searchInput = document.getElementById('search-input');
  searchInput.addEventListener('keydown', query);

  async function query() {
    searchResults.innerHTML = (await search(searchEngine, {term: searchInput.value, properties: '*'}))
      .hits
      .map(i => `<a href="${i.document.url}" class="list-group-item list-group-item-action">${i.document.title}</a>`)
      .join('')
  }
```

Enjoy.

Thank you @mickaeltr !
