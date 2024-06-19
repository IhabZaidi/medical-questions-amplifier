<template>
  <div class="flex p-4">
    <!-- left panel -->
    <div class="w-1/2 p-4">
      <div class="mb-6">
        <label class="block text-sm font-medium text-gray-700" for="question">The question</label>
        <div class="relative">
          <textarea
            v-model="form.question"
            id="question"
            class="mt-1 block w-3/5 h-20 shadow-sm sm:text-sm border border-gray-300 rounded-md resize-none"
          ></textarea>
          <button
            @click="startRecognition('question')"
            class="absolute inset-y-0 right-0 flex items-center px-3 text-gray-500"
          >
            🎤
          </button>
        </div>
      </div>

      <div class="mb-6">
        <label class="block text-sm font-medium text-gray-700" for="correct-answer">Correct answer</label>
        <div class="relative">
          <textarea
            v-model="form.correctAnswer"
            id="correct-answer"
            class="mt-1 block w-3/5 h-20 shadow-sm sm:text-sm border border-gray-300 rounded-md resize-none"
          ></textarea>
          <button
            @click="startRecognition('correctAnswer')"
            class="absolute inset-y-0 right-0 flex items-center px-3 text-gray-500"
          >
            🎤
          </button>
        </div>
      </div>

      <div v-for="(_, index) in form.wrongAnswers" :key="index" class="mb-6">
        <label class="block text-sm font-medium text-gray-700" :for="'wrong-answer-' + index">Wrong answer {{ index + 1 }}</label>
        <div class="relative">
          <textarea
            v-model="form.wrongAnswers[index]"
            :id="'wrong-answer-' + index"
            class="mt-1 block w-3/5 h-20 shadow-sm sm:text-sm border border-gray-300 rounded-md resize-none"
          ></textarea>
          <button
            @click="startRecognition(`wrongAnswers[${index}]`)"
            class="absolute inset-y-0 right-0 flex items-center px-3 text-gray-500"
          >
            🎤
          </button>
        </div>
      </div>

      <div class="mb-6">
        <label class="block text-sm font-medium text-gray-700" for="comment">Comment</label>
        <div class="relative">
          <textarea
            v-model="form.comment"
            id="comment"
            class="mt-1 block w-3/5 h-20 shadow-sm sm:text-sm border border-gray-300 rounded-md resize-none"
          ></textarea>
          <button
            @click="startRecognition('comment')"
            class="absolute inset-y-0 right-0 flex items-center px-3 text-gray-500"
          >
            🎤
          </button>
        </div>
      </div>

      <div class="flex justify-end space-x-6">
        <button @click="toggleLanguage" class="bg-blue-500 text-white px-4 py-2 rounded-md">
          Switch to {{ language === 'en' ? 'French' : 'English' }}
        </button>
        <button @click="addToJSON" class="bg-blue-500 text-white px-4 py-2 rounded-md">
          Add to JSON
        </button>
      </div>
    </div>

    <!-- right panel -->
    <div class="w-1/2 p-4">
      <div class="flex justify-end mb-4">
        <button @click="copyJSON" class="bg-green-500 text-white px-4 py-2 rounded-md">
          Copy JSON
        </button>
      </div>
      <textarea
        v-model="jsonOutput"
        readonly
        class="w-full h-96 p-2 border border-gray-300 rounded-md resize-none"
      ></textarea>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, reactive, ref, onMounted } from 'vue';

export default defineComponent({
  name: 'SpeechInputForm',
  setup() {
    const form = reactive({
      question: '',
      correctAnswer: '',
      wrongAnswers: ['', '', ''],
      comment: '',
    });

    const jsonCollection = ref<any>([]);
    const jsonOutput = ref('');

    const language = ref('en');
    let recognition: SpeechRecognition;

    const initializeRecognition = () => {
      const Recognition = (window as any).SpeechRecognition || (window as any).webkitSpeechRecognition;
      recognition = new Recognition();
      recognition.continuous = false;
      recognition.interimResults = false;
      recognition.lang = language.value === 'en' ? 'en-US' : 'fr-FR';

      recognition.onresult = (event: SpeechRecognitionEvent) => {
        const transcript = event.results[0][0].transcript;
        if (currentField.value) {
          if (currentField.value.startsWith('wrongAnswers')) {
            const index = parseInt(currentField.value.match(/\d+/)?.[0] ?? '0');
            form.wrongAnswers[index] = transcript;
          } else {
            (form as any)[currentField.value] = transcript;
          }
        }
      };

      recognition.onerror = (event: any) => {
        console.error('Speech recognition error', event.error);
      };
    };

    const currentField = ref<string | null>(null);

    const startRecognition = (field: string) => {
      if (recognition) {
        currentField.value = field;
        recognition.start();
      }
    };

    const toggleLanguage = () => {
      language.value = language.value === 'en' ? 'fr' : 'en';
      if (recognition) {
        recognition.lang = language.value === 'en' ? 'en-US' : 'fr-FR';
      }
    };

    const addToJSON = () => {
      const newEntry = {
        question: form.question,
        answer: form.correctAnswer,
        suggestions: form.wrongAnswers,
        comment: form.comment,
      };
      jsonCollection.value.push(newEntry);
      jsonOutput.value = JSON.stringify(jsonCollection.value, null, 2);
      // Clear form fields after adding to JSON
      form.question = '';
      form.correctAnswer = '';
      form.wrongAnswers = ['', '', ''];
      form.comment = '';
    };

    const copyJSON = () => {
      navigator.clipboard.writeText(jsonOutput.value).then(
        () => {
          alert('JSON copied to clipboard!');
        },
        (err) => {
          console.error('Could not copy JSON: ', err);
        }
      );
    };

    onMounted(() => {
      initializeRecognition();
    });

    return {
      form,
      language,
      startRecognition,
      toggleLanguage,
      addToJSON,
      jsonOutput,
      copyJSON,
    };
  },
});
</script>
