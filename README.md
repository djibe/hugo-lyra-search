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
  const [
    {create, search, insertBatch},
    {stemmer},
    index
  ] = await Promise.all([
    import('https://cdn.jsdelivr.net/npm/@lyrasearch/lyra@0.4.12/dist/index.js'),
    import('https://cdn.jsdelivr.net/npm/@lyrasearch/lyra@0.4.12/dist/stemmer/fr.min.js'),
    import('/index2.json', {assert: {type: 'json'}})
  ]);

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
  await insertBatch(searchEngine, index.default);

  const searchInput = document.getElementById('search-input');
  ['change', 'cut', 'input', 'paste', 'search'].forEach(type => searchInput.addEventListener(type, query));

  async function query(event) {
    const searchResponse = await search(searchEngine, {term: event.target.value, properties: '*'});
    document.getElementById('search-results').innerHTML = searchResponse
      .hits
      .map(i => `<a href="${i.document.url}" class="list-group-item list-group-item-action">${i.document.title}</a>`)
      .join('')
  }
</script>
```

Enjoy.

Thank you @mickaeltr !
