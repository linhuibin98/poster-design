<!--
 * @Author: ShawnPhang
 * @Date: 2021-12-16 16:20:16
 * @Description: 瀑布流组件
 * @LastEditors: ShawnPhang <https://m.palxp.cn>
 * @LastEditTime: 2023-12-11 11:45:24
-->
<template>
  <div ref="imgWaterFall" :style="{ height: countHeight + 'px' }" class="img-water-fall">
    <!-- backgroundImage: `url(${item.cover})` -->
    <div v-for="(item, i) in list" :key="i + 'iwf'" :style="{ top: item.top + 'px', left: item.left + 'px', width: width + 'px', height: item.height + 'px' }" class="img-box" @click.stop="selectItem(item, i)">
      <edit-model v-if="edit" :options="edit" :data="{ item, i }">
        {{ item.isDelect }}
        <div v-if="item.isDelect" class="list__mask">已删除</div>
        <el-image v-if="!item.fail" class="img" :src="item.cover" lazy loading="lazy" @error="loadError(item)" />
        <div v-else class="fail_img">{{ item.title }}</div>
      </edit-model>
      <el-image v-else class="img" :src="item.cover" lazy loading="lazy" @error="loadError(item)" />
    </div>
  </div>
</template>

<script>
const NAME = 'img-water-fall'
import { defineComponent, toRefs, reactive, watch } from 'vue'

export default defineComponent({
  name: NAME,
  props: {
    listData: {
      type: Array,
      required: true,
    },
    edit: {}
  },
  emits: ['select', 'load'],
  setup(props, { emit }) {
    const state = reactive({
      width: 146, // 图片的宽度
      list: [],
      countHeight: 0,
    })

    const columnHeights = [] // 列的高度
    const columnNums = 2 // 总共有多少列
    const gap = 7 // 图片之间的间隔

    watch(
      () => props.listData,
      () => {
        columnHeights.length = 0
        const widthLimit = state.width * columnNums //  + gap * (columnNums - 1) // 每行宽度
        const cloneList = JSON.parse(JSON.stringify(props.listData))
        for (let i = 0; i < cloneList.length; i++) {
          let index = i % columnNums
          const item = cloneList[i]
          item.height = (item.height / item.width) * state.width // 图片高度
          item.left = index * (widthLimit / columnNums + gap) // 定位
          item.top = columnHeights[index] + gap || 0 // 定位
          // columnHeights[index] = isNaN(columnHeights[index]) ? item.height : item.height + columnHeights[index] + gap // 记录列高度
          // 找出最短边
          if (isNaN(columnHeights[index])) {
            columnHeights[index] = item.height
          } else {
            index = columnHeights.indexOf(Math.min(...columnHeights))
            item.left = index * (widthLimit / columnNums + gap)
            item.top = columnHeights[index] + gap || 0
            columnHeights[index] = item.height + columnHeights[index] + gap
          }
        }
        state.countHeight = Math.max(...columnHeights)
        state.list = cloneList
      },
    )

    const load = () => {
      emit('load')
    }
    const selectItem = (value, index) => {
      emit('select', value)
    }
    const loadError = (item) => {
      item.fail = true
    }

    return {
      ...toRefs(state),
      load,
      selectItem,
      loadError,
    }
  },
})
</script>

<style lang="less" scoped>
.fail_img {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #999999;
}
.img-water-fall {
  position: relative;
  margin-left: 14px;
  .img-box {
    position: absolute !important;
    cursor: pointer;
    position: relative;
    background-size: cover;
    border-radius: 5px;
    border: 1px solid #e0e5ea;
    overflow: hidden;
    .img {
      display: block;
      width: 100%;
      height: 100%;
    }
  }
  .img-box:hover::before {
    content: ' ';
    background: rgba(0, 0, 0, 0.15);
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    z-index: 1;
    pointer-events: none;
  }
}
.list {
  &__mask {
    position: absolute;
    z-index: 2;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    color: #ffffff;
    display: flex;
    align-items: center;
    justify-content: center;
  }
}
</style>
