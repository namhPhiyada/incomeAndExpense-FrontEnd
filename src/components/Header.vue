<template>
    <div class="container-header">
        <div class="bar-header">
            <h1 class="text-header">บัญชีรายรับรายจ่าย</h1>
            <div class="user-info">
                <span class="material-symbols-outlined">account_circle</span>
                <div class="user-text">
                    <h3>ผู้ใช้: admin</h3>
                    <h4>{{ currentDateTime }}</h4>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'MainHeader',
    data() {
        return {
            currentDateTime: ''
        };
    },
    methods: {
        updateDateTime() {
            const now = new Date();
            const formattedDate = now.toLocaleDateString('th-TH', {
                day: '2-digit',
                month: '2-digit',
                year: 'numeric'
            });
            const formattedTime = now.toLocaleTimeString('th-TH', {
                hour: '2-digit',
                minute: '2-digit',
                second: '2-digit'
            });
            this.currentDateTime = `${formattedDate} ${formattedTime}`;
        }
    },
    mounted() {
        this.updateDateTime();
        setInterval(this.updateDateTime, 1000);
    }
};
</script>


<style scoped>
.container-header {
    width: 100%;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.bar-header {
    background: linear-gradient(90deg, #006A7D 0%, #4a7e89 100%);
    /* background-color: #006A7D; */
    color: #E9F874;
    padding: 1.5rem 3rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    border-bottom: 5px solid #E9F874;
}

.text-header {
    font-family: "Prompt", sans-serif;
    font-size: 2.5vw;
    font-weight: bold;
    margin: 0;
}

.user-info {
    display: flex;
    align-items: center;
    gap: 1rem;
}

.material-symbols-outlined {
    font-size: 3.5vw;
}

.user-text h3,
.user-text h4 {
    margin: 0;
    font-family: "Prompt", sans-serif;
    font-weight: normal;
}

/* Responsive สำหรับหน้าจอขนาดกลางถึงเล็ก */
@media (max-width: 768px) {
    .bar-header {
        flex-direction: column;
        align-items: flex-start;
        padding: 1rem 2rem;
    }

    .text-header {
        font-size: 6vw;
        margin-bottom: 1rem;
    }

    .material-symbols-outlined {
        font-size: 6vw;
    }
}

/* Responsive สำหรับจอใหญ่ */
@media (min-width: 1200px) {
    .text-header {
        font-size: 2vw;
    }

    .material-symbols-outlined {
        font-size: 2.5vw;
    }
}
</style>
