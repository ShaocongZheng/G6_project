<template>
  <div style="height:100vh">
    <Split v-model="splitCoefficient" :on-move-end="resizeCanvas()">
      <div slot="left" class="demo-split-pane">
        <Editor ref="editor" @RightFormData="setRightFormData" :formdata="formRight"></Editor>
      </div>
      <div slot="right" class="demo-split-pane">
        <Card>
          <p slot="title">模块信息</p>
          <Form :model="formRight" label-position="right" :label-width="100">
            <FormItem label="X坐标">
              <Input v-model="formRight.x" />
            </FormItem>
            <FormItem label="Y坐标">
              <Input v-model="formRight.y" />
            </FormItem>
            <FormItem label="模块名">
              <Input v-model="formRight.label" @on-change="updateRightForm" />
            </FormItem>
            <FormItem>
              <Button type="primary" @click="updateRightForm">保存</Button>
              <!-- <Button type="success">Success</Button> -->
              <!-- <Button type="warning">Warning</Button> -->
              <Button type="error">删除</Button>
            </FormItem>
          </Form>
        </Card>
      </div>
    </Split>
  </div>
</template>

<script>
import Editor from './Editor/index'
function debounce(func, wait) {
  let timeout
  return function() {
    let context = this
    let args = arguments

    if (timeout) clearTimeout(timeout)

    timeout = setTimeout(() => {
      func.apply(context, args)
    }, wait)
  }
}
export default {
  name: 'G6Graph',
  components: {
    Editor
  },
  data() {
    return {
      splitCoefficient: 0.5,

      formRight: {
        input1: '',
        input2: '',
        input3: '',
        x: 0,
        y: 0,
        label: '',
        ipList: ''
      }
    }
  },
  computed: {
    // leftPanelWidth () {
    //   if (document.getElementById('container')) {
    //     return document.getElementById('container').scrollWidth || 500
    //   }
    //   return 700
    // }
  },

  created() {
    // 阻止浏览器默认右键菜单
    document.oncontextmenu = function(event) {
      if (event.stopPropagation) {
        event.stopPropagation()
      }
      if (event.preventDefault) {
        event.preventDefault()
      }
    }
    document.onkeydown = e => {
      let key = window.event.keyCode
      if (key == 46) {
        this.$refs.editor.delSelectedNodes()
      }
    }
  },
  mounted() {},
  methods: {
    resizeCanvas: debounce(function(w, h) {

      let width = document.body.scrollWidth * this.splitCoefficient
      let height = document.getElementsByClassName('demo-split-pane')[0].clientHeight
      console.log('resizeCanvas', width, height)
      this.$refs.editor.graph.changeSize(width - 40, height - 80)
    }, 100),
    updateRightForm() {
      this.$refs.editor.updateNode(this.formRight)
    },
    setRightFormData(value){
      this.formRight =value
    }
  }
}
</script>

<!-- Add 'scoped' attribute to limit CSS to this component only -->
<style scoped>
</style>
