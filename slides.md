---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS (experimental)
css: unocss
---

# SolidJSを学ぶ

---

# Agenda

- SolidJSとは？
- 用意されているAPI
- アプリ作ってみた
- まとめ

---

# 今日のひとこと

- 音楽理論難しすぎる
- ギターも練習中
  - 数年前に買って積んでいたジャズギター教本
  - スケールを叩き込んでひたすら運指練習
  - 目標はキーやコード進行を理解したうえでの耳コピ

---

# SolidJSとは？

<div style="display: flex; justify-content: space-between">
<ul>
  <li>State of JSに突如登場</li>
  <li>ReactとKnockout.jsに影響を受けたライブラリ</li>
  <li>TypeScriptサポート</li>
  <li>Astro & Viteサポート</li>
</ul>
<img src="/state.png" style="width: 500px" alt="">
</div>

---

# SolidJSの考え方

1. 宣言型データ

データの動作の記述を宣言に結びつけること。データの動作のすべての側面を1つの場所にパッケージ化する。

---

# SolidJSの考え方

2. 消えるコンポーネント

コンポーネント関数は一度呼び出されると消滅する。<br>コンポーネントはコードを整理するために存在し、他の用途はあまりない。

---

# SolidJSの考え方

3. リード / ライトの分離

正確な制御と予測可能性が良いシステムを作る。<br>必ずしも一方通行である必要がないが、どこで書き込みをして良いかなどを意識的に決定する必要がある。

---

# SolidJSの考え方

4. シンプルはイージーに勝る

きめ細やかなリアクティビティのために苦労して得た教訓。<br>明示的で一貫性のある規約は、より多くの努力が必要でも価値はある。<br>目的は、基盤となる最小限のツールを提供することにある。

---

# その他特徴

<div style="display: flex; flex-direction: column">
  <div>
    <h3>DOMを直接制御</h3>
    <p>Reactは仮想DOMを操作するが、SolidはDOMを直接触る</p>
  </div>
  <div>
    <h3>パフォーマンス</h3>
    <div style="display: flex; justify-content: space-between">
      <p>Vanillaと変わりないパフォーマンスらしい</p>
      <img src="/performance.png" alt="" style="width: 400px">
    </div>
  </div>
</div>

---

# createSignal

Reactでいう `useState`<br>
`count` は state を返す getter の役割で、あらゆる場所での変更を検知している

```tsx
import { createSignal } from "solid-js"

export const Counter = () => {
    const [count, setCount] = createSignal(0)
  
    const increment = () => setCount(count() + 1)
    return (
        <button type="button" onClick={increment}>
          {count()}
        </button>
    )
} 
```

---

# createEffect

Reactでいう `useEffect`<br>
なんと第二引数が存在しない！（ `createSignal` の `count` が変更を検知してくれる ）

```tsx
import { createSignal, createEffect } from "solid-js"

export const Counter = () => {
  const [count, setCount] = createSignal(0)

  const increment = () => setCount(count() + 1)
  
  createEffect(() => {
      console.log(`count: ${count()}`)
  })
  return (
      <button type="button" onClick={increment}>
        {count()}
      </button>
  )
} 
```

---

# untrack

Signalの読み取りを追跡したくない場合は、 `untrack` を利用する

```ts
createEffect(() => {
    console.log(foo(), untrack(bar))
})
```

---

# on

`createEffect`自体に計算のための明示的な依存関係を指定もできる<br>
`defer` で最初の変更時にのみ実行させることも可能

```ts
createEffect(on(foo, (foo) => {
    console.log(foo, bar())
}, { defer: true }))
```

---

# 制御フロー

JSXではJSを使ってテンプレートのロジックフローを制御できる。<br>
Solidは仮想DOMがないため、 `Array.prototype.map` などを使用すると、<br>
更新のたびに全てのDOMノードを無駄に再作成してしまう。<br>
そこで、Solidでは、テンプレートヘルパーのようなものをコンポーネントでラップして提供している。

---

# Show

基本的な条件分岐を `<Show>` コンポーネントで表現できる
`fallback` プロパティは `else` の役割を果たす

```tsx
<Show when={isOpen()} fallback={() => <button onClick={toggle}>Open</button>}>
  <button onClick={toggle}>Close</button>
</Show>
```

---

# For

オブジェクトの配列をループするためのコンポーネント<br>
配列が変更されると、 `<For>` はDOM内のアイテムを再生成せず、更新したりできる

```tsx
const Foo = () => {
    const [cats, setCats] = createSignal([
      { id: 'foo', name: 'Maru' },
      { id: 'bar', name: 'Henri' }
    ])
  
    return (
      <For each={cats()}>{(cat, i) =>
        <li>{i() + 1}: {cat.name}</li>
      }</For>
    )
}
```

---

# Switch

2つ以上の条件分岐の際に使用するコンポーネント<br>
最初に `true` と評価されたものをレンダリングして停止し、全て失敗した場合は、 `fallback` をレンダリングする

```tsx
<Switch fallback={<p>{number()} is between 5 and 10}</p>}>
  
  <Match when={number() > 10}>
    <p>I am {number()} year's old.</p>
  </Match>
  
  <Match when={5 > number()}>
    <p>This is a pen.</p>
  </Match>
  
</Switch>
```

---

# Portal

