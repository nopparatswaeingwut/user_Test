<template>
  <div class="page-background">
    <div class="d-flex justify-content-between align-items-center mb-3">
      <h2>รายชื่อ</h2>
      <button type="button" class="btn btn-primary ms-auto" title="เพิ่มข้อมูล" @click="showModal = true" >
        <i class="fas fa-plus"></i>
      </button>
    </div>

    <div>
      <div class="table-responsive">
        <table class="table table-striped table-bordered">
          <thead>
            <tr>
              <th>#</th>
              <th>ชื่อ-นามสกุล</th>
              <th>ชื่อเล่น</th>
              <th>วันเดือนปีเกิด</th>
              <th>อายุ</th>
              <th>เพศ</th>
              <th>จัดการข้อมูล</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(user, index) in paginatedUsers" :key="user._id">
              <td>{{ index + 1 + (currentPage - 1) * itemsPerPage }}</td>
              <td>{{ user.name }}</td>
              <td>{{ user.nickName }}</td>
              <td>{{ formatDate(user.birthDay) }}</td>
              <td>{{ user.age }}</td>
              <td>{{ user.gender }}</td>
              <td>
                <button class="btn btn-sm btn-primary me-2 mb-2" title="แก้ไข" @click="editUser(user)">
                  <i class="fas fa-edit"></i>
                </button>
                <button class="btn btn-sm btn-danger me-2 mb-2" title="ลบ" @click="deleteUser(user)">
                  <i class="fas fa-trash-alt"></i>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <nav>
        <ul class="pagination justify-content-center">
          <li class="page-item" :class="{ disabled: currentPage === 1 }">
            <a class="page-link" @click="changePage(currentPage - 1)">ก่อนหน้า</a>
          </li>
          <li v-for="page in totalPages" :key="page" class="page-item" :class="{ active: currentPage === page }">
            <a class="page-link" @click="changePage(page)">{{ page }}</a>
          </li>
          <li class="page-item" :class="{ disabled: currentPage === totalPages }">
            <a class="page-link" @click="changePage(currentPage + 1)">ถัดไป</a>
          </li>
        </ul>
      </nav>
    </div>

    <PopupUser v-if="showModal" @close="resetForm()" @save="addData" :isEditing="isEditing">
      <template #body>
        <form @submit.prevent="isEditing ? updateUser() : addData()">
          <div class="mb-3">
            <label for="fullName" class="form-label">ชื่อ-นามสกุล</label>
            <input type="text" class="form-control" id="fullName" v-model="fullName" required pattern="[ก-๙A-Za-z\s]+"
              title="กรุณากรอกตัวอักษรและช่องว่างเท่านั้น">
          </div>

          <div class="mb-3">
            <label for="nickname" class="form-label">ชื่อเล่น</label>
            <input type="text" class="form-control" id="nickname" v-model="nickName" required>
          </div>
          <div class="mb-3">
            <label for="birthdate" class="form-label">วันเดือนปีเกิด</label>
            <input type="date" class="form-control" id="birthdate" v-model="birthDate" required>
          </div>
          <div class="mb-3">
            <label for="age" class="form-label">อายุ</label>
            <input type="number" class="form-control" id="age" v-model="age" required>
          </div>
          <div class="mb-3">
            <label for="gender" class="form-label">เพศ</label>
            <select class="form-select" id="gender" v-model="gender" required>
              <option value="">เลือกเพศ</option>
              <option value="ชาย">ชาย</option>
              <option value="หญิง">หญิง</option>
            </select>
          </div>
        </form>
      </template>

      <template #footer>
        <button type="button" class="btn btn-secondary" @click="resetForm() ">ปิด</button>
        <button type="submit" class="btn btn-primary"
          @click="isEditing ? updateUser() : addData()">บันทึกข้อมูล</button>
      </template>
    </PopupUser>
  </div>
</template>



