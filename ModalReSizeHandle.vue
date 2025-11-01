<template>
  <AButton class="fullScreenIcon" type="text" size="small" @click="switchWindowStatus" ref="resizeHandleRef">
    <Icon style="color: rgb(140,140,140)" name="icon-zuidahua" v-if="!isMaxSize"/>
    <Icon style="color: rgb(140,140,140)" name="icon-icon-repeat" v-if="isMaxSize"/>
  </AButton>
  <Icon name="icon-resizebottomright" class="resize-handle" @mousedown="onResizeMouseDown"
  ></Icon>
</template>

<script setup lang="ts">
/**
 * 使用方式：
 * const modalStyle = ref({
 *   width: size.value.width,
 *   bodyHeight: size.value.height,//
 *   paddingBottom: 0,
 *   maxWidth: '100%'
 * });
 * <AModal :bodyStyle="{height: modalStyle.bodyHeight}"
 *         :style="modalStyle"
 *         wrap-class-name="resizeModal"
 *     >
 *       <ModalReSizeHandle v-model:style="modalStyle"></ModalReSizeHandle>
 *     </AModal>
 */
import _ from "lodash";

const props = defineProps<{
  style: any | undefined,
}>();

const emit = defineEmits<{
  (e: 'update:style', value: any): void
}>()

const isMaxSize = ref()
const resizeHandleRef = ref();
const formHeight = 116;//表单框的高度

function ini() {
  addMoveEvent();

}

function addMoveEvent() {
  const headerEl = resizeHandleRef.value.$el.parentElement.parentElement.getElementsByClassName('ant-modal-header')[0];
  headerEl.addEventListener('mousedown', onMoveMouseDown)
  headerEl.addEventListener('mouseup', onMoveUp)
  headerEl.addEventListener('dblclick', onMoveDbClick)
}

function getModalRect(): DOMRect {
  //ant-modal
  return resizeHandleRef.value.$el.parentElement.parentElement.parentElement.parentElement.getBoundingClientRect();
}

function getMaxRect(): DOMRect {
  //ant-modal-wrap
  return resizeHandleRef.value.$el.parentElement.parentElement.parentElement.parentElement?.parentElement.getBoundingClientRect();
}

onMounted(() => {
  ini()
})
let normalLeft = 0;
let normalTop = 0;
let normalWidth = 0;
let normalHeight = 0;

function switchWindowStatus() {
  if (isMaxSize.value) {
    const s = {
      top: normalTop + 'px',
      marginLeft: normalLeft + 'px',
      width: normalWidth + 'px',
      bodyHeight: normalHeight - formHeight + 'px',
    }
    emit('update:style', {...props.style, ...s})

  } else {
    const rect = getModalRect();
    normalLeft = rect.left;
    normalTop = rect.top;
    normalWidth = rect.width;
    normalHeight = rect.height;

    const s = {
      top: 0,
      marginLeft: 0,
      marginRight: 0,
      maxWidth: '100%',
      width: '100%',
      bodyHeight: `calc(100vh - ${formHeight}px)`,
    }

    emit('update:style', {...props.style, ...s})

  }
  isMaxSize.value = !isMaxSize.value

}


let startX = 0
let startY = 0
let startWidth = 0
let startTop = 0
let startLeft = 0
let startHeight = 0

let downFlag = 0;

function onMoveUp() {
  downFlag = 0;
}

function onMoveDbClick() {
  switchWindowStatus();
}

/** 鼠标按下时开始监听拖拽 */
const onMoveMouseDown = (e: MouseEvent): void => {
  e.preventDefault()

  downFlag = _.random(1, 9999);
  const tmpVal = downFlag;
  // 设置一个延迟，如果在此期间未发生 mouseup，则认为是拖动
  setTimeout(() => {
    if (tmpVal != downFlag) return;
    startMoveWindow(e)
  }, 100); // 延迟时间可以根据需要调整
}

function startMoveWindow(e: MouseEvent) {
  startX = e.clientX
  startY = e.clientY
  const rect = getModalRect();
  startTop = rect.top;
  startLeft = rect.left;
  document.addEventListener('mousemove', onMove)
  document.addEventListener('mouseup', onMoveEnd)
}

const onMove = (e: MouseEvent): void => {
  const maxRect = getMaxRect();


  if (isMaxSize.value) {
    const maxTop = maxRect.height - normalHeight;
    const maxLeft = maxRect.width - normalWidth;
    const top = Math.max(Math.min(e.clientY - 15, maxTop), 0);
    const left = Math.max(Math.min(e.clientX - normalWidth / 2, maxLeft), 0);
    startTop = top;
    startLeft = left;
    const s = {
      top: top + 'px',
      marginLeft: left + 'px',
      width: normalWidth + 'px',
      bodyHeight: normalHeight - formHeight + 'px',
    }
    isMaxSize.value = !isMaxSize.value;
    emit('update:style', {...props.style, ...s})
    console.log('update:style', top, left);

    return;
  }

  if (e.buttons !== 1)//左键已经释放了，不要响应移动
    onMoveEnd()

  const deltaX = (e.clientX - startX)
  const deltaY = (e.clientY - startY)
  const rect = getModalRect();
  const maxTop = maxRect.height - rect.height;
  const maxLeft = maxRect.width - rect.width;
  const s = {
    top: Math.min(maxTop, Math.max(startTop + deltaY, 0)) + 'px',
    marginLeft: Math.min(maxLeft, Math.max(startLeft + deltaX, 0)) + 'px',//得设置这个，因为默认用的是左右对齐
  }
  emit('update:style', {...props.style, ...s})

}
const onMoveEnd = (): void => {
  document.removeEventListener('mousemove', onMove)
  document.removeEventListener('mouseup', onMoveEnd)
}


const onResizeMouseDown = (e: MouseEvent): void => {
  e.preventDefault()
  startX = e.clientX
  startY = e.clientY

  // 假设 size.value = { width: '600px', height: '400px' }
  const rect = getModalRect();
  startWidth = rect.width;
  startHeight = rect.height;

  document.addEventListener('mousemove', onResizing)
  document.addEventListener('mouseup', onResizeEnd)
}

/** 拖拽中更新宽度和高度 */
const onResizing = (e: MouseEvent): void => {
  let deltaX = (e.clientX - startX) * 2
  const deltaY = (e.clientY - startY)
  if (props.style?.marginLeft) {//手动设置了marginLeft则不会左右居中了，那横向尺寸不需要*2了
    deltaX = deltaX / 2;
  }
  const newWidth = startWidth + deltaX + 'px'
  const newBodyHeight = startHeight + deltaY - formHeight + 'px'

  const s = {
    width: newWidth,
    bodyHeight: newBodyHeight,
  }

  emit('update:style', {...props.style, ...s})


}

/** 拖拽结束时移除监听 */
const onResizeEnd = (): void => {
  document.removeEventListener('mousemove', onResizing)
  document.removeEventListener('mouseup', onResizeEnd)
}


</script>

<style scoped lang="scss">
.fullScreenIcon {
  font-size: 15px;
  position: absolute;
  top: 15.5px;
  right: 45px;
  padding: 0 2px;
}

.resize-handle {
  font-size: 13px;
  position: absolute;
  right: 0;
  bottom: 0;
  color: black;
  padding: 4px;
  cursor: se-resize;
}

:global(.resizeModal .ant-modal-content) {
  padding-top: 0 !important;

}

:global(.resizeModal .ant-modal-header) {
  padding-top: 20px;

}

</style>
