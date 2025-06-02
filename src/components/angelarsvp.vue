<template>
    <div class="container" role="main" aria-label="18th Birthday RSVP Invitation">
      <h1 class="title">You're Invited</h1>
      <br>
      <h2 class="subtitle">To Celebrate Angela’s 18th Birthday</h2>
      <p class="invitation-text">Join us in celebrating the joy and happiness on ANGELA's special day! We cordially invite you to be a part of a celebration filled with laughter, hugs, and unforgettable moments.</p>
  
      <form @submit.prevent="handleSubmit" aria-label="RSVP Form">
        <label for="name">Your Name:</label>
        <input type="text" id="name" v-model="name" placeholder="Enter your full name" required autocomplete="off" @input="handleNameInput" />
  
        <label>Will you attend?</label>
        <div class="radio-group" role="radiogroup" aria-labelledby="attendanceLabel">
          <label>
            <input @click="handleAttendanceChange('Yes')" type="radio" v-model="attendance" value="Yes" required /> Yes
          </label>
          <label>
            <input @click="handleAttendanceChange('No')" type="radio" v-model="attendance" value="No" /> No
          </label>
        </div>
  
        <!-- <p v-if="isEighteenthCandle && attendance === 'Yes'" class="invitation-text dialog-special-guest">
          You are one of the 18 candles, {{ eighteenthCandleName }}!
        </p> -->

        <p v-if="personalInfo.position === 'One of the 18th candles' && attendance === 'Yes'"
          class="invitation-text dialog-special-guest">
          You are one of the 18th candles, {{ eighteenthCandleName }}!
        </p>

        <p v-else-if="personalInfo.position === 'One of the 18th Roses' && attendance === 'Yes'"
          class="invitation-text dialog-special-guest">
          You are one of the 18th Roses, {{ eighteenthRosesName }}!
        </p>

        <p v-else-if="personalInfo.position === 'One of the 18th Shots' && attendance === 'Yes'"
          class="invitation-text dialog-special-guest">
          You are one of the 18th Shots, {{ eighteenthShotsName }}!
        </p>

        <p v-else-if="personalInfo.position === 'One of the 18th candles and 18th Roses' && attendance === 'Yes'"
          class="invitation-text dialog-special-guest">
          You are one of the 18th candles and 18th Roses, {{ eighteenthCandleName || eighteenthRosesName }}!
        </p>

        <p v-else-if="personalInfo.position === 'One of the 18th Shot and 18th Roses' && attendance === 'Yes'"
          class="invitation-text dialog-special-guest">
          You are One of the 18th Shot and 18th Roses, {{ eighteenthShotsName || eighteenthRosesName }}!
        </p>

        <p v-else-if="attendance === 'Yes' && personalInfo.position === ''" class="invitation-text">
            Welcome to the party!
        </p>

  
        <p class="invitation-text">Bring your enthusiasm and festive spirit! We hope you can make it and celebrate with us.</p>
  
        <!-- <button @click="savePersonalInfo" type="submit" aria-label="Submit RSVP">Submit RSVP</button> -->
        <button type="submit" aria-label="Submit RSVP">Submit RSVP</button>
      </form>
      <div v-if="thankYouMessage" class="thankyou-message" role="alert" aria-live="polite">
        {{ thankYouMessage }}
      </div>
    </div>
  </template>
  
  <script setup lang="ts">
import { onMounted, reactive, ref, computed } from 'vue';
import { doc, setDoc, collection, getDocs } from 'firebase/firestore';
import { db } from '@/firebase';

// Reactive state for form inputs
const name = ref<string>('');
const attendance = ref<string>('');
const thankYouMessage = ref<string>('');

