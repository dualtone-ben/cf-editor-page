<template>
  <div>
    <div class="w-full flex justify-end p-4">
      <UButton @click="resetConfig()" variant="outline" color="red" class="">Logout</UButton>
    </div>
    <div class="w-screen min-h-[80vh] flex flex-col gap-2 justify-center items-center">
      <Head>
        <Title>Zones</Title>
      </Head>

      <div class="flex flex-col gap-4 justify-center items-center my-2">
        <h1 class="text-xl font-semibold">Zones</h1>
        <UInput icon="i-heroicons-magnifying-glass-20-solid" v-model="searchQuery" type="text" placeholder="Search" ref="searchInput" size="md" color="white" class="w-full sm:w-auto" />
      </div>
      <div v-if="!loading" :class="{ 'sm:grid-cols-1 lg:grid-cols-1 full:grid-cols-1' : zones.length < 2, 'sm:grid-cols-2 lg:grid-cols-3 full:grid-cols-4' : zones.length > 1 }" class="grid gap-4 justify-center items-center w-11/12 md:w-10/12 full:w-1/2">
        <div v-if="zones.length > 0" @click="setZone(zone)" class="bg-stone-100 rounded-md px-4 py-2 hover:bg-stone-200 dark:bg-stone-700 dark:hover:bg-stone-800 cursor-pointer group flex justify-between items-center"
          v-for="zone in filteredZones">
          <div class="flex gap-4 items-center">
            <UTooltip v-if="zone.status === 'active' && zone.paused !== true" text="Active">
              <UIcon name="i-clarity-circle-solid" class="text-green-400" />
            </UTooltip>
            <UTooltip v-if="zone.status !== 'active' && zone.paused !== true" text="Inactive">
              <UIcon name="i-clarity-circle-solid" class="text-red-500 animate-pulse" />
            </UTooltip>
            <UTooltip v-if="zone.paused === true" text="Domain is paused">
              <UIcon name="i-clarity-pause-solid" class="text-orange-400" />
            </UTooltip>
            <p>{{ zone.name }}</p>
          </div>
          <UIcon name="i-clarity-arrow-line" class="rotate-90 opacity-0 group-hover:opacity-100"/>
        </div>
        <div v-else>
          <p>No zones found</p>
        </div>
      </div>
      <div v-else>
        <Loader />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      apiKey: '',
      zones: [],
      loading: true,
      searchQuery: '',
    }
  },
  computed: {
        filteredZones() {
            return this.zones.filter((record) => {
                return record.name.toLowerCase().includes(this.searchQuery.toLowerCase())
            })
        }
    },
  mounted() {
    this.apiKey = localStorage.getItem('cf-api-key')
    if (!this.apiKey) {
      this.$router.push('/login')
    }
    else {
      this.getZones()
    }
  },
  methods: {
    saveApiKey() {
      localStorage.setItem('cf-api-key', this.apiKey);
      this.$router.reload()
    },
    setZone(zone) {
      localStorage.setItem('cf-zone-id', zone.id);
      localStorage.setItem('cf-zone-name', zone.name);
      this.$router.push('/records')
    },
    resetConfig() {
      localStorage.removeItem('cf-api-key');
      localStorage.removeItem('cf-zone-id');
      localStorage.removeItem('cf-zone-name');
      localStorage.removeItem('cf-dns-id');
      localStorage.removeItem('cf-dns-name');
      this.$router.push('/login')
    },
    sort(field) {
            this.dnsRecords.sort((a, b) => a[field].localeCompare(b[field]));
        },
    async getZones() {
      const response = await fetch('/api/zones', {
        method: 'POST',
        body: JSON.stringify({
          apiKey: this.apiKey
        }),
      })
      if (response.ok) {
        const data = await response.json();
        this.zones = data.result;
        this.loading = false;
      } else {
        console.error('HTTP-Error: ' + response.status);
        this.loading = false;
      }
    }
  }
}
</script>