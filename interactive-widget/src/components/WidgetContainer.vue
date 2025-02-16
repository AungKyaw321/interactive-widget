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
        <span class="widget-title"></span>
        <!-- ✅ Title on the left -->
        <img :src="logo" alt="logo" class="logo-header" />
        <!-- ✅ Logo on the right -->
      </div>

      <!-- First Question (Problem Description) -->
      <div v-if="!problemDescription" class="question-container">
        <span class="text-question-text"
          >What problem are you trying to solve?</span
        >

        <!-- Answer Box Below Question -->
        <div class="answer-container">
          <textarea
            v-model="userInput"
            placeholder="Describe your issue..."
            class="wide-textarea"
          ></textarea>
        </div>

        <!-- Submit Button Below Input -->
        <button
          @click="submitProblemDescription"
          :disabled="!userInput.trim()"
          class="submit-btn"
        >
          Submit
        </button>
      </div>

      <div v-else-if="currentQuestion" class="question-container">
        <span class="text-question-text">{{ currentQuestion.text }}</span>

        <!-- Answer Submission Directly Below -->
        <div class="answer-container">
          <div
            v-if="currentQuestion.type === 'text-enum'"
            class="text-enum-options"
          >
            <button
              v-for="option in currentQuestion.options"
              :key="option"
              @click="selectAnswer(option)"
              class="wide-answer-button"
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
            class="wide-input"
            placeholder="Type your answer here..."
          />
          <input
            v-else-if="currentQuestion.type === 'number'"
            type="number"
            v-model="userAnswer"
            class="wide-input"
            placeholder="Enter a number..."
          />
        </div>

        <!-- Submit Button Below Input -->
        <button
          @click="submitAnswer"
          :disabled="!userAnswer && !selectedOption"
          class="submit-btn"
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

      <!-- Admin Access Button (Now at the Bottom) -->
      <button class="admin-access-btn" @click="togglePasswordPrompt">
        Admin Access
      </button>

      <!-- Password Input (Now Below the Button) -->
      <div v-if="showPasswordPrompt" class="password-panel">
        <label class="text">Enter Admin Password:</label>
        <input type="password" v-model="adminPassword" class="password-input" />
        <button class="password-submit-btn" @click="checkPassword">
          Submit Password
        </button>
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
    const logo = ref("/logo2.png");
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
    const isEditing = ref(false); // ✅ Track if user is in edit mode
    const editingQuestionId = ref(null); // ✅ Stores ID of the question being edited

    const bgColor = ref("#212121");
    const newBgColor = ref("#212121");

    const fontSize = ref(16);
    const newFontSize = ref(16);

    const fontFamily = ref("Courier New, sans-serif");
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

      if (answer) {
        if (isEditing.value && editingQuestionId.value !== null) {
          // ✅ Editing mode: update the existing answer
          const index = answers.value.findIndex(
            (a) => a.question.id === editingQuestionId.value
          );
          if (index !== -1) {
            answers.value[index].answer = answer; // Update the answer
          }

          // ✅ Reset editing mode and stop question flow
          isEditing.value = false;
          editingQuestionId.value = null;
          currentQuestionIndex.value = null;
          generateSolution(); // ✅ Recalculate the solution immediately
        } else {
          // ✅ Normal answering flow
          answers.value.push({
            question: currentQuestions.value[currentQuestionIndex.value],
            answer,
          });

          if (currentQuestionIndex.value < currentQuestions.value.length - 1) {
            currentQuestionIndex.value++; // Move to next question
          } else {
            currentQuestionIndex.value = null; // End question flow
            generateSolution(); // ✅ Always reevaluate the solution
          }
        }
      }

      // Reset inputs
      userAnswer.value = "";
      selectedOption.value = "";
    }

    function editAnswer(questionId) {
      // ✅ Enable edit mode and isolate the specific question
      isEditing.value = true;
      editingQuestionId.value = questionId;

      // ✅ Set the current question to the one being edited
      currentQuestionIndex.value = currentQuestions.value.findIndex(
        (q) => q.id === questionId
      );
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
      editingQuestionId,
      isEditing,
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
  padding: 8px;
  overflow-y: auto;
}

