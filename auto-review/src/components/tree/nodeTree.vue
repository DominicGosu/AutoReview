<template>
  <li class="node-tree">
    <div class="node" :class="{'active':node.fileItem == activeFile && !isFolder}" @click="onClick(node)">
    <span class="node-folder" v-if="isFolder">{{ node.name }}</span>
    <span class="node-file" v-else >{{ node.name }}</span>
  </div>

    <ul v-if="isFolder">
      <node-tree v-for="(child,index) in node.children" :node="child" :activeFile="activeFile" :key="index" @onActiveFile="onActiveFile"></node-tree>
    </ul>
  </li>
</template>

<script>
export default {
  props: {
    node: Object,
    activeFile:Object
  },
  computed:
  {
    isFolder()
    {
      return this.node.children && this.node.children.length;
    }
  },
  methods:{
    onClick(file)
    {
      let me = this;
      if(!me.isFolder)
      {
        me.onActiveFile(file);
      }
    },
    onActiveFile(file)
    {
      let me = this;
      me.$emit('onActiveFile',file);
    }
  }
};
</script>
<style scoped>
  .node:hover
  {
    background-color: #ccc;
    color: #000;
  }
  .node-tree{
    list-style: none;
  }
  .node-folder
  {
    font-weight: bold;
  }
  .active{
    background-color: #04AA6D!important;
    color: #ffffff!important;
  }
</style>