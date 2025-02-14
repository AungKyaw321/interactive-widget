<template>
  <div>
    <!-- Floating Logo Button (Opens the Widget) -->
    <img class="floating-logo" src="/logo1.png" @click="toggleWidget" />

    <!-- Chat Widget (Hidden Until Opened) -->
    <div v-if="isOpen" class="widget-container" :style="{ backgroundColor: bgColor, fontSize: fontSize + 'px', fontFamily: fontFamily }">
      <div class="widget-header">
        <img :src="logo" alt="logo" class="logo-header" />
        <button class="close-button" @click="toggleWidget">✖</button>
      </div>

      <!-- Step 1: Ask for Problem Description -->
      <div v-if="!problemDescription">
        <span class="text">What problem are you trying to solve?</span>
        <textarea v-model="userInput" placeholder="Describe your issue..." class="problem-input"></textarea>
        <button @click="submitProblemDescription" :disabled="!userInput.trim()">Submit</button>
      </div>

      <!-- Step 2: Show One Question at a Time -->
      <div v-else-if="currentQuestion">
        <span class="text question-text">{{ currentQuestion.text }}</span>

        <div v-if="currentQuestion.type === 'text-enum'" class="text-enum-options">
          <button v-for="option in currentQuestion.options" :key="option" @click="selectAnswer(option)">
            {{ option }}
          </button>
        </div>

        <div v-else-if="currentQuestion.type === 'image-enum'" class="image-options">
          <img
            v-for="option in currentQuestion.options"
            :key="option"
            :src="option"
            @click="selectAnswer(option)"
            class="question-image"
            alt="Option Image"
          />
        </div>

        <input v-else-if="currentQuestion.type === 'text'" type="text" v-model="userAnswer" class="responsive-input" />
        <input v-else-if="currentQuestion.type === 'number'" type="number" v-model="userAnswer" class="responsive-input" />

        <!-- Fixed Submit Button -->
        <button @click="submitAnswer" :disabled="!userAnswer && !selectedOption">Submit</button>
      </div>

      <!-- Answered Questions (Always Visible) -->
      <AnsweredQuestions v-if="answers.length > 0" :answers="answers" @edit="editAnswer" />

      <!-- Step 3: Display the Solution -->
      <div v-if="solution">
        <span class="text">Recommended Solution:</span>
        <span class="text">{{ solution }}</span>
      </div>

      <FeedbackForm v-if="solution" @feedback="handleFeedback" />

      <!-- Edit Button to Show Customization Form -->
      <button class="edit-btn" @click="toggleEdit">Edit</button>

      <!-- Customization Panel -->
      <div v-if="showEdit" class="edit-panel">
        <label class="text">Change Background Color:</label>
        <input type="color" v-model="newBgColor" />

        <label class="text">Change Font Size (px):</label>
        <input type="number" v-model="newFontSize" min="10" max="30" />

        <label class="text">Change Font Style:</label>
        <select v-model="newFontFamily">
          <option value="Arial, sans-serif">Arial</option>
          <option value="'Times New Roman', serif">Times New Roman</option>
          <option value="'Courier New', monospace">Courier New</option>
          <option value="Verdana, sans-serif">Verdana</option>
        </select>

        <label class="text">Change Logo:</label>
        <input type="file" @change="handleLogoUpload" />

        <button @click="applyChanges">Apply</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from "vue";
import AnsweredQuestions from "./AnsweredQuestions.vue";
import FeedbackForm from "./FeedbackForm.vue";
import questionsData from "../mockData/questions";

