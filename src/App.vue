<template>
  <header>
    ant-design-vue tree 增删改
  </header>
  <main>
    <a-button type="primary" @click="addComp" v-if="!treeData.length">添加根节点</a-button>
    <a-tree v-if="treeData && treeData.length > 0" :autoExpandParent="true" :tree-data="treeData" :defaultExpandAll="true" showLine blockNode>
      <template v-slot:title="nodeData">
        <span> {{nodeData.name}} </span>
        <a-button-group style="float:right">
          <!--            <a-button size="small" @click="slotAddSame(nodeData)" icon="plus-circle" title="添加同级"></a-button>-->
          <a-button size="small" @click="addComp(nodeData)" title="添加下级">
            <plus-square-outlined />
          </a-button>
          <a-button size="small" @click="slotModify(nodeData)" title="修改">
            <form-outlined />
          </a-button>
          <a-popconfirm title="确定删除该节点吗？" ok-text="确定" cancel-text="取消" @confirm="confirmDel">
            <a-button size="small" @click="slotDelete(nodeData)" title="删除">
              <delete-outlined />
            </a-button>
          </a-popconfirm>
        </a-button-group>
      </template>
    </a-tree>
    <a-modal v-model:visible="visible" title="新建节点" @ok="handleOk">
      名称：<a-input v-model:value="compName" placeholder="请输入节点名称" />
    </a-modal>
    <a-button class="get-btn" type="primary" @click="getTreeData" v-if="treeData.length">获取树的值</a-button>
  </main>
</template>

<script setup>
import {
  PlusSquareOutlined,
  FormOutlined,
  DeleteOutlined
} from '@ant-design/icons-vue';
import { notification } from 'ant-design-vue';
import { ref, toRaw } from 'vue';

const treeData = ref([])
const parentNode = ref({})
const visible = ref(false);
const compName = ref('')
const isUpdate = ref(false)

const showModal = () => {
  visible.value = true;
};

const handleOk = () => {
  if (isUpdate.value) {
    Object.assign(parentNode.value.dataRef, { name: compName.value})
  } else {
    if (!parentNode.value.name) {
      treeData.value.push({ name: compName.value, children: [], key: 0 })
    } else {
      parentNode.value.children.push({ name: compName.value, children: [], key: Math.random() })
    }
  }
  visible.value = false;
};

// 增加下级节点
function addComp(nodeData){
  isUpdate.value = false
  compName.value = ''
  parentNode.value = nodeData
  showModal()
}

// 修改当前节点
function slotModify(nodeData) {
  isUpdate.value = true
  parentNode.value = nodeData
  compName.value = nodeData.dataRef.name
  showModal()
}

// 删除当前节点
function slotDelete(nodeData) {
  parentNode.value = nodeData
}

function confirmDel() {
  Object.assign(parentNode.value.dataRef, null)
  searchOption(parentNode.value.dataRef, treeData.value)
}

function searchOption (option, arr, obj = {}) {
  //首先循环arr最外层数据
  for (let s = 0; s < arr.length; s++) {
    //如果匹配到了arr最外层中的我需要删除的数据
    if (arr[s].key === option.key) {
      //删除即删除即可
      arr.splice(s, 1)
      break
    } else if (arr[s].children && arr[s].children.length > 0) {
      // 递归条件
      searchOption(option, arr[s].children, obj)
    } else {
      continue
    }
  }
}

const openNotification = (msg) => {
  notification.open({
    message: 'tree value',
    description: msg,
    onClick: () => {
      console.log('Notification Clicked!');
    },
    duration: 3,
  });
};

function getTreeData() {
  let details = toRaw(treeData.value)
  console.log(details)
  openNotification(JSON.stringify(details))
}

</script>

<style scoped>

header {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  font-weight: 600;
}
main{
  margin: 100px 400px;
}
.get-btn {
  margin-top: 50px;
}
</style>
