<template>
  <div style="height:100vh">
    <Split v-model="splitCoefficient"
           :on-move-end="resizeCanvas()">
      <div slot="left"
           class="demo-split-pane"
           id="container">
        <!-- <div id="container"> -->
        <Switch size="large"
                false-color="orange">
          <span slot="open">编辑</span>
          <span slot="close">预览</span>
        </Switch>
        <label>模式切换：</label>
        <RadioGroup v-model="editMode"
                    type="button"
                    vertical>
          <Radio label="default">默认</Radio>
          <Radio label="addNode">添加节点</Radio>
          <Radio label="addEdge">添加连线</Radio>
          <!-- <Radio label="杭州"></Radio> -->
        </RadioGroup>
        <!-- </div> -->
      </div>
      <div slot="right"
           class="demo-split-pane">
        <Card>
          <p slot="title">模块信息</p>
          <Form :model="formRight"
                label-position="right"
                :label-width="100">
            <FormItem label="X坐标">
              <Input v-model="formRight.x" />
            </FormItem>
            <FormItem label="Y坐标">
              <Input v-model="formRight.y" />
            </FormItem>
            <FormItem label="模块名">
              <Input v-model="formRight.label" />
            </FormItem>
            <FormItem>
              <Button type="primary"
                      @click="updateNode">保存</Button>
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
import G6 from '@antv/g6'
function debounce (func, wait) {
  let timeout;
  return function () {
    let context = this;
    let args = arguments;

    if (timeout) clearTimeout(timeout);

    timeout = setTimeout(() => {
      func.apply(context, args)
    }, wait);
  }
}
export default {
  name: 'G6Graph',
  components: {
    // Editor
  },
  data () {
    return {
      value: [20, 50],
      editMode: "default",
      splitCoefficient: 0.5,
      selectedNodes: {},
      formRight: {
        input1: '',
        input2: '',
        input3: '',
        x: 0,
        y: 0,
        label: '',
        ipList: '',
      },
      graph: null,

    }
  },
  computed: {
    leftPanelWidth () {
      if (document.getElementById('container')) {
        return document.getElementById('container').scrollWidth || 500
      }

      return 700
    }
  },
  watch: {
    editMode (value) {
      // console.log(value)
      this.graph.setMode(value)
    },
    leftPanelWidth (value) {
      if (document.getElementById('container')) {
        this.resizeCanvas(value, document.getElementById('container').clientHeight || 500)
      }

    }
  },
  created () {
    // 阻止浏览器默认右键菜单
    document.oncontextmenu = function (event) {
      if (event.stopPropagation) {
        event.stopPropagation()
      }
      if (event.preventDefault) {
        event.preventDefault()
      }
    }
    document.onkeydown = e => {
      let key = window.event.keyCode;
      if (key == 46 || key == 8) {
        this.delSelectedNodes()
      }
    }
  },
  mounted () {

    const _this = this
    let addedCount = 0

    G6.registerBehavior('default', {
      // Set the events and the corresponding responsing function for this behavior
      getEvents () {
        // The event is canvas:click, the responsing function is onClick
        return {
          'node:click': 'onNodeClick'
        }
      },
      onNodeClick (ev) {
        _this.selectedNodes = [ev.item]
      }
    })
    // Register a custom behavior: add a node when user click the blank part of canvas
    G6.registerBehavior('click-add-node', {
      // Set the events and the corresponding responsing function for this behavior
      getEvents () {
        // The event is canvas:click, the responsing function is onClick
        return {
          'canvas:click': 'onClick',
          'node:click': 'onNodeClick'
        }
      },
      // Click event
      onClick (ev) {
        const self = this
        const graph = self.graph
        console.log(ev)
        // Add a new node
        graph.addItem('node', {
          x: ev.canvasX,
          y: ev.canvasY,
          size: [80, 45],
          type: 'rect',
          id: `node-${addedCount}`,
          label: `node-${addedCount}`, // Generate the unique id,
          style: {
            radius: 6,
          }
        })
        this.data = graph.save()
        addedCount++
      },
      onNodeClick (ev) {
        _this.selectedNodes = [ev.item]
      }
    })
    // Register a custom behavior: click two end nodes to add an edge
    G6.registerBehavior('click-add-edge', {
      // Set the events and the corresponding responsing function for this behavior
      getEvents () {
        return {
          'node:click': 'onClick', // The event is canvas:click, the responsing function is onClick
          mousemove: 'onMousemove', // The event is mousemove, the responsing function is onMousemove
          'edge:click': 'onEdgeClick' // The event is edge:click, the responsing function is onEdgeClick
          // onmouseover: 'onMouseover'
        }
      },
      // The responsing function for node:click defined in getEvents
      onClick (ev) {
        const self = this
        const node = ev.item
        _this.selectedNodes = [node]
        const graph = self.graph
        // The position where the mouse clicks
        const point = { x: ev.x, y: ev.y }
        const model = node.getModel()
        if (self.addingEdge && self.edge) {
          graph.updateItem(self.edge, {
            target: model.id
          })

          self.edge = null
          self.addingEdge = false
        } else {
          // Add anew edge, the end node is the current node user clicks
          self.edge = graph.addItem('edge', {
            source: model.id,
            target: model.id,
            style: {
              endArrow: true,
              cursor: 'pointer',
              lineWidth: 2
            }
          })
          self.addingEdge = true
        }
        console.log(ev)
      },
      // The responsing function for mousemove defined in getEvents
      onMousemove (ev) {
        const self = this
        // The current position the mouse clicks
        const point = { x: ev.x, y: ev.y }
        if (self.addingEdge && self.edge) {
          // Update the end node to the current node the mouse clicks
          self.graph.updateItem(self.edge, {
            target: point
          })
        }
      },
      // The responsing function for edge:click defined in getEvents
      onEdgeClick (ev) {
        const self = this
        const currentEdge = ev.item
        if (self.addingEdge && self.edge === currentEdge) {
          self.graph.removeItem(self.edge)
          self.edge = null
          self.addingEdge = false
        }
      },
      onMouseover (ev) {
        console.log(ev)
      }
    })


    // Initial data
    const data = {
      nodes: [
        {
          id: 'node1',
          x: 100,
          y: 200,
          label: 'node1'
        },
        {
          id: 'node2',
          x: 300,
          y: 200
        },
        {
          id: 'node3',
          x: 300,
          y: 300
        }
      ],
      edges: [
        {
          id: 'edge1',
          target: 'node2',
          source: 'node1'
        }
      ],

    }

    // const graphContainer = document.getElementById('container')


    // const selector = document.createElement('select')
    // selector.id = 'selector'
    // const selection1 = document.createElement('option')
    // selection1.value = 'default'
    // selection1.innerHTML = 'Default Mode'
    // const selection2 = document.createElement('option')
    // selection2.value = 'addNode'
    // selection2.innerHTML = 'Add Node (By clicking canvas)'
    // const selection3 = document.createElement('option')
    // selection3.value = 'addEdge'
    // selection3.innerHTML = 'Add Edge (By clicking two end nodes)'
    // selector.appendChild(selection1)
    // selector.appendChild(selection2)
    // selector.appendChild(selection3)
    // graphContainer.appendChild(selector)

    const width = document.getElementById('container').scrollWidth
    // console.log(width)
    const height = document.getElementById('container').scrollHeight || 500
    this.graph = new G6.Graph({
      container: 'container',
      width: 700,
      height,
      // The sets of behavior modes
      modes: {
        // Defualt mode
        default: [
          'node:click',
          'drag-node',
          'click-select',
          // 'drag-canvas',
          'zoom-canvas',
          {
            type: 'brush-select',
            onSelect: nodes => {
              this.selectedNodes = nodes
            }
          }
        ],
        // Adding node mode
        addNode: ['click-add-node', 'click-select'],
        // Adding edge mode
        addEdge: ['click-add-edge', 'click-select']
      },
      // The node styles in different states
      nodeStateStyles: {
        // The node styles in selected state
        selected: {
          stroke: '#666',
          lineWidth: 2,
          fill: 'steelblue'
        },
        // 鼠标 hover 上节点，即 hover 状态为 true 时的样式
        hover: {
          fill: 'lightsteelblue',
          cursor: 'pointer'
        },
      },
      edgeStateStyles: {
        // 鼠标点击边，即 click 状态为 true 时的样式
        click: {
          stroke: 'steelblue',
        },
      },

    })

    // 鼠标进入节点
    this.graph.on('node:mouseenter', e => {
      const nodeItem = e.item; // 获取鼠标进入的节点元素对象
      this.graph.setItemState(nodeItem, 'hover', true); // 设置当前节点的 hover 状态为 true
    });

    // 鼠标离开节点
    this.graph.on('node:mouseleave', e => {
      const nodeItem = e.item; // 获取鼠标离开的节点元素对象
      this.graph.setItemState(nodeItem, 'hover', false); // 设置当前节点的 hover 状态为 false
    });

    // 点击节点
    this.graph.on('node:click', e => {
      // 先将所有当前是 click 状态的节点置为非 click 状态
      const clickNodes = this.graph.findAllByState('node', 'click');
      clickNodes.forEach(cn => {
        this.graph.setItemState(cn, 'click', false);
      });
      const nodeItem = e.item; // 获取被点击的节点元素对象
      console.log(nodeItem)
      this.formRight = nodeItem.getModel()
      console.log(this.formRight)
      this.graph.setItemState(nodeItem, 'click', true); // 设置当前节点的 click 状态为 true
    });
    // 点击边
    this.graph.on('edge:click', e => {
      // 先将所有当前是 click 状态的边置为非 click 状态
      const clickEdges = this.graph.findAllByState('edge', 'click');
      clickEdges.forEach(ce => {
        this.graph.setItemState(ce, 'click', false);
      });
      const edgeItem = e.item; // 获取被点击的边元素对象
      this.graph.setItemState(edgeItem, 'click', true); // 设置当前边的 click 状态为 true
    });
    this.graph.data(data)
    this.graph.render()


  },
  methods: {
    resizeCanvas: debounce(function (w, h) {
      if (this.graph == undefined || this.graph == null)
        return;

      console.log(
        document.body.scrollWidth * this.splitCoefficient)
      let width = document.body.scrollWidth * this.splitCoefficient
      let height = document.getElementById('container').clientHeight
      console.log("resizeCanvas", width, height);
      this.graph.changeSize(width - 40, height - 80)
    }, 100),
    delSelectedNodes () {
      console.log(this.selectedNodes)
      if (this.selectedNodes) {
        if (this.selectedNodes.length > 0) {
          for (var item of this.selectedNodes) {
            this.graph.removeItem(item)
          }
        }
        else {
          this.graph.removeItem(this.selectedNodes)
        }

      }
    },
    updateNode () {

      const item = this.graph.findById(this.formRight.id)
      console.log(item)
      this.graph.updateItem(item)
    }
  }
}
</script>

<!-- Add 'scoped' attribute to limit CSS to this component only -->
<style scoped>
#container {
  /* width: 800px; */
  /* height: 800px; */
  background-color: #f1f1f1;
  display: inline-block;
  /* width: 100%; */
  height: calc(100vh - 20px);
  user-select: none;
  overflow: hidden;
  background: #f8f9fa;
}
</style>
