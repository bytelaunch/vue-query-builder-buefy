<template>
  <div class="vqb-group" :class="classObject">
    <div class="vqb-group-heading" :class="{ 'panel-heading': styled }">
      <div class="match-type-container">


        <b-field :label="labels.matchType">
          <b-select placeholder="Select a name" v-model="query.logicalOperator">
            <option v-for="(label, index) in labels.matchTypes" :key="index" :value="label.id">
              {{ label.label }}
            </option>
          </b-select>
        </b-field>


        <b-button type="is-danger"
                  icon-right="delete"
                  v-if="this.depth > 1"
                  @click="remove">Delete Group
        </b-button>

      </div>
    </div>

    <div class="vqb-group-body">
      <div class="rule-actions" style="padding-top:30px">


        <b-field grouped>
          <b-field>
            <b-select placeholder="Select a name" v-model="selectedRule">
              <option v-for="(rule, index) in rules" :key="index" :value="rule">{{ rule.label }}</option>
            </b-select>
          </b-field>

          <b-field>
            <div class="buttons">

              <b-button type="is-primary" @click="addRule" v-html="labels.addRule"></b-button>
              <b-button type="is-primary" outlined v-if="this.depth < this.maxDepth" @click="addGroup"
                        v-html="labels.addGroup"></b-button>
            </div>
          </b-field>
        </b-field>

      </div>

      <div class="children">
        <component
          v-for="(child, index) in query.children"
          :key="index"
          :is="child.type"
          :type="child.type"
          :query.sync="child.query"
          :ruleTypes="ruleTypes"
          :rules="rules"
          :rule="ruleById(child.query.rule)"
          :index="index"
          :maxDepth="maxDepth"
          :depth="depth + 1"
          :styled="styled"
          :labels="labels"
          v-on:child-deletion-requested="removeChild">
        </component>
      </div>
    </div>
  </div>
</template>

<script>
  import QueryBuilderRule from './QueryBuilderRule.vue'
  import deepClone from '../utilities.js'
  import Dialog from 'buefy'

  export default {
    name: 'query-builder-group',

    components: {
      Dialog,
      QueryBuilderRule
    },

    props: ['ruleTypes', 'type', 'query', 'rules', 'index', 'maxDepth', 'depth', 'styled', 'labels'],

    methods: {
      ruleById (ruleId) {
        var rule = null

        this.rules.forEach(function (value) {
          if (value.id === ruleId) {
            rule = value
            return false
          }
        })

        return rule
      },

      addRule () {
        let updated_query = deepClone(this.query)
        let child = {
          type: 'query-builder-rule',
          query: {
            rule: this.selectedRule.id,
            selectedOperator: this.selectedRule.operators[0],
            selectedOperand: typeof this.selectedRule.operands === 'undefined' ? this.selectedRule.label : this.selectedRule.operands[0],
            value: null
          }
        }
        // A bit hacky, but `v-model` on `select` requires an array.
        if (this.ruleById(child.query.rule).type === 'multi-select') {
          child.query.value = []
        }
        updated_query.children.push(child)
        this.$emit('update:query', updated_query)
      },

      addGroup () {
        let updated_query = deepClone(this.query)
        if (this.depth < this.maxDepth) {
          updated_query.children.push({
            type: 'query-builder-group',
            query: {
              logicalOperator: this.labels.matchTypes[0].id,
              children: []
            }
          })
          this.$emit('update:query', updated_query)
        }
      },

      remove () {
        this.$emit('child-deletion-requested', this.index)
      },

      removeChild (index) {
        let updated_query = deepClone(this.query)
        updated_query.children.splice(index, 1)
        this.$emit('update:query', updated_query)
      }
    },

    data () {
      return {
        selectedRule: this.rules[0]
      }
    },

    computed: {
      classObject () {
        var classObject = {
          'panel panel-default': this.styled,
        }

        classObject['depth-' + this.depth.toString()] = this.styled

        return classObject
      }
    }
  }
</script>
