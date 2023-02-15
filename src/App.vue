<script setup>
import { ref, onMounted } from "vue";

const visualView = ref(null);
const historyUndo = ref([]);
const historyRedo = ref([]);
const selectedTextLength = ref(0);
let clickedUndo = false;
let correctPos = 0;

function highlight() {
  document.querySelectorAll(".visual-view > *").forEach((el) => {
    if (el.classList.value.includes("active")) {
      el.classList.remove("active-el");
    }
  });
  if (this.nodeName === "IMG") {
    this.classList.add("active-el");
    removeSelection();
  }
}

function removeSelection() {
  if (window.getSelection) {
    if (window.getSelection().empty) {
      window.getSelection().empty();
    } else if (window.getSelection().removeAllRanges) {
      window.getSelection().removeAllRanges();
    }
  } else if (document.selection) {
    document.selection.empty();
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
  document.querySelectorAll(".visual-view > *").forEach((el) => {
    el.addEventListener("click", highlight);
  });
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
  document.querySelectorAll(".visual-view > *").forEach((el) => {
    el.addEventListener("click", highlight);
  });
  historyRedo.value.pop();
}

function wrap(tag) {
  const selection = window.getSelection().getRangeAt(0);
  const node = window.getSelection().getRangeAt(0).startContainer.parentNode;
  const nodeText = node.innerText;
  const selectedText = selection.extractContents();

  if (selectedText.querySelector("img")) {
    selectedTextLength.value = 0;
    selection.insertNode(selectedText);
    removeSelection();

    const tagName = node.nodeName.toLowerCase();
    const slicedIndex = Array.from(node.parentNode.children).indexOf(node);
    const prev = node.parentNode.children[slicedIndex];
    const next = node.parentNode.children[slicedIndex + 1];
    if (prev.nodeName === next.nodeName) {
      const restoredElem = document.createElement(tagName);
      restoredElem.innerText = nodeText;
      prev.remove();
      next.remove();
      document
        .querySelector(".visual-view")
        .insertBefore(
          restoredElem,
          document.querySelector(".visual-view").children[slicedIndex]
        );
    }

    alert("Выделите текст без картинки");
    return;
  }

  const afterText = node.innerText;
  let wrapElement;

  if (tag === "title") {
    wrapElement = document.createElement("h3");
  }
  if (tag === "paragraph") {
    wrapElement = document.createElement("p");
  }

  wrapElement.appendChild(selectedText);
  const beforeText = node.innerText;
  const newTag = node.nodeName.toLowerCase();
  const beforeElement = document.createElement(newTag);
  beforeElement.innerText = beforeText.slice(
    0,
    window.getSelection().anchorOffset - 1
  );
  const afterElement = document.createElement(newTag);
  afterElement.innerText = afterText.slice(
    window.getSelection().anchorOffset - 1 + wrapElement.innerText.length
  );

  const nodeIndex = Array.from(node.parentNode.children).indexOf(node);
  node.parentNode.insertBefore(
    afterElement,
    node.parentNode.children[nodeIndex]
  );
  node.parentNode.insertBefore(
    wrapElement,
    node.parentNode.children[nodeIndex]
  );
  node.parentNode.insertBefore(
    beforeElement,
    node.parentNode.children[nodeIndex]
  );
  node.remove();
}

function placeImage() {
  console.log("image");
}

function copyHtml() {
  navigator.clipboard.write([
    new ClipboardItem({
      "text/html": new Blob([document.querySelector(".visual-view").innerHTML], { type: "text/html" }),
    }),
  ]);
  alert("Сохранено в виде HTML");
  const pos = document.querySelector(".container").getBoundingClientRect().x;
  if (pos !== correctPos) {
    let move = correctPos - pos;
    document.querySelector(".container").style.transform = `translate(-${move}px, -${move}px)`;
  }
}

function keyDownHandler(e) {
  if (
    (e.key.length === 1 ||
      e.key === "Delete" ||
      e.key === "Enter" ||
      e.key === "Backspace") &&
    !e.ctrlKey
  ) {
    const elem = visualView.value.cloneNode(true);
    historyUndo.value.push(elem);
    historyRedo.value = [];
    clickedUndo = false;
  }

  if (e.key === "Delete") {
    historyUndo.value.pop();
    const selected = document.querySelector(".active-el");
    selected.classList.remove("active-el");
    const elem = visualView.value.cloneNode(true);
    historyUndo.value.push(elem);
    selected.remove();
  }

  const key = e.key;
  if (key.toString() === "z" && e.ctrlKey && historyUndo.value.length > 0) {
    undo();
  }
}

onMounted(() => {
  document.querySelectorAll(".visual-view > *").forEach((el) => {
    el.addEventListener("click", highlight);
  });
  document.addEventListener("click", (e) => {
    if (e.target.nodeName !== "IMG") {
      document.querySelectorAll(".visual-view > *").forEach((el) => {
        if (el.classList.value.includes("active")) {
          el.classList.remove("active-el");
        }
      });
    }
  });
  document.querySelector(".visual-view").addEventListener("mouseup", () => {
    try {
      selectedTextLength.value = window
      .getSelection()
      .getRangeAt(0)
      .toString().length;
    } catch (error) {
      console.log();
    }
  });
  document
    .querySelector(".visual-view")
    .addEventListener("keydown", keyDownHandler);
  correctPos = document.querySelector(".container").getBoundingClientRect().x;
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
          :disabled="selectedTextLength === 0"
          @click="wrap('title')"
        />
        <va-button
          icon="text_fields"
          color="shadow"
          icon-color="#262824"
          class="mr-3"
          :disabled="selectedTextLength === 0"
          @click="wrap('paragraph')"
        />
        <va-button
          icon="image"
          color="shadow"
          icon-color="#262824"
          class="mr-3"
          @click="placeImage"
        />
        <a href="#" class="va-link ml-3" @click="copyHtml"
          >Скопировать HTML</a
        >
      </div>
    </div>
    <div class="content">
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
        <h3>Смотрите какие обезьянки</h3>
        <img
          src="https://i.ibb.co/6Rz6d5D/image-1.png"
          alt="Обезьянки"
        />
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
* {
  transition-property: none !important;
}

body {
  margin: 0;
  padding: 0;
  min-height: 100vh;
  scrollbar-width: thin;
  width: 100vw;
  overflow-x: hidden;
}

h1,
h2,
h3,
h4,
h5,
h6,
p {
  color: currentColor;
}

h1,
h2,
h3,
h4,
h5,
h6 {
  font-weight: 700 !important;
}

h1 {
  font-size: 3rem !important;
  line-height: 3.5rem !important;
}

h2 {
  font-size: 2.5rem !important;
  line-height: 3rem !important;
}

h3 {
  font-size: 2rem !important;
  line-height: 2.5rem !important;
}

h4 {
  font-size: 1.75rem !important;
  line-height: 2rem !important;
}

h5 {
  font-size: 1.5rem !important;
  line-height: 1.75rem !important;
}

h6 {
  font-size: 1.25rem !important;
  line-height: 1.5rem !important;
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
  border-radius: 4px;
}

.content {
  padding: 30px;
  background: #fff;
  min-height: calc(100vh - 175px);
  margin-bottom: 30px;
  border-radius: 4px;
}

.visual-view {
  min-height: calc(100vh - 205px);

  & > * {
    margin-bottom: 30px;

    &:last-child {
      margin-bottom: 0;
    }
  }
}

p {
  font-weight: 400;
  font-size: 15px;
  line-height: 23px;
}

img {
  max-height: 300px;
  border-radius: 4px;
}

.active-el {
  outline: 2px solid var(--va-link-color);
}
</style>