// 18th Candel
const eighteenCandel = ref<{ name: string; value: string }[]>([
  { name: 'Ms. Angelica Aquino', value: 'Angelica Aquino' },
  { name: 'Mrs. Marlyn Aquino', value: 'Marlyn Aquino' },
  { name: 'Ms. Precy Aspiras', value: 'Precy Aspiras' },
  { name: 'Ms. Jessica Aspiras', value: 'Jessica Aspiras' },
  { name: 'Mrs. Luisa Canales', value: 'Luisa Canales' },
  { name: 'Ms. Lourie Kay Cincua', value: 'Lourie Kay Cincua' },
  { name: 'Mrs. Liezel Cincua', value: 'Liezel Cincua' },
  { name: 'Ms. Jenifer Aytona', value: 'Jenifer Aytona' },
  { name: 'Mrs. Clarissa Dionaldo', value: 'Clarissa Dionaldo' },
  { name: 'Mrs. Charmie De Torres', value: 'Charmie De Torres' },
  { name: 'Mrs. Joan Magno', value: 'Joan Magno' },
  { name: 'Mrs. Glenda Robriguez', value: 'Glenda Robriguez' },
  { name: 'Ms. Mila Ansagay', value: 'Mila Ansagay' },
  { name: 'Dra. Weng Catanaoan', value: 'Weng Catanaoan' },
  { name: 'Mrs. Rowena Datinginoo', value: 'Rowena Datinginoo' },
  { name: 'Mr. Ma. Zarah Muli', value: 'Zarah Muli' },
  { name: 'Mrs. Thelma Muli', value: 'Thelma Muli' },
  { name: 'Mr. Angelo Muli', value: 'Angelo Muli' },
]);

// 18th Roses
const eighteeRoses = ref<{ name: string; value: string }[]>([
  { name: 'Mr. Arjay magno', value: 'Arjay magno' },
  { name: 'Mr. Reynaldo Aquino', value: 'Reynaldo Aquino' },
  { name: 'Mr. Renz Gonzales', value: 'Renz Gonzales' },
  { name: 'Mr. Ronald Villo', value: 'Ronald Villo' },
  { name: 'Mr. Efren Rodriguez', value: 'Efren Rodriguez' },
  { name: 'Mr. Ralph Louren Cincua', value: 'Ralph Louren Cincua' },
  { name: 'Mr. Wingard Baraquiel', value: 'Wingard Baraquiel' },
  { name: 'Mr. Jemuel Hisita', value: 'Jemuel Hisita' },
  { name: 'Mr. Rexie Van Galanta', value: 'Rexie Van Galanta' },
  { name: 'Mr. John Lumbis', value: 'John Lumbis' },
  { name: 'Mr. Aaron Josh Corbita', value: 'Aaron Josh Corbita' },
  { name: 'Mr. Marvy James Bayani', value: 'Marvy James Bayani' },
  { name: 'Mr. Jhon Ryan Dellomas', value: 'Jhon Ryan Dellomas' },
  { name: 'Mr. Wilredo Andres', value: 'Wilredo Andres' },
  { name: 'Mr. Prince Charles Giron', value: 'Prince Charles Giron' },
  { name: 'Ms. Jestoni Cabezas', value: 'Jestoni Cabezas' },
  { name: 'Mrs. Maria Thelma Muli', value: 'Maria Thelma Muli' },
  { name: 'Mr. Angelo Muli', value: 'Angelo Muli' },
]);

