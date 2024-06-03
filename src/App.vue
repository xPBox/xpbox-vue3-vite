<script setup lang="ts">
import { Runtime } from '@xpbox/sdk'
import { onMounted, ref } from 'vue'

const runtime = new Runtime()

const showJson = ref({})
let isReady = false

onMounted(() => {
  const ready = () => {
    if (!isReady) {
      isReady = true
      // 程序准备好了才显示，如果没有调用这个命令，页面上的组件不会被显示
      runtime.app.ready()
    }
  }

  // 获取配置信息
  runtime.config.get().then(({ data, isReady }) => {
    if (isReady) {
      showJson.value = data
      ready()
    }
  })
  // 应用加载的时候可能还没有拉到最新的配置，也就是说上面的get不一定能拉到真正的配置，
  // 所以需要监听配置更新事件，当xPBox拉取到配置信息的时候会下发通知。这个通知不仅是程
  // 序拉取到最新的配置，也可能是当前App或者其他App修改了配置也会触发更新通知。
  runtime.config.onUpdate(({ data }) => {
    showJson.value = data
    ready()
  })
})

const onSetExtraData = () => {
  // 使用update只会更新提供的字段
  runtime.config.update({ extra: 1 })
}

const onSetRandomData = () => {
  // 使用replace会替换掉整个配置
  runtime.config.replace({ random: Math.random() })
}
</script>

<template>
  <div class="main">
    <div class="title">
      Hello xPBox
    </div>

    <div class="json">
      {{JSON.stringify(showJson, undefined, 4)}}
    </div>

    <button class="btn" @click="onSetRandomData">
      设置随机数据
    </button>

    <button class="btn" @click="onSetExtraData">
      更新其他数据
    </button>
  </div>
</template>

<style scoped>
.main {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  overflow: hidden;
}

.title {
  font-size: 26px;
  font-weight: 500;
  color: #333;
}

.json {
  margin-top: 12px;
  padding: 12px 24px;
  border-radius: 12px;
  background: #cccccc80;
  word-break: break-word;
  white-space: pre-line;
}

.btn {
  margin-top: 12px;
}
</style>
