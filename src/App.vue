<template>
	<ScoreBoard :winCount="this.winCount" :loseCount="this.loseCount" />
	<template v-if="this.question">
		<h1 v-html="this.question"></h1>
		<template v-for="(answer, index) in this.answers" :key="index">
			<input
				type="radio"
				name="options"
				:value="answer"
				v-model="this.chosenAnswer"
				:disabled="this.answerSubmitted"
			/>
			<label v-html="answer"></label>
			<br />
		</template>
		<button
			class="send"
			type="button"
			@click="this.submitAnswer"
			v-if="!this.answerSubmitted"
		>
			Send
		</button>
		<section v-if="this.answerSubmitted" class="result">
			<template v-if="this.chosenAnswer == this.correctAnswer">
				<h4>
					&#9989; Congratulations, the answer "{{
						this.correctAnswer
					}}" is correct.
				</h4>
			</template>
			<template v-else>
				<h4>
					&#10060; I'm sorry, you picked the wrong answer. The correct
					is "{{ this.correctAnswer }}".
				</h4>
			</template>
			<button @click="this.getNewQuestion()" class="send" type="button">
				Next question
			</button>
		</section>
	</template>
</template>

<script>
import ScoreBoard from '@/components/ScoreBoard.vue';
export default {
	name: 'App',
	components: {
		ScoreBoard,
	},
	data() {
		return {
			question: undefined,
			correctAnswer: undefined,
			incorrectAnswers: undefined,
			chosenAnswer: undefined,
			answerSubmitted: false,
			winCount: 0,
			loseCount: 0,
		};
	},
	methods: {
		submitAnswer: function () {
			if (!this.chosenAnswer) {
				alert('Please select an answer');
				return;
			}
			this.answerSubmitted = true;
			if (this.correctAnswer === this.chosenAnswer) {
				this.winCount++;
				localStorage.setItem('winCount', this.winCount);
			} else {
				localStorage.setItem('loseCount', this.loseCount);
			}
		},
		getNewQuestion: function () {
			this.answerSubmitted = false;
			this.question = this.chosenAnswer = undefined;
			this.axios
				.get('https://opentdb.com/api.php?amount=1&category=15')
				.then((response) => {
					this.question = response.data.results[0].question;
					this.correctAnswer =
						response.data.results[0].correct_answer;
					this.incorrectAnswers =
						response.data.results[0].incorrect_answers;
				});
		},
	},
	computed: {
		answers() {
			const answers = [...this.incorrectAnswers];
			answers.splice(
				Math.round(Math.random() * answers.length),
				0,
				this.correctAnswer
			);
			return answers;
		},
	},
	created() {
		this.getNewQuestion();
		if (localStorage.getItem('winCount')) {
			this.winCount = Number(localStorage.getItem('winCount'));
		}
		if (localStorage.getItem('loseCount')) {
			this.loseCount = Number(localStorage.getItem('loseCount'));
		}
	},
};
</script>

<style lang="scss">
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin: 60px auto;
	max-width: 960px;
}

h1 {
	margin-top: 40px;
}

input[type='radio'] {
	margin: 12px 4px;
}

button.send {
	margin-top: 12px;
	height: 40px;
	min-width: 120px;
	padding: 0 16px;
	color: #fff;
	background-color: #1867c0;
	border: 1px solid #1867c0;
	cursor: pointer;
}

section.score {
	border-bottom: 1px solid black;
	padding: 24px;
	font-size: 18px;

	span {
		padding: 8px;
		font-weight: bold;
		border: 1px solid black;
	}
}
</style>