// 18th Shot
const eighteenShots = ref<{ name: string; value: string }[]>([
  { name: 'Ms. Daphne Asombrado', value: 'Daphne Asombrado' },
  { name: 'Ms. Renoakyle Alcantara', value: 'Renoakyle Alcantara' },
  { name: 'Ms. Chelsy Llorca', value: 'Chelsy Llorca' },
  { name: 'Ms. Brillian Vertudes', value: 'Brillian Vertudes' },
  { name: 'Ms. Jhon Ryan Dellomas', value: 'Jhon Ryan Dellomas' },
  { name: 'Ms. Rhiangelie Sinsoro', value: 'Rhiangelie Sinsoro' },
  { name: 'Ms. Shermaine Leoperio', value: 'Shermaine Leoperio' },
  { name: 'Ms. Angeline Lolin', value: 'Angeline Lolin' },
  { name: 'Ms. Brydhe Ganub', value: 'Brydhe Ganub' },
  { name: 'Ms. Ferleene Monog', value: 'Ferleene Monog' },
  { name: 'Ms. Stephanie Bangit', value: 'Stephanie Bangit' },
  { name: 'Ms. Rexie Van Galanta', value: 'Rexie Van Galanta' },
  { name: 'Ms. Mariel Panganiban', value: 'Mariel Panganiban' },
  { name: 'Ms. Frinces Aron', value: 'Frinces Aron' },
  { name: 'Ms. Queency Basilan', value: 'Queency Basilan' },
  { name: 'Ms. Dhanice Atibula', value: 'Dhanice Atibula' },
  { name: 'Ms. Charline Muello', value: 'Charline Muello' },
  { name: 'Mr. Xyra Gail Camaje', value: 'Xyra Gail Camaje' },
]);

const isEighteenthCandle = ref<boolean>(false);
const eighteenthCandleName = ref<string>('');
const isEighteenthRoses = ref<boolean>(false);
const eighteenthRosesName = ref<string>('');
const isEighteenthShots = ref<boolean>(false);
const eighteenthShotsName = ref<string>('');

// Reactive object to hold the data to be saved to Firebase
const personalInfo = reactive({
  name: '',
  attended: '',
  position: ''
});



const findPerson = () => {
  const inputNameLower = name.value.toLowerCase();

  // Find in both lists independently
  const personAsCandle = eighteenCandel.value.find(r => r.value.toLowerCase().includes(inputNameLower));
  const personAsRose = eighteeRoses.value.find(r => r.value.toLowerCase().includes(inputNameLower));
  const personAsShots = eighteenShots.value.find(r => r.value.toLowerCase().includes(inputNameLower));

  // --- Reset all relevant values first ---
  isEighteenthCandle.value = false;
  eighteenthCandleName.value = '';
  isEighteenthRoses.value = false;
  eighteenthRosesName.value = '';
  isEighteenthShots.value = false;
  eighteenthShotsName.value = '';
  personalInfo.position = ''; // Always reset position

  // --- Determine position and set flags ---
  if (personAsCandle) {
    isEighteenthCandle.value = true;
    eighteenthCandleName.value = personAsCandle.name;
    personalInfo.position = 'One of the 18th candles';
  }

  if (personAsRose) {
    isEighteenthRoses.value = true;
    eighteenthRosesName.value = personAsRose.name;

    if (!personAsCandle) { 
      personalInfo.position = 'One of the 18th Roses';
    }
  }

  // Optional: If you want a combined position when found in both
  if (personAsCandle && personAsRose) {
    personalInfo.position = 'One of the 18th candles and 18th Roses';
  } else if (personAsShots && personAsRose) {
    personalInfo.position = 'One of the 18th Shot and 18th Roses';
  }
  
  else if (personAsCandle) {
      personalInfo.position = 'One of the 18th candles';
  } else if (personAsRose) {
      personalInfo.position = 'One of the 18th Roses';
  } else if (personAsShots) {
      personalInfo.position = 'One of the 18th Shots';
  }
};

// Handle input for the name field
const handleNameInput = () => {

  thankYouMessage.value = '';
  eighteenthCandleName.value = '';

  isEighteenthCandle.value = false;
  isEighteenthRoses.value = false;
  eighteenthRosesName.value = '';
  personalInfo.position = '';
  attendance.value = '';
  if (attendance.value === 'Yes') {
    findPerson();
  } else {
    isEighteenthCandle.value = false;
    eighteenthCandleName.value = '';
    eighteenthRosesName.value = '';
    personalInfo.position = ''; // Clear position if attendance is not 'Yes'
  }
};

