<template>
  <div class="carousel-root">
    <header class="carousel-header">
      <h1>Галерея майбутнього</h1>
      <p class="carousel-description">
        <span class="desc-accent">Обирай найкращі фото одним кліком — твій вибір світиться у списку нижче!</span>
      </p>
    </header>
    <div class="carousel-window">
      <div class="carousel-track" :style="trackStyle" @transitionend="handleTransitionEnd">
        <div
          v-for="(img, i) in visibleImages"
          :key="img.url"
          class="carousel-img"
          :class="{ selected: selected.includes(img.url) }"
          @click="toggleSelect(img.url)"
          tabindex="0"
        >
          <img :src="img.url" :alt="'image-' + i" />
        </div>
      </div>
      <div class="carousel-nav">
        <button class="nav-btn prev" @click="prev" aria-label="Назад">&#8592;</button>
        <button class="nav-btn next" @click="next" aria-label="Далі">&#8594;</button>
      </div>
    </div>
    <div class="carousel-indicators">
      <span
        v-for="(img, i) in images"
        :key="img.url"
        :class="['carousel-indicator', { active: currentIndex === i }]"
        @click="goTo(i)"
      ></span>
    </div>
    <h3 class="selected-title">Вибрані зображення:</h3>
    <div class="selected-list-outer">
      <div class="selected-list-container">
        <transition-group name="selected-list" tag="ul" class="selected-list">
          <li v-for="url in selected" :key="url">{{ url }}</li>
        </transition-group>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, watch, onMounted } from "vue";
export default defineComponent({
  name: "ImageCarousel",
  props: {
    images: { type: Array, required: true }
  },
  setup(props) {
    const selected = ref<string[]>([]);
    const currentIndex = ref(0);
    const slidesToShow = ref(1);

    // Адаптація кількості видимих слайдів до ширини екрану
    const updateSlidesToShow = () => {
      const w = window.innerWidth;
      if (w < 600) slidesToShow.value = 1;
      else if (w < 900) slidesToShow.value = 2;
      else if (w < 1200) slidesToShow.value = 3;
      else slidesToShow.value = 4;
    };
    onMounted(() => {
      updateSlidesToShow();
      window.addEventListener("resize", updateSlidesToShow);
    });

    // Оновити індекс та вибране при зміні масиву зображень
    watch(() => props.images, () => {
      currentIndex.value = 0;
      selected.value = [];
    });

    // Визначення видимих зображень для каруселі
    const visibleImages = computed(() => {
      const imgs = props.images;
      const count = slidesToShow.value;
      const result = [];
      for (let i = 0; i < count; i++) {
        result.push(imgs[(currentIndex.value + i) % imgs.length]);
      }
      return result;
    });

    // Стиль для треку каруселі (для transition)
    const trackStyle = computed(() => ({
      transform: `translateX(0)`,
      transition: "transform 0.4s cubic-bezier(0.65,0,0.35,1)",
    }));

    // Перехід до попереднього слайда
    function prev() {
      currentIndex.value =
        (currentIndex.value - 1 + props.images.length) % props.images.length;
    }
    // Перехід до наступного слайда
    function next() {
      currentIndex.value = (currentIndex.value + 1) % props.images.length;
    }
    // Перехід до конкретного індексу
    function goTo(idx: number) {
      currentIndex.value = idx;
    }
    function handleTransitionEnd() {}
    // Вибір/зняття вибору зображення
    function toggleSelect(url: string) {
      const idx = selected.value.indexOf(url);
      if (idx === -1) selected.value.push(url);
      else selected.value.splice(idx, 1);
    }

    // Клавіатурна навігація ← →
    onMounted(() => {
      window.addEventListener('keydown', e => {
        if (e.key === 'ArrowLeft') prev();
        if (e.key === 'ArrowRight') next();
      });
    });

    return {
      selected,
      visibleImages,
      prev,
      next,
      goTo,
      toggleSelect,
      trackStyle,
      handleTransitionEnd,
      currentIndex,
      images: props.images
    };
  },
});
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css?family=Orbitron:700&display=swap');

