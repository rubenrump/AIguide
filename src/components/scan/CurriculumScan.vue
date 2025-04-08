<template>
    <div class="p-4 bg-yellow-100 text-yellow-900 font-bold mb-4 rounded">
  ✅ CurriculumScan component is zichtbaar
</div>

    <div class="container mx-auto px-4 py-8">
      <p class="text-xs text-gray-500">[debug] showIntro: {{ showIntro }} | showResults: {{ showResults }}</p>
  
      <!-- Intro-sectie -->
      <div v-if="showIntro" class="bg-white p-6 rounded-xl shadow">
        <h2 class="text-xl font-bold mb-4">🧪 CurriculumScan Handleiding</h2>
        <p class="mb-4">Deze tool helpt je toetsen en opdrachten te beoordelen op AI-proofheid en jouw rol als docent.</p>
        <ul class="list-disc list-inside mb-4">
          <li>Kies een opdracht, toets of lesactiviteit</li>
          <li>Beantwoord 10 reflectievragen</li>
          <li>Krijg een beoordeling + tips</li>
        </ul>
        <div class="mt-6 text-right">
          <button @click="startScan" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">➡️ Start scan</button>
        </div>
      </div>
  
      <!-- Vragenlijst -->
      <div v-else-if="!showResults">
        <h2 class="text-2xl font-bold mb-6">🔍 Start de CurriculumScan</h2>
        <form @submit.prevent="calculateResults">
          <div v-for="(question, index) in questions" :key="index" class="mb-6">
            <label class="block font-semibold text-gray-800 mb-2">{{ index + 1 }}. {{ question.text }}</label>
            <div class="space-x-4">
              <label><input type="radio" :name="'q' + index" value="Ja" v-model="question.answer" /> Ja</label>
              <label><input type="radio" :name="'q' + index" value="Nee" v-model="question.answer" /> Nee</label>
              <label><input type="radio" :name="'q' + index" value="Twijfel" v-model="question.answer" /> Twijfel</label>
            </div>
            <textarea v-model="question.note" class="mt-2 w-full p-2 border rounded" rows="2" placeholder="Opmerking of reflectie..."></textarea>
          </div>
          <button type="submit" class="bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700">📊 Bekijk resultaat</button>
        </form>
      </div>
  
      <!-- Resultaten -->
      <div v-else>
        <h2 class="text-2xl font-bold mb-4">📊 Resultaat van de Scan</h2>
        <p class="mb-4">Hieronder zie je jouw reflectie samengevat.</p>
  
        <ul ref="resultRef" class="mb-6 space-y-2">
          <li v-for="(question, index) in questions" :key="index">
            <p><strong>{{ index + 1 }}. {{ question.text }}</strong><br />
            Antwoord: <span class="font-semibold">{{ question.answer || 'Geen antwoord' }}</span><br />
            <em v-if="question.note">Notitie: {{ question.note }}</em></p>
          </li>
        </ul>
  
        <div class="bg-gray-100 p-4 rounded-xl">
          <p class="mb-2 font-semibold">📌 Algemene reflectie:</p>
          <ul class="list-disc list-inside text-sm">
            <li v-if="yesCount >= 7">✔️ Je ontwerp toont sterke menselijke en educatieve meerwaarde. Overweeg AI bewust in te zetten als ondersteuning.</li>
            <li v-else-if="yesCount >= 4">⚠️ Je ontwerp is deels AI-proof. Herzie onderdelen die AI makkelijk kan overnemen.</li>
            <li v-else>❗Er is veel risico op AI-overname of beperkte leerwaarde. Overweeg herontwerp.</li>
          </ul>
        </div>
  
        <div class="mt-6 space-x-4">
          <button @click="resetScan" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">🔁 Nieuwe scan starten</button>
          <button @click="downloadPDF" class="bg-gray-800 text-white px-4 py-2 rounded hover:bg-gray-900">⬇️ Download als PDF</button>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  
  const showIntro = ref(true);
  const showResults = ref(false);
  const yesCount = ref(0);
  const resultRef = ref(null);
  
  const questions = ref([
    { text: 'Kan AI deze opdracht of toets deels of geheel overnemen in voorbereiding of beoordeling?', answer: '', note: '' },
    { text: 'Zou AI dit werk sneller of efficiënter kunnen uitvoeren dan jij als docent?', answer: '', note: '' },
    { text: 'Zet je AI bewust in bij deze opdracht of toets?', answer: '', note: '' },
    { text: 'Is deze activiteit duurzaam verantwoord qua AI-gebruik?', answer: '', note: '' },
    { text: 'Is er ruimte voor eigenheid, keuzes of verrassingen voor de leerling?', answer: '', note: '' },
    { text: 'Is de opdracht of toets direct gekoppeld aan een duidelijk leerdoel?', answer: '', note: '' },
    { text: 'Is er feedback of formatieve toetsing ingebouwd?', answer: '', note: '' },
    { text: 'Voeg jij als docent iets toe wat AI niet kan?', answer: '', note: '' },
    { text: 'Worden de ethische richtlijnen rondom AI nageleefd?', answer: '', note: '' },
    { text: 'Stimuleert deze opdracht of toets bewustwording over AI zelf?', answer: '', note: '' }
  ]);
  
  function startScan() {
    console.log('✅ startScan triggered');
    alert('Startscan werkt 🎉');
    showIntro.value = false;
    showResults.value = false;
  }
  
  function calculateResults() {
    yesCount.value = questions.value.filter(q => q.answer === 'Ja').length;
    showResults.value = true;
  }
  
  function resetScan() {
    questions.value.forEach(q => {
      q.answer = '';
      q.note = '';
    });
    showResults.value = false;
    showIntro.value = false;
  }
  
  async function downloadPDF() {
    const html2pdf = (await import('html2pdf.js')).default;
    const element = resultRef.value;
    const opt = {
      margin: 0.5,
      filename: 'CurriculumScan-resultaat.pdf',
      image: { type: 'jpeg', quality: 0.98 },
      html2canvas: { scale: 2 },
      jsPDF: { unit: 'in', format: 'letter', orientation: 'portrait' }
    };
    html2pdf().set(opt).from(element).save();
  }
  </script>
  
  <style scoped>
  th, td {
    border-bottom: 1px solid #e5e7eb;
  }
  </style>
  