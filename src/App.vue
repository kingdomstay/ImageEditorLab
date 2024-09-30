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
        <el-menu-item index="3" @click="activateZooming">
          <el-icon><zoomIn /></el-icon>
          <span>Масштабирование</span>
        </el-menu-item>
        <el-menu-item index="4" @click="activateResizing">
          <el-icon><crop /></el-icon>
          <span>Изменение изображения</span>
        </el-menu-item>
        <el-menu-item index="5" @click="activateSaving">
          <el-icon><download /></el-icon>
          <span>Сохранение</span>
        </el-menu-item>
        <el-menu-item index="6" @click="activateMoving">
          <el-icon><rank /></el-icon>
          <span>Перемещение</span>
        </el-menu-item>
        <el-menu-item index="7" @click="activatePicking">
          <el-icon><stamp /></el-icon>
          <span>Пипетка</span>
        </el-menu-item>
      </el-menu>
    </el-aside>
    <el-main style="padding: 0; background-color: var(--el-bg-color-overlay); position: relative;">
      <canvas id="canvas" style="width: 100%; height: 100%; display: block;"></canvas>
    </el-main>
    <el-aside style="border-left: 1px solid var(--el-menu-border-color); padding: 1rem; ">
      <el-form v-if="selectedSection === 'position'">
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
      <el-form v-if="selectedSection === 'zoom'">
        <el-form-item>
          <el-text size="large" style="padding-bottom: 1rem;">Масштаб изображения</el-text>
          <el-select v-model="scalePercentage" @change="updateScale">
            <el-option v-for="scale in scaleOptions" :key="scale" :label="`${scale}`" :value="scale"></el-option>
          </el-select>
          <el-slider style="margin: auto 10px;" v-model="scalePercentage" @change="updateScale" :min="12" :max="300" :step="1" />
        </el-form-item>
      </el-form>
      <el-form v-if="selectedSection === 'resize'" label-position="top">
    <el-form-item>
      <el-text size="large" style="padding-bottom: 1rem;">Изменение размера изображения</el-text>
    </el-form-item>
    
    <el-form-item>
      <el-text>Общее количество пикселей: {{ originalPixels }} MP (до) / {{ newPixels }} MP (после)</el-text>
    </el-form-item>

    <el-form-item label="Режим изменения размера">
      <el-select v-model="resizeMode" @change="updateFields">
        <el-option label="Процент" value="percent"></el-option>
        <el-option label="Пиксели" value="pixels"></el-option>
      </el-select>
    </el-form-item>

    <el-form-item label="Новая ширина">
      <el-input v-model="newWidth" :placeholder="resizeMode === 'percent' ? 'Процент' : 'Ширина (пиксели)'" />
    </el-form-item>
    
    <el-form-item label="Новая высота">
      <el-input v-model="newHeight" :placeholder="resizeMode === 'percent' ? 'Процент' : 'Высота (пиксели)'" />
    </el-form-item>

    <el-form-item>
      <el-checkbox v-model="maintainAspectRatio" @change="toggleAspectRatio" class="aspect-ratio-checkbox">
        <span>Сохранить пропорции</span>
      </el-checkbox>
    </el-form-item>

    <el-form-item label="Алгоритм интерполяции">
      <el-select v-model="interpolationMethod">
        <el-option label="Ближайший сосед" value="nearest" />
      </el-select>
      <el-tooltip effect="dark" content="Алгоритм ближайшего соседа используется для быстрого уменьшения изображения без сложных вычислений." style="margin-left: 10px;">
        <el-button size="mini">?</el-button>
      </el-tooltip>
    </el-form-item>

    <el-form-item>
      <el-button type="primary" @click="confirmResize">Подтвердить</el-button>
    </el-form-item>
  </el-form>
      <el-form v-if="selectedSection === 'save'">
        <el-form-item>
          <el-text size="large" style="padding-bottom: 1rem;">Сохранение изображения</el-text>
          <el-button @click="saveImage">Сохранить локально</el-button>
        </el-form-item>
      </el-form>
      <el-result v-if="selectedSection === null" icon="info" title="Нет доступных действий">
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
const selectedSection = ref(null);
const cursorPosition = ref({ x: -1, y: -1 });
const pixelColor = ref({r: 0, g: 0, b: 0, a: 0})
const resizeMode = ref('percent');
const newWidth = ref(100);
const newHeight = ref(100);
const maintainAspectRatio = ref(true);
const originalPixels = ref(0);
const newPixels = ref(0);
const interpolationMethod = ref('nearest');

const toggleAspectRatio = () => {
  if (maintainAspectRatio.value) {
    newHeight.value = Math.round((newWidth.value * imgHeight.value) / imgWidth.value);
  }
};

const updateFields = () => {
  if (resizeMode.value === 'percent') {
    newWidth.value = 100;
    newHeight.value = 100;
  }
};

