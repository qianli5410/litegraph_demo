<script setup lang="ts">
import { nextTick } from 'vue';
import { ElMessage } from 'element-plus';

type LiteGraphInstance = any;

let graph: LiteGraphInstance | null = null;
let graphcanvas: any = null

function saveGraph() {
  if (graph) {
    const json = JSON.stringify(graph.serialize());
    const blob = new Blob([json], { type: 'application/json' });
    const url = URL.createObjectURL(blob);

    const a = document.createElement('a');
    a.href = url;
    a.download = 'graph.json';
    a.style.display = 'none';
    document.body.appendChild(a);

    a.click();

    // 清理资源
    setTimeout(() => {
      URL.revokeObjectURL(url);
      document.body.removeChild(a);
    }, 0);
    return
  }
  ElMessage.error('请先创建一个图形');
}

async function initLiteGraphFromJson(json: any): Promise<void> {

  // @ts-ignore
  graph = new LGraph();
  // @ts-ignore

  graphcanvas = new LGraphCanvas("#mycanvas", graph);

  graph.configure(json);

  graph.start();

  // 获取结果
  const node = graph.getNodeById(59);
  console.log(node.value);
  // onSwitch()

}

async function handleFileSelect(): Promise<void> {
  const input = document.createElement('input');
  input.type = 'file';
  input.accept = 'application/json';
  input.style.display = 'none';

  input.addEventListener('change', async (event: any) => {
    const file = event.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = async (e) => {
        // @ts-ignore
        const result = e.target.result;
        if (result) {
          // @ts-ignore
          const json = JSON.parse(result);
          await initLiteGraphFromJson(json);
        }
      };
      reader.readAsText(file);
    }
  });

  document.body.appendChild(input);
  input.click();
}

async function initLiteGraph(): Promise<void> {

  // @ts-ignore
  graph = new LGraph();

  // @ts-ignore
  graphcanvas = new LGraphCanvas("#mycanvas", graph);
  // graphcanvas.canvas.style.display = "none";

  // @ts-ignore
  const node_const = LiteGraph.createNode("basic/const") as any;
  node_const.pos = [200, 200];
  graph.add(node_const);
  node_const.setValue(7.8);

  // @ts-ignore
  const node_watch = LiteGraph.createNode("basic/watch");
  node_watch.pos = [700, 200];
  graph.add(node_watch);

  node_const.connect(0, node_watch, 0);

  // 处理结果
  console.log("node_watch：", node_watch);
  setTimeout(() => {
    console.log("node_watch.value：", node_watch.value);
  }, 0);

  graph.start();
  const anode = graph.getNodeById(2);
  console.log(anode.value);
  

}

const onSwitch = () => {
  console.log(graphcanvas);
  console.log(graphcanvas.live_mode);

  graphcanvas.switchLiveMode(true);
  graphcanvas.draw();
}
nextTick(async () => {
  const canvas = document.getElementById('mycanvas') as HTMLCanvasElement;
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight - 110;
  await initLiteGraph();
});

</script>
 
<template>
  <h3 class="title">litegraph.js 示例</h3>
  <div class="litegraph">
    <canvas id='mycanvas'></canvas>
  </div>
  <div class="btn-box">
    <el-button type="primary" plain @click="saveGraph">保存到本地 <el-icon size="20" style="margin-left: 4px;">
        <Download />
      </el-icon></el-button> <!-- 添加保存按钮 -->
    <el-button type="primary" plain @click="handleFileSelect">上传JSON生成节点流 <el-icon size="20" style="margin-left: 4px;">
        <Upload />
      </el-icon> </el-button> <!-- 添加保存按钮 -->
    <el-button type="primary" plain @click="onSwitch">切换</el-button> <!-- 添加保存按钮 -->

  </div>
</template>
 
<style scoped lang='scss'>
.title {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 50px;
  background-color: #1d2129;
  color: aliceblue;
}

.btn-box {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 50px;
}
</style>