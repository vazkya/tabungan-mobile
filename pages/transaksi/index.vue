<template>
    <div class="container-fluid">
        <div class="row" style="padding-top: 250px;">
            <h2 class="my-4" style="color: black;">RIWAYAT TRANSAKSI</h2>
        <div class="col-lg-12">
            <nuxt-link to="/admin">
            <button
                type="button"
                class="btn btn-lg rounded-5 px-5 bg-secondary text-white"
                style="margin-bottom: 30px"
                >
                KEMBALI
            </button>
            </nuxt-link>
            <table class="table table-striped">
            <thead>
                <tr>
                <th>No</th>
                <th>NAMA</th>
                <th>TANGGAL</th>
                <th>BULAN</th>
                <th>KEPERLUAN</th>
                <th>JUMLAH</th>
                <th>AKSI</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(visitor, i) in visitors" :key="visitor.id">
                <td>{{ i + 1 }}</td>
                <td>{{ visitor.nama.nama }}</td>
                <td>{{ visitor.tanggal }}</td>
                <td>{{ visitor.bulan.nama }}</td>
                <td>{{ visitor.keperluan.nama }}</td>
                <td>Rp. {{ visitor.jumlah }}</td>
                <td>
                    <button @click="editData(visitor)" class="btn btn-primary">Edit</button><br>
                    <button style="margin-top: 5px;" @click="deleteTransaction(visitor.id)" class="btn btn-danger">Hapus</button>
                </td>
                </tr>
            </tbody>
            </table>

            <div v-if="isEditing" class="modal">
            <div class="modal-content">
                <h3>EDIT TRANSAKSI</h3>
                <div class="form-group">
                    <label for="tanggal">Tanggal</label>
                    <input id="tanggal" type="date" v-model="selectedTransaction.tanggal" />
                </div>
                <div class="form-group">
                    <label for="bulan">Bulan</label>
                    <input id="bulan" v-model="selectedTransaction.bulan.id" />
                    </div>
                <div class="form-group">
                    <label for="keperluan">Keperluan</label>
                    <input id="keperluan" v-model="selectedTransaction.keperluan.id" />
                </div>
                <div class="form-group">
                    <label for="jumlah">Jumlah</label>
                    <input id="jumlah" type="number" v-model="selectedTransaction.jumlah" />
                </div>
                <div class="button-group">
                    <button @click="updateData" class="simpan bg-primary">SIMPAN</button>
                    <button @click="isEditing = false" class="batal bg-secondary">BATAL</button>
                </div>
            </div>
        </div>
    </div>
    </div>
    </div>
</template>

<script setup>
useHead({title: "riwayat", meta: [{nama: "riwayat", content: "HALAMAN RIWAYAT"}]});
import { ref, onMounted } from 'vue';
const supabase = useSupabaseClient();
const visitors = ref([]);
const isEditing = ref(false);
const selectedTransaction = ref({});

const fetchTransactions = async () => {
try {
    const { data, error } = await supabase
    .from("transaksi")
    .select(`*, nama(*), bulan(*), keperluan(*)`)
    .order('id', { ascending: false });

    if (error) throw error;

    visitors.value = data;
    console.log("Fetched transactions:", visitors.value);
} catch (error) {
    console.error("Error fetching transactions:", error.message);
}
};

const editData = (visitor) => {
    selectedTransaction.value = { ...visitor };
    console.log("Editing transaction:", selectedTransaction.value);
    isEditing.value = true;
};


const updateData = async () => {
    try {
    console.log("Updating transaction:", selectedTransaction.value);
    const { error } = await supabase.from("transaksi").update({
        nama: selectedTransaction.value.nama.id,
        tanggal: selectedTransaction.value.tanggal,
        bulan: selectedTransaction.value.bulan.id,
        keperluan: selectedTransaction.value.keperluan.id,
        jumlah: selectedTransaction.value.jumlah,
    }).eq('id', selectedTransaction.value.id);

    if (error) throw error;

    isEditing.value = false;
    await fetchTransactions(); 
} catch (error) {
    console.error("Error updating data:", error.message);
}
};

const deleteTransaction = async (id) => {
const confirmed = confirm("Apakah Anda yakin ingin menghapus transaksi ini?");
    if (confirmed) {
        try {
            const { error } = await supabase.from('transaksi').delete().eq('id', id);
            if (error) throw error;

        visitors.value = visitors.value.filter(visitor => visitor.id !== id);
    } catch (error) {
        console.error("Error deleting transaction:", error.message);
        }
    }
};

onMounted(() => {
    fetchTransactions();
});
</script>

<style>
.modal {
    position: fixed;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
}

.modal-content {
    background-color: white; 
    padding: 20px;
    border-radius: 5px;
    width: 400px;
    display: flex;
    flex-direction: column;
    justify-content: center; 
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

h3{
    text-align:center ;
    margin-bottom: 20px;
    color: black;
}
.form-group{
    display: flex;
    justify-content: space-between;
    align-items:center ;
    margin-bottom: 15px;
    width: 100%;
    color: black;
}
.form-group label {
    flex: 1;
    text-align: right;
    margin-right: 10px;
}
.form-group input {
    flex: 2;
    padding: 5px;
    border-radius: 4px;
    border: 1px solid black;
}
.button-group{
    display: flex;
    justify-content: center;
    margin-top: 20px;
}
.simpan, .batal {
    width: 100px;
    color: white;
    margin: 0 10px;
}
</style>
