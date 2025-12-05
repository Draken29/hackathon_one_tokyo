<script>
    let selectedFile = null;
    let previewUrl = '';
    let storedFiles = [];
    const STORAGE_KEY = 'uploaded_images';
    const STORAGE_DATE_KEY = 'uploaded_images_date';
    const DAILY_LIMIT = 5;
    let errorMessage = '';

    const today = new Date().toISOString().slice(0, 10); // YYYY-MM-DD

    // تحميل الصور من localStorage مع مراعاة تاريخ اليوم
    if (typeof window !== 'undefined') {
      const savedDate = localStorage.getItem(STORAGE_DATE_KEY);
      const saved = localStorage.getItem(STORAGE_KEY);

      if (saved && savedDate === today) {
        storedFiles = JSON.parse(saved);
        if (storedFiles.length) {
          previewUrl = storedFiles[storedFiles.length - 1].url;
        }
      } else {
        // يوم جديد: نفرغ الصور ونحدّث التاريخ
        storedFiles = [];
        localStorage.setItem(STORAGE_KEY, JSON.stringify([]));
        localStorage.setItem(STORAGE_DATE_KEY, today);
      }
    }

    function saveToLocalStorage(files) {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(files));
      localStorage.setItem(STORAGE_DATE_KEY, today);
    }

    function handleFileChange(event) {
      errorMessage = '';


      if (storedFiles.length >= DAILY_LIMIT) {
        errorMessage = 'You have reached your limit for today (5 images). Try again tomorrow 🌙';
        
        event.target.value = '';
        
        return;
      }

      const file = event.target.files?.[0];
      if (!file) return;

      selectedFile = file;

      const reader = new FileReader();
      reader.onload = () => {
        const dataUrl = reader.result;
        previewUrl = dataUrl;

        const newFiles = [
          ...storedFiles,
          {
            name: file.name,
            size: file.size,
            type: file.type,
            url: dataUrl,
            date: today
          }
        ];

        storedFiles = newFiles;
        saveToLocalStorage(newFiles);
      };
      reader.readAsDataURL(file);
    }
</script>

<main id="main-container" class="w-screen h-screen" >

     <nav id="nav-container" class="navbar fixed top-0 left-0 w-full z-50  backdrop-blur-md text-white px-10 py-4 flex justify-between items-center">
    <div class="text-2xl font-bold ">Tokyo</div>

    <ul class="flex gap-8 text-lg">
      <li><a href="#home" class="hover:text-gray-300">Upload Image</a></li>
      <li><a href="#home" class="hover:text-gray-300">Dashboard</a></li>
      <li><a href="#home" class="hover:text-gray-300">logout</a></li>
  </ul>
  </nav>
    

    <div
      id="upload-container"
      class="text-2xl rounded-2xl relative  top-[25%] left-[15%] transition-all duration-300 w-[50%] h-[60%]"
    >
      <div class="flex flex-col items-center justify-center h-full transition-transform duration-300 hover:scale-105">
        <h1 class="text-white text-3xl font-bold mb-4 animate-pulse">Uploadede your photo</h1>
        <input
          type="file"
          id="fileInput"
          class="hidden"
          accept=".png,.jpg,.jpeg,.gif,.webp"
          on:change={handleFileChange}
        />
        <label
          for="fileInput"
          class="cursor-pointer p-3 bg-gray-200 rounded-xl hover:bg-gray-300 transition flex items-center justify-center"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="lucide lucide-image-up-icon lucide-image-up w-[55px] h-[55px] flex-shrink-0"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path d="M10.3 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2v10l-3.1-3.1a2 2 0 0 0-2.814.014L6 21"/>
            <path d="m14 19.5 3-3 3 3"/>
            <path d="M17 22v-5.5"/>
            <circle cx="9" cy="9" r="2"/>
          </svg>
        </label>       

        {#if errorMessage}
          <p class="mt-2 text-red-400 text-sm">{errorMessage}</p>
        {/if}

        {#if previewUrl}
        <!-- معاينة آخر صورة مرفوعة -->
        <div class="mt-4 flex flex-col items-center gap-2">
          <span class="text-white text-md">Preview:</span>
          <h5 class="text-white/50 text-sm">
              your limit is only 5 photos per day
              ({storedFiles.length}/{DAILY_LIMIT})
            </h5>
          <img src={previewUrl} alt="Uploaded preview" class="max-h-32 rounded-lg" />
        </div>
      {/if}
      </div>
    </div>
     <div id="upload-container" class="  text-2xl rounded-2xl relative top-[-35%]  left-[70%]  w-[25%] h-[60%]">
           <p class="text-white text-3xl font-bold mb-4 animate-pulse text-center relative top-[5%]">the text is</p>
          </div>
    
</main>

<style>
    #upload-container {
        background: #1f1f1f;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
          box-shadow:
      0 0 0 1px rgba(255, 255, 255, 0.55) inset,
      0 0 10px rgba(255, 255, 255, 0.28);

      font-family:
    "Bradley Hand",        /* متوفر على بعض أنظمة macOS/iOS */
    "Segoe Print",         /* Windows */
    "Lucida Handwriting",  /* Windows قديم */
    "Apple Chancery",      /* macOS */
    "Comic Sans MS",       /* Windows/macOS */
    cursive;
    }

    #main-container{
        background-color: #1f1f1f;
    }

    #nav-container{
        box-shadow:
      0 0 0 1px rgba(255, 255, 255, 0.55) inset,
      0 0 10px rgba(255, 255, 255, 0.28);
      font-family:
    "Bradley Hand",        /* متوفر على بعض أنظمة macOS/iOS */
    "Segoe Print",         /* Windows */
    "Lucida Handwriting",  /* Windows قديم */
    "Apple Chancery",      /* macOS */
    "Comic Sans MS",       /* Windows/macOS */
    cursive;
      background: #262626
    }
</style>