<script setup>
import defaultImg from "@/assets/img/image.png";
import { ref, onMounted } from "vue";

const visualView = ref(null);
const historyUndo = ref([]);
const historyRedo = ref([]);
let clickedUndo = false;

function highlight() {
  document.querySelectorAll(".visual-view > *").forEach((el) => {
    if (el.classList.value.includes("active")) {
      console.log(el.classList);
      el.classList.remove("active-el");
    }
  });
  if (this.nodeName === "DIV") {
    this.classList.add("active-el");
  }
}

function undo() {
  if (!clickedUndo) {
    let elem = visualView.value.cloneNode(true);
    historyRedo.value.push(elem);
    elem = historyUndo.value[historyUndo.value.length - 1].cloneNode(true);
    historyRedo.value.push(elem);
  } else {
    if (historyRedo.value.length >= 1) {
      const elem =
        historyUndo.value[historyUndo.value.length - 1].cloneNode(true);
      historyRedo.value.push(elem);
    } else {
      let elem =
        historyUndo.value[historyUndo.value.length - 1].cloneNode(true);
      historyRedo.value.push(elem);
      historyUndo.value.pop();
      elem = historyUndo.value[historyUndo.value.length - 1].cloneNode(true);
      historyRedo.value.push(elem);
    }
  }
  visualView.value.replaceChildren(
    ...historyUndo.value[historyUndo.value.length - 1].children
  );
  historyUndo.value.pop();
  clickedUndo = true;
}

function redo() {
  if (historyUndo.value.length >= 1) {
    const elem =
      historyRedo.value[historyRedo.value.length - 1].cloneNode(true);
    historyUndo.value.push(elem);
  } else {
    let elem = historyRedo.value[historyRedo.value.length - 1].cloneNode(true);
    historyUndo.value.push(elem);
    historyRedo.value.pop();
    elem = historyRedo.value[historyRedo.value.length - 1].cloneNode(true);
    historyUndo.value.push(elem);
  }
  visualView.value.replaceChildren(
    ...historyRedo.value[historyRedo.value.length - 1].children
  );
  historyRedo.value.pop();
}

function makeTitle() {
  console.log("title");
}

function makeParagraph() {
  console.log("paragraph");
}

function placeImage() {
  console.log("image");
}

function copyHtml() {
  console.log("html");
}

function keyDownHandler(e) {
  if ((e.key.length === 1 || e.key === 'Delete' || e.key === 'Enter' || e.key === 'Backspace') && !e.ctrlKey) {
    const elem = visualView.value.cloneNode(true);
    historyUndo.value.push(elem);
    historyRedo.value = [];
    clickedUndo = false;
  }

  const key = e.key;
  if (key.toString() === 'z' && e.ctrlKey && historyUndo.value.length > 0) {
    undo();
  }
}

onMounted(() => {
  document.querySelectorAll(".visual-view > *").forEach((el) => {
    el.addEventListener("click", highlight);
  });
  document
    .querySelector(".visual-view")
    .addEventListener("keydown", keyDownHandler);
});
</script>

<template>
  <div class="container">
    <div class="toolbar">
      <div class="d-flex align-center">
        <va-button
          icon="undo"
          color="shadow"
          icon-color="#262824"
          class="mr-3"
          :disabled="historyUndo.length < 1"
          @click="undo"
        />
        <va-button
          icon="redo"
          color="shadow"
          icon-color="#262824"
          class="mr-3"
          :disabled="historyRedo.length < 1"
          @click="redo"
        />
        <va-button
          icon="title"
          color="shadow"
          icon-color="#262824"
          class="mr-3"
          @click="makeTitle"
        />
        <va-button
          icon="text_fields"
          color="shadow"
          icon-color="#262824"
          class="mr-3"
          @click="makeParagraph"
        />
        <va-button
          icon="image"
          color="shadow"
          icon-color="#262824"
          class="mr-3"
          @click="placeImage"
        />
        <a href="#" class="va-link ml-3" @click="copyHtml">Скопировать HTML</a>
      </div>
    </div>
    <div class="content">
      <div>{{ history }}</div>
      <div class="visual-view" ref="visualView" contenteditable>
        <p>
          Таким образом консультация с широким активом представляет собой
          интересный эксперимент проверки позиций, занимаемых участниками в
          отношении поставленных задач. С другой стороны постоянное
          информационно-пропагандистское обеспечение нашей деятельности
          представляет собой интересный эксперимент проверки форм развития.
          Идейные соображения высшего порядка, а также укрепление и развитие
          структуры влечет за собой процесс внедрения и модернизации
          соответствующий условий активизации. Задача организации, в особенности
          же реализация намеченных плановых заданий играет важную роль в
          формировании дальнейших направлений развития. Повседневная практика
          показывает, что постоянное информационно-пропагандистское обеспечение
          нашей деятельности играет важную роль в формировании существенных
          финансовых и административных условий.
        </p>
        <br />
        <h3 class="va-h3">Смотрите какие обезьянки</h3>
        <br />
        <va-image :src="defaultImg" />
        <br />
        <br />
        <p>
          Таким образом консультация с широким активом представляет собой
          интересный эксперимент проверки позиций, занимаемых участниками в
          отношении поставленных задач. С другой стороны постоянное
          информационно-пропагандистское обеспечение нашей деятельности
          представляет собой интересный эксперимент проверки форм развития.
          Идейные соображения высшего порядка, а также укрепление и развитие
          структуры влечет за собой процесс внедрения и модернизации
          соответствующий условий активизации. Задача организации, в особенности
          же реализация намеченных плановых заданий играет важную роль в
          формировании дальнейших направлений развития. Повседневная практика
          показывает, что постоянное информационно-пропагандистское обеспечение
          нашей деятельности играет важную роль в формировании существенных
          финансовых и административных условий.
        </p>
        <br />
        <p>
          Товарищи! новая модель организационной деятельности требуют от нас
          анализа направлений прогрессивного развития. Задача организации, в
          особенности же постоянный количественный рост и сфера нашей активности
          требуют от нас анализа позиций, занимаемых участниками в отношении
          поставленных задач. Задача организации, в особенности же реализация
          намеченных плановых заданий требуют от нас анализа системы обучения
          кадров, соответствует насущным потребностям.
        </p>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
body {
  margin: 0;
  padding: 0;
  min-height: 100vh;
  scrollbar-width: thin;
  width: 100vw;
  overflow-x: hidden;
}

#app {
  width: 100%;
  min-height: 100vh;
}

.container {
  width: 100%;
  min-height: 100vh;
  max-width: 1024px;
  margin: 0 auto;
  padding-top: 30px;
}

.toolbar {
  padding: 15px 30px;
  background: #fff;
  margin-bottom: 15px;
}

.content {
  padding: 30px;
  background: #fff;
  min-height: calc(100vh - 175px);
  margin-bottom: 30px;
}

.visual-view {
  min-height: calc(100vh - 205px);
}

p {
  font-weight: 400;
  font-size: 15px;
  line-height: 23px;
}

.va-image {
  max-height: 300px;
}

.active-el {
  outline: 2px solid var(--va-link-color);
}
</style>
