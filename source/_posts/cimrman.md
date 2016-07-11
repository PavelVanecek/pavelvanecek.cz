---
title: Jak jsem vyhrál bustu Járy Cimrmana
date: 2014-03-07 14:35:27
tags:
---

V březnu 2014 mi přišla od [@seznam_cz](https://twitter.com/seznam_cz) busta Járy Cirmana. Ta se strhanými rysy.

![Cimrmanova busta se strhanými rysy](/images/cimrman.jpg)

Podařilo se mi totiž do jednoho tweetu nacpat nejvíc Cimrmanových vynálezů:

<blockquote class="twitter-tweet" data-lang="en-gb"><p lang="en" dir="ltr"><a href="https://twitter.com/seznam_cz">@seznam_cz</a> curl <a href="http://t.co/JxmgxL98j3">http://t.co/JxmgxL98j3</a> |iconv -f iso8859-1 |xpath &quot;//div[<a href="https://twitter.com/id">@id</a>=&#39;bodyContent&#39;]//ul//text()&quot; 2&gt;/dev/null <a href="http://t.co/cYyLUJ61uB">pic.twitter.com/cYyLUJ61uB</a></p>&mdash; Pavel Vanecek (@Corkscreewe) <a href="https://twitter.com/Corkscreewe/status/438643052991623168">26 February 2014</a></blockquote>

Jak jsem k tomu dospěl?

## Nápad

Pár odpovědí se už sešlo, ale řekl jsem si, že nebude těžké je trumfnout. Můj první nápad byl vyparsovat z wikipedie všechny vynálezy, seřadit od nejkratšího a nakopírovat do tweetu.

Hned druhý ale byl: když už mám vynálezy naparsované, proč nezkusit do tweetu nacpat celý skript?

## Co skript musí splňovat

Takové malé MVP.

- Vypsat vynálezy
- Vejít se do 106 znaků (11 zabere handle na @seznam_cz, 23 odkaz na wiki)

## Nice to have

Stanovil jsem si pár dalších cílů:

- Nezávislost na externích knihovnách
- Multiplatformnost
- Plná funkčnost jen přes copy + paste

## Postupná iterace k výsledku

První pokus: Nativní DOM API
_Všechny skripty jsou odsazené pro lepší čitelnost. Sčítám znaky kromě mezer a newline, kde nejsou potřeba._

```javascript
Array.prototype.forEach.call(
  document.querySelectorAll("#mw-content-text ul:first-of-type li"),
  function(element) { console.log(element.textContent)}
)
```

Skvělé, dělá to přesně to, co potřebuju. Jenže je to 150 znaků dlouhé. Je tu ale prostor pro zkrácení:

```javascript
[].forEach.call(
  $$("#bodyContent li"),
  function(e){console.log(e.innerText)}
)
```

76 znaků. Paráda. Skript sice kvůli kratšímu selektoru vypisuje i navigaci, ale tu jsem ochotný obětovat. Zbyde mi ještě místo na nějaké to moudro.

Jenže: na uživatele kladu příliš velké nároky. Musí sám kliknout na odkaz, pak si vzpomenout, že zapomněl skript zkopírovat, překliknout se zpátky, otevřít devtools nebo firebug, … je toho moc. Na podrobný návod není prostor.

Dá se vyrobit odkaz, který rovnou spustí JavaScript? Nedá. A je to tak dobře.

## curl

Multiplatformnost tady vzdávám, přesouvám se do bashe. Předpokládám, že publikum na Twitteru bude rozumět.

Budu potřebovat `|` pipe. wget má přepínač -O pro směřování na stdout. curl mi tyto dva znaky ušetří.

Čím parsovat HTML? [Regulárním výrazem asi ne](http://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags/1732454), nevešel by se do tweetu.

Externí knihovnu se mi využívat nechtělo. Ale když už jsem si odříznul Windows, můžu pokračovat: OSX má předinstalovaný xpath příkaz.

```bash
curl -L http://cs.wikipedia.org/wiki/Cimrmanovy_vynálezy |\
xpath "//div[@id='bodyContent']//li//text()"
```

53 znaků, spousta chyb, špatně formátovaná čeština. Wide character in print značí potíže s UTF8. Hrabat do [perlového kódu xpath](http://ahinea.com/en/tech/perl-unicode-struggle.html) v jednom tweetu nezvládnu, takže iconv:

```bash
curl -L http://cs.wikipedia.org/wiki/Cimrmanovy_vynálezy |\
iconv -f iso8859-1|\
xpath "//div[@id='bodyContent']//ul//text()"
```

72 znaků. Měl bych použít i `-t utf-8` pro specifikaci cílového kódování, ale na mém stroji defaultuje správně.

Ještě se zbavím značek pro začátek a konec tagu, které perl moudře vypisuje na stderr:

```bash
curl -L http://cs.wikipedia.org/wiki/Cimrmanovy_vynálezy |\
iconv -f iso8859-1|\
xpath "//div[@id='bodyContent']//ul//text()" 2>/dev/null
```

84 znaků. Kdybych použil delší selektor, nevypsala by se navigace, ale takto mám ještě rezervu na screenshot s důkazem.

## Výsledek

Očekával jsem drobné pobavení, možná pár favourite, ale výhru určitě ne :)

<blockquote class="twitter-tweet" data-lang="en-gb"><p lang="cs" dir="ltr"><a href="https://twitter.com/Corkscreewe">@Corkscreewe</a> Za originální zpracování odpovědi a nejvíce vynálezů přímo v tweetu vyhráváte. Více info v DM.</p>&mdash; Seznam.cz (@seznam_cz) <a href="https://twitter.com/seznam_cz/status/438720083704418304">26 February 2014</a></blockquote>

Skript funguje jenom v OSX a copy + paste jsem taky nesplnil: Twitter za odkaz přidává trojtečku.

## Upraveno 2016

Wikipedia oproti roku 2014 přesměrovává na https a navíc zvládá diakritiku v URL a nespoléhá se jen na prohlížeč. To jsou dva redirecty a několik znaků navíc. Aby vše fungovalo, přidal jsem k volání curl parametr `-L`.

<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
