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
        <el-tooltip effect="light" content="Инструмент коррекции градиента" placement="right" :disabled="!helpLabelsActive">
          <el-menu-item index="8" @click="selectSection('gradient')" :disabled="!imageUploaded">
            <el-icon><histogram /></el-icon>
            <span>Кривые</span>
          </el-menu-item>
        </el-tooltip>
        <el-tooltip effect="light" content="Инструмент фильтра" placement="right" :disabled="!helpLabelsActive">
          <el-menu-item index="9" @click="selectSection('filter')" :disabled="!imageUploaded">
            <el-icon><setting /></el-icon>
            <span>Фильтр</span>
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
  
      <el-empty v-if="selectedSection === 'save'" description=" ">
        <el-text size="large">Сохранение изображения</el-text>
        <el-button style="margin-top: 1rem;" @click="saveImage">Сохранить локально</el-button>
      </el-empty>

      <el-empty v-if="selectedSection === 'move'" description=" ">
        <el-text size="large">Зажмите левую клавишу, чтобы начать перемещение курсором</el-text>
      </el-empty>

      <el-form v-if="selectedSection === 'picker'">
        <el-text size="large" style="margin-bottom: 1rem;">Инструмент "Пипетка"</el-text>
        <el-form-item>
          <el-text size="large" style="padding-bottom: 1rem;">Первый цвет</el-text>

          <div class="color-preview" :style="{backgroundColor: `rgba(${firstPixelColor.r},${firstPixelColor.g},${firstPixelColor.b},${firstPixelColor.a})`}"></div>
          <!-- RGB -->
          <el-input readonly v-if="selectedFormat === 'RGB'"
                    :model-value="firstPixelColor.r? firstPixelColor.r: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>R</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'RGB'"
                    :model-value="firstPixelColor.g? firstPixelColor.g: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>G</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'RGB'"
                    :model-value="firstPixelColor.b? firstPixelColor.b: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>B</template>
          </el-input>

          <!-- XYZ -->
          <el-input readonly v-if="selectedFormat === 'XYZ'"
                    :model-value="firstPixelColor.r? rgbToXyz(firstPixelColor).x: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>X</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'XYZ'"
                    :model-value="firstPixelColor.g? rgbToXyz(firstPixelColor).y: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>Y</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'XYZ'"
                    :model-value="firstPixelColor.b? rgbToXyz(firstPixelColor).z: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>Z</template>
          </el-input>

          <!-- LAB -->
          <el-input readonly v-if="selectedFormat === 'LAB'"
                    :model-value="firstPixelColor.r? xyzToLab(rgbToXyz(firstPixelColor)).l: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>L</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'LAB'"
                    :model-value="firstPixelColor.g? xyzToLab(rgbToXyz(firstPixelColor)).a: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>A</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'LAB'"
                    :model-value="firstPixelColor.b? xyzToLab(rgbToXyz(firstPixelColor)).b: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>B</template>
          </el-input>
          <el-input readonly
              :model-value="firstCursorPosition.x >= 0? firstCursorPosition.x + ' px': undefined"
              placeholder="Нет данных" style="margin-top: 0.5rem;"
          >
            <template class="minified-input-label" #prepend>X</template>
          </el-input>
          <el-input readonly
                    :model-value="firstCursorPosition.y >= 0? firstCursorPosition.y + ' px': undefined"
              placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>Y</template>
          </el-input>
        </el-form-item>

        <el-form-item>
          <el-text size="large" style="margin-bottom: 1rem;">Второй цвет</el-text>

          <div class="color-preview" :style="{backgroundColor: `rgba(${secondPixelColor.r},${secondPixelColor.g},${secondPixelColor.b},${secondPixelColor.a})`}"></div>
          <!-- RGB -->
          <el-input readonly v-if="selectedFormat === 'RGB'"
                    :model-value="secondPixelColor.r? secondPixelColor.r: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>R</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'RGB'"
                    :model-value="secondPixelColor.g? secondPixelColor.g: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>G</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'RGB'"
                    :model-value="secondPixelColor.b? secondPixelColor.b: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>B</template>
          </el-input>

          <!-- XYZ -->
          <el-input readonly v-if="selectedFormat === 'XYZ'"
                    :model-value="secondPixelColor.r? rgbToXyz(secondPixelColor).x: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>X</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'XYZ'"
                    :model-value="secondPixelColor.g? rgbToXyz(secondPixelColor).y: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>Y</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'XYZ'"
                    :model-value="secondPixelColor.b? rgbToXyz(secondPixelColor).z: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>Z</template>
          </el-input>

          <!-- LAB -->
          <el-input readonly v-if="selectedFormat === 'LAB'"
                    :model-value="secondPixelColor.r? xyzToLab(rgbToXyz(secondPixelColor)).l: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>L</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'LAB'"
                    :model-value="secondPixelColor.g? xyzToLab(rgbToXyz(secondPixelColor)).a: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>A</template>
          </el-input>
          <el-input readonly v-if="selectedFormat === 'LAB'"
                    :model-value="secondPixelColor.b? xyzToLab(rgbToXyz(secondPixelColor)).b: 0"
                    placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>B</template>
          </el-input>
          <el-input readonly
              :model-value="secondCursorPosition.x >= 0? secondCursorPosition.x + ' px': undefined"
              placeholder="Нет данных" style="margin-top: 0.5rem;"
          >
            <template class="minified-input-label" #prepend>X</template>
          </el-input>
          <el-input readonly
                    :model-value="secondCursorPosition.y >= 0? secondCursorPosition.y + ' px': undefined"
              placeholder="Нет данных"
          >
            <template class="minified-input-label" #prepend>Y</template>
          </el-input>
        </el-form-item>

        <el-tooltip
        class="box-item"
        effect="dark"
        content="R, G, B — значения от 0 до 255<br/>X (0-95), Y (0-100), Z (0-108)<br/>L (0-100), a (-128 до 128), b (-128 до 128)"
        placement="bottom"
        raw-content
        >
          <el-button style="display: block; width: 100%;" @click="changeColorFormat">Сменить цветовое пространство</el-button>
        </el-tooltip>
        
        <el-divider/>
        <el-text size="large" style="display: block;">Расчёт контрастности цвета</el-text>
        <el-text type="info" v-if="firstPixelColor.r == -1 || secondPixelColor.r == -1">Сперва выберите два цвета</el-text>
        <el-button v-if="firstPixelColor.r != -1 && secondPixelColor.r != -1" style="display: block; width: 100%;" @click="getContrastRatio(firstPixelColor, secondPixelColor)">Рассчитать контрастность</el-button>
        <el-text style="display: block;" v-if="contrastRatio != -1"><span style="color: #9D3C3C" v-if="contrastRatio < 4.5">Контраст недостаточен ({{ contrastRatio.toFixed(2) }}:1)</span><span style="color: #419D3C;" v-else>Контрастность достаточна ({{ contrastRatio.toFixed(2) }}:1)</span></el-text>
      </el-form>

      <el-form v-show="selectedSection === 'gradient'">
        <el-text size="large" style="padding-bottom: 2rem;">Коррекция градиента</el-text>
        <div style="margin-top: 1rem;">
          <!-- background: #333333 -->
          <canvas ref="gradientCanvas" width="256" height="256" style="width: 255px; height: 255px; background: #FFF;"></canvas>
        </div>
        <el-form-item style="margin-bottom: 0.5rem;">
          <el-col :span="11">
            <el-input-number
              @change="changeCurvePoint"
              v-model="curvePoints.enter.in"
              :placeholder="'In'"
              :min="0"
              :max="255"
              controls-position="right"
              style="width: 100%;"
            />
          </el-col>
          <el-col :span="1"></el-col>
          <el-col :span="11">
            <el-input-number
              @change="changeCurvePoint"
              v-model="curvePoints.enter.out"
              :placeholder="'Out'"
              :min="0"
              :max="255"
              controls-position="right"
              style="width: 100%;"
            />
          </el-col>
        </el-form-item>
        <el-form-item style="margin-bottom: 0;">
          <el-col :span="11">
            <el-input-number
              @change="changeCurvePoint"
              v-model="curvePoints.exit.in"
              :placeholder="'In'"
              :min="0"
              :max="255"
              controls-position="right"
              style="width: 100%;"
            />
          </el-col>
          <el-col :span="1"></el-col>
          <el-col :span="11">
            <el-input-number
              @change="changeCurvePoint"
              v-model="curvePoints.exit.out"
              :placeholder="'Out'"
              :min="0"
              :max="255"
              controls-position="right"
              style="width: 100%;"
            />
          </el-col>
        </el-form-item>
        <p v-if="curveErrorMessage">Ошибка: {{ curveErrorMessage }}</p>
        <el-form-item>
          <el-checkbox v-model="showCurvePreview" label="Показывать предпросмотр" size="large" />
        </el-form-item>
        <canvas v-show="showCurvePreview" id="tempCurveCanvas" width="256" height="256" style="width: 255px; height: 255px; background: #FFF;"></canvas>
        <div style="margin-bottom: .25rem;">
          <el-button style="display: block; width: 100%;" @click="resetCurvePoints">Сбросить</el-button>
        </div>
        <div>
          <el-button style="display: block; width: 100%;" type="primary" @click="downloadFullSizeImage">Изменить</el-button>
        </div>
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

