<template>
  <div class="club-detail-page">
    <!-- 返回按钮 -->
    <button @click="goBack" class="back-button">
      <span class="arrow">←</span>
    </button>

    <!-- 顶部：社团Logo和描述 -->
    <div class="club-header-section">
      <div class="club-logo-card">
        <img :src="club?.imageUrl || '/images/wavingdog.jpg'" alt="Club Logo" class="club-logo" />
      </div>
      
      <div class="club-description-card">
        <p class="club-description">{{ club?.description || 'Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.' }}</p>
        <div class="club-tags">
          <span v-for="tag in club?.tags" :key="tag" class="tag">#{{ tag }}</span>
        </div>
      </div>
    </div>

    <!-- 社团活动 -->
    <div class="club-events-section">
      <h2 class="section-title">Club Events</h2>
      <div class="events-list">
        <div v-for="event in clubEvents" :key="event.id" class="event-item" @click="goToEvent(event.id)">
          <img :src="event.imageUrl || '/images/wavingdog.jpg'" alt="Event" class="event-image" />
          <div class="event-details">
            <p class="event-description">{{ event.description || event.title }}</p>
          </div>
        </div>
        <div v-if="clubEvents.length === 0" class="no-events">
          <p>No events available yet.</p>
        </div>
      </div>
    </div>

    <!-- 相关社团 -->
    <div class="relevant-clubs-section">
      <h2 class="section-title">Relevant Clubs</h2>
      <div class="clubs-grid">
        <div v-for="relatedClub in relevantClubs" :key="relatedClub.id" class="club-card" @click="goToClub(relatedClub.id)">
          <img :src="relatedClub.imageUrl || '/images/wavingdog.jpg'" alt="Club" class="club-thumbnail" />
        </div>
        <div v-if="relevantClubs.length === 0" class="placeholder-cards">
          <div v-for="i in 8" :key="i" class="club-card-placeholder"></div>
        </div>
      </div>
    </div>

    <!-- 联系信息 -->
    <div class="contact-info-section">
      <h2 class="section-title">Contact Info</h2>
      <div class="contact-grid">
        <div class="contact-column">
          <div class="contact-item">
            <strong>Address:</strong>
            <p>{{ club?.address || '4800 Union Bay Place NE' }}</p>
            <p>{{ club?.city || 'Seattle, WA 98105' }}</p>
            <p>United States</p>
          </div>
          <div class="contact-item">
            <strong>Contact Email:</strong>
            <p>{{ club?.contactEmail || 'casa@uw.edu' }}</p>
          </div>
          <div class="contact-item">
            <strong>Phone Number:</strong>
            <p>{{ club?.phone || '206-4685924' }}</p>
          </div>
        </div>

        <div class="contact-column">
          <div class="contact-item">
            <strong>Campus Office:</strong>
            <p>{{ club?.campusOffice || 'Husky Union Building 5B-256' }}</p>
          </div>
          <div class="contact-item">
            <strong>UW email address:</strong>
            <p>{{ club?.uwEmail || 'casa@uw.edu' }}</p>
          </div>
          <div class="contact-item">
            <strong>Renee Infelise</strong>
          </div>
        </div>

        <div class="contact-column">
          <div class="contact-item">
            <strong>UW Department Advisor:</strong>
            <p>{{ club?.departmentAdvisor || 'NONE' }}</p>
          </div>
          <div class="contact-item">
            <strong>Off-Campus Advisor:</strong>
            <p>{{ club?.offCampusAdvisor || 'NONE' }}</p>
          </div>
          <div class="contact-item">
            <strong>Local/national affiliation organization:</strong>
            <p>{{ club?.affiliation || 'NONE' }}</p>
          </div>
          <div class="contact-item">
            <strong>Other Social Media Contact:</strong>
            <p>{{ club?.socialMedia || 'NONE' }}</p>
          </div>
          <div class="social-icons">
            <a v-if="club?.discord" :href="club.discord" target="_blank" class="social-icon discord">
              Discord
            </a>
            <a v-if="club?.instagram" :href="club.instagram" target="_blank" class="social-icon instagram">
              Instagram
            </a>
            <a v-if="club?.wechat" :href="club.wechat" target="_blank" class="social-icon wechat">
              WeChat
            </a>
            <a v-if="club?.linkedin" :href="club.linkedin" target="_blank" class="social-icon linkedin">
              LinkedIn
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useClubStore } from '../stores/club';
import { useEventStore } from '../stores/event';
import type { Club } from '../types/club';
import type { Event } from '../types/event';

const route = useRoute();
const router = useRouter();
const clubStore = useClubStore();
const eventStore = useEventStore();

const clubId = computed(() => route.params.id as string);
const club = ref<Club | null>(null);
const clubEvents = ref<Event[]>([]);
const relevantClubs = ref<Club[]>([]);

const goBack = () => {
  router.push('/clubs');
};

const goToEvent = (eventId: string) => {
  router.push(`/events?eventId=${eventId}`);
};

const goToClub = (clubId: string) => {
  router.push(`/clubs/${clubId}`);
};

const loadClubData = async () => {
  // 加载社团数据
  if (clubStore.clubs.length === 0) {
    await clubStore.fetchClubs();
  }
  
  club.value = clubStore.clubs.find(c => c.id === clubId.value) || null;
  
  // 加载社团相关的活动
  if (eventStore.events.length === 0) {
    await eventStore.fetchEvents();
  }
  
  // 筛选出该社团的活动
  clubEvents.value = eventStore.events.filter(e => 
    e.organizerId === club.value?.organizerId || 
    e.organizerName === club.value?.name
  ).slice(0, 5);
  
  // 加载相关社团（基于标签相似度）
  if (club.value?.tags && club.value.tags.length > 0) {
    relevantClubs.value = clubStore.clubs
      .filter(c => c.id !== clubId.value)
      .filter(c => {
        if (!c.tags || c.tags.length === 0) return false;
        return c.tags.some(tag => club.value?.tags?.includes(tag));
      })
      .slice(0, 8);
  }
};

onMounted(() => {
  loadClubData();
});
</script>

<style scoped>
.club-detail-page {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  position: relative;
}

.back-button {
  position: absolute;
  top: 1rem;
  left: 1rem;
  background: white;
  border: 2px solid #ddd;
  border-radius: 8px;
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 1.5rem;
  z-index: 10;
}

.back-button:hover {
  background: #f5f5f5;
  border-color: #999;
  transform: translateX(-3px);
}

.arrow {
  color: #333;
  font-weight: bold;
}

.club-header-section {
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 2rem;
  margin-top: 3rem;
  margin-bottom: 2rem;
}

.club-logo-card {
  background: white;
  border: 2px solid #ddd;
  border-radius: 16px;
  padding: 2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 250px;
}

.club-logo {
  max-width: 100%;
  max-height: 250px;
  object-fit: contain;
}

.club-description-card {
  background: white;
  border: 2px solid #ddd;
  border-radius: 16px;
  padding: 2rem;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.club-description {
  font-size: 1rem;
  line-height: 1.6;
  color: #333;
  margin-bottom: 1.5rem;
}

.club-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.tag {
  background: #f0f0f0;
  padding: 0.4rem 0.8rem;
  border-radius: 20px;
  font-size: 0.9rem;
  color: #555;
  border: 1px solid #ddd;
}

.section-title {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 1rem;
  color: #333;
}

.club-events-section {
  background: white;
  border: 2px solid #ddd;
  border-radius: 16px;
  padding: 2rem;
  margin-bottom: 2rem;
}

.events-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.event-item {
  display: flex;
  gap: 1rem;
  padding: 1rem;
  background: #f9f9f9;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.2s;
  border: 1px solid #eee;
}

.event-item:hover {
  background: #f0f0f0;
  transform: translateX(5px);
}

.event-image {
  width: 150px;
  height: 100px;
  object-fit: cover;
  border-radius: 8px;
  flex-shrink: 0;
}

.event-details {
  flex: 1;
  display: flex;
  align-items: center;
}

.event-description {
  font-size: 0.95rem;
  line-height: 1.5;
  color: #555;
}

.no-events {
  text-align: center;
  padding: 2rem;
  color: #999;
}

.relevant-clubs-section {
  background: white;
  border: 2px solid #ddd;
  border-radius: 16px;
  padding: 2rem;
  margin-bottom: 2rem;
}

.clubs-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1.5rem;
}

.club-card {
  aspect-ratio: 1;
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  transition: transform 0.2s;
  border: 2px solid #eee;
}

.club-card:hover {
  transform: scale(1.05);
}

.club-thumbnail {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.placeholder-cards {
  display: contents;
}

.club-card-placeholder {
  aspect-ratio: 1;
  background: #e0e0e0;
  border-radius: 12px;
  border: 2px solid #ddd;
}

.contact-info-section {
  background: white;
  border: 2px solid #ddd;
  border-radius: 16px;
  padding: 2rem;
}

.contact-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 2rem;
}

.contact-column {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.contact-item {
  font-size: 0.9rem;
}

.contact-item strong {
  display: block;
  color: #333;
  margin-bottom: 0.3rem;
  font-size: 0.95rem;
}

.contact-item p {
  color: #666;
  margin: 0.2rem 0;
  line-height: 1.4;
}

.social-icons {
  display: flex;
  gap: 0.8rem;
  margin-top: 0.5rem;
  flex-wrap: wrap;
}

.social-icon {
  padding: 0.6rem 1.2rem;
  background: #333;
  color: white;
  text-decoration: none;
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 600;
  transition: all 0.2s;
  display: inline-block;
}

.social-icon:hover {
  transform: scale(1.05);
}

.social-icon.discord {
  background: #5865F2;
}

.social-icon.instagram {
  background: linear-gradient(45deg, #f09433 0%, #e6683c 25%, #dc2743 50%, #cc2366 75%, #bc1888 100%);
}

.social-icon.wechat {
  background: #07C160;
}

.social-icon.linkedin {
  background: #0077B5;
}

@media (max-width: 1024px) {
  .club-header-section {
    grid-template-columns: 1fr;
  }
  
  .contact-grid {
    grid-template-columns: 1fr;
  }
  
  .clubs-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 768px) {
  .club-detail-page {
    padding: 1rem;
  }
  
  .clubs-grid {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .event-item {
    flex-direction: column;
  }
  
  .event-image {
    width: 100%;
    height: 150px;
  }
}

@media (max-width: 576px) {
  .back-button {
    width: 40px;
    height: 40px;
    font-size: 1.2rem;
  }
  
  .section-title {
    font-size: 1.2rem;
  }
  
  .clubs-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 1rem;
  }
}
</style>
