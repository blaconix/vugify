<template lang="pug">
q-layout(view='lhh Lpr lFf')
  q-header(elevated)
    q-toolbar
      q-avatar
        img(src='vugify.png')
      q-toolbar-title Vugify
      q-btn(flat, round, dense, icon='settings')
        q-menu
          .column.q-pa-md.q-gutter-y-md
            .text-h6.q-mb-sm Settings
            q-toggle(v-model='commas', dense, label='Attributes comma' @input='walkDOM')
            q-input(v-model='spaces', outlined, color="primary", dense, type='number', label='# of spaces' @input='walkDOM')
            q-toggle(v-model='firstTemplate', dense, label='Transform root template' @input='walkDOM')
  q-page-container
    q-page(:style-fn='fullHeight')
      q-splitter.fit(v-model='splitter', separator-class='bg-primary', separator-style='width: 5px; box-shadow: rgba(0, 0, 0, 0.05)')
        template(v-slot:before)
          prism-editor.editor(v-model='html', :highlight='highlightHtml', @input='walkDOM')
        template(v-slot:after)
          prism-editor.editor(v-model='pug.trim()', :highlight='highlightPug')
</template>

<script>
import { PrismEditor } from 'vue-prism-editor'
import 'vue-prism-editor/dist/prismeditor.min.css'

import { highlight, languages } from 'prismjs/components/prism-core'
import 'prismjs/components/prism-markup'
import 'prismjs/components/prism-pug'
import 'prismjs/themes/prism-tomorrow.css'

const parser = new DOMParser()
export default {
  name: 'PageIndex',
  components: {
    PrismEditor
  },
  data () {
    return {
      commas: false,
      spaces: 2,
      firstTemplate: true,
      scriptAndStyle: '',
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
    this.walkDOM()
  },
  methods: {
    fullHeight (offset) {
      return { height: offset ? `calc(100vh - ${offset}px)` : '100vh' }
    },

    highlightHtml (code) {
      return highlight(code, languages.html)
    },

    highlightPug (code) {
      return highlight(code, languages.pug)
    },

    removeSelfClosingTags (html) {
      const split = html.split('/>')
      let newHtml = ''

      for (var i = 0; i < split.length - 1; i++) {
        var edsplit = split[i].split('<')
        newHtml += split[i] + '></' + edsplit[edsplit.length - 1].split(' ')[0] + '>'
      }

      newHtml = newHtml + split[split.length - 1]
      if (newHtml.startsWith('<template>')) {
        newHtml = newHtml.replace(/template/, 'tmpl-pug')
      }

      const containsScript = newHtml.indexOf('<script>')
      if (containsScript !== -1) {
        const parts = newHtml.split('<script>')
        this.scriptAndStyle = '\n\n<script>' + parts[1]
        newHtml = parts[0]
      }
      return newHtml.replace(/template/g, 'tmpl')
    },

    convertElement (node, depth) {
      const classList = node.className.split(' ').map(className => className && '.' + className).join('')
      const id = node.id ? '#' + node.id : ''
      const attributes = Array.from(node.attributes)
        .filter(attribute => attribute.localName !== 'id' && attribute.localName !== 'class')
        .map(attribute => attribute.localName + (attribute.nodeValue && `='${attribute.nodeValue.replace(/'/g, '`')}'`))
        .join(this.commas ? ', ' : ' ')
      const name = (node.localName !== 'div' ? node.localName !== 'tmpl' ? node.localName : 'template' : '')
      const text = [].reduce.call(node.childNodes, (a, b) => { return a + (b.nodeType === 3 ? b.textContent : '') }, '').trim()
      return ' '.repeat(depth * this.spaces) + name + id + classList + (attributes && `(${attributes})`) + (text && ' ' + text) + '\n'
    },

    walkDOM () {
      this.pug = ''
      this.scriptAndStyle = ''

      let el = this.doc.body.querySelectorAll('*')
      let templateRoot = false
      if (this.doc.body.getElementsByTagName('tmpl-pug').length) {
        el = this.doc.body.getElementsByTagName('tmpl-pug')[0].querySelectorAll('*')
        templateRoot = true
      }
      const depths = new Map()
      depths.set(templateRoot ? this.doc.body.getElementsByTagName('tmpl-pug')[0] : this.doc.body, -1)
      el.forEach((e) => {
        const p = e.parentNode
        const d = depths.get(p)
        depths.set(e, d + 1)

        this.pug += this.convertElement(e, d + 1)
      })

      if (templateRoot && this.firstTemplate) {
        this.pug = '<template lang="pug">\n' + this.pug + '</template>'
      }

      this.pug += this.scriptAndStyle
    }
  }
}
</script>

<style lang="sass">
body
  background: #222831 !important

.editor
  padding: 20px
  .prism-editor__container
    padding: 10px
    border-radius: 8px
    background: lighten(#222831, 4) !important
    font-family: 'Consolas'
    textarea
      padding: 10px
</style>