const firstCursorPosition = ref({x: -1, y: -1});
const firstPixelColor = ref({r: -1, g: -1, b: -1, a: -1});
const secondCursorPosition = ref({x: -1, y: -1});
const secondPixelColor = ref({r: -1, g: -1, b: -1, a: -1});
const pickerActive = ref(false);
const altPickerActive = ref(false);
const selectedFormat = ref('XYZ');
const contrastRatio = ref(-1);

const gradientCanvas = ref(null);
const showCurvePreview = ref(false);
const curveErrorMessage = ref(null);

const imgCurvePreviewRenderedWidth = ref(255)
const imgCurvePreviewRenderedHeight = ref(255);

const curvePoints = ref({
  "enter": {
      "in": 0,
      "out": 0,
    },
    "exit": {
      "in": 255,
      "out": 255,
    },
});

const renderGradient = () => {
  const canvas = gradientCanvas.value;
  canvas.width = 255;
  canvas.height = 255;
  const ctx = canvas.getContext('2d', {
      willReadFrequently: true
    });
  
  ctx.clearRect(0, 0, canvas.width, canvas.height);
    const colorsHistData = getColorsHistData();
    buildColorRows(colorsHistData);
    
    ctx.beginPath();
    
    ctx.lineWidth = 3;
    ctx.strokeStyle = "blue";
    ctx.moveTo(0, 255);
    ctx.lineTo(255, 0);
    ctx.closePath();
    ctx.stroke();
    
    ctx.strokeStyle = "black";
    ctx.lineWidth = 1;
    ctx.beginPath();
    ctx.moveTo(0, 255 - curvePoints.value.enter.out);
    ctx.lineTo(curvePoints.value.enter.in, 255 - curvePoints.value.enter.out);
    ctx.arc(curvePoints.value.enter.in, 255 - curvePoints.value.enter.out, 5, 0, 2 * Math.PI);
    ctx.lineTo(curvePoints.value.exit.in, 255 - curvePoints.value.exit.out);
    ctx.arc(curvePoints.value.exit.in, 255 - curvePoints.value.exit.out, 5, 0, 2 * Math.PI);
    ctx.lineTo(255, 255 - curvePoints.value.exit.out);
    ctx.stroke();
}

