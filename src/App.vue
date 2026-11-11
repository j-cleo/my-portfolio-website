<script setup>
  import Navbar from './components/Navbar.vue';
  import Landing from './components/Landing.vue';
  import Projects from './components/Projects.vue';
  import Tools from './components/Tools.vue';
  import Contact from './components/Contact.vue';
  import Footer from './components/Footer.vue';
</script>

<template>
  <Navbar />
  <div class="container-fluid">
    <Landing />
    <Projects />
    <Tools />
    <Contact />
  </div>
  <Footer />
</template>