<script>
  import { createEventDispatcher } from 'svelte';
  export let users;
  
  // تهيئة أداة الإرسال
  const dispatch = createEventDispatcher();

  // 🔴 دالة إرسال حدث الحذف 🔴
  function deleteUser(id) {
    // 🛑 يجب إرسال الحدث deleteUser مع الـ ID 🛑
    dispatch('deleteUser', id); 
  }
  
  // ... (بقية الدوال: viewDetails, openModal) ...
</script>

<div class="table-container">
  <table>
    <tbody>
      {#each users as user (user.id)}
        <tr>
          <td>
            <button class="action-btn view" on:click={() => dispatch('viewDetails', user)}>Details</button> 
            <button class="action-btn edit" on:click={() => dispatch('openModal', user)}>Edit</button>
            <button class="action-btn delete" on:click={() => deleteUser(user.id)}>Delete</button> 
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>