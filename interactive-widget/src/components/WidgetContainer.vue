<template>
  <div>
    <img class="floating-logo" :src="logo" @click="toggleWidget" />

    <div
      v-if="isOpen"
      class="widget-container"
      :style="{
        backgroundColor: bgColor,
        fontSize: fontSize + 'px',
        fontFamily: fontFamily,
      }"
    >
      <div class="widget-header">
        <img :src="logo" alt="logo" class="logo-header" />
      </div>

      <div v-if="!problemDescription">
        <span class="text">What problem are you trying to solve?</span>
        <textarea
          v-model="userInput"
          placeholder="Describe your issue..."
          class="problem-input"
        ></textarea>
        <button @click="submitProblemDescription" :disabled="!userInput.trim()">
          Submit
        </button>
      </div>

      <div v-else-if="currentQuestion">
        <span class="text question-text">{{ currentQuestion.text }}</span>

        <div
          v-if="currentQuestion.type === 'text-enum'"
          class="text-enum-options"
        >
          <button
            v-for="option in currentQuestion.options"
            :key="option"
            @click="selectAnswer(option)"
          >
            {{ option }}
          </button>
        </div>

        <div
          v-else-if="currentQuestion.type === 'image-enum'"
          class="image-options"
        >
          <img
            v-for="option in currentQuestion.options"
            :key="option"
            :src="option"
            @click="selectAnswer(option)"
            class="question-image"
            alt="Option Image"
          />
        </div>

        <input
          v-else-if="currentQuestion.type === 'text'"
          type="text"
          v-model="userAnswer"
          class="responsive-input"
        />
        <input
          v-else-if="currentQuestion.type === 'number'"
          type="number"
          v-model="userAnswer"
          class="responsive-input"
        />

        <button
          @click="submitAnswer"
          :disabled="!userAnswer && !selectedOption"
        >
          Submit
        </button>
      </div>

      <AnsweredQuestions
        v-if="answers.length > 0"
        :answers="answers"
        @edit="editAnswer"
      />

      <div v-if="solution">
        <span class="text">Recommended Solution:</span>
        <span class="text">{{ solution }}</span>
      </div>

      <FeedbackForm v-if="solution" @feedback="handleFeedback" />

      <!-- Admin Access -->
      <button class="edit-btn" @click="togglePasswordPrompt">
        Admin Access
      </button>

      <!-- Password Input -->
      <div v-if="showPasswordPrompt" class="password-panel">
        <label class="text">Enter Admin Password:</label>
        <input type="password" v-model="adminPassword" />
        <button @click="checkPassword">Submit Password</button>
        <p v-if="passwordError" class="error-text">Incorrect password!</p>
      </div>

      <!-- Admin Panel -->
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
        <input type="file" @change="handleLogoUpload" accept="image/*" />

        <button @click="applyChanges">Apply Changes</button>
        <button class="edit-btn logout-btn" @click="logoutAdmin">Logout</button>
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
    const showPasswordPrompt = ref(false);
    const adminPassword = ref("");
    const passwordError = ref(false);
    const correctPassword = "admin123";

    const bgColor = ref("#212121");
    const newBgColor = ref("#212121");

    const fontSize = ref(16);
    const newFontSize = ref(16);

    const fontFamily = ref("Arial, sans-serif");
    const newFontFamily = ref("Arial, sans-serif");

    function toggleWidget() {
      isOpen.value = !isOpen.value;
    }

    function togglePasswordPrompt() {
      showPasswordPrompt.value = !showPasswordPrompt.value;
      passwordError.value = false;
    }

    function checkPassword() {
      if (adminPassword.value === correctPassword) {
        showEdit.value = true;
        showPasswordPrompt.value = false;
        adminPassword.value = "";
      } else {
        passwordError.value = true;
        adminPassword.value = "";
      }
    }

    function logoutAdmin() {
      showEdit.value = false;
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
      answers.value.push({
        question: currentQuestions.value[currentQuestionIndex.value],
        answer,
      });

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
      const questionIndex = answers.value.findIndex(
        (a) => a.question.id === questionId
      );
      if (questionIndex !== -1) {
        currentQuestionIndex.value = questionIndex;
        answers.value.splice(questionIndex, 1);
      }
    }

    function generateSolution() {
      let solutionText =
        "Based on your responses, here is your recommended solution.";

      const categoryAnswer = answers.value.find(
        (a) => a.question.id === 1
      )?.answer;
      const severityAnswer = answers.value.find(
        (a) => a.question.id === 3
      )?.answer;

      if (categoryAnswer === "Software") {
        solutionText = "Try restarting your software and check for updates.";
      } else if (categoryAnswer === "Hardware") {
        solutionText =
          "Check hardware connections and ensure components are properly seated.";
      } else {
        solutionText = "For further troubleshooting, please contact support.";
      }

      if (severityAnswer >= 8) {
        solutionText +=
          " Since the severity is high, you should seek immediate professional help.";
      }

      solution.value = solutionText;
    }

    const currentQuestion = computed(() => {
      return currentQuestionIndex.value !== null
        ? currentQuestions.value[currentQuestionIndex.value]
        : null;
    });

    function handleFeedback(feedback) {
      console.log("User feedback:", feedback);
    }

    function handleLogoUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = () => {
          logo.value = reader.result;
        };
        reader.readAsDataURL(file);
      }
    }

    return {
      isOpen,
      currentQuestion,
      userAnswer,
      currentQuestion,
      selectedOption,
      solution,
      handleFeedback,
      editAnswer,
      toggleWidget,
      logo,
      problemDescription,
      userInput,
      answers,
      showEdit,
      showPasswordPrompt,
      adminPassword,
      passwordError,
      togglePasswordPrompt,
      checkPassword,
      logoutAdmin,
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
      generateSolution,
      handleLogoUpload,
    };
  },
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
  justify-content: flex-end; /* Align logo to the right */
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

.edit-panel input,
.edit-panel select {
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
