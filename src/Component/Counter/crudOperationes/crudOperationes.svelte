<script>
  import { writable, derived, get } from 'svelte/store';

  function createPersistedStore(key, initialValue) {
  const isClient = typeof window !== 'undefined';

  const stored = isClient ? localStorage.getItem(key) : null;
  const parsed = stored ? JSON.parse(stored) : initialValue;
  const store = writable(parsed);

  if (isClient) {
    store.subscribe((value) => {
      localStorage.setItem(key, JSON.stringify(value));
    });
  }

  return store;
}


  const users = createPersistedStore("users", []);
  const name = writable("");
  const email = writable("");
  const number = writable("");
  const selectedUserId = writable(null);
  const searchTerm = writable("");

  let idCounter = 1;

if (typeof window !== 'undefined') {
  idCounter = Number(localStorage.getItem("idCounter")) || 1;
}

function updateIdCounter() {
  idCounter++;
  if (typeof window !== 'undefined') {
    localStorage.setItem("idCounter", idCounter);
  }
}


  // Derived store for filtered results
  const filteredUsers = derived(
    [users, searchTerm],
    ([$users, $searchTerm]) =>
      $users.filter(
        (user) =>
          user.name.toLowerCase().includes($searchTerm.toLowerCase()) ||
          user.email.toLowerCase().includes($searchTerm.toLowerCase()) ||
          user.number.includes($searchTerm)
      )
  );

  function createOrUpdate() {
    const nameValue = get(name).trim();
    const emailValue = get(email).trim();
    const numberValue = get(number).trim();
    const selectedId = get(selectedUserId);

    if (!nameValue || !emailValue || !numberValue) return;

    if (selectedId) {
      users.update(($users) =>
        $users.map((user) =>
          user.id === selectedId
            ? { ...user, name: nameValue, email: emailValue, number: numberValue }
            : user
        )
      );
    } else {
      users.update(($users) => [
        ...$users,
        { id: idCounter, name: nameValue, email: emailValue, number: numberValue }
      ]);
      updateIdCounter();
    }

    resetForm();
  }

  function editUser(user) {
    name.set(user.name);
    email.set(user.email);
    number.set(user.number);
    selectedUserId.set(user.id);
  }

  function deleteUser(id) {
    users.update(($users) => $users.filter((user) => user.id !== id));
    resetForm();
  }

  function resetForm() {
    name.set("");
    email.set("");
    number.set("");
    selectedUserId.set(null);
  }
</script>

<!-- UI -->
<div class="max-w-4xl mx-auto p-4 space-y-6">
  <h2 class="text-2xl font-bold text-center">
    {$selectedUserId ? 'Update User' : 'Add New User'}
  </h2>

  <!-- Form -->
  <div class="space-y-3">
    <input type="text" class="w-full border rounded px-3 py-2" placeholder="Name" bind:value={$name} />
    <input type="email" class="w-full border rounded px-3 py-2" placeholder="Email" bind:value={$email} />
    <input type="text" class="w-full border rounded px-3 py-2" placeholder="Phone" bind:value={$number} />

    <div class="flex gap-3">
      <button
        on:click={createOrUpdate}
        class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
        disabled={!$name || !$email || !$number}
      >
        {$selectedUserId ? 'Update' : 'Create'}
      </button>
      <button
        on:click={resetForm}
        class="bg-gray-500 text-white px-4 py-2 rounded hover:bg-gray-600"
      >
        Reset
      </button>
    </div>
  </div>

  <!-- Search -->
  <input
    type="text"
    class="w-full border rounded px-3 py-2 mt-4"
    placeholder="Search by name, email, or phone number"
    bind:value={$searchTerm}
  />

  <!-- Table -->
  <table class="w-full table-auto border mt-6 border-collapse border-gray-300">
    <thead class="bg-gray-100">
      <tr>
        <th class="border px-3 py-2">#</th>
        <th class="border px-3 py-2">Name</th>
        <th class="border px-3 py-2">Email</th>
        <th class="border px-3 py-2">Phone</th>
        <th class="border px-3 py-2">Actions</th>
      </tr>
    </thead>
    <tbody>
      {#each $filteredUsers as user, i}
        <tr class="text-center">
          <td class="border px-3 py-2">{i + 1}</td>
          <td class="border px-3 py-2">{user.name}</td>
          <td class="border px-3 py-2">{user.email}</td>
          <td class="border px-3 py-2">{user.number}</td>
          <td class="border px-3 py-2 space-x-2">
            <button
              on:click={() => editUser(user)}
              class="bg-yellow-500 text-white px-3 py-1 rounded hover:bg-yellow-600"
            >
              Edit
            </button>
            <button
              on:click={() => deleteUser(user.id)}
              class="bg-red-600 text-white px-3 py-1 rounded hover:bg-red-700"
            >
              Delete
            </button>
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>
