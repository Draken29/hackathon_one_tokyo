<script>
  import { onMount } from 'svelte';
  import ConfirmDeleteModal from './ConfirmDeleteModal.svelte';
  import UserTable from './UserTable.svelte';
  import UserDetails from './UserDetails.svelte';
  import './admine.css';
  let selectedUser = null;
  let isModalOpen = false;
let users = [
    // لا يتم حفظ كلمات المرور هنا كبيانات صريحة عادةً، لكن يتم تضمين حقل 'password' في النموذج فقط لأغراض العرض/الإدخال.
    { id: 1, name: 'Ahmed Ali', email: 'ahmad@example.com', password: 'password123' },
    { id: 2, name: 'Fatima Mohamed', email: 'fatima@example.com', password: 'password456' },
    { id: 3, name: 'Omar Hassan', email: 'omar@example.com', password: 'password789' },
    { id: 4, name: 'Sara Ibrahim', email: 'sara@example.com', password: 'password012' }
  ];
  

  function viewDetails(user) {
    selectedUser = user;
    alert(`Showing details for ${user.name}`);
  }
  function goBack() {
    selectedUser = null; 
  }

  let currentEditingUser = null; // null للإضافة، أو كائن المستخدم للتعديل

  // بيانات النموذج الجديدة (تم إضافة حقل password مؤقت)
  let newUser = { name: '', email: '', password: '' };

  // 🔴 1. دالة فتح النموذج (إضافة/تعديل)
  function openModal(user = null) {
    if (user) {
      currentEditingUser = user;
      // نسخ بيانات المستخدم للتعديل، وترك حقل كلمة المرور فارغاً للأمان
      newUser = { ...user, password: '' }; 
    } else {
      currentEditingUser = null;
      newUser = { name: '', email: '', password: '' };
    }
    isModalOpen = true;
  }

  // إغلاق النموذج
  function closeModal() {
    isModalOpen = false;
  }

  // 🔴 2. دالة حفظ المستخدم (إضافة أو تعديل)
  function saveUser() {
    if (newUser.password.length < 6 && !currentEditingUser) {
        alert("Password must be at least 6 characters long.");
        return;
    }
    
    // إنشاء كائن المستخدم للحفظ (بدون كلمة المرور إذا كانت فارغة في وضع التعديل)
    let userToSave = { 
        id: currentEditingUser ? currentEditingUser.id : (users.length > 0 ? Math.max(...users.map(u => u.id)) + 1 : 1),
        name: newUser.name,
        email: newUser.email,
        // ملاحظة: في بيئة حقيقية، يتم تشفير كلمة المرور هنا قبل الحفظ
    };

    if (currentEditingUser) {
      // التعديل: تحديث البيانات الحالية
      // @ts-ignore
      users = users.map(u => u.id === currentEditingUser.id ? userToSave : u);
    } else {
      // الإضافة
      // @ts-ignore
      users = [...users, userToSave];
    }
    closeModal();
  }

  // 🔴 3. دالة حذف المستخدم
  function deleteUser(id) {
    if (confirm('Are you sure you want to delete this user?')) {
      users = users.filter(user => user.id !== id);
    }
  }
let isConfirmModalOpen = false;
  let userToDeleteId = null;

  // ... (دوال viewDetails, goBack, openModal, closeModal, saveUser) ...

  // 🔴 1. دالة بدء عملية الحذف (تفتح النموذج بدلاً من alert) 🔴
  function startDelete(id) {
    userToDeleteId = id;
    isConfirmModalOpen = true; // فتح نموذج التأكيد
  }

  // 🔴 2. دالة تنفيذ الحذف الفعلي (تستدعى من النموذج) 🔴
  function confirmDelete() {
    if (userToDeleteId !== null) {
      users = users.filter(user => user.id !== userToDeleteId);
    }
    closeConfirmModal();
  }

  // دالة إغلاق نموذج التأكيد
  function closeConfirmModal() {
    isConfirmModalOpen = false;
    userToDeleteId = null; // إعادة تعيين المعرّف
  }
</script>
<main>
<div class="user-management-panel" dir="ltr">
  
  <div class="details-section">
  {#if selectedUser}
    <div class="details-view"> 
        
        <header class="header">
            <button class="back-button" on:click={goBack}>
                ← Back to User List
            </button>
            <h1>Details for {selectedUser.name}</h1>
        </header>
        
        <div class="summary-card">
            <h3>User Summary</h3>
            <p><strong>Registration Date on Website:</strong> {selectedUser.registrationDate}</p> 
            <p><strong>Email:</strong> {selectedUser.email}</p>
        </div>
        
        <h2>Image Upload History</h2>
        {#if selectedUser.activity && selectedUser.activity.length > 0}
            <div class="activity-log">
                {#each selectedUser.activity as activityItem, index}
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
        
    </div> {:else}
    <header class="header">
      </header>
    {/if}
  
  </div>
  <header class="header">
    <h1>User Management</h1>
    <button class="add-button" on:click={() => openModal()}>
      + Add New User
    </button>
  </header>

  <div class="table-container">
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Email</th>
          <th>Password</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        {#each users as user (user.id)}
          <tr>
            <td>{user.id}</td>
            <td>{user.name}</td>
            <td>{user.email}</td>
            <td>{user.password}</td>
            <td>
<button class="action-btn view" on:click={() => viewDetails(user)}>Details</button>              <button class="action-btn edit" on:click={() => openModal(user)}>Edit</button>
<button class="action-btn delete" on:click={() => startDelete(user.id)}>Delete</button>            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</div>
{#if isConfirmModalOpen}
  <ConfirmDeleteModal 
    on:confirm={confirmDelete}
    on:cancel={closeConfirmModal}
  />
{/if}
{#if isModalOpen}
  <div class="modal-backdrop" on:click={closeModal}>
    <div class="modal-content" on:click|stopPropagation>
      <h2>{currentEditingUser ? 'Edit User' : 'Add New User'}</h2>
      
      <form on:submit|preventDefault={saveUser}>
        
        <div class="form-group">
          <label for="name">Name:</label>
          <input id="name" type="text" bind:value={newUser.name} required />
        </div>
        
        <div class="form-group">
          <label for="email">Email:</label>
          <input id="email" type="email" bind:value={newUser.email} required />
        </div>
        
        <div class="form-group">
          <label for="password">Password: {currentEditingUser ? '(Leave blank to keep existing)' : '(Required)'}</label>
          <input 
            id="password" 
            type="password" 
            bind:value={newUser.password} 
            required={!currentEditingUser} 
            minlength={currentEditingUser ? 0 : 6}
          />
        </div>
        
        <div class="form-actions">
          <button type="button" class="cancel-btn" on:click={closeModal}>Cancel</button>
          <button type="submit" class="save-btn">{currentEditingUser ? 'Save Changes' : 'Add User'}</button>
        </div>
      </form>
    </div>
  </div>
{/if}
</main>