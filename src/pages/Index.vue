<template lang='pug'>
q-page(:style-fn='fullHeight')
  q-splitter.fit(v-model='splitter', separator-class='bg-primary', separator-style='width: 3px')
    template(v-slot:before)
      prism-editor(v-model='html', @change='walk(doc.body)', language='html')
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
    this.walk(this.doc.body)
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
      return newHtml.replace(/template/g, 'template1')
    },

    convertElement (node, depth) {
      if (node.nodeType === 3 && node.textContent.trim().length) {
        return ' ' + (node.textContent.trim())
      }

      if (node.nodeType !== 1) return
      if (depth === 0 && node.localName === 'template1') return

      const classList = node.className.split(' ').map(className => className && '.' + className).join('')
      const id = node.id ? '#' + node.id : ''
      const attributes = Array.from(node.attributes)
        .filter(attribute => attribute.localName !== 'id' && attribute.localName !== 'class')
        .map(attribute => attribute.localName + (attribute.nodeValue && `='${attribute.nodeValue.replace(/'/g, '`')}'`))
        .join(', ')

      return (`${' '.repeat(depth * 2)}` + (node.localName !== 'div' ? node.localName !== 'template1' ? node.localName : 'template' : '') + id + classList + `${attributes && `(${attributes})`}`)
    },

    walk (node) {
      this.pug = ''
      let depth = 0
      const treeWalker = document.createTreeWalker(node, NodeFilter.SHOW_ALL)

      while (treeWalker.nextNode()) {
        const currentNode = treeWalker.currentNode

        const element = this.convertElement(currentNode, depth)
        if (element) {
          if (currentNode.nodeType === 1 && this.pug) {
            this.pug += '\n'
          }

          this.pug += element

          if (currentNode.hasChildNodes()) {
            depth++
          } else if (!currentNode.nextSibling) {
            depth--
          }
        }
      }
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