// Handle attendance radio button change
const handleAttendanceChange = (value: string) => {
  attendance.value = value;

  if (attendance.value === 'Yes') {
    if (!name.value) {
      alert('Please enter your name first.');
      attendance.value = ''; // Reset attendance if name is not entered
      isEighteenthCandle.value = false;
      eighteenthCandleName.value = '';
      isEighteenthRoses.value = false;
    eighteenthRosesName.value = '';
      personalInfo.position = '';
    } else {
      findPerson();
    }
  } else {
    isEighteenthCandle.value = false;
    eighteenthCandleName.value = '';
    isEighteenthRoses.value = false;
    eighteenthRosesName.value = '';
    personalInfo.position = '';
    
  }
};

// Function to save personal info to Firebase
const savePersonalInfo = async () => {
  try {
    const userId = "user_" + Date.now();
    const userDocRef = doc(db, "users", userId);

    // Prepare data to save, ensuring 'createdAt' is always included
    const dataToSave = {
      name: personalInfo.name,
      attended: personalInfo.attended,
      position: personalInfo.position,
      createdAt: new Date()
    };

    await setDoc(userDocRef, dataToSave);
    console.log("Document successfully written!");
  } catch (error) {
    alert("Error saving RSVP: " + error);
  }
};

// Handle form submission
const handleSubmit = async () => {
  if (!name.value || !attendance.value) {
    alert('Please enter your name and select attendance.');
    return;
  }

  // Set personalInfo reactive object values
  personalInfo.name = name.value;
  personalInfo.attended = attendance.value === 'Yes' ? 'Attending' : 'Not Attending';
  // The 'position' is already set by findPerson when name or attendance changes

  await savePersonalInfo(); // Save to Firebase

  thankYouMessage.value = `Thank you, ${name.value}! Your RSVP that you will ${attendance.value === 'Yes' ? 'attend' : 'not attend'} has been received.`;

  // Clear the input fields after submission
  name.value = '';
  attendance.value = '';

  // Reset candle status and personalInfo after submission
  isEighteenthCandle.value = false;
  eighteenthCandleName.value = '';
  personalInfo.name = '';
  personalInfo.attended = '';
  personalInfo.position = '';
};

// Fetch users (RSVPs) on component mount (for demonstration/debugging)
const usersList = ref<any[]>([]);
const fetchUsers = async () => {
  usersList.value = []; // Clear previous data
  try {
    const querySnapshot = await getDocs(collection(db, "users"));
    const fetchedUsers: any[] = [];
    querySnapshot.forEach((doc) => {
      fetchedUsers.push({ id: doc.id, ...doc.data() });
    });
    usersList.value = fetchedUsers;
    console.log("Fetched Users:", usersList.value);
  } catch (error: any) {
    console.error("Error fetching RSVPs: ", error);
  }
};

onMounted(() => {
  fetchUsers();
});
</script>
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Great+Vibes&family=Poppins:wght@400;600&display=swap');

/* Reset */
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: linear-gradient(135deg, #ffe6f0 0%, #ffcce3 100%);
  color: #660033;
  min-height: 100vh;
  /* Ensure full viewport height */
  display: flex;
  justify-content: center; /* Centers horizontally */
  align-items: center;     /* Centers vertically */
  /* Remove padding from body to let container handle its own spacing */
  /* padding: 1rem; */ /* Remove or set to 0 if you want strict centering */
  overflow-x: hidden; /* Prevent horizontal scrollbar on small screens */
}

.container {
  background: #ffd2e4cc;
  max-width: 450px;
  width: 100%;
  border-radius: 20px;
  box-shadow:
    0 0 15px 5px rgba(255, 192, 203, 0.25),
    0 0 30px 10px rgba(255, 182, 193, 0.3);
  padding: 2.5rem 2rem 3rem 2rem; /* Maintain container's internal padding */
  text-align: center;
  overflow: hidden;
  transform: scale(0.95);
  opacity: 0;
  animation: containerLoad 0.8s ease-out forwards;

  /* Add margin auto for block-level centering when it doesn't fill the flex container */
  /* While display:flex on body usually handles this, margin:auto adds robustness */
  margin: auto;
}

@keyframes containerLoad {
  to {
    transform: scale(1);
    opacity: 1;
  }
}

