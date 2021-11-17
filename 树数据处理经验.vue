<template>
  <div>自用：树数据的处理经验</div>
</template>

<script>
export default {
  mounted() {
    /**
     * 因为这里是要2个请求接口都返回后才能处理，所以使用Promise.all做一个等待
     * 请求接口就是正常的http请求，就不再赘述
     */
    Promise.all([this.getDevice(), this.getOrgan()]).then((res) => {
      this.treeHandleData(res[0], res[1])
    })
  },
  methods: {
    // 树数据处理
    treeHandleData(deviceArr, organArr) {
      /**
       * 使用说用，处理树形数据的合并
       * @params deviceArr是一个多维数组
       * [
       *   {name:'qq',id:1,children:[
       *      {name:'qq-dd', id:12, children:[
       *      .......]},
       *      .......
       *   ]},
       *   {name:'aa',id:2,children:[.....]},
       *   ......
       * ]
       * @params organArr也是一维数组，但需要改变结构为多维数组
       */
      const idArr = [];
      const temArr = [];
      // 这里是对有重复数据的需要处理的数组操作
      deviceArr.map((item) => {
        // 判断有没有父的id数组
        if (item.Parent) {
          // 遍历整个父id数组
          item.Parent.map((param) => {
            // 用临时数组idArr做监测，是否是重复的值
            if (idArr.indexOf(param) === -1) {
              // 没有的话就添加进临时数组temArr
              temArr.push({
                name: param,
                arr: [item],
              });
              // 然后把重复的值放入临时数组idArr以便后面循环继续判断
              idArr.push(param);
            } else {
              // 临时数组idArr监测到有重复的数据
              temArr.map((data) => {
                // 临时数组temArr去比对然后就添加到同一个子数组里
                if (data.name === param) {
                  data.arr.push(item);
                }
              });
            }
          });
        }
      });
      const parentArr = [];
      // 这里是对数据的根（即最上层的）做处理
      organArr.map((item) => {
        if (!item.Parent) {
          // 如果没有父id，就是根级
          const objP = {
            name: item.Name,
            id: item.Id,
            path: item.Path,
            icon: "el-icon-folder-opened",
            children: [],
          };
          // 添加到临时数组parentArr里
          parentArr.push(objP);
        }
      });
      // 为方便后续递归，使用function
      childrenFn(parentArr);
      function childrenFn(parent) {
        // 把上面已经添加好根级的数组放进来做第一次操作
        parent.map((item) => {
          // 遍历之前的一维数组
          organArr.map((param) => {
            // 如果这个数组的父id和传进来的数组的id一样
            if (item.id === param.Parent) {
              const objC = {
                name: param.Name,
                id: param.Id,
                path: param.Path,
                icon: "el-icon-folder-opened",
                children: [],
              };
              // 在这个传进来的数组里的children里添加对应数据
              item.children.push(objC);
            }
          });
          // 这是使用了临时数组temArr，要做合并处理
          temArr.map((data) => {
            // 判断设置好的name和传进来的数组的id是不是一样
            if (item.id === data.name) {
              data.arr.map((tem) => {
                const objP = {
                  name: tem.Name,
                  icon: "el-icon-s-platform",
                  id: tem.Id,
                  detail: tem,
                };
                // 一样的话就构造数据，添加进传进数组的children里去
                item.children.push(objP);
              });
            }
          });
          // 判断有没有children，因为设备是没有的
          if (item.children) {
            // 做一个递归处理，循环这个操作，直到没有children层级
            childrenFn(item.children);
          }
        });
      }
      // 把最终合成数据赋值给树的组件，组件展示数据
      this.treeData = parentArr;
    },
  },
};
</script>