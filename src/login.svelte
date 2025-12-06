<script>
  import { onMount, createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  // حقول النموذج
  let name = '';
  let password = '';
  let remember = false;
  let showPassword = false;

  // حالة الواجهة
  let loading = false;
  let formError = '';
  let passwordError = '';

  onMount(() => {
    const rememberedEmail = localStorage.getItem('rememberedEmail');
    if (rememberedEmail) {
      remember = true;
    }
  });

  function validate() {
    formError = '';
    passwordError = '';

    if (!password || password.length < 6) {
      passwordError = 'Password must be at least 6 characters.';
    }

    return !passwordError;
  }

  async function onSubmit(e) {
    e.preventDefault();
    if (!validate()) return;

    loading = true;
    formError = '';

    try {
  
      dispatch('login', { name, password });
      loading = false;
    } catch (err) {
      formError = err?.message || 'An unexpected error occurred.';
      loading = false;
    }
  }
</script>

<main class="page" dir="ltr">
  <form class="login-card" on:submit|preventDefault={onSubmit} novalidate>
    <h1 class="title">Log In</h1>

    {#if formError}
      <div class="alert" role="alert">{formError}</div>
    {/if}

    <!-- حقل الاسم (Username) -->
    <div class="field">
      <label for="name">Username</label>
      <input
        id="name"
        name="name"
        type="text"
        placeholder="your name"
        bind:value={name}
        class="name-input"
        required
      />
    </div>

    <!-- حقل الباسورد فقط -->
    <div class="field">
      <label for="password">Password</label>
      <div class="password-row">
        <input
          id="password"
          name="password"
          type={showPassword ? 'text' : 'password'}
          placeholder="••••••••"
          bind:value={password}
          aria-invalid={passwordError ? 'true' : 'false'}
          aria-describedby={passwordError ? 'password-error' : undefined}
          required
        />
        <button
          type="button"
          class="ghost-btn"
          aria-label={showPassword ? 'Hide password' : 'Show password'}
          on:click={() => (showPassword = !showPassword)}
        >
          {showPassword ? 'Hide' : 'Show'}
        </button>
      </div>
      {#if passwordError}<p id="password-error" class="error">{passwordError}</p>{/if}
    </div>
<br><br>

    <button class="primary-btn" type="submit" disabled={loading}>
      {#if loading}
        <span class="spinner" aria-hidden="true"></span>
        Logging in…
      {:else}
        Log In
      {/if}
    </button>
  </form>
</main>

<style>
  /* خلفية رمادي داكن (gray medium) */
  :global(html, body) {
    height: 100%;

    font-family: "Caveat", "Patrick Hand", system-ui, -apple-system, "Segoe UI", Roboto, Arial, sans-serif;
  }
  :global(body) {
    margin: 0;
   font-family:
    "Bradley Hand",        /* متوفر على بعض أنظمة macOS/iOS */
    "Segoe Print",         /* Windows */
    "Lucida Handwriting",  /* Windows قديم */
    "Apple Chancery",      /* macOS */
    "Comic Sans MS",       /* Windows/macOS */
    cursive;
    background: #1f1f1f; /* رمادي داكن متوسط */
    color: #f0f0f0;
    letter-spacing: 0.2px;
  }

  .page {
    min-height: 100dvh;
    display: grid;
    place-items: center;
    padding: 16px;
  }

  /* البطاقة بإطار أبيض متوهّج */
  .login-card {
      left: 5%;
    top: 10%;
    width: 60%;
    height: auto;    /* دعي المحتوى يحدد الارتفاع */
    min-height: 60vh;

    background: #262626; /* أغمق قليلاً من الخلفية */
    border: 1px solid rgba(255, 255, 255, 0.85);
    border-radius: 16px;
    padding: 24px;
    box-shadow:
      0 0 0 1px rgba(255, 255, 255, 0.55) inset, /* خط داخلي خفيف */
      0 0 10px rgba(255, 255, 255, 0.35),       /* توهج قريب */
      0 0 24px rgba(255, 255, 255, 0.22);       /* توهج أبعد */
    backdrop-filter: blur(2px);
  }
  /* أحياناً ما نحتاج البطاقة بدون توهج إضافي */
  .glow-none {
    box-shadow:
      0 0 0 1px rgba(255, 255, 255, 0.55) inset,
      0 0 10px rgba(255, 255, 255, 0.28);
  }

  .title {
    margin: 0 0 16px;
    font-size: 1.5rem;
    font-weight: 700;
    letter-spacing: 0.2px;
    color: #ffffff;
  }

  .alert {
    background: #3a1020;
    border: 1px solid #d04b7d;
    color: #ffd9e5;
    padding: 10px 12px;
    border-radius: 10px;
    font-size: 0.95rem;
    margin-bottom: 12px;
  }

  .field {
    margin-bottom: 14px;
  }

  label {
    display: block;
    margin-bottom: 6px;
    font-size: 0.95rem;
    color: #f5f5f5;
  }

  input[type='password'] {
    width: 100%;
    padding: 12px 12px;
    border-radius: 10px;
    border: 1px solid rgba(255, 255, 255, 0.35);
    background: #1b1b1b;
    color: #f3f3f3;
    outline: none;
    transition: border-color 0.15s ease, box-shadow 0.15s ease;
  }

  /* حقل الاسم يكون قصير */
  .name-input {
    display: inline-block;
    width: auto;
    max-width: 200px;
    padding: 8px 10px;
    border-radius: 10px;
    border: 1px solid rgba(255, 255, 255, 0.35);
    background: #1b1b1b;
    color: #f3f3f3;
    outline: none;
    transition: border-color 0.15s ease, box-shadow 0.15s ease;
  }

  .name-input::placeholder {
    color: #888888;
  }

  .name-input:focus {
    border-color: #ffffff;
    box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.18);
  }

  .password-row {
    display: flex;
    gap: 8px;
    align-items: center;
  }
  .password-row input {
    flex: 1;
  }

  .ghost-btn {
    border: 1px solid rgba(255, 255, 255, 0.35);
    background: transparent;
    color: #fafafa;
    border-radius: 10px;
    padding: 10px 12px;
    cursor: pointer;
    white-space: nowrap;
    transition: background 0.15s ease, border-color 0.15s ease;
  }
  .ghost-btn:hover {
    background: rgba(255, 255, 255, 0.06);
    border-color: rgba(255, 255, 255, 0.55);
  }

  .row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin: 6px 0 16px;
    gap: 8px;
    display: inline-flex;
  }

  .link {
    color: #ffffff;
    opacity: 0.9;
    font-size: 0.95rem;
    text-decoration: none;
  }
  .link:hover {
    opacity: 1;
    text-decoration: underline;
  }

  .primary-btn {
    width: 100%;
    padding: 12px 12px;
    border: 0;
    border-radius: 12px;
    background: linear-gradient(90deg, #ffffff, #e6e6e6);
    color: #1a1a1a;
    font-weight: 700;
    cursor: pointer;
    display: inline-flex;
    gap: 8px;
    align-items: center;
    justify-content: center;
    transition: transform 0.05s ease, box-shadow 0.15s ease;
    box-shadow:
      0 1px 0 rgba(255, 255, 255, 0.8) inset,
      0 6px 18px rgba(255, 255, 255, 0.2);
  }
  .primary-btn:hover {
    box-shadow:
      0 1px 0 rgba(255, 255, 255, 0.9) inset,
      0 8px 22px rgba(255, 255, 255, 0.28);
  }
  .primary-btn:active {
    transform: translateY(1px);
  }
  .primary-btn[disabled] {
    opacity: 0.8;
    cursor: not-allowed;
  }

  .spinner {
    width: 16px;
    height: 16px;
    border: 2px solid rgba(0, 0, 0, 0.2);
    border-top-color: #1a1a1a;
    border-radius: 50%;
    display: inline-block;
    animation: spin 0.8s linear infinite;
  }
  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }

  .error {
    color: #ffb3bf;
    font-size: 0.88rem;
    margin-top: 6px;
  }

  .note {
    margin-top: 10px;
    font-size: 0.9rem;
    color: #e0e0e0;
    text-align: center;
    opacity: 0.9;
  }

  .welcome {
    text-align: center;
    color: #fff;
  }

  .secondary {
    margin-top: 14px;
    border: 1px solid rgba(255, 255, 255, 0.55);
    background: transparent;
    color: #ffffff;
    border-radius: 10px;
    padding: 10px 12px;
    cursor: pointer;
    transition: background 0.15s ease, border-color 0.15s ease;
  }
  .secondary:hover {
    background: rgba(255, 255, 255, 0.06);
    border-color: rgba(255, 255, 255, 0.8);
  }
</style>