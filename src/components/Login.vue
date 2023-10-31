<template>
  <div>
    <h2>Логін</h2>
    <input v-model="input" type="text" placeholder="ім'я студента" required />
    <button @click="login">Увійти</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      input: '',
    };
  },
  methods: {
    login() {
      const studentName = this.input.trim();
      this.axios.get(`http://34.82.81.113:3000/students/name/${studentName}`).then((response) => {
               if((response.data === null) || (response.data.name == "CastError")) {
                   return
               }
               this.$store.commit('setUser', response.data)
               this.$router.push('/');
           })
       }
   }
}
</script>

<style scoped>
</style>