const getColorsHistData = () => {
  const canvas = document.getElementById('canvas');
  const ctx = canvas.getContext('2d', {
    willReadFrequently: true,
  });
  
  const srcData = ctx.getImageData(imgOffsetX.value, imgOffsetY.value, imgRenderedWidth.value, imgRenderedHeight.value).data;
  
  const colorsHistData = {
    "r": new Map(),
    "g": new Map(),
    "b": new Map(),
  };
  
  for (let i = 0; i < srcData.length; i += 4) {
    // Красный канал
    if (colorsHistData["r"].has(srcData[i])) {
      colorsHistData["r"].set(srcData[i], (colorsHistData["r"].get(srcData[i]) || 0) + 1);
    } else {
      colorsHistData["r"].set(srcData[i], 1); 
    }
    
    // Зеленый канал
    if (colorsHistData["g"].has(srcData[i + 1])) {
      colorsHistData["g"].set(srcData[i + 1], (colorsHistData["g"].get(srcData[i + 1]) || 0) + 1);
    } else {
      colorsHistData["g"].set(srcData[i + 1], 1); 
    }
    
    // Синий канал
    if (colorsHistData["b"].has(srcData[i + 2])) {
      colorsHistData["b"].set(srcData[i + 2], (colorsHistData["b"].get(srcData[i + 2]) || 0) + 1);
    } else {
      colorsHistData["b"].set(srcData[i + 2], 1); 
    }
  }
  
  console.log(colorsHistData);
  return colorsHistData;
};

