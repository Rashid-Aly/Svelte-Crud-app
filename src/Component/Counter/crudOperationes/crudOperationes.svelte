<script>
  import { writable } from "svelte/store";
  import { tick } from "svelte";

  let users = writable([]);
  let name = writable("");
  let email = writable("");
  let number = writable("");
  let selectedUserId = writable(null);
  let idCounter = 1;

  function createOrUpdate() {
    if ($selectedUserId) {
      // Update
      users.update(($users) =>
        $users.map((user) =>
          user.id === $selectedUserId
            ? { ...user, name: $name, email: $email, number: $number }
            : user
        )
      );
    } else {
      // Create
      users.update(($users) => [
        ...$users,
        { id: idCounter++, name: $name, email: $email, number: $number }
      ]);
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

<div class="max-w-xl mx-auto p-4 space-y-6">
  <h2 class="text-2xl font-bold text-center">
    {$selectedUserId ? 'Update User' : 'Add New User'}
  </h2>

  <!-- Form -->
  <div class="space-y-3">
    <input
      type="text"
      class="w-full border border-gray-400 rounded px-3 py-2"
      placeholder="Name"
      bind:value={$name}
    />
    <input
      type="email"
      class="w-full border border-gray-400 rounded px-3 py-2"
      placeholder="Email"
      bind:value={$email}
    />
    <input
      type="number"
      class="w-full border border-gray-400 rounded px-3 py-2"
      placeholder="Phone Number"
      bind:value={$number}
    />

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

  <!-- Table -->
  <table class="w-full table-auto border border-collapse border-gray-300 mt-6">
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
      {#each $users as user, i}
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
