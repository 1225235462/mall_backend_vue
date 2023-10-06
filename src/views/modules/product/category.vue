<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    ></el-switch>
    <el-button v-if="draggable" @click="batchSave">保存拖拽结果</el-button>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <!-- <span>test</span> -->
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            auto-complete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 认</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      draggable: false,
      updateNodes: [],
      maxLevel: 1,
      title: "",
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productUnit: "",
        icon: "",
        catId: null,
      },
      menus: [],
      expandedKey: [],
      dialogVisible: false,
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
    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除 ${data.name} 菜单？`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(() => {
            // console.log("delete success!");
            this.$message({
              message: "delete success!",
              type: "success",
            });
            this.getMenus();
            this.expandedKey = [node.parent.data.catId];
          });
          console.log("remove", node, data);
        })
        .catch(() => {});
    },
    append(data) {
      console.log("append", data);
      this.title = "添加";
      this.dialogVisible = true;
      this.category = {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productUnit: "",
        icon: "",
        catId: null,
      };
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    edit(data) {
      console.log("edit", data);
      this.title = "修改";
      this.dialogVisible = true;

      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
        params: this.$http.adornData({}),
      }).then(({ data }) => {
        console.log("http edit", data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
    },

    //添加三级分类
    addCategory() {
      console.log("add", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(() => {
        this.$message({
          message: "append success!",
          type: "success",
        });
        this.getMenus();
        this.dialogVisible = false;
        // this.category.name = "";
        this.expandedKey = [this.category.parentCid];
      });
    },

    //修改三级分类
    editCategory() {
      console.log("update", this.category);
      var { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      }).then(() => {
        this.$message({
          message: "edit success!",
          type: "success",
        });
        this.getMenus();
        this.dialogVisible = false;
        this.category.name = "";
        this.expandedKey = [this.category.parentCid];
      });
    },

    submitData() {
      if (this.category.catId == null) {
        this.addCategory();
      } else {
        this.editCategory();
      }
    },

    //拖拽版本1

    // allowDrop(draggingNode, dropNode, type) {
    //   console.log("allowDrop", draggingNode, dropNode, type);

    //   this.countNodeLevel(draggingNode.data);

    //   let deep = this.maxLevel - draggingNode.data.catLevel + 1;
    //   console.log("deep", deep);

    //   if (type == "inner") {
    //     return deep + dropNode.level <= 3;
    //   } else {
    //     return deep + dropNode.parent.level <= 3;
    //   }

    // },

    // countNodeLevel(node) {
    //   if (node.children != null && node.children.length > 0) {
    //     for (let i = 0; i < node.children.length; i++) {
    //       this.countNodeLevel(node.children[i]);
    //     }
    //   } else {
    //     this.maxLevel = node.catLevel;
    //   }
    // },

    //拖拽版本2
    allowDrop(draggingNode, dropNode, type) {
      console.log("allowDrop", draggingNode, dropNode, type);

      this.countNodeLevel(draggingNode);

      let deep = this.maxLevel - draggingNode.level + 1;
      console.log("deep", deep);

      if (type == "inner") {
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },

    countNodeLevel(node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          this.countNodeLevel(node.childNodes[i]);
        }
      } else {
        this.maxLevel = node.level;
      }
    },

    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("drag end", draggingNode, dropNode, dropType);

      let pCid = 0;
      let siblings = null;
      if (dropType == "before" || dropType == "after") {
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }

      for (let i in siblings) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            catLevel = siblings[i].level;
            this.updateChildNodeLevel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel,
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
      // console.log("updateNodes", this.updateNodes);
      // this.$http({
      //   url: this.$http.adornUrl("/product/category/update/sort"),
      //   method: "post",
      //   data: this.$http.adornData(this.updateNodes, false),
      // }).then(() => {
      //   this.$message({
      //     message: "dragging success!",
      //     type: "success",
      //   });
      //   this.expandedKey = [pCid];
      //   this.getMenus();
      // });
    },

    updateChildNodeLevel(node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i in node.childNodes) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.level + 1,
          });
        }
      }
    },

    batchSave() {
      console.log("updateNodes", this.updateNodes);
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(() => {
        this.$message({
          message: "dragging success!",
          type: "success",
        });
        this.updateNodes = [];
        this.getMenus();
      });
    },

    batchDelete() {
      let catIds = [];
      let names = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log("checkedNodes", checkedNodes);
      for (let node of checkedNodes) {
        catIds.push(node.catId);
        names.push(node.name);
      }
      this.$confirm(`是否删除 [${names}] 菜单？`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          }).then(() => {
            this.$message({
              message: "delete success!",
              type: "success",
            });
            this.getMenus();
          });
        })
        .catch(() => {});
    },
  },
  created() {
    this.getMenus();
  },
};
</script>

<style></style>