<template>
  <el-container class="app-view">
    <el-aside style="width: auto;">
      <el-menu
          style="min-height: 100%; padding-top: 2rem"
          default-active="1"
          collapse
      >
        <el-tooltip effect="light" content="Отслеживание курсора на изображении" placement="right" :disabled="!helpLabelsActive">
          <el-menu-item index="2" id="startTool" @click="selectSection('position')">
            <el-icon><Position /></el-icon>
            <span>Курсор</span>
          </el-menu-item>
        </el-tooltip>
        <el-tooltip effect="light" content="Масштабирование изображения" placement="right" :disabled="!helpLabelsActive">
          <el-menu-item index="3" @click="selectSection('zoom')" :disabled="!imageUploaded">
          <el-icon><zoomIn /></el-icon>
          <span>Масштабирование</span>
        </el-menu-item>
        </el-tooltip>
        <el-tooltip effect="light" content="Изменение размера изображения" placement="right" :disabled="!helpLabelsActive">
          <el-menu-item index="4" @click="selectSection('resize')" :disabled="!imageUploaded">
            <el-icon><crop /></el-icon>
            <span>Изменение изображения</span>
          </el-menu-item>
        </el-tooltip>
        <el-tooltip effect="light" content="Сохранение изображения в хранилище" placement="right" :disabled="!helpLabelsActive">
          <el-menu-item index="5" @click="selectSection('save')" :disabled="!imageUploaded">
            <el-icon><download /></el-icon>
            <span>Сохранение</span>
          </el-menu-item>
        </el-tooltip>
        <el-tooltip effect="light" content="Инструмент руки для перемещения изображения" placement="right" :disabled="!helpLabelsActive">
          <el-menu-item index="6" @click="selectSection('move')" :disabled="!imageUploaded">
            <el-icon><rank /></el-icon>
            <span>Перемещение</span>
          </el-menu-item>
        </el-tooltip>
        <el-tooltip effect="light" content="Инструмент пипетки для получения цвета и поиска оптимальной контрастности" placement="right" :disabled="!helpLabelsActive">
          <el-menu-item index="7" @click="selectSection('picker')" :disabled="!imageUploaded">
            <el-icon><stamp /></el-icon>
            <span>Пипетка</span>
          </el-menu-item>
        </el-tooltip>
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
            <template class="minified-input-label" #prepend>X</template>
          </el-input>
          <el-input readonly
                    :model-value="imgHeight"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>Y</template>
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-text size="large" style="padding-bottom: 1rem;">Позиция курсора</el-text>
          <el-input readonly
              :model-value="cursorPosition.x >= 0? cursorPosition.x + ' px': undefined"
              placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>X</template>
          </el-input>
          <el-input readonly
                    :model-value="cursorPosition.y >= 0? cursorPosition.y + ' px': undefined"
              placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>Y</template>
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-text size="large" style="padding-bottom: 1rem;">Цвет пикселя</el-text>

          <div class="color-preview" :style="{backgroundColor: `rgba(${pixelColor.r},${pixelColor.g},${pixelColor.b},${pixelColor.a})`}"></div>
          <el-input readonly
                    :model-value="pixelColor.r || cursorPosition.x >= 0 || cursorPosition.y >= 0? pixelColor.r: undefined"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>R</template>
          </el-input>
          <el-input readonly
                    :model-value="pixelColor.g || cursorPosition.x >= 0 || cursorPosition.y >= 0? pixelColor.g: undefined"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>G</template>
          </el-input>
          <el-input readonly
                    :model-value="pixelColor.b || cursorPosition.x >= 0 || cursorPosition.y >= 0? pixelColor.b: undefined"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>B</template>
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
      <el-empty v-if="selectedSection === null" description="Для начала работы загрузи изображение">
          <el-button style="margin: .2rem auto;" type="primary" icon="folderOpened" @click="uploadLocalImage" >Из локального хранилища</el-button>
          <el-button style="margin: .2rem auto;" type="default" icon="link" @click="uploadUrlImage" >Через URL адрес</el-button>
        </el-empty>
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

const isDragging = ref(false);  // Отслеживает, когда пользователь перемещает изображение
const dragStart = ref({ x: 0, y: 0 });  // Начальная точка нажатия
const imgStartOffset = ref({ x: 0, y: 0 });  // Смещение изображения при начале перемещения

const imageUploaded = ref(0);

const helpLabelsActive = ref(0); // Модификатор для tooltip's

const movingActive = ref(false); // Модификатор для move

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
          imageUploaded.value = 1;
          document.getElementById("startTool").click();
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
      imageUploaded.value = 1;
      document.getElementById("startTool").click();
    };
    img.onerror = () => {
      alert('Не удалось загрузить изображение. Проверьте URL.');
    };
  }
};