モーダルなどを、通常のレンダリングフローの外の要素に挿入しておくことができる<br>
デフォルトでは、 `document.body` 内の `<div>` にレンダリングされる

```tsx
<Portal>
  <div role='dialog'>
    <h3>Modal</h3>
  </div>
</Portal>
```

---

# props

Solidのpropsオブジェクトは読み取り専用で、Objectゲッターでラップされたリアクティブなプロパティを持っている。これにより、呼び出し元がSignalを使用しているか、静的な値かに関わらず一貫した形式を持つことができる。

propsオブジェクトを分割代入したり、スプレッド演算子を使うと、リアクティビティ性が失われてしまう。
[一応ライブラリはありそう](https://github.com/solidjs-community/solid-primitives/tree/main/packages/destructure)

```tsx
export const Foo: Component<Props> = (props) => {
    // Not Working !
    const { id, name } = props
    return ...
}
```

---

# splitProps

propsオブジェクトを分割代入する例

```tsx
export const Foo: Component<Props> = (props) => {
    const [local, others] = splitProps(props, ['id', 'name'])
    return <MyComponent {...others}>{local.id}: {local.name}</MyComponent>
}
```

---

# Nested Reactivity

例えば、、、<br>

ユーザーのリストを持っていて、ある名前を更新すると、リスト自体には影響を与えずに<br>DOM内の1箇所だけを更新することができる。

---

# Nested Reactivity

Todo Listの例<br>
`addTodo` と `toggleTodo` 実行時に再レンダリングする

<div style="display: flex; gap: 20px">
<div style="width: 500px">
```tsx
const [todos, setTodos] = createSignal([])
let input;
let todoId = 0;

const addTodo = (text) => {
  setTodos([...todos(), { id: ++todoId, text, completed: false }]);
}
const toggleTodo = (id) => {
  setTodos(todos().map((todo) => (
  todo.id !== id ? todo : { ...todo, completed: !todo.completed }
)));
}
```
</div>

<div style="width: 500px">
```tsx
<For each={todos()}>
  {(todo) => {
    const { id, text } = todo;
    console.log(`Creating ${text}`)
    return <div>
      <input
        type="checkbox"
        checked={todo.completed}
        onchange={[toggleTodo, id]}
      />
      <span
        style={{ 
          "text-decoration": todo.completed 
            ? "line-through" 
            : "none"}}
      >{text}</span>
    </div>
  }}
</For>
```
</div>

</div>



---

# Nested Reactivity

Todo Listの例<br>
再レンダリングを防ぐ、 `todo.completed()` を参照するようにする<br>
=> しかし、かなり面倒で分かりづらい

```ts
const addTodo = (text) => {
  const [completed, setCompleted] = createSignal(false);
  setTodos([...todos(), { id: ++todoId, text, completed, setCompleted }]);
};
```

```ts
const toggleTodo = (id) => {
  const index = todos().findIndex((t) => t.id === id);
  const todo = todos()[index];
  if (todo) todo.setCompleted(!todo.completed())
}
```

---

# createStore

Solidでネストしたリアクティビティを担保したい場合、 `createStore` を利用するのが良い感じ<br>
Vue3と同じく `Proxy` でオブジェクトがラップされる


```tsx
const [store, setStore] = createStore({ todos: [] });
const addTodo = (text) => {
  setStore('todos', (todos) => [...todos, { id: ++todoId, text, completed: false }]);
};
const toggleTodo = (id) => {
  setStore('todos', (t) => t.id === id, 'completed', (completed) => !completed);
};
```

---

# mutation

Immerにインスパイアされた `produce` によって、setter関数内で、 Store オブジェクトの書き込みができる<br>
これによってより何をやっているか推論しやすくなる

```tsx
const addTodo = (text) => {
  setStore(
    'todos',
    produce((todos) => {
      todos.push({ id: ++todoId, text, completed: false });
    }),
  );
};
```

```tsx
const toggleTodo = (id) => {
  setStore(
    'todos',
    todo => todo.id === id,
    produce((todo) => (todo.completed = !todo.completed)),
  );
};
```

---

# No Context

`useContext` , `Provide / Inject` 的なコンテキストも存在するが、<br>
そこまで必要ではないな、という場合にも対応可能なテクニックがある

```tsx
import { createSignal } from 'solid-js';

export default createSignal(0);

// どこか別の場所で:
import counter from './counter';
const [count, setCount] = counter;
```

---

# No Context

`createEffect` や `createMemo` を使う場合は、 `createRoot` が必須

```tsx
import { createSignal, createMemo, createRoot } from "solid-js";

function createCounter() {
  const [count, setCount] = createSignal(0);
  const increment = () => setCount(count() + 1);
  const doubleCount = createMemo(() => count() * 2);
  return { count, doubleCount, increment };
}

export default createRoot(createCounter);
```

---

# Suspense

もちろん非同期コンポーネントも用意されている

```tsx
<>
  <h1>Welcome</h1>
  <Suspense fallback={<p>Loading...</p>}>
    <Greeting name="Chiba Tetsuya" />
  </Suspense>
</>
```

---

# アプリを作ってみた

---

# まとめ（SolidJS完全に理解した）

- 最初はReactっぽいかなと思っていたが、意外とVue3と近しい思想でもありそう
  - 例えばReactで `Proxy` を使っているのって `valtio` とか？
- まだエコシステムは揃ってなさそう
  - どうやらTanStackが Solid Query を作ってそう🤔