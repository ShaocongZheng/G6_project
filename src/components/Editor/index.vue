<style scoped >
.materials-editor {
  display: inline-block;
  width: 100%;
  height: 100%;
  user-select: none;
  overflow: hidden;
  background: #f8f9fa;
}
</style>
<template>
  <div class="materials-editor"
       @click="handleEditorClick">
    <ToolBar></ToolBar>
    <!-- <Sketchpad></Sketchpad>
    <PanelLeft></PanelLeft>
    <PanelRight></PanelRight>
    <PreviewModel></PreviewModel>
    <ContextMenu></ContextMenu> -->
  </div>
</template>

<script>
// import { mapGetters } from 'vuex'

import ToolBar from './containers/ToolBar'
// import Sketchpad from './containers/Sketchpad'
// import PanelLeft from './containers/PanelLeft'
// import PanelRight from './containers/PanelRight'
// import PreviewModel from './containers/PreviewModel'
// import ContextMenu from './containers/ContextMenu'
import utils from '@/global/g6/utils'
// 扩展了节点、边的G6
import G6 from '@/global/g6/index'
// 导航器
import Minimap from '@antv/g6/lib/plugins/minimap'
// 自定义栅格插件
import XGrid from '@/global/g6/plugins/XGrid'
// 背景图
import XBackground from '@/global/g6/plugins/XBackground'
// 全屏
// import screenfull from 'screenfull'
// 热键
import Mousetrap from 'mousetrap'
// import config from './config/index'
export default {
  name: 'Editor',
  components: {
    ToolBar
  }
}
</script>