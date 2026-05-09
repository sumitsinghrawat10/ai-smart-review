<script setup>
import { ref } from 'vue'

const selectedCategory = ref('')
const selectedRating = ref(0)
const generatedReviews = ref([])
const isLoading = ref(false)
const copyStatus = ref(null)

const categoryGroups = [
  {
    group: '🍽️ Food & Drink',
    items: [
      { id: 1, name: 'Restaurant' },
      { id: 2, name: 'Café & Coffee Shop' },
      { id: 3, name: 'Bakery & Desserts' },
      { id: 4, name: 'Street Food & Fast Food' },
    ]
  },
  {
    group: '🏥 Health & Wellness',
    items: [
      { id: 5, name: 'Hospital & Clinic' },
      { id: 6, name: 'Pharmacy' },
      { id: 7, name: 'Gym & Fitness Center' },
      { id: 8, name: 'Salon & Spa' },
    ]
  },
  {
    group: '🎓 Education',
    items: [
      { id: 9, name: 'School & College' },
      { id: 10, name: 'Coaching & Tuition Center' },
      { id: 11, name: 'Online Course & EdTech' },
    ]
  },
  {
    group: '✈️ Travel & Stay',
    items: [
      { id: 12, name: 'Hotel & Resort' },
      { id: 13, name: 'Travel Agency' },
      { id: 14, name: 'Airlines & Transport' },
    ]
  },
  {
    group: '🛍️ Shopping & Lifestyle',
    items: [
      { id: 15, name: 'Clothing & Fashion Store' },
      { id: 16, name: 'Electronics & Tech Store' },
      { id: 17, name: 'Supermarket & Grocery' },
      { id: 18, name: 'Online Shopping' },
    ]
  },
  {
    group: '🏦 Services',
    items: [
      { id: 19, name: 'Bank & Financial Service' },
      { id: 20, name: 'Real Estate & Housing' },
      { id: 21, name: 'Repair & Home Service' },
      { id: 22, name: 'Government & Public Service' },
    ]
  },
]

const setRating = (n) => {
  selectedRating.value = selectedRating.value === n ? 0 : n
}

const generateAIReview = async () => {
  const apiKey = "AIzaSyBYCvSN_OsF9a1F9vE7HKHdc6m4tf4ydkM"
  const model = "gemini-2.5-flash"
  const url = `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`

  const starsText = selectedRating.value
    ? `${selectedRating.value}-star`
    : 'unrated'

  const payload = {
    contents: [{
      parts: [{
        text: `You are a review generator for a ${selectedCategory.value} business.

Generate exactly 3 realistic customer reviews for a ${starsText} rating experience.

Rules:
- Each review must be 2-4 sentences and under 150 words
- Make each unique in tone: one brief and to-the-point, one detailed, one emotional
- Match tone to rating: negative/frustrated for 1-2 stars, mixed/neutral for 3, happy/enthusiastic for 4-5 stars
- Write about general experiences only: atmosphere, service quality, value for money, cleanliness, wait time, staff behavior — relevant to the category
- Do NOT use any made-up staff names or specific product/menu item names
- Sound like a real human — use casual language and occasional fillers like "honestly", "tbh", "really"
- Never use generic phrases like "great experience" or "highly recommend" without explaining WHY
- Return ONLY a valid JSON array of 3 strings. No markdown, no code fences, no numbering, no explanation.

Example format: ["Review one here.", "Review two here.", "Review three here."]`
      }]
    }]
  }

  isLoading.value = true
  generatedReviews.value = []

  try {
    const response = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    })

    if (!response.ok) {
      const errorData = await response.json()
      throw new Error(`API Error: ${response.status} - ${errorData.error?.message || 'Unknown'}`)
    }

    const data = await response.json()
    const rawText = data.candidates?.[0]?.content?.parts?.[0]?.text || '[]'
    const cleaned = rawText.replace(/```json|```/g, '').trim()
    generatedReviews.value = JSON.parse(cleaned)

  } catch (err) {
    console.error('Fetch failed:', err)
    generatedReviews.value = [`⚠️ Error: ${err.message}`]
  } finally {
    isLoading.value = false
  }
}

const copyToClipboard = (text, index) => {
  navigator.clipboard.writeText(text)
  copyStatus.value = index
  setTimeout(() => { copyStatus.value = null }, 2000)
}
</script>

