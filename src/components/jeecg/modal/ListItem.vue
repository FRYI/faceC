<template>
  <a-auto-complete
    :data-source="dataSource"
    style="width: 500px"
    placeholder="input here"
    :filter-option="filterOption"
  />
</template>
<script>
  export default {
    data() {
      return {
        dataSource: ['Burns Bay Road', 'Downing Street', 'Wall Street'],
      };
    },
    methods: {
      filterOption(input, option) {
        return (
          option.componentOptions.children[0].text.toUpperCase().indexOf(input.toUpperCase()) >= 0
        );
      },
    },
  };
</script>
