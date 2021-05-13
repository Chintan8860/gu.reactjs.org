---
title: અમાન્ય Hook Call ની ચેતવણી
layout: single
permalink: warnings/invalid-hook-call-warning.html
---

 તમે કદાચ અહીં છો કારણ કે તમને નીચેનો ભૂલ સંદેશ મળ્યો છે:

 > Hooks ફક્ત functional component ની અંદર જ બોલાવી શકાય. 

અહીં આ ત્રણ સામાન્ય કારણો છે જેને તમે જોઈ રહ્યાં છો:

1. તમારી પાસે React અને React DOM ના **અલગ અલગ વર્ઝન** હોઈ શકે છે.
2. તમે કદાચ  **[Hooks ના નિયમો] તોડો છો (/docs/hooks-rules.html)**.
3. તમારા જોડે કદાચ  **એકથી વધારે React ની કોપી હોઈ શકે છે** એક જ app ની અંદર.

ચાલો આ દરેક કેસને જોઈએ. 

## React and React DOM ના અલગ અલગ વર્ઝન {#mismatching-versions-of-react-and-react-dom}

તમે કદાચ `react-dom` નું વર્ઝન (< 16.8.0) અથવા `react-native` (< 0.59) વાપરતા હસો જે Hooks સપોર્ટ નહીં કરતાં હોય. તમે તમારી એપ્લિકેશન ફોલ્ડરમાં કમાંડ `npm ls react-dom` અથવા `npm ls react-native` ચલાવીને જોઈ શકો છો કે તમે કયું વર્ઝન વાપરો છો. જો તમને એક થી વધારે મળે તો એ પણ પ્રોબ્લેમ કરી શકે છે(વધુ માહિતી નીચે).

## Hooks ના નિયમ તોડવા {#breaking-the-rules-of-hooks}

તમે હૂકસ નો વપરાશ ત્યારેજ કરી શકો **જયારે React function component રેન્ડર કરતું હોય**:

* ✅ Functional componenet ની બોડી માં માત્ર સૌથી ઉપર જ hooks ને call કરવો. 
* ✅ [custom Hook](/docs/hooks-custom.html)ની બોડી માં સૌથી ઉપર જ hooks ને call કરવો.

**આના વિષેની વધુ માહિતી [Hooks ના નિયમો](/docs/hooks-rules.html).**

```js{2-3,8-9}
function Counter() {
  // ✅ Good: top-level in a function component
  const [count, setCount] = useState(0);
  // ...
}

function useWindowWidth() {
  // ✅ Good: top-level in a custom Hook
  const [width, setWidth] = useState(window.innerWidth);
  // ...
}
```

મૂંઝવણ ને ટાળવા, Hooks ને બીજા કેસમાં call કરવા supported **નથી**:

* 🔴 Class components માં Hooks ને call કરવા નહીં.
* 🔴 Event handlers માં Hooks ને call કરવા નહીં.
* 🔴 Hooks ને functions ની અંદર call ના કરો જે `useMemo`, `useReducer`, or `useEffect` ની અંદર વપરાયા હોય.

જો તમે આ નિયમો તોડસો તો તમને આ error જોવા મળસે. 

```js{3-4,11-12,20-21}
function Bad1() {
  function handleClick() {
    // 🔴 Bad: inside an event handler (to fix, move it outside!)
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad2() {
  const style = useMemo(() => {
    // 🔴 Bad: inside useMemo (to fix, move it outside!)
    const theme = useContext(ThemeContext);
    return createStyle(theme);
  });
  // ...
}

class Bad3 extends React.Component {
  render() {
    // 🔴 Bad: inside a class component
    useEffect(() => {})
    // ...
  }
}
```

તમે [`eslint-plugin-react-hooks` plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks) વાપરી શકો છે જે આ ભૂલો શોધવામાં મદદ કરશે. 

