<script>
  import Dashboard from "./dashboard.svelte";
  import Uploaded from "./uploaded.svelte";
  import Admin from "./admin.svelte";

  let activePage = "uploaded"; 
  let isAdmin = false;

  // استدعِ هذه الدالة بعد ما المستخدم يدخل البيانات
  function login(username, password) {
    if (username === "maryam" && password === "11") {
      isAdmin = true;
    } else {
      isAdmin = false;
    }
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

  function logout(e) {
    e.preventDefault();
    isAdmin = false;
    activePage = "uploaded";
  }
</script>

<main>
  <div>
    <nav id="nav-container" class="navbar fixed top-0 left-0 w-full z-50  backdrop-blur-md text-white px-10 py-4 flex justify-between items-center">
      <div class="text-2xl font-bold ">Tokyo</div>

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
              admine
            </a>
          </li>
        {/if}

        <li>
          <a href="#logout" class="hover:text-gray-300" on:click={logout}>
            logout
          </a>
        </li>
      </ul>
    </nav>

    {#if activePage === "dashboard"}
      <Dashboard/>
    {:else if activePage === "uploaded"}
      <Uploaded/>
    {:else if activePage === "admin" && isAdmin}
      <Admin/>
    {/if}
  </div>
</main>

<style>
  /* ...existing code... */
</style>
