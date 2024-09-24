<template>
    <div ref="lottieContainer" class="lottie-animation"></div>
  </template>
  
  <script>
  import lottie from 'lottie-web';
  
  export default {
    mounted() {
      this.$nextTick(() => {
        this.loadLottieAnimation();
      });
    },
    methods: {
      loadLottieAnimation() {
        lottie.loadAnimation({
          container: this.$refs.lottieContainer, 
          renderer: 'svg',
          loop: true,
          autoplay: true,
          path: '/assets/logoAnimation.json', 
        });
      },
    },
  };
  </script>
  
  <style scoped>
  .lottie-animation {
    width: 200px; 
    height: 200px;
    margin: 0 auto;
  }
  </style>
  