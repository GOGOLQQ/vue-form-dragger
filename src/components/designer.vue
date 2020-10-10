<template>
  <el-container class="dr-designer" :style="size">
    <!-- === start 左侧模块面板 === -->
    <el-aside class="dr-modeler" :width="modelerConfig.width">
      <draggable :list="modelerConfig.list" v-bind="{group:{name: 'viewer',pull:'clone',put:false},sort: false}" :move=moveCommand>
        <div class="dr-module-item" v-for="(item, index) in modelerConfig.list" :key="index">{{ item.name }}</div>
      </draggable>
    </el-aside>
    <!-- === end 左侧模块面板 === -->

    <!-- === start 中间视图面板 === -->
    <el-main class="dr-viewer">
      <el-form inline>
        <container root :selector.sync="selector" :data="list" style="height: 100%" />
      </el-form>
    </el-main>
    <!-- === end 中间视图面板 === -->

    <!-- === start 参数面板 === -->
    <el-aside class="dr-parameter">
      <el-tabs value="setter">
        <el-tab-pane label="数据结构" name="data">
            <el-container>
                <div style="overflow: auto; width: 100%">
                  <pre
                      style="font-family: 'Courier New', serif; padding: 5px"
                  >{{ json }}</pre>
                </div>
            </el-container>
        </el-tab-pane>
        <el-tab-pane label="配置项" name="setter">
          <parameter v-if="selector" :data.sync="selector" />
        </el-tab-pane>
        <el-tab-pane label="其他" name="other">
          <div style="padding: 20px 20px 5px 20px">
            <el-row :gutter="20" justify="space-between" type="flex" style="text-align: center">
              <el-col :span="12">
                <el-button size="mini" style="width: 100px" @click="generator">生成代码</el-button>
              </el-col>
              <el-col :span="12">
                <el-button size="mini" style="width: 100px" @click="previewer">预览</el-button>
              </el-col>
            </el-row>
          </div>
        </el-tab-pane>
      </el-tabs>
    </el-aside>
    <!-- === end 参数面板 === -->

    <!-- === start 代码展示区 === -->
    <el-dialog class="code_dialog" title="代码展示" :visible.sync="codeVisible">
      <div style="width: 100%; border: 1px solid gainsboro; overflow: auto; max-height: 400px" v-highlight>
        <pre style="font-size: 12px; margin: 2px"><code class="html" style="font-family: 'Courier New', serif" v-text="template" /></pre>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button size="mini" @click="codeVisible = false">取 消</el-button>
        <el-button size="mini" type="primary" @click="codeVisible = false">确 定</el-button>
      </span>
    </el-dialog>
    <!-- === end 代码展示区 === -->

    <!-- === start 预览展示区 === -->
    <el-dialog id="preview_dialog" class="preview_dialog" title="预览展示" :visible.sync="previewVisible">
      <div id="preview"></div>
      <div slot="footer" class="dialog-footer">
        <el-button size="mini" @click="previewVisible = false">关闭</el-button>
        <el-button size="mini" type="primary" @click="metadata">获取数据</el-button>
      </div>
    </el-dialog>
    <!-- === end 预览展示区 === -->
  </el-container>
</template>

<script>
import Vue from 'vue'
import formatter from 'vue-beautify'
import { modelerConfig } from './config'
import draggable from 'vuedraggable'
import container from './container'
import parameter from './parameter'

import Coder from '../utils/genarator'


export default {
  name: 'designer',
  data() {
    return {
      modelerConfig,
      list: [],
      selector: undefined,
      codeVisible: false,
      previewVisible: false,
      template: undefined,
      preview: undefined
    }
  },
  props: {
    size: {
      type: Object,
      default: function () {
        return {
          width: '100%',
          height: '100%'
        }
      }
    }
  },
  components: {
    draggable,
    container,
    parameter
  },
  computed: {
    json() {
      return JSON.stringify(this.list, null, 2)
    }
  },
  methods: {
    moveCommand(e) {
      // 容器之间不能同级拖入
      if (e.draggedContext.element.type === 'container' && e.relatedContext.element) {
        return false
      }
      // 其他组件不能与容器同级拖入
      if (e.relatedContext.list) {
        return !e.relatedContext.list.filter(item => item.type === 'container').length > 0
      }
    },
    addCommand(e) {
      console.log('addCommand', e)
    },
    generator() {
      let html = Coder(this.list)
      this.template = formatter(html.template)
      this.codeVisible = true
    },
    previewer() {
      this.previewVisible = true
      const self = this
      Vue.nextTick(function () {
        // DOM 更新了
        const element = self.$el.querySelector('#preview_dialog')
        const children = element.querySelector('#preview')
        if (!children) {
          element.querySelector(".el-dialog__body").innerHTML = '<div id="preview"/>'
        }
        const Preview = Vue.extend(Coder(self.list))
        self.preview = new Preview().$mount('#preview')
      })
    },
    metadata() {
      this.$alert(JSON.stringify(this.preview.local), '数据获取')
    }
  }
}
</script>

<style lang="scss">
/*整体容器样式*/
.dr-designer {
  border: 1px solid slategrey;
  height: 100%;
  font-size: 12px;
}

/*模块面板样式*/
.dr-modeler {
  border-right: 1px solid slategrey;
  .dr-module-item {
    border: 1px solid grey;
    margin: 4px;
    width: 89px;
    display: inline-block;
    line-height: 25px;
    text-align: center;
  }
  .dr-module-item:hover {
    box-shadow: 2px 2px 2px #888888;
    cursor: move;
  }
}

/*视图面板样式*/
.dr-viewer {
  height: 100%;
  overflow: auto;
  form {
    height: 100%;
  }
}

/*参数面板样式*/
.dr-parameter {
  border-left: 1px solid slategrey;
  .el-tabs {
    height: 100%;
    > .el-tabs__header {
      margin-bottom: 0;
    }
  }
  .el-tabs__content {
    height: calc(100% - 40px);
    > .el-tab-pane {
      height: 100%;
    }
    section {
      height: 100%;
    }
  }
}
</style>