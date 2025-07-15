<template>
    <div class="container-data-table">
        <div class="bar-data-table">
            <button class="btn btn-danger" @click="deleteSelected" :disabled="!hasSelectedRows">
                ลบรายการที่เลือก ({{ selectedCount }})
            </button>
            <div class="search-container">
                <i class="pi pi-search search-icon" />
                <input type="text" v-model="searchText" class="search-input"
                    placeholder="ค้นหาด้วยวันที่ (04/07/2025)" />
                <button class="btn btn-primary refresh-btn" @click="clearFilteredRows()">refresh</button>
            </div>
        </div>
        <div class="data-table-content">
            <table>
                <thead>
                    <tr>
                        <th>
                            <input type="checkbox" v-model="selectAll" @change="toggleAll" />
                            เลือกทั้งหมด
                        </th>
                        <th>ลำดับ</th>
                        <th>วันเดือนปี</th>
                        <th>รายการ</th>
                        <th>รายรับ</th>
                        <th>รายจ่าย</th>
                        <th>ยอดคงเหลือ</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in filteredRows" :key="item.id || index">
                        <td>
                            <input type="checkbox" v-model="item.selected" />
                        </td>
                        <td>{{ index + 1 }}</td>
                        <td>{{ formatDate(item.date) }}</td>
                        <td>{{ item.item }}</td>
                        <td>{{ formatCurrency(item.income) }}</td>
                        <td>{{ formatCurrency(item.expanse) }}</td>
                        <td>{{ formatCurrency(item.income - item.expanse) }}</td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div v-if="showConfirmPopup" class="popup-overlay">
            <div class="popup-content">
                <p>{{ confirmPopupMessage }}</p>
                <button @click="confirmDelete">ยืนยัน</button>
                <button @click="showConfirmPopup = false">ยกเลิก</button>
            </div>
        </div>
        <!-- Popup สำหรับแจ้งผลลบ -->
        <div v-if="showDeletePopup" class="popup-overlay">
            <div class="popup-content">
                <p>{{ deletePopupMessage }}</p>
                <button @click="showDeletePopup = false">ปิด</button>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'DataTable',
    data() {
        return {
            selectAll: false,
            rows: [],
            showDeletePopup: false,
            deletePopupMessage: '',
            showConfirmPopup: false,
            confirmPopupMessage: '',
            rowsToDelete: [],
            searchText: ''
        };
    },
    computed: {
        filteredRows() {
            if (!this.searchText || this.searchText.trim() === '') {
                return this.rows;
            }

            const search = this.searchText.trim().toLowerCase();
            return this.rows.filter(item => {
                const formattedDate = this.formatDate(item.date).toLowerCase();
                return formattedDate.includes(search);
            });
        },
        hasSelectedRows() {
            return this.filteredRows.some(item => item.selected);
        },
        selectedCount() {
            return this.filteredRows.filter(item => item.selected).length;
        }
    },

    created() {
        this.fetchData();
    },
    methods: {
        fetchData() {
            fetch('http://localhost:8081/data')
                .then(response => response.json())
                .then(data => {
                    this.rows = data.map(item => ({
                        ...item,
                        selected: false
                    }));
                })
                .catch(error => {
                    console.error('Error fetching data:', error);
                });
        },

        toggleAll() {
            this.rows.forEach(item => {
                item.selected = this.selectAll;
            });
        },

        deleteSelected() {
            const selectedRows = this.rows.filter(row => row.selected);

            if (selectedRows.length === 0) {
                this.deletePopupMessage = 'กรุณาเลือกรายการที่ต้องการลบ';
                this.showDeletePopup = true;
                return;
            }

            // แสดง popup ยืนยัน
            this.confirmPopupMessage = `คุณต้องการลบ ${selectedRows.length} รายการหรือไม่?`;
            this.rowsToDelete = selectedRows;
            this.showConfirmPopup = true;
        },

        async confirmDelete() {
            this.showConfirmPopup = false;
            try {
                const deletePromises = this.rowsToDelete.map(item =>
                    fetch(`http://localhost:8081/data/${item.id}`, {
                        method: 'DELETE',
                        headers: {
                            'Content-Type': 'application/json'
                        }
                    })
                );

                const responses = await Promise.all(deletePromises);
                const failedDeletes = responses.filter(response => !response.ok);

                if (failedDeletes.length > 0) {
                    this.deletePopupMessage = `ลบไม่สำเร็จ ${failedDeletes.length} รายการ`;
                } else {
                    this.deletePopupMessage = 'ลบข้อมูลเรียบร้อยแล้ว';
                }
                this.showDeletePopup = true;

                this.fetchData();
                this.selectAll = false;

            } catch (error) {
                console.error('Error deleting data:', error);
                this.deletePopupMessage = 'เกิดข้อผิดพลาดในการลบข้อมูล';
                this.showDeletePopup = true;
            }
        },

        formatCurrency(value) {
            return new Intl.NumberFormat('th-TH', {
                style: 'currency',
                currency: 'THB'
            }).format(value);
        },

        formatDate(dateStr) {
            if (!dateStr) return '';
            const date = new Date(dateStr);
            const d = date.getDate().toString().padStart(2, '0');
            const m = (date.getMonth() + 1).toString().padStart(2, '0');
            const y = date.getFullYear();
            return `${d}/${m}/${y}`;
        },

        clearFilteredRows() {
            this.searchText = '';
            this.selectAll = false;
            this.rows.forEach(item => {
                item.selected = false;
            });
        }
    }
};
</script>



