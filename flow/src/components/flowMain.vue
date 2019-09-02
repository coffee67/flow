<template>
  <div class="appcontainer">
    <div class="flow-menu">
      <div class="menu-item" v-for="item in menueList" draggable="true" @dragstart="drag(item)">
        <i :class="item.icon"></i>
        <div>{{item.name}}</div>
      </div>
      <hr>
      <div class="menu-item" @click="isConnect=true">
        <i class="el-icon-bottom-right"></i>
        <div>连线</div>
      </div>
      <div class="menu-item" @click="isConnect=false">
        <i class="el-icon-rank"></i>
        <div>选择</div>
      </div>

    </div>
    <div class="flow-detail">
      <div id="flowContent"
           ref="flowContent"
           @drop="drop($event)"
           @dragover="allowDrop($event)"
           @click="editFlow()"
           @dblclick="isConnect=false">
        <flowNode v-for="node in data.nodeList" :key="node.id"
                  :node="node"
                  :id="node.id"
                  :isconnect="isConnect"
                  @delete-node="deleteNode"
                  @change-node-site="changeNodeSite"
                  @edit-node="editNode">
        </flowNode>
      </div>
      <div class="savebtn">
        <el-button type="primary" size="mini" @click="saveData()">保 存</el-button>
      </div>
    </div>
    <div class="flow-edit">
      <div class="flow-edit-content">
        <edit-flow ref="flowEdit" v-show="editType=='flow'"></edit-flow>
        <edit-node ref="nodeForm" v-show="editType=='node'"></edit-node>
        <edit-line ref="lineForm" v-show="editType=='line'" @line-save="lineLabelSave"></edit-line>
      </div>
    </div>

  </div>
</template>

