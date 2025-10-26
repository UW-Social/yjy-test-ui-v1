<template>
  <div class="card event-card" @click="$emit('open-card', club)">
    <img :src="club.imageUrl || '/images/wavingdog.jpg'" alt="Club Image" />
    <div class="event-card-content">
      <!-- 左侧内容 -->
      <div class="event-card-left">
        <h3 class="event-title">{{ club.name }}</h3>
        <p class="description">{{ club.description }}</p>
        <div v-if="club.link" class="event-link">
          <a :href="club.link" target="_blank" rel="noopener noreferrer">Learn More</a>
        </div>
      </div>
      <!-- 右侧内容 -->
      <div class="event-card-right">
        <p class="event-time">{{ club.meetingSchedule || 'Meeting schedule TBD' }}</p>
        <p class="event-location">Location: {{ club.location || 'TBD' }}</p>
        <p class="event-location">Category: {{ formatClubCategory(club.category) }}</p>
        <p v-if="club.memberCount" class="event-location">Members: {{ club.memberCount }}</p>
        <el-button
          v-if="String(club.organizerId) === String(currentUserId)"
          @click.stop="handleDelete"
          type="primary"
          round
          size="small"
          class="delete-btn purple-btn"
          style="margin-top: 1rem;"
        >
          Delete
        </el-button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { defineProps, toRefs } from 'vue';
import { doc, deleteDoc } from 'firebase/firestore';
import { db } from '../firebase/config';
import { useClubStore } from '../stores/club';
import { formatClubCategory, type Club } from '../types/club';
import '../assets/eventcard.css';
import { ElButton } from 'element-plus';
import 'element-plus/es/components/button/style/css';

const props = defineProps<{
  club: Club;
  currentUserId?: string | number;
}>();

const { club } = toRefs(props);
const clubStore = useClubStore();

// Clicking the whole card emits to open detail; store selection not strictly needed here

const handleDelete = async () => {
  if (!confirm('Are you sure you want to delete this club?')) return;
  try {
    await deleteDoc(doc(db, 'clubs', props.club.id));
    alert('Club deleted.');
    await clubStore.fetchClubs();
  } catch (e) {
    alert('Failed to delete club.');
  }
};
</script>

<style scoped>
/* Use eventcard.css for layout and keep only button tweaks here */
.delete-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  letter-spacing: 0.5px;
  box-shadow: 0 2px 8px rgba(128, 0, 255, 0.08);
  background: #7c3aed !important;
  border-color: #7c3aed !important;
  color: #fff !important;
  transition: background 0.2s, box-shadow 0.2s;
}

.delete-btn:hover {
  background: #a78bfa !important;
  border-color: #a78bfa !important;
  color: #fff !important;
  box-shadow: 0 4px 16px rgba(128, 0, 255, 0.15);
}
</style>
