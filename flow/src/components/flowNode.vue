<template>
    <div class="node-item"
         ref="node"
         :style="flowNodeContainer"
         @mouseenter="showDelete"
         @mouseleave="hideDelete"
         @mouseup="changeNodeSite"
         @click.stop="editNode">
        <!-- <div class="node-titel">
            <div class="node-icon" v-show="mouseEnter">
                <i class="el-icon-delete" @@click.stop="deleteNode"></i>
            </div>
        </div> -->
        <div class="node-con"><i :class="iconClass" class="type-icon"></i><span>{{node.label}}</span></div>
        <div class="node-del" v-show="mouseEnter" @click.stop="deleteNode">
          <i class="el-icon-circle-close"></i>
        </div>

        <!--连线用--//触发连线的区域-->
        <div class="flow-node-drag" v-show="isconnect"></div>
    </div>
</template>

<script>
    export default {
        props: {
            node: Object,
            isconnect: Boolean,
        },
        data() {
            return {
                mouseEnter: false
            }
        },
        computed: {
            // 节点容器样式
            flowNodeContainer: {
                get() {
                    return {
                        position: 'absolute',
                        minWidth: '80px',
                        top: this.node.top,
                        left: this.node.left,
                        boxShadow: this.mouseEnter ? '#66a6e0 0px 0px 12px 0px' : '',
                    }
                }
            },
             iconClass() {
                 if (this.node.Type == 1) {
                     return 'el-icon-help';
                 } else if (this.node.Type == 2) {
                     return 'el-icon-s-help';
                 } else if (this.node.Type == 3) {
                     return 'el-icon-user';
                 } else if (this.node.Type == 4) {
                     return 'el-icon-s-tools';
                 }
             }
        },
        methods: {
            // 删除节点
            deleteNode() {
                this.$emit('delete-node', this.node.id)
            },
            // 编辑节点
            editNode() {
                this.$emit('edit-node', this.node.id)
            },
            // 鼠标进入
            showDelete() {
                this.mouseEnter = true
            },
            // 鼠标离开
            hideDelete() {
                this.mouseEnter = false
            },
            // 鼠标移动后抬起
            changeNodeSite() {
                this.$emit('changeNodeSite', {
                    nodeId: this.node.id,
                    left: this.$refs.node.style.left,
                    top: this.$refs.node.style.top,
                })
            }
        }
    }
</script>
<style>
  .node-item {
    border-radius: 4px;
    box-shadow: 0 0 2px #696969;
    cursor: move;
    overflow: initial;
    background: #fbf4dc;
    padding: 0 10px;
  }

  .node-titel {
    height: 20px;
    background: #ffc400;
  }

  .node-icon {
    position: absolute;
    top: 0px;
    right: 0px;
    line-height: 20px
  }

  .node-icon i {
    cursor: pointer;
  }

  .node-con {
    text-align: center;
    line-height: 30px;
  }
  .node-del {
      position: absolute;
      color: red;
      font-size: 16px;
      cursor: pointer;
      top: -8px;
      right: -8px;
  }
  .flow-node-drag {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    opacity: 0;
    cursor: crosshair;
  }
</style>
