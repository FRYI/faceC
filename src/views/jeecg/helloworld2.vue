<template>
  <div>
    hello world!
  </div>
</template>
<script>
  export default {
    data () {
      return {
        description: '表单page用于向用户收集或验证信息，基础表单常见于数据item较少的表单场景。',
        value: 1,

        // form
        form: this.$form.createForm(this),

      }
    },
    methods: {
      // handler
      handleSubmit (e) {
        e.preventDefault()
        this.form.validateFields((err, values) => {
          if (!err) {
            // eslint-disable-next-line no-console
            console.log('Received values of form: ', values)
          }
        })
      }
    }
  }
</script>