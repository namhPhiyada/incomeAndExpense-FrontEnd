<template>
    <div class="container-detail">
        <div class="bar-detail">
            <div class="row">
                <label class="label">วันเดือนปี</label>
                <div class="flex-auto">
                    <Calendar v-model="icondisplay" showIcon iconDisplay="input" inputId="icondisplay"
                        dateFormat="dd/mm/yy" placeholder="dd/mm/yyyy"
                        :class="{ 'input-error-date': inputError.icondisplay }" />
                </div>
            </div>
            <div class="row">
                <label class="label">รายการ</label>
                <input type="text" v-model="item" class="input" :class="{ 'input-error': inputError.item }"
                    placeholder="ซื้อของใช้เข้าบ้าน" />
            </div>
            <div class="row">
                <label class="label">รายรับ</label>
                <input type="number" v-model="income" class="input" :class="{ 'input-error': inputError.income }"
                    placeholder="1000" min="0" />
            </div>
            <div class="row">
                <label class="label">รายจ่าย</label>
                <input type="number" v-model="expanse" class="input" :class="{ 'input-error': inputError.expanse }"
                    placeholder="1000" min="0" />
            </div>
            <div class="button-group">
                <button class="btn btn-primary" @click="sendData()">เพิ่ม</button>
                <!-- <button class="btn btn-primary">แก้ไข</button> -->
            </div>

        </div>
        <!-- Popup Add-->
        <div v-if="showPopupAdd" class="popup-overlay">
            <div class="popup-content">
                <p>{{ popupMessage }}</p>
                <button @click="showPopupAdd = false">ปิด</button>
            </div>
        </div>

        <div v-if="showPopupCheckValue" class="popup-overlay">
            <div class="popup-content">
                <p>{{ popupMessage }}</p>
                <button @click="showPopupCheckValue = false">ปิด</button>
            </div>
        </div>

    </div>
    <!-- DataTable อยู่ด้านล่าง -->
    <div class="data-table-wrapper">
        <DataTable ref="dataTable" />
    </div>
</template>


<script>
import { ref, watch } from 'vue';
import DataTable from './DataTable.vue';

export default {
    name: 'MainComponent',
    components: {
        DataTable
    },
    setup() {
        const icondisplay = ref();
        const item = ref('');
        const income = ref();
        const expanse = ref();
        const dataTable = ref(null);
        const showPopupAdd = ref(false);
        const showPopupCheckValue = ref(false);
        const popupMessage = ref('');
        const inputError = ref({
            icondisplay: false,
            item: false,
            income: false,
            expanse: false
        });

        // เพิ่ม watcher สำหรับแต่ละ input
        watch(icondisplay, () => { inputError.value.icondisplay = false; });
        watch(item, () => { inputError.value.item = false; });
        watch(income, () => { inputError.value.income = false; });
        watch(expanse, () => { inputError.value.expanse = false; });

        const checkValue = () => {
            inputError.value = {
                icondisplay: !icondisplay.value,
                item: !item.value,
                income: income.value === undefined || income.value === null || income.value < 0,
                expanse: expanse.value === undefined || expanse.value === null || expanse.value < 0
            };
            if (
                inputError.value.icondisplay ||
                inputError.value.item ||
                inputError.value.income ||
                inputError.value.expanse
            ) {
                popupMessage.value = 'กรุณากรอกข้อมูลให้ครบถ้วนและถูกต้อง';
                showPopupCheckValue.value = true;
                return false;
            }
            return true;
        };

        const sendData = async () => {
            if (!checkValue()) {
                return;
            }
            try {
                const response = await fetch('http://localhost:8081/data', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        date: icondisplay.value,
                        item: item.value,
                        income: income.value,
                        expanse: expanse.value
                    })
                });

                if (response.ok) {
                    popupMessage.value = 'เพิ่มข้อมูลเรียบร้อย';
                    showPopupAdd.value = true;
                    dataTable.value.fetchData();
                    icondisplay.value = null;
                    item.value = '';
                    income.value = 0;
                    expanse.value = 0;
                } else {
                    popupMessage.value = 'เกิดข้อผิดพลาดในการเพิ่มข้อมูล';
                    showPopupAdd.value = true;
                }
            } catch (error) {
                console.error('Error sending data:', error);
                popupMessage.value = 'เกิดข้อผิดพลาดในการเชื่อมต่อ';
                showPopupAdd.value = true;
            }
        };

        return {
            icondisplay,
            item,
            income,
            expanse,
            sendData,
            dataTable,
            showPopupAdd,
            showPopupCheckValue,
            popupMessage,
            inputError
        };
    },
    methods: {
        checkValue() {
            if (!this.icondisplay || !this.item || this.income < 0 || this.expanse < 0) {
                this.popupMessage = 'กรุณากรอกข้อมูลให้ครบถ้วนและถูกต้อง';
                this.showPopupCheckValue = true;
                return false;
            }
            return true;
        },
    }
};
</script>