const drawImage = (resetPosition = true) => {
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
  if (resetPosition) {
    imgOffsetX.value = (canvasWidth - imgRenderedWidth.value) / 2 + padding;
    imgOffsetY.value = (canvasHeight - imgRenderedHeight.value) / 2 + padding;
  }

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

// Активация разделов
const selectSection = (value) => {
  if (selectedSection.value === 'move') {
    const canvas = document.getElementById('canvas') as HTMLCanvasElement;
    canvas.style.cursor = null;
    drawImage();
  }
  selectedSection.value = value;
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

//// Инструмент рука

// Обработчик нажатия кнопки мыши (захват изображения)
const handleDragMouseDown = (event: MouseEvent) => {
  if (selectedSection.value === 'move' || movingActive.value) {
    const canvas = document.getElementById('canvas') as HTMLCanvasElement;

    console.log('work')
    isDragging.value = true;
    dragStart.value = { x: event.clientX, y: event.clientY };
    imgStartOffset.value = { x: imgOffsetX.value, y: imgOffsetY.value };
    canvas.style.cursor = "grabbing";
  }
};

// Обработчик движения мыши (перемещение изображения)
const handleDragMouseMove = (event: MouseEvent) => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;

  if ((selectedSection.value === 'move' || movingActive.value) && !isDragging.value) {
    canvas.style.cursor = "grab";
  }
  
  if (isDragging.value) {
    console.log('imgOffsetX:', imgOffsetX.value, 'imgOffsetY:', imgOffsetY.value);
    const deltaX = event.clientX - dragStart.value.x;
    const deltaY = event.clientY - dragStart.value.y;

    console.log('deltaX:', deltaX, 'deltaY', deltaY);

    console.log(imgStartOffset.value.x + deltaX, imgStartOffset.value.y + deltaY)

    imgOffsetX.value = imgStartOffset.value.x + deltaX;
    imgOffsetY.value = imgStartOffset.value.y + deltaY;

    drawImage(false);  // Перерисовываем изображение с новыми смещениями, без центрирования
  }
};

// Обработчик отпускания мыши (окончание перемещения)
const handleDragMouseUp = () => {
  if (isDragging.value) {
    const canvas = document.getElementById('canvas') as HTMLCanvasElement;

    isDragging.value = false;
    canvas.style.cursor = null;
  }
};

// Обработчик нажатия клавиши space
const handleSpaceKeyDown = (event) => {
  if (event.keyCode === 32 && !movingActive.value && selectedSection.value !== 'move' && imageUploaded.value) {
    movingActive.value = true;
  }
}

// Обработчик отпускания клавиши space
const handleSpaceKeyUp = (event) => {
  if (event.keyCode === 32 && selectedSection.value !== 'move' && imageUploaded.value) {
    const canvas = document.getElementById('canvas') as HTMLCanvasElement;

    isDragging.value = false;
    movingActive.value = false;
    canvas.style.cursor = null;
  }
}

// Обработчик нажатия клавиши `
const handleAltKeyDown = (event) => {
  if (event.keyCode === 192 && helpLabelsActive.value === 0) {
    helpLabelsActive.value = 1;
    document.body.style.cursor = "help";
  }
}

// Обработчик отпускания клавиши `
const handleAltKeyUp = (event) => {
  if (event.keyCode === 192) {
    helpLabelsActive.value = 0;
    document.body.style.cursor = null;
  }
}



onMounted(() => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  window.addEventListener('resize', resizeCanvas);
  canvas.addEventListener('wheel', handleWheel);
  canvas.addEventListener('mousemove', handleMouseMove);
  
  // Рука
  canvas.addEventListener('mousedown', handleDragMouseDown);
  canvas.addEventListener('mousemove', handleDragMouseMove);
  canvas.addEventListener('mouseup', handleDragMouseUp);
  canvas.addEventListener('mouseleave', handleDragMouseUp);
  // ====

  // Модификатор-помощник
  window.addEventListener('keydown', handleAltKeyDown)
  window.addEventListener('keyup', handleAltKeyUp)
  // ====================

  // Модификатор-рука
  window.addEventListener('keydown', handleSpaceKeyDown)
  window.addEventListener('keyup', handleSpaceKeyUp)
  // ====================
  resizeCanvas();
});

onUnmounted(() => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  window.removeEventListener('resize', resizeCanvas);
  canvas.removeEventListener('wheel', handleWheel);
  canvas.removeEventListener('mousemove', handleMouseMove);

  // Рука
  canvas.removeEventListener('mousedown', handleDragMouseDown);
  canvas.removeEventListener('mousemove', handleDragMouseMove);
  canvas.removeEventListener('mouseup', handleDragMouseUp);
  canvas.removeEventListener('mouseleave', handleDragMouseUp);
  // ====

  // Модификатор-помощник
  window.removeEventListener('keydown', handleAltKeyDown)
  window.removeEventListener('keyup', handleAltKeyUp)
  // ====================

  // Модификатор-рука
  window.removeEventListener('keydown', handleSpaceKeyDown)
  window.removeEventListener('keyup', handleSpaceKeyUp)
  // ====================
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
.minified-input-label {
  width: 1rem;
}
</style>