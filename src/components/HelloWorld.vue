<template>
  <div>
    <v-toolbar flat>
      <v-toolbar-title class="paragraph">
        <span class="font-weight-bold">Opportunities</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-toolbar-title class="paragraph">
        <span class="font-weight-bold">Approximate savings</span>
      </v-toolbar-title>
    </v-toolbar>

    <v-data-table
      :items="opportunityAudits"
      class="elevation-5 mb-5"
      hide-actions
      hide-headers
    >
      <template v-slot:items="props">
        <td
          class="rect-container"
          v-if="getScore(props.item.score) > 40"
          >
          <div class="rect-element element-1 red"></div>
          <div class="rect-element element-2" style="
            position: absolute;
            width: 21px;
            height: 21px;
            border: 1px solid red;
            border-radius: 6px;"
          ></div>
        </td>
        <td
          class="rect-container"
          v-else
          >
          <div class="rect-element element-1 orange darken-1"></div>
          <div class="rect-element element-2" style="
            position: absolute;
            width: 21px;
            height: 21px;
            border: 1px solid orange;
            border-radius: 6px;"
          ></div>
        </td>
        <td>{{ props.item.title }}</td>
        <td
          class="red--text font-weight-normal"
          v-if="getScore(props.item.score) > 40"
          >{{ props.item.displayValue }}
          <v-progress-linear
            color="error"
            background-color="grey"
            background-opacity="0.3"
            :value=getScore(props.item.score)
          ></v-progress-linear>
        </td>
        
        <td
          class="warning--text font-weight-normal"
          v-else
          >{{ props.item.displayValue }}
          <v-progress-linear
            color="warning"
            background-color="grey"
            background-opacity="0.3"
            :value=getScore(props.item.score)
          ></v-progress-linear>
        </td>

        <td>
          <v-expansion-panel popout focusable>
            <v-expansion-panel-content expand-icon="arrow_drop_down">
              <template v-slot:header>
                <div></div>
              </template>
              <v-card>
                <v-card-text>{{ props.item.description }}</v-card-text>
              </v-card>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </td>
      </template>
    </v-data-table>
    
    <v-toolbar flat>
      <v-toolbar-title class="paragraph">
        <span class="font-weight-bold">Diagnostics</span>
      </v-toolbar-title>
      <v-toolbar-title class="paragraph">
        <span class="font-weight-light caption">-Details of the performance of your application</span>
      </v-toolbar-title>
    </v-toolbar>

    <v-data-table
      :items="diagnosticAudits"
      class="elevation-5 mb-5"
      hide-actions
      hide-headers
    >
      <template v-slot:items="props">
        <td class="rect-container">
          <div class="rect-element element-1 red"></div>
          <div class="rect-element element-2" style="
            position: absolute;
            width: 21px;
            height: 21px;
            border: 1px solid red;
            border-radius: 6px;"
          ></div>
        </td>
        <td>{{ props.item.title }}</td>
        
        <td
          class="error--text font-weight-normal text-xs-right"
          >{{ props.item.displayValue }}
        </td>

        <td>
          <v-expansion-panel popout focusable>
            <v-expansion-panel-content expand-icon="arrow_drop_down">
              <template v-slot:header>
                <div></div>
              </template>
              <v-card>
                <v-card-text>{{ props.item.description }}</v-card-text>
              </v-card>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </td>
      </template>
    </v-data-table>
    
    <v-toolbar flat>
      <v-toolbar-title class="paragraph">
        <span class="font-weight-bold">Successfull audits</span>
      </v-toolbar-title>
    </v-toolbar>

    <v-data-table
      :items="passedAudits"
      class="elevation-5"
      hide-actions
      hide-headers
    >
      <template v-slot:items="props">
        <td class="rect-container">
          <div class="rect-element element-1 green lighten-1"></div>
          <div class="rect-element element-2" style="
            position: absolute;
            width: 21px;
            height: 21px;
            border: 1px solid green;
            border-radius: 6px;"
          ></div>
        </td>
        <td>{{ props.item.title }}</td>
          
        <td
          class="green--text accent-3 font-weight-normal text-xs-right"
          >{{ props.item.displayValue }}
        </td>

        <td>
          <v-expansion-panel popout focusable>
            <v-expansion-panel-content expand-icon="arrow_drop_down">
              <template v-slot:header>
                <div></div>
              </template>
              <v-card>
                <v-card-text>{{ props.item.description }}</v-card-text>
              </v-card>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </td>
      </template>
    </v-data-table>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        response: {},
        audits: {},
        categories: {},
        idToGroup: {},
        opportunityAudits: [],
        diagnosticAudits: [],
        passedAudits: []
      }
    },
    methods: {
      showAsPassed(audit) {
        switch (audit.scoreDisplayMode) {
            case 'manual':
            case 'notApplicable':
              return true;
            case 'error':
            case 'informative':
              return false;
            case 'numeric':
            case 'binary':
            default:
              return Number(audit.score) >= 0.9;
        }
      },
      getScore(scoreValue) {
        return scoreValue * 50;
      }
    },
    computed: {
    },
    mounted() {
      fetch("https://www.googleapis.com/pagespeedonline/v5/runPagespeed/?url=https://habr.com/")
        .then(response => response.json())
        .then(data => {
          this.response = data;
          this.audits = this.response.lighthouseResult.audits;
          this.categories = this.response.lighthouseResult.categories.performance.auditRefs;

          this.categories.forEach(el => {
            this.idToGroup[el.id] = el.group;
          });
          
          this.opportunityAudits = Object.values(this.audits)
            .filter(audit => this.idToGroup[audit.id] === 'load-opportunities' && !this.showAsPassed(audit));

          this.diagnosticAudits = Object.values(this.audits)
            .filter(audit => this.idToGroup[audit.id] === 'diagnostics' && !this.showAsPassed(audit));

          this.passedAudits = Object.values(this.audits)
            .filter(audit => (this.idToGroup[audit.id] === 'load-opportunities' || this.idToGroup[audit.id] === 'diagnostics') &&
            this.showAsPassed(audit));
        })
        .catch(err => new Error(err));
    }
  }
</script>

<style>
  td {
    width: 50%;
  }

  .v-progress-linear {
    margin: 0;
    border-radius: 5px;
  }

  .v-expansion-panel {
    box-shadow: 0px 0px 0px 0px rgba(0,0,0,0), 0px 0px 0px 0px rgba(0,0,0,0), 0px 0px 0px 0px rgba(0,0,0,0);
  }

  .rect-container {
    width: 5%;
    position: relative;
  }

  .rect-element {
    width: 20px;
    height: 20px;
    border: 2px solid;
    border-radius: 5px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .element-1 {
    opacity: 0.5;
  }
</style>
