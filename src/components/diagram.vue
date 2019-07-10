<template>
  <div class="goDemo">
    <el-row style="margin-top:10px;">
      <el-col :span="18">
        <div id="mygoChart" style="width:100%;height:300px; background-color: #F5F5F5;"></div>
      </el-col>
    </el-row>
    <el-dialog title="树图命名" :visible.sync="dialogTreeNameVisible">
      <el-form>
        <el-form-item label="树图名称">
          <el-input type="text" v-model="tree_group_name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogTreeNameVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveTreeName">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog title="节点编辑" :visible.sync="dialogUpdateFormVisible">
      <el-form :model="node_form">
        <el-form-item v-for="(v,i) in lables_col" :key="i" :label="v.cn_name">
          <el-input
            :readonly="v.en_name=='text'?false:true"
            :type="v.en_name=='text'?'textarea':'text'"
            :rows="2"
            v-model="node_form[v.en_name]"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogUpdateFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveNodeInfo">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>


<script>
import go from "gojs";
import 'gojs/extensions/Figures'
var $ = go.GraphObject.make;

// This works because we have overridden the /extensionsTS/tsconfig.json file
// in the options to the loader: 'ts-loader', in the webpack.config.js
import { GuidedDraggingTool } from "gojs/extensionsTS/GuidedDraggingTool.ts";
import { DrawCommandHandler } from "gojs/extensionsTS/DrawCommandHandler.ts";

