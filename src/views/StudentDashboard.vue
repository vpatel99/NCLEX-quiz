<template>
    <div class="student">
        <div class="content">
            <!-- name pulled from store to display -->
            <h1 class="name">
                Welcome {{ this.$store.state.user.name }}. Here is how you have
                done in your past quizzes:
            </h1>
            <!-- display past quiz scores -->
            <div class="scores" v-if="scores.length !== 0">
                <div class="centered" v-for="item in scores" :key="item.id">
                    <p class="student__score__section">
                        {{ item.title }}
                    </p>
                    <n-progress
                        type="line"
                        :percentage="item.score"
                        :indicator-placement="'inside'"
                    />
                </div>
            </div>
            <div v-else>
                <div class="centered">
                    <p class="empty">You haven't taken any quizzes yet!</p>
                </div>
            </div>
            <!-- show available quizzes to take for user -->
            <QuizzesContainer :quizzes="quizzes"></QuizzesContainer>
        </div>
    </div>
</template>

<script>
import { NProgress } from "naive-ui";
import { ref } from "vue";
import { supabase } from "../supabase/init";
import { useStore } from "vuex";
import { computed } from "vue";
import QuizzesContainer from "../components/QuizzesContainer.vue";

export default {
    name: "StudentDashboard",
    components: {
        NProgress,
        QuizzesContainer,
    },
    setup() {
        const store = useStore();
        const quizzes = ref([]);
        const scores = ref([]);
        const question = ref([]);
        const dataLoaded = ref(null);
        const count = computed(() => store.state.user);
        const getData = async () => {
            //get users past scores from database, saveed to scores variable
            try {
                let { data: score, error } = await supabase
                    .from("scores")
                    .select("*")
                    .eq("user", count.value.uid);
                if (error) throw error;
                const completed_quizzes = score.map((x) => x.quiz);
                scores.value = score;

                let { data: quiz, error2 } = await supabase
                    .from("quiz")
                    .select("*")
                    .not("quiz_id", "in", `(${completed_quizzes})`);
                if (error) throw error2;
                quizzes.value = quiz;
                dataLoaded.value = true;
            } catch (error) {
                console.warn(error.message);
            }
        };
        getData();
        return { count, quizzes, scores, dataLoaded, question };
    },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.student {
    color: black;
    text-align: center;
}
.content {
    background: white;
    padding-left: 10%;
    padding-right: 10%;
}
.student__score__section {
    font-size: 1.5rem;
}
.student__score {
    font-weight: 700;
}
.empty {
    text-align: center;
}
.centered {
    justify-content: center;
    align-items: center;
    text-align: left;
    padding-bottom: 2%;
    animation: fadein 1s;
}
.scores {
    padding: 5%;
    box-shadow: 0px 0px 7px rgba(0, 0, 0, 0.25);
    margin-top: 3%;
    margin-bottom: 3%;
}
.name {
  -webkit-animation: fadein 1s linear; /* Safari, Chrome and Opera > 12.1 */
       -moz-animation: fadein 1s; /* Firefox < 16 */
        -ms-animation: fadein 1s; /* Internet Explorer */
         -o-animation: fadein 1s; /* Opera < 12.1 */
            animation: fadein 1s;
}
.quizzes {
    -webkit-animation: fadein 1s linear; /* Safari, Chrome and Opera > 12.1 */
       -moz-animation: fadein 1s linear; /* Firefox < 16 */
        -ms-animation: fadein 1s linear; /* Internet Explorer */
         -o-animation: fadein 1s linear; /* Opera < 12.1 */
            animation: fadein 1s linear;
}
</style>
