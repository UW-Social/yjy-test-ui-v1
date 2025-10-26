<template>
  <div class="club-form">
    <div class="form-container">
      <h2>{{ isEditing ? 'Edit Club' : 'Create New Club' }}</h2>
      
      <form @submit.prevent="handleSubmit" class="form-content">
        <div class="form-group">
          <label for="name">Club Name *</label>
          <input
            id="name"
            v-model="formData.name"
            type="text"
            required
            placeholder="Enter club name"
          >
        </div>

        <div class="form-group">
          <label for="description">Description *</label>
          <textarea
            id="description"
            v-model="formData.description"
            required
            placeholder="Describe your club, its mission, and activities..."
            rows="5"
          ></textarea>
        </div>

        <div class="form-row">
          <div class="form-group">
            <label for="category">Category *</label>
            <select id="category" v-model="formData.category" required>
              <option value="">Select category</option>
              <option value="ACADEMIC">Academic</option>
              <option value="SPORTS">Sports</option>
              <option value="ARTS">Arts & Culture</option>
              <option value="TECHNOLOGY">Technology</option>
              <option value="VOLUNTEERING">Volunteering</option>
              <option value="CULTURAL">Cultural</option>
              <option value="PROFESSIONAL">Professional</option>
              <option value="SOCIAL">Social</option>
              <option value="OTHER">Other</option>
            </select>
          </div>

          <div class="form-group">
            <label for="location">Meeting Location</label>
            <input
              id="location"
              v-model="formData.location"
              type="text"
              placeholder="Where does your club meet?"
            >
          </div>
        </div>

        <div class="form-group">
          <label for="meetingSchedule">Meeting Schedule</label>
          <input
            id="meetingSchedule"
            v-model="formData.meetingSchedule"
            type="text"
            placeholder="e.g., Every Wednesday 6PM"
          >
        </div>

        <div class="form-row">
          <div class="form-group">
            <label for="contactEmail">Contact Email</label>
            <input
              id="contactEmail"
              v-model="formData.contactEmail"
              type="email"
              placeholder="club@example.com"
            >
          </div>

          <div class="form-group">
            <label for="link">Website/Social Media</label>
            <input
              id="link"
              v-model="formData.link"
              type="url"
              placeholder="https://..."
            >
          </div>
        </div>

        <div class="form-group">
          <label for="tags">Tags (comma-separated)</label>
          <input
            id="tags"
            v-model="tagsInput"
            type="text"
            placeholder="e.g., coding, networking, beginner-friendly"
          >
          <small>Separate tags with commas</small>
        </div>

        <div class="form-group">
          <label for="imageUrl">Club Image URL</label>
          <input
            id="imageUrl"
            v-model="formData.imageUrl"
            type="url"
            placeholder="https://example.com/image.jpg"
          >
          <small>Provide a URL to an image for your club</small>
        </div>

        <div class="form-actions">
          <button type="button" @click="$emit('cancel')" class="cancel-btn">
            Cancel
          </button>
          <button type="submit" class="submit-btn" :disabled="isSubmitting">
            {{ isSubmitting ? 'Saving...' : (isEditing ? 'Update Club' : 'Create Club') }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { collection, addDoc, updateDoc, doc, serverTimestamp } from 'firebase/firestore';
import { db } from '../firebase/config';
import { useUserStore } from '../stores/user';
import { useClubStore } from '../stores/club';
import type { Club } from '../types/club';

const props = defineProps<{
  club?: Club;
}>();

const emit = defineEmits(['cancel', 'success']);

const userStore = useUserStore();
const clubStore = useClubStore();
const isSubmitting = ref(false);
const tagsInput = ref('');

const isEditing = computed(() => !!props.club);

const formData = ref({
  name: '',
  description: '',
  category: '',
  location: '',
  meetingSchedule: '',
  contactEmail: '',
  link: '',
  imageUrl: '',
});

onMounted(() => {
  if (props.club) {
    formData.value = {
      name: props.club.name,
      description: props.club.description,
      category: props.club.category,
      location: props.club.location || '',
      meetingSchedule: props.club.meetingSchedule || '',
      contactEmail: props.club.contactEmail || '',
      link: props.club.link || '',
      imageUrl: props.club.imageUrl || '',
    };
    tagsInput.value = props.club.tags?.join(', ') || '';
  }
});

const handleSubmit = async () => {
  if (!userStore.userProfile) {
    alert('Please log in to create a club.');
    return;
  }

  isSubmitting.value = true;

  try {
    const tags = tagsInput.value
      .split(',')
      .map(tag => tag.trim())
      .filter(tag => tag.length > 0);

    const clubData = {
      name: formData.value.name,
      description: formData.value.description,
      category: formData.value.category,
      location: formData.value.location || undefined,
      meetingSchedule: formData.value.meetingSchedule || undefined,
      contactEmail: formData.value.contactEmail || undefined,
      link: formData.value.link || undefined,
      imageUrl: formData.value.imageUrl || undefined,
      tags: tags.length > 0 ? tags : undefined,
      organizerId: userStore.userProfile.uid,
      organizerName: userStore.userProfile.displayName,
      memberCount: props.club?.memberCount || 1,
      members: props.club?.members || [userStore.userProfile.uid],
      updatedAt: serverTimestamp(),
    };

    if (isEditing.value && props.club) {
      // Update existing club
      const clubRef = doc(db, 'clubs', props.club.id);
      await updateDoc(clubRef, clubData);
      alert('Club updated successfully!');
    } else {
      // Create new club
      await addDoc(collection(db, 'clubs'), {
        ...clubData,
        createdAt: serverTimestamp(),
      });
      alert('Club created successfully!');
    }

    await clubStore.fetchClubs();
    emit('success');
  } catch (error) {
    console.error('Error saving club:', error);
    alert('Failed to save club. Please try again.');
  } finally {
    isSubmitting.value = false;
  }
};
</script>

<style scoped>
.club-form {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

.form-container {
  background: white;
  border-radius: 16px;
  padding: 2rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

h2 {
  font-size: 2rem;
  margin-bottom: 2rem;
  color: #333;
  text-align: center;
}

.form-content {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

label {
  font-weight: 600;
  color: #333;
  font-size: 0.95rem;
}

input,
textarea,
select {
  padding: 0.75rem;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.3s;
  font-family: inherit;
}

input:focus,
textarea:focus,
select:focus {
  outline: none;
  border-color: #7c3aed;
}

textarea {
  resize: vertical;
  min-height: 100px;
}

small {
  color: #666;
  font-size: 0.85rem;
}

.form-actions {
  display: flex;
  gap: 1rem;
  margin-top: 1rem;
}

.cancel-btn,
.submit-btn {
  flex: 1;
  padding: 1rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.cancel-btn {
  background: #f0f0f0;
  color: #666;
}

.cancel-btn:hover {
  background: #e0e0e0;
}

.submit-btn {
  background: #7c3aed;
  color: white;
}

.submit-btn:hover:not(:disabled) {
  background: #6d28d9;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(124, 58, 237, 0.3);
}

.submit-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

@media (max-width: 768px) {
  .club-form {
    padding: 1rem;
  }

  .form-container {
    padding: 1.5rem;
  }

  .form-row {
    grid-template-columns: 1fr;
  }

  h2 {
    font-size: 1.5rem;
  }
}
</style>