export default {
  name: "Diagram",
  data() {
    return {
      tree_group_name: "tree_model",
      dialogTreeNameVisible: false,
      div_show: false,
      nodeIdCounter: 50,
      dialogUpdateFormVisible: false,
      sel_nodeInfo: null,
      lables_col: [
        { en_name: "key", cn_name: "编码" },
        { en_name: "text", cn_name: "内容" },
        { en_name: "group", cn_name: "树标" }
      ],
      node_form: { key: "", text: "", group: "" },
      modelData: {
        nodeDataArray: [
          // 节点信息数组
          {
            key: 8,
            text: "根节点",
            group: "tree_1204"
          },
          {
            key: 1,
            text: "节点一",
            group: "tree_1204"
          },
          {
            key: 2,
            text: "节点二",
            group: "tree_1204"
          },
          {
            key: 3,
            text: "节点三",
            group: "tree_1204"
          },
          {
            key: 4,
            text: "节点四",
            group: "tree_1204"
          },
          {
            key: 5,
            text: "节点五",
            group: "tree_1204"
          }
        ],
        linkDataArray: [
          // 节点关系数组
          { id: 1001, from: 8, to: 1, group: "tree_1204" },
          { id: 1002, from: 8, to: 2, group: "tree_1204" },
          { id: 1003, from: 1, to: 3, group: "tree_1204" },
          { id: 1004, from: 1, to: 4, group: "tree_1204" },
          { id: 1005, from: 4, to: 5, group: "tree_1204" }
        ]
      },
      rm_list_node: [],
      rm_list_link: []
    };
  },
  mounted() {
    var mySelf = this;
    const MAKE = go.GraphObject.make;
    // console.log(DrawCommandHandler)

    // 图表初始化配置
    mySelf.myDiagram = MAKE(go.Diagram, "mygoChart", {
      "grid.visible": true,
      "grid.gridCellSize": new go.Size(30, 20),
      "draggingTool.isGridSnapEnabled": true,
      "resizingTool.isGridSnapEnabled": true,
      "rotatingTool.snapAngleMultiple": 90,
      "rotatingTool.snapAngleEpsilon": 45,
      "undoManager.isEnabled": true,
      commandHandler: new DrawCommandHandler(), // defined in DrawCommandHandler.js
      // 'commandHandler.copiesTree': true,

      grid: MAKE(
        // 网格样式
        go.Panel,
        "Grid",
        MAKE(go.Shape, "LineH", {
          stroke: "lightgray",
          strokeWidth: 0.5,
          interval: 5
        }),
        MAKE(go.Shape, "LineV", {
          stroke: "lightgray",
          strokeWidth: 0.5,
          interval: 5
        })
      ),
      padding: 20,
      initialDocumentSpot: go.Spot.TopCenter, // 树图显示位置
      initialViewportSpot: go.Spot.TopCenter,
      maxSelectionCount: 1,
      allowDrop: true,
      allowDelete: true,
      allowCopy: true,
      allowClipboard: true,
      // LinkDrawn: mySelf.showLinkLabel, // 节点连线
      LinkRelinked: mySelf.showLinkLabel,
      scrollsPageOnFocus: false,

      "toolManager.mouseWheelBehavior": go.ToolManager.WheelZoom, // 有鼠标滚轮事件放大和缩小
      layout: MAKE(go.TreeLayout, {
        angle: 90,
        arrangement: go.TreeLayout.ArrangementHorizontal
      }),

      ModelChanged: function(e) {
        // console.log(e)
        // if (e.isTransactionFinished)
        //     console.log(e)
      },
      ChangedSelection: function(e) {
        // self.$emit('changed-selection', e)
      },
      LinkReshaped: function(e) {
        console.log(JSON.parse(mySelf.myDiagram.model.toJson()).linkDataArray);
        console.log(e.subject.data);
        mySelf.myDiagram.model.removeLinkData(e.subject.data);
        // mySelf.myDiagram.model.addLinkData({from: e.subject.data.from, to: e.subject.data.to, group: 'tree_001'})
      }
    });

    mySelf.myDiagram.addDiagramListener("Modified", function(e) {
      var button = document.getElementById("SaveButton");
      if (button) button.disabled = !mySelf.myDiagram.isModified;
      var idx = document.title.indexOf("*");
      if (mySelf.myDiagram.isModified) {
        if (idx < 0) document.title += "*";
      } else {
        if (idx >= 0) document.title = document.title.substr(0, idx);
      }
    });

    mySelf.myDiagram.nodeTemplateMap.add(
      "Process",
      MAKE(
        go.Node,
        "Auto",
        {
          locationSpot: new go.Spot(0.5, 0.5),
          locationObjectName: "SHAPE",
          resizable: true,
          resizeObjectName: "SHAPE"
        },
        new go.Binding("location", "pos", go.Point.parse).makeTwoWay(
          go.Point.stringify
        ),
        MAKE(
          go.Shape,
          "Club",
          {
            name: "SHAPE",
            strokeWidth: 2,
            fill: MAKE(go.Brush, "Linear", {
              start: go.Spot.Left,
              end: go.Spot.Right,
              0: "gray",
              0.5: "white",
              1: "gray"
            }),
            minSize: new go.Size(50, 50),
            portId: "",
            fromSpot: go.Spot.AllSides,
            toSpot: go.Spot.AllSides
          },
          new go.Binding("desiredSize", "size", go.Size.parse).makeTwoWay(
            go.Size.stringify
          )
        ),
        MAKE(
          go.TextBlock,
          {
            alignment: go.Spot.Center,
            textAlign: "center",
            margin: 5,
            editable: true
          },
          new go.Binding("text").makeTwoWay()
        )
      )
    );

    mySelf.myDiagram.nodeTemplateMap.add(
      "Valve",
      MAKE(
        go.Node,
        "Vertical",
        {
          locationSpot: new go.Spot(0.5, 1, 0, -21),
          locationObjectName: "SHAPE",
          selectionObjectName: "SHAPE",
          rotatable: true
        },
        new go.Binding("angle").makeTwoWay(),
        new go.Binding("location", "pos", go.Point.parse).makeTwoWay(
          go.Point.stringify
        ),
        MAKE(
          go.TextBlock,
          {
            alignment: go.Spot.Center,
            textAlign: "center",
            margin: 5,
            editable: true
          },
          new go.Binding("text").makeTwoWay(),
          // keep the text upright, even when the whole node has been rotated upside down
          new go.Binding("angle", "angle", function(a) {
            return a === 180 ? 180 : 0;
          }).ofObject()
        ),
        MAKE(go.Shape, {
          name: "SHAPE",
          geometryString:
            "F1 M0 0 L40 20 40 0 0 20z M20 10 L20 30 M12 30 L28 30",
          strokeWidth: 2,
          fill: MAKE(go.Brush, "Linear", {
            0: "gray",
            0.35: "white",
            0.7: "gray"
          }),
          portId: "",
          fromSpot: new go.Spot(1, 0.35),
          toSpot: new go.Spot(0, 0.35)
        })
      )
    );

    mySelf.myDiagram.linkTemplate = MAKE(
      go.Link,
      {
        routing: go.Link.AvoidsNodes,
        curve: go.Link.JumpGap,
        corner: 10,
        reshapable: true,
        toShortLength: 7
      },
      new go.Binding("points").makeTwoWay(),
      // mark each Shape to get the link geometry with isPanelMain: true
      MAKE(go.Shape, { isPanelMain: true, stroke: "black", strokeWidth: 17 }),
      MAKE(go.Shape, { isPanelMain: true, stroke: "white", strokeWidth: 13 }),
      MAKE(go.Shape, {
        isPanelMain: true,
        stroke: "red",
        strokeWidth: 6,
        name: "PIPE",
        strokeDashArray: [10, 10]
      }),
      MAKE(go.Shape, { toArrow: "Triangle", fill: "black", stroke: null })
    );

    // mySelf.myDiagram.add(
    //   MAKE(go.Part, 'Horizontal',
    //     MAKE(go.Shape, 'Rectangle', { width: 40, height: 60, margin: 4, fill: 'red' }),
    //     MAKE(go.Shape, 'RoundedRectangle', { width: 40, height: 60, margin: 4, fill: null }),
    //     MAKE(go.Shape, 'Ellipse', { width: 40, height: 60, margin: 4, fill: null }),
    //     MAKE(go.Shape, 'Diamond', { width: 40, height: 60, margin: 4, fill: null }),
    //     MAKE(go.Shape, 'TriangleRight', { width: 40, height: 60, margin: 4, fill: null }),
    //     MAKE(go.Shape, 'TriangleDown', { width: 40, height: 60, margin: 4, fill: null }),
    //     MAKE(go.Shape, 'TriangleLeft', { width: 40, height: 60, margin: 4, fill: null }),
    //     MAKE(go.Shape, 'TriangleUp', { width: 40, height: 60, margin: 4, fill: null }),
    //     MAKE(go.Shape, 'MinusLine', { width: 40, height: 60, margin: 4, fill: null }),
    //     MAKE(go.Shape, 'PlusLine', { width: 40, height: 60, margin: 4, fill: null }),
    //     MAKE(go.Shape, 'XLine', { width: 40, height: 60, margin: 4, fill: null })
    //   ))

    this.load();

    this.loop();

    // mySelf.myDiagram.nodeTemplate = MAKE(
    //   go.Node,
    //   'Horizontal',
    //   {background: '#44CCFF' },
    //   MAKE(
    //     go.Picture,
    //     { margin: 10, width: 50, height: 50, background: 'red' },
    //     new go.Binding('source')
    //   ),
    //   MAKE(
    //     go.TextBlock,
    //     'Default Text',
    //     { margin: 12, stroke: 'white', font: 'bold 16px sans-serif' },
    //     new go.Binding('text', 'name'))
    // )

    // var model = new go.Model()

    // model.nodeDataArray = [
    //   { name: 'Don Meow', source: 'cat1.png' },
    //   { name: 'Copricat', source: 'cat2.png' },
    //   { name: 'Demeter', source: 'cat3.png' },
    //   { /* Empty node data */ }
    // ]
    // mySelf.myDiagram.model = model
  },
  methods: {
    // 加载一颗树
    loadTreeData() {
      this.myDiagram.model = go.Model.fromJson(this.modelData);
    },

    // 初始化新树
    initTreeModel() {
      // 初始化一颗新树
      this.dialogTreeNameVisible = true;
    },
    loop() {
      let temp = this;
      var diagram = this.myDiagram;
      setTimeout(function() {
        var oldskips = diagram.skipsUndoManager;
        diagram.skipsUndoManager = true;
        diagram.links.each(function(link) {
          var shape = link.findObject("PIPE");
          var off = shape.strokeDashOffset - 10;
          shape.strokeDashOffset = off <= 0 ? 20 : off;
        });
        diagram.skipsUndoManager = oldskips;
        temp.loop();
      }, 100);
    },
    load() {
      this.myDiagram.model = go.Model.fromJson({
        class: "go.GraphLinksModel",
        nodeDataArray: [
          { key: "P1", category: "Process", pos: "150 120", text: "Process" },
          { key: "P2", category: "Process", pos: "330 320", text: "Tank" },
          { key: "V1", category: "Valve", pos: "270 120", text: "V1" },
          { key: "P3", category: "Process", pos: "150 420", text: "Pump" },
          {
            key: "V2",
            category: "Valve",
            pos: "150 280",
            text: "VM",
            angle: 270
          },
          {
            key: "V3",
            category: "Valve",
            pos: "270 420",
            text: "V2",
            angle: 180
          },
          {
            key: "P4",
            category: "Process",
            pos: "450 140",
            text: "Reserve Tank"
          },
          { key: "V4", category: "Valve", pos: "390 60", text: "VA" },
          {
            key: "V5",
            category: "Valve",
            pos: "450 260",
            text: "VB",
            angle: 90
          }
        ],
        linkDataArray: [
          { from: "P1", to: "V1" },
          { from: "P3", to: "V2" },
          { from: "V2", to: "P1" },
          { from: "P2", to: "V3" },
          { from: "V3", to: "P3" },
          { from: "V1", to: "V4" },
          { from: "V4", to: "P4" },
          { from: "V1", to: "P2" },
          { from: "P4", to: "V5" },
          { from: "V5", to: "P2" }
        ]
      });
    },

    // 新增节点关系
    addLink(link) {
      // console.log(link)
      this.myDiagram.model.addLinkData(link);
      // 后台添加新增的link数据
    },
    // 存储树的组别名称
    saveTreeName() {
      this.myDiagram.model.nodeDataArray = [];
      this.myDiagram.model.linkDataArray = [];
      console.log({ key: 1, text: "NewNode", group: this.tree_group_name });
      var newemp = { key: 1, text: "NewNode", group: this.tree_group_name };
      this.myDiagram.model.addNodeData(newemp);
      this.dialogTreeNameVisible = false;
    },
    // 获取新增节点的key（或id）
    getNextKey() {
      // 新增节点的key，可以从数据库中获取表中最大的node的ID
      var key = this.nodeIdCounter;
      while (this.myDiagram.model.findNodeDataForKey(key) !== null) {
        key = this.nodeIdCounter++;
      }
      return key;
    },
    // 获取整棵树的数据
    getTreeData() {
      let varset = this.myDiagram.model.toJson();
      console.log("这是获取编辑后的树的总数据：");
      console.log(varset);
    },

    // 默认添加子节点
    addNewNodeClick(e, obj) {
      const newkey = this.getNextKey();
      var clicked = obj.part; // 获取点击节点的数据
      if (clicked !== null) {
        var thisemp = clicked.data;

        console.log({ text: "NewNode", group: thisemp.group });
        console.log({ from: thisemp.key, to: "new_key", group: thisemp.group });

        var newemp = {
          key: newkey,
          text: "NewNode",
          group: thisemp.group
        };
        var newlink = { from: thisemp.key, to: newkey, group: thisemp.group };

        this.myDiagram.model.addNodeData(newemp);
        this.myDiagram.model.addLinkData(newlink);
        this.myDiagram.requestUpdate();
      }
    },

    // 节点编辑
    updateNodeInfo(e, obj) {
      this.dialogUpdateFormVisible = true;
      this.sel_nodeInfo = obj.part.adornedPart.data;
      const nodedata = obj.part.adornedPart.data;
      this.node_form = {
        key: nodedata.key,
        text: nodedata.text,
        group: nodedata.group
      };
    },

    // 编辑节点关系提交
    saveNodeInfo() {
      console.log(this.node_form.key); // 修改的节点的ID
      console.log(this.node_form); // 修改后节点的内容

      const txtnew = this.node_form.text;
      this.myDiagram.model.setDataProperty(
        this.myDiagram.selection.first().data,
        "text",
        txtnew
      );
      this.dialogUpdateFormVisible = false;
    },

    // 移除节点及其对应子节点
    removeNodes(e, obj) {
      var node = obj.part.adornedPart;
      this.rm_list_node = []; // 需要删除节点的ID （key）
      this.rm_list_link = []; // 需要删除的关系的ID
      const that = this;
      node.findTreeParts().iterator.each(e => {
        if (e.data.hasOwnProperty("key")) {
          that.rm_list_node.push(e.data.key);
        } else {
          that.rm_list_link.push(e.data.id);
        }
      });

      if (node !== null) {
        this.$confirm(
          "此操作将移除当前节点及与该节点相关各子节点, 是否继续?",
          "提示",
          {
            confirmButtonText: "确定",
            cancelButtonText: "取消",
            type: "warning"
          }
        )
          .then(() => {
            this.myDiagram.startTransaction("remove dept");
            this.myDiagram.removeParts(node.findTreeParts());
            this.myDiagram.commitTransaction("remove dept");

            console.log(this.rm_list_node);
            console.log(this.rm_list_link);

            this.$message({
              type: "success",
              message: "相关节点移除成功!"
            });
          })
          .catch(() => {
            this.$message({
              type: "info",
              message: "已取消移除"
            });
          });
      }
    },

    // 节点样式
    nodeStyle() {
      return [
        new go.Binding("location", "loc", go.Point.parse).makeTwoWay(
          go.Point.stringify
        ),
        {
          locationSpot: go.Spot.Center
        }
      ];
    },
    // 手动增加节点关系方法
    makePort(name, align, spot, output, input) {
      const MAKE = go.GraphObject.make;
      var horizontal =
        align.equals(go.Spot.Top) || align.equals(go.Spot.Bottom);
      return MAKE(go.Shape, {
        fill: "transparent",
        strokeWidth: 0,
        width: horizontal ? NaN : 8,
        height: !horizontal ? NaN : 8,
        alignment: align,
        stretch: horizontal
          ? go.GraphObject.Horizontal
          : go.GraphObject.Vertical,
        portId: name,
        fromSpot: spot,
        fromLinkable: output,
        toSpot: spot,
        toLinkable: input,
        cursor: "pointer",
        mouseEnter: function(e, port) {
          if (!e.diagram.isReadOnly) port.fill = "#00bfff";
        },
        mouseLeave: function(e, port) {
          port.fill = "transparent";
        }
      });
    },

    // text样式
    textStyle() {
      return {
        font: "11pt Helvetica, Arial, sans-serif",
        stroke: "whitesmoke"
      };
    },

    showLinkLabel(e) {
      console.log(e.subject.fromNode.data);
      var label = e.subject.findObject("LABEL");
      console.log(label);
      if (label !== null) {
        label.visible = e.subject.fromNode.data.category === "Conditional";
      }
    }
  }
};
</script>

<style scoped>
.editor_table {
  background-color: burlywood !important;
  /* width: 400px;
  position: absolute;
  top: 50px; */
}
</style>