<style scoped>
.bar-data-table {
    display: flex;
    align-items: center;
    gap: 18px;
    margin-bottom: 18px;
    flex-wrap: wrap;
}

.search-container {
    display: flex;
    align-items: center;
    gap: 10px;
    background: #f8fafc;
    border-radius: 10px;
    padding: 6px 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
    position: relative;
}

.refresh-btn {
    margin-left: 8px;
    /* ใช้ style เดิมของ btn-primary */
}

.search-input {
    width: 300px;
    padding: 8px 12px 8px 36px;
    border: none;
    border-radius: 8px;
    font-size: 16px;
    color: #333;
    background: #f1f5f9;
    outline: none;
    box-sizing: border-box;
    transition: box-shadow 0.2s;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.03);
}

.search-input:focus {
    box-shadow: 0 0 0 2px #60a5fa;
    background: #fff;
}

.search-icon {
    position: absolute;
    top: 50%;
    left: 22px;
    transform: translateY(-50%);
    color: #60a5fa;
    font-size: 18px;
    pointer-events: none;
}

.refresh-button {
    margin-left: 8px;
}

.btn-primary {
    background: linear-gradient(90deg, #508f9a 0%, #006A7D 100%);
    color: #fff;
    border: none;
    border-radius: 8px;
    padding: 8px 18px;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    box-shadow: 0 2px 8px rgba(96, 165, 250, 0.08);
    transition: background 0.2s, box-shadow 0.2s;
}

.btn-primary:hover {
    background: linear-gradient(90deg, #006A7D 0%, #508f9a 100%);
    box-shadow: 0 4px 16px rgba(37, 99, 235, 0.12);
}

.btn-danger {
    background: linear-gradient(90deg, #f87171 0%, #dc2626 100%);
    color: white;
    border: none;
    padding: 8px 20px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 16px;
    font-weight: 600;
    box-shadow: 0 2px 8px rgba(220, 38, 38, 0.08);
    transition: background 0.2s, box-shadow 0.2s;
}

.btn-danger:hover:not(:disabled) {
    /* background: linear-gradient(90deg, #dc2626 0%, #f87171 100%); */
    box-shadow: 0 4px 16px rgba(220, 38, 38, 0.12);
}

.btn-danger:disabled {
    background: #f3f4f6;
    color: #bdbdbd;
    cursor: not-allowed;
    box-shadow: none;
}

.search-bar {
    display: flex;
    align-items: center;
    gap: 8px;
    margin-top: 8px;
}


.input-search:focus {
    border-color: #006A7D;
    outline: none;
}

.pi-search {
    font-size: 18px;
    color: #666;
}

.popup-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
}

.popup-content {
    background: #fff;
    padding: 30px 40px;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
    text-align: center;
}

.popup-content button {
    background: linear-gradient(90deg, #f87171 0%, #dc2626 100%);
    margin-top: 15px;
    margin-right: 10px;
    padding: 8px 20px;
    border: none;
    /* background: #dc3545; */
    color: #fff;
    border-radius: 6px;
    cursor: pointer;
    font-size: large;
    font-weight: 700;
}

.popup-content button:hover {
    background: linear-gradient(90deg, #dc2626 0%, #f87171 100%);
}

.popup-content p {
    font-size: large;
    font-weight: 700;
}

/* .btn-danger {
    background-color: #dc3545;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 4px;
    cursor: pointer;
    margin-top: 10px;
} */

/* .btn-danger:hover {
    background-color: #c82333;
} */

.btn-danger:disabled {
    background-color: #6c757d;
    cursor: not-allowed;
}

.container-data-table {
    padding: 20px;
    background-color: #ffffff;
    border-radius: 16px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    overflow-x: auto;
}

.data-table-content table {
    width: 100%;
    border-collapse: collapse;
    background-color: #ffffff;
    border-radius: 12px;
    overflow: hidden;
}

.data-table-content th,
.data-table-content td {
    padding: 12px 16px;
    text-align: left;
    border-bottom: 1px solid #e5e7eb;
}

.data-table-content th {
    background-color: #f3f4f6;
    font-weight: 600;
    color: #374151;
}

.data-table-content td {
    color: #4b5563;
}

.data-table-content tr:hover {
    background-color: #f9f9f9;
    transition: background-color 0.2s;
}

.data-table-content tr:nth-child(even) {
    background-color: #f8f9fa;
}

.data-table-content input[type="checkbox"] {
    width: 16px;
    height: 16px;
    accent-color: #3b82f6;
}

@media (max-width: 768px) {
    .data-table-content table {
        font-size: 14px;
    }

    .data-table-content th,
    .data-table-content td {
        padding: 8px 12px;
    }
}
</style>