<style scoped>
.btn-primary {
    background: linear-gradient(90deg, #508f9a 0%, #006A7D 100%);
    color: #fff;
    border: none;
    border-radius: 8px;
    padding: 8px 18px;
    font-weight: 600;
    cursor: pointer;
    box-shadow: 0 2px 8px rgba(96, 165, 250, 0.08);
    transition: background 0.2s, box-shadow 0.2s;
}

.btn-primary:hover {
    background: linear-gradient(90deg, #006A7D 0%, #508f9a 100%);
    box-shadow: 0 4px 16px rgba(37, 99, 235, 0.12);
}

.container-detail {
    display: flex;
    justify-content: center;
    padding: 20px;
    max-width: 100%;
}

.bar-detail {
    width: 100%;
    max-width: 600px;
    display: flex;
    flex-direction: column;
    gap: 15px;
}

.row {
    display: flex;
    align-items: center;
    gap: 10px;
    flex-wrap: wrap;
}

.label {
    width: 100px;
    font-family: "Prompt", sans-serif;
    font-size: 1rem;
    color: #006A7D;
    text-align: right;
}

.input {
    flex: 1;
    min-width: 200px;
    padding: 8px 12px;
    border: none;
    border-radius: 10px;
    background-color: #D9D9D9;
    font-family: "Prompt", sans-serif;
    font-size: 1rem;
}

.input-error {
    border: 2px solid #e53935 !important;
    background-color: #fff0f0;
}

.input-error-date {
    border: 2px solid #e53935 !important;
    border-radius: 10px;
    background-color: #fff0f0;

}

.button-group {
    display: flex;
    justify-content: center;
    gap: 20px;
    margin-top: 10px;
    flex-wrap: wrap;
    margin-left: 17%;
}

.btn {
    padding: 10px 25px;
    font-family: "Prompt", sans-serif;
    font-size: 1rem;
    background-color: #006A7D;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    width: 20%;
}

.btn:hover {
    opacity: 0.85;
}

.flex-auto {
    flex: 1;
    min-width: 200px;
}

:deep(.p-inputtext) {
    flex: 1;
    width: 100%;
    min-width: 487px;
    padding: 8px 12px;
    border: none;
    border-radius: 10px;
    background-color: #D9D9D9;
    font-family: "Prompt", sans-serif;
    font-size: 1rem;
}

/* เว้นระยะห่างด้านบน DataTable */
.data-table-wrapper {
    margin-top: 20px;
    width: 100%;
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
    background: linear-gradient(90deg, #508f9a 0%, #006A7D 100%);
    margin-top: 15px;
    padding: 8px 20px;
    border: none;
    /* background: #006A7D; */
    color: #fff;
    border-radius: 6px;
    cursor: pointer;
    font-size: large;
    font-weight: 700;
}

.popup-content button:hover {
    background: linear-gradient(90deg, #006A7D 0%, #508f9a 100%);
}

.popup-content p {
    font-size: large;
    font-weight: 700;
}

/* 📱 Responsive for small screens */
@media (max-width: 600px) {
    .row {
        flex-direction: column;
        align-items: flex-start;
    }

    .label {
        text-align: left;
        width: 100%;
        margin-bottom: 5px;
    }

    .input {
        width: 100%;
    }

    .button-group {
        margin-left: 0;
        justify-content: center;
    }

    .flex-auto {
        width: 100%;
        min-width: 0;
    }

    :deep(.p-inputtext) {
        width: 100%;
        min-width: 0;
        max-width: 100%;
        font-size: 0.95rem;
    }
}
</style>