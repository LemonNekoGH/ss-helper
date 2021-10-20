<template>
  <el-container>
    <el-header class="ss-header">
      <h3>Shadowsocks Helper</h3>
    </el-header>
    <el-main>
      <h4>已编码的 ss 链接</h4>
      <div class="main-item">
        <el-input placeholder="已编码的 ss 链接" v-model="encoded">
          <template #prepend>ss://</template>
        </el-input>
        <el-button type="primary" @click="fromEncoded">转换</el-button>
      </div>
      <h4>已解码的 ss 链接</h4>
      <div class="main-item">
        <el-input placeholder="已解码的 ss 链接" v-model="decoded">
          <template #prepend>ss://</template>
        </el-input>
        <el-button type="primary">转换</el-button>
      </div>
      <h4><code>config.json</code> 内容</h4>
      <div class="main-item">
        <el-input placeholder="在此输入 config 文件内容" type="textarea" v-model="config" rows="8" resize="none"></el-input>
        <el-button type="primary">转换</el-button>
      </div>
      <div class="qr-code" v-if="qr">
        <h4>二维码</h4>
        <qrcode-vue :value="qr" :size="300"></qrcode-vue><br>
      </div>
    </el-main>
  </el-container>
</template>
<script lang="ts">
import {defineComponent, ref, toRaw} from 'vue'
import QrcodeVue from 'qrcode.vue'
import {ElMessageBox} from "element-plus";

export default defineComponent({
  components: {
    QrcodeVue
  },
  setup() {
    const encoded = ref('')
    const decoded = ref('')
    const config = ref('')
    const qr = ref('')

    // 将已解码的 ss 链接转换为 config.json
    const decodedToConfig = (decodedStr: string) => {
      const firstColon = decodedStr.indexOf(':')
      const method = decodedStr.substring(0, firstColon)
      const atIndex = decodedStr.indexOf('@')
      const password = decodedStr.substring(firstColon + 1, atIndex)
      const lastColon = decodedStr.lastIndexOf(':')
      const ip = decodedStr.substring(atIndex + 1, lastColon)
      const port = decodedStr.substring(lastColon + 1)

      const configObj =  {
        server: ip,
        server_port: parseInt(port),
        local_port: 1080,
        password,
        timeout: 600,
        method
      }

      return JSON.stringify(configObj, null, 2)
    }

    // 解码 ss 链接
    const fromEncoded = () => {
      let value = encoded.value + ''
      if (value.startsWith('ss://')) {
        value = value.substring(5)
      }
      // 检测是否为 Outline 格式
      if (value.endsWith('/?outline=1')) {
        const index = value.indexOf('@')
        const indexEnd = value.indexOf('/?outline=1')
        decoded.value = atob(value.substring(0, index)) + value.substring(index, indexEnd)
        qr.value = 'ss://' + btoa(decoded.value)
      } else {
        decoded.value = atob(value)
        qr.value = value
      }
      // 转换为 config.json
      config.value = decodedToConfig(decoded.value)
    }

    const fromDecoded = () => {
      let value = decoded.value
      if (value.startsWith('ss://')) {
        value = value.substring(5)
      }
      encoded.value = btoa(value)
      config.value = decodedToConfig(value)
    }

    const fromConfig = () => {
      try {
        const obj = JSON.parse(config.value)
      } catch (e) {
        ElMessageBox.alert('解析 config 时出错')
      }
    }

    return {
      encoded,
      decoded,
      config,
      qr,
      fromEncoded,
      fromDecoded,
      fromConfig
    }
  }
})
</script>
<style lang="less">
body {
  margin: 0;
}
.ss-header {
  border-bottom: solid var(--el-border-color-base) 1px;
}
.main-item {
  display: flex;
  column-gap: 10px;
  margin-top: 10px;
}
.qr-code canvas {
  display: block;
  margin: 0 auto;
}
</style>