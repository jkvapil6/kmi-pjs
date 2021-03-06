# Úvod do Platformy JS

V úvodní lekci kurzu si zopakujeme základy jazyka JS, jeho syntax a sémantiku, rozdíly mezi verzemi, práci se složenými daty (objekty, pole), asynchronními funkcemi a zaměříme se na specifické vlastnosti tohoto jazkya.

Pro spouštění ukázek kódu doporučuji použít [REPL.it](https://repl.it/) (případně kód spouštět pomocí node.js - více v další lekci)

## 1. Deklarace a definice Vanilla JS vs ES6+ 

Rozdíly v rozsahu platnosti a deklaracích / definicích proměnných a funkcí.

### Vanilla JS


```javascript
var x = 42

fn(x) // funkci lze volat před samotnou deklarací

function fn(x) {
  return x + 1
}

var x = 666 // var lze redeklarovat
 
///
/// co se stane? Ve scope se přepíše vazba fn 
///
function fn(x) {
  return x - 1
}
```


Ve většině případů chceme zamezit (nechtěnné) redefinici a redeklaraci - proto preferujeme pro vytvoření *vazeb** použití const (neboli navázání hodnoty na konstantu). V případech, kdy záměrně chceme někdy v budoucnu vazby změnit, použijeme let. Var používat nebudeme (více např. [zde](https://blog.usejournal.com/awesome-javascript-no-more-var-working-title-999428999994)).

(* Ve funkcionálním programování se pro definice proměnných používá výraz "vytvořit vazby mezi symboly (proměnnými) a hodnotami", což odpovídá přesněji přístupu JS.)


### ECMAScript 6

```javascript
let x = 42
let x = 666   // error -> let nelze redeklarovat 
x = 666       // ok

fn(x)         // error -> funkce není deklarovaná

///
/// deklarace funkce jako navázání lambda výrazu na symbol fn
///
const fn = (x) => {
  return x + 1
}

///
/// error -> nelze redeklarovat
///
const fn = (x) => {
  return x + 1
}

fn = x  // error -> const nelze redefinovat
```


## 2. Funkce 


S funkcemi lze v JS manipulovat stejně jako s hodnotami (High-order functions).


```javascript
///
/// lambda v jednom řádku vrací poslední vyhodnocený výraz 
///
const fn = (x) => 42 && 666 && x + 1   

fn(42) // -> 43

///
/// funkce jako argumenty funkcí
///
const fn2 = (x, fn) => {
  return fn(x)
}

fn2(42, fn) // -> 43

///
/// funkce jako návratové hodnoty
///
const fn3 = () => {
  return (x) => x + 1 
}

fn3() // -> function

fn3()(42) // -> 43
```

## 3. Objekty a pole

Složená data mohou být v JS realizována jako *referenční datové typy* - **pole** nebo **objekty** (JS objekt != JSON, uvidíme později). V čem je však práce s nimi občas zrádná a neintuitivní je jejich duplikace. Na následujících příkladech si ukážeme, jak k této problematice přistupovat. Uvažujme následující příklad:


```javascript
const homer = {
  name: "Homer",
  age: 39,
  children: [{name: "Bart"}, {name: "Lisa"}],
  greeting: () => "Hello, my name is Homer"
}

const copy = homer
homer.age = 42

copy.age == homer.age // -> true
```


Proč se změnil věk oboum objektům? Zkopírovala se pouze reference na daný objekt, tudíž přiřazujeme novou hodnotu tomu samému objektu. 

Zkusme nyní využít tzv. **destrukturalizaci** (spread operátor ... ), která převede objekt na posloupnost párů <key, value> a vytvoří je jako nové hodnoty.


```javascript
const shallowCopy = {...homer}
shallowCopy.age = 39

homer.age == shallowCopy.age // -> false
```


Nyní se nám změna věku už projevila správně. Co když změníme jméno dítěte původního objeku? 


```javascript
shallowCopy.children[0].name = "Meggie"

shallowCopy.children[0].name == homer.children[0].name // -> true
```


Změna se provede i ve zkopírovaném objektu. Pomůže Object.assign? 


```javascript
const shallowCopy2 = Object.assign({}, homer);
shallowCopy2.children[0].name = "Bart"

shallowCopy2.children[0].name == homer.children[0].name // -> true
```


Jak vidíme, ani toto nám pro zanořená data nepomohlo. Jak tedy provést plnohodnotnou duplikaci objektu?


```javascript
const deepCopy = JSON.parse(JSON.stringify(homer)) 
deepCopy.children[0].name = "Meggie"

deepCopy.children[0].name == homer.children[0].name // -> false
```


S využitím základních tříd a funkcí JS můžeme objekt převést na řetězec znaků a z něj na JSON, čímž docílíme zkopírování všech zanořených datových struktur. Nyní si můžeme všimnout, že díky funkci JSON.parse jsme získali sice nový objekt, ale bez funkce greeting. To samé platí i pro víceřádkové stringy, které se nahradí znaky konce řádků, jelikož formát JSON víceřádkové řetězce nepodporuje.


```javascript
const str = JSON.parse(JSON.stringify({str: `Multi

  line`
}))

console.log(str) // -> { str: 'Multi\n\n  line' }
```


Spolehlivé duplikace zanořených datových struktur docílíme pouze vytvořením vlastních pomocných funkcí (nebo použítím existujících knihoven. Více v dalších lekcích o Node.js a Imutabilních datových strukturách)



## 4. Array functions

Pro práci s poli můžeme využít (high-order) funkce a provádět tak s daty různé transformace.


```javascript
const simpsons = [
  {name: "Homer", age: 39}, 
  {name: "Marge", age: 36},
  {name: "Bart", age: 10},
  {name: "Lisa", age: 8}
]

// vyfiltrujeme pouze mladistvé členy rodiny Simpsonových
simpsons.filter(s => s.age <= 21)

// vybereme pouze jména
simpsons.map(s => s.name)

// sečteme věk všech simpsonů
simpsons.map(s => s.age)
  .reduce((acum, age) => acum + age)

// nedestruktivní operace
const parents = simpsons.slice(0, 2) // získáme členy rodiny od indexu 0 do indexu 2 (vyjma)

// destruktivní operace (mění původní strukturu - přidá prvek na konec pole)
simpsons.push({name: "Megie", age: 1})

// destruktivní operace (odebere první prvek)
const homer = simpsons.shift()
```


## 5. Asynchronní JS


Později zjistíme (zejména při práci s frontendem), že velmi často potřebujeme reagovat na různé typy asynchronních událostí - typicky ve chvíli, kdy dostaneme odpoveď od serveru s nějakými daty. K tomu se dříve používaly tzv. **callbacky**, neboli funkce, které umožňují dál pracovat např. s načtenými daty někdy v budoucnu. 

Nejjednoduším příkladem asynchronního kódu je následující funkce setTimeout, která bere jako argument callback, jenž se má vykonat po uběhnutí 2000ms.  


```javascript
const callback = () => console.log("Callback triggered!")
const ms = 2000
setTimeout(callback, ms)
console.log("Hello")
```


Po spuštění vidíme, že se asynchronní funkce jakoby přeskočí, vypíše se "Hello" a po 2 sekundách se vypíše "Callback triggered!". Pro hlubší pochopení si můžete přečíst něco o [JS Event Loop](https://javascript.info/event-loop). Nám bude v tuto chvíli stačit, že vyhodnocování není lineární (přestože je JS kód vykonáván v jednom vlákně) a že existuje nějaká fronta asynchronních událostí.


### Problém s callbacky 

Představme si, že je třeba takto vykonat 5 zřetězených asynchronních funkcí.


```javascript
const goToPub = (callback) => {
  console.log("Going to pub..")
  setTimeout(callback, 500)
}

const orderBeer = (callback) => {
  console.log("Ordering beer..")
  setTimeout(callback, 500)
}

const drinkBeer = (callback) => {
  console.log("Drinking beer..")
  setTimeout(callback, 500)
}

const payBeer = (callback) => {
  console.log("Paying beer..")
  setTimeout(callback, 500)
}

const goHome = () => {
  setTimeout(() => console.log("Going home.."), 500)
} 

// "pyramid of doom":

goToPub(
  () => orderBeer(
    () => drinkBeer(
      () => payBeer(
        () => goHome()
      )
    )
  )
)
```


Tímto způsobem předávání callbacků se stane kód velmi rychle nepřehledný (Callback hell).


### Promises


Proto se do JS zavedla třída [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) (neboli "příslib" dokončení výpočtu někdy v budoucnu), která má 3 stavy:

* pending: initial state, neither fulfilled nor rejected.
* fulfilled: meaning that the operation was completed successfully.
* rejected: meaning that the operation failed.

a umožňuje tak vracet nedokončený výpočet jako objekt.


### Async / await

Klíčová slova async / await pak vytváří nádstavbu nad přísliby, což umožňuje elegantně pracovat s asynchronním kódem.


```javascript
///
/// Pomocná funkce simulující asynchronnost
/// 
const delay = (ms) => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(true)
    }, ms)
  })
}

const goToPub = async () => {
  await delay(500)
  console.log("Going to pub..")
}

const orderBeer = async () => {
  await delay(500)
  console.log("Ordering beer..")
}

const drinkBeer = async () => {
  await delay(500)
  console.log("Drinking beer..")
}

const payBeer = async () => {
  await delay(500)
  console.log("Paying beer..")
}

const goHome = async () => {
  await delay(500)
  console.log("Going home..")
} 

///
/// Testovací funkce
///
const enjoyFriday = async () => {
  await goToPub()
  await orderBeer()
  await drinkBeer()
  await payBeer()
  await goHome()
}

enjoyFriday()
```

Všimněme si definice a volání asynchronní funkce `enjoyFriday` - klíčové slovo `await` lze použít pouze uvnitř asynchronní funkce.

## Úkol

Implementujte (a otestujte) sadu pomocných funkcí, které později použijete v projektu:

* `deepCopy(obj)` - provede správnou duplikaci libovolně zanořených datových struktur
* `getNowDate()` - vrátí aktuální čas (string) ve formátu DD.MM.YYYY HH:MM:SS (s pomocí třídy Date)

## Reference

Podrobnější výčet vlastností jazyka + příklady [zde](https://github.com/airbnb/javascript/blob/master/README.md).