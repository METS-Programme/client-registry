<template>
  <footer class="footer">
    <div>
      <div>National Client Registry</div>
      <div>Copyright &copy; {{ currentYear }} Uganda Ministry of Health</div>
      <div>Supported by METS</div>
    </div>
  </footer>
</template>

<script>
export default {

  data() {
    return {
      currentDate: "",
      currentYear: new Date().getFullYear(),
    };
  },

  created() {
    this.getCurrentDate().then((date) => {
      this.currentDate = date
    });
  },

  methods: {
    async getCurrentDate() {
      const date = new Date();
      const options = { year: 'numeric', month: 'long', day: 'numeric' };
      return date.toLocaleDateString(undefined, options);
    },
  },
};
</script>

<style lang="scss" scoped>
.footer {
  bottom: 0;
  position: sticky;
  height: auto;
  padding: 10px;
  background-color: #f8f9fa;
  text-align: center;
}

.footer-content {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  gap: 2px;
}
</style>
