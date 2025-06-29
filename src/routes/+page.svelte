<script lang="ts">
  import { onMount, tick } from 'svelte';
  import { fade, fly, scale } from 'svelte/transition';
  import { page } from '$app/stores';
  import { derived } from 'svelte/store';

  // Placeholder wedding date (YYYY-MM-DDTHH:MM:SS)
  const weddingDate = new Date('2025-08-24T12:20:00');

  // Get guest name from URL param
  const guestName = derived(page, ($page) => {
    const params = new URLSearchParams($page.url.search);
    return params.get('guest') || 'Dear Guest';
  });

  // Countdown logic
  let countdown = { days: 0, hours: 0, minutes: 0, seconds: 0, finished: false };
  let interval: ReturnType<typeof setInterval>;

  function updateCountdown() {
    const now = new Date();
    const diff = weddingDate.getTime() - now.getTime();
    if (diff <= 0) {
      countdown = { days: 0, hours: 0, minutes: 0, seconds: 0, finished: true };
      clearInterval(interval);
      return;
    }
    countdown = {
      days: Math.floor(diff / (1000 * 60 * 60 * 24)),
      hours: Math.floor((diff / (1000 * 60 * 60)) % 24),
      minutes: Math.floor((diff / (1000 * 60)) % 60),
      seconds: Math.floor((diff / 1000) % 60),
      finished: false
    };
  }

  onMount(() => {
    updateCountdown();
    interval = setInterval(updateCountdown, 1000);
    return () => clearInterval(interval);
  });

  // RSVP options
  let bringingChildren = false;
  let plusOne = false;
  let dietaryNeeds = false;
  let rsvpSent = false;
  let rsvpLoading = false;
  let rsvpError = '';
  let appearAtCeremony = false;

  async function sendRSVP(name: string) {
    rsvpLoading = true;
    rsvpError = '';
    try {
      // Placeholder Telegram bot URL
      const url1 = `https://api.telegram.org/bot8189926925:AAFpLohjTEIjtBwavVozi5bPODy8nKxTdtg/sendMessage?chat_id=1787011904&text=${encodeURIComponent(name)} приняли приглашение! Дети - ${bringingChildren}; В загсе - ${appearAtCeremony}`
      const url2 = `https://api.telegram.org/bot8189926925:AAFpLohjTEIjtBwavVozi5bPODy8nKxTdtg/sendMessage?chat_id=351234630&text=${encodeURIComponent(name)} приняли приглашение! Дети - ${bringingChildren}; В загсе - ${appearAtCeremony}`
      await fetch(url1);
      await fetch(url2);
      rsvpSent = true;
    } catch (e) {
      rsvpError = 'Не увалось подтвердить участие. Пожалуйста, попробуйте снова.';
    } finally {
      rsvpLoading = false;
    }
  }

  // Data for schedule cards
  const schedule = [
    {
      title: 'Церемония',
      time: '12:20',
      img: 'https://avatars.mds.yandex.net/get-altay/5498343/2a0000017da3be50363be347b3717233cf09/XXXL',
      desc: 'Торжественная церемония бракосочетания. Зал вместит всех желающих'
    },
    {
      title: 'Трансфер',
      time: '13:30',
      img: 'https://images.unsplash.com/photo-1503376780353-7e6692767b70?auto=format&fit=crop&w=450&q=80', // автобус/транспорт
      desc: 'Организованный трансфер гостей от ЗАГСа до площадки'
    },
    {
      title: 'Фуршет',
      time: '14:00',
      img: 'https://avatars.mds.yandex.net/get-altay/13063086/2a000001940d5e15343b61d6db0013123304/XXXL', // фуршет, закуски
      desc: 'Еда и напитки для гостей.'
    },
    {
      title: 'Банкет',
      time: '15:00',
      img: 'https://avatars.mds.yandex.net/get-altay/13212052/2a00000190911495ed68811963cdf24de663/XXXL', // банкет на природе, гости, зелень
      desc: 'Праздничный банкет, поздравления и конкурсы.'
    },
    {
      title: 'Танцы',
      time: '20:00',
      img: 'https://media1.tenor.com/m/Yo7x_bQgHtcAAAAC/dance-nsjdnsnd.gif',
      desc: 'Танцы и веселье до поздней ночи!'
    }
  ];

  // Data for wishes cards
  const wishes = [
    {
      title: 'О Дресс-коде',
      img: 'https://cymbeline.com/wp-content/uploads/2024/07/SPIREE-2-scaled.jpg', // стильная одежда
      desc: 'На нашей свадьбе не будет определенного дресс-кода, приходите в комфортной для вас одежде!'
    },
    {
      title: 'О подарках',
      img: 'https://cs5.livemaster.ru/storage/d8/94/dca7ed97ea6195c58311169d89lx--svadebnyj-salon-konvert-dlya-deneg-na-svadbu.jpg', // подарок/коробка
      desc: 'Пожалуйста, не дарите на цветы! Мы не успеем насладиться их красотой и ароматом. Если хотите подарить нам ценный и нужный подарок, мы будем очень благодарны за вклад в бюджет нашей молодой семьи.'
    },
    {
      title: 'О комфорте',
      img: 'data:image/jpeg;base64,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', // плед/открытая площадка
      desc: 'Свадебный вечер пройдет на открытой площадке. Для вашего комфорта просим приготовить верхнюю одежду (и плавки по желанию).'
    },

    {
      title: 'О поздравлении',
      img: 'https://images.unsplash.com/photo-1511671782779-c97d3d27a1d4?auto=format&fit=crop&w=400&q=80', // микрофон/аплодисменты
      desc: 'Мы поощряем ваше творчество, поэтому будем рады вашему творческому подходу к поздравлению!'
    }
  ];

  // inView action for fade-in on scroll
  function inView(node: HTMLElement, { threshold = 0.01 } = {}) {
    let observer: IntersectionObserver;
    function handle(entries: IntersectionObserverEntry[]) {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          node.classList.add('fade-in-section');
          observer.disconnect();
        }
      });
    }
    observer = new IntersectionObserver(handle, { threshold });
    observer.observe(node);
    // Fallback: if already visible on mount, add class immediately
    setTimeout(() => {
      const rect = node.getBoundingClientRect();
      if (
        rect.top < window.innerHeight &&
        rect.bottom > 0
      ) {
        node.classList.add('fade-in-section');
        observer.disconnect();
      }
    }, 0);
    return {
      destroy() {
        observer.disconnect();
      }
    };
  }

  let mapImageError = false;

  // RSVP checkbox visibility from URL params
  const showChildren = derived(page, ($page) => {
    const params = new URLSearchParams($page.url.search);
    return !!params.get('children');
  });
  const showCeremony = derived(page, ($page) => {
    const params = new URLSearchParams($page.url.search);
    return !!params.get('ceremony');
  });

  // Функция для правильных окончаний (русский)
  function plural(n: number, forms: [string, string, string]) {
    return forms[
      n % 10 === 1 && n % 100 !== 11
        ? 0
        : n % 10 >= 2 && n % 10 <= 4 && (n % 100 < 10 || n % 100 >= 20)
        ? 1
        : 2
    ];
  }

  let showFloatingBtn = true;
  let rsvpButton: HTMLButtonElement | null = null;
  let rsvpButtonVisible = false;
  onMount(() => {
    let observer: IntersectionObserver | null = null;
    tick().then(() => {
      if (typeof window !== 'undefined' && rsvpButton) {
        observer = new window.IntersectionObserver(
          ([entry]) => {
            rsvpButtonVisible = entry.isIntersecting;
            showFloatingBtn = !rsvpButtonVisible;
          },
          { threshold: 0.1 }
        );
        observer.observe(rsvpButton);
      }
    });
    return () => {
      if (observer && rsvpButton) observer.disconnect();
    };
  });
  function scrollToRSVP() {
    const el = document.getElementById('rsvp');
    if (el) {
      el.scrollIntoView({ behavior: 'smooth', block: 'start' });
    }
  }
