<script setup>
import { ref } from "vue";

const showModal = ref(false);
const newNote = ref("");
const notes = ref([]);
const editingIndex = ref(-1);
const editNoteText = ref("");
const editModalShow = ref(false);
const previewNote = ref(null);
const isPinned = ref(false);
const newTodo = ref("");
const todos = ref([]);

function getRandomColor() {
  const color = "hsl(" + Math.random() * 360 + ", 100%, 75%)";
  return color;
}

const addNote = () => {
  notes.value.push({
    id: Math.floor(Math.random() * 100000),
    text: newNote.value,
    completed: false,
    color: getRandomColor(),
    date: new Date(),
    todos: todos.value
  })
  showModal.value = false;
  newNote.value = "";
  todos.value = [];
}


const deleteNote = (index) => {
  notes.value.splice(index, 1);
  previewNote.value = null; // reset previewNote
  editModalShow.value = false; // hide edit modal if it's open for the deleted note
  todos.value = []; // remove todos for the deleted note
};


const editNoteModal = (index) => {
  editingIndex.value = index;
  if (notes.value[index]) {
    editNoteText.value = notes.value[index].text;
    todos.value = notes.value[index].todos || [];
    editModalShow.value = true;
    previewNote.value = null; // reset previewNote
  }
};


const saveNote = () => {
  if (editingIndex.value >= 0 && editingIndex.value < notes.value.length) {
    notes.value[editingIndex.value].text = editNoteText.value;
    notes.value[editingIndex.value].todos = todos.value;
    editingIndex.value = -1;
    editNoteText.value = "";
    editModalShow.value = false;
    previewNote.value = null; // reset previewNote
    todos.value = [];
  }
};


const togglePinned = (index) => {
  isPinned.value = !isPinned.value;
  const note = notes.value[index];
  notes.value.splice(index, 1);
  notes.value.unshift(note);
};

const showColorPicker = (event, index) => {
  event.stopPropagation();
  const colors = ["red", "white", "green"];
  const colorPicker = document.createElement("div");
  colorPicker.className = "color-picker";
  colorPicker.style.top = "386px";
  colorPicker.style.position = "absolute";
  colorPicker.style.zIndex = "999";
  colorPicker.style.display = "flex";
  colorPicker.style.flexWrap = "wrap";
  colorPicker.style.gap = "10px";
  colorPicker.style.padding = "10px";
  colorPicker.style.backgroundColor = "white";
  colorPicker.style.border = "1px solid gray";
  colorPicker.style.boxShadow = "2px 2px 10px rgba(0, 0, 0, 0.2)";

  // Get the position of the clicked note relative to the top of the viewport
  const noteRect = event.target.getBoundingClientRect();
  const noteTop = noteRect.top + window.pageYOffset;

  // Set the position of the color picker div to be below the clicked note
  colorPicker.style.left = noteRect.left + "px";
  colorPicker.style.top = noteTop + noteRect.height + "px";

  colors.forEach((color) => {
    const colorButton = document.createElement("button");
    colorButton.className = "color-button";
    colorButton.style.backgroundColor = color;
    colorButton.addEventListener("click", () => {
      notes.value[index].color = color;
      colorPicker.remove();
    });

    colorPicker.appendChild(colorButton);
  });


  document.body.appendChild(colorPicker);
};



const toggleCompleted = (todo) => {
  todo.completed = !todo.completed;
};
const addTodo =()=> {
  if (newTodo.value.trim() !== "") {
    todos.value.push({
      text: newTodo.value,
      completed: false
    });
    newTodo.value = "";
  }
}


</script>

<template>
  <main>
    <div v-if="showModal || editModalShow || previewNote" class="overlay" @click="showModal = false; editModalShow = false; previewNote = null">
      <div class="modal" @click.stop>
        <p v-if="!editModalShow && !previewNote" @click="showModal = false">x</p>
        <p v-else @click="showModal = false; editModalShow = false; previewNote = null">x</p>
        <template v-if="!editModalShow && !previewNote">
          <div>
            <input type="text" v-model="newTodo" @keydown.enter="addTodo" placeholder="Add a to do item">
            <ul>
              <li v-for="(todo, index) in todos" :key="index">
                <label :for="'checkbox-' + index">{{ todo.text }}</label>
                <input type="checkbox" :id="'checkbox-' + index" v-model="todo.completed">
                <button @click="deleteTodo(index)">Delete</button>
              </li>
            </ul>
            <button @click="addNote">Add</button>
          </div>
        </template>

        <template v-else-if="editNoteModal(index)">
  <textarea  v-model="editedNote.text"></textarea>
  <input v-model="todoInput" placeholder="Add a todo item">
  <button @click="addTodo">Add Todo</button>
  <ul>
    <li v-for="(todo, todoIndex) in todos">
      {{ todo }}
      <button @click="removeTodo(todoIndex)">X</button>
    </li>
  </ul>
  <button @click="saveNote">Save</button>