const confirmResize = () => {
  if (resizeMode.value === 'percent' && (newWidth.value <= 0 || newHeight.value <= 0)) {
    alert('Введите допустимое значение для ширины и высоты.');
    return;
  }

  let scaleX = resizeMode.value === 'percent' ? newWidth.value / 100 : newWidth.value / imgWidth.value;
  let scaleY = resizeMode.value === 'percent' ? newHeight.value / 100 : newHeight.value / imgHeight.value;

  if (maintainAspectRatio.value) {
    const aspectRatio = imgWidth.value / imgHeight.value;
    if (scaleX > scaleY) {
      scaleX = scaleY;
      newWidth.value = Math.round(imgWidth.value * scaleX);
    } else {
      scaleY = scaleX;
      newHeight.value = Math.round(imgHeight.value * scaleY);
    }
  }

  const canvas = document.getElementById('canvas');
  const ctx = canvas.getContext('2d');
  const srcImageData = ctx.getImageData(imgOffsetX.value, imgOffsetY.value, imgRenderedWidth.value, imgRenderedHeight.value);

  const resizedImageData = resizeNearestNeighbor(srcImageData, newWidth.value, newHeight.value);

  const newCanvas = document.createElement('canvas');
  const newCtx = newCanvas.getContext('2d');
  newCanvas.width = newWidth.value;
  newCanvas.height = newHeight.value;
  newCtx.putImageData(resizedImageData, 0, 0);

  imgRef.value = new Image();
  imgRef.value.src = newCanvas.toDataURL();
  imgRef.value.onload = () => {
    drawImage();
  };

  newPixels.value = (newWidth.value * newHeight.value) / 1000000;
};

const resizeNearestNeighbor = (srcImageData, width, height) => {
  const srcPixels = srcImageData.data;
  const srcWidth = srcImageData.width;
  const srcHeight = srcImageData.height;
  const dstImageData = new ImageData(width, height);
  const dstPixels = dstImageData.data;

  const xFactor = srcWidth / width;
  const yFactor = srcHeight / height;

  let dstIndex = 0;
  let srcIndex, offset;

  for (let y = 0; y < height; y++) {
    offset = ((y * yFactor) | 0) * srcWidth;

    for (let x = 0; x < width; x++) {
      srcIndex = (offset + x * xFactor) << 2;

      dstPixels[dstIndex] = srcPixels[srcIndex];
      dstPixels[dstIndex + 1] = srcPixels[srcIndex + 1];
      dstPixels[dstIndex + 2] = srcPixels[srcIndex + 2];
      dstPixels[dstIndex + 3] = srcPixels[srcIndex + 3];
      dstIndex += 4;
    }
  }

  return dstImageData;
};

// Опции для выбора масштаба
const scaleOptions = ref([12, 25, 50, 75, 100, 150, 200, 250, 300]);
const scalePercentage = ref(100); // начальный масштаб в процентах

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

  const padding = 50; // Отступ 50 пикселей

  const canvasWidth = canvas.width - padding * 2;  // Учитываем отступ по X
  const canvasHeight = canvas.height - padding * 2; // Учитываем отступ по Y
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

  // Теперь добавляем отступы по X и Y
  imgOffsetX.value = (canvasWidth - imgRenderedWidth.value) / 2 + padding;
  imgOffsetY.value = (canvasHeight - imgRenderedHeight.value) / 2 + padding;

  ctx.clearRect(0, 0, canvas.width, canvas.height);  // Очищаем canvas перед отрисовкой
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

// Активация режима отображения позиции курсора и цвета пикселя
const activatePositioning = () => {
  selectedSection.value = 'position';
};

// Активация режима отображения позиции курсора и цвета пикселя
const activateZooming = () => {
  selectedSection.value = 'zoom';
};

const activateResizing = () => {
  selectedSection.value = 'resize';
}

const activateSaving = () => {
  selectedSection.value = 'save';
}

const activateMoving = () => {
  selectedSection.value = 'move';
}

const activatePicking = () => {
  selectedSection.value = 'picker';
}

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

/////// МАСШТАБИРОВАНИЕ
// Обработчик изменения масштаба
const updateScale = () => {
  scale.value = scalePercentage.value / 100; // Обновляем значение scale
  drawImage(); // Перерисовываем изображение с новым масштабом
};

// Обработчик изменения масштаба через селектор
const updateScaleFromSelect = (value: number) => {
  scalePercentage.value = value; // Обновляем значение с селектора
  updateScale(); // Обновляем масштаб
};

// Обработчик колёсика мыши для изменения масштаба
const handleWheel = (event: WheelEvent) => {
  event.preventDefault();
  scalePercentage.value += event.deltaY * -0.1; // Обновляем значение масштаба
  scalePercentage.value = Math.min(Math.max(12, scalePercentage.value), 300); // Ограничиваем значение
  updateScale(); // Обновляем масштаб
};



/// СОХРАНЕНИЕ
const saveImage = () => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  const ctx = canvas.getContext('2d');
  if (!ctx || !imgRef.value) return;

  // Создаём новый canvas для сохранения
  const saveCanvas = document.createElement('canvas');
  const saveCtx = saveCanvas.getContext('2d');

  // Устанавливаем размеры нового canvas
  saveCanvas.width = imgRenderedWidth.value;  // Ширина сохраняемой области
  saveCanvas.height = imgRenderedHeight.value; // Высота сохраняемой области

  // Копируем область изображения на новый canvas
  saveCtx?.drawImage(
    imgRef.value,
    0,
    0,
    imgRef.value.width,
    imgRef.value.height,
    0,
    0,
    imgRenderedWidth.value,
    imgRenderedHeight.value
  );

  // Получаем данные изображения в формате PNG
  const dataURL = saveCanvas.toDataURL('image/png');

  // Создаём ссылку для скачивания
  const link = document.createElement('a');
  link.href = dataURL; // Устанавливаем ссылку на изображение
  link.download = 'image.png'; // Устанавливаем имя файла для скачивания
  link.click(); // Симулируем клик, чтобы инициировать скачивание
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