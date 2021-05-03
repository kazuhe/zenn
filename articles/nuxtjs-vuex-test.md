---
title: "Nuxt で Vuex (ストアモジュール分割) に依存したコンポーネントをテストする"
emoji: "⛰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxtjs", "vuex", "jest", "test"]
published: true
---
# はじめに
Nuxtでストアをモジュール分割したVuexをテストする際、コンポーネント側でコールした`mapGetters('counter', ['count'])`などの名前空間を認識してくれない現象が発生します。

NuxtはVuexのモジュールを自動でインポートしてVuexインスタンスを生成してくれますが、テスト時はこの自動インポートが行われないので名前空間を認識できない様です。

以下に対策を記しますが、他にも良い方法があればコメントいただければ幸いです。

# 前提条件
この記事はVuexに依存しているページコンポーネントをテストします。

また、テストは`jest`に加えてVue.js向けのテストライブラリである`vue-test-utils`がインストールされている前提で記述しています。

https://github.com/vuejs/vue-test-utils

# テスト対象
## Vuex store
まずはVuexモジュールの例です。
前述の通りNuxtは以下のコードだけで自動で名前空間付きのモジュールとして解釈してくれます。
```js:store/counter.js
const state = () => ({
  count: 0
})

const getters = {
  count: state => state.count
}

const mutations = {
  increment(state) {
    state.count++
  }
}

const actions = {
  increment({ commit }) {
    commit('increment')
  }
}

export default {
  state,
  getters,
  mutations,
  actions
}
```

## Pages
次にページコンポーネントです。Vuexから値とアクションを取得して表示しているだけの簡単なものです。今回はこのコンポーネントをテストします。
```vue:pages/counter.vue
<template>
  <div>
    <p>Count: {{ count }}</p>
    <button type="button" @click="increment">Increment</button>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'

export default {
  computed: {
    ...mapGetters('counter', ['count'])
  },

  methods: {
    ...mapActions('counter', ['increment'])
  }
}
</script>
```

# テストコード
単純に`store/counter.js`をimportしてVuexインスタンスを生成しても、`counter`モジュールが名前空間として登録されていないので以下の様にエラーがでます。
```
 [vuex] module namespace not found in mapGetters(): counter/
 ```

モジュールオブジェクト（この例ではコピーしたオブジェクト）に`namespaced`プロパティを追加します。これによってNuxtに頼らずにVuexに`counter`名前空間を登録することでき、ページコンポーネントの`mapGetters('counter', ['count'])`がテストでも動作するようになります。
```js:test/pages/counter.test.js
import Vuex from 'vuex'
import { shallowMount, createLocalVue } from '@vue/test-utils'
import cloneDeep from 'lodash.clonedeep'
import counterPage from '~/pages/counter'
import counterStore from '~/store/counter'

const localVue = createLocalVue()
localVue.use(Vuex)

describe('pages/counter.vue', () => {
  let store

  beforeEach(() => {
    const clone = cloneDeep(counterStore)

    // ※モジュールを名前空間として登録する為の設定
    clone.namespaced = true

    // ※モックのVuexインスタンスを生成
    store = new Vuex.Store({
      modules: {
        counter: clone
      }
    })
  })

  it('ボタンのクリックでカウントの値が「+1」される', () => {
    const wrapper = shallowMount(counterPage, { store, localVue })
    expect(wrapper.vm.count).toBe(0)
    wrapper.find('button').trigger('click')
    expect(wrapper.vm.count).toBe(1)
  })
})
```

# 参考
https://vuex.vuejs.org/ja/guide/modules.html
https://vue-test-utils.vuejs.org/ja/guides/#%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88%E5%86%85%E3%81%AE-vuex-%E3%82%92%E3%83%86%E3%82%B9%E3%83%88%E3%81%99%E3%82%8B
