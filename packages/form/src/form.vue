<template>
  <form class="el-form" :class="[
    labelPosition ? 'el-form--label-' + labelPosition : '',
    { 'el-form--inline': inline }
  ]">
    <slot></slot>
  </form>
</template>
<script>
  import objectAssign from 'element-ui/src/utils/merge';

  export default {
    name: 'ElForm',

    componentName: 'ElForm',

    // 注入
    provide() {
      return {
        elForm: this
      };
    },

    props: {
      model: Object,
      rules: Object,
      labelPosition: String,
      labelWidth: String,
      labelSuffix: {
        type: String,
        default: ''
      },
      inline: Boolean,
      inlineMessage: Boolean,
      statusIcon: Boolean,
      showMessage: {
        type: Boolean,
        default: true
      },
      size: String,
      disabled: Boolean,
      validateOnRuleChange: {
        type: Boolean,
        default: true
      },
      hideRequiredAsterisk: {
        type: Boolean,
        default: false
      }
    },
    watch: {
      rules() {
        //  rules 属性改变后立即触发一次验证
        if (this.validateOnRuleChange) {
          this.validate(() => {});
        }
      }
    },
    data() {
      return {
        fields: []
      };
    },
    created() {
      // Input、Select等表单组件的兼容Form组件的关键
      this.$on('el.form.addField', (field) => {
        if (field) {
          this.fields.push(field);
        }
      });
      /* istanbul ignore next */
      this.$on('el.form.removeField', (field) => {
        if (field.prop) {
          this.fields.splice(this.fields.indexOf(field), 1);
        }
      });
    },
    methods: {
      // 对整个表单进行重置，将所有字段值重置为初始值并移除校验结果
      resetFields() {
        if (!this.model) {
          console.warn('[Element Warn][Form]model is required for resetFields to work.');
          return;
        }
        this.fields.forEach(field => {
          // 其实就是单独调用formItem的实例方法
          field.resetField();
        });
      },
      // 移除表单表单的校验结果
      clearValidate(props = []) {
        const fields = props.length
          ? (typeof props === 'string'
            ? this.fields.filter(field => props === field.prop)
            : this.fields.filter(field => props.indexOf(field.prop) > -1)
          ) : this.fields;
        fields.forEach(field => {
          // 其实就是单独调用formItem的实例方法
          field.clearValidate();
        });
      },
      // 对整个表单进行校验的方法
      validate(callback) {
        if (!this.model) {
          console.warn('[Element Warn][Form]model is required for validate to work!');
          return;
        }

        let promise;
        // 没有回调函数，则生成promise
        if (typeof callback !== 'function' && window.Promise) {
          promise = new window.Promise((resolve, reject) => {
            callback = function(valid) {
              valid ? resolve(valid) : reject(valid);
            };
          });
        }

        let valid = true;
        let count = 0;
        // 如果需要验证的fields为空，调用验证时立刻返回callback
        if (this.fields.length === 0 && callback) {
          callback(true);
        }
        let invalidFields = {};
        this.fields.forEach(field => {
          field.validate('', (message, field) => {
            if (message) {
              valid = false;
            }
            invalidFields = objectAssign({}, invalidFields, field);
            if (typeof callback === 'function' && ++count === this.fields.length) {
              callback(valid, invalidFields);
            }
          });
        });

        if (promise) {
          return promise;
        }
      },
      // 对部分表单字段进行校验的方法
      validateField(props, cb) {
        props = [].concat(props);
        const fields = this.fields.filter(field => props.indexOf(field.prop) !== -1);
        if (!fields.length) {
          confirm.warn('[Element Warn]please pass correct props!');
          return;
        }

        fields.forEach(field => {
          field.validate('', cb);
        });
      }
    }
  };
</script>
