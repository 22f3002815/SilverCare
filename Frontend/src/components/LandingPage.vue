<template>
  <div class="landing-wrapper">
    <!-- Navigation -->
    <nav class="navbar navbar-expand-lg fixed-top custom-navbar px-lg-5 py-3">
      <a class="navbar-brand" href="#">SilverCare<span>.</span></a>
      <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarNav">
        <ul class="navbar-nav ms-auto">
          <li class="nav-item">
            <a class="nav-link" href="#features">Features</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" href="#about">About</a>
          </li>
           <li class="nav-item">
            <a class="nav-link" href="#contact">Contact</a>
          </li>
        </ul>
        <div class="d-flex gap-2 ms-lg-4">
          <router-link to="/login" class="btn btn-outline">Login</router-link>
          <router-link to="/signup" class="btn btn-primary">Get Started</router-link>
        </div>
      </div>
    </nav>

    <!-- Hero Section -->
    <section class="hero-section d-flex align-items-center text-white">
      <div class="container text-center">
        <h1 class="hero-title">Compassionate Care for a Life of Dignity.</h1>
        <p class="hero-subtitle">
          Connecting you with skilled, compassionate providers for a better quality of life.
        </p>
        <router-link to="/signup" class="btn btn-primary btn-lg mt-4">
          Find Care Today
        </router-link>
      </div>
    </section>

    <!-- Features Section -->
    <section id="features" class="features-section">
      <div class="container">
        <div class="text-center mb-5">
          <h2 class="section-title">Peace of Mind for Your Family</h2>
          <p class="section-subtitle">Everything you need to ensure their well-being.</p>
        </div>
        <div class="row g-4">
          <div class="col-md-4">
            <div class="feature-card">
              <div class="feature-icon">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2a10 10 0 0 0-10 10 10 10 0 0 0 10 10 10 10 0 0 0 10-10A10 10 0 0 0 12 2Z"></path><path d="m14.3 14.3 4.25 4.25"></path><path d="M12 18a6 6 0 0 0 6-6 6 6 0 0 0-6-6 6 6 0 0 0-6 6 6 6 0 0 0 6 6Z"></path></svg>
              </div>
              <h3>Medication Reminders</h3>
              <p>Never miss a dose with timely alerts and a clear schedule for caregivers and seniors.</p>
            </div>
          </div>
          <div class="col-md-4">
            <div class="feature-card">
              <div class="feature-icon">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect><line x1="16" y1="2" x2="16" y2="6"></line><line x1="8" y1="2" x2="8" y2="6"></line><line x1="3" y1="10" x2="21" y2="10"></line></svg>
              </div>
              <h3>Daily Schedule</h3>
              <p>Organize appointments, activities, and daily routines in one accessible calendar.</p>
            </div>
          </div>
          <div class="col-md-4">
            <div class="feature-card">
              <div class="feature-icon">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path><circle cx="9" cy="7" r="4"></circle><path d="M23 21v-2a4 4 0 0 0-3-3.87"></path><path d="M16 3.13a4 4 0 0 1 0 7.75"></path></svg>
              </div>
              <h3>Caregiver Coordination</h3>
              <p>Seamlessly share updates and coordinate tasks between family members and professional caregivers.</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- About Section -->
    <section id="about" class="about-section">
      <div class="container">
        <div class="row align-items-center">
          <div class="col-lg-6">
            <img src="https://images.unsplash.com/photo-1576765608866-5b510410c1e8?q=80&w=600&auto=format&fit=crop" class="img-fluid rounded-3" alt="Happy senior citizen">
          </div>
          <div class="col-lg-6">
            <h2 class="section-title">Dedicated to a Life of Comfort and Dignity</h2>
            <p class="text-secondary">We are dedicated to providing the highest level of care to the elderly, ensuring their comfort, and independence. Our team of professionals is trained to handle a wide range of needs with compassion and respect.</p>
          </div>
        </div>
      </div>
    </section>

    <!-- Footer -->
    <footer id="contact" class="footer">
      <div class="container">
        <div class="row">
          <div class="col-lg-4 mb-4 mb-lg-0">
            <h4 class="footer-brand">SilverCare<span>.</span></h4>
            <p class="text-muted">Your trusted partner in senior care.</p>
          </div>
          <div class="col-lg-4 mb-4 mb-lg-0">
            <h5 class="footer-heading">Contact Us</h5>
            <ul class="list-unstyled">
              <li>Email: support@silvercare.com</li>
              <li>Phone: +91-8010676679</li>
            </ul>
          </div>
          <div class="col-lg-4">
            <h5 class="footer-heading">Follow Us</h5>
            <p class="text-muted">Stay connected for updates and tips.</p>
          </div>
        </div>
        <div class="text-center text-muted pt-4 mt-4 border-top">
          &copy; {{ new Date().getFullYear() }} SilverCare. All Rights Reserved.
        </div>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted } from 'vue';

const handleScroll = () => {
  const navbar = document.querySelector('.custom-navbar');
  if (window.scrollY > 50) {
    navbar.classList.add('scrolled');
  } else {
    navbar.classList.remove('scrolled');
  }
};

onMounted(() => {
  window.addEventListener('scroll', handleScroll);
});

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll);
});
</script>

<style scoped>
/* --- Font Import --- */
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap');

