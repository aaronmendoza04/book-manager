<template>
  <ion-page>
    <ion-content class="glass-bg">
      <div class="glass-container">
        
        <h1 class="stoic-title">Book Collection Manager.</h1>
        
        <!-- FORM SECTION -->
        <div class="glass-card">
          <ion-item class="glass-item" lines="none">
            <ion-input v-model="book.title" label="Title" label-placement="floating"></ion-input>
          </ion-item>
          
          <ion-item class="glass-item" lines="none">
            <ion-input v-model="book.author" label="Author" label-placement="floating"></ion-input>
          </ion-item>
          
          <ion-item class="glass-item" lines="none">
            <ion-select v-model="book.category" label="Category" label-placement="floating">
              <ion-select-option value="Fiction">Fiction</ion-select-option>
              <ion-select-option value="Non-Fiction">Non-Fiction</ion-select-option>
              <ion-select-option value="Academic">Academic</ion-select-option>
            </ion-select>
          </ion-item>
          
          <ion-item class="glass-item" lines="none">
            <ion-input v-model="book.year" type="number" label="Publication Year" label-placement="floating"></ion-input>
          </ion-item>
          
          <ion-item class="glass-item" lines="none">
            <ion-toggle v-model="book.isAvailable" color="success">Available in Library</ion-toggle>
          </ion-item>
          
          <div class="form-actions">
            <button class="minimal-btn" @click="saveBook">
              {{ editingId ? 'Update Record' : 'Commit Record' }}
            </button>
            <button v-if="editingId" class="minimal-btn cancel-btn" @click="cancelEdit">Cancel</button>
          </div>
        </div>

        <h2 class="stoic-subtitle">Archive.</h2>
        
        <!-- LIST SECTION -->
        <div class="glass-card list-section" v-for="b in bookList" :key="b.id">
          <div class="book-info">
            <h3 class="book-title">{{ b.title }}</h3>
            <p class="book-author">{{ b.author }}</p>
            <p class="book-meta">{{ b.category }} — {{ b.year }}</p>
            
            <!-- NEW: Update and Delete Buttons -->
            <div class="action-buttons">
              <button class="text-btn edit-btn" @click="prepareEdit(b)">Edit</button>
              <button class="text-btn delete-btn" @click="deleteBook(b.id)">Delete</button>
            </div>
          </div>
          
          <div class="book-status">
            <span :class="['status-indicator', b.isAvailable ? 'active' : 'inactive']"></span>
            <span class="status-text">{{ b.isAvailable ? 'Available' : 'Checked Out' }}</span>
          </div>
        </div>
        
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { 
  IonPage, IonContent, IonItem, IonInput, 
  IonSelect, IonSelectOption, IonToggle 
} from '@ionic/vue';
import { db } from '@/firebase';
import { ref as dbRef, push, onValue, update, remove } from 'firebase/database';

// 1. Data containers
const book = ref({
  title: '',
  author: '',
  category: '',
  year: '',
  isAvailable: true
});

const bookList = ref<any[]>([]);

// NEW: Tracks if we are editing an existing book instead of making a new one
const editingId = ref<string | null>(null);

// 2. READ: Load Function
onMounted(() => {
  const dbConnection = dbRef(db, 'books');
  onValue(dbConnection, (snapshot) => {
    const rawData = snapshot.val();
    if (rawData) {
      bookList.value = Object.keys(rawData).map(key => ({
        id: key,
        ...rawData[key]
      })).reverse();
    } else {
      bookList.value = []; 
    }
  });
});

// 3. CREATE & UPDATE: Save Function
const saveBook = async () => {
  if (!book.value.title || !book.value.author) return;
  
  if (editingId.value) {
    // UPDATE LOGIC: Overwrite the existing record in Firebase
    const specificBookRef = dbRef(db, `books/${editingId.value}`);
    await update(specificBookRef, book.value);
  } else {
    // CREATE LOGIC: Push a brand new record to Firebase
    const dbConnection = dbRef(db, 'books');
    await push(dbConnection, book.value);
  }
  
  cancelEdit(); // Reset the form after saving
};