<script>
import PopupUser from './PopupUser.vue';
export default {
  components: {
    PopupUser
  },
  data() {
    return {
      showModal: false,
      isEditing: false,
      fullName: '',
      nickName: '',
      birthDate: '',
      age: '',
      gender: '',
      userId: null,
      users: [],
      currentPage: 1,
      itemsPerPage: 10,
      currentUser: null,
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.users.length / this.itemsPerPage);
    },
    paginatedUsers() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      return this.users.slice(start, start + this.itemsPerPage);
    },
  },
  methods: {
    addData() {
      const Pattern = /^[ก-๙A-Za-z\s]+$/;

      if (!this.fullName || !Pattern.test(this.fullName)) {
        alert("กรุณากรอกชื่อ-นามสกุลให้ถูกต้อง โดยไม่ใช้ตัวอักษรพิเศษ-ตัวเลข");
        return;
      }
      if (this.fullName.length > 60) {
        alert("ชื่อ-นามสกุลต้องไม่เกิน 60 ตัวอักษร");
        return;
      }

      if (!this.nickName || !Pattern.test(this.nickName)) {
        alert("กรุณากรอกชื่อเล่นให้ถูกต้อง โดยไม่ใช้ตัวอักษรพิเศษ-ตัวเลข");
        return;
      }
      if (this.nickName.length > 25) {
        alert("ชื่อ-นามสกุลต้องไม่เกิน 60 ตัวอักษร");
        return;
      }
      if (!this.birthDate) {
        alert("กรุณากรอกวันเดือนปีเกิด");
        return;
      }

      if (this.age === "" || this.age < 0 || this.age > 120) {
        alert("กรุณากรอกอายุให้ถูกต้อง");
        return;
      }

      if (this.gender === "" || !this.gender) {
        alert("กรุณาเลือกเพศ");
        return;
      }

      const myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/json");

      const raw = JSON.stringify({
        name: this.fullName,
        nickName: this.nickName,
        birthDay: this.birthDate,
        age: this.age,
        gender: this.gender
      });

      const requestOptions = {
        method: "POST",
        headers: myHeaders,
        body: raw,
        redirect: "follow"
      };

      fetch("http://localhost:3000/users", requestOptions)
        .then((response) => response.json())
        .then((result) => {
          console.log(result)
          this.getUser();
          this.resetForm();
        })
        .catch((error) => console.error(error));
    },

    getUser() {
      const myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/json");
      const raw = JSON.stringify();
      const requestOptions = {
        method: "GET",
        headers: myHeaders,
        body: raw,
        redirect: "follow"
      };

      fetch("http://localhost:3000/users", requestOptions)
        .then((response) => response.json())
        .then((result) => {
          this.users = result;
        })
        .catch((error) => console.error(error));

    },
    editUser(user) {
      this.userId = user._id;
      this.currentUser = user;
      this.fullName = user.name;
      this.nickName = user.nickName;
      const date = new Date(user.birthDay);
      this.birthDate = date.toISOString().split('T')[0];
      this.age = user.age;
      this.gender = user.gender;
      this.isEditing = true;
      this.showModal = true;
    },

    updateUser() {
      const Pattern = /^[ก-๙A-Za-z\s]+$/;

      if (!this.fullName || !Pattern.test(this.fullName)) {
        alert("กรุณากรอกชื่อ-นามสกุลให้ถูกต้อง โดยไม่ใช้ตัวอักษรพิเศษ-ตัวเลข");
        return;
      }
      if (this.fullName.length > 60) {
        alert("ชื่อ-นามสกุลต้องไม่เกิน 60 ตัวอักษร");
        return;
      }
      if (!this.nickName || !Pattern.test(this.nickName)) {
        alert("กรุณากรอกชื่อเล่นให้ถูกต้อง โดยไม่ใช้ตัวอักษรพิเศษ-ตัวเลข");
        return;
      }
      if (this.nickName.length > 25) {
        alert("ชื่อ-นามสกุลต้องไม่เกิน 60 ตัวอักษร");
        return;
      }

      if (!this.birthDate) {
        alert("กรุณากรอกวันเดือนปีเกิด");
        return;
      }

      if (this.age === "" || this.age < 0 || this.age > 120) {
        alert("กรุณากรอกอายุให้ถูกต้อง");
        return;
      }

      if (this.gender === "" || !this.gender) {
        alert("กรุณาเลือกเพศ");
        return;
      }

      const myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/json");

      const raw = JSON.stringify({
        id: this.userId,
        name: this.fullName,
        nickName: this.nickName,
        birthDay: this.birthDate,
        age: this.age,
        gender: this.gender
      });

      const requestOptions = {
        method: "PUT",
        headers: myHeaders,
        body: raw,
        redirect: "follow"
      };

      fetch("http://localhost:3000/users", requestOptions)
        .then((response) => response.json())
        .then((result) => {
          console.log(result);
          this.getUser();
          this.resetForm();
        })
        .catch((error) => console.error(error));

    },

    deleteUser(user) {
      this.userId = user._id;
      const myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/json");

      const raw = JSON.stringify({
        "id": this.userId
      });

      const requestOptions = {
        method: "DELETE",
        headers: myHeaders,
        body: raw,
        redirect: "follow"
      };

      fetch("http://localhost:3000/users", requestOptions)
        .then((response) => response.json())
        .then((result) => {
          console.log(result);
          this.getUser();
          this.resetForm();
        })
        .catch((error) => console.error(error));
    },

    resetForm() {
      this.showModal = false;
      this.isEditing = false;
      this.userId = '';
      this.fullName = '';
      this.nickName = '';
      this.birthDate = '';
      this.age = null;
      this.gender = '';
      this.currentUser = null;
    },
    changePage(page) {
      if (page < 1 || page > this.totalPages) return;
      this.currentPage = page;
    },
    formatDate(date) {
      const options = { year: "numeric", month: "2-digit", day: "2-digit" };
      return new Date(date).toLocaleDateString("th-TH", options);
    },
  },
  mounted() {
    this.getUser()
  },
};
</script>

<style scoped>
.page-background {
  background-color: #d3d3d3;
  min-height: 100vh;
  padding: 20px;
}

tr {
  text-align: center;
}

.pagination {
  cursor: pointer;
}
</style>