const buildRGBColorRows = (data, color) => {
    const canvas = gradientCanvas.value;
    const ctx = canvas.getContext('2d', {
    willReadFrequently: true
    });
    const maxVal = Math.max(...data["r"].values(), ...data["g"].values(), ...data["b"].values());
    if (color === "r") {
      ctx.fillStyle = 'rgba(255, 0, 0, 0.65)';
    } else if (color === "g") {
      ctx.fillStyle = 'rgba(0, 255, 0, 0.65)';
    } else {
      ctx.fillStyle = 'rgba(0, 0, 255, 0.65)';
    }
    for (let i of [...data[color].keys()].sort()) {

      const h = Math.floor((data[color].get(i) || 0) * 256 / maxVal);
      ctx.fillRect(i, canvas.height, 1, -h);
    }
  };

  const buildColorRows = (data) => {
    const canvas = gradientCanvas.value;
    const ctx = canvas.getContext('2d', {
      willReadFrequently: true
    });
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    buildRGBColorRows(data, "r");
    buildRGBColorRows(data, "g");
    buildRGBColorRows(data, "b");
  };

  const resetCurvePoints = () => {
    curvePoints.value = {
      "enter": {
      "in": 0,
      "out": 0,
    },
    "exit": {
      "in": 255,
      "out": 255,
    },
    }
    changeCurvePoint();
  }

  const changeCurvePoint = () => {
    curveErrorMessage.value = null;
    if (curvePoints.value.exit.in <= curvePoints.value.enter.in) {
      curvePoints.value.exit.in = 0;
      curveErrorMessage.value = 'Входное значение первой точки не может быть больше или равно входному значению второй';
      return;
    }
    renderGradient();
    renderTempImage();
  }

  const getTempImageData = () => {
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d', {
      willReadFrequently: true,
    });

    scalePercentage.value = 100;
  
    updateScale();

    const srcImageData = ctx.getImageData(imgOffsetX.value, imgOffsetY.value, imgRef.value.width, imgRef.value.height).data;
    const newImageData = new Uint8ClampedArray(imgRef.value.width * imgRef.value.height * 4);
    
    const x1 = curvePoints.value.enter.in;
    const y1 = curvePoints.value.enter.out;
    const x2 = curvePoints.value.exit.in;
    const y2 = curvePoints.value.exit.out;

    const a = (y2 - y1) / (x2 - x1);
    const b = y1 - a * x1;

    const changePixelGammaCorrection = (i) => {
      if (srcImageData[i] <= x1) {
        newImageData[i] = y1;
      } else if (srcImageData[i] >= x2) {
        newImageData[i] = y2;
      } else {
        newImageData[i] = a * srcImageData[i] + b;
      }
    };

    for (let i = 0; i < newImageData.length; i += 4) {
      changePixelGammaCorrection(i);
      changePixelGammaCorrection(i + 1);
      changePixelGammaCorrection(i + 2);
      newImageData[i + 3] = srcImageData[i + 3];
    }

    const tempImageData = new ImageData(newImageData, imgRef.value.width, imgRef.value.height);
    return tempImageData;
  }

  const renderTempImage = async () => {
    const tempImageData = getTempImageData();

    const canvas = document.getElementById('tempCurveCanvas') as HTMLCanvasElement;
    const ctx = canvas.getContext('2d', {
      willReadFrequently: true
    });

    resizeImageData(tempImageData, tempImageData.width, tempImageData.height);
    

    // createImageBitmap(tempImageData).then(img => {
      
    //   ctx.drawImage(
    //     img,
    //     0,
    //     0,
    //     imgCurvePreviewRenderedWidth.value,
    //     imgCurvePreviewRenderedHeight.value
    //     );
    // })

    
    
  }

  const resizeImageData = async (imageData, width, height) => {
  const resizeWidth = width >> 0;  // Округляем до целого
  const resizeHeight = height >> 0;
  
  // Создаем уменьшенную копию изображения
  const ibm = await window.createImageBitmap(imageData, 0, 0, imageData.width / 2, imageData.height / 2, {
    resizeWidth, 
    resizeHeight
  });

  const canvas = document.getElementById('tempCurveCanvas') as HTMLCanvasElement;
  
  // Задаем размеры канваса в зависимости от нового размера изображения
  canvas.width = resizeWidth;
  canvas.height = resizeHeight;

  const ctx = canvas.getContext('2d');

  // Удаляем масштабирование, чтобы изображение правильно вписалось
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.drawImage(ibm, 0, 0, resizeWidth, resizeHeight);
  
  // Возвращаем данные изображения
  return ctx.getImageData(0, 0, resizeWidth, resizeHeight);
};


