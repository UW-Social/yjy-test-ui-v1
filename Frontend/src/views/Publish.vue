<template>
  <div class="publish-page">
    <!-- 背景 SVG -->
    <div class="background-container">
      <img src="../../public/svg/background4.svg" class="background-svg" />
    </div>
    
    <div class="page-header">
      <h1>{{ publishType === 'event' ? 'Create Your Event' : 'Create Your Club' }}</h1>
      <p>{{ publishType === 'event' ? 'Bring people together and make connections happen' : 'Start a community and grow your organization' }}</p>
      
      <!-- Toggle buttons -->
      <div class="type-selector">
        <button 
          :class="{ active: publishType === 'event' }" 
          @click="publishType = 'event'"
        >
          📅 Event
        </button>
        <button 
          :class="{ active: publishType === 'club' }" 
          @click="publishType = 'club'"
        >
          🏛️ Club
        </button>
      </div>
    </div>
    
    <EventForm v-if="publishType === 'event'" />
    <ClubForm v-else @cancel="handleCancel" @success="handleSuccess" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import EventForm from '../components/EventForm.vue';
import ClubForm from '../components/ClubForm.vue';

const router = useRouter();
const publishType = ref<'event' | 'club'>('event');

const handleCancel = () => {
  router.push('/clubs');
};

const handleSuccess = () => {
  router.push('/clubs');
};
</script>

<style scoped>
.publish-page {
  position: relative;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  padding-top: 120px;
  padding-bottom: 4rem;
}

.background-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  overflow: hidden;
}

.background-svg {
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0.3;
}

.page-header {
  position: relative;
  z-index: 1;
  text-align: center;
  margin-bottom: 3rem;
  padding: 0 1rem;
}

.page-header h1 {
  font-size: 2.5rem;
  font-weight: 700;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 0.5rem;
}

.page-header p {
  font-size: 1.1rem;
  color: #666;
  max-width: 500px;
  margin: 0 auto 1.5rem;
}

.type-selector {
  display: flex;
  gap: 1rem;
  justify-content: center;
  margin-top: 1.5rem;
}

.type-selector button {
  padding: 0.75rem 2rem;
  border: 2px solid #e0e0e0;
  background: white;
  border-radius: 12px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.type-selector button:hover {
  border-color: #7c3aed;
  transform: translateY(-2px);
}

.type-selector button.active {
  background: #7c3aed;
  color: white;
  border-color: #7c3aed;
}

@media (max-width: 768px) {
  .publish-page {
    padding-top: 100px;
  }
  
  .page-header h1 {
    font-size: 2rem;
  }

  .type-selector {
    flex-direction: column;
    align-items: stretch;
    padding: 0 1rem;
  }

  .type-selector button {
    width: 100%;
  }
  
  .page-header p {
    font-size: 1rem;
  }
}
</style> 