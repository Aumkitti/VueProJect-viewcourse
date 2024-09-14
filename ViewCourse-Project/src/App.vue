<template>
  <div id="app">
    <h1>หลักสูตร In-house Training</h1>

    <!-- Search bar -->
    <div class="search-container">
      <input
        type="text"
        v-model="searchQuery"
        placeholder="ค้นหาหลักสูตร..."
      />
    </div>

    <!-- Button for adding a new course -->
    <div class="button-container">
      <button @click="openModal">เพิ่มหลักสูตร</button>
      <button @click="exportToPDF">Export to PDF</button>
    </div>

    <!-- Table displaying the list of courses -->
    <table id="coursesTable">
      <thead>
        <tr>
          <th>เลือก</th>
          <th>ลำดับ</th>
          <th>ชื่อหลักสูตรฝึกอบรม</th>
          <th>หมวดหลักสูตร</th>
          <th>วิทยากร</th>
          <th>จำนวนวัน</th>
          <th>รายละเอียด</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(course, index) in filteredCourses"
          :key="index"
        >
          <td><input type="checkbox" /></td>
          <td>{{ index + 1 }}</td>
          <td>{{ course.name }}</td>
          <td>{{ course.category }}</td>
          <td>{{ course.instructor }}</td>
          <td>{{ course.days }} วัน</td>
          <td><button @click="editCourse(index)">รายละเอียด</button></td>
        </tr>
      </tbody>
    </table>

    <!-- Modal for adding or editing a course -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="closeModal">&times;</span>
        <h2>{{ editIndex === -1 ? 'เพิ่มหลักสูตรใหม่' : 'แก้ไขหลักสูตร' }}</h2>

        <form @submit.prevent="submitForm">
          <label for="courseName">ชื่อหลักสูตร:</label>
          <input
            type="text"
            id="courseName"
            v-model="course.name"
            required
          />

          <label for="courseCategory">หมวดหลักสูตร:</label>
          <input
            type="text"
            id="courseCategory"
            v-model="course.category"
            required
          />

          <label for="courseInstructor">วิทยากร:</label>
          <input
            type="text"
            id="courseInstructor"
            v-model="course.instructor"
            required
          />

          <label for="courseDays">จำนวนวัน:</label>
          <input
            type="number"
            id="courseDays"
            v-model="course.days"
            required
          />

          <button type="submit">{{ editIndex === -1 ? 'เพิ่มหลักสูตร' : 'อัปเดตหลักสูตร' }}</button>
        </form>
      </div>
    </div>
  </div>
</template>



<script setup>
import { reactive, ref, computed } from 'vue';
import jsPDF from 'jspdf';
import 'jspdf-autotable'; // Ensure this import is correct

const courses = ref([]);
const course = reactive({
  name: '',
  category: '',
  instructor: '',
  days: ''
});
const editIndex = ref(-1);
const showModal = ref(false);
const searchQuery = ref('');

const openModal = () => {
  resetForm();
  showModal.value = true;
};

const closeModal = () => {
  showModal.value = false;
};

const submitForm = () => {
  if (editIndex.value === -1) {
    courses.value.push({ ...course });
  } else {
    courses.value[editIndex.value] = { ...course };
  }
  closeModal();
};

const editCourse = (index) => {
  Object.assign(course, courses.value[index]);
  editIndex.value = index;
  showModal.value = true;
};

const resetForm = () => {
  course.name = '';
  course.category = '';
  course.instructor = '';
  course.days = '';
  editIndex.value = -1;
};

// Computed property to filter courses based on the search query
const filteredCourses = computed(() => {
  const query = searchQuery.value.toLowerCase();
  return courses.value.filter(course =>
    course.name.toLowerCase().includes(query) ||
    course.category.toLowerCase().includes(query) ||
    course.instructor.toLowerCase().includes(query)
  );
});


const exportToPDF = () => {
  const doc = new jsPDF();


  const tableData = filteredCourses.value.map((course, index) => [
    '',
    index + 1,
    course.name,
    course.category,
    course.instructor,
    course.days + ' วัน',
    '' 
  ]);


  doc.autoTable({
    head: [['เลือก', 'ลำดับ', 'ชื่อหลักสูตรฝึกอบรม', 'หมวดหลักสูตร', 'วิทยากร', 'จำนวนวัน', 'รายละเอียด']],
    body: tableData
  });


  doc.save('courses.pdf');
};
</script>




<style scoped>
#app {
  font-family: Arial, sans-serif;
  max-width: 1200px;
  width: 1200px;
  margin: 20px auto;
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 20px rgb(255, 255, 255);
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
  font-size: 28px;
  font-weight: 600;
}

.search-container {
  text-align: center;
  margin-bottom: 20px;
}

.search-container input {
  width: 100%;
  max-width: 300px;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 14px;
}

.button-container {
  text-align: center;
  margin-bottom: 20px;
}

button {
  padding: 10px 20px;
  background-color: #000000;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #333;
}

/* Table styling */
table {
  width: 800px;
  border-collapse: collapse;
  background-color: #fff;
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  overflow: hidden;
}

th, td {
  padding: 15px;
  text-align: left;
  font-size: 14px;
  color: #555;
}

th {
  background-color: #000000;
  color: white;
  font-weight: 600;
}

td {
  border-bottom: 1px solid #eee;
}

tr:hover {
  background-color: #f1f1f1;
}

input[type="checkbox"] {
  cursor: pointer;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  color: #000000;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 90%;
  max-width: 500px;
}

.close {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 20px;
  cursor: pointer;
}
</style>