.title {
  font-family: 'Great Vibes', cursive;
  font-size: 3.6rem;
  color: #e60073;
  margin-bottom: 0;
  margin-top: 0;
  letter-spacing: 2px;
  text-shadow: 0 0 8px #ffffff, 0 0 15px #ffbaef;
  font-style: italic;
  animation: textFadeIn 1s ease-out 0.3s forwards;
  opacity: 0;
  transform: translateY(-20px);
}

.subtitle {
  font-weight: 600;
  font-size: 1.25rem;
  margin-top: 6px;
  margin-bottom: 1.8rem;
  color: #000000;
  animation: textFadeIn 1s ease-out 0.5s forwards;
  opacity: 0;
  transform: translateY(-20px);
}

.invitation-text {
  font-family: 'Quiche', sans-serif;
  font-size: 1.1rem;
  margin-bottom: 1.8rem;
  color: #000000;
  animation: textFadeIn 1s ease-out 0.7s forwards;
  opacity: 0;
  transform: translateY(-20px);
}

@keyframes textFadeIn {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

form {
  display: flex;
  flex-direction: column;
  gap: 1.2rem;
  animation: formSlideIn 1s ease-out 0.9s forwards;
  opacity: 0;
  transform: translateY(20px);
}

@keyframes formSlideIn {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

label {
  font-weight: 600;
  font-size: 1rem;
  color: #660033;
  text-align: left;
}

input[type="text"] {
  padding: 0.8rem 1rem;
  border: 2px solid #ff99cc;
  border-radius: 15px;
  font-size: 1rem;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}
input[type="text"]:focus {
  outline: none;
  border-color: #e60073;
  box-shadow: 0 0 10px rgba(230, 0, 115, 0.6);
}

.radio-group {
  display: flex;
  gap: 1.5rem;
  justify-content: center;
  flex-wrap: wrap;
}

.radio-group label {
  font-weight: 600;
  font-size: 1rem;
  cursor: pointer;
  transition: color 0.3s ease, transform 0.2s ease;
}

.radio-group label:hover {
  color: #e60073;
  transform: translateY(-2px);
}

input[type="radio"] {
  accent-color: #e60073;
  cursor: pointer;
  transform: scale(1.1);
}

button {
  background: linear-gradient(45deg, #ff66b3, #e60073);
  color: white;
  font-weight: 700;
  font-size: 1.15rem;
  padding: 0.9rem 0;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  box-shadow: 0 0 20px rgba(255, 0, 153, 0.6);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

button:hover {
  background: linear-gradient(45deg, #e60073, #ff66b3);
  box-shadow: 0 0 30px rgba(255, 51, 153, 0.8);
  transform: translateY(-3px);
}

button:active {
  transform: translateY(0);
  box-shadow: 0 0 10px rgba(255, 0, 153, 0.4);
}

button::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 5px;
  height: 5px;
  background: rgba(255, 255, 255, 0.5);
  border-radius: 50%;
  opacity: 0;
  transform: scale(1);
  transition: all 0.5s ease-out;
}

button:active::after {
  transform: scale(200);
  opacity: 1;
  transition: 0s;
}

.thankyou-message {
  font-size: 1.3rem;
  font-weight: 600;
  color: #cc0066;
  margin-top: 1.6rem;
  animation: fadeInUp 0.8s ease forwards;
  text-shadow: 0 1px 5px rgba(204, 0, 102, 0.2);
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.dialog-special-guest {
  font-size: 1.2rem;
  font-weight: bold;
  color: #e60073;
  margin-bottom: 1rem;
  animation: pulseEffect 1.5s infinite alternate;
}

@keyframes pulseEffect {
  from {
    transform: scale(1);
    opacity: 1;
  }
  to {
    transform: scale(1.03);
    opacity: 0.9;
  }
}

.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
  backdrop-filter: blur(4px);
  animation: fadeInOverlay 0.3s ease;
}

@keyframes fadeInOverlay {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.dialog-content {
  background-color: #ffffff;
  border-radius: 1rem;
  padding: 2rem;
  text-align: center;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
  width: 90%;
  max-width: 500px;
  animation: dialogPopIn 0.4s cubic-bezier(0.68, -0.55, 0.27, 1.55) forwards;
}

@keyframes dialogPopIn {
  from {
    opacity: 0;
    transform: scale(0.8) translateY(-20px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.dialog-title {
  font-size: 2rem;
  font-family: 'Great Vibes', cursive;
  color: #f5889f;
  margin-bottom: 1rem;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.dialog-description {
  font-size: 1.1rem;
  color: #6b7280;
  margin-bottom: 1.5rem;
  line-height: 1.6;
}
.dialog-additional-info {
  font-size: 1rem;
  color: #660033;
  margin-bottom: 1.5rem;
  font-style: italic;
}

/* --- Media Queries for Responsiveness --- */

/* For screens smaller than 768px (e.g., tablets in portrait, large phones) */
@media (max-width: 768px) {
  .container {
    padding: 2rem 1.8rem 2.5rem 1.8rem;
    /* max-width: 400px; - Removed, let width:100% handle this within padding */
  }
  .title {
    font-size: 3.2rem;
  }
  .subtitle {
    font-size: 1.15rem;
  }
  .invitation-text {
    font-size: 1rem;
  }
  .dialog-content {
    max-width: 450px;
  }
  .dialog-title {
    font-size: 1.8rem;
  }
  .dialog-description {
    font-size: 1rem;
  }
}

/* For screens smaller than 500px (typical mobile phones) */
@media (max-width: 500px) {
  body {
    padding: 0; /* Remove body padding on small screens to give full space to container */
  }
  .container {
    max-width: 100vw; /* Take full viewport width */
    padding: 1.8rem 1.2rem 2.2rem 1.2rem;
    border-radius: 0; /* Optional: Make corners sharp on full-width mobile */
    min-height: 100vh; /* Make container take full height on small screens */
    display: flex;
    flex-direction: column;
    justify-content: center; /* Center content vertically within container */
    align-items: center; /* Center content horizontally within container */
  }
  .title {
    font-size: 2.8rem;
  }
  .subtitle {
    font-size: 1rem;
  }
  .invitation-text {
    font-size: 0.95rem;
  }
  label {
    font-size: 0.95rem;
  }
  input[type="text"] {
    padding: 0.7rem 0.9rem;
    font-size: 0.9rem;
  }
  .radio-group {
    flex-direction: column;
    gap: 0.8rem;
    align-items: flex-start;
  }
  .radio-group label {
    font-size: 0.95rem;
  }
  button {
    font-size: 1.05rem;
    padding: 0.8rem 0;
  }
  .thankyou-message {
    font-size: 1.1rem;
  }
  .dialog-content {
    padding: 1.5rem;
    border-radius: 1rem;
  }
  .dialog-title {
    font-size: 1.75rem;
  }
  .dialog-description {
    font-size: 0.95rem;
  }
  .dialog-additional-info {
    font-size: 0.9rem;
  }
}

/* For very small screens (e.g., iPhone SE/mini) */
@media (max-width: 375px) {
  .container {
    padding: 1.5rem 1rem 1.8rem 1rem;
  }
  .title {
    font-size: 2.4rem;
  }
  .subtitle {
    font-size: 0.9rem;
  }
  .invitation-text {
    font-size: 0.85rem;
  }
  label {
    font-size: 0.9rem;
  }
  input[type="text"] {
    padding: 0.6rem 0.8rem;
    font-size: 0.85rem;
  }
  button {
    font-size: 1rem;
    padding: 0.7rem 0;
  }
  .thankyou-message {
    font-size: 1rem;
  }
  .dialog-content {
    padding: 1.2rem;
  }
  .dialog-title {
    font-size: 1.5rem;
  }
  .dialog-description {
    font-size: 0.9rem;
  }
}
</style>