const downloadFullSizeImage = (filename = 'processed-image.png') => {
  const originalCanvas = document.createElement('canvas');
  
  // Используем исходные размеры изображения
  const originalWidth = imgRef.value.width;
  const originalHeight = imgRef.value.height;

  // Устанавливаем размеры канваса равными исходному изображению
  originalCanvas.width = originalWidth;
  originalCanvas.height = originalHeight;
  
  const ctx = originalCanvas.getContext('2d');
  
  // Получаем обработанные данные изображения (например, из tempImageData)
  const tempImageData = getTempImageData(); // Здесь используется ваша функция для получения обработанных данных
  
  // Отрисовываем данные на канвас в оригинальном размере
  ctx.putImageData(tempImageData, 0, 0);

  // Преобразуем содержимое канваса в данные изображения (по умолчанию PNG)
  const imageUrl = originalCanvas.toDataURL('image/png');  // Можете изменить на 'image/jpeg' для JPG формата
  

  const newImg = new Image();
    newImg.src = imageUrl;

    imgRef.value = newImg;

    drawImage()
}


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
  const ctx = canvas.getContext('2d', {
      willReadFrequently: true
    });
  const srcImageData = ctx.getImageData(imgOffsetX.value, imgOffsetY.value, imgRenderedWidth.value, imgRenderedHeight.value);

  const resizedImageData = resizeNearestNeighbor(srcImageData, newWidth.value, newHeight.value);

  const newCanvas = document.createElement('canvas');
  const newCtx = newCanvas.getContext('2d', {
      willReadFrequently: true
    });
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
  const ctx = canvas.getContext('2d', {
      willReadFrequently: true
    });
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
  if (selectedSection.value === 'gradient') {
    renderGradient();
  }
}

