<template>
  <el-container class="app-view">
    <el-aside style="width: auto;">
      <el-menu
          style="min-height: 100%; padding-top: 2rem"
          default-active="1"
          collapse
      >
        <el-sub-menu index="1">
          <template #title>
            <el-icon><PictureFilled /></el-icon>
            <span>Загрузить изображение</span>
          </template>
          <el-menu-item-group title="Загрузить изображение">
            <el-menu-item index="1-1" @click="uploadLocalImage">Из локального хранилища</el-menu-item>
            <el-menu-item index="1-2" @click="uploadUrlImage">Через URL адрес</el-menu-item>
          </el-menu-item-group>
        </el-sub-menu>
        <el-menu-item index="2" @click="activatePositioning">
          <el-icon><Position /></el-icon>
          <span>Курсор</span>
        </el-menu-item>
        <el-menu-item index="3" @click="doSomethingElse">
          <el-icon><setting /></el-icon>
          <span>Настройки</span>
        </el-menu-item>
      </el-menu>
    </el-aside>
    <el-main style="padding: 0; background-color: var(--el-bg-color-overlay); position: relative;">
      <canvas id="canvas" style="width: 100%; height: 100%; display: block;"></canvas>
    </el-main>
    <el-aside style="border-left: 1px solid var(--el-menu-border-color); padding: 1rem; ">
      <el-form v-if="showPosition">
        <el-form-item v-if="imgWidth && imgHeight">
          <el-text size="large" style="padding-bottom: 1rem;">Размер изображения</el-text>
          <el-input readonly
                    :model-value="imgWidth"
                    placeholder="Нет данных"
          >
            <template style="width: 1rem" #prepend>X</template>
          </el-input>
          <el-input readonly
                    :model-value="imgHeight"
                    placeholder="Нет данных"
          >
            <template style="width: 1rem" #prepend>Y</template>
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-text size="large" style="padding-bottom: 1rem;">Позиция курсора</el-text>
          <el-input readonly
              :model-value="cursorPosition.x >= 0? cursorPosition.x + ' px': undefined"
              placeholder="Нет данных"
          >
            <template style="width: 1rem" #prepend>X</template>
          </el-input>
          <el-input readonly
                    :model-value="cursorPosition.y >= 0? cursorPosition.y + ' px': undefined"
              placeholder="Нет данных"
          >
            <template style="width: 1rem" #prepend>Y</template>
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-text size="large" style="padding-bottom: 1rem;">Цвет пикселя</el-text>

          <div class="color-preview" :style="{backgroundColor: `rgba(${pixelColor.r},${pixelColor.g},${pixelColor.b},${pixelColor.a})`}"></div>
          <el-input readonly
                    :model-value="pixelColor.r || cursorPosition.x >= 0 || cursorPosition.y >= 0? pixelColor.r: undefined"
                    placeholder="Нет данных"
          >
            <template style="width: 1rem" #prepend>R</template>
          </el-input>
          <el-input readonly
                    :model-value="pixelColor.g || cursorPosition.x >= 0 || cursorPosition.y >= 0? pixelColor.g: undefined"
                    placeholder="Нет данных"
          >
            <template style="width: 1rem" #prepend>G</template>
          </el-input>
          <el-input readonly
                    :model-value="pixelColor.b || cursorPosition.x >= 0 || cursorPosition.y >= 0? pixelColor.b: undefined"
                    placeholder="Нет данных"
          >
            <template style="width: 1rem" #prepend>B</template>
          </el-input>
        </el-form-item>
      </el-form>
      <el-result v-if="!showPosition" icon="info" title="Нет доступных действий">
        <template #sub-title>
          <p>Сперва выберите инструмент в боковой панели</p>
        </template>
      </el-result>
    </el-aside>
  </el-container>
</template>

<script lang="ts" setup>
import { onMounted, onUnmounted, ref } from 'vue';

const imgRef = ref<HTMLImageElement | null>(null);
const scale = ref(1);
const imgOffsetX = ref(0);  // Смещение изображения по X
const imgOffsetY = ref(0);  // Смещение изображения по Y
const imgRenderedWidth = ref(0);
const imgRenderedHeight = ref(0);
const imgWidth = ref(0);
const imgHeight = ref(0)
const showPosition = ref(false);
const cursorPosition = ref({ x: -1, y: -1 });
const pixelColor = ref({r: 0, g: 0, b: 0, a: 0})

const uploadLocalImage = () => {
  const input = document.createElement('input');
  input.type = 'file';
  input.accept = 'image/*';
  input.onchange = (event: Event) => {
    const file = (event.target as HTMLInputElement).files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        const img = new Image();
        img.src = e.target?.result as string;
        img.onload = () => {
          imgRef.value = img;
          drawImage();
        };
      };
      reader.readAsDataURL(file);
    }
  };
  input.click();
};

