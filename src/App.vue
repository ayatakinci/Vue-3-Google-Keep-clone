<script setup>
import { ref } from "vue"

const showModal = ref(false)
const newNote = ref("")
const notes = ref([])
const editingIndex = ref(-1);
const editNoteText = ref("");
const editModalShow = ref(false);

function getRandomColor() {
  const color = "hsl(" + Math.random() * 360 + ", 100%, 75%)";
  return color;
}

const addNote = () => {
  notes.value.push({
    id: Math.floor(Math.random() * 100000),
    text: newNote.value,
    color: getRandomColor(),
    date: new Date()
  })
  showModal.value = false;
  newNote.value = ""
}

const deleteNote = (index) => {
  notes.value.splice(index, 1);
};

const editNoteModal = (index) => {
  editingIndex.value = index; editNoteModal
  editNoteText.value = notes.value[index].text;
  editModalShow.value = true;
};


const saveNote = () => {
  notes.value[editingIndex.value].text = editNoteText.value;
  editingIndex.value = -1;
  editNoteText.value = "";
  editModalShow.value = false;
};
const editedNote = ref("");
</script>

<template>
  <main>
    <div v-if="showModal || editModalShow" class="overlay" @click="editModalShow = false">
      <div class="modal" @click.stop>
        <p v-if="!editModalShow" @click="showModal = false">x</p>
        <p v-else @click="editModalShow = false">x</p>
        <template v-if="!editModalShow">
          <textarea v-model="newNote" @keydown.enter="addNote"></textarea>
          <button @click="addNote">Add Note</button>
        </template>
        <template v-else>
          <textarea v-model="editNoteText"></textarea>
          <button @click="saveNote">Save</button>
        </template>
      </div>
    </div>
    <div class="container">
      <header>
        <h1>Notes</h1>
        <button @click="showModal = true">+</button>
      </header>
      <div class="cards-container">
        <div v-for="(note, index) in notes" class="card" :style="{ backgroundColor: note.color }">
          <p class="main-text">{{ note.text }}</p>
          <p class="date">{{ note.date.toLocaleDateString("en-US") }}</p>
          <div class="buttons-container">
            <button @click="editNoteModal(index)">Edit</button>
            <button @click="deleteNote(index)">Delete</button>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>




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
</style>