.carousel-root {
  width: 100vw;
  min-height: 100vh;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: radial-gradient(ellipse at 60% 30%, #181a2b 70%, #23213a 100%);
  backdrop-filter: blur(3px);
  overflow-x: hidden;
  overflow-y: auto;
}

/* --- Футуристичний заголовок --- */
.carousel-header {
  width: 100vw;
  max-width: 1200px;
  text-align: center;
  margin: 0 0 18px 0;
  padding-top: 34px;
}
.carousel-header h1 {
  font-family: 'Orbitron', 'Segoe UI', Arial, sans-serif;
  font-size: 2.8em;
  font-weight: 700;
  color: #45faff;
  letter-spacing: 0.04em;
  margin-bottom: 8px;
  text-shadow: 0 0 32px #45faff, 0 0 12px #4f5bff;
  background: linear-gradient(90deg, #45faff 0%, #0cf 70%, #4f5bff 100%);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 2px 32px #45faff);
}
.carousel-description {
  font-family: 'Segoe UI', Arial, sans-serif;
  color: #eaf8f3;
  font-size: 1.35em;
  margin: 0 auto 8px auto;
  max-width: 700px;
  text-align: center;
  letter-spacing: 0.02em;
  text-shadow: 0 0 20px #0cf, 0 0 2px #444;
  opacity: 0.96;
  background: none;
  font-weight: 400;
  /* Елегантне підкреслення з неоновою анімацією */
  position: relative;
  padding-bottom: 10px;
}
.carousel-description .desc-accent {
  color: #45faff;
  font-weight: 600;
  text-shadow: 0 0 16px #45faff, 0 0 4px #4f5bff;
  padding-bottom: 2px;
  background: linear-gradient(90deg, #45faff 0%, #4f5bff 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: accent-glow 2.3s infinite alternate;
}
@keyframes accent-glow {
  0% { text-shadow: 0 0 16px #45faff, 0 0 4px #4f5bff; }
  100% { text-shadow: 0 0 32px #45faff, 0 0 14px #4f5bff; }
}

/* --- Карусель --- */
.carousel-window {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: rgba(30,35,55,0.82);
  border-radius: 38px;
  box-shadow: 0 8px 76px 0 rgba(0,255,255,0.13), 0 4px 52px 0 rgba(0,0,0,0.35);
  padding: 24px 20px 18px 20px;
  position: relative;
  overflow: visible;
  width: 150vw;
  max-width: 1250px;
  min-height: 320px;
  border: 2px solid rgba(87,255,255,0.25);
  margin-bottom: 18px;
}

.carousel-track {
  width: 100%;
  display: flex;
  justify-content: center;
  gap: 36px;
  align-items: center;
}

.carousel-img {
  border-radius: 28px;
  overflow: hidden;
  box-shadow: 0 8px 56px 0 rgba(0,255,255,0.13), 0 2px 20px rgba(0,0,0,0.2);
  cursor: pointer;
  border: 3px solid transparent;
  background: rgba(44,62,80,0.85);
  transition: border-color 0.3s, transform 0.35s, box-shadow 0.35s, filter 0.25s, width 0.3s, height 0.3s;
  position: relative;
  outline: none;
  filter: drop-shadow(0 0 24px #45faff);
  flex: 1 1 0;
  min-width: 0;
  max-width: 100%;
  width: 310px;
  height: 210px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.carousel-img.selected {
  border-color: #45faff;
  box-shadow: 0 8px 44px 0 #45faff, 0 2px 18px rgba(0,0,0,0.2);
  filter: drop-shadow(0 0 36px #45faff);
}

.carousel-img.selected::after {
  content: '';
  position: absolute;
  top: 18px; right: 18px;
  width: 40px; height: 40px;
  background: url('data:image/svg+xml;utf8,<svg fill="black" viewBox="0 0 24 24" stroke="cyan" xmlns="http://www.w3.org/2000/svg"><path stroke-width="3" stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7"/></svg>') no-repeat center;
  border-radius: 50%;
  background-color: rgba(69,250,255,0.85);
  box-shadow: 0 2px 12px rgba(69,250,255,0.15);
}

.carousel-img:hover, .carousel-img:focus {
  transform: scale(1.09) rotateZ(-2deg);
  box-shadow: 0 16px 84px #45faff;
  border-color: #0cf;
  filter: brightness(1.08) drop-shadow(0 0 64px #45faff);
}

.carousel-img img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 28px;
  filter: grayscale(0.08) brightness(0.98) contrast(1.06);
  transition: filter 0.25s;
}

/* --- Кнопки навігації --- */
.nav-btn {
  background: linear-gradient(90deg, #232d3c 0%, #45faff 100%);
  border: none;
  color: #45faff;
  font-size: 1.2em;
  padding: 0.14em 0.8em;
  border-radius: 12px;
  cursor: pointer;
  margin: 34px;
  box-shadow: 0 0 22px #45faff, 0 0 8px #4f5bff inset;
  transition: background 0.23s, color 0.23s, box-shadow 0.23s, transform 0.18s;
  z-index: 2;
  position: relative;
  font-family: 'Orbitron', 'Segoe UI', Arial, sans-serif;
  font-weight: 700;
  letter-spacing: 0.08em;
  border: 2px solid #45faff;
  outline: none;
}

.nav-btn:hover, .nav-btn:focus {
  background: linear-gradient(90deg, #45faff 0%, #4f5bff 100%);
  color: #232d3c;
  box-shadow: 0 0 38px #45faff, 0 0 15px #4f5bff inset, 0 0 14px #fff inset;
  transform: scale(1.11) translateY(-2px);
  border-color: #4f5bff;
}

.nav-btn:active {
  background: linear-gradient(90deg, #4f5bff 0%, #45faff 100%);
  color: #fff;
  box-shadow: 0 0 22px #4f5bff;
  transform: scale(0.97) translateY(1px);
}

/* --- Індикатори слайдів --- */
.carousel-indicators {
  display: flex;
  justify-content: center;
  margin: 18px 0 6px 0;
  gap: 12px;
}
.carousel-indicator {
  width: 20px; height: 20px;
  border-radius: 50%;
  background: #232d3c;
  border: 2px solid #45faff;
  box-shadow: 0 0 18px #45faff;
  transition: background 0.2s, border-color 0.2s;
  cursor: pointer;
  opacity: 0.7;
}
.carousel-indicator.active {
  background: #45faff;
  border-color: #4f5bff;
  opacity: 1;
}

/* --- Заголовок вибраного списку --- */
.selected-title {
  width: 100vw;
  max-width: 850px;
  margin: 16px 0 8px 0;
  font-family: 'Orbitron', 'Segoe UI', Arial, sans-serif;
  font-size: 1.2em;
  font-weight: 700;
  color: #45faff;
  text-align: center;
  letter-spacing: 0.05em;
  text-shadow: 0 0 10px #45faff, 0 0 2px #4f5bff;
  opacity: 0.89;
}

/* --- Список вибраних (скрол) --- */
.selected-list-outer {
  position: relative;
  width: 100vw;
  min-height: 120px;
  display: flex;
  align-items: flex-start;
  justify-content: center;
}
.selected-list-container {
  width: 95vw;
  min-height: 70px;
  max-height: 238px;
  overflow-y: auto;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  scrollbar-width: thin;
  scrollbar-color: #45faff #232d3c;
  padding-bottom: 6px;
}

.selected-list {
  max-width: 95vw;
  word-break: break-all;
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 2px;
  justify-content: flex-start;
  width: 100%;
}
.selected-list li {
  background: rgba(69,250,255,0.13);
  padding: 0.4em 1.2em;
  border-radius: 14px;
  transition: all 0.5s;
  font-size: 1.01em;
  margin: 0;
  color: #45faff;
  border: 1.2px solid #45faff;
  box-shadow: 0 0 8px #45faff;
  letter-spacing: 0.03em;
  text-shadow: 0 0 4px #45faff, 0 0 1px #4f5bff;
  white-space: nowrap;
  overflow-x: auto;
  height: 32px;
  display: flex;
  align-items: center;
}
.selected-list-enter-active, .selected-list-leave-active {
  transition: all 0.5s cubic-bezier(0.65,0,0.35,1);
}
.selected-list-enter-from {
  opacity: 0;
  transform: translateX(-60px) scale(0.8);
}
.selected-list-leave-to {
  opacity: 0;
  transform: translateX(60px) scale(1.2);
}

/* --- Неоновий скролбар для списку вибраних --- */
.selected-list-container::-webkit-scrollbar {
  width: 8px;
  background: transparent;
}
.selected-list-container::-webkit-scrollbar-thumb {
  background: linear-gradient(180deg, #45faff 40%, #4f5bff 100%);
  border-radius: 4px;
}
.selected-list-container::-webkit-scrollbar-track {
  background: transparent;
}

/* --- Адаптація --- */
@media (max-width: 1400px) {
  .carousel-header h1 { font-size: 2.1em; }
  .carousel-description { font-size: 1.13em; }
  .carousel-window { max-width: 1050px; }
  .carousel-img { width: 220px; height: 145px; }
}
@media (max-width: 900px) {
  .carousel-header { max-width: 100vw; padding-top: 11px; }
  .carousel-header h1 { font-size: 1.55em; }
  .carousel-description { font-size: 1em; }
  .carousel-window { max-width: 100vw; min-height: 160px; padding: 7px 2px 2px 2px; border-radius: 16px; }
  .carousel-track { gap: 8px; }
  .carousel-img { height: clamp(120px, 35vw, 240px); border-radius: 14px; width: 140px; }
  .selected-title { font-size: 1.03em; margin: 12px 0 5px 0; }
  .selected-list-outer { min-height: 90px; }
  .selected-list-container { min-height: 60px; max-height: 154px; }
}

@media (max-width: 600px) {
  .carousel-header { max-width: 100vw; padding-top: 6px; margin-bottom: 4px; }
  .carousel-header h1 { font-size: 1.1em; margin-bottom: 2px; }
  .carousel-description { font-size: 0.92em; margin-bottom: 4px; }
  .carousel-root { padding: 0; min-height: 100vh; width: 100vw; }
  .carousel-window { padding: 0; border-radius: 0; width: 100vw; max-width: 100vw; background: rgba(30,35,55,0.94); min-height: 0; box-shadow: none; margin-bottom: 7px; }
  .carousel-track { gap: 0; min-height: 110px; }
  .carousel-img { width: 100vw; max-width: 100vw; height: clamp(90px, 42vw, 200px); border-radius: 0; box-shadow: none; margin: 0; }
  .carousel-nav { gap: 24px; margin-top: 14px; }
  .nav-btn { font-size: 1.2em; padding: 0.2em 0.7em; border-radius: 10px; margin: 0 2px; border-width: 2px; }
  .carousel-indicators { gap: 7px; margin: 9px 0 2px 0; }
  .carousel-indicator { width: 14px; height: 14px; border-width: 1.5px; }
  .selected-title { font-size: 0.98em; margin: 7px 0 4px 0; }
  .selected-list-outer { min-height: 56px; padding-bottom: 4px; }
  .selected-list-container { min-height: 38px; max-height: 238px; padding-bottom: 2px; }
  .selected-list { gap: 2px; font-size: 0.93em; max-width: 99vw; }
  .selected-list li { padding: 0.2em 0.7em; border-radius: 7px; font-size: 0.97em; height: 32px; }
}
</style>