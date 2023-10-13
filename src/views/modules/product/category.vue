<template>
  <div>
    <el-tree :data="menus" :props="defaultProps" node-key="catId" show-checkbox :default-expanded-keys="expandedKeys"
      @node-expand="onNodeExpand" @node-collapse="onNodeCollapse" :auto-expand-parent=false>
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level < 3" type="text" size="mini" @click.stop="() => append(data)">
            Append
          </el-button>
          <el-button v-if="node.isLeaf" type="text" size="mini" @click.stop="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog title="添加分类" :visible.sync="appendCategoryDialogVisible">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="appendCategoryDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="appendCategory">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      expandedKeys: [],
      appendCategoryDialogVisible: false,
      category: { name: "", parentCid: 0, catLevel: 0, showStatus: 1, sort: 0 }
    }
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get',
      }).then(({ data }) => {
        console.log("Get menu data successfully.", data.data)
        this.menus = data.data
      })
    },

    append(data) {
      console.log("append", data);
      this.category.name = "";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      // this.category.showStatus = 1;
      // this.category.sort = 1;
      this.appendCategoryDialogVisible = true;
    },

    appendCategory() {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({ data: retData }) => {
        this.$message({
          type: retData.code === 0 ? "success" : "error",
          message: retData.code === 0 ?
            `添加分类【${this.category.name}】成功` : `添加分类【${this.category.name}】失败`
        });
        this.appendCategoryDialogVisible = false;
        this.getMenus();
      })
    },

    remove(node, data) {
      console.log("remove", node, data)

      this.$confirm(`删除分类【${data.name}】吗?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData([data.catId], false)
        }).then(({ data: retData }) => {
          this.$message({
            type: retData.code === 0 ? "success" : "error",
            message: retData.code === 0 ?
              "删除分类【${data.name}】成功" : "删除分类【${data.name}】失败"
          });
          this.getMenus();
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },

    onNodeExpand(data, _node, _component) {
      this.expandedKeys.push(data.catId);
    },

    onNodeCollapse(data, _node, _component) {
      let targetIndex = this.expandedKeys.indexOf(data.catId);
      // console.log(targetIndex);
      if (targetIndex !== -1) {
        this.expandedKeys.splice(this.expandedKeys.indexOf(data.catId), 1);
      }
    }
  },
  created() {
    this.getMenus();
  }
}
</script>