</script>

<!-- Google Fonts for elegant headers -->
<svelte:head>
  <link href="https://fonts.googleapis.com/css2?family=Great+Vibes&family=Montserrat:wght@400;700&display=swap" rel="stylesheet">
</svelte:head>

<style>
  :global(body) {
    margin: 0;
    font-family: 'Montserrat', Arial, sans-serif;
    background: linear-gradient(120deg, #f8fafc 0%, #e0e7ef 100%), url('https://www.transparenttextures.com/patterns/flowers.png');
    min-height: 100vh;
    background-blend-mode: lighten;
  }
  .container {
    max-width: 1100px;
    margin: 0 auto;
    padding: 2.5rem 1rem;
    background: rgba(255,255,255,0.92);
    border-radius: 2.5rem;
    box-shadow: 0 8px 32px rgba(60,60,100,0.08);
    margin-top: 2.5rem;
    margin-bottom: 2.5rem;
  }
  .hero-bg {
    width: 100vw;
    height: 400px;
    background: url('https://images.unsplash.com/photo-1506744038136-46273834b3fb?auto=format&fit=crop&w=1200&q=80'), linear-gradient(120deg, #f8fafc 0%, #e0e7ef 100%);
    background-size: cover;
    background-position: center;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #fff;
    font-size: 2.5rem;
    font-weight: 700;
    letter-spacing: 2px;
    margin-bottom: -140px;
    z-index: 0;
    position: relative;
    box-shadow: 0 8px 32px rgba(60,60,100,0.10);
    text-shadow: 0 2px 8px rgba(60,60,100,0.18);
  }
  .hero-bg .hero-title {
    font-family: 'Great Vibes', cursive;
    font-size: 3.8rem;
    font-weight: 400;
    letter-spacing: 3px;
    color: #fff;
    text-align: center;
    text-shadow: 0 2px 8px rgba(60,60,100,0.18);
  }
  .section {
    margin: 3.5rem 0;
    padding: 2.5rem 0;
    border-bottom: 1px solid #e5e7eb;
    min-height: 340px;
    transition: opacity 0.8s cubic-bezier(.4,0,.2,1), transform 0.8s cubic-bezier(.4,0,.2,1);
  }
  .section:last-child {
    border-bottom: none;
  }
  .section:not(.fade-in-section) {
    opacity: 0;
    transform: translateY(60px);
  }
  .fade-in-section {
    opacity: 1 !important;
    transform: translateY(0) !important;
  }
  .location-preview {
    min-width: 0;
    width: 450px;
    height: 220px;
    border-radius: 1.5rem;
    overflow: hidden;
    position: relative;
    box-shadow: 0 2px 12px rgba(60,60,100,0.10);
  }

  .side-scroll {
    display: flex;
    overflow-x: auto;
    overflow-y: visible;
    gap: 2.5rem;
    padding-top: 1.5rem;
    padding-bottom: 2.5rem;
    padding-left: 2.5rem;
    padding-right: 2.5rem;
    scroll-snap-type: x mandatory;
    -webkit-overflow-scrolling: touch;
    margin-bottom: 0.5rem;
    position: relative;
  }
  .side-scroll::-webkit-scrollbar {
    height: 12px;
    background: #f1f5f9;
  }
  .side-scroll::-webkit-scrollbar-thumb {
    background: #e0e7ef;
    border-radius: 6px;
  }
  .card {
    min-width: 340px;
    max-width: 360px;
    background: #fff;
    border-radius: 1.5rem;
    box-shadow: 0 2px 16px rgba(60,60,100,0.13);
    padding: 2rem 1.5rem 2.2rem 1.5rem;
    display: flex;
    flex-direction: column;
    align-items: center;
    scroll-snap-align: start;
    transition: transform 0.18s, box-shadow 0.18s;
    cursor: grab;
    position: relative;
    border: 2px solid #f3e8ff;
  }
  .card:hover {
    transform: scale(1.04) translateY(-4px);
    box-shadow: 0 8px 32px rgba(168,85,247,0.13);
    border-color: #a855f7;
    z-index: 2;
  }
  .card-img {
    width: 100%;
    height: 200px;
    object-fit: cover;
    border-radius: 1.2rem;
    margin-bottom: 1.2rem;
    background: #f3e8ff;
  }
  .card-title {
    font-family: 'Great Vibes', cursive;
    font-size: 2.1rem;
    color: #a855f7;
    margin-bottom: 0.5rem;
    text-align: center;
  }
  .card-time {
    font-size: 1.3rem;
    color: #6366f1;
    margin-bottom: 0.5rem;
    font-weight: 600;
  }
  .card-desc {
    font-size: 1.2rem;
    color: #475569;
    text-align: center;
  }
  .rsvp-btn {
    background: linear-gradient(90deg, #6366f1 0%, #60a5fa 100%);
    color: #fff;
    font-size: 1.3rem;
    padding: 1.1rem 3rem;
    border: none;
    border-radius: 2.5rem;
    cursor: pointer;
    font-weight: 600;
    box-shadow: 0 4px 16px rgba(60,60,100,0.10);
    transition: transform 0.15s, box-shadow 0.15s;
    margin-top: 2rem;
  }
  .rsvp-btn:hover {
    transform: translateY(-2px) scale(1.03);
    box-shadow: 0 8px 32px rgba(60,60,100,0.13);
  }
  .checkboxes-center {
    display: flex;
    justify-content: center;
    width: 100%;
    margin: 0 auto;
    max-width: 340px;
  }
  .checkboxes {
    display: flex;
    flex-direction: column;
    gap: 1.1rem;
    margin-top: 1.7rem;
    align-items: flex-start;
    justify-content: center;
  }
  .countdown {
    display: flex;
    gap: 2rem;
    justify-content: center;
    align-items: center;
    font-size: 2.3rem;
    font-weight: 600;
    color: #6366f1;
    margin: 2rem 0 2.5rem 0;
    letter-spacing: 1px;
    animation: pulse 1.5s infinite alternate;
  }
  .countdown .unit {
    display: flex;
    flex-direction: column;
    align-items: center;
    min-width: 80px;
    font-family: 'Montserrat', Arial, sans-serif;
    background: #f1f5f9;
    border-radius: 1rem;
    padding: 0.7rem 1.1rem;
    box-shadow: 0 2px 8px rgba(60,60,100,0.07);
    transition: background 0.3s;
    will-change: background;
    font-size: 1.7rem;
    margin: 0 0.3rem;
    animation: flipIn 0.7s cubic-bezier(.4,0,.2,1);
  }
  .countdown .label {
    font-size: 1.1rem;
    color: #64748b;
    font-family: 'Montserrat', Arial, sans-serif;
    font-weight: 400;
    margin-top: 0.2rem;
  }
  @keyframes pulse {
    from { opacity: 1; }
    to { opacity: 0.7; }
  }
  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(40px); }
    to { opacity: 1; transform: translateY(0); }
  }
  @keyframes flipIn {
    from { transform: rotateX(90deg); opacity: 0; }
    to { transform: rotateX(0deg); opacity: 1; }
  }
  .fade-in {
    animation: fadeIn 1.2s cubic-bezier(.4,0,.2,1) both;
  }
  .header-elegant {
    font-family: 'Great Vibes', cursive;
    font-size: 3rem;
    color: #a855f7;
    margin-bottom: 1.2rem;
    text-align: center;
    letter-spacing: 2px;
    font-weight: 400;
  }
  .subheader {
    font-family: 'Montserrat', Arial, sans-serif;
    font-size: 1.5rem;
    color: #64748b;
    text-align: center;
    margin-bottom: 1.5rem;
  }
  .map-link {
    display: block;
    position: relative;
    border-radius: 1.5rem;
    overflow: hidden;
    transition: box-shadow 0.2s;
  }
  .map-link:hover {
    box-shadow: 0 0 0 4px #a855f733;
    cursor: pointer;
  }
  .map-previews-row {
    display: flex;
    gap: 2rem;
    justify-content: center;
    margin-bottom: 1.2rem;
    flex-wrap: wrap;
  }
  @media (max-width: 700px) {
    .map-previews-row {
      flex-direction: column;
      align-items: center;
      gap: 1rem;
    }
  }
  .checkboxes label {
    display: flex;
    align-items: center;
    gap: 0.7rem;
    line-height: 1;
    font-size: 1.15rem;
    font-weight: 500;
    color: #222;
    cursor: pointer;
    padding: 0.4rem 0.7rem;
    border-radius: 1.2rem;
    transition: background 0.2s;
    justify-content: flex-start;
    text-align: left;
  }
  .checkboxes label:hover {
    background: #f3e8ff;
  }
  .checkboxes input[type="checkbox"] {
    appearance: none;
    width: 1.4em;
    height: 1.4em;
    border: 2px solid #a855f7;
    border-radius: 0.5em;
    background: #fff;
    transition: border-color 0.2s, box-shadow 0.2s;
    margin-right: 0.5em;
    position: relative;
    cursor: pointer;
    outline: none;
    box-shadow: 0 1px 4px rgba(168,85,247,0.08);
    display: inline-block;
    vertical-align: middle;
  }
  .checkboxes input[type="checkbox"]:checked {
    background: #a855f7;
    border-color: #a855f7;
  }
  .checkboxes input[type="checkbox"]:checked::after {
    content: '';
    display: block;
    position: absolute;
    left: 0.38em;
    top: 0.18em;
    width: 0.35em;
    height: 0.7em;
    border: solid #fff;
    border-width: 0 0.18em 0.18em 0;
    transform: rotate(45deg);
  }
  .floating-rsvp-btn {
    position: fixed;
    right: 2.2rem;
    bottom: 2.2rem;
    z-index: 1000;
    background: linear-gradient(90deg, #6366f1 0%, #60a5fa 100%);
    color: #fff;
    font-size: 1.15rem;
    padding: 1.1rem 2.2rem 1.1rem 2.7rem;
    border: none;
    border-radius: 2.5rem;
    cursor: pointer;
    font-weight: 600;
    box-shadow: 0 4px 24px rgba(60,60,100,0.18);
    display: flex;
    align-items: center;
    gap: 0.7rem;
    transition: transform 0.18s, box-shadow 0.18s, opacity 0.4s;
    opacity: 0.96;
    animation: floatIn 0.7s cubic-bezier(.4,0,.2,1);
  }
  .floating-rsvp-btn.hide {
    opacity: 0;
    pointer-events: none;
    visibility: hidden;
    transition: opacity 0.4s cubic-bezier(.4,0,.2,1), visibility 0.4s cubic-bezier(.4,0,.2,1);
  }
  .floating-rsvp-btn:hover {
    transform: translateY(-2px) scale(1.04);
    box-shadow: 0 8px 32px rgba(60,60,100,0.22);
    opacity: 1;
  }
  @keyframes floatIn {
    from { opacity: 0; transform: translateY(40px) scale(0.95); }
    to { opacity: 0.96; transform: translateY(0) scale(1); }
  }
  @media (max-width: 600px) {
    .floating-rsvp-btn {
      right: 1rem;
      bottom: 1rem;
      font-size: 1rem;
      padding: 0.9rem 1.5rem 0.9rem 2.1rem;
    }
  }
</style>

<div class="hero-bg fade-in">
  <span class="hero-title">Together Forever</span>
</div>
<div class="container fade-in" style="animation-delay:0.3s">
  <section class="section fade-in-section">
    <div class="subheader">Дорогие <span style="font-size:2.5rem; font-weight:700; color:#181818; font-family:'Great Vibes',cursive; font-style:italic; transition:color 0.3s;">{$guestName}!</span></div>
    <div class="header-elegant">Владимир & Екатерина</div>
    <div class="subheader">
      С радостью приглашаем вас на нашу свадьбу,<br>
      которая состоится <span style="font-weight:700;">24.08.2025</span> через
    </div>
    <div class="countdown">
      {#if countdown.finished}
        <span style="font-size:1.7rem; color:#22c55e;">Настал наш день!</span>
      {:else}
        <div class="unit" in:scale>{countdown.days}<span class="label">{plural(countdown.days, ['день', 'дня', 'дней'])}</span></div>
        <div class="unit" in:scale>{countdown.hours}<span class="label">{plural(countdown.hours, ['час', 'часа', 'часов'])}</span></div>
        <div class="unit" in:scale>{countdown.minutes}<span class="label">мин</span></div>
        <div class="unit" in:scale>{countdown.seconds}<span class="label">сек</span></div>
      {/if}
    </div>
    <div class="subheader" style="margin-top: 0.5rem;">в следующих локациях</div>
    <div class="map-previews-row">
      <div style="display: flex; flex-direction: column; align-items: center;">
        <a href="https://yandex.ru/maps/?ll=37.583374%2C55.798301&z=17&pt=37.583374,55.798301,pm2rdm" target="_blank" class="map-link">
          <div class="location-preview" in:fly={{ y: 30, duration: 900 }}>
            <img src="https://static-maps.yandex.ru/1.x/?ll=37.583374,55.798301&size=450,220&z=17&l=map&pt=37.583374,55.798301,pm2rdm" alt="ЗАГС" style="width:100%;height:100%;object-fit:cover;border-radius:1.5rem;display:block;" />
          </div>
        </a>
        <div style="font-size:1.1rem; margin-top:0.5rem; text-align:center;">ЗАГС: Дворец бракосочетания №4</div>
      </div>
      <div style="display: flex; flex-direction: column; align-items: center;">
        <a href="https://yandex.ru/maps/?ll=37.691631%2C55.888557&z=17&pt=37.691631,55.888557,pm2rdm" target="_blank" class="map-link">
          <div class="location-preview" in:fly={{ y: 30, duration: 900 }}>
            <img src="https://static-maps.yandex.ru/1.x/?ll=37.691631,55.888557&size=450,220&z=17&l=map&pt=37.691631,55.888557,pm2rdm" alt="Площадка" style="width:100%;height:100%;object-fit:cover;border-radius:1.5rem;display:block;" />
          </div>
        </a>
        <div style="font-size:1.1rem; margin-top:0.5rem; text-align:center;">Площадка: Бакинский бульвар</div>
      </div>
    </div>
  </section>

  <!-- Schedule: Side scrolling cards -->
  <section class="section" use:inView on:mouseenter={() => {}}>
    <div class="header-elegant">Программа</div>
    <div class="side-scroll">
      {#each schedule as event (event.title)}
        <div class="card" in:scale={{ duration: 400 }}>
          <img class="card-img" src={event.img} alt={event.title} />
          <div class="card-title">{event.title}</div>
          <div class="card-time">{event.time}</div>
          <div class="card-desc">{event.desc}</div>
        </div>
      {/each}
    </div>
  </section>

  <!-- Wishes: Side scrolling cards for dress code and gifts -->
  <section class="section" use:inView on:mouseenter={() => {}}>
    <div class="header-elegant">Пожелания</div>
    <div class="side-scroll">
      {#each wishes as wish (wish.title)}
        <div class="card" in:scale={{ duration: 400 }}>
          <img class="card-img" src={wish.img} alt={wish.title} />
          <div class="card-title">{wish.title}</div>
          <div class="card-desc">{wish.desc}</div>
        </div>
      {/each}
    </div>
  </section>

  <section class="section" id="rsvp" use:inView on:mouseenter={() => {}}>
    <div class="header-elegant">Анкета гостя</div>
    <div class="subheader" style="margin-bottom:1.2rem; font-size:1.18rem;">
      Просим подтвердить свое присутствие на мероприятии до:
      <div style="font-weight:700; color:#a855f7; font-size:1.25em; margin-top:0.3em;">24.07.2025</div>
    </div>
    <div class="checkboxes-center">
      <div class="checkboxes">
        {#if $showChildren}
          <label><input type="checkbox" bind:checked={bringingChildren}> Приду с детьми</label>
        {/if}
        {#if $showCeremony}
          <label><input type="checkbox" bind:checked={appearAtCeremony}> Буду на церемонии</label>
        {/if}
      </div>
    </div>
    {#if !rsvpSent}
      <div style="display:flex; justify-content:center; margin-top: 1.5rem;">
        <button class="rsvp-btn" bind:this={rsvpButton} on:click={() => $guestName && sendRSVP($guestName)} disabled={rsvpLoading}>
          {rsvpLoading ? 'Отправка...' : 'Подтвердить участие'}
        </button>
      </div>
      {#if rsvpError}
        <div style="color:#ef4444; margin-top:0.7rem; text-align:center;">{rsvpError}</div>
      {/if}
    {:else}
      <div style="color:#22c55e; font-size:1.4rem; margin-top:1.5rem; text-align:center;">Спасибо за подтверждение!</div>
    {/if}
    <div style="max-width:320px; margin:2.2rem auto 0 auto; text-align:left;">
      <div style="font-size:1.15rem; font-weight:600; color:#a855f7; margin-bottom:0.7rem; letter-spacing:0.5px;">Контакты</div>
      <div style="margin-bottom:0.5rem; display:flex; align-items:center; gap:0.5em;">
        <svg width="18" height="18" fill="none" viewBox="0 0 24 24"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z" fill="#6366f1"/></svg>
        <a href="https://t.me/RudaBooBa" style="color:#222; text-decoration:underline;">Жених</a>
      </div>
      <div style="margin-bottom:0.5rem; display:flex; align-items:center; gap:0.5em;">
        <svg width="18" height="18" fill="none" viewBox="0 0 24 24"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z" fill="#f472b6"/></svg>
        <a href="https://t.me/suprunova_katy" style="color:#222; text-decoration:underline;">Невеста</a>
      </div>
      <div style="margin-bottom:0.5rem; display:flex; align-items:center; gap:0.5em;">
        <svg width="18" height="18" fill="none" viewBox="0 0 24 24"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z" fill="#10b981"/></svg>
        <a href="https://t.me/realnormalno" style="color:#222; text-decoration:underline;">Заводила</a>
      </div>
    </div>
  </section>
</div>

{#if !rsvpSent}
  <button class="floating-rsvp-btn{showFloatingBtn ? '' : ' hide'}" on:click={scrollToRSVP} aria-label="Перейти к анкете гостя">
    <svg width="22" height="22" viewBox="0 0 22 22" fill="none" xmlns="http://www.w3.org/2000/svg" style="margin-right:0.5em;"><circle cx="11" cy="11" r="11" fill="#fff" fill-opacity="0.18"/><path d="M11 5V17M11 17L6 12M11 17L16 12" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
    Подтвердить участие
  </button>
{/if}