const uploadUrlImage = () => {
  const url = prompt('Введите URL изображения:');
  if (url) {
    const img = new Image();
    img.crossOrigin = '';
    img.src = url;
    img.onload = () => {
      imgRef.value = img;
      drawImage();
    };
    img.onerror = () => {
      alert('Не удалось загрузить изображение. Проверьте URL.');
    };
  }
};

const drawImage = () => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  const ctx = canvas.getContext('2d');
  const img = imgRef.value;
  if (!ctx || !img) return;

  const canvasWidth = canvas.width;
  const canvasHeight = canvas.height;
  const imageAspectRatio = img.width / img.height;
  imgWidth.value = img.width;
  imgHeight.value = img.height;
  const canvasAspectRatio = canvasWidth / canvasHeight;

  if (imageAspectRatio > canvasAspectRatio) {
    imgRenderedWidth.value = canvasWidth * scale.value;
    imgRenderedHeight.value = imgRenderedWidth.value / imageAspectRatio;
  } else {
    imgRenderedHeight.value = canvasHeight * scale.value;
    imgRenderedWidth.value = imgRenderedHeight.value * imageAspectRatio;
  }

  imgOffsetX.value = (canvasWidth - imgRenderedWidth.value) / 2;
  imgOffsetY.value = (canvasHeight - imgRenderedHeight.value) / 2;

  ctx.clearRect(0, 0, canvasWidth, canvasHeight);
  ctx.drawImage(
      img,
      imgOffsetX.value,
      imgOffsetY.value,
      imgRenderedWidth.value,
      imgRenderedHeight.value
  );
};

// Обработчик изменения размера окна
const resizeCanvas = () => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  canvas.width = canvas.clientWidth;
  canvas.height = canvas.clientHeight;
  drawImage();
};

// Обработчик колёсика мыши для изменения масштаба
const handleWheel = (event: WheelEvent) => {
  event.preventDefault();
  scale.value += event.deltaY * -0.001;
  scale.value = Math.min(Math.max(.125, scale.value), 4); // Ограничиваем масштаб от 0.125 до 4
  drawImage();
};

// Активация режима отображения позиции курсора и цвета пикселя
const activatePositioning = () => {
  showPosition.value = !showPosition.value;
};

// Обработчик движения мыши
const handleMouseMove = (event: MouseEvent) => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  const ctx = canvas.getContext('2d');
  if (!ctx || !imgRef.value) return;

  const rect = canvas.getBoundingClientRect();

  // Координаты курсора относительно canvas
  const canvasX = event.clientX - rect.left;
  const canvasY = event.clientY - rect.top;

  // Проверка, попадает ли курсор в область изображения
  if (
      canvasX >= imgOffsetX.value &&
      canvasX <= imgOffsetX.value + imgRenderedWidth.value &&
      canvasY >= imgOffsetY.value &&
      canvasY <= imgOffsetY.value + imgRenderedHeight.value
  ) {
    // Координаты курсора относительно изображения
    const x = Math.floor((canvasX - imgOffsetX.value) * (imgRef.value.width / imgRenderedWidth.value));
    const y = Math.floor((canvasY - imgOffsetY.value) * (imgRef.value.height / imgRenderedHeight.value));

    cursorPosition.value = { x, y };

    // Получение цвета пикселя из canvas на основе преобразованных координат
    const pixelData = ctx.getImageData(canvasX, canvasY, 1, 1).data;
    //pixelColor.value = `rgba(${pixelData[0]}, ${pixelData[1]}, ${pixelData[2]}, ${pixelData[3] / 255})`;
    pixelColor.value = {r: pixelData[0], g: pixelData[1], b: pixelData[2], a: pixelData[3] / 255}
  } else {
    cursorPosition.value = { x: -1, y: -1 };
    pixelColor.value = {r: 0, g: 0, b: 0, a: 1}
  }
};

onMounted(() => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  window.addEventListener('resize', resizeCanvas);
  canvas.addEventListener('wheel', handleWheel);
  canvas.addEventListener('mousemove', handleMouseMove);
  resizeCanvas();
});

onUnmounted(() => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  window.removeEventListener('resize', resizeCanvas);
  canvas.removeEventListener('wheel', handleWheel);
  canvas.removeEventListener('mousemove', handleMouseMove);
});

const doSomethingElse = () => {
  console.log('Другое действие выполнено');
};

</script>

<style scoped>
.color-preview {
  border-radius: var(--el-border-radius-base) var(--el-border-radius-base) 0 0;
  box-shadow: 1px 0 0 0 var(--el-border-color) inset, 0 1px 0 0 var(--el-border-color) inset, 0 -1px 0 0 var(--el-border-color) inset,  -1px 0 0 0 var(--el-border-color) inset;
  height: 32px;
  width: 100%;
  background-color: rgba(0, 0, 0, 1);
}
</style>
