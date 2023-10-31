<template>
  <div>
    <h3>Кількість студентів: {{ studentsCount }}</h3>
    <h3>Пошук</h3>
    <input v-model="search" placeholder="Пошук за прізвищем" />

    <div>
      <h3>Додати нового студента</h3>
      <input v-model="newStudentName" placeholder="Ім'я студента" />
      <select v-model="newStudentGroup">
        <option v-for="groupOption in groupOptions" :value="groupOption">{{ groupOption }}</option>
      </select>
      <input v-model="newStudentMark" placeholder="Рік народження" />
      <input type="checkbox" v-model="newStudentIsDonePr" /> Практика
      <button @click="addStudent">Додати студента</button>
    </div>

    <table class="table">
      <thead>
        <tr>
          <th>ПІБ</th>
          <th>Група</th>
          <th>Рік народження</th>
          <th>Практика</th>
          <th>Дії</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="student in students"
          :key="student._id"
          :class="{ 'bg-warning': isMatched(student) }"
        >
          <td>
            <router-link :to="'/student-info/' + student._id">{{ student.name }}</router-link>
          </td>
          <td>{{ student.group }}</td>
          <td>{{ student.mark }}</td>
          <td>
            <input type="checkbox" v-model="student.isDonePr" />
          </td>
          <td>
            <a href="#" @click="deleteStudent(student._id)" v-show="getCurrentUser && student.group === getCurrentUser.group">Видалити</a>
            <span @click="editStudent(student)" class="edit-icon" v-show="getCurrentUser && student.group === getCurrentUser.group">✏️</span>
          </td>
        </tr>
      </tbody>
    </table>

    <div v-if="editingStudent">
      <h3>Редагувати студента</h3>
      <input v-model="editingStudentName" placeholder="Ім'я студента" />
      <select v-model="editingStudentGroup">
        <option v-for="groupOption in groupOptions" :value="groupOption">{{ groupOption }}</option>
      </select>
      <input v-model="editingStudentMark" placeholder="Рік народження" />
      <input type="checkbox" v-model="editingStudentIsDonePr" /> Практика
      <button @click="updateStudent(student)">Оновити</button>
      <button @click="cancelEdit">Скасувати</button>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';
import { useStore } from 'vuex';

export default {
  setup() {
    const store = useStore();
    const search = ref('');
    const newStudentName = ref('');
    const newStudentGroup = ref('');
    const newStudentMark = ref('');
    const newStudentIsDonePr = ref(false);
    const editingStudent = ref(null);
    const editingStudentName = ref('');
    const editingStudentGroup = ref('');
    const editingStudentMark = ref(0);
    const editingStudentIsDonePr = ref(false);

    const students = ref([]);

    const groupOptions = ["RPZ 20 1/9", "RPZ 20 2/9"];

    const fetchData = async () => {
      try {
        const response = await axios.get('http://34.82.81.113:3000/students');
        students.value = response.data;
        store.commit('setCount', students.value.length);
      } catch (error) {
        console.log(error);
      }
    };

    const studentsCount = computed(() => {
      return store.getters.getCount;
    });

    const isMatched = (student) => {
      return search.value && student.name.includes(search.value);
    };

    const addStudent = () => {
      const newStudent = {
        name: newStudentName.value,
        group: newStudentGroup.value,
        mark: parseInt(newStudentMark.value),
        isDonePr: newStudentIsDonePr.value,
      };

      axios.post('http://34.82.81.113:3000/students', newStudent)
        .then((response) => {
          console.log(response.data);
          students.value.push(response.data);
          store.commit('setCount', students.value.length);
        })
        .catch((error) => {
          console.log(error);
        });

      newStudentName.value = '';
      newStudentGroup.value = '';
      newStudentMark.value = '';
      newStudentIsDonePr.value = false;
    };

    const removeStudent = (id) => {
      axios.delete(`http://34.82.81.113:3000/students/${id}`)
        .then((response) => {
          console.log(response.data);
          students.value = students.value.filter((student) => student._id !== id);
          store.commit('setCount', students.value.length);
        })
        .catch((error) => {
          console.log(error);
        });
    };

    const editStudent = (student) => {
      editingStudent.value = student;
      editingStudentName.value = student.name;
      editingStudentGroup.value = student.group;
      editingStudentMark.value = student.mark;
      editingStudentIsDonePr.value = student.isDonePr;
      editingStudent._id = student._id;
    };

    const updateStudent = () => {
      const updatedStudent = {
        name: editingStudentName.value,
        group: editingStudentGroup.value,
        mark: parseInt(editingStudentMark.value),
        isDonePr: editingStudentIsDonePr.value,
      };

      axios.put(`http://34.82.81.113:3000/students/${editingStudent._id}`, updatedStudent)
        .then((response) => {
          console.log(response.data);
          const index = students.value.findIndex((student) => student._id === editingStudent._id);
          if (index !== -1) {
            students.value[index] = response.data;
          }
          editingStudent.value = null;
        })
        .catch((error) => {
          console.log(error);
        });
    };

    const cancelEdit = () => {
      editingStudent.value = null;
    };

    onMounted(() => {
      fetchData();
    });

    const getCurrentUser = computed(() => store.getters.getUser);

    return {
      search,
      newStudentName,
      newStudentGroup,
      newStudentMark,
      newStudentIsDonePr,
      students,
      isMatched,
      addStudent,
      removeStudent,
      editStudent,
      editingStudent,
      editingStudentName,
      editingStudentGroup,
      editingStudentMark,
      editingStudentIsDonePr,
      updateStudent,
      cancelEdit,
      studentsCount,
      groupOptions,
      getCurrentUser,
    };
  },
};
</script>

<style scoped>
  .edit-icon {
    cursor: pointer;
    margin-left: 20px;
  }
</style>




