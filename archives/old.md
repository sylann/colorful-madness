
# Honorable mentions (previously used scripts)

This produced a css rule containing all colors with their name as custom properties:

```js
(function () {
  const colorHex = e => e.previousElementSibling.title.slice(-7).toLowerCase()
  const colorName = e => e.textContent
  const cssVariable = s => s.toLowerCase()
    .replace(/[^\w]/g, '-') // replace non letter caracters wiht dashes
    .replace(/(-+(?=-)|-+$)/g, '') // remove trailing and extraneous consecutive dashes
  const hexBase = h => `#${h[1]}${h[3]}${h[5]}`
  const elements = Array.from(document.querySelector('.mw-parser-output').querySelectorAll('p + p'))
  copy(
    ':root {\n'
    + elements
      .map(e => ({ name: colorName(e), hex: colorHex(e) }))
      .sort((a, b) => (a.hex < b.hex ? -1 : a.hex > b.hex ? 1 : 0)) // sort by hex value
      .map(c => `  --${cssVariable(c.name)}: ${c.hex}; /* ${hexBase(c.hex)} */`)
      .join('\n')
    + '\n}\n'
  )
})()
```

Which will produce the following output (extract):

```css
:root {
  --black: #000000;
  --registration-black: #000000;
  --navy-blue: #000080;
  --dark-blue: #00008b;
  --duke-blue: #00009c;
  --medium-blue: #0000cd;
  --blue: #0000ff;
  --phthalo-blue: #000f89;
  --zaffre: #0014a8;
  --blue-pantone: #0018a8;
  /* ... */
}
```

Full result at [colors](./colors.css)

---

This version gave an array sorted by hue.

```js
(function () {
  const elements = Array.from(document.querySelector('.mw-parser-output').querySelectorAll('p + p'))
  const toColorObject = element => {
    const _title = element.previousElementSibling.title
    const [h, s, l] = _title.slice(8).split(')')[0].replace('-', '0').replace('°', '').split(' ')
    const hex = _title.slice(-7).toLowerCase()
    const name = element.textContent
    const sortKey = ('000' + h).slice(-3) + s + l // not necessary anymore
    return { sortKey, hex, name, h: Number(h), s: Number(s.slice(0, -1)), l: Number(l.slice(0, -1)) }
  }
  const compare = k => (a, b) => a[k] < b[k] ? -1 : a[k] > b[k] ? 1 : 0
  copy('[' + elements.map(toColorObject).sort(compare('sortKey')).map(JSON.stringify).join(',') + ']')
})()
```

The first version where the result could directly be pasted in an existing page waiting for the objects.

```js
(function () {
  const colorHex = e => e.previousElementSibling.title.slice(-7).toLowerCase()
  const colorHsl = e => e.previousElementSibling.title.slice(8).split(')')[0].replace('-', '0').replace('°', '').replace(' ', ', ')
  const sortableHsl = e => {
    // const [h, s, l] = /^𝗛𝗦𝗩 \(((\d+|-)°? (\d+)% (\d+)%)\).*/.exec(e.previousElementSibling.title).slice(2)
    const str = e.previousElementSibling.title.slice(8).split(')')[0].replace('-', '0').replace('°', '')
    const [h, s, l] = str.split(' ')
    return `/* ${('000' + h).slice(-3)} ${s} ${l} */`
  }
  const colorName = e => e.textContent
  const elements = Array.from(document.querySelector('.mw-parser-output').querySelectorAll('p + p'))
  copy(elements.map(e => `        ${sortableHsl(e)}{ hex: '${colorHex(e)}', name: "${colorName(e)}" },`).sort().join('\n'))
})()
```

Other tested scripts:

```js
var colorHex = e => e.previousElementSibling.title.slice(-7).toLowerCase()
var colorName = e => e.textContent
var elements = Array.from(document.querySelector('.mw-parser-output').querySelectorAll('p + p'))
// copy a js object - 1, trick using json parse
copy(JSON.parse('{' + elements.map(e => `"${colorHex(e)}": "${colorName(e)}"`).sort().join() + '}'))
// copy a js object - 2, evil reduce
copy(elements.reduce((obj, e) => { obj[colorHex(e)] = colorName(e); return obj }, {}))
// copy a text list
copy(elements.map(e => `${colorHex(e)}: ${colorName(e)}`).sort().join('\n'))
```
