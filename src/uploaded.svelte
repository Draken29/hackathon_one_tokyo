<script>
  let selectedFile = null;
  let previewUrl = '';
  let storedFiles = [];
  const STORAGE_KEY = 'uploaded_images';
  const STORAGE_DATE_KEY = 'uploaded_images_date';
  const DAILY_LIMIT = 5;
  let errorMessage = '';

  let uploading = false;
  let apiError = '';
  let textFromApi = '';

  const today = new Date().toISOString().slice(0, 10); // YYYY-MM-DD

  // رفع الصور من localStorage مع مراعاة تاريخ اليوم
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

  // رفع الصورة للـ backend
  async function uploadToApi(file) {
    apiError = '';
    uploading = true;
    textFromApi = '';

    try {
      const token = localStorage.getItem('authToken');
      if (!token) {
        uploading = false;
        apiError = 'You must be logged in before uploading images.';
        return null;
      }

      // تأكد أن الملف موجود فعلاً قبل الإرسال
      if (!file) {
        uploading = false;
        apiError = 'No file selected.';
        return null;
      }

      const formData = new FormData();
      // احرص أن يكون نفس الاسم الذي تستخدمه في Postman (في الغالب "file")
      formData.append('file', file);

      const res = await fetch(
        'https://uri-pharmacies-acc-hero.trycloudflare.com/api/images/upload/',
        {
          method: 'POST',
          headers: {
            Authorization: `Bearer ${token}`
            // لا تضف Content-Type، المتصفح يحدد multipart تلقائياً
          },
          body: formData
        }
      );

      const resClone = res.clone();

      let data = null;
      try {
        data = await res.json();
      } catch (_) {
        // إذا الرد ليس JSON (مثلاً HTML traceback في 500) نتعامل معه كنص أدناه
      }

      console.log('=== upload status ===', res.status);
      console.log('=== upload raw response (JSON) ===', data);

      if (!res.ok) {
        // اقرأ النص الخام مرة واحدة فقط لعرض رسالة 500/400 الحقيقية من Django
        let textBody = '';
        try {
          textBody = await resClone.text();
          console.log('=== 5xx/4xx raw body ===', textBody);
        } catch (_) {
          // ignore
        }

        if (res.status >= 500) {
          apiError =
            (data && (data.message || data.detail || data.error)) ||
            textBody ||
            'Internal server error (500). Check backend logs for details.';
          uploading = false;
          return null;
        }

        if (res.status === 400) {
          // أمثلة: {"detail":"No file"} أو أخطاء سيريالايزر
          if (data?.detail) {
            apiError = String(data.detail);
          } else if (data?.file) {
            apiError = Array.isArray(data.file)
              ? data.file.join(' ')
              : String(data.file);
          } else if (data?.image) {
            apiError = Array.isArray(data.image)
              ? data.image.join(' ')
              : String(data.image);
          } else if (data?.non_field_errors) {
            apiError = Array.isArray(data.non_field_errors)
              ? data.non_field_errors.join(' ')
              : String(data.non_field_errors);
          } else if (textBody) {
            apiError = textBody;
          } else {
            apiError =
              data?.message ||
              data?.detail ||
              JSON.stringify(data);
          }
          uploading = false;
          return null;
        }

        if (res.status === 401 || res.status === 403) {
          apiError = 'You are not authorized. Please login first.';
        } else {
          apiError =
            (data && (data.message || data.detail || data.error)) ||
            textBody ||
            'Failed to upload image to server.';
        }

        uploading = false;
        return null;
      }

      if (!data || typeof data !== 'object') {
        apiError = 'Unexpected response from server.';
        uploading = false;
        return null;
      }

      const imageUrl =
        typeof data.image_url === 'string'
          ? data.image_url
          : null;

      // حاول استخراج نص قابل للعرض من ollama_response أو additional_meta
      let extractedText = '';
      if (typeof data.ollama_response === 'string') {
        extractedText = data.ollama_response;
      } else if (
        data.ollama_response &&
        typeof data.ollama_response === 'object'
      ) {
        if (typeof data.ollama_response.text === 'string') {
          extractedText = data.ollama_response.text;
        } else if (typeof data.ollama_response.content === 'string') {
          extractedText = data.ollama_response.content;
        } else if (
          Array.isArray(data.ollama_response.choices) &&
          data.ollama_response.choices[0] &&
          data.ollama_response.choices[0].message &&
          data.ollama_response.choices[0].message.content
        ) {
          extractedText = String(
            data.ollama_response.choices[0].message.content
          );
        } else {
          extractedText = JSON.stringify(data.ollama_response);
        }
      } else if (typeof data.additional_meta === 'string') {
        extractedText = data.additional_meta;
      } else if (
        data.additional_meta &&
        typeof data.additional_meta === 'object'
      ) {
        if (typeof data.additional_meta.text === 'string') {
          extractedText = data.additional_meta.text;
        } else {
          extractedText = JSON.stringify(data.additional_meta);
        }
      }

      if (!imageUrl) {
        console.warn('No image_url returned, using placeholder preview.');
      }

      uploading = false;
      return { imageUrl, extractedText, data };
    } catch (err) {
      console.error('upload error', err);
      apiError = (err && err.message) || 'Network error during upload.';
      uploading = false;
      return null;
    }
  }

  async function handleFileChange(event) {
    errorMessage = '';
    apiError = '';
    textFromApi = '';

    if (storedFiles.length >= DAILY_LIMIT) {
      errorMessage = 'You have reached your limit for today (5 images). Try again tomorrow 🌙';
      event.target.value = '';
      return;
    }

    const file = event.target.files && event.target.files[0];
    if (!file) {
      apiError = 'No file selected.';
      return;
    }

    selectedFile = file;

    const result = await uploadToApi(file);
    if (!result) {
      event.target.value = '';
      return;
    }

    const { imageUrl, extractedText, data } = result;

    const fallbackUrl = data?.id
      ? `https://uri-pharmacies-acc-hero.trycloudflare.com/api/images/${data.id}/file/`
      : '';

    previewUrl = imageUrl || fallbackUrl;
    textFromApi = extractedText || '';

    const newFiles = [
      ...storedFiles,
      {
        name: data?.filename || file.name,
        size: data?.size_bytes ?? file.size,
        type: data?.content_type || file.type,
        url: previewUrl,
        date: today,
        id: data?.id,
        ocr_status: data?.ocr_status
      }
    ];

    storedFiles = newFiles;
    saveToLocalStorage(newFiles);
    event.target.value = '';
  }
