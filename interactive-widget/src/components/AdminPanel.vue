<template>
  <div class="admin-panel">
    <h2>Admin Settings</h2>

    <label>Upload Logo:</label>
    <input type="file" @change="uploadLogo" />

    <label>Primary Color:</label>
    <input type="color" v-model="primaryColor" @input="updateStyles" />

    <label>Font:</label>
    <select v-model="selectedFont" @change="updateStyles">
      <option value="Arial">Arial</option>
      <option value="Verdana">Verdana</option>
      <option value="Times New Roman">Times New Roman</option>
    </select>

    <label>Font Size:</label>
    <input type="number" v-model="fontSize" @input="updateStyles" />

    <button @click="$emit('close')">Close</button>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";

export default {
  emits: ["close", "logoUpdated"],
  setup(_, { emit }) {
    const primaryColor = ref(localStorage.getItem("primaryColor") || "#007bff");
    const selectedFont = ref(localStorage.getItem("selectedFont") || "Arial");
    const fontSize = ref(localStorage.getItem("fontSize") || 16);

    function uploadLogo(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = () => {
          const newLogo = reader.result;
          localStorage.setItem("customLogo", newLogo);
          emit("logoUpdated", newLogo);
        };
        reader.readAsDataURL(file);
      }
    }

    function updateStyles() {
      localStorage.setItem("primaryColor", primaryColor.value);
      localStorage.setItem("selectedFont", selectedFont.value);
      localStorage.setItem("fontSize", fontSize.value);

      document.documentElement.style.setProperty("--primary-color", primaryColor.value);
      document.documentElement.style.setProperty("--font-family", selectedFont.value);
      document.documentElement.style.setProperty("--font-size", fontSize.value + "px");
    }

    onMounted(() => {
      updateStyles();
    });

    return { primaryColor, selectedFont, fontSize, uploadLogo, updateStyles };
  }
};
</script>

<style scoped>
.admin-panel {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  color: black;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.3);
  width: 300px;
}

button {
  background-color: var(--primary-color, #007bff);
  color: white;
  padding: 8px 12px;
  border: none;
  margin-top: 10px;
  cursor: pointer;
}

button:hover {
  background-color: darkblue;
}
</style>
