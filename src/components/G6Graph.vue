<template>
  <div>
    <div id="container">
      <Switch size="large"
              false-color="orange">
        <span slot="open">编辑</span>
        <span slot="close">预览</span>
      </Switch>
    </div>
  </div>
</template>

<script>
import G6 from '@antv/g6'
export default {
  name: 'G6Graph',
  components: {
    // Editor
  },
  data () {
    return {
      data: {
        value: [20, 50],
        nodes: [
          {
            id: 'node1',
            label: 'Circle1',
            x: 150,
            y: 150,
            size: 100

          },
          {
            id: 'node2',
            label: 'Circle2',
            x: 400,
            y: 150

          }
        ],
        edges: [
          {
            source: 'node1',
            target: 'node2'
          }
        ]
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
  },
  mounted () {
    /**
     * 该案例演示切换交互模式，在不同模式下实现拖动节点、增加节点、增加边的交互行为。
     */
    let addedCount = 0
    // Register a custom behavior: add a node when user click the blank part of canvas
    G6.registerBehavior('click-add-node', {
      // Set the events and the corresponding responsing function for this behavior
      getEvents () {
        // The event is canvas:click, the responsing function is onClick
        return {
          'canvas:click': 'onClick'
        }
      },
      // Click event
      onClick (ev) {
        const self = this
        const graph = self.graph
        // Add a new node
        graph.addItem('node', {
          x: ev.canvasX,
          y: ev.canvasY,
          size: [80, 45],
          type: 'rect',
          id: `node-${addedCount}` // Generate the unique id
        })
        addedCount++
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
            target: model.id
          })
          self.addingEdge = true
        }
        console.log(ev)
        graph.setItemState(ev.item, 'hover1', true)
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
        console.log(1)
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

    const graphContainer = document.getElementById('container')

    // Add a selector to DOM
    const selector = document.createElement('select')
    selector.id = 'selector'
    const selection1 = document.createElement('option')
    selection1.value = 'default'
    selection1.innerHTML = 'Default Mode'
    const selection2 = document.createElement('option')
    selection2.value = 'addNode'
    selection2.innerHTML = 'Add Node (By clicking canvas)'
    const selection3 = document.createElement('option')
    selection3.value = 'addEdge'
    selection3.innerHTML = 'Add Edge (By clicking two end nodes)'
    selector.appendChild(selection1)
    selector.appendChild(selection2)
    selector.appendChild(selection3)
    graphContainer.appendChild(selector)

    const width = document.getElementById('container').scrollWidth
    const height = document.getElementById('container').scrollHeight || 500
    const graph = new G6.Graph({
      container: 'container',
      width,
      height,
      // The sets of behavior modes
      modes: {
        // Defualt mode
        default: [
          'drag-node',
          'click-select',
          'drag-canvas',
          'zoom-canvas',
          { type: 'brush-select', }
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
        }
      },
      nodeStateStyles: {
        hover1: {
          fill: 'green',
          fontSize: 20,
          label: 'nnnnnn',
          'node1': {

          }
        }
      },
    })
    graph.data(data)
    graph.render()

    // Listen to the selector, change the mode when the selector is changed
    selector.addEventListener('change', e => {
      const value = e.target.value
      // change the behavior mode
      graph.setMode(value)
    })
  }
}
</script>

<!-- Add 'scoped' attribute to limit CSS to this component only -->
<style scoped>
#container {
  width: 800px;
  height: 800px;
  background-color: #f1f1f1;
  display: inline-block;
  width: 100vw;
  height: 100vh;
  user-select: none;
  overflow: hidden;
  background: #f8f9fa;
}
</style>
