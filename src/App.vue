<template>
<div id="app" class="ui center aligned container">
<header class="ui basic segment">
  <h1 class="ui header">
    <img class="ui rounded image" src="favicon.ico">
    <div class="content">UniHakka
      <div class="sub header">臺灣客家語舊編碼替換套件</div>
    </div>
  </h1>
</header>
<section class="ui basic segment">
  <!-- buttons -->
  <button @click="clearArticle" class="ui compact red button">清除</button>
  <button :disabled="articleEmpty || action" @click="inspectArticle"
    class="ui compact green button">分析</button>
  <button :disabled="articleEmpty || !action" @click="replaceAllArticle"
    class="ui compact blue button">全部替換</button>
  <a href="https://github.com/m80126colin/UniHakka" target="_blank"
    class="ui compact basic icon button"><i class="github alternate icon"></i></a>
  <!-- inspect -->
  <template v-if="action">
    <div class="ui horizontal divider header">
      <i class="info icon"></i>
      分析結果
    </div>
    <div v-if="inspect.length" class="ui stackable doubling four column grid">
    <div v-for="(insp, index) in inspect" :key="index" class="column">
    <div class="ui left aligned clearing segment">
      <button @click="replaceArticle(insp.original, insp.toUnicode)"
        class="ui right floated mini compact circular blue button">替換</button>
      <button @click="findArticle(insp.original)"
        class="ui right floated mini compact circular yellow button">標示</button>
      <div class="ui list">
        <div class="item">
          <strong>原字元：</strong>{{ insp.original }}
          <small>({{ insp.code }})</small>
        </div>
        <div class="ui input item">
          <strong>建議字元：</strong>
          <input type="text" v-model.trim="insp.toUnicode">
        </div>
        <div class="item">
          <strong>資訊：</strong>{{ insp.info }}
        </div>
      </div>
      </div>
    </div>
    </div>
    <div v-else class="ui basic segment">無私有區編碼</div>
  </template>
  <!-- article -->
  <div class="ui horizontal divider header">
    <i class="sticky note icon"></i>
    文章
  </div>
  <editor-content id="article" ref="article" :editor="editor"
    class="ui left aligned basic segment"></editor-content>
  <div class="ui divider header"></div>
</section>
<!-- copyright -->
<footer class="ui basic segment">
  Powered by <a href="https://cli.vuejs.org/" target="_blank">Vue cli</a><br />
  <strong>UniHakka</strong> released under MIT License desinged by <a href="https://github.com/m80126colin" target="_blank">Hsu Subang</a>
</footer>
</div>
</template>

<script>
import Vue from 'vue';
import UniHakka from '@m80126colin/uni-hakka';
import { Editor, EditorContent } from 'tiptap';
import { Placeholder, Search } from 'tiptap-extensions';

class UniHakkaSearch extends Search {
  constructor(options = {}) {
    options.findClass = 'ui uni-hakka circular label'
    super(options)
  }
  get name() {
    return 'uni-hakka-search'
  }
  commands() {
    const cmds = super.commands()
    return {
      ...cmds,
      getContext: () => this.getContext()
    }
  }
  getContext() {
    return ({ doc }) => {
      let result = ''
      doc.descendants(node => {
        if (node.isText)
          result += node.text
        else
          result += '\n'
      })
      return result
    }
  }
};

export default {
  name: 'UniHakkaDemo',
  components: { EditorContent },
  data() {
    const app = this
    const hakka = new UniHakka()
    return {
      hakka,
      action: false,
      inspect: [],
      editor: new Editor({
        onUpdate() {
          app.action = false
        },
        extensions: [
          new UniHakkaSearch(),
          new Placeholder({
            emptyEditorClass: 'empty-editor',
            emptyNodeClass: 'empty',
            emptyNodeText: '請在這裡輸入文字 ...',
            showOnlyCurrent: false
          })
        ]
      })
    }
  },
  beforeDestroy() {
    this.editor.destroy()
  },
  computed: {
    articleEmpty() {
      return this.editor.getHTML().length <= 7
    }
  },
  methods: {
    clearArticle() {
      const app = this
      app.editor.setContent('')
      app.action = false
    },
    inspectArticle() {
      const app = this
      const text = app.editor.commands.getContext()
      const result = app.hakka.inspect(text)
      Vue.set(app, 'inspect', result)
      app.action = true
    },
    findArticle(chr) {
      this.editor.commands.find(chr)
    },
    replaceArticle(chr, target) {
      const app = this
      app.editor.commands.find(chr)
      app.editor.commands.replaceAll(target)
      app.action = true
    },
    replaceAllArticle() {
      const app = this
      app.inspect.forEach(({ original, toUnicode }) => {
        app.editor.commands.find(original)
        app.editor.commands.replaceAll(toUnicode)
        app.action = true
      })
      app.action = false
    }
  }
}
</script>

<style>
#article>div:focus {
  outline: none;
}
#article p.empty-editor:first-child::before {
  content: attr(data-empty-text);
  color: gray;
}
#article div:focus>p.empty-editor:first-child::before {
  content: none;
}

span.ui.uni-hakka.circular.label {
  background-color: rgba(255, 0, 0, 0.12);
}

@font-face {
  font-family: 'HanaMinB';
  src: url('../asset/HanaMinB.ttf');
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif, 'HanaMinB';
  padding-top: 5rem;
  padding-bottom: 5rem;
}
.ui.input.item > input {
  font-family: inherit;
}
</style>