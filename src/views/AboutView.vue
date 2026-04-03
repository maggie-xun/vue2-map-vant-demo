<template>
  <div class="about">
    <h1>穿梭树组件</h1>
    <div class="transfer-wrapper">
      <!-- 左侧面板 -->
      <div class="panel left-panel">
        <div class="panel-header">可选列表</div>
        <el-tree ref="leftTree" :data="treeDataSource" :props="defaultProps" node-key="id" show-checkbox
          default-expand-all @check="handleLeftCheck">
        </el-tree>
      </div>

      <!-- 中间穿梭按钮 -->
      <div class="transfer-buttons">
        <el-button type="primary" :disabled="leftChecked.length === 0" @click="transferToRight">
          添加 &gt;
        </el-button>
        <el-button type="primary" :disabled="rightChecked.length === 0" @click="transferToLeft">
          &lt; 移除
        </el-button>
      </div>

      <!-- 右侧面板 -->
      <div class="panel right-panel">
        <div class="panel-header">已选列表 ({{ selectedData.length }})</div>
        <el-tree ref="rightTree" :data="selectedTreeData" :props="defaultProps" node-key="id" show-checkbox
          default-expand-all :expand-on-click-node="false" @check="handleRightCheck">
          <span class="custom-node" slot-scope="{ node, data }">
            <span>{{ node.label }}</span>
            <span class="remove-btn" @click.stop="removeSingleItem(data.id)">×</span>
          </span>
        </el-tree>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "AboutView",
  data() {
    return {
      treeDataSource: [
        {
          id: 1,
          label: "技术部",
          children: [
            {
              id: 11,
              label: "前端组",
              children: [
                { id: 111, label: "React" },
                { id: 112, label: "Vue" },
                { id: 113, label: "Angular" }
              ]
            },
            {
              id: 12,
              label: "后端组",
              children: [
                { id: 121, label: "Java" },
                { id: 122, label: "Node.js" },
                { id: 123, label: "Python" }
              ]
            },
            {
              id: 13,
              label: "运维组",
              children: [
                { id: 131, label: "Docker" },
                { id: 132, label: "Kubernetes" }
              ]
            }
          ]
        },
        {
          id: 2,
          label: "产品部",
          children: [
            {
              id: 21,
              label: "产品设计",
              children: [
                { id: 211, label: "UI设计" },
                { id: 212, label: "交互设计" }
              ]
            },
            {
              id: 22,
              label: "产品经理",
              children: [
                { id: 221, label: "需求分析" },
                { id: 222, label: "项目管理" }
              ]
            }
          ]
        },
        {
          id: 3,
          label: "运营部",
          children: [
            {
              id: 31,
              label: "数据分析",
              children: [
                { id: 311, label: "用户分析" },
                { id: 312, label: "流量分析" }
              ]
            },
            {
              id: 32,
              label: "市场推广",
              children: [
                { id: 321, label: "SEO优化" },
                { id: 322, label: "广告投放" }
              ]
            }
          ]
        }
      ],
      selectedData: [],
      selectedTreeData: [],
      defaultProps: {
        children: "children",
        label: "label"
      },
      leftChecked: [],
      rightChecked: []
    };
  },
  methods: {
    handleLeftCheck(data, checkedObj) {
      const checkedKeys = checkedObj.checkedKeys;
      const halfCheckedKeys = checkedObj.halfCheckedKeys;
      this.leftChecked = [...checkedKeys, ...halfCheckedKeys];
    },
    handleRightCheck(data, checkedObj) {
      this.rightChecked = checkedObj.checkedKeys;
    },
    transferToRight() {
      const newItems = this.leftChecked.filter((id) => !this.selectedData.includes(id));
      newItems.forEach((id) => {
        if (!this.selectedData.includes(id)) {
          this.selectedData.push(id);
        }
      });
      // 更新右侧树
      this.buildSelectedTree();
      // 添加后用 setChecked 逐个设置节点，不触发级联选中父节点
      this.$nextTick(() => {
        if (this.$refs.leftTree) {
          // 先清除所有选中
          this.$refs.leftTree.setCheckedKeys([]);
          // 逐个设置已选节点
          this.selectedData.forEach((id) => {
            this.$refs.leftTree.setChecked(id, true, false);
          });
        }
        this.leftChecked = [...this.selectedData];
      });
    },
    transferToLeft() {
      this.rightChecked.forEach((id) => {
        const idx = this.selectedData.indexOf(id);
        if (idx !== -1) {
          this.selectedData.splice(idx, 1);
        }
      });
      this.$nextTick(() => {
        if (this.$refs.rightTree) {
          this.$refs.rightTree.setCheckedKeys([]);
        }
        // 更新左侧选中状态
        if (this.$refs.leftTree) {
          this.$refs.leftTree.setCheckedKeys([]);
          this.selectedData.forEach((id) => {
            this.$refs.leftTree.setChecked(id, true, false);
          });
        }
      });
      this.rightChecked = [];
    },
    buildSelectedTree() {
      // 深拷贝源树结构
      const copyTree = JSON.parse(JSON.stringify(this.treeDataSource));

      // 过滤：只保留已选中的节点

      // 过滤：只保留已选中的节点，并移除空children
      const filterTree = (nodes) => {
        return nodes
          .filter((node) => this.selectedData.includes(node.id))
          .map((node) => {
            if (node.children) {
              const filteredChildren = filterTree(node.children);
              // 如果children为空数组，删除children属性
              if (filteredChildren.length === 0) {
                return { ...node, children: [] };
              }
              return {
                ...node,
                children: filteredChildren
              };
            }
            return node;
          });
      };


      this.selectedTreeData = filterTree(copyTree);
    },
    findNodeById(id, data = this.treeDataSource) {
      for (const item of data) {
        if (item.id === id) return item;
        if (item.children) {
          const found = this.findNodeById(id, item.children);
          if (found) return found;
        }
      }
      return null;
    },
    // 获取节点的父节点ID
    getParentId(id, data = this.treeDataSource, parentId = null) {
      for (const item of data) {
        if (item.id === id) return parentId;
        if (item.children) {
          const found = this.getParentId(id, item.children, item.id);
          if (found !== null) return found;
        }
      }
      return null;
    },
    // 检查节点是否还有子节点在 selectedData 中
    hasSelectedChildren(parentId) {
      const parent = this.findNodeById(parentId);
      if (!parent || !parent.children) return false;
      return parent.children.some(child => this.selectedData.includes(child.id));
    },
    removeSingleItem(id) {
      const idx = this.selectedData.indexOf(id);
      if (idx !== -1) {
        this.selectedData.splice(idx, 1);

        // 检查父节点是否还有其他选中的子节点
        const parentId = this.getParentId(id);
        if (parentId && !this.hasSelectedChildren(parentId)) {
          // 父节点没有其他选中子节点，也删除父节点
          const parentIdx = this.selectedData.indexOf(parentId);
          if (parentIdx !== -1) {
            this.selectedData.splice(parentIdx, 1);
          }
        }

        // 更新右侧树
        this.buildSelectedTree();
        // 更新左侧树选中状态
        this.$nextTick(() => {
          if (this.$refs.leftTree) {
            this.$refs.leftTree.setCheckedKeys([]);
            this.selectedData.forEach((itemId) => {
              this.$refs.leftTree.setChecked(itemId, true, false);
            });
          }
        });
      }
    }
  }
};
</script>

<style scoped>
.about {
  padding: 20px;
  background: #f5f7fa;
  min-height: 100vh;
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

.transfer-wrapper {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  gap: 20px;
  padding: 0 50px;
}

.panel {
  width: 350px;
  height: 500px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.panel-header {
  padding: 15px;
  background: #409eff;
  color: #fff;
  font-weight: bold;
  text-align: center;
}

.panel :deep(.el-tree) {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
}

.transfer-buttons {
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 15px;
  padding-top: 150px;
}

.custom-node {
  display: flex;
  justify-content: space-between;
  width: 100%;
  padding-right: 10px;
  box-sizing: border-box;
}

.remove-btn {
  color: #f56c6c;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  line-height: 1;
}

.remove-btn:hover {
  color: #c4564a;
}
</style>
