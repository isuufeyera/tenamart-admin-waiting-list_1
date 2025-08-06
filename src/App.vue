<template>
  <div class="min-h-screen bg-gray-50">
    <header class="bg-tena-green text-white shadow">
      <div class="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
        <h1 class="text-3xl font-bold">TenaMart Waiting List Admin</h1>
      </div>
    </header>

    <main class="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
      <div class="px-4 py-6 sm:px-0">
        <!-- Filters and Actions -->
        <div class="mb-8 bg-white p-6 rounded-lg shadow">
          <div
            class="flex flex-col md:flex-row md:items-center md:justify-between gap-4"
          >
            <div class="w-full md:w-1/3">
              <label
                for="search"
                class="block text-sm font-medium text-gray-700 mb-1"
                >Search</label
              >
              <input
                v-model="searchQuery"
                type="text"
                id="search"
                class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-tena-green focus:border-tena-green"
                placeholder="Search by name or email"
              />
            </div>

            <div class="w-full md:w-1/3">
              <label
                for="source-filter"
                class="block text-sm font-medium text-gray-700 mb-1"
                >Filter by Source</label
              >
              <select
                v-model="sourceFilter"
                id="source-filter"
                class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-tena-green focus:border-tena-green"
              >
                <option value="">All Sources</option>
                <option v-for="source in sources" :key="source" :value="source">
                  {{ source }}
                </option>
              </select>
            </div>

            <div class="flex items-end">
              <button
                @click="exportToCSV"
                class="px-4 py-2 bg-tena-green text-white rounded-md hover:bg-tena-green-dark focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-tena-green"
              >
                Export to CSV
              </button>
            </div>
          </div>
        </div>

        <!-- Chart Visualization -->
        <SignupChart
          v-if="showChart"
          :users="filteredUsers"
          class="mb-8 bg-white p-6 rounded-lg shadow"
        />

        <!-- User Cards Grid -->
        <div v-if="loading" class="flex justify-center items-center h-64">
          <div
            class="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-tena-green"
          ></div>
        </div>

        <div v-else>
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            <UserCard
              v-for="user in paginatedUsers"
              :key="user.id"
              :user="user"
              @delete-user="confirmDeleteUser"
              @block-user="confirmBlockUser"
            />
          </div>

          <PaginationControls
            v-if="filteredUsers.length > itemsPerPage"
            :current-page="currentPage"
            :total-pages="totalPages"
            @page-change="changePage"
            class="mt-8"
          />
        </div>
      </div>
    </main>

    <!-- Delete Confirmation Modal -->
    <div
      v-if="showDeleteModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4"
    >
      <div class="bg-white rounded-lg p-6 max-w-md w-full">
        <h3 class="text-lg font-medium mb-4">Confirm User Deletion</h3>
        <p class="mb-6">
          Are you sure you want to delete {{ userToDelete.name }}? This action
          cannot be undone.
        </p>
        <div class="flex justify-end space-x-3">
          <button
            @click="showDeleteModal = false"
            class="px-4 py-2 border border-gray-300 rounded-md text-gray-700 hover:bg-gray-50"
          >
            Cancel
          </button>
          <button
            @click="deleteUser"
            class="px-4 py-2 bg-red-600 text-white rounded-md hover:bg-red-700"
          >
            Delete
          </button>
        </div>
      </div>
    </div>

    <!-- Block Confirmation Modal -->
    <div
      v-if="showBlockModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4"
    >
      <div class="bg-white rounded-lg p-6 max-w-md w-full">
        <h3 class="text-lg font-medium mb-4">Confirm User Block</h3>
        <p class="mb-6">
          Are you sure you want to block {{ userToBlock.name }}? They will be
          removed from the waiting list.
        </p>
        <div class="flex justify-end space-x-3">
          <button
            @click="showBlockModal = false"
            class="px-4 py-2 border border-gray-300 rounded-md text-gray-700 hover:bg-gray-50"
          >
            Cancel
          </button>
          <button
            @click="blockUser"
            class="px-4 py-2 bg-yellow-600 text-white rounded-md hover:bg-yellow-700"
          >
            Block
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import UserCard from './components/UserCard.vue';
import PaginationControls from './components/PaginationControls.vue';
import SignupChart from './components/SignupChart.vue';
import { useUsers } from './composables/useUsers';

const {
  users,
  loading,
  sources,
  filteredUsers,
  searchQuery,
  sourceFilter,
  deleteUserById,
  blockUserById,
} = useUsers();

// Pagination
const itemsPerPage = 9;
const currentPage = ref(1);

const paginatedUsers = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredUsers.value.slice(start, end);
});

const totalPages = computed(() => {
  return Math.ceil(filteredUsers.value.length / itemsPerPage);
});

const changePage = (page) => {
  currentPage.value = page;
};

// Modals
const showDeleteModal = ref(false);
const showBlockModal = ref(false);
const userToDelete = ref(null);
const userToBlock = ref(null);

const confirmDeleteUser = (user) => {
  userToDelete.value = user;
  showDeleteModal.value = true;
};

const confirmBlockUser = (user) => {
  userToBlock.value = user;
  showBlockModal.value = true;
};

const deleteUser = () => {
  deleteUserById(userToDelete.value.id);
  showDeleteModal.value = false;
};

const blockUser = () => {
  blockUserById(userToBlock.value.id);
  showBlockModal.value = false;
};

// Chart visibility
const showChart = ref(true);

// Export to CSV
const exportToCSV = () => {
  const headers = ['Name', 'Email', 'Signup Date', 'Source', 'Status'];
  const csvContent = [
    headers.join(','),
    ...filteredUsers.value.map(
      (user) =>
        `"${user.name}","${user.email}","${user.signupDate}","${user.source}","${user.status}"`
    ),
  ].join('\n');

  const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  link.setAttribute('href', url);
  link.setAttribute('download', 'tenamart_waiting_list.csv');
  link.style.visibility = 'hidden';
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');

body {
  font-family: 'Inter', sans-serif;
}

.bg-tena-green {
  background-color: #2e8b57;
}

.bg-tena-green-dark {
  background-color: #267349;
}

.focus\:ring-tena-green:focus {
  --tw-ring-color: #2e8b57;
}

.focus\:border-tena-green:focus {
  --tw-border-opacity: 1;
  border-color: #2e8b57;
}
</style>