<script>
  import {jsPlumb} from 'jsplumb'
  import flowNode from './flowNode.vue'
  import editFlow from './editFlow.vue'
  import editNode from './editNode.vue'
  import editLine from './editLine.vue'
  export default {
    name: 'flowMain',
    data() {
      return {
        menueList: [{
            type: 1,
            name: '起点',
            icon: 'el-icon-help'
          },
          {
            type: 2,
            name: '终点',
            icon: 'el-icon-s-help'
          },
          {
            type: 3,
            name: '人工活动',
            icon: 'el-icon-user'
          },
          {
            type: 4,
            name: '自动活动',
            icon: 'el-icon-s-tools'
          },
        ],
        jsPlumb: null, // jsPlumb 实例
        index: 1,
        // 默认设置参数
        jsplumbSetting: {
          // 动态锚点、位置自适应
          Anchors: ['Top', 'TopCenter', 'TopRight', 'TopLeft', 'Right', 'RightMiddle', 'Bottom', 'BottomCenter',
            'BottomRight', 'BottomLeft', 'Left', 'LeftMiddle'
          ],
          Container: 'flowContent',
          // 连线的样式 StateMachine、Flowchart,有四种默认类型：Bezier（贝塞尔曲线），Straight（直线），Flowchart（流程图），State machine（状态机）
          Connector: 'Straight',
          //这个是鼠标拉出来的线的属性
          ConnectionOverlays: [
            ["Label", {
              label: "连线文本",
              id: "label-1",
              cssClass: "csslabel"
            }]
          ],
          // 鼠标不能拖动删除线
          ConnectionsDetachable: false,
          // 删除线的时候节点不删除
          DeleteEndpointsOnDetach: false,
          // 连线的端点
          // Endpoint: ["Dot", {radius: 5}],
          Endpoint: ["Rectangle", {
            height: 10,
            width: 10
          }],
          // 线端点的样式
          EndpointStyle: {
            fill: 'rgba(255,255,255,0)',
            outlineWidth: 1
          },
          LogEnabled: true, //是否打开jsPlumb的内部日志记录
          // 绘制线
          PaintStyle: {
            stroke: '#409eff',
            strokeWidth: 2
          },
          // 绘制箭头
          Overlays: [
            ['Arrow', {
              width: 8,
              length: 8,
              location: 1
            }],
          ],
          RenderMode: "svg"
        },
        // jsplumb连接参数
        jsplumbConnectOptions: {
          isSource: true,
          isTarget: true,
          // 动态锚点、提供了4个方向 Continuous、AutoDefault
          anchor: "Continuous",
        },
        jsplumbSourceOptions: {
          filter: ".flow-node-drag",  //触发连线的区域
          /*"span"表示标签，".className"表示类，"#id"表示元素id*/
          filterExclude: false,
          anchor: "Continuous",
          allowLoopback: false
        },
        jsplumbTargetOptions: {
          filter: ".flow-node-drag",
          /*"span"表示标签，".className"表示类，"#id"表示元素id*/
          filterExclude: false,
          anchor: "Continuous",
          allowLoopback: false
        },
        // 是否加载完毕
        loadEasyFlowFinish: false,
        // 数据
        data: {
          flowInfo: {
            Id: this.getUUID(),
            Name: '我的流程',
            Remark: '',
          },
          nodeList: [],
          lineList: []
        },
        currentItem: '', //临时存添加的元素
        isConnect: false, //判断是否连接
        timer: null, //定时器,判断单双击,
        currentConnect: '', //当前的连接线
        currentLine: '', //当前连接线数据
        editType: '', //编辑的类型
      }
    },
    components: {
      flowNode,
      editFlow,
      editNode,
      editLine
    },
    created() {},
    mounted() {
      this.jsPlumb = jsPlumb.getInstance();
      this.$nextTick(() => {
        this.init();
      });
      this.editFlow()
    },
    methods: {
      init() {
        const _this = this
        this.jsPlumb.ready(function() {

          // 导入默认配置
          _this.jsPlumb.importDefaults(_this.jsplumbSetting)
          // 会使整个jsPlumb立即重绘。
          _this.jsPlumb.setSuspendDrawing(false, true);
          // 初始化节点
          _this.loadEasyFlow()

          // 单点连接线（编辑label）,
          _this.jsPlumb.bind('click', function(conn, originalEvent) {
            //clearTimeout(this.timer);
            //this.timer = setTimeout(function () { // 这里采用执行自定义事件的方式
            //    console.log("click", conn);
            //    _this.editLine(conn);
            //}, 300); // 延迟300ms执行单击事件,区分双击事件
            console.log("click", conn);
            _this.editLine(conn);
            console.log(conn.getOverlay("label-1"));
            //conn.getOverlay("label-1").setLabel('大肥肚'); //初始化label
          })
          // 双击连接线（删除）,
          _this.jsPlumb.bind('dblclick', function (conn, originalEvent) {
             clearTimeout(this.timer);
             console.log("dblclick", conn)

             _this.$confirm('确定删除所点击的线吗?', '提示', {
                 confirmButtonText: '确定',
                 cancelButtonText: '取消',
                 type: 'warning'
             }).then(() => {
                 _this.jsPlumb.deleteConnection(conn)
             }).catch(() => { })
          })
          // 连线
          _this.jsPlumb.bind("connection", function(evt) {
            console.log('connection', evt)
            let from = evt.source.id
            let to = evt.target.id
            if (_this.loadEasyFlowFinish) {
              _this.data.lineList.push({
                from: from,
                to: to,
                label: '连线名称',
                id: _this.getUUID(),
                Remark: '',
              })
            };
            setTimeout(function() {
              _this.editLine(evt.connection);
            }, 100);
          })

          // 删除连线
          _this.jsPlumb.bind("connectionDetached", function(evt) {
            console.log('connectionDetached', evt)
            _this.deleteLine(evt.sourceId, evt.targetId)
          })

          // 改变线的连接节点
          _this.jsPlumb.bind("connectionMoved", function(evt) {
            console.log('connectionMoved', evt)
            _this.changeLine(evt.originalSourceId, evt.originalTargetId)
          })

          // 单击endpoint
          // jsPlumb.bind("endpointClick", function (evt) {
          //   console.log('endpointClick', evt)
          // })
          //
          // // 双击endpoint
          // jsPlumb.bind("endpointDblClick", function (evt) {
          //   console.log('endpointDblClick', evt)
          // })

          // contextmenu 右键
          _this.jsPlumb.bind("contextmenu", function(evt) {
            console.log('contextmenu', evt)
          })


          // beforeDrop
          _this.jsPlumb.bind("beforeDrop", function(evt) {
            console.log('beforeDrop', evt)
            let from = evt.sourceId
            let to = evt.targetId
            if (from === to) {
              _this.$message.error('不能连接自己');
              return false
            }
            if (_this.hasLine(from, to)) {
              _this.$message.error('不能重复连线');
              return false
            }
            if (_this.hashOppositeLine(from, to)) {
              _this.$message.error('不能回环哦');
              return false
            }
            return true
          })

          // beforeDetach
          _this.jsPlumb.bind("beforeDetach", function(evt) {
            console.log('beforeDetach', evt)
          })
        })

      },
      // 加载流程图
      loadEasyFlow() {

        // 初始化节点
        for (var i = 0; i < this.data.nodeList.length; i++) {
          let node = this.data.nodeList[i]
          // 设置源点，可以拖出线连接其他节点
          this.jsPlumb.makeSource(node.id, this.jsplumbSourceOptions)
          // // 设置目标点，其他源点拖出的线可以连接该节点
          this.jsPlumb.makeTarget(node.id, this.jsplumbTargetOptions)
          // jsPlumb.addEndpoint(node.id)
          // 设置可拖拽
          // jsPlumb.draggable(node.id, {
          //     containment: 'parent',
          //     grid: [10, 10]
          // })

          this.jsPlumb.draggable(node.id, {
            containment: 'parent'
          })

          // jsPlumb.draggable(node.id)
        }

        // 初始化连线
        for (var i = 0; i < this.data.lineList.length; i++) {
          let line = this.data.lineList[i]
          let connection = this.jsPlumb.connect({
            source: line.from,
            target: line.to,
          }, this.jsplumbConnectOptions);

          connection.getOverlay("label-1").setLabel(line.label); //初始化label
        }
        this.$nextTick(function() {
          this.loadEasyFlowFinish = true
        })
      },
      // 添加新的节点
      addNode(temp) {
        console.log('添加节点', temp)
        this.data.nodeList.push(temp);
        this.$nextTick(function() {
          this.jsPlumb.makeSource(temp.id, this.jsplumbSourceOptions)
          this.jsPlumb.makeTarget(temp.id, this.jsplumbTargetOptions)
          this.jsPlumb.draggable(temp.id, {
            containment: 'parent'
          })

        })
      },
      // 删除线
      deleteLine(from, to) {
        this.data.lineList = this.data.lineList.filter(function(line) {
          return line.from !== from && line.to !== to
        })
      },
      // 改变连线
      changeLine(oldFrom, oldTo) {
        this.deleteLine(oldFrom, oldTo)
      },
      // 改变节点的位置
      changeNodeSite(data) {
        for (var i = 0; i < this.data.nodeList.length; i++) {
          let node = this.data.nodeList[i]
          if (node.id === data.nodeId) {
            node.left = data.left
            node.top = data.top
          }
        }
      },
      //删除节点
      deleteNode(nodeId) {
        this.$confirm('确定要删除节点' + nodeId + '?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          closeOnClickModal: false
        }).then(() => {
          // this.data.nodeList = this.data.nodeList.filter(function(node) {
          // 	if (node.id === nodeId) {
          // 		node.show = false
          // 	}
          // 	return true
          // })
          this.data.nodeList.forEach((item, index) => {
            if (item.id === nodeId) {
              this.data.nodeList.splice(index, 1);
            }
          })

          this.$nextTick(function() {
            console.log('删除' + nodeId)
            this.jsPlumb.removeAllEndpoints(nodeId);
          })
        }).catch(() => {})
        return true
      },
      //编辑节点
      editNode(nodeId) {
        //console.log('编辑节点', nodeId)
        this.editType = 'node';
        this.$nextTick(function() {
          this.$refs.nodeForm.init(this.data, nodeId)
        })
      },
      editLine(conn) {
        var _this = this;
        _this.currentConnect = conn;
        _this.data.lineList.forEach(function(item, index) {
          if (item.from == conn.sourceId && item.to == conn.targetId) {
            _this.currentLine = item;
            _this.editType = 'line';
            _this.$nextTick(function() {
              _this.$refs.lineForm.init(item)
            })
            return;
          }
        });
      },
      //删除线
      delLine(conn) {
        this.$confirm('确定删除所点击的线吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.jsPlumb.deleteConnection(conn)
        }).catch(() => {})
      },
      // 是否具有该线
      hasLine(from, to) {
        for (var i = 0; i < this.data.lineList.length; i++) {
          var line = this.data.lineList[i]
          if (line.from === from && line.to === to) {
            return true
          }
        }
        return false
      },
      // 是否含有相反的线
      hashOppositeLine(from, to) {
        return this.hasLine(to, from)
      },
      lineLabelSave(line) {
        this.currentConnect.getOverlay("label-1").setLabel(line.label);
        //this.$set(this.currentLine, 'label', line.label);
      },
      drag(item) {
        this.currentItem = item;
      },
      drop(event) {
        //event.preventDefault();
        var _obj = this.$refs.flowContent;
        var temp = {
          id: this.getUUID(),
          label: this.currentItem.name,
          top: event.offsetY + 'px',
          left: event.offsetX + 'px',
          Type: this.currentItem.type
        }
        this.addNode(temp);
        this.editNode(temp.id);
      },
      allowDrop(event) {
        event.preventDefault()
      },
      getUUID() {
        var s = [];
        var hexDigits = "0123456789abcdef";
        for (var i = 0; i < 36; i++) {
          s[i] = hexDigits.substr(Math.floor(Math.random() * 0x10), 1);
        }
        s[14] = "4"; // bits 12-15 of the time_hi_and_version field to 0010
        s[19] = hexDigits.substr((s[19] & 0x3) | 0x8, 1); // bits 6-7 of the clock_seq_hi_and_reserved to 01
        s[8] = s[13] = s[18] = s[23] = "-";
        var uuid = s.join("");
        return uuid;
      },
      saveData() {
        //console.log(this.jsPlumb)
        //console.log(this.jsPlumb.Defaults)
        //console.log('线', this.jsPlumb.getConnections())
        console.log(this.data);
      },
      editFlow() {
        this.editType = 'flow';
        this.$nextTick(function() {
          this.$refs.flowEdit.init(this.data.flowInfo);
        })
      }
    }
  }
