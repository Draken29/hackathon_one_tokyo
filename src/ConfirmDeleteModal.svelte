<script>
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();

  function handleConfirm() {
    dispatch('confirm');
  }

  function handleCancel() {
    dispatch('cancel');
  }
</script>

<div class="modal-backdrop" on:click={handleCancel}>
  <div class="modal-content" on:click|stopPropagation>
    <h2>Confirm Deletion</h2>
    <p>Are you sure you want to delete this user? This action cannot be undone.</p>
    
    <div class="form-actions">
      <button type="button" class="cancel-btn" on:click={handleCancel}>Cancel</button>
      <button 
        type="button" 
        class="action-btn delete" 
        style="margin-left: 10px;" 
        on:click={handleConfirm}>
        Confirm Delete
      </button>
    </div>
  </div>
</div>

<style>
  /* ملاحظة: استخدم التنسيقات السابقة التي زودتك بها للمودال */
  .modal-backdrop {
    position: fixed; top: 0; left: 0; width: 100%; height: 100%; 
    background-color: rgba(0, 0, 0, 0.8); display: flex; 
    justify-content: center; align-items: center; z-index: 2000;
  }

  .modal-content {
    background-color: #262626;
    padding: 30px;
    border-radius: 10px;
    width: 90%;
    max-width: 400px;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.8);
    color: #f0f0f0;
    text-align: center;
  }

  .modal-content h2 { color: #ff5252; margin-bottom: 15px; }
  .modal-content p { margin-bottom: 25px; color: #aaaaaa; }

  .form-actions { display: flex; justify-content: center; margin-top: 15px; }
  .cancel-btn { background-color: #555; color: white; padding: 10px 15px; border-radius: 6px; border: none; cursor: pointer; }
  .action-btn.delete { background-color: #dc3545; color: white; padding: 10px 15px; border-radius: 6px; border: none; cursor: pointer; }
</style>