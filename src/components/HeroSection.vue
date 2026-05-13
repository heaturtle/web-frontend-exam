<template>
  <section class="hero">
    <img
      src="@/assets/images/Background-01.png"
      class="hero__background"
      alt=""
    />

    <div class="hero__overlay"></div>

    <div class="hero__character-wrapper">
      <img
        src="@/assets/images/Character-01-White.png"
        class="hero__character-shadow"
        alt=""
      />

      <img
        src="@/assets/images/Character-01.png"
        class="hero__character"
        alt=""
      />

      <img
        ref="leftEye"
        src="@/assets/images/LeftEye-01.png"
        class="eye eye--left"
        alt=""
      />

      <img
        ref="rightEye"
        src="@/assets/images/RightEye-01.png"
        class="eye eye--right"
        alt=""
      />
    </div>

    <img src="@/assets/images/Logo-01.png" class="hero__logo" alt="" />
  </section>

  <section class="job-section">
    <div class="job-section__container">
      <div class="job-section__title-wrapper">
        <h2 class="job-section__title">適合前端工程師的好工作</h2>
      </div>

      <div class="filter">
        <div class="filter__group">
          <label class="filter__label"> 公司名稱 </label>

          <input
            v-model="companyKeyword"
            class="filter__input"
            type="text"
            placeholder="請輸入公司名稱"
          />
        </div>

        <div class="filter__group">
          <label class="filter__label"> 教育程度 </label>

          <select v-model="selectedEducation" class="filter__select">
            <option value="">不限</option>

            <option
              v-for="item in educationList"
              :key="item.id"
              :value="item.id"
            >
              {{ item.label }}
            </option>
          </select>
        </div>

        <div class="filter__group">
          <label class="filter__label"> 薪水範圍 </label>

          <select v-model="selectedSalary" class="filter__select">
            <option value="">不限</option>

            <option v-for="item in salaryList" :key="item.id" :value="item.id">
              {{ item.label }}
            </option>
          </select>
        </div>

        <button class="filter__button">條件搜尋</button>
      </div>

      <div class="job-grid">
        <article
          v-for="job in paginatedJobList"
          :key="job.companyName"
          class="job-card"
        >
          <h3 class="job-card__title">
            {{ job.companyName }}
          </h3>

          <div class="job-card__meta">👤 {{ job.jobTitle }}</div>

          <div class="job-card__meta">
            🎓
            {{ getEducationLabel(job.educationId) }}
          </div>

          <div class="job-card__meta">
            💰
            {{ getSalaryLabel(job.salaryId) }}
          </div>

          <p class="job-card__description">
            {{ job.preview }}
          </p>

          <a href="#" class="job-card__button"> 查看細節 </a>
        </article>
      </div>

      <div class="pagination">
        <!-- 上一頁 -->
        <button class="pagination__item" @click="goToPreviousPage">‹</button>

        <!-- 頁碼 -->
        <button
          v-for="page in totalPages"
          :key="page"
          class="pagination__item"
          :class="{
            'pagination__item--active': currentPage === page,
          }"
          @click="changePage(page)"
        >
          {{ page }}
        </button>

        <!-- 下一頁 -->
        <button class="pagination__item" @click="goToNextPage">›</button>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from "vue";

import educationList from "@/constants/educationList";
import salaryList from "@/constants/salaryList";
import jobList from "@/constants/jobList";
const companyKeyword = ref("");
const selectedEducation = ref("");
const selectedSalary = ref("");
const getEducationLabel = (educationId: number): string => {
  const education = educationList.find(
    (item: { id: number; label: string }) => item.id === educationId,
  );

  return education?.label || "不限";
};

const getSalaryLabel = (salaryId: number): string => {
  const salary = salaryList.find(
    (item: { id: number; label: string }) => item.id === salaryId,
  );

  return salary?.label || "面議";
};

const currentPage = ref(1);
const ITEMS_PER_PAGE = 6;
const totalPages = computed(() => {
  return Math.ceil(filteredJobList.value.length / ITEMS_PER_PAGE);
});
const filteredJobList = computed(() => {
  return jobList.filter((job: any) => {
    const matchCompany = job.companyName.includes(companyKeyword.value);

    const matchEducation =
      !selectedEducation.value ||
      String(job.educationId) === selectedEducation.value;

    const matchSalary =
      !selectedSalary.value || String(job.salaryId) === selectedSalary.value;

    return matchCompany && matchEducation && matchSalary;
  });
});
const paginatedJobList = computed(() => {
  const startIndex = (currentPage.value - 1) * ITEMS_PER_PAGE;

  const endIndex = startIndex + ITEMS_PER_PAGE;

  return filteredJobList.value.slice(startIndex, endIndex);
});

const changePage = (page: number): void => {
  currentPage.value = page;
};

const goToPreviousPage = (): void => {
  if (currentPage.value > 1) {
    currentPage.value -= 1;
  }
};

const goToNextPage = (): void => {
  if (currentPage.value < totalPages.value) {
    currentPage.value += 1;
  }
};

const leftEye = ref<HTMLImageElement | null>(null);

const rightEye = ref<HTMLImageElement | null>(null);

const MAX_EYE_MOVEMENT = 2;

const updateEyePosition = (eye: HTMLImageElement, event: MouseEvent): void => {
  const rect = eye.getBoundingClientRect();

  const centerX = rect.left + rect.width / 2;

  const centerY = rect.top + rect.height / 2;

  const angle = Math.atan2(event.clientY - centerY, event.clientX - centerX);

  const moveX = Math.cos(angle) * MAX_EYE_MOVEMENT;

  const moveY = Math.sin(angle) * MAX_EYE_MOVEMENT;

  eye.style.transform = `translate(${moveX}px, ${moveY}px)`;
};

const handleMouseMove = (event: MouseEvent): void => {
  if (leftEye.value) {
    updateEyePosition(leftEye.value, event);
  }

  if (rightEye.value) {
    updateEyePosition(rightEye.value, event);
  }
};

onMounted(() => {
  window.addEventListener("mousemove", handleMouseMove);
});

onUnmounted(() => {
  window.removeEventListener("mousemove", handleMouseMove);
});
</script>

<style scoped lang="scss">
@use "@/assets/styles/hero-section.scss";
</style>
