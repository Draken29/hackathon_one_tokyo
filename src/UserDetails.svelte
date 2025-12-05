<script>
  import { createEventDispatcher } from 'svelte';

  // 1. استقبال كائن المستخدم المحدد كـ Prop
  export let user; 

  const dispatch = createEventDispatcher();

  function goBack() {
    // إرسال حدث 'goBack' لطلب العودة إلى الجدول في App.svelte
    dispatch('goBack');
  }
</script>

<div class="details-view"> 
    <header class="header">
        <button class="back-button" on:click={goBack}>
            ← Back to User List
        </button>
        <h1>Details for {user.name}</h1>
    </header>

    <div class="summary-card">
        <h3>User Summary</h3>
        <p><strong>Registration Date on Website:</strong> {user.registrationDate || 'N/A'}</p> 
        <p><strong>Email:</strong> {user.email}</p>
        <p><strong>Role:</strong> {user.role}</p>
    </div>

    <h2>Image Upload History</h2>
    
    {#if user.activity && user.activity.length > 0}
        <div class="activity-log">
            {#each user.activity as activityItem, index}
                <div class="activity-item">
                    <h4>Upload {index + 1}</h4>
                    <p><strong>Date of Upload:</strong> {activityItem.date}</p>
                    <p><strong>Uploaded Image Name:</strong> {activityItem.fileName}</p>
                    <p><strong>Text Result/Description:</strong> {activityItem.textResult || 'N/A'}</p>
                </div>
            {/each}
        </div>
    {:else}
        <p class="no-activity">No image uploads recorded for this user.</p>
    {/if}
</div>

<style>
  /* ... (تنسيقات CSS التي زودتك بها سابقاً) ... */
</style>