<template>
  <div class="container">
    <header>
      <h1>Smart Reviewer</h1>
      <p>AI-powered review generator✨</p>
    </header>

    <div class="card">
      <div class="form-group">
        <label>Choose Category</label>
        <select v-model="selectedCategory" class="custom-select">
          <option disabled value="">Select Business Type...</option>
          <optgroup v-for="group in categoryGroups" :key="group.group" :label="group.group">
            <option v-for="cat in group.items" :key="cat.id" :value="cat.name">
              {{ cat.name }}
            </option>
          </optgroup>
        </select>
      </div>

      <div class="star-rating">
        <p>Your Experience <span class="star-label">({{ selectedRating > 0 ? selectedRating + ' Stars' : 'Not rated' }})</span></p>
        <div class="stars">
          <span
            v-for="n in 5"
            :key="n"
            @click="setRating(n)"
            class="star"
            :class="{ active: n <= selectedRating }"
          >
            {{ n <= selectedRating ? '★' : '☆' }}
          </span>
        </div>
      </div>

      <button @click="generateAIReview" :disabled="isLoading || !selectedCategory" class="gen-btn">
        <span v-if="isLoading" class="loader"></span>
        {{ isLoading ? 'Generating...' : '✨ Generate Reviews' }}
      </button>
    </div>

    <div v-if="generatedReviews.length > 0" class="reviews-list">
      <h3>Pick a review to copy:</h3>
      <div v-for="(review, index) in generatedReviews" :key="index" class="review-card">
        <p>{{ review }}</p>
        <button @click="copyToClipboard(review, index)" class="copy-btn">
          {{ copyStatus === index ? '✅ Copied!' : '📋 Copy' }}
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
* { font-family: 'Poppins', sans-serif; }

.container {
  padding: 10px 0px;
  max-width: 560px;
  margin: 0 auto;
  color: #2c3e50;
}

header {
  text-align: center;
  margin-bottom: 30px;
}

h1 {
  margin: 0;
  font-size: 1.8rem;
  font-weight: 700;
  color: #42b883;
}

header p {
  margin: 4px 0 0;
  font-size: 0.9rem;
  color: #94a3b8;
  font-weight: 400;
}

.card {
  background: white;
  padding: 28px;
  border-radius: 16px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.07);
}

.form-group {
  margin-bottom: 22px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  font-size: 0.88rem;
  color: #475569;
}

.custom-select {
  width: 100%;
  padding: 12px 14px;
  border: 2px solid #e2e8f0;
  border-radius: 10px;
  outline: none;
  font-family: 'Poppins', sans-serif;
  font-size: 0.9rem;
  color: #2c3e50;
  background: #f8fafc;
  transition: border-color 0.2s;
  cursor: pointer;
}

.custom-select:focus {
  border-color: #42b883;
  background: white;
}

.star-rating {
  text-align: center;
  margin-bottom: 26px;
}

.star-rating p {
  margin: 0 0 10px;
  font-size: 0.9rem;
  font-weight: 500;
  color: #475569;
}

.star-label {
  color: #42b883;
  font-weight: 600;
}

.stars {
  display: flex;
  justify-content: center;
  gap: 6px;
}

.star {
  font-size: 2.6rem;
  cursor: pointer;
  color: #e2e8f0;
  transition: color 0.15s, transform 0.15s;
  user-select: none;
}

.star.active {
  color: #FBBF24;
  transform: scale(1.15);
}

.star:hover {
  transform: scale(1.2);
}

.gen-btn {
  width: 100%;
  background: #42b883;
  color: white;
  padding: 14px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-size: 0.95rem;
  font-family: 'Poppins', sans-serif;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: background 0.2s, transform 0.1s;
}

.gen-btn:hover:not(:disabled) {
  background: #38a16e;
  transform: translateY(-1px);
}

.gen-btn:disabled {
  background: #cbd5e0;
  cursor: not-allowed;
  transform: none;
}

.loader {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(255,255,255,0.4);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  display: inline-block;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.reviews-list {
  margin-top: 30px;
  animation: fadeIn 0.4s ease;
}

.reviews-list h3 {
  font-size: 0.95rem;
  font-weight: 600;
  color: #475569;
  margin-bottom: 14px;
}

.review-card {
  background: #f8fafc;
  padding: 18px 20px;
  border-radius: 12px;
  border-left: 4px solid #42b883;
  margin-bottom: 14px;
  transition: transform 0.2s;
}

.review-card:hover {
  transform: translateX(4px);
}

.review-card p {
  margin: 0 0 12px;
  font-size: 0.88rem;
  line-height: 1.65;
  color: #334155;
}

.copy-btn {
  padding: 6px 14px;
  border: 1.5px solid #e2e8f0;
  border-radius: 6px;
  background: white;
  cursor: pointer;
  font-size: 0.78rem;
  font-family: 'Poppins', sans-serif;
  font-weight: 500;
  color: #475569;
  transition: border-color 0.2s, color 0.2s;
}

.copy-btn:hover {
  border-color: #42b883;
  color: #42b883;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}
</style>