<template>
  <div class="vqb-rule">
    <b-field grouped>

      <b-field :label="rule.label">
        <b-select placeholder="Select a name" v-if="typeof rule.operands !== 'undefined'"
                  v-model="query.selectedOperand">
          <option v-for="(operand, index) in rule.operands" :key="index">{{ operand }}</option>
        </b-select>
      </b-field>


      <b-field v-if="typeof rule.operators !== 'undefined' && rule.operators.length > 1">
        <b-select placeholder="Select a name"
                  v-model="query.selectedOperator">
          <option v-for="(operator, index) in rule.operators" v-bind:value="operator" :key="index">
            {{ operator }}
          </option>
        </b-select>
      </b-field>


      <b-field v-if="rule.inputType === 'text'">
        <b-input type="text" v-model="query.value" :placeholder="labels.textInputPlaceholder"></b-input>
      </b-field>

      <b-field v-if="rule.inputType === 'number'">
        <b-numberinput v-model="number"></b-numberinput>
      </b-field>


      <b-field label="Select a date" v-if="rule.inputType === 'date'">
        <b-datepicker
          v-model="query.value"
          :placeholder="labels.textInputPlaceholder"
          icon="calendar-today">
        </b-datepicker>
      </b-field>


      <template v-if="isCustomComponent">
        <component :value="query.value" @input="updateQuery" :is="rule.component"></component>
      </template>

      <div class="block" v-if="rule.inputType === 'checkbox'">
        <b-checkbox v-for="(choice, index) in rule.choices"
                    v-model="query.value"
                    :native-value="choice.value"
                    :key="index">
          {{choice.label}}
        </b-checkbox>
      </div>


      <div class="block" v-if="rule.inputType === 'radio'">
        <b-radio
          v-for="(choice, index) in rule.choices"
          v-model="query.value"
          :native-value="choice.value"
          :key="index">
          {{choice.label}}
        </b-radio>

      </div>


      <b-field v-if="rule.inputType === 'select'">
        <b-select placeholder="Select a name"
                  :multiple="rule.type === 'multi-select'"
                  v-model="query.value">
          <template v-for="(option, option_key) in selectOptions">
            <option v-if="!Array.isArray(option)" :value="option.value" :key="option_key">
              {{ option.label }}
            </option>
            <optgroup v-if="Array.isArray(option)" :label="option_key" :key="option_key">
              <option v-for="(sub_option, index) in option" :value="sub_option.value" :key="index">{{ sub_option.label
                }}
              </option>
            </optgroup>
          </template>
        </b-select>
      </b-field>


      <b-field>
      <b-button type="is-danger"
                icon-right="delete"
                @click="remove">Delete Rule
      </b-button>
      </b-field>
    </b-field>
  </div>
</template>

<script>
  import deepClone from '../utilities.js'
  import BField from 'buefy/src/components/field/Field'

  export default {
    name: 'query-builder-rule',
    components: {BField},
    props: ['query', 'index', 'rule', 'styled', 'labels'],

    beforeMount () {
      if (this.rule.type === 'custom-component') {
        this.$options.components[this.id] = this.rule.component
      }
    },

    methods: {
      remove: function () {
        this.$emit('child-deletion-requested', this.index)
      },
      updateQuery (value) {
        let updated_query = deepClone(this.query)
        updated_query.value = value
        this.$emit('update:query', updated_query)
      },
    },

    computed: {
      isCustomComponent () {
        return this.rule.type === 'custom-component'
      },

      selectOptions () {
        if (typeof this.rule.choices === 'undefined') {
          return {}
        }

        return this.rule.choices.reduce(function (groups, item, index) {
          let key = item['group']
          if (typeof key !== 'undefined') {
            groups[key] = groups[key] || []
            groups[key].push(item)
          } else {
            groups[index] = item
          }

          return groups
        }, {})
      },
    },

    mounted () {
      let updated_query = deepClone(this.query)

      // Set a default value for these types if one isn't provided already
      if (this.query.value === null) {
        if (this.rule.inputType === 'checkbox') {
          updated_query.value = []
        }
        if (this.rule.type === 'select') {
          updated_query.value = this.rule.choices[0].value
        }
        if (this.rule.type === 'custom-component') {
          updated_query.value = null
          if (typeof this.rule.default !== 'undefined') {
            updated_query.value = deepClone(this.rule.default)
          }
        }

        this.$emit('update:query', updated_query)
      }
    }
  }
</script>