.widget-header {
  display: flex;
  justify-content: space-between; /* Title on left, logo on right */
  align-items: center;
  background: var(--primary-color);
  padding: 15px;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
  height: 20%; /* ✅ Ensure header height */
  width: 100%;
  box-sizing: border-box;
}

/* Add spacing between the header and the first question */
.question-container {
  margin-top: 15px; /* ✅ Adds space between header and question */
  padding: 10px;
  width: 100%;
}
/* Header Title */
.widget-title {
  font-size: 2rem; /* ✅ Adjust size for readability */
  font-weight: bold;
  color: var(--text-color);
}

/* Smaller Header Logo */
.logo-header {
  width: 64px; /* ✅ Slightly bigger for visibility */
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

/* Ensures all selectable images are the same size */
.question-image {
  width: 30%; /* ✅ Image takes 80% of the container width */
  max-width: 150px; /* ✅ Prevents images from becoming too large */
  height: auto; /* ✅ Maintain aspect ratio */
  object-fit: cover; /* ✅ Ensures the image fills the space properly */
  border-radius: 6px; /* ✅ Optional: Rounds corners for a cleaner look */
  cursor: pointer;
  margin: 5px;
  transition: transform 0.2s ease-in-out;
}

.question-image:hover {
  transform: scale(1.05); /* ✅ Slight zoom effect on hover */
}

.image-options {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px; /* ✅ Adds spacing between images */
}

/* Admin Access Button */
.admin-access-btn {
  width: 100%; /* ✅ Full width */
  background-color: var(--primary-color);
  color: white;
  padding: 10px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  text-align: center;
  margin-top: 10px;
}

.admin-access-btn:hover {
  background: var(--secondary-color);
}

/* Password Panel (Now Below Button, Requires Scrolling) */
.password-panel {
  margin-top: 20px; /* ✅ Adds space between button and input */
  padding: 15px;
  background: var(--secondary-color);
  border-radius: 6px;
}

.password-input {
  width: 100%;
  padding: 8px;
  margin-top: 5px;
  background: var(--primary-color);
  color: white;
  border: 1px solid #555;
  border-radius: 5px;
}

.password-submit-btn {
  margin-top: 10px;
  padding: 8px;
  width: 100%;
  background: var(--primary-color);
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.password-submit-btn:hover {
  background: var(--secondary-color);
}

.answer-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0 10px; /* Adds even padding on both sides */
  box-sizing: border-box;
}

/* Wider Input Box (Like 2nd Image) */
.wide-input {
  width: 90%; /* ✅ Expands across almost the full width */
  padding: 10px;
  border-radius: 6px;
  border: 1px solid #555;
  background: var(--secondary-color);
  color: white;
}

/* Wider Text Buttons for Enum Type Answers */
.wide-answer-button {
  width: 90%;
  padding: 10px;
  margin: 5px 0;
  border-radius: 6px;
  background: var(--primary-color);
  color: white;
  border: none;
  cursor: pointer;
  text-align: center;
}

.wide-textarea {
  width: 90%; /* Ensures it expands almost fully */
  height: 60px;
  padding: 10px; /* Keeps consistent padding */
  border-radius: 6px;
  border: 1px solid #555;
  background: var(--secondary-color);
  color: white;
  resize: none;
  box-sizing: border-box; /* Ensures padding doesn’t affect width */
  text-align: left; /* Aligns text to the left */
}

.wide-answer-button:hover {
  background: var(--secondary-color);
}

/* Submit Button Below Input */
.submit-btn {
  width: 30%;
  padding: 10px;
  margin-top: 10px;
  border-radius: 6px;
  background: var(--primary-color);
  color: white;
  border: none;
  cursor: pointer;
}

.submit-btn:hover {
  background: var(--secondary-color);
}
</style>
