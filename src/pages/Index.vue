<template lang='pug'>
q-page(:style-fn='fullHeight')
  q-splitter.fit(v-model='splitter', separator-class='bg-primary', separator-style='width: 3px')
    template(v-slot:before)
      prism-editor(v-model='html', @change='walkDOM(doc.body)', language='html')
    template(v-slot:after)
      prism-editor(v-model='pug', language='pug')
</template>

<script>
import 'prismjs'
import 'prismjs/themes/prism-tomorrow.css'
import 'prismjs/components/prism-pug'

import 'vue-prism-editor/dist/VuePrismEditor.css'
import PrismEditor from 'vue-prism-editor'

const parser = new DOMParser()
export default {
  name: 'PageIndex',
  components: {
    PrismEditor
  },
  data () {
    return {
      html: `<template>
  <div class="q-pa-md">
    <q-table
      title="Treats"
      :data="data"
      :columns="columns"
      row-key="id"
      :filter="filter"
      :loading="loading"
    >

      <template v-slot:top>
        <q-btn flat dense color="primary" :disable="loading" @click="addRow">Add row</q-btn>
        <q-btn class="on-right" flat dense color="primary" :disable="loading" label="Remove row" @click="removeRow" />
        <q-space />
        <q-input borderless dense debounce="300" color="primary" v-model="filter">
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
      </template>

    </q-table>
  </div>
</template>

<!-- Snippet taken from Table component of Quasar Framework -->`,
      pug: '',
      splitter: 50
    }
  },
  computed: {
    doc () {
      return parser.parseFromString(this.removeSelfClosingTags(this.html), 'text/html')
    }
  },
  mounted () {
    this.walkDOM(this.doc.body)
  },
  methods: {
    fullHeight (offset) {
      return { height: offset ? `calc(100vh - ${offset}px)` : '100vh' }
    },

    removeSelfClosingTags (html) {
      const split = html.split('/>')
      let newHtml = ''

      for (var i = 0; i < split.length - 1; i++) {
        var edsplit = split[i].split('<')
        newHtml += split[i] + '></' + edsplit[edsplit.length - 1].split(' ')[0] + '>'
      }

      newHtml = newHtml + split[split.length - 1]
      return newHtml.replace(/template/g, 'tmpl')
    },

    convertElement (node, depth) {
      const classList = node.className.split(' ').map(className => className && '.' + className).join('')
      const id = node.id ? '#' + node.id : ''
      const attributes = Array.from(node.attributes)
        .filter(attribute => attribute.localName !== 'id' && attribute.localName !== 'class')
        .map(attribute => attribute.localName + (attribute.nodeValue && `='${attribute.nodeValue.replace(/'/g, '`')}'`))
        .join(', ')
      const name = (node.localName !== 'div' ? node.localName !== 'tmpl' ? node.localName : 'template' : '')
      const text = [].reduce.call(node.childNodes, (a, b) => { return a + (b.nodeType === 3 ? b.textContent : '') }, '').trim()
      return ' '.repeat(depth * 2) + name + id + classList + (attributes && `(${attributes})`) + (text && ' ' + text) + '\n'
    },

    walkDOM (doc) {
      this.pug = ''
      const el = doc.querySelectorAll('*')
      const depths = new Map()
      depths.set(doc, -1)
      el.forEach((e) => {
        const p = e.parentNode
        const d = depths.get(p)
        depths.set(e, d + 1)

        this.pug += this.convertElement(e, d + 1)
      })
      console.log(depths)
    }
  }
}
</script>

<style lang='stylus'>
body
  background #2D2D2D

.q-field__control
  height 100% !important
</style>