</script>

<main id="main-container" class="w-screen h-screen">
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

      {#if apiError}
        <p class="mt-2 text-red-400 text-sm">{apiError}</p>
      {/if}

      {#if uploading}
        <p class="mt-2 text-blue-300 text-sm">Uploading image...</p>
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

  <div
    id="upload-container"
    class="text-2xl rounded-2xl relative top-[-35%] left-[70%] w-[25%] h-[60%]"
  >
    <p class="text-white text-3xl font-bold mb-4 animate-pulse text-center relative top-[5%]">
      the text is
    </p>
    {#if textFromApi}
      <p class="text-white text-lg px-4 mt-4 break-words">
        {textFromApi}
      </p>
    {:else}
      <p class="text-white/40 text-sm px-4 mt-4 text-center">
        Upload an image to see the extracted text here.
      </p>
    {/if}
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
      "Bradley Hand",
      "Segoe Print",
      "Lucida Handwriting",
      "Apple Chancery",
      "Comic Sans MS",
      cursive;
  }

  #main-container {
    background-color: #1f1f1f;
  }

  #nav-container {
    box-shadow:
      0 0 0 1px rgba(255, 255, 255, 0.55) inset,
      0 0 10px rgba(255, 255, 255, 0.28);
    font-family:
      "Bradley Hand",
      "Segoe Print",
      "Lucida Handwriting",
      "Apple Chancery",
      "Comic Sans MS",
      cursive;
    background: #262626;
  }
</style>