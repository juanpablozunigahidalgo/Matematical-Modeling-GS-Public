<script setup lang="ts">
defineOptions({ name: "InvestorFooter" });

const origin = "https://www.gettashower.com";
</script>

<template>
  <footer class="gs-footer">
    <div class="gs-footer-inner">
      <div class="gs-footer-grid">
        <div class="gs-footer-brand">
          <h3 class="gs-footer-logo">GettaShower</h3>
          <p class="gs-footer-tagline">
            Global shower network for urban cyclists, commuters, city dwellers, city hikers and tourists.
            Refresh yourself, save time, and live the city on your terms.
          </p>
          <div class="gs-footer-row">
            <svg class="gs-footer-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
              <path d="M12 21s7-4.5 7-11a7 7 0 10-14 0c0 6.5 7 11 7 11z" stroke-linecap="round" stroke-linejoin="round" />
              <circle cx="12" cy="10" r="2.5" />
            </svg>
            <span>Göteborg, Sweden</span>
          </div>
          <div class="gs-footer-social-block">
            <h5 class="gs-footer-social-title">Follow us in social media</h5>
            <div class="gs-footer-social">
              <a
                href="https://www.youtube.com/@gettashower"
                class="gs-footer-social-btn"
                aria-label="GettaShower on YouTube"
                target="_blank"
                rel="noopener noreferrer"
              >
                <svg class="gs-footer-social-svg" viewBox="0 0 24 24" fill="currentColor" aria-hidden="true">
                  <path
                    d="M23.5 6.2c-.3-1.1-1.2-1.9-2.3-2.2C19.6 3.5 12 3.5 12 3.5s-7.6 0-9.2.5c-1.1.3-2 1.1-2.3 2.2C0 7.9 0 12 0 12s0 4.1.5 5.8c.3 1.1 1.2 1.9 2.3 2.2 1.7.5 9.2.5 9.2.5s7.6 0 9.2-.5c1.1-.3 2-1.1 2.3-2.2.5-1.7.5-5.8.5-5.8s0-4.1-.5-5.8zM9.5 15.5v-7l6.5 3.5-6.5 3.5z"
                  />
                </svg>
              </a>
              <a
                href="https://www.tiktok.com/@gettashower"
                class="gs-footer-social-btn"
                aria-label="GettaShower on TikTok"
                target="_blank"
                rel="noopener noreferrer"
              >
                <svg class="gs-footer-social-svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
                  <path d="M9 18a5 5 0 005-5V7a7 7 0 007 7" stroke-linecap="round" />
                  <path d="M15 3v8.5A4.5 4.5 0 0110.5 16H10" stroke-linecap="round" />
                </svg>
              </a>
            </div>
          </div>
        </div>

        <div class="gs-footer-col">
          <h4 class="gs-footer-heading">Organization</h4>
          <nav class="gs-footer-links" aria-label="Organization links">
            <a :href="`${origin}/`" target="_blank" rel="noopener noreferrer">Home</a>
            <a :href="`${origin}/getapp`" target="_blank" rel="noopener noreferrer">Get the App</a>
            <a :href="`${origin}/host`" target="_blank" rel="noopener noreferrer">Become a Host</a>
          </nav>
        </div>

        <div id="contact" class="gs-footer-col gs-footer-contact">
          <h4 class="gs-footer-heading">Contact</h4>
          <a href="mailto:info@gettashower.com" class="gs-footer-contact-link">
            <svg class="gs-footer-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
              <path d="M4 6h16v12H4z" stroke-linejoin="round" />
              <path d="M4 7l8 6 8-6" stroke-linecap="round" stroke-linejoin="round" />
            </svg>
            <span>info@gettashower.com</span>
          </a>
          <a href="tel:+46729971641" class="gs-footer-contact-link">
            <svg class="gs-footer-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
              <path
                d="M6.6 4.6l-.4-.9a1 1 0 00-1.3-.5l-1.9.8a1 1 0 00-.6 1.2A16 16 0 0020.8 17l.8-1.9a1 1 0 00-.5-1.3l-.9-.4a1 1 0 00-1.1.3l-1.3 1.4a12 12 0 01-5.4-5.4l1.4-1.3a1 1 0 00.3-1.1z"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>
            <span>+46729971641</span>
          </a>
        </div>
      </div>

      <div class="gs-footer-bottom">
        <p class="gs-footer-copy">© 2026 GettaShower. All rights reserved.</p>
        <div class="gs-footer-legal">
          <a :href="`${origin}/privacy-policy`" target="_blank" rel="noopener noreferrer">Privacy Policy</a>
          <a :href="`${origin}/terms-of-service-eu`" target="_blank" rel="noopener noreferrer">Terms of Service</a>
        </div>
      </div>
    </div>
  </footer>
</template>

<style scoped>
.gs-footer {
  margin-top: auto;
  background: var(--gs-footer-bg, #0d1533);
  color: #ffffff;
  width: 100%;
}

.gs-footer-inner {
  max-width: 1280px;
  margin: 0 auto;
  padding: 3.5rem 1.5rem 2.5rem;
}

@media (max-width: 639px) {
  .gs-footer-inner {
    padding: 2.75rem 1rem 2rem;
  }
}

.gs-footer-grid {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr;
  gap: 2rem;
}

.gs-footer-logo {
  margin: 0 0 1rem;
  font-size: 1.5rem;
  font-weight: 800;
  letter-spacing: -0.02em;
  color: #ffffff;
}

.gs-footer-tagline {
  margin: 0;
  max-width: 28rem;
  line-height: 1.65;
  color: rgba(255, 255, 255, 0.92);
  font-size: 0.95rem;
}

.gs-footer-row {
  display: flex;
  align-items: center;
  gap: 0.45rem;
  margin-top: 1.35rem;
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.95);
}

.gs-footer-icon {
  width: 1rem;
  height: 1rem;
  flex-shrink: 0;
  opacity: 0.95;
}

.gs-footer-social-block {
  margin-top: 1.5rem;
}

.gs-footer-social-title {
  margin: 0 0 0.65rem;
  font-size: 0.875rem;
  font-weight: 600;
  color: #ffffff;
}

.gs-footer-social {
  display: flex;
  gap: 0.65rem;
}

.gs-footer-social-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 3rem;
  height: 3rem;
  border-radius: 0.75rem;
  border: 1px solid rgba(255, 255, 255, 0.3);
  background: rgba(255, 255, 255, 0.04);
  color: #ffffff;
  transition: background 0.2s ease, border-color 0.2s ease, transform 0.2s ease;
}

.gs-footer-social-btn:hover {
  background: rgba(255, 255, 255, 0.14);
  border-color: rgba(255, 255, 255, 0.45);
  transform: scale(1.03);
}

.gs-footer-social-svg {
  width: 1.35rem;
  height: 1.35rem;
}

.gs-footer-heading {
  margin: 0 0 1rem;
  font-size: 1rem;
  font-weight: 600;
  color: #ffffff;
}

.gs-footer-links {
  display: flex;
  flex-direction: column;
  gap: 0.45rem;
}

.gs-footer-links a {
  color: #ffffff;
  text-decoration: none;
  font-size: 0.95rem;
  transition: opacity 0.15s ease;
}

.gs-footer-links a:hover {
  opacity: 0.88;
}

.gs-footer-contact-link {
  display: flex;
  align-items: center;
  gap: 0.45rem;
  color: #ffffff;
  text-decoration: none;
  font-size: 0.875rem;
  margin-bottom: 0.65rem;
  transition: opacity 0.15s ease;
}

.gs-footer-contact-link:hover {
  opacity: 0.88;
}

.gs-footer-bottom {
  margin-top: 3rem;
  padding-top: 2rem;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

@media (min-width: 768px) {
  .gs-footer-bottom {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }
}

.gs-footer-copy {
  margin: 0;
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.95);
}

.gs-footer-legal {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem 1.5rem;
  justify-content: center;
}

@media (min-width: 768px) {
  .gs-footer-legal {
    justify-content: flex-end;
  }
}

.gs-footer-legal a {
  color: #ffffff;
  font-size: 0.875rem;
  text-decoration: none;
  transition: opacity 0.15s ease;
}

.gs-footer-legal a:hover {
  opacity: 0.88;
}

@media (max-width: 900px) {
  .gs-footer-grid {
    grid-template-columns: 1fr;
  }
}
</style>
