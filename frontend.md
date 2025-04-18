- [Embed SolidJS into Vue2](#embed-solidjs-into-vue2)

# Embed SolidJS into Vue2

In solidjs project

`index.tsx`
```ts
/* @refresh reload */
import { render, } from 'solid-js/web';

import App from './App';
declare global {
  interface Window {
    renderSolid: (elem: HTMLElement,data?:any) => any
  }
}
window.renderSolid =  (elem,data)=> {
  render(() => <App name={data.name} onChanged={data.onChanged} onSubmit={data.onSubmit} />, elem!);
}

// if dev
if (import.meta.env.DEV) {
  window.renderSolid(document.getElementById('root')!,{ name: 'Solid', onChanged: (e:string) => console.log(e), onSubmit: (e:string) => console.log(e) });
}
```
`App.tsx`
```tsx
import { createSignal, type Component } from 'solid-js';

interface AppProps {
  name: string;
  onChanged: (s: string) => void;
  onSubmit: (s: string) => void;
}
const App: Component<AppProps> = (props) => {
  const [text, setText] = createSignal(props.name)
  return <div>
    <h1>My Solid App</h1>
    <span>{text()} </span><br />
    <input type="text" value={text()} onInput={e => {
      setText(e.currentTarget.value);
      props.onChanged(e.currentTarget.value);
    }}></input>
    <button onClick={() => {
      props.onSubmit(text());
    }
    }>OK</button>
  </div>;
};

export default App;
```

build your solidjs
```sh
cp dist/assets/index-*.js ../vue-project/public/solidapp.js
```

---
In Vue2 project

`index.html`
```html
<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title><%= htmlWebpackPlugin.options.title %></title>
    <script type="module" src="solidapp.js"></script>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>
```

`App.vue`
```vue
<template>
  <div>
    <h1>{{ name }}</h1>
    <p>
      For a guide and recipes on how to configure / customize this project,<br>
      check out the
    </p>
    <fieldset>
      <legend>Legend</legend>
      <div ref="myapp"></div>
    </fieldset>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data(){
    return {
      name:''
    }
  },
  mounted(){
    console.log('mounted');
    
    const app=this.$refs.myapp;
      window.renderSolid(app,{
        name:this.name,
        onChanged:(s)=>{
          this.name=s;
        },
        onSubmit:(s)=>{
          alert(s);
        }
      });
  }
}
</script>
```
