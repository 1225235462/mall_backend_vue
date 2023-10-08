<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    node-key="catId"
    ref="menuTree"
    @node-click="nodeclick"
  ></el-tree>
</template>

<script>
export default {
  data() {
    return {
      menus: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        methob: "get",
        // params: this.$http.adornParams({})
      }).then(({ data }) => {
        console.log("success!", data.data);
        this.menus = data.data;
      });
    },
    nodeclick(data, node, component) {
      console.log("nodeclick", data, node, component);

      this.$emit("tree-node-click", data, node, component);
    },
  },
  created() {
    this.getMenus();
  },
};
</script>

<style>
</style>