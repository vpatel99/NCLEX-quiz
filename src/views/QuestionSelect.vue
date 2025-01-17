<template>
  <n-config-provider :theme-overrides="this.themeOverrides" class="wrapper">
    <div class="main">
      <div class="content">
        <div class="quiz">
          <h2>Quiz Management</h2>
        </div>
        <br />
        <!-- if creating a new quiz enter quiz name, saved to newQuizName variable -->
        <div class="newQ">
          <h3>Creating a new quiz? Enter quiz name here:</h3>
          <n-input
            v-model:value="newQuizName"
            type="text"
            class="form-field"
            id="newQuizName"
            name="newQuizName"
            :input-props="{ type: 'clearable' }"
            placeholder="Enter New Quiz Name"
          />
          <div class="space"></div>
          <n-button
            @click="createQuiz"
            size="large"
            type="primary"
            color="#fdcb6e"
            text-color="black"
            class="button"
            >Add Quiz</n-button
          >
        </div>
        <br />
        <div class="qType">
          <h3>Creating a new question? Select question type here:</h3>
          <n-radio-group v-model:value="typeSelected" name="radiogroup">
            <!-- if typeSelected===1, multChoice; if typeSelected===2, selAll; etc... -->
            <n-radio :value="1" class="choice-text">Multiple Choice</n-radio>
            <n-radio :value="2" class="choice-text">Select All</n-radio>
            <n-radio :value="3" class="choice-text">DropDown Sentence</n-radio>
            <n-radio :value="4" class="choice-text">DropDown Table</n-radio>
            <n-radio :value="5" class="choice-text">Matrix Table</n-radio>
            <n-radio :value="6" class="choice-text">Highlight Table</n-radio>
          </n-radio-group>
          <div class="space"></div>
          <n-button
            @click="createQuestion"
            size="large"
            type="primary"
            color="#fdcb6e"
            text-color="black"
            class="button"
            >Create Question</n-button
          >
        </div>
      </div>
    </div>
  </n-config-provider>
</template>
<script>
import {
  NButton,
  NInput,
  NRadio,
  NRadioGroup,
  NConfigProvider,
  useMessage,
} from "naive-ui";
import { supabase } from "../supabase/init";

export default {
  name: "QuestionSelect",
  components: {
    NButton,
    NInput,
    NRadio,
    NRadioGroup,
    NConfigProvider,
  },
  props: {
    msg: String,
  },
  setup() {
    const message = useMessage();
    return {
      createSuccessMessage(msg, time) {
        message.success(msg, { duration: time });
      },
      createErrorMessage(msg, time) {
        message.error(msg, { duration: time });
      },
    };
  },
  data() {
    return {
      newQuizName: "",
      typeSelected: 0,
      quizList: {},
      themeOverrides: {
        common: {
          primaryColor: "#FF8C00",
        },
      },
    };
  },
  methods: {
    async createQuiz() {
      //send new quiz name to database and create new data row
      await supabase.from("quiz").insert([{ title: this.newQuizName }]);

      this.createSuccessMessage("Success! New quiz was created!", 10000);
    },
    //get list of quizzes to pass to newQ page as prop
    async createQuestion() {
      try {
        let { data: quiz, error } = await supabase.from("quiz").select("*");
        if (error) throw error;
        this.quizList = quiz.map((obj) => ({
          label: obj.title,
          value: obj.quiz_id,
        }));
      } catch (error) {
        console.log(error.message);
      }

      let { data: qids, error } = await supabase.from("quiz").select("*");
      if (error) {
        this.createErrorMessage(error.message, 10000);
      }
      this.$store.commit("SET_QUIZ", qids);
      //route to correct page based on typeSelected value
      switch (this.typeSelected) {
        case 0:
          this.createErrorMessage("Please select a question type.");
          break;
        case 1:
          // Multiple Choice
          this.$router.push({ name: "NewMultChoice" });
          break;
        case 2:
          // Select All
          this.$router.push({ name: "NewSelectAll" });
          break;
        case 3:
          // DropDown Sentence
          this.$router.push({ name: "NewDDS" });
          break;
        case 4:
          // DropDown Table
          this.$router.push({ name: "NewDDT" });
          break;
        case 5:
          // Matrix Table
          this.$router.push({ name: "NewMatrix" });
          break;
        case 6:
          // Highlight
          this.$router.push({ name: "NewHighlight" });
          break;
      }
    },
  },
};
</script>

<style scoped>
.main {
  font-family: "Montserrat";
}
.top {
  background-color: #0094ff;
  padding-top: 5%;
  padding-bottom: 3%;
}
.content {
  background: white;
  justify-content: center;
  padding: 10px;
  padding-top: 3%;
  padding-left: 10%;
  padding-right: 10%;
  width: auto;
  margin: auto;
  margin-bottom: 15px;
  align-content: center;
  text-align: center;
}
.quiz {
  text-align: center;
  padding: 10px;
  margin-right: auto;
  margin-left: auto;
  width: 50%;
  background: linear-gradient(
    172.4deg,
    #24a3ff 5.89%,
    #24a3ff 5.9%,
    #0038ff 91.52%
  );
  border-radius: 5%;
  box-shadow: 10px 10px 5px #cac9c9;
}
.n-radio {
  border: 1px #808080 solid;
  box-shadow: 10px 10px 5px #cac9c9;
  border-radius: 10px;
  width: 72vw;
  padding: 20px 1.5vw;
  margin-top: 10px;
}
h2 {
  color: white;
  font-weight: 900;
  text-align: center;
  padding: 10px;
  margin: 5px;
  margin-left: 10%;
  margin-right: 10%;
}
h3 {
  text-align: center;
}
.n-button {
  background-color: #ffc633;
}
h1 {
  color: white;
}
table {
  border: 1px solid black;
  margin-left: auto;
  margin-right: auto;
  margin-top: 15px;
  padding: 5px;
}
th,
td {
  border: 1px solid black;
  width: 20%;
  margin-left: auto;
  margin-right: auto;
  padding: 5px;
}

.form-field {
  width: 75%;
}

router-link,
a {
  color: rgb(75, 85, 97);
  text-decoration: underline;
}

.space {
  margin-bottom: 20px;
}
@import url(https://fonts.googleapis.com/css?family=Montserrat);
</style>