/* --- CSS Variables for Theming --- */
:root {
  --font-primary: 'Poppins', sans-serif;
  --primary-color: #005a9c; /* A deep, trustworthy blue */
  --secondary-color: #f8f9fa;
  --text-dark: #212529;
  --text-light: #ffffff;
  --text-muted: #6c757d;
  --border-color: #dee2e6;
  --shadow-sm: 0 4px 6px rgba(0,0,0,0.05);
  --border-radius: 12px;
}

/* --- Base Styles --- */
.landing-wrapper {
  font-family: var(--font-primary);
  color: var(--text-dark);
  scroll-behavior: smooth;
  background-color: #fff;
}

/* --- Navigation Bar --- */
.custom-navbar {
  transition: background-color 0.4s ease, padding 0.4s ease;
}
.custom-navbar.scrolled {
  background-color: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  box-shadow: var(--shadow-sm);
  padding-top: 0.75rem;
  padding-bottom: 0.75rem;
}
.custom-navbar.scrolled .navbar-brand,
.custom-navbar.scrolled .nav-link {
  color: var(--text-dark) !important;
}
.custom-navbar.scrolled .navbar-brand span {
  color: var(--primary-color) !important;
}
.custom-navbar.scrolled .btn-outline {
  border-color: var(--primary-color);
  color: var(--primary-color);
}
.custom-navbar.scrolled .btn-outline:hover {
  background-color: var(--primary-color);
  color: var(--text-light);
}

.navbar-brand {
  font-weight: 700;
  font-size: 1.75rem;
  color: var(--text-light);
}
.navbar-brand span {
  color: var(--primary-color);
}
.nav-link {
  font-weight: 500;
  color: var(--text-light);
  transition: color 0.3s ease;
}
.nav-link:hover {
  color: var(--primary-color);
}
.custom-navbar.scrolled .nav-link:hover {
  color: var(--primary-color) !important;
}

/* --- Buttons --- */
.btn {
  padding: 0.6rem 1.5rem;
  border-radius: 8px;
  font-weight: 600;
  transition: all 0.3s ease;
}
.btn-primary {
  background-color: var(--primary-color);
  border: 1px solid var(--primary-color);
  color: var(--text-light);
}
.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 15px rgba(0, 90, 156, 0.2);
}
.btn-outline {
  border: 1px solid var(--text-light);
  color: var(--text-light);
}
.btn-outline:hover {
  background-color: var(--text-light);
  color: var(--primary-color);
}

/* --- Hero Section --- */
.hero-section {
  background: linear-gradient(rgba(0, 30, 60, 0.6), rgba(0, 30, 60, 0.6)), url('https://images.unsplash.com/photo-1584515933487-779824d29309?q=80&w=1920&auto=format&fit=crop');
  background-size: cover;
  background-position: center;
  height: 100vh;
  min-height: 700px;
}
.hero-title {
  font-size: 3.5rem;
  font-weight: 700;
  line-height: 1.2;
}
.hero-subtitle {
  font-size: 1.25rem;
  max-width: 650px;
  margin: 1rem auto 0;
  opacity: 0.9;
}

/* --- General Section Styling --- */
.features-section, .about-section {
  padding: 6rem 0;
}
.section-title {
  font-size: 2.5rem;
  font-weight: 700;
  margin-bottom: 1rem;
}
.section-subtitle {
  font-size: 1.1rem;
  color: var(--text-muted);
  max-width: 600px;
  margin: 0 auto;
}

/* --- Features Section --- */
.features-section {
    background-color: #f8f9fa;
}
.feature-card {
  background-color: #fff;
  padding: 2.5rem 2rem;
  border-radius: var(--border-radius);
  text-align: center;
  box-shadow: var(--shadow-sm);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  height: 100%;
}
.feature-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 10px 20px rgba(0,0,0,0.08);
}
.feature-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background-color: #e6f0ff;
  color: var(--primary-color);
  margin-bottom: 1.5rem;
}
.feature-icon svg {
    width: 28px;
    height: 28px;
}
.feature-card h3 {
  font-size: 1.25rem;
  font-weight: 600;
  margin-bottom: 0.75rem;
}
.feature-card p {
  color: var(--text-muted);
  font-size: 0.95rem;
}

/* --- About Section --- */
.about-section img {
    box-shadow: var(--shadow-sm);
}

/* --- Footer --- */
.footer {
  background-color: var(--text-dark);
  color: #adb5bd;
  padding: 4rem 0;
}
.footer-brand {
  font-weight: 700;
  font-size: 1.75rem;
  color: var(--text-light);
}
.footer-brand span {
  color: var(--primary-color);
}
.footer-heading {
  color: var(--text-light);
  font-weight: 600;
  margin-bottom: 1rem;
}
.footer .list-unstyled li {
  margin-bottom: 0.5rem;
}

/* --- Responsive Adjustments --- */
@media (max-width: 991.98px) {
  .hero-title {
    font-size: 2.5rem;
  }
  .custom-navbar {
    background-color: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(10px);
  }
  .navbar-brand, .nav-link {
    color: var(--text-dark) !important;
  }
  .navbar-brand span {
    color: var(--primary-color) !important;
  }
  .btn-outline {
    border-color: var(--primary-color);
    color: var(--primary-color);
  }
}
</style>