</script>

<style>
  .appcontainer {
    display: flex;
    height: calc(100vh - 20px);
    color: #606266;
  }

  .flow-detail {
    flex: 1;
    border: 1px solid #EBEEF5;
    padding: 12px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, .1);
    word-break: break-all;
    border-radius: 4px;
    position: relative;
  }

  #flowContent {
    width: 100%;
    height: 100%;
    position: relative;
  }

  .item {
    position: absolute;
    width: 60px;
    height: 90px;
    border: 1px solid #007AFF;
  }

  .flow-menu {
    width: 100px;
    padding: 10px;
    margin: 0 10px;
    border: 1px solid #EBEEF5;
    padding: 12px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, .1);
    word-break: break-all;
    border-radius: 4px;
  }

  .menu-item {
    text-align: center;
    box-shadow: 0 0 4px #696969;
    margin-bottom: 8px;
    cursor: pointer;
    font-size: 14px;
    color: #606266;
    padding: 5px 0;
    border-radius: 5px;
  }

  .menu-item>i {
    font-size: 30px;
  }

  .flow-edit {
    width: 300px;
    padding: 0 10px;
  }

  .flow-edit-content {
    background: #FFF;
    color: #606266;
    line-height: 1.4;
    text-align: justify;
    font-size: 14px;
    border: 1px solid #EBEEF5;
    padding: 12px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, .1);
    word-break: break-all;
    border-radius: 4px;
  }
  .csslabel {
        color: #434343;
        font-weight: 400;
        z-index: 10;
        font-size: 12px;
        color: #409eff;
        background: #fff;
    }
    .csslabel .label-text {
        background-color: white;
    }
  .savebtn {
    position: absolute;
    top: 5px;
    right: 5px;
  }
</style>