// 4. UPDATE SETUP: Puts the selected book's data back into the form
const prepareEdit = (b: any) => {
  book.value = { 
    title: b.title, 
    author: b.author, 
    category: b.category, 
    year: b.year, 
    isAvailable: b.isAvailable 
  };
  editingId.value = b.id; // Tell the app we are in edit mode
};

// Cancels the edit and clears the form
const cancelEdit = () => {
  book.value = { title: '', author: '', category: '', year: '', isAvailable: true };
  editingId.value = null;
};

// 5. DELETE: Removes the specific book from Firebase
const deleteBook = async (id: string) => {
  const specificBookRef = dbRef(db, `books/${id}`);
  await remove(specificBookRef);
};
</script>

<style scoped>
/* Deep Ocean Blue Background */
.glass-bg {
  --background: linear-gradient(135deg, #020b14 0%, #0a192f 100%);
}

.glass-container {
  padding: 32px 24px;
  min-height: 100%;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

/* Typography */
.stoic-title {
  font-size: 28px;
  font-weight: 300;
  letter-spacing: -0.5px;
  margin-bottom: 32px;
  color: #ccd6f6;
}

.stoic-subtitle {
  font-size: 14px;
  font-weight: 500;
  letter-spacing: 2px;
  margin-top: 48px;
  margin-bottom: 16px;
  color: #8892b0;
  text-transform: uppercase;
}

/* Glassmorphism Cards */
.glass-card {
  background: rgba(16, 32, 64, 0.4);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 16px;
}

/* Form Inputs */
.glass-item {
  --background: transparent;
  --color: #ccd6f6;
  --border-color: rgba(255, 255, 255, 0.1);
  margin-bottom: 8px;
}

ion-input, ion-select {
  color: #ccd6f6;
}

/* Custom Minimalist Buttons */
.form-actions {
  display: flex;
  gap: 12px;
}

.minimal-btn {
  flex: 1;
  padding: 16px;
  margin-top: 24px;
  background: rgba(100, 255, 218, 0.1);
  color: #64ffda;
  border: 1px solid rgba(100, 255, 218, 0.2);
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  cursor: pointer;
  transition: all 0.3s ease;
}

.minimal-btn:active {
  background: rgba(100, 255, 218, 0.2);
}

.cancel-btn {
  background: rgba(240, 62, 62, 0.05);
  color: #f03e3e;
  border: 1px solid rgba(240, 62, 62, 0.2);
}

.cancel-btn:active {
  background: rgba(240, 62, 62, 0.1);
}

/* List Layout */
.list-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px;
}

.book-title {
  margin: 0 0 6px 0;
  font-size: 18px;
  font-weight: 500;
  color: #ccd6f6;
}

.book-author {
  margin: 0 0 12px 0;
  font-size: 14px;
  color: #8892b0;
}

.book-meta {
  margin: 0;
  font-size: 12px;
  color: #64ffda;
}

/* Action Buttons (Edit/Delete) */
.action-buttons {
  display: flex;
  gap: 12px;
  margin-top: 16px;
}

.text-btn {
  background: transparent;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 1px;
  cursor: pointer;
  padding: 4px 10px;
  border-radius: 4px;
  transition: all 0.2s ease;
}

.edit-btn {
  color: #8892b0;
  border: 1px solid rgba(136, 146, 176, 0.3);
}

.delete-btn {
  color: #f03e3e;
  border: 1px solid rgba(240, 62, 62, 0.3);
}

/* Availability Status Indicator */
.book-status {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 8px;
}

.status-indicator {
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

.status-indicator.active {
  background-color: #64ffda;
  box-shadow: 0 0 10px rgba(100, 255, 218, 0.4);
}

.status-indicator.inactive {
  background-color: #f03e3e;
  box-shadow: 0 0 10px rgba(240, 62, 62, 0.4);
}

.status-text {
  font-size: 11px;
  letter-spacing: 1px;
  text-transform: uppercase;
  color: #8892b0;
}
</style>