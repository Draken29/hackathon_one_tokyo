<script>
  import Dashboard from "./dashboard.svelte";
  import Uploaded from "./uploaded.svelte";
  import Admin from "./admin.svelte";
  import Login from "./login.svelte";

  let activePage = "login"; // نبدأ بصفحة تسجيل الدخول
  let isAdmin = false;
  let authToken = null; // التوكن المستخدم في الرفع

  // الدالة تستقبل detail كامل من الـ Login
  function login(detail) {
    const { name, token, isAdmin: isAdminFromApi, raw } = detail || {};

    // إذا الـ API يرجّع isAdmin / is_admin نستخدمه
    // وإذا ما يرجّع، نخلي baraa / baraa أدمن كحالة خاصة (fallback)
    if (isAdminFromApi === true) {
      isAdmin = true;
    } else if (name === "baraa") {
      // ملاحظة: هنا تعتمدين على الاسم فقط لأن الـ password ما عاد يُرسل من Login
      isAdmin = true;
    } else {
      isAdmin = false;
    }

    // خزّن التوكن في المتغير المحلي + localStorage إذا موجود
    authToken = token || null;
    if (token) {
      localStorage.setItem("authToken", token);
    }

    // بعد تسجيل الدخول نذهب لصفحة Uploaded
    activePage = "uploaded";
  }

  function openUpload(e) {
    e.preventDefault();
    activePage = "uploaded";
  }

  function openDashboard(e) {
    e.preventDefault();
    activePage = "dashboard";
  }

  function admine(e) {
    e.preventDefault();
    if (isAdmin) {
      activePage = "admin";
    }
  }

</script>

<main>
  <div>
    <nav id="nav-container" class="navbar fixed top-0 left-0 w-full z-50  backdrop-blur-md text-white px-10 py-4 flex justify-between items-center">
      <div class="text-2xl font-bold ">Tokyo</div>

      {#if activePage !== "login"}
        <ul class="flex gap-8 text-lg">
          <li>
            <a href="#home" class="hover:text-gray-300" on:click={openUpload}>
              Upload Image
            </a>
          </li>
          <li>
            <a href="#dashboard" class="hover:text-gray-300" on:click={openDashboard}>
              Dashboard
            </a>
          </li>

          {#if isAdmin}
            <li>
              <a href="#admin" class="hover:text-gray-300" on:click={admine}>
                admin
              </a>
            </li>
          {/if}
        </ul>
      {/if}
    </nav>

    {#if activePage === "login"}
      <Login
        on:login={event => login(event.detail)}
      />
    {:else if activePage === "dashboard"}
      <Dashboard/>
    {:else if activePage === "uploaded"}
      <!-- هنا نرسل التوكن إلى صفحة الرفع -->
      <Uploaded token={authToken} />
    {:else if activePage === "admin" && isAdmin}
      <Admin/>
    {/if}
  </div>
</main>

<style>
  /* ...existing code... */
</style>
