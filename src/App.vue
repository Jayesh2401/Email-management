<template>
  <div>
    <h1>Email Management</h1>
    <EmailManager />
  </div>
</template>

<script>
import EmailManager from './components/EmailManager.vue';

export default {
  components: {
    EmailManager
  }
};
</script>

<style>
h1 {
  text-align: center;
}
</style>