</template>

        <template v-else>
          <div class="preview-card" :style="{ backgroundColor: previewNote.color }">
            <p class="main-text">{{ previewNote.text }}</p>
            <p class="date">{{ previewNote.date.toLocaleDateString("en-US") }}</p>
            <ul v-if="previewNote.todos">
              <li v-for="(todo, index) in previewNote.todos" :key="index">
                <label :for="'preview-checkbox-' + index">{{ todo.text }}</label>
                <input type="checkbox" :id="'preview-checkbox-' + index" :checked="todo.completed" disabled>
              </li>
            </ul>
          </div>
        </template>

      </div>
    </div>

    <div class="container">
      <header>
        <h1>Notes</h1>
        <button @click="showModal = true">+</button>
      </header>
      <div class="cards-container">
        <div v-for="(note, index) in notes" class="card" :style="{ backgroundColor: note.color } "  :class="{ pinned: isPinned && index === 0 }"
          @click="previewNote = note">
          <button  @click="togglePinned(index)">Pin</button>
          <div class="change-color-container">
            <button @click.stop="showColorPicker($event, index)">Change color</button>
          </div>

          <p class="main-text">{{ note.text }}</p>
          <p class="date">{{ note.date.toLocaleDateString("en-US") }}</p>
          <ul v-if="note.todos">
            <li v-for="(todo, index) in note.todos" :key="index">
              <label :for="'todo-checkbox-' + index">{{ todo.text }}</label>
              <input type="checkbox" :id="'todo-checkbox-' + index" :checked="todo.completed" disabled>
            </li>
          </ul>
          <div class="buttons-container">
            <button @click="editNoteModal(index)">Edit</button>
            <button @click="deleteNote(index)">Delete</button>
          </div>
       
</div>
        </div>
      </div>

  </main>
</template>


<!--
  ? this is a deneme
  ! what do you think
  * this is a deneme
  todo:this is a todo
 -->
<style scoped>
.container {
  max-width: 1000px;
  padding: 10px;
  margin: 0 auto;
  color:black;
}

h1 {
  font-weight: bold;
  margin-bottom: 25px;
  font-size: 75px;
  color: white;
}

.card {
  width: 225px;
  height: 225px;
  background-color: rgb(237, 182, 44);
  padding: 10px;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  margin-right: 20px;
  margin-bottom: 20px;
}

.main-text {
  line-height: 1.25;
  font-size: 17.5px;
  font-weight: bold;
}

.date {
  font-size: 12.5px;
  margin-top: auto;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

header button {
  border: none;
  padding: 10px;
  width: 50px;
  height: 50px;
  cursor: pointer;
  background-color: rgb(21, 20, 20);
  border-radius: 1000px;
  color: white;
  font-size: 20px;
}

.cards-container {
  display: flex;
  flex-wrap: wrap;
}

.overlay {
  position: absolute;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.77);
  transform: translate(-50%, -50%);
  top: 50%;
  left: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
}

main {
  height: 100vh;
  width: 100vw;
}

.modal {
  width: 750px;
  background-color: white;
  border-radius: 10px;
  padding: 30px;
  position: relative;
  display: flex;
  flex-direction: column;
}

.modal button {
  padding: 10px 20px;
  font-size: 20px;
  width: 100%;
  background-color: blueviolet;
  border: none;
  color: white;
  cursor: pointer;
  margin-top: 15px;
}

.modal p {
  margin-left: auto;
  font-size: 20px;
  z-index: 100000;
  cursor: pointer;
}

textarea {
  width: 100%;
  height: 200px;
  padding: 5px;
  font-size: 20px;
}

.pinned {
 
  border: 3px solid gold;
}
.color-menu {
  position: absolute;
  top: calc(100% + 10px);
  left: 0;
  display: flex;
  flex-direction: row;
  gap: 10px;
  background-color: #ffffff;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
  padding: 5px;
  z-index: 999;
}

.color-option {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  cursor: pointer;
}

.color-option.red {
  background-color: #ff0000;
}

.color-option.white {
  background-color: #ffffff;
}

.color-option.green {
  background-color: #00ff00;
}
.color-picker {
  position: absolute;
  z-index: 999;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  padding: 10px;
  background-color: white;
  border: 1px solid gray;
  box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.2);
}
li {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  color:black
}

label {
  margin-right: 10px;
  flex-grow: 1;
  text-decoration: none;
}

input[type="checkbox"] {
  margin-right: 10px;
}

button {
  background-color: #fff;
  border: none;
  color: #333;
  cursor: pointer;
  font-size: 16px;
  margin-left: 10px;
}
li.completed label {
  text-decoration: line-through;
  color: gray;
}

</style>