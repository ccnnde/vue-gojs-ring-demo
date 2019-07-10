<template>
  <div id="mygoChart" style="width:100%;height:600px; background-color: #F5F5F5;"></div>
</template>


<script>
import go from "gojs";
import { SectorReshapingTool } from "../tool/SectorReshapingTool.ts";

const $ = go.GraphObject.make;

export default {
  name: "Diagram",
  data() {
    return {
      nodeDataArray: [
        {
          key: "0",
          category: "OriginalRing",
          width: 150,
          height: 150,
          angle: 270,
          showScale: false,
          minValue: 0,
          maxValue: 100,
          minorScaleNum: 5,
          majorScaleNum: 10,
          backgroundColor: "rgba(238, 238, 238, 1)",
          stroke: null,
          tickColor: "rgba(0, 0, 0, 1)",
          tip: "no scale"
        },
        {
          key: "1",
          category: "OriginalRing",
          width: 150,
          height: 150,
          angle: 270,
          showScale: true,
          minValue: 0,
          maxValue: 100,
          minorScaleNum: 5,
          majorScaleNum: 10,
          backgroundColor: "rgba(238, 238, 238, 1)",
          stroke: null,
          tickColor: "rgba(0, 0, 0, 1)",
          tip: "show Scale"
        },
        {
          key: "2",
          category: "OriginalRing",
          width: 150,
          height: 150,
          angle: 270,
          showScale: true,
          minValue: 0,
          maxValue: 100,
          minorScaleNum: 5,
          majorScaleNum: 5,
          backgroundColor: "rgba(238, 238, 238, 1)",
          stroke: null,
          tickColor: "rgba(0, 0, 0, 1)",
          tip: "offset"
        },
        {
          key: "3",
          category: "ChangedRing",
          width: 150,
          height: 150,
          angle: 270,
          showScale: false,
          minValue: 0,
          maxValue: 100,
          minorScaleNum: 5,
          majorScaleNum: 10,
          backgroundColor: "rgba(238, 238, 238, 1)",
          stroke: null,
          tickColor: "rgba(0, 0, 0, 1)",
          tip: "no scale"
        },
        {
          key: "4",
          category: "ChangedRing",
          width: 150,
          height: 150,
          angle: 270,
          showScale: true,
          minValue: 0,
          maxValue: 100,
          minorScaleNum: 5,
          majorScaleNum: 10,
          backgroundColor: "rgba(238, 238, 238, 1)",
          stroke: null,
          tickColor: "rgba(0, 0, 0, 1)",
          tip: "show scale"
        },
        {
          key: "5",
          category: "ChangedRing",
          width: 150,
          height: 150,
          angle: 270,
          showScale: true,
          minValue: 0,
          maxValue: 100,
          minorScaleNum: 5,
          majorScaleNum: 5,
          backgroundColor: "rgba(238, 238, 238, 1)",
          stroke: null,
          tickColor: "rgba(0, 0, 0, 1)",
          tip: "still offset"
        }
      ]
    };
  },
  mounted() {
    this.myDiagram = $(go.Diagram, "mygoChart", {
      "resizingTool.computeResize": function(a, b, c, d, e, f) {
        return go.ResizingTool.prototype.computeResize.call(
          this,
          a,
          b,
          c,
          d,
          e,
          false
        );
      }
    });
    this.myDiagram.toolManager.mouseDownTools.add(new SectorReshapingTool());

    // the original template
    this.myDiagram.nodeTemplateMap.add(
      "OriginalRing",
      $(
        go.Node,
        "Auto",
        {
          locationSpot: go.Spot.Center,
          selectionObjectName: "PAD",
          resizeObjectName: "PAD",
          background: "transparent",
          rotatable: false,
          resizable: true
        },
        $(
          go.Panel,
          "Graduated",
          new go.Binding("graduatedTickBase", "minValue"),
          new go.Binding("graduatedMin", "minValue"),
          new go.Binding("graduatedMax", "maxValue"),
          new go.Binding("graduatedTickUnit", "", this.getTickUnit),
          // the outer circle
          $(
            go.Shape,
            { name: "PAD" },
            new go.Binding("width", "width").makeTwoWay(),
            new go.Binding("height", "height").makeTwoWay(),
            new go.Binding("fill", "backgroundColor"),
            new go.Binding("geometry", "", this.makeOuterSector)
          ),
          // the short tick
          $(
            go.Shape,
            { geometryString: "M0 0 V5", alignmentFocus: go.Spot.Bottom },
            new go.Binding("stroke", "tickColor"),
            new go.Binding("visible", "showScale")
          ),
          // the long tick
          $(
            go.Shape,
            { geometryString: "M0 0 V10", alignmentFocus: go.Spot.Bottom },
            new go.Binding("interval", "minorScaleNum"),
            new go.Binding("stroke", "tickColor"),
            new go.Binding("visible", "showScale")
          ),
          // the label
          $(
            go.TextBlock,
            {
              alignmentFocus: go.Spot.Center,
              segmentOrientation: go.Link.OrientUpright,
              segmentOffset: new go.Point(0, -20)
            },
            new go.Binding("interval", "minorScaleNum"),
            new go.Binding("visible", "showScale")
          )
        ),
        // the inner sector
        $(
          go.Shape,
          { strokeWidth: 0, fill: "red" },
          new go.Binding("geometry", "", this.makeInnerSector)
        ),
        // the inner circle
        $(
          go.Shape,
          "Circle",
          { fill: "#fff" },
          new go.Binding("width", "", this.getInnerRadius),
          new go.Binding("height", "", this.getInnerRadius),
          new go.Binding("stroke")
        ),
        // the text
        $(go.TextBlock, new go.Binding("text", "tip"))
      )
    );

    // the changed template
    this.myDiagram.nodeTemplateMap.add(
      "ChangedRing",
      $(
        go.Node,
        "Spot",
        {
          locationSpot: go.Spot.Center,
          selectionObjectName: "PAD",
          resizeObjectName: "PAD",
          background: "transparent",
          rotatable: false,
          resizable: true
        },
        $(
          go.Shape,
          "Circle",
          {
            fill: null,
            stroke: null
          },
          new go.Binding("width", "width"),
          new go.Binding("height", "height")
        ),
        $(
          go.Panel,
          "Graduated",
          { alignmentFocusName: "PAD" },
          new go.Binding("graduatedTickBase", "minValue"),
          new go.Binding("graduatedMin", "minValue"),
          new go.Binding("graduatedMax", "maxValue"),
          new go.Binding("graduatedTickUnit", "", this.getTickUnit),
          // the outer circle
          $(
            go.Shape,
            { name: "PAD" },
            new go.Binding("width", "width").makeTwoWay(),
            new go.Binding("height", "height").makeTwoWay(),
            new go.Binding("fill", "backgroundColor"),
            new go.Binding("geometry", "", this.makeOuterSector)
          ),
          // the short tick
          $(
            go.Shape,
            { geometryString: "M0 0 V5", alignmentFocus: go.Spot.Bottom },
            new go.Binding("stroke", "tickColor"),
            new go.Binding("visible", "showScale")
          ),
          // the long tick
          $(
            go.Shape,
            { geometryString: "M0 0 V10", alignmentFocus: go.Spot.Bottom },
            new go.Binding("interval", "minorScaleNum"),
            new go.Binding("stroke", "tickColor"),
            new go.Binding("visible", "showScale")
          ),
          // the label
          $(
            go.TextBlock,
            {
              alignmentFocus: go.Spot.Center,
              segmentOrientation: go.Link.OrientUpright,
              segmentOffset: new go.Point(0, -20)
            },
            new go.Binding("interval", "minorScaleNum"),
            new go.Binding("visible", "showScale")
          )
        ),
        // the inner sector
        $(
          go.Shape,
          { strokeWidth: 0, fill: "green" },
          new go.Binding("geometry", "", this.makeInnerSector)
        ),
        // the inner circle
        $(
          go.Shape,
          "Circle",
          { fill: "#fff" },
          new go.Binding("width", "", this.getInnerRadius),
          new go.Binding("height", "", this.getInnerRadius),
          new go.Binding("stroke")
        ),
        // the text
        $(go.TextBlock, new go.Binding("text", "tip"))
      )
    );

    this.myDiagram.model = $(go.GraphLinksModel, {
      nodeDataArray: this.nodeDataArray
    });
  },
  methods: {
    makeOuterSector(data) {
      let radius = data.width / 2,
        angle = SectorReshapingTool.getAngle(data),
        sweep = 360,
        start = new go.Point(radius, 0).rotate(angle),
        geo;

      geo = new go.Geometry().add(
        new go.PathFigure(radius + start.x, radius + start.y) // start point
          .add(
            new go.PathSegment(
              go.PathSegment.Arc,
              angle,
              sweep, // angles
              radius,
              radius, // center
              radius,
              radius
            )
          ) // radius
      );

      return geo;
    },

    makeInnerSector(data) {
      let { width, angle } = data,
        radius = width / 2,
        sweep = 108,
        start = new go.Point(radius, 0).rotate(angle),
        geo;

      geo = new go.Geometry()
        .add(
          new go.PathFigure(radius + start.x, radius + start.y) // start point
            .add(
              new go.PathSegment(
                go.PathSegment.Arc,
                angle,
                sweep, // angles
                radius,
                radius, // center
                radius,
                radius
              )
            ) // radius
            .add(
              new go.PathSegment(go.PathSegment.Line, radius, radius).close()
            )
        )
        .add(new go.PathFigure(0, 0)) // make sure the Geometry always includes the whole circle
        .add(new go.PathFigure(2 * radius, 2 * radius));

      return geo;
    },

    getInnerRadius({ width }) {
      let innerWidth = width * 0.8;
      return innerWidth;
    },

    getTickUnit({ minValue, maxValue, majorScaleNum, minorScaleNum }) {
      let unit = Number(
        ((maxValue - minValue) / majorScaleNum / minorScaleNum).toFixed(2)
      );
      return unit;
    }
  }
};
</script>

<style scoped>
</style>