// Обработчик движения мыши
const handleMouseMove = (event: MouseEvent) => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  const ctx = canvas.getContext('2d', {
      willReadFrequently: true
    });
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

    // Если инструмент пипетки активен
    if (selectedSection.value === 'picker') {
      pickerActive.value = true;
    }
  } else {
    pickerActive.value = false;
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
  const ctx = canvas.getContext('2d', {
      willReadFrequently: true
    });
  if (!ctx || !imgRef.value) return;

  // Создаём новый canvas для сохранения
  const saveCanvas = document.createElement('canvas');
  const saveCtx = saveCanvas.getContext('2d', {
      willReadFrequently: true
    });

  // Устанавливаем размеры нового canvas
  saveCanvas.width = imgRef.value.width;  // Ширина сохраняемой области
  saveCanvas.height = imgRef.value.height; // Высота сохраняемой области

  // Копируем область изображения на новый canvas
  saveCtx?.drawImage(
    imgRef.value,
    0,
    0
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


//// Инструмент пипетка
const handleClickPicker = () => {
  if (pickerActive.value && altPickerActive.value && (cursorPosition.value.x >= 0 || cursorPosition.value.y >= 0)) {
    secondPixelColor.value = pixelColor.value;
    secondCursorPosition.value = cursorPosition.value;
    return;
  }
  if (pickerActive.value && (cursorPosition.value.x >= 0 || cursorPosition.value.y >= 0)) {
    firstPixelColor.value = pixelColor.value;
    firstCursorPosition.value = cursorPosition.value;
    return;
  }
}

//// Функции для обработки цветовых палитр
const rgbToXyz = (rgb) => {
  let r = rgb.r / 255
  let g = rgb.g / 255
  let b = rgb.b / 255

  if (r > 0.04045) {
    r = Math.pow(((r + 0.055) / 1.055), 2.4)
  } else {
    r = r / 12.92
  }

  if (g > 0.04045) {
    g = Math.pow(((g + 0.055) / 1.055), 2.4)
  } else {
    g = g / 12.92
  }

  if (b > 0.04045) {
    b = Math.pow(((b + 0.055) / 1.055), 2.4)
  } else {
    b = b / 12.92
  }

  r *= 100
  g *= 100
  b *= 100

  // Observer = 2°, Illuminant = D65
  const x = r * 0.4124 + g * 0.3576 + b * 0.1805
  const y = r * 0.2126 + g * 0.7152 + b * 0.0722
  const z = r * 0.0193 + g * 0.1192 + b * 0.9505

  return { x, y, z}
}
const xyzToLab = (xyz) => {
  // Observer = 2°, Illuminant = D65
  let x = xyz.x / 95.047
  let y = xyz.y / 100.000
  let z = xyz.z / 108.883

  if (x > 0.008856) {
    x = Math.pow(x, 0.333333333)
  } else {
    x = 7.787 * x + 0.137931034
  }

  if (y > 0.008856) {
    y = Math.pow(y, 0.333333333)
  } else {
    y = 7.787 * y + 0.137931034
  }

  if (z > 0.008856) {
    z = Math.pow(z, 0.333333333)
  } else {
    z = 7.787 * z + 0.137931034
  }

  const l = (116 * y) - 16
  const a = 500 * (x - y)
  const b = 200 * (y - z)

  return { l, a, b }
}

// Функция для расчета относительной яркости
function getLuminance(r, g, b) {
  const [R, G, B] = [r, g, b].map(value => {
    const normalized = value / 255;
    return normalized <= 0.03928
      ? normalized / 12.92
      : Math.pow((normalized + 0.055) / 1.055, 2.4);
  });

  // Формула относительной яркости
  return 0.2126 * R + 0.7152 * G + 0.0722 * B;
}

// Функция для расчета контраста между двумя цветами
function getContrastRatio(color1, color2) {
  const luminance1 = getLuminance(color1.r, color1.g, color1.b);
  const luminance2 = getLuminance(color2.r, color2.g, color2.b);

  // Более светлый цвет делим на более темный
  const lighter = Math.max(luminance1, luminance2);
  const darker = Math.min(luminance1, luminance2);

  contrastRatio.value = (lighter + 0.05) / (darker + 0.05);
}

const changeColorFormat = () => {
  switch (selectedFormat.value) {
    case "RGB":
      selectedFormat.value = "XYZ";
      break;
    
    case "XYZ":
      selectedFormat.value = "LAB";
      break;
    
    default:
      selectedFormat.value = "RGB";
      break;
  }
}

//// Горячие клавиши

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

// Обработчик нажатия клавиши Left Alt
const handleLeftAltKeyDown = (event) => {
  if ((event.altKey || event.ctrlKey || event.shiftKey) && !altPickerActive.value) {
    altPickerActive.value = true;
  }
}

// Обработчик отпускания клавиши Left Alt
const handleLeftAltKeyUp = (event) => {
  if (altPickerActive) {
    altPickerActive.value = false;
  }
}

onMounted(() => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  window.addEventListener('resize', resizeCanvas);
  canvas.addEventListener('wheel', handleWheel);
  canvas.addEventListener('mousemove', handleMouseMove);
  canvas.addEventListener('click', handleClickPicker);
  
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

  // Модификатор-пипетка
  window.addEventListener('keydown', handleLeftAltKeyDown)
  window.addEventListener('keyup', handleLeftAltKeyUp)
  resizeCanvas();
});

onUnmounted(() => {
  const canvas = document.getElementById('canvas') as HTMLCanvasElement;
  window.removeEventListener('resize', resizeCanvas);
  canvas.removeEventListener('wheel', handleWheel);
  canvas.removeEventListener('mousemove', handleMouseMove);
  canvas.removeEventListener('click', handleClickPicker);

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

  // Модификатор-пипетка
  window.removeEventListener('keydown', handleLeftAltKeyDown)
  window.removeEventListener('keyup', handleLeftAltKeyUp)
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