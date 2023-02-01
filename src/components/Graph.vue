<template>
  <div ref="graph"></div>
</template>
<script>
import ForceGraph from 'force-graph';
import {DATA} from "@/constants";
const graph = ForceGraph();
export default {
  name: 'Graph',
  data() {
    return {
      defaultNodeColor: '#444444fa',
      highlightColor: '#999999fa',
      isLinked: (node1, node2) => {
        // checks for a link between 2 nodes
        return node1.links.map(l => (l.source)).includes(node2) ||
            node1.links.map(l => (l.target)).includes(node2) ||
            node2.links.map(l => (l.source)).includes(node1) ||
            node2.links.map(l => (l.target)).includes(node1);
      }
    }
  },
  methods: {
    drawGraph() {
      graph(this.$refs.graph)
          .graphData({nodes: this.getNodes, links: this.getLinks})
          .onNodeClick((node) => {
            graph
              .graphData()
              .nodes
              .forEach(n => {
                // clear previous highlights
                n.color = this.defaultNodeColor;
                // if node is linked to clicked node, highlight
                if (this.isLinked(n, node)) {
                  n.color = this.highlightColor;
                }
              });
          // highlight the clicked node
          node.color = this.highlightColor;
        })
          .nodeCanvasObject((node, ctx, globalScale) => {
            ctx.font = `12px Sans-Serif`;
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillStyle = node.color;
            ctx.beginPath();
            ctx.arc(node.x, node.y, node.val, 0, 2 * Math.PI, false);
            ctx.fill();
            ctx.fillText(
                node.name,
                node.x,
                node.y + node.val + 3,
            );
          });
    },
    getLinksForNode(node) {
      const links = [];
      node.links.forEach(link => {
        links.push({
          source: node.title,
          target: link,
        })
      });
      return links;
    }
  },
  computed: {
    getNodes() {
      return DATA.map(d => ({
        id: d.title,
        name: d.title,
        val: Math.cbrt(d.text.length),
        links: this.getLinksForNode(d),
        color: this.defaultNodeColor
      }))
    },
    getLinks() {
      const links = [];
      this.getNodes.forEach(node => {
        links.push(...node.links);
      })
      return links;
    }
  },
  mounted() {
    this.drawGraph();
  }
}
</script>