>Note
>
>[Custom Hooks](/docs/hooks-custom.html) કદાચ બીજા Hooks ને call કરી શકે છે. (તેમનો મુખ્ય હેતુ જ એ છે). આ કામ કરે છે કારણ કે custom Hooks ત્યારે જ call થાય કે જયારે functional component રેન્ડર થતો હોય. 


## નકલી React {#duplicate-react}

Hooks ને કામ કરવા માટે, તમારી application ના code નું `react` import એ `react-dom` પેકેજના એજ module ના `react` import ને રીસોલ્વ કરતું હોવું જોઈએ.  

જો આ `react` imports બે અલગ objects ને રીસોલ્વ કરતાં હોય તો, તમે આ warning જોશો. આ કદાચ ત્યારે થઈ શકે જો તમારા પાસે **ભૂલ થી બે કોપી** `react` પેકેજની આવી જાય.

જો તમે package management માટે node વાપરતા હોય, તો તમે તમારા પ્રોજેક્ટ ફોલ્ડર માં આ ચાલુ કરીને જોઈ શકો છો:

    npm ls react

જો તમે એક થી વધારે React જોઈ શકો તો તમારે શા માટે આ બન્યું તે શોધવાની જરૂર પડશે અને dependency tree સુધારવું પડશે. ઉદાહરણ તરીકે, કદાચ તમે વાપરતા હસો એ library `react` ને dependency તરીકે લેતું હસે (peer dependency ની જગ્યાએ). જ્યાં સુધી એ library ફિક્સ ના થાય, [Yarn resolutions](https://yarnpkg.com/lang/en/docs/selective-version-resolutions/) એક શક્ય કાર્ય છે.

તમે આ સમસ્યા ને debug કરી શકો છો, કેટલાક logs ઉમેરીને અને ફરીથી development server ચાલુ કરીને:

```js
// Add this in node_modules/react-dom/index.js
window.React1 = require('react');

// Add this in your component file
require('react-dom');
window.React2 = require('react');
console.log(window.React1 === window.React2);
```

જો એ `false` પ્રિન્ટ કરે તો તેનો અર્થ તમારા પાસે બે Reacts છે અને તમારે શા માટે આવું થયું તે શોધવાની જરૂર છે. [આ issue](https://github.com/facebook/react/issues/13991)માં લોકો દ્વારા મળેલા કેટલાક સામાન્ય કારણો શામેલ છે.

આ સમસ્યા ત્યારે પણ આવી શકે છે જ્યારે તમે `npm link` કે તેના જેવુ વાપરો. એ સ્થિતિ માં, તમારું bundler કદાચ બે Reacts "જોઈ" શકે — એક એપ્લિકેશન ફોલ્ડરમાં અને એક one તમારી library ના ફોલ્ડરમાં. એ ધારીને કે `myapp` અને `mylib` એ બાજુ-બાજુ ના ફોલ્ડર્સ છે, એક શક્ય ઉપાય `npm link ../myapp/node_modules/react` ને `mylib` માંથી ચાલુ કરવી. આ library ne એપ્લિકેશનની React કોપી વાપરવાનું કેહશે. 

>Note
>
>સામાન્ય રીતે, React એકથી વધારે સ્વતંત્ર કોપી એક પેજ પર વાપરવાનું સપોર્ટ કરે છે (ઉદાહરણ તરીકે, જો એક app અને third-party widget બંને વાપરી શકે છે). એ ત્યારેજ ટૂટે છે જ્યારે `require('react')` એ component અને `react-dom`, કે જેનાથી કોપી render થઈ, વચ્ચે અલગ-અલગ રીસોલ્વ કરે.

## બીજા કારણો {#other-causes}

જો કશું પણ કામ ના કરે તો, મહેરબાની કરીને કમેંટ કરજો [આ issue](https://github.com/facebook/react/issues/13991)અને અમે મદદ કરવાની કોશિસ કરીસું. નાનું ઉદાહરણ બનાવવાનો પ્રયાસ કરો કે જેનાથી આ થયું — તમે સમસ્યા શોધી શકશો જેમ જેમ તમે કરતાં જાઓ.
