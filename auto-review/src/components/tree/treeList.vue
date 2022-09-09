<template>
    <div class="tree">
      <ul class="tree-list">
        <node-tree :node="data" :activeFile="activeFile" @onActiveFile="activeItem"></node-tree>
      </ul>
    </div>
   </template>
   
   <script>
   import nodeTree from "./nodeTree";
   
   export default {
    props:{
      data:{
        type:Object
      },
      activeFile:{
        type:Object
      }
    },
    components: {
        nodeTree
    },
    methods:{
      activeItem(file)
      {
        let me = this;
        me.$emit('update:activeFile',file.fileItem);
        me.$emit('onActiveFile',file.fileItem);
      }
    }
   };
   </script>
   
   <style>
    .tree-list
    {
      padding:0;
    }
   .tree-list ul {
    margin: 6px 0;
    padding-left:16px;
   }
   </style>