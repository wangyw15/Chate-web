<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { BrowserMultiFormatReader } from '@zxing/library'

const reader = new BrowserMultiFormatReader()
const cameras = ref<MediaDeviceInfo[]>([])
const selectedCamera = ref('')
const cameraView = ref<HTMLVideoElement>()
const jan = ref('')

onMounted(async () => {
    cameras.value = await reader.listVideoInputDevices()
})

function checkJAN(jan: string): boolean {
    if (jan.length !== 13) {
        return false
    }
    let sum = 0
    for (let i = 0; i < 12; i++) {
        let n = parseInt(jan[i])
        if (i % 2 === 0) {
            sum += n
        } else {
            sum += n * 3
        }
    }
    let check = (10 - (sum % 10)) % 10
    return check === parseInt(jan[12])
}

async function scan() {
    if (cameras.value.length === 0) {
        alert('没有摄像头')
        return
    } else if (cameras.value.length === 1) {
        selectedCamera.value = cameras.value[0].deviceId
    }

    if (selectedCamera.value === '') {
        alert('请选择摄像头')
        return
    }
    if (cameraView.value === undefined) {
        return
    }
    let result = await reader.decodeFromVideoDevice(selectedCamera.value, cameraView.value, (result, error) => {
        if (result) {
            if (checkJAN(result.getText())) {
                jan.value = result.getText()
                reader.reset()
            }
        }
    })
}
</script>

<style>
.container>div {
    margin: 0 0 1rem 0;
}
</style>

<template>
    <ClientOnly>
        <div class="container">
            <div>
                <video ref="cameraView" width="300" height="200" style="border: 1px solid gray; "></video>
            </div>
            <div>
                <span v-if="cameras.length === 0" >此设备没有摄像头</span>
                <span v-else-if="cameras.length === 1">使用默认摄像头</span>
                <el-select v-else v-model="selectedCamera">
                    <el-option v-for="camera in cameras"
                               :key="camera.deviceId"
                               :label="camera.label"
                               :value="camera.deviceId"
                               />
                </el-select>
            </div>
            <div>
                <el-button
                    type="primary"
                    @click="scan"
                    :disabled="selectedCamera === ''"
                    >
                    扫码输入JAN
                </el-button>
            </div>
            <div>
                <el-input v-model="jan" placeholder="JAN"/>
            </div>
        </div>
    </ClientOnly>
</template>
