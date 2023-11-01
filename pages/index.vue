<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { BrowserMultiFormatReader } from '@zxing/library'

const devAPI = 'https://localhost:8080'
const prodAPI = ''
const api = import.meta.env.DEV ? devAPI : prodAPI

const reader = new BrowserMultiFormatReader()
const cameras = ref<MediaDeviceInfo[]>([])
const selectedCamera = ref('')
const cameraView = ref<HTMLVideoElement>()
const jan = ref('')
const image = ref('')

onMounted(async () => {
    cameras.value = await reader.listVideoInputDevices()
    if (cameras.value.length > 0) {
        selectedCamera.value = cameras.value[0].deviceId
    }
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

function showImage() {
    if (!checkJAN(jan.value)) {
        return
    }
    image.value = `${api}/api/image/${jan.value}`
}

async function scan() {
    if (selectedCamera.value === '') {
        alert('请选择摄像头')
        return
    }
    
    if (cameraView.value === undefined) {
        return
    }
    await reader.decodeFromVideoDevice(selectedCamera.value, cameraView.value, (result, error) => {
        if (result) {
            if (checkJAN(result.getText())) {
                jan.value = result.getText()
                reader.reset()
                showImage();
            }
        }
    })
}
</script>

<style>
.container>div {
    margin: 0 0 1rem 0;
}

video {
    width: 100%;
    height: 200px;
    border: 1px solid gray;
}

.button-container {
    display: flex;
    flex-direction: row;
    justify-content: space-around;
}

.button-container>button {
    width: 45%;
}

.image-slot {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    background: var(--el-fill-color-light);
    color: var(--el-text-color-secondary);
    font-size: 14px;
}

.dot {
    animation: dot 2s infinite steps(3, start);
    overflow: hidden;
}
</style>

<template>
    <ClientOnly>
        <div class="container">
            <div>
                <video ref="cameraView"></video>
            </div>
            <div>
                <span v-if="cameras.length === 0" >此设备没有摄像头</span>
                <span v-else-if="cameras.length === 1">使用默认摄像头</span>
                <el-select v-else v-model="selectedCamera" style="width: 100%;" placeholder="选择摄像头">
                    <el-option v-for="camera in cameras"
                               :key="camera.deviceId"
                               :label="camera.label"
                               :value="camera.deviceId"
                               />
                </el-select>
            </div>
            <div>
                <el-input v-model="jan" placeholder="JAN"/>
            </div>
            <div class="button-container">
                <el-button
                    type="primary"
                    @click="scan"
                    :disabled="selectedCamera === ''"
                    >
                    扫码输入JAN
                </el-button>
                <el-button
                    type="primary"
                    @click="showImage"
                    :disabled="!checkJAN(jan)"
                    >
                    <span v-if="checkJAN(jan)">查询图片</span>
                    <span v-else>检查JAN是否正确</span>
                </el-button>
            </div>
            <div>
                <el-image :src="image">
                    <template #placeholder>
                        <div class="image-slot">Loading<span class="dot">...</span></div>
                    </template>
                    <template #error>
                        <div class="image-slot">此处应有图片</div>
                    </template>
                </el-image>
            </div>
        </div>
    </ClientOnly>
</template>
