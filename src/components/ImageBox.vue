<template>
	<div
		class="w-full bg-rose-100 rounded-md shadow-sm overflow-hidden relative"
	>
		<loading-overlay :loading="loading" />

		<el-image
			class="block w-full h-40 lg:h-60"
			:src="src"
			fit="cover"
			hide-on-click-modal
      :zoom-rate="1.2"
      :max-scale="7"
      :min-scale="0.2"
      :preview-src-list="srcList"
      :initial-index="4"
			@load="loading = false"
		/>
		<div class="w-full absolute left-0 bottom-0 bg-slate-800/70 backdrop-blur-sm">
			<div class="p-2">
				<div class="w-full flex items-center text-white">
					<div class="flex-1 w-full truncate">
						<el-tooltip :content="name" placement="top-start">
							{{ name }}
						</el-tooltip>
					</div>
					<div
						v-if="mode === 'converted'"
						class="w-6 h-6 flex items-center justify-center cursor-pointer"
						@click="emit('delete')"
					>
						<font-awesome-icon :icon="faTimesCircle" />
					</div>
				</div>
				<span class="text-xs text-gray-300 flex items-center">
					{{ formatBytes(size) }}
					<el-divider v-if="uploadedAt" direction="vertical" />
					<span v-if="uploadedAt">
						{{ new Date(uploadedAt).toLocaleDateString() }}
					</span>
				</span>
			</div>
			<div v-if="mode === 'uploaded'">
				<el-divider class="m-0" />
				<div class="w-full flex text-white h-9 text-center text-sm">
					<el-tooltip :content="copyUrl" placement="top-start">
            <el-popover
                placement="bottom"
                trigger="click">
              <template #reference>
                <el-button slot="reference" class="flex-1 flex items-center justify-center cursor-pointer">链接</el-button>
              </template>
              <p>选择链接格式</p>
              <el-radio-group v-model="urlType">
                <el-radio label="Markdown">Markdown</el-radio>
                <el-radio label="HTML">HTML</el-radio>
                <el-radio label="BBCode">BBCode</el-radio>
                <el-radio label="LINK">LINK</el-radio>
              </el-radio-group>
              <div
                  class="flex-1 flex items-center justify-center cursor-pointer"
                  @click="copyLink(copyUrl)"
              >
                <font-awesome-icon :icon="faCopy" class="mr-2" />
                拷贝
              </div>
            </el-popover>
					</el-tooltip>
					<el-divider direction="vertical" class="h-full" />
					<el-popconfirm
						title="确认删除图片吗？"
						confirm-button-type="danger"
						@confirm="
							() => {
								// (e: Event) => boolean ???
								loading = true
								emit('delete')
								return true
							}
						"
					>
						<template #reference>
							<div class="flex-1 flex items-center justify-center cursor-pointer">
								<font-awesome-icon :icon="faTrashAlt" class="mr-2" />
								删除
							</div>
						</template>
					</el-popconfirm>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup lang="ts">
import { faTimesCircle, faTrashAlt, faCopy } from '@fortawesome/free-regular-svg-icons'
import copy from 'copy-to-clipboard'
import formatBytes from '../utils/format-bytes'
import {ElTooltip, ElDivider, ElPopconfirm, ElImage, ElMessage, ElPopover, ElRadioGroup, ElRadio} from "element-plus";
import { ref } from 'vue'
import LoadingOverlay from '../components/LoadingOverlay.vue'

const props = defineProps<{
	src: string
	copyUrl:string
	name: string
	size: number
	mode: 'converted' | 'uploaded'
	uploadedAt?: number
	expiresAt?: number
}>()
const emit = defineEmits(['delete'])

const imageError = ref(false)
const loading = ref(true)
const urlType = ref('Markdown')
const srcList = ref([props.src])

const markdownLink = (link: String, filename: String) => {
  return `![${filename}](${link})`
}
const htmlLink = (link: String, filename: String) => {
  return `<a href="${link}" target="_blank" title="${filename}"><img src="${link}"></a>`
}
const bbcodeLink = (link: String, filename: String) => {
  return `[img]${link}[/img]`
}
const copyLink = (link : string) => {
  let file = link.split('/')
  if (urlType.value == 'Markdown') {
    link = markdownLink(link, file[file.length - 1])
  }
  if (urlType.value == 'HTML') {
    link = htmlLink(link, file[file.length - 1])
  }
  if (urlType.value == 'BBCode') {
    link = bbcodeLink(link, file[file.length - 1])
  }
  const res = copy(link)
  if (res) {
    ElMessage.success('链接复制成功')
  } else {
    ElMessage.success('链接复制失败')
  }
}
</script>
