<template>
  <section class="flex flex-col items-center justify-center px-4 mb-4">
    <div class="text-[24px] font-semibold text-dark">Tambah Data</div>
    <p class="mt-2 text-base leading-7 text-center mb-[50px] text-grey">
      Kas Kecil<br />
    </p>

    <div class="w-full card-baru">
      <div class="flex mb-8 justify-center">
        <button
          class="bg-red-700 text-white px-4 py-2 mr-0 rounded-l-lg hover:bg-red-800"
          :class="{ 'bg-opacity-50': !inputType }"
          @click.prevent="changeDataPengeluaran"
        >
          Pengeluaran
        </button>
        <button
          class="bg-green-700 text-white px-4 py-2 ml-0 rounded-r-lg hover:bg-green-800"
          :class="{ 'bg-opacity-50': inputType }"
          @click="changeDataPemasukan"
        >
          Pemasukan
        </button>
        
      </div>
      <form class="w-full card-baru" @submit.prevent="createKaskecil">
        <p v-if="listerror" class="m-3">
        <b>Tolong Di Cek Kembali :</b>
        <ul class="list-disc">
          <li v-for="error in errors" :key="error[0]" class="text-red-500">
          {{ error[0] }}
          </li>
        </ul>
      </p>
        <div class="grid grid-cols-3 gap-4">
          <div class="form-group">
            <label for="" class="text-grey">Tanggal</label>
            <input
              type="date"
              class="input-field2"
              required
              v-model="kaskecil.tanggal"
            />
          </div>
          <div class="form-group col-span-2">
            <label for="" class="text-grey">No Bukti Penerima</label>
            <input
              type="text"
              class="input-field2"
              v-model="kaskecil.nobuktikas"
              required
            />
          </div>
        </div>
        <div class="form-group">
          <label for="" class="text-grey">Uraian Transaksi</label>
          <input type="text" class="input-field2" v-model="kaskecil.name" />
        </div>

        <div class="grid grid-cols-2 gap-4">
          <div class="form-group">
            <label for="" class="text-grey">Penerima</label>
            <input
              type="text"
              class="input-field2"
              v-model="kaskecil.penerima"
            />
          </div>
        </div>
        <div class="form-group">
          <label for="" class="text-grey">Pilih Akun Debit</label>
          <p v-if="$fetchState.pending">Fetching debits...</p>
          <select
            v-else
            v-model="kaskecil.coadebit_id"
            name="debit"
            id=""
            class="appearance-none input-field2 form-icon-chevron_down w-9/12"
          >
            <option
              :value="item.id"
              v-for="item in debit.data.result.data"
              :key="'B' + item.id"
            >
              {{ item.name }}
            </option>
          </select>
        </div>
        <div class="form-group">
          <label for="" class="text-grey">Pilih Akun Kredit</label>
          <p v-if="$fetchState.pending">Fetching roles...</p>
          <select
            v-else
            v-model="kaskecil.coakredit_id"
            name="kredit"
            id=""
            class="appearance-none input-field2 form-icon-chevron_down w-9/12"
          >
            <option
              :value="item2.id"
              v-for="item2 in kredit.data.result.data"
              :key="'A' + item2.id"
            >
              {{ item2.name }}
            </option>
          </select>
        </div>
        <div class="grid grid-cols-2 gap-4">
          <div class="form-group">
            <label for="" class="text-grey">Jumlah</label>
            <input type="text" class="input-field2" v-model="kaskecil.jumlah" />
          </div>
          <div class="form-group">
            <label for="" class="text-grey">Ref</label>
            <input
              type="text"
              class="input-field2"
              v-model="kaskecil.ref"
              
            />
          </div>
        </div>

        <button type="submit" class="w-full btn btn-lazismu mt-[14px]">
          Create
        </button>
      </form>
    </div>
  </section>
</template>

<script>
export default {
  layout: 'form',
  middleware: 'auth',
  data() {
    return {
      debit: [],
      kredit: [],
      temp: [],
      kaskecil: {
        name: '',
        penerima: '',
        nobuktikas: '',
        tanggal: '',
        ref: '',
        jumlah: '',
        coadebit_id: '',
        coakredit_id: '',
        cabang_id: this.$route.params.id,
        jenis_data: 0,
      },
      cabang_id: this.$route.params.id,
      inputType: false,
      cabang: [],
      listerror: false,
      errors: [],
    }
  },
  async fetch() {
    ;(this.debit = await this.$axios.get('/coa', {
      params: {
        limit: 100,
        cabang_id: this.cabang_id,
        tipe: false,
      },
    })),
      (this.kredit = await this.$axios.get('/coa', {
        params: {
          limit: 100,
          cabang_id: this.cabang_id,
          tipe: 1,
        },
      }))
  },
  methods: {
    changeDataPengeluaran() {
      if (this.inputType === false) {
        console.log('ganti ke pengeluaran')
        this.kaskecil.jenis_data = 1
        this.inputType = true
        this.temp = this.debit
        this.debit = this.kredit
        this.kredit = this.temp
      }
      console.log('tetap')
    },
    changeDataPemasukan() {
      if (this.inputType === true) {
        this.inputType = false
        this.kaskecil.jenis_data = 0
        this.temp = this.debit
        this.debit = this.kredit
        this.kredit = this.temp
        console.log('ganti ke pemasukan')
      }
    },
    async createKaskecil() {
      try {
        //send registration data to server
        let response = await this.$axios.post('/kaskecil', this.kaskecil)
        //Redirect to my kaskecil page
        this.$router.push({
          name: 'cabang-id-kaskecil',
          params: {
            id: this.cabang_id,
          },
        })
        console.log(response)
      } catch (error) {
        this.errors = error.response.data.errors
        this.listerror = true
      }
    },
  },
}
</script>
