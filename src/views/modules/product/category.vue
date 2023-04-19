<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="可拖拽"
      inactive-text="不可拖拽"
    >
    </el-switch>
    <el-button type="danger" @click="() => del()">批量删除</el-button>
    <el-tree
      :data="menu"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKeys"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="tree"
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
            edit
          </el-button>
          <el-button
            v-if="node.childNodes.length === 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name"></el-input>
        </el-form-item>
        <el-form-item label="分类图标">
          <el-input v-model="category.icon"></el-input>
        </el-form-item>
        <el-form-item label="分类计量单位">
          <el-input v-model="category.productUnit"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submit()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    return {
      delCategory: [],
      draggable: false,
      updateCategory: [],
      maxLevel: 0,
      title: "",
      submitType: "",
      category: {
        catId: 0,
        icon: "",
        productUnit: "",
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
      },
      dialogVisible: false,
      menu: [],
      expandedKeys: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  //方法集合
  methods: {
    //批量删除节点
    del() {
      //获取当前选中的节点数组
      let nodes = this.$refs.tree.getCheckedNodes();
      if(nodes.length <= 0){
        return;
      }
      for (let i = 0; i < nodes.length; i++) {
        this.delCategory.push(nodes[i].catId);
      }
      this.$confirm(`此操作将删除【${this.delCategory}】, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(this.delCategory, false),
          }).then(({ data }) => {
            this.getDataList();
            this.$message({
              message: `操作成功`,
              type: "success",
            });
          });
        })
        .catch(() => {});
    },
    //弹窗提交类型
    submit() {
      if (this.submitType === "add") {
        this.addCategory();
      }
      if (this.submitType === "edit") {
        this.editCategory();
      }
    },
    //获取最新数据
    getDataList() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
        params: this.$http.adornParams(),
      }).then(({ data }) => {
        console.log(data);
        this.menu = data.categoryEntities;
      });
    },
    //添加分类
    addCategory() {
      //需要更新的数据单独回传
      var { parentCid, catLevel, showStatus, name, sort, productUnit, icon } =
        this.category;
      var data = {
        parentCid,
        catLevel,
        showStatus,
        name,
        sort,
        productUnit,
        icon,
      };
      this.dialogVisible = false;
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(data, false),
      })
        .then(({ data }) => {
          this.getDataList();
          this.expandedKeys = [this.category.parentCid];
          this.$message({
            message: `成功添加【${this.category.name}】`,
            type: "success",
          });
        })
        .catch(({ data }) => {
          this.getDataList();
          this.expandedKeys = [this.category.parentCid];
          this.$message.error("添加失败！");
        });
    },
    append(data) {
      this.submitType = "add";
      this.title = "添加分类";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
    },
    //编辑分类
    editCategory() {
      //需要更新的数据单独回传
      var { catId, icon, productUnit, name } = this.category;
      var data = { catId, icon, productUnit, name };
      this.dialogVisible = false;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(data, false),
      }).then(({ data }) => {
        this.getDataList();
        this.expandedKeys = [catId];
        this.$message({
          message: "操作成功",
          type: "success",
        });
      });
    },
    edit(data) {
      console.log(data);
      this.submitType = "edit";
      this.title = "编辑分类";
      this.dialogVisible = true;
      //回显数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        console.log(data);
        this.category.catId = data.data.catId;
        this.category.name = data.data.name;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        console.log(this.category);
      });
    },
    //删除分类项
    remove(node, data) {
      var ids = [data.catId];

      this.$confirm(`此操作将删除【${data.name}】, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
              });
              //刷新菜单
              this.getDataList();
              //要展开的父节点
              this.expandedKeys = [node.parent.data.catId];
            } else {
              this.$message.error(data.msg);
            }
          });
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    //允许拖拽
    allowDrop(draggingNode, dropNode, type) {
      //恢复默认值
      this.maxLevel = 0;
      this.countDepth(draggingNode.data);
      //拖动节点的深度
      let deep = this.maxLevel - draggingNode.data.catLevel + 1;
      if (type == "inner") {
        return deep + dropNode.data.catLevel <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    //计算节点的最大层级（要计算当前节点的最大深度：拖动节点的最大层级 - 拖动节点的层级 + 1）
    countDepth(node) {
      if (node != null && node.children != null && node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          if (node.children[i].catLevel > this.maxLevel) {
            this.maxLevel = node.children[i].catLevel;
          }
          this.countDepth(node.children[i]);
        }
      } else {
        this.maxLevel = node.catLevel;
      }
    },
    //拖拽成功完成时触发的事件
    handleDrop(draggingNode, dropNode, dropType, ev) {
      //初始化要更新的数组
      this.updateCategory = [];
      //1、拖拽后的节点的最新父节点
      //2、拖拽后的节点的最新层级
      //2.1 拖拽后的节点的子节点的最新层级数
      //3、拖拽后节点的顺序
      let cPid = 0;
      let newLevel = 0;
      let brotherNodes = dropNode.parent.childNodes;
      let cid = draggingNode.data.catId; //拖动节点id
      for (let i = 0; i < brotherNodes.length; i++) {
        if (dropType == "before" || dropType == "after") {
          if (brotherNodes[i].data.catId == cid) {
            cPid = dropNode.data.parentCid;
            newLevel = dropNode.level;
            this.updateCategory.push({
              catId: cid,
              parentCid: cPid,
              catLevel: newLevel,
              sort: i,
            });
            if (newLevel != draggingNode.data.catLevel) {
              this.updateChildNodesLevel(draggingNode.data, newLevel);
            }
            continue;
          }
        } else {
          if (brotherNodes[i].data.catId == cid) {
            cPid = dropNode.data.catId;
            newLevel = dropNode.level + 1;
            this.updateCategory.push({
              catId: cid,
              parentCid: cPid,
              catLevel: newLevel,
              sort: i,
            });
            if (newLevel != draggingNode.data.catLevel) {
              this.updateChildNodesLevel(draggingNode.data, newLevel);
            }
            continue;
          }
        }
        //更新兄弟节点顺序
        this.updateCategory.push({
          catId: brotherNodes[i].data.catId,
          sort: i,
        });
      }
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateCategory, false),
      }).then(({ data }) => {
        //更新菜单
        this.getDataList();
        this.expandedKeys = [cPid];
        this.$message({
          type: "success",
          message: "更新成功!",
        });
      });
    },
    //计算拖拽后的节点的子节点的最新层级数
    //pNode:父节点  pLevel：父节点的层级数
    updateChildNodesLevel(pNode, pLeve) {
      if (pNode != null && pNode.children.length > 0) {
        for (let i = 0; i < pNode.children.length; i++) {
          this.updateCategory.push({
            catId: pNode.children[i].catId,
            catLevel: pLeve + 1,
          });
          this.updateChildNodesLevel(pNode.children[i], pLeve + 1);
        }
      }
    },
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},

  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getDataList();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style lang='scss' scoped>
//@import url(); 引入公共css类
</style>