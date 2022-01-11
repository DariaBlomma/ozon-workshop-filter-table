<script>
import Common from '@/components/common';

export default {
  name: 'App',
  render(createElement) {
    return createElement (
      'div',
      {
        attrs: {
          id: 'app',
        },
      },
      [
        createElement(Common),
      ],
    );
  },
};
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
