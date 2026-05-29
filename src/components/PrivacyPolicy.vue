<template>
  <div class="page" @mousemove="trackMouse">

    <!-- Accepted screen -->
    <Transition name="fade">
      <div v-if="accepted" class="accepted">
        <div class="accepted-icon">🎉</div>
        <h1>You're all set!</h1>
        <p>Thanks for accepting our Privacy Policy.</p>
      </div>
    </Transition>

    <!-- Modal -->
    <Transition name="slide">
      <div v-if="!accepted" class="modal">

        <div class="modal-header">
          <span class="lock-icon">🔒</span>
          <div>
            <h1>Privacy Policy</h1>
            <p class="subtitle">Last updated: May 2026</p>
          </div>
        </div>

        <div class="modal-body">
          <section>
            <h2>1. Data We Collect</h2>
            <p>We collect information you provide directly to us, such as when you create an account,
            make a purchase, or contact us for support. This includes your name, email address,
            payment information, and any other details you choose to share.</p>
          </section>

          <section>
            <h2>2. How We Use Your Data</h2>
            <p>Your data is used solely to provide and improve our services. We do not sell,
            trade, or otherwise transfer your personal information to outside parties without
            your consent, except to trusted third parties who assist us in operating our website.</p>
          </section>

          <section>
            <h2>3. Cookies</h2>
            <p>We use cookies to enhance your experience. You may choose to disable cookies
            through your browser settings, though this may affect some functionality of our
            services. Essential cookies are required for the site to function properly.</p>
          </section>

          <section>
            <h2>4. Data Retention</h2>
            <p>We retain your personal information for as long as your account is active or as
            needed to provide you services. You may request deletion of your data at any time
            by contacting our support team at privacy@example.com.</p>
          </section>

          <section>
            <h2>5. Your Rights</h2>
            <p>You have the right to access, correct, or delete your personal data. You also
            have the right to object to processing, request restriction, and data portability.
            To exercise these rights, please contact us directly.</p>
          </section>
        </div>

        <div class="modal-footer">
          <button class="btn-accept" @click="accept">
            <span>✓</span> Accept & Continue
          </button>

          <!-- Refuse button: fixed once mounted so it can flee anywhere on screen -->
          <button
            ref="refuseBtn"
            class="btn-refuse"
            :class="{ 'is-fixed': isFixed }"
            :style="refuseStyle"
            @click="handleRefuse"
          >
            <span>✕</span> Refuse
          </button>
        </div>

      </div>
    </Transition>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from 'vue'

const mouse       = ref({ x: 0, y: 0 })
const refusePos   = ref({ x: 0, y: 0 })
const isFixed     = ref(false)
const accepted    = ref(false)
const refuseBtn   = ref(null)

const BTN_W = 120
const BTN_H = 46

const refuseStyle = computed(() => {
  if (!isFixed.value) return {}
  return {
    position: 'fixed',
    left: refusePos.value.x + 'px',
    top:  refusePos.value.y + 'px',
    transition: 'left 0.1s ease-out, top 0.1s ease-out',
    zIndex: 9999,
    margin: 0,
  }
})

function trackMouse(e) {
  mouse.value = { x: e.clientX, y: e.clientY }
  if (!isFixed.value || accepted.value) return

  const cx = refusePos.value.x + BTN_W / 2
  const cy = refusePos.value.y + BTN_H / 2
  const dx = e.clientX - cx
  const dy = e.clientY - cy
  const dist = Math.hypot(dx, dy)

  if (dist < 130) {
    const angle = Math.atan2(dy, dx)
    const flee  = 180

    let nx = refusePos.value.x - Math.cos(angle) * flee
    let ny = refusePos.value.y - Math.sin(angle) * flee

    const pad = 12
    nx = Math.max(pad, Math.min(window.innerWidth  - BTN_W - pad, nx))
    ny = Math.max(pad, Math.min(window.innerHeight - BTN_H - pad, ny))

    refusePos.value = { x: nx, y: ny }
  }
}

function accept() {
  accepted.value = true
}

function handleRefuse() {
  // If somehow clicked, just flee again — should be nearly impossible
}

onMounted(() => {
  nextTick(() => {
    if (refuseBtn.value) {
      const rect = refuseBtn.value.getBoundingClientRect()
      refusePos.value = { x: rect.left, y: rect.top }
      isFixed.value = true
    }
  })
})
</script>

<style scoped>
/* ── Page ── */
.page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
  font-family: 'Segoe UI', system-ui, sans-serif;
  padding: 24px;
  user-select: none;
}

/* ── Modal ── */
.modal {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 20px;
  width: 100%;
  max-width: 600px;
  box-shadow: 0 32px 80px rgba(0, 0, 0, 0.6);
  overflow: hidden;
}

.modal-header {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 28px 32px 20px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}

.lock-icon {
  font-size: 2.4rem;
  filter: drop-shadow(0 0 12px #7c6af4);
}

.modal-header h1 {
  margin: 0;
  font-size: 1.6rem;
  font-weight: 700;
  color: #fff;
  letter-spacing: -0.3px;
}

.subtitle {
  margin: 4px 0 0;
  font-size: 0.8rem;
  color: rgba(255,255,255,0.4);
}

/* ── Body ── */
.modal-body {
  padding: 24px 32px;
  max-height: 340px;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: rgba(255,255,255,0.2) transparent;
}

.modal-body section {
  margin-bottom: 20px;
}

.modal-body h2 {
  font-size: 0.9rem;
  font-weight: 600;
  color: #a78bfa;
  margin: 0 0 6px;
  text-transform: uppercase;
  letter-spacing: 0.6px;
}

.modal-body p {
  font-size: 0.9rem;
  line-height: 1.7;
  color: rgba(255,255,255,0.7);
  margin: 0;
}

/* ── Footer ── */
.modal-footer {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 12px;
  padding: 20px 32px 28px;
  border-top: 1px solid rgba(255,255,255,0.08);
  position: relative;
  min-height: 80px;
}

/* ── Buttons ── */
button {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 28px;
  border-radius: 12px;
  font-size: 0.92rem;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: transform 0.15s, box-shadow 0.15s, background 0.15s;
  white-space: nowrap;
}

.btn-accept {
  background: linear-gradient(135deg, #6d28d9, #7c3aed);
  color: #fff;
  box-shadow: 0 4px 20px rgba(109, 40, 217, 0.5);
}

.btn-accept:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 28px rgba(109, 40, 217, 0.7);
}

.btn-accept:active {
  transform: translateY(0);
}

.btn-refuse {
  background: transparent;
  color: rgba(255,255,255,0.55);
  border: 1px solid rgba(255,255,255,0.2);
}

.btn-refuse:hover {
  background: rgba(255, 80, 80, 0.1);
  border-color: rgba(255,80,80,0.4);
  color: #ff6b6b;
}

/* When the button is fixed-positioned, remove margin so layout doesn't shift */
.btn-refuse.is-fixed {
  cursor: not-allowed;
}

/* ── Accepted screen ── */
.accepted {
  text-align: center;
  color: #fff;
}

.accepted-icon {
  font-size: 5rem;
  margin-bottom: 20px;
  animation: pop 0.4s ease;
}

.accepted h1 {
  font-size: 2.2rem;
  margin: 0 0 12px;
  background: linear-gradient(90deg, #a78bfa, #60a5fa);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.accepted p {
  color: rgba(255,255,255,0.6);
  font-size: 1.05rem;
}

/* ── Transitions ── */
.fade-enter-active, .fade-leave-active { transition: opacity 0.4s; }
.fade-enter-from, .fade-leave-to       { opacity: 0; }

.slide-enter-active { transition: all 0.4s cubic-bezier(0.34, 1.56, 0.64, 1); }
.slide-leave-active { transition: all 0.3s ease; }
.slide-enter-from   { opacity: 0; transform: translateY(30px) scale(0.96); }
.slide-leave-to     { opacity: 0; transform: translateY(-10px) scale(0.98); }

@keyframes pop {
  0%   { transform: scale(0); }
  70%  { transform: scale(1.2); }
  100% { transform: scale(1); }
}
</style>