export default {
  components: { AnsweredQuestions, FeedbackForm },
  setup() {
    const isOpen = ref(false);
    const logo = ref("/logo1.png");
    const answers = ref([]);
    const currentQuestions = ref([...questionsData]);
    const problemDescription = ref("");
    const userInput = ref("");
    const solution = ref("");
    const currentQuestionIndex = ref(0);
    const userAnswer = ref("");
    const selectedOption = ref("");

    const showEdit = ref(false);
    const bgColor = ref("#212121"); // Default dark background color
    const newBgColor = ref("#212121");

    const fontSize = ref(16); // Default font size
    const newFontSize = ref(16);

    const fontFamily = ref("Arial, sans-serif"); // Default font family
    const newFontFamily = ref("Arial, sans-serif");

    function toggleWidget() {
      isOpen.value = !isOpen.value;
    }

    function toggleEdit() {
      showEdit.value = !showEdit.value;
    }

    function applyChanges() {
      bgColor.value = newBgColor.value;
      fontSize.value = newFontSize.value;
      fontFamily.value = newFontFamily.value;
      showEdit.value = false;
    }

    function submitProblemDescription() {
      if (userInput.value.trim()) {
        problemDescription.value = userInput.value;
        currentQuestionIndex.value = 0;
      }
    }

    function selectAnswer(option) {
      selectedOption.value = option;
    }

    function submitAnswer() {
      if (!currentQuestions.value.length) return;

      const answer = selectedOption.value || userAnswer.value;
      answers.value.push({ question: currentQuestions.value[currentQuestionIndex.value], answer });

      if (currentQuestionIndex.value < currentQuestions.value.length - 1) {
        currentQuestionIndex.value++;
      } else {
        currentQuestionIndex.value = null;
        generateSolution();
      }

      userAnswer.value = "";
      selectedOption.value = "";
    }

    function editAnswer(questionId) {
      const questionIndex = answers.value.findIndex((a) => a.question.id === questionId);
      if (questionIndex !== -1) {
        currentQuestionIndex.value = questionIndex;
        answers.value.splice(questionIndex, 1);
      }
    }

    function generateSolution() {
      let solutionText = "Based on your responses, here is your recommended solution.";

      const categoryAnswer = answers.value.find((a) => a.question.id === 1)?.answer;
      const severityAnswer = answers.value.find((a) => a.question.id === 3)?.answer;

      if (categoryAnswer === "Software") {
        solutionText = "Try restarting your software and check for updates.";
      } else if (categoryAnswer === "Hardware") {
        solutionText = "Check hardware connections and ensure components are properly seated.";
      } else {
        solutionText = "For further troubleshooting, please contact support.";
      }

      if (severityAnswer >= 8) {
        solutionText += " Since the severity is high, you should seek immediate professional help.";
      }

      solution.value = solutionText;
    }

    const currentQuestion = computed(() => {
      return currentQuestionIndex.value !== null ? currentQuestions.value[currentQuestionIndex.value] : null;
    });

    function handleFeedback(feedback) {
      console.log("User feedback:", feedback);
    }

    // Logo upload handler
    function handleLogoUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = () => {
          logo.value = reader.result; // Set the new logo as the source
        };
        reader.readAsDataURL(file);
      }
    }

    return {
      isOpen,
      toggleWidget,
      logo,
      problemDescription,
      userInput,
      answers,
      currentQuestions,
      solution,
      showEdit,
      toggleEdit,
      applyChanges,
      bgColor,
      newBgColor,
      fontSize,
      newFontSize,
      fontFamily,
      newFontFamily,
      submitProblemDescription,
      selectAnswer,
      submitAnswer,
      editAnswer,
      handleFeedback,
      currentQuestion,
      userAnswer,
      selectedOption,
      handleLogoUpload,
    };
  }
};
</script>

<style>
/* Dark Theme */
:root {
  --bg-color: #212121;
  --text-color: #ffffff;
  --primary-color: #424242;
  --secondary-color: #333333;
}

/* Floating Logo Button */
.floating-logo {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 60px;
  cursor: pointer;
}

/* Chatbox */
.widget-container {
  position: fixed;
  bottom: 80px;
  right: 20px;
  width: 90vw;
  max-width: 350px;
  min-width: 250px;
  height: 500px;
  background: var(--bg-color);
  color: var(--text-color);
  border-radius: 10px;
  padding: 16px;
  overflow-y: auto;
}

/* Header */
.widget-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: var(--primary-color);
  padding: 10px;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
}

/* Smaller Header Logo */
.logo-header {
  width: 24px;
  height: auto;
}

/* Customization Panel */
.edit-panel {
  margin-top: 20px;
}

.edit-panel label {
  margin-top: 10px;
  display: block;
}

.edit-panel input, .edit-panel select {
  width: 100%;
  padding: 8px;
  margin-top: 8px;
  background: var(--secondary-color);
  color: white;
  border: 1px solid #555;
}

.edit-btn {
  background-color: var(--primary-color);
  color: white;
  padding: 8px;
  margin-top: 12px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  width: 100%;
}

.edit-btn:hover {
  background: var(--secondary-color);